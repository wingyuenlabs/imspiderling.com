---
Title: Day 007 on My journey to becoming a CSS Pro with Keith Grant
Description: 
Author: Adebiyi Itunuayo
Date: 2025-09-08T21:37:56.000Z
Robots: noindex,nofollow
Template: index
---
<p>Continuing from <a href="https://dev.to/ffff0000h/day-006-on-my-journey-to-becoming-a-css-pro-with-keith-grant-1fkg">Day 006</a>.<br>
Alright, so Keith says he's going to touch on <em>Layer</em>, <em>Scope proximity</em> later in the book.<br>
Let's talk about <strong>Inheritance</strong> and <strong>Special values</strong>.</p>

<p>Inheritance is simply the default "cascaded value" that descendant elements take from their parent element.<br>
Cascaded value means a declaration that “wins” the cascade, or is more specific after the conflict is resolved.</p>

<p><a href="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fgj1qmbcwbjuldua17wg4.png" class="article-body-image-wrapper"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fgj1qmbcwbjuldua17wg4.png" alt="Inheritance Tree" width="395" height="192"></a></p>

<p>Inheritance is done in a top-down manner.<br>
Certain properties of the <code>&lt;html&gt;</code> element are inherited by the <code>&lt;body&gt;</code> and its subsequent descendants. Knowing which properties are inherited is important to understanding why certain things happen when resolving issues in the cascade. Check <a href="https://www.w3.org/TR/CSS2/propidx.html" rel="noopener noreferrer">this</a> out to see a comprehensive list of the properties that are inherited by elements from their parents and by their descendants, keep it bookmarked!</p>



<p>Special values:<br>
Keith talks about four special values namely; <code>inherit</code>, <code>initial</code>, <code>unset</code>, <code>revert</code>. These values can be applied to any property.</p>

<ul>
<li> <code>inherit</code>: As the word suggests, when applied to a property enables inheritance of that property from the parent element even if it is not inherited by default, it forces it to be inherited.</li>
</ul>

<p>For example: The following with no style<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight plaintext"><code>&lt;!doctype html&gt;
&lt;html lang=”en-US”&gt;

&lt;head&gt;
    &lt;meta charset="utf-8" /&gt;
    &lt;link href="styles.css" rel="stylesheet" type="text/css" /&gt;
&lt;/head&gt;

&lt;body&gt;
    &lt;header class="page-header"&gt;
        &lt;h1 id="page-title" class="title"&gt;Journey to becoming a CSS Pro&lt;/h1&gt;
        &lt;nav&gt;
            &lt;ul id="main-nav" class="nav"&gt;
                &lt;li&gt;&lt;a href="/"&gt;Home&lt;/a&gt;&lt;/li&gt;
                &lt;li&gt;&lt;a href="/easy"&gt;Easy&lt;/a&gt;&lt;/li&gt;
                &lt;li&gt;&lt;a href="/intermediate"&gt;Intermediate&lt;/a&gt;&lt;/li&gt;
                &lt;li&gt;&lt;a href="/pro" class="featured"&gt;Pro&lt;/a&gt;&lt;/li&gt;
            &lt;/ul&gt;
        &lt;/nav&gt;
    &lt;/header&gt;
&lt;/body&gt;

&lt;/html&gt;
</code></pre>

</div>



<p>results in:</p>

<p><a href="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2F3fy7y1rzpmpu6b9sn0rb.png" class="article-body-image-wrapper"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2F3fy7y1rzpmpu6b9sn0rb.png" alt="Example of the inherit special value" width="559" height="315"></a></p>

<p>with style:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight plaintext"><code>.page-header {
    color:green;
}
</code></pre>

</div>



<p>results in:</p>

<p><a href="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fow44cwn2rbxgtlt4hzk7.png" class="article-body-image-wrapper"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fow44cwn2rbxgtlt4hzk7.png" alt="Example of the inherit special value" width="559" height="315"></a><br>
Notice:<br>
The <code>&lt;h1&gt;</code>, <code>&lt;ul&gt;</code>, and <code>&lt;li&gt;</code> elements inherit the <code>color</code> property from their parent. However the <code>&lt;a&gt;</code> element doesn't automatically inherit its parent's <code>color</code>. If we want it to take on the parent's <code>color</code>, we use the keyword <code>inherit</code>.</p>

<p>with the keyword:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight plaintext"><code>.page-header {
    color:green;
}

a {
    color: inherit;
}

</code></pre>

</div>



<p>results in:</p>

<p><a href="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fdcxx6615htsm4x2hcbpe.png" class="article-body-image-wrapper"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fdcxx6615htsm4x2hcbpe.png" alt="Example of the inherit special value" width="521" height="240"></a></p>

<ul>
<li> <code>initial</code>: This resets the value of a property to its default value (according to spec), not necessarily the browser's default rendering as the user-agent maker can assign different values for usability reasons.
Example:
For the <code>&lt;a&gt;</code> element;
<a href="https://www.w3.org/TR/css-color-4/" rel="noopener noreferrer">Spec </a>says: <code>color</code> <strong>initial</strong> value is <code>CanvasText</code> (usually black or a similarly dark color)</li>
</ul>

<p>User-agents usually style <code>&lt;a&gt;</code> with <strong>blue for unvisited</strong> and <strong>purple for visited</strong> in their default stylesheet.</p>

<p>with the <code>initial</code> keyword:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight plaintext"><code>.page-header {
    color:green;
}

a {
    color: initial;
}
</code></pre>

</div>



<p>it results in:</p>

<p><a href="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2F2cu6yigiodkbhw7gy0y2.png" class="article-body-image-wrapper"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2F2cu6yigiodkbhw7gy0y2.png" alt="Example of the initial special value" width="521" height="240"></a><br>
<code>initial</code> ignores the user-agent style and goes straight back to the specification's style.</p>

<ul>
<li> <code>unset</code>: This is a combination of both <code>inital</code> and <code>inherit</code>, in that when applied:

<ul>
<li> to an inherited property, it sets the value to <code>inherit</code>
</li>
<li> to a non-inherited property, it sets the value to <code>initial</code>
For example:
</li>
</ul>


</li>

</ul>

<div class="highlight js-code-highlight">
<pre class="highlight plaintext"><code>.page-header {
    color:green;
}

a {
    color: unset;
}
</code></pre>

</div>



<p>The <code>color</code> property is a property that can be inherited, hence, setting the value to <code>unset</code> applies the <code>inherit</code>  function to it and the element it is applied to takes on the color value of its parent element. <br>
  which results in:</p>

<p><a href="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fimh4ox1p0rw3t47073z6.png" class="article-body-image-wrapper"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fimh4ox1p0rw3t47073z6.png" alt="Example of the inherit special value" width="521" height="240"></a><br>
If <code>color</code> were not an inherited property, it would be set to the <code>initial</code> value; the spec style value, whatever value it is.</p>

<ul>
<li> <code>revert</code>: This keyword basically resets a property's cascaded value to what it would have been if no declarations from the current cascade origin were present.
If the declaration using revert comes from the author, it drops author-level declaration for that property and falls back to the user-level declaration (if one exists), otherwise the user-agent declaration for that property is used.
Example:
The following resets the <code>&lt;a&gt;</code> element's <code>color</code> to the user-agent default style of the blue/purple color, depending on whether they've been visited or not.
</li>
</ul>

<div class="highlight js-code-highlight">
<pre class="highlight plaintext"><code>.page-header {
    color:green;
}

a {
    color: revert;
}

</code></pre>

</div>



<p>which results in:</p>

<p><a href="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fwbtmz23u5inqhxm4qc1t.png" class="article-body-image-wrapper"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fwbtmz23u5inqhxm4qc1t.png" alt="Example of the revert special value" width="521" height="240"></a></p>

<p>N.B:<br>
Some links appearing the images have been visited, hence the uneven colors.</p>

<p>I believe anyone can become a CSS Pro, and therefore I am. </p>

