# EBS benchmark testing

## Background
Customer want to deployment Solr Cloud on AWS EC2 ask AWS to provide benchmark data about EBS volume which target IOPS required > 30,000 and Solr engine need SSD volumes.

[Benchmark EBS Volumes official guide](https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/benchmark_procedures.html)

1. Launch an EBS-optimized instance.
2. Create new EBS volumes.
3. Attach the volumes to your EBS-optimized instance.
4. Configure and mount the block device.
5. Install a tool to benchmark I/O performance.
6. Benchmark the I/O performance of your volumes.
7. Delete your volumes and terminate your instance.

[EBS RAID configuration](https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/raid-config.html)

Creating a RAID 0 array allows you to achieve a higher level of performance for a file system than you can provision on a single Amazon EBS volume. A RAID 1 array offers a "mirror" of your data for extra redundancy.

RAID 5 and RAID 6 are not recommended for Amazon EBS because the parity write operations of these RAID modes consume some of the IOPS available to your volumes. 

You can creat a RAID array with hight volumes such as (we recommend 6 volumes), which offers a high level of performance. But for this customer case testing, 4 volumes RAID will used. 

## Test case
1. Raid0 on instance storage for different instance type
- [i3.4xlarge 2 x 1900GiB NVMe SSD Raid0 on instance storage](script/i3.4xlarge-nvmessd-ebs-benchmark.md)
- [m5d.8xlarge 2 x 600GiB NVMe SSD Raid0 on instance storage](script/m5d.4xlarge-nvmessd-ebs-benchmark.md)

2. Compare io1 high IOPS v.s gp2 RAID0 v.s io1 RAID0
Note: If you need IOPS and high throughput you need use the large instance type to avoid instance limits 
- [m5.8xlarge io1 2048 provision IOPS 40000](script/m5.8xlarge-io1-ebs-benchmark.md)
- [m5.12xlarge gp2 3000GiB x 4 Raid0 provision IOPS 36000](script/m5.12xlarge-gp2-raid-ebs-benchmark.md)
- [m5.12xlarge io1 2048GiB provision IOPS 40000](script/m5.12xlarge-io1-ebs-benchmark.md)
- [m5.12xlarge io1 500GiB x4 Raid0, provision IOPS 44000](script/m5.12xlarge-io1-raid-ebs-benchmark.md)

3. Testing snapshot
- [c4.xlarge EBS Snapshot testing](script/c4.xlarge-gp2-io1-benchmark.md)


## Fio Testing Result
| instance type | IOPS randread 16 KB I/O | Throughput (MBps) randread 16 KB I/O | IOPS randwrite 16 KB I/O |  Throughput (MBps) randwrite 16 KB I/O | IOPS randread 4 KB I/O |  Throughput (MBps) randread 4 KB I/O | IOPS randwrite 4 KB I/O |  Throughput (MBps) randwrite 4 KB I/O| 
| :----------- | :----------- | :----------- | :----------- | :----------- | :----------- | :----------- | :----------- | :----------- | 
| m5.8xlarge  | 30327 | 485 | 30139 | 482 | N/A | N/A | N/A | N/A |
| m5.12xlarge io1 | 40308 | 644.9 | 40216 | 634.4 | N/A | N/A | N/A | N/A |
| m5.12xlarge gp2 raid0 | 36299 | 580.8 | 36155 | 578.5 | N/A | N/A | N/A | N/A |
| m5.12xlarge io1 raid0 | 40295 | 644.7 | 40219 | 643.5 | N/A | N/A | N/A | N/A |
| m5d.8xlarge Single NVMe SSD | 205133 | 3205.3 | 32233 | 515.7 | 437578 | 1709.3 | 117463 | 469.8 |
| m5d.8xlarge NVMe SSD Raid0 | 135220 | 2112.9 | 64470 | 1007.4 | 470710 | 1838.8 | 234654 | 938.6 |
| I3.4xlarge Single NVMe SSD | 216421 | 3381.7 | 48958 | 783.3 | 581387 | 2271.5 | 180559 | 722.2 | 
| I3.4xlarge NVMe SSD Raid0 | 243184 | 3799.8 | 97678 | 1526.3 | 824841 | 3222.4 | 357232 | 1395.5


## Reference

[Testing tool: fio](https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/benchmark_procedures.html)

[Testing tool: vdbench](https://www.oracle.com/technetwork/server-storage/vdbench-downloads-1901681.html)

[optimize the performance of my Amazon EBS Provisioned IOPS volumes](https://aws.amazon.com/premiumsupport/knowledge-center/optimize-ebs-provisioned-iops/)

[AWS IO Performance: What’s Bottlenecking Me Now? It not just the volume](https://medium.com/@TheLaddersEng/aws-io-performance-whats-bottlenecking-me-now-9b10b877bbef)

[Config raid](https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/raid-config.html)

[Optimizing Disk Performance for Instance Store Volumes](https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/disk-performance.html)
