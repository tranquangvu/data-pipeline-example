# Big data pipeline example

Modern data pipeline build with streamsets, kafka, spark, ...

### Setup pipeline

- Unzip sdc.zip to `./sdc` for first time to load neccessary streamsets lib (https://drive.google.com/file/d/1gMk1YQCOsPYA84iOy5MwoixxDsJ2kpG4/view?usp=sharing)

- Start pipeline: `docker-compose up -d`

- Stop pipeline: `docker-compose down`

### Debezium

```
  curl -X POST -H "Accept:application/json" -H "Content-Type: application/json" -d @sourcedb-connector.json http://localhost:8083/connectors

  docker exec -it kafka /bin/bash
  ./bin/kafka-console-consumer.sh --topic dbserver1.public.items --from-beginning --bootstrap-server kafka:9092
```
