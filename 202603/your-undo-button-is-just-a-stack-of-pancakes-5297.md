---
Title: Your Undo Button is Just a Stack of Pancakes
Description: 
Author: Doogal Simpson
Date: 2026-03-09T22:06:53.000Z
Robots: noindex,nofollow
Template: index
---
<p><strong>TL;DR: I implement undo functionality using a Stack data structure because it follows the Last-In-First-Out (LIFO) principle. Each state change is pushed onto the stack. When I trigger an undo, I pop the most recent action, reverting the application state in O(1) time without the overhead of re-indexing an entire array.</strong></p>

<p>I see developers treat the undo button like a magic time-travel feature, but it’s actually just basic application of stack logic. If I try to track user history with a standard list, I’m setting myself up for index management headaches and unnecessary O(n) operations. In my experience, if you aren't using a stack to manage your undo buffer, you're essentially building a performance bottleneck by design.</p>

<h2>
  
  
  Why is the undo function always a Stack?
</h2>

<p>I use a stack because it enforces a strict linear history where the only accessible element is the most recent one. This ensures that every undo call reverses the exact last action performed, preventing state corruption and keeping the pointer overhead at a minimum.</p>

<p>When I'm building an app that needs to track state, I don't want to scan through a massive array or shift indices every time a user types a character. I need a data structure where I can drop a state snapshot on the top and grab it back just as quickly. A stack is the only structure that makes sense here because it mirrors the way we naturally backtrack through a sequence of events. It’s about efficiency: pushing and popping from the top of a stack is a constant-time operation.</p>

<h2>
  
  
  How does the pancake analogy explain LIFO?
</h2>

<p>LIFO (Last-In-First-Out) means the last pancake I put on the plate is the first one I eat. In software, the last action I push to the history stack is the first one I pop off when the user hits undo.</p>

<p>Think about a physical stack of pancakes. If I try to wedge a new pancake into the bottom of the pile, I'm looking at an O(n) operation and a broken breakfast. In code, that's just bad architecture. Instead, I just drop the new action on top. When I need to undo, I don't go digging for the first action I ever performed; I take the one right off the top. If I try to pull from the bottom, I’m just asking for index shifting nightmares and a collapsed state. You always eat from the top because it’s the only place where the data is "cheap" to access.</p>

<h2>
  
  
  Stack vs. Array: Why Efficiency Wins for History
</h2>

<p>I choose a stack over a general-purpose array because a stack limits the interface to exactly what I need: Push and Pop. This restriction prevents me from accidentally modifying middle-history elements and keeps implementation effort low. Here is how they compare in a history management context:</p>

<div class="table-wrapper-paragraph"><table>
<thead>
<tr>
<th>Feature</th>
<th>Stack (LIFO)</th>
<th>Standard Array</th>
</tr>
</thead>
<tbody>
<tr>
<td><strong>Insertion Complexity</strong></td>
<td>O(1) - Constant time push</td>
<td>O(n) - If shifting to index 0</td>
</tr>
<tr>
<td><strong>Deletion Complexity</strong></td>
<td>O(1) - Constant time pop</td>
<td>O(n) - If re-indexing history</td>
</tr>
<tr>
<td><strong>Index Management</strong></td>
<td>None required (Top pointer)</td>
<td>High (Tracking shifted indices)</td>
</tr>
<tr>
<td><strong>Implementation Effort</strong></td>
<td>Minimal (Native push/pop)</td>
<td>High (Manual slice/splice)</td>
</tr>
</tbody>
</table></div>

<h2>
  
  
  How do I implement this in a real state management loop?
</h2>

<p>In my code, I don't store every single tiny movement. I store state snapshots. The logic is simple: push the current state before a change, and pop it back when the user wants to revert.<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight javascript"><code><span class="kd">const</span> <span class="nx">history</span> <span class="o">=</span> <span class="p">[];</span> 

<span class="kd">const</span> <span class="nx">trackChange</span> <span class="o">=</span> <span class="p">(</span><span class="nx">state</span><span class="p">)</span> <span class="o">=&gt;</span> <span class="p">{</span>
  <span class="nx">history</span><span class="p">.</span><span class="nf">push</span><span class="p">(</span><span class="nx">state</span><span class="p">);</span>
<span class="p">};</span>

<span class="kd">const</span> <span class="nx">handleUndo</span> <span class="o">=</span> <span class="p">()</span> <span class="o">=&gt;</span> <span class="p">{</span>
  <span class="k">if </span><span class="p">(</span><span class="nx">history</span><span class="p">.</span><span class="nx">length</span> <span class="o">&lt;</span> <span class="mi">2</span><span class="p">)</span> <span class="k">return</span><span class="p">;</span>
  <span class="nx">history</span><span class="p">.</span><span class="nf">pop</span><span class="p">();</span> <span class="c1">// Remove current state</span>
  <span class="kd">const</span> <span class="nx">prevState</span> <span class="o">=</span> <span class="nx">history</span><span class="p">[</span><span class="nx">history</span><span class="p">.</span><span class="nx">length</span> <span class="o">-</span> <span class="mi">1</span><span class="p">];</span>
  <span class="nf">applyToUI</span><span class="p">(</span><span class="nx">prevState</span><span class="p">);</span>
<span class="p">};</span>
</code></pre>

</div>



<h2>
  
  
  How do I manage the memory footprint of a massive stack?
</h2>

<p>I don't let an undo stack grow infinitely because it will eventually eat the application's entire memory footprint. I implement a hard limit on the stack depth to keep things manageable.</p>

<p>Imagine a user working on a project for eight hours. If I keep every single state change, the browser will eventually grind to a halt. To solve this, I treat the stack like a circular buffer or I simply shift the bottom element off once the stack exceeds a limit—say, 100 actions. It’s a trade-off: I lose the "first pancake" from three hours ago, but I keep the app from crashing. It's about protecting the user's current session over their ancient history.</p>

<h2>
  
  
  FAQ
</h2>

<h3>
  
  
  Why not use a Queue for undo operations?
</h3>

<p>I can't use a Queue because it uses FIFO (First-In-First-Out) logic. If I did, hitting undo would revert the first thing you did when you opened the app three hours ago, rather than your most recent mistake. That’s a UX disaster.</p>

<h3>
  
  
  How does the 'Redo' button work alongside the stack?
</h3>

<p>I use a second stack for Redo. When I pop a state from the Undo stack, I don't delete it; I push it onto the Redo stack. It’s just moving pancakes between two plates. If the user makes a <em>new</em> change, I clear the Redo stack entirely.</p>

<h3>
  
  
  Can I store diffs instead of full state snapshots?
</h3>

<p>Yes, and for large applications, I should. Storing a full snapshot of a massive state on every keystroke is a waste of RAM. Storing the "diff" or the specific command performed allows me to keep the O(1) stack logic while significantly reducing the memory footprint per pancake.</p>

