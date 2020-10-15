# mongodb-step-by-step
Full, step-by-step MongoDB 4.4.1 installation guide

## Introduction

This guide shows how to set up a MongoDB 4.4.1 replica set from scratch. It guides through the process of launching a completely fresh Ubuntu 18.04 in AWS, finishing with a full replica set with user and replica set authentication configured. Optionally, you can also dump and restore data from your old mongo in the process.

## Prerequisites

1. AWS EC2 instance running Ubuntu 18.04
2. (Optional) Bake your own AMI via any configuration tool (e.g. Packer) in order to install any other packages you might need like awscli, docker, jq etc.

## Step-by-step guide

After you launch your EC2 instance (type, volume size, security settings all depend on your needs), first step would be to install mongo 4.4.1. For this, use the `install-mongo.sh` script provided in this repo. The scripts will install mongo and create a `/data/db` directory which I will actually use as the data store dir for the mongo. This dir will be mounted on a separate EBS volume which we will cover in this guide as well.

1. When installation is complete, next steps are configuring the EBS volume:

```
lsblk                                   #   outputs the volumes attached to the EC2, find the one that is not the root volume (in my case /dev/nvme1n1)
sudo file -s /dev/nvme1n1               #   check if there is a file system for the volume, should display /dev/nvme1n1: data
sudo mkfs -t xfs /dev/nvme1n1           #   create a file system
sudo mount /dev/nvme1n1 /data/db        #   mount the directory to the volume
sudo chown -R mongodb:mongodb /data     #   add permissions for the mongodb user
```

2. Edit the **/etc/mongod.conf**

```
sudo vim /etc/mongod.conf

Change the dbPath to /data/db and also change the bindIp parameter to 0.0.0.0 (open on all IPs but keep in mind that the EC2 needs to be in a private subnet and that we will add authentication security to it as well)
```