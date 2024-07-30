## Debezium Tutorial ##

This repository contains multiple topologies of services defined through various Docker Compose files, designed to help you learn and experiment with Debezium. Each topology represents a different configuration of services, providing a range of learning scenarios and use cases.

## Prerequisites ##


* Docker Desktop installed 


### Core Concepts ###


Before explaining how avro can be configured, let's clarify some important terminology and concepts.</br>

**What is an Avro Schema?** 


An `Avro schema` defines the structure of your data in a compact and efficient binary format. It serves as a blueprint for how data is serialized and deserialized, ensuring consistency and compatibility across different systems.
To illustrate, consider the `cdc-schema.json` file, which defines the `Change Event Value Schema` for the customers table. The provided schema represents a nested Avro schema. 

Here's a breakdown of its structure:

Top-level record (Envelope):

* type: "record" indicates that this schema defines a record
* name: "Envelope" is the name of the record
* namespace: dbserver1.inventory.customers
* fields:
  * before: It represents state of the record before an update.
  * after: It represents the state of the record after an update.
  * source: It contains metadata about the source of the data change, with fields like version, connector, name, ts_ms, snapshot, db, sequence, table, server_id, gtid, file, pos, row, thread, and query.
  * op: A string representing the operation type (e.g., insert, update, delete).
  * ts_ms: A timestamp representing when the operation occurred.
  * transaction: This can be null or a nested record (block) with fields id, total_order, and data_collection_order.</br>

Nested record (Value):

* type: "record"
* name: "Value"
* fields:
  * id: An integer representing the ID of the customer.
  * first_name: A string representing the first name of the customer.
  * last_name: A string representing the last name of the customer.
  * email: A string representing the email of the customer.

This is what an Avro schema looks like with a much simpler example:


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

A record adhering to this schema might look like this in JSON format:

```
{
  "id": 1,
  "name": "Alice",
  "email": "alice@example.com"
}
```

Take a look at [Avro documentation](https://avro.apache.org/docs/1.11.1/specification/_print/#preamble) for more info.




**What is a Serializer in Kafka?**

A serializer in Kafka is responsible for converting data objects into a byte array format that can be sent to Kafka brokers. Since Kafka only understands byte arrays for its messages, serializers ensure that data is correctly transformed for transmission. Kafka provides various serializers for different data types, each suited to specific use cases.


Kafka provides serializers for common data types. Some of them are:

1. **String Serialization**: Converts string data into byte arrays. Commonly used for text-based data such as log entries, text messages, and notifications.

**Example Scenario**: A logging system that sends application log messages to a Kafka topic for centralized logging and analysis.
* Log Message:"INFO: User login successful at 2024-07-29 12:00:00"


2. **Byte Array Serialization**: Handles data already in byte array format, suitable for raw binary datal. Commonly used for transmitting images, files, or any pre-serialized binary data.</br>

**Example Scenario**: A file transfer system that sends binary files (e.g., images) via Kafka.


3. **JSON Serialization**: Converts objects to JSON strings and then to byte arrays for readable and structured data. Ideal for sending structured data such as records or complex objects.</br>

**Example Scenario**: A system that sends user data as JSON objects to Kafka for processing and storage.

* Key: Person ID (e.g., "123")
* Value: JSON representation of the Person object (e.g., {"name": "John Doe", "age": 30})

4. **Avro Serialization**: Uses Avro for data serialization, providing efficient serialization and a rich data structure. Suitable for high-performance and schema-based data processing, ensuring data consistency and type enforcement.

**Example Scenario**: A data pipeline that sends complex records defined by Avro schemas to Kafka for efficient processing.

* Key: Person ID (e.g., "123")
* Value: JSON representation of the Person object (e.g., {"name": "John Doe", "age": 30})






### What is Avro Binary Format? ### 
Imagine you have a message that you want to send to someone, but you want to make it as small and efficient as possible. Avro binary format is like a special way of packaging that message so it's really compact, fast to send, and easy for the receiver to understand.


* Compact Size: Avro binary format takes your data and shrinks it down. This means it uses less space and is faster to move around, like sending a short text instead of a long letter.
* Schema-Based: A schema is like a blueprint or a set of instructions that explains how your data is organized. Avro uses this schema to know exactly what kind of data it's dealing with.

* Self-Describing: When you send your data, Avro includes the schema with it. This way, the person receiving your data knows exactly how to read it, even if they’ve never seen it before. However, in the context of Confluent's Avro serializer, the serializer includes a special identifier (schema ID) and a magic byte at the beginning of the message instead of the full schema. The schema ID refers to the schema stored in the schema registry. Take a look at the [Avro Serializer](https://docs.confluent.io/platform/current/schema-registry/fundamentals/serdes-develop/serdes-avro.html) for more insights.

* Schema Evolution: Over time, you might need to change your data (like adding a new ingredient to your recipe). Avro makes it easy to update your schema without breaking everything.
You can add new fields or change things around, and it ensures old data can still be read.



### Using the Avro message format ####

Avro message format can be configured one of two ways, in the Kafka Connect worker configuration or in the connector configuration. Using Avro in conjunction with the schema registry allows for much more compact messages.

#### Kafka Connect Worker configuration ####

The Compose file configures the Connect service to use the Avro (de-)serializers for the Connect instance. Run the following commands to get the services up and running. 
Files to be used:
* docker-compose-mysql-avro-worker.yaml 
* register-mysql-avro.json

Run the following commands to start the services:

```
export DEBEZIUM_VERSION=2.0.1.Final
docker compose -f docker-compose-mysql-avro-worker.yaml up
```

**note**: You're going to notice five services up and running: one for the MySQL database, one for ZooKeeper, one for the Schema Registry, one for the Kafka broker, and one for Kafka Connect.


To check the status of the Apicurio Registry, use the following command:

```
curl -i -H "Accept:application/json" localhost:8081
```

A successful response will return a status code of 200 OK .

To list all schemas registered in the `Confluent Schema Registry`, use the following command:

```
curl -H "Accept:application/json" localhost:8081/subjects | jq
```

Since no schema is registered, the output will be an empty array.

To list the available connectors on Kafka connect, use the following command:

```
curl -X GET localhost:8083/connectors | jq
```

Since no connector is deployed, the output will be an empty array.


To list the available connector plugins installed, use the following command:

```
curl -X GET localhost:8083/connector-plugins | jq
```

To list all the topics on the Kafka broker, run the following command:
```
docker exec debezium-practical-examples-kafka-1 /kafka/bin/kafka-topics.sh \
    --bootstrap-server kafka:9092 \
    --list
```

Since the connector is not up, there will be no topics other than the default ones.

Let's start out first connector:

```
curl -i -X POST -H "Accept:application/json" -H "Content-Type:application/json" http://localhost:8083/connectors/ -d @register-mysql.json
```

To review the configuration file used to create the connector, run:

```
curl -X GET http://localhost:8083/connectors/inventory-connector/config | jq
```


You can access the first version of the schema for customers values like so:
```
curl -X GET http://localhost:8081/subjects/dbserver1.inventory.customers-value/versions/1 | jq '.schema | fromjson'
```


The Schema Registry provides the `kafka-avro-console-consumer`, a specialized tool for consuming Avro-encoded messages. It integrates with the Confluent Schema Registry to automatically handle Avro serialization and deserialization:


```
docker exec debezium-practical-examples-schema-registry-1 /usr/bin/kafka-avro-console-consumer \
    --bootstrap-server kafka:9092 \
    --from-beginning \
    --property print.key=true \
    --property schema.registry.url=http://schema-registry:8081 \
    --topic dbserver1.inventory.customers
```

The output will look like this:

```
{"id":1001}	{"before":null,"after":{"dbserver1.inventory.customers.Value":{"id":1001,"first_name":"Sally","last_name":"Thomas","email":"sally.thomas@acme.com"}},"source":{"version":"2.0.1.Final","connector":"mysql","name":"dbserver1","ts_ms":1722093025000,"snapshot":{"string":"first_in_data_collection"},"db":"inventory","sequence":null,"table":{"string":"customers"},"server_id":0,"gtid":null,"file":"mysql-bin.000003","pos":157,"row":0,"thread":null,"query":null},"op":"r","ts_ms":{"long":1722093025250},"transaction":null}
{"id":1002}	{"before":null,"after":{"dbserver1.inventory.customers.Value":{"id":1002,"first_name":"George","last_name":"Bailey","email":"gbailey@foobar.com"}},"source":{"version":"2.0.1.Final","connector":"mysql","name":"dbserver1","ts_ms":1722093025000,"snapshot":{"string":"true"},"db":"inventory","sequence":null,"table":{"string":"customers"},"server_id":0,"gtid":null,"file":"mysql-bin.000003","pos":157,"row":0,"thread":null,"query":null},"op":"r","ts_ms":{"long":1722093025250},"transaction":null}
{"id":1003}	{"before":null,"after":{"dbserver1.inventory.customers.Value":{"id":1003,"first_name":"Edward","last_name":"Walker","email":"ed@walker.com"}},"source":{"version":"2.0.1.Final","connector":"mysql","name":"dbserver1","ts_ms":1722093025000,"snapshot":{"string":"true"},"db":"inventory","sequence":null,"table":{"string":"customers"},"server_id":0,"gtid":null,"file":"mysql-bin.000003","pos":157,"row":0,"thread":null,"query":null},"op":"r","ts_ms":{"long":1722093025250},"transaction":null}
{"id":1004}	{"before":null,"after":{"dbserver1.inventory.customers.Value":{"id":1004,"first_name":"Anne","last_name":"Kretchmar","email":"annek@noanswer.org"}},"source":{"version":"2.0.1.Final","connector":"mysql","name":"dbserver1","ts_ms":1722093025000,"snapshot":{"string":"last_in_data_collection"},"db":"inventory","sequence":null,"table":{"string":"customers"},"server_id":0,"gtid":null,"file":"mysql-bin.000003","pos":157,"row":0,"thread":null,"query":null},"op":"r","ts_ms":{"long":1722093025250},"transaction":null}
```

Each message includes detailed information about the state of the record before and after the change, metadata about the event, and the type of operation performed. This allows users to have a comprehensive, real-time view of database modifications. 
Keep the terminal window or tab open and visible on your screen.





If you alter the structure of the `customers` table in the database and trigger another change event, a new version of that schema will be available in the schema registry. Follow these steps to achieve this:

1. Log into the MySQL container (use VSCode for that).

2. Log into the MySQL RDBMS:

```
mysql -u root -p
```

3. Switch to the inventory database

```
use inventory;
```

4. Alter the `customers` table structure: For example, add a new column:

```
ALTER TABLE customers ADD COLUMN phone VARCHAR(20);
```

5. Trigger a change event by updating the table: Insert a new row to reflect the schema change;

```
INSERT INTO customers (id, first_name, last_name, email, phone) VALUES (1050, 'John', 'Doe', 'john.doe@acme.com', '123-456-7890');
```

Now, if you look at your consumer, you will notice that a new line appeared because a new schema version was used for the newly added row, which is necessary to accommodate the structural changes in the table. 

To verifity that a new schema was added, list all schema versions:

```
curl -X GET http://localhost:8081/subjects/dbserver1.inventory.customers-value/versions/ | jq 
```

You should see an array with two elements. Retrieve the first schema version:


```
curl -X GET http://localhost:8081/subjects/dbserver1.inventory.customers-value/versions/1 | jq '.schema | fromjson'
```

Retrieve the second schema version:

```
curl -X GET http://localhost:8081/subjects/dbserver1.inventory.customers-value/versions/2 | jq '.schema | fromjson'
```



Now, open a new consumer using the following command:

```
docker exec debezium-practical-examples-kafka-1 /kafka/bin/kafka-console-consumer.sh \
    --bootstrap-server kafka:9092 \
    --from-beginning \
    --property print.key=true \
    --topic dbserver1.inventory.customers
```


The kafka-console-consumer is a generic consumer script that does not handle any specific serialization format out of the box.

Since our data is stored in Avro binary format, this consumer will display unreadable byte data instead of decoding the Avro messages.
You can also check the logs of the Kafka Connect container to make sure that for both KEY_CONVERTER and VALUE_CONVERTER the avro converter was set properly.

```
docker logs debezium-practical-examples-connect-1 | head -n 50
```

or 

```
docker logs debezium-practical-example-connect-1 | head -n 50 | grep -i converter
```

Now, stop and remove containers, and networks for the next section:

```
docker compose -f docker-compose-mysql-avro-worker.yaml down
```




#### Debezium Connector configuration ####


 We will use the following files:

* docker-compose-mysql-avro-connector.yaml
* register-mysql-avro.json
* register-mysql-nonavro.json



In this example, we will set up two pipelines:

* A pipeline where data is serialized in Avro format.
* A pipeline where data is serialized in JSON or String format.

Run the following commands to start the services:

```
export DEBEZIUM_VERSION=2.0.1.Final
docker compose -f docker-compose-mysql-avro-connector.yaml up
```


**Setting Up the Non-Avro Connector** 

Let's create our first connector, which will handle data that is not in Avro format:

```
curl -i -X POST -H "Accept:application/json" -H "Content-Type:application/json" http://localhost:8083/connectors/ -d @register-mysql-nonavro.json
```

Key Points:

* No Explicit Avro Serializer: The configuration does not explicitly set any Avro Serializer.
* Topics Creation: For each table listed in the `table.include.list` parameter, the connector will create corresponding topics using the specified `topic.prefix`. In this case, the following topics will be created:

  * dbmytest1.inventory.geom
  * dbmytest1.inventory.orders



**Setting Up the Avro Connector**

Next, let's create our second connector, which will handle data serialized in Avro format:

```
curl -i -X POST -H "Accept:application/json" -H "Content-Type:application/json" http://localhost:8083/connectors/ -d @register-mysql-avro.json
```

Key Points:

* Explicit Avro Serializer: The configuration explicitly sets the Avro Serializer.
* Topics Creation: For each table listed in the `table.include.list parameter`, the connector will create corresponding Kafka topics using the specified topic.prefix. In this case, the following topics will be created:

  * dbserver.inventory.customers
  * dbserver.inventory.addresses


To review the configuration for each connector, use the following commands:

For the non-Avro connector:

```
curl -X GET localhost:8083/connectors/nonavro-connector/config | jq
```

For the Avro connector:

```
curl -X GET localhost:8083/connectors/inventory-connector/config | jq
```

To list all the topics created on the Kafka broker upon creation of each connector, run the following command:

```
docker exec debezium-practical-examples-kafka-1 /kafka/bin/kafka-topics.sh \
    --bootstrap-server kafka:9092 \
    --list
```

You can also query the schema registry to determine which topics are set up to handle Avro serialization:

```
curl -H "Accept:application/json" localhost:8081/subjects | jq
```



Now, let's create a consumer from the Kafka container to consume data from a topic where data is serialized in Avro format:

```
docker exec debezium-practical-examples-kafka-1 /kafka/bin/kafka-console-consumer.sh \
    --bootstrap-server kafka:9092 \
    --from-beginning \
    --property print.key=true \
    --topic dbserver1.inventory.customers
```

Since our data is stored in Avro binary format, this consumer will display unreadable byte data instead of decoding the Avro messages.


Let's use a consumer that understands Avro serialization:


```
docker exec debezium-practical-examples-schema-registry-1 /usr/bin/kafka-avro-console-consumer \
    --bootstrap-server kafka:9092 \
    --from-beginning \
    --property print.key=true \
    --property schema.registry.url=http://schema-registry:8081 \
    --topic dbserver1.inventory.customers
```

The will properly decode the messages using the schema registry. If you alter the structure of the customers table in the database and trigger another change event, a new version of that schema will be available in the schema registry. Follow the steps outlined earlier in the documentation for creating consumers to observe the changes.

To consume data from the topic `dbmytest1.inventory.geom`, you can simply create a consumer using the kafka-console-consumer tooL


```
docker exec debezium-practical-examples-kafka-1 /kafka/bin/kafka-console-consumer.sh \
    --bootstrap-server kafka:9092 \
    --from-beginning \
    --property print.key=true \
    --topic dbmytest1.inventory.geom
```

Keep the terminal open and observe the incoming messages. To see the consumer in action, we need to trigger a change event by adding a new row to the `geom` table:


1. Log into the MySQL container (use VSCode for that).

2. Log into the MySQL RDBMS:

```
mysql -u root -p
```

3. Switch to the inventory database

```
use inventory;
```

4. Trigger a change event by updating the table: Insert a new row to reflect the schema change;

```
INSERT INTO geom (id, g, h, size) VALUES (4, ST_GeomFromText('POINT(40.748817 -73.985428)'), NULL)
```

You should see a new message indicating that a change in the geom table was detected.





**note**: If you consume messages from the topic `dbmytest1.inventory.geom` using the kafka-avro-console-consumer you will still encounter problem with deserializing. This because the consumer expects Avro-serialized data but is receiving data in different format;


























































##### EXTRA

This time the consumer will be able to 
The kafka-console-consumer is a generic consumer script that does not handle any specific serialization format out of the box but since our data are stored in a format which is not Avro, it will be able to consume those data. To find out which serializer was used to convert data into binary and viceversa you can log into the connect container:

```
 docker logs debezium-practical-examples-connect-1 | head -n 50
```

or 

```
 docker logs debezium-practical-examples-connect-1 | head -n 50 | grep -i _converter
```



 Let's create our second connector that will deal with data which will deal with data in avro format:


```
curl -i -X POST -H "Accept:application/json" -H "Content-Type:application/json" http://localhost:8083/connectors/ -d @register-mysql-avro.json
```



What is important to highlight here are two things:

1. We do set explicity any Avro Serializer

2. For each table listed in `table.include.list` parameter, the connector will create corresponding Kafka topics using the specified topic.prefix. In our case three topics will be created:


* dbserver.inventory.customers
* dbserver.inventory.addresses



Let's review the configuration for each connector once more time:


```
curl -X GET localhost:8083/connectors/inventory-connector/config | jq
```

```
curl -X GET localhost:8083/connectors/nonavro-connector/config | jq
```



To list all the topics that we created on the Kafka broker upon creation of each connector, run the following command:


```
 docker exec debezium-practical-examples-kafka-1 /kafka/bin/kafka-topics.sh \
    --bootstrap-server kafka:9092 \
    --list
```



Now, open a new consumer from the kafka container using the following command:


```
docker exec debezium-practical-examples-kafka-1 /kafka/bin/kafka-console-consumer.sh \
    --bootstrap-server kafka:9092 \
    --from-beginning \
    --property print.key=true \
    --topic dbmytest1.inventory.orders
```










```
curl -i -X POST -H "Accept:application/json" -H "Content-Type:application/json" http://localhost:8083/connectors/ -d @register-mysql-nonavro.json
```


Now, open a new consumer from the kafka container using the following command:


```
docker exec debezium-practical-examples-kafka-1 /kafka/bin/kafka-console-consumer.sh \
    --bootstrap-server kafka:9092 \
    --from-beginning \
    --property print.key=true \
    --topic dbmytest1.inventory.orders
```



This time the consumer will be able to consume the data since we configuered neither  Avro serialiaer and nor specifiee the location of the schema registry. Keep the terminal window or tab open and visible on your screen.
Let's repeat the steps above one by one. 

If you alter the structure of the `orders` table in the database and trigger another change event, a new version of that schema will be available in the Apicurio Registry. Follow these steps to achieve this:

1. Log into the MySQL container (use VSCode for that).

2. mysql -u root -p

Switch to the inventory database

3. use inventory;

Alter the `geom` table structure: For example, add a new column:

4. ALTER TABLE geom ADD COLUMN size VARCHAR(20);

 Trigger a change event by updating the table: Insert a new row to reflect the schema change;

5. INSERT INTO geom (id, g, h, size) VALUES (4, ST_GeomFromText('POINT(40.748817 -73.985428)'), NULL,'10');





Now, if you look at your consumer, you will notice that a new line appeared because because a change in the `geom` table was detected. However, altough the table structure change that will reflect in no change in the schema. In fact, there will be no schema with the prefix `dbmytest1`.



Create a new consumer that understands Avro :

```
docker exec debezium-practical-examples-schema-registry-1 /usr/bin/kafka-avro-console-consumer \
    --bootstrap-server kafka:9092 \
    --from-beginning \
    --property print.key=true \
    --property schema.registry.url=http://schema-registry:8081 \
    --topic dbserver1.inventory.customers
```


Let's repeat the steps above one by one. 

If you alter the structure of the `orders` table in the database and trigger another change event, a new version of that schema will be available in the Confluent Registry. Follow these steps to achieve this:

1. Log into the MySQL container (use VSCode for that).

2. Log into the MySQL RDBMS:

```
mysql -u root -p
```


3. Switch to the `inventory` database 

```
use inventory;
```
4. Alter the `customers` table structure: For example, add a new column:

```
ALTER TABLE customers ADD COLUMN phone VARCHAR(20);
```

5. Trigger a change event by updating the table: Insert a new row to reflect the schema change;

```
INSERT INTO customers (id, first_name, last_name, email, phone) VALUES (1050, 'John', 'Doe', 'john.doe@acme.com', '123-456-7890');
```



To verify that a new schema was added, list all schema versions:

```
curl -X GET http://localhost:8081/subjects/dbserver1.inventory.customers-value/versions/ | jq 
```

You should see an array with two elements. Retrieve the first schema version:


```
curl -X GET http://localhost:8081/subjects/dbserver1.inventory.customers-value/versions/1 | jq '.schema | fromjson'
```

Retrieve the second schema version:

```
curl -X GET http://localhost:8081/subjects/dbserver1.inventory.customers-value/versions/2 | jq '.schema | fromjson'
```





Now, open a new consumer from the kafka container using the following command that consumes data stored in Avro format:


```
docker exec debezium-practical-examples-kafka-1 /kafka/bin/kafka-console-consumer.sh \
    --bootstrap-server kafka:9092 \
    --from-beginning \
    --property print.key=true \
    --topic dbmytest1.inventory.customers
```




Now please delete the connector to run the next section:
```
curl -i -X DELETE -H "Accept:application/json" localhost:8083/connectors/inventory-connector 
```






























#####
10. check status, pause, and resume the connector worker

http :8083/connectors/inventory-connector/status -b   
curl -H "Accept:application/json" localhost:8083/connectors/inventory-connector/status | jq


http PUT :8083/connectors/inventory-connector/pause -b
curl -i -X PUT -H "Accept:application/json" localhost:8083/connectors/inventory-connector/pause


http PUT :8083/connectors/technologists/resume -b
curl -i -X PUT -H "Accept:application/json" localhost:8083/connectors/inventory-connector/resume


11. delete the running connector

http DELETE :8083/connectors/technologists -b
curl -i -X DELETE -H "Accept:application/json" localhost:8083/connectors/inventory-connector 


#####




The kafka-console-consumer is a generic consumer script that can consume messages from Kafka topics and print them to the console. It does not handle any specific serialization format out of the box and relies on the user to specify the appropriate deserializers if needed.
 The script will not properly decode Avro messages and will display unreadable byte data.

he kafka-avro-console-consumer is a specialized version of the console consumer provided by Confluent. It is designed to work with Avro-encoded messages and integrates with the Confluent Schema Registry to automatically handle Avro serialization and deserialization.

