# kafka-connect-poc

# Cria o connector

curl --location 'http://localhost:8083/connectors' \
--header 'Content-Type: application/json' \
--data '{
  "name": "mirror-source-connector",
  "config": {
    "connector.class": "org.apache.kafka.connect.mirror.MirrorSourceConnector",
    "clusters": "source,destination",
    "source.cluster.alias": "source",
    "target.cluster.alias": "destination",
    "source.cluster.bootstrap.servers": "kafka-a:9092",
    "target.cluster.bootstrap.servers": "kafka-b:9093",
    "source->destination.enabled": "true",
    "source->destination.topics": ".*",
    "replication.factor": "1",
    "checkpoints.topic.replication.factor": "1",
    "heartbeats.topic.replication.factor": "1",
    "offset-syncs.topic.replication.factor": "1",
    "offset.storage.replication.factor": "1",
    "status.storage.replication.factor": "1",
    "config.storage.replication.factor": "1"
  }
}
'

# Entrar no container
docker exec -it {nome-ou-id-do-container} bash

# Cria topico
kafka-topics --bootstrap-server kafka:9092 --create --topic {nome do topico} --replication-factor 1 --partitions 1

# Lista os topicos
kafka-topics --bootstrap-server kafka:9092 --list

# Consumer topico criado no destino
docker exec -it kafka-connect-poc-kafka-a-1 kafka-console-consumer --bootstrap-server kafka-b:9093 --topic source.topico-1 --from-beginning

# Consumer topico origem
docker exec -it kafka-connect-poc-kafka-a-1 kafka-console-consumer --bootstrap-server kafka-a:9092 --topic topico-1 --from-beginning

# Producer topico origem
docker exec -it kafka-connect-poc-kafka-a-1 kafka-console-producer --bootstrap-server kafka-a:9092 --topic topico-1
