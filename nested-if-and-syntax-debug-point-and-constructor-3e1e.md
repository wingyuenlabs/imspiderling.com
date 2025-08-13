---
Title: Nested if and syntax , Debug Point and Constructor
Description: Nested if:
syntax(for):
syntax(while):
Debug point:
What is constructor?
This keyword:
In java, this keyword is a special reference variable that referes to the current object - the object whose metho...
Author: Preethi Nandhagopal
Date: 2025-08-12T06:08:06.000Z
Robots: noindex,nofollow
Template: index
---
<p>Nested if:<br>
It means putting one if statement inside another if statement.<br>
Its used when you need to check multiple conditions and the second condition should only be checked if the first one is true.<br>
syntax(if):<br>
if (condition1) {-&gt;outer loop<br>
if (condition2){-&gt;inner loop<br>
}<br>
}</p>

<p>syntax(for):<br>
for(initialization1;condition1;update1){<br>
for(initialization2;condition2;update2){<br>
}<br>
}</p>

<p>syntax(while):<br>
while(condition1){<br>
while(condition2){<br>
}<br>
}</p>

<p>Debug point:<br>
In java (also called a breakpoint) is a marker you set in a code so that the program pause execution at that exact line when you run it in debug mode.<br>
Purpose:<br>
To impact the value of variables at a certain point in your program.<br>
To trace the flow of execution.<br>
To find and fix bugs more easily.</p>

<p>What is constructor?<br>
A special method that is automatically called when an object of a class created. Its main job is to initialize the object's data (variables) when the object is created.<br>
Keypoints:<br>
Same name as the class name.<br>
No return type.<br>
Automatically called runs as soon as you use new to create an object.<br>
Default constructor is invisible.</p>

<p>This keyword:<br>
In java, this keyword is a special reference variable that referes to the current object - the object whose method or constructor is being executed.<br>
Used when local variable (method parameters) have the same name as instance variable.<br>
syntax:<br>
this.name=name;</p>

