---
title: Working with Avro
layout: post
categories: [userguides, mapreduce, 1.0.0-rc4]
tags: [mapreduce-ug]
version: 1.0.0-rc4
order : 9
description: Working with Avro
---

# Working with Avro
All data stored in Kiji cells are serialized and deserialized using <a href="http://avro.apache.org">Apache Avro</a>. Each logical unit of data in Avro has a type. The type of a datum is called a schema. A schema may be a simple primitive such as an integer or string, or may be a composition of other schemas such as an array or record.

Avro data is serialized and deserialized in several programming languages using their language-specific types. In Java, for example, data with an Avro INT schema is manifested as a java.lang.Integer object. A MAP schema is manifested as a java.util.Map. The full mapping from Avro schemas to Java types can be found in the <a href="http://avro.apache.org/docs/current/api/java/org/apache/avro/generic/package-summary.html#package_description">Avro documentation</a>.

## Using Avro in KijiSchema
When implementing a gatherer's gather() method or a producer or bulk importer's produce() method, use the KijiRowData object to read data from the current Kiji table row. Avro serialization is taken care of for you; the call to `getValue()` or `getMostRecentValue()` will automatically return the type specified in the table layout. For example, to read an Avro string value from the most recent value of the info:name column, call KijiRowData.getMostRecentValue("info", "name"). It will be returned to you as a java.lang.CharSequence. If you are reading data with a more complex schema, KijiSchema will return that instead.

To write typed data into a Kiji cell from your producer or bulk importer's produce() method, use the Context passed into KijiProducer's produce() method. The put() method is overloaded to accept a variety of Java types; Avro serialization is handled for you. If you need to store data with a more complex type, you can pass this Avro object into the put method() as well.  For example, to write custom Address complex Avro type:

{% highlight java %}

    final EntityId user = table.getEntityId("Abraham Lincoln");
    final Address addr = new Address();
    addr.setAddr1("1600 Pennsylvania Avenue");
    addr.setCity("Washington");
    addr.setState("DC");
    addr.setZip("20500");

    context.put(user, "info", "address", addr);
{% endhighlight %}

## Using Avro in MapReduce

You may also find it useful to read and write Avro data between your mappers and reducers. Jobs run by Kiji can also use Avro data for MapReduce keys and values. To use Avro data as your gatherer, mapper, or reducer's output key, use the org.apache.mapred.AvroKey class. You must also specify the writer schema for your key by implementing the org.kiji.mapreduce.AvroKeyWriter interface. For example, to output an Integer key from a gatherer:

{% highlight java %}
public class MyAvroGatherer
    extends KijiGatherer<AvroKey<Integer>, Text>
    implements AvroKeyWriter {
  // ...

  @Override
  protected void gather(KijiRowData input, GathererContext context)
      throws IOException, InterruptedException {
    // ...
    context.write(new AvroKey<Integer>(5), new Text("myvalue"));
  }

  @Override
  public Schema getAvroKeyWriterSchema(Configuration conf) throws IOException {
    return Schema.create(Schema.Type.INTEGER);
  }
}
{% endhighlight %}

Likewise, an org.apache.mapred.AvroValue may be used for Avro data as the output value. Implement the AvroValueWriter interface to specify the writer schema. To use Avro data as your bulk importer, mapper, or reducer's input key or value, wrap it in an AvroKey (or AvroValue for values) and implement AvroKeyReader (or AvroValueReader) to specify the reader schema.

