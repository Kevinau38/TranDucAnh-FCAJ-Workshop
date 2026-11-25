---
title : "Introduction"
weight : 1
chapter : false
pre : " <b> 1. </b> "
---

#### Multi-Attach

A Multi-Attach enabled volume can be attached to up to **16 instances** at the same time. The main rules:

| Rule | Detail |
|---|---|
| Volume type | Only Provisioned IOPS SSD: `io2` (recommended) or `io1` |
| Instances | Must be built on the **Nitro System** and be in the **same Availability Zone** as the volume. They can be in different VPCs or subnets |
| Boot volume | A Multi-Attach volume cannot be a boot volume, and you cannot enable Multi-Attach while launching an instance |
| File system | Standard file systems such as **XFS** and **ext4** are not designed to be mounted by several servers at the same time. Use a cluster file system, or make sure that only one instance has the file system mounted read-write |
| Cost | No extra charge. You pay the normal price of the `io2` volume and its provisioned IOPS |

In this lab the volume is formatted with XFS. The production instance mounts it, writes the backup and unmounts it. The test instance then mounts it read-only. The two instances never write to the file system at the same time.

#### NVMe reservations

NVMe reservations are part of the NVMe standard. Each instance first **registers** a key on the volume. A registered instance can then **acquire** a reservation of a given type. The volume enforces the reservation for every I/O it receives.

| Command | `nvme-cli` command | Purpose |
|---|---|---|
| Reservation Register | `nvme resv-register` | Register, unregister or replace the key of this instance |
| Reservation Acquire | `nvme resv-acquire` | Take a reservation, or preempt the reservation of another key |
| Reservation Release | `nvme resv-release` | Release or clear the reservation |
| Reservation Report | `nvme resv-report` | Show the registered keys and who holds the reservation |

The reservation type (`--rtype`) decides what the other instances can do:

| rtype | Type | Holder | Other registered instances | Unregistered instances |
|---|---|---|---|---|
| 1 | Write Exclusive | Read and write | Read only | Read only |
| 2 | Exclusive Access | Read and write | No access | No access |
| 3 | Write Exclusive - Registrants Only | Read and write | Read and write | Read only |
| 4 | Exclusive Access - Registrants Only | Read and write | Read and write | No access |
| 5 | Write Exclusive - All Registrants | Read and write | Read and write | Read only |
| 6 | Exclusive Access - All Registrants | Read and write | Read and write | No access |

The lab uses type **1, Write Exclusive**: the production instance is the only writer, and the test instance can still read the backup.

#### Requirements

- NVMe reservations work on Multi-Attach enabled `io2` volumes created after September 18, 2023, and `io1` volumes created after October 1, 2026. For older volumes, detach the volume from all instances and attach it again.
- Supported operating systems: Amazon Linux 2 and later, RHEL 8.3 and later, SUSE Linux Enterprise 12 SP3 and later, Windows Server 2016 and later. The lab uses **Amazon Linux 2023**.
- Using NVMe reservations has no extra cost.
