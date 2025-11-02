---
Title: JetBrains ReSharper for Visual Studio
Description: 
Author: Karen Payne
Date: 2025-11-02T21:49:09.000Z
Robots: noindex,nofollow
Template: index
---
<h2>
  
  
  Introduction
</h2>

<p><a href="https://www.jetbrains.com/resharper/" rel="noopener noreferrer">JetBrains ReSharper</a> is an extension for Microsoft Visual Studio that provides numerous features to enhance developer productivity, surpassing that of developers who do not use ReSharper.</p>

<p>The purpose here is to introduce developers to several useful features that many developers just starting out with ReSharper may not be aware of and can immediately benefit from. </p>

<p>The author feels that ReSharper is a must-have addition to all developers' toolbox/</p>

<p>Also, check out <a href="https://blog.jetbrains.com/dotnet/2025/10/06/9-resharper-features-every-dotnet-developer-should-know/" rel="noopener noreferrer">9 ReSharper Features Every .NET Developer Should Know</a>.</p>

<h2>
  
  
  Keyboard shortcuts
</h2>

<p>Once a developer learns which are the most important features for them, yet may not be used often.</p>

<p>Create a GitHub Markdown file, as shown below (see the sample in the source repository), where the last column contains a link to the command documentation.</p>

<p><a href="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fdmzb18e4jft81pk3f3fi.png" class="article-body-image-wrapper"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fdmzb18e4jft81pk3f3fi.png" alt="markdown file sample" width="800" height="660"></a></p>

<h3>
  
  
  Present in Visual Studio
</h3>

<p>Under Tools menu, External Tools create a new command (requires VS Code to be installed).</p>

<ul>
<li>Title: Resharper keyboard shortcuts</li>
<li>Command: c:\users\USERNAME\AppData\Local\Programs\Microsoft VS Code\Code.exe</li>
<li>Arguments: The name of the file, for instance resharperShortcuts.md</li>
<li>Initial directory: Location of the file under arguments</li>
</ul>

<h2>
  
  
  Writing documentation
</h2>

<p>With a paid subscription to <a href="https://www.jetbrains.com/ai/" rel="noopener noreferrer">AI Assistant</a>, a developer never need to worry about having excellent documentation. Yes, Visual Studio can generate help but no where as done with AI Assistant.</p>

<h2>
  
  
  Extract Interface
</h2>

<p>📚 <a href="https://www.jetbrains.com/help/resharper/Refactorings__Extract_Interface.html" rel="noopener noreferrer">Documentation</a></p>

<p>This refactoring helps create a new interface based on a selected type. ReSharper suggests choosing members to be transferred to the new interface. After extraction, the original type is updated to implement the new interface.</p>

<p>As shown in the video, each interface is created in its own file, which is in the Models folder. The next step is to create a folder named Interfaces and drag the interfaces into the Interface folder, which will trigger Visual Studio to ask to update each Interfaces namespace.</p>

<p><iframe width="710" height="399" src="https://www.youtube.com/embed/LgXzkTEpSdw">
</iframe>
</p>

<h2>
  
  
  Adjust Namespaces
</h2>

<p>📚 <a href="https://www.jetbrains.com/help/resharper/Refactorings__Adjust_Namespaces.html" rel="noopener noreferrer">Documentation</a></p>

<p>This command is a bulk fix that helps you synchronize namespaces with folder structure in any scope, which can be as large as your whole solution.</p>

<p>This is a feature I used often. An example is selecting several classes from an instance of Visual Studio and dragging them to another instance of Visual Studio.</p>

<p>Once the above operation is complete, use 'adjust namespaces' to ensure the files match the current project namespaces.</p>

<p>In this example, the files are in a folder called Models. </p>

<ul>
<li>Right-click on the folder.</li>
<li>Select Refactor</li>
<li>Select Adjust Namespaces.</li>
</ul>

<p><a href="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2F9j4gswefk6mvlbxm5hco.png" class="article-body-image-wrapper"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2F9j4gswefk6mvlbxm5hco.png" alt="shows menu" width="752" height="131"></a></p>

<p>The following dialog appears; click the <code>Next &gt;</code> button for ReSharper to adjust the namespace of each selected file.</p>

<p><a href="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fl95u5seriwuj2mzmoeel.png" class="article-body-image-wrapper"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fl95u5seriwuj2mzmoeel.png" alt="the dialog" width="800" height="401"></a></p>

<h2>
  
  
  Convert Static to Extension Method refactoring
</h2>

<p>📚 <a href="https://www.jetbrains.com/help/resharper/Refactorings__Convert_Static_to_Extension_Method.html" rel="noopener noreferrer">Documentation</a></p>

<p>This refactoring helps you convert a static method to an extension method, provided that the static method:</p>

<ul>
<li>Has at least one parameter.</li>
<li>Resides in a non-generic, non-nested static class</li>
</ul>

<p>This conversion is a time saver, as shown below, here, the shortcut is CTRL + Q, Q</p>

<p><iframe width="710" height="399" src="https://www.youtube.com/embed/UXfbfIFYzaI">
</iframe>
</p>

<h2>
  
  
  Code templates
</h2>

<p>📚 <a href="https://www.jetbrains.com/resharper/features/code_templates.html" rel="noopener noreferrer">Documentation</a></p>

<p>Code templates that help write common code constructs faster, surround existing code. Below is one of several template types.</p>

<h3>
  
  
  Postfix templates
</h3>

<p>Postfix templates help you transform expressions that you have already typed without jumping backwards. </p>

<p><strong>Example</strong></p>

<p>A developer wants to use a foreach loop on a list with a variable named 'people'.</p>

<ul>
<li>Type people.<strong>f</strong>
</li>
<li>Select <code>foreach</code>
</li>
</ul>

<p>A foreach is generated with the cursor on a suggested variable name, which the developer can use or change.</p>

<p><a href="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2F494gzthg7nhdoz44ymeo.png" class="article-body-image-wrapper"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2F494gzthg7nhdoz44ymeo.png" alt="using foreach" width="444" height="584"></a></p>

<h2>
  
  
  PostFix templates
</h2>

<p>📚 <a href="https://www.jetbrains.com/help/resharper/Postfix_Templates.html" rel="noopener noreferrer">Documentation</a></p>

<p>Postfix templates help you transform expressions that you have already typed without jumping backwards — just type a dot after an expression and pick a template from the completion list.</p>

<p>For examples and source code see <a href="https://dev.to/karenpayneoregon/learn-resharper-postfix-and-source-templates-32lo">Learn Resharper PostFix and Source Templates</a></p>

<h2>
  
  
  Regular expression to GeneratedRegex attribute
</h2>

<p>ReSharper automatically detects opportunities to refactor traditional regex usage into the modern [<a href="https://learn.microsoft.com/en-us/dotnet/standard/base-types/regular-expression-source-generators#source-generation" rel="noopener noreferrer">GeneratedRegex</a>] attribute pattern.</p>

<h3>
  
  
  Benefits of GeneratedRegex
</h3>

<ul>
<li>
<strong>Performance</strong>: The regex is compiled and optimized at compile time, leading to faster startup times and lower runtime overhead compared to JIT-compiling the regex at runtime.</li>
<li>
<strong>Efficiency</strong>: It avoids the need for manual caching of Regex objects, as the generated code provides a singleton instance.</li>
</ul>

<p><strong>Example</strong></p>

<p>Given the following code, using a regular expression.<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight csharp"><code><span class="k">public</span> <span class="k">class</span> <span class="nc">Validator</span>
<span class="p">{</span>
    <span class="k">public</span> <span class="kt">bool</span> <span class="nf">IsEmailValid</span><span class="p">(</span><span class="kt">string</span> <span class="n">email</span><span class="p">)</span> 
        <span class="p">=&gt;</span> <span class="n">Regex</span><span class="p">.</span><span class="nf">IsMatch</span><span class="p">(</span><span class="n">email</span><span class="p">,</span> <span class="s">@"^[^@\s]+@[^@\s]+\.[^@\s]+$"</span><span class="p">);</span>
<span class="p">}</span>
</code></pre>

</div>



<p>Place the text cursor in the regular expressions and a light bulb appears with the option Convert to <code>GeneratedRegexAttruibte</code> as shown below.</p>

<p><a href="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fc4w3jgdeucj0k9yoiv23.png" class="article-body-image-wrapper"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fc4w3jgdeucj0k9yoiv23.png" alt="shows convert dialog" width="800" height="349"></a></p>

<p>Accept the convert. During the conversion:</p>

<ul>
<li>The class becomes a partial class</li>
<li>A second dialog is presented with recommendations for the name of the GeneratedRegex.</li>
</ul>

<p>After the conversion.<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight csharp"><code><span class="k">public</span> <span class="k">partial</span> <span class="k">class</span> <span class="nc">Validator</span> <span class="c1">// Note: The class must be partial</span>
<span class="p">{</span>
    <span class="k">public</span> <span class="kt">bool</span> <span class="nf">IsEmailValid</span><span class="p">(</span><span class="kt">string</span> <span class="n">email</span><span class="p">)</span> 
        <span class="p">=&gt;</span> <span class="nf">EmailRegex</span><span class="p">().</span><span class="nf">IsMatch</span><span class="p">(</span><span class="n">email</span><span class="p">);</span>

    <span class="c1">// ReSharper generates this attribute and partial method definition</span>
    <span class="p">[</span><span class="nf">GeneratedRegex</span><span class="p">(</span><span class="s">@"^[^@\s]+@[^@\s]+\.[^@\s]+$"</span><span class="p">)]</span>
    <span class="k">private</span> <span class="k">static</span> <span class="k">partial</span> <span class="n">Regex</span> <span class="nf">EmailRegex</span><span class="p">();</span>
<span class="p">}</span>
</code></pre>

</div>



<p>💡 See the above and a more complex code sample in the project ToGeneratedRegexSample.</p>

<h2>
  
  
  INotifyPropertyChanged support
</h2>

<p>📚 <a href="https://www.jetbrains.com/help/resharper/Coding_Assistance__INotifyPropertyChanged_Support.html" rel="noopener noreferrer">Documentation</a></p>

<p>When a developer wants to use INotifyPropertyChanged, add : INotifyPropertyChanged after the class name, place the text cursor on INotifyPropertyChanged followed by selecting the first option under the red bulb.</p>

<p><a href="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fvvg73qt9drucvt321zq9.png" class="article-body-image-wrapper"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fvvg73qt9drucvt321zq9.png" alt="shows menu option" width="800" height="376"></a></p>

<p>Afterwards.</p>

<p><a href="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2F30xemlkfc9iz9ikrtme2.png" class="article-body-image-wrapper"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2F30xemlkfc9iz9ikrtme2.png" alt="Now INotifyPropertyChanged  has been implemented" width="800" height="544"></a></p>

<p>Next, place the text cursor on a property, select the yellow 💡 which ReSharper provides several options to implement property change.</p>

<p>Click a top-level menu item to implement for the current property while clicking a sub menu option while implement property change for all class properties.</p>

<p><a href="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fkp0lyod1ep917nqg6f15.png" class="article-body-image-wrapper"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fkp0lyod1ep917nqg6f15.png" alt="shows menu options" width="800" height="282"></a></p>

<p>In this case the selected option was <code>To Property with 'SetField'</code>.</p>

<p><a href="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2F72j8uakc2x482k5811e7.png" class="article-body-image-wrapper"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2F72j8uakc2x482k5811e7.png" alt="shows each property using SetField" width="800" height="767"></a></p>

<h3>
  
  
  Using AI Assistant
</h3>

<p>With the class open, try the following prompt:</p>

<p><em>Add INotifyPropertyChanged to Person class using SetField</em></p>

<p>Once satisfied, use the copy button in the top right corner for the response, followed by overwriting the current class.</p>

<h2>
  
  
  Move Types into Matching Files
</h2>

<p>📚 <a href="https://www.jetbrains.com/help/resharper/Refactorings__Move_Types_into_Matching_Files.html" rel="noopener noreferrer">Documentation</a></p>

<p>This feature is useful, for instance, when a developer creates several classes in a single file with the intention of later extracting each class to its own file.</p>

<p>The reason for creating multiple classes in one file is to avoid having to switch between each class in Visual Studio. The author does this because she can also create the classes faster than manually creating each class file. When finished, ReSharper will move each class to a new file.</p>

<p><strong>Example</strong></p>

<p>Given the following with several classes in a single file, Person.cs.<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight csharp"><code><span class="k">public</span> <span class="k">class</span> <span class="nc">Person</span>
<span class="p">{</span>
    <span class="k">public</span> <span class="kt">int</span> <span class="n">Id</span> <span class="p">{</span> <span class="k">get</span><span class="p">;</span> <span class="k">set</span><span class="p">;</span> <span class="p">}</span>
    <span class="k">public</span> <span class="kt">bool</span> <span class="n">Active</span> <span class="p">{</span> <span class="k">get</span><span class="p">;</span> <span class="k">set</span><span class="p">;</span> <span class="p">}</span>
    <span class="k">public</span> <span class="n">required</span> <span class="kt">string</span> <span class="n">FirstName</span> <span class="p">{</span> <span class="k">get</span><span class="p">;</span> <span class="k">set</span><span class="p">;</span> <span class="p">}</span>
    <span class="k">public</span> <span class="n">required</span> <span class="kt">string</span> <span class="n">LastName</span> <span class="p">{</span> <span class="k">get</span><span class="p">;</span> <span class="k">set</span><span class="p">;</span> <span class="p">}</span>
    <span class="k">public</span> <span class="n">required</span> <span class="n">Gender</span> <span class="n">Gender</span> <span class="p">{</span> <span class="k">get</span><span class="p">;</span> <span class="k">set</span><span class="p">;</span> <span class="p">}</span>
    <span class="k">public</span> <span class="n">required</span> <span class="n">Address</span> <span class="n">Address</span> <span class="p">{</span> <span class="k">get</span><span class="p">;</span> <span class="k">set</span><span class="p">;</span> <span class="p">}</span>
<span class="p">}</span>
<span class="k">public</span> <span class="k">class</span> <span class="nc">Address</span>
<span class="p">{</span>
    <span class="k">public</span> <span class="kt">int</span> <span class="n">Id</span> <span class="p">{</span> <span class="k">get</span><span class="p">;</span> <span class="k">set</span><span class="p">;</span> <span class="p">}</span>
    <span class="k">public</span> <span class="n">required</span> <span class="kt">string</span> <span class="n">Street</span> <span class="p">{</span> <span class="k">get</span><span class="p">;</span> <span class="k">set</span><span class="p">;</span> <span class="p">}</span>
    <span class="k">public</span> <span class="n">required</span> <span class="kt">string</span> <span class="n">City</span> <span class="p">{</span> <span class="k">get</span><span class="p">;</span> <span class="k">set</span><span class="p">;</span> <span class="p">}</span>
    <span class="k">public</span> <span class="n">required</span> <span class="kt">string</span> <span class="n">State</span> <span class="p">{</span> <span class="k">get</span><span class="p">;</span> <span class="k">set</span><span class="p">;</span> <span class="p">}</span>
<span class="p">}</span>
<span class="k">public</span> <span class="k">enum</span> <span class="n">Gender</span>
<span class="p">{</span>
    <span class="n">Male</span><span class="p">,</span>
    <span class="n">Female</span><span class="p">,</span>
    <span class="n">Other</span>
<span class="p">}</span>
</code></pre>

</div>



<ul>
<li>Right-click on the file in Solution Explorer</li>
<li>Select Refactor</li>
<li>Select <code>Move Type to Matching Files...</code>
</li>
</ul>

<p><a href="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fvrzi1kbbgaasbtdg816d.png" class="article-body-image-wrapper"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fvrzi1kbbgaasbtdg816d.png" alt="shows menu item Move Type to Matching Files..." width="800" height="722"></a></p>

<p>A dialog is presented with options. Click Next button to complete the move each class to new files.</p>

<p><a href="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fv586sh5qmc47qid3alqe.png" class="article-body-image-wrapper"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fv586sh5qmc47qid3alqe.png" alt="options dialog" width="800" height="286"></a></p>

<p>💡 Try it in project MoveTypesIntoMatchingFiles\Models\Person.cs</p>

<p><a href="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fvkdcod7xilseuyb2gyot.png" class="article-body-image-wrapper"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fvkdcod7xilseuyb2gyot.png" alt="Shows before and after" width="363" height="340"></a></p>

<h2>
  
  
  Convert Anonymous to Named Type refactoring
</h2>

<p>📚 <a href="https://www.jetbrains.com/help/resharper/Refactorings__Convert_Anonymous_to_Named_Type.html" rel="noopener noreferrer">Documentation</a></p>

<p>Anonymous types are very convenient when you need to process a result of a query locally. However, if you need to pass the result around your program or if there are several queries that return the similar name/value parts, you may probably need a named type.</p>

<h3>
  
  
  Example
</h3>

<p><a href="https://github.com/karenpayneoregon/resharper2025-samples/tree/master/AnonymousToNameTypeSample" class="ltag_cta ltag_cta--branded" rel="noopener noreferrer">Source code</a>
</p>

<p>In the following method, the variable duplicates can only be used within the method.<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight csharp"><code><span class="k">public</span> <span class="k">static</span> <span class="k">void</span> <span class="nf">GetDuplicateActiveMembers</span><span class="p">(</span><span class="n">IEnumerable</span><span class="p">&lt;</span><span class="n">Member</span><span class="p">&gt;</span> <span class="n">list</span><span class="p">)</span>
<span class="p">{</span>
    <span class="kt">var</span> <span class="n">duplicates</span> <span class="p">=</span> <span class="n">list</span>
        <span class="p">.</span><span class="nf">GroupBy</span><span class="p">(</span><span class="n">member</span> <span class="p">=&gt;</span> <span class="k">new</span> <span class="p">{</span> <span class="n">member</span><span class="p">.</span><span class="n">FirstName</span><span class="p">,</span> <span class="n">member</span><span class="p">.</span><span class="n">SurName</span><span class="p">,</span> <span class="n">member</span><span class="p">.</span><span class="n">Active</span> <span class="p">})</span>
        <span class="p">.</span><span class="nf">Where</span><span class="p">(</span><span class="k">group</span> <span class="p">=&gt;</span> <span class="k">group</span><span class="p">.</span><span class="n">Key</span><span class="p">.</span><span class="n">Active</span> <span class="p">&amp;&amp;</span> <span class="k">group</span><span class="p">.</span><span class="nf">Count</span><span class="p">()</span> <span class="p">&gt;</span> <span class="m">1</span><span class="p">)</span>
        <span class="p">.</span><span class="nf">Select</span><span class="p">(</span><span class="k">group</span> <span class="p">=&gt;</span> <span class="k">new</span>
        <span class="p">{</span>
            <span class="n">FullName</span> <span class="p">=</span> <span class="s">$"</span><span class="p">{</span><span class="k">group</span><span class="p">.</span><span class="n">Key</span><span class="p">.</span><span class="n">FirstName</span><span class="p">}</span><span class="s"> </span><span class="p">{</span><span class="k">group</span><span class="p">.</span><span class="n">Key</span><span class="p">.</span><span class="n">SurName</span><span class="p">}</span><span class="s">"</span><span class="p">,</span>
            <span class="n">Members</span> <span class="p">=</span> <span class="k">group</span><span class="p">.</span><span class="nf">ToList</span><span class="p">()</span>
        <span class="p">})</span>
        <span class="p">.</span><span class="nf">ToList</span><span class="p">();</span>
<span class="p">}</span>
</code></pre>

</div>



<p>To allow duplicates to be returned, place the cursor on <code>new</code> and press the shortcut key assigned to ReSharper_Anonymous2Declared command.</p>

<p><a href="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fh1qolnl2htznaekqj6tg.png" class="article-body-image-wrapper"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fh1qolnl2htznaekqj6tg.png" alt="shows keyboard shortcut" width="749" height="492"></a></p>

<p>The following dialog appears; select a name, scope, and other options as needed. Click the Next button and ReSharper generates code.</p>

<p><a href="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2F1ljwhlhcp1qxrna87waf.png" class="article-body-image-wrapper"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2F1ljwhlhcp1qxrna87waf.png" alt="dialog" width="800" height="481"></a></p>

<p>Modify the return type as shown below.<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight csharp"><code>    <span class="k">public</span> <span class="k">static</span> <span class="n">List</span><span class="p">&lt;</span><span class="n">GroupMember</span><span class="p">&gt;</span> <span class="nf">GetDuplicateActiveMembers</span><span class="p">(</span><span class="n">IEnumerable</span><span class="p">&lt;</span><span class="n">Member</span><span class="p">&gt;</span> <span class="n">list</span><span class="p">)</span> <span class="p">=&gt;</span>
        <span class="n">list</span>
            <span class="p">.</span><span class="nf">GroupBy</span><span class="p">(</span><span class="n">member</span> <span class="p">=&gt;</span> <span class="p">(</span><span class="n">Name</span><span class="p">:</span> <span class="n">member</span><span class="p">.</span><span class="n">FirstName</span><span class="p">,</span> <span class="n">member</span><span class="p">.</span><span class="n">SurName</span><span class="p">,</span> <span class="n">member</span><span class="p">.</span><span class="n">Active</span><span class="p">))</span>
            <span class="p">.</span><span class="nf">Where</span><span class="p">(</span><span class="k">group</span> <span class="p">=&gt;</span>
            <span class="p">{</span>
                <span class="k">if</span> <span class="p">(!</span><span class="k">group</span><span class="p">.</span><span class="n">Key</span><span class="p">.</span><span class="n">Active</span><span class="p">)</span> <span class="k">return</span> <span class="k">false</span><span class="p">;</span>
                <span class="k">return</span> <span class="k">group</span><span class="p">.</span><span class="nf">Count</span><span class="p">()</span> <span class="p">&gt;</span> <span class="m">1</span><span class="p">;</span>

            <span class="p">})</span>
            <span class="p">.</span><span class="nf">Select</span><span class="p">(</span><span class="n">item</span> <span class="p">=&gt;</span>
                <span class="k">new</span> <span class="nf">GroupMember</span><span class="p">(</span><span class="s">$"</span><span class="p">{</span><span class="n">item</span><span class="p">.</span><span class="n">Key</span><span class="p">.</span><span class="n">Name</span><span class="p">}</span><span class="s"> </span><span class="p">{</span><span class="n">item</span><span class="p">.</span><span class="n">Key</span><span class="p">.</span><span class="n">SurName</span><span class="p">}</span><span class="s">"</span><span class="p">,</span> <span class="p">[..</span> <span class="n">item</span><span class="p">]))</span>
            <span class="p">.</span><span class="nf">ToList</span><span class="p">();</span>
</code></pre>

</div>



<p>The convert to name type can also be accessed from ReSharper menu.</p>

<p><a href="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fk0x56nm147unltlya0l2.png" class="article-body-image-wrapper"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fk0x56nm147unltlya0l2.png" alt="menu option" width="800" height="373"></a></p>

<h2>
  
  
  Import missing namespaces
</h2>

<p>📚 <a href="https://www.jetbrains.com/help/resharper/Coding_Assistance__Importing_Namespaces.html" rel="noopener noreferrer">Documentation</a></p>

<p>When you use types whose namespaces have not been imported in the file, ReSharper helps you locate these types and add the missing namespace import directives.</p>

<h3>
  
  
  Example 1
</h3>

<p>A developer obtains the following code. Without ReSharper, Visual Studio will ask for missing using statements and provide recommendations.</p>

<blockquote>
<p><strong>Note</strong><br>
The reason for using a large example is for showing how well auto Import missing namespaces works.<br>
</p>
</blockquote>

<div class="highlight js-code-highlight">
<pre class="highlight csharp"><code><span class="k">public</span> <span class="k">class</span> <span class="nc">Info</span>
<span class="p">{</span>
    <span class="k">public</span> <span class="k">readonly</span> <span class="k">record</span> <span class="nc">struct</span> <span class="nf">CallerDetails</span><span class="p">(</span><span class="kt">string</span><span class="p">?</span> <span class="n">AssemblyName</span><span class="p">,</span> <span class="kt">string</span><span class="p">?</span> <span class="n">AssemblyVersion</span><span class="p">,</span>
        <span class="kt">string</span><span class="p">?</span> <span class="n">TargetFramework</span><span class="p">,</span> <span class="kt">string</span><span class="p">?</span> <span class="n">TypeName</span><span class="p">,</span> <span class="kt">string</span><span class="p">?</span> <span class="n">MethodName</span><span class="p">,</span> <span class="kt">string</span><span class="p">?</span> <span class="n">FilePath</span><span class="p">,</span> <span class="kt">int</span> <span class="n">LineNumber</span><span class="p">);</span>

    <span class="p">[</span><span class="nf">MethodImpl</span><span class="p">(</span><span class="n">MethodImplOptions</span><span class="p">.</span><span class="n">NoInlining</span><span class="p">)]</span>
    <span class="k">private</span> <span class="k">static</span> <span class="n">CallerDetails</span> <span class="nf">BuildCallerDetails</span><span class="p">(</span><span class="kt">string</span><span class="p">?</span> <span class="n">memberName</span><span class="p">,</span> <span class="kt">string</span><span class="p">?</span> <span class="n">filePath</span><span class="p">,</span> <span class="kt">int</span> <span class="n">lineNumber</span><span class="p">)</span>
    <span class="p">{</span>
        <span class="kt">var</span> <span class="n">callingAsm</span> <span class="p">=</span> <span class="n">Assembly</span><span class="p">.</span><span class="nf">GetCallingAssembly</span><span class="p">();</span>
        <span class="kt">string</span><span class="p">?</span> <span class="n">typeName</span> <span class="p">=</span> <span class="k">null</span><span class="p">;</span>

        <span class="k">try</span>
        <span class="p">{</span>
            <span class="kt">var</span> <span class="n">st</span> <span class="p">=</span> <span class="k">new</span> <span class="nf">StackTrace</span><span class="p">(</span><span class="n">skipFrames</span><span class="p">:</span> <span class="m">1</span><span class="p">,</span> <span class="n">fNeedFileInfo</span><span class="p">:</span> <span class="k">false</span><span class="p">);</span>
            <span class="kt">var</span> <span class="n">frame</span> <span class="p">=</span> <span class="n">st</span><span class="p">.</span><span class="nf">GetFrame</span><span class="p">(</span><span class="m">0</span><span class="p">);</span>
            <span class="n">typeName</span> <span class="p">=</span> <span class="n">frame</span><span class="p">?.</span><span class="nf">GetMethod</span><span class="p">()?.</span><span class="n">DeclaringType</span><span class="p">?.</span><span class="n">FullName</span><span class="p">;</span>
        <span class="p">}</span>
        <span class="k">catch</span>
        <span class="p">{</span>
            <span class="c1">// Best-effort; leave typeName null if anything goes sideways.</span>
        <span class="p">}</span>

        <span class="kt">var</span> <span class="n">asmName</span> <span class="p">=</span> <span class="n">callingAsm</span><span class="p">.</span><span class="nf">GetName</span><span class="p">();</span>
        <span class="kt">var</span> <span class="n">framework</span> <span class="p">=</span> <span class="n">callingAsm</span><span class="p">.</span><span class="n">GetCustomAttribute</span><span class="p">&lt;</span><span class="n">TargetFrameworkAttribute</span><span class="p">&gt;()?.</span><span class="n">FrameworkName</span><span class="p">;</span>

        <span class="k">return</span> <span class="k">new</span> <span class="nf">CallerDetails</span><span class="p">(</span>
            <span class="n">AssemblyName</span><span class="p">:</span> <span class="n">asmName</span><span class="p">?.</span><span class="n">Name</span><span class="p">,</span>
            <span class="n">AssemblyVersion</span><span class="p">:</span> <span class="n">asmName</span><span class="p">?.</span><span class="n">Version</span><span class="p">?.</span><span class="nf">ToString</span><span class="p">(),</span>
            <span class="n">TargetFramework</span><span class="p">:</span> <span class="n">framework</span><span class="p">,</span>
            <span class="n">TypeName</span><span class="p">:</span> <span class="n">typeName</span><span class="p">,</span>
            <span class="n">MethodName</span><span class="p">:</span> <span class="n">memberName</span><span class="p">,</span>
            <span class="n">FilePath</span><span class="p">:</span> <span class="n">filePath</span><span class="p">,</span>
            <span class="n">LineNumber</span><span class="p">:</span> <span class="n">lineNumber</span><span class="p">);</span>
    <span class="p">}</span>

    <span class="p">[</span><span class="nf">MethodImpl</span><span class="p">(</span><span class="n">MethodImplOptions</span><span class="p">.</span><span class="n">NoInlining</span><span class="p">)]</span>
    <span class="k">public</span> <span class="k">static</span> <span class="kt">string</span> <span class="nf">GetCopyright</span><span class="p">()</span>
    <span class="p">{</span>
        <span class="kt">var</span> <span class="n">asm</span> <span class="p">=</span> <span class="n">Assembly</span><span class="p">.</span><span class="nf">GetCallingAssembly</span><span class="p">();</span>
        <span class="kt">var</span> <span class="n">attr</span> <span class="p">=</span> <span class="n">asm</span><span class="p">.</span><span class="n">GetCustomAttribute</span><span class="p">&lt;</span><span class="n">AssemblyCopyrightAttribute</span><span class="p">&gt;();</span>
        <span class="k">return</span> <span class="n">attr</span><span class="p">?.</span><span class="n">Copyright</span> <span class="p">??</span> <span class="s">"No copyright information found."</span><span class="p">;</span>
    <span class="p">}</span>

    <span class="p">[</span><span class="nf">MethodImpl</span><span class="p">(</span><span class="n">MethodImplOptions</span><span class="p">.</span><span class="n">NoInlining</span><span class="p">)]</span>
    <span class="k">public</span> <span class="k">static</span> <span class="kt">string</span> <span class="nf">GetCopyright</span><span class="p">(</span><span class="k">out</span> <span class="n">CallerDetails</span> <span class="n">caller</span><span class="p">,</span> <span class="p">[</span><span class="n">CallerMemberName</span><span class="p">]</span> <span class="kt">string</span><span class="p">?</span> <span class="n">memberName</span> <span class="p">=</span> <span class="k">null</span><span class="p">,</span> <span class="p">[</span><span class="n">CallerFilePath</span><span class="p">]</span> <span class="kt">string</span><span class="p">?</span> <span class="n">filePath</span> <span class="p">=</span> <span class="k">null</span><span class="p">,</span> <span class="p">[</span><span class="n">CallerLineNumber</span><span class="p">]</span> <span class="kt">int</span> <span class="n">lineNumber</span> <span class="p">=</span> <span class="m">0</span><span class="p">)</span>
    <span class="p">{</span>
        <span class="n">caller</span> <span class="p">=</span> <span class="nf">BuildCallerDetails</span><span class="p">(</span><span class="n">memberName</span><span class="p">,</span> <span class="n">filePath</span><span class="p">,</span> <span class="n">lineNumber</span><span class="p">);</span>
        <span class="k">return</span> <span class="nf">GetCopyright</span><span class="p">();</span>
    <span class="p">}</span>

    <span class="p">[</span><span class="nf">MethodImpl</span><span class="p">(</span><span class="n">MethodImplOptions</span><span class="p">.</span><span class="n">NoInlining</span><span class="p">)]</span>
    <span class="k">public</span> <span class="k">static</span> <span class="kt">string</span> <span class="nf">GetCompany</span><span class="p">()</span>
    <span class="p">{</span>
        <span class="kt">var</span> <span class="n">asm</span> <span class="p">=</span> <span class="n">Assembly</span><span class="p">.</span><span class="nf">GetCallingAssembly</span><span class="p">();</span>
        <span class="kt">var</span> <span class="n">attr</span> <span class="p">=</span> <span class="n">asm</span><span class="p">.</span><span class="n">GetCustomAttribute</span><span class="p">&lt;</span><span class="n">AssemblyCompanyAttribute</span><span class="p">&gt;();</span>
        <span class="k">return</span> <span class="n">attr</span><span class="p">?.</span><span class="n">Company</span> <span class="p">??</span> <span class="s">"No company information found."</span><span class="p">;</span>
    <span class="p">}</span>

    <span class="p">[</span><span class="nf">MethodImpl</span><span class="p">(</span><span class="n">MethodImplOptions</span><span class="p">.</span><span class="n">NoInlining</span><span class="p">)]</span>
    <span class="k">public</span> <span class="k">static</span> <span class="kt">string</span> <span class="nf">GetCompany</span><span class="p">(</span><span class="k">out</span> <span class="n">CallerDetails</span> <span class="n">caller</span><span class="p">,</span> <span class="p">[</span><span class="n">CallerMemberName</span><span class="p">]</span> <span class="kt">string</span><span class="p">?</span> <span class="n">memberName</span> <span class="p">=</span> <span class="k">null</span><span class="p">,</span> <span class="p">[</span><span class="n">CallerFilePath</span><span class="p">]</span> <span class="kt">string</span><span class="p">?</span> <span class="n">filePath</span> <span class="p">=</span> <span class="k">null</span><span class="p">,</span> <span class="p">[</span><span class="n">CallerLineNumber</span><span class="p">]</span> <span class="kt">int</span> <span class="n">lineNumber</span> <span class="p">=</span> <span class="m">0</span><span class="p">)</span>
    <span class="p">{</span>
        <span class="n">caller</span> <span class="p">=</span> <span class="nf">BuildCallerDetails</span><span class="p">(</span><span class="n">memberName</span><span class="p">,</span> <span class="n">filePath</span><span class="p">,</span> <span class="n">lineNumber</span><span class="p">);</span>
        <span class="k">return</span> <span class="nf">GetCompany</span><span class="p">();</span>
    <span class="p">}</span>
    <span class="p">[</span><span class="nf">MethodImpl</span><span class="p">(</span><span class="n">MethodImplOptions</span><span class="p">.</span><span class="n">NoInlining</span><span class="p">)]</span>
    <span class="k">public</span> <span class="k">static</span> <span class="kt">string</span> <span class="nf">GetProduct</span><span class="p">()</span>
    <span class="p">{</span>
        <span class="kt">var</span> <span class="n">asm</span> <span class="p">=</span> <span class="n">Assembly</span><span class="p">.</span><span class="nf">GetCallingAssembly</span><span class="p">();</span>
        <span class="kt">var</span> <span class="n">attr</span> <span class="p">=</span> <span class="n">asm</span><span class="p">.</span><span class="n">GetCustomAttribute</span><span class="p">&lt;</span><span class="n">AssemblyProductAttribute</span><span class="p">&gt;();</span>
        <span class="k">return</span> <span class="n">attr</span><span class="p">?.</span><span class="n">Product</span> <span class="p">??</span> <span class="s">"No product information found."</span><span class="p">;</span>
    <span class="p">}</span>

    <span class="p">[</span><span class="nf">MethodImpl</span><span class="p">(</span><span class="n">MethodImplOptions</span><span class="p">.</span><span class="n">NoInlining</span><span class="p">)]</span>
    <span class="k">public</span> <span class="k">static</span> <span class="kt">string</span> <span class="nf">GetProduct</span><span class="p">(</span><span class="k">out</span> <span class="n">CallerDetails</span> <span class="n">caller</span><span class="p">,</span> <span class="p">[</span><span class="n">CallerMemberName</span><span class="p">]</span> <span class="kt">string</span><span class="p">?</span> <span class="n">memberName</span> <span class="p">=</span> <span class="k">null</span><span class="p">,</span> <span class="p">[</span><span class="n">CallerFilePath</span><span class="p">]</span> <span class="kt">string</span><span class="p">?</span> <span class="n">filePath</span> <span class="p">=</span> <span class="k">null</span><span class="p">,</span> <span class="p">[</span><span class="n">CallerLineNumber</span><span class="p">]</span> <span class="kt">int</span> <span class="n">lineNumber</span> <span class="p">=</span> <span class="m">0</span><span class="p">)</span>
    <span class="p">{</span>
        <span class="n">caller</span> <span class="p">=</span> <span class="nf">BuildCallerDetails</span><span class="p">(</span><span class="n">memberName</span><span class="p">,</span> <span class="n">filePath</span><span class="p">,</span> <span class="n">lineNumber</span><span class="p">);</span>
        <span class="k">return</span> <span class="nf">GetProduct</span><span class="p">();</span>
    <span class="p">}</span>

    <span class="p">[</span><span class="nf">MethodImpl</span><span class="p">(</span><span class="n">MethodImplOptions</span><span class="p">.</span><span class="n">NoInlining</span><span class="p">)]</span>
    <span class="k">public</span> <span class="k">static</span> <span class="n">Version</span> <span class="nf">GetVersion</span><span class="p">()</span>
    <span class="p">{</span>
        <span class="kt">var</span> <span class="n">asm</span> <span class="p">=</span> <span class="n">Assembly</span><span class="p">.</span><span class="nf">GetCallingAssembly</span><span class="p">();</span>
        <span class="k">return</span> <span class="n">asm</span><span class="p">.</span><span class="nf">GetName</span><span class="p">().</span><span class="n">Version</span> <span class="p">??</span> <span class="k">new</span> <span class="nf">Version</span><span class="p">(</span><span class="m">1</span><span class="p">,</span> <span class="m">0</span><span class="p">,</span> <span class="m">0</span><span class="p">,</span> <span class="m">0</span><span class="p">);</span>
    <span class="p">}</span>

    <span class="p">[</span><span class="nf">MethodImpl</span><span class="p">(</span><span class="n">MethodImplOptions</span><span class="p">.</span><span class="n">NoInlining</span><span class="p">)]</span>
    <span class="k">public</span> <span class="k">static</span> <span class="n">Version</span> <span class="nf">GetVersion</span><span class="p">(</span><span class="k">out</span> <span class="n">CallerDetails</span> <span class="n">caller</span><span class="p">,</span> <span class="p">[</span><span class="n">CallerMemberName</span><span class="p">]</span> <span class="kt">string</span><span class="p">?</span> <span class="n">memberName</span> <span class="p">=</span> <span class="k">null</span><span class="p">,</span> <span class="p">[</span><span class="n">CallerFilePath</span><span class="p">]</span> <span class="kt">string</span><span class="p">?</span> <span class="n">filePath</span> <span class="p">=</span> <span class="k">null</span><span class="p">,</span> <span class="p">[</span><span class="n">CallerLineNumber</span><span class="p">]</span> <span class="kt">int</span> <span class="n">lineNumber</span> <span class="p">=</span> <span class="m">0</span><span class="p">)</span>
    <span class="p">{</span>
        <span class="n">caller</span> <span class="p">=</span> <span class="nf">BuildCallerDetails</span><span class="p">(</span><span class="n">memberName</span><span class="p">,</span> <span class="n">filePath</span><span class="p">,</span> <span class="n">lineNumber</span><span class="p">);</span>
        <span class="k">return</span> <span class="nf">GetVersion</span><span class="p">();</span>
    <span class="p">}</span>
<span class="p">}</span>
</code></pre>

</div>



<p>With ReSharper, after pasting the code into, in this case, Info.cs, the following statements are added, which allow the code to compile.<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight csharp"><code><span class="k">using</span> <span class="nn">System.Diagnostics</span><span class="p">;</span>
<span class="k">using</span> <span class="nn">System.Reflection</span><span class="p">;</span>
<span class="k">using</span> <span class="nn">System.Runtime.CompilerServices</span><span class="p">;</span>
<span class="k">using</span> <span class="nn">System.Runtime.Versioning</span><span class="p">;</span>
</code></pre>

</div>



<h2>
  
  
  Example 2
</h2>

<p>In some cases, auto import is not available, and the developer will be prompted as shown below.</p>

<p><a href="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fg96chbrb6n3at0g81veq.png" class="article-body-image-wrapper"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fg96chbrb6n3at0g81veq.png" alt="prompting for recommendation for using statement" width="496" height="82"></a></p>




<h2>
  
  
  Summary
</h2>

<p>ReSharper has more features than the average developer will ever need, while this is true. The best way to determine what is best for you is to take the time to read the great documentation and create a markdown file as described at the start of this article.</p>

<p>There are <a href="https://www.jetbrains.com/resharper/buy/?section=commercial&amp;billing=yearly&amp;special-offers=discounts" rel="noopener noreferrer">three subscriptions</a></p>

<ul>
<li>ReSharper which will fit every developer's needs</li>
<li>dotUltimate which besides ReSharper has several helpful tools and JetBrains AI Pro</li>
<li>All Products Pack which besides ReSharper has 18 tools.</li>
</ul>

<p>How to decide? When unsure, select ReSharper or start a 30-day trial. Download a product and follow the instructions provided.</p>

<h3>
  
  
  Author comment
</h3>

<p>The author has been using ReSharper since before 2015.</p>

<p>There are times when she has three instances of Microsoft Visual Studio open at once and ReSharper never hinders performance.</p>




<p><a href="https://github.com/karenpayneoregon/resharper2025-samples" class="ltag_cta ltag_cta--branded" rel="noopener noreferrer">Source code</a>
</p>

