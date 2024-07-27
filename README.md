## Debezium Tutorial ##

deploys the topology of services as defined in the

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
* * before: It represents state of the record before an update.
* * after: It represents the state of the record after an update.
* * source: It contains metadata about the source of the data change, with fields like version, connector, name, ts_ms, snapshot, db, sequence, table, server_id, gtid, file, pos, row, thread, and query.
* * op: A string representing the operation type (e.g., insert, update, delete).
* * ts_ms: A timestamp representing when the operation occurred.
* * transaction: This can be null or a nested record (block) with fields id, total_order, and data_collection_order.</br>

Nested record (Value):

* type: "record"
* name: "Value"
* fields:
* * id: An integer representing the ID of the customer.
* * first_name: A string representing the first name of the customer.
* * last_name: A string representing the last name of the customer.
* * email: A string representing the email of the customer.

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


```

A record adhering to this schema might look like this in JSON format
{
  "id": 1,
  "name": "Alice",
  "email": "alice@example.com"
}
```

Take a look at [Avro documentation](https://avro.apache.org/docs/1.11.1/specification/_print/#preamble).


**Avro Converter**

Helps Kafka Connect interact with other systems by converting data into Avro binary format:


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

#### Kafka Connect Worker configuration ####
Configuring Avro at the Kafka Connect worker involves using the same steps above for MySQL but instead using the docker-compose-mysql-avro-worker.yaml configuration file instead. The Compose file configures the Connect service to use the Avro (de-)serializers for the Connect instance and starts one more additional service, the Confluent schema registry.


