---
Title: An Enum Alternative to the Factory Pattern: The Pros, Cons, and Hidden Dangers
Description: 
Author: Ketki Ambekar
Date: 2025-09-03T21:31:09.000Z
Robots: noindex,nofollow
Template: index
---
<p>The Factory Pattern is a go-to solution for creating objects, but what if there were a way to make it more flexible and scalable? Let's explore how an enum can be used to drive a dynamic object creation process.</p>

<p>Say you have a program that wants to ingest some data. The said data could be read from a variety of sources and formats, like text dumps, PDF files, XML gateways, SQL data dumps, text scraped from websites, etc.</p>

<p>With an interface <code>IDataReader</code> as our super type, we could create a separate concrete class for reading each type of data. Let's talk about selecting the right concrete class for reading data in the calling client class.</p>

<h2>
  
  
  <strong>The Factory Pattern Technique:</strong>
</h2>

<p>Typically, you'd go about implementing this by creating a Factory class. To put it simply, it is a class that takes care of the logic by which we determine which object to create. We could use a variable called <code>readerType</code> whose value could be configured or received through an API call, among other ways.  In the example below, we determine which concrete object of the <code>IDataReader</code> type needs to be returned using a factory class that employs a switch logic.<br>
<br>
 </p>
<div class="highlight js-code-highlight">
<pre class="highlight csharp"><code>
<span class="c1">//C#</span>

<span class="k">using</span> <span class="nn">FactoryPatternEnumDemo.DataReaders</span><span class="p">;</span>
<span class="k">namespace</span> <span class="nn">FactoryPatternEnumDemo</span><span class="p">;</span>

<span class="k">public</span> <span class="k">static</span> <span class="k">class</span> <span class="nc">DataReaderFactory</span>
<span class="p">{</span>
<span class="err"> </span> <span class="err"> </span> <span class="k">public</span> <span class="k">static</span> <span class="n">IDataReader</span> <span class="nf">GetDataReader</span><span class="p">(</span><span class="kt">string</span> <span class="n">readerType</span><span class="p">)</span>
<span class="err"> </span> <span class="err"> </span> <span class="p">{</span>
<span class="err"> </span> <span class="err"> </span> <span class="err"> </span> <span class="err"> </span> <span class="k">switch</span><span class="p">(</span><span class="n">readerType</span><span class="p">)</span>
<span class="err"> </span> <span class="err"> </span> <span class="err"> </span> <span class="err"> </span> <span class="p">{</span>
<span class="err"> </span> <span class="err"> </span> <span class="err"> </span> <span class="err"> </span> <span class="err"> </span> <span class="err"> </span> <span class="k">case</span> <span class="s">"CSV"</span><span class="p">:</span>
<span class="err"> </span> <span class="err"> </span> <span class="err"> </span> <span class="err"> </span> <span class="err"> </span> <span class="err"> </span> <span class="err"> </span> <span class="err"> </span> <span class="k">return</span> <span class="k">new</span> <span class="nf">CSVReader</span><span class="p">();</span>

<span class="err"> </span> <span class="err"> </span> <span class="err"> </span> <span class="err"> </span> <span class="err"> </span> <span class="err"> </span> <span class="k">case</span> <span class="s">"XML"</span><span class="p">:</span>
<span class="err"> </span> <span class="err"> </span> <span class="err"> </span> <span class="err"> </span> <span class="err"> </span> <span class="err"> </span> <span class="err"> </span> <span class="err"> </span> <span class="k">return</span> <span class="k">new</span> <span class="nf">XMLReader</span><span class="p">();</span>

<span class="err"> </span> <span class="err"> </span> <span class="err"> </span> <span class="err"> </span> <span class="err"> </span> <span class="err"> </span> <span class="k">case</span> <span class="s">"JSON"</span><span class="p">:</span>
<span class="err"> </span> <span class="err"> </span> <span class="err"> </span> <span class="err"> </span> <span class="err"> </span> <span class="err"> </span> <span class="err"> </span> <span class="err"> </span> <span class="k">return</span> <span class="k">new</span> <span class="nf">JSONReader</span><span class="p">();</span>

<span class="err"> </span> <span class="err"> </span> <span class="err"> </span> <span class="err"> </span> <span class="err"> </span> <span class="err"> </span> <span class="k">case</span> <span class="s">"DBReader"</span><span class="p">:</span>
<span class="err"> </span> <span class="err"> </span> <span class="err"> </span> <span class="err"> </span> <span class="err"> </span> <span class="err"> </span> <span class="err"> </span> <span class="err"> </span> <span class="k">return</span> <span class="k">new</span> <span class="nf">DBReader</span><span class="p">();</span>

<span class="err"> </span> <span class="err"> </span> <span class="err"> </span> <span class="err"> </span> <span class="err"> </span> <span class="err"> </span> <span class="k">default</span><span class="p">:</span>
<span class="err"> </span> <span class="err"> </span> <span class="err"> </span> <span class="err"> </span> <span class="err"> </span> <span class="err"> </span> <span class="err"> </span> <span class="err"> </span> <span class="k">return</span> <span class="k">new</span> <span class="nf">AnyTextReader</span><span class="p">();</span>

<span class="err"> </span> <span class="err"> </span> <span class="err"> </span> <span class="err"> </span> <span class="p">}</span>

<span class="err"> </span> <span class="err"> </span> <span class="p">}</span>

<span class="p">}</span>

<span class="k">public</span> <span class="k">static</span> <span class="k">class</span> <span class="nc">DataReaderClient</span>
<span class="p">{</span>
<span class="err"> </span> <span class="err"> </span> <span class="err"> </span> <span class="err"> </span> <span class="k">private</span> <span class="k">static</span> <span class="k">void</span> <span class="nf">Main</span><span class="p">(</span><span class="kt">string</span><span class="p">[]</span> <span class="n">args</span><span class="p">)</span>
<span class="err"> </span> <span class="err"> </span> <span class="err"> </span> <span class="err"> </span> <span class="p">{</span> 
<span class="err"> </span> <span class="err"> </span> <span class="err"> </span> <span class="err"> </span> <span class="err"> </span> <span class="err"> </span> <span class="c1">//Example of object creation via a Factory Class</span>
<span class="err"> </span> <span class="err"> </span> <span class="err"> </span> <span class="err"> </span> <span class="err"> </span> <span class="err"> </span> <span class="n">IDataReader</span> <span class="n">dataReader</span> <span class="p">=</span> <span class="n">DataReaderFactory</span><span class="p">.</span><span class="nf">GetDataReader</span><span class="p">(</span><span class="s">"JSON"</span><span class="p">);</span> <span class="err"> </span> <span class="err"> </span> 

<span class="err"> </span> <span class="err"> </span> <span class="err"> </span> <span class="err"> </span> <span class="p">}</span>
<span class="p">}</span>

</code></pre>

</div>

<h2>
  
  
  <strong>The Enum Technique:</strong>
</h2>

<p>Another, and a comparatively terse technique for object creation, is the Enum method. We could achieve this using only the <code>DataReaderClient</code> class, an Enum, and an extremely generic and reusable extension method to read the data annotations in an enum, without needing to create a separate Factory class. </p>

<p>Let's see the implementation of that approach in the code below.<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight csharp"><code>
<span class="c1">//C#</span>

<span class="k">public</span> <span class="k">enum</span> <span class="n">DataReadersEnum</span>
<span class="p">{</span>
<span class="err"> </span> <span class="err"> </span> <span class="p">[</span><span class="nf">Description</span><span class="p">(</span><span class="s">"FactoryPatternEnumDemo.DataReaders.CsvReader"</span><span class="p">)]</span>
<span class="err"> </span> <span class="err"> </span> <span class="n">Csv</span><span class="p">,</span><span class="err"> </span> <span class="err">  </span>

<span class="err"> </span> <span class="err"> </span> <span class="p">[</span><span class="nf">Description</span><span class="p">(</span><span class="s">"FactoryPatternEnumDemo.DataReaders.JsonReader"</span><span class="p">)]</span>
<span class="err"> </span> <span class="err"> </span> <span class="n">Json</span><span class="p">,</span><span class="err"> </span> <span class="err">  </span>

<span class="err"> </span> <span class="err"> </span> <span class="p">[</span><span class="nf">Description</span><span class="p">(</span><span class="s">"FactoryPatternEnumDemo.DataReaders.XmlReader"</span><span class="p">)]</span>
<span class="err"> </span> <span class="err"> </span> <span class="n">Xml</span><span class="p">,</span><span class="err"> </span> <span class="err">  </span>

<span class="err"> </span> <span class="err"> </span> <span class="p">[</span><span class="nf">Description</span><span class="p">(</span><span class="s">"FactoryPatternEnumDemo.DataReaders.DbReader"</span><span class="p">)]</span>
<span class="err"> </span> <span class="err"> </span> <span class="n">Db</span><span class="p">,</span><span class="err"> </span>
<span class="err"> </span> <span class="err"> </span> <span class="err"> </span> <span class="err"> </span> <span class="p">[</span><span class="nf">Description</span><span class="p">(</span><span class="s">"FactoryPatternEnumDemo.DataReaders.AnyTextReader"</span><span class="p">)]</span>
<span class="err"> </span> <span class="err"> </span> <span class="n">AnyText</span> <span class="err">  </span>

<span class="p">}</span>

<span class="k">public</span> <span class="k">static</span> <span class="k">class</span> <span class="nc">DataReaderExtensions</span>
<span class="p">{</span>
<span class="err"> </span> <span class="err"> </span> <span class="k">public</span> <span class="k">static</span> <span class="kt">string</span> <span class="nf">GetClassName</span><span class="p">(</span><span class="k">this</span> <span class="n">DataReadersEnum</span> <span class="n">source</span><span class="p">)</span>
<span class="err"> </span> <span class="err"> </span> <span class="p">{</span>
<span class="err"> </span> <span class="err"> </span> <span class="err"> </span> <span class="err"> </span> <span class="n">FieldInfo</span> <span class="n">fi</span> <span class="p">=</span> <span class="n">source</span><span class="p">.</span><span class="nf">GetType</span><span class="p">()?.</span><span class="nf">GetField</span><span class="p">(</span><span class="n">source</span><span class="p">.</span><span class="nf">ToString</span><span class="p">());</span>

<span class="err"> </span> <span class="err"> </span> <span class="err"> </span> <span class="err"> </span> <span class="n">DescriptionAttribute</span><span class="p">[]</span> <span class="n">attributes</span> <span class="p">=</span> <span class="p">(</span><span class="n">DescriptionAttribute</span><span class="p">[])</span><span class="n">fi</span><span class="p">.</span><span class="nf">GetCustomAttributes</span><span class="p">(</span>
<span class="err"> </span> <span class="err"> </span> <span class="err"> </span> <span class="err"> </span> <span class="k">typeof</span><span class="p">(</span><span class="n">DescriptionAttribute</span><span class="p">),</span> <span class="k">false</span><span class="p">);</span>

<span class="err"> </span> <span class="err"> </span> <span class="err"> </span> <span class="err"> </span> <span class="k">if</span> <span class="p">(</span><span class="n">attributes</span> <span class="p">!=</span> <span class="k">null</span> <span class="p">&amp;&amp;</span> <span class="n">attributes</span><span class="p">.</span><span class="n">Length</span> <span class="p">&gt;</span> <span class="m">0</span><span class="p">)</span> <span class="k">return</span> <span class="n">attributes</span><span class="p">[</span><span class="m">0</span><span class="p">].</span><span class="n">Description</span><span class="p">;</span>

<span class="err"> </span> <span class="err"> </span> <span class="err"> </span> <span class="err"> </span> <span class="k">else</span> <span class="k">return</span> <span class="kt">string</span><span class="p">.</span><span class="n">Empty</span><span class="p">;</span>

<span class="err"> </span> <span class="err"> </span> <span class="p">}</span>
<span class="p">}</span>

<span class="k">public</span> <span class="k">class</span> <span class="nc">DataReaderClient</span>
<span class="p">{</span>
      <span class="k">private</span> <span class="k">static</span> <span class="k">void</span> <span class="nf">Main</span><span class="p">(</span><span class="kt">string</span><span class="p">[]</span> <span class="n">args</span><span class="p">)</span>
<span class="err"> </span> <span class="err"> </span> <span class="err"> </span> <span class="p">{</span> <span class="err"> </span>
          <span class="c1">//Object creation using the data annotations in the Enum</span>
<span class="err"> </span> <span class="err"> </span>       <span class="kt">string</span> <span class="n">dataReaderClassName</span> <span class="p">=</span><span class="n">DataReaders</span><span class="p">.</span><span class="n">JSON</span><span class="p">.</span><span class="nf">GetClassName</span><span class="p">();</span>

<span class="err"> </span> <span class="err"> </span> <span class="n">Type</span> <span class="n">type</span> <span class="p">=</span> <span class="n">Type</span><span class="p">.</span><span class="nf">GetType</span><span class="p">(</span><span class="n">dataReaderClassName</span><span class="p">);</span><span class="err"> </span>

<span class="err"> </span> <span class="err"> </span> <span class="n">IDataReader</span> <span class="n">dataReader2</span> <span class="p">=</span> <span class="p">(</span><span class="n">IDataReader</span><span class="p">)</span> <span class="n">Activator</span><span class="p">.</span><span class="nf">CreateInstance</span><span class="p">(</span><span class="n">type</span><span class="p">);</span>
    <span class="p">}</span>

<span class="p">}</span>

</code></pre>

</div>



<p>This method allows us to create objects dynamically using reflection. Let's compare the two techniques along with their pros and cons. </p>

<h3>
  
  
  <strong>The Factory Pattern: The Pros</strong>
</h3>

<p><strong>Decoupling the calling class from concrete implementations.</strong> By separating the logic of actually creating the classes from the rest of the flow of the code, we ensure that if we ever need to add more types <code>IDataReader</code> classes, we need not touch the main calling code logic. </p>

<p><strong>A SOLID-proof technique.</strong> This structure of code upholds the relevant SOLID principles - the  <em>Single Responsibility Principle</em> (the caller code is separated from object creation logic), the <em>Open/Closed Principle</em> (we only need to add more cases to the switch statement to extend without modifying the existing statements), and the <em>Dependency Injection Principle</em>. </p>

<p><strong>DRY? Check!</strong> The logic of object creation (the switch statement in our case) need not be repeated in multiple places where we need to create the <code>IDataReader</code> object(s). </p>

<p><strong>Centralized object creation</strong> means that we are easily able to keep track of the objects that are created at runtime if we need to. The program runs in a highly controlled manner, and we can ascertain the behavior of the code at compile time to ensure no there are no sudden surprises. </p>

<p>All this makes our code super easy to maintain and test. </p>

<h3>
  
  
  <strong>The Factory Pattern: The Cons</strong>
</h3>

<p><strong>We end up creating extra classes.</strong> There are some cases where this might be overkill. This can make the code harder to follow for someone new to the project.</p>

<p><strong>Dependency Injection can be more complicated</strong> and outdated than what is offered by modern DI frameworks (eg, Unity Containers), which often handle object creation and dependency management automatically.</p>

<p><strong>Can lead to a 'God' factory.</strong>  As an application grows, there's a risk of creating a single, massive factory class responsible for creating many different types of objects. This "God" factory becomes a central point of change and a maintenance nightmare. </p>

<h3>
  
  
  <strong>The Enum technique: The Pros</strong>
</h3>

<p><strong>Adherence to the Open/Closed Principle (in a different way):</strong> This approach can be seen as highly "open for extension." You can add a new reader type simply by adding a new entry to the enum and creating the corresponding class. There is no question of modifying the factory's switch statement, which means the core creation logic remains "closed for modification."</p>

<p><strong>Reduced Boilerplate for Simple Cases:</strong> When the object creation logic is a straightforward one-to-one mapping from an identifier to a class, this approach can seem less verbose than a full-fledged factory. You avoid the explicit case statement for each type in the factory.</p>

<h3>
  
  
  <strong>The Enum Technique: The Cons</strong>
</h3>

<p><strong>Loss of Type safety:</strong> Making sure the data annotation descriptions do correspond to an actual module in the assembly is a delicate operation and does require thorough testing. The IDE fails to automatically refactor any renaming operations. Any renaming operations can result in bugs that are not immediately obvious and can take some time to track down. In contrast, errors of a similar nature can be caught during compile time while using the factory class. </p>

<p><strong>Performance Issues:</strong> Anytime we use reflection in any scenario, we must exercise caution. Because reflection loops through all members of an assembly, we can inadvertently and often cause a quadratic runtime. The mantra is 'reflection is slow'. Although the reflection library was refactored in the .NET 8 release and is supposed to have made the operation speedier, when in doubt, stick to the aforementioned mantra. Especially when using it with any sort of collections. Much can be said about reflection; please be on the lookout for subsequent articles where we benchmark reflections from different frameworks and discuss use cases and best practices for it. </p>

<p><strong>Hard to debug:</strong> This is another side effect of using reflection. It's not straightforward to dry-run the code, especially for a less seasoned eye. Sometimes we are compelled to run the code to understand the flow of the code. We may inadvertently end up running some sensitive lines of code.  This can potentially escalate into a reputational nightmare for an organization, think of an intern accidentally sending an email to the production user base. </p>

<h3>
  
  
  <strong>In Conclusion</strong>
</h3>

<p>You might have rightly concluded by now that the factory pattern is the recommended approach over dynamic object creation using Enums. The latter is actually an anti-pattern and should not be used unless there is a dire situation. You can safely and practically read that as 'never'.</p>

<p>The working demo of the code included in this article can be obtained at <a href="https://github.com/ketkiambekar/FactoryPatternEnumDemo" rel="noopener noreferrer">this link</a>.</p>

<p>Got thoughts? I'd love to hear from you in the comment section below.</p>

<p>Happy coding!</p>

<p> © Ketki Ambekar 2025</p>

