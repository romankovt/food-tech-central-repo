# Needed connectors

Postgres debezium connector for mercury:

```json
{
    "name": "postgres-tardis",
    "config": {
        "connector.class": "io.debezium.connector.postgresql.PostgresConnector",
        "publication.autocreate.mode": "filtered",
        "database.user": "debezium",
        "database.dbname": "tardis-user",
        "database.hostname": "tardis-postgres",
        "database.password": "debezium",
        "name": "postgres-tardis",
        "database.server.name": "tardis-user",
        "table.include.list": "public.users",
        "database.port": "5432",
        "plugin.name": "pgoutput"
    },
    "tasks": [
        {
            "connector": "postgres-tardis",
            "task": 0
        }
    ],
    "type": "source"
}

```

Debezium connector for elasticsearch:
```json
{
    "name": "postgres-tardis-jdbc-sink",
    "config": {
        "connector.class": "io.debezium.connector.postgresql.PostgresConnector",
        "database.user": "debezium",
        "database.dbname": "tardis-user",
        "slot.name": "debezium2",
        "tasks.max": "1",
        "transforms": "route",
        "database.server.name": "tardis-user-jdbc-sink",
        "database.port": "5432",
        "plugin.name": "pgoutput",
        "transforms.route.type": "org.apache.kafka.connect.transforms.RegexRouter",
        "transforms.route.regex": "([^.]+)\\.([^.]+)\\.([^.]+)",
        "transforms.route.replacement": "$3",
        "database.hostname": "tardis-postgres",
        "database.password": "debezium",
        "name": "postgres-tardis-jdbc-sink"
    },
    "tasks": [
        {
            "connector": "postgres-tardis-jdbc-sink",
            "task": 0
        }
    ],
    "type": "source"
}

```


Sink to mercury postgres:

```json
{
    "name": "jdbc-sink-tardis-user",
    "config": {
        "connector.class": "io.confluent.connect.jdbc.JdbcSinkConnector",
        "tasks.max": "1",
        "topics": "users",
        "transforms": "unwrap",
        "delete.enabled": "true",
        "transforms.unwrap.drop.tombstones": "false",
        "name": "jdbc-sink-tardis-user",
        "transforms.unwrap.type": "io.debezium.transforms.ExtractNewRecordState",
        "auto.create": "true",
        "connection.url": "jdbc:postgresql://mercury-postgres:5432/mercury-partner?user=mercury-partner",
        "insert.mode": "upsert",
        "pk.mode": "record_key",
        "pk.fields": "id"
    },
    "tasks": [
        {
            "connector": "jdbc-sink-tardis-user",
            "task": 0
        }
    ],
    "type": "sink"
}
```


Sink to elasticsearch:
```json
{
  "name": "elastic-sink",
  "config": {
      "connector.class": "io.confluent.connect.elasticsearch.ElasticsearchSinkConnector",
      "tasks.max": "1",
      "topics": "users",
      "connection.url": "http://elastic:9200",
      "transforms": "unwrap,key",
      "transforms.unwrap.type": "io.debezium.transforms.ExtractNewRecordState",
      "transforms.unwrap.drop.tombstones": "false",
      "transforms.key.type": "org.apache.kafka.connect.transforms.ExtractField$Key",
      "transforms.key.field": "id",
      "key.ignore": "false",
      "type.name": "users",
      "behavior.on.null.values": "delete"
  }
}

```

