---
title : "Connect to RDS with pgAdmin 4 through SSH tunnel"
weight : 2
chapter : false
pre : " <b> 3.2. </b> "
---

#### Overview

`rdspg-lab-db` is in a private subnet and has **Public access** set to **No**, so your computer cannot connect to it directly. In this step, pgAdmin 4 connects to the database through an **SSH tunnel** via the bastion host:

```
pgAdmin (your computer) --SSH :22--> Bastion host (public subnet) --:5432--> RDS (private subnet)
```

pgAdmin opens a temporary local port on your computer (for example `127.0.0.1:57211`). Traffic sent to that port travels inside the SSH connection to the bastion host, which forwards it to the RDS endpoint on port 5432. RDS sees the connection coming from the bastion host, which `rds-sg` allows.

| Tab | Field | Value |
|---|---|---|
| Connection | Host name/address | RDS **endpoint** |
| SSH Tunnel | Tunnel host | **Public IPv4 address** of the bastion host |

{{% notice warning %}}
Do not mix up the two hosts. **Host name/address** is the RDS endpoint. **Tunnel host** is the public IP address of the bastion host.
{{% /notice %}}

#### Check the SSH connection first

Before you configure pgAdmin, confirm that you can reach the bastion host over SSH from Windows PowerShell.

1. Move the key pair to your `.ssh` folder and restrict its permissions. OpenSSH refuses a private key that other users can read.

    ```powershell
    mkdir $HOME\.ssh -Force
    Move-Item "<path-to>\rdspg-lab-key.pem" "$HOME\.ssh\rdspg-lab-key.pem"
    icacls "$HOME\.ssh\rdspg-lab-key.pem" /inheritance:r
    icacls "$HOME\.ssh\rdspg-lab-key.pem" /grant:r "$($env:USERNAME):(R)"
    ```

2. Connect to the bastion host. Replace `<BASTION_PUBLIC_IP>` with the public IPv4 address from step 2.6. Type `yes` when SSH asks you to confirm the host key.

    ```powershell
    ssh -i "$HOME\.ssh\rdspg-lab-key.pem" ec2-user@<BASTION_PUBLIC_IP>
    ```

3. On the bastion host, check that the endpoint resolves to a private IP address and that port 5432 is open. Replace `<RDS_ENDPOINT>` with your endpoint.

    ```bash
    getent hosts <RDS_ENDPOINT>
    timeout 5 bash -c '</dev/tcp/<RDS_ENDPOINT>/5432' && echo "PORT 5432 OPEN"
    ```

    The first command returns an address in `10.0.128.0/20` or `10.0.144.0/20`. The second command prints `PORT 5432 OPEN`. Type `exit` to leave the bastion host.

{{% notice tip %}}
If SSH returns `Connection timed out`, your public IP address has probably changed. Edit the inbound rule of `bastion-sg` and choose **My IP** again.
{{% /notice %}}

#### Register the server in pgAdmin

4. Open pgAdmin 4:
    - (1) In **Quick Links**, choose **Add New Server**.

    ![Add New Server](/images/3-connect/3.2/1.png)

5. In the **General** tab:
    - (1) **Name**: `rdspg-lab-db`

    ![General tab](/images/3-connect/3.2/2.png)

6. In the **Connection** tab:
    - (1) Choose the **Connection** tab.
    - (2) **Host name/address**: the RDS endpoint
    - (3) **Port**: `5432`
    - (4) **Maintenance database**: `postgres`
    - (5) **Username**: `postgres`
    - (6) **Password**: the master password. Keep **Save password?** off.

    ![Connection tab](/images/3-connect/3.2/3.png)

7. In the **SSH Tunnel** tab:
    - (1) Choose the **SSH Tunnel** tab.
    - (2) Turn on **Use SSH tunneling**.
    - (3) **Tunnel host**: the public IPv4 address of the bastion host
    - (4) **Tunnel port**: `22`
    - (5) **Username**: `ec2-user`
    - (6) **Authentication**: **Identity file**
    - (7) **Identity file**: `C:\Users\<your-user>\.ssh\rdspg-lab-key.pem`
    - (8) Choose **Save**.

    ![SSH Tunnel tab](/images/3-connect/3.2/4.png)

{{% notice note %}}
Keep the SSH **Password** empty and **Prompt for identity file password?** off. The key pair created by AWS has no passphrase.
{{% /notice %}}

#### Verify the connection

8. In **Object Explorer**, expand **rdspg-lab-db** > **Databases**. The **Dashboard** tab shows live activity from the database.

    ![Connected](/images/3-connect/3.2/5.png)

| Database | Description |
|---|---|
| `postgres` | The default database that PostgreSQL always creates |
| `rdsadmin` | Reserved for Amazon RDS to manage the instance. You cannot connect to it |

{{% notice info %}}
If pgAdmin shows **fe_sendauth: no password supplied**, the SSH tunnel works but the **Password** in the **Connection** tab is empty. Enter the master password and save again.
{{% /notice %}}
