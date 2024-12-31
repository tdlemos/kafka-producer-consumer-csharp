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

Copy the following into a project file named producer.csproj in the producer subdirectory:

'''xml

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

'''