## Debezium Tutorial ##

deploys the topology of services as defined in the

## Prerequisites ##


* Docker Desktop installed 


### Core Concepts ###


Before explaining how avro can be configured, let's clarify some important terminology and concepts.
**What is an Avro Schema?** 
An Avro schema defines the structure of your data in a compact and efficient binary format. It serves as a blueprint for how data is serialized and deserialized, ensuring consistency and compatibility across different systems.
To illustrate, consider the `cdc-schema.json` file, which defines the `Change Event Value Schema` for the customers table. 

For example, take a look at the `cdc-schema.json` file. It's the `Change Event Value Schema` for the `customers` table.
The provided schema represents a nested Avro schema. Here's a breakdown of its structure:

Top-level record (Envelope):

* name: Envelope
* namespace: dbserver1.inventory.customers
* fields:
* * before: It represents state of the record before an update.
* * after: It represents the state of the record after an update.
* * source: It contains metadata about the source of the data change, with fields like version, connector, name, ts_ms, snapshot, db, sequence, table, server_id, gtid, file, pos, row, thread, and query.
* * op: A string representing the operation type (e.g., insert, update, delete).
* * ts_ms: A timestamp representing when the operation occurred.
* * transaction: This can be null or a nested record (block) with fields id, total_order, and data_collection_order.
Nested record (Value):

name: Value
fields:
id: An integer representing the ID of the customer.
first_name: A string representing the first name of the customer.
last_name: A string representing the last name of the customer.
email: A string representing the email of the customer.
Source record:

name: Source
namespace: io.debezium.connector.mysql
fields: Various metadata fields as mentioned above.
Transaction record (block):

name: block
namespace: event
fields: id, total_order, and data_collection_order.
This schema is typical for capturing change data capture (CDC) events, where each event captures the state before and after the change, along with metadata about the source and the nature of the change.


It a nested avro schema. Let s keep things simple. This is what a avro schema looks like:


```
{
  "type": "record",
  "name": "User",
  "namespace": "com.example",
  "fields": [
    {
      "name": "id",
      "type": "int"
    },
    {
      "name": "name",
      "type": "string"
    },
    {
      "name": "email",
      "type": "string"
    }
  ]
}
```

Breakdown of the schema:

* type: "record" indicates that this schema defines a record.
* name: "User" is the name of the record.
* namespace: "com.example" specifies the namespace, which helps to avoid name conflicts.
* fields: An array that defines the fields within the record.
  * id: An integer field representing the user's ID.
  * name: A string field representing the user's name.
  * email: A string field representing the user's email address.


```

A record adhering to this schema might look like this in JSON format
{
  "id": 1,
  "name": "Alice",
  "email": "alice@example.com"
}
```

Take a look at[Avro documentation](https://avro.apache.org/docs/1.11.1/specification/_print/#preamble).


**Avro Converter**

Helps Kafka Connect interact with other systems by converting data into Avro binary format


* Serialization: Imagine you have data in a simple format like JSON, which is easy to read but not very efficient to send over the network. The Avro Converter takes this JSON data and packs it into a smaller, more efficient format called Avro binary format
* Deserialization: When the data reaches its destination, it needs to be unpacked back into its original format, like JSON, so that it can be easily used.
The Avro Converter handles this unpacking, transforming the Avro binary format back into JSON.
* Schema Management: The Avro Converter works with a schema registry, which is a central place where all these blueprints (schemas) are stored. This registry helps keep track of all the different versions of the data structure, ensuring that both the sender and receiver understand the data in the same way.

**Avro Serializer**

Helps Kafka producers and consumers work with data in Avro format.


* Serialization: When a Kafka producer wants to send data, it needs to pack this data efficiently.
The Avro Serializer converts data from its original format, like Java objects, into the compact Avro binary format. This makes the data smaller and faster to send.
* Deserialization: When a Kafka consumer receives data, it needs to unpack this data back into its original format to use it.
The Avro Serializer takes the Avro binary format and converts it back into the original format, like Java objects, so that the data can be easily processed.
* Schema Integration: Just like the Avro Converter, the Avro Serializer uses schemas (blueprints) to know how the data should be structured. It works with the schema registry to check that the data matches the expected structure before sending or after receiving it. This ensures that the data is consistent and reliable, as both the sender and receiver use the same blueprint to understand the data.


### What is Avro Binary Format? ### 
Imagine you have a message that you want to send to someone, but you want to make it as small and efficient as possible. Avro binary format is like a special way of packaging that message so it's really compact, fast to send, and easy for the receiver to understand.


* Compact Size: Avro binary format takes your data and shrinks it down. This means it uses less space and is faster to move around, like sending a short text instead of a long letter.
* Schema-Based: A schema is like a blueprint or a set of instructions that explains how your data is organized. Avro uses this schema to know exactly what kind of data it's dealing with.

* Self-Describing: When you send your data, Avro includes the schema with it. This way, the person receiving your data knows exactly how to read it, even if they’ve never seen it before. However, in the context of Confluent's Avro serializer, the serializer includes a special identifier (schema ID) and a magic byte at the beginning of the message instead of the full schema. The schema ID refers to the schema stored in the schema registry. Take a look at the [Avro Serializer](https://docs.confluent.io/platform/current/schema-registry/fundamentals/serdes-develop/serdes-avro.html) for more insights.

* Schema Evolution: Over time, you might need to change your data (like adding a new ingredient to your recipe). Avro makes it easy to update your schema without breaking everything.
You can add new fields or change things around, and it ensures old data can still be read.



### Using the Avro message format ####

Avro message format can be configured one of two ways, in the Kafka Connect worker configuration or in the connector configuration. Using Avro in conjunction with the schema registry allows for much more compact messages. 

#### Minikube ####
Minikube is designed to run a local Kubernetes cluster on your machine for development and testing. It creates a VM (or uses a container) to run the Kubernetes cluster, and this VM/container runs its own Docker daemon, isolated from your host machine’s Docker daemon. You start Minikube using a command, choosing a VM driver (like VirtualBox) or a container driver (like Docker). For example, you might use minikube start.

In terms of networking, the Minikube VM/container has its own network configuration. Services are not accessible from localhost by default; you need to use commands like minikube service to access them or find the Minikube IP and NodePort. For example, you can access a service using `minikube service service-name`. Take a look at the [Minikube documentation](https://minikube.sigs.k8s.io/docs/handbook/accessing/) for more insights.  To use Docker images with Minikube, you need to build them inside Minikube’s Docker environment by pointing your shell to Minikube’s Docker daemon, using a command like eval $(minikube docker-env). Take a look at the [Minikube documentation](https://minikube.sigs.k8s.io/docs/tutorials/docker_desktop_replacement/#steps) for more insights.

Minikube offers flexibility and is closer to production setups but requires manual configuration for network access and image management.


#### Kubernetes on Docker Desktop ####

Docker Desktop integrates Kubernetes to provide an easy-to-use local Kubernetes environment for development. It runs a lightweight VM to host the Docker daemon and Kubernetes components, and both Docker and Kubernetes share the same Docker daemon, simplifying image management. Kubernetes can be enabled through Docker Desktop settings with a simple toggle, and the Kubernetes components are managed within the same VM used by Docker.

In terms of networking, Docker Desktop provides seamless network integration, making services accessible via localhost and the host’s IP address. For example, you can access a service using `curl http://localhost:NodePort.` Docker images built on your host machine are immediately available to the Kubernetes cluster without additional configuration, and you can build an image using a command like `docker build -t my-image:latest .`.

Docker Desktop simplifies development with a unified Docker and Kubernetes environment, providing seamless network access and a shared Docker daemon.



## Deploying Debezium on Minikube ##
This guide complements the official Debezium documentation by highlighting key steps and providing additional resources to help you avoid common issues. Follow the [Debezium documentation](https://debezium.io/documentation/reference/stable/operations/kubernetes.html) as the primary source for deploying Debezium on Minikube.


Kubernetes files to deploy:

1. A Secret called `debezium-secret` containing base64-encoded credentials for connecting to the MySQL database.

2. A Role called `connector-configuration-role` which grants read access to a specific Kubernetes Secret named `debezium-secret` within the `debezium-example` namespace.

3. A RoleBinding called `connector-configuration-role-binding` which binds the Role to the Kafka Connect cluster service account.

4. A Deployment called `mysql` for deploying a MySQL database.

5. A service called `mysql`. 


Custom Resources using Strimzi Kafka CRD:

1. Kafka which deploys the Kafka cluster

2. KafkaConnect which deploys the Kafka Connect cluster with the necessary plugins.

3. KafkaConnector which configures the connector for capturing changes from MySQL and streaming them to Kafka.


#### Important Notes ####
Issue with KafkaConnect configuration: When applying the configuration file from the [Debezium documentation](https://debezium.io/documentation/reference/stable/operations/kubernetes.html), you may encounter issues with the version field specified in the YAML file. To resolve this, update the version from 3.1.0 to 3.7.1, or alternatively, use the `debezium-connect-cluster.yaml` file directly.

Issue with Shell Commands: When creating the KafkaConnector using shell commands as suggested in the tutorial, you may encounter issues with reading database credentials. It is recommended to store the YAML configuration in a file and apply it using kubectl apply -f ..... For more information, refer to this Stack Overflow post for troubleshooting [KafkaConnector not reaading database credentials](https://stackoverflow.com/questions/75831703/strimzi-kafkaconnector-not-reading-database-credentials-from-secrets).













Follow this [Debezium documentation](https://debezium.io/documentation/reference/stable/operations/kubernetes.html) for deploying Debezium on Minikube in combinatior of the yaml files and other info written here. All the object resources that are used in the documentation are already in this repoo ready for use so you won t have to copy paste anything you will only need to replace the IP address present in the `debezium-kafka-connector.yaml` file with the IP address of the registry where you can push and pull. 


ps: . This is because you will probably get into some issues when running creating the KafkaConnector using the shell (see Creating a Debezium Connector), you will probably run into some issues if are following the documentation . check here for more info 
















###

kubectl get crds | grep strimzi | awk '{print $1}' | xargs kubectl delete crd

###









## Run the deployment on minikube ##

Follow these steps:

1. Start minikube:
```
minikube start
```

2. Configure your shell to use Minikube's Docker daemon. 
Minikube has it own Docker daemon. To build Docker images directly within Minikube, you need to point your shell to 
Minikube's Docker daemon.
Check the Docker environment for Minikube:
```
minikube docker-env
```
Point your shell to minikube's docker-daemon, run:
```
eval $(minikube docker-env)
```
To make sure that your Docker CLI is now configured to use the Docker daemon inside your Minikube environment, run:
```
docker info | grep -i "name:"
```
If the command returns `Name: minikube`, it confirms that the Docker daemon is running inside the Minikube VM.


3. Build your Docker image. Navigate go to the directory containing your Dockerfile and build your image. using the provided Dockerfile:
```
docker build -t gprocida6g/print-secrets:1.0 .
```

4. Verify the Docker Image. After building the image, verify that it was built inside your Minikube environment by listing the images:

```
docker images
```
You should see `gprocida6g/objects-printer:1.0` in the list of images.


If you wish to delete a Docker image, you can use the following command:

```
docker rmi <image-name> or <image-id>
```

For example, to delete the image `gprocida6g/objects-printer:1.0`, you can run:

```
docker rmi gprocida6g/objects-printer:1.0
```


## Create a Role Resource ##

Use an imperative command:

```
kubectl create role pod-listing-role \
  --verb=get,list \
  --resource=pods,secrets \
  --namespace=kafka \
  --dry-run=client -o yaml > my-role.yaml
```

Or just use the provided `my-role.yaml` file. <br />
The Role defined in the `my-role.yaml` file grants permissions to perform get and list operations on the pods and secrets resources within the `debezium-example` namespace. This means any ServiceAccount, User, or Group that this Role is bound to can view and list the pods and secrets in that namespace. Apply the Role:
```
kubectl apply -f my-role.yaml
```


## Create a Rolebinding resource ## 

Use an imperative command:

```
kubectl create rolebinding pod-listing-binding \
  --role=pod-listing-role \
  --serviceaccount=kafka:default \
  --dry-run=client \
  --namespace=kafka \
  -o yaml > my-role-binding.yaml
```

or just use the provided `my-role-binding.yaml` file. <br />
The RoleBinding defined in the file `my-role-binding.yaml` grants the permissions defined in the `pod-listing-role` Role to the default ServiceAccount in the debezium-example namespace. This means that the default ServiceAccount in the debezium-example namespace will have the permissions to get and list pods and secrets within that namespace. Apply the RoleBinding:
```
kubectl apply -f my-role-binding.yaml
```



## Create the deployment

kubectl create deploy print-secrets \
  --image=gprocida6g/print-secrets:1.0 \
  --namespace=debezium-example\
  --dry-run=client -o yaml > print-secrets.yaml

# Modify the pod configuration by adding

imagePullPolicy: Never 

within the 'containers' field








### other useful command:

docker rmi gprocida6g/print-pods:1.0 (if you wish to update the image you need first to delete the image and then build it again)







# Apply all the newly created resources.





You could push the image to your Docker Hub or use the field imagePullPolicy: Never (image 
already present locally).


# Create a role resource named 'pod-listing-role' with specific permission

kubectl create role pod-listing-role \
  --verb=get,list \
  --resource=pods,secrets \
  --namespace=kafka \
  --dry-run=client -o yaml > my-role.yaml


## Create a rolebinding resource that binds the pod-listing-role to the default ServiceAccount

kubectl create rolebinding pod-listing-binding \
  --role=pod-listing-role \
  --serviceaccount=kafka:default \
  --dry-run=client \
  --namespace=kafka \
  -o yaml > my-role-binding.yaml


## Create a pod using the image giprocida/axual-debug:1.0 

kubectl run debug-pod \
  --image=giprocida/axual-debug:1.0 \
  --namespace=kafka \
  --dry-run=client -o yaml -- sleep 4000 > debug-pod.yaml

## Modify the pod configuration by adding

imagePullPolicy: Never 

within the 'containers' field

Apply all the newly created resources.


The previously described procedure is automated through the use of the scripts: create-resources.sh and run-debug.sh.

Run the script create-resource.sh to create all the necessary resources.
Run the script run-debug.sh to apply all the newly created resources.



## How to run it ##

Follow these steps:


## Build the docker image



You could push the image to your Docker Hub or use the field imagePullPolicy: Never (image 
already present locally).


# Create a role resource named 'pod-listing-role' with specific permission

kubectl create role pod-listing-role \
  --verb=get,list \
  --resource=pods,secrets \
  --namespace=kafka \
  --dry-run=client -o yaml > my-role.yaml


## Create a rolebinding resource that binds the pod-listing-role to the default ServiceAccount

kubectl create rolebinding pod-listing-binding \
  --role=pod-listing-role \
  --serviceaccount=kafka:default \
  --dry-run=client \
  --namespace=kafka \
  -o yaml > my-role-binding.yaml


## Create a pod using the image giprocida/axual-debug:1.0 

kubectl run debug-pod \
  --image=giprocida/axual-debug:1.0 \
  --namespace=kafka \
  --dry-run=client -o yaml -- sleep 4000 > debug-pod.yaml

## Modify the pod configuration by adding

imagePullPolicy: Never 

within the 'containers' field

Apply all the newly created resources.


The previously described procedure is automated through the use of the scripts: create-resources.sh and run-debug.sh.

Run the script create-resource.sh to create all the necessary resources.
Run the script run-debug.sh to apply all the newly created resources.

