# Strimzi Topic Controller

## Create topic through ConfigMap

```
oc create -f topic-handling/topic.yaml
```

In order to check that the Topic Controller has dected the new config map and created a related topic in the Kafka cluster, we can run the official `kafka-topics.sh` tool on one of the brokers

```
oc exec -it my-cluster-kafka-0 -- bin/kafka-topics.sh --zookeeper my-cluster-zookeeper:2181 --describe
Topic:created-as-configmap      PartitionCount:1        ReplicationFactor:1     Configs:retention.ms=4600000,cleanup.policy=compact
        Topic: created-as-configmap     Partition: 0    Leader: 4       Replicas: 4     Isr: 4
```

Let's increase the partitions number now.
It's possible just updating the related config map and changing the `partitions` data field from 1 to 3, for example using the "edit" command provided by the `oc` tool.

```
oc edit cm created-as-configmap
```

The Topic Controller detects this update and updates the related Kafka topic accordingly.
We can check that describing the topic one more time.

```
oc exec -it my-cluster-kafka-0 -- bin/kafka-topics.sh --zookeeper my-cluster-zookeeper:2181 --describe
Topic:created-as-configmap      PartitionCount:3        ReplicationFactor:1     Configs:retention.ms=4600000,cleanup.policy=compact
        Topic: created-as-configmap     Partition: 0    Leader: 4       Replicas: 4     Isr: 4
        Topic: created-as-configmap     Partition: 1    Leader: 0       Replicas: 0     Isr: 0
        Topic: created-as-configmap     Partition: 2    Leader: 1       Replicas: 1     Isr: 1
```

## Create topic through Kafka

```
oc exec -it my-cluster-kafka-0 -- bin/kafka-topics.sh --zookeeper my-cluster-zookeeper:2181 --create --topic created-in-kafka --partitions 3 --replication-factor 1 --config cleanup.policy=compact
```

Show how a related ConfigMap is created by the Topic Controller.

```
oc get configmap created-in-kafka -o yaml
```

## Application

Deploy two simple Kafka consumer and producer applications using a topic named `my-topic` for exchanging messages.

```
oc create -f topic-handling/example-app.yaml
```