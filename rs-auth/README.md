# Add replica set security to your MongoDB 4.4.1 replica set

## Prerequisites

In order to finish the following steps, you must first complete the full setup that is shown in the root [README.md](https://github.com/antonioned/mongodb-step-by-step/blob/main/README.md) file of this repo.

## Step-by-step guide

**1. Create a keyfile in the primary node**

The usage of a keyfile in the replica set is totally optional, it adds an additional level of security for the replica set nodes. Compared to the user auth, this is strictly for the set configuration only, the nodes use the keyfile to communicate and authenticate between themselves.

```
openssl rand -base64 756 > /data/mongo.key
chmod 400 /data/mongo.key
chown mongodb:mongodb /data/mongo.key

#   Copy this keyfile to all the nodes. Again, I use s3 for this.
aws s3 cp /data/mongo.key s3://S3_BUCKET

#   In the secondaries:
sudo su
cd /data
aws s3 cp s3://S3_BUCKET/mongo.key .
chmod 400 mongo.key
chown mongodb:mongodb mongo.key
```

**2. Shut down all nodes**

Shut down all the nodes one by one, with the **primary being last**. This is mandatory, because it will prevent potential issues within the replica set.

```
use admin
db.shutdownServer()
```

**3. Edit the /etc/mongod.conf**

After the mongos are offline, shut down, add the following in the file:

```
security:
  keyFile: /data/mongo.key
```
After this, restart the mongod process in all of the nodes, starting with the primary `sudo systemctl restart mongod`.
When this is completed, the replica set should come back online with possible changes in the nodes priorites, it might happen that one of the secondaries assumed the role of a primary after the process. If you want to force the "original" primary to be primary again, change the nodes priority in the replica set:

```
#   Run the following in the current primary mongo shell:

cfg = rs.conf()
cfg.members[0].priority = 0.5
cfg.members[1].priority = 0.5
cfg.members[2].priority = 1         #   if you need the 3rd member to be the new primary
rs.reconfig(cfg)
```