# SOFA

================================================================

Summary
-------
Virtualization technology could provide a unique benefit to protect any legacy application systems from hardware failures. The reliability of virtual machines running on virtualized servers is not only threatened by hardware failures beneath the whole virtual infrastructure, but also nosy hypervisors that essentially support virtual machines cannot be trusted.

We develop the opensource tool, Cuju, which is a virtualization based fault tolerance technique with epoch-based synchronization. There are several performance optimization technologies are applied by Cuju, including a non-stop/pipelined, continuously migration, dirty tracking for guest virtual memory/virtual device status, and eliminate data transfer between QEMU and KVM.

Cuju shows that these optimizations have greatly saved the processor usage, synchronization bandwidth and have significantly improved VM network throughput and latency at the same time.

For more information see: https://cuju-ft.github.io/cuju-web/home.html

# Hardware Support
---
CPU: Intel CPU with at least 10 virtual cpus and with 2.8GHz each \
Memory: 64G \
Motherboard: Supermicro X10DRU-i+ version 1.02A \
HBA card: LSI Logic / Symbios Logic SAS3008 PCI-Express Fusion-MPT SAS-3 \
(PS. For LSI SAS3008, we need 3 HBA cards for 24 SDDs.) \
SSDs: They must `support the trim command` including SanDisk SDSSDH3 and SanDisk Ultra II 00RL.


# Sofa Install Guide
## The Environment Prepare
---
- Install OS. centos 7.0 ~ 7.4
- Install necessary packages for compiling sofa sources 
```
$ yum -y install gcc
$ yum -y install gcc-c++
$ uname -r
3.10.0-327.10.1.el7.tb.x86_64
```
Google and download kernel-devel-$(uname -r).rpm \
[Notice] Install kernel-devel-$(uname -r).rpm. $(uname -r) must exactly match your kernel version. \
In my machine my kernel development package is kernel-devel-3.10.0-327.10.1.el7.tb.x86_64.rpm. 
```
$ rpm -ivh kernel-devel-3.10.0-327.10.1.el7.tb.x86_64.rpm
$ yum -y install json-c-devel 
$ yum –y install json-devel 
$ yum -y install rpm-build
$ yum -y install libaio
$ yum -y install libaio-devel
```

## Build SOFA
---
* Clone SOFA on your folder from Github
```
$ cd /home/$(whoami)
$ git clone https://github.com/sofa/sofa.git
```
* Compile SOFA and build SOFA-0.1-4.x86_64.rpm
```
$ cd /home/$(whoami)/sofa
$ sh ./gen_sofa_package.sh
./gen_sofa_package.sh platform=linux verion=0.1 release=4 tmp_dir=/home/sofa/SOFA_Release/
./gen_sofa_package.sh start to generate sofa release packages
.......................................
.......................................
+ exit 0
RPM Preparation Done!
Generate sofa packages success
```
* Deploy SOFA
```
$ cd packages
$ sh ./undeploy_sofa.sh
$ sh ./deploy_sofa.sh 
./deploy_sofa.sh: deply SOFA version=0.1 release=4
Preparing...                          ################################# [100%]
Updating / installing...
.......................................
.......................................
SOFA_Release/lfsmdr.ko
SOFA_Release/rfsioctl
SOFA_Release/noop2.ko
./deploy_sofa.sh: deploy SOFA done
```
* Check SOFA folder
```
$  ls -l /usr/sofa
drwxr-xr-x 4 root root      209 Feb  5 15:59 bin
drwxr-xr-x 2 root root       29 Feb  5 15:59 config
-rwxr-xr-x 1 root root 13455600 Feb  5 15:59 lfsmdr.ko
drwxr-xr-x 2 root root       84 Feb  5 15:59 lib
-rwxr-xr-x 1 root root   261936 Feb  5 16:00 noop2.ko
-rwxr-xr-x 1 root root    13160 Feb  5 15:59 rfsioctl
-rwxr-xr-x 1 root root  4860512 Feb  5 15:59 sofa.ko
-rwxr-xr-x 1 root root    62832 Feb  5 15:59 sofa_daemon
```

## Configure sofa_config.xml
---
* SOFA settings in sofa_config.xml \
At the first time before you start sofa, you must setup config file in /usr/sofa/config/sofa_config.xml. Later, if you would like to change your config files, you should update config file in /usr/`data`/sofa/config/sofa_config.xml directly.

```
$ vim /usr/sofa/config/sofa_config.xml
<?xml version="1.0"?>
<?xml-stylesheet type="text/xsl" href="configuration.xsl"?>
<configuration>
    <property>
        <name>lfsm_reinstall</name>
        <value>1</value>
    </property>
    <property>
        <name>lfsm_cn_ssds</name>
        <value>2</value>
        <setting>b-c</setting>
    </property>
.......................................
.......................................
    <property>
        <name>lfsm_gc_on_lower_ratio</name>
        <value>125</value>
    </property>
</configuration>
```
- Major settings definition in sofa_config.xml
    - lfsm_reinstall   : When a user starts SOFA, if SOFA keeps existing data or clears all data. \
          - value      : 0: keep existing data; 1: clear all data.
          
    - lfsm_cn_ssds     : Assign which SSDs to SOFA   \
          - value      : How many SSDs Sofa uses. \
          - settings   : Assign which SSDs to SOFA. 
          
    - cn_ssds_per_hpeu : Indicate how many SSDs there are in a protection group.  \
                         For Raid5 protection mode:  3 <= cn_ssds_per_hpeu <= 16 
                         
    - lfsm_cn_pgroup   : Indicates How many protection group SOFA uses.  \
                         For Sofa, lfsm_cn_ssds >= cn_ssds_per_hpeu * lfsm_cn_prgroup. \
                         The number of spare disks = lfsm_cn_ssds - ( cn_ssds_per_hpeu * lfsm_cn_prgroup ) 
                         
    - lfsm_io_thread   : Assign specific vcores to SOFA's IO threads \
          - value      ：How many vcores SOFA uses for IO thread. Default value: 7 or 8. \
          - settings   : Vcores ID. Please don't use the first vcore of physical CPU. 
                         
    - lfsm_bh_thread   : Assign specific vcores to SOFA's bottom half threads. \
          - value      : How many vcores SOFA uses for IO's bottom half threads. Default value: 3 or 4. \
          - settings   : Vcores ID. Please don't use the same vcore which lfsm_io_threads lists. 
                         
    - hba_intr_name    : Assign specifc vcores to HBA's interrupt handler \
          - value      : Get the prefix of HBA's interrupt name in /proc/interrupt \
          - settings:  : Vcores ID. Please use independent vcore and don't use the same vcore which lfsm_io_thread and lfsm_bh_thread list.                
          
* Configure SOFA settings \
    Step1. Assign lfsm_cn_ssds, cn_ssds_per_hpeu and lfsm_cn_pgroup \
    Step2. Assign vcores of lfsm_io_thread and lfsm_bh_thread to SOFA. \
    Step3. Assign vocres to HBA's interrupt handler.     

    - Step1. Assign lfsm_cn_ssds, cn_ssds_per_hpeu and lfsm_cn_pgroup                 
    
         Check how many ssds there are in your system
         
            ```
            $ lsblk
            NAME   MAJ:MIN RM   SIZE RO TYPE MOUNTPOINT
            sda      8:0    0 931.5G  0 disk 
            |-sda1   8:1    0   500M  0 part /boot
            |-sda2   8:2    0  31.4G  0 part [SWAP]
            |-sda3   8:3    0    50G  0 part /
            |-sda4   8:4    0     1K  0 part 
             -sda5   8:5    0 849.6G  0 part /home
            sdb      8:16   0 447.1G  0 disk 
            sdc      8:32   0 447.1G  0 disk 
            sdd      8:48   0 447.1G  0 disk 
            sde      8:64   0 447.1G  0 disk 
            sdf      8:80   0 447.1G  0 disk 
            sdg      8:96   0 447.1G  0 disk 
            sdh      8:112  0 447.1G  0 disk 
            sdi      8:128  0 447.1G  0 disk 
            sdj      8:144  0 447.1G  0 disk 
            sdk      8:160  0 447.1G  0 disk 
            sdl      8:176  0 447.1G  0 disk 
            sdm      8:192  0 447.1G  0 disk 
            sdn      8:208  0 447.1G  0 disk 
            sdo      8:224  0 447.1G  0 disk 
            sdp      8:240  0 447.1G  0 disk 
            sdq     65:0    0 447.1G  0 disk 
            sdr     65:16   0 447.1G  0 disk 
            sds     65:32   0 447.1G  0 disk 
            sdt     65:48   0 447.1G  0 disk 
            sdu     65:64   0 447.1G  0 disk
            ```          
          
         In my system there are 20 available ssds from sdb to sdu except sda used for operation system. So, I assign 20 SSDs to SOFA with 2 protecton groups, which means each group is assigned 10 SSDs.                              
         
            ```
            <property>
                <name>lfsm_cn_ssds</name>
                <value>20</value>
                <setting>b,c,d,e,f,g,h,i,j,k,l,m,n,o,p,q,r,s,t,u</setting>
            </property>
            <property>
                <name>cn_ssds_per_hpeu</name>
                <value>10</value>
            </property>
            <property>
                <name>lfsm_cn_pgroup</name>
                <value>2</value>
            </property> 
            ```

    - Step2. Assign vcores of lfsm_io_thread and lfsm_bh_thread to SOFA
    
         Check how many vcores there are in your machine. 0~39 vcores and total 40 vcores in my machine. 
          
            ```
            $ cat /proc/cpuinfo | grep processor
            processor	: 0
            processor	: 1
            processor	: 2
            .......................................
            .......................................
            processor	: 37
            processor	: 38
            processor	: 39
            ```
        
         Know these vcores are located on which physical cores. In my machine, physical cpu 0 has 0 - 9 and 20 - 29 vcores. Physical cpu 1 has 10 - 19 and 30 - 39 vcores. [Notice] Please don't use the first vcores in any physical machine. In my case, 0 is the first vcore on physical cpu 0 and 10 is the frist vcore on physcial cpu 1. So, vocre 0 and 10 are not assigned to SOFA.
            ![](https://i.imgur.com/ayxOPBr.jpg)

         Given SOFA performance, please assign vcores which is located in the same physical CPU. And, in default SOFA prefers to assign 7:3 or 8:4.  (lfsm_io_thread:lfsm_bh_thread) 
         
            ```
            <property>
                <name>lfsm_io_thread</name>
                <value>7</value>
                <setting>1,2,3,4,5,6,7</setting>
            </property>
            <property>
                <name>lfsm_bh_thread</name>
                <value>3</value>
                <setting>8,9,20</setting>
            </property>
            ```
        
    - Step3. Assign vocres to HBA's interrupt handler
        
         Check your HBA card and get `bus:device.function` of HBA card. In my computer, there are 3 HBA cards and the frist hba card lists `02:00.0`. 
         
            ```
            $ lspci   | grep LSI
            02:00.0 Serial Attached SCSI controller: LSI Logic / Symbios Logic SAS3008 PCI-Express Fusion-MPT SAS-3 (rev 02)
            03:00.0 Serial Attached SCSI controller: LSI Logic / Symbios Logic SAS3008 PCI-Express Fusion-MPT SAS-3 (rev 02)
            83:00.0 Serial Attached SCSI controller: LSI Logic / Symbios Logic SAS3008 PCI-Express Fusion-MPT SAS-3 (rev 02)
            ```        
        
         Get your HBA card driver by bus:device.function.
         
            ```
            $ lspci -v -s 02:00.0  | grep "Kernel driver"
            Kernel driver in use: mpt3sas
            ```
        
         Get the prefix of interrupt handler for HBA cards. In my machine, my interrupt handlers are mpt3sas0, mpt3sas1 and mpt3sas2. 
         
            ```
            $  cat /proc/interrupts  | grep mpt3sas  |  awk  '{ print $NF }'
            mpt3sas0-msix0
            mpt3sas0-msix1
            mpt3sas0-msix2
            mpt3sas0-msix3
            .......................................
            mpt3sas1-msix0
            mpt3sas1-msix1
            mpt3sas1-msix2
            mpt3sas1-msix3
            ....................................... 
            mpt3sas2-msix0
            mpt3sas2-msix1
            mpt3sas2-msix2
            .......................................            
            ```
        
         Assign vcore and interrupt handler's prefix to hba_intr_name proterty of sofa_config.xml. And I assign vcores which is located on the same physical CPU as vcores of lfsm_io_thread and lfsm_bh_thread are.          
         
            ```
            <property>
                <name>hba_intr_name</name>
                <value>mpt3sas0</value>
                <setting>21</setting>
            </property>
            <property>
                <name>hba_intr_name</name>
                <value>mpt3sas1</value>
                <setting>22</setting>
            </property>
            <property>
                <name>hba_intr_name</name>
                <value>mpt3sas2</value>
                <setting>23</setting>
            </property>
            ```
        
    - List all sofa_config.xml setting
    
            ```
            <?xml version="1.0"?>
            <?xml-stylesheet type="text/xsl" href="configuration.xsl"?>
            <configuration>
                <property>
                    <name>lfsm_reinstall</name>
                    <value>1</value>
                </property>
                <property>
                    <name>lfsm_cn_ssds</name>
                    <value>4</value>
                    <setting>b,c,d,e</setting>
                </property>
                <property>
                    <name>cn_ssds_per_hpeu</name>
                    <value>2</value>
                </property>
                <property>
                    <name>lfsm_cn_pgroup</name>
                    <value>2</value>
                </property>
                <property>
                    <name>lfsm_only</name>
                    <value>1</value>
                </property>
                <property>
                    <name>lfsm_raid6</name>
                    <value>1</value>
                </property>
                <property>
                    <name>lfsm_io_thread</name>
                    <value>10</value>
                    <setting> 11,12,13,14,15,16,17,18,19,30 </setting>
                </property>
                <property>
                    <name>lfsm_bh_thread</name>
                    <value>5</value>
                    <setting> 31,32,33,34,35 </setting>
                </property>
            .......................................
            .......................................
                <property>
                    <name>hba_intr_name</name>
                    <value>mpt3sas0</value>
                    <setting>19,20</setting>
                </property>
                <property>
                    <name>hba_intr_name</name>
                    <value>mpt3sas1</value>
                    <setting>21,22</setting>
                </property>
                <property>
                    <name>hba_intr_name</name>
                    <value>mpt3sas2</value>
                    <setting>23</setting>
                </property>
            </configuration>
            ```

## Run And Stop SOFA
---
* Run SOFA
```
$ sh /usr/sofa/bin/start-sofa.sh
```
* Check if SOFA services are available 
    - check if /dev/lfsm exists
    - check kernel log with specific message
```
$ lsblk
.......................................
sdw     65:96   0 447.1G  0 disk 
lfsm   252:0    0   1.3T  0 disk
$ dmesg | grep "main INFO initial all sofa components done"
[239305.799428] [SOFA] main INFO initial all sofa components done
```



* Stop SOFA
```
$ sh /usr/sofa/bin/stop-sofa.sh
```

## Uninstall SOFA
---
* After stopping SOFA, you can use following commands to remove SOFA in your system.
```
$ sh /home/$(whoami)/sofa/packages/undeploy_sofa.sh
```






*A recommended topology below:*
![](https://i.imgur.com/ayxOPBr.jpg)

* If you only have one or two machine, you can reference this setting.

*Another recommended topology below:*
![](https://i.imgur.com/38d0kzJ.png)

- Open the Intel virtualization support (VT-x) in your bios.
- Install OS in all nodes: [Ubuntu-16.04-desktop-amd64.iso (Ubuntu 16.04.0)](https://drive.google.com/file/d/0B9au9R9FzSWKUjRZclBXbXB0eEk/view)
- Install related packages in all nodes
```
 $ sudo apt-get update
 $ sudo apt-get install ssh vim gcc make gdb fakeroot build-essential \
kernel-package libncurses5 libncurses5-dev zlib1g-dev \
libglib2.0-dev qemu xorg bridge-utils openvpn vncviewer \
libssl-dev libpixman-1-dev nfs-common git
```
- Set up the bridge and network environment 
    - You can follow our recommended topology to set up the network environment 
    - The example of network interfaces set up below (edit your `/etc/network/interfaces`):
- NFS node
```
auto lo
iface lo inet loopback

auto eth0
iface eth0 inet static
address 192.168.11.1
netmask 255.255.255.0
gateway 192.168.11.254
dns-nameservers 8.8.8.8 
```

eth0 is your physical NIC name, please modify it according to your actual NIC name


- Primary node

```
auto lo
iface lo inet loopback

auto br0
iface br0 inet static
bridge_ports eth0
bridge_maxwait 0
address 192.168.11.2
netmask 255.255.255.0
gateway 192.168.11.254
dns-nameservers 8.8.8.8

auto eth0
iface eth0 inet static
address 0.0.0.0

auto eth1
iface eth1 inet static
address 192.168.111.1
netmask 255.255.255.0
```

- Backup node
```
auto lo
iface lo inet loopback

auto br0
iface br0 inet static
bridge_ports eth0
bridge_maxwait 0
address 192.168.11.3
netmask 255.255.255.0
gateway 192.168.11.254
dns-nameservers 8.8.8.8

auto eth0
iface eth0 inet static
address 0.0.0.0

auto eth1
iface eth1 inet static
address 192.168.111.2
netmask 255.255.255.0 
```

- Build the high-speed connections (ex. 10G NIC) with Primary and Backup nodes by the `eth1`

- After editing these network interfaces, type `/etc/init.d/networking restart` or `reboot`

### NFS Node Setup

- Install the NFS service (Network FileSystem) in NFS node; then create a NFS folder placing the VM image
```
 $ sudo apt-get install nfs-kernel-server
```
- Insert this line in `/etc/exports` to add your NFS folder: 
```
 /home/[your username]/nfsfolder *(rw,no_root_squash,no_subtree_check) 
```
- After editing `/etc/exports`, type `/etc/init.d/nfs-kernel-server restart` or `reboot`

- Go to your nfs folder, then download [Cuju](https://github.com/Cuju-ft/Cuju) and build a VM image file (or download our [Ubuntu-16.04 VM image](https://drive.google.com/file/d/0B9au9R9FzSWKNjZpWUNlNDZLcEU/view?usp=sharing) file, the `account/password` is `root/root`), they will be synced with Primary and Backup node.

### Primary and Backup Node Setup
- Mount the NFS folder
```
$ sudo mkdir /mnt/nfs
$ sudo mount -t nfs 192.168.11.1:/home/[your username]/nfsfolder /mnt/nfs
```
## Build Cuju
---
* Clone Cuju on your NFS folder from Github
```
$ cd /mnt/nfs
$ git clone https://github.com/Cuju-ft/Cuju.git
```
* Configure & Compile Cuju-ft

```
$ cd Cuju
$ ./configure --enable-cuju --enable-kvm --disable-pie --target-list=x86_64-softmmu
$ make clean
$ make -j8
```

* Configure, Compile & insmod Cuju-kvm module `*1` `*2`

```
$ cd Cuju/kvm
$ ./configure
$ make clean
$ make -j8
$ ./reinsmodkvm.sh
```
P.S.
>`*1` If you meet `error: incompatible type for argument 5 of '__get_user_pages_unlocked'`, you can use this patch:
>```
>$ cd Cuju
>$ patch -p1 < ./patch/__get_user_pages_unlocked.patch
>```
>
>`*2` If you meet `error: implicit declaration of function 'use_eager_fpu' [-werror=implicit-function-declaration]`, you can use this patch:
>```
>$ cd Cuju
>$ patch -p1 < ./patch/use_eager_fpu.patch
>```

Execute Cuju
-------
* Before launching your VM, you should update kvm module in Primary and Backup nodes: 
```
$ cd /mnt/nfs/Cuju/kvm
$ ./reinsmodkvm.sh
```

* Boot VM (on Primary Host)
* ```runvm.sh```

```
sudo ./x86_64-softmmu/qemu-system-x86_64 \
-drive if=none,id=drive0,cache=none,format=raw,file=/mnt/nfs/Ubuntu20G-1604.img \
-device virtio-blk,drive=drive0 \
-m 1G -enable-kvm \
-net tap,ifname=tap0 -net nic,model=virtio,vlan=0,macaddr=ae:ae:00:00:00:25 \
-cpu host \
-vga std -chardev socket,id=mon,path=/home/[your username]/vm1.monitor,server,nowait -mon chardev=mon,id=monitor,mode=readline

```

You need to change the guest image path (`file=/mnt/nfs/Ubuntu20G-1604.img`) and monitor path (`path=/home/[your username]/vm1.monitor`) for your environment


* Use VNC to see the console

```
$ vncviewer :5900 &
```

The default `account/password` is `root/root` if you use we provide guest image

* Start Receiver (on Backup Host)
* ```recv.sh```

```
sudo x86_64-softmmu/qemu-system-x86_64 \
-drive if=none,id=drive0,cache=none,format=raw,file=/mnt/nfs/Ubuntu20G-1604.img \
-device virtio-blk,drive=drive0 \
-m 1G -enable-kvm \
-net tap,ifname=tap1 -net nic,model=virtio,vlan=0,macaddr=ae:ae:00:00:00:25 \
-vga std -chardev socket,id=mon,path=/home/[your username]/vm1r.monitor,server,nowait -mon chardev=mon,id=monitor,mode=readline \
-cpu host \
-incoming tcp:0:4441,ft_mode
```

* You need to follow Boot VM script to change the related parameter or you can use following script to replace Receiver start script (if your VM start script is runvm.sh)
* ```recv.sh```

```
sed -e 's/mode=readline/mode=readline -incoming tcp\:0\:4441,ft_mode/g' -e 's/vm1.monitor/vm1r.monitor/g' -e 's/tap0/tap1/g' ./runvm.sh > tmp.sh
chmod +x ./tmp.sh
./tmp.sh
```

* After VM boot and Receiver ready, you can execute following script to enter FT mode
* ```ftmode.sh```
```
sudo echo "migrate_set_capability cuju-ft on" | sudo nc -U /home/[your username]/vm1.monitor
sudo echo "migrate -c tcp:192.168.111.2:4441" | sudo nc -U /home/[your username]/vm1.monitor
```
You need to change the ip address and port (`tcp:192.168.111.2:4441`) for your environment, this is Backup Host's IP
And change the monitor path (`/home/[your username]/vm1.monitor`) for your environment

* If you successfully start Cuju, you will see the following message show on Primary side:
![](https://i.imgur.com/nUdwKkB.jpg)

* If you want to test failover
 You can `kill` or `ctrl-c` VM on the Primary Host
![](https://i.imgur.com/JWIhtDz.png)

* You will need new session with vncviewer:
   * If you have Primary Host and Backup Host, execute on Backup Host: <br>`$ vncviewer :5900 &`
   * If you only have Primary Host with two VM: <br>`$ vncviewer :5901 &`
