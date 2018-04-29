# Strimzi Topic Controller

## Create topic through ConfigMap

```
oc create -f topic-handling/topic.yaml
```

In order to check that the Topic Controller has dected the new config map and created a related topic in the Kafka cluster, we can run the official `kafka-topics.sh` tool on one of the brokers

```
oc exec -it my-cluster-kafka-0 -- bin/kafka-topics.sh --zookeeper my-cluster-zookeeper:2181 --list
created-as-configmap
```

We can also describing it for getting more information.

```
oc exec -it my-cluster-kafka-0 -- bin/kafka-topics.sh --zookeeper my-cluster-zookeeper:2181 --describe --topic created-as-configmap
Topic:my-topic	PartitionCount:1	ReplicationFactor:1	Configs:
	Topic: created-as-configmap	Partition: 0	Leader: 1	Replicas: 1	Isr: 1
```

Let's increase the partitions number now.
It's possible just updating the related config map and changing the `partitions` data field from 1 to 3, for example using the "edit" command provided by the `oc` tool.

```
oc edit cm my-topic
```

The Topic Controller detects this update and updates the related Kafka topic accordingly.
We can check that describing the topic one more time.

```
oc exec -it my-cluster-kafka-0 -- bin/kafka-topics.sh --zookeeper my-cluster-zookeeper:2181 --describe --topic created-as-configmap
Topic:created-as-configmap	PartitionCount:3	ReplicationFactor:1	Configs:
	Topic: created-as-configmap	Partition: 0	Leader: 1	Replicas: 1	Isr: 1
	Topic: created-as-configmap	Partition: 1	Leader: 2	Replicas: 2	Isr: 2
	Topic: created-as-configmap	Partition: 2	Leader: 3	Replicas: 3	Isr: 3
```

## Create topic through Kafka

```
oc exec -it my-cluster-kafka-0 -- bin/kafka-topics.sh --zookeeper my-cluster-zookeeper:2181 --create --topic created-in-kafka --partitions 3 --replication-factor 1 --config cleanup.policy=compact
```

Show how a related ConfigMap is created by the Topic Controller.

```
oc get configmap created-in-kafka -o yaml
```