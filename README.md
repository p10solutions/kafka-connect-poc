# kafka-connect-poc

services:
  # Zookeeper cluster A
  zookeeper-a:
    image: confluentinc/cp-zookeeper:latest
    environment:
      ZOOKEEPER_CLIENT_PORT: 2181

  # Kafka cluster A
  kafka-a:
    image: confluentinc/cp-kafka:latest
    depends_on:
      - zookeeper-a
    ports:
      - 9092:9092
    environment:
      KAFKA_BROKER_ID: 1
      KAFKA_ZOOKEEPER_CONNECT: zookeeper-a:2181
      KAFKA_ADVERTISED_LISTENERS: PLAINTEXT://kafka-a:9092
      KAFKA_OFFSETS_TOPIC_REPLICATION_FACTOR: 1
      KAFKA_TRANSACTION_STATE_LOG_REPLICATION_FACTOR: 1
      KAFKA_TRANSACTION_STATE_LOG_MIN_ISR: 1

  # Zookeeper cluster B
  zookeeper-b:
    image: confluentinc/cp-zookeeper:latest
    environment:
      ZOOKEEPER_CLIENT_PORT: 2182

  # Kafka cluster B
  kafka-b:
    image: confluentinc/cp-kafka:latest
    depends_on:
      - zookeeper-b
    ports:
      - 9093:9093
    environment:
      KAFKA_BROKER_ID: 2
      KAFKA_ZOOKEEPER_CONNECT: zookeeper-b:2182
      KAFKA_ADVERTISED_LISTENERS: PLAINTEXT://kafka-b:9093
      KAFKA_OFFSETS_TOPIC_REPLICATION_FACTOR: 1
      KAFKA_TRANSACTION_STATE_LOG_REPLICATION_FACTOR: 1
      KAFKA_TRANSACTION_STATE_LOG_MIN_ISR: 1

  # Kafka Connect com MirrorSourceConnector
  kafka-connect:
    image: confluentinc/cp-kafka-connect:latest
    # image: confluentinc/cp-kafka-connect:6.2.1
    # image: confluentinc/cp-kafka-connect:7.5.3
    depends_on:
      - kafka-a
      - kafka-b
    ports:
      - 8083:8083
    environment:
      CONNECT_REST_ADVERTISED_HOST_NAME: kafka-connect
      CONNECT_BOOTSTRAP_SERVERS: kafka-b:9093
      CONNECT_REST_PORT: 8083
      CONNECT_GROUP_ID: kafka-connect-group
      CONNECT_CONFIG_STORAGE_TOPIC: connect-configs
      CONNECT_OFFSET_STORAGE_TOPIC: connect-offsets
      CONNECT_STATUS_STORAGE_TOPIC: connect-status
      CONNECT_KEY_CONVERTER: org.apache.kafka.connect.converters.ByteArrayConverter
      CONNECT_VALUE_CONVERTER: org.apache.kafka.connect.converters.ByteArrayConverter
      CONNECT_CONFIG_STORAGE_REPLICATION_FACTOR: 1
      CONNECT_OFFSET_STORAGE_REPLICATION_FACTOR: 1
      CONNECT_STATUS_STORAGE_REPLICATION_FACTOR: 1
      CONNECT_PLUGIN_PATH: '/usr/share/java,/usr/share/confluent-hub-components'
      # Habilita criação automática dos tópicos no Kafka Connect
      CONNECT_TOPIC_CREATION_ENABLE: "true"



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
