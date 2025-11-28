---
Title: The Secret Life of Go: Arrays and Slices
Description: 
Author: Aaron Rose
Date: 2025-11-28T21:49:45.000Z
Robots: noindex,nofollow
Template: index
---
<h2>
  
  
  Chapter 4: Collections and the Art of Growing Lists
</h2>

<p>Thursday morning brought rain. Ethan arrived at the archive shaking water from his jacket, balancing a coffee tray and a small pastry box.</p>

<p>Eleanor glanced up from her laptop. "Wet out there?"</p>

<p>"November in New York." He set down the coffees and opened the box. "Rugelach today. Chocolate and cinnamon."</p>

<p>"You're learning the neighborhood." She took one and bit into it. "What made you choose rugelach?"</p>

<p>"The baker said they're good for thinking. Something about the layers."</p>

<p>Eleanor smiled. "Layers. Perfect. Today we're talking about collections—how to store multiple pieces of data. And yes, there are layers to this story."</p>

<p>She opened a new file. "Tell me, Ethan. If I asked you to store five numbers, how would you do it?"</p>

<p>"Five variables?"</p>

<p>"You could. <code>num1</code>, <code>num2</code>, <code>num3</code>, <code>num4</code>, <code>num5</code>. What if I asked for a hundred numbers?"</p>

<p>"That would be... painful."</p>

<p>"Exactly. So every language has collections—ways to store groups of related data. Go has two: arrays and slices. Let's start with arrays."<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight go"><code><span class="k">package</span> <span class="n">main</span>

<span class="k">import</span> <span class="s">"fmt"</span>

<span class="k">func</span> <span class="n">main</span><span class="p">()</span> <span class="p">{</span>
    <span class="k">var</span> <span class="n">numbers</span> <span class="p">[</span><span class="m">5</span><span class="p">]</span><span class="kt">int</span>
    <span class="n">numbers</span><span class="p">[</span><span class="m">0</span><span class="p">]</span> <span class="o">=</span> <span class="m">10</span>
    <span class="n">numbers</span><span class="p">[</span><span class="m">1</span><span class="p">]</span> <span class="o">=</span> <span class="m">20</span>
    <span class="n">numbers</span><span class="p">[</span><span class="m">2</span><span class="p">]</span> <span class="o">=</span> <span class="m">30</span>
    <span class="n">numbers</span><span class="p">[</span><span class="m">3</span><span class="p">]</span> <span class="o">=</span> <span class="m">40</span>
    <span class="n">numbers</span><span class="p">[</span><span class="m">4</span><span class="p">]</span> <span class="o">=</span> <span class="m">50</span>

    <span class="n">fmt</span><span class="o">.</span><span class="n">Println</span><span class="p">(</span><span class="n">numbers</span><span class="p">)</span>
    <span class="n">fmt</span><span class="o">.</span><span class="n">Println</span><span class="p">(</span><span class="s">"First number:"</span><span class="p">,</span> <span class="n">numbers</span><span class="p">[</span><span class="m">0</span><span class="p">])</span>
    <span class="n">fmt</span><span class="o">.</span><span class="n">Println</span><span class="p">(</span><span class="s">"Length:"</span><span class="p">,</span> <span class="nb">len</span><span class="p">(</span><span class="n">numbers</span><span class="p">))</span>
<span class="p">}</span>
</code></pre>

</div>



<p>"An array in Go is a fixed-size collection. Look at the declaration: <code>var numbers [5]int</code>. The <code>[5]</code> means this array holds exactly five integers. Not four. Not six. Five."</p>

<p>Ethan studied the code. "And we access them with brackets?"</p>

<p>"Yes. Arrays are zero-indexed—the first element is at position 0, the last at position 4. Think of it like floors in a building if the ground floor is floor zero."</p>

<p>Eleanor ran the code:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight plaintext"><code>[10 20 30 40 50]
First number: 10
Length: 5
</code></pre>

</div>



<p>"The <code>len</code> function returns the length—how many elements the array holds. For arrays, this never changes. It's fixed at compile time."</p>

<p>"What happens if I try to add a sixth number?"</p>

<p>"The compiler stops you. Watch:" She typed:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight go"><code><span class="n">numbers</span><span class="p">[</span><span class="m">5</span><span class="p">]</span> <span class="o">=</span> <span class="m">60</span>  <span class="c">// This won't compile</span>
</code></pre>

</div>



<p>"Go will say: 'index out of range.' Arrays are rigid. Their size is part of their type—<code>[5]int</code> is a different type from <code>[6]int</code>. They're not interchangeable."</p>

<p>Ethan frowned. "That seems... restrictive?"</p>

<p>"It is. And that's why you almost never use arrays in Go." Eleanor closed the file. "Arrays exist, they're the foundation, but in practice, you use slices. Let me show you."</p>

<p>She opened a new file:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight go"><code><span class="k">package</span> <span class="n">main</span>

<span class="k">import</span> <span class="s">"fmt"</span>

<span class="k">func</span> <span class="n">main</span><span class="p">()</span> <span class="p">{</span>
    <span class="n">numbers</span> <span class="o">:=</span> <span class="p">[]</span><span class="kt">int</span><span class="p">{</span><span class="m">10</span><span class="p">,</span> <span class="m">20</span><span class="p">,</span> <span class="m">30</span><span class="p">,</span> <span class="m">40</span><span class="p">,</span> <span class="m">50</span><span class="p">}</span>

    <span class="n">fmt</span><span class="o">.</span><span class="n">Println</span><span class="p">(</span><span class="n">numbers</span><span class="p">)</span>
    <span class="n">fmt</span><span class="o">.</span><span class="n">Println</span><span class="p">(</span><span class="s">"Length:"</span><span class="p">,</span> <span class="nb">len</span><span class="p">(</span><span class="n">numbers</span><span class="p">))</span>

    <span class="n">numbers</span> <span class="o">=</span> <span class="nb">append</span><span class="p">(</span><span class="n">numbers</span><span class="p">,</span> <span class="m">60</span><span class="p">)</span>
    <span class="n">fmt</span><span class="o">.</span><span class="n">Println</span><span class="p">(</span><span class="s">"After append:"</span><span class="p">,</span> <span class="n">numbers</span><span class="p">)</span>
    <span class="n">fmt</span><span class="o">.</span><span class="n">Println</span><span class="p">(</span><span class="s">"New length:"</span><span class="p">,</span> <span class="nb">len</span><span class="p">(</span><span class="n">numbers</span><span class="p">))</span>
<span class="p">}</span>
</code></pre>

</div>



<p>"Look at the declaration: <code>numbers := []int{10, 20, 30, 40, 50}</code>. No size in the brackets. That's a slice."</p>

<p>"What's the difference?"</p>

<p>"A slice is a dynamic view into an array. It can grow. It can shrink. It's flexible." Eleanor ran the code:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight plaintext"><code>[10 20 30 40 50]
Length: 5
After append: [10 20 30 40 50 60]
New length: 6
</code></pre>

</div>



<p>"See? We added a sixth element with <code>append</code>, and the slice grew. No compiler error. No fixed size. Notice how we assign the result back: <code>numbers = append(numbers, 60)</code>. Always do this—<code>append</code> may return a slice with a different backing array."</p>

<p>Ethan leaned forward. "So slices are just better arrays?"</p>

<p>"Not quite. Let me show you what's really happening." Eleanor drew on her notepad:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight plaintext"><code>Slice:           [10, 20, 30]
                  ↓   ↓   ↓
Backing Array:   [10, 20, 30, __, __, __]
                  ↑           ↑
                  start       capacity
</code></pre>

</div>



<p>"A slice is actually three pieces of information: a pointer to an underlying array, a length (how many elements you're using), and a capacity (how many elements the backing array can hold before it needs to grow)."</p>

<p>She typed a new example:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight go"><code><span class="k">package</span> <span class="n">main</span>

<span class="k">import</span> <span class="s">"fmt"</span>

<span class="k">func</span> <span class="n">main</span><span class="p">()</span> <span class="p">{</span>
    <span class="n">numbers</span> <span class="o">:=</span> <span class="nb">make</span><span class="p">([]</span><span class="kt">int</span><span class="p">,</span> <span class="m">3</span><span class="p">,</span> <span class="m">6</span><span class="p">)</span>
    <span class="n">fmt</span><span class="o">.</span><span class="n">Println</span><span class="p">(</span><span class="s">"Slice:"</span><span class="p">,</span> <span class="n">numbers</span><span class="p">)</span>
    <span class="n">fmt</span><span class="o">.</span><span class="n">Println</span><span class="p">(</span><span class="s">"Length:"</span><span class="p">,</span> <span class="nb">len</span><span class="p">(</span><span class="n">numbers</span><span class="p">))</span>
    <span class="n">fmt</span><span class="o">.</span><span class="n">Println</span><span class="p">(</span><span class="s">"Capacity:"</span><span class="p">,</span> <span class="nb">cap</span><span class="p">(</span><span class="n">numbers</span><span class="p">))</span>

    <span class="n">numbers</span><span class="p">[</span><span class="m">0</span><span class="p">]</span> <span class="o">=</span> <span class="m">10</span>
    <span class="n">numbers</span><span class="p">[</span><span class="m">1</span><span class="p">]</span> <span class="o">=</span> <span class="m">20</span>
    <span class="n">numbers</span><span class="p">[</span><span class="m">2</span><span class="p">]</span> <span class="o">=</span> <span class="m">30</span>

    <span class="n">fmt</span><span class="o">.</span><span class="n">Println</span><span class="p">(</span><span class="s">"</span><span class="se">\n</span><span class="s">After setting values:"</span><span class="p">)</span>
    <span class="n">fmt</span><span class="o">.</span><span class="n">Println</span><span class="p">(</span><span class="s">"Slice:"</span><span class="p">,</span> <span class="n">numbers</span><span class="p">)</span>

    <span class="n">numbers</span> <span class="o">=</span> <span class="nb">append</span><span class="p">(</span><span class="n">numbers</span><span class="p">,</span> <span class="m">40</span><span class="p">,</span> <span class="m">50</span><span class="p">,</span> <span class="m">60</span><span class="p">)</span>
    <span class="n">fmt</span><span class="o">.</span><span class="n">Println</span><span class="p">(</span><span class="s">"</span><span class="se">\n</span><span class="s">After appending 3 elements:"</span><span class="p">)</span>
    <span class="n">fmt</span><span class="o">.</span><span class="n">Println</span><span class="p">(</span><span class="s">"Slice:"</span><span class="p">,</span> <span class="n">numbers</span><span class="p">)</span>
    <span class="n">fmt</span><span class="o">.</span><span class="n">Println</span><span class="p">(</span><span class="s">"Length:"</span><span class="p">,</span> <span class="nb">len</span><span class="p">(</span><span class="n">numbers</span><span class="p">))</span>
    <span class="n">fmt</span><span class="o">.</span><span class="n">Println</span><span class="p">(</span><span class="s">"Capacity:"</span><span class="p">,</span> <span class="nb">cap</span><span class="p">(</span><span class="n">numbers</span><span class="p">))</span>

    <span class="n">numbers</span> <span class="o">=</span> <span class="nb">append</span><span class="p">(</span><span class="n">numbers</span><span class="p">,</span> <span class="m">70</span><span class="p">)</span>
    <span class="n">fmt</span><span class="o">.</span><span class="n">Println</span><span class="p">(</span><span class="s">"</span><span class="se">\n</span><span class="s">After appending 1 more element:"</span><span class="p">)</span>
    <span class="n">fmt</span><span class="o">.</span><span class="n">Println</span><span class="p">(</span><span class="s">"Slice:"</span><span class="p">,</span> <span class="n">numbers</span><span class="p">)</span>
    <span class="n">fmt</span><span class="o">.</span><span class="n">Println</span><span class="p">(</span><span class="s">"Length:"</span><span class="p">,</span> <span class="nb">len</span><span class="p">(</span><span class="n">numbers</span><span class="p">))</span>
    <span class="n">fmt</span><span class="o">.</span><span class="n">Println</span><span class="p">(</span><span class="s">"Capacity:"</span><span class="p">,</span> <span class="nb">cap</span><span class="p">(</span><span class="n">numbers</span><span class="p">))</span>
<span class="p">}</span>
</code></pre>

</div>



<p>"<code>make([]int, 3, 6)</code> creates a slice with length 3 and capacity 6. The three elements start at zero—Go's default value for integers. We can use three elements immediately, but the backing array has room for six."</p>

<p>Eleanor ran the code:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight plaintext"><code>Slice: [0 0 0]
Length: 3
Capacity: 6

After setting values:
Slice: [10 20 30]

After appending 3 elements:
Slice: [10 20 30 40 50 60]
Length: 6
Capacity: 6

After appending 1 more element:
Slice: [10 20 30 40 50 60 70]
Length: 7
Capacity: 12
</code></pre>

</div>



<p>"Watch the capacity. We start with 6. After we fill it and append one more element, capacity jumps to 12. Go automatically allocated a bigger backing array and copied the data over."</p>

<p>"Why not just grow by one each time?"</p>

<p>"Performance. Allocating memory is expensive. Copying data is expensive. So Go grows the backing array strategically—it grows more aggressively for small slices, less aggressively for large ones. You pay the cost occasionally rather than constantly."</p>

<p>Let's visualize what happened:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight plaintext"><code>Initial state: make([]int, 3, 6)
Slice length: 3, Capacity: 6
[10, 20, 30, __, __, __]
 ↑           ↑
 used        available

After append(numbers, 40, 50, 60):
Slice length: 6, Capacity: 6
[10, 20, 30, 40, 50, 60]
 ↑                      ↑
 used (full capacity)

After append(numbers, 70):
Go allocates new backing array (capacity grows strategically)
Slice length: 7, Capacity: 12
[10, 20, 30, 40, 50, 60, 70, __, __, __, __, __]
 ↑                          ↑
 used                       available
</code></pre>

</div>



<p>Ethan traced through the diagram. "So when I append, I'm not always allocating new memory?"</p>

<p>"Exactly. If there's room in the backing array, <code>append</code> just adds the element and updates the length. When there's no room, it allocates a bigger array, copies everything over, then adds the element. That's why you always assign the result back—<code>append</code> might return a completely new slice."</p>

<p>"That sounds complicated."</p>

<p>"It is. But you don't have to think about it. <code>append</code> handles the complexity. You just write <code>numbers = append(numbers, 70)</code> and Go does the right thing."</p>

<p>Eleanor poured more coffee. "Now, here's something subtle. Watch this:"<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight go"><code><span class="k">package</span> <span class="n">main</span>

<span class="k">import</span> <span class="s">"fmt"</span>

<span class="k">func</span> <span class="n">main</span><span class="p">()</span> <span class="p">{</span>
    <span class="n">original</span> <span class="o">:=</span> <span class="p">[]</span><span class="kt">int</span><span class="p">{</span><span class="m">10</span><span class="p">,</span> <span class="m">20</span><span class="p">,</span> <span class="m">30</span><span class="p">}</span>
    <span class="n">fmt</span><span class="o">.</span><span class="n">Println</span><span class="p">(</span><span class="s">"Original:"</span><span class="p">,</span> <span class="n">original</span><span class="p">)</span>

    <span class="n">slice1</span> <span class="o">:=</span> <span class="n">original</span><span class="p">[</span><span class="m">0</span><span class="o">:</span><span class="m">2</span><span class="p">]</span>
    <span class="n">slice2</span> <span class="o">:=</span> <span class="n">original</span><span class="p">[</span><span class="m">1</span><span class="o">:</span><span class="m">3</span><span class="p">]</span>

    <span class="n">fmt</span><span class="o">.</span><span class="n">Println</span><span class="p">(</span><span class="s">"Slice1 (elements 0-1):"</span><span class="p">,</span> <span class="n">slice1</span><span class="p">)</span>
    <span class="n">fmt</span><span class="o">.</span><span class="n">Println</span><span class="p">(</span><span class="s">"Slice2 (elements 1-2):"</span><span class="p">,</span> <span class="n">slice2</span><span class="p">)</span>

    <span class="n">slice1</span><span class="p">[</span><span class="m">1</span><span class="p">]</span> <span class="o">=</span> <span class="m">999</span>

    <span class="n">fmt</span><span class="o">.</span><span class="n">Println</span><span class="p">(</span><span class="s">"</span><span class="se">\n</span><span class="s">After modifying slice1[1]:"</span><span class="p">)</span>
    <span class="n">fmt</span><span class="o">.</span><span class="n">Println</span><span class="p">(</span><span class="s">"Original:"</span><span class="p">,</span> <span class="n">original</span><span class="p">)</span>
    <span class="n">fmt</span><span class="o">.</span><span class="n">Println</span><span class="p">(</span><span class="s">"Slice1:"</span><span class="p">,</span> <span class="n">slice1</span><span class="p">)</span>
    <span class="n">fmt</span><span class="o">.</span><span class="n">Println</span><span class="p">(</span><span class="s">"Slice2:"</span><span class="p">,</span> <span class="n">slice2</span><span class="p">)</span>
<span class="p">}</span>
</code></pre>

</div>



<p>"This is called slicing—creating a new slice from an existing one. <code>original[0:2]</code> means 'give me a slice starting at index 0, up to but not including index 2.'"</p>

<p>She ran the code:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight plaintext"><code>Original: [10 20 30]
Slice1 (elements 0-1): [10 20]
Slice2 (elements 1-2): [20 30]

After modifying slice1[1]:
Original: [10 999 30]
Slice1: [10 999]
Slice2: [999 30]
</code></pre>

</div>



<p>Ethan stared at the output. "They're all connected?"</p>

<p>"They share the same backing array. When we modified <code>slice1[1]</code>, we changed the second element of the backing array. So <code>original</code> and <code>slice2</code> see the change too—they're all views into the same underlying data."</p>

<p>Eleanor drew another diagram:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight plaintext"><code>                original
              [10, 20, 30]
               ↓   ↓   ↓
Backing Array: [10, 20, 30]
               ↓   ↓
            slice1: [10, 20]
                   ↓   ↓
                slice2: [20, 30]

After slice1[1] = 999:
Backing Array: [10, 999, 30]
               ↓    ↓
            slice1: [10, 999]
                        ↓   ↓
                   slice2: [999, 30]
               ↓    ↓    ↓
              original: [10, 999, 30]
</code></pre>

</div>



<p>"This is powerful but dangerous. If you're not careful, modifying one slice can change another. Go's philosophy is to give you the power and trust you to use it wisely."</p>

<p>"How do I avoid that?"</p>

<p>"If you need an independent copy, use the <code>copy</code> function:"<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight go"><code><span class="k">package</span> <span class="n">main</span>

<span class="k">import</span> <span class="s">"fmt"</span>

<span class="k">func</span> <span class="n">main</span><span class="p">()</span> <span class="p">{</span>
    <span class="n">original</span> <span class="o">:=</span> <span class="p">[]</span><span class="kt">int</span><span class="p">{</span><span class="m">10</span><span class="p">,</span> <span class="m">20</span><span class="p">,</span> <span class="m">30</span><span class="p">}</span>
    <span class="n">independent</span> <span class="o">:=</span> <span class="nb">make</span><span class="p">([]</span><span class="kt">int</span><span class="p">,</span> <span class="nb">len</span><span class="p">(</span><span class="n">original</span><span class="p">))</span>
    <span class="nb">copy</span><span class="p">(</span><span class="n">independent</span><span class="p">,</span> <span class="n">original</span><span class="p">)</span>

    <span class="n">independent</span><span class="p">[</span><span class="m">1</span><span class="p">]</span> <span class="o">=</span> <span class="m">999</span>

    <span class="n">fmt</span><span class="o">.</span><span class="n">Println</span><span class="p">(</span><span class="s">"Original:"</span><span class="p">,</span> <span class="n">original</span><span class="p">)</span>
    <span class="n">fmt</span><span class="o">.</span><span class="n">Println</span><span class="p">(</span><span class="s">"Independent:"</span><span class="p">,</span> <span class="n">independent</span><span class="p">)</span>
<span class="p">}</span>
</code></pre>

</div>



<p>Eleanor ran it:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight plaintext"><code>Original: [10 20 30]
Independent: [10 999 30]
</code></pre>

</div>



<p>"Now they're separate. We allocated a new slice with <code>make</code>, then used <code>copy</code> to duplicate the data. Changes to one don't affect the other."</p>

<p>Let's look at common slice operations:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight go"><code><span class="k">package</span> <span class="n">main</span>

<span class="k">import</span> <span class="s">"fmt"</span>

<span class="k">func</span> <span class="n">main</span><span class="p">()</span> <span class="p">{</span>
    <span class="c">// Creating slices</span>
    <span class="n">slice1</span> <span class="o">:=</span> <span class="p">[]</span><span class="kt">int</span><span class="p">{</span><span class="m">1</span><span class="p">,</span> <span class="m">2</span><span class="p">,</span> <span class="m">3</span><span class="p">}</span>           <span class="c">// Literal</span>
    <span class="n">slice2</span> <span class="o">:=</span> <span class="nb">make</span><span class="p">([]</span><span class="kt">int</span><span class="p">,</span> <span class="m">5</span><span class="p">)</span>            <span class="c">// Make with length 5</span>
    <span class="n">slice3</span> <span class="o">:=</span> <span class="nb">make</span><span class="p">([]</span><span class="kt">int</span><span class="p">,</span> <span class="m">3</span><span class="p">,</span> <span class="m">10</span><span class="p">)</span>        <span class="c">// Make with length 3, capacity 10</span>

    <span class="c">// Appending (always assign back!)</span>
    <span class="n">slice1</span> <span class="o">=</span> <span class="nb">append</span><span class="p">(</span><span class="n">slice1</span><span class="p">,</span> <span class="m">4</span><span class="p">,</span> <span class="m">5</span><span class="p">,</span> <span class="m">6</span><span class="p">)</span>

    <span class="c">// Slicing</span>
    <span class="n">subset</span> <span class="o">:=</span> <span class="n">slice1</span><span class="p">[</span><span class="m">1</span><span class="o">:</span><span class="m">4</span><span class="p">]</span>               <span class="c">// Elements 1, 2, 3</span>

    <span class="c">// Iterating</span>
    <span class="k">for</span> <span class="n">i</span> <span class="o">:=</span> <span class="m">0</span><span class="p">;</span> <span class="n">i</span> <span class="o">&lt;</span> <span class="nb">len</span><span class="p">(</span><span class="n">slice1</span><span class="p">);</span> <span class="n">i</span><span class="o">++</span> <span class="p">{</span>
        <span class="n">fmt</span><span class="o">.</span><span class="n">Println</span><span class="p">(</span><span class="s">"Index:"</span><span class="p">,</span> <span class="n">i</span><span class="p">,</span> <span class="s">"Value:"</span><span class="p">,</span> <span class="n">slice1</span><span class="p">[</span><span class="n">i</span><span class="p">])</span>
    <span class="p">}</span>

    <span class="c">// Range (cleaner iteration)</span>
    <span class="k">for</span> <span class="n">index</span><span class="p">,</span> <span class="n">value</span> <span class="o">:=</span> <span class="k">range</span> <span class="n">slice1</span> <span class="p">{</span>
        <span class="n">fmt</span><span class="o">.</span><span class="n">Println</span><span class="p">(</span><span class="s">"Index:"</span><span class="p">,</span> <span class="n">index</span><span class="p">,</span> <span class="s">"Value:"</span><span class="p">,</span> <span class="n">value</span><span class="p">)</span>
    <span class="p">}</span>

    <span class="c">// Range, ignoring index</span>
    <span class="k">for</span> <span class="n">_</span><span class="p">,</span> <span class="n">value</span> <span class="o">:=</span> <span class="k">range</span> <span class="n">slice1</span> <span class="p">{</span>
        <span class="n">fmt</span><span class="o">.</span><span class="n">Println</span><span class="p">(</span><span class="s">"Value:"</span><span class="p">,</span> <span class="n">value</span><span class="p">)</span>
    <span class="p">}</span>
<span class="p">}</span>
</code></pre>

</div>



<p>"The <code>range</code> keyword is beautiful. It iterates over a slice, giving you both the index and the value. If you don't need the index, use <code>_</code> to ignore it."</p>

<p>Ethan typed along. "Why underscore?"</p>

<p>"In Go, if you declare a variable and don't use it, the compiler complains. But <code>_</code> is the 'blank identifier'—it means 'I know there's a value here, but I'm explicitly ignoring it.' It's honest code."</p>

<p>Eleanor closed her laptop. "That's enough for today. Arrays exist but you rarely use them. Slices are the workhorse—dynamic, flexible, efficient. They're backed by arrays, they can share data, and <code>append</code> handles growth automatically."</p>

<p>She finished her rugelach. "Next time: maps. How Go lets you store key-value pairs."</p>

<p>Ethan gathered the cups. "Eleanor?"</p>

<p>"Yes?"</p>

<p>"Why have arrays at all if slices are better?"</p>

<p>Eleanor smiled. "Performance. Slices have a tiny bit of overhead—that three-piece structure I mentioned. For most code, it doesn't matter. But in performance-critical code—game engines, scientific computing, low-level systems—arrays can be faster because they're just raw memory, no indirection."</p>

<p>"So it's about giving you the choice?"</p>

<p>"Always. Go gives you the simple tool that works 99% of the time, and the precise tool for when you need control. Arrays are precise. Slices are practical. Most code uses slices. But when you need every nanosecond, arrays are there."</p>

<p>Ethan climbed the stairs, thinking about layers and backing arrays and the way data could share space without you realizing. In Python, lists just grew. In Go, growth had a mechanism, visible and controllable.</p>

<p>Maybe that was the pattern again: Go showed you the machinery. It didn't hide the cost of operations. And in return, you could reason about performance, about memory, about what your code actually did.</p>




<h2>
  
  
  Key Concepts from Chapter 4
</h2>

<p><strong>Arrays:</strong> Fixed-size collections declared with <code>[n]type</code>. Size is part of the type. Rarely used directly in Go.</p>

<p><strong>Slices:</strong> Dynamic collections declared with <code>[]type</code>. The practical way to work with sequences in Go.</p>

<p><strong>Slice structure:</strong> Three components—pointer to backing array, length (number of elements), capacity (size of backing array).</p>

<p><strong>make function:</strong> <code>make([]type, length, capacity)</code> creates a slice. Capacity is optional; if omitted, it equals length.</p>

<p><strong>append function:</strong> <code>append(slice, elements...)</code> adds elements to a slice. Returns a new slice (may have new backing array if it grew). Always assign the result back.</p>

<p><strong>Slicing operation:</strong> <code>slice[start:end]</code> creates a new slice viewing part of an existing slice. Shares the backing array.</p>

<p><strong>len and cap:</strong> <code>len(slice)</code> returns current number of elements. <code>cap(slice)</code> returns capacity of backing array.</p>

<p><strong>copy function:</strong> <code>copy(destination, source)</code> copies elements from one slice to another. Creates independent data.</p>

<p><strong>range keyword:</strong> Iterates over slices (and other collections). Returns index and value: <code>for i, v := range slice</code>.</p>

<p><strong>Blank identifier:</strong> <code>_</code> explicitly ignores a value. Required when range gives you values you don't need.</p>

<p><strong>Shared backing arrays:</strong> Multiple slices can view the same underlying array. Modifying one can affect others.</p>

<p><strong>Zero values:</strong> Slices created with <code>make</code> are initialized with the zero value of their type (0 for int, "" for string, etc.).</p>




<p><em>Next chapter: Maps—where Ethan learns about key-value pairs, and Eleanor explains why Go's map syntax is so clean.</em></p>




<p><em>Aaron Rose is a software engineer and technology writer at <a href="https://www.tech-reader.blog" rel="noopener noreferrer">tech-reader.blog</a> and the author of <a href="https://amazon.com/author/aaron.rose" rel="noopener noreferrer">Think Like a Genius</a>.</em></p>

