# Postgres Master-Slave Installation and Configuration


Set hostname on each node like below
``` bash
 192.168.1 25 -> sudo hostnamectl set-hostname postgres-master
 192.168.1 26 -> sudo hostnamectl set-hostname postgres-slave

```

Installation Overview for Postgres
``` bash
Server_1-Master        192.168.1.25        postgres-master
Server_2-Slave         192.168.1.26        postgres-slave 

```

Add your nodes info in **/etc/hosts** file like below
``` bash
sudo vim /etc/hosts

192.168.1.25        postgres-master
192.168.1.26        postgres-slave 

```

Postgresql must install two server node. For this.
The following commands run;

``` bash
# Latest version install
sudo apt update
sudo apt-get install -y postgresql postgresql-client postgresql-contrib


# Specific version install. for instance; for postgres 15
sudo sh -c 'echo "deb http://apt.postgresql.org/pub/repos/apt $(lsb_release -cs)-pgdg main" > /etc/apt/sources.list.d/pgdg.list'
wget --quiet -O - https://www.postgresql.org/media/keys/ACCC4CF8.asc | sudo apt-key add -
sudo apt-get update
sudo apt -y install postgresql-15 postgresql-client-15 postgresql-contrib-15

```

Giving symbolic link all associated nodes
``` bash 
sudo ln -s /usr/lib/postgresql/15/bin/* /usr/sbin/

```

Then, **Master** postgres backup to config. 
``` bash
# Postgres install path 
cd /etc/postgresql/15/main/

# List
ls -al or ll

# Backup config
sudo cp postgresql.conf old_postgresql.conf

# Openning postgres.conf with vim
sudo vim postgresql.conf
    listen_addresses='master_ip'
    listen_addresses='192.168.1.25'

```

Must define user for postgres.
``` bash
sudo su
su - postgres
psql

# Instance
CREATE USER user_name WITH REPLICATION ENCRYPTED PASSWORD 'password';

# Create user
CREATE USER masteruser WITH REPLICATION ENCRYPTED PASSWORD 'NuXQ3PZpX5YR';

# list of User
\du+

```

**Note:** Password is important because we will use this password from **slave** machine

Configure pg_hba. We enter the slave machine information on Master machine
``` bash
sudo vim pg_hba.conf

# last line add
host    replication     user_name            slave_ip                scram-sha-256

host    replication     all             192.168.1.26/24                 scram-sha-256

```

Start postgresql service on master
``` bash
# Start
sudo systemctl start postgresql.service

# Enable
sudo systemctl enable postgresql.service

# Restart
sudo systemctl restart postgresql.service

# Status control
sudo systemctl status postgresql.service
```





Now, Slave machine congfiguration.
``` bash
# status
sudo systemctl status postgresql

# Remove conf.
sudo su
cp -r /var/lib/postgresql/15/main/ /var/lib/postgresql/15/main_old/
rm -rf /var/lib/postgresql/15/main/

# Backup slave machine before postgres service start
sudo systemctl start postgresql.service

# Postgres
su - postgres

pg_basebackup -h master_ip -D /var/lib/postgresql/15/main/ -U user_name -P -v -R -X stream -C -S slave_name

pg_basebackup -h 192.168.1.25 -D /var/lib/postgresql/15/main/ -U masteruser -P -v -R -X stream -C -S slave1

# Restart
sudo systemctl restart postgresql.service
```

We are trying to create database from database slave machine,
``` bash
sudo su
su - postgres
psql

# Create
create database deneme;
```

Create databse and table on Postgres master
``` bash
sudo su
su - postgres
psql
# Create database
create database test;

# Database list
\l+ 

# Changed schmea
\c test; 

# Create Table
Create table Person ( name text, surname text);

# Table List
\dt 

# Insert to table
INSERT INTO Person VALUES ('deniz','turkmen');

# List table
Select * from Person;

```

Teston slave
``` bash
sudo su
su - postgres
psql
# Select schema
\c test;

# table list
\dt+

# Show data
select * from Person;
```

Replication connection check
``` bash
select * from pg_replication_slots ;

```