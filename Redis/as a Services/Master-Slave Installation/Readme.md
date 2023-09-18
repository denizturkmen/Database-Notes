# Master-Slave Installation and Configuration Redis on Ubuntu

What is the **REDIS?**



Set hostname on each node like below
``` bash
 192.168.1 25 -> sudo hostnamectl set-hostname redis-master
 192.168.1 26 -> sudo hostnamectl set-hostname redis-slave

```

Installation Overview
``` bash
Server_1        192.168.1.25        Redis Master
Server_2        192.168.1.26        Redis Slave

```

Add your nodes info in **/etc/hosts** file like below
``` bash
sudo vim /etc/hosts

 192.168.1 25       redis-master
 192.168.1 26       redis-slave

```


The following run commands for **redis install** 

**Note:** We install redis on all nodes
``` bash

# 1st way: install
sudo apt-add-repository ppa:chris-lea/redis-server
sudo apt update && sudo apt upgrade -y
sudo apt install -y redis-server

# 2nd way: install on ubuntu/debian
sudo apt install -y lsb-release curl gpg
curl -fsSL https://packages.redis.io/gpg | sudo gpg --dearmor -o /usr/share/keyrings/redis-archive-keyring.gpg

echo "deb [signed-by=/usr/share/keyrings/redis-archive-keyring.gpg] https://packages.redis.io/deb $(lsb_release -cs) main" | sudo tee /etc/apt/sources.list.d/redis.list

sudo apt-get update
sudo apt-get install -y redis-server

#reboot machine
sudo reboot

# enable service
sudo systemctl enable redis-server

# start service
sudo systemctl start redis-server

# restart
sudo systemctl restart redis-server

# status
sudo systemctl status redis-server

```

Issue: Increased maximum number of open files to XXXXX
``` bash
# exec redis-server
redis-server

# fixing incease size
sudo sysctl -w net.core.somaxconn=65535
sudo su
echo never > /sys/kernel/mm/transparent_hugepage/enabled

# Checking
redis-cli
    ping
```

Let's open the redis port from the firewall for master access
``` bash
# updated package
sudo apt update

# install ufw
sudo apt install ufw

# enable port
sudo ufw enable
sudo ufw allow 22
sudo ufw allow 6379
or
sudo ufw allow in on enp0s3 to any port 6379

# status
sudo ufw status

```


Configuration on **master node** for redis
``` bash
# go to related directory
cd /etc/redis/

# Open config
sudo vim redis.conf

    bind master_ip
    protected-mode no
    port 6379
    requirepass redis123
    save 900 1
    save 300 10
    save 60 10000

```

Configuration on **slave node** for redis
``` bash
# go to related directory
cd /etc/redis/

# Open config
sudo vim redis.conf

    bind slave_ip
    protected-mode no
    port 6379
    requirepass redis123
    masterauth redis123
    save 900 1
    save 300 10
    save 60 10000
    replicaof master_ip redis_port

# restart
sudo systemctl restart redis-server

# status
sudo systemctl status redis-server

```

Master-Slave checking
``` bash
# master
redis-cli -h master_ip redis_port
ping
    auth password
    info replication
# slave
redis-cli -h master_ip redis_port
ping
    auth password
    info replication

```

Usefull command
``` bash
# on master
set master master-slave-test
    keys *      
    get master  

# on slave
set slave write-yok



```



# Referance
``` bash


```