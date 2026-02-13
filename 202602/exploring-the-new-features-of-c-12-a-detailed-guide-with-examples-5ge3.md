---
Title: Exploring the New Features of C# 12: A Detailed Guide with Examples
Description: 
Author: Ali Suleyman TOPUZ
Date: 2026-02-13T22:02:39.000Z
Robots: noindex,nofollow
Template: index
---
<p>The release of C# 12, bundled with .NET 8, has introduced several exciting features that enhance the language’s expressiveness and performance. Here’s an in-depth look at these features, complete with coding examples for each.</p>

<h4>
  
  
  <strong>Collection Expressions</strong>
</h4>

<p>This new syntax allows for the creation of collections like arrays, lists, and spans in a more concise way. You can use the spread operator .. to inline elements from one collection into another, streamlining the process of combining multiple collections</p>

<h4>
  
  
  <strong>Primary Constructors</strong>
</h4>

<p>A significant addition to C# 12, primary constructors enable more concise and clear syntax by allowing constructors to be declared inline with the type declaration. This feature is applicable to various types including class, struct, record class, and record struct, and is particularly useful for initializing member fields or properties and facilitating dependency injection</p>

<h4>
  
  
  <strong>Inline Arrays</strong>
</h4>

<p>Inline arrays are struct-based, fixed-length arrays that provide a performance boost, particularly in scenarios involving arrays of structures. This feature enables you to work with memory buffers more efficiently, without needing unsafe code</p>

<h4>
  
  
  <strong>Default Lambda Parameters</strong>
</h4>

<p>C# 12 introduces the ability to define default values for parameters in lambda expressions. This feature brings added flexibility and expressiveness to lambda expressions, making them more concise and adaptable</p>

<h4>
  
  
  <strong>ref readonly Parameters</strong>
</h4>

<p>Building upon the ref parameters feature, C# 12 introduces ref readonly parameters, which allow passing parameters by reference in a read-only context. This is particularly useful for methods that need the memory location of an argument without modifying it</p>

<h4>
  
  
  <strong>Alias Any Type with **</strong> using**
</h4>

<p>The using directive has been extended to allow aliases for any type, not just named types. This means you can create semantic aliases for complex types like tuples, arrays, pointers, or other unsafe types​​.</p>

<h4>
  
  
  <strong>Experimental Attribute</strong>
</h4>

<p>This new attribute can be used to mark types, methods, or assemblies as experimental. The compiler issues a warning when accessing methods or types annotated with this attribute, making it clearer which parts of your code are experimental and potentially subject to change​​.</p>

<h4>
  
  
  <strong>Interceptors</strong>
</h4>

<p>An experimental feature in C# 12, interceptors allow for the redirection of method calls, which can be used for optimization. However, it’s important to note that this feature is still in the experimental phase and not recommended for production use yet</p>

<p>These features collectively enhance C# by making the language more efficient, expressive, and flexible, while also introducing new capabilities for performance optimization and code clarity. The community response to these updates has been largely positive, reflecting the ongoing evolution and improvement of the C# language.</p>

<h4>
  
  
  Let’s go deeper with some coding examples
</h4>

<h4>
  
  
  Collection Expressions
</h4>

<p>Collection expressions simplify the creation of collections like arrays, lists, and spans. They offer a more concise way to combine elements from multiple collections.<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight plaintext"><code>int[] someNumbers = [1, 2, 3];
List&lt;string&gt; fruits = ["apple", "banana", "cherry"];
Span&lt;char&gt; span = ['h', 'e', 'l', 'l', 'o'];
</code></pre>

</div>



<h4>
  
  
  Primary Constructors
</h4>

<p>Primary constructors streamline the declaration of constructors inline with the type, making the code more concise and readable.<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight plaintext"><code>public class Person(string firstName, string lastName)
{
    public string FirstName { get; } = firstName;
    public string LastName { get; } = lastName;
}
</code></pre>

</div>



<h4>
  
  
  Inline Arrays
</h4>

<p>Inline arrays are fixed-size, struct-based arrays that improve performance by reducing allocations and copying.<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight plaintext"><code>[System.Runtime.CompilerServices.InlineArray(5)]
public struct FixedSizeBuffer
{
    private int _element0;
    // Usage
    var buffer = new FixedSizeBuffer();
    for (int i = 0; i &lt; 5; i++) buffer[i] = i;
}
</code></pre>

</div>



<h4>
  
  
  Default Lambda Parameters
</h4>

<p>C# 12 allows specifying default values for lambda expression parameters, offering more flexibility.<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight plaintext"><code>var addNumbers = (int x, int y = 1) =&gt; x + y;
Console.WriteLine(addNumbers(5)); // Output: 6
Console.WriteLine(addNumbers(5, 5)); // Output: 10
</code></pre>

</div>



<h4>
  
  
  ref readonly Parameters
</h4>

<p>ref readonly parameters enable passing arguments by reference in a read-only context, enhancing performance and safety.<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight plaintext"><code>void Display(in int x, in int y)
{
    Console.WriteLine($"x: {x}, y: {y}");
}
</code></pre>

</div>



<h4>
  
  
  Alias Any Type with using
</h4>

<p>Extending the using directive to alias any type, including complex types like tuples and arrays.<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight plaintext"><code>using Matrix = int[][];
Matrix matrix = [[1, 2, 3], [4, 5, 6]];
</code></pre>

</div>



<h4>
  
  
  Experimental Attribute
</h4>

<p>The ExperimentalAttribute marks features as experimental, alerting developers of potential future changes.<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight plaintext"><code>[Experimental]
public void ExperimentalMethod() { }
</code></pre>

</div>



<h4>
  
  
  Conclusion
</h4>

<p>C# 12 introduces a range of features that enhance the language’s functionality, improve performance, and offer new programming paradigms. While most of these features are immediately beneficial, some, like interceptors, are still experimental and should be used with caution. As with any new technology, it’s essential to understand these features thoroughly and test them in safe environments before integrating them into production code. For more detailed examples and in-depth explanations, refer to the official <a href="https://learn.microsoft.com/en-us/dotnet/csharp/whats-new/csharp-12" rel="noopener noreferrer">C# documentation</a> and resources such as Microsoft’s Learn platform, as well as community forums and tech blogs.</p>

