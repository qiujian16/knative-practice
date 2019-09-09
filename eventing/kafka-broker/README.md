## Install kafka
1. install kafka operator
```
curl -L https://github.com/strimzi/strimzi-kafka-operator/releases/download/0.13.0/strimzi-cluster-operator-0.13.0.yaml   | sed 's/namespace: .*/namespace: kafka/' > kafka.yaml
kubectl apply -f kafka.yaml
```
2. install kafka cr (no persistence needed)
```
curl https://raw.githubusercontent.com/strimzi/strimzi-kafka-operator/0.13.0/examples/kafka/kafka-ephemeral.yaml > kafka-cr.yaml
kubectl apply -f kafka-cr.yaml
```

You will then see kafka bootstrap service and other services in kafka namesapce
```
NAME                          TYPE        CLUSTER-IP       EXTERNAL-IP   PORT(S)                      AGE
my-cluster-kafka-bootstrap    ClusterIP   10.100.133.152   <none>        9091/TCP,9092/TCP,9093/TCP   4h16m
my-cluster-kafka-brokers      ClusterIP   None             <none>        9091/TCP,9092/TCP,9093/TCP   4h16m
my-cluster-zookeeper-client   ClusterIP   10.111.86.29     <none>        2181/TCP                     4h17m
my-cluster-zookeeper-nodes    ClusterIP   None             <none>        2181/TCP,2888/TCP,3888/TCP   4h17m
```

## Install kafka broker
1. install ko, and set docker registry and ensure that docker credential is configure in `~/.docker/config.json`
```
go get github.com/google/ko/cmd/ko
export KO_DOCKER_REPO=docker.io/xxx
```
2. clone eventing-contrib repository
```
git clone https://github.com/knative/eventing-contrib.git
mv event-contrib $GOPAH/src/knative.dev/
```
3. change kafka channel configmap in `$GOPATH/src/knative.dev/event-contrib/kafka/channel/config/400-kafka-config.yaml`
```
apiVersion: v1
kind: ConfigMap
metadata:
  name: config-kafka
  namespace: knative-eventing
data:
  # Broker URL. Replace this with the URLs for your kafka cluster,
  # which is in the format of my-cluster-kafka-bootstrap.my-kafka-namespace:9092.
  bootstrapServers: my-cluster-kafka-bootstrap.kafka:9092
```
4. install kafka channel controller
```
cd $GOPATH/src/knative.dev/event-contrib/kafka/channel
ko apply -f config/
```
5. install kafka broker
```
apiVersion: eventing.knative.dev/v1alpha1
kind: Broker
metadata:
  name: kafka
spec:
  channelTemplateSpec:
    apiVersion: messaging.knative.dev/v1alpha1
    kind: KafkaChannel
    spec:
      numPartitions: 3
      replicationFactor: 1
```

