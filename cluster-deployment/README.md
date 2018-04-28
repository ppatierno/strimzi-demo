# Strimzi Cluster Controller

## Start OpenShift cluster

```
oc cluster up
oc login -u system:admin
```

## Deploy Cluster Controller

Install OpenShift templates.

```
oc create -f cluster-deployment/templates
```

Deply the Cluster Controller.

```
oc create -f cluster-deployment/install
```

## Deploy Cluster

Deploy an ephemeral cluster.

```
oc new-app strimzi-ephemeral
```

Show the running pods.

```
watch oc get pods
```

Show the cluster config map.

```
oc get cm my-cluster -o yaml
```

## Update Cluster

Increase the number of nodes from 3 to 5 updating the `kafka-nodes` data field in the `my-cluster` config map.

```
oc edit cm my-cluster
```

Show the running pods.

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

Show the brokers "rolling update" (or through the OpenShift console).

```
watch oc get pods
```

Check the broker log with the new value.

```
oc logs my-cluster-kafka-0 | grep default.replication.factor
```
