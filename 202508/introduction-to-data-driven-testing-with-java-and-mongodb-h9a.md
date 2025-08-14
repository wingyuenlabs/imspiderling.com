---
Title: Introduction to Data-Driven Testing with Java and MongoDB
Description: 
Author: MongoDB Guests
Date: 2025-08-14T20:59:34.000Z
Robots: noindex,nofollow
Template: index
---
<p><strong>This article was written by <a href="https://www.linkedin.com/in/otaviojava/" rel="noopener noreferrer">Otavio Santana</a>, a renowned contributor to the Java and open-source ecosystems.</strong></p>

<p>As applications expand, the complexity of the rules they enforce also increases. In many systems, these rules are embedded within the data, primarily in database queries that filter, join, or compute based on real-world conditions. However, the tests for these queries are often shallow, repetitive, or, worse yet, completely absent. When there is an error in the database logic, the application may still compile successfully, but the business can suffer significant consequences.</p>

<p><strong>Data-driven testing</strong> (DDT) is important because it enables you to validate business behavior across various scenarios without having to duplicate test logic. When used in conjunction with a document database like MongoDB, it becomes a powerful tool to ensure that your queries align not only with the data structure but also with the intended business outcomes. For Java applications where persistence logic goes beyond basic CRUD operations, data-driven testing helps you safeguard what matters most: correctness, clarity, and confidence.</p>

<p>In this tutorial, you’ll:</p>

<ul>
<li>Model a domain (Room, RoomType, RoomStatus).</li>
<li>Write semantic repository queries using Jakarta Data.</li>
<li>Run data-driven tests using JUnit 5 and AssertJ.</li>
<li>Validate MongoDB queries in isolation using Testcontainers and Weld.</li>
</ul>

<p>You can find all the code presented in this tutorial in the <a href="https://github.com/soujava/data-driven-test-mongodb" rel="noopener noreferrer">GitHub repository</a>:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight shell"><code>git clone git@github.com:soujava/data-driven-test-mongodb.git
</code></pre>

</div>



<h2>
  
  
  Prerequisites
</h2>

<p>For this tutorial, you’ll need:</p>

<ul>
<li>Java 21.</li>
<li>Maven.</li>
<li>A MongoDB cluster.

<ul>
<li>
<a href="https://www.mongodb.com/cloud/atlas/register?utm_campaign=devrel&amp;utm_source=third-party-content&amp;utm_medium=cta&amp;utm_content=data_driven_test_dev&amp;utm_term=otavio.santana" rel="noopener noreferrer">MongoDB Atlas</a> (Option 1)</li>
<li>Docker (Option 2)</li>
</ul>


</li>

</ul>

<p>You can use the following Docker command to start a standalone MongoDB instance:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight shell"><code>docker run <span class="nt">--rm</span> <span class="nt">-d</span> <span class="nt">--name</span> mongodb-instance <span class="nt">-p</span> 27017:27017 mongo
</code></pre>

</div>



<p>Data-driven testing is a technique where a single test is executed with different input data sets to validate multiple scenarios. Instead of writing repetitive test methods, you define combinations of inputs and expected outcomes, making it easier to test edge cases, business rules, and regression scenarios with clarity and efficiency.</p>

<p>In this age, data is the heart of any software system. Ensuring consistency and validating the code based on several data conditions will increase your quality. In this process, we can use a database to inject the data as input, to check the queries themselves, and also store the results as a report.</p>

<p>This approach aligns naturally with MongoDB, where application logic frequently relies on query conditions—particularly when filtering based on domain-specific rules. In a typical Java application, these queries are embedded in repositories or services and are rarely tested beyond simple "find by ID" checks. DDT allows you to test the <em>intent</em> behind queries, such as a hotel management system: Is a room considered available? Does it match VIP criteria? Should it be cleaned?</p>

<p>In this tutorial, we’ll use a Java SE project—without any heavyweight frameworks—to demonstrate how to combine Jakarta Data, JNoSQL, and JUnit 5 to write expressive, testable queries against MongoDB. Our focus will be on clarity, maintainability, and aligning tests with the business language, not just with database fields.</p>

<h2>
  
  
  Step 1: Create the entities
</h2>

<p>The first step is to create a plain Maven project, where we can use Maven Archetype<a href="https://maven.apache.org/archetypes/maven-archetype-quickstart/" rel="noopener noreferrer"> quick start</a>. After making the Maven project, the next step is to include Jupiter, Mockito, AssertJ, and Testcontainers for test proposals. For the Java integration and MongoDB, we will explore it using the Java Enterprise specification, Jakarta EE, where we will utilize both specifications, Jakarta NoSQL and Jakarta Data, both of which are implemented by Eclipse JNoSQL. We don't need to spend a considerable amount of time setting it up; you can clone the GitHub repository. The pom.xml shows the dependencies using Java with the Apache Maven Project:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight java"><code><span class="o">&lt;</span><span class="n">dependencies</span><span class="o">&gt;</span>
        <span class="o">&lt;</span><span class="n">dependency</span><span class="o">&gt;</span>
            <span class="o">&lt;</span><span class="n">groupId</span><span class="o">&gt;</span><span class="n">net</span><span class="o">.</span><span class="na">datafaker</span><span class="o">&lt;/</span><span class="n">groupId</span><span class="o">&gt;</span>
            <span class="o">&lt;</span><span class="n">artifactId</span><span class="o">&gt;</span><span class="n">datafaker</span><span class="o">&lt;/</span><span class="n">artifactId</span><span class="o">&gt;</span>
            <span class="o">&lt;</span><span class="n">version</span><span class="o">&gt;</span><span class="mf">2.4</span><span class="o">.</span><span class="mi">4</span><span class="o">&lt;/</span><span class="n">version</span><span class="o">&gt;</span>
        <span class="o">&lt;/</span><span class="n">dependency</span><span class="o">&gt;</span>
        <span class="o">&lt;</span><span class="n">dependency</span><span class="o">&gt;</span>
            <span class="o">&lt;</span><span class="n">groupId</span><span class="o">&gt;</span><span class="n">org</span><span class="o">.</span><span class="na">eclipse</span><span class="o">.</span><span class="na">jnosql</span><span class="o">.</span><span class="na">databases</span><span class="o">&lt;/</span><span class="n">groupId</span><span class="o">&gt;</span>
            <span class="o">&lt;</span><span class="n">artifactId</span><span class="o">&gt;</span><span class="n">jnosql</span><span class="o">-</span><span class="n">mongodb</span><span class="o">&lt;/</span><span class="n">artifactId</span><span class="o">&gt;</span>
            <span class="o">&lt;</span><span class="n">version</span><span class="o">&gt;</span><span class="err">$</span><span class="o">{</span><span class="n">jnosql</span><span class="o">.</span><span class="na">version</span><span class="o">}&lt;/</span><span class="n">version</span><span class="o">&gt;</span>
        <span class="o">&lt;/</span><span class="n">dependency</span><span class="o">&gt;</span>
        <span class="o">&lt;</span><span class="n">dependency</span><span class="o">&gt;</span>
            <span class="o">&lt;</span><span class="n">groupId</span><span class="o">&gt;</span><span class="n">org</span><span class="o">.</span><span class="na">jboss</span><span class="o">.</span><span class="na">weld</span><span class="o">.</span><span class="na">se</span><span class="o">&lt;/</span><span class="n">groupId</span><span class="o">&gt;</span>
            <span class="o">&lt;</span><span class="n">artifactId</span><span class="o">&gt;</span><span class="n">weld</span><span class="o">-</span><span class="n">se</span><span class="o">-</span><span class="n">shaded</span><span class="o">&lt;/</span><span class="n">artifactId</span><span class="o">&gt;</span>
            <span class="o">&lt;</span><span class="n">version</span><span class="o">&gt;</span><span class="err">$</span><span class="o">{</span><span class="n">weld</span><span class="o">.</span><span class="na">se</span><span class="o">.</span><span class="na">core</span><span class="o">.</span><span class="na">version</span><span class="o">}&lt;/</span><span class="n">version</span><span class="o">&gt;</span>
            <span class="o">&lt;</span><span class="n">scope</span><span class="o">&gt;</span><span class="n">compile</span><span class="o">&lt;/</span><span class="n">scope</span><span class="o">&gt;</span>
        <span class="o">&lt;/</span><span class="n">dependency</span><span class="o">&gt;</span>
        <span class="o">&lt;</span><span class="n">dependency</span><span class="o">&gt;</span>
            <span class="o">&lt;</span><span class="n">groupId</span><span class="o">&gt;</span><span class="n">org</span><span class="o">.</span><span class="na">eclipse</span><span class="o">&lt;/</span><span class="n">groupId</span><span class="o">&gt;</span>
            <span class="o">&lt;</span><span class="n">artifactId</span><span class="o">&gt;</span><span class="n">yasson</span><span class="o">&lt;/</span><span class="n">artifactId</span><span class="o">&gt;</span>
            <span class="o">&lt;</span><span class="n">version</span><span class="o">&gt;</span><span class="mf">3.0</span><span class="o">.</span><span class="mi">4</span><span class="o">&lt;/</span><span class="n">version</span><span class="o">&gt;</span>
            <span class="o">&lt;</span><span class="n">scope</span><span class="o">&gt;</span><span class="n">compile</span><span class="o">&lt;/</span><span class="n">scope</span><span class="o">&gt;</span>
        <span class="o">&lt;/</span><span class="n">dependency</span><span class="o">&gt;</span>
        <span class="o">&lt;</span><span class="n">dependency</span><span class="o">&gt;</span>
            <span class="o">&lt;</span><span class="n">groupId</span><span class="o">&gt;</span><span class="n">io</span><span class="o">.</span><span class="na">smallrye</span><span class="o">.</span><span class="na">config</span><span class="o">&lt;/</span><span class="n">groupId</span><span class="o">&gt;</span>
            <span class="o">&lt;</span><span class="n">artifactId</span><span class="o">&gt;</span><span class="n">smallrye</span><span class="o">-</span><span class="n">config</span><span class="o">-</span><span class="n">core</span><span class="o">&lt;/</span><span class="n">artifactId</span><span class="o">&gt;</span>
            <span class="o">&lt;</span><span class="n">version</span><span class="o">&gt;</span><span class="mf">3.13</span><span class="o">.</span><span class="mi">3</span><span class="o">&lt;/</span><span class="n">version</span><span class="o">&gt;</span>
            <span class="o">&lt;</span><span class="n">scope</span><span class="o">&gt;</span><span class="n">compile</span><span class="o">&lt;/</span><span class="n">scope</span><span class="o">&gt;</span>
        <span class="o">&lt;/</span><span class="n">dependency</span><span class="o">&gt;</span>
        <span class="o">&lt;</span><span class="n">dependency</span><span class="o">&gt;</span>
            <span class="o">&lt;</span><span class="n">groupId</span><span class="o">&gt;</span><span class="n">org</span><span class="o">.</span><span class="na">eclipse</span><span class="o">.</span><span class="na">microprofile</span><span class="o">.</span><span class="na">config</span><span class="o">&lt;/</span><span class="n">groupId</span><span class="o">&gt;</span>
            <span class="o">&lt;</span><span class="n">artifactId</span><span class="o">&gt;</span><span class="n">microprofile</span><span class="o">-</span><span class="n">config</span><span class="o">-</span><span class="n">api</span><span class="o">&lt;/</span><span class="n">artifactId</span><span class="o">&gt;</span>
            <span class="o">&lt;</span><span class="n">version</span><span class="o">&gt;</span><span class="mf">3.1</span><span class="o">&lt;/</span><span class="n">version</span><span class="o">&gt;</span>
            <span class="o">&lt;</span><span class="n">scope</span><span class="o">&gt;</span><span class="n">compile</span><span class="o">&lt;/</span><span class="n">scope</span><span class="o">&gt;</span>
        <span class="o">&lt;/</span><span class="n">dependency</span><span class="o">&gt;</span>
        <span class="o">&lt;</span><span class="n">dependency</span><span class="o">&gt;</span>
            <span class="o">&lt;</span><span class="n">groupId</span><span class="o">&gt;</span><span class="n">org</span><span class="o">.</span><span class="na">junit</span><span class="o">.</span><span class="na">jupiter</span><span class="o">&lt;/</span><span class="n">groupId</span><span class="o">&gt;</span>
            <span class="o">&lt;</span><span class="n">artifactId</span><span class="o">&gt;</span><span class="n">junit</span><span class="o">-</span><span class="n">jupiter</span><span class="o">-</span><span class="n">api</span><span class="o">&lt;/</span><span class="n">artifactId</span><span class="o">&gt;</span>
            <span class="o">&lt;</span><span class="n">version</span><span class="o">&gt;</span><span class="err">$</span><span class="o">{</span><span class="n">junit</span><span class="o">.</span><span class="na">version</span><span class="o">}&lt;/</span><span class="n">version</span><span class="o">&gt;</span>
            <span class="o">&lt;</span><span class="n">scope</span><span class="o">&gt;</span><span class="n">test</span><span class="o">&lt;/</span><span class="n">scope</span><span class="o">&gt;</span>
        <span class="o">&lt;/</span><span class="n">dependency</span><span class="o">&gt;</span>
        <span class="o">&lt;</span><span class="n">dependency</span><span class="o">&gt;</span>
            <span class="o">&lt;</span><span class="n">groupId</span><span class="o">&gt;</span><span class="n">org</span><span class="o">.</span><span class="na">junit</span><span class="o">.</span><span class="na">jupiter</span><span class="o">&lt;/</span><span class="n">groupId</span><span class="o">&gt;</span>
            <span class="o">&lt;</span><span class="n">artifactId</span><span class="o">&gt;</span><span class="n">junit</span><span class="o">-</span><span class="n">jupiter</span><span class="o">-</span><span class="n">engine</span><span class="o">&lt;/</span><span class="n">artifactId</span><span class="o">&gt;</span>
            <span class="o">&lt;</span><span class="n">version</span><span class="o">&gt;</span><span class="err">$</span><span class="o">{</span><span class="n">junit</span><span class="o">.</span><span class="na">version</span><span class="o">}&lt;/</span><span class="n">version</span><span class="o">&gt;</span>
            <span class="o">&lt;</span><span class="n">scope</span><span class="o">&gt;</span><span class="n">test</span><span class="o">&lt;/</span><span class="n">scope</span><span class="o">&gt;</span>
        <span class="o">&lt;/</span><span class="n">dependency</span><span class="o">&gt;</span>
        <span class="o">&lt;</span><span class="n">dependency</span><span class="o">&gt;</span>
            <span class="o">&lt;</span><span class="n">groupId</span><span class="o">&gt;</span><span class="n">org</span><span class="o">.</span><span class="na">junit</span><span class="o">.</span><span class="na">jupiter</span><span class="o">&lt;/</span><span class="n">groupId</span><span class="o">&gt;</span>
            <span class="o">&lt;</span><span class="n">artifactId</span><span class="o">&gt;</span><span class="n">junit</span><span class="o">-</span><span class="n">jupiter</span><span class="o">-</span><span class="n">params</span><span class="o">&lt;/</span><span class="n">artifactId</span><span class="o">&gt;</span>
            <span class="o">&lt;</span><span class="n">version</span><span class="o">&gt;</span><span class="err">$</span><span class="o">{</span><span class="n">junit</span><span class="o">.</span><span class="na">version</span><span class="o">}&lt;/</span><span class="n">version</span><span class="o">&gt;</span>
            <span class="o">&lt;</span><span class="n">scope</span><span class="o">&gt;</span><span class="n">test</span><span class="o">&lt;/</span><span class="n">scope</span><span class="o">&gt;</span>
        <span class="o">&lt;/</span><span class="n">dependency</span><span class="o">&gt;</span>
        <span class="o">&lt;</span><span class="n">dependency</span><span class="o">&gt;</span>
            <span class="o">&lt;</span><span class="n">groupId</span><span class="o">&gt;</span><span class="n">org</span><span class="o">.</span><span class="na">mockito</span><span class="o">&lt;/</span><span class="n">groupId</span><span class="o">&gt;</span>
            <span class="o">&lt;</span><span class="n">artifactId</span><span class="o">&gt;</span><span class="n">mockito</span><span class="o">-</span><span class="n">core</span><span class="o">&lt;/</span><span class="n">artifactId</span><span class="o">&gt;</span>
            <span class="o">&lt;</span><span class="n">version</span><span class="o">&gt;</span><span class="err">$</span><span class="o">{</span><span class="n">mockito</span><span class="o">.</span><span class="na">verson</span><span class="o">}&lt;/</span><span class="n">version</span><span class="o">&gt;</span>
            <span class="o">&lt;</span><span class="n">scope</span><span class="o">&gt;</span><span class="n">test</span><span class="o">&lt;/</span><span class="n">scope</span><span class="o">&gt;</span>
        <span class="o">&lt;/</span><span class="n">dependency</span><span class="o">&gt;</span>
        <span class="o">&lt;</span><span class="n">dependency</span><span class="o">&gt;</span>
            <span class="o">&lt;</span><span class="n">groupId</span><span class="o">&gt;</span><span class="n">org</span><span class="o">.</span><span class="na">mockito</span><span class="o">&lt;/</span><span class="n">groupId</span><span class="o">&gt;</span>
            <span class="o">&lt;</span><span class="n">artifactId</span><span class="o">&gt;</span><span class="n">mockito</span><span class="o">-</span><span class="n">junit</span><span class="o">-</span><span class="n">jupiter</span><span class="o">&lt;/</span><span class="n">artifactId</span><span class="o">&gt;</span>
            <span class="o">&lt;</span><span class="n">version</span><span class="o">&gt;</span><span class="err">$</span><span class="o">{</span><span class="n">mockito</span><span class="o">.</span><span class="na">verson</span><span class="o">}&lt;/</span><span class="n">version</span><span class="o">&gt;</span>
            <span class="o">&lt;</span><span class="n">scope</span><span class="o">&gt;</span><span class="n">test</span><span class="o">&lt;/</span><span class="n">scope</span><span class="o">&gt;</span>
        <span class="o">&lt;/</span><span class="n">dependency</span><span class="o">&gt;</span>
        <span class="o">&lt;</span><span class="n">dependency</span><span class="o">&gt;</span>
            <span class="o">&lt;</span><span class="n">groupId</span><span class="o">&gt;</span><span class="n">org</span><span class="o">.</span><span class="na">jboss</span><span class="o">.</span><span class="na">weld</span><span class="o">&lt;/</span><span class="n">groupId</span><span class="o">&gt;</span>
            <span class="o">&lt;</span><span class="n">artifactId</span><span class="o">&gt;</span><span class="n">weld</span><span class="o">-</span><span class="n">junit5</span><span class="o">&lt;/</span><span class="n">artifactId</span><span class="o">&gt;</span>
            <span class="o">&lt;</span><span class="n">version</span><span class="o">&gt;</span><span class="mf">5.0</span><span class="o">.</span><span class="mi">1</span><span class="o">.</span><span class="na">Final</span><span class="o">&lt;/</span><span class="n">version</span><span class="o">&gt;</span>
            <span class="o">&lt;</span><span class="n">scope</span><span class="o">&gt;</span><span class="n">test</span><span class="o">&lt;/</span><span class="n">scope</span><span class="o">&gt;</span>
        <span class="o">&lt;/</span><span class="n">dependency</span><span class="o">&gt;</span>
        <span class="o">&lt;</span><span class="n">dependency</span><span class="o">&gt;</span>
            <span class="o">&lt;</span><span class="n">groupId</span><span class="o">&gt;</span><span class="n">org</span><span class="o">.</span><span class="na">testcontainers</span><span class="o">&lt;/</span><span class="n">groupId</span><span class="o">&gt;</span>
            <span class="o">&lt;</span><span class="n">artifactId</span><span class="o">&gt;</span><span class="n">mongodb</span><span class="o">&lt;/</span><span class="n">artifactId</span><span class="o">&gt;</span>
            <span class="o">&lt;</span><span class="n">version</span><span class="o">&gt;</span><span class="mf">1.21</span><span class="o">.</span><span class="mi">3</span><span class="o">&lt;/</span><span class="n">version</span><span class="o">&gt;</span>
            <span class="o">&lt;</span><span class="n">scope</span><span class="o">&gt;</span><span class="n">test</span><span class="o">&lt;/</span><span class="n">scope</span><span class="o">&gt;</span>
        <span class="o">&lt;/</span><span class="n">dependency</span><span class="o">&gt;</span>
        <span class="o">&lt;</span><span class="n">dependency</span><span class="o">&gt;</span>
            <span class="o">&lt;</span><span class="n">groupId</span><span class="o">&gt;</span><span class="n">org</span><span class="o">.</span><span class="na">assertj</span><span class="o">&lt;/</span><span class="n">groupId</span><span class="o">&gt;</span>
            <span class="o">&lt;</span><span class="n">artifactId</span><span class="o">&gt;</span><span class="n">assertj</span><span class="o">-</span><span class="n">core</span><span class="o">&lt;/</span><span class="n">artifactId</span><span class="o">&gt;</span>
            <span class="o">&lt;</span><span class="n">version</span><span class="o">&gt;</span><span class="mf">3.27</span><span class="o">.</span><span class="mi">3</span><span class="o">&lt;/</span><span class="n">version</span><span class="o">&gt;</span>
            <span class="o">&lt;</span><span class="n">scope</span><span class="o">&gt;</span><span class="n">test</span><span class="o">&lt;/</span><span class="n">scope</span><span class="o">&gt;</span>
        <span class="o">&lt;/</span><span class="n">dependency</span><span class="o">&gt;</span>
    <span class="o">&lt;/</span><span class="n">dependencies</span><span class="o">&gt;</span>
</code></pre>

</div>



<p>With the project done, the next step is setting and defining the entities. In our sample, we will explore a simple hotel management system, where we will extract some use cases to further explore the data-driven test. Naturally, a hotel management system brings way more complexity than that. Thus, we won't cover points such as payment. Therefore, we will create a Room entity and its enums that will bring the Value Object perspective.</p>

<p>In the src/main/java directory, create a <em>Room</em> class:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight java"><code><span class="kn">import</span> <span class="nn">jakarta.nosql.Column</span><span class="o">;</span>
<span class="kn">import</span> <span class="nn">jakarta.nosql.Entity</span><span class="o">;</span>
<span class="kn">import</span> <span class="nn">jakarta.nosql.Id</span><span class="o">;</span>

<span class="kn">import</span> <span class="nn">java.util.Objects</span><span class="o">;</span>

<span class="nd">@Entity</span>
<span class="kd">public</span> <span class="kd">class</span> <span class="nc">Room</span> <span class="o">{</span>

    <span class="nd">@Id</span>
    <span class="kd">private</span> <span class="nc">String</span> <span class="n">id</span><span class="o">;</span>

    <span class="nd">@Column</span>
    <span class="kd">private</span> <span class="kt">int</span> <span class="n">number</span><span class="o">;</span>

    <span class="nd">@Column</span>
    <span class="kd">private</span> <span class="nc">RoomType</span> <span class="n">type</span><span class="o">;</span>

    <span class="nd">@Column</span>
    <span class="kd">private</span> <span class="nc">RoomStatus</span> <span class="n">status</span><span class="o">;</span>

    <span class="nd">@Column</span>
    <span class="kd">private</span> <span class="nc">CleanStatus</span> <span class="n">cleanStatus</span><span class="o">;</span>

    <span class="nd">@Column</span>
    <span class="kd">private</span> <span class="kt">boolean</span> <span class="n">smokingAllowed</span><span class="o">;</span>

    <span class="nd">@Column</span>
    <span class="kd">private</span> <span class="kt">boolean</span> <span class="n">underMaintenance</span><span class="o">;</span>
<span class="o">}</span>

<span class="kd">public</span> <span class="kd">enum</span> <span class="nc">RoomStatus</span> <span class="o">{</span>
    <span class="no">AVAILABLE</span><span class="o">,</span>
    <span class="no">RESERVED</span><span class="o">,</span>
    <span class="no">UNDER_MAINTENANCE</span><span class="o">,</span>
    <span class="no">OUT_OF_SERVICE</span>
<span class="o">}</span>

<span class="kd">public</span> <span class="kd">enum</span> <span class="nc">RoomType</span> <span class="o">{</span>
    <span class="no">STANDARD</span><span class="o">,</span>
    <span class="no">DELUXE</span><span class="o">,</span>
    <span class="no">SUITE</span><span class="o">,</span>
    <span class="no">VIP_SUITE</span>
<span class="o">}</span>

<span class="kd">public</span> <span class="kd">enum</span> <span class="nc">CleanStatus</span> <span class="o">{</span>
    <span class="no">CLEAN</span><span class="o">,</span>
    <span class="no">DIRTY</span><span class="o">,</span>
    <span class="no">INSPECTION_NEEDED</span>
<span class="o">}</span>

</code></pre>

</div>



<p>Explanation of annotations:</p>

<ul>
<li>
<strong>@Entity</strong>: Marks the Room class as a database entity for management by Jakarta NoSQL.</li>
<li>
<strong><a class="mentioned-user" href="https://dev.to/id">@id</a></strong>: Indicates the primary identifier for the entity, uniquely distinguishing each document in the MongoDB collection.</li>
<li>
<strong><a class="mentioned-user" href="https://dev.to/column">@column</a></strong>: Maps fields (roomNumber, type) for reading from or writing to MongoDB.</li>
</ul>

<p>Finally, with the entity done, we will create a repository where the goal is to find rooms by type, insert a new room, and check for availability on rooms—both standard and VIP rooms:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight java"><code><span class="nd">@Repository</span>
<span class="kd">public</span> <span class="kd">interface</span> <span class="nc">RoomRepository</span> <span class="o">{</span>

    <span class="nd">@Query</span><span class="o">(</span><span class="s">"WHERE type = 'VIP_SUITE' AND status = 'AVAILABLE' AND underMaintenance = false"</span><span class="o">)</span>
    <span class="nc">List</span><span class="o">&lt;</span><span class="nc">Room</span><span class="o">&gt;</span> <span class="nf">findVipRoomsReadyForGuests</span><span class="o">();</span>

    <span class="nd">@Query</span><span class="o">(</span><span class="s">" WHERE type &lt;&gt; 'VIP_SUITE' AND status = 'AVAILABLE' AND cleanStatus = 'CLEAN'"</span><span class="o">)</span>
    <span class="nc">List</span><span class="o">&lt;</span><span class="nc">Room</span><span class="o">&gt;</span> <span class="nf">findAvailableStandardRooms</span><span class="o">();</span>

    <span class="nd">@Query</span><span class="o">(</span><span class="s">"WHERE cleanStatus &lt;&gt; 'CLEAN' AND status &lt;&gt; 'OUT_OF_SERVICE'"</span><span class="o">)</span>
    <span class="nc">List</span><span class="o">&lt;</span><span class="nc">Room</span><span class="o">&gt;</span> <span class="nf">findRoomsNeedingCleaning</span><span class="o">();</span>

    <span class="nd">@Query</span><span class="o">(</span><span class="s">"WHERE smokingAllowed = true AND status = 'AVAILABLE'"</span><span class="o">)</span>
    <span class="nc">List</span><span class="o">&lt;</span><span class="nc">Room</span><span class="o">&gt;</span> <span class="nf">findAvailableSmokingRooms</span><span class="o">();</span>

    <span class="nd">@Save</span>
    <span class="kt">void</span> <span class="nf">save</span><span class="o">(</span><span class="nc">List</span><span class="o">&lt;</span><span class="nc">Room</span><span class="o">&gt;</span> <span class="n">rooms</span><span class="o">);</span>

    <span class="nd">@Save</span>
    <span class="nc">Room</span> <span class="nf">newRoom</span><span class="o">(</span><span class="nc">Room</span> <span class="n">room</span><span class="o">);</span>
    <span class="kt">void</span> <span class="nf">deleteBy</span><span class="o">();</span>

    <span class="nd">@Query</span><span class="o">(</span><span class="s">"WHERE type = :type"</span><span class="o">)</span>
    <span class="nc">List</span><span class="o">&lt;</span><span class="nc">Room</span><span class="o">&gt;</span> <span class="nf">findByType</span><span class="o">(</span><span class="nd">@Param</span><span class="o">(</span><span class="s">"type"</span><span class="o">)</span> <span class="nc">RoomType</span> <span class="n">type</span><span class="o">);</span>
<span class="o">}</span>

</code></pre>

</div>



<p>We have those queries that explore Jakarta Data Queries, but are they properly working? In the next step, we will generate some tests to check it.</p>

<h2>
  
  
  Step 2: Create a database container
</h2>

<p>After the entity and repository are done, the next step is to generate the test structure. Use Testcontainer to ensure that the database is running when we use this container. We will generate an enum to implement the Singleton pattern, which will create a MongoDB database container and start a MongoDB instance for testing purposes. </p>

<p>Thus, at src/test/java, create the DatabaseContainer class.<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight java"><code><span class="kn">import</span> <span class="nn">org.eclipse.jnosql.communication.Settings</span><span class="o">;</span>
<span class="kn">import</span> <span class="nn">org.eclipse.jnosql.databases.mongodb.communication.MongoDBDocumentConfiguration</span><span class="o">;</span>
<span class="kn">import</span> <span class="nn">org.eclipse.jnosql.databases.mongodb.communication.MongoDBDocumentConfigurations</span><span class="o">;</span>
<span class="kn">import</span> <span class="nn">org.eclipse.jnosql.databases.mongodb.communication.MongoDBDocumentManager</span><span class="o">;</span>
<span class="kn">import</span> <span class="nn">org.eclipse.jnosql.databases.mongodb.communication.MongoDBDocumentManagerFactory</span><span class="o">;</span>
<span class="kn">import</span> <span class="nn">org.eclipse.jnosql.mapping.core.config.MappingConfigurations</span><span class="o">;</span>
<span class="kn">import</span> <span class="nn">org.testcontainers.containers.GenericContainer</span><span class="o">;</span>
<span class="kn">import</span> <span class="nn">org.testcontainers.containers.wait.strategy.Wait</span><span class="o">;</span>

<span class="kn">import</span> <span class="nn">java.util.HashMap</span><span class="o">;</span>
<span class="kn">import</span> <span class="nn">java.util.Map</span><span class="o">;</span>
<span class="kd">public</span> <span class="kd">enum</span> <span class="nc">DatabaseContainer</span> <span class="o">{</span>

    <span class="no">INSTANCE</span><span class="o">;</span>

    <span class="kd">private</span> <span class="kd">final</span> <span class="nc">GenericContainer</span><span class="o">&lt;?&gt;</span> <span class="n">mongodb</span> <span class="o">=</span>
            <span class="k">new</span> <span class="nc">GenericContainer</span><span class="o">&lt;&gt;(</span><span class="s">"mongo:latest"</span><span class="o">)</span>
                    <span class="o">.</span><span class="na">withExposedPorts</span><span class="o">(</span><span class="mi">27017</span><span class="o">)</span>
                    <span class="o">.</span><span class="na">waitingFor</span><span class="o">(</span><span class="nc">Wait</span><span class="o">.</span><span class="na">defaultWaitStrategy</span><span class="o">());</span>

    <span class="o">{</span>
        <span class="n">mongodb</span><span class="o">.</span><span class="na">start</span><span class="o">();</span>
    <span class="o">}</span>
    <span class="kd">public</span> <span class="nc">MongoDBDocumentManager</span> <span class="nf">get</span><span class="o">(</span><span class="nc">String</span> <span class="n">database</span><span class="o">)</span> <span class="o">{</span>
        <span class="nc">Settings</span> <span class="n">settings</span> <span class="o">=</span> <span class="n">getSettings</span><span class="o">(</span><span class="n">database</span><span class="o">);</span>
        <span class="nc">MongoDBDocumentConfiguration</span> <span class="n">configuration</span> <span class="o">=</span> <span class="k">new</span> <span class="nc">MongoDBDocumentConfiguration</span><span class="o">();</span>
        <span class="nc">MongoDBDocumentManagerFactory</span> <span class="n">factory</span> <span class="o">=</span> <span class="n">configuration</span><span class="o">.</span><span class="na">apply</span><span class="o">(</span><span class="n">settings</span><span class="o">);</span>
        <span class="k">return</span> <span class="n">factory</span><span class="o">.</span><span class="na">apply</span><span class="o">(</span><span class="n">database</span><span class="o">);</span>
    <span class="o">}</span>


    <span class="kd">private</span> <span class="nc">Settings</span> <span class="nf">getSettings</span><span class="o">(</span><span class="nc">String</span> <span class="n">database</span><span class="o">)</span> <span class="o">{</span>
        <span class="nc">Map</span><span class="o">&lt;</span><span class="nc">String</span><span class="o">,</span><span class="nc">Object</span><span class="o">&gt;</span> <span class="n">settings</span> <span class="o">=</span> <span class="k">new</span> <span class="nc">HashMap</span><span class="o">&lt;&gt;();</span>
        <span class="n">settings</span><span class="o">.</span><span class="na">put</span><span class="o">(</span><span class="nc">MongoDBDocumentConfigurations</span><span class="o">.</span><span class="na">HOST</span><span class="o">.</span><span class="na">get</span><span class="o">()+</span><span class="s">".1"</span><span class="o">,</span> <span class="n">host</span><span class="o">());</span>
        <span class="n">settings</span><span class="o">.</span><span class="na">put</span><span class="o">(</span><span class="nc">MappingConfigurations</span><span class="o">.</span><span class="na">DOCUMENT_DATABASE</span><span class="o">.</span><span class="na">get</span><span class="o">(),</span> <span class="n">database</span><span class="o">);</span>
        <span class="k">return</span> <span class="nc">Settings</span><span class="o">.</span><span class="na">of</span><span class="o">(</span><span class="n">settings</span><span class="o">);</span>
    <span class="o">}</span>

    <span class="kd">public</span> <span class="nc">String</span> <span class="nf">host</span><span class="o">()</span> <span class="o">{</span>
        <span class="k">return</span> <span class="n">mongodb</span><span class="o">.</span><span class="na">getHost</span><span class="o">()</span> <span class="o">+</span> <span class="s">":"</span> <span class="o">+</span> <span class="n">mongodb</span><span class="o">.</span><span class="na">getFirstMappedPort</span><span class="o">();</span>
    <span class="o">}</span>
<span class="o">}</span>
</code></pre>

</div>



<p>In the code, we will generate a MongoDB database instance by container that we have a DatabaseConfiguration.</p>

<p>With the database container ready, the next step is to instruct CDI to use a MongoDB container from Testcontainer during testing, rather than any production configuration. We can do this by exploring the CDI capability of alternatives.</p>

<p>At src//test/java, create the ManagerSupplier class.<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight java"><code><span class="kn">import</span> <span class="nn">jakarta.annotation.Priority</span><span class="o">;</span>
<span class="kn">import</span> <span class="nn">jakarta.enterprise.context.ApplicationScoped</span><span class="o">;</span>
<span class="kn">import</span> <span class="nn">jakarta.enterprise.inject.Alternative</span><span class="o">;</span>
<span class="kn">import</span> <span class="nn">jakarta.enterprise.inject.Default</span><span class="o">;</span>
<span class="kn">import</span> <span class="nn">jakarta.enterprise.inject.Produces</span><span class="o">;</span>
<span class="kn">import</span> <span class="nn">jakarta.enterprise.inject.Typed</span><span class="o">;</span>
<span class="kn">import</span> <span class="nn">jakarta.interceptor.Interceptor</span><span class="o">;</span>
<span class="kn">import</span> <span class="nn">org.eclipse.jnosql.communication.semistructured.DatabaseManager</span><span class="o">;</span>
<span class="kn">import</span> <span class="nn">org.eclipse.jnosql.databases.mongodb.communication.MongoDBDocumentManager</span><span class="o">;</span>
<span class="kn">import</span> <span class="nn">org.eclipse.jnosql.mapping.Database</span><span class="o">;</span>
<span class="kn">import</span> <span class="nn">org.eclipse.jnosql.mapping.DatabaseType</span><span class="o">;</span>

<span class="kn">import</span> <span class="nn">java.util.function.Supplier</span><span class="o">;</span>

<span class="nd">@ApplicationScoped</span>
<span class="nd">@Alternative</span>
<span class="nd">@Priority</span><span class="o">(</span><span class="nc">Interceptor</span><span class="o">.</span><span class="na">Priority</span><span class="o">.</span><span class="na">APPLICATION</span><span class="o">)</span>
<span class="kd">public</span> <span class="kd">class</span> <span class="nc">ManagerSupplier</span> <span class="kd">implements</span> <span class="nc">Supplier</span><span class="o">&lt;</span><span class="nc">DatabaseManager</span><span class="o">&gt;</span> <span class="o">{</span>

    <span class="nd">@Produces</span>
    <span class="nd">@Database</span><span class="o">(</span><span class="nc">DatabaseType</span><span class="o">.</span><span class="na">DOCUMENT</span><span class="o">)</span>
    <span class="nd">@Default</span>
    <span class="nd">@Typed</span><span class="o">({</span><span class="nc">DatabaseManager</span><span class="o">.</span><span class="na">class</span><span class="o">,</span> <span class="nc">MongoDBDocumentManager</span><span class="o">.</span><span class="na">class</span><span class="o">})</span>
    <span class="kd">public</span> <span class="nc">MongoDBDocumentManager</span> <span class="nf">get</span><span class="o">()</span> <span class="o">{</span>
        <span class="k">return</span> <span class="nc">DatabaseContainer</span><span class="o">.</span><span class="na">INSTANCE</span><span class="o">.</span><span class="na">get</span><span class="o">(</span><span class="s">"hotel"</span><span class="o">);</span>
    <span class="o">}</span>
<span class="o">}</span>
</code></pre>

</div>



<p>On this class, we can see that we are overwriting the behavior at test where we are using the DatabaseContainer where the database is called hotel. With the structure done, the next step is playing with tests and DDT.</p>

<h2>
  
  
  Step 3: Generate our first DDT
</h2>

<p>One of the goals of data-driven testing is to achieve test coverage across various input combinations and capture the expected output. The first test we will generate is to verify that we are saving the information properly in the database. In this case, it does not matter which room is selected; it should insert and also generate an ID for it.</p>

<p>At src/test/java, create the RoomServiceTest class.<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight java"><code><span class="nd">@EnableAutoWeld</span>
<span class="nd">@AddPackages</span><span class="o">(</span><span class="n">value</span> <span class="o">=</span> <span class="o">{</span><span class="nc">Database</span><span class="o">.</span><span class="na">class</span><span class="o">,</span> <span class="nc">EntityConverter</span><span class="o">.</span><span class="na">class</span><span class="o">,</span> <span class="nc">DocumentTemplate</span><span class="o">.</span><span class="na">class</span><span class="o">,</span> <span class="nc">MongoDBTemplate</span><span class="o">.</span><span class="na">class</span><span class="o">})</span>
<span class="nd">@AddPackages</span><span class="o">(</span><span class="nc">Room</span><span class="o">.</span><span class="na">class</span><span class="o">)</span>
<span class="nd">@AddPackages</span><span class="o">(</span><span class="nc">ManagerSupplier</span><span class="o">.</span><span class="na">class</span><span class="o">)</span>
<span class="nd">@AddPackages</span><span class="o">(</span><span class="nc">MongoDBTemplate</span><span class="o">.</span><span class="na">class</span><span class="o">)</span>
<span class="nd">@AddPackages</span><span class="o">(</span><span class="nc">Reflections</span><span class="o">.</span><span class="na">class</span><span class="o">)</span>
<span class="nd">@AddPackages</span><span class="o">(</span><span class="nc">Converters</span><span class="o">.</span><span class="na">class</span><span class="o">)</span>
<span class="nd">@AddExtensions</span><span class="o">({</span><span class="nc">ReflectionEntityMetadataExtension</span><span class="o">.</span><span class="na">class</span><span class="o">,</span> <span class="nc">DocumentExtension</span><span class="o">.</span><span class="na">class</span><span class="o">})</span>
<span class="kd">class</span> <span class="nc">RoomServiceTest</span> <span class="o">{</span>

<span class="nd">@Inject</span>
<span class="kd">private</span> <span class="nc">RoomRepository</span> <span class="n">repository</span><span class="o">;</span>

<span class="kd">private</span> <span class="kd">static</span> <span class="kd">final</span> <span class="nc">Faker</span> <span class="no">FAKER</span> <span class="o">=</span> <span class="k">new</span> <span class="nc">Faker</span><span class="o">();</span>

<span class="nd">@ParameterizedTest</span>
<span class="nd">@MethodSource</span><span class="o">(</span><span class="s">"room"</span><span class="o">)</span>
<span class="kt">void</span> <span class="nf">shouldSaveRoom</span><span class="o">(</span><span class="nc">Room</span> <span class="n">room</span><span class="o">)</span> <span class="o">{</span>
    <span class="nc">Room</span> <span class="n">updateRoom</span> <span class="o">=</span> <span class="k">this</span><span class="o">.</span><span class="na">repository</span><span class="o">.</span><span class="na">newRoom</span><span class="o">(</span><span class="n">room</span><span class="o">);</span>
    <span class="nc">SoftAssertions</span><span class="o">.</span><span class="na">assertSoftly</span><span class="o">(</span><span class="n">softly</span> <span class="o">-&gt;</span> <span class="o">{</span>
        <span class="n">softly</span><span class="o">.</span><span class="na">assertThat</span><span class="o">(</span><span class="n">updateRoom</span><span class="o">).</span><span class="na">isNotNull</span><span class="o">();</span>
        <span class="n">softly</span><span class="o">.</span><span class="na">assertThat</span><span class="o">(</span><span class="n">updateRoom</span><span class="o">.</span><span class="na">getId</span><span class="o">()).</span><span class="na">isNotNull</span><span class="o">();</span>
        <span class="n">softly</span><span class="o">.</span><span class="na">assertThat</span><span class="o">(</span><span class="n">updateRoom</span><span class="o">.</span><span class="na">getNumber</span><span class="o">()).</span><span class="na">isEqualTo</span><span class="o">(</span><span class="n">room</span><span class="o">.</span><span class="na">getNumber</span><span class="o">());</span>
        <span class="n">softly</span><span class="o">.</span><span class="na">assertThat</span><span class="o">(</span><span class="n">updateRoom</span><span class="o">.</span><span class="na">getType</span><span class="o">()).</span><span class="na">isEqualTo</span><span class="o">(</span><span class="n">room</span><span class="o">.</span><span class="na">getType</span><span class="o">());</span>
        <span class="n">softly</span><span class="o">.</span><span class="na">assertThat</span><span class="o">(</span><span class="n">updateRoom</span><span class="o">.</span><span class="na">getStatus</span><span class="o">()).</span><span class="na">isEqualTo</span><span class="o">(</span><span class="n">room</span><span class="o">.</span><span class="na">getStatus</span><span class="o">());</span>
        <span class="n">softly</span><span class="o">.</span><span class="na">assertThat</span><span class="o">(</span><span class="n">updateRoom</span><span class="o">.</span><span class="na">getCleanStatus</span><span class="o">()).</span><span class="na">isEqualTo</span><span class="o">(</span><span class="n">room</span><span class="o">.</span><span class="na">getCleanStatus</span><span class="o">());</span>
        <span class="n">softly</span><span class="o">.</span><span class="na">assertThat</span><span class="o">(</span><span class="n">updateRoom</span><span class="o">.</span><span class="na">isSmokingAllowed</span><span class="o">()).</span><span class="na">isEqualTo</span><span class="o">(</span><span class="n">room</span><span class="o">.</span><span class="na">isSmokingAllowed</span><span class="o">());</span>
    <span class="o">});</span>
<span class="o">}</span>

  <span class="kd">static</span> <span class="nc">Stream</span><span class="o">&lt;</span><span class="nc">Arguments</span><span class="o">&gt;</span> <span class="nf">room</span><span class="o">()</span> <span class="o">{</span>
        <span class="k">return</span> <span class="nc">Stream</span><span class="o">.</span><span class="na">of</span><span class="o">(</span><span class="nc">Arguments</span><span class="o">.</span><span class="na">of</span><span class="o">(</span><span class="n">getRoom</span><span class="o">(),</span> <span class="nc">Arguments</span><span class="o">.</span><span class="na">of</span><span class="o">(</span><span class="n">getRoom</span><span class="o">(),</span> <span class="nc">Arguments</span><span class="o">.</span><span class="na">of</span><span class="o">(</span><span class="n">getRoom</span><span class="o">()))));</span>
    <span class="o">}</span>


    <span class="kd">private</span> <span class="kd">static</span> <span class="nc">Room</span> <span class="nf">getRoom</span><span class="o">()</span> <span class="o">{</span>
        <span class="k">return</span> <span class="k">new</span> <span class="nf">RoomBuilder</span><span class="o">()</span>
                <span class="o">.</span><span class="na">roomNumber</span><span class="o">(</span><span class="no">FAKER</span><span class="o">.</span><span class="na">number</span><span class="o">().</span><span class="na">numberBetween</span><span class="o">(</span><span class="mi">100</span><span class="o">,</span> <span class="mi">999</span><span class="o">))</span>
                <span class="o">.</span><span class="na">type</span><span class="o">(</span><span class="n">randomEnum</span><span class="o">(</span><span class="nc">RoomType</span><span class="o">.</span><span class="na">class</span><span class="o">))</span>
                <span class="o">.</span><span class="na">status</span><span class="o">(</span><span class="n">randomEnum</span><span class="o">(</span><span class="nc">RoomStatus</span><span class="o">.</span><span class="na">class</span><span class="o">))</span>
                <span class="o">.</span><span class="na">cleanStatus</span><span class="o">(</span><span class="n">randomEnum</span><span class="o">(</span><span class="nc">CleanStatus</span><span class="o">.</span><span class="na">class</span><span class="o">))</span>
                <span class="o">.</span><span class="na">smokingAllowed</span><span class="o">(</span><span class="no">FAKER</span><span class="o">.</span><span class="na">bool</span><span class="o">().</span><span class="na">bool</span><span class="o">())</span>
                <span class="o">.</span><span class="na">build</span><span class="o">();</span>
    <span class="o">}</span>

<span class="kd">private</span> <span class="kd">static</span> <span class="o">&lt;</span><span class="no">T</span> <span class="kd">extends</span> <span class="nc">Enum</span><span class="o">&lt;?&gt;&gt;</span> <span class="no">T</span> <span class="nf">randomEnum</span><span class="o">(</span><span class="nc">Class</span><span class="o">&lt;</span><span class="no">T</span><span class="o">&gt;</span> <span class="n">enumClass</span><span class="o">)</span> <span class="o">{</span>
    <span class="no">T</span><span class="o">[]</span> <span class="n">constants</span> <span class="o">=</span> <span class="n">enumClass</span><span class="o">.</span><span class="na">getEnumConstants</span><span class="o">();</span>
    <span class="kt">int</span> <span class="n">index</span> <span class="o">=</span> <span class="nc">ThreadLocalRandom</span><span class="o">.</span><span class="na">current</span><span class="o">().</span><span class="na">nextInt</span><span class="o">(</span><span class="n">constants</span><span class="o">.</span><span class="na">length</span><span class="o">);</span>
    <span class="k">return</span> <span class="n">constants</span><span class="o">[</span><span class="n">index</span><span class="o">];</span>
<span class="o">}</span>

<span class="o">}</span>
</code></pre>

</div>



<p>At the header of RoomService, we have a couple of annotations to activate Weld, the CDI implementation, in addition to which classes and packages the CDI should scan. It facilitates and makes the test startup lighter than scanning the whole class. Here, we are using AssertJ to further explore the fluent API for checking the database. We are using soft assertions that execute the whole validations and then show which conditions have break. It is way more useful when we need to do several validations in a single method.</p>

<p>We will generate a new test scenario that allows us to find rooms by type. Naturally, we want to ensure that it works for any kind of search. At the same class, we will generate a method where we will inject the enum by parameter, as you can see in the code below:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight java"><code><span class="nd">@ParameterizedTest</span><span class="o">(</span><span class="n">name</span> <span class="o">=</span> <span class="s">"should find rooms by type {0}"</span><span class="o">)</span>
<span class="nd">@EnumSource</span><span class="o">(</span><span class="nc">RoomType</span><span class="o">.</span><span class="na">class</span><span class="o">)</span>
<span class="kt">void</span> <span class="nf">shouldFindRoomByType</span><span class="o">(</span><span class="nc">RoomType</span> <span class="n">type</span><span class="o">)</span> <span class="o">{</span>
    <span class="nc">List</span><span class="o">&lt;</span><span class="nc">Room</span><span class="o">&gt;</span> <span class="n">rooms</span> <span class="o">=</span> <span class="k">this</span><span class="o">.</span><span class="na">repository</span><span class="o">.</span><span class="na">findByType</span><span class="o">(</span><span class="n">type</span><span class="o">);</span>
    <span class="nc">SoftAssertions</span><span class="o">.</span><span class="na">assertSoftly</span><span class="o">(</span><span class="n">softly</span> <span class="o">-&gt;</span> <span class="n">softly</span><span class="o">.</span><span class="na">assertThat</span><span class="o">(</span><span class="n">rooms</span><span class="o">).</span><span class="na">allMatch</span><span class="o">(</span><span class="n">room</span> <span class="o">-&gt;</span> <span class="n">room</span><span class="o">.</span><span class="na">getType</span><span class="o">().</span><span class="na">equals</span><span class="o">(</span><span class="n">type</span><span class="o">)));</span>
<span class="o">}</span>
</code></pre>

</div>



<p>We are injecting several inputs to validate the tests, and we can explore it even further to see if the tests qualify:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight java"><code><span class="kn">import</span> <span class="nn">jakarta.inject.Inject</span><span class="o">;</span>
<span class="kn">import</span> <span class="nn">net.datafaker.Faker</span><span class="o">;</span>
<span class="kn">import</span> <span class="nn">org.assertj.core.api.SoftAssertions</span><span class="o">;</span>
<span class="kn">import</span> <span class="nn">org.eclipse.jnosql.databases.mongodb.mapping.MongoDBTemplate</span><span class="o">;</span>
<span class="kn">import</span> <span class="nn">org.eclipse.jnosql.mapping.Database</span><span class="o">;</span>
<span class="kn">import</span> <span class="nn">org.eclipse.jnosql.mapping.core.Converters</span><span class="o">;</span>
<span class="kn">import</span> <span class="nn">org.eclipse.jnosql.mapping.document.DocumentTemplate</span><span class="o">;</span>
<span class="kn">import</span> <span class="nn">org.eclipse.jnosql.mapping.document.spi.DocumentExtension</span><span class="o">;</span>
<span class="kn">import</span> <span class="nn">org.eclipse.jnosql.mapping.reflection.Reflections</span><span class="o">;</span>
<span class="kn">import</span> <span class="nn">org.eclipse.jnosql.mapping.reflection.spi.ReflectionEntityMetadataExtension</span><span class="o">;</span>
<span class="kn">import</span> <span class="nn">org.eclipse.jnosql.mapping.semistructured.EntityConverter</span><span class="o">;</span>
<span class="kn">import</span> <span class="nn">org.jboss.weld.junit5.auto.AddExtensions</span><span class="o">;</span>
<span class="kn">import</span> <span class="nn">org.jboss.weld.junit5.auto.AddPackages</span><span class="o">;</span>
<span class="kn">import</span> <span class="nn">org.jboss.weld.junit5.auto.EnableAutoWeld</span><span class="o">;</span>
<span class="kn">import</span> <span class="nn">org.junit.jupiter.api.AfterEach</span><span class="o">;</span>
<span class="kn">import</span> <span class="nn">org.junit.jupiter.api.BeforeEach</span><span class="o">;</span>
<span class="kn">import</span> <span class="nn">org.junit.jupiter.api.Test</span><span class="o">;</span>
<span class="kn">import</span> <span class="nn">org.junit.jupiter.params.ParameterizedTest</span><span class="o">;</span>
<span class="kn">import</span> <span class="nn">org.junit.jupiter.params.provider.Arguments</span><span class="o">;</span>
<span class="kn">import</span> <span class="nn">org.junit.jupiter.params.provider.EnumSource</span><span class="o">;</span>
<span class="kn">import</span> <span class="nn">org.junit.jupiter.params.provider.MethodSource</span><span class="o">;</span>

<span class="kn">import</span> <span class="nn">java.util.List</span><span class="o">;</span>
<span class="kn">import</span> <span class="nn">java.util.concurrent.ThreadLocalRandom</span><span class="o">;</span>
<span class="kn">import</span> <span class="nn">java.util.stream.Stream</span><span class="o">;</span>


<span class="nd">@EnableAutoWeld</span>
<span class="nd">@AddPackages</span><span class="o">(</span><span class="n">value</span> <span class="o">=</span> <span class="o">{</span><span class="nc">Database</span><span class="o">.</span><span class="na">class</span><span class="o">,</span> <span class="nc">EntityConverter</span><span class="o">.</span><span class="na">class</span><span class="o">,</span> <span class="nc">DocumentTemplate</span><span class="o">.</span><span class="na">class</span><span class="o">,</span> <span class="nc">MongoDBTemplate</span><span class="o">.</span><span class="na">class</span><span class="o">})</span>
<span class="nd">@AddPackages</span><span class="o">(</span><span class="nc">Room</span><span class="o">.</span><span class="na">class</span><span class="o">)</span>
<span class="nd">@AddPackages</span><span class="o">(</span><span class="nc">ManagerSupplier</span><span class="o">.</span><span class="na">class</span><span class="o">)</span>
<span class="nd">@AddPackages</span><span class="o">(</span><span class="nc">MongoDBTemplate</span><span class="o">.</span><span class="na">class</span><span class="o">)</span>
<span class="nd">@AddPackages</span><span class="o">(</span><span class="nc">Reflections</span><span class="o">.</span><span class="na">class</span><span class="o">)</span>
<span class="nd">@AddPackages</span><span class="o">(</span><span class="nc">Converters</span><span class="o">.</span><span class="na">class</span><span class="o">)</span>
<span class="nd">@AddExtensions</span><span class="o">({</span><span class="nc">ReflectionEntityMetadataExtension</span><span class="o">.</span><span class="na">class</span><span class="o">,</span> <span class="nc">DocumentExtension</span><span class="o">.</span><span class="na">class</span><span class="o">})</span>
<span class="kd">class</span> <span class="nc">RoomServiceTest</span> <span class="o">{</span>

    <span class="nd">@Inject</span>
    <span class="kd">private</span> <span class="nc">RoomRepository</span> <span class="n">repository</span><span class="o">;</span>

    <span class="kd">private</span> <span class="kd">static</span> <span class="kd">final</span>  <span class="nc">Faker</span> <span class="no">FAKER</span> <span class="o">=</span> <span class="k">new</span> <span class="nc">Faker</span><span class="o">();</span>

    <span class="nd">@BeforeEach</span>
    <span class="kt">void</span> <span class="nf">setUP</span><span class="o">()</span> <span class="o">{</span>

        <span class="nc">Room</span> <span class="n">vipRoom1</span> <span class="o">=</span> <span class="k">new</span> <span class="nc">RoomBuilder</span><span class="o">()</span>
                <span class="o">.</span><span class="na">roomNumber</span><span class="o">(</span><span class="mi">101</span><span class="o">)</span>
                <span class="o">.</span><span class="na">type</span><span class="o">(</span><span class="nc">RoomType</span><span class="o">.</span><span class="na">VIP_SUITE</span><span class="o">)</span>
                <span class="o">.</span><span class="na">status</span><span class="o">(</span><span class="nc">RoomStatus</span><span class="o">.</span><span class="na">AVAILABLE</span><span class="o">)</span>
                <span class="o">.</span><span class="na">cleanStatus</span><span class="o">(</span><span class="nc">CleanStatus</span><span class="o">.</span><span class="na">CLEAN</span><span class="o">)</span>
                <span class="o">.</span><span class="na">smokingAllowed</span><span class="o">(</span><span class="kc">false</span><span class="o">)</span>
                <span class="o">.</span><span class="na">build</span><span class="o">();</span>

        <span class="nc">Room</span> <span class="n">vipRoom2</span> <span class="o">=</span> <span class="k">new</span> <span class="nc">RoomBuilder</span><span class="o">()</span>
                <span class="o">.</span><span class="na">roomNumber</span><span class="o">(</span><span class="mi">102</span><span class="o">)</span>
                <span class="o">.</span><span class="na">type</span><span class="o">(</span><span class="nc">RoomType</span><span class="o">.</span><span class="na">VIP_SUITE</span><span class="o">)</span>
                <span class="o">.</span><span class="na">status</span><span class="o">(</span><span class="nc">RoomStatus</span><span class="o">.</span><span class="na">AVAILABLE</span><span class="o">)</span>
                <span class="o">.</span><span class="na">cleanStatus</span><span class="o">(</span><span class="nc">CleanStatus</span><span class="o">.</span><span class="na">CLEAN</span><span class="o">)</span>
                <span class="o">.</span><span class="na">smokingAllowed</span><span class="o">(</span><span class="kc">true</span><span class="o">)</span>
                <span class="o">.</span><span class="na">build</span><span class="o">();</span>

        <span class="nc">Room</span> <span class="n">standardRoom1</span> <span class="o">=</span> <span class="k">new</span> <span class="nc">RoomBuilder</span><span class="o">()</span>
                <span class="o">.</span><span class="na">roomNumber</span><span class="o">(</span><span class="mi">201</span><span class="o">)</span>
                <span class="o">.</span><span class="na">type</span><span class="o">(</span><span class="nc">RoomType</span><span class="o">.</span><span class="na">STANDARD</span><span class="o">)</span>
                <span class="o">.</span><span class="na">status</span><span class="o">(</span><span class="nc">RoomStatus</span><span class="o">.</span><span class="na">AVAILABLE</span><span class="o">)</span>
                <span class="o">.</span><span class="na">cleanStatus</span><span class="o">(</span><span class="nc">CleanStatus</span><span class="o">.</span><span class="na">CLEAN</span><span class="o">)</span>
                <span class="o">.</span><span class="na">smokingAllowed</span><span class="o">(</span><span class="kc">false</span><span class="o">)</span>
                <span class="o">.</span><span class="na">build</span><span class="o">();</span>

        <span class="nc">Room</span> <span class="n">standardRoom2</span> <span class="o">=</span> <span class="k">new</span> <span class="nc">RoomBuilder</span><span class="o">()</span>
                <span class="o">.</span><span class="na">roomNumber</span><span class="o">(</span><span class="mi">202</span><span class="o">)</span>
                <span class="o">.</span><span class="na">type</span><span class="o">(</span><span class="nc">RoomType</span><span class="o">.</span><span class="na">DELUXE</span><span class="o">)</span>
                <span class="o">.</span><span class="na">status</span><span class="o">(</span><span class="nc">RoomStatus</span><span class="o">.</span><span class="na">AVAILABLE</span><span class="o">)</span>
                <span class="o">.</span><span class="na">cleanStatus</span><span class="o">(</span><span class="nc">CleanStatus</span><span class="o">.</span><span class="na">CLEAN</span><span class="o">)</span>
                <span class="o">.</span><span class="na">smokingAllowed</span><span class="o">(</span><span class="kc">false</span><span class="o">)</span>
                <span class="o">.</span><span class="na">build</span><span class="o">();</span>

        <span class="nc">Room</span> <span class="n">dirtyReservedRoom</span> <span class="o">=</span> <span class="k">new</span> <span class="nc">RoomBuilder</span><span class="o">()</span>
                <span class="o">.</span><span class="na">roomNumber</span><span class="o">(</span><span class="mi">301</span><span class="o">)</span>
                <span class="o">.</span><span class="na">type</span><span class="o">(</span><span class="nc">RoomType</span><span class="o">.</span><span class="na">DELUXE</span><span class="o">)</span>
                <span class="o">.</span><span class="na">status</span><span class="o">(</span><span class="nc">RoomStatus</span><span class="o">.</span><span class="na">RESERVED</span><span class="o">)</span>
                <span class="o">.</span><span class="na">cleanStatus</span><span class="o">(</span><span class="nc">CleanStatus</span><span class="o">.</span><span class="na">DIRTY</span><span class="o">)</span>
                <span class="o">.</span><span class="na">smokingAllowed</span><span class="o">(</span><span class="kc">false</span><span class="o">)</span>
                <span class="o">.</span><span class="na">build</span><span class="o">();</span>

        <span class="nc">Room</span> <span class="n">dirtySuiteRoom</span> <span class="o">=</span> <span class="k">new</span> <span class="nc">RoomBuilder</span><span class="o">()</span>
                <span class="o">.</span><span class="na">roomNumber</span><span class="o">(</span><span class="mi">302</span><span class="o">)</span>
                <span class="o">.</span><span class="na">type</span><span class="o">(</span><span class="nc">RoomType</span><span class="o">.</span><span class="na">SUITE</span><span class="o">)</span>
                <span class="o">.</span><span class="na">status</span><span class="o">(</span><span class="nc">RoomStatus</span><span class="o">.</span><span class="na">UNDER_MAINTENANCE</span><span class="o">)</span>
                <span class="o">.</span><span class="na">cleanStatus</span><span class="o">(</span><span class="nc">CleanStatus</span><span class="o">.</span><span class="na">INSPECTION_NEEDED</span><span class="o">)</span>
                <span class="o">.</span><span class="na">smokingAllowed</span><span class="o">(</span><span class="kc">false</span><span class="o">)</span>
                <span class="o">.</span><span class="na">build</span><span class="o">();</span>

        <span class="nc">Room</span> <span class="n">smokingAllowedRoom</span> <span class="o">=</span> <span class="k">new</span> <span class="nc">RoomBuilder</span><span class="o">()</span>
                <span class="o">.</span><span class="na">roomNumber</span><span class="o">(</span><span class="mi">401</span><span class="o">)</span>
                <span class="o">.</span><span class="na">type</span><span class="o">(</span><span class="nc">RoomType</span><span class="o">.</span><span class="na">STANDARD</span><span class="o">)</span>
                <span class="o">.</span><span class="na">status</span><span class="o">(</span><span class="nc">RoomStatus</span><span class="o">.</span><span class="na">AVAILABLE</span><span class="o">)</span>
                <span class="o">.</span><span class="na">cleanStatus</span><span class="o">(</span><span class="nc">CleanStatus</span><span class="o">.</span><span class="na">CLEAN</span><span class="o">)</span>
                <span class="o">.</span><span class="na">smokingAllowed</span><span class="o">(</span><span class="kc">true</span><span class="o">)</span>
                <span class="o">.</span><span class="na">build</span><span class="o">();</span>

        <span class="n">repository</span><span class="o">.</span><span class="na">save</span><span class="o">(</span><span class="nc">List</span><span class="o">.</span><span class="na">of</span><span class="o">(</span>
                <span class="n">vipRoom1</span><span class="o">,</span> <span class="n">vipRoom2</span><span class="o">,</span>
                <span class="n">standardRoom1</span><span class="o">,</span> <span class="n">standardRoom2</span><span class="o">,</span>
                <span class="n">dirtyReservedRoom</span><span class="o">,</span> <span class="n">dirtySuiteRoom</span><span class="o">,</span>
                <span class="n">smokingAllowedRoom</span>
        <span class="o">));</span>

    <span class="o">}</span>

    <span class="nd">@AfterEach</span>
    <span class="kt">void</span> <span class="nf">cleanUp</span><span class="o">()</span> <span class="o">{</span>
        <span class="n">repository</span><span class="o">.</span><span class="na">deleteBy</span><span class="o">();</span>
    <span class="o">}</span>

    <span class="nd">@ParameterizedTest</span><span class="o">(</span><span class="n">name</span> <span class="o">=</span> <span class="s">"should find rooms by type {0}"</span><span class="o">)</span>
    <span class="nd">@EnumSource</span><span class="o">(</span><span class="nc">RoomType</span><span class="o">.</span><span class="na">class</span><span class="o">)</span>
    <span class="kt">void</span> <span class="nf">shouldFindRoomByType</span><span class="o">(</span><span class="nc">RoomType</span> <span class="n">type</span><span class="o">)</span> <span class="o">{</span>
        <span class="nc">List</span><span class="o">&lt;</span><span class="nc">Room</span><span class="o">&gt;</span> <span class="n">rooms</span> <span class="o">=</span> <span class="k">this</span><span class="o">.</span><span class="na">repository</span><span class="o">.</span><span class="na">findByType</span><span class="o">(</span><span class="n">type</span><span class="o">);</span>
        <span class="nc">SoftAssertions</span><span class="o">.</span><span class="na">assertSoftly</span><span class="o">(</span><span class="n">softly</span> <span class="o">-&gt;</span> <span class="n">softly</span><span class="o">.</span><span class="na">assertThat</span><span class="o">(</span><span class="n">rooms</span><span class="o">).</span><span class="na">allMatch</span><span class="o">(</span><span class="n">room</span> <span class="o">-&gt;</span> <span class="n">room</span><span class="o">.</span><span class="na">getType</span><span class="o">().</span><span class="na">equals</span><span class="o">(</span><span class="n">type</span><span class="o">)));</span>
    <span class="o">}</span>

    <span class="nd">@ParameterizedTest</span>
    <span class="nd">@MethodSource</span><span class="o">(</span><span class="s">"room"</span><span class="o">)</span>
    <span class="kt">void</span> <span class="nf">shouldSaveRoom</span><span class="o">(</span><span class="nc">Room</span> <span class="n">room</span><span class="o">)</span> <span class="o">{</span>
        <span class="nc">Room</span> <span class="n">updateRoom</span> <span class="o">=</span> <span class="k">this</span><span class="o">.</span><span class="na">repository</span><span class="o">.</span><span class="na">newRoom</span><span class="o">(</span><span class="n">room</span><span class="o">);</span>

        <span class="nc">SoftAssertions</span><span class="o">.</span><span class="na">assertSoftly</span><span class="o">(</span><span class="n">softly</span> <span class="o">-&gt;</span> <span class="o">{</span>
            <span class="n">softly</span><span class="o">.</span><span class="na">assertThat</span><span class="o">(</span><span class="n">updateRoom</span><span class="o">).</span><span class="na">isNotNull</span><span class="o">();</span>
            <span class="n">softly</span><span class="o">.</span><span class="na">assertThat</span><span class="o">(</span><span class="n">updateRoom</span><span class="o">.</span><span class="na">getId</span><span class="o">()).</span><span class="na">isNotNull</span><span class="o">();</span>
            <span class="n">softly</span><span class="o">.</span><span class="na">assertThat</span><span class="o">(</span><span class="n">updateRoom</span><span class="o">.</span><span class="na">getNumber</span><span class="o">()).</span><span class="na">isEqualTo</span><span class="o">(</span><span class="n">room</span><span class="o">.</span><span class="na">getNumber</span><span class="o">());</span>
            <span class="n">softly</span><span class="o">.</span><span class="na">assertThat</span><span class="o">(</span><span class="n">updateRoom</span><span class="o">.</span><span class="na">getType</span><span class="o">()).</span><span class="na">isEqualTo</span><span class="o">(</span><span class="n">room</span><span class="o">.</span><span class="na">getType</span><span class="o">());</span>
            <span class="n">softly</span><span class="o">.</span><span class="na">assertThat</span><span class="o">(</span><span class="n">updateRoom</span><span class="o">.</span><span class="na">getStatus</span><span class="o">()).</span><span class="na">isEqualTo</span><span class="o">(</span><span class="n">room</span><span class="o">.</span><span class="na">getStatus</span><span class="o">());</span>
            <span class="n">softly</span><span class="o">.</span><span class="na">assertThat</span><span class="o">(</span><span class="n">updateRoom</span><span class="o">.</span><span class="na">getCleanStatus</span><span class="o">()).</span><span class="na">isEqualTo</span><span class="o">(</span><span class="n">room</span><span class="o">.</span><span class="na">getCleanStatus</span><span class="o">());</span>
            <span class="n">softly</span><span class="o">.</span><span class="na">assertThat</span><span class="o">(</span><span class="n">updateRoom</span><span class="o">.</span><span class="na">isSmokingAllowed</span><span class="o">()).</span><span class="na">isEqualTo</span><span class="o">(</span><span class="n">room</span><span class="o">.</span><span class="na">isSmokingAllowed</span><span class="o">());</span>
        <span class="o">});</span>
    <span class="o">}</span>


    <span class="nd">@Test</span>
    <span class="kt">void</span> <span class="nf">shouldFindRoomReadyToGuest</span><span class="o">()</span> <span class="o">{</span>
        <span class="nc">List</span><span class="o">&lt;</span><span class="nc">Room</span><span class="o">&gt;</span> <span class="n">rooms</span> <span class="o">=</span> <span class="k">this</span><span class="o">.</span><span class="na">repository</span><span class="o">.</span><span class="na">findAvailableStandardRooms</span><span class="o">();</span>
        <span class="nc">SoftAssertions</span><span class="o">.</span><span class="na">assertSoftly</span><span class="o">(</span><span class="n">softly</span> <span class="o">-&gt;</span> <span class="o">{</span>
            <span class="n">softly</span><span class="o">.</span><span class="na">assertThat</span><span class="o">(</span><span class="n">rooms</span><span class="o">).</span><span class="na">hasSize</span><span class="o">(</span><span class="mi">3</span><span class="o">);</span>
            <span class="n">softly</span><span class="o">.</span><span class="na">assertThat</span><span class="o">(</span><span class="n">rooms</span><span class="o">).</span><span class="na">allMatch</span><span class="o">(</span><span class="n">room</span> <span class="o">-&gt;</span> <span class="n">room</span><span class="o">.</span><span class="na">getStatus</span><span class="o">().</span><span class="na">equals</span><span class="o">(</span><span class="nc">RoomStatus</span><span class="o">.</span><span class="na">AVAILABLE</span><span class="o">));</span>
            <span class="n">softly</span><span class="o">.</span><span class="na">assertThat</span><span class="o">(</span><span class="n">rooms</span><span class="o">).</span><span class="na">allMatch</span><span class="o">(</span><span class="n">room</span> <span class="o">-&gt;</span> <span class="o">!</span><span class="n">room</span><span class="o">.</span><span class="na">isUnderMaintenance</span><span class="o">());</span>
        <span class="o">});</span>
    <span class="o">}</span>

    <span class="nd">@Test</span>
    <span class="kt">void</span> <span class="nf">shouldFindVipRoomsReadyForGuests</span><span class="o">()</span> <span class="o">{</span>
        <span class="nc">List</span><span class="o">&lt;</span><span class="nc">Room</span><span class="o">&gt;</span> <span class="n">rooms</span> <span class="o">=</span> <span class="k">this</span><span class="o">.</span><span class="na">repository</span><span class="o">.</span><span class="na">findVipRoomsReadyForGuests</span><span class="o">();</span>
        <span class="nc">SoftAssertions</span><span class="o">.</span><span class="na">assertSoftly</span><span class="o">(</span><span class="n">softly</span> <span class="o">-&gt;</span> <span class="o">{</span>
            <span class="n">softly</span><span class="o">.</span><span class="na">assertThat</span><span class="o">(</span><span class="n">rooms</span><span class="o">).</span><span class="na">hasSize</span><span class="o">(</span><span class="mi">2</span><span class="o">);</span>
            <span class="n">softly</span><span class="o">.</span><span class="na">assertThat</span><span class="o">(</span><span class="n">rooms</span><span class="o">).</span><span class="na">allMatch</span><span class="o">(</span><span class="n">room</span> <span class="o">-&gt;</span> <span class="n">room</span><span class="o">.</span><span class="na">getType</span><span class="o">().</span><span class="na">equals</span><span class="o">(</span><span class="nc">RoomType</span><span class="o">.</span><span class="na">VIP_SUITE</span><span class="o">));</span>
            <span class="n">softly</span><span class="o">.</span><span class="na">assertThat</span><span class="o">(</span><span class="n">rooms</span><span class="o">).</span><span class="na">allMatch</span><span class="o">(</span><span class="n">room</span> <span class="o">-&gt;</span> <span class="n">room</span><span class="o">.</span><span class="na">getStatus</span><span class="o">().</span><span class="na">equals</span><span class="o">(</span><span class="nc">RoomStatus</span><span class="o">.</span><span class="na">AVAILABLE</span><span class="o">));</span>
            <span class="n">softly</span><span class="o">.</span><span class="na">assertThat</span><span class="o">(</span><span class="n">rooms</span><span class="o">).</span><span class="na">allMatch</span><span class="o">(</span><span class="n">room</span> <span class="o">-&gt;</span> <span class="o">!</span><span class="n">room</span><span class="o">.</span><span class="na">isUnderMaintenance</span><span class="o">());</span>
        <span class="o">});</span>
    <span class="o">}</span>

    <span class="nd">@Test</span>
    <span class="kt">void</span> <span class="nf">shouldFindAvailableSmokingRooms</span><span class="o">()</span> <span class="o">{</span>
        <span class="nc">List</span><span class="o">&lt;</span><span class="nc">Room</span><span class="o">&gt;</span> <span class="n">rooms</span> <span class="o">=</span> <span class="k">this</span><span class="o">.</span><span class="na">repository</span><span class="o">.</span><span class="na">findAvailableSmokingRooms</span><span class="o">();</span>
        <span class="nc">SoftAssertions</span><span class="o">.</span><span class="na">assertSoftly</span><span class="o">(</span><span class="n">softly</span> <span class="o">-&gt;</span> <span class="o">{</span>
            <span class="n">softly</span><span class="o">.</span><span class="na">assertThat</span><span class="o">(</span><span class="n">rooms</span><span class="o">).</span><span class="na">hasSize</span><span class="o">(</span><span class="mi">2</span><span class="o">);</span>
            <span class="n">softly</span><span class="o">.</span><span class="na">assertThat</span><span class="o">(</span><span class="n">rooms</span><span class="o">).</span><span class="na">allMatch</span><span class="o">(</span><span class="n">room</span> <span class="o">-&gt;</span> <span class="n">room</span><span class="o">.</span><span class="na">isSmokingAllowed</span><span class="o">());</span>
            <span class="n">softly</span><span class="o">.</span><span class="na">assertThat</span><span class="o">(</span><span class="n">rooms</span><span class="o">).</span><span class="na">allMatch</span><span class="o">(</span><span class="n">room</span> <span class="o">-&gt;</span> <span class="n">room</span><span class="o">.</span><span class="na">getStatus</span><span class="o">().</span><span class="na">equals</span><span class="o">(</span><span class="nc">RoomStatus</span><span class="o">.</span><span class="na">AVAILABLE</span><span class="o">));</span>
        <span class="o">});</span>
    <span class="o">}</span>

    <span class="nd">@Test</span>
    <span class="kt">void</span> <span class="nf">shouldFindRoomsNeedingCleaning</span><span class="o">()</span> <span class="o">{</span>
        <span class="nc">List</span><span class="o">&lt;</span><span class="nc">Room</span><span class="o">&gt;</span> <span class="n">rooms</span> <span class="o">=</span> <span class="k">this</span><span class="o">.</span><span class="na">repository</span><span class="o">.</span><span class="na">findRoomsNeedingCleaning</span><span class="o">();</span>
        <span class="nc">SoftAssertions</span><span class="o">.</span><span class="na">assertSoftly</span><span class="o">(</span><span class="n">softly</span> <span class="o">-&gt;</span> <span class="o">{</span>
            <span class="n">softly</span><span class="o">.</span><span class="na">assertThat</span><span class="o">(</span><span class="n">rooms</span><span class="o">).</span><span class="na">hasSize</span><span class="o">(</span><span class="mi">2</span><span class="o">);</span>
            <span class="n">softly</span><span class="o">.</span><span class="na">assertThat</span><span class="o">(</span><span class="n">rooms</span><span class="o">).</span><span class="na">allMatch</span><span class="o">(</span><span class="n">room</span> <span class="o">-&gt;</span> <span class="o">!</span><span class="n">room</span><span class="o">.</span><span class="na">getCleanStatus</span><span class="o">().</span><span class="na">equals</span><span class="o">(</span><span class="nc">CleanStatus</span><span class="o">.</span><span class="na">CLEAN</span><span class="o">));</span>
            <span class="n">softly</span><span class="o">.</span><span class="na">assertThat</span><span class="o">(</span><span class="n">rooms</span><span class="o">).</span><span class="na">allMatch</span><span class="o">(</span><span class="n">room</span> <span class="o">-&gt;</span> <span class="o">!</span><span class="n">room</span><span class="o">.</span><span class="na">getStatus</span><span class="o">().</span><span class="na">equals</span><span class="o">(</span><span class="nc">RoomStatus</span><span class="o">.</span><span class="na">OUT_OF_SERVICE</span><span class="o">));</span>
        <span class="o">});</span>
    <span class="o">}</span>

    <span class="kd">static</span> <span class="nc">Stream</span><span class="o">&lt;</span><span class="nc">Arguments</span><span class="o">&gt;</span> <span class="nf">room</span><span class="o">()</span> <span class="o">{</span>
        <span class="k">return</span> <span class="nc">Stream</span><span class="o">.</span><span class="na">of</span><span class="o">(</span><span class="nc">Arguments</span><span class="o">.</span><span class="na">of</span><span class="o">(</span><span class="n">getRoom</span><span class="o">(),</span> <span class="nc">Arguments</span><span class="o">.</span><span class="na">of</span><span class="o">(</span><span class="n">getRoom</span><span class="o">(),</span> <span class="nc">Arguments</span><span class="o">.</span><span class="na">of</span><span class="o">(</span><span class="n">getRoom</span><span class="o">()))));</span>
    <span class="o">}</span>

    <span class="kd">private</span> <span class="kd">static</span> <span class="nc">Room</span> <span class="nf">getRoom</span><span class="o">()</span> <span class="o">{</span>
        <span class="k">return</span> <span class="k">new</span> <span class="nf">RoomBuilder</span><span class="o">()</span>
                <span class="o">.</span><span class="na">roomNumber</span><span class="o">(</span><span class="no">FAKER</span><span class="o">.</span><span class="na">number</span><span class="o">().</span><span class="na">numberBetween</span><span class="o">(</span><span class="mi">100</span><span class="o">,</span> <span class="mi">999</span><span class="o">))</span>
                <span class="o">.</span><span class="na">type</span><span class="o">(</span><span class="n">randomEnum</span><span class="o">(</span><span class="nc">RoomType</span><span class="o">.</span><span class="na">class</span><span class="o">))</span>
                <span class="o">.</span><span class="na">status</span><span class="o">(</span><span class="n">randomEnum</span><span class="o">(</span><span class="nc">RoomStatus</span><span class="o">.</span><span class="na">class</span><span class="o">))</span>
                <span class="o">.</span><span class="na">cleanStatus</span><span class="o">(</span><span class="n">randomEnum</span><span class="o">(</span><span class="nc">CleanStatus</span><span class="o">.</span><span class="na">class</span><span class="o">))</span>
                <span class="o">.</span><span class="na">smokingAllowed</span><span class="o">(</span><span class="no">FAKER</span><span class="o">.</span><span class="na">bool</span><span class="o">().</span><span class="na">bool</span><span class="o">())</span>
                <span class="o">.</span><span class="na">build</span><span class="o">();</span>
    <span class="o">}</span>

    <span class="kd">private</span> <span class="kd">static</span> <span class="o">&lt;</span><span class="no">T</span> <span class="kd">extends</span> <span class="nc">Enum</span><span class="o">&lt;?&gt;&gt;</span> <span class="no">T</span> <span class="nf">randomEnum</span><span class="o">(</span><span class="nc">Class</span><span class="o">&lt;</span><span class="no">T</span><span class="o">&gt;</span> <span class="n">enumClass</span><span class="o">)</span> <span class="o">{</span>
        <span class="no">T</span><span class="o">[]</span> <span class="n">constants</span> <span class="o">=</span> <span class="n">enumClass</span><span class="o">.</span><span class="na">getEnumConstants</span><span class="o">();</span>
        <span class="kt">int</span> <span class="n">index</span> <span class="o">=</span> <span class="nc">ThreadLocalRandom</span><span class="o">.</span><span class="na">current</span><span class="o">().</span><span class="na">nextInt</span><span class="o">(</span><span class="n">constants</span><span class="o">.</span><span class="na">length</span><span class="o">);</span>
        <span class="k">return</span> <span class="n">constants</span><span class="o">[</span><span class="n">index</span><span class="o">];</span>
    <span class="o">}</span>
<span class="o">}</span>
</code></pre>

</div>



<h2>
  
  
  Conclusion
</h2>

<p>In software development, the gap between business rules and database logic is often where subtle bugs and misunderstandings live. By adopting data-driven testing, we shift the focus from checking technical details to validating actual business behavior—across a wide range of scenarios and edge cases.</p>

<p>In this tutorial, you learned how to apply this approach using Java SE, Jakarta Data, Eclipse JNoSQL, and MongoDB. You saw how to express queries with business semantics, isolate your tests using Testcontainers, and validate outcomes with JUnit 5 and AssertJ. More than just testing correctness, this style helps you design repositories and queries that align with the domain itself. That’s the real power of data-driven testing: It turns your tests into a source of clarity, documentation, and confidence.</p>

<p>Ready to explore the benefits of MongoDB Atlas? Get started now by <a href="https://www.mongodb.com/lp/cloud/atlas/try4-reg?utm_campaign=devrel&amp;utm_source=third-party-content&amp;utm_medium=cta&amp;utm_content=data_driven_test_dev&amp;utm_term=otavio.santana" rel="noopener noreferrer">trying MongoDB Atlas</a>.</p>

<p>Any questions? Come chat with us in the <a href="https://www.mongodb.com/community/forums/?utm_campaign=devrel&amp;utm_source=third-party-content&amp;utm_medium=cta&amp;utm_content=data_driven_test_dev&amp;utm_term=otavio.santana" rel="noopener noreferrer">MongoDB Community Forum</a>.</p>

<p><strong>References</strong>:</p>

<ul>
<li><a href="https://github.com/soujava/data-driven-test-mongodb" rel="noopener noreferrer">Source code</a></li>
</ul>

