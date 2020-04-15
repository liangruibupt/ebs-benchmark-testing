# m5d.8xlarge 32 128 2 x 600 NVMe SSD Raid0

#  test 1, without raid
```bash
##Use the lsblk command to view your available disk devices and their mount points
[ec2-user@ip-10-0-0-142 ~]$ sudo lsblk
NAME          MAJ:MIN RM   SIZE RO TYPE MOUNTPOINT
nvme1n1       259:0    0 558.8G  0 disk
nvme2n1       259:1    0 558.8G  0 disk
nvme0n1       259:2    0    20G  0 disk
├─nvme0n1p1   259:3    0    20G  0 part /
└─nvme0n1p128 259:4    0     1M  0 part

##Determine whether there is a file system on the volume
[ec2-user@ip-10-0-0-142 ~]$ sudo file -s /dev/nvme1n1
/dev/nvme1n1: data
[ec2-user@ip-10-0-0-142 ~]$ sudo file -s /dev/nvme2n1
/dev/nvme2n1: data


##use the mkfs -t command to create a file system on the volume
[ec2-user@ip-10-0-0-142 ~]$  sudo mkfs -t xfs /dev/nvme1n1
meta-data=/dev/nvme1n1           isize=512    agcount=4, agsize=36621094 blks
         =                       sectsz=512   attr=2, projid32bit=1
         =                       crc=1        finobt=1, sparse=0
data     =                       bsize=4096   blocks=146484375, imaxpct=25
         =                       sunit=0      swidth=0 blks
naming   =version 2              bsize=4096   ascii-ci=0 ftype=1
log      =internal log           bsize=4096   blocks=71525, version=2
         =                       sectsz=512   sunit=0 blks, lazy-count=1
realtime =none                   extsz=4096   blocks=0, rtextents=0

##mount the volume
[ec2-user@ip-10-0-0-142 ~]$ sudo mkdir -p /mnt/p_iops_vol0
[ec2-user@ip-10-0-0-142 ~]$ sudo mount /dev/nvme1n1 /mnt/p_iops_vol0
[ec2-user@ip-10-0-0-142 ~]$ sudo chown ec2-user:ec2-user /mnt/p_iops_vol0/
[ec2-user@ip-10-0-0-142 ~]$ sudo lsblk
NAME          MAJ:MIN RM   SIZE RO TYPE MOUNTPOINT
nvme1n1       259:0    0 558.8G  0 disk /mnt/p_iops_vol0
nvme2n1       259:1    0 558.8G  0 disk
nvme0n1       259:2    0    20G  0 disk
├─nvme0n1p1   259:3    0    20G  0 part /
└─nvme0n1p128 259:4    0     1M  0 part
[ec2-user@ip-10-0-0-142 ~]$ df -h
Filesystem      Size  Used Avail Use% Mounted on
devtmpfs         62G     0   62G   0% /dev
tmpfs            62G     0   62G   0% /dev/shm
tmpfs            62G  556K   62G   1% /run
tmpfs            62G     0   62G   0% /sys/fs/cgroup
/dev/nvme0n1p1   20G  1.4G   19G   7% /
tmpfs            13G     0   13G   0% /run/user/1000
tmpfs            13G     0   13G   0% /run/user/0
/dev/nvme1n1    559G  603M  558G   1% /mnt/p_iops_vol0

## install XFS file system support
[ec2-user@ip-10-0-0-142 ~]$ sudo yum install -y xfsprogs
Loaded plugins: extras_suggestions, langpacks, priorities, update-motd
amzn2-core                                                                                            | 2.4 kB  00:00:00
Package xfsprogs-4.5.0-18.amzn2.0.1.x86_64 already installed and latest version
Nothing to do

## Install Benchmark Tools
[ec2-user ~]$ sudo yum install -y fio

## Disable C-States
[ec2-user@ip-10-0-0-123 ~]$ sudo cpupower idle-info | grep "Number of idle states:"
```

## performs 16 KB random write operations
```bash
sudo fio --ioengine=libaio --iodepth=32 --refill_buffers --randrepeat=0 --directory=/mnt/p_iops_vol0 --name fio_test_file --direct=1 --rw=randwrite --bs=16k --size=1G --numjobs=16 --time_based --runtime=180 --group_reporting --norandommap
OR
sudo fio --directory=/mnt/p_iops_vol0 --name fio_test_file --direct=1 --rw=randwrite --bs=16k --size=1G --numjobs=16 --time_based --runtime=180 --group_reporting --norandommap
fio_test_file: (groupid=0, jobs=16): err= 0: pid=12965: Thu Aug  8 05:41:48 2019
  write: io=90656MB, bw=515729KB/s, iops=32233, runt=180001msec
    clat (usec): min=78, max=38463, avg=495.87, stdev=66.27
     lat (usec): min=78, max=38463, avg=495.98, stdev=66.27
    clat percentiles (usec):
     |  1.00th=[  201],  5.00th=[  490], 10.00th=[  494], 20.00th=[  498],
     | 30.00th=[  498], 40.00th=[  498], 50.00th=[  498], 60.00th=[  502],
     | 70.00th=[  502], 80.00th=[  502], 90.00th=[  506], 95.00th=[  506],
     | 99.00th=[  516], 99.50th=[  524], 99.90th=[  556], 99.95th=[  980],
     | 99.99th=[ 2992]
    lat (usec) : 100=0.01%, 250=1.10%, 500=54.79%, 750=44.03%, 1000=0.04%
    lat (msec) : 2=0.02%, 4=0.01%, 10=0.01%, 50=0.01%
  cpu          : usr=0.25%, sys=1.47%, ctx=5805336, majf=0, minf=132
  IO depths    : 1=100.0%, 2=0.0%, 4=0.0%, 8=0.0%, 16=0.0%, 32=0.0%, >=64=0.0%
     submit    : 0=0.0%, 4=100.0%, 8=0.0%, 16=0.0%, 32=0.0%, 64=0.0%, >=64=0.0%
     complete  : 0=0.0%, 4=100.0%, 8=0.0%, 16=0.0%, 32=0.0%, 64=0.0%, >=64=0.0%
     issued    : total=r=0/w=5801979/d=0, short=r=0/w=0/d=0, drop=r=0/w=0/d=0
     latency   : target=0, window=0, percentile=100.00%, depth=1

Run status group 0 (all jobs):
  WRITE: io=90656MB, aggrb=515728KB/s, minb=515728KB/s, maxb=515728KB/s, mint=180001msec, maxt=180001msec

Disk stats (read/write):
  nvme1n1: ios=8/5796341, merge=0/3, ticks=4/2824684, in_queue=2695428, util=100.00%
```

## performs 16 KB random read operations
```bash
sudo fio --ioengine=libaio --iodepth=32 --refill_buffers --randrepeat=0 --directory=/mnt/p_iops_vol0 --name fio_test_file --direct=1 --rw=randread --bs=16k --size=1G --numjobs=16 --time_based --runtime=180 --group_reporting --norandommap 
fio_test_file: (groupid=0, jobs=16): err= 0: pid=17742: Thu Aug  8 08:56:44 2019
  read : io=576963MB, bw=3205.3MB/s, iops=205133, runt=180008msec
    slat (usec): min=1, max=116, avg= 2.56, stdev= 1.50
    clat (usec): min=0, max=15499, avg=2493.04, stdev=3557.35
     lat (usec): min=2, max=15502, avg=2495.60, stdev=3557.67
    clat percentiles (usec):
     |  1.00th=[    0],  5.00th=[    0], 10.00th=[    0], 20.00th=[    1],
     | 30.00th=[    3], 40.00th=[    4], 50.00th=[    7], 60.00th=[   11],
     | 70.00th=[ 7520], 80.00th=[ 7584], 90.00th=[ 7648], 95.00th=[ 7648],
     | 99.00th=[ 7712], 99.50th=[ 7776], 99.90th=[ 7840], 99.95th=[ 7840],
     | 99.99th=[ 7968]
    lat (usec) : 2=27.26%, 4=6.57%, 10=23.08%, 20=9.39%, 50=0.74%
    lat (usec) : 100=0.01%, 250=0.01%, 500=0.01%, 750=0.01%, 1000=0.01%
    lat (msec) : 2=0.01%, 4=0.06%, 10=32.88%, 20=0.01%
  cpu          : usr=1.41%, sys=3.68%, ctx=4692621, majf=0, minf=2201
  IO depths    : 1=0.1%, 2=0.1%, 4=0.1%, 8=0.1%, 16=0.1%, 32=100.0%, >=64=0.0%
     submit    : 0=0.0%, 4=100.0%, 8=0.0%, 16=0.0%, 32=0.0%, 64=0.0%, >=64=0.0%
     complete  : 0=0.0%, 4=100.0%, 8=0.0%, 16=0.0%, 32=0.1%, 64=0.0%, >=64=0.0%
     issued    : total=r=36925661/w=0/d=0, short=r=0/w=0/d=0, drop=r=0/w=0/d=0
     latency   : target=0, window=0, percentile=100.00%, depth=32

Run status group 0 (all jobs):
   READ: io=576963MB, aggrb=3205.3MB/s, minb=3205.3MB/s, maxb=3205.3MB/s, mint=180008msec, maxt=180008msec

Disk stats (read/write):
  nvme1n1: ios=12159769/84, merge=0/2210, ticks=91816720/480, in_queue=94162784, util=100.00%

# without --ioengine=libaio --iodepth=32
sudo fio --directory=/mnt/p_iops_vol0 --name fio_test_file --direct=1 --rw=randread --bs=16k --size=1G --numjobs=16 --time_based --runtime=180 --group_reporting --norandommap 
fio_test_file: (groupid=0, jobs=16): err= 0: pid=13030: Thu Aug  8 05:45:02 2019
  read : io=190492MB, bw=1058.3MB/s, iops=67730, runt=180001msec
    clat (usec): min=1, max=782, avg=235.84, stdev=50.38
     lat (usec): min=1, max=782, avg=235.88, stdev=50.38
    clat percentiles (usec):
     |  1.00th=[  135],  5.00th=[  169], 10.00th=[  181], 20.00th=[  197],
     | 30.00th=[  209], 40.00th=[  219], 50.00th=[  231], 60.00th=[  241],
     | 70.00th=[  255], 80.00th=[  274], 90.00th=[  302], 95.00th=[  326],
     | 99.00th=[  386], 99.50th=[  410], 99.90th=[  462], 99.95th=[  486],
     | 99.99th=[  540]
    lat (usec) : 2=0.02%, 4=0.18%, 10=0.01%, 20=0.01%, 50=0.01%
    lat (usec) : 100=0.01%, 250=66.09%, 500=33.66%, 750=0.03%, 1000=0.01%
  cpu          : usr=0.46%, sys=2.07%, ctx=12166172, majf=0, minf=195
  IO depths    : 1=100.0%, 2=0.0%, 4=0.0%, 8=0.0%, 16=0.0%, 32=0.0%, >=64=0.0%
     submit    : 0=0.0%, 4=100.0%, 8=0.0%, 16=0.0%, 32=0.0%, 64=0.0%, >=64=0.0%
     complete  : 0=0.0%, 4=100.0%, 8=0.0%, 16=0.0%, 32=0.0%, 64=0.0%, >=64=0.0%
     issued    : total=r=12191517/w=0/d=0, short=r=0/w=0/d=0, drop=r=0/w=0/d=0
     latency   : target=0, window=0, percentile=100.00%, depth=1

Run status group 0 (all jobs):
   READ: io=190492MB, aggrb=1058.3MB/s, minb=1058.3MB/s, maxb=1058.3MB/s, mint=180001msec, maxt=180001msec

Disk stats (read/write):
  nvme1n1: ios=12151206/51, merge=0/17, ticks=2799456/0, in_queue=2775796, util=100.00%
```

## performs 4 KB random write operations
```bash
sudo fio --ioengine=libaio --iodepth=32 --refill_buffers --randrepeat=0 --directory=/mnt/p_iops_vol0 --name fio_test_file --direct=1 --rw=randwrite --bs=4k --size=1G --numjobs=16 --time_based --runtime=180 --group_reporting --norandommap
#sudo fio --directory=/mnt/p_iops_vol0 --name fio_test_file --direct=1 --rw=randwrite --bs=4k --size=1G --numjobs=16 --time_based --runtime=180 --group_reporting --norandommap
fio_test_file: (groupid=0, jobs=16): err= 0: pid=9588: Thu Aug  8 06:11:47 2019
  write: io=82592MB, bw=469853KB/s, iops=117463, runt=180001msec
    clat (usec): min=34, max=2474, avg=135.80, stdev=11.57
     lat (usec): min=34, max=2474, avg=135.86, stdev=11.57
    clat percentiles (usec):
     |  1.00th=[   68],  5.00th=[  127], 10.00th=[  131], 20.00th=[  133],
     | 30.00th=[  135], 40.00th=[  137], 50.00th=[  137], 60.00th=[  139],
     | 70.00th=[  139], 80.00th=[  141], 90.00th=[  143], 95.00th=[  145],
     | 99.00th=[  159], 99.50th=[  167], 99.90th=[  189], 99.95th=[  201],
     | 99.99th=[  227]
    lat (usec) : 50=0.06%, 100=1.37%, 250=98.57%, 500=0.01%, 750=0.01%
    lat (usec) : 1000=0.01%
    lat (msec) : 2=0.01%, 4=0.01%
  cpu          : usr=0.75%, sys=4.16%, ctx=21143653, majf=0, minf=147
  IO depths    : 1=100.0%, 2=0.0%, 4=0.0%, 8=0.0%, 16=0.0%, 32=0.0%, >=64=0.0%
     submit    : 0=0.0%, 4=100.0%, 8=0.0%, 16=0.0%, 32=0.0%, 64=0.0%, >=64=0.0%
     complete  : 0=0.0%, 4=100.0%, 8=0.0%, 16=0.0%, 32=0.0%, 64=0.0%, >=64=0.0%
     issued    : total=r=0/w=21143490/d=0, short=r=0/w=0/d=0, drop=r=0/w=0/d=0
     latency   : target=0, window=0, percentile=100.00%, depth=1

Run status group 0 (all jobs):
  WRITE: io=82592MB, aggrb=469852KB/s, minb=469852KB/s, maxb=469852KB/s, mint=180001msec, maxt=180001msec

Disk stats (read/write):
  nvme1n1: ios=0/21137476, merge=0/3, ticks=0/2732788, in_queue=2834504, util=100.00%
```

## performs 4 KB random read operations
```bash
sudo fio --ioengine=libaio --iodepth=32 --refill_buffers --randrepeat=0 --directory=/mnt/p_iops_vol0 --name fio_test_file --direct=1 --rw=randread --bs=4k --size=1G --numjobs=16 --time_based --runtime=180 --group_reporting --norandommap 
fio_test_file: (groupid=0, jobs=16): err= 0: pid=17888: Thu Aug  8 09:01:34 2019
  read : io=307677MB, bw=1709.3MB/s, iops=437578, runt=180003msec
    slat (usec): min=1, max=68, avg= 2.11, stdev= 1.43
    clat (usec): min=0, max=5094, avg=1167.67, stdev=1079.10
     lat (usec): min=1, max=5096, avg=1169.77, stdev=1079.43
    clat percentiles (usec):
     |  1.00th=[    0],  5.00th=[    1], 10.00th=[    1], 20.00th=[    5],
     | 30.00th=[    9], 40.00th=[   15], 50.00th=[ 2128], 60.00th=[ 2160],
     | 70.00th=[ 2160], 80.00th=[ 2192], 90.00th=[ 2224], 95.00th=[ 2256],
     | 99.00th=[ 2320], 99.50th=[ 2352], 99.90th=[ 2416], 99.95th=[ 2448],
     | 99.99th=[ 3152]
    lat (usec) : 2=11.51%, 4=5.65%, 10=13.66%, 20=12.88%, 50=2.45%
    lat (usec) : 100=0.02%, 250=0.01%, 500=0.01%, 750=0.01%, 1000=0.07%
    lat (msec) : 2=1.01%, 4=52.73%, 10=0.01%
  cpu          : usr=3.01%, sys=6.65%, ctx=10311509, majf=0, minf=648
  IO depths    : 1=0.1%, 2=0.1%, 4=0.1%, 8=0.1%, 16=0.1%, 32=100.0%, >=64=0.0%
     submit    : 0=0.0%, 4=100.0%, 8=0.0%, 16=0.0%, 32=0.0%, 64=0.0%, >=64=0.0%
     complete  : 0=0.0%, 4=100.0%, 8=0.0%, 16=0.0%, 32=0.1%, 64=0.0%, >=64=0.0%
     issued    : total=r=78765368/w=0/d=0, short=r=0/w=0/d=0, drop=r=0/w=0/d=0
     latency   : target=0, window=0, percentile=100.00%, depth=32

Run status group 0 (all jobs):
   READ: io=307677MB, aggrb=1709.3MB/s, minb=1709.3MB/s, maxb=1709.3MB/s, mint=180003msec, maxt=180003msec

Disk stats (read/write):
  nvme1n1: ios=42383508/1036, merge=0/6746, ticks=91297176/1648, in_queue=110712184, util=100.00%
  
# without --ioengine=libaio --iodepth=32
sudo fio --directory=/mnt/p_iops_vol0 --name fio_test_file --direct=1 --rw=randread --bs=4k --size=1G --numjobs=16 --time_based --runtime=180 --group_reporting --norandommap 
fio_test_file: (groupid=0, jobs=16): err= 0: pid=9686: Thu Aug  8 06:16:04 2019
  read : io=100751MB, bw=573157KB/s, iops=143289, runt=180001msec
    clat (usec): min=1, max=588, avg=111.26, stdev=24.64
     lat (usec): min=1, max=588, avg=111.30, stdev=24.64
    clat percentiles (usec):
     |  1.00th=[   83],  5.00th=[   88], 10.00th=[   91], 20.00th=[   93],
     | 30.00th=[   96], 40.00th=[  100], 50.00th=[  105], 60.00th=[  109],
     | 70.00th=[  114], 80.00th=[  123], 90.00th=[  145], 95.00th=[  163],
     | 99.00th=[  201], 99.50th=[  219], 99.90th=[  258], 99.95th=[  274],
     | 99.99th=[  310]
    lat (usec) : 2=0.01%, 4=0.01%, 50=0.01%, 100=39.18%, 250=60.68%
    lat (usec) : 500=0.13%, 750=0.01%
  cpu          : usr=0.96%, sys=4.10%, ctx=25791895, majf=0, minf=164
  IO depths    : 1=100.0%, 2=0.0%, 4=0.0%, 8=0.0%, 16=0.0%, 32=0.0%, >=64=0.0%
     submit    : 0=0.0%, 4=100.0%, 8=0.0%, 16=0.0%, 32=0.0%, 64=0.0%, >=64=0.0%
     complete  : 0=0.0%, 4=100.0%, 8=0.0%, 16=0.0%, 32=0.0%, 64=0.0%, >=64=0.0%
     issued    : total=r=25792201/w=0/d=0, short=r=0/w=0/d=0, drop=r=0/w=0/d=0
     latency   : target=0, window=0, percentile=100.00%, depth=1

Run status group 0 (all jobs):
   READ: io=100751MB, aggrb=573156KB/s, minb=573156KB/s, maxb=573156KB/s, mint=180001msec, maxt=180001msec

Disk stats (read/write):
  nvme1n1: ios=25761420/6, merge=0/0, ticks=2725544/0, in_queue=2850996, util=100.00%
```

# Testing with RAID0
```
#$ sudo umount /mnt/p_iops_vol0 && sudo umount /dev/md0
#$ sudo mdadm --stop /dev/md0
#$ sudo mdadm --remove /dev/md0
```

## create a 2-volume stripe Raid0
```bash
[ec2-user@ip-10-0-0-142 ~]$ sudo umount /mnt/p_iops_vol0
[ec2-user@ip-10-0-0-142 ~]$ sudo lsblk
NAME          MAJ:MIN RM   SIZE RO TYPE MOUNTPOINT
nvme1n1       259:0    0 558.8G  0 disk
nvme2n1       259:1    0 558.8G  0 disk
nvme0n1       259:2    0    20G  0 disk
├─nvme0n1p1   259:3    0    20G  0 part /
└─nvme0n1p128 259:4    0     1M  0 part

[ec2-user@ip-10-0-0-142 ~]$ sudo mdadm --create /dev/md0 --level=0 --chunk=64 --raid-devices=2 /dev/nvme1n1 /dev/nvme2n1
mdadm: Defaulting to version 1.2 metadata
mdadm: array /dev/md0 started.

## Allow time for the RAID array to initialize and synchronize. 
[ec2-user@ip-10-0-0-142 ~]$ sudo cat /proc/mdstat
Personalities : [raid0]
md0 : active raid0 nvme2n1[1] nvme1n1[0]
      1171612800 blocks super 1.2 64k chunks

unused devices: <none>


[ec2-user@ip-10-0-0-142 ~]$ sudo mdadm --detail /dev/md0
/dev/md0:
           Version : 1.2
     Creation Time : Thu Aug  8 06:23:03 2019
        Raid Level : raid0
        Array Size : 1171612800 (1117.34 GiB 1199.73 GB)
      Raid Devices : 2
     Total Devices : 2
       Persistence : Superblock is persistent

       Update Time : Thu Aug  8 06:23:03 2019
             State : clean
    Active Devices : 2
   Working Devices : 2
    Failed Devices : 0
     Spare Devices : 0

        Chunk Size : 64K

Consistency Policy : none

              Name : 0
              UUID : d8553e35:365e792d:f43f442c:16e4fc82
            Events : 0

    Number   Major   Minor   RaidDevice State
       0     259        0        0      active sync   /dev/nvme1n1
       1     259        1        1      active sync   /dev/nvme2n1

[ec2-user@ip-10-0-0-142 ~]$ sudo lsblk
NAME          MAJ:MIN RM   SIZE RO TYPE  MOUNTPOINT
nvme1n1       259:0    0 558.8G  0 disk
└─md0           9:0    0   1.1T  0 raid0
nvme2n1       259:1    0 558.8G  0 disk
└─md0           9:0    0   1.1T  0 raid0
nvme0n1       259:2    0    20G  0 disk
├─nvme0n1p1   259:3    0    20G  0 part  /
└─nvme0n1p128 259:4    0     1M  0 part

##use the mkfs -t command to create a file system on the volume
[ec2-user@ip-10-0-0-142 ~]$ sudo mkdir -p /mnt/p_iops_vol0 && sudo mkfs.xfs -L MY_RAID /dev/md0
meta-data=/dev/md0               isize=512    agcount=32, agsize=9153232 blks
         =                       sectsz=512   attr=2, projid32bit=1
         =                       crc=1        finobt=1, sparse=0
data     =                       bsize=4096   blocks=292903200, imaxpct=5
         =                       sunit=16     swidth=32 blks
naming   =version 2              bsize=4096   ascii-ci=0 ftype=1
log      =internal log           bsize=4096   blocks=143024, version=2
         =                       sectsz=512   sunit=16 blks, lazy-count=1
realtime =none                   extsz=4096   blocks=0, rtextents=0


##To ensure that the RAID array is reassembled automatically on boot, create a configuration file to contain the RAID information:
[ec2-user@ip-10-0-0-142 ~]$ sudo mdadm --detail --scan | sudo tee -a /etc/mdadm.conf
ARRAY /dev/md0 metadata=1.2 name=0 UUID=d8553e35:365e792d:f43f442c:16e4fc82

##Create a new ramdisk image to properly preload the block device modules for your new RAID configuration:
[ec2-user@ip-10-0-0-142 ~]$ sudo dracut -H -f /boot/initramfs-$(uname -r).img $(uname -r)

##mount the volume
[ec2-user@ip-10-0-0-142 ~]$ sudo mount -t xfs /dev/md0 /mnt/p_iops_vol0
[ec2-user@ip-10-0-0-142 ~]$ sudo chown ec2-user:ec2-user /mnt/p_iops_vol0/
[ec2-user@ip-10-0-0-142 ~]$ sudo lsblk
NAME          MAJ:MIN RM   SIZE RO TYPE  MOUNTPOINT
nvme1n1       259:0    0 558.8G  0 disk
└─md0           9:0    0   1.1T  0 raid0 /mnt/p_iops_vol0
nvme2n1       259:1    0 558.8G  0 disk
└─md0           9:0    0   1.1T  0 raid0 /mnt/p_iops_vol0
nvme0n1       259:2    0    20G  0 disk
├─nvme0n1p1   259:3    0    20G  0 part  /
└─nvme0n1p128 259:4    0     1M  0 part
[ec2-user@ip-10-0-0-142 ~]$ df -h
Filesystem      Size  Used Avail Use% Mounted on
devtmpfs         62G     0   62G   0% /dev
tmpfs            62G     0   62G   0% /dev/shm
tmpfs            62G  568K   62G   1% /run
tmpfs            62G     0   62G   0% /sys/fs/cgroup
/dev/nvme0n1p1   20G  1.4G   19G   7% /
tmpfs            13G     0   13G   0% /run/user/1000
tmpfs            13G     0   13G   0% /run/user/0
/dev/md0        1.1T  1.2G  1.1T   1% /mnt/p_iops_vol0


##Automatically Mount an Attached Volume After Reboot
##Create a backup of your /etc/fstab file
sudo cp /etc/fstab /etc/fstab.orig


##Add the following entry to /etc/fstab to mount the device at the specified mount point.
sudo vim /etc/fstab
LABEL=MY_RAID       /mnt/p_iops_vol0   xfs    defaults,nofail        0       2

##To verify that your entry works
[ec2-user@ip-10-0-0-123 ~]$ sudo umount /mnt/p_iops_vol0
[ec2-user@ip-10-0-0-123 ~]$ sudo mount -a
[ec2-user@ip-10-0-0-123 ~]$ df -h


## Disable C-States
[ec2-user@ip-10-0-0-123 ~]$ sudo cpupower idle-info | grep "Number of idle states:"
```

# Run the testing
## performs 4 KB random write operations
```bash
sudo fio --directory=/mnt/p_iops_vol0 --name fio_test_file --direct=1 --rw=randwrite --bs=4k --size=1G --numjobs=16 --time_based --runtime=180 --group_reporting --norandommap
fio_test_file: (groupid=0, jobs=16): err= 0: pid=16252: Thu Aug  8 06:39:54 2019
  write: io=165007MB, bw=938701KB/s, iops=234675, runt=180001msec
    clat (usec): min=24, max=433, avg=67.74, stdev=34.41
     lat (usec): min=24, max=433, avg=67.81, stdev=34.41
    clat percentiles (usec):
     |  1.00th=[   27],  5.00th=[   29], 10.00th=[   30], 20.00th=[   33],
     | 30.00th=[   36], 40.00th=[   41], 50.00th=[   55], 60.00th=[   85],
     | 70.00th=[   97], 80.00th=[  105], 90.00th=[  115], 95.00th=[  121],
     | 99.00th=[  131], 99.50th=[  135], 99.90th=[  145], 99.95th=[  151],
     | 99.99th=[  173]
    lat (usec) : 50=48.04%, 100=25.03%, 250=26.94%, 500=0.01%
  cpu          : usr=1.53%, sys=8.95%, ctx=42241857, majf=0, minf=162
  IO depths    : 1=100.0%, 2=0.0%, 4=0.0%, 8=0.0%, 16=0.0%, 32=0.0%, >=64=0.0%
     submit    : 0=0.0%, 4=100.0%, 8=0.0%, 16=0.0%, 32=0.0%, 64=0.0%, >=64=0.0%
     complete  : 0=0.0%, 4=100.0%, 8=0.0%, 16=0.0%, 32=0.0%, 64=0.0%, >=64=0.0%
     issued    : total=r=0/w=42241763/d=0, short=r=0/w=0/d=0, drop=r=0/w=0/d=0
     latency   : target=0, window=0, percentile=100.00%, depth=1

Run status group 0 (all jobs):
  WRITE: io=165007MB, aggrb=938700KB/s, minb=938700KB/s, maxb=938700KB/s, mint=180001msec, maxt=180001msec

Disk stats (read/write):
    md0: ios=0/42190574, merge=0/0, ticks=0/0, in_queue=0, util=0.00%, aggrios=0/21120887, aggrmerge=0/0, aggrticks=0/1287348, aggrin_queue=1178496, aggrutil=100.00%
  nvme2n1: ios=0/21126781, merge=0/0, ticks=0/1909764, in_queue=1834768, util=100.00%
  nvme1n1: ios=0/21114993, merge=0/0, ticks=0/664932, in_queue=522224, util=94.47%
```

## performs 4 KB random read operations
```bash
sudo fio --directory=/mnt/p_iops_vol0 --name fio_test_file --direct=1 --rw=randread --bs=4k --size=1G --numjobs=16 --time_based --runtime=180 --group_reporting --norandommap 
fio_test_file: (groupid=0, jobs=16): err= 0: pid=16315: Thu Aug  8 06:48:31 2019
  read : io=113191MB, bw=643927KB/s, iops=160981, runt=180001msec
    clat (usec): min=34, max=544, avg=98.98, stdev=17.92
     lat (usec): min=34, max=544, avg=99.02, stdev=17.92
    clat percentiles (usec):
     |  1.00th=[   75],  5.00th=[   83], 10.00th=[   84], 20.00th=[   86],
     | 30.00th=[   88], 40.00th=[   90], 50.00th=[   94], 60.00th=[  100],
     | 70.00th=[  103], 80.00th=[  107], 90.00th=[  119], 95.00th=[  137],
     | 99.00th=[  167], 99.50th=[  179], 99.90th=[  209], 99.95th=[  221],
     | 99.99th=[  251]
    lat (usec) : 50=0.01%, 100=59.61%, 250=40.37%, 500=0.01%, 750=0.01%
  cpu          : usr=1.06%, sys=4.93%, ctx=28976964, majf=0, minf=148
  IO depths    : 1=100.0%, 2=0.0%, 4=0.0%, 8=0.0%, 16=0.0%, 32=0.0%, >=64=0.0%
     submit    : 0=0.0%, 4=100.0%, 8=0.0%, 16=0.0%, 32=0.0%, 64=0.0%, >=64=0.0%
     complete  : 0=0.0%, 4=100.0%, 8=0.0%, 16=0.0%, 32=0.0%, 64=0.0%, >=64=0.0%
     issued    : total=r=28976854/w=0/d=0, short=r=0/w=0/d=0, drop=r=0/w=0/d=0
     latency   : target=0, window=0, percentile=100.00%, depth=1

Run status group 0 (all jobs):
   READ: io=113191MB, aggrb=643926KB/s, minb=643926KB/s, maxb=643926KB/s, mint=180001msec, maxt=180001msec

Disk stats (read/write):
    md0: ios=28940939/6, merge=0/0, ticks=0/0, in_queue=0, util=0.00%, aggrios=14488427/3, aggrmerge=0/0, aggrticks=1353942/0, aggrin_queue=1251998, aggrutil=100.00%
  nvme2n1: ios=14493475/3, merge=0/0, ticks=1353252/0, in_queue=1250616, util=100.00%
  nvme1n1: ios=14483379/3, merge=0/0, ticks=1354632/0, in_queue=1253380, util=100.00%


#fio --bs=4k --ioengine=libaio --iodepth=32 --direct=1 --rw=read --time_based --runtime=600  --refill_buffers --norandommap --randrepeat=0 --group_reporting --name=fio-read --size=10G --filename=/dev/sdb

## optimize the fio parameter --ioengine=libaio --iodepth=32 redo performs 4 KB random read operations
sudo fio --ioengine=libaio --iodepth=32 --directory=/mnt/p_iops_vol0 --name fio_test_file --direct=1 --rw=randread --refill_buffers --randrepeat=0 --bs=4k --size=1G --numjobs=16 --time_based --runtime=180 --group_reporting --norandommap 
fio_test_file: (groupid=0, jobs=16): err= 0: pid=16736: Thu Aug  8 07:37:01 2019
  read : io=330972MB, bw=1838.8MB/s, iops=470710, runt=180002msec
    slat (usec): min=1, max=588, avg= 2.61, stdev= 1.48
    clat (usec): min=52, max=4211, avg=1084.72, stdev=945.08
     lat (usec): min=66, max=4213, avg=1087.33, stdev=945.09
    clat percentiles (usec):
     |  1.00th=[  100],  5.00th=[  110], 10.00th=[  118], 20.00th=[  131],
     | 30.00th=[  151], 40.00th=[  187], 50.00th=[  612], 60.00th=[ 1896],
     | 70.00th=[ 2040], 80.00th=[ 2128], 90.00th=[ 2192], 95.00th=[ 2224],
     | 99.00th=[ 2288], 99.50th=[ 2320], 99.90th=[ 2384], 99.95th=[ 2416],
     | 99.99th=[ 2480]
    lat (usec) : 100=0.98%, 250=45.80%, 500=2.88%, 750=0.68%, 1000=0.30%
    lat (msec) : 2=15.32%, 4=34.04%, 10=0.01%
  cpu          : usr=4.07%, sys=11.93%, ctx=40577603, majf=0, minf=647
  IO depths    : 1=0.1%, 2=0.1%, 4=0.1%, 8=0.1%, 16=0.1%, 32=100.0%, >=64=0.0%
     submit    : 0=0.0%, 4=100.0%, 8=0.0%, 16=0.0%, 32=0.0%, 64=0.0%, >=64=0.0%
     complete  : 0=0.0%, 4=100.0%, 8=0.0%, 16=0.0%, 32=0.1%, 64=0.0%, >=64=0.0%
     issued    : total=r=84728883/w=0/d=0, short=r=0/w=0/d=0, drop=r=0/w=0/d=0
     latency   : target=0, window=0, percentile=100.00%, depth=32

Run status group 0 (all jobs):
   READ: io=330972MB, aggrb=1838.8MB/s, minb=1838.8MB/s, maxb=1838.8MB/s, mint=180002msec, maxt=180002msec

Disk stats (read/write):
    md0: ios=84705240/0, merge=0/0, ticks=0/0, in_queue=0, util=0.00%, aggrios=42364441/0, aggrmerge=0/0, aggrticks=45664264/0, aggrin_queue=59221400, aggrutil=100.00%
  nvme2n1: ios=42359649/0, merge=0/0, ticks=84739564/0, in_queue=110771132, util=100.00%
  nvme1n1: ios=42369234/0, merge=0/0, ticks=6588964/0, in_queue=7671668, util=100.00%

## optimize the fio parameter --ioengine=libaio --iodepth=32 redo performs 4 KB random write operations
## the result is the same as previous result without --ioengine=libaio --iodepth=32
sudo fio --ioengine=libaio --iodepth=32 --directory=/mnt/p_iops_vol0 --name fio_test_file --direct=1 --rw=randwrite --refill_buffers --randrepeat=0 --bs=4k --size=1G --numjobs=16 --time_based --runtime=180 --group_reporting --norandommap
fio_test_file: (groupid=0, jobs=16): err= 0: pid=16940: Thu Aug  8 08:12:51 2019
  write: io=164996MB, bw=938619KB/s, iops=234654, runt=180005msec
    slat (usec): min=2, max=591, avg= 4.59, stdev= 2.35
    clat (usec): min=0, max=28783, avg=2175.91, stdev=2638.82
     lat (usec): min=24, max=28785, avg=2180.50, stdev=2638.83
    clat percentiles (usec):
     |  1.00th=[   30],  5.00th=[   34], 10.00th=[   36], 20.00th=[   40],
     | 30.00th=[   44], 40.00th=[   50], 50.00th=[  652], 60.00th=[ 2416],
     | 70.00th=[ 3376], 80.00th=[ 4448], 90.00th=[ 5856], 95.00th=[ 7008],
     | 99.00th=[10048], 99.50th=[11456], 99.90th=[14144], 99.95th=[15552],
     | 99.99th=[18304]
    lat (usec) : 2=0.01%, 4=0.01%, 10=0.01%, 20=0.01%, 50=39.70%
    lat (usec) : 100=9.70%, 250=0.37%, 500=0.13%, 750=0.14%, 1000=0.11%
    lat (msec) : 2=5.35%, 4=19.59%, 10=23.83%, 20=1.07%, 50=0.01%
  cpu          : usr=4.34%, sys=11.12%, ctx=28181988, majf=0, minf=171
  IO depths    : 1=0.1%, 2=0.1%, 4=0.1%, 8=0.1%, 16=0.1%, 32=100.0%, >=64=0.0%
     submit    : 0=0.0%, 4=100.0%, 8=0.0%, 16=0.0%, 32=0.0%, 64=0.0%, >=64=0.0%
     complete  : 0=0.0%, 4=100.0%, 8=0.0%, 16=0.0%, 32=0.1%, 64=0.0%, >=64=0.0%
     issued    : total=r=0/w=42239036/d=0, short=r=0/w=0/d=0, drop=r=0/w=0/d=0
     latency   : target=0, window=0, percentile=100.00%, depth=32

Run status group 0 (all jobs):
  WRITE: io=164996MB, aggrb=938619KB/s, minb=938619KB/s, maxb=938619KB/s, mint=180005msec, maxt=180005msec

Disk stats (read/write):
    md0: ios=0/42187795, merge=0/0, ticks=0/0, in_queue=0, util=0.00%, aggrios=0/21119523, aggrmerge=0/0, aggrticks=0/45631314, aggrin_queue=49620818, aggrutil=100.00%
  nvme2n1: ios=0/21116042, merge=0/0, ticks=0/793884, in_queue=639812, util=87.56%
  nvme1n1: ios=0/21123005, merge=0/0, ticks=0/90468744, in_queue=98601824, util=100.00%
```


## performs 4 KB read operations
```bash
sudo fio --ioengine=libaio --iodepth=32 --directory=/mnt/p_iops_vol0 --name fio_test_file --direct=1 --rw=read --refill_buffers --randrepeat=0 --bs=4k --size=1G --numjobs=16 --time_based --runtime=180 --group_reporting --norandommap 
fio_test_file: (groupid=0, jobs=16): err= 0: pid=16675: Thu Aug  8 07:31:03 2019
  read : io=229640MB, bw=1275.8MB/s, iops=326592, runt=180004msec
    slat (usec): min=1, max=550, avg= 2.69, stdev= 1.60
    clat (usec): min=6, max=6411, avg=1564.62, stdev=1424.36
     lat (usec): min=21, max=6413, avg=1567.31, stdev=1424.40
    clat percentiles (usec):
     |  1.00th=[   93],  5.00th=[  124], 10.00th=[  143], 20.00th=[  175],
     | 30.00th=[  213], 40.00th=[  282], 50.00th=[  788], 60.00th=[ 2480],
     | 70.00th=[ 2832], 80.00th=[ 3120], 90.00th=[ 3408], 95.00th=[ 3632],
     | 99.00th=[ 4048], 99.50th=[ 4192], 99.90th=[ 4512], 99.95th=[ 4640],
     | 99.99th=[ 4960]
    lat (usec) : 10=0.01%, 20=0.01%, 50=0.12%, 100=1.34%, 250=34.80%
    lat (usec) : 500=12.18%, 750=1.46%, 1000=0.86%
    lat (msec) : 2=3.68%, 4=44.29%, 10=1.28%
  cpu          : usr=2.85%, sys=8.49%, ctx=27985999, majf=0, minf=663
  IO depths    : 1=0.1%, 2=0.1%, 4=0.1%, 8=0.1%, 16=0.1%, 32=100.0%, >=64=0.0%
     submit    : 0=0.0%, 4=100.0%, 8=0.0%, 16=0.0%, 32=0.0%, 64=0.0%, >=64=0.0%
     complete  : 0=0.0%, 4=100.0%, 8=0.0%, 16=0.0%, 32=0.1%, 64=0.0%, >=64=0.0%
     issued    : total=r=58787917/w=0/d=0, short=r=0/w=0/d=0, drop=r=0/w=0/d=0
     latency   : target=0, window=0, percentile=100.00%, depth=32

Run status group 0 (all jobs):
   READ: io=229640MB, aggrb=1275.8MB/s, minb=1275.8MB/s, maxb=1275.8MB/s, mint=180004msec, maxt=180004msec

Disk stats (read/write):
    md0: ios=58771651/0, merge=0/0, ticks=0/0, in_queue=0, util=0.00%, aggrios=29393958/0, aggrmerge=0/0, aggrticks=45775500/0, aggrin_queue=54101972, aggrutil=100.00%
  nvme2n1: ios=29393875/0, merge=0/0, ticks=64639704/0, in_queue=77000604, util=100.00%
  nvme1n1: ios=29394042/0, merge=0/0, ticks=26911296/0, in_queue=31203340, util=100.00%
```

## performs 4 KB mix random read / write operations
```bash
sudo fio --ioengine=libaio --iodepth=32 --directory=/mnt/p_iops_vol0 --name fio_test_file --direct=1 --rw=randrw --refill_buffers --randrepeat=0 --bs=4k --size=1G --numjobs=16 --time_based --runtime=180 --group_reporting --norandommap 
fio_test_file: (groupid=0, jobs=16): err= 0: pid=16788: Thu Aug  8 07:41:50 2019
  read : io=165004MB, bw=938674KB/s, iops=234668, runt=180003msec
    slat (usec): min=1, max=184, avg= 3.43, stdev= 2.34
    clat (usec): min=38, max=9776, avg=998.92, stdev=914.11
     lat (usec): min=40, max=9779, avg=1002.35, stdev=914.15
    clat percentiles (usec):
     |  1.00th=[  113],  5.00th=[  135], 10.00th=[  155], 20.00th=[  197],
     | 30.00th=[  243], 40.00th=[  314], 50.00th=[  628], 60.00th=[ 1240],
     | 70.00th=[ 1496], 80.00th=[ 1704], 90.00th=[ 2128], 95.00th=[ 2896],
     | 99.00th=[ 3536], 99.50th=[ 4128], 99.90th=[ 5024], 99.95th=[ 5536],
     | 99.99th=[ 7008]
  write: io=165023MB, bw=938779KB/s, iops=234694, runt=180003msec
    slat (usec): min=2, max=278, avg= 4.78, stdev= 2.77
    clat (usec): min=0, max=9732, avg=1172.38, stdev=1209.01
     lat (usec): min=25, max=9736, avg=1177.16, stdev=1209.06
    clat percentiles (usec):
     |  1.00th=[   42],  5.00th=[   51], 10.00th=[   57], 20.00th=[   66],
     | 30.00th=[   76], 40.00th=[   92], 50.00th=[  548], 60.00th=[ 1768],
     | 70.00th=[ 2024], 80.00th=[ 2224], 90.00th=[ 2576], 95.00th=[ 3408],
     | 99.00th=[ 4016], 99.50th=[ 4576], 99.90th=[ 5536], 99.95th=[ 5920],
     | 99.99th=[ 7264]
    lat (usec) : 2=0.01%, 4=0.01%, 10=0.01%, 20=0.01%, 50=1.93%
    lat (usec) : 100=19.34%, 250=18.92%, 500=8.68%, 750=1.97%, 1000=1.88%
    lat (msec) : 2=26.16%, 4=20.30%, 10=0.81%
  cpu          : usr=7.23%, sys=18.87%, ctx=49269310, majf=0, minf=154
  IO depths    : 1=0.1%, 2=0.1%, 4=0.1%, 8=0.1%, 16=0.1%, 32=100.0%, >=64=0.0%
     submit    : 0=0.0%, 4=100.0%, 8=0.0%, 16=0.0%, 32=0.0%, 64=0.0%, >=64=0.0%
     complete  : 0=0.0%, 4=100.0%, 8=0.0%, 16=0.0%, 32=0.1%, 64=0.0%, >=64=0.0%
     issued    : total=r=42241041/w=42245780/d=0, short=r=0/w=0/d=0, drop=r=0/w=0/d=0
     latency   : target=0, window=0, percentile=100.00%, depth=32

Run status group 0 (all jobs):
   READ: io=165004MB, aggrb=938674KB/s, minb=938674KB/s, maxb=938674KB/s, mint=180003msec, maxt=180003msec
  WRITE: io=165023MB, aggrb=938779KB/s, minb=938779KB/s, maxb=938779KB/s, mint=180003msec, maxt=180003msec

Disk stats (read/write):
    md0: ios=42189232/42193967, merge=0/0, ticks=0/0, in_queue=0, util=0.00%, aggrios=21120520/21122895, aggrmerge=0/0, aggrticks=20912592/24455224, aggrin_queue=53858610, aggrutil=100.00%
  nvme2n1: ios=21121119/21122696, merge=0/0, ticks=36326976/47040020, in_queue=99555956, util=100.00%
  nvme1n1: ios=21119922/21123095, merge=0/0, ticks=5498208/1870428, in_queue=8161264, util=100.00%
```

## performs 16 KB random write operations
```bash
sudo fio --ioengine=libaio --iodepth=32 --refill_buffers --randrepeat=0 --directory=/mnt/p_iops_vol0 --name fio_test_file --direct=1 --rw=randwrite --bs=16k --size=1G --numjobs=16 --time_based --runtime=180 --group_reporting --norandommap
OR
sudo fio --directory=/mnt/p_iops_vol0 --name fio_test_file --direct=1 --rw=randwrite --bs=16k --size=1G --numjobs=16 --time_based --runtime=180 --group_reporting --norandommap
fio_test_file: (groupid=0, jobs=16): err= 0: pid=16096: Thu Aug  8 06:30:01 2019
  write: io=181324MB, bw=1007.4MB/s, iops=64470, runt=180001msec
    clat (usec): min=32, max=18217, avg=247.69, stdev=211.90
     lat (usec): min=32, max=18217, avg=247.80, stdev=211.90
    clat percentiles (usec):
     |  1.00th=[   35],  5.00th=[   36], 10.00th=[   37], 20.00th=[   38],
     | 30.00th=[   40], 40.00th=[   43], 50.00th=[   84], 60.00th=[  434],
     | 70.00th=[  454], 80.00th=[  466], 90.00th=[  486], 95.00th=[  498],
     | 99.00th=[  502], 99.50th=[  506], 99.90th=[  516], 99.95th=[  532],
     | 99.99th=[ 1080]
    lat (usec) : 50=46.56%, 100=3.68%, 250=0.34%, 500=46.55%, 750=2.85%
    lat (usec) : 1000=0.01%
    lat (msec) : 2=0.01%, 4=0.01%, 10=0.01%, 20=0.01%
  cpu          : usr=0.49%, sys=2.76%, ctx=11607788, majf=0, minf=149
  IO depths    : 1=100.0%, 2=0.0%, 4=0.0%, 8=0.0%, 16=0.0%, 32=0.0%, >=64=0.0%
     submit    : 0=0.0%, 4=100.0%, 8=0.0%, 16=0.0%, 32=0.0%, 64=0.0%, >=64=0.0%
     complete  : 0=0.0%, 4=100.0%, 8=0.0%, 16=0.0%, 32=0.0%, 64=0.0%, >=64=0.0%
     issued    : total=r=0/w=11604741/d=0, short=r=0/w=0/d=0, drop=r=0/w=0/d=0
     latency   : target=0, window=0, percentile=100.00%, depth=1

Run status group 0 (all jobs):
  WRITE: io=181324MB, aggrb=1007.4MB/s, minb=1007.4MB/s, maxb=1007.4MB/s, mint=180001msec, maxt=180001msec

Disk stats (read/write):
    md0: ios=4/11591031, merge=0/0, ticks=0/0, in_queue=0, util=0.00%, aggrios=2/5802531, aggrmerge=0/0, aggrticks=0/1391800, aggrin_queue=1285542, aggrutil=100.00%
  nvme2n1: ios=4/5804073, merge=0/0, ticks=0/2583576, in_queue=2479000, util=100.00%
  nvme1n1: ios=0/5800990, merge=0/1, ticks=0/200024, in_queue=92084, util=37.34%
```

## performs 16 KB random read operations
```bash
sudo fio --ioengine=libaio --iodepth=32 --refill_buffers --randrepeat=0 --directory=/mnt/p_iops_vol0 --name fio_test_file --direct=1 --rw=randread --bs=16k --size=1G --numjobs=16 --time_based --runtime=180 --group_reporting --norandommap 
fio_test_file: (groupid=0, jobs=16): err= 0: pid=17049: Thu Aug  8 08:25:35 2019
  read : io=380324MB, bw=2112.9MB/s, iops=135220, runt=180008msec
    slat (usec): min=2, max=101, avg= 3.15, stdev= 1.60
    clat (usec): min=59, max=14931, avg=3782.80, stdev=3607.17
     lat (usec): min=86, max=14933, avg=3785.95, stdev=3607.30
    clat percentiles (usec):
     |  1.00th=[  106],  5.00th=[  120], 10.00th=[  137], 20.00th=[  163],
     | 30.00th=[  189], 40.00th=[  231], 50.00th=[ 2096], 60.00th=[ 7328],
     | 70.00th=[ 7456], 80.00th=[ 7520], 90.00th=[ 7584], 95.00th=[ 7648],
     | 99.00th=[ 7712], 99.50th=[ 7776], 99.90th=[ 8032], 99.95th=[ 8256],
     | 99.99th=[ 9024]
    lat (usec) : 100=0.31%, 250=42.55%, 500=6.36%, 750=0.08%, 1000=0.08%
    lat (msec) : 2=0.56%, 4=0.73%, 10=49.32%, 20=0.01%
  cpu          : usr=1.21%, sys=4.28%, ctx=15502955, majf=0, minf=2184
  IO depths    : 1=0.1%, 2=0.1%, 4=0.1%, 8=0.1%, 16=0.1%, 32=100.0%, >=64=0.0%
     submit    : 0=0.0%, 4=100.0%, 8=0.0%, 16=0.0%, 32=0.0%, 64=0.0%, >=64=0.0%
     complete  : 0=0.0%, 4=100.0%, 8=0.0%, 16=0.0%, 32=0.1%, 64=0.0%, >=64=0.0%
     issued    : total=r=24340734/w=0/d=0, short=r=0/w=0/d=0, drop=r=0/w=0/d=0
     latency   : target=0, window=0, percentile=100.00%, depth=32

Run status group 0 (all jobs):
   READ: io=380324MB, aggrb=2112.9MB/s, minb=2112.9MB/s, maxb=2112.9MB/s, mint=180008msec, maxt=180008msec

Disk stats (read/write):
    md0: ios=24333825/6, merge=0/0, ticks=0/0, in_queue=0, util=0.00%, aggrios=12170367/3, aggrmerge=0/0, aggrticks=45963134/0, aggrin_queue=48625954, aggrutil=100.00%
  nvme2n1: ios=12167984/2, merge=0/0, ticks=89323392/0, in_queue=94707868, util=100.00%
  nvme1n1: ios=12172750/4, merge=0/0, ticks=2602876/0, in_queue=2544040, util=100.00%
```

## if need very high disk performance, you can run 
##https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/disk-performance.html
##Skip below steps for our case
#dd if=/dev/zero bs=1M|tee /dev/nvme1n1|tee /dev/nvme2n1|tee > /dev/sdd
##Configuring drives for RAID initializes them by writing to every drive location. 
##When configuring software-based RAID, make sure to change the minimum reconstruction speed:
#[ec2-user@ip-10-0-0-142 ~]$ sudo su
#[root@ip-10-0-0-142 ec2-user]# echo $((30*1024)) > /proc/sys/dev/raid/speed_limit_min
