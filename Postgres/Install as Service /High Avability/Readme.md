# High Avability Postgres Installation with Patroni 

**Postgres:**

**Patroni:**


Set hostname on each node like below
``` bash
 192.168.1 25 -> sudo hostnamectl set-hostname node01
 192.168.1 26 -> sudo hostnamectl set-hostname node02
 192.168.1 27 -> sudo hostnamectl set-hostname etcdhaproxy

```

Installation Overview
``` bash
Server_1        192.168.1.25        postgres + patroni 
Server_2        192.168.1.26        postgres + patroni
Server_3        192.168.1.27        etcd + HaProxy

```

Add your nodes info in **/etc/hosts** file like below
``` bash
sudo vim /etc/hosts

 192.168.1 25       node01
 192.168.1 26       node02
 192.168.1 27       etcdhaproxy

```

We install postgres on all associated nodes. For this,
Install postgres 15 on server_1 and server2
``` bash
sudo sh -c 'echo "deb http://apt.postgresql.org/pub/repos/apt $(lsb_release -cs)-pgdg main" > /etc/apt/sources.list.d/pgdg.list'
wget --quiet -O - https://www.postgresql.org/media/keys/ACCC4CF8.asc | sudo apt-key add -
sudo apt-get update
sudo apt -y install postgresql-15 postgresql-server-dev-15
```

Postgres 15 install with script on server_1 and server_2
``` bash
sudo su -

curl https://raw.githubusercontent.com/denizturkmen/Database-Notes/main/Postgres/Install%20as%20Service%20/High%20Avability/install-postgres.sh

```

Giving symbolic link all associated nodes
``` bash 
sudo ln -s /usr/lib/postgresql/15/bin/* /usr/sbin/

```


Must define user for postgres and replicator.
``` bash
su -
su - postgres
psql
ALTER USER postgres PASSWORD 'NuXQ3PZpX5YR';
CREATE USER replicator WITH ENCRYPTED PASSWORD 'NuXQ3PZpX5YR';

```

Stop postgresql service on server_1 and server_2
``` bash
# Stopped
sudo systemctl stop postgresql.service

# Status control
sudo systemctl status postgresql.service

```


Now install patroni using python on all associated nodes. For this,
Install patroni on server_1 and server_2
``` bash
sudo apt install python3-pip python3-dev libpq-dev -y
sudo pip3 install --upgrade pip
sudo pip install patroni --root-user-action=ignore
sudo pip install python-etcd --root-user-action=ignore
sudo pip install psycopg2 --root-user-action=ignore

---

sudo apt -y install python3 python3-pip
sudo apt install python3-pip python3-dev libpq-dev -y
sudo pip3 install --upgrade pip
sudo -H pip3 install --upgrade testresources
sudo -H pip3 install --upgrade setuptools
sudo -H pip3 install psycopg2
sudo -H pip3 install patroni
sudo -H pip3 install python-etcd

```

Etcd and haproxy install on server_3
``` bash
# package updated
sudo apt update && sudo apt upgrade -y

# Install
sudo apt install -y etcd haproxy

```

Configured etcd on server_3
``` bash
# Configure
sudo vim /etc/default/etcd

    ETCD_LISTEN_PEER_URLS="http://etcd_server_ip:2380,http://127.0.0.1:7001"
    ETCD_LISTEN_CLIENT_URLS="http://127.0.0.1:2379, http://etcd_server_ip:2379"
    ETCD_INITIAL_ADVERTISE_PEER_URLS="http://etcd_server_ip:2380"
    ETCD_INITIAL_CLUSTER="etcd0=http://etcd_server_ip:2380,"
    ETCD_ADVERTISE_CLIENT_URLS="http://etcd_server_ip:2379"
    ETCD_INITIAL_CLUSTER_TOKEN="cluster1"
    ETCD_INITIAL_CLUSTER_STATE="new"

# Restart etcd service
sudo systemctl restart etcd

# Status check service
sudo systemctl status etcd

```


Now Patroni uses a YML file to store its configuration. You will need to create a patroni.yml file on both server_1 and server_2:
Configure Patroni for on server_1
Opening **patroni.yml** with vim
**sudo vim /etc/patroni.yml**
``` bash
# For server_1
server_1-patroni.yml copy paste

# For server_2
server_2-patroni.yml copy paste

```

We need to create data directories. Then we need to authorize these directories.
``` bash
sudo mkdir -p /data/patroni
sudo chown postgres:postgres /data/patroni/
sudo chmod 700 /data/patroni/

```

We need to creta patroni servive. For this,
Opening **/etc/systemd/system/patroni.service**  with vim editor.
``` bash
[Unit]
Description=Runners to orchestrate a high-availability PostgreSQL
After=syslog.target network.target

[Service]
Type=simple

User=postgres
Group=postgres

ExecStart=/usr/local/bin/patroni /etc/patroni.yml
KillMode=process
TimeoutSec=30
Restart=no

[Install]
WantedBy=multi-user.targ

```

Then we have to start clustering.
``` bash
sudo systemctl daemon-reload
sudo systemctl enable patroni 
sudo systemctl enable postgresql
sudo systemctl start patroni
sudo systemctl start postgresql

# Patroni service status
sudo systemctl status patroni.service

```

Check patroni cluster
``` bash
sudo patronictl -c /etc/patroni.yml list

```

Finally, Configure haproxy
``` bash

# Configure
sudo vim /etc/haproxy/haproxy.cfg

# Started and status 
sudo systemctl start haproxy
sudo systemctl enable haproxy
sudo systemctl restart haproxy
sudo systemctl status haproxy

# Checking the config issue
sudo haproxy -c -V -f /etc/haproxy/haproxy.cfg

```

Postgresql connect command
``` bash
# Template
su - 
psql -h haproxy_ip_addresses -p haproxy_port -U postgres -W

For instance
su - 
psql -h 192.168.1.27 -p 5000 -U postgres -W

```

``` bash

## Create schema
$ CREATE SCHEMA SCHEMA_NAME;
$ CREATE SCHEMA sea;

## Schema Authorization
$ CREATE SCHEMA SCHEMA_NAME AUTHORIZATION USER_Name;
$ CREATE SCHEMA TEST AUTHORIZATION postgres;

## Creating a database
$ CREATE DATABASE DATABASE_NAME;
$ CREATE DATABASE person;

## Create Table
CREATE TABLE person (
	id int4 NULL,
	adi varchar NULL,
	soyadi varchar NULL,
	telefon varchar NULL,
	date_time date NULL
);


## Insert
$ INSERT INTO person (id, adi, soyadi, telefon, date_time) VALUES( 1, 'Deniz', 'TURKMEN', '0532 763 12 32', current_timestamp);
$ INSERT INTO person (id, adi, soyadi, telefon, date_time) VALUES( 2, 'Ozlem', 'ECEM', '0532 432 12 32', current_timestamp);

## Deletion of Schema
$ DROP SCHEMA SCHEMA_NAME

## Delete Database
$ DROP DATABASE DATABASE_NAME

```


Useful commands for postgresql
``` bash
\conninfo -> shows database information.
\q  -> Exit
\c  -> Select Database
\l  -> List databases
\l+ -> Detailed Databases listing
\dn -> Listing Schemas
\dt -> Listing tables
\df -> List functions
\dv -> List relations
\x  -> Shows output regularly


## Version
select version();

```