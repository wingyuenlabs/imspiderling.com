---
Title: Building a Simple Ticket Tracker CLI in Go
Description: 
Author: Christian Ameachi
Date: 2025-11-22T21:49:28.000Z
Robots: noindex,nofollow
Template: index
---
<h3>
  
  
  A lightweight, command-line alternative to complex ticketing systems.
</h3>

<p>Using golang and cobra-cli, I built a simple command-line interface for managing support tickets. Tickets are stored locally in a CSV file.</p>




<h2>
  
  
  Introduction
</h2>

<p>As developers, we often find ourselves juggling multiple tasks, bugs, and feature requests. While tools like Jira, Trello, or GitHub Issues are powerful, sometimes you just need something <strong>simple</strong>, <strong>fast</strong>, and <strong>local</strong> to track your daily work without leaving the terminal.</p>

<p>That's why I built <strong>Ticket CLI</strong>—a simple command-line tool written in Go to track daily tickets and store them in a CSV file. No servers, no databases, just a binary and a text file.</p>

<h2>
  
  
  The Tech Stack
</h2>

<p>For this project, I choose:</p>

<ul>
<li>
<strong><a href="https://go.dev/" rel="noopener noreferrer">Go</a></strong>: For its speed, simplicity, and ability to compile into a single binary.</li>
<li>
<strong><a href="https://github.com/spf13/cobra" rel="noopener noreferrer">Cobra</a></strong>: The industry standard for building modern CLI applications in Go. It handles flag parsing, subcommands, and help text generation effortlessly.</li>
<li>
<strong>Standard Library (<code>encoding/csv</code>)</strong>: To keep dependencies low, I used Go's built-in CSV support for data persistence.</li>
</ul>

<h2>
  
  
  How It Works
</h2>

<p>The project follows a standard Go CLI structure:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight plaintext"><code>ticket-cli/
├── cmd/            # Cobra commands (add, list, delete)
├── internal/       # Business logic
│   └── storage/    # CSV handling
└── main.go         # Entry point
</code></pre>

</div>



<h3>
  
  
  1. The Command Structure
</h3>

<p>Using Cobra, I defined commands like <code>add</code>, <code>list</code>, and <code>delete</code>. Here's a snippet of how the <code>add</code> command handles flags to create a new ticket:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight go"><code><span class="c">// cmd/add.go</span>
<span class="k">var</span> <span class="n">addCmd</span> <span class="o">=</span> <span class="o">&amp;</span><span class="n">cobra</span><span class="o">.</span><span class="n">Command</span><span class="p">{</span>
    <span class="n">Use</span><span class="o">:</span>   <span class="s">"add"</span><span class="p">,</span>
    <span class="n">Short</span><span class="o">:</span> <span class="s">"Add a new ticket"</span><span class="p">,</span>
    <span class="n">Run</span><span class="o">:</span> <span class="k">func</span><span class="p">(</span><span class="n">cmd</span> <span class="o">*</span><span class="n">cobra</span><span class="o">.</span><span class="n">Command</span><span class="p">,</span> <span class="n">args</span> <span class="p">[]</span><span class="kt">string</span><span class="p">)</span> <span class="p">{</span>
        <span class="c">// default values logic...</span>

        <span class="n">t</span> <span class="o">:=</span> <span class="n">storage</span><span class="o">.</span><span class="n">Ticket</span><span class="p">{</span>
            <span class="n">ID</span><span class="o">:</span>          <span class="n">fmt</span><span class="o">.</span><span class="n">Sprintf</span><span class="p">(</span><span class="s">"%d"</span><span class="p">,</span> <span class="n">time</span><span class="o">.</span><span class="n">Now</span><span class="p">()</span><span class="o">.</span><span class="n">UnixNano</span><span class="p">()),</span>
            <span class="n">Title</span><span class="o">:</span>       <span class="n">flagTitle</span><span class="p">,</span>
            <span class="n">Customer</span><span class="o">:</span>    <span class="n">flagCustomer</span><span class="p">,</span>
            <span class="n">Priority</span><span class="o">:</span>    <span class="n">flagPriority</span><span class="p">,</span>
            <span class="n">Status</span><span class="o">:</span>      <span class="n">flagStatus</span><span class="p">,</span>
            <span class="n">Description</span><span class="o">:</span> <span class="n">flagDescription</span><span class="p">,</span>
        <span class="p">}</span>

        <span class="k">if</span> <span class="n">err</span> <span class="o">:=</span> <span class="n">storage</span><span class="o">.</span><span class="n">AppendTicket</span><span class="p">(</span><span class="n">t</span><span class="p">);</span> <span class="n">err</span> <span class="o">!=</span> <span class="no">nil</span> <span class="p">{</span>
            <span class="n">fmt</span><span class="o">.</span><span class="n">Println</span><span class="p">(</span><span class="s">"Error saving ticket:"</span><span class="p">,</span> <span class="n">err</span><span class="p">)</span>
            <span class="k">return</span>
        <span class="p">}</span>
        <span class="n">fmt</span><span class="o">.</span><span class="n">Println</span><span class="p">(</span><span class="s">"Ticket saved with ID:"</span><span class="p">,</span> <span class="n">t</span><span class="o">.</span><span class="n">ID</span><span class="p">)</span>
    <span class="p">},</span>
<span class="p">}</span>
</code></pre>

</div>



<h3>
  
  
  2. Data Persistence (The "Database")
</h3>

<p>Instead of setting up SQLite or a JSON store, I opted for CSV. It's human-readable and easy to debug. The <code>internal/storage</code> package handles reading and writing to <code>tickets.csv</code>.<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight go"><code><span class="c">// internal/storage/storage.go</span>
<span class="k">func</span> <span class="n">AppendTicket</span><span class="p">(</span><span class="n">t</span> <span class="n">Ticket</span><span class="p">)</span> <span class="kt">error</span> <span class="p">{</span>
    <span class="c">// ... (file opening logic)</span>
    <span class="n">w</span> <span class="o">:=</span> <span class="n">csv</span><span class="o">.</span><span class="n">NewWriter</span><span class="p">(</span><span class="n">f</span><span class="p">)</span>
    <span class="k">defer</span> <span class="n">w</span><span class="o">.</span><span class="n">Flush</span><span class="p">()</span>

    <span class="n">rec</span> <span class="o">:=</span> <span class="p">[]</span><span class="kt">string</span><span class="p">{</span><span class="n">t</span><span class="o">.</span><span class="n">ID</span><span class="p">,</span> <span class="n">t</span><span class="o">.</span><span class="n">Date</span><span class="p">,</span> <span class="n">t</span><span class="o">.</span><span class="n">Title</span><span class="p">,</span> <span class="n">t</span><span class="o">.</span><span class="n">Customer</span><span class="p">,</span> <span class="n">t</span><span class="o">.</span><span class="n">Priority</span><span class="p">,</span> <span class="n">t</span><span class="o">.</span><span class="n">Status</span><span class="p">,</span> <span class="n">t</span><span class="o">.</span><span class="n">Description</span><span class="p">}</span>
    <span class="k">return</span> <span class="n">w</span><span class="o">.</span><span class="n">Write</span><span class="p">(</span><span class="n">rec</span><span class="p">)</span>
<span class="p">}</span>
</code></pre>

</div>



<h2>
  
  
  Installation &amp; Usage
</h2>

<p>You can clone the repo and build it yourself:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight shell"><code>git clone https://github.com/yourusername/ticket-cli
<span class="nb">cd </span>ticket-cli
go mod tidy
go build <span class="nt">-o</span> ticket-cli <span class="nb">.</span>
</code></pre>

</div>



<h3>
  
  
  Adding a Ticket
</h3>



<div class="highlight js-code-highlight">
<pre class="highlight shell"><code>./ticket-cli add <span class="nt">--title</span> <span class="s2">"Fix login bug"</span> <span class="nt">--priority</span> high <span class="nt">--customer</span> <span class="s2">"Acme Corp"</span>
</code></pre>

</div>



<h3>
  
  
  Listing Tickets
</h3>



<div class="highlight js-code-highlight">
<pre class="highlight shell"><code>./ticket-cli list
</code></pre>

</div>



<h3>
  
  
  Filter by date
</h3>



<div class="highlight js-code-highlight">
<pre class="highlight shell"><code>./ticket-cli list <span class="nt">--date</span> 2025-11-15
</code></pre>

</div>



<h3>
  
  
  CSV Storage
</h3>

<p>A CSV file is created automatically at the Project directory, and it keeps getting updated, once a new ticket is added using the <code>.ticket-cli add --flags</code></p>

<h3>
  
  
  Columns :
</h3>



<div class="highlight js-code-highlight">
<pre class="highlight plaintext"><code>ID, Date, Title, Customer, Priority, Status, Description
</code></pre>

</div>



<h2>
  
  
  Future Improvements
</h2>

<p>This is just an MVP. Some ideas for the future include:</p>

<ul>
<li>
<strong>JSON/SQLite Storage</strong>: For more complex querying.</li>
<li>
<strong>TUI (Text User Interface)</strong>: Using <code>bubbletea</code> for an interactive dashboard.</li>
<li>
<strong>Cloud Sync</strong>: Syncing tickets to a Gist or S3 bucket.</li>
</ul>

<h2>
  
  
  Conclusion
</h2>

<p>Building CLI tools in Go is a rewarding experience. Cobra makes the interface professional, and Go's standard library handles the rest. If you're looking for a weekend project, try building your own developer tools!</p>




<p><em>Check out the code on <a href="https://github.com/Amae69/support_ticket-cli" rel="noopener noreferrer">GitHub</a>.</em></p>

