# Pepper-Box for Azure Event Hubs
___

Pepper-Box is kafka load generator plugin for jmeter. It allows to send kafka messages of type plain text (JSON, XML, CSV or any other custom format) as well as java serialized objects.

*For all the details abuot Pepper-Box, like requirements, setup and how to get started, please refer to the [original project](https://github.com/GSLabDev/pepper-box).*

The version you can find here has been modified in order to leverage the kafka endpoint provided by [Azure Event Hubs](https://docs.microsoft.com/en-us/azure/event-hubs/event-hubs-for-kafka-ecosystem-overview). You can easily set up a load test and send messages to your Event Hub with a simple command line instruction.

### Environment setup

In order to start sending messages, you need to setup your Azure environment first: please refer to Microsoft official documentation to [create a Kafka enabled Event Hub namespace](https://docs.microsoft.com/en-us/azure/event-hubs/event-hubs-quickstart-kafka-enabled-event-hubs#create-a-kafka-enabled-event-hubs-namespace).

In terms of editor, I would recommend [Visual Studio Code](https://code.visualstudio.com/) for development and test. Adding a simple extension, you can also monitor the messages coming into your Event Hub.

## Getting started

After cloning the repo, you just need to provide the credentials to allow your application to send data to your Event Hub. These parameters must be added in the producer properties file.
In this sample, we added the [files folder](https://github.com/erryB/pepper-box/tree/master/files), which includes 2 properties files, for simple and complete tests. In both of them, as you can see below, the *bootstrap.servers* requires your Event Hub FQDN and the *sasl.jaas.config* requires your Event Hubs Connection String. Follow [these instructions](https://docs.microsoft.com/en-us/azure/event-hubs/event-hubs-get-connection-string) if you don't know where to find these information.
Eventually, you need to use the name of the specific Event Hub you want to address in your namespace as *kafka.topic.name*.

```
bootstrap.servers=<your-EH-FQDN>:9093
security.protocol=SASL_SSL
sasl.mechanism=PLAIN
sasl.jaas.config=org.apache.kafka.common.security.plain.PlainLoginModule required username="$ConnectionString" password="<your-EH-connectionString>";
kafka.topic.name=<your-EH>
```

*And that's it!*
If you use VSCode you can create a debug configuration or you can just run the application specifying all the required parameters, as described in Pepperbox original project:
--schema-file
--producer-config-file
--throughput-per-producer
--test-duration
--num-producers

*One more tip if you use VSCode*: you can add and configure [Azure Event Hub Explorer](https://marketplace.visualstudio.com/items?itemName=Summer.azure-event-hub-explorer) to actually see your messages coming in your Event Hub!

### What's different from the original Pepper-box project

Existing kafka applications can work with Event Hubs, but *Apache Kafka protocol 1.0 and later* is required, as mentioned in the [official documentation](https://docs.microsoft.com/en-us/azure/event-hubs/event-hubs-for-kafka-ecosystem-overview). Therefore, we updated the kafka version in the *pom.xml* file to *1.0.2*

```<groupId>pepper-box</groupId>
    <artifactId>pepper-box</artifactId>
    <version>1.0</version>
    <properties>
        <jmeter.version>3.1</jmeter.version>
        <kafka.version>1.0.2</kafka.version>
```

Be careful because this change has some effects: we had to change imports for *Security protocol* and *Time* to be able to build the project.  

Microsoft provides a [sample](https://github.com/Azure/azure-event-hubs-for-kafka) and some [guidelines](https://docs.microsoft.com/en-us/azure/event-hubs/event-hubs-quickstart-kafka-enabled-event-hubs) to explain how to stream messages into Event Hubs for Apache Kafka. Following the same approach, we made some changes to *PepperboxLoadGenerator.java* file: we added a static method called *createProducer* which returns a *KafkaProducer* to be used as final parameter in the constructor. All the checks on the input files are performed within this method, which is called in the *main* before the creation of the *PepperboxLoadGenerator*, as you can see below.

```
public static void main(String[] args) {
    [...]
    try {
        int totalThreads = options.valueOf(threadCount);
        for (int i = 0; i < totalThreads; i++) {
            final KafkaProducer<String, String> producer = createProducer(options.valueOf(producerConfig));
            PepperBoxLoadGenerator jsonProducer = new PepperBoxLoadGenerator(options.valueOf(schemaFile), options.valueOf(producerConfig), options.valueOf(throughput), options.valueOf(duration), producer);
            jsonProducer.start();
        }
    } catch (Exception e) {
        LOGGER.log(Level.SEVERE, "Failed to generate load", e);
    }
}
```
