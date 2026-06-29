# Debezium Tutorial

CDC streaming with Debezium, Kafka, and MySQL, MongoDB, or PostgreSQL.

## MySQL

### Services

| Service | Image | Port |
|---------|-------|------|
| Kafka | `quay.io/debezium/kafka:3.5` | 9092 |
| MySQL | `quay.io/debezium/example-mysql:3.5` | 3306 |
| Connect | `quay.io/debezium/connect:3.5` | 8083 |
| Watcher | `quay.io/debezium/kafka:3.5` | — |

### Quick Start

```bash
docker compose -f docker-compose-mysql.yml up -d
```

Register the MySQL connector:

```bash
curl -i -X POST http://localhost:8083/connectors \
  -H "Content-Type: application/json" \
  -d '{
    "name": "inventory-connector",
    "config": {
      "connector.class": "io.debezium.connector.mysql.MySqlConnector",
      "tasks.max": "1",
      "database.hostname": "mysql",
      "database.port": "3306",
      "database.user": "mysqluser",
      "database.password": "mysqlpw",
      "database.server.id": "184054",
      "database.server.name": "dbserver1",
      "database.include.list": "inventory",
      "schema.history.internal.kafka.bootstrap.servers": "kafka:9092",
      "schema.history.internal.kafka.topic": "schema-changes.inventory"
    }
  }'
```

Watch the CDC events:

```bash
docker compose -f docker-compose-mysql.yml up watcher
```

---

## MongoDB

### Services

| Service | Image | Port |
|---------|-------|------|
| Kafka | `quay.io/debezium/kafka:3.5` | 9092 |
| MongoDB | `mongo:7` (replica set `rs0`) | 27017 |
| Connect | `quay.io/debezium/connect:3.5` | 8083 |
| Watcher | `quay.io/debezium/kafka:3.5` | — |

### Quick Start

```bash
docker compose -f docker-compose-mongodb.yml up -d
```

Wait for MongoDB replica set to initialize (healthcheck auto-runs `rs.initiate()`), then register the connector:

```bash
curl -i -X POST http://localhost:8083/connectors \
  -H "Content-Type: application/json" \
  -d '{
    "name": "inventory-connector",
    "config": {
      "connector.class": "io.debezium.connector.mongodb.MongoDbConnector",
      "mongodb.connection.string": "mongodb://mongodb:27017/?replicaSet=rs0",
      "topic.prefix": "fulfillment",
      "collection.include.list": "inventory[.]*"
    }
  }'
```

Insert test data:

```bash
docker exec -it mongodb mongosh inventory \
  --eval 'db.customers.insertOne({"_id": "1001", "first_name": "Alice", "last_name": "Johnson", "email": "alice@example.com"})'
```

Watch the CDC events:

```bash
docker compose -f docker-compose-mongodb.yml up watcher
```

---

## PostgreSQL

### Services

| Service | Image | Port |
|---------|-------|------|
| Kafka | `quay.io/debezium/kafka:3.5` | 9092 |
| PostgreSQL | `quay.io/debezium/example-postgres:3.5` | 5432 |
| Connect | `quay.io/debezium/connect:3.5` | 8083 |
| Watcher | `quay.io/debezium/kafka:3.5` | — |

### Quick Start

```bash
docker compose -f docker-compose-postgresql.yml up -d
```

Register the PostgreSQL connector:

```bash
curl -i -X POST http://localhost:8083/connectors \
  -H "Content-Type: application/json" \
  -d '{
    "name": "inventory-connector",
    "config": {
      "connector.class": "io.debezium.connector.postgresql.PostgresConnector",
      "database.hostname": "postgres",
      "database.port": "5432",
      "database.user": "postgres",
      "database.password": "postgres",
      "database.dbname": "inventory",
      "topic.prefix": "fulfillment",
      "table.include.list": "public.inventory",
      "plugin.name": "pgoutput"
    }
  }'
```

Insert test data:

```bash
docker exec -it postgres psql -U postgres -d inventory \
  -c "INSERT INTO customers (id, first_name, last_name, email) VALUES (1001, 'Alice', 'Johnson', 'alice@example.com');"
```

Watch the CDC events:

```bash
docker compose -f docker-compose-postgresql.yml up watcher
```
