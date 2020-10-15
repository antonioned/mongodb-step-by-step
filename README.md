# mongodb-step-by-step
Full, step-by-step MongoDB 4.4.1 installation guide. :leaves: :leaves:

## Introduction

This guide shows how to set up MongoDB 4.4.1 from scratch. It is divided into 4 different guides, all adding to the initial installation guide and setup. 

All of the steps are configurable, the installation part is the only mandatory one :smile:

You can always choose to do a mongodb standalone with or w/o user authentication. A replica set with or w/o keyfile authentication, dump and restore or just create fresh and clean mongo server. :herb:

## Prerequisites

1. AWS EC2 instance running Ubuntu 18.04
2. (Optional) Bake your own AMI via any configuration tool (e.g. Packer) in order to install any other packages you might need like awscli, docker, jq etc.

## Step-by-step guide

After you launch your EC2 instance (type, volume size, security settings all depend on your needs), first step would be to install mongo 4.4.1. For this, use the `install-mongo.sh` script provided in this repo. The scripts will install mongo and create a `/data/db` directory which I will actually use as the data store dir for the mongo. This dir will be mounted on a separate EBS volume which we will cover in this guide as well.

**1. When installation is complete, next steps are configuring the EBS volume:** :black_square_button:

```
lsblk                                   #   outputs the volumes attached to the EC2, find the one that is not the root volume (in my case /dev/nvme1n1)
sudo file -s /dev/nvme1n1               #   check if there is a file system for the volume, should display /dev/nvme1n1: data
sudo mkfs -t xfs /dev/nvme1n1           #   create a file system
sudo mount /dev/nvme1n1 /data/db        #   mount the directory to the volume
sudo chown -R mongodb:mongodb /data     #   add permissions for the mongodb user
```

**1.1 Configure the volume to be mounted on reboot (Important)** :hammer:

This step is very important since you need the volume to mount on reboot of the instance. If not, the data will be there, but the mongo will not work properly.

```
sudo lsblk -o +UUID         #   check the UUID of the /dev/nvme1n1
sudo vim /etc/fstab

Add the following:

UUID=THE_UUID_FROM_LSBLK /data/db xfs defaults,nofail  0  2
```

**2. Edit the /etc/mongod.conf** :hammer:

```
sudo vim /etc/mongod.conf

Change the dbPath to /data/db 
Change the bindIp parameter to 0.0.0.0 (open on all IPs but keep in mind that the EC2 needs to be
in a private subnet and that we will add authentication security to it  as well)
```

**3. (Optional) Add logrotation to the mongodb log file** :arrows_clockwise:

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

**5. Edit the /etc/mongod.conf** :hammer:

```
sudo vim /etc/mongod.conf

Add the private IP of your EC2 instance in the bindIp parameter:

bindIp: 0.0.0.0 10.8.4.25

sudo systemctl restart mongod
```

With this, your mongodb server is installed and ready to be used. Whether you want to continue the set up with all the additional configurations available:

- replica set
- user authentication
- keyfile authentication
- dump and restore

it is tottaly up to you.

For the steps about configuring a replica set, please continue with the [/replica-set/README.md](https://github.com/antonioned/mongodb-step-by-step/blob/main/replica-set/README.md) file.

For the steps about adding user authentication to your replica set, please continue with the [/user-auth/README.md](https://github.com/antonioned/mongodb-step-by-step/blob/main/user-auth/README.md) file.

For the steps about adding replica set keyfile authentication, please continue with the [/rs-auth/README.md](https://github.com/antonioned/mongodb-step-by-step/blob/main/rs-auth/README.md) file.

For the steps about dump and restore of existing mongo data, please continue with [/restore/README.md](https://github.com/antonioned/mongodb-step-by-step/blob/main/restore/README.md) file.
