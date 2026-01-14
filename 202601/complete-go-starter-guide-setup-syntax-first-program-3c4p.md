---
Title: Complete Go Starter Guide: Setup, Syntax & First Program
Description: 
Author: RS
Date: 2026-01-14T21:13:12.000Z
Robots: noindex,nofollow
Template: index
---
<p>Have you ever felt like you want to code and learn a new programming language, but there are so many languages to choose from: Java, Python, Rust, Go? I researched, and Go caught my interest, as highlighted by the JetBrains report <a href="https://www.jetbrains.com/lp/devecosystem-data-playground/" rel="noopener noreferrer">here</a>. Go was used by approximately 4.8 million users. Many companies have widely accepted Go for backend development and libraries due to its high performance. There are many advantages of Go, which we will cover in future articles. A bit of background on Go is that it was created at Google, and its first public release was in 2009. Go was created to address issues with large-scale systems at Google, such as slow compilation, complexity, and poor concurrency support. C++ and Java do solve some issues, but the learning curve is much steeper than with developer-friendly, readable languages like Python. The focus was on creating a simple language that would inherit many advantages of languages like C++. By the end of this article, we will have Go installed and will have written our first program.</p>




<h1>
  
  
  Installation &amp; Verification
</h1>

<h2>
  
  
  Step 1: Download &amp; install
</h2>

<p>Head to <a href="https://go.dev/dl/" rel="noopener noreferrer">https://go.dev/dl/</a> and grab the installer for your OS. Double-click and install once downloaded. The installer will set up Go and configure essential paths. There are multiple ways to get Go; you can use package managers like Homebrew for macOS.</p>

<h2>
  
  
  Step 2: Verify Installation
</h2>

<p>Run the following command in your terminal (bash or command prompt):<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight plaintext"><code>go version
&gt;&gt;&gt; go version go1.25.5 darwin/arm64

</code></pre>

</div>



<p>You should see output like go1.21.5. This confirms that Go is installed and ready to use.</p>




<h1>
  
  
  What Are Go Modules?
</h1>

<p>A Go module is a collection of Go files that includes a go.mod file. This file tracks your project’s name and its dependencies. Previously, this was done via a special workspace (GOPATH), but we don’t need that anymore.</p>

<p>Let’s create one: Open your terminal and run the following commands:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight plaintext"><code>mkdir my-go-app
cd my-go-app
# go mod init &lt;the-go-module-name-you-like&gt;
go mod init my-first-go-project
</code></pre>

</div>



<p>The command creates a go.mod file. This is now the root of your Go project. All the code and commands will run relative to this folder going forward.</p>

<p><a href="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Ft7aukj1lc5tel5drodql.png" class="article-body-image-wrapper"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Ft7aukj1lc5tel5drodql.png" alt="Go mod init" width="550" height="149"></a></p>

<p><a href="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fecr8az4g8larg6u7dl0s.png" class="article-body-image-wrapper"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fecr8az4g8larg6u7dl0s.png" alt="Directory" width="800" height="119"></a></p>




<h1>
  
  
  Nostalgic “Hello, World!”
</h1>

<p>Inside the folder, create a file named main.go. Open it, add all the following code, and save the file:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight plaintext"><code>package main // Every executable program starts with package main

import "fmt" 
// Unlike Python, where methods like print, round are readily available
// in Go functions are always in a package, and we need fmt to use print text.

func main(){ // The main function is where the program begins
 fmt.Println("Hello, World!") // This line prints the text
}
</code></pre>

</div>






<h1>
  
  
  Running &amp; Building: go run vs go build
</h1>

<h2>
  
  
  go run — For Quick Testing
</h2>

<p>Why we use it: To quickly check if your code works or not; no extra files are created.<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight plaintext"><code>go run main.go
</code></pre>

</div>



<p><a href="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fpsmob9odv11xkzokpg9r.png" class="article-body-image-wrapper"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fpsmob9odv11xkzokpg9r.png" alt="Go Run Command" width="376" height="194"></a></p>

<h2>
  
  
  go build — To Create an Executable
</h2>

<ol>
<li><p>The command will compile your code and create a standalone binary file, which you can share and run anytime, even without having Go installed.</p></li>
<li><p>After execution, you will see an executable created in the folder, which you can run and share.</p></li>
<li><p>Why we use it: If we want to distribute a program or run it later.</p></li>
</ol>

<p><code>go build main.go</code></p>




<h1>
  
  
  Current Project Layout
</h1>

<p>You can execute the following command to see what the project looks like. You can add more .go files and play around.<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight plaintext"><code>ls -la # use `dir` for Windows

go-app/
├── go.mod       # Your module definition file
├── main.go      # Your source code
└── main.exe     # Your compiled program (if you built it)
</code></pre>

</div>






<h1>
  
  
  Variables and Basic Types
</h1>

<p>Variables are containers used for storing data. Go is a statically-typed language, which means you must define the type of data that the variable will hold.</p>

<h2>
  
  
  Basic Types
</h2>

<ol>
<li><p>string: Text (e.g., “Hello”)</p></li>
<li><p>int: Whole number (e.g., 10, -10)</p></li>
<li><p>float64: Decimal numbers (e.g., 3.14)</p></li>
<li><p>bool: True or False</p></li>
</ol>

<h2>
  
  
  Three Ways to Declare Variables
</h2>

<h4>
  
  
  Full Declaration (Verbose) — Declare the variable name, type, and optionally assign a value
</h4>



<div class="highlight js-code-highlight">
<pre class="highlight plaintext"><code>var name string = "Tom Cruise"
var age int
age = 25
</code></pre>

</div>



<h4>
  
  
  Type Inference (Shorter) — Go will figure out the type based on the value assigned.
</h4>



<div class="highlight js-code-highlight">
<pre class="highlight plaintext"><code>var name = "Tom" // Go will figure out it's a string
var age = 30
</code></pre>

</div>



<h4>
  
  
  Short Declaration (Mostly Inside Functions) — Use the := operator. It declares and initialises the variable in a single step.
</h4>



<div class="highlight js-code-highlight">
<pre class="highlight plaintext"><code>package main

import "fmt"

func main(){
    name := "Tom"   // String
    age := 25       // Integer
    isActor := true // Boolean
    height := 5.5   // Float64

    fmt.Println(name, age, isActor, height)
}
</code></pre>

</div>






<h1>
  
  
  Putting it All Together
</h1>

<p>Here’s a sample main.go with all these concepts.<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight plaintext"><code>package main

import "fmt"

func main(){
    var name string = "Tom"  // Full Declaration
    var age = 25            // Type Inference
    isActor := true        // Short Declaration (inside main function)
    height := 5.5 // Float64

    fmt.Println(name, age, isActor, height) // using variables

    age = 26 // changing variable
    fmt.Println(name, age, isActor, height) // using variables
}
</code></pre>

</div>






<h1>
  
  
  What’s Next?
</h1>

<p>We have installed Go, created a project with a module, run our first program, and learned about basic data types and variables.</p>

<p>What you can do:</p>

<p>Try playing with the main.go file.</p>

<ol>
<li><p>Change the message in the print statement.</p></li>
<li><p>Create new variables and try storing new values in them.</p></li>
<li><p>Create an int variable and try storing a string in it. Comment on what happens below 🙂</p></li>
<li><p>Use go build to create an executable and send it to someone, and ask them to execute it.</p></li>
</ol>

<p>In subsequent articles, we will learn more about the syntax, including conditionals and loops.</p>

<p>If you are stuck or have any questions, drop a comment with the output, and I will try my best to answer them all.</p>

