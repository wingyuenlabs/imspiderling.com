---
Title: BSON to JSON: Efficient Data Conversion with Java
Description: 
Author: Mario García
Date: 2026-02-02T21:36:53.000Z
Robots: noindex,nofollow
Template: index
---
<p>Through this blog post, you will learn how to convert a BSON document to JSON using Java.</p>

<h2>
  
  
  BSON to JSON with Java
</h2>

<p>If you’re a Java developer, there are two ways to read BSON documents and convert them to JSON.</p>

<ul>
<li><p>Using the MongoDB Java Driver to query an active database</p></li>
<li><p>Reading and parsing local .bson files at the byte level</p></li>
</ul>

<p>Let's create a sample project.<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight plaintext"><code>mvn archetype:generate \
  -DgroupId=com.your-domain \
  -DartifactId=bson-to-json \
  -DarchetypeArtifactId=maven-archetype-quickstart \
  -DinteractiveMode=false
</code></pre>

</div>



<p>Now, add dependencies and configure the project by editing the <code>pom.xml</code> file as follows:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight plaintext"><code>&lt;project xmlns="http://maven.apache.org/POM/4.0.0" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
  xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/maven-v4_0_0.xsd"&gt;
  &lt;modelVersion&gt;4.0.0&lt;/modelVersion&gt;

  &lt;groupId&gt;com.your-domain&lt;/groupId&gt;
  &lt;artifactId&gt;bson-to-json&lt;/artifactId&gt;
  &lt;packaging&gt;jar&lt;/packaging&gt;
  &lt;version&gt;1.0-SNAPSHOT&lt;/version&gt;
  &lt;name&gt;bson-to-json&lt;/name&gt;

  &lt;properties&gt;
    &lt;maven.compiler.release&gt;11&lt;/maven.compiler.release&gt;
    &lt;project.build.sourceEncoding&gt;UTF-8&lt;/project.build.sourceEncoding&gt;
  &lt;/properties&gt;

  &lt;dependencies&gt;
    &lt;dependency&gt;
      &lt;groupId&gt;org.mongodb&lt;/groupId&gt;
      &lt;artifactId&gt;mongodb-driver-sync&lt;/artifactId&gt;
      &lt;version&gt;5.3.1&lt;/version&gt;
    &lt;/dependency&gt;

    &lt;dependency&gt;
      &lt;groupId&gt;org.slf4j&lt;/groupId&gt;
      &lt;artifactId&gt;slf4j-nop&lt;/artifactId&gt;
      &lt;version&gt;2.0.9&lt;/version&gt;
    &lt;/dependency&gt;

    &lt;dependency&gt;
      &lt;groupId&gt;junit&lt;/groupId&gt;
      &lt;artifactId&gt;junit&lt;/artifactId&gt;
      &lt;version&gt;4.13.2&lt;/version&gt; &lt;scope&gt;test&lt;/scope&gt;
    &lt;/dependency&gt;
  &lt;/dependencies&gt;

  &lt;build&gt;
    &lt;plugins&gt;
      &lt;plugin&gt;
        &lt;groupId&gt;org.codehaus.mojo&lt;/groupId&gt;
        &lt;artifactId&gt;exec-maven-plugin&lt;/artifactId&gt;
        &lt;version&gt;3.1.0&lt;/version&gt;
        &lt;configuration&gt;
          &lt;mainClass&gt;com.your-domain.App&lt;/mainClass&gt;
          &lt;cleanupDaemonThreads&gt;false&lt;/cleanupDaemonThreads&gt;
        &lt;/configuration&gt;
      &lt;/plugin&gt;
    &lt;/plugins&gt;
  &lt;/build&gt;
&lt;/project&gt;
</code></pre>

</div>



<ul>
<li>
<p>Lock the project to a specific Java version (in this case, Java 11) to ensure consistent compilation across different environments<br>
</p>
<pre class="highlight plaintext"><code>  &lt;properties&gt;
    &lt;maven.compiler.release&gt;11&lt;/maven.compiler.release&gt;
    &lt;project.build.sourceEncoding&gt;UTF-8&lt;/project.build.sourceEncoding&gt;
  &lt;/properties&gt;
</code></pre>

</li>
<li>
<p>Add the MongoDB driver as dependency<br>
</p>
<pre class="highlight plaintext"><code>    &lt;dependency&gt;
      &lt;groupId&gt;org.mongodb&lt;/groupId&gt;
      &lt;artifactId&gt;mongodb-driver-sync&lt;/artifactId&gt;
      &lt;version&gt;5.3.1&lt;/version&gt;
    &lt;/dependency&gt;
</code></pre>

</li>
<li>
<p>Silence internal MongoDB driver logs using SLF4J-NOP<br>
</p>
<pre class="highlight plaintext"><code>    &lt;dependency&gt;
      &lt;groupId&gt;org.slf4j&lt;/groupId&gt;
      &lt;artifactId&gt;slf4j-nop&lt;/artifactId&gt;
      &lt;version&gt;2.0.9&lt;/version&gt;
    &lt;/dependency&gt;
</code></pre>

</li>
<li>
<p>Automate dependency linking and silence MongoDB's background thread warnings by using the <code>exec-maven-plugin</code><br>
</p>
<pre class="highlight plaintext"><code>  &lt;build&gt;
    &lt;plugins&gt;
      &lt;plugin&gt;
        &lt;groupId&gt;org.codehaus.mojo&lt;/groupId&gt;
        &lt;artifactId&gt;exec-maven-plugin&lt;/artifactId&gt;
        &lt;version&gt;3.1.0&lt;/version&gt;
        &lt;configuration&gt;
        &lt;mainClass&gt;com.your-domain.App&lt;/mainClass&gt;
        &lt;cleanupDaemonThreads&gt;false&lt;/cleanupDaemonThreads&gt;
        &lt;/configuration&gt;
      &lt;/plugin&gt;
    &lt;/plugins&gt;
  &lt;/build&gt;
</code></pre>

</li>
</ul>

<h3>
  
  
  Using the MongoDB Java Driver to query an active database
</h3>

<p>Now, edit the <code>App.java</code> file stored in the <code>src/main/java/com/your-domain/</code>.<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight plaintext"><code>package com.your-domain;

import com.mongodb.client.MongoClient;
import com.mongodb.client.MongoClients;
import com.mongodb.client.MongoCollection;
import com.mongodb.client.MongoDatabase;
import org.bson.Document;
import org.bson.json.JsonWriterSettings;

import java.io.FileWriter;
import java.io.IOException;

public class App {
    public static void main(String[] args) {
        String uri = "mongodb://user:password@localhost:27017/?authSource=admin";

        try (MongoClient mongoClient = MongoClients.create(uri)) {
            MongoDatabase database = mongoClient.getDatabase("database");
            MongoCollection&lt;Document&gt; collection = database.getCollection("collection");

            JsonWriterSettings settings = JsonWriterSettings.builder().indent(true).build();

            try (FileWriter file = new FileWriter("collection.json")) {
                file.write("[\n");

                for (Document doc : collection.find()) {
                    file.write(doc.toJson(settings) + ",\n");
                }

                file.write("]");
                System.out.println("Exported successfully!");
            } catch (IOException e) {
                e.printStackTrace();
            }
        }
    }
}
</code></pre>

</div>



<ul>
<li>
<p>Declares the package that the Java class belongs to<br>
</p>
<pre class="highlight plaintext"><code>package com.your-domain;
</code></pre>

</li>
<li>
<p>Establish connection to the MongoDB instance<br>
</p>
<pre class="highlight plaintext"><code>        try (MongoClient mongoClient = MongoClients.create(uri))
</code></pre>


<p>Where <code>uri</code> is the connection string.</p>
</li>
<li>
<p>Target the specific database and collection from which the data will be exported<br>
</p>
<pre class="highlight plaintext"><code>            MongoDatabase database = mongoClient.getDatabase("company");
            MongoCollection&lt;Document&gt; collection = database.getCollection("employees");
</code></pre>

</li>
<li>
<p>Pretty format the content of the JSON file<br>
</p>
<pre class="highlight plaintext"><code>            JsonWriterSettings settings = JsonWriterSettings.builder().indent(true).build();
</code></pre>

</li>
<li>
<p>Manually construct and write a JSON array by wrapping the converted documents in brackets<br>
</p>
<pre class="highlight plaintext"><code>                file.write("[\n");
                for (Document doc : collection.find()) {
                    file.write(doc.toJson(settings) + ",\n");
                }
                file.write("]");
</code></pre>

</li>
</ul>

<h3>
  
  
  Reading and parsing local .bson files at the byte level
</h3>

<p>Now, edit the <code>App.java</code> file stored in the <code>src/main/java/com/your-domain/</code>.<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight plaintext"><code>package com.your-domain;

import org.bson.Document;
import org.bson.BsonType;
import org.bson.codecs.DocumentCodec;
import org.bson.codecs.DecoderContext;
import org.bson.BsonBinaryReader;
import org.bson.io.ByteBufferBsonInput;
import org.bson.ByteBufNIO;
import org.bson.json.JsonWriterSettings;

import java.io.FileInputStream;
import java.io.FileWriter;
import java.nio.ByteBuffer;
import java.nio.channels.FileChannel;

public class App {
    public static void main(String[] args) throws Exception {
        String inputPath = "collection.bson";
        String outputPath = "collection.json";
        JsonWriterSettings settings = JsonWriterSettings.builder().indent(true).build();
        DocumentCodec codec = new DocumentCodec();

        try (FileInputStream fis = new FileInputStream(inputPath);
             FileChannel channel = fis.getChannel();
             FileWriter writer = new FileWriter(outputPath)) {

            ByteBuffer buffer = ByteBuffer.allocate((int) channel.size());
            channel.read(buffer);
            buffer.flip();

            try (ByteBufferBsonInput bsonInput = new ByteBufferBsonInput(new ByteBufNIO(buffer));
                 BsonBinaryReader reader = new BsonBinaryReader(bsonInput)) {

                writer.write("[\n");
                boolean first = true;

                while (buffer.hasRemaining()) {

                    if (reader.readBsonType() == BsonType.END_OF_DOCUMENT) {
                        break;
                    }

                    if (!first) {
                        writer.write(",\n");
                    }

                    Document doc = codec.decode(reader, DecoderContext.builder().build());
                    writer.write(doc.toJson(settings));
                    first = false;
                }

                writer.write("\n]");
                System.out.println("Conversion completed successfully: " + outputPath);
            }
        }
    }
}
</code></pre>

</div>



<ul>
<li>
<p>Initialize the file paths, configures the JSON output with indentation for readability and sets up the engine for translating binary data into Java objects<br>
</p>
<pre class="highlight plaintext"><code>    String inputPath = "collection.bson";
    String outputPath = "collection.json";
    JsonWriterSettings settings = JsonWriterSettings.builder().indent(true).build();
    DocumentCodec codec = new DocumentCodec();
</code></pre>

</li>
<li>
<p>Allocates a memory buffer the exact size of the file and uses a <code>FileChannel</code> to load the binary data for processing<br>
</p>
<pre class="highlight plaintext"><code>            ByteBuffer buffer = ByteBuffer.allocate((int) channel.size());
            channel.read(buffer);
            buffer.flip();
</code></pre>

</li>
<li><p>Wraps the memory buffer into a <code>ByteBufferBsonInput</code> and creates a <code>BsonBinaryReader</code> to navigate the binary structure<br>
</p></li>
</ul>

<div class="highlight js-code-highlight">
<pre class="highlight plaintext"><code>                try (ByteBufferBsonInput bsonInput = new ByteBufferBsonInput(new ByteBufNIO(buffer));
                  BsonBinaryReader reader = new BsonBinaryReader(bsonInput)) {
</code></pre>

</div>



<ul>
<li>Writes the opening bracket and uses a loop to iterate through the buffer until no documents remain
</li>
</ul>

<div class="highlight js-code-highlight">
<pre class="highlight plaintext"><code>                  writer.write("[\n");
                  boolean first = true;

                  while (buffer.hasRemaining()) {
                      if (reader.readBsonType() == BsonType.END_OF_DOCUMENT) {
                          break;
                      }
                      // ... loop logic
                  }
</code></pre>

</div>



<ul>
<li>Translates each binary BSON segment into a Java Document and writes it to the file as a formatted JSON string
</li>
</ul>

<div class="highlight js-code-highlight">
<pre class="highlight plaintext"><code>                  Document doc = codec.decode(reader, DecoderContext.builder().build());
                  writer.write(doc.toJson(settings));
</code></pre>

</div>



<p>The BSON file must be in the root directory of your Maven project. Now, you can run the above code by executing the following command:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight plaintext"><code>mvn clean compile exec:java
</code></pre>

</div>



<p>It will do a clean build, delete the <code>target</code> folder, compile the Java code, and run the project.</p>

<p>After running, you will find the corresponding JSON file in the root directory of your project</p>

<h2>
  
  
  Conclusion
</h2>

<p>If you’re a developer, you can use the MongoDB Java driver to query and analyze database collections before exporting them to JSON, or directly parse local BSON files for a quick format conversion.</p>

