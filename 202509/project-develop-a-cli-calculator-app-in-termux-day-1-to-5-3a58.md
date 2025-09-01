---
Title: Project: Develop a CLI Calculator App in Termux – Day 1 to 5
Description: 
Author: Stephano Kambeta
Date: 2025-09-01T21:16:29.000Z
Robots: noindex,nofollow
Template: index
---
<p>
If you’ve ever thought, “I need to build something small but useful in <a href="https://terminaltools.blogspot.com/2025/01/how-to-install-termux-on-android-phone.html" rel="noopener noreferrer">Termux</a>,” then this 5-day CLI calculator project is for you.  
By the end of this guide, you’ll have your very own command-line calculator app running on your Android device — and more importantly, you’ll have learned enough to tackle bigger projects like  
<a href="https://terminaltools.blogspot.com/2025/07/quick-termux-projects-you-can-do.html" rel="noopener noreferrer">mini networking tools</a> or even lightweight automation scripts.
</p>




<h2>Why a CLI Calculator?</h2>

<p>
Before you dismiss this as too basic, hear me out. A CLI calculator is the perfect starter project:  
</p>

<ul>
  <li>It builds your confidence with Python and Bash scripting.</li>
  <li>You’ll learn to structure your code logically — a skill that scales.</li>
  <li>It prepares you for automation tasks like log parsing or even <a href="https://terminaltools.blogspot.com/2025/01/how-to-install-and-use-nmap-in-termux.html" rel="noopener noreferrer">network scans</a>.</li>
</ul>

<p>
Plus, in the cybersecurity space, scripting even simple tools is a core skill. Whether you’re building phishing detectors, working with APIs, or integrating security tools, these fundamentals carry over. 
</p>




<h2>Day 1 – Setting Up the Environment</h2>

<p>
Start with a clean Termux setup. If you’re new, check out <a href="https://terminaltools.blogspot.com/2024/12/things-to-do-after-installing-termux.html" rel="noopener noreferrer">this post on initial configurations</a>.  
Run:
</p>

<pre><code>pkg update &amp;&amp; pkg upgrade
pkg install python git nano
</code></pre>

<p>
Once Python is installed, verify it:
</p>

<pre><code>python --version</code></pre>

<p>
That’s it for Day 1 — your environment is ready. You’ve laid the foundation for everything that follows.
</p>




<h2>Day 2 – Writing the Basic Script</h2>

<p>
Create a new Python file:
</p>

<pre><code>nano calculator.py</code></pre>

<p>Paste this:</p>

<pre><code>def add(a, b): return a + b
def subtract(a, b): return a - b
def multiply(a, b): return a * b
def divide(a, b): return a / b if b != 0 else "Error: Division by zero"

while True:
    print("\n1. Add\n2. Subtract\n3. Multiply\n4. Divide\n5. Exit")
    choice = input("Choose an option: ")

    if choice == '5':
        break

    a = float(input("Enter first number: "))
    b = float(input("Enter second number: "))

    if choice == '1':
        print("Result:", add(a, b))
    elif choice == '2':
        print("Result:", subtract(a, b))
    elif choice == '3':
        print("Result:", multiply(a, b))
    elif choice == '4':
        print("Result:", divide(a, b))
    else:
        print("Invalid choice. Try again.")
</code></pre>

<p>
Run the script with:
</p>

<pre><code>python calculator.py</code></pre>




<h2>Day 3 – Improving the User Experience</h2>

<p>
By now, the app works but feels raw. Let’s make it cleaner:
</p>

<ul>
  <li>Add colors using <code>colorama</code>:</li>
</ul>

<pre><code>pip install colorama</code></pre>

<ul>
  <li>Refactor the code into functions for better readability.</li>
  <li>Add error handling for non-numeric inputs.</li>
</ul>

<p>
This is where you start thinking like a developer, not just a coder.  
Want to explore automation next? Look into <a href="https://terminaltools.blogspot.com/2025/01/how-to-install-and-use-ngrok-in-termux.html" rel="noopener noreferrer">tunneling tools</a> to integrate APIs with your scripts.
</p>




<h2>Day 4 – Adding Features</h2>

<p>
Let’s make it more than just a calculator:
</p>

<ul>
  <li>Implement a “history” feature by saving operations to a text file.</li>
  <li>Add a percentage calculation option.</li>
  <li>Include a square root or power function.</li>
</ul>

<pre><code>import math

def percentage(a, b):
    return (a / b) * 100
</code></pre>

<p>
Adding features is a great way to challenge yourself and keep the momentum going. Just like when you try advanced scripts for <a href="https://terminaltools.blogspot.com/2025/05/wifi-honey.html" rel="noopener noreferrer">Wi-Fi testing</a>, complexity comes in small steps.
</p>




<h2>Day 5 – Packaging and Sharing</h2>

<p>
By now, you have a fully functional CLI calculator. Let’s package it so others can use it:
</p>

<ul>
  <li>Create a dedicated folder for your app.</li>
  <li>Add a README.md explaining how to use it.</li>
  <li>Upload to GitHub for easy sharing.</li>
</ul>

<p>
If you’re serious about security, remember to follow <a href="https://terminaltools.blogspot.com/2025/08/network-security-tips-for-small-business.html" rel="noopener noreferrer">basic security practices</a> — even for small scripts.  
Many open-source tools get exploited because of poor permissions or exposed tokens.
</p>




<h2>Next Steps</h2>

<p>
Congratulations! You’ve built your first CLI app in Termux. Now, where to next? Here are some project ideas:
</p>

<ul>
  <li>Build a currency converter using APIs.</li>
  <li>Create a password generator that integrates with <a href="https://terminaltools.blogspot.com/2025/08/maxphisher-in-termux.html" rel="noopener noreferrer">phishing detection tools</a>.</li>
  <li>Explore penetration testing with <a href="https://terminaltools.blogspot.com/2025/01/how-to-install-and-use-nmap-in-termux.html" rel="noopener noreferrer">Nmap</a> or <a href="https://terminaltools.blogspot.com/2024/01/utilizing-aircrack-ng-in-termux.html" rel="noopener noreferrer">Aircrack-ng</a>.</li>
</ul>

<p>
Your calculator project isn’t just a calculator — it’s a stepping stone. Keep building, keep experimenting, and you’ll soon find yourself confident in larger Termux projects, whether it’s security automation, API integrations, or even <a href="https://terminaltools.blogspot.com/2025/04/turn-your-android-into-a-web-server-how-to-install-and-use-nginx-in-termux.html" rel="noopener noreferrer">hosting a mini web server</a>.
</p>




<h2>Final Thoughts</h2>

<p>
Small projects like this are the secret to mastering Termux. They build practical skills that will help you understand more complex tools, like those used in <a href="https://terminaltools.blogspot.com/2025/05/it-security.html" rel="noopener noreferrer">enterprise security setups</a> or <a href="https://terminaltools.blogspot.com/2025/08/cyber-security-plan-for-small-business.html" rel="noopener noreferrer">business-level risk management</a>.  
And if you ever doubt the importance of practicing with “simple” scripts, remember — the pros started small too.
</p>

