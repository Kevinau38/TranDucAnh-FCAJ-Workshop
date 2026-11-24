---
title : "Create a Lightsail database"
weight : 1
chapter : false
pre : " <b> 1. </b> "
---

A Lightsail **managed database** runs MySQL or PostgreSQL for you: AWS installs the engine, applies patches, and takes daily backups. WordPress stores its posts and settings in this database, so you create it first. It takes about 10–15 minutes to become available, and you deploy the WordPress instance while it starts.

1. Open the [Lightsail console](https://lightsail.aws.amazon.com/). In the left menu, choose **Databases**, then choose **Create database**.

![Databases page](/images/1/1-databases.png)

2. Under **Select a Region**, choose **Singapore (ap-southeast-1)**. Keep the default Availability Zone.

![Select the Region](/images/1/2-region.png)

3. Under **Pick your database**, choose **MySQL** and keep the latest 8.4 version.

![Pick MySQL](/images/1/3-engine.png)

4. Do **not** choose **Specify login credentials**. Lightsail then creates the user `dbmasteruser` and a strong password, and stores the password for you. In the next chapters you read it with the AWS CLI, so you never copy it by hand.

5. Choose **Specify the master database name** and enter `wordpress`. Lightsail adds the prefix `db`, so the database name is `dbwordpress`.

![Master database name](/images/1/5-db-name.png)

6. Under **Choose your database plan**, keep **Standard** and choose the **USD 15** plan (1 GB memory, 2 vCPUs, 40 GB SSD).

![Database plan](/images/1/4-plan.png)

{{% notice info %}}
**High Availability** plans keep a standby copy in a second Availability Zone and cost twice as much. Standard is enough for this lab.
{{% /notice %}}

7. Under **Identify your database**, enter `fcj-lightsail-db`. Under **Key-value tags**, add the tag `Project` = `fcj-lightsail`. Choose **Create database**.

![Name and tag](/images/1/6-name-tag.png)

8. The database shows **Creating**. Continue with the next chapter while it starts.

![Database creating](/images/1/7-creating.png)

9. When the status is **Available**, open the database. The **Connect** tab shows the user name `dbmasteruser`, the hidden password, and the endpoint that WordPress connects to.

![Connection details](/images/1/8-connect.png)

{{% notice note %}}
A Lightsail database accepts connections only from your Lightsail resources in the same Region by default (**Public mode** is off). You can check this on the **Networking** tab.
{{% /notice %}}
