# SOFA

================================================================

Summary
-------
Virtualization technology could provide a unique benefit to protect any legacy application systems from hardware failures. The reliability of virtual machines running on virtualized servers is not only threatened by hardware failures beneath the whole virtual infrastructure, but also nosy hypervisors that essentially support virtual machines cannot be trusted.

We develop the opensource tool, Cuju, which is a virtualization based fault tolerance technique with epoch-based synchronization. There are several performance optimization technologies are applied by Cuju, including a non-stop/pipelined, continuously migration, dirty tracking for guest virtual memory/virtual device status, and eliminate data transfer between QEMU and KVM.

Cuju shows that these optimizations have greatly saved the processor usage, synchronization bandwidth and have significantly improved VM network throughput and latency at the same time.

For more information see: https://cuju-ft.github.io/cuju-web/home.html

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
Notice. Install kernel-devel-XXX.rpm. XXX must exactly match your kernel version. \
In this case the necessary rpm is 3.10.0-327.10.1.el7.tb.x86_64.rpm. 
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
* Configure /usr/sofa/config/sofa_config.xml \
At the first time, before you start sofa, you must setup config file in /usr/sofa/config/sofa_config.xml. Later, if you would like to change your config files, you should update config file in /usr/`data`/sofa/config/sofa_config.xml.

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
- Settings definition in sofa_config.xml
    - lfsm_reinstall   : When starting SOFA, if sofa keeps original data or clears data out. \
          - value      : 0: keep original data, 1: clear out all data.
    - lfsm_cn_ssds     : Assign which ssds to SOFA  \
          - value      : SOFA use how many ssds.
          - settings   : Assign which ssds to SOFA. 
    - cn_ssds_per_hpeu :   \
          - value      :           
    - lfsm_cn_pgroup   :   \
          - value      : SOFA uses how many protection group.
    - lfsm_io_thread   :   \ 
          - value   
          - settings   : 
    - lfsm_bh_thread   :   \
          - value      : 
          - settings   :        
    - hba_intr_name    :   \
          - value      :
          - settings:  :           
          
          
       
       
       
          
          
          
     If we assign 4 ssds including sdb,sdc,sde,sdd to SOFA, the config should be set up by \
```     
<name>lfsm_cn_ssds</name>
<value>4</value>
<setting>b,c,d,e</setting>
```

 
lfsm_cn_ssds:


* Major settings in sofa_config.xml


* Check how many ssds there are in your system.
```
$ lsblk
NAME   MAJ:MIN RM   SIZE RO TYPE MOUNTPOINT
sda      8:0    0 931.5G  0 disk 
|-sda1   8:1    0   500M  0 part /boot
|-sda2   8:2    0  31.4G  0 part [SWAP]
|-sda3   8:3    0    50G  0 part /
|-sda4   8:4    0     1K  0 part 
`-sda5   8:5    0 849.6G  0 part /home
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
In my system there are 20 available ssds from sdb to sdu except sda used for operation system.


## Run And Stop SOFA
---
* Run SOFA and generated /dev/lfsm block device
```
$ sh /usr/sofa/bin/start-sofa.sh
$ lsblk
.......................................
sdw     65:96   0 447.1G  0 disk 
lfsm   252:0    0   1.3T  0 disk
```
* Stop SOFA
```
$ sh /usr/sofa/bin/stop-sofa.sh
```


*A recommended topology below:*
![](https://i.imgur.com/DuKZweZ.png)

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
