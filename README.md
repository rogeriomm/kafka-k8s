
   * https://strimzi.io/quickstarts/


# Applying Strimzi installation file

   * https://strimzi.io/install/latest?namespace=kafka

# Provision the Apache Kafka cluster
```shell
cd k8s/yaml
if ! kubectl get ns kafka; then kubectl create namespace kafka; fi
kubectl apply -f . -n kafka
```

```shell
kubectl wait kafka/my-cluster --for=condition=Ready --timeout=300s -n kafka 
```

# Send and receive messages
```shell
kubectl -n kafka run kafka-consumer -ti --image=quay.io/strimzi/kafka:0.31.0-kafka-3.2.1 --rm=true --restart=Never -- bin/kafka-console-consumer.sh --bootstrap-server my-cluster-kafka-bootstrap:9092 --topic my-topic --from-beginning
```

```shell
kubectl -n kafka run kafka-producer -ti --image=quay.io/strimzi/kafka:0.31.0-kafka-3.2.1 --rm=true --restart=Never -- bin/kafka-console-producer.sh --bootstrap-server my-cluster-kafka-bootstrap:9092 --topic my-topic
```

# References
   * https://strimzi.io/quickstarts/
   * https://github.com/strimzi/strimzi-kafka-operator/tree/0.27.0/examples: This folder contains different examples of Strimzi custom resources and demonstrates how they can be used.
      * https://github.com/strimzi/strimzi-kafka-operator/tree/0.27.0/examples/kafka
    