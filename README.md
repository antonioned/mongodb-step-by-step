# mongodb-step-by-step
Full, step-by-step MongoDB 4.4.1 installation guide

## Introduction

This guide shows how to set up a MongoDB 4.4.1 replica set from scratch. It guides through the process of launching a completely fresh Ubuntu 18.04 in AWS, finishing with a full replica set with user and replica set authentication configured. Optionally, you can also dump and restore data from your old mongo in the process.

## Prerequisites

1. AWS EC2 instance running Ubuntu 18.04
2. (Optional) Bake your own AMI via any configuration tool (e.g. Packer) in order to install any other packages you might need like awscli, docker, jq etc.

## Step-by-step guide

After you launch your EC2 instance (type, volume size, security settings all depend on your needs), first step would be to install mongo 4.4.1. For this, use the `install-mongo.sh` script provided in this repo. The scripts will install mongo and create a `/data/db` directory which I will actually use as the data store dir for the mongo. This dir will be mounted on a separate EBS volume which we will cover in this guide as well.

**1. When installation is complete, next steps are configuring the EBS volume:**

```
lsblk                                   #   outputs the volumes attached to the EC2, find the one that is not the root volume (in my case /dev/nvme1n1)
sudo file -s /dev/nvme1n1               #   check if there is a file system for the volume, should display /dev/nvme1n1: data
sudo mkfs -t xfs /dev/nvme1n1           #   create a file system
sudo mount /dev/nvme1n1 /data/db        #   mount the directory to the volume
sudo chown -R mongodb:mongodb /data     #   add permissions for the mongodb user
```

**1.1 Configure the volume to be mounted on reboot (Important)**

This step is very important since you need the volume to mount on reboot of the instance. If not, the data will be there, but the mongo will not work properly.

```
sudo lsblk -o +UUID         #   check the UUID of the /dev/nvme1n1
sudo vim /etc/fstab

Add the following:

UUID=THE_UUID_FROM_LSBLK /data/db xfs defaults,nofail  0  2
```

**2. Edit the /etc/mongod.conf**

```
sudo vim /etc/mongod.conf

Change the dbPath to /data/db 
Change the bindIp parameter to 0.0.0.0 (open on all IPs but keep in mind that the EC2 needs to be
in a private subnet and that we will add authentication security to it  as well)
```

**3. (Optional) Add logrotation to the mongodb log file**

Check if logrotate is installed, if not install it.

```
cd /etc/logrotate.d/
vim mongodb             # create the mongodb logrotate file
```
Add this to the mongodb file:
```
var/log/mongodb/*.log {
       weekly
       rotate 10
       copytruncate
       delaycompress
       compress
       notifempty
       missingok
}
```
Depending on your needs you can configure this file accordingly.

**4. (Optional) At this point you can make an AMI of the instance, ready to be used for future needs either for launching a mongo standalone or replica set.**

**5. Edit the /etc/mongod.conf**

```
sudo vim /etc/mongod.conf

Add the private IP of your EC2 instance in the bindIp parameter:

bindIp: 0.0.0.0 10.8.4.25

sudo systemctl restart mongod
```

**6. Configure the replica set**

After the previous steps have been configured on all of the 3 EC2 instances that will be members of the mongo replica set, next steps will be creating the replica set itself.

**6.1 Edit the /etc/hosts file**

In order to ensure the 3 nodes can communicate between each other, we need to edit the `/etc/hosts` file and add the private IPs and hostnames to them:

```
sudo vim /etc/hosts

Add the following to the file:

10.8.4.25 mongo-primary
10.8.4.110 mongo-secondary-1
10.8.4.4 mongo-secondary-2
```
This has to be done in all 3 nodes, same content added in the hosts file. After edit, **reboot** all of the 3 nodes, `sudo reboot`.

**6.2 Edit the /etc/mongod.conf**

```
sudo vim /etc/mongod.conf

Add the replSetName parameter with the name of your replica set, replicaSet used in this example:

replication:
  replSetName: replicaSet

sudo systemctl restart mongod       #   after edit, restart the mongod process
```

**6.3 Initialize the replica set**

After all nodes are back up, ssh into the primary and run the mongo shell to initiate the replica set:

```
mongo
rs.initiate()
```

**6.4 Add the secondary nodes (the remaining 2 EC2 instances)**

```
rs.add('mongo-secondary-1:27017')
rs.add('mongo-secondary-2:27017')
```

With this step, the replica set configuration is completed. The steps for adding security are optional but **strongly** recommended. The steps for doing a dump and restore in the primary are totally left for choice, if the scenario you have is migrating data then you can use them, if you just need a brand new mongodb server, then no dump and restore for you.

For the steps about adding user authentication to your replica set, please continue with the [/user-auth/README.md](https://github.com/antonioned/mongodb-step-by-step/blob/main/user-auth/README.md) file.

For the steps about adding replica set keyfile authentication, please continue with the [/rs-auth/README.md](https://github.com/antonioned/mongodb-step-by-step/blob/main/rs-auth/README.md) file.

For the steps about dump and restore of existing mongo data, please continue with [/restore/README.md](https://github.com/antonioned/mongodb-step-by-step/blob/main/restore/README.md) file.
