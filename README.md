![Quix - Build realtime applications, faster](https://github.com/quixai/.github/blob/main/profile/Quix-GitHub-banner.jpg)

[//]: <> (This will be a banner image w/ the name e.g. Quix Streams.)

[![Quix on Twitter](https://img.shields.io/twitter/url?label=Twitter&style=social&url=https%3A%2F%2Ftwitter.com%2Fquix_io)](https://twitter.com/quix_io)
[![The Stream Community Slack](https://img.shields.io/badge/-The%20Stream%20Slack-blueviolet)](https://quix.io/slack-invite)
[![Linkedin](https://img.shields.io/badge/LinkedIn-0A66C2.svg?logo=linkedin)](https://www.linkedin.com/company/70925173/)
[![Events](https://img.shields.io/badge/-Events-blueviolet)](https://quix.io/community#events)
[![YouTube](https://img.shields.io/badge/YouTube-FF0000.svg?logo=youtube)](https://www.youtube.com/channel/UCrijXvbQg67m9-le28c7rPA)
[![Docs](https://img.shields.io/badge/-Docs-blueviolet)](https://www.quix.io/docs/sdk/introduction.html)
[![Roadmap](https://img.shields.io/badge/-Roadmap-red)](BLANKroadmap.url)
[![Discussions](https://img.shields.io/badge/-Discussions-green)](BLANKdiscussions)

## What is Quix Streams 🏎

<b>Quix Streams</b> is a library for developing real-time streaming applications focused on simplicity and high-performance. It's designed to be used for high-performance telemetry services when you need to process high volumes of time-series data in nanoseconds precision.

Quix Streams does not use any Domain Specific Language or Embedded framework, it's just another library that you can use in your code base. This means you can use any data processing library of your favourite language together with Quix Streams.

Quix Streams is currently available in:
- Python 
- C#

Using Quix Streams, you currently can:

- Write time-series and non time-series data to a Kafka Topic

- Read time-series and non time-series data from a Kafka Topic

- Process data by reading it from one Kafka Topic and writing back the results to another one.

- Group data by Streams attaching metadata to them

## Features 💍

This library provides several features and solves common problems you face when developing real-time streaming applications. 

<details>
    <summary><b>Streaming context</b></summary>
    Quix Streams handles stream contexts for you, so all the data from one data source is bundled in the same scope. This allows you to attach metadata to streams.
</details>

<details>
    <summary><b>In-memory data processing</b></summary>
    Quix Streams is designed to make in-memory data processing very efficient. It uses several cpu and memory optimizations in conjunction with the message broker capabilities to achieve maximum throughput with very minimum latency.
</details>

<details>
    <summary><b>Built-in time-series buffers</b></summary>
    If you’re sending data at high frequency, processing each message can be costly. The library provides built-in time-series buffers for reading and writing allowing several configurations for balancing between latency and cost.
</details>

<details>
    <summary><b>Support for data frames</b></summary>
    In many use cases, multiple time-series parameters are emitted at the same time, so they share one timestamp. Handling this data independently is wasteful. This library uses an optimized tabular system and can work for instance with Pandas DataFrames natively. Each row has a timestamp and user-defined tags as indexes.
</details>

<details>
    <summary><b>Message splitting</b></summary>
    Quix Streams automatically handles large messages on the producer side, splitting them up if required. You no longer need to worry about Kafka message limits. On the consumer side, those messages are automatically merged back.
</details>

<details>
    <summary><b>Message compression</b></summary>
    Quix Streams compresses messages using built-in codecs like Protobuf, reducing them by an average factor of 10 times. 
</details>

<details>
    <summary><b>Data serialization and de-serialization</b></summary>
    Serialization can be painful, especially if it is done with performance in mind. Quix streams serialize and deserialize native types using different codecs so you don’t have to worry about that.
</details>

<details>
    <summary><b>Multiple data types</b></summary>
    This library allows you to attach different types of data to your timestamps, like Numbers, Strings or Binary data.
</details>

<details>
    <summary><b>Message Broker configuration</b></summary>
    Many configuration settings are needed to use Kafka at its best, and the ideal configuration takes time. The library take care about Kafka configuration by default allowing refined configuration only when needed.
</details>

<details>
    <summary><b>Checkpointing</b></summary>
    Quix Streams allows manual or automatic checkpointing when you read data from a Kafka Topic. This provides the ability to inform the Message Broker that you have already processed messages up to one point.
</details>

<details>
    <summary><b>Horizontal scaling</b></summary>
    Quix Streams handles horizontal scale out of the box via the streaming context feature. You can scale the processing services, from one replica to many and back to one, and the library ensures that the data load is always shared between your replicas consistenly.
</details>

For a detailed overview of features, [visit our docs.](https://www.quix.io/docs/sdk/introduction.html)

## Getting Started 🏎

### Installing the library
You can <b>install</b> the library using the package manager for Python Packages:

```shell
pip install quix-streams
```

### Writing time-series data
Now you can start writing some time-series data into a Kafka Topic. Here's an example of how to <b>Write</b> time-series data into a Kafka Topic with Python.

```python
import time
import datetime
import math

from quixstreaming import StreamingClient

# Client connecting to Kafka instance locally without authentication. 
client = StreamingClient('127.0.0.1:9092')

# Open the output topic where to write data to.
output_topic = client.open_output_topic("your-kafka-topic")

stream = output_topic.create_stream()
stream.properties.name = "Hello World python stream"
stream.properties.metadata["my-metadata"] = "my-metadata-value"
stream.parameters.buffer.time_span_in_milliseconds = 100   # Send data in 100 ms chunks

print("Sending values for 30 seconds.")

for index in range(0, 3000):
    stream.parameters \
        .buffer \
        .add_timestamp(datetime.datetime.utcnow()) \
        .add_value("ParameterA", math.sin(index / 200.0)) \
        .add_value("ParameterB", "string value: " + str(index)) \
        .add_value("ParameterC", bytearray.fromhex("51 55 49 58")) \
        .write()
    time.sleep(0.01)

print("Closing stream")
stream.close()
```

### Reading time-series data
Once we have setup our producer it's time to see how to read data from a topic. Here's an example of how to <b>Read</b> time-series data from a Kafka Topic with Python:

```python
import pandas as pd

from quixstreaming import *
from quixstreaming.app import App

# Client connecting to Kafka instance locally without authentication. 
client = StreamingClient('127.0.0.1:9092')

# Open the input topic where to read data from.
# For testing purposes we don't use consumer group and always read from latest data.
input_topic = client.open_input_topic("your-kafka-topic", consumer_group=None, auto_offset_reset=AutoOffsetReset.Latest)

# read streams
def on_stream(input_stream: StreamReader):

    # read data (as Pandas DataFrame)
    def on_read_pandas(df: pd.DataFrame):
        print(df.to_string())

    input_stream.parameters.on_read_pandas += on_read_pandas

# Hook up events before initiating read to avoid losing out on any data
input_topic.on_stream_received += on_stream

print("Listening to streams. Press CTRL-C to exit.")
# Handle graceful exit
App.run()
```

Quix Streams allows multiple configurations to leverage resources while reading and writing data from a Topic depending on the use case, frequencies, language and data types. 

For full documentation of how to [<b>Read</b>](https://www.quix.io/docs/sdk/read.html) and [<b>Write</b>](https://www.quix.io/docs/sdk/read.html) time-series and non time-series data with Quix Streams, [visit our docs](https://www.quix.io/docs/sdk/introduction.html).

## Using Quix Streams with Quix SaaS

This library doesn't have any dependency to any comercial product, but if you use it together with [Quix SaaS platform](https://www.quix.io) you will get some advantatges out of box during your development process like auto-configuration, monitoring, data explorer, data persistence, pipeline visualization, etc.

## Contribution Guide ✍️

We are very open to any feedback or contributions to this OSS project ❤️. Contributing is a great way to learn and we especially welcome those who haven't contributed to an OSS project before. You can contribute by [reporting a bug or submitting a feature request](URL).

## Need help? 🙋‍♀️
If you run into any problems, ask on #quixhelp in The Stream Slack channel or open a question on our [discussion board](URL)

## Roadmap 🚗

You can find out more about future features in the [public roadmap](URL)

## Community 👭

Join other software engineers in [The Stream](https://quix.io/slack-invite), an online community of people interested in all things data streaming. This is a space to both listen to and share learnings.

🙌  [Join our Slack community!](https://quix.io/slack-invite)

## License 📄

Quix Streams is licensed under the Apache 2.0 license. View a copy of the License file [here](LICENSE.MD).

## Stay in touch 👋

You can follow us on [Twitter](https://twitter.com/quix_io) and [Linkedin](https://www.linkedin.com/company/70925173) where we share what's new, forthcoming community events and the occasional meme.  

If you have any questions or feedback - [write to us](https://quix.io/company/#get-in-touch)!