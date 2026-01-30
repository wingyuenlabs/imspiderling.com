---
Title: How a simple pointer exercise had me pulling my hair out.
Description: 
Author: Jemma
Date: 2026-01-30T21:31:18.000Z
Robots: noindex,nofollow
Template: index
---
<p>I was tasked to write out (by hand) the outputs for a given pointer exercise. It seemed simple enough, but when I got to a certain stage, I found myself lost. And the harder I thought, the deeper I went down a rabbit hole of confusion. Asking for AI did not help. <br>
I will give the example that stumped me, some of the mistakes I made, why they were wrong and the principles that I learned from this experience.<br>
Spoiler, in the end I went back to the introduction to pointers in my course material, and thinking in this simplistic way decluttered my brain and the answer became clear.</p>

<p>Here is the code;</p>

<p>`<br>
int f(int** r, int** s) {<br>
    int temp = <strong>r;<br>
    int temp2 = **s;<br>
    int *z = *r;<br>
    *r = *s;<br>
    *s = z;<br>
    printf("</strong>r = %d\n", <strong>r);<br>
    printf("</strong>s = %d\n", **s);<br>
    *z += 3;<br>
    **s -= 8;<br>
    **r -= 19;<br>
    return temp + temp2;<br>
}</p>

<p>int main(void) {<br>
    int a = 80;<br>
    int b = 12;<br>
    int *p = &amp;a;<br>
    int *q = &amp;b;<br>
    int x = f(&amp;p, &amp;q);<br>
    printf("x = %d\n", x);<br>
    printf("*p = %d\n", *p);<br>
    printf("*q = %d\n", *q);<br>
    printf("a = %d\n", a);<br>
    printf("b = %d\n", b);<br>
    return EXIT_SUCCESS;<br>
}<br>
`</p>

<p>This code will output the following.<br>
**r = 12<br>
**s = 80<br>
x = 92<br>
*p = -7<br>
*q = 75<br>
a = 75<br>
b = -7</p>

<p>These are the mistakes I made;<br>
I returned an int value of 64 (not 92) back to main from function f. In lines 5 and 6, the pointers are dereferenced to the int values of 80 and 12 respectively. But I mistakenly believed that the arithmetic on **r and **s affected the final value of temp and temp2 not realizing that those initializations in lines 5 and 6 are locked into the memory addresses of temp and temp2 and to manipulate those values I would need to explicitly perform arithmetic on temp and temp2. e.g. temp += 3.</p>

<p>Again, I changed the wrong addresses. Instead of changing what *r and *s points to within the function f stack, I changed what *p and *q point to in the main stack. </p>

<p>Finally, I forgot that *z was a pointer to an int. So, I was thinking that I had to perform arithmetic on the address at *p. So, I believed that *z += 3 meant *z = &amp;p + 3. My thinking was that if I was supposed to perform arithmetic on int a, then the expression at line 7 would be *z = **r. This one got me so confused and going around in circles.</p>

<p>My take-aways; <br>
Go back to basics.<br>
Write out every step, even if it seems straight forward or insignificant.<br>
Remember what the type is<br>
Visualize the memory addresses </p>

