# Kafka Connect Azure IoT Hub
________________________

Kafka Connect Azure IoT Hub is a Kafka source connector for  pumping data from
[Azure IoT Hub](https://azure.microsoft.com/en-us/services/iot-hub/) to [Apache Kafka](https://kafka.apache.org/). This
allows getting the telemetry data sent by Azure IoT Hub connected devices to your Kafka installation, so that it can
 then be consumed by Kafka consumers down the stream.

### Configuration

All the configuration settings required to run Kafka Connector for IoT Hub are in the
"[connect-iothub-source.properties](connect-iothub-source.properties)" file. The properties related to Azure IoT Hub can be obtained from the
[Azure Portal](https://portal.azure.com). Here are the configurable values -

| Config | Value Type | Required | Default Value | Description |
|-------------|-------------|-------------|-------------|----------------|
| Kafka.Topic | String | Yes |  | This is the topic to which the data will be written to. If the topic exists in Kafka, it will be used, else it will be created. To configure the topic properly (partitions, retention policy, etc.) please use the appropriate [Kafka tools](https://kafka.apache.org/documentation#quickstart_createtopic). |
| tasks.max | Int | Yes |  | Maximum number of tasks that should be created for this connector. More tasks means more parallelism. For optimal performance, this should equal the number of IoTHub partitions. |
| IotHub.EventHubCompatibleName | String | Yes |  | The EventHub compatible name for the IoT Hub. In the Azure Portal, you can find the value under to "IoT Hub" >> your hub >> "Messaging" >> "Event Hub-compatible name" |
| IotHub.EventHubCompatibleNamespace | String | Yes |  | The EventHub ccompatible namespace for the IoT Hub. In the Azure Portal, you can find the value under "IoT Hub" >> your hub > "Messaging" >> "Event Hub-compatible endpoint". Use only the name part. For example, for "sb://iothub-ns-toketi-1234-9876.servicebus.windows.net/", use "iothub-ns-toketi-1234-9876" |
| IotHub.AccessKeyName | String | Yes |  | The access key name for the IoT Hub. In the Azure Portal, you can find the value under "IoT Hub" >> your hub >> "Shared access policies". You can use the predefined value "service". |
| IotHub.AccessKeyValue | String | Yes |  | The access key for the IoT Hub. In the Azure Portal, you can find the value under "IoT Hub" >> your hub >> "Shared access policies" >> key name >> "Primary key" |
| IotHub.ConsumerGroup | String | Yes |  | The access key for the IoT Hub. In the Azure Portal, you can find the value under "IoT Hub" >> your hub > "Messaging" >> Consumer groups. You can use the $Default group or create a new one (recommended) |
| IotHub.Partitions | String | Yes |  | The access key for the IoT Hub. In the Azure Portal, navigate to "IoT Hub" >> your hub >> "Messaging" >> "Partitions". |
| IotHub.StartTime | String | No | (Unused if not supplied) | The time from which to start retrieving messages from IoT Hub. The value should be in UTC and in the format yyyy-mm-ddThh:mm:ssZ. This setting is mutually exclusive with IotHub.Offsets. |
| IotHub.Offsets | String | No | (Unused if not supplied) | The offsets for each IoT Hub partition from which to start retrieving messages from IoTHub, as a comma separated string. For example, for 4 partitions, the value would be something like "abc,lmn,pqr,xyz". This setting is mutually exclusive with IotHub.StartTime. |
| BatchSize | Int | No | 100 | The size of each batch for retrieving entries from IoT Hub. |

> Note: If IotHub.StartTime is specified, then the value for IotHub.Offsets is ignored.
> If neither IotHub.StartTime not IotHub.Offsets are specified, then the messages are retrieved from the IoT Hub from
the beginning.

Here is a sample connect-iothub-source.properties file. (Some of these values are not configurable, and hence omitted
  from the list above)
```
connector.class=com.microsoft.azure.iot.kafka.connect.IotHubSourceConnector
name=AzureIotHubConnector
tasks.max=1
Kafka.Topic=IotTopic
IotHub.EventHubCompatibleName=iothub-toketi
IotHub.EventHubCompatibleNamespace=iothub-ns-toketi-18552-1628
IotHub.AccessKeyName=service
IotHub.AccessKeyValue=skdfljiqondfldnm
IotHub.ConsumerGroup=$Default
IotHub.Partitions=4
IotHub.StartTime=2016-11-28T00:00:00Z
IotHub.Offsets=
BatchSize=100
```

### Building and running

###### Pre-requisites

* You need to have an Azure IoT Hub from which you want to copy data to Kafka. Get started
  [here](https://docs.microsoft.com/en-us/azure/iot-hub/).

* You also need to have Apache Kafka installed and running. Get started
[here](https://kafka.apache.org/documentation#quickstart).

###### Steps

Kafka Connect is a generic tool to copy data between Kafka and other systems (like Azure IoT Hub). To copy data from
Azure IoT Hub to Kafka, you need to make the code from this repository available to Kafka Connect and configure it to
use the right Azure IoT Hub and Kafka settings. For more details on using Kafka Connect, you can refer to the
[Kafka Connect user guide](http://docs.confluent.io/3.0.0/connect/userguide.html).

1. Build the source code after cloning this repository using the following command, which will generate a jar file with
all the dependencies embedded in it.
```
sbt assembly
```
Alternatively, you can directly download the jar file for Kafka Connect IoT Hub from [here](https://dl.bintray.com/microsoftazuretoketi/toketi-repo/com/microsoft/azure/iot/kafka-connect-iothub_2.11/0.5/kafka-connect-iothub-assembly_2.11-0.5.jar).

2. Binplace the jar file in the Kafka installation libs folder (usually under KAFKA_HOME/libs).

3. Update the config file "[connect-iothub-source.properties](connect-iothub-source.properties)" with the appropriate values as described in the section
above. Binplace the file "connect-iothub-source.properties" in the Kafka installation config folder (usually under
  KAFKA_HOME/config).

4. Update the Kafka Connect configuration file ("config/connect-standalone.properties" or
"config/connect-distributed.properties") to point to the Kafka bootstrap servers.

5. Make sure Kafka server and Zookeeper are running, as described
[here](https://kafka.apache.org/documentation#quickstart).

6. Start Kafka connector in
[standalone mode](http://docs.confluent.io/3.0.0/connect/userguide.html#standalone-worker-configuration) to stream data
from IoT Hub to Kafka with the following command from Kafka home -

```
bin/connect-standalone.sh config/connect-standalone.properties config/connect-iothub-source.properties
```
For distributed mode, the connector configuration will stay the same. For the detailed steps on how to do this, please
follow the
[Confluent Kafka Connect documentation](http://docs.confluent.io/3.0.0/connect/userguide.html#distributed-worker-configuration)
on this topic.

### Data format

The data in the IoT Hub is retrieved and pumped into the specified Kafka topic as a JSON blob string, with the following
 format. This allows data with any schema to be copied over to Kafka for consumption downstream.

```json
{
  "content":"The raw content sent by the device",
  "systemProperties":"Map of key-value pairs containing system properties",
  "properties":"Map of key-value pairs containing any custom properties set by the device"
}
```

## Future work

* Add support to use the schema name in the message in IoT Hub as the topic name. This will allow sending data with
different schemas to their respective topics. (For example, for a device recording and sending temperature and humidity
  data to IoT Hub, the data can be pumped to "temperature" and "humidity" topics respectively, for easier consumption).

* Add a Kafka sink connector to send messages from Kafka to IoT devices via the IoT Hub.

## Contribute Code

If you want/plan to contribute, we ask you to sign a [CLA](https://cla.microsoft.com/)
(Contribution License Agreement). A friendly bot will remind you about it when you submit
a pull-request.

If you are sending a pull request, we kindly request to check the code style with IntelliJ IDEA,
importing the settings from `Codestyle.IntelliJ.xml`.
