# Do a dump and restore to your MongoDB 4.4.1 replica set

## Prerequisites

In order to finish the following steps, you must first complete the installation steps shown in the root [README.md](https://github.com/antonioned/mongodb-step-by-step/blob/main/README.md) file of this repo.

## Step-by-step guide

After the previous steps have been configured on all of the 3 EC2 instances that will be members of the mongo replica set, next steps will be creating the replica set itself.

**1. Edit the /etc/hosts file** :hammer:

In order to ensure the 3 nodes can communicate between each other, we need to edit the `/etc/hosts` file and add the private IPs and hostnames to them:

```
sudo vim /etc/hosts

Add the following to the file:

10.8.4.25 mongo-primary
10.8.4.110 mongo-secondary-1
10.8.4.4 mongo-secondary-2
```
This has to be done in all 3 nodes, same content added in the hosts file. After edit, **reboot** all of the 3 nodes, `sudo reboot`. :arrows_counterclockwise:

**2. Edit the /etc/mongod.conf** :hammer:

```
sudo vim /etc/mongod.conf

Add the replSetName parameter with the name of your replica set, replicaSet used in this example:

replication:
  replSetName: replicaSet

sudo systemctl restart mongod       #   after edit, restart the mongod process
```

**3. Initialize the replica set** :point_up:

After all nodes are back up, ssh into the primary and run the mongo shell to initiate the replica set:

```
mongo
rs.initiate()
```

**4. Add the secondary nodes (the remaining 2 EC2 instances)** :heavy_plus_sign: :heavy_plus_sign:

```
rs.add('mongo-secondary-1:27017')
rs.add('mongo-secondary-2:27017')
```

With this step, the replica set configuration is completed. The steps for adding security are optional but **strongly** recommended. :bangbang: