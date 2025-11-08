---
Title: JavaScript Ternary Operator
Description: 
Author: Meg
Date: 2025-11-08T21:04:24.000Z
Robots: noindex,nofollow
Template: index
---
<p>One of my favorite parts of the JavaScript language is the JavaScript Ternary Operator.</p>

<p>At heart, it’s a succinct, clean way of writing an if/else statement.</p>

<p>You start by asking a question, <br>
"Is this item a circle?"</p>

<p>If the item is a circle, it takes path one<br>
If the item is not a circle, it takes path two</p>

<p>You would write it in JavaScript like this:</p>

<p>circle ? true : false</p>

<p>A ternary operator can also be stacked or ‘nested’. </p>

<p>In this example the item we’ll use is a triangle.</p>

<p>You would use the same first condition:</p>

<p>circle ? true : _____</p>

<p>And replace the false condition with a second ternary:</p>

<p>triangle ? true : false</p>

<p>So a nested ternary would be:</p>

<p>circle ? true : triangle ? true : false</p>

<p>You can stack as many conditions as you like and adjust the formatting to visually appear like a switch statement:</p>

<p>circle ? <br>
true : triangle ? <br>
true : square ? <br>
true : rectangle ?</p>

<p>You are also able to return with a ternary operator by listing "return" before your condition:</p>

<p>return circle ? true : false </p>

<p>Unpacking this statement, it should read:<br>
Is the item a circle? <br>
If the item is a circle, return true<br>
If the item is not a circle, return false</p>

<p>By using "return" with a ternary you must always return from either condition. It's not able to be used in instances where you only want to return on true, but not false, and vice versa.</p>

<p>All in all, I hope this helped understand the JavaScript Ternary Operator a bit better and why I find it so visually clean and fun to use.</p>

