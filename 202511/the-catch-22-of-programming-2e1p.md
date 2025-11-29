---
Title: The Catch-22 of programming
Description: 
Author: Graham Trott
Date: 2025-11-29T21:06:33.000Z
Robots: noindex,nofollow
Template: index
---
<p>The Catch-22:</p>

<ol>
<li>Only skilled programmers can guide AI to create good software.</li>
<li>It takes a long time to gain the skills to become a coding expert, starting with a long period of simple coding at the entry level.</li>
<li>AI is taking away the jobs of entry-level programmers.</li>
</ol>

<p>In the early days of computing, the route to becoming a programmer was straightforward. You just did it. Training courses were initially nonexistent, so we had to learn from textbooks and data sheets. The ones who had the motivation to stick with it were the ones who became professional programmers.</p>

<p>After that, in some ways it got a lot easier. Information became more plentiful, and with the arrival of first email then the World Wide Web, there were people you could ask and articles you could read without having to order a book and wait for it to arrive. Courses became available for pretty much anything.</p>

<p>In other ways, though, it didn’t get easier. Software became more complex, and a myriad of tools and libraries became regarded as essential things to know. Job listings started to fill up with acronyms and left many applicants feeling their efforts had all been for nothing, as the prize continued to remain just out of reach.</p>

<p>Then along came AI, and suddenly it seemed a programmer's life became simpler again. No more need to remember every function in a library; just ask the AI to find the one you need. But just as we started to get used to this new way of working, these upstart AI systems started to go for our jobs. And now we’re told that pretty much all programming will soon be done entirely by AI engines.</p>

<p>Or so it seems. However, if you actually ask AI to write software, you’ll soon discover that what comes back might be classifiable as a prototype but is likely to be totally unfit for delivery as a finished product. The only way to ensure a good product is to closely monitor the entire process from start to finish, enforce good working practices and test everything as it gets written. The only person who can do that is a competent programmer.</p>

<p>So we have a Catch-22 situation. To get a good product you need to be an expert to drive AI in the right way. But because AI is taking over all the mundane programming work, few will ever spend the time and effort to become expert. Who in their right mind spends hours churning out boilerplate code when AI will spit it out in seconds? But if you don’t do it yourself you’ll never really learn how to, and you won’t be able to spot the mistakes made by AI. Likewise, you’ll never really know what any given programming language is capable of, and whether AI is making best use of it.</p>

<p>This is a situation that is only just developing but is set to get pretty serious quite quickly as the supply of top-level programmers dries up. AI-generated products will frequently never develop past the prototype stage, while many will remain incomplete and generally unfit for purpose.</p>

<h2>
  
  
  What are our options?
</h2>

<p>At the heart of the problem is the way we communicate with computers. There are basically 2 ways:</p>

<ol>
<li><p>Using traditional high-level languages. This means code written in C++, Java, Python or any other mainstream programming language. As explained above, the number of people who will become skilled at this work is set to decrease as AI takes over the lower levels of coding.</p></li>
<li><p>Using natural language, e.g. English. This is the new way that has come into its own with the arrival of AI. However, program code must be unambiguous and human language is usually anything but. This is the enemy of AI, as it will usually work with what it thinks you mean rather than question a possible ambiguity, so the result may be subtly or even grossly different to that which you wanted.</p></li>
</ol>

<p>Coding can be categorised in several ways, but one that I find useful is to distinguish between tools and products. Tools include operating systems and code libraries, and products are the things that are built with the tools. The thing that for me stands out like a lighthouse beacon is that coders tend to use the same language for both, when in fact they often require very different forms of language. A computer algorithm expressed in English is clumsy and hard to follow, and a set of business rules that makes perfect sense in English becomes impenetrable when converted to Python or Java. Real-world requirements are often long chains of ‘if-then-else’ logic, with a frequent need for ‘goto’ to jump from one place to another, which is of course a complete no-no in the world of structured programming. Maybe what we need is a different form of language.</p>

<p>However much automation is provided, there will always be skilled toolmakers. Although it has limited storage capacity, the human brain is capable of insights no machine can match. So why waste such talent by asking such people to simply mind AI while it does all the hard work? Right now, the answer is that there is no separate class of coder with the ability to express clearly what is needed and guide the AI to create it, without having taken the long route to becoming a professional user of conventional languages and tools.</p>

<p>Some people are better than others at expressing clear logic. Many become lawyers, and they use a highly constrained form of English that most of us find difficult to understand, but which has that vital property of being unambiguous. Maybe we need something akin to this when we want to guide AI?</p>

<h2>
  
  
  An example
</h2>

<p>To help clarify where I'm heading, here's an example. Let's suppose the task is to create a new app with a windowing user interface. The first thing is to decide where it will run, so I'll say on any desktop OS. This probably suggests Python should be the language to code for, and I'll constrain it a bit further by specifying PySide to handle the graphics. The app will mimic a traditional web page, with a central data area, left and right side bars, a title bar and a footer. There's a menu bar with a single menu having 2 items, About and Exit, each with simple behaviours. It looks like this:</p>

<p><a href="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fxl0crefwy0hey4z29qy5.png" class="article-body-image-wrapper"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fxl0crefwy0hey4z29qy5.png" alt="An example" width="638" height="507"></a></p>

<p>Here's an AI prompt to create the app as shown above:</p>

<blockquote>
<p>Using PySide6, create a main window with the following attributes:<br>
Location: 100,100<br>
Size: 640,480<br>
Title: My New App<br>
The window contains 3 rows.<br>
The first row is a panel taking 20% of the window height, containing a label with the text ‘Header’. <br>
The second row has a height that is what is left after the other 2 rows are created. It has 3 elements: 1 - a panel taking 20% of the window width, with a label having the text ‘Left’. 2 - a panel taking whatever width is left after the other 2 items are created, with a label having the text ‘Center’. 3 - a panel taking 15% of the window width with a label having the text ‘Right’. <br>
The third row is a panel taking 15% of the window height, containing a label with the text ‘Footer’. <br>
The window has a menu bar with a single ‘File’menu having 2 items:<br>
An ‘About’ item that when clicked shows a simple dialog. <br>
An ‘Exit’ item that terminates the app when clicked.</p>
</blockquote>

<p>This is a fairly clumsy way to program. It's largely unstructured, and when changes are needed whole sections may have to be rewritten. Most importantly, to avoid ambiguity it requires discipline in the choice of wording; the kind of discipline that only comes with experience. Since there’s no “compiler” to parse the text and spot problems with language or syntax, errors can easily slip through unnoticed.</p>

<p>So let’s see if we can use a simplified form of English to describe this window. The syntax I’ve chosen is arbitrary. To be clear, when I refer to English I mean text that is mostly if not completely confined to English words, not mathematical symbols. And it’s terse; this is a recipe, not a poem. Let’s call this the pseudocode version.<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight plaintext"><code>! Variables used in the script
window Window
layout Content
panel Panel
label Header
panel Panel2
label Left
label Center
label Right
label Footer

! Create a window
create Window
   at 100 100
   size 640 480
   title `My New App`

! Add the main content; a series of nested panels and labels
create Content type rows in Window

! The first row
create Panel height 20% in Content
create Header text `Header` in Panel

! The second row
create Panel type columns in Panel
create Panel2 in Panel
create Left width 20% text `Left` in Panel2
create Panel2 in Panel
create Center text `Centre` in Panel2
create Panel2 in Panel
create Right width 15% text `Right` in Panel2

! The third row
create Panel height 15% in Content
create Footer text `Footer` in Panel

show Window
</code></pre>

</div>



<p>This is 38 lines, compared to roughly 100 for the Python code that resulted from the AI query.</p>

<p>This version has a big advantage over the natural language version. It’s compilable to a form that will run directly. This means it can be used as the product, rather than being just a specification of how to build the product. That’s demonstrated by the last line, which will run the script on a suitably-equipped system. Compilers and runtimes of this kind are not hard to write; in fact, something very close to this already exists in the Python world. See <a href="https://github.com/easycoder/easycoder-py" rel="noopener noreferrer">https://github.com/easycoder/easycoder-py</a>.</p>

<h2>
  
  
  How does this work with AI?
</h2>

<p>Can AI use pseudocode instead of natural language to generate a fully native application in whatever underlying language you prefer? To answer this question, I did the following:</p>

<p>First, I submitted the original natural-language request to DeepSeek, which generated Python code. The image above is a screenshot of the running code.</p>

<p>I then asked DeepSeek if it could recognise the pseudocode version. It came back with the confirmation that the two versions were in fact identical, and that if a suitable dictionary were built it could easily handle pseudocode instead of natural language. It described the approach as </p>

<blockquote>
<p>a declarative UI specification system where:<br>
Pseudocode = Human-readable UI description<br>
Keyword dictionary = Mapping to actual UI components and properties<br>
Generator = Interprets pseudocode → creates actual UI code</p>
</blockquote>

<p>then went on to comment that</p>

<blockquote>
<p>This is similar to how many modern UI frameworks work (React, Vue components) but with an even more abstracted, human-readable syntax.</p>
</blockquote>

<p>and offered the following:</p>

<blockquote>
<p>Benefits of This Approach:<br>
Rapid Prototyping - Write UI structure in simple English-like syntax<br>
Consistency - Enforced patterns through keyword definitions<br>
Platform Independence - Same pseudocode could generate PySide6, Tkinter, web UI, etc.<br>
Non-technical Friendly - Designers can specify layouts without coding<br>
Version Control Friendly - Pseudocode is clean and diff-able</p>
</blockquote>

<h2>
  
  
  Other Benefits
</h2>

<p>In the example above, many assumptions have been made. The way items are positioned and sized in their parent containers, the foreground and background colors, the fonts used for text, border widths and so on. In the browser world these are all handled by stylesheets, and in fact PySide handles most common stylesheet definitions. In any practical application, some form of stylesheet will have to be defined, holding global values and values that are specific to a class of on-screen widget. Doing all of this using natural language is painful to say the least, but a variant of the same pseudocode can be used to create JSON stylesheet files that can be read by the compiler. The language will need a vocabulary that allows the user to define which styles belong where, but this is not a hard task. DeepSeek offered to design such a system for me, but I’m not yet ready quite yet to dive further into this. </p>

<p>A second benefit is that by using pseudocode, products can be developed quickly with or without AI, or by combining both. If people start to use pseudocode, AI will gradually be able to provide more assistance in creating applications by examining examples of those already created.</p>

<p>The script uses a lot of variables to represent screen elements. Some of these are reused but others hold values that are of interest for the lifetime of the application. The natural language version might name such elements, but the syntax to refer to them will be clumsy and impede understanding. Having them available as code elements allows them to be accessed at any time simply by name.</p>

<p>These scripts can be compiled on any computer and then run; hence the last line above - <code>show Window</code>. The entire script can be run from the command line, directly from the source text. The performance may not be sparkling, but as a means of catching errors it's unbeatable as corrections can be made instantly and the run started again. Typical compilation times are in the tens of milliseconds, even for quite large scripts. The runtime even includes a visual debugger that allows breakpoints to be added and variables examined.</p>

<p>Since performance is rarely an issue with prototypes, the compiler and runtime for these scripts can check very aggressively for potential coding errors during the development phase. A typical example is the wrong type of variable being used, or an attempt to use a variable that has no value assigned to it. Once the prototype is seen to run without errors the script can be submitted to AI for recoding in Python, JavaScript, C++ or whatever is best for the target environment. This maximises performance.</p>

<p>The biggest benefit takes me back to the start of this article. Pseudocode allows very large chunks of functionality to be represented by a fairly concise English syntax. New functionality, created by experts in their fields, gets added as extra pseudocode vocabulary, which is easily understood by AI and by people familiar with the domain in which they are working. It is fairly easy to take a given piece of functionality and create a new language feature for it, and it does not usually require any great knowledge of how that functionality is coded. Take Google Maps, for example, which has a remarkably simple API for such a complex and comprehensive product. We will still need highly skilled programmers to create such products, but pseudocode unlocks the door to making such products accessible to those with far more limited skills, to create a new ecosystem of products. </p>

<p>And let’s not forget the problem of maintenance. Having advanced applications coded in pseudocode that is understood both by AI and by human programmers makes future maintenance far simpler than requiring every maintenance engineer to know the details of every programming language and its libraries.</p>

<h2>
  
  
  Summary
</h2>

<p>The adoption of a suitable pseudocode for describing a wide range of applications will reduce the need for highly skilled coders and create a demand instead for logical thinkers who understand problem domains and can translate requirements into a form that can run as a prototype then be taken by AI to be recoded as necessary for the target environment. Since more people fall into the second category than the first, this should help avoid a severe shortage of people with advanced skills once all the entry-level jobs have been taken by AI.</p>

