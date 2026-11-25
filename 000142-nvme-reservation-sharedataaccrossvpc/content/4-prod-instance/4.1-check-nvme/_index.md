---
title : "Check the NVMe device"
weight : 1
chapter : false
pre : " <b> 4.1 </b> "
---

1. In the EC2 console, select `prod-postgres` and choose **Connect** → **Session Manager** → **Connect**. A terminal opens in the browser as `ssm-user`. Start a login shell:

   ```bash
   bash -l
   cd ~
   ```

2. Install `nvme-cli` and PostgreSQL 18 from the Amazon Linux 2023 repositories. The packages come through the S3 gateway endpoint:

   ```bash
   sudo dnf install -y -q nvme-cli postgresql18-server
   ```

3. Find the shared volume and check its reservation capabilities:

   ```bash
   lsblk -o NAME,SIZE,TYPE,MOUNTPOINTS
   sudo nvme list
   sudo nvme id-ns -H /dev/nvme1n1 | grep -A 8 rescap
   ```

   - (1) `nvme1n1` is the 4 GiB volume, not yet formatted or mounted.
   - (2) The serial number (`SN`) of `/dev/nvme1n1` is the volume ID without the dash, `vol0776038ead9621b7d`. This is how you match a device to an EBS volume.
   - (3) `rescap` (reservation capabilities) is `0x7f`: the volume supports all six reservation types.

   ![NVMe devices and reservation capabilities](/images/4-prod-instance/4.1-nvme-list.png)

{{% notice note %}}
The terminal screenshots are shown with a light background to make them easier to read. Your Session Manager terminal has a dark background.
{{% /notice %}}
