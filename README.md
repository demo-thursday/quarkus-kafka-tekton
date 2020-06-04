# Reactive and Imperative Architectures with Quarkus and Kafka

Demo data: 2020-06-04

## Source Material

This demo is based on the [Quarkus Kafka quickstart](https://quarkus.io/guides/kafka) demo from the [Quarkus](https://quarkus.io) site.  This demo uses [AMQ Streams](https://www.redhat.com/en/resources/amq-streams-datasheet), or the upstream [Strmizi](https://strimzi.io/) operator to provision a Kafka cluster and topic on an OpenShift cluster.

## Prerequisites

* OpenShift 4.3+ cluster.
* Maven 3.6 installed locally.
* Github account.

## 0. Install Dev Tools

1. Install the OpenShift Pipelines Operator (1.0.1 or newer)
2. Install Nexus: `oc apply -k infra`
3. Install app environments: `oc apply -k app/overlays/all`
4. Install Tekton resourcdes: `oc apply -k tekton`

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

Since we also want health endpoints, let's add the `smallrye-health` extension:
```
mvn quarkus:add-extension -Dextensions="smallrye-health"
```

In case you were wondering, all that command really does is add the health dependency to the POM file.
```
    <dependency>
      <groupId>io.quarkus</groupId>
      <artifactId>quarkus-smallrye-health</artifactId>
    </dependency>
```

You now have a Quarkus project with the dependencies needed to consume and produce messages from Kafka topics!  It's pretty empty , though, so let's add some code.  But first...

## 3. Git Commit and Push

Create a new GitHub repo for your application.  Once you have done this, follow the instructions in your new GitHub repo to init your repo and commit/push your code to master.

For this demo, I'll use [CodeReady Workspaces](https://developers.redhat.com/products/codeready-workspaces/overview).  To do this I'll drop the `devfile.yaml` file found in this repository in the root of my new app repository and change the git url to that of my own repository before the initial commit.

```
git init
git add --all
git commit -m "Initial commit."
git remote add origin <repo url>
git push -u origin master
```

If using CodeReady Workspaces, you can launch your workspace in your browser with the following link:
```
https://<codeready worksapces url>/f?url=<github repo url>
```

Otherwise, open this project in your favorite IDE so we can add some classes and configuration.


## 4. Code and Config

### Create Classes

Once you have the project opened in your IDE, create the first three classes from the guide:
* [PriceGenerator.java](https://quarkus.io/guides/kafka#the-price-generator)
* [PriceConverter.java](https://quarkus.io/guides/kafka#the-price-converter)
* [PriceResource.java](https://quarkus.io/guides/kafka#the-price-resource)

### Add Configuration

Update the main configuration file found at `src/main/resources/application.properties` to include the following configuration:

```
# Configure the Kafka sink (we write to it)
kafka.bootstrap.servers=my-cluster-kafka-bootstrap.kafka:9092
mp.messaging.outgoing.generated-price.connector=smallrye-kafka
mp.messaging.outgoing.generated-price.topic=prices
mp.messaging.outgoing.generated-price.value.serializer=org.apache.kafka.common.serialization.IntegerSerializer

# Configure the Kafka source (we read from it)
mp.messaging.incoming.prices.connector=smallrye-kafka
mp.messaging.incoming.prices.value.deserializer=org.apache.kafka.common.serialization.IntegerDeserializer

# Add Kafka health check.
quarkus.kafka.health.enabled=true

# Package app as an uber jar.
quarkus.package.uber-jar=true
```

This configuration tells the app where to find our Kafka cluster (`my-cluster-kafka-bootstrap.kafka:9092`), as well as the connectors, topics, and serializers/deserializers to use.

Finally, the `quarkus.pakage.uber-jar=true` property, well, indicated that the app should be packaged as an uber jar.  Without this, the app jar would *not* include dependencies, they would be in the `/target/lib` directory after build.

### Commit and Push

If you want to have your build start automatically, first create a Github webhook forr your repo.  Use the URL exposed by the trigger route in the `cicd` project and set the payload type to `json`.  That's all you should need to do.

Finally, commit and push your code.

```
git add --all
git commit -m "Added code."
git push origin master
```

This should trigger your OpenShift Pipelines build.

## 5. Build and Run

If you don't want to setup a webhook, you can trigger your pipeline run manually.

First, edit the file `pipelinerun/run-build.yaml` and update the git url paramater to match your Github repository.

Finally, create a new `PipelineRun`, but make sure you're in the correrct namespace first!

```
oc project cicd
oc create -f pipelinerun/run-build.yaml
```

This should kick off the build pipeline.