---
title : "AWS RDS PostgreSQL for Developers"
weight : 1
chapter : false
---
# Amazon RDS for PostgreSQL for developers

#### Overview

In this workshop, you write Node.js code that connects to an Amazon RDS for PostgreSQL database the way a production application should: over TLS with the server certificate checked, without a password in the code or in a configuration file, and with credentials that AWS rotates for you. Then you deploy a small web application that reads and writes the database.

![Workshop architecture](/images/1/architecture.png)

#### What you learn

- Let RDS manage the master password in AWS Secrets Manager, and read it from an EC2 instance with an IAM role.
- Connect from Node.js with the `pg` driver over TLS with `verify-full`.
- Find the endpoint and the credentials at run time with the AWS SDK for JavaScript v3.
- Build and run a CRUD web application with Express, Handlebars, and RDS for PostgreSQL.
