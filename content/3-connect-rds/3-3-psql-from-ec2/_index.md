---
title : "Connect to RDS with psql from the App server"
weight : 3
chapter : false
pre : " <b> 3.3. </b> "
---

#### Overview

In this step, you connect to `rdspg-lab-db` with the PostgreSQL client `psql` from `rdspg-app-server`. This step verifies two parts of the design from chapter 2:

| Design | How you verify it |
|---|---|
| The App server is in a private subnet without a NAT gateway | It cannot reach the internet, but it can still install `psql` through the **S3 Gateway endpoint** |
| `rds-sg` allows port 5432 from `app-sg` | `psql` connects, and RDS sees the App server's private IP address as the client |

The App server has no public IP address, so you reach it through the bastion host with SSH **ProxyJump**. The private key stays on your computer. Do not copy the private key to the bastion host.

#### Connect to the App server through the bastion host

1. On your computer, create the SSH configuration file. Replace `<BASTION_PUBLIC_IP>` with the public IPv4 address of `rdspg-bastion-host`, and `<APP_SERVER_PRIVATE_IP>` with the private IPv4 address of `rdspg-app-server`.

    ```powershell
    @"
    Host rdspg-bastion
        HostName <BASTION_PUBLIC_IP>
        User ec2-user
        IdentityFile ~/.ssh/rdspg-lab-key.pem

    Host rdspg-app
        HostName <APP_SERVER_PRIVATE_IP>
        User ec2-user
        IdentityFile ~/.ssh/rdspg-lab-key.pem
        ProxyJump rdspg-bastion
    "@ | Set-Content -Path "$HOME\.ssh\config" -Encoding ascii
    ```

{{% notice note %}}
Create the file with PowerShell instead of Notepad. Notepad may save it as `config.txt`, which SSH does not read.
{{% /notice %}}

2. Connect to the App server. Type `yes` when SSH asks you to confirm the host key.

    ```powershell
    Get-Content $HOME\.ssh\config
    ssh rdspg-app
    ```

    SSH connects to the bastion host first and then to the App server. The prompt shows the private IP address of the App server.

    ![SSH to the App server](/images/3-connect/3.3/1.png)

#### Install the PostgreSQL client

3. On the App server, check internet access, list the available PostgreSQL client versions, and install the client that matches the RDS major version.

    ```bash
    curl -m 5 -sI https://www.google.com || echo "NO INTERNET"
    sudo dnf list 'postgresql1?'
    sudo dnf install -y postgresql18
    psql --version
    ```

    The App server has no internet access, but `dnf` still installs `postgresql18` because Amazon Linux 2023 repositories are hosted on Amazon S3.

    ![Install psql](/images/3-connect/3.3/2.png)

#### Copy the RDS certificate bundle

To verify the server certificate of RDS (`sslmode=verify-full`), `psql` needs the RDS certificate bundle `global-bundle.pem`. The App server cannot download it from the internet, so you download it on your computer and copy it with `scp` through the bastion host.

4. On your computer, run:

    ```powershell
    Invoke-WebRequest https://truststore.pki.rds.amazonaws.com/global/global-bundle.pem -OutFile "$HOME\Downloads\global-bundle.pem"
    scp "$HOME\Downloads\global-bundle.pem" rdspg-app:~/
    ```

    ![Copy the certificate bundle](/images/3-connect/3.3/3.png)

#### Connect to RDS with psql

5. On the App server, connect to the database. Replace the endpoint with your own and enter the master password when prompted.

    ```bash
    export RDSHOST="<RDS_ENDPOINT>"
    psql "host=$RDSHOST port=5432 dbname=postgres user=postgres sslmode=verify-full sslrootcert=./global-bundle.pem"
    ```

6. Check which IP addresses the connection uses:

    ```sql
    SELECT inet_server_addr() AS rds_ip, inet_client_addr() AS client_ip;
    ```

    The connection uses **TLSv1.3**. `rds_ip` is the private IP address of RDS, and `client_ip` is the private IP address of the App server.

    ![psql connected](/images/3-connect/3.3/4.png)

7. Show the connection details:

    ```sql
    \conninfo
    ```

    ![Connection information](/images/3-connect/3.3/5.png)

| Field | Value | Meaning |
|---|---|---|
| Host Address | `10.0.130.72` | The endpoint resolves to a private IP address inside the VPC |
| SSL Connection | `true` | The connection is encrypted, and the server certificate is verified |
| Superuser | `off` | The master user is not a PostgreSQL superuser. On Amazon RDS, it has the `rds_superuser` role, and AWS keeps the real superuser to manage the instance |

8. Type `\q` to exit `psql`, then `exit` to leave the App server.

{{% notice tip %}}
In `psql`, run one meta-command such as `\conninfo` per line. If you paste several lines at once, `psql` treats the following text as arguments of the meta-command.
{{% /notice %}}

#### Troubleshooting

| Error | Cause | Fix |
|---|---|---|
| `Connection timed out` when you connect to the bastion host | Your public IP address changed | Edit the inbound rule of `bastion-sg` and choose **My IP** again |
| `Could not resolve hostname rdspg-app` | You ran the command on an EC2 instance instead of your computer | Check the prompt. Run `ssh rdspg-app` from PowerShell on your computer |
| SSH works the first time, then fails after you stop and start the bastion host | The public IP address of the bastion host changed | Update `HostName` of `rdspg-bastion` in `$HOME\.ssh\config` |
