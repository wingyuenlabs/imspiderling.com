---
Title: A Strange Permutation Substring Sequence: Introducing OEIS A359012
Description: 
Author: John Samuel
Date: 2026-03-13T21:53:42.000Z
Robots: noindex,nofollow
Template: index
---
<p>I’ve been exploring a new integer sequence on the OEIS, A359012, and turned it into a small experiment in “data‑driven number theory”. This post briefly introduces the sequence and shares links to the live dataset and code.</p>

<h2>
  
  
  What is A359012?
</h2>

<p>A359012 tracks integers (k) that reappear inside permutation numbers (xPy) when (k) is formed by concatenating the digits of (x) and (y). </p>

<p>More concretely: </p>

<ul>
<li>Take natural numbers (x) and (y).
</li>
<li>Form (k) by writing the digits of (x) followed by the digits of (y).
</li>
<li>Compute (xPy), the number of permutations of (x) objects taken (y) at a time.
</li>
<li>If the decimal expansion of (xPy) contains (k) as a contiguous substring, then (k) belongs to A359012. </li>
</ul>

<p>The first few terms (with one witness each) look like this: </p>

<div class="table-wrapper-paragraph"><table>
<thead>
<tr>
<th>k</th>
<th>x</th>
<th>y</th>
<th>xPy (permutations)</th>
</tr>
</thead>
<tbody>
<tr>
<td>318</td>
<td>31</td>
<td>8</td>
<td>318073392000</td>
</tr>
<tr>
<td>557</td>
<td>55</td>
<td>7</td>
<td>1022755734000</td>
</tr>
<tr>
<td>692</td>
<td>69</td>
<td>2</td>
<td>4692</td>
</tr>
<tr>
<td>729</td>
<td>72</td>
<td>9</td>
<td>30885807297945600</td>
</tr>
<tr>
<td>2226</td>
<td>222</td>
<td>6</td>
<td>111822261510960</td>
</tr>
</tbody>
</table></div>

<p>In each case, (k) appears as a substring of the decimal representation of (xPy). </p>

<h2>
  
  
  A few empirical facts
</h2>

<p>Using a brute‑force generator up to (k &lt; 10^6), the current dataset looks like this:</p>

<ul>
<li>712 terms below 997 398. </li>
<li>Term lengths are heavily concentrated at 6 digits (565 out of 712 terms). </li>
<li>708 of 712 witnesses end in zero.</li>
<li>Only 4 palindromes appear so far: 9999, 29092, 343343, 805508. </li>
<li>The largest gap between consecutive terms is 26 763. </li>
</ul>

<p>So A359012 is quite sparse, with strong biases in how its terms are embedded inside permutation values. </p>

<h2>
  
  
  Links
</h2>

<p>If you want to dig into the sequence or the code, here are the main entry points:</p>

<ul>
<li><p>OEIS entry (definition and references):  <a href="https://oeis.org/A359012" rel="noopener noreferrer">https://oeis.org/A359012</a> </p></li>
<li><p>Live “Sequence Atlas” with the canonical dataset and dashboards (all powered directly from the CSV):  <a href="https://johnsamuel.info/A359012/" rel="noopener noreferrer">https://johnsamuel.info/A359012/</a> </p></li>
<li><p>Source code, generator, and analysis scripts (Python + static site): <a href="https://github.com/johnsamuelwrites/A359012" rel="noopener noreferrer">https://github.com/johnsamuelwrites/A359012</a> </p></li>
</ul>

<p>Developers might be especially interested in how the project treats an OEIS sequence as a reproducible data pipeline: generator → CSV → analysis → visual exploration. </p>

