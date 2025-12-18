---
Title: Introducing Marlin
Description: 
Author: Toby Inkster
Date: 2025-12-18T21:44:05.000Z
Robots: noindex,nofollow
Template: index
---
<p>Does the Perl world need another object-oriented programming framework?</p>

<p>To be honest, probably not.</p>

<p>But here’s why you might want to give Marlin a try anyway.</p>

<ul>
<li><p>Most of your constructors and accessors will be implemented in XS and be really, really fast.</p></li>
<li><p>If you accept a few basic principles like “attributes should usually be read-only”, it can be really, really concise to declare a class and its attributes.</p></li>
</ul>

<h2>
  
  
  An example
</h2>



<div class="highlight js-code-highlight">
<pre class="highlight perl"><code>  <span class="k">use</span> <span class="nv">v5</span><span class="mf">.20.0</span><span class="p">;</span>
  <span class="k">use</span> <span class="nv">experimental</span> <span class="sx">qw(signatures)</span><span class="p">;</span>

  <span class="c1"># Import useful constants, types, etc.</span>
  <span class="k">use</span> <span class="nn">Marlin::</span><span class="nv">Util</span> <span class="o">-</span><span class="nv">all</span><span class="p">,</span> <span class="o">-</span><span class="nv">lexical</span><span class="p">;</span>
  <span class="k">use</span> <span class="nn">Types::</span><span class="nv">Common</span> <span class="o">-</span><span class="nv">all</span><span class="p">,</span> <span class="o">-</span><span class="nv">lexical</span><span class="p">;</span>

  <span class="nb">package</span> <span class="nv">Person</span> <span class="p">{</span>
    <span class="k">use</span> <span class="nv">Marlin</span>
      <span class="p">'</span><span class="s1">given_name!</span><span class="p">'</span>   <span class="o">=&gt;</span> <span class="nv">NonEmptyStr</span><span class="p">,</span>
      <span class="p">'</span><span class="s1">family_name!</span><span class="p">'</span>  <span class="o">=&gt;</span> <span class="nv">NonEmptyStr</span><span class="p">,</span>
      <span class="p">'</span><span class="s1">name_style</span><span class="p">'</span>    <span class="o">=&gt;</span> <span class="p">{</span> <span class="s">enum</span> <span class="o">=&gt;</span> <span class="p">[</span><span class="sx">qw/western eastern/</span><span class="p">],</span> <span class="s">default</span> <span class="o">=&gt;</span> <span class="p">'</span><span class="s1">western</span><span class="p">'</span> <span class="p">},</span>
      <span class="p">'</span><span class="s1">full_name</span><span class="p">'</span>     <span class="o">=&gt;</span> <span class="p">{</span> <span class="s">is</span> <span class="o">=&gt;</span> <span class="nv">lazy</span><span class="p">,</span> <span class="s">builder</span> <span class="o">=&gt;</span> <span class="nv">true</span> <span class="p">},</span>
      <span class="p">'</span><span class="s1">birth_date?</span><span class="p">';</span>

    <span class="k">sub </span><span class="nf">_build_full_name</span> <span class="p">( $self ) {</span>
      <span class="k">return</span> <span class="nb">sprintf</span><span class="p">(</span> <span class="p">'</span><span class="s1">%s %s</span><span class="p">',</span> <span class="nb">uc</span><span class="p">(</span><span class="nv">$self</span><span class="o">-&gt;</span><span class="nv">family_name</span><span class="p">),</span> <span class="nv">$self</span><span class="o">-&gt;</span><span class="nv">given_name</span> <span class="p">)</span>
        <span class="k">if</span> <span class="nv">$self</span><span class="o">-&gt;</span><span class="nv">name_style</span> <span class="ow">eq</span> <span class="p">'</span><span class="s1">eastern</span><span class="p">';</span>
      <span class="k">return</span> <span class="nb">sprintf</span><span class="p">(</span> <span class="p">'</span><span class="s1">%s %s</span><span class="p">',</span> <span class="nv">$self</span><span class="o">-&gt;</span><span class="nv">given_name</span><span class="p">,</span> <span class="nv">$self</span><span class="o">-&gt;</span><span class="nv">family_name</span> <span class="p">);</span>
    <span class="p">}</span>
  <span class="p">}</span>

  <span class="nb">package</span> <span class="nv">Payable</span> <span class="p">{</span>
    <span class="k">use</span> <span class="nn">Marlin::</span><span class="nv">Role</span>
      <span class="o">-</span><span class="s">requires</span> <span class="o">=&gt;</span> <span class="p">[</span> <span class="p">'</span><span class="s1">bank\_details</span><span class="p">'</span> <span class="p">];</span>

    <span class="k">sub </span><span class="nf">make_payment</span> <span class="p">( $self ) {</span>
      <span class="o">...</span><span class="p">;</span>
    <span class="p">}</span>
  <span class="p">}</span>

  <span class="nb">package</span> <span class="nv">Employee</span> <span class="p">{</span>
    <span class="k">use</span> <span class="nv">Marlin</span>
      <span class="o">-</span><span class="s">extends</span> <span class="o">=&gt;</span> <span class="p">[</span> <span class="p">'</span><span class="s1">Person</span><span class="p">'</span> <span class="p">],</span>
      <span class="o">-</span><span class="s">with</span>    <span class="o">=&gt;</span> <span class="p">[</span> <span class="p">'</span><span class="s1">Payable</span><span class="p">'</span> <span class="p">],</span>
      <span class="p">'</span><span class="s1">bank_details!</span><span class="p">'</span> <span class="o">=&gt;</span> <span class="nv">HashRef</span><span class="p">,</span>
      <span class="p">'</span><span class="s1">employee_id!</span><span class="p">'</span>  <span class="o">=&gt;</span> <span class="nv">Int</span><span class="p">,</span>
      <span class="p">'</span><span class="s1">manager?</span><span class="p">'</span>      <span class="o">=&gt;</span> <span class="p">{</span> <span class="s">isa</span> <span class="o">=&gt;</span> <span class="p">'</span><span class="s1">Employee</span><span class="p">'</span> <span class="p">};</span>
  <span class="p">}</span>

  <span class="k">my</span> <span class="nv">$manager</span> <span class="o">=</span> <span class="nv">Employee</span><span class="o">-&gt;</span><span class="k">new</span><span class="p">(</span>
    <span class="s">given_name</span>    <span class="o">=&gt;</span> <span class="p">'</span><span class="s1">Simon</span><span class="p">',</span>
    <span class="s">family_name</span>   <span class="o">=&gt;</span> <span class="p">'</span><span class="s1">Lee</span><span class="p">',</span>
    <span class="s">name_style</span>    <span class="o">=&gt;</span> <span class="p">'</span><span class="s1">eastern</span><span class="p">',</span>
    <span class="s">employee_id</span>   <span class="o">=&gt;</span> <span class="mi">1</span><span class="p">,</span>
    <span class="s">bank_details</span>  <span class="o">=&gt;</span> <span class="p">{},</span>
  <span class="p">);</span>

  <span class="k">my</span> <span class="nv">$staff</span> <span class="o">=</span> <span class="nv">Employee</span><span class="o">-&gt;</span><span class="k">new</span><span class="p">(</span>
    <span class="s">given_name</span>    <span class="o">=&gt;</span> <span class="p">'</span><span class="s1">Lea</span><span class="p">',</span>
    <span class="s">family_name</span>   <span class="o">=&gt;</span> <span class="p">'</span><span class="s1">Simons</span><span class="p">',</span>
    <span class="s">employee_id</span>   <span class="o">=&gt;</span> <span class="mi">2</span><span class="p">,</span>
    <span class="s">bank_details</span>  <span class="o">=&gt;</span> <span class="p">{},</span>
    <span class="s">manager</span>       <span class="o">=&gt;</span> <span class="nv">$manager</span><span class="p">,</span>
  <span class="p">);</span>

  <span class="nb">printf</span><span class="p">(</span>
    <span class="p">"</span><span class="s2">%s's manager is %s.</span><span class="se">\n</span><span class="p">",</span>
    <span class="nv">$staff</span><span class="o">-&gt;</span><span class="nv">full_name</span><span class="p">,</span>
    <span class="nv">$staff</span><span class="o">-&gt;</span><span class="nv">manager</span><span class="o">-&gt;</span><span class="nv">full_name</span><span class="p">,</span>
  <span class="p">)</span> <span class="k">if</span> <span class="nv">$staff</span><span class="o">-&gt;</span><span class="nv">has_manager</span><span class="p">;</span>
</code></pre>

</div>



<p>Some things you might notice:</p>

<ul>
<li><p>It supports most of the features of Moose… or most of the ones you <em>actually use</em> anyway.</p></li>
<li><p>Declaring an attribute is often as simple as listing it’s name on the <code>use Marlin</code> line.</p></li>
<li><p>It can be followed by some options, but if you’re happy with Marlin’s defaults (read-only attributes), it doesn’t need to be.</p></li>
<li><p>You can use the <code>!</code> to quickly mark an attribute as required instead of the longer <code>{ required =&gt; true }</code>.</p></li>
<li><p>You can use <code>?</code> to request a predicate method instead of the longer <code>{ predicate =&gt; true }</code>.</p></li>
</ul>

<h2>
  
  
  Benchmarks
</h2>

<p>My initial benchmarking shows that Marlin is <em>fast</em>.</p>

<h3>
  
  
  Constructors
</h3>



<div class="highlight js-code-highlight">
<pre class="highlight plaintext"><code>         Rate   Tiny  Plain    Moo  Moose Marlin   Core
Tiny   1317/s     --    -2%   -48%   -53%   -54%   -72%
Plain  1340/s     2%     --   -47%   -53%   -53%   -72%
Moo    2527/s    92%    89%     --   -11%   -12%   -47%
Moose  2828/s   115%   111%    12%     --    -2%   -40%
Marlin 2873/s   118%   114%    14%     2%     --   -39%
Core   4727/s   259%   253%    87%    67%    65%     --
</code></pre>

</div>



<p>Only the new Perl core <code>class</code> keyword generates a constructor faster than Marlin’s. And it is significantly faster; there’s no denying that. However, object construction is only part of what you are likely to need.</p>

<h3>
  
  
  Accessors
</h3>



<div class="highlight js-code-highlight">
<pre class="highlight plaintext"><code>          Rate   Tiny  Moose  Plain   Core    Moo Marlin
Tiny   17345/s     --    -1%    -3%    -7%   -36%   -45%
Moose  17602/s     1%     --    -2%    -6%   -35%   -44%
Plain  17893/s     3%     2%     --    -4%   -34%   -44%
Core   18732/s     8%     6%     5%     --   -31%   -41%
Moo    27226/s    57%    55%    52%    45%     --   -14%
Marlin 31688/s    83%    80%    77%    69%    16%     --
</code></pre>

</div>



<p>By accessors, I’m talking about not just standard getter and setters, but also predicate methods and clearers. Marlin and Moo both use Class::XSAccessor when possible, giving them a significant lead over the others. Marlin uses some sneaky tricks to squeeze out a little bit of extra performance by creating aliases for parent class methods directly in the child class symbol tables, allowing Perl to bypass a lot of the normal method resolution stuff.</p>

<p>I really expected <code>class</code> to do a lot better than it does. Its readers and writers are basically implemented in pure Perl currently, though I guess there’s scope to improve them in future releases.</p>

<h3>
  
  
  Native Traits / Handles Via / Delegations
</h3>



<div class="highlight js-code-highlight">
<pre class="highlight plaintext"><code>         Rate   Tiny   Core  Plain  Moose    Moo Marlin
Tiny    675/s     --   -56%   -57%   -59%   -61%   -61%
Core   1518/s   125%     --    -4%    -8%   -13%   -13%
Plain  1581/s   134%     4%     --    -4%    -9%   -10%
Moose  1642/s   143%     8%     4%     --    -5%    -6%
Moo    1736/s   157%    14%    10%     6%     --    -1%
Marlin 1752/s   160%    15%    11%     7%     1%     --
</code></pre>

</div>



<p>If you don’t know what I mean by native traits, it’s the ability to create small methods like this:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight perl"><code>  <span class="k">sub </span><span class="nf">add_language</span> <span class="p">( $self, $lang ) {</span>
    <span class="nb">push</span> <span class="nv">$self</span><span class="o">-&gt;</span><span class="nv">languages</span><span class="o">-&gt;</span><span class="err">@</span><span class="o">*</span><span class="p">,</span> <span class="nv">$lang</span><span class="p">;</span>
  <span class="p">}</span>
</code></pre>

</div>



<p>As part of the attribute definition:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight perl"><code>  <span class="k">use</span> <span class="nv">Marlin</span>
    <span class="s">languages</span> <span class="o">=&gt;</span> <span class="p">{</span>
      <span class="s">is</span>           <span class="o">=&gt;</span> <span class="p">'</span><span class="s1">ro</span><span class="p">',</span>
      <span class="s">isa</span>          <span class="o">=&gt;</span> <span class="nv">ArrayRef</span><span class="p">[</span><span class="nv">Str</span><span class="p">],</span>
      <span class="s">default</span>      <span class="o">=&gt;</span> <span class="o">[]</span><span class="p">,</span>
      <span class="s">handles_via</span>  <span class="o">=&gt;</span> <span class="p">'</span><span class="s1">Array</span><span class="p">',</span>
      <span class="s">handles</span>      <span class="o">=&gt;</span> <span class="p">{</span> <span class="s">add_language</span> <span class="o">=&gt;</span> <span class="p">'</span><span class="s1">push</span><span class="p">',</span> <span class="s">count_languages</span> <span class="o">=&gt;</span> <span class="p">'</span><span class="s1">count</span><span class="p">'</span> <span class="p">},</span>
    <span class="p">};</span>
</code></pre>

</div>



<p>There’s not an awful lot of difference between the performance of most of these, but Marlin <em>slightly</em> wins. Marlin and Moose are also the only frameworks that include this out of the box without needing extension modules.</p>

<p>By the way, that <code>default =&gt; []</code> was not a typo. You can set an empty arrayref or empty hashref as a default, and Marlin will assume you meant something like <code>default =&gt; sub { [] }</code>, but it cleverly skips over needing to actually call the coderef (slow), instead creating a reference to a new empty array in XS (fast)!</p>

<h3>
  
  
  Combined
</h3>



<div class="highlight js-code-highlight">
<pre class="highlight plaintext"><code>         Rate   Tiny  Plain   Core  Moose    Moo Marlin
Tiny    545/s     --   -48%   -56%   -58%   -60%   -64%
Plain  1051/s    93%     --   -16%   -19%   -22%   -31%
Core   1249/s   129%    19%     --    -4%    -8%   -18%
Moose  1304/s   139%    24%     4%     --    -4%   -14%
Moo    1355/s   148%    29%     8%     4%     --   -11%
Marlin 1519/s   179%    45%    22%    17%    12%     --
</code></pre>

</div>



<p>A realistic bit of code that constructs some objects and calls a bunch of accessors and delegations on them. Marlin performs very well.</p>

<h2>
  
  
  Lexical accessors and private attributes
</h2>

<p>Marlin has first class support for lexical methods!<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight perl"><code>  <span class="k">use</span> <span class="nv">v5</span><span class="mf">.42.0</span><span class="p">;</span>

  <span class="nb">package</span> <span class="nv">Widget</span> <span class="p">{</span>
    <span class="k">use</span> <span class="nv">Marlin</span>
      <span class="s">name</span>        <span class="o">=&gt;</span> <span class="p">{</span> <span class="s">isa</span> <span class="o">=&gt;</span> <span class="nv">Str</span> <span class="p">},</span>
      <span class="s">internal_id</span> <span class="o">=&gt;</span> <span class="p">{</span> <span class="s">reader</span> <span class="o">=&gt;</span> <span class="p">'</span><span class="s1">my internal_id</span><span class="p">',</span> <span class="s">storage</span> <span class="o">=&gt;</span> <span class="p">'</span><span class="s1">PRIVATE</span><span class="p">'</span> <span class="p">};</span>

    <span class="o">...</span>

    <span class="nb">printf</span> <span class="p">"</span><span class="s2">%d: %s</span><span class="se">\n</span><span class="p">",</span> <span class="nv">$w</span><span class="o">-&gt;</span><span class="nv">&amp;internal_id</span><span class="p">,</span> <span class="nv">$w</span><span class="o">-&gt;</span><span class="nv">name</span><span class="p">,</span> 
  <span class="p">}</span>

  <span class="c1"># dies because internal_id is lexically scoped</span>
  <span class="nv">Widget</span><span class="o">-&gt;</span><span class="k">new</span><span class="o">-&gt;</span><span class="nv">&amp;internal_id</span><span class="p">;</span>
</code></pre>

</div>



<p>Support for the <code>-&gt;&amp;</code> operator was added in Perl 5.42. On older Perls (from Perl 5.12 onwards), lexical methods are still supported but you need to use function call syntax (<code>internal_id($w)</code>).</p>

<p>The <code>storage =&gt; "PRIVATE"</code> hint tells Marlin to use inside-out storage for that attribute, meaning that trying to access the internal_id by poking into the object’s internals (<code>$obj-&gt;{internal_id}</code>) won’t work.</p>

<p>This gives you true private attributes.</p>

<p>On Perl 5.18 and above, you can of course declare lexical methods using the normal <code>my sub foo</code> syntax, so you have private attributes as well as private methods.</p>

<h2>
  
  
  Constant attributes
</h2>



<div class="highlight js-code-highlight">
<pre class="highlight perl"><code>  <span class="nb">package</span> <span class="nv">Person</span> <span class="p">{</span>
    <span class="k">use</span> <span class="nv">Marlin</span>
      <span class="s">name</span>         <span class="o">=&gt;</span> <span class="p">{</span> <span class="s">isa</span> <span class="o">=&gt;</span> <span class="nv">Str</span><span class="p">,</span> <span class="s">required</span> <span class="o">=&gt;</span> <span class="nv">true</span> <span class="p">},</span>
      <span class="s">species_name</span> <span class="o">=&gt;</span> <span class="p">{</span> <span class="s">isa</span> <span class="o">=&gt;</span> <span class="nv">Str</span><span class="p">,</span> <span class="s">constant</span> <span class="o">=&gt;</span> <span class="p">"</span><span class="s2">Homo sapiens</span><span class="p">"</span> <span class="p">};</span>
  <span class="p">}</span>
</code></pre>

</div>



<p>Constant attributes are declared like regular attributes, but are always very read-only and illegal to pass to the constructor.</p>

<p>Like other attributes, they support delegations, provided the delegated method isn’t one which could change the value.</p>

<h2>
  
  
  Perl version support
</h2>

<p>Although some of the lexical features need newer versions of Perl, Marlin runs on Perl versions as old as 5.8.8.</p>

<h2>
  
  
  Future directions
</h2>

<p>Some ideas I’ve had:</p>

<ul>
<li>  If Moose is loaded, create meta object protocol stuff for Marlin classes and roles, like Moo does.</li>
</ul>

