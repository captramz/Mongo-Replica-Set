# Mongo Replica Set Instructions

Here are the instructions that worked for me to create a Mongo Replica Set instance when deploying to a Docker Swarm cluster.

## Generate Key Required for Mongo Replica Set

Generate key.

```bash
openssl rand --base64 765 > <path-to-keyfile>
```

Change permissions to key file.

```bash
chmod 400 <path-to-keyfile>
```

Change ownership to 999.

```bash
chown 999 <path-to-keyfile>
```

---

## Initiate Replica Set

Get to the mongo CLI by typing `mongosh`, then initiate the replica set.

```javascript
rs.initiate(
  {
    _id: "<mongo-set-name>",
    members: [
      { _id: 1, host: "mongo1:27017" },
      { _id: 2, host: "mongo2:27017" },
      { _id: 3, host: "mongo3:27017" },
    ],
  },
  { force: true }
);
```

---

## Create Admin User

Switch to admin db.

```bash
use admin
```

Create admin user.

```javascript
db.createUser({
  user: "<username>",
  pwd: "<password>",
  roles: [
    { role: "userAdminAnyDatabase", db: "admin" },
    { role: "dbAdminAnyDatabase", db: "admin" },
  ],
});
```

Create OPLOG user.

```javascript
db.createUser({
  user: "oplogger",
  pwd: "<password>",
  roles: [{ role: "read", db: "local" }],
});
```

---

## Create Application Databse and User

Create database.

```bash
use <db-name>
```

```javascript
db.createUser({
  user: "<username>",
  pwd: "<password>",
  roles: [{ role: "readWrite", db: "<db-name>" }],
});
```
