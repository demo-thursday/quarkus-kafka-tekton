# Reactive and Imperative Architectures with Quarkus and Kafka

Demo data: 2020-06-04

## Source Material

This demo is based on the [Quarkus Kafka quickstart](https://quarkus.io/guides/kafka) demo from the [Quarkus](https://quarkus.io) site.  This demo uses [AMQ Streams](https://www.redhat.com/en/resources/amq-streams-datasheet), or the upstream [Strmizi](https://strimzi.io/) operator to provision a Kafka cluster and topic on an OpenShift cluster.

## Prerequisites

* OpenShift 4.3+ cluster.
* Maven 3.6 installed locally.
* Github account.

## 1. Install Kafka

First, start by installing Kafka on your OpenShift cluster.  We'll use the upstream Strimzi operator.  If you have a subscription for Red Hat AMQ or Red Hat Integration, you can also use the fully supported AMQ Streams operator.

After you have logged into OpenShift as an admin:

1. Create a new project/namespace called `kafka`.
2. In the Admin dashboard, go to `Operators -> OperatorHub`
3. Search for "Strimzi".
4. Install the operator into the `kafka` namespace.
5. Once installed, click on `Installed Operators`, then click on Strimzi.
6. Now, click `Kafka -> Create Kafka`
7. You can keep the defaults (name = my-cluster, 3 x Zookeeper, 3 x replica, ephemeral storage) and create.

It will take a few minutes for the images to pull and the zookeepers and replicas to spin up.  Once this is done we will create a topic.

1. Click on `Installed Operators`, then click on Strimzi.
2. Click on `Kafka Topic -> Create Kafka Topic`.
3. Change the `name` attribute in the yaml to `prices`, then create the new topic.

We're done for now with Kafka.  We now have a Kafka cluster and a "prices" topic ready to use.

## 2. Generate Quarkus App

On your local machine, run the following command to generate the base process.

```
mvn io.quarkus:quarkus-maven-plugin:1.4.2.Final:create \
    -DprojectGroupId=org.acme \
    -DprojectArtifactId=kafka-quickstart \
    -Dextensions="kafka"
cd kafka-quickstart
```

You now have a Quarkus project with the dependencies needed to consume and produce messages from Kafka topics!  It's pretty empty , though, so let's add some code.

Open this project in your favorite IDE so we can add some classes and configuration.

### 3. Code and Config

Once you have the project opened in your IDE, create the first three classes from the guide:
* [PriceGenerator.java](https://quarkus.io/guides/kafka#the-price-generator)
* [PriceConverter.java](https://quarkus.io/guides/kafka#the-price-converter)
* [PriceResource.java](https://quarkus.io/guides/kafka#the-price-resource)


