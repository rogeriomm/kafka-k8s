# Install
   * [5.2.3. Deploying the Cluster Operator to watch multiple namespaces](https://strimzi.io/docs/operators/latest/deploying.html#deploying-cluster-operator-to-watch-multiple-namespaces-str)

# Provision the Apache Kafka cluster
   * Change context
```shell
labtools-k8s set-context cluster
```
   * Create namespace "kafka", "kafka-project"
```shell
if ! kubectl get ns kafka; then kubectl create namespace kafka; fi
for ns in  kafka-project-1 kafka-project-2 kafka-project-3 kafka-project-4 kafka-project-5
do 
  if ! kubectl get ns $ns; then kubectl create namespace $ns; fi
done
kubectl get namespace
```
   * For each namespace listed, install the RoleBindings
```shell
for ns in  kafka-project-1 kafka-project-2 kafka-project-3 kafka-project-4 kafka-project-5
do 
  kubectl create -f k8s/install/cluster-operator/020-RoleBinding-strimzi-cluster-operator.yaml -n $ns
  kubectl create -f k8s/install/cluster-operator/023-RoleBinding-strimzi-cluster-operator.yaml -n $ns
  kubectl create -f k8s/install/cluster-operator/031-RoleBinding-strimzi-cluster-operator-entity-operator-delegation.yaml -n $ns
done
```

   * Deploy the Cluster Operator:
```shell
kubectl create -f k8s/install/cluster-operator -n kafka
```

   * Check the status of the deployment:
```shell
kubectl get deployments -n kafka
```







# Test
   * Create test-cluster project
```shell
kubectl apply -f k8s/kafka-project.yaml
```
 
   * Wait
```shell
kubectl wait kafka/test-cluster --for=condition=Ready --timeout=300s -n kafka-project-1
```

   * Show Kafka resources
```shell
kubectl get k
kubectl get kt
kubectl get ku
```


# Send and receive messages
   * Find the IP address of the Minikube node and port of bootstrap service
```shell
ip=$(kubectl get nodes --output=jsonpath='{range .items[*]}{.status.addresses[?(@.type=="InternalIP")].address}{"\n"}{end}')
port=$(kubectl get service test-cluster-kafka-external-bootstrap -n kafka-project-1 -o=jsonpath='{.spec.ports[0].nodePort}{"\n"}')
echo $ip:$port > broker.txt
cat broker.txt
```

   * Test bootstrap service TCP connectivity
```shell
nc -v $ip $port
```

   * Change context to cluster2
```shell
labtools-k8s set-context cluster2
```

   * Producer. Cluster 2 send messages to Strimzi on cluster 1
```shell
kubectl -n zeppelin run kafka-producer -ti --image=quay.io/strimzi/kafka:0.31.0-kafka-3.2.1 --rm=true --restart=Never -- \
     bin/kafka-console-producer.sh --bootstrap-server $(cat broker.txt) --topic my-topic
```

   * Consumer. Cluster 2 receive messages from Strimzi on cluster 1
```shell
kubectl -n zeppelin run kafka-consumer -ti --image=quay.io/strimzi/kafka:0.31.0-kafka-3.2.1 --rm=true --restart=Never -- \
     bin/kafka-console-consumer.sh --bootstrap-server $(cat broker.txt) --topic my-topic --from-beginning
```

   * Verify that you see the incoming messages in the consumer console.

   * Delete consumer/producer
```shell
kubectl delete -n zeppelin pod kafka-consumer kafka-producer
```

# References
   * https://strimzi.io/quickstarts/
   * https://github.com/strimzi/strimzi-kafka-operator/tree/0.27.0/examples: This folder contains different examples of Strimzi custom resources and demonstrates how they can be used.
      * https://github.com/strimzi/strimzi-kafka-operator/tree/0.27.0/examples/kafka
    