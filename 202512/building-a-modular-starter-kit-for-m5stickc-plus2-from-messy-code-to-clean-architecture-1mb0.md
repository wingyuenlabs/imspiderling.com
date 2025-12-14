---
Title: Building a Modular Starter Kit for M5StickC-Plus2: From Messy Code to Clean Architecture
Description: 
Author: ChristopherDebray
Date: 2025-12-14T21:35:23.000Z
Robots: noindex,nofollow
Template: index
---
<h2>
  
  
  Why Another M5Stack Project?
</h2>

<blockquote>
<p>When I first got my M5StickC-Plus2, I was excited to build something cool. But like many developers, I quickly hit a wall of... boring setup work.</p>
</blockquote>

<p>You know the drill: configuring buttons, managing display coordinates, handling menus, dealing with power management, setting up timers. Before I could even start on the fun part of my project, I had to write hundreds of lines of infrastructure code.</p>

<p>Libraries help, but they come with their own problems:</p>

<ul>
<li>Black boxes: You can't see or modify how they work internally</li>
<li>Over-abstraction: Sometimes you need fine-grained control</li>
<li>Learning curve: Each library has its own API to learn</li>
<li>Dependencies: One library pulls in five others</li>
</ul>

<p>I wanted something different: <strong>a starter kit where you own all the code</strong>.</p>

<h2>
  
  
  The Philosophy: A Foundation, Not a Framework
</h2>

<p>This project isn't a library you import. It's a <strong>starting point you customize</strong>.</p>

<p>Think of it like this:</p>

<ul>
<li>Library: "Here's a menu system, use these methods"</li>
<li>This starter: "Here's how I built a menu system, change whatever you want"</li>
</ul>

<p>You get:</p>

<ul>
<li>✅ Full source code you can read and understand</li>
<li>✅ Working examples you can modify</li>
<li>✅ Architectural patterns you can extend</li>
<li>✅ No hidden dependencies or magic</li>
</ul>

<p>If you don't like how the menu scrolling works? Change it. Want different colors? Modify the display handler. Need a different button layout? Update the controls.</p>

<h2>
  
  
  The Journey: From Arduino IDE to PlatformIO
</h2>

<p>I started this project in Arduino IDE (as many do), but quickly switched to <strong>VSCode + PlatformIO</strong>. Here's why:</p>

<h3>
  
  
  Arduino IDE Pain Points
</h3>



<div class="highlight js-code-highlight">
<pre class="highlight cpp"><code><span class="c1">// Where is this function defined?</span>
<span class="c1">// Which library does this come from?</span>
<span class="c1">// Good luck finding it...</span>
<span class="n">M5</span><span class="p">.</span><span class="n">Lcd</span><span class="p">.</span><span class="n">setCursor</span><span class="p">(</span><span class="mi">10</span><span class="p">,</span> <span class="mi">80</span><span class="p">);</span>
</code></pre>

</div>



<h3>
  
  
   PlatformIO Wins
</h3>

<ul>
<li>
<strong>IntelliSense</strong>: Auto-completion that actually works</li>
<li>
<strong>Go to Definition</strong>: Jump to any function's source</li>
<li>
<strong>Project Structure</strong>: Proper file organization (the biggest issue for me)</li>
<li>
<strong>Library Management</strong>: Clear dependency handling</li>
<li>
<strong>Modern C++</strong>: Full C++11/14/17 support</li>
</ul>

<p>The switch took an hour. It saved me dozens of hours afterward.</p>

<h2>
  
  
  Key Architecture Decisions
</h2>

<h3>
  
  
   1. The Page System: Lifecycle Management
</h3>

<p>Early on, I realized I needed multiple "screens" or "pages". But switching between them was messy:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight cpp"><code><span class="c1">// ❌ The messy way</span>
<span class="k">if</span> <span class="p">(</span><span class="n">currentPage</span> <span class="o">==</span> <span class="mi">0</span><span class="p">)</span> <span class="p">{</span>
    <span class="n">drawClock</span><span class="p">();</span>
<span class="p">}</span> <span class="k">else</span> <span class="nf">if</span> <span class="p">(</span><span class="n">currentPage</span> <span class="o">==</span> <span class="mi">1</span><span class="p">)</span> <span class="p">{</span>
    <span class="n">drawMenu</span><span class="p">();</span>
<span class="p">}</span> <span class="k">else</span> <span class="nf">if</span> <span class="p">(</span><span class="n">currentPage</span> <span class="o">==</span> <span class="mi">2</span><span class="p">)</span> <span class="p">{</span>
    <span class="n">drawSettings</span><span class="p">();</span>
<span class="p">}</span>
</code></pre>

</div>



<p>I needed a proper lifecycle. Enter the <strong>PageManager</strong>:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight cpp"><code><span class="k">class</span> <span class="nc">PageBase</span> <span class="p">{</span>
    <span class="k">virtual</span> <span class="kt">void</span> <span class="n">setup</span><span class="p">()</span> <span class="o">=</span> <span class="mi">0</span><span class="p">;</span>    <span class="c1">// Called when entering page</span>
    <span class="k">virtual</span> <span class="kt">void</span> <span class="n">loop</span><span class="p">()</span> <span class="o">=</span> <span class="mi">0</span><span class="p">;</span>     <span class="c1">// Called every frame</span>
    <span class="k">virtual</span> <span class="kt">void</span> <span class="n">cleanup</span><span class="p">()</span> <span class="o">=</span> <span class="mi">0</span><span class="p">;</span>  <span class="c1">// Called when leaving page</span>
<span class="p">};</span>
</code></pre>

</div>



<p>Now each page manages itself:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight cpp"><code><span class="kt">void</span> <span class="n">ClockPage</span><span class="o">::</span><span class="n">setup</span><span class="p">()</span> <span class="p">{</span>
    <span class="n">display</span><span class="o">-&gt;</span><span class="n">clearScreen</span><span class="p">();</span>
    <span class="n">clockHandler</span><span class="o">-&gt;</span><span class="n">drawClock</span><span class="p">(</span><span class="mi">0</span><span class="p">);</span>
<span class="p">}</span>

<span class="kt">void</span> <span class="n">ClockPage</span><span class="o">::</span><span class="n">loop</span><span class="p">()</span> <span class="p">{</span>
    <span class="k">if</span> <span class="p">(</span><span class="n">hasActiveMenu</span><span class="p">())</span> <span class="k">return</span><span class="p">;</span>  <span class="c1">// Pause if menu is open</span>
    <span class="c1">// Update clock every second</span>
<span class="p">}</span>
</code></pre>

</div>



<p><strong>Lesson learned</strong>: Give each component its own lifecycle. Don't manage everything from main().</p>

<h3>
  
  
  2. The Menu Stack: Nested Menus Done Right
</h3>

<p>Menus were surprisingly hard. I wanted:</p>

<ul>
<li>A main menu</li>
<li>Submenus (Settings → Display Settings → Brightness)</li>
<li>A "back" button that works correctly</li>
</ul>

<p>The solution? A <strong>stack</strong> (Last In, First Out):<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight cpp"><code><span class="k">class</span> <span class="nc">MenuManager</span> <span class="p">{</span>
<span class="nl">private:</span>
    <span class="n">MenuHandler</span><span class="o">*</span> <span class="n">menuStack</span><span class="p">[</span><span class="n">MAX_MENU_STACK</span><span class="p">];</span>
    <span class="kt">int</span> <span class="n">stackSize</span><span class="p">;</span>

<span class="nl">public:</span>
    <span class="kt">void</span> <span class="n">pushMenu</span><span class="p">(</span><span class="n">MenuHandler</span><span class="o">*</span> <span class="n">menu</span><span class="p">)</span> <span class="p">{</span>
        <span class="n">menuStack</span><span class="p">[</span><span class="n">stackSize</span><span class="o">++</span><span class="p">]</span> <span class="o">=</span> <span class="n">menu</span><span class="p">;</span>
        <span class="n">menu</span><span class="o">-&gt;</span><span class="n">draw</span><span class="p">();</span>
    <span class="p">}</span>

    <span class="kt">void</span> <span class="nf">popMenu</span><span class="p">()</span> <span class="p">{</span>
        <span class="n">stackSize</span><span class="o">--</span><span class="p">;</span>
        <span class="k">if</span> <span class="p">(</span><span class="n">stackSize</span> <span class="o">&gt;</span> <span class="mi">0</span><span class="p">)</span> <span class="p">{</span>
            <span class="n">menuStack</span><span class="p">[</span><span class="n">stackSize</span> <span class="o">-</span> <span class="mi">1</span><span class="p">]</span><span class="o">-&gt;</span><span class="n">draw</span><span class="p">();</span>  <span class="c1">// Redraw previous menu</span>
        <span class="p">}</span>
    <span class="p">}</span>
<span class="p">};</span>
</code></pre>

</div>



<p>Now submenus just work:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight plaintext"><code>Clock Page
  → Open Menu
    → Settings
      → Display
        → [Back]
      → [Back]
    → [Back]
  Clock Page (restored)
</code></pre>

</div>



<p><strong>Lesson learned</strong>: Choose the right data structure. A stack naturally handles nested navigation.</p>

<h3>
  
  
   3. The Pointer Function vs std::function Saga
</h3>

<p>This was a <strong>4-hour debugging session</strong> that taught me a crucial C++ lesson.</p>

<p>I wanted menu callbacks that could access class members:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight cpp"><code><span class="c1">// I wanted to do this:</span>
<span class="n">mainMenu</span><span class="o">-&gt;</span><span class="n">addItem</span><span class="p">(</span><span class="s">"Start Timer"</span><span class="p">,</span> <span class="p">[</span><span class="k">this</span><span class="p">]()</span> <span class="p">{</span>
    <span class="n">clockHandler</span><span class="o">-&gt;</span><span class="n">startTimer</span><span class="p">();</span>  <span class="c1">// Access class member</span>
<span class="p">});</span>
</code></pre>

</div>



<p>But I got errors:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight plaintext"><code>error: no suitable conversion from lambda to void (*)()
</code></pre>

</div>



<p>The problem? My MenuItem struct used old C-style function pointers:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight c"><code><span class="c1">// ❌ Old way (C-style)</span>
<span class="k">struct</span> <span class="n">MenuItem</span> <span class="p">{</span>
    <span class="kt">void</span> <span class="p">(</span><span class="o">*</span><span class="n">callback</span><span class="p">)();</span>  <span class="c1">// Can't capture 'this'!</span>
<span class="p">};</span>
</code></pre>

</div>



<p>The fix? Modern C++ std::function:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight cpp"><code><span class="c1">// ✅ New way (C++11)</span>
<span class="k">struct</span> <span class="nc">MenuItem</span> <span class="p">{</span>
    <span class="n">std</span><span class="o">::</span><span class="n">function</span><span class="o">&lt;</span><span class="kt">void</span><span class="p">()</span><span class="o">&gt;</span> <span class="n">callback</span><span class="p">;</span>  <span class="c1">// Can capture anything!</span>
<span class="p">};</span>
</code></pre>

</div>



<p>Now this works:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight cpp"><code><span class="n">mainMenu</span><span class="o">-&gt;</span><span class="n">addItem</span><span class="p">(</span><span class="s">"Settings"</span><span class="p">,</span> <span class="p">[</span><span class="k">this</span><span class="p">]()</span> <span class="p">{</span>
    <span class="n">openSettingsSubmenu</span><span class="p">();</span>  <span class="c1">// 'this' captured, works perfectly</span>
<span class="p">});</span>
</code></pre>

</div>



<p><strong>Lesson learned</strong>: Use std::function for callbacks in modern C++. It's more flexible and handles lambdas with captures.</p>

<h3>
  
  
  4. Display Positioning: No More Magic Numbers
</h3>

<p>Early code looked like this:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight cpp"><code><span class="c1">// ❌ What does this even mean?</span>
<span class="n">M5</span><span class="p">.</span><span class="n">Lcd</span><span class="p">.</span><span class="n">setCursor</span><span class="p">(</span><span class="mi">10</span><span class="p">,</span> <span class="mi">80</span><span class="p">);</span>
<span class="n">M5</span><span class="p">.</span><span class="n">Lcd</span><span class="p">.</span><span class="n">setTextSize</span><span class="p">(</span><span class="mi">3</span><span class="p">);</span>
<span class="n">M5</span><span class="p">.</span><span class="n">Lcd</span><span class="p">.</span><span class="n">print</span><span class="p">(</span><span class="s">"Hello"</span><span class="p">);</span>
</code></pre>

</div>



<p>I created the <strong>DisplayHandler</strong> to abstract positions:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight cpp"><code><span class="c1">// ✅ Semantic and clear</span>
<span class="n">display</span><span class="o">-&gt;</span><span class="n">displayMainTitle</span><span class="p">(</span><span class="s">"Hello"</span><span class="p">);</span>
<span class="n">display</span><span class="o">-&gt;</span><span class="n">displaySubtitle</span><span class="p">(</span><span class="s">"Subtitle"</span><span class="p">);</span>
<span class="n">display</span><span class="o">-&gt;</span><span class="n">displayStatus</span><span class="p">(</span><span class="s">"Ready"</span><span class="p">,</span> <span class="n">MSG_SUCCESS</span><span class="p">);</span>
</code></pre>

</div>



<p>Behind the scenes:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight cpp"><code><span class="kt">void</span> <span class="nf">displayMainTitle</span><span class="p">(</span><span class="k">const</span> <span class="kt">char</span><span class="o">*</span> <span class="n">text</span><span class="p">,</span> <span class="n">MessageType</span> <span class="n">type</span><span class="p">)</span> <span class="p">{</span>
    <span class="n">M5</span><span class="p">.</span><span class="n">Lcd</span><span class="p">.</span><span class="n">setTextSize</span><span class="p">(</span><span class="n">SIZE_TITLE</span><span class="p">);</span>  <span class="c1">// Consistent size</span>
    <span class="n">M5</span><span class="p">.</span><span class="n">Lcd</span><span class="p">.</span><span class="n">setTextColor</span><span class="p">(</span><span class="n">getColorForType</span><span class="p">(</span><span class="n">type</span><span class="p">));</span>

    <span class="kt">int</span> <span class="n">x</span> <span class="o">=</span> <span class="p">(</span><span class="n">SCREEN_WIDTH</span> <span class="o">-</span> <span class="n">textWidth</span><span class="p">)</span> <span class="o">/</span> <span class="mi">2</span><span class="p">;</span>  <span class="c1">// Auto-center</span>
    <span class="kt">int</span> <span class="n">y</span> <span class="o">=</span> <span class="n">ZONE_CENTER_Y</span> <span class="o">-</span> <span class="mi">20</span><span class="p">;</span>

    <span class="n">M5</span><span class="p">.</span><span class="n">Lcd</span><span class="p">.</span><span class="n">setCursor</span><span class="p">(</span><span class="n">x</span><span class="p">,</span> <span class="n">y</span><span class="p">);</span>
    <span class="n">M5</span><span class="p">.</span><span class="n">Lcd</span><span class="p">.</span><span class="n">print</span><span class="p">(</span><span class="n">text</span><span class="p">);</span>
<span class="p">}</span>
</code></pre>

</div>



<p><strong>Lesson learned</strong>: Abstract low-level details. Your future self will thank you.</p>

<h3>
  
  
  5. Deep Sleep: The 3-Hour Power Management Bug
</h3>

<p>The M5StickC-Plus2 has great battery life... if you use deep sleep correctly.</p>

<p>My first attempt:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight cpp"><code><span class="c1">// ❌ This crashes the device on wake-up</span>
<span class="n">esp_deep_sleep_start</span><span class="p">();</span>
</code></pre>

</div>



<p>After diving into documentation and forums, I found the issue: <strong>GPIO4 must stay HIGH during sleep</strong> or the device loses power.</p>

<p>The working solution:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight cpp"><code><span class="kt">void</span> <span class="nf">M5deepSleep</span><span class="p">(</span><span class="kt">uint64_t</span> <span class="n">microseconds</span><span class="p">)</span> <span class="p">{</span>
    <span class="c1">// CRITICAL: Keep power pin high</span>
    <span class="n">pinMode</span><span class="p">(</span><span class="mi">4</span><span class="p">,</span> <span class="n">OUTPUT</span><span class="p">);</span>
    <span class="n">digitalWrite</span><span class="p">(</span><span class="mi">4</span><span class="p">,</span> <span class="n">HIGH</span><span class="p">);</span>
    <span class="n">gpio_hold_en</span><span class="p">(</span><span class="n">GPIO_NUM_4</span><span class="p">);</span>
    <span class="n">gpio_deep_sleep_hold_en</span><span class="p">();</span>

    <span class="n">esp_sleep_enable_timer_wakeup</span><span class="p">(</span><span class="n">microseconds</span><span class="p">);</span>
    <span class="n">esp_deep_sleep_start</span><span class="p">();</span>
<span class="p">}</span>
</code></pre>

</div>



<p>This powers my Pomodoro timer: 25 minutes of sleep, wake up, beep alarm, show clock.</p>

<p><strong>Lesson learned</strong>: Hardware-specific quirks require hardware-specific solutions. Don't always assume standard APIs work out of the box.</p>

<h2>
  
  
  Button Controls: Finding the Ergonomic Sweet Spot
</h2>

<p>The M5StickC-Plus2 has three buttons:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight plaintext"><code>      ______PWR          (side)
                    A    (front)
      ___B_____          (side, opposite)
</code></pre>

</div>



<p>After testing different layouts, I settled on:</p>

<p><strong>No menu active</strong>:</p>

<ul>
<li>PWR: Change page</li>
<li>A: Open menu</li>
<li>B: Page-specific action (e.g., start timer on double click)</li>
</ul>

<p><strong>Menu active</strong>:</p>

<ul>
<li>PWR: Navigate down</li>
<li>A: Select item</li>
<li>B (short): Navigate up</li>
<li>B (long hold): Close menu</li>
</ul>

<p>Why this layout?</p>

<ul>
<li>
<strong>Side buttons for navigation</strong>: Easier to press while holding device</li>
<li>
<strong>Center button for actions</strong>: Most important button in prime position</li>
<li>
<strong>Long press for "back"</strong>: Prevents accidental exits</li>
</ul>

<p><strong>Lesson learned</strong>: Button ergonomics matter. Test on actual hardware, not just in your head.</p>

<h2>
  
  
   The Tech Stack
</h2>

<ul>
<li>
<strong>Platform</strong>: M5StickC-Plus2</li>
<li>
<strong>IDE</strong>: VSCode + PlatformIO</li>
<li>
<strong>Language</strong>: C++ (C++11 features)</li>
<li>
<strong>Libraries</strong>: M5Unified</li>
<li>
<strong>Architecture</strong>: OOP with composition pattern
</li>
</ul>

<div class="highlight js-code-highlight">
<pre class="highlight plaintext"><code>lib/
├── display_handler.h      # Display abstraction
├── menu_handler.h         # Individual menu logic
├── menu_manager.h         # Menu stack
├── page_manager.h         # Page lifecycle
├── clock_handler.h        # Time &amp; timers
├── battery_handler.h      # Power management
└── pages/
    ├── page_base.h        # Abstract base class
    └── clock_page.h       # Default clock page
</code></pre>

</div>



<h2>
  
  
  What I'd Do Differently
</h2>

<h3>
  
  
  1. Start with std::function
</h3>

<p>Don't use C-style function pointers for callbacks. Go straight to std::function.<br>
Althought it is more consuming than the pointers, but i didn’t have time to figure a better option (for now)</p>

<h3>
  
  
  2. Test Deep Sleep Early
</h3>

<p>Don't wait until the end to test power management. It's hardware-dependent and can break everything.</p>

<h3>
  
  
  3. Design Button Layout on Paper
</h3>

<p>Sketch the button layout before writing code. Changing it later affects everything.</p>

<h3>
  
  
  4. Use PlatformIO from Day 1
</h3>

<p>Don't start in Arduino IDE. The migration takes time and breaks things.</p>

<h2>
  
  
   What Worked Really Well
</h2>

<h3>
  
  
   1. Composition Over Inheritance
</h3>

<p>Every page gets a <code>DisplayHandler*</code> and <code>MenuManager*</code>. They don't inherit display logic—they compose it.</p>

<h3>
  
  
   2. Clear Separation of Concerns
</h3>

<p><code>*_handler.h</code>: Focused, reusable components</p>

<p><code>*_manager.h</code>: Complex orchestration</p>

<p><code>*_utils.h</code>: Utility functions</p>

<h3>
  
  
   3. Lambda Callbacks
</h3>

<p>Being able to write <code>[this]() { myMethod(); }</code> inline makes code so much cleaner than separate callback functions.</p>

<h3>
  
  
   4. The Base Page Pattern
</h3>

<p>Every page inherits from <code>PageBase</code>, which provides menu management for free. No duplicate code.</p>

<h2>
  
  
   Try It Yourself
</h2>

<p>The complete starter kit is <a href="https://github.com/ChristopherDebray/M5StickC-Plus2-starter" rel="noopener noreferrer"><strong>available on GitHub</strong></a>. Clone it, upload to your M5StickC-Plus2, and you'll have:</p>

<ul>
<li>✅ A working clock page</li>
<li>✅ Battery indicator</li>
<li>✅ Menu system with submenus</li>
<li>✅ Page navigation</li>
<li>✅ Pomodoro timer</li>
<li>✅ All the code to modify</li>
</ul>

<p>Want to build a fitness tracker? Keep the page system, replace the clock logic.</p>

<p>Building a game? Use the menu system for your settings, swap in your game loop.</p>

<p>Creating an IoT dashboard? The display handler abstracts all the positioning for you.</p>

<h2>
  
  
   Final Thoughts
</h2>

<p>Building this starter kit taught me that <strong>good architecture is invisible</strong>. When it works, you don't think about pages or menus—you just build features.</p>

<p>That's the goal: give you the boring stuff so you can focus on the interesting stuff.</p>

<p>The M5StickC-Plus2 is a fantastic device. With the right foundation, you can build something amazing in a weekend instead of spending that weekend setting up infrastructure.</p>

<p>Now go build something cool. 🚀</p>

<h2>
  
  
  Resources
</h2>

<ul>
<li><a href="https://github.com/m5stack/M5Unified" rel="noopener noreferrer">M5Unified</a></li>
</ul>

