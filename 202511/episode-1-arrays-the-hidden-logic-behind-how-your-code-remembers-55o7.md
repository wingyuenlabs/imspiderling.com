---
Title: Episode 1: Arrays — The Hidden Logic Behind How Your Code Remembers
Description: 
Author: nedajahanfar
Date: 2025-11-09T21:45:44.000Z
Robots: noindex,nofollow
Template: index
---
<p><strong>What Is an Array?</strong></p>

<p>In short, an array is a sequence of values stored contiguously (back-to-back) in memory.<br>
All values in the array must be of the same data type (e.g., integers, characters, etc.).</p>

<p>When an array is created, the computer allocates a continuous block of memory large enough for all its elements.<br>
Unlike individual variables — which might be placed randomly in memory — array elements are stored side by side.</p>

<p>For example, an array of three integers will occupy 12 bytes in total (since an integer is typically 4 bytes), with the four bytes for each integer stored immediately next to the next.</p>

<p><strong>Why Arrays Are Zero-Indexed</strong></p>

<p>When an array is created, the computer:</p>

<p>Allocates a contiguous block of memory large enough for all elements.<br>
Stores the address of the first element (called the base address).<br>
The variable name (e.g., numbers) doesn’t hold all the elements — it only holds this starting address.</p>

<p>So when you write:</p>

<p>let numbers = [10, 20, 30];</p>

<p>The computer knows:</p>

<p>The base address (say, 0x1000) points to the first element (10).</p>

<p>Each integer takes 4 bytes.</p>

<p>Therefore:</p>

<p>numbers[0] → address 0x1000<br><br>
numbers[1] → address 0x1004<br><br>
numbers[2] → address 0x1008</p>

<p>This is why arrays start at index 0 —<br>
to access any element, the computer uses the formula:</p>

<p>address = base_address + (index × size_of_each_element)</p>

<p>If indexing started at 1, every lookup would require an extra subtraction (index - 1) —<br>
an unnecessary calculation. Zero-based indexing keeps memory access clean and direct.</p>

<p><strong>When and Why to Use Arrays</strong></p>

<p>Arrays are one of the most common choices for storing data — but when are they actually the best choice?</p>

<p>In general, there are four main operations you might want to perform on a dataset:</p>

<p>Insertion (adding a new item)</p>

<p>Deletion (removing an item)</p>

<p>Selection / Access (finding or reading an item)</p>

<p>Sorting (reordering items)</p>

<p>Let’s see how arrays handle each of these, both in the best and worst cases.</p>

<p><strong>Deletion</strong></p>

<p>It depends on where you delete from:</p>

<p>From the end: very efficient → O(1)<br>
The computer simply forgets the last item — no need to move anything.</p>

<p>From the beginning or middle: slower → O(n)<br>
Imagine removing the first book from a shelf — every other book needs to shift one position left to fill the empty spot.</p>

<p>So deleting the first element is the worst case since the program has to move n–1 items.</p>

<p><strong>Insertion</strong></p>

<p>Again, position matters:</p>

<p>Add to the end: easy → O(1)<br>
The computer just adds one more element to the next empty slot.</p>

<p>Add to the beginning or middle: expensive → O(n)<br>
All the other items need to shift one index to make room.</p>

<p><strong>Selection (Accessing or Searching)</strong></p>

<p>If you already know the index: instant → O(1)<br>
Arrays shine here — the address formula lets you jump directly to any element.</p>

<p>If you don’t know the index: you must look through all items → O(n)</p>

<p><strong>Sorting Is Special..</strong></p>

<p>While most array operations have fixed theoretical costs defined by the structure itself<br>
(e.g., deletion ≈ O(n), insertion ≈ O(n)), sorting is unique because your algorithm choice can change its Big O class entirely —<br>
from O(n²) with simple methods like Bubble Sort to O(n log n) with smarter ones like Merge Sort.</p>

<p>For example:</p>

<p>Sorting an array with Bubble Sort → O(n²)<br>
Sorting the same array with Merge Sort → O(n log n)<br>
Here, the data is the same, but the algorithm changes the efficiency.</p>

<p>Why?<br>
Because sorting isn’t an inherent feature of arrays — it’s an operation applied externally using an algorithm.<br>
In contrast, operations like “add” or “access” are built into how arrays work.</p>

<p>LeetCode Example — Insertion in Arrays</p>

<p>Problem: LeetCode #1389 — Create Target Array in the Given Order</p>

<p>Task:<br>
Given two arrays of integers, nums and index, create a target array following these rules:</p>

<p>Start with an empty array.</p>

<p>For each i, insert nums[i] at position index[i] inside the target array.</p>

<p>Repeat until all elements are processed.</p>

<p>It is guaranteed that all insertions will be valid.</p>

<p>As mentioned earlier, adding to the end of an array (like using .push() in JavaScript) is fast — O(1).<br>
But inserting elements at random positions forces the computer to shift all other elements, which costs O(n) per insertion.</p>

<p>Since this problem requires doing that n times, the total cost becomes O(n²).</p>

<p><em>Step-by-Step Logic</em></p>

<p>When thinking through the solution:</p>

<p>Loop through the index array from start to end.</p>

<p>For each index i, find the corresponding number nums[i].</p>

<p>Insert nums[i] into the target array at position index[i].</p>

<p>If that position is already occupied, shift all following elements right.</p>

<p>Translating That Into Code:</p>

<p><code>function arrayMaker(index, nums) {<br>
  let target = [];<br>
  for (let i = 0; i &lt; index.length; i++) {<br>
    target.splice(index[i], 0, nums[i]);<br>
  }<br>
  return target;<br>
}</code></p>

<p>The .splice() method in JavaScript handles insertion directly.</p>

<p>It takes three arguments:</p>

<p>The index where to insert.</p>

<p>How many elements to delete (here, 0).</p>

<p>The new element to insert.</p>

<p>Example:</p>

<p><code>target.splice(2, 0, 5);</code></p>

<p>Inserts 5 at position 2, shifting everything after it to the right.</p>

<p>What We Learned</p>

<p>Inserting at the end of an array → O(1)</p>

<p>Inserting in the middle or beginning → O(n)</p>

<p>Doing this repeatedly → O(n²)</p>

<p>Arrays are great for reading and appending data,<br>
but repeated insertions at random positions make them inefficient —<br>
that’s where other data structures like linked lists come in.</p>

<p>Deletion Example — LeetCode #27</p>

<p>Problem: Remove Element</p>

<p>Given an integer array nums and a value val,<br>
remove all occurrences of val in-place and return the new length of the array.</p>

<p><code>function removeElement(nums, val) {<br>
  let k = 0;<br>
  for (let i = 0; i &lt; nums.length; i++) {<br>
    if (nums[i] != val) {<br>
      nums[k] = nums[i];<br>
      k++;<br>
    }<br>
  }<br>
  return k;<br>
}</code></p>

<p>Explanation</p>

<p>We use two pointers:</p>

<p>i → reads each element</p>

<p>k → marks the position where the next valid element should go</p>

<p>If the current value isn’t equal to val, we copy it to nums[k] and move k forward.<br>
This overwrites unwanted values as we go — deleting in-place without creating a new array.</p>

<p>Physical deletion (using .splice() or .slice() repeatedly) is not optimal,<br>
because each deletion shifts all following elements one step left —<br>
making it O(n²) when done multiple times.</p>

<p>Instead, overwriting valid elements keeps it O(n) and efficient.</p>

<p><em>LeetCode #1365 — Counting Smaller Numbers</em></p>

<p>Problem: How Many Numbers Are Smaller Than the Current Number</p>

<p>We return, for each number, how many others are smaller than it.</p>

<p><code>var smallerNumbersThanCurrent = function(nums) {<br>
  let output = [];<br>
  for (let i = 0; i &lt; nums.length; i++) {<br>
    let count = 0;<br>
    for (let j = 0; j &lt; nums.length; j++) {<br>
      if (nums[i] &gt; nums[j]) count++;<br>
    }<br>
    output[i] = count;<br>
  }<br>
  return output;<br>
};</code></p>

<p><em>Explanation</em></p>

<p>This approach compares each element with every other element.<br>
If nums[i] &gt; nums[j], we increase the counter.<br>
The result shows how many numbers are smaller than each one.</p>

<p>This works fine for small arrays but is O(n²) in complexity.</p>

<p>We can reduce it to O(n log n) using sorting and mapping —<br>
but first, let’s move to a different array problem that highlights another concept: selection.</p>

<p><em>LeetCode #448 — Find All Numbers Disappeared in an Array</em></p>

<p>Step 1 — Brute Force</p>

<p>Compare each number against all possible values 1..n.<br>
Works, but too slow → O(n²).</p>

<p>Step 2 — Smarter (Using a Set)<br>
function missingNums(nums) {<br>
  let max = nums.length;<br>
  let fullArray = Array.from({ length: max }, (_, i) =&gt; i + 1);<br>
  let set = new Set(nums);<br>
  let missing = fullArray.filter(num =&gt; !set.has(num));<br>
  return missing;<br>
}</p>

<p>How it works:</p>

<p>Array.from({ length: max }, (_, i) =&gt; i + 1) creates [1, 2, 3, ..., n].</p>

<p>new Set(nums) removes duplicates and enables O(1) lookups.</p>

<p>.filter() keeps only numbers missing from the Set.</p>

<p>Time: O(n)<br>
Space: O(n)</p>

<p>Step 3 — Can You Still Do It Better?</p>

<p>Well, yes — you can make this even more efficient,<br>
reduce the time complexity, and avoid using extra space.</p>

<p><em>Let’s see how</em></p>

<p>`function DisappearedNums(nums) {<br>
  let target = [];</p>

<p>for (let i = 0; i &lt; nums.length; i++) {<br>
    let index = Math.abs(nums[i]) - 1;<br>
    if (nums[index] &gt; 0) {<br>
      nums[index] = -nums[index];<br>
    }<br>
  }</p>

<p>for (let i = 0; i &lt; nums.length; i++) {<br>
    if (nums[i] &gt; 0) {<br>
      target.push(i + 1);<br>
    }<br>
  }</p>

<p>return target;<br>
}`</p>

<p><strong>What’s Happening Here?</strong></p>

<p>Imagine you have a classroom full of students and want to see who’s absent.<br>
Each student goes and marks their seat to show they attended.<br>
Because arrays are zero-based, each student marks the seat at value - 1.</p>

<p>At the end, we check which seats remain unmarked (still positive) —<br>
those correspond to the absent students.</p>

<p>This approach cleverly uses the same array as its own tracker,<br>
so no extra arrays or Sets are needed.</p>

<p>Time Complexity: O(n)<br>
Space Complexity: O(1)</p>

<p>This is one of the most efficient ways to solve the problem.<br>
It shows how the right data structure, combined with the right algorithm,<br>
can dramatically improve performance :)</p>

