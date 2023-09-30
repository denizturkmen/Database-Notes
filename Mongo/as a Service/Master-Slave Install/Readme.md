# MongoDB Master-Slave Installation and Configuration Guide


Set hostname on each node like below
``` bash
 192.168.1 40 -> sudo hostnamectl set-hostname master-primary
 192.168.1 41 -> sudo hostnamectl set-hostname slave-secondary-1
 192.168.1 42 -> sudo hostnamectl set-hostname slave-secondary-2

```

Installation Overview
``` bash
  Hostname             Role            IP            CPU    RAM    
master-primary       Primary        192.168.1.40      4      4
slave-secondary-1    Secondary      192.168.1.41      2      3
slave-secondary-2    Secondary      192.168.1.42      2      3

```

Add your nodes info in **/etc/hosts** file like below
``` bash
sudo vim /etc/hosts

 192.168.1.40       master-primary
 192.168.1.41       slave-secondary-1
 192.168.1.42       slave-secondary-2

```

Now let's install MongoCE** on all nodes.
``` bash
# dowland
wget -qO - https://www.mongodb.org/static/pgp/server-5.0.asc | sudo apt-key add -

# install gnupg
sudo apt-get install gnupg

# apt package add to apt
echo "deb [ arch=amd64,arm64 ] https://repo.mongodb.org/apt/ubuntu focal/mongodb-org/5.0 multiverse" | sudo tee /etc/apt/sources.list.d/mongodb-org-5.0.list
sudo apt update

# install mongodb
sudo apt-get install -y mongodb-org

# checking mongodb version
mongod --version

# reload deamon
sudo systemctl daemon-reload

# started mongo
sudo systemctl start mongod

# enabled mongo
sudo systemctl enable mongod

# stopped mongo
sudo systemctl stop mongod

# status mongo
sudo systemctl status mongod

# restarted mongo
sudo systemctl restart mongod

```

We need to create authentication to ensure the connection between servers with **openssl**

Run the following command on the host machine.
``` bash
# create key
openssl rand -base64 741 > /home/master1/prirepkey

# owner mongod
sudo chown mongodb:mongodb /home/master1/prirepkey

# write and read permission this key
sudo chmod 600 /home/master1/prirepkey

# Send slave-1 and slave-2 with key file scp
sudo scp -r prirepkey worker1@192.168.1.41:/home/worker1
sudo scp -r prirepkey worker2@192.168.1.42:/home/worker2

# owner of slave-1 and grant permission
sudo chown mongodb:mongodb /home/worker1/prirepkey
sudo chmod 600 /home/worker1/prirepkey

# owner of slave-2 and grant permission
sudo chown mongodb:mongodb /home/worker2/prirepkey
sudo chmod 600 /home/worker2/prirepkey

```


Now We need to create root user from master machine
``` bash
# mongo
mongo

# switched toı admin user;
use admin;

# create to admin user
db.createUser({
  user: "admin",
  pwd: "denizturkmen",
  roles: [{role: "root", db: "admin"}]
})

# checking admin user
mongo -u admin -p
or 
mongo -u admin -p denizturkmen


```


Configuration **mongod.conf** all on the nodes.
``` bash
# diretory create
sudo mkdir -p /etc/mongo/key

# cp to key 
sudo cp prirepkey /etc/mongo/key/

# owner mongod for keyfile
cd /etc/mongo/key
sudo chown -R mongodb:mongodb prirepkey 

# go to directory
cd /etc

# edit mongo.conf on all node
cd /etc
sudo vim mongod.conf

# network interfaces
net:
  port: 27017
  bindIp: 0.0.0.0

# security
security:
  authorization: enabled
  keyFile: "/etc/mongo/key/prirepkey"
 
# replication
replication:
  replSetName: slaverep

# restarted mongo
sudo service mongod restart

# status mongo
sudo service mongod status

```

Controlling secondary machine from primary machine
``` bash
# command
mongo --host hostname -p port_number

# for slave-1
mongo --host slave-secondary-1 -p 27017

# for slave-2
mongo --host slave-secondary-2 -p 27017


```

Starting replication nodes from master machine 
``` bash
# login to mongo
mongo -u admin -p denizturkmen

# start replication
rs.initiate()

# add replication slave-1 node
rs.add("slave-secondary-1:27017")

# add repliation slave-2 node
rs.add("slave-secondary-2:27017")

# showing all node
rs.conf()
or
rs.status()

```

Testing replication on slave-1 and slave-2
``` bash
# slave-1
mongo -u admin -p denizturkmen

# verify replication
rs.secondaryOk()

# show databases
show dbs

# slave-2
mongo -u admin -p denizturkmen

# verify replication
rs.secondaryOk()

# show databases
show dbs
```

**Note:** This command connects you to the master from whatever machine you connect to.
``` bash
# for example. on the slave-2
mongo --host "slaverep/master-primary,slave-secondary-1,slave-secondary-2" -u admin -p denizturkmen

# list databases
show dbs

# create databases
use testdb

# insert
db.person.insert( { name: "deniz", surname: "turkmen" } )

# list of collectins
show collections

# show record
db.person.find()

# checking slave-1 machine
mongo -u admin -p denizturkmen
rs.secondaryOk()

# list databases
show dbs

# create databases
use testdb

# show record
db.person.find()

```


If the primary machine falls, we define priority to select the remaining nodes as primary. for this;
``` bash
# login on the master machine
mongo -u admin -p denizturkmen

# config
cfg = rs.conf()
cfg.members[0].priority = 0.5
cfg.members[1].priority = 2
cfg.members[2].priority = 2

# re-config
rs.reconfig(cfg)

# check
rs.config()


```