# .NET 8 Kafka - Producer & consumer

In this example, you will build C# client applications which produce and consume messages from an Apache Kafka® cluster.

This example assumes that you already have .NET Core (>= 8.0) installed.

## Create a Project

```cmd

mkdir kafka-dotnet-getting-started 
cd kafka-dotnet-getting-started
mkdir producer
mkdir consumer

```

Next we’ll create two different C# project files, one for the producer and one for the consumer. The project files specify the output type of project artifact which is an executable for both the producer and consumer. It also specifies the required dependencies that the .NET platform needs for the project.

Copy the following into a project file named **producer.csproj** in the **producer** subdirectory:

```xml

<Project Sdk="Microsoft.NET.Sdk">

  <PropertyGroup>
    <OutputType>Exe</OutputType>
    <TargetFramework>net8.0</TargetFramework>
    <StartupObject>Producer</StartupObject>
  </PropertyGroup>

  <ItemGroup>
    <PackageReference Include="Confluent.Kafka" Version="2.3" />
  </ItemGroup>

</Project>

```

Copy the following into a project file named **consumer.csproj** in the **consumer** subdirectory:

```XML

<Project Sdk="Microsoft.NET.Sdk">

  <PropertyGroup>
    <OutputType>Exe</OutputType>
    <TargetFramework>net8.0</TargetFramework>
    <StartupObject>Consumer</StartupObject>
  </PropertyGroup>

  <ItemGroup>
    <PackageReference Include="Confluent.Kafka" Version="2.3" />
  </ItemGroup>

</Project>

```

## Build Producer

Create the .NET producer application by pasting the following C# code into a file named **producer/producer.cs**.

```CSharp

using Confluent.Kafka;

const string topic = "purchases";

string[] users = { "eabara", "jsmith", "sgarcia", "jbernard", "htanaka", "awalther" };
string[] items = { "book", "alarm clock", "t-shirts", "gift card", "batteries" };

var config = new ProducerConfig
{
    // User-specific properties that you must set
    BootstrapServers = "<BOOTSTRAP SERVERS>",
    SaslUsername     = "<CLUSTER API KEY>",
    SaslPassword     = "<CLUSTER API SECRET>",

    // Fixed properties
    SecurityProtocol = SecurityProtocol.SaslSsl,
    SaslMechanism    = SaslMechanism.Plain,
    Acks             = Acks.All
};

using (var producer = new ProducerBuilder<string, string>(config).Build())
{
    var numProduced = 0;
    Random rnd = new Random();
    const int numMessages = 10;
    for (int i = 0; i < numMessages; ++i)
    {
        var user = users[rnd.Next(users.Length)];
        var item = items[rnd.Next(items.Length)];

        producer.Produce(topic, new Message<string, string> { Key = user, Value = item },
            (deliveryReport) =>
            {
                if (deliveryReport.Error.Code != ErrorCode.NoError) {
                    Console.WriteLine($"Failed to deliver message: {deliveryReport.Error.Reason}");
                }
                else {
                    Console.WriteLine($"Produced event to topic {topic}: key = {user,-10} value = {item}");
                    numProduced += 1;
                }
            });
    }

    producer.Flush(TimeSpan.FromSeconds(10));
    Console.WriteLine($"{numProduced} messages were produced to topic {topic}");
}

```

Fill in the appropriate **BootstrapServers** endpoint and any additional security configuration needed inline where the **ProducerConfig** object is instantiated.

You can test the syntax before preceding by compiling with:

```cmd

cd producer
dotnet build producer.csproj

```

## Build Consumer

Next, create the .NET consumer application by pasting the following C# code into a file named **consumer/consumer.cs**.

```CSharp

using Confluent.Kafka;
using System;
using System.Threading;

var config = new ConsumerConfig
{
    // User-specific properties that you must set
    BootstrapServers = "<BOOTSTRAP SERVERS>",
    SaslUsername     = "<CLUSTER API KEY>",
    SaslPassword     = "<CLUSTER API SECRET>",

    // Fixed properties
    SecurityProtocol = SecurityProtocol.SaslSsl,
    SaslMechanism    = SaslMechanism.Plain,
    GroupId          = "kafka-dotnet-getting-started",
    AutoOffsetReset  = AutoOffsetReset.Earliest
};

const string topic = "purchases";

CancellationTokenSource cts = new CancellationTokenSource();
Console.CancelKeyPress += (_, e) => {
    e.Cancel = true; // prevent the process from terminating.
    cts.Cancel();
};

using (var consumer = new ConsumerBuilder<string, string>(config).Build())
{
    consumer.Subscribe(topic);
    try {
        while (true) {
            var cr = consumer.Consume(cts.Token);
            Console.WriteLine($"Consumed event from topic {topic}: key = {cr.Message.Key,-10} value = {cr.Message.Value}");
        }
    }
    catch (OperationCanceledException) {
        // Ctrl-C was pressed.
    }
    finally{
        consumer.Close();
    }
}

```

Fill in the appropriate **BootstrapServers** endpoint and any additional security configuration needed inline where the **ConsumerConfig** object is instantiated.

You can test the syntax before preceding by compiling with:

```cmd

cd ../consumer
dotnet build consumer.csproj
cd ..

```