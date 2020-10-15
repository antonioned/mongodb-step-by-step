# Add user authentication to your MongoDB 4.4.1 replica set

## Prerequisites

In order to finish the following steps, you must first complete the full setup that is shown in the root [README.md](https://github.com/antonioned/mongodb-step-by-step/blob/main/README.md) file of this repo.

## Step-by-step guide

**1. Add users to your mongo replica set primary**

The first step for adding security to your mongodb set is creating users. Best approach would be to create one super user which will have root privileges to the whole cluster and all dbs, and one user that will be used by your client application with access only to the specific databases the application needs.

```
#   log into mongo shell
mongo

#   switch to admin db
use admin

#   create the root user
db.createUser({user: "root", pwd: "root123", roles: [{ role: "root", db: "admin" } ]})

#   create your client application user username with access to SOME_DB
db.createUser({user: "username", pwd: "test123", roles: [{ role: "readWrite", db: "SOME_DB" },{ role: "dbAdmin", db: "SOME_DB" }]})

#   if you need the app user to have access to additional databases, run
db.grantRolesToUser("username", [{ role: "readWrite", db: "SOME_OTHER_DB" }, { role: "dbAdmin", db: "SOME_OTHER_DB" }])

#   exit the mongo shell
exit
```
Wait a couple of minutes for the replica set to sync. Then, restart the mongod process `sudo systemctl restart mongod` and check its status, should be `active (running)`. After all this is done, you will have to authenticate with one of the users you created in order to be able to view databases, collections and run commands in the mongo shell. Running just `mongo` to login into the shell will succeed, but the commands will error out or show no results.

**2 Use user authentication**
In order to start the mongo shell authenticated, you need to run:

```
mongo -u root -p

оr

mongo -u username -p
```
This will prompt for a password. It is recommended that you enter the password like this instead of putting it in the terminal command like: `mongo -u root -p root123` since if latter is used, the password will be visible in the terminal history of the EC2 instance (not good at all).