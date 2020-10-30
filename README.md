# VNNOR big data pipeline example

Modern data pipeline example with:
  - Kafka
  - Streamsets
  - Spark, Jupyter Notebook
  - Elasticsearch & Kibana
  - Postgres

### Setup

- Extract [sdc.zip](https://drive.google.com/file/d/1gMk1YQCOsPYA84iOy5MwoixxDsJ2kpG4/view?usp=sharing) to `./volumes/sdc` on first start time.

- Start pipe network:
  ```
    docker-compose -f docker-compose-elk.yml up -d
    docker-compose -f docker-compose-sdc.yml up -d
    docker-compose -f docker-compose-spark.yml up -d
  ```

- Stop pipe network:
  ```
    docker-compose -f docker-compose-spark.yml down
    docker-compose -f docker-compose-elk.yml down
    docker-compose -f docker-compose-sdc.yml down
  ```

### Create databases

```
  docker exec -it pgdb /bin/bash
  psql -h localhost -p 5432 -U postgres

  CREATE DATABASE trackings;
  CREATE DATABASE converted_trackings;

  \c trackings
  CREATE TABLE "public"."items" ("id" serial,"event_id" varchar,"device_id" varchar,"created" timestamp,"has_position" bool,"raw" json, PRIMARY KEY ("id"));

  \c converted_trackings
  CREATE TABLE "public"."items" ("id" serial,"event_id" varchar,"device_id" varchar,"created" bigint,"has_position" bool,"temperature" float, PRIMARY KEY ("id"));
```

### Debezium connector

```
  // create debezium connector
  curl -X POST -H "Accept:application/json" -H "Content-Type: application/json" -d @configs/debezium-connector.json http://localhost:8083/connectors

  // watching events
  docker exec -it kafka /bin/bash
  ./bin/kafka-console-consumer.sh --topic dbserver1.public.items --from-beginning --bootstrap-server kafka:9092
```

### Jupyter Notebook

```
  ! pip install psycopg2-binary
  ! pip install elasticsearch

  import psycopg2
  import pprint
  from elasticsearch import Elasticsearch

  pg = psycopg2.connect("postgres://postgres:password@pgdb:5432/trackings")
  es = Elasticsearch([{'host': HOST, 'port': PORT}])

  cursor = pg.cursor()
  cursor.execute('SELECT device_id, avg(temperature) FROM items GROUP BY device_id')
  rows = cursor.fetchall()
  pg.close()

  pprint.pprint(rows)

  for row in rows:
    es.index(index='tracking_avg_temperatures', id=row[0], body={ 'device_id': row[0], 'avg_temperature': row[1] })
```
