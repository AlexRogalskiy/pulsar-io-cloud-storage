---
description: The Cloud Storage sink connector pulls messages from Pulsar topics and persists messages to Cloud Storage.
author: ["ASF"]
contributors: ["ASF"]
language: Java
document: 
source: "https://github.com/streamnative/pulsar-io-cloud-storage/tree/v2.5.1/src/main/java/org/apache/pulsar/io/jcloud"
license: Apache License 2.0
tags: ["Pulsar IO", "Cloud Storage", "Sink"]
alias: Cloud Storage Sink
features: ["Use the Cloud Storage sink connector to sync data from Pulsar"]
license_link: "https://www.apache.org/licenses/LICENSE-2.0"
icon: "https://a0.awsstatic.com/libra-css/images/logos/aws_logo_smile_179x109.png"
download: "https://github.com/streamnative/pulsar-io-cloud-storage/releases/download/v2.5.1/pulsar-io-cloud-storage-2.5.1.nar"
support: StreamNative
support_link: https://streamnative.io
support_img: "/images/connectors/streamnative.png"
dockerfile: 
owner_name: ""
owner_img: ""
id: "io-cloud-storage-sink"
---

Cloud Storage Sink connector for Pulsar

The Cloud Storage Sink connector supports exporting data from Pulsar Topic to cloud storage in Avro, json, parquet and other formats, such as aws-s3, google-GCS, etc. According to your environment, the Cloud Storage sink connector can guarantee exactly-once support for exporting to cloud storage.

The Cloud Storage Sink connector receives data from Pulsar, and then uploads the data to the cloud storage. The partition program is used to split the data of each pulsar partition into multiple blocks. Each data block is represented as an object. The virtual path corresponds to the topic, using the pulsar partition and the starting offset of this data block for encoding. If the partition program is not specified in the configuration, the default partition program that retains the pulsar partition is used. The size of each data block depends on the number of records written to the cloud storage and the architecture compatibility.

## Features
The Pulsar IO Cloud Storage Sink connector provides the following features:

- Exactly Once Delivery: Records that are exported using a deterministic partitioner are delivered with exactly-once semantics regardless of the eventual consistency of Cloud Storage.
- Data Format with or without a Schema: Plug and play, the connector supports writing data to cloud storage in Avro, JSON and parquet formats. Generally, the connector can accept any format that provides an implementation of the Format interface.
- Partitioner: The connector supports the TimeBasedPartitioner class based on the Pulsar message publishTime TimeStamp. Time-based partitioning options are daily or hourly.
- More object storage support: The connector uses jclouds as an implementation of cloud storage. You can use the jclouds object storage jar to connect to more types of object storage. If you need to customize credentials, you can register ʻorg.apache.pulsar.io.jcloud.credential.JcloudsCredential` via SPI.

The Cloud Storage sink connector pulls messages from Pulsar topics and persists messages to Cloud Storage.

# Installation

```
git clone https://github.com/streamnative/pulsar-io-cloud-storage.git
cd pulsar-io-cloud-storage/
mvn clean install -DskipTests
cp target/pulsar-io-cloud-storage-0.0.1.nar $PULSAR_HOME/pulsar-io-cloud-storage-0.0.1.nar
```

# Configuration 

The Cloud Storage sink connector supports the following properties.

## Cloud Storage sink connector configuration

| Name | Type|Required | Default | Description |
|------|----------|----------|---------|-------------|
| `provider` |String| True | " " (empty string) | The Cloud Storage type. for example, `aws-s3`,`google-cloud-storage`|
| `accessKeyId` |String| True | " " (empty string) | The Cloud Storage access key ID. |
| `secretAccessKey` | String| True | " " (empty string) | The Cloud Storage secret access key. |
| `role` | String |False | " " (empty string) | The Cloud Storage role. |
| `roleSessionName` | String| False | " " (empty string) | The Cloud Storage role session name. |
| `bucket` | String| True | " " (empty string) | The Cloud Storage bucket. |
| `endpoint` | String| False | " " (empty string) | The Cloud Storage endpoint. |
| `formatType` | String| False | "json" | The data format type: JSON, Avro, or Parquet. By default, it is set to JSON. |
| `partitionerType` | String| False |"partition" | The partition type. It can be configured by partition or time. By default, the partition type is configured by partition. |
| `timePartitionPattern` | String| False |"yyyy-MM-dd" | The format pattern of the time partition. For details, refer to the Java date and time format. |
| `timePartitionDuration` | String| False |"1d" | The time interval divided by time, such as 1d, or 1h. |
| `batchSize` | int | False |10 | The number of records submitted in batch. |
| `batchTimeMs` | long | False |1000 | The interval for batch submission. |
| `sliceTopicPartitionPath` | Boolean | False |1000 |  When it is set to `true`, split the partitioned topic name into separate folders in the bucket path. |
| `withMetadata` | Boolean | False |false | Save message attributes to metadata. |
| `withTopicPartitionNumber` | Boolean | False |true | When it is set to `true`, include topic partition number to the object path. |
| `gcsServiceAccountKeyFilePath` | String | False | "" | Path to the GCS credentials file. If empty, the credentials file will be read from the `GOOGLE_APPLICATION_CREDENTIALS` environment variable. |
| `gcsServiceAccountKeyFileContent` | String | False | "" | The contents of the JSON service key file. If empty, credentials are read from `gcsServiceAccountKeyFilePath` file. |

## Configure Cloud Storage sink connector

Before using the Cloud Storage sink connector, you need to create a configuration file through one of the following methods.

* JSON 

    ```json
    {
       "tenant": "public",
       "namespace": "default",
       "name": "cloud-storage-sink",
       "inputs": [
          "user-avro-topic"
       ],
       "archive": "connectors/pulsar-io-cloud-storage-0.0.1.nar",
       "parallelism": 1,
       "configs": {
          "provider": "aws-s3",
          "accessKeyId": "accessKeyId",
          "secretAccessKey": "secretAccessKey",
          "role": "none",
          "roleSessionName": "none",
          "bucket": "testBucket",
          "region": "local",
          "endpoint": "us-standard",
          "formatType": "parquet",
          "partitionerType": "time",
          "timePartitionPattern": "yyyy-MM-dd",
          "timePartitionDuration": "1d",
          "batchSize": 10,
          "batchTimeMs": 1000
       }
    }
    ```
    
* YAML

    ```yaml
    tenant: "public"
    namespace: "default"
    name: "cloud-storage-sink"
    inputs: 
      - "user-avro-topic"
    archive: "connectors/pulsar-io-cloud-storage-0.0.1.nar"
    parallelism: 1
    
    configs:
      provider: "aws-s3",
      accessKeyId: "accessKeyId"
      secretAccessKey: "secretAccessKey"
      role: "none"
      roleSessionName: "none"
      bucket: "testBucket"
      region: "local"
      endpoint: "us-standard"
      formatType: "parquet"
      partitionerType: "time"
      timePartitionPattern: "yyyy-MM-dd"
      timePartitionDuration: "1d"
      batchSize: 10
      batchTimeMs: 1000
    ```

# Usage

1. Prepare the Cloud Storage service. In this example, we use `s3mock` as an example.

    ```
    docker pull apachepulsar/s3mock:latest
    docker run -p 9090:9090 -e initialBuckets=pulsar-integtest apachepulsar/s3mock:latest
    ```

2. Put the `pulsar-io-cloud-storage-2.5.1.nar` in the Pulsar connector catalog.

    ```
    cp pulsar-io-cloud-storage-2.5.1.nar $PULSAR_HOME/connectors/pulsar-io-cloud-storage-2.5.1.nar
    ```

3. Start Pulsar in the standalone mode.

    ```
    $PULSAR_HOME/bin/pulsar standalone
    ```

4. Run the Cloud Storage sink connector locally.

    ```
    $PULSAR_HOME/bin/pulsar-admin sink localrun --sink-config-file cloud-storage-sink-config.yaml
    ```

5. Send Pulsar messages. In this example, the schema of the topic only supports `avro` or `json`.

   ```java
     try (
                PulsarClient pulsarClient = PulsarClient.builder()
                        .serviceUrl("pulsar://localhost:6650")
                        .build();
                Producer<TestRecord> producer = pulsarClient.newProducer(Schema.AVRO(TestRecord.class))
                        .topic("public/default/test-parquet-avro")
                        .create();
                ) {
                List<TestRecord> testRecords = Arrays.asList(
                        new TestRecord("key1", 1, null),
                        new TestRecord("key2", 1, new TestRecord.TestSubRecord("aaa"))
                );
                for (TestRecord record : testRecords) {
                    producer.send(record);
                }
            }
   ```

6. Validate Cloud Storage data.
    To get the path, you can use the [jclould](https://jclouds.apache.org/start/install/) to verify the file, as shown below.
    ```java
    Properties overrides = new Properties();
    overrides.put(“jclouds.s3.virtual-host-buckets”, “false”);
    BlobStoreContext blobStoreContext = ContextBuilder.newBuilder(“aws-s3”)
            .credentials(
                    “accessKeyId”,
                    “secretAccessKey”
            )
            .endpoint(“http://localhost:9090”) // replace to s3mock url
            .overrides(overrides)
            .buildView(BlobStoreContext.class);
    BlobStore blobStore = blobStoreContext.getBlobStore();
    final long sequenceId = FunctionCommon.getSequenceId(message.getMessageId());
    final String path = “public/default/test-parquet-avro” + File.separator + “2020-09-14" + File.separator + sequenceId + “.parquet”;
    final boolean blobExists = blobStore.blobExists(“testBucket”, path);
    Assert.assertTrue(“the sink record does not exist”, blobExists);
    ```
    You can find the sink data in your `testBucket` Bucket. The path is something like `public/default/test-parquet-avro/2020-09-14/1234.parquet`.
    The path consists of three parts, the basic part of the topic, partition information, and format suffix.
    - Basic part of topic: `public/default/test-parquet-avro/`
        This part consists of the tenant, namespace, and topic name of the input topic.
    - Partition information: `2020-09-14/${messageSequenceId}`
        The date is generated based on the `partitionerType` parameter in the configuration. And the `${messageSequenceId}` is generated by `FunctionCommon.getSequenceId(message.getMessageId())`.
    - Format suffix: `.parquet`
        This part is generated based on the `formatType` parameter in the configuration.
