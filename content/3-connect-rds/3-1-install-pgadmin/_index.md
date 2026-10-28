---
title : "Install pgAdmin 4"
weight : 1
chapter : false
pre : " <b> 3.1. </b> "
---

#### Overview

[pgAdmin 4](https://www.pgadmin.org/) is a popular open-source administration and development tool for PostgreSQL. In this step, you install pgAdmin 4 on your computer. In the next step, you use it to connect to `rdspg-lab-db` through the bastion host.

| Area | Use |
|---|---|
| **Object Explorer** | Browse servers, databases, schemas, and tables as a tree |
| **Dashboard, Properties, SQL** tabs | View details of the object selected in Object Explorer |
| **Add New Server** | Register a connection to a PostgreSQL server |
| **Query Tool** | Write and run SQL statements |

#### Download pgAdmin 4

1. Open the [pgAdmin 4 for Windows download page](https://www.pgadmin.org/download/pgadmin-4-windows/):
    - (1) Choose **pgAdmin 4 v9.17**.

    ![pgAdmin download page](/images/3-connect/3.1/1.png)

2. On the file list page:
    - (1) Choose `pgadmin4-9.17-x64.exe` to download the installer.

    ![pgAdmin installer file](/images/3-connect/3.1/2.png)

{{% notice warning %}}
This workshop uses **pgAdmin 4 v9.17**, not the latest v9.18. When this workshop was updated, v9.18 on Windows failed to create an SSH tunnel with an identity file (reported as PostgreSQL [BUG #19711](https://www.postgresql.org/message-id/19711-ba992e3a50a3fe0f%40postgresql.org)), while v9.17 worked with the same settings. If a newer version fixes the issue, you can use it instead.
{{% /notice %}}

![pgAdmin 4 v9.18 SSH tunnel error](/images/3-connect/3.1/3.png)

{{% notice note %}}
The installer is signed by the pgAdmin project lead (**Open Source Developer, David John Page**). If Windows SmartScreen shows a warning because it cannot be reached, check the publisher and choose **Run**.
{{% /notice %}}

#### Install pgAdmin 4

3. Run the installer. In **Select install mode**:
    - (1) Choose **Install for me only (recommended)**.

    ![Select install mode](/images/3-connect/3.1/4.png)

{{% notice info %}}
**Install for me only** does not require administrator privileges. **Install for all users** installs pgAdmin for every account on the computer and requires administrator privileges.
{{% /notice %}}

4. Keep the default options in the remaining screens, then choose **Install** and **Finish**.

#### Verify the installation

5. Open **pgAdmin 4** from the Start menu. In **Object Explorer**, **Servers** is empty because you have not registered any server yet.

    ![pgAdmin 4](/images/3-connect/3.1/5.png)

{{% notice tip %}}
If PostgreSQL is already installed on your computer, pgAdmin may add those local servers to **Servers** automatically. To keep the list clean, right-click each one and choose **Remove Server**. This only removes the server from the pgAdmin list. It does not uninstall PostgreSQL or delete any data.
{{% /notice %}}
