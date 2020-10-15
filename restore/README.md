# Do a dump and restore to your MongoDB 4.4.1 replica set

## Prerequisites

In order to finish the following steps, you must first complete the full setup that is shown in the root [README.md](https://github.com/antonioned/mongodb-step-by-step/blob/main/README.md) file of this repo.

## Step-by-step guide

**1. Dump the source database**

In my case, I had to do a dump and restore from a standalone MongoDB server running on 3.4. I did it straight away, with no intermediate upgrades, the restore in the target instance finished completely successfully, no errors whatsoever. In order to confirm this, a client application check and test is **mandatory**, something that at the point of writing these instructions is still pending for me.

Run the following on the source MongoDB instance:

```
# Force file syncronization and lock writes to stop writes to your db:
db.fsyncLock()

#   Dump the db data. 
#   The commands are ran against a standalone mongodb EC2 with no auth enabled, so no need of user auth. 
DATA_DIR is the folder where your mongo stores its data.

cd DATA_DIR
mongodump —db DATABASE_NAME
cd dump
tar -czvf DATABASE_NAME.tar.gz DATABASE_NAME

#   Move the data dump file to the target EC2 primary. I use S3 for this.
aws s3 cp DATABASE_NAME.tar.gz s3://S3_BUCKET
```

**2. Restore the data**

After this, ssh into the primary of the newly created replica set and restore the data.

```
#   Fetch it in the target EC2. Make sure there is enough disk space in the target instance, depending of 
how big is the data you want to restore.

mkdir -p /data/db/dump
cd /data/db/dump
aws s3 cp s3://S3_BUCKET/DATABASE_NAME.tar.gz .
tar -xzvf DATABASE_NAME.tar.gz
rm -rf DATABASE_NAME.tar.gz
mongorestore /data/db/dump
```
Depending on the size of the data you are restoring, the process will vary in length. After the restoring is finished, wait for a few minutes in order for the secondaries to sync to the primary and you are all set!

Steps to be done after are switching your application to connect to the newly created replia set to verify the data is restored properly. If good, you can safely remove the source mongodb server.
In case of a needed rollback, don't forge to unlock writes to the standalone mongodb with `db.fsyncUnlock()`.