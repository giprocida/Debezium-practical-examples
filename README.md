## Debezium Tutorial ##



This project demonstrates how to easily deploy `Debezium` on Kubernetes using Minikube. Additionally, to provide a practical understanding of Kubernetes resources like ServiceAccount, Role, and RoleBinding, this project explains their purposes and showcases their functionality.

The deployment, which utilizes the permissions defined in the `Role` object, prints all the containers for each pod and all the secrets from the `debezium-example` namespace where Debezium resides.
Before we start, let's first highlight some key differences between Kubernetes running on Docker Desktop and Kubernetes running on Minikube.

## Prerequisites ##


* Docker Desktop installed 


### Overview: How Minikube and Kubernetes on Docker Desktop Work ###

To use Avro-style messages instead of JSON, Avro can be configured one of two ways, in the Kafka Connect worker configuration or in the connector configuration. Using Avro in conjunction with the schema registry allows for much more compact messages. [Avro documentation](https://avro.apache.org/docs/1.11.1/specification/_print/#preamble). For example:
Change Event Value Schema for the customers table (see the file called). It a nested avro schema blah blah blah. Let s keep things simple. This is what a avro schema looks like:


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

Before we move forward, let's clarify some important terminology and concepts:

**Avro Converter**
**Role**: Converts data between Kafka Connect's internal format and Avro format.

**Functions**:

Serialization: Transforms native data (e.g., JSON) into Avro binary format.
Deserialization: Transforms Avro binary data back into its native format.
Schema Management: Manages schema registration and retrieval using a schema registry.

**Avro Serializer**
**Role**: Handles Avro data serialization and deserialization in Kafka producers and consumers.

**Functions**:

Serialization: Converts native data (e.g., Java objects) into Avro binary format.
Deserialization: Converts Avro binary data back into its native format.
Schema Integration: Ensures data conforms to the schema and interacts with the schema registry.


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

