Postgresql Install on Kubernetes

All you have to do is run the following command.
``` bash
# Install
kubectl apply -f postgres-full.yaml

# Checking
kubectl get pods -n postgres

# Checking Deploynent
kubectl get deployments -n postgres

# Checking Service
kubectl get services -n postgres

# Checking Persistent Volume
kubectl get pv -n postgres

# Checking Persistent Volume Claims
kubectl get pvc -n postgres

# Pod exec
kubectl get pods -it -n namespace pod_name -- bash
    psql -U postgres

```


Adding user for postgres
``` bash
CREATE USER root SUPERUSER PASSWORD 'root';
CREATE DATABASE root;
GRANT ALL PRIVILEGES ON DATABASE root TO root;

```

``` bash

## Create schema
$ CREATE SCHEMA SCHEMA_NAME;
$ CREATE SCHEMA deniz;

## Schema Authorization
$ CREATE SCHEMA SCHEMA_NAME AUTHORIZATION USER_Name;
$ CREATE SCHEMA TEST AUTHORIZATION postgres;

## Creating a database
$ CREATE DATABASE DATABASE_NAME;
$ CREATE DATABASE person;
$ \l+

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



Backing up and restore
``` bash
# Backup
$ kubectl exec pod_name -n postgresql -- bash -c "pg_dump -U postgres person" > database.sql

# Restore
$ cat database.sql | kubectl exec -i pod_name -n postgresql -- psql -U postgres -d person
```