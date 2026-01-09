---
Title: MongoDB Advanced Aggregations With Spring Boot and Amazon Corretto
Description: 
Author: MongoDB Guests
Date: 2026-01-09T21:38:19.000Z
Robots: noindex,nofollow
Template: index
---
<p><em>This post was written by Aasawari Sahasrabuddhe.</em></p>

<p>In this tutorial, we'll get into the understanding of aggregations and explore how to construct aggregation pipelines within your Spring Boot applications.</p>

<p>This tutorial serves as a complement to the example code template accessible in the <a href="https://github.com/mongodb-developer/java-spring-boot-mongodb-starter" rel="noopener noreferrer">GitHub repository</a>. The code utilises sample data, which will be introduced later in the tutorial.</p>

<p>As indicated in the tutorial title, we'll compile the Java code using Amazon Corretto.</p>

<p>We recommend following the tutorial meticulously, progressing through each stage of the aggregation pipeline creation process.</p>

<p>Let's dive in!</p>

<h2>
  
  
  Prerequisites
</h2>

<p>This tutorial follows a few specifications, mentioned below. Before you start practicing it, please make sure you have all the necessary downloads and uploads in your environment.</p>

<ol>
<li><a href="https://aws.amazon.com/corretto/?filtered-posts.sort-by=item.additionalFields.createdDate&amp;filtered-posts.sort-order=desc" rel="noopener noreferrer">Amazon Corretto 21 JDK</a></li>
<li>
<a href="https://www.mongodb.com/cloud/atlas/register/?utm_campaign=devrel&amp;utm_source=third-party-content&amp;utm_medium=cta&amp;utm_content=aggregations-spring-boot-amazon-corretto&amp;utm_term=megan.grant" rel="noopener noreferrer">A free Atlas tier</a>, also known as an M0 cluster</li>
<li>
<a href="https://www.mongodb.com/docs/atlas/sample-data/?utm_campaign=devrel&amp;utm_source=third-party-content&amp;utm_medium=cta&amp;utm_content=aggregations-spring-boot-amazon-corretto&amp;utm_term=megan.grant#load-data-into-atlas" rel="noopener noreferrer">Sample data loaded</a> in the cluster</li>
<li>Spring Data version 4.2.2</li>
<li>MongoDB version 6.0.3</li>
<li>MongoDB Java Driver version 4.11.1 </li>
</ol>

<p>Let’s understand each of these in detail.</p>

<h2>
  
  
  Understanding and installing Corretto
</h2>

<p>Corretto comes with the ability to be a no-cost, multiplatform, production-ready open JDK. It also provides the ability to work across multiple distributions of Linux, Windows, and macOS.</p>

<p>You can read more about Amazon Corretto in <a href="https://pages.awscloud.com/Introduction-to-Amazon-Corretto-A-No-Cost-Distribution-of-OpenJDK_2019_0305-DEV_OD.html?trk=TW#:~:text=Amazon%20Corretto%20is%20a%20no,with%20the%20Java%20SE%20standard." rel="noopener noreferrer">Introduction to Amazon Corretto: A No-Cost Distribution of OpenJDK</a>.</p>

<p>We will begin the tutorial with the first step of installing the <a href="https://aws.amazon.com/corretto/?filtered-posts.sort-by=item.additionalFields.createdDate&amp;filtered-posts.sort-order=desc" rel="noopener noreferrer">Amazon Corretto 21 JDK</a> and setting up your IDE with the correct JDK.</p>

<p>Step 1: Install <a href="https://docs.aws.amazon.com/corretto/latest/corretto-21-ug/downloads-list.html" rel="noopener noreferrer">Amazon Corretto 21</a> from the official website based on the operating system specifications.</p>

<p>Step 2: If you are on macOS, you will need to set the JAVA_HOME variable with the path for the Corretto. To do this, go to the system terminal and set the variable JAVA_HOME as:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight plaintext"><code>export JAVA_HOME=/Library/Java/JavaVirtualMachines/amazon-corretto-21.jdk/Contents/Home 
</code></pre>

</div>



<p>Once the variable is set, you should check if the installation is done correctly using:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight plaintext"><code>java --version
openjdk 21.0.2 2024-01-16 LTS
OpenJDK Runtime Environment Corretto-21.0.2.13.1 (build 21.0.2+13-LTS)
OpenJDK 64-Bit Server VM Corretto-21.0.2.13.1 (build 21.0.2+13-LTS, mixed mode, sharing)
</code></pre>

</div>



<p>For any other operating system, you will need to follow the steps mentioned in the official documentation from Java on <a href="https://www.java.com/en/download/help/path.html" rel="noopener noreferrer">how to set or change the PATH system variable</a> and check if the version has been set.</p>

<p>Once the JDK is installed on the system, you can set up your IDE of choice to use Amazon Corretto to compile the code.</p>

<p>At this point, you have all the necessary environment components ready to kickstart your application.</p>

<h2>
  
  
  Creating the Spring Boot application
</h2>

<p>In this part of the tutorial, we're going to explore how to write aggregation queries for a Spring Boot application.</p>

<p>Aggregations in MongoDB are like super-powered tools for doing complex calculations on your data and getting meaningful results back. They work by applying different operations to your data and then giving you the results in a structured way.</p>

<p>But before we get into the details, let's first understand what an aggregation pipeline is and how it operates in MongoDB.</p>

<p>Think of an aggregation pipeline as a series of steps or stages that MongoDB follows to process your data. Each stage in the pipeline performs a specific task, like filtering or grouping your data in a certain way. And just like a real pipeline, data flows through each stage, with the output of one stage becoming the input for the next. This allows you to build up complex operations step by step to get the results you need.</p>

<p>By now, you should have the <a href="https://www.mongodb.com/docs/atlas/sample-data/?utm_campaign=devrel&amp;utm_source=third-party-content&amp;utm_medium=cta&amp;utm_content=aggregations-spring-boot-amazon-corretto&amp;utm_term=megan.grant" rel="noopener noreferrer">sample data loaded</a> in your MongoDB Atlas cluster. In this tutorial, we will be using the <code>sample_supplies.sales</code> collection for our aggregation queries.</p>

<p>The next step is cloning the repository from the link to test the aggregations. You can start by cloning the repository using the below command:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight plaintext"><code>git clone https://github.com/mongodb-developer/spring-boot-mongodb-aggregations.git
</code></pre>

</div>



<p>Once the above step is complete, upon forking and cloning the repository to your local environment, it's essential to update the connection string in the designated placeholder within the <code>application.properties</code> file. This modification enables seamless connectivity to your cluster during project execution.</p>

<h2>
  
  
  README
</h2>

<p>After cloning the repository and changing the URI in the environment variables, you can try running the REST APIs in your Postman application.</p>

<p>All the extra information and commands you need to get this project going are in the <a href="https://github.com/mongodb-developer/spring-boot-mongodb-aggregations/blob/master/README.md" rel="noopener noreferrer">README.md</a> file which you can read on GitHub.</p>

<h2>
  
  
  Writing aggregation queries in Spring
</h2>

<p>The Aggregation Framework support in Spring Data MongoDB is based on the following key abstractions:</p>

<ul>
<li>Aggregation</li>
<li>AggregationDefinition</li>
<li>AggregationResults</li>
</ul>

<p>While writing the aggregation queries, the first step is to generate the pipelines to perform the computations using the operations supported.</p>

<p>The documentation on <a href="https://docs.spring.io/spring-data/mongodb/reference/mongodb/aggregation-framework.html" rel="noopener noreferrer">spring.io</a> explains each step clearly and gives simple examples to help you understand.</p>

<p>For the tutorial, we have the REST APIs defined in the <a href="https://github.com/mongodb-developer/spring-boot-mongodb-aggregations/blob/master/src/main/java/com/tutorialCrud/controller/SalesController.java" rel="noopener noreferrer">SalesController.java</a> class, and the methods have been mentioned in the <a href="https://github.com/mongodb-developer/spring-boot-mongodb-aggregations/blob/master/src/main/java/com/tutorialCrud/repository/SalesRepository.java" rel="noopener noreferrer">SalesRepository.java</a> class.</p>

<p>The first aggregation makes use of a simple <a href="https://www.mongodb.com/docs/manual/reference/operator/aggregation/match/?utm_campaign=devrel&amp;utm_source=third-party-content&amp;utm_medium=cta&amp;utm_content=aggregations-spring-boot-amazon-corretto&amp;utm_term=megan.grant" rel="noopener noreferrer">$match</a> operation to find all the documents where the <code>storeLocation</code> has been specified as the match value.<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight plaintext"><code>db.sales.aggregate([{ $match: { "storeLocation": "London"}}])
</code></pre>

</div>



<p>And now, when we convert the aggregation to the Spring Boot function, it will look like this:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight plaintext"><code>@Override
public List&lt;SalesDTO&gt; matchOp(String matchValue) {
MatchOperation matchStage = match(new Criteria("storeLocation").is(matchValue));
Aggregation aggregation = newAggregation(matchStage);
AggregationResults&lt;SalesDTO&gt; results = mongoTemplate.aggregate(aggregation, "sales", SalesDTO.class);
return results.getMappedResults();
}
</code></pre>

</div>



<p>In this Spring Boot method, we utilise the <code>MatchOperation</code> to filter documents based on the specified criteria, which in this case is the <code>storeLocation</code> matching the provided value. The aggregation is then executed using the <code>mongoTemplate</code> to aggregate data from the <code>sales</code> collection into <code>SalesDTO</code> objects, returning the mapped results.</p>

<p>The REST API can be tested using the curl command in the terminal which shows all documents where <code>storeLocation</code> is <code>London</code>.</p>

<p>The next aggregation pipeline that we have defined with the rest API is to group all documents according to <code>storeLocation</code> and then calculate the total sales and the average satisfaction based on the <code>matchValue</code>. This stage makes use of the <code>GroupOperation</code> to perform the evaluation.<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight plaintext"><code>@Override
public List&lt;GroupDTO&gt; groupOp(String matchValue) {
MatchOperation matchStage = match(new Criteria("storeLocation").is(matchValue));
GroupOperation groupStage = group("storeLocation").count()
                                                          .as("totalSales")
                                                          .avg("customer.satisfaction")
                                                          .as("averageSatisfaction");
ProjectionOperation projectStage = project("storeLocation", "totalSales", "averageSatisfaction");
Aggregation aggregation = newAggregation(matchStage, groupStage, projectStage);
AggregationResults&lt;GroupDTO&gt; results = mongoTemplate.aggregate(aggregation, "sales", GroupDTO.class);
return results.getMappedResults();
}
</code></pre>

</div>



<p>The REST API call would look like the following:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight shell"><code>curl http://localhost:8080/api/sales/aggregation/groupStage/Denver | jq
</code></pre>

</div>



<p><a href="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2F9b66soluc1n22qclv785.png" class="article-body-image-wrapper"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2F9b66soluc1n22qclv785.png" alt="REST API call" width="800" height="207"></a></p>

<p>The next REST API is an extension that will streamline the above aggregation. In this case, we will be calculating the total sales for each store location. Therefore, you do not need to specify the store location and directly get the value for all the locations.<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight plaintext"><code>@Override
public List&lt;TotalSalesDTO&gt; TotalSales() {
GroupOperation groupStage = group("storeLocation").count().as("totalSales");
SkipOperation skipStage = skip(0);
LimitOperation limitStage = limit(10);
Aggregation aggregation = newAggregation(groupStage, skipStage, limitStage);
AggregationResults&lt;TotalSalesDTO&gt; results = mongoTemplate.aggregate(aggregation, "sales", TotalSalesDTO.class);
return results.getMappedResults();
}
</code></pre>

</div>



<p>And the REST API calls look like the following:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight shell"><code>curl http://localhost:8080/api/sales/aggregation/TotalSales | jq
</code></pre>

</div>



<p><a href="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fo08l77ityoooc0dq4ilm.png" class="article-body-image-wrapper"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fo08l77ityoooc0dq4ilm.png" alt=" " width="800" height="512"></a></p>

<p>The next API makes use of <a href="https://www.mongodb.com/docs/manual/reference/operator/aggregation/sort/?utm_campaign=devrel&amp;utm_source=third-party-content&amp;utm_medium=cta&amp;utm_content=aggregations-spring-boot-amazon-corretto&amp;utm_term=megan.grant" rel="noopener noreferrer">$sort</a> and <a href="https://www.mongodb.com/docs/manual/reference/operator/aggregation/limit/?utm_campaign=devrel&amp;utm_source=third-party-content&amp;utm_medium=cta&amp;utm_content=aggregations-spring-boot-amazon-corretto&amp;utm_term=megan.grant" rel="noopener noreferrer">$limit</a> operations to calculate the top five items sold in each category.<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight plaintext"><code>@Override
public List&lt;PopularDTO&gt; findPopularItems() {
UnwindOperation unwindStage = unwind("items");
GroupOperation groupStage = group("$items.name").sum("items.quantity").as("totalQuantity");
SortOperation sortStage = sort(Sort.Direction.DESC, "totalQuantity");
LimitOperation limitStage = limit(5);
Aggregation aggregation = newAggregation(unwindStage,groupStage, sortStage, limitStage);
return mongoTemplate.aggregate(aggregation, "sales", PopularDTO.class).getMappedResults();
}
</code></pre>

</div>





<div class="highlight js-code-highlight">
<pre class="highlight shell"><code>curl http://localhost:8080/api/sales/aggregation/PopularItem | jq
</code></pre>

</div>



<p><a href="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fifk8kg31i8non7ap7z4w.png" class="article-body-image-wrapper"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fifk8kg31i8non7ap7z4w.png" alt="Top 5 items sold in every category" width="800" height="406"></a></p>

<p>The last API mentioned makes use of the <a href="https://www.mongodb.com/docs/manual/reference/operator/aggregation/bucket/" rel="noopener noreferrer">$bucket</a> to create buckets and then calculates the count and total amount spent within each bucket.<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight plaintext"><code>@Override
public List&lt;BucketsDTO&gt; findTotalSpend(){
ProjectionOperation projectStage = project()
                .and(ArrayOperators.Size.lengthOfArray("items")).as("numItems")
                .and(ArithmeticOperators.Multiply.valueOf("price")
                        .multiplyBy("quantity")).as("totalAmount");

BucketOperation bucketStage = bucket("numItems")
                .withBoundaries(0, 3, 6, 9)
                .withDefaultBucket("Other")
                .andOutputCount().as("count")
                .andOutput("totalAmount").sum().as("totalAmount");

Aggregation aggregation = newAggregation(projectStage, bucketStage);
return mongoTemplate.aggregate(aggregation, "sales", BucketsDTO.class).getMappedResults();
}
</code></pre>

</div>





<div class="highlight js-code-highlight">
<pre class="highlight shell"><code>curl http://localhost:8080/api/sales/aggregation/buckets | jq
</code></pre>

</div>



<p><a href="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2F8x4nbbx16fq069fjdg6u.png" class="article-body-image-wrapper"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2F8x4nbbx16fq069fjdg6u.png" alt="calculates the count and total amount spent within each bucket" width="800" height="608"></a></p>

<h2>
  
  
  Conclusion
</h2>

<p>This tutorial provides a comprehensive overview of aggregations in MongoDB and how to implement them in a Spring Boot application. We have learned about the significance of aggregation queries for performing complex calculations on data sets, leveraging MongoDB's aggregation pipeline to streamline this process effectively.</p>

<p>As you continue to experiment and apply these concepts in your applications, feel free to reach out on our <a href="https://www.mongodb.com/community/forums/?utm_campaign=devrel&amp;utm_source=third-party-content&amp;utm_medium=cta&amp;utm_content=aggregations-spring-boot-amazon-corretto&amp;utm_term=megan.grant" rel="noopener noreferrer">MongoDB community forums</a>. Remember to explore further resources in our <a href="https://www.mongodb.com/docs/?utm_campaign=devrel&amp;utm_source=third-party-content&amp;utm_medium=cta&amp;utm_content=aggregations-spring-boot-amazon-corretto&amp;utm_term=megan.grant" rel="noopener noreferrer">documentation</a> to deepen your understanding and refine your skills in working with MongoDB aggregations.</p>

