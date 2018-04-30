# Strimzi Cluster Controller

## Start OpenShift cluster

Start a local OpenShift cluster using the `oc` tool.

```
oc cluster up
oc login -u system:admin
```

> You could use the `minishift` tool as well.

## Deploy Cluster Controller

Install OpenShift templates by Strimzi which provide Kafka Cluster (ephemeral and persistent), Kafka Connect and Kafka Connect S2I installations.

```
oc create -f cluster-deployment/templates
```

Deploy the Cluster Controller which will be in charge to handle cluster config map in order to create/update/delete Kafka clusters.

```
oc create -f cluster-deployment/install
```

## Deploy Cluster

Deploy an ephemeral cluster. In this case the Kafka logs are stored in a volume at Pod level so all the messages are lost when the Pod shutdown.

```
oc new-app strimzi-ephemeral
```

Show the running pods which are started by the underlying StatefulSets created by the Cluster Controller.

```
watch oc get pods
```

Show the cluster config map which is created by the above template.

```
oc get cm my-cluster -o yaml
```

## Update Cluster

Increase the number of Kafka brokers from 3 to 5 updating the `kafka-nodes` data field in the `my-cluster` config map.

```
oc edit cm my-cluster
```

Show the running pods; two more Pods are started.

```
watch oc get pods
```

Update the broker config parameter `KAFKA_DEFAULT_REPLICATION_FACTOR` from 1 to 2.
Before that let's check the broker log with the current value.

```
oc logs my-cluster-kafka-0 | grep default.replication.factor
```

Edit the `my-cluster` config map.

```
oc edit cm my-cluster
```

This kind of update is much more complex because changing the Kafka broker configuration we want all the running brokers being update so it means that each broker needs to be restarted in order to get the new configuration.
In this case, detecting the config map update, the Cluster Controller starts a "rolling update" and each broker Pod is killed one by one and then restarted with the new configuration.

```
watch oc get pods
```

Check the broker log with the new value.

```
oc logs my-cluster-kafka-0 | grep default.replication.factor
```
