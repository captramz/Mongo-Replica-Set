# Mongo Replica Set Instructions

Here are the instructions that worked for me to create a Mongo Replica Set instance when deploying to a Docker Swarm cluster.

## Deploy Docker Swarm Mongo Stack

```yaml
#docker-compose.yaml
version: "3.8"

services:
  admin:
    image: alpine:latest
    stdin_open: true
    tty: true
    volumes:
      - ssl:/ssl

  mongo1:
    image: mongo:5.0.17
    command: mongod --keyFile /ssl/file.key --replSet my-replica-set
    depends_on:
      - mongo2
      - mongo3
    environment:
      MONGO_INITDB_ROOT_USERNAME: $MONGO_INITDB_ROOT_USERNAME
      MONGO_INITDB_ROOT_PASSWORD: $MONGO_INITDB_ROOT_PASSWORD
    ports:
      - 27017:27017
    volumes:
      - ssl:/ssl
      - data1:/data/db
      - config1:/data/configdb

  mongo2:
    image: mongo:5.0.17
    command: mongod --keyFile /ssl/file.key --replSet my-replica-set
    environment:
      MONGO_INITDB_ROOT_USERNAME: $MONGO_INITDB_ROOT_USERNAME
      MONGO_INITDB_ROOT_PASSWORD: $MONGO_INITDB_ROOT_PASSWORD
    ports:
      - 27018:27017
    volumes:
      - ssl:/ssl
      - data2:/data/db
      - config2:/data/configdb

  mongo3:
    image: mongo:5.0.17
    command: mongod --keyFile /ssl/file.key --replSet my-replica-set
    environment:
      MONGO_INITDB_ROOT_USERNAME: $MONGO_INITDB_ROOT_USERNAME
      MONGO_INITDB_ROOT_PASSWORD: $MONGO_INITDB_ROOT_PASSWORD
    ports:
      - 27019:27017
    volumes:
      - ssl:/ssl
      - data3:/data/db
      - config3:/data/configdb

volumes:
  ssl:
  data1:
  data2:
  data3:
  config1:
  config2:
  config3:
```

To deploy the stack, run the following command.

```bash
docker stack deploy -c docker-compose.yml mongo
```

## Admin Set Up

Attach to the admin container's terminal and run the following command to install `openssl`.

```bash
apk add openssl
```

### Generate Key Required for Mongo Replica Set

Generate key. **Note** make sure to place it in the `/ssl` volume.

```bash
openssl rand --base64 765 > /ssl/file.key
```

Change permissions to key file.

```bash
chmod 400 /ssl/file.key
```

Change ownership to 999.

```bash
chown 999 /ssl/file.key
```

## Initiate Replica Set

Get to the mongo CLI by typing `mongosh`, then initiate the replica set.

```javascript
rs.initiate(
  {
    _id: "my-replica-set",
    members: [
      { _id: 1, host: "mongo1:27017" },
      { _id: 2, host: "mongo2:27017" },
      { _id: 3, host: "mongo3:27017" },
    ],
  },
  { force: true }
);
```

## Create Admin User

Switch to admin db.

```mongosh
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

## Create Application Database and User

Create database. This is the database that you will be using for your application.

```mongosh
use <db-name>
```

```javascript
db.createUser({
  user: "<username>",
  pwd: "<password>",
  roles: [{ role: "readWrite", db: "<db-name>" }],
});
```
