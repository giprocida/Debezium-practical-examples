# Use the Debezium Connect base image
ARG DEBEZIUM_VERSION=2.0.1.Final
FROM quay.io/debezium/connect:$DEBEZIUM_VERSION

LABEL maintainer="Debezium Community"

# Switch to root to install dependencies
USER root

# Install necessary libraries
RUN microdnf -y install libaio && microdnf clean all

# Switch back to kafka user
USER kafka

# Expose Kafka Connect port
EXPOSE 8083

# Define volumes
VOLUME ["/kafka/data","/kafka/logs","/kafka/config"]

# Copy entrypoint script and configurations
COPY --chown=kafka:kafka docker-entrypoint.sh /docker-entrypoint.sh
COPY --chown=kafka:kafka log4j.properties $KAFKA_HOME/config/log4j.properties
COPY --chown=kafka:kafka docker-maven-download.sh /usr/local/bin/docker-maven-download

# Switch to root to set permissions
USER root
# Set execute permissions for the entrypoint script
RUN chmod +x /docker-entrypoint.sh
# Set execute permissions for the docker-maven-download script
RUN chmod +x /usr/local/bin/docker-maven-download

# Switch back to kafka user
USER kafka

# Set environment variables
ENV KAFKA_CONNECT_PLUGINS_DIR=$KAFKA_HOME/connect \
    EXTERNAL_LIBS_DIR=$KAFKA_HOME/external_libs \
    CONNECT_PLUGIN_PATH=$KAFKA_CONNECT_PLUGINS_DIR \
    MAVEN_DEP_DESTINATION=$KAFKA_HOME/libs \
    CONFLUENT_VERSION=7.0.1 \
    AVRO_VERSION=1.10.1 \
    APICURIO_VERSION=2.1.5.Final \
    GUAVA_VERSION=31.0.1-jre

# Create necessary directories
RUN mkdir -p "$KAFKA_CONNECT_PLUGINS_DIR" "$EXTERNAL_LIBS_DIR"

# Download necessary plugins
RUN docker-maven-download confluent kafka-connect-avro-converter "$CONFLUENT_VERSION" fd03a1436f29d39e1807e2fb6f8e415a
RUN docker-maven-download confluent kafka-connect-avro-data "$CONFLUENT_VERSION" d27f30e9eca4ef1129289c626e9ce1f1
RUN docker-maven-download confluent kafka-avro-serializer "$CONFLUENT_VERSION" c72420603422ef54d61f493ca338187c
RUN docker-maven-download confluent kafka-schema-serializer "$CONFLUENT_VERSION" 9c510db58119ef66d692ae172d5b1204
RUN docker-maven-download confluent kafka-schema-registry-client "$CONFLUENT_VERSION" 7449df1f5c9a51c3e82e776eb7814bf1
RUN docker-maven-download confluent common-config "$CONFLUENT_VERSION" aab5670de446af5b6f10710e2eb86894
RUN docker-maven-download confluent common-utils "$CONFLUENT_VERSION" 74bf5cc6de2748148f5770bccd83a37c
RUN docker-maven-download central org/apache/avro avro "$AVRO_VERSION" 35469fee6d74ecbadce4773bfe3a204c
RUN docker-maven-download apicurio "$APICURIO_VERSION" bd7adf3f599968db4529fe4592d07bc8
RUN docker-maven-download central com/google/guava guava "$GUAVA_VERSION" bb811ca86cba6506cca5d415cd5559a7

# Set entrypoint and command
ENTRYPOINT ["/docker-entrypoint.sh"]
CMD ["start"]
