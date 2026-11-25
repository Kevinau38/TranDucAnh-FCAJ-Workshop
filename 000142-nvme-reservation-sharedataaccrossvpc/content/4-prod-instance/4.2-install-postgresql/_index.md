---
title : "Create the production database"
weight : 2
chapter : false
pre : " <b> 4.2 </b> "
---

1. Initialize PostgreSQL, start it, and create the database `test_erp` with a table `clients` and three rows:

   ```bash
   sudo postgresql-setup --initdb && sudo systemctl enable --now postgresql && systemctl is-active postgresql
   sudo -u postgres psql -c "CREATE DATABASE test_erp;"
   sudo -u postgres psql -d test_erp -c "CREATE TABLE clients (id SERIAL PRIMARY KEY, first_name VARCHAR(50), last_name VARCHAR(50), role VARCHAR(50));" -c "INSERT INTO clients (first_name, last_name, role) VALUES ('John', 'Smith', 'CEO'), ('Jane', 'Doe', 'CTO'), ('Minh', 'Nguyen', 'DBA');" -c "SELECT * FROM clients;"
   ```

   - (1) The PostgreSQL service is `active`.
   - (2) The database `test_erp` is created.
   - (3) The table `clients` holds three rows.

   ![PostgreSQL database](/images/4-prod-instance/4.2-postgresql.png)

PostgreSQL listens only on `localhost` by default. Nothing in this lab connects to it over the network, so you do not need to change `postgresql.conf` or `pg_hba.conf`.
