---
title : "Introduction"
weight : 1
chapter : false
pre : " <b> 1. </b> "
---


#### Amazon S3 performance

Amazon S3 scales automatically to high request rates. An application can reach at least **3,500 PUT/COPY/POST/DELETE** and **5,500 GET/HEAD** requests per second **per partitioned prefix**, and there is no limit to the number of prefixes in a bucket. To go faster you **parallelize**: split a large object into parts and upload the parts at the same time (multipart upload), spread objects over several prefixes, or run several transfers at once. Large data lake workloads use this to fill the network interface of an instance, up to 100 Gb/s on a single EC2 instance.

The AWS CLI does this for you with the `aws s3` commands (`cp`, `sync`, `mv`). Three settings control how hard it works:

| Setting | Meaning |
|---|---|
| `max_concurrent_requests` | How many requests the CLI sends at the same time (default 10) |
| `multipart_threshold` | Objects larger than this are uploaded in parts (default 8 MB) |
| `multipart_chunksize` | Size of each part (default 8 MB) |

#### Amazon EFS performance

Amazon EFS is a serverless, elastic NFS file system. Its performance depends on how you create it:

| Option | Choices |
|---|---|
| Storage class | **EFS Standard** (data in several Availability Zones) or **EFS One Zone** (one Availability Zone); each has an Infrequent Access and an Archive class with higher first-byte latency |
| Performance mode | **General Purpose** (lowest latency, recommended) or **Max I/O** (only for older workloads that need extreme parallelism) |
| Throughput mode | **Elastic** (scales with the workload, pay per GB transferred), **Provisioned** (fixed throughput) or **Bursting** (throughput grows with the amount of data stored, with burst credits) |

Because every NFS operation travels over the network, EFS has higher latency per operation than a local EBS volume. The way to get high throughput and IOPS from EFS is the same as for S3: **many threads, many files, many directories** at the same time.

The lab file system uses **EFS Standard**, **General Purpose** and **Bursting**. A new file system starts with enough burst credits for the whole lab.
