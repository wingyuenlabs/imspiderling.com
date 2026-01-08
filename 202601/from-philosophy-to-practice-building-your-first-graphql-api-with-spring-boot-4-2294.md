---
Title: From Philosophy to Practice: Building Your First GraphQL API with Spring Boot 4
Description: 
Author: dbc2201
Date: 2026-01-08T21:49:14.000Z
Robots: noindex,nofollow
Template: index
---
<h2>
  
  
  This post is for you if:
</h2>

<ol>
<li>You read (or skimmed) my <a href="https://dev.to/link-to-previous-post">REST vs GraphQL: Two Philosophies, Two Eras, One Endless Debate</a> article</li>
<li>You nodded along to "GraphQL thinks in queries, the client knows best"</li>
<li>Then you thought: "Cool philosophy, but how do I actually <em>build</em> one?"</li>
<li>You're a Java/Spring developer who learns by doing, not just reading</li>
</ol>

<p>If that's you, welcome to Part 2. Philosophy time is over. Now we build.</p>




<h2>
  
  
  The Context: Where We Left Off
</h2>

<p>In the previous article, I argued that GraphQL was born from Facebook's 2011 mobile nightmare—the realization that fetching a News Feed story shouldn't require 5+ HTTP round trips. We explored the philosophical difference:</p>

<blockquote>
<p><strong>REST asks:</strong> "What resource do you want?"<br><br>
<strong>GraphQL asks:</strong> "What data do you need?"</p>
</blockquote>

<p>We also saw Spring Framework 7 embrace API versioning as a first-class citizen, acknowledging that the real world is messier than academic purity.</p>

<p>Now, let's see what "the client knows best" looks like in actual Java code.</p>




<h2>
  
  
  What We're Building
</h2>

<p>A simple Book API. Nothing fancy—just enough to understand the core concepts:</p>

<ul>
<li>Query a book by ID</li>
<li>Get the book's author (demonstrating nested data fetching)</li>
<li>See how GraphQL resolves relationships without multiple endpoints</li>
</ul>

<p>By the end, you'll have a running GraphQL server and understand <em>why</em> the code is structured the way it is.</p>




<h2>
  
  
  Prerequisites
</h2>

<p>Before we start, make sure you have:</p>

<ul>
<li>
<strong>Java 25</strong> (LTS as of September 2025—or Java 21 if you're not ready to upgrade yet)</li>
<li>
<strong>Maven 3.9.12</strong> (the current stable release)</li>
<li>
<strong>Your favorite IDE</strong> (IntelliJ IDEA, VS Code with Java extensions, or even vim if you're that person)</li>
</ul>




<h2>
  
  
  Step 1: Create the Project
</h2>

<p>Head to <a href="https://start.spring.io/" rel="noopener noreferrer">Spring Initializr</a> and configure:</p>

<div class="table-wrapper-paragraph"><table>
<thead>
<tr>
<th>Setting</th>
<th>Value</th>
</tr>
</thead>
<tbody>
<tr>
<td>Project</td>
<td>Maven</td>
</tr>
<tr>
<td>Language</td>
<td>Java</td>
</tr>
<tr>
<td>Spring Boot</td>
<td>4.0.1 (latest stable)</td>
</tr>
<tr>
<td>Group</td>
<td><code>com.example</code></td>
</tr>
<tr>
<td>Artifact</td>
<td><code>bookstore-graphql</code></td>
</tr>
<tr>
<td>Name</td>
<td><code>bookstore-graphql</code></td>
</tr>
<tr>
<td>Packaging</td>
<td>Jar</td>
</tr>
<tr>
<td>Java</td>
<td>25</td>
</tr>
</tbody>
</table></div>

<p><strong>Dependencies to add:</strong></p>

<ul>
<li>Spring Web</li>
<li>Spring for GraphQL</li>
</ul>

<p>Click <strong>Generate</strong>, unzip, and open in your IDE.</p>

<p>Alternatively, here's the <code>pom.xml</code> you should end up with:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight xml"><code><span class="cp">&lt;?xml version="1.0" encoding="UTF-8"?&gt;</span>
<span class="nt">&lt;project</span> <span class="na">xmlns=</span><span class="s">"http://maven.apache.org/POM/4.0.0"</span>
         <span class="na">xmlns:xsi=</span><span class="s">"http://www.w3.org/2001/XMLSchema-instance"</span>
         <span class="na">xsi:schemaLocation=</span><span class="s">"http://maven.apache.org/POM/4.0.0 
         https://maven.apache.org/xsd/maven-4.0.0.xsd"</span><span class="nt">&gt;</span>
    <span class="nt">&lt;modelVersion&gt;</span>4.0.0<span class="nt">&lt;/modelVersion&gt;</span>

    <span class="nt">&lt;parent&gt;</span>
        <span class="nt">&lt;groupId&gt;</span>org.springframework.boot<span class="nt">&lt;/groupId&gt;</span>
        <span class="nt">&lt;artifactId&gt;</span>spring-boot-starter-parent<span class="nt">&lt;/artifactId&gt;</span>
        <span class="nt">&lt;version&gt;</span>4.0.1<span class="nt">&lt;/version&gt;</span>
        <span class="nt">&lt;relativePath/&gt;</span>
    <span class="nt">&lt;/parent&gt;</span>

    <span class="nt">&lt;groupId&gt;</span>com.example<span class="nt">&lt;/groupId&gt;</span>
    <span class="nt">&lt;artifactId&gt;</span>bookstore-graphql<span class="nt">&lt;/artifactId&gt;</span>
    <span class="nt">&lt;version&gt;</span>0.0.1-SNAPSHOT<span class="nt">&lt;/version&gt;</span>
    <span class="nt">&lt;name&gt;</span>bookstore-graphql<span class="nt">&lt;/name&gt;</span>
    <span class="nt">&lt;description&gt;</span>GraphQL API with Spring Boot 4<span class="nt">&lt;/description&gt;</span>

    <span class="nt">&lt;properties&gt;</span>
        <span class="nt">&lt;java.version&gt;</span>25<span class="nt">&lt;/java.version&gt;</span>
    <span class="nt">&lt;/properties&gt;</span>

    <span class="nt">&lt;dependencies&gt;</span>
        <span class="c">&lt;!-- Spring Web (still needed for HTTP transport) --&gt;</span>
        <span class="nt">&lt;dependency&gt;</span>
            <span class="nt">&lt;groupId&gt;</span>org.springframework.boot<span class="nt">&lt;/groupId&gt;</span>
            <span class="nt">&lt;artifactId&gt;</span>spring-boot-starter-web<span class="nt">&lt;/artifactId&gt;</span>
        <span class="nt">&lt;/dependency&gt;</span>

        <span class="c">&lt;!-- Spring for GraphQL --&gt;</span>
        <span class="nt">&lt;dependency&gt;</span>
            <span class="nt">&lt;groupId&gt;</span>org.springframework.boot<span class="nt">&lt;/groupId&gt;</span>
            <span class="nt">&lt;artifactId&gt;</span>spring-boot-starter-graphql<span class="nt">&lt;/artifactId&gt;</span>
        <span class="nt">&lt;/dependency&gt;</span>

        <span class="c">&lt;!-- Testing --&gt;</span>
        <span class="nt">&lt;dependency&gt;</span>
            <span class="nt">&lt;groupId&gt;</span>org.springframework.boot<span class="nt">&lt;/groupId&gt;</span>
            <span class="nt">&lt;artifactId&gt;</span>spring-boot-starter-test<span class="nt">&lt;/artifactId&gt;</span>
            <span class="nt">&lt;scope&gt;</span>test<span class="nt">&lt;/scope&gt;</span>
        <span class="nt">&lt;/dependency&gt;</span>
        <span class="nt">&lt;dependency&gt;</span>
            <span class="nt">&lt;groupId&gt;</span>org.springframework.graphql<span class="nt">&lt;/groupId&gt;</span>
            <span class="nt">&lt;artifactId&gt;</span>spring-graphql-test<span class="nt">&lt;/artifactId&gt;</span>
            <span class="nt">&lt;scope&gt;</span>test<span class="nt">&lt;/scope&gt;</span>
        <span class="nt">&lt;/dependency&gt;</span>
    <span class="nt">&lt;/dependencies&gt;</span>

    <span class="nt">&lt;build&gt;</span>
        <span class="nt">&lt;plugins&gt;</span>
            <span class="nt">&lt;plugin&gt;</span>
                <span class="nt">&lt;groupId&gt;</span>org.springframework.boot<span class="nt">&lt;/groupId&gt;</span>
                <span class="nt">&lt;artifactId&gt;</span>spring-boot-maven-plugin<span class="nt">&lt;/artifactId&gt;</span>
            <span class="nt">&lt;/plugin&gt;</span>
        <span class="nt">&lt;/plugins&gt;</span>
    <span class="nt">&lt;/build&gt;</span>
<span class="nt">&lt;/project&gt;</span>
</code></pre>

</div>



<p><strong>A note on versions:</strong> Spring Boot 4.0.1 ships with Spring for GraphQL 2.0.1, Spring Framework 7.0.2, and GraphQL Java 25.0. The versions are managed by the parent POM, so you don't need to specify them explicitly. This is one of Spring Boot's gifts to humanity.</p>




<h2>
  
  
  Step 2: Define the Schema (Schema-First Development)
</h2>

<p>Here's something important: <strong>GraphQL is schema-first by design.</strong></p>

<p>Unlike REST, where you might design your endpoints as you go, GraphQL forces you to think about your data contract upfront. The schema <em>is</em> the API documentation. The schema <em>is</em> the contract between client and server.</p>

<p>Create the file <code>src/main/resources/graphql/schema.graphqls</code>:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight graphql"><code><span class="k">type</span><span class="w"> </span><span class="n">Query</span><span class="w"> </span><span class="p">{</span><span class="w">
    </span><span class="n">bookById</span><span class="p">(</span><span class="n">id</span><span class="p">:</span><span class="w"> </span><span class="nb">ID</span><span class="p">!):</span><span class="w"> </span><span class="n">Book</span><span class="w">
    </span><span class="n">allBooks</span><span class="p">:</span><span class="w"> </span><span class="p">[</span><span class="n">Book</span><span class="p">!]!</span><span class="w">
</span><span class="p">}</span><span class="w">

</span><span class="k">type</span><span class="w"> </span><span class="n">Book</span><span class="w"> </span><span class="p">{</span><span class="w">
    </span><span class="n">id</span><span class="p">:</span><span class="w"> </span><span class="nb">ID</span><span class="p">!</span><span class="w">
    </span><span class="n">title</span><span class="p">:</span><span class="w"> </span><span class="nb">String</span><span class="p">!</span><span class="w">
    </span><span class="n">pageCount</span><span class="p">:</span><span class="w"> </span><span class="nb">Int</span><span class="w">
    </span><span class="n">author</span><span class="p">:</span><span class="w"> </span><span class="n">Author</span><span class="p">!</span><span class="w">
</span><span class="p">}</span><span class="w">

</span><span class="k">type</span><span class="w"> </span><span class="n">Author</span><span class="w"> </span><span class="p">{</span><span class="w">
    </span><span class="n">id</span><span class="p">:</span><span class="w"> </span><span class="nb">ID</span><span class="p">!</span><span class="w">
    </span><span class="n">firstName</span><span class="p">:</span><span class="w"> </span><span class="nb">String</span><span class="p">!</span><span class="w">
    </span><span class="n">lastName</span><span class="p">:</span><span class="w"> </span><span class="nb">String</span><span class="p">!</span><span class="w">
</span><span class="p">}</span><span class="w">
</span></code></pre>

</div>



<p>Let me break down what's happening here:</p>

<p><strong><code>type Query</code></strong> — This is the entry point. Every GraphQL API has a Query type that defines what clients can ask for. Think of it as your "read operations" menu.</p>

<p><strong><code>bookById(id: ID!): Book</code></strong> — A query that takes a required (<code>!</code>) ID and returns a Book. The <code>!</code> means "non-null"—the client <em>must</em> provide this argument.</p>

<p><strong><code>[Book!]!</code></strong> — This reads as: "a non-null array of non-null Books." Yes, you can have null arrays OR arrays with null elements in GraphQL. The <code>!</code> placement matters.</p>

<p><strong>Nested types</strong> — Notice how <code>Book</code> has an <code>author</code> field of type <code>Author</code>. This is where GraphQL shines. The client can ask for just the book title, or dive deep into author details—same endpoint, same schema, different queries.</p>




<h2>
  
  
  Step 3: Create the Domain Models
</h2>

<p>Now we need Java classes that match our schema. Java records are perfect for this—immutable, concise, and they generate <code>equals()</code>, <code>hashCode()</code>, and <code>toString()</code> for free.</p>

<p>Create <code>src/main/java/com/example/bookstoregraphql/model/Book.java</code>:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight java"><code><span class="kn">package</span> <span class="nn">com.example.bookstoregraphql.model</span><span class="o">;</span>

<span class="kd">public</span> <span class="n">record</span> <span class="nf">Book</span><span class="o">(</span>
    <span class="nc">String</span> <span class="n">id</span><span class="o">,</span>
    <span class="nc">String</span> <span class="n">title</span><span class="o">,</span>
    <span class="nc">Integer</span> <span class="n">pageCount</span><span class="o">,</span>
    <span class="nc">String</span> <span class="n">authorId</span>  <span class="c1">// Note: we store the ID, not the full Author object</span>
<span class="o">)</span> <span class="o">{</span>
    <span class="c1">// Sample data - in reality, this would come from a database</span>
    <span class="kd">private</span> <span class="kd">static</span> <span class="kd">final</span> <span class="n">java</span><span class="o">.</span><span class="na">util</span><span class="o">.</span><span class="na">List</span><span class="o">&lt;</span><span class="nc">Book</span><span class="o">&gt;</span> <span class="no">BOOKS</span> <span class="o">=</span> <span class="n">java</span><span class="o">.</span><span class="na">util</span><span class="o">.</span><span class="na">List</span><span class="o">.</span><span class="na">of</span><span class="o">(</span>
        <span class="k">new</span> <span class="nf">Book</span><span class="o">(</span><span class="s">"book-1"</span><span class="o">,</span> <span class="s">"Harry Potter and the Philosopher's Stone"</span><span class="o">,</span> <span class="mi">223</span><span class="o">,</span> <span class="s">"author-1"</span><span class="o">),</span>
        <span class="k">new</span> <span class="nf">Book</span><span class="o">(</span><span class="s">"book-2"</span><span class="o">,</span> <span class="s">"Moby Dick"</span><span class="o">,</span> <span class="mi">635</span><span class="o">,</span> <span class="s">"author-2"</span><span class="o">),</span>
        <span class="k">new</span> <span class="nf">Book</span><span class="o">(</span><span class="s">"book-3"</span><span class="o">,</span> <span class="s">"Interview with the Vampire"</span><span class="o">,</span> <span class="mi">371</span><span class="o">,</span> <span class="s">"author-3"</span><span class="o">),</span>
        <span class="k">new</span> <span class="nf">Book</span><span class="o">(</span><span class="s">"book-4"</span><span class="o">,</span> <span class="s">"The Great Gatsby"</span><span class="o">,</span> <span class="mi">180</span><span class="o">,</span> <span class="s">"author-4"</span><span class="o">),</span>
        <span class="k">new</span> <span class="nf">Book</span><span class="o">(</span><span class="s">"book-5"</span><span class="o">,</span> <span class="s">"Clean Code"</span><span class="o">,</span> <span class="mi">464</span><span class="o">,</span> <span class="s">"author-5"</span><span class="o">)</span>
    <span class="o">);</span>

    <span class="kd">public</span> <span class="kd">static</span> <span class="nc">Book</span> <span class="nf">getById</span><span class="o">(</span><span class="nc">String</span> <span class="n">id</span><span class="o">)</span> <span class="o">{</span>
        <span class="k">return</span> <span class="no">BOOKS</span><span class="o">.</span><span class="na">stream</span><span class="o">()</span>
            <span class="o">.</span><span class="na">filter</span><span class="o">(</span><span class="n">book</span> <span class="o">-&gt;</span> <span class="n">book</span><span class="o">.</span><span class="na">id</span><span class="o">().</span><span class="na">equals</span><span class="o">(</span><span class="n">id</span><span class="o">))</span>
            <span class="o">.</span><span class="na">findFirst</span><span class="o">()</span>
            <span class="o">.</span><span class="na">orElse</span><span class="o">(</span><span class="kc">null</span><span class="o">);</span>
    <span class="o">}</span>

    <span class="kd">public</span> <span class="kd">static</span> <span class="n">java</span><span class="o">.</span><span class="na">util</span><span class="o">.</span><span class="na">List</span><span class="o">&lt;</span><span class="nc">Book</span><span class="o">&gt;</span> <span class="nf">getAll</span><span class="o">()</span> <span class="o">{</span>
        <span class="k">return</span> <span class="no">BOOKS</span><span class="o">;</span>
    <span class="o">}</span>
<span class="o">}</span>
</code></pre>

</div>



<p>Create <code>src/main/java/com/example/bookstoregraphql/model/Author.java</code>:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight java"><code><span class="kn">package</span> <span class="nn">com.example.bookstoregraphql.model</span><span class="o">;</span>

<span class="kd">public</span> <span class="n">record</span> <span class="nf">Author</span><span class="o">(</span>
    <span class="nc">String</span> <span class="n">id</span><span class="o">,</span>
    <span class="nc">String</span> <span class="n">firstName</span><span class="o">,</span>
    <span class="nc">String</span> <span class="n">lastName</span>
<span class="o">)</span> <span class="o">{</span>
    <span class="kd">private</span> <span class="kd">static</span> <span class="kd">final</span> <span class="n">java</span><span class="o">.</span><span class="na">util</span><span class="o">.</span><span class="na">List</span><span class="o">&lt;</span><span class="nc">Author</span><span class="o">&gt;</span> <span class="no">AUTHORS</span> <span class="o">=</span> <span class="n">java</span><span class="o">.</span><span class="na">util</span><span class="o">.</span><span class="na">List</span><span class="o">.</span><span class="na">of</span><span class="o">(</span>
        <span class="k">new</span> <span class="nf">Author</span><span class="o">(</span><span class="s">"author-1"</span><span class="o">,</span> <span class="s">"Joanne"</span><span class="o">,</span> <span class="s">"Rowling"</span><span class="o">),</span>
        <span class="k">new</span> <span class="nf">Author</span><span class="o">(</span><span class="s">"author-2"</span><span class="o">,</span> <span class="s">"Herman"</span><span class="o">,</span> <span class="s">"Melville"</span><span class="o">),</span>
        <span class="k">new</span> <span class="nf">Author</span><span class="o">(</span><span class="s">"author-3"</span><span class="o">,</span> <span class="s">"Anne"</span><span class="o">,</span> <span class="s">"Rice"</span><span class="o">),</span>
        <span class="k">new</span> <span class="nf">Author</span><span class="o">(</span><span class="s">"author-4"</span><span class="o">,</span> <span class="s">"F. Scott"</span><span class="o">,</span> <span class="s">"Fitzgerald"</span><span class="o">),</span>
        <span class="k">new</span> <span class="nf">Author</span><span class="o">(</span><span class="s">"author-5"</span><span class="o">,</span> <span class="s">"Robert C."</span><span class="o">,</span> <span class="s">"Martin"</span><span class="o">)</span>
    <span class="o">);</span>

    <span class="kd">public</span> <span class="kd">static</span> <span class="nc">Author</span> <span class="nf">getById</span><span class="o">(</span><span class="nc">String</span> <span class="n">id</span><span class="o">)</span> <span class="o">{</span>
        <span class="k">return</span> <span class="no">AUTHORS</span><span class="o">.</span><span class="na">stream</span><span class="o">()</span>
            <span class="o">.</span><span class="na">filter</span><span class="o">(</span><span class="n">author</span> <span class="o">-&gt;</span> <span class="n">author</span><span class="o">.</span><span class="na">id</span><span class="o">().</span><span class="na">equals</span><span class="o">(</span><span class="n">id</span><span class="o">))</span>
            <span class="o">.</span><span class="na">findFirst</span><span class="o">()</span>
            <span class="o">.</span><span class="na">orElse</span><span class="o">(</span><span class="kc">null</span><span class="o">);</span>
    <span class="o">}</span>
<span class="o">}</span>
</code></pre>

</div>



<blockquote>
<p>💡 <strong>Production Note:</strong> We're using static lists inside records for simplicity. In a real application, you'd inject <code>BookRepository</code> and <code>AuthorRepository</code> (Spring Data JPA, for example) and fetch data from a database. The GraphQL resolution pattern stays the same—only the data source changes.</p>
</blockquote>

<p><strong>Why <code>authorId</code> instead of <code>Author</code> in the Book record?</strong></p>

<p>This is a deliberate design choice. In the GraphQL schema, <code>Book.author</code> is of type <code>Author</code>. But in our Java model, we store <code>authorId</code> as a String.</p>

<p>Why? Because <strong>GraphQL resolves fields lazily</strong>. If the client only asks for <code>book.title</code>, we never need to fetch the author. If they ask for <code>book.author.firstName</code>, <em>then</em> we resolve it. Storing the full Author object would force eager loading—exactly the N+1 problem we're trying to avoid.</p>




<h2>
  
  
  Step 4: Create the Controller (Data Fetchers)
</h2>

<p>This is where the magic happens. Spring for GraphQL uses annotations that feel familiar if you've written REST controllers, but they work fundamentally differently.</p>

<p>Create <code>src/main/java/com/example/bookstoregraphql/controller/BookController.java</code>:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight java"><code><span class="kn">package</span> <span class="nn">com.example.bookstoregraphql.controller</span><span class="o">;</span>

<span class="kn">import</span> <span class="nn">com.example.bookstoregraphql.model.Author</span><span class="o">;</span>
<span class="kn">import</span> <span class="nn">com.example.bookstoregraphql.model.Book</span><span class="o">;</span>
<span class="kn">import</span> <span class="nn">org.springframework.graphql.data.method.annotation.Argument</span><span class="o">;</span>
<span class="kn">import</span> <span class="nn">org.springframework.graphql.data.method.annotation.QueryMapping</span><span class="o">;</span>
<span class="kn">import</span> <span class="nn">org.springframework.graphql.data.method.annotation.SchemaMapping</span><span class="o">;</span>
<span class="kn">import</span> <span class="nn">org.springframework.stereotype.Controller</span><span class="o">;</span>

<span class="kn">import</span> <span class="nn">java.util.List</span><span class="o">;</span>

<span class="nd">@Controller</span>
<span class="kd">public</span> <span class="kd">class</span> <span class="nc">BookController</span> <span class="o">{</span>

    <span class="c1">// Maps to: Query.bookById(id: ID!)</span>
    <span class="nd">@QueryMapping</span>
    <span class="kd">public</span> <span class="nc">Book</span> <span class="nf">bookById</span><span class="o">(</span><span class="nd">@Argument</span> <span class="nc">String</span> <span class="n">id</span><span class="o">)</span> <span class="o">{</span>
        <span class="k">return</span> <span class="nc">Book</span><span class="o">.</span><span class="na">getById</span><span class="o">(</span><span class="n">id</span><span class="o">);</span>
    <span class="o">}</span>

    <span class="c1">// Maps to: Query.allBooks</span>
    <span class="nd">@QueryMapping</span>
    <span class="kd">public</span> <span class="nc">List</span><span class="o">&lt;</span><span class="nc">Book</span><span class="o">&gt;</span> <span class="nf">allBooks</span><span class="o">()</span> <span class="o">{</span>
        <span class="k">return</span> <span class="nc">Book</span><span class="o">.</span><span class="na">getAll</span><span class="o">();</span>
    <span class="o">}</span>

    <span class="c1">// Maps to: Book.author</span>
    <span class="c1">// This is called ONLY when the client requests the author field</span>
    <span class="nd">@SchemaMapping</span>
    <span class="kd">public</span> <span class="nc">Author</span> <span class="nf">author</span><span class="o">(</span><span class="nc">Book</span> <span class="n">book</span><span class="o">)</span> <span class="o">{</span>
        <span class="k">return</span> <span class="nc">Author</span><span class="o">.</span><span class="na">getById</span><span class="o">(</span><span class="n">book</span><span class="o">.</span><span class="na">authorId</span><span class="o">());</span>
    <span class="o">}</span>
<span class="o">}</span>
</code></pre>

</div>



<p>Let me explain these annotations because they're doing a lot of work:</p>

<p><strong><code>@QueryMapping</code></strong> — Binds this method to a field in the <code>Query</code> type. The method name (<code>bookById</code>) must match the field name in your schema. Spring for GraphQL registers this as a "data fetcher" for that field.</p>

<p><strong><code>@Argument</code></strong> — Extracts a named argument from the GraphQL query. The parameter name (<code>id</code>) must match the argument name in your schema.</p>

<p><strong><code>@SchemaMapping</code></strong> — This is the interesting one. It binds to a field on a <em>type</em> (not Query). When a client asks for <code>book.author</code>, GraphQL needs to know how to resolve that. The first parameter (<code>Book book</code>) tells Spring: "When resolving the <code>author</code> field on a <code>Book</code>, give me the parent Book object."</p>

<p><strong>The key insight:</strong> <code>@SchemaMapping</code> methods are only called when needed. If the client's query doesn't include <code>author</code>, this method never executes. That's lazy resolution in action.</p>




<h2>
  
  
  Checkpoint: What We've Built So Far
</h2>

<p>Before we test anything, let's make sure the mental model is clear.</p>

<p><strong>Schema → Code Mapping:</strong></p>

<div class="table-wrapper-paragraph"><table>
<thead>
<tr>
<th>GraphQL Schema</th>
<th>Java Code</th>
<th>Notes</th>
</tr>
</thead>
<tbody>
<tr>
<td><code>type Query</code></td>
<td>
<code>@QueryMapping</code> methods</td>
<td>Entry points for reads</td>
</tr>
<tr>
<td><code>type Book</code></td>
<td><code>record Book(...)</code></td>
<td>Domain model</td>
</tr>
<tr>
<td><code>author: Author!</code></td>
<td><code>@SchemaMapping Author author(Book book)</code></td>
<td>Lazy-resolved field</td>
</tr>
<tr>
<td><code>id: ID!</code></td>
<td><code>String id</code></td>
<td>GraphQL <code>ID</code> maps to <code>String</code>
</td>
</tr>
<tr>
<td><code>pageCount: Int</code></td>
<td><code>Integer pageCount</code></td>
<td>Nullable in schema → <code>Integer</code> (not <code>int</code>)</td>
</tr>
</tbody>
</table></div>

<p><strong>The Resolution Flow:</strong></p>

<ol>
<li>Client sends a query to <code>/graphql</code>
</li>
<li>Spring matches <code>bookById</code> to <code>@QueryMapping public Book bookById(...)</code>
</li>
<li>If the query includes <code>author</code>, Spring calls <code>@SchemaMapping public Author author(Book book)</code>
</li>
<li>If the query omits <code>author</code>, that method is never invoked</li>
</ol>

<p>This is the "client knows best" philosophy in action. The server doesn't decide what to return—it <em>reacts</em> to what's requested.</p>

<p><strong>🧪 Quick Check:</strong> Before moving on, can you predict what happens if you:</p>

<ul>
<li>Query <code>bookById</code> with an ID that doesn't exist?</li>
<li>Query <code>allBooks</code> but only request <code>title</code>—does the <code>author()</code> method run?</li>
</ul>

<p>Hold those predictions. We'll test them in a moment.</p>




<h2>
  
  
  Step 5: Enable GraphiQL (The Interactive Playground)
</h2>

<p>GraphiQL is an in-browser IDE for exploring GraphQL APIs. It provides auto-completion, documentation browsing, and query history. Essential for development.</p>

<p>Add to <code>src/main/resources/application.properties</code>:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight properties"><code><span class="c"># Enable GraphiQL playground
</span><span class="py">spring.graphql.graphiql.enabled</span><span class="p">=</span><span class="s">true</span>

<span class="c"># Optional: customize the path (default is /graphiql)
# spring.graphql.graphiql.path=/playground
</span>
<span class="c"># Optional: show schema in GraphiQL
</span><span class="py">spring.graphql.schema.printer.enabled</span><span class="p">=</span><span class="s">true</span>
</code></pre>

</div>






<h2>
  
  
  Step 6: Run and Test
</h2>

<p>Start the application:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight shell"><code>./mvnw spring-boot:run
</code></pre>

</div>



<p>Or from your IDE, run the main class.</p>

<p>Open your browser to: <strong><a href="http://localhost:8080/graphiql" rel="noopener noreferrer">http://localhost:8080/graphiql</a></strong></p>

<p>You should see the GraphiQL interface. Now let's run some queries.</p>

<h3>
  
  
  Query 1: Get a book with minimal fields
</h3>



<div class="highlight js-code-highlight">
<pre class="highlight graphql"><code><span class="k">query</span><span class="w"> </span><span class="p">{</span><span class="w">
  </span><span class="n">bookById</span><span class="p">(</span><span class="n">id</span><span class="p">:</span><span class="w"> </span><span class="s2">"book-1"</span><span class="p">)</span><span class="w"> </span><span class="p">{</span><span class="w">
    </span><span class="n">id</span><span class="w">
    </span><span class="n">title</span><span class="w">
  </span><span class="p">}</span><span class="w">
</span><span class="p">}</span><span class="w">
</span></code></pre>

</div>



<p><strong>Response:</strong><br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight json"><code><span class="p">{</span><span class="w">
  </span><span class="nl">"data"</span><span class="p">:</span><span class="w"> </span><span class="p">{</span><span class="w">
    </span><span class="nl">"bookById"</span><span class="p">:</span><span class="w"> </span><span class="p">{</span><span class="w">
      </span><span class="nl">"id"</span><span class="p">:</span><span class="w"> </span><span class="s2">"book-1"</span><span class="p">,</span><span class="w">
      </span><span class="nl">"title"</span><span class="p">:</span><span class="w"> </span><span class="s2">"Harry Potter and the Philosopher's Stone"</span><span class="w">
    </span><span class="p">}</span><span class="w">
  </span><span class="p">}</span><span class="w">
</span><span class="p">}</span><span class="w">
</span></code></pre>

</div>



<p>Notice: We didn't ask for <code>author</code>, so the <code>author()</code> method in our controller was never called. No unnecessary database hits.</p>

<h3>
  
  
  Query 2: Get a book with author details
</h3>



<div class="highlight js-code-highlight">
<pre class="highlight graphql"><code><span class="k">query</span><span class="w"> </span><span class="p">{</span><span class="w">
  </span><span class="n">bookById</span><span class="p">(</span><span class="n">id</span><span class="p">:</span><span class="w"> </span><span class="s2">"book-1"</span><span class="p">)</span><span class="w"> </span><span class="p">{</span><span class="w">
    </span><span class="n">id</span><span class="w">
    </span><span class="n">title</span><span class="w">
    </span><span class="n">pageCount</span><span class="w">
    </span><span class="n">author</span><span class="w"> </span><span class="p">{</span><span class="w">
      </span><span class="n">firstName</span><span class="w">
      </span><span class="n">lastName</span><span class="w">
    </span><span class="p">}</span><span class="w">
  </span><span class="p">}</span><span class="w">
</span><span class="p">}</span><span class="w">
</span></code></pre>

</div>



<p><strong>Response:</strong><br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight json"><code><span class="p">{</span><span class="w">
  </span><span class="nl">"data"</span><span class="p">:</span><span class="w"> </span><span class="p">{</span><span class="w">
    </span><span class="nl">"bookById"</span><span class="p">:</span><span class="w"> </span><span class="p">{</span><span class="w">
      </span><span class="nl">"id"</span><span class="p">:</span><span class="w"> </span><span class="s2">"book-1"</span><span class="p">,</span><span class="w">
      </span><span class="nl">"title"</span><span class="p">:</span><span class="w"> </span><span class="s2">"Harry Potter and the Philosopher's Stone"</span><span class="p">,</span><span class="w">
      </span><span class="nl">"pageCount"</span><span class="p">:</span><span class="w"> </span><span class="mi">223</span><span class="p">,</span><span class="w">
      </span><span class="nl">"author"</span><span class="p">:</span><span class="w"> </span><span class="p">{</span><span class="w">
        </span><span class="nl">"firstName"</span><span class="p">:</span><span class="w"> </span><span class="s2">"Joanne"</span><span class="p">,</span><span class="w">
        </span><span class="nl">"lastName"</span><span class="p">:</span><span class="w"> </span><span class="s2">"Rowling"</span><span class="w">
      </span><span class="p">}</span><span class="w">
    </span><span class="p">}</span><span class="w">
  </span><span class="p">}</span><span class="w">
</span><span class="p">}</span><span class="w">
</span></code></pre>

</div>



<p>Same endpoint. Different data. Client decides.</p>

<p><strong>🧪 Try this yourself:</strong> Before moving on, modify the query above to only request <code>pageCount</code>. Notice what happens—you get exactly what you asked for, nothing more. Now add <code>author { firstName }</code> back in. Watch how the response shape changes based purely on your query.</p>

<h3>
  
  
  Query 3: Get all books (with selective fields)
</h3>



<div class="highlight js-code-highlight">
<pre class="highlight graphql"><code><span class="k">query</span><span class="w"> </span><span class="p">{</span><span class="w">
  </span><span class="n">allBooks</span><span class="w"> </span><span class="p">{</span><span class="w">
    </span><span class="n">title</span><span class="w">
    </span><span class="n">author</span><span class="w"> </span><span class="p">{</span><span class="w">
      </span><span class="n">lastName</span><span class="w">
    </span><span class="p">}</span><span class="w">
  </span><span class="p">}</span><span class="w">
</span><span class="p">}</span><span class="w">
</span></code></pre>

</div>



<p><strong>Response:</strong><br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight json"><code><span class="p">{</span><span class="w">
  </span><span class="nl">"data"</span><span class="p">:</span><span class="w"> </span><span class="p">{</span><span class="w">
    </span><span class="nl">"allBooks"</span><span class="p">:</span><span class="w"> </span><span class="p">[</span><span class="w">
      </span><span class="p">{</span><span class="w"> </span><span class="nl">"title"</span><span class="p">:</span><span class="w"> </span><span class="s2">"Harry Potter and the Philosopher's Stone"</span><span class="p">,</span><span class="w"> </span><span class="nl">"author"</span><span class="p">:</span><span class="w"> </span><span class="p">{</span><span class="w"> </span><span class="nl">"lastName"</span><span class="p">:</span><span class="w"> </span><span class="s2">"Rowling"</span><span class="w"> </span><span class="p">}</span><span class="w"> </span><span class="p">},</span><span class="w">
      </span><span class="p">{</span><span class="w"> </span><span class="nl">"title"</span><span class="p">:</span><span class="w"> </span><span class="s2">"Moby Dick"</span><span class="p">,</span><span class="w"> </span><span class="nl">"author"</span><span class="p">:</span><span class="w"> </span><span class="p">{</span><span class="w"> </span><span class="nl">"lastName"</span><span class="p">:</span><span class="w"> </span><span class="s2">"Melville"</span><span class="w"> </span><span class="p">}</span><span class="w"> </span><span class="p">},</span><span class="w">
      </span><span class="p">{</span><span class="w"> </span><span class="nl">"title"</span><span class="p">:</span><span class="w"> </span><span class="s2">"Interview with the Vampire"</span><span class="p">,</span><span class="w"> </span><span class="nl">"author"</span><span class="p">:</span><span class="w"> </span><span class="p">{</span><span class="w"> </span><span class="nl">"lastName"</span><span class="p">:</span><span class="w"> </span><span class="s2">"Rice"</span><span class="w"> </span><span class="p">}</span><span class="w"> </span><span class="p">},</span><span class="w">
      </span><span class="p">{</span><span class="w"> </span><span class="nl">"title"</span><span class="p">:</span><span class="w"> </span><span class="s2">"The Great Gatsby"</span><span class="p">,</span><span class="w"> </span><span class="nl">"author"</span><span class="p">:</span><span class="w"> </span><span class="p">{</span><span class="w"> </span><span class="nl">"lastName"</span><span class="p">:</span><span class="w"> </span><span class="s2">"Fitzgerald"</span><span class="w"> </span><span class="p">}</span><span class="w"> </span><span class="p">},</span><span class="w">
      </span><span class="p">{</span><span class="w"> </span><span class="nl">"title"</span><span class="p">:</span><span class="w"> </span><span class="s2">"Clean Code"</span><span class="p">,</span><span class="w"> </span><span class="nl">"author"</span><span class="p">:</span><span class="w"> </span><span class="p">{</span><span class="w"> </span><span class="nl">"lastName"</span><span class="p">:</span><span class="w"> </span><span class="s2">"Martin"</span><span class="w"> </span><span class="p">}</span><span class="w"> </span><span class="p">}</span><span class="w">
    </span><span class="p">]</span><span class="w">
  </span><span class="p">}</span><span class="w">
</span><span class="p">}</span><span class="w">
</span></code></pre>

</div>



<h3>
  
  
  Query 4: Multiple queries in one request
</h3>



<div class="highlight js-code-highlight">
<pre class="highlight graphql"><code><span class="k">query</span><span class="w"> </span><span class="p">{</span><span class="w">
  </span><span class="n">potter</span><span class="p">:</span><span class="w"> </span><span class="n">bookById</span><span class="p">(</span><span class="n">id</span><span class="p">:</span><span class="w"> </span><span class="s2">"book-1"</span><span class="p">)</span><span class="w"> </span><span class="p">{</span><span class="w">
    </span><span class="n">title</span><span class="w">
  </span><span class="p">}</span><span class="w">
  </span><span class="n">cleanCode</span><span class="p">:</span><span class="w"> </span><span class="n">bookById</span><span class="p">(</span><span class="n">id</span><span class="p">:</span><span class="w"> </span><span class="s2">"book-5"</span><span class="p">)</span><span class="w"> </span><span class="p">{</span><span class="w">
    </span><span class="n">title</span><span class="w">
    </span><span class="n">author</span><span class="w"> </span><span class="p">{</span><span class="w">
      </span><span class="n">firstName</span><span class="w">
      </span><span class="n">lastName</span><span class="w">
    </span><span class="p">}</span><span class="w">
  </span><span class="p">}</span><span class="w">
</span><span class="p">}</span><span class="w">
</span></code></pre>

</div>



<p><strong>Response:</strong><br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight json"><code><span class="p">{</span><span class="w">
  </span><span class="nl">"data"</span><span class="p">:</span><span class="w"> </span><span class="p">{</span><span class="w">
    </span><span class="nl">"potter"</span><span class="p">:</span><span class="w"> </span><span class="p">{</span><span class="w">
      </span><span class="nl">"title"</span><span class="p">:</span><span class="w"> </span><span class="s2">"Harry Potter and the Philosopher's Stone"</span><span class="w">
    </span><span class="p">},</span><span class="w">
    </span><span class="nl">"cleanCode"</span><span class="p">:</span><span class="w"> </span><span class="p">{</span><span class="w">
      </span><span class="nl">"title"</span><span class="p">:</span><span class="w"> </span><span class="s2">"Clean Code"</span><span class="p">,</span><span class="w">
      </span><span class="nl">"author"</span><span class="p">:</span><span class="w"> </span><span class="p">{</span><span class="w">
        </span><span class="nl">"firstName"</span><span class="p">:</span><span class="w"> </span><span class="s2">"Robert C."</span><span class="p">,</span><span class="w">
        </span><span class="nl">"lastName"</span><span class="p">:</span><span class="w"> </span><span class="s2">"Martin"</span><span class="w">
      </span><span class="p">}</span><span class="w">
    </span><span class="p">}</span><span class="w">
  </span><span class="p">}</span><span class="w">
</span><span class="p">}</span><span class="w">
</span></code></pre>

</div>



<p>Two queries, one HTTP request. Aliases (<code>potter:</code>, <code>cleanCode:</code>) let you query the same field multiple times with different arguments.</p>




<h2>
  
  
  What Just Happened?
</h2>

<p>Let's reflect on what we built and <em>why</em> it matters.</p>

<h3>
  
  
  The REST Equivalent Would Have Been:
</h3>



<div class="highlight js-code-highlight">
<pre class="highlight plaintext"><code>GET /api/books/book-1                    → Full book object (whether you need it all or not)
GET /api/books/book-1?include=author     → Custom query param handling
GET /api/books/book-1/author             → Separate endpoint for author
GET /api/books                           → All books (hope you don't need authors too)
GET /api/books?include=author            → More custom handling
</code></pre>

</div>



<p>Five endpoints (at minimum), plus decisions about query parameters, response shapes, and documentation.</p>

<h3>
  
  
  With GraphQL We Have:
</h3>



<div class="highlight js-code-highlight">
<pre class="highlight plaintext"><code>POST /graphql
</code></pre>

</div>



<p>One endpoint. The query itself describes what you want. The schema documents what's possible.</p>

<h3>
  
  
  The Tradeoffs (Because There Are Always Tradeoffs):
</h3>

<div class="table-wrapper-paragraph"><table>
<thead>
<tr>
<th>Benefit</th>
<th>Cost</th>
</tr>
</thead>
<tbody>
<tr>
<td>Client flexibility</td>
<td>Learning curve for clients</td>
</tr>
<tr>
<td>Self-documenting schema</td>
<td>Schema maintenance burden</td>
</tr>
<tr>
<td>No over-fetching</td>
<td>Potential for expensive queries</td>
</tr>
<tr>
<td>Single endpoint</td>
<td>Harder to cache at HTTP level</td>
</tr>
<tr>
<td>Strong typing</td>
<td>More upfront design work</td>
</tr>
</tbody>
</table></div>




<h2>
  
  
  The N+1 Problem: A Preview
</h2>

<p>Remember the N+1 problem from the previous article? Our current implementation has it.</p>

<p>If you call <code>allBooks</code> and request <code>author</code> for each, the <code>author()</code> method gets called once per book. Five books = five <code>Author.getById()</code> calls.</p>

<p>In a real application with a database, that's five SQL queries when one would suffice.</p>

<p>The solution? <strong>Batching</strong>. Spring for GraphQL provides two options: the traditional <code>DataLoader</code> pattern (from GraphQL Java) or the Spring-native <code>@BatchMapping</code> annotation. Both batch those five requests into a single fetch. We'll cover both approaches in the next article.</p>

<p>For now, just know: <strong>production GraphQL requires batching</strong>. Don't skip it.</p>




<h2>
  
  
  Project Structure Recap
</h2>



<div class="highlight js-code-highlight">
<pre class="highlight plaintext"><code>bookstore-graphql/
├── pom.xml
└── src/main/
    ├── java/com/example/bookstoregraphql/
    │   ├── BookstoreGraphqlApplication.java
    │   ├── controller/
    │   │   └── BookController.java
    │   └── model/
    │       ├── Book.java
    │       └── Author.java
    └── resources/
        ├── application.properties
        └── graphql/
            └── schema.graphqls
</code></pre>

</div>






<h2>
  
  
  Troubleshooting Common Issues
</h2>

<div class="table-wrapper-paragraph"><table>
<thead>
<tr>
<th>Symptom</th>
<th>Likely Cause</th>
<th>Fix</th>
</tr>
</thead>
<tbody>
<tr>
<td><code>SchemaResourceResolver: No schema files found</code></td>
<td>Schema file in wrong location</td>
<td>Must be at <code>src/main/resources/graphql/schema.graphqls</code> (note the <code>.graphqls</code> extension, not <code>.graphql</code>)</td>
</tr>
<tr>
<td><code>No DataFetcher for field 'bookById'</code></td>
<td>Method name doesn't match schema</td>
<td>Ensure <code>@QueryMapping</code> method name matches the field name in your schema exactly</td>
</tr>
<tr>
<td><code>Cannot resolve symbol 'QueryMapping'</code></td>
<td>Missing import</td>
<td>Add: <code>import org.springframework.graphql.data.method.annotation.*;</code>
</td>
</tr>
<tr>
<td>GraphiQL shows blank page</td>
<td>GraphiQL not enabled</td>
<td>Ensure <code>spring.graphql.graphiql.enabled=true</code> in <code>application.properties</code>
</td>
</tr>
<tr>
<td>
<code>NullPointerException</code> in <code>author()</code> method</td>
<td>Book has authorId with no matching Author</td>
<td>Add null check: <code>return author != null ? author : null;</code> or ensure all author IDs have matching entries</td>
</tr>
<tr>
<td>Port 8080 already in use</td>
<td>Another app running</td>
<td>Add <code>server.port=8081</code> to <code>application.properties</code> or stop the other process</td>
</tr>
</tbody>
</table></div>




<h2>
  
  
  What's Next?
</h2>

<p>This tutorial covered the basics: schema definition, data fetchers, and the mental model of GraphQL resolution. But production GraphQL involves more:</p>

<ol>
<li>
<strong>DataLoader and batching</strong> — Solving the N+1 problem</li>
<li>
<strong>Mutations</strong> — Creating, updating, and deleting data</li>
<li>
<strong>Subscriptions</strong> — Real-time data with WebSocket</li>
<li>
<strong>Error handling</strong> — Partial responses and error types</li>
<li>
<strong>Authentication &amp; Authorization</strong> — Securing your schema</li>
<li>
<strong>Testing</strong> — Spring GraphQL's testing utilities</li>
</ol>

<p>If there's interest, I'll continue this series. Let me know in the comments which topic you'd want next.</p>




<h2>
  
  
  The Takeaway
</h2>

<p>GraphQL isn't magic. It's a different way of thinking about data access—one where the client describes what it needs rather than hoping the server guessed correctly.</p>

<p>Spring for GraphQL makes this approachable for Java developers. The annotations feel familiar. The integration with Spring Boot is seamless. And with Java records, your domain models stay clean.</p>

<p>But remember what I said in the philosophy article:</p>

<blockquote>
<p>Technologies carry the DNA of the problems they solved.</p>
</blockquote>

<p>GraphQL carries Facebook's DNA: complex relationships, mobile-first thinking, and client-driven data needs. If your use case matches, GraphQL is powerful. If you're building simple CRUD APIs for internal services, REST might still be the pragmatic choice.</p>

<p>The skill isn't knowing GraphQL. The skill is knowing <em>when</em> to use it.</p>




<h2>
  
  
  Resources
</h2>

<ul>
<li><a href="https://docs.spring.io/spring-graphql/reference/" rel="noopener noreferrer">Spring for GraphQL Documentation</a></li>
<li>
<a href="https://leanpub.com/graphql-java" rel="noopener noreferrer">GraphQL Java Book</a> by Andreas Marek (the GraphQL Java maintainer)</li>
<li><a href="https://spec.graphql.org/" rel="noopener noreferrer">GraphQL Specification</a></li>
<li>
<a href="https://github.com/your-repo/bookstore-graphql" rel="noopener noreferrer">Source code for this tutorial</a> <em>(coming soon)</em>
</li>
</ul>




<p><em>Found this useful? Have questions? Drop a comment below. And if you haven't read the philosophy piece yet, <a href="https://dev.to/link-to-previous-post">start there</a>—it'll make everything here click.</em></p>

