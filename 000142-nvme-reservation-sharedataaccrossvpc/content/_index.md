---
title : "Share Data Across VPCs with EBS Multi-Attach and NVMe Reservations"
weight : 1
chapter : false
---

# Share Data Across VPCs with EBS Multi-Attach and NVMe Reservations

#### Overview

Amazon EBS **Multi-Attach** lets you attach one Provisioned IOPS SSD volume (`io1` or `io2`) to several EC2 instances in the same Availability Zone. The instances do not need to be in the same VPC: an EBS volume is attached through the Nitro hardware of the host, not through the network of the VPC. You can use this to move data between two isolated environments, for example from production to test, without opening any network path between them.

When two instances can write to the same disk, you need a way to decide **who is allowed to write**. Multi-Attach enabled volumes support **NVMe reservations**, a set of standard storage fencing commands. One instance holds a reservation, and the volume itself rejects writes from the other instances.

In this lab you:

- Create two VPCs, `prod-vpc` and `test-vpc`, with private subnets in the same Availability Zone and no route to the internet.
- Run PostgreSQL 18 on one Amazon Linux 2023 instance in each VPC, and connect to them with Session Manager.
- Attach one Multi-Attach enabled `io2` volume to both instances.
- Back up a database from the production instance to the shared volume while it holds a **Write Exclusive** reservation.
- Prove that the test instance can read the volume but cannot write to it, then restore the database on the test instance.
- Hand the write access over to the test instance.

#### Architecture

![Lab architecture](/images/1-introduction/architecture.png)

{{% notice info %}}
This is an updated version of the workshop. The instances now run in private subnets and are reached with Session Manager, without public IP addresses, SSH keys or open inbound ports. The NVMe reservation part was rewritten with commands that were tested on a real volume. All screenshots were taken in **Asia Pacific (Tokyo) ap-northeast-1**.
{{% /notice %}}

{{% notice warning %}}
Cost: two `t3.micro` instances, six interface endpoints and a 4 GiB `io2` volume with 100 IOPS cost about 0.12 USD per hour in Tokyo. Do part 8 as soon as you finish.
{{% /notice %}}
