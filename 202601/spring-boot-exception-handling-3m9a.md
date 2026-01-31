---
Title: SPRING BOOT EXCEPTION HANDLING
Description: 
Author: Er. Bhupendra
Date: 2026-01-31T21:27:39.000Z
Robots: noindex,nofollow
Template: index
---
<div class="highlight js-code-highlight">
<pre class="highlight markdown"><code><span class="gh"># Java &amp; Spring Boot Exception Handling Notes</span>
<span class="p">
---
</span>
<span class="gu">## 1. What is Exception?</span>
Exception = unwanted situation that breaks normal flow of program.

Goal of exception handling:
<span class="p">-</span> Program crash na ho
<span class="p">-</span> Error ko control karna
<span class="p">-</span> Proper message dena
<span class="p">
---
</span>
<span class="gu">## 2. Java Exception Hierarchy</span>

</code></pre>

</div>



<p>Throwable<br>
├── Error<br>
└── Exception<br>
├── RuntimeException (Unchecked)<br>
└── Other Exceptions (Checked)<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight plaintext"><code>
---

## 3. Types of Exception (Java Level)

### (A) Checked Exception
- Compile time pe check hoti hai
- Handle karna compulsory hai (`try-catch` OR `throws`)
- Extends: `Exception` (but not RuntimeException)

Examples:
- IOException
- SQLException
- ClassNotFoundException
- FileNotFoundException
- InterruptedException

---

### (B) Unchecked Exception
- Runtime pe aati hai
- Handle karna compulsory nahi
- Extends: `RuntimeException`

Examples:
- NullPointerException
- ArithmeticException
- ArrayIndexOutOfBoundsException
- NumberFormatException
- IllegalArgumentException

---

### (C) Error
- JVM / system level problem
- Normally handle nahi karte
- Custom Error banana bad practice

Examples:
- OutOfMemoryError
- StackOverflowError

---

## 4. Checked vs Unchecked Rule

</code></pre>

</div>



<p>extends Exception        → Checked Exception<br>
extends RuntimeException → Unchecked Exception<br>
extends Error           → Error (avoid)<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight plaintext"><code>
---

## 5. Can we extend other exceptions?

Technically YES:
- Can extend IOException, NullPointerException, etc.

But Best Practice:
- Extend only:
  - Exception (for checked)
  - RuntimeException (for unchecked)

Avoid:
- Extending NullPointerException, IOException, SQLException etc.

---

## 6. try-catch vs throws

### try-catch:
- Exception yahin handle hoti hai

```

java
try {
   FileReader fr = new FileReader("a.txt");
} catch (IOException e) {
   System.out.println("File error");
}


</code></pre>

</div>




<h3>
  
  
  throws:
</h3>

<ul>
<li>Caller ko responsibility deta hai</li>
</ul>

<div class="highlight js-code-highlight">
<pre class="highlight plaintext"><code>
java
public void readFile() throws IOException {
   FileReader fr = new FileReader("a.txt");
}


</code></pre>

</div>

<p>Rule:<br>
Checked Exception = try-catch OR throws (one compulsory)</p>




<h2>
  
  
  7. throw vs throws
</h2>

<ul>
<li>
<code>throw</code> → manually exception throw karna</li>
<li>
<code>throws</code> → method signature me likhna</li>
</ul>

<div class="highlight js-code-highlight">
<pre class="highlight plaintext"><code>
java
throw new IOException("File not found");


</code></pre>

</div>
<div class="highlight js-code-highlight">
<pre class="highlight plaintext"><code>
java
public void read() throws IOException { }


</code></pre>

</div>




<h2>
  
  
  8. Common Exception Meaning
</h2>

<h3>
  
  
  ClassNotFoundException
</h3>

<ul>
<li>Jab runtime pe class nahi milti</li>
</ul>

<h3>
  
  
  FileNotFoundException
</h3>

<ul>
<li>Jab file nahi milti / path galat</li>
</ul>

<h3>
  
  
  IOException
</h3>

<ul>
<li>File / network / stream problem</li>
</ul>

<h3>
  
  
  SQLException
</h3>

<ul>
<li>SQL database problem (MySQL, PostgreSQL, Oracle)</li>
</ul>

<p>Not used in MongoDB.</p>

<p>MongoDB uses:</p>

<ul>
<li>MongoException</li>
<li>DuplicateKeyException</li>
</ul>




<h3>
  
  
  InterruptedException
</h3>

<ul>
<li>Jab thread sleep/wait/join me ho</li>
<li>Aur interrupt ho jaye</li>
</ul>




<h2>
  
  
  9. Custom Exception
</h2>

<h3>
  
  
  Checked Custom Exception
</h3>

<div class="highlight js-code-highlight">
<pre class="highlight plaintext"><code>
java
class MyCheckedException extends Exception {
}


</code></pre>

</div>
<h3>
  
  
  Unchecked Custom Exception
</h3>
<div class="highlight js-code-highlight">
<pre class="highlight plaintext"><code>
java
class MyUncheckedException extends RuntimeException {
}


</code></pre>

</div>




<h2>
  
  
  10. Java vs Spring Boot Exception Handling
</h2>

<h3>
  
  
  Java Level:
</h3>

<ul>
<li>try-catch</li>
<li>throws</li>
<li>method level handling</li>
</ul>

<h3>
  
  
  Spring Boot Level:
</h3>

<ul>
<li>Controller/API level handling</li>
<li>Client ko JSON response dena</li>
</ul>

<p>Uses:</p>

<ul>
<li>@ExceptionHandler</li>
<li>@ControllerAdvice / @RestControllerAdvice</li>
</ul>




<h2>
  
  
  11. Spring Boot Exception Handling Example
</h2>

<h3>
  
  
  Custom Exception
</h3>

<div class="highlight js-code-highlight">
<pre class="highlight plaintext"><code>
java
class UserNotFoundException extends RuntimeException {
   public UserNotFoundException(String msg){
      super(msg);
   }
}


</code></pre>

</div>
<h3>
  
  
  Global Handler
</h3>
<div class="highlight js-code-highlight">
<pre class="highlight plaintext"><code>
java
@RestControllerAdvice
class GlobalExceptionHandler {

   @ExceptionHandler(UserNotFoundException.class)
   public ResponseEntity&lt;String&gt; handle(UserNotFoundException e){
      return ResponseEntity.status(404).body(e.getMessage());
   }

   @ExceptionHandler(Exception.class)
   public ResponseEntity&lt;String&gt; handleAll(Exception e){
      return ResponseEntity.status(500).body("Server Error");
   }
}


</code></pre>

</div>
<h3>
  
  
  Controller
</h3>
<div class="highlight js-code-highlight">
<pre class="highlight plaintext"><code>
java
@GetMapping("/user/{id}")
public User getUser(@PathVariable int id){
   return repo.findById(id)
     .orElseThrow(() -&gt; new UserNotFoundException("User not found"));
}


</code></pre>

</div>




<h2>
  
  
  12. Key Interview Lines
</h2>

<ul>
<li>Checked exception = compile time, must handle</li>
<li>Unchecked exception = runtime, optional handling</li>
<li>SQLException only for SQL DB, not MongoDB</li>
<li>
<p>Custom exception:</p>

<ul>
<li>extends Exception → checked</li>
<li>extends RuntimeException → unchecked</li>
</ul>


</li>

<li><p>Spring Boot handles API errors using @ControllerAdvice</p></li>

</ul>




<h2>
  
  
  13. Summary
</h2>

<div class="table-wrapper-paragraph"><table>
<thead>
<tr>
<th>Type</th>
<th>Extend</th>
<th>Handle</th>
</tr>
</thead>
<tbody>
<tr>
<td>Checked</td>
<td>Exception</td>
<td>try-catch OR throws</td>
</tr>
<tr>
<td>Unchecked</td>
<td>RuntimeException</td>
<td>optional</td>
</tr>
<tr>
<td>Error</td>
<td>Error</td>
<td>avoid</td>
</tr>
</tbody>
</table></div>




<p>END</p>

<div class="highlight js-code-highlight">
<pre class="highlight plaintext"><code>


</code></pre>

</div>

