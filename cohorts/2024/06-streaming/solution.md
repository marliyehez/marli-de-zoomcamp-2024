### Question 1: Redpanda version

```bash
$ docker exec -it redpanda-1 rpk help
rpk is the Redpanda CLI & toolbox

Usage:
  rpk [command]

Available Commands:
  acl         Manage ACLs and SASL users
  cloud       Interact with Redpanda cloud
  cluster     Interact with a Redpanda cluster
  container   Manage a local container cluster
  debug       Debug the local Redpanda process
  generate    Generate a configuration template for related services
  group       Describe, list, and delete consumer groups and manage their offsets
  help        Help about any command
  iotune      Measure filesystem performance and create IO configuration file
  plugin      List, download, update, and remove rpk plugins
  redpanda    Interact with a local Redpanda process
  topic       Create, delete, produce to and consume from Redpanda topics
  version     Check the current version
  wasm        Deploy and remove inline WASM engine scripts

Flags:
  -h, --help      Help for rpk
  -v, --verbose   Enable verbose logging (default: false)

Use "rpk [command] --help" for more information about a command.
```

```bash
$ docker exec -it redpanda-1 rpk version
v22.3.5 (rev 28b2443)
```

### Question 2: Creating a topic

```bash
$ docker exec -it redpanda-1 rpk help topic
Create, delete, produce to and consume from Redpanda topics

Usage:
  rpk topic [command]

Available Commands:
  add-partitions Add partitions to existing topics
  alter-config   Set, delete, add, and remove key/value configs for a topic
  consume        Consume records from topics
  create         Create topics
  delete         Delete topics
  describe       Describe a topic
  list           List topics, optionally listing specific topics
  produce        Produce records to a topic

Flags:
      --brokers strings         Comma-separated list of broker ip:port pairs (e.g. --brokers '192.168.78.34:9092,192.168.78.35:9092,192.179.23.54:9092'). Alternatively, you may set the REDPANDA_BROKERS environment variable with the comma-separated list of broker addresses
      --config string           Redpanda config file, if not set the file will be searched for in the default locations
  -h, --help                    Help for topic
      --password string         SASL password to be used for authentication
      --sasl-mechanism string   The authentication mechanism to use. Supported values: SCRAM-SHA-256, SCRAM-SHA-512
      --tls-cert string         The certificate to be used for TLS authentication with the broker
      --tls-enabled             Enable TLS for the Kafka API (not necessary if specifying custom certs)
      --tls-key string          The certificate key to be used for TLS authentication with the broker
      --tls-truststore string   The truststore to be used for TLS communication with the broker
      --user string             SASL user to be used for authentication

Global Flags:
  -v, --verbose   Enable verbose logging (default: false)

Use "rpk topic [command] --help" for more information about a command.
```

```bash
$ docker exec -it redpanda-1 rpk topic create --help
Create topics.

All topics created with this command will have the same number of partitions,
replication factor, and key/value configs.

For example,

        create -c cleanup.policy=compact -r 3 -p 20 foo bar

will create two topics, foo and bar, each with 20 partitions, 3 replicas, and
the cleanup.policy=compact config option set.

Usage:
  rpk topic create [TOPICS...] [flags]

Flags:
  -d, --dry                        Dry run: validate the topic creation request; do not create topics
  -h, --help                       Help for create
  -p, --partitions int32           Number of partitions to create per topic; -1 defaults to the cluster's default_topic_partitions (default -1)
  -r, --replicas int16             Replication factor (must be odd); -1 defaults to the cluster's default_topic_replications (default -1)
  -c, --topic-config stringArray   key=value; Config parameters (repeatable; e.g. -c cleanup.policy=compact)

Global Flags:
      --brokers strings         Comma-separated list of broker ip:port pairs (e.g. --brokers '192.168.78.34:9092,192.168.78.35:9092,192.179.23.54:9092'). Alternatively, you may set the REDPANDA_BROKERS environment variable with the comma-separated list of broker addresses
      --config string           Redpanda config file, if not set the file will be searched for in the default locations
      --password string         SASL password to be used for authentication
      --sasl-mechanism string   The authentication mechanism to use. Supported values: SCRAM-SHA-256, SCRAM-SHA-512
      --tls-cert string         The certificate to be used for TLS authentication with the broker
      --tls-enabled             Enable TLS for the Kafka API (not necessary if specifying custom certs)
      --tls-key string          The certificate key to be used for TLS authentication with the broker
      --tls-truststore string   The truststore to be used for TLS communication with the broker
      --user string             SASL user to be used for authentication
  -v, --verbose                 Enable verbose logging (default: false)
```

```bash
$ docker exec -it redpanda-1 rpk topic create test-topic
TOPIC       STATUS
test-topic  OK
```

### Question 3: Connecting to the Kafka server

The output of the last command:

```Python
producer.bootstrap_connected()
>> True
```

### Question 4: Sending data to the stream

How much time did it take? Where did it spend most of the time?

- Sending the messages
- Flushing
- Both took approximately the same amount of time

The output:

```Python
Sent: {'number': 0}
Sent: {'number': 1}
Sent: {'number': 2}
Sent: {'number': 3}
Sent: {'number': 4}
Sent: {'number': 5}
Sent: {'number': 6}
Sent: {'number': 7}
Sent: {'number': 8}
Sent: {'number': 9}
took 0.51 seconds
```

From the output, we can see that it took **0.51 seconds**. Since it sent 10 messages, it took at least 0.5 \* 10 = 0.50 seconds to send the messages. Therefore, most of the time was spent on **sending the messages**.

#### Reading data with rpk:

```bash
docker exec -it redpanda-1 rpk topic consume test-topic
{
  "topic": "test-topic",
  "value": "{\"number\": 0}",
  "timestamp": 1710757834114,
  "partition": 0,
  "offset": 0
}
{
  "topic": "test-topic",
  "value": "{\"number\": 1}",
  "timestamp": 1710757834166,
  "partition": 0,
  "offset": 1
}
{
  "topic": "test-topic",
  "value": "{\"number\": 2}",
  "timestamp": 1710757834217,
  "partition": 0,
  "offset": 2
}
{
  "topic": "test-topic",
  "value": "{\"number\": 3}",
  "timestamp": 1710757834268,
  "partition": 0,
  "offset": 3
}
{
  "topic": "test-topic",
  "value": "{\"number\": 4}",
  "timestamp": 1710757834319,
  "partition": 0,
  "offset": 4
}
{
  "topic": "test-topic",
  "value": "{\"number\": 5}",
  "timestamp": 1710757834371,
  "partition": 0,
  "offset": 5
}
{
  "topic": "test-topic",
  "value": "{\"number\": 6}",
  "timestamp": 1710757834423,
  "partition": 0,
  "offset": 6
}
{
  "topic": "test-topic",
  "value": "{\"number\": 7}",
  "timestamp": 1710757834474,
  "partition": 0,
  "offset": 7
}
{
  "topic": "test-topic",
  "value": "{\"number\": 8}",
  "timestamp": 1710757834525,
  "partition": 0,
  "offset": 8
}
{
  "topic": "test-topic",
  "value": "{\"number\": 9}",
  "timestamp": 1710757834577,
  "partition": 0,
  "offset": 9
}
```


### Question 5: Sending the Trip Data
Took about **106 seconds**.
```Python
t0 = time.time()

topic_name = 'green-trips'


for row in df_green.itertuples(index=False):
    row_dict = {col: getattr(row, col) for col in row._fields}
    producer.send(topic_name, value=row_dict)
    
producer.flush()
    
t1 = time.time()
print(f'took {(t1 - t0):.2f} seconds')
```
Output:
```Python
# Many outputs showing dataframe and the number of batch
-------------------------------------------
Batch: 270
-------------------------------------------
-------------------------------------------
Batch: 274
-------------------------------------------
took 106.14 seconds
```

### Question 6: Parsing the data

```Python
green_stream \
    .writeStream \
    .format("console") \
    .start()
```
How does the record look after parsing (including the rest of the output):

```
> 4/03/18 14:17:47 WARN ResolveWriteToStream: Temporary checkpoint location created which is deleted normally when the query didn't fail: /tmp/temporary-937cb844-6a4b-4468-a910-ca076cc07edc. If it's required to delete it under any circumstances, please set spark.sql.streaming.forceDeleteTempCheckpointLocation to true. Important to know deleting temp checkpoint folder is best effort.
> 24/03/18 14:17:47 WARN ResolveWriteToStream: spark.sql.adaptive.enabled is not supported in streaming DataFrames/Datasets and will be disabled.


<pyspark.sql.streaming.StreamingQuery at 0x7f9884c78150>


-------------------------------------------
Batch: 0
-------------------------------------------
+--------------------+---------------------+------------+------------+---------------+-------------+----------+
|lpep_pickup_datetime|lpep_dropoff_datetime|PULocationID|DOLocationID|passenger_count|trip_distance|tip_amount|
+--------------------+---------------------+------------+------------+---------------+-------------+----------+
| 2019-10-01 00:18:11|  2019-10-01 00:22:38|          43|         263|            1.0|          0.8|       0.0|
| 2019-10-01 00:09:31|  2019-10-01 00:24:47|         255|         228|            2.0|          7.5|       0.0|
| 2019-10-01 00:37:40|  2019-10-01 00:41:49|         181|         181|            1.0|          0.9|       0.0|
| 2019-10-01 00:08:13|  2019-10-01 00:17:56|          97|         188|            1.0|         2.52|      2.26|
| 2019-10-01 00:35:01|  2019-10-01 00:43:40|          65|          49|            1.0|         1.47|      1.86|
| 2019-10-01 00:28:09|  2019-10-01 00:30:49|           7|         179|            1.0|          0.6|       1.0|
| 2019-10-01 00:28:26|  2019-10-01 00:32:01|          41|          74|            1.0|         0.56|       0.0|
| 2019-10-01 00:14:01|  2019-10-01 00:26:16|         255|          49|            1.0|         2.42|       0.0|
| 2019-10-01 00:03:03|  2019-10-01 00:17:13|         130|         131|            1.0|          3.4|      2.85|
| 2019-10-01 00:26:02|  2019-10-01 00:39:58|         112|         196|            1.0|         5.88|       0.0|
| 2019-10-01 00:18:11|  2019-10-01 00:22:38|          43|         263|            1.0|          0.8|       0.0|
| 2019-10-01 00:09:31|  2019-10-01 00:24:47|         255|         228|            2.0|          7.5|       0.0|
| 2019-10-01 00:37:40|  2019-10-01 00:41:49|         181|         181|            1.0|          0.9|       0.0|
| 2019-10-01 00:08:13|  2019-10-01 00:17:56|          97|         188|            1.0|         2.52|      2.26|
| 2019-10-01 00:35:01|  2019-10-01 00:43:40|          65|          49|            1.0|         1.47|      1.86|
| 2019-10-01 00:28:09|  2019-10-01 00:30:49|           7|         179|            1.0|          0.6|       1.0|
| 2019-10-01 00:28:26|  2019-10-01 00:32:01|          41|          74|            1.0|         0.56|       0.0|
| 2019-10-01 00:14:01|  2019-10-01 00:26:16|         255|          49|            1.0|         2.42|       0.0|
| 2019-10-01 00:03:03|  2019-10-01 00:17:13|         130|         131|            1.0|          3.4|      2.85|
+--------------------+---------------------+------------+------------+---------------+-------------+----------+
```

### Question 7: Most popular destination
The most popular destination is **East Harlem North	(LocationID: 74)**.

```Python
popular_destinations = (green_stream
    .withColumn('timestamp', F.current_timestamp())
    .groupBy(F.window(F.col("timestamp"), "5 minutes"), "DOLocationID")
    .count()
    .orderBy(F.desc('count'))
)

query = popular_destinations \
    .writeStream \
    .outputMode("complete") \
    .format("console") \
    .option("truncate", "false") \
    .start()

query.awaitTermination(20) # 20 seconds timeout
```
Output:
```
-------------------------------------------
Batch: 0
-------------------------------------------
+------------------------------------------+------------+-----+
|window                                    |DOLocationID|count|
+------------------------------------------+------------+-----+
|{2024-03-19 03:25:00, 2024-03-19 03:30:00}|74          |35482|
|{2024-03-19 03:25:00, 2024-03-19 03:30:00}|42          |31884|
|{2024-03-19 03:25:00, 2024-03-19 03:30:00}|41          |28122|
|{2024-03-19 03:25:00, 2024-03-19 03:30:00}|75          |25680|
|{2024-03-19 03:25:00, 2024-03-19 03:30:00}|129         |23860|
|{2024-03-19 03:25:00, 2024-03-19 03:30:00}|7           |23066|
|{2024-03-19 03:25:00, 2024-03-19 03:30:00}|166         |21690|
|{2024-03-19 03:25:00, 2024-03-19 03:30:00}|236         |15826|
|{2024-03-19 03:25:00, 2024-03-19 03:30:00}|223         |15084|
|{2024-03-19 03:25:00, 2024-03-19 03:30:00}|238         |14636|
|{2024-03-19 03:25:00, 2024-03-19 03:30:00}|82          |14584|
|{2024-03-19 03:25:00, 2024-03-19 03:30:00}|181         |14564|
|{2024-03-19 03:25:00, 2024-03-19 03:30:00}|95          |14488|
|{2024-03-19 03:25:00, 2024-03-19 03:30:00}|244         |13466|
|{2024-03-19 03:25:00, 2024-03-19 03:30:00}|61          |13212|
|{2024-03-19 03:25:00, 2024-03-19 03:30:00}|116         |12678|
|{2024-03-19 03:25:00, 2024-03-19 03:30:00}|138         |12288|
|{2024-03-19 03:25:00, 2024-03-19 03:30:00}|97          |12100|
|{2024-03-19 03:25:00, 2024-03-19 03:30:00}|49          |10442|
|{2024-03-19 03:25:00, 2024-03-19 03:30:00}|151         |10306|
+------------------------------------------+------------+-----+
only showing top 20 rows
```
From the lookup CSV:
```Bash
{
  'LocationID': 74,
  'Borough': 'Manhattan',
  'Zone': 'East Harlem North',
  'service_zone': 'Boro Zone'
}
```