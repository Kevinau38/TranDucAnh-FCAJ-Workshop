---
title : "Amazon EFS performance"
weight : 4
chapter : false
pre : " <b> 4. </b> "
---


In this part you write to the EFS file system mounted at `/efs` in different ways. Every NFS operation goes over the network to EFS, so a single operation takes longer than on a local disk. EFS makes up for it by serving many operations in parallel: the more threads, files and directories you use at the same time, the more throughput and IOPS you get, up to the limit of the file system.

The lab file system uses the **Bursting** throughput mode. A small file system like this one can burst to about 100 MiB/s (around 110 MB/s), which is the ceiling you will see in some of the tests. With **Elastic** throughput, the same tests can go much higher.
