# RabbitMQ consumer and sender

A simple docker container that will receive messages from a RabbitMQ queue.  The receiver will receive a single message at a time (per instance), and sleep for 1 second to simulate performing work.  When adding a massive amount of queue messages, container will scale out according to the event source (RabbitMQ).

## Pre-requisites

* [k3s Cluster](https://k3s.io/)

## Setup

This setup will go through creating a RabbitMQ queue on the cluster and deploying this consumer and publisher.

First you should clone the project:

```cli
git clone https://github.com/vwake7/sample-go-rabbitmq
cd sample-go-rabbitmq
```

### Creating a RabbitMQ queue

#### [Install Helm](https://helm.sh/docs/using_helm/)

#### Install RabbitMQ via Helm

Since the Helm stable repository was migrated to the [Bitnami Repository](https://github.com/helm/charts/tree/master/stable/rabbitmq), add the Bitnami repo and use it during the installation:

```cli
helm repo add bitnami https://charts.bitnami.com/bitnami
```

##### Helm 3

RabbitMQ Helm Chart version 7.0.0 or later

```cli
helm install rabbitmq --set auth.username=user --set auth.password=PASSWORD bitnami/rabbitmq --wait
```

**Notes:**

* If you are running the rabbitMQ image on KinD, you will run into permission issues unless you set `volumePermissions.enabled=true`. Use the following command if you are using KinD:

    ```cli
    helm install rabbitmq --set auth.username=user --set auth.password=PASSWORD --set volumePermissions.enabled=true bitnami/rabbitmq --wait
    ```

* With RabbitMQ Helm Chart version 6.x.x or earlier, username and password should be specified with rabbitmq.username and rabbitmq.password parameters [https://hub.helm.sh/charts/bitnami/rabbitmq](https://hub.helm.sh/charts/bitnami/rabbitmq)

##### Helm 2

RabbitMQ Helm Chart version 7.0.0 or later

```cli
helm install --name rabbitmq --set auth.username=user --set auth.password=PASSWORD bitnami/rabbitmq --wait
```

**Notes:**

* If running this demo on a computer with a ARM Processor, refer to the earlier note
* If using KinD refer to the earlier note
* For RabbitMQ Helm Chart version 6.x.x or earlier, refer to the earlier note

#### Wait for RabbitMQ to Deploy

⚠️ Be sure to wait until the deployment has completed before continuing. ⚠️

```cli
kubectl get po

NAME         READY   STATUS    RESTARTS   AGE
rabbitmq-0   1/1     Running   0          3m3s
```

### Deploying a RabbitMQ consumer

#### Deploy a consumer

```cli
kubectl apply -f deploy/deploy-consumer.yaml
```

#### Validate the consumer has deployed

```cli
kubectl get deploy
```

[This consumer](https://github.com/kedacore/sample-go-rabbitmq/blob/master/cmd/receive/receive.go) is set to consume one message per instance, sleep for 1 second, and then acknowledge completion of the message.  This is used to simulate work.  

### Publishing messages to the queue

#### Deploy the publisher job

The following job will publish 300 messages to the "hello" queue the deployment is listening to. As the queue builds up, You can modify the exact number of published messages in the `deploy-publisher-job.yaml` file.

```cli
kubectl apply -f deploy/deploy-publisher-job.yaml
```

#### Validate the deployment scales

```cli
kubectl get deploy -w
```

You can watch the pods spin up and start to process queue messages.  As the message length continues to increase, more pods will be pro-actively added.  

You can see the number of messages vs the target per pod as well:

```cli
kubectl get hpa
```

## Cleanup resources

```cli
kubectl delete job rabbitmq-publish
kubectl delete deploy rabbitmq-consumer
helm delete rabbitmq
```
