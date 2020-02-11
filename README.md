# SOFA

================================================================

Summary
-------

SOFA (Software Orchestrated Flash Array) is a log-structured flash array management system, which not only converts the random writes to sequential writes, but also fairly dispatches all writes IO requests across all SSDs. Also, the flash translation layer (FTL) in SOFA is designed on top of rather than below disk array management logic, so SOFA easily manages global resource including global wear leveling and garbage collection across SSDs, which means that SOFA is able to avoid the load and wear imbalance problems associated with existing flash disk array systems. More importantly, SOFA brings the benefits of linear scalability on throughput of writes with constituent SSDs. Besides, SOFA successfully makes throughput of random write reach that of sequential write.

The advantage of SOFA in comparison with standard Raid5 protection.
* SOFA supports 1M random write IOPS with 20 SSDs, which runs up 10 times faster than standard Raid5 protection.
* SOFA significantly extends 1.8 times longer lifetime than standard Raid5 protection. \
    9 random writes in standard Raid5 protection need 18 read and 18 write IO requests. Regarding Sofa, 9 random writes only need 9 write IO requests and 1 XOR write request. Compared with standard Raid5, lifetime prolonging in SOFA is 1.8 (18/10)  times longer. 



The Features of Commercial or Primary Version 
-------

For primary or commercial version, our SOFA has many features including SOFA over ISCSI with 1M random write IOPS, write limit, crash recovery, scale up, RAID6 protection, volume manager and high availability. 

With respect to SOFA over ISCSI, when user accesses SOFA over ISCSI from remote machines, SOFA still reaches 1M random write IOPS.

About write limit feature, users can set up the write limit for each protection group. When one protection group reaches the write limit, our system sends email notification and transfers write requests to other available groups. 

Regarding crash recovery feature, when SSDs fail and system also crashes, after rebooting your machine and restarting SOFA, SOFA can rebuild data successfully for failed SSDs and can restore system to its normal state.

For scale up feature, if there are more spare disks in your machine, SOFA can be scaled up on-the-fly at runtime with more groups and with more space in the block device. 

In respect of RAID6 protection feature, SOFA tolerates two failed disks in the whole system. For example, when 2 SSDs fail in 1 protection group, this protection group still can serve Read IO. Also, SOFA will transfer Write IO to another protection group. Besides, under RAID6 protection, SOFA still reaches 900K random write IOPS without obviously degrading its performance. 

With regard to volume manager (VM) feature, SOFA supports creating volumes, taking snapshots and cloning volumes. Also, logical volumes on SOFA can be thinly provisioned, so that the amount of used space in a volume is much smaller than the amount of allocated space in a volume.

Concerning high availability (HA) feature, SOFA adopt active-active mode, so the incoming IO can be spread  evenly out on two SOFA servers. If one crashes, the other can continue to serve IO requests without any downtime. So, high availability brings more reliability of SOFA.

# Hardware Support
---
CPU: Intel CPU with at least 10 virtual CPUs and with 2.8GHz each \
Memory: 64G \
Motherboard: Supermicro X10DRU-i+ version 1.02A \
HBA card: LSI Logic / Symbios Logic SAS3008 PCI-Express Fusion-MPT SAS-3 \
(PS. For LSI SAS3008, we need 3 HBA cards to support 24 SDDs.) \
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
At the first time before you start sofa, you must setup the setting file in /usr/sofa/config/sofa_config.xml. Later, if you would like to change your setting files, you should update setting file in /usr/`data`/sofa/config/sofa_config.xml directly.

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
        <value>3</value>
        <setting>b,c,d</setting>
    </property>
.......................................
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
    
         Check how many SSDs there are in your system
         
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
          
         In my system there are 20 available SSDs from sdb to sdu except sda used for operation system. So, I assign 20 SSDs to SOFA with 2 protection groups, which means each group is assigned with 10 SSDs.                              
         
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
    
         Check how many vcores there are in your machine. 0~39 vcores and 40 vcores in total in my machine. 
          
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
        
         Know these vcores are located on which physical cores. In my machine, physical CPU 0 has 0 - 9 and 20 - 29 vcores. Physical CPU 1 has 10 - 19 and 30 - 39 vcores. [Notice] Please don't use the first vcores in any physical machine. In my case, 0 is the first vcore on physical CPU 0 and 10 is the first vcore on physical CPU 1. So, vocre 0 and 10 are not assigned to SOFA.
         
         ![](https://i.imgur.com/ayxOPBr.jpg)

         Given SOFA performance, please assign vcores which is located in the same physical CPU. And, in default the ratio of lfsm_io_thread to lfsm_bh_thread for SOFA is 7:3 or 8:4.
         
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
        
    - List my major sofa_config.xml setting
    
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
                <property>
                    <name>lfsm_only</name>
                    <value>1</value>
                </property>                
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
            .......................................
            .......................................
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
