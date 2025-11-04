---
Title: Create a Text Editor With Go - Setup
Description: 
Author: Andres Court
Date: 2025-11-04T21:30:44.000Z
Robots: noindex,nofollow
Template: index
---
<p>For a couple of months now, I've been searching for new projects to work on, where I can learn more about <a href="https://go.dev" rel="noopener noreferrer">Go</a>, and not really wanted to continue with Web Development.</p>

<p>I came across a tutorial where they <a href="https://viewsourcecode.org/snaptoken/kilo/index.html" rel="noopener noreferrer">Build Your Own Text Editor</a> which is using the C language.</p>

<p>This is my attempt on porting that tutorial to Go, this will be divided in several posts that I will be publishing as I continue with the process.</p>

<p>In this first chapter, we will be setting up the project and enter raw mode so we can build from there our text editor.</p>

<h2>
  
  
  Setting up the project
</h2>

<blockquote>
<p><strong>Note:</strong>  This project has been developed and tested on LINUX, if you'd like to port this to Mac or Windows, you are welcome to make a PR in that regard</p>
</blockquote>

<p><strong>Requisites</strong></p>

<ul>
<li>Go language: If you don't have it already installed, you can do it following the <a href="https://go.dev/doc/install" rel="noopener noreferrer">installation instructions</a>
</li>
<li>A code editor: In this tutorial we will be using <a href="https://neovim.org" rel="noopener noreferrer">NeoVim</a>
</li>
<li>A terminal: I will be using <a href="https://alacritty.org/" rel="noopener noreferrer">Alacritty</a>
</li>
</ul>

<p><strong>Start up</strong></p>

<p>On your terminal we will create the project directory, and initialize the go module:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight shell"><code><span class="nb">mkdir </span>kilo-go
<span class="nb">cd </span>kilo-go
go mod init github.com/alcb1310/kilo-go
</code></pre>

</div>



<blockquote>
<p><strong>Note:</strong> Remember to change the module name for your own</p>
</blockquote>

<p>This will be a series of articles where we will be going through all the steps to code the <em>Text Editor</em></p>

