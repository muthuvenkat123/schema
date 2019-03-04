# Registryless Avro Converter

This is an Avro converter for Kafka Connect that does not depend on Confluent Schema Registry. It
shares much of the same underlying code as Confluent's `AvroConverter`, and should work the same in
practice less any features that deal with the Schema Registry itself.

We developed this converter at MailChimp to facilitate R&D with Connect and use cases where pushing
the Schema Registry Avro Format through Kafka was not desirable or we couldn't justify the overhead
of a Schema Registry.

## Using the Converter

### Setup

1. You must have **Java 8** as your runtime environment.
2. **Confluent Platform 4.0**: as this plugin relies on various Confluent libraries that are
  distributed with CP (e.g. their avro converter, etc). You can download the tarball
  [here](http://packages.confluent.io/archive/4.0/confluent-oss-4.0.0-2.11.zip). Vanilla Kafka
  Connect won't have some of these dependencies.
3. Configure a `plugin.path` in your connect setup and drop a RegistrylessAvroConverter JAR in that
  path so that its picked up with Kafka Connect starts.

Once you've confirmed that the binary is in place, then in a properties file or JSON connector
configuration you can specify this converter for keys and/or values.

### Configuration

To use the RegistrylessAvroConverter, simply provide it in the `key.converter` or `value.converter`
setting for your connector. RAC can run with or without an explicit reader or writer schema. If an
explicit schema is not provided, the schema used will be determined at runtime.

**N.B.** Schemas determined at runtime could vary depending on how your connector is implemented
and how it generates Connect Data Schemas. We recommend understanding the semantics of your
Connectors before using the schemaless configuration for sources.

Here's an example of how we might define RAC for use with keys and values without an explicit schema
in standalone mode:

```
key.converter=me.frmr.kafka.connect.RegistrylessAvroConverter
value.converter=me.frmr.kafka.connect.RegistrylessAvroConverter
```

And this is how you would define a RAC _with_ an explicit schema:

```
key.converter=me.frmr.kafka.connect.RegistrylessAvroConverter
key.converter.schema.path=/path/to/schema/file.avsc
value.converter=me.frmr.kafka.connect.RegistrylessAvroConverter
value.converter.schema.path=/path/to/schema/file.avsc
```

## Building the Converter

This converter uses Gradle. Building the project is as simple as:

```
./gradlew build
```

## General Notes

* This project is a bit weird because it's designed to be run in a Kafka Connect runtime. So
  all of the dependencies are `compileOnly` because they're available on the classpath at runtime.
* If you're testing this locally, it's a bit weird in much the same way. You'll need to copy
  the JAR into an appropriate `lib/` folder so the class is visible to Kafka Connect for local
  testing.

## Contributing

Pull requests and issues are welcome! If you think you've spotted a problem or you just have a
question do not hesitate to [open an issue](https://github.com/farmdawgnation/registryless-avro-converter/issues/new).

### Roadmap

We're planning on making the following major improvements moving forward:

* No longer requiring a reader schema be provided
* Supporting variable schema cache size
