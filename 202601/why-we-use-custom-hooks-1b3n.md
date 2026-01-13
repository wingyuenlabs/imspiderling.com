---
Title: Why We Use Custom Hooks.
Description: 
Author: Anish Ali
Date: 2026-01-13T21:36:28.000Z
Robots: noindex,nofollow
Template: index
---
<p><a href="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fk5cgmk0zn0qmr7m25x92.png" class="article-body-image-wrapper"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fk5cgmk0zn0qmr7m25x92.png" alt="Why We Use Custom Hooks." width="720" height="281"></a></p>

<p>In this article we will answer every question in your thoughts so be ready and lets get started. Lets understand how custom hooks works and how actually hooks works in practice.</p>

<p>We see a lot of things can be done by custom hooks but we don’t know why, when &amp; where to use it?</p>

<h2>
  
  
  What is Custom Hook.
</h2>

<blockquote>
<p>Custom hooks in React are reusable JavaScript functions that allow you to extract and share stateful logic between components.</p>
</blockquote>

<p>There are alot of stuff that we have to do in our apps that react didn’t give us out of the box so they introduced the Custom Hooks concept to make our own.<br>
Without wasting anytime let’s go deep into this.</p>

<blockquote>
<p>tip: First of all not all hooks or custom-hooks return [value, setValue] type of thing this is just and array useState return with 0 and 1 index and in 0 index it return the value and in 1 index it returns the function to change 0 index value<br>
</p>


</blockquote>

<div class="highlight js-code-highlight">
<pre class="highlight plaintext"><code>const useStateOwn = (defaultValue:any) =&gt; { 
    let state = defaultValue
    const setChangeState = (changeValue:any) =&gt; { 
        state = changeValue
    } 
    return [state,setChangeState] 
}
</code></pre>

</div>



<p>This is a simplified example of how useState works internally. It’s not React’s real implementation, just for your understanding.</p>

<p><a href="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fjpyxei2rkjadid4o47ym.png" class="article-body-image-wrapper"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fjpyxei2rkjadid4o47ym.png" alt="Devider" width="720" height="180"></a></p>

<p>Now let’s understand how to create custom hook &amp; later we understand when and why we should use this hook.</p>

<p>This is the basic of the basic custom hook.<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight plaintext"><code>export function useCustomHook() {
 return 'some text return'
}
</code></pre>

</div>



<p>This is the basic custom hook. It may looks like a function but it’s not.<br>
Hooks are used inside Functional Components &amp; Custom Hooks not anywhere else. Wait what did i just say, custom hooks are used inside custom hooks ?</p>

<p><a href="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fn6kp8x453jrc0lqp52ne.png" class="article-body-image-wrapper"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fn6kp8x453jrc0lqp52ne.png" alt="Hooks usecase" width="720" height="343"></a></p>

<p>Will explain you later. First let’s just find the difference between custom hooks and functions.</p>
<h4>
  
  
  Let’s say you have 2 functions
</h4>


<div class="highlight js-code-highlight">
<pre class="highlight plaintext"><code>function foo () {
    return 10
} 
</code></pre>

</div>



<div class="highlight js-code-highlight">
<pre class="highlight plaintext"><code>function bar () {
    // we can all another function here 
    const fooValue = foo(); // just like this one
    return fooValue 
}
</code></pre>

</div>


<p>We can call 1 function into another function as you see the above, but the problem relies with hooks / custom hooks is that we cannot call custom hooks inside of the functions.</p>

<p><a href="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fjpyxei2rkjadid4o47ym.png" class="article-body-image-wrapper"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fjpyxei2rkjadid4o47ym.png" alt="Devider" width="720" height="180"></a></p>

<p>Let me just create an example of that as well for better understanding.<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight plaintext"><code>function getDataCurrent (){
    const currentData = useCustomHook() // just like this one 
    // because using this in function will give us this error

    /*
        React Hook "useCustomHook" is called in function "getDataCurrent"
        that is neither a React function component 
        nor a custom React Hook function. React component names must 
        start with an uppercase letter. 
        React Hook names must start with the 
        word "use".eslintreact-hooks/rules-of-hooks 
    */
}
// as you see the error clearly most of the things got clear 
</code></pre>

</div>



<p>There is few rules that you have to remember and that’s it.</p>

<ol>
<li>it must start with “use” string</li>
<li>it cannot be call inside an function or nested function</li>
<li>any open return statement will be use after hooks.</li>
</ol>

<p><a href="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fzsi4kgxr3uscrdd37gx0.png" class="article-body-image-wrapper"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fzsi4kgxr3uscrdd37gx0.png" alt="Rules" width="720" height="538"></a></p>

<p>Now that we know the basic of How Custom hook is created and when / where is should be created.</p>

<p><a href="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fjpyxei2rkjadid4o47ym.png" class="article-body-image-wrapper"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fjpyxei2rkjadid4o47ym.png" alt="Devider" width="720" height="180"></a></p>

<p>Let’s move on to the usecase of custom hooks.<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight plaintext"><code>function useTestCustomHook () {
    // you can call whatever inside a custom hook.
    // useCustomHook() ! a custom hook
    // test() ! a function 
    // &lt;Page/&gt; ! a Component
}
</code></pre>

</div>



<p>In this example we created custom hook with a name “useTestCustomHook” but as you see in the comments i’ve clearly mention that custom hook will call anything inside of it. Just because it’s an client side code so you can use anything of clientside inside of it.</p>

<h4>
  
  
  You can doo unlimited stuff with custom hooks let me explain.
</h4>

<ol>
<li>You can return function / functions</li>
<li>You can return multiple useStates at once to make your code clean even throught it is used in only single file.</li>
<li>You can split your logic and UI from  to make the code even more organized.</li>
<li>You can pass arguments into customHook and use them accordingly just like a function but with hooks power.</li>
</ol>

<p>We will go into the logic section later where we can show the before &amp; after effect of using custom hook so stay tuned.</p>

<p><a href="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fd1du2bhlnpunstzmb7qp.png" class="article-body-image-wrapper"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fd1du2bhlnpunstzmb7qp.png" alt="Details" width="720" height="343"></a></p>

<p>Hope you understand this soo far and I’m trying my best to make this topic as simple as possible.</p>

<h4>
  
  
  Let’s move to the practical example.
</h4>

<p>Let’s say we have 2 components that needs user data and that user data will fetch from api and save on state,<br>
If we normally do that we have to call that api in both components and save there data on seperate states and when something changes we have to change those 2 components as well and vise versa.</p>

<p>Okey now suppose we have UserProfile &amp; UserSettings components.<br>
First I’ll create those 2 components without custom hook and see what it looks like.</p>

<p>Now we have UserProfile component that looks like this.<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight plaintext"><code>import { useEffect, useState } from 'react'
import axios from 'axios'

function UserProfile() {
  const [data, setData] = useState(null);
  const [isLoading, setIsLoading] = useState(true);
  const [error, setError] = useState(null);

  useEffect(() =&gt; {
    async function fetchData() {
      setIsLoading(true);
      try {
        const response = await axios.get(`/api/user`);
        setData(response.data);
      } catch (err) {
        setError(err);
      } finally {
        setIsLoading(false);
      }
    }
    fetchData();
  }, []);

  if (isLoading) return &lt;p&gt;Loading user data...&lt;/p&gt;;
  if (error) return &lt;p style={{ color: 'red' }}&gt;Error loading user: {error.message}&lt;/p&gt;;

  return (
    &lt;div&gt;
      &lt;h3&gt;{data.name}&lt;/h3&gt;
      &lt;p&gt;Email: {data.email}&lt;/p&gt;
      &lt;p&gt;Phone: {data.email}&lt;/p&gt;
    &lt;/div&gt;
  );
}
</code></pre>

</div>



<p>And we have UserSettings Component which looks like this.<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight plaintext"><code>import { useEffect, useState } from 'react'
import axios from 'axios'

function UserSettings() {
  const [data, setData] = useState(null);
  const [isLoading, setIsLoading] = useState(true);
  const [error, setError] = useState(null);

  useEffect(() =&gt; {
    async function fetchData() {
      setIsLoading(true);
      try {
        const response = await axios.get(`/api/user`);
        setData(response.data);
      } catch (err) {
        setError(err);
      } finally {
        setIsLoading(false);
      }
    }
    fetchData();
  }, []);

  if (isLoading) return &lt;p&gt;Loading user data...&lt;/p&gt;;
  if (error) return &lt;p style={{ color: 'red' }}&gt;Error loading user: {error.message}&lt;/p&gt;;

  return (
    &lt;div&gt;
      &lt;h3&gt;Settings Page&lt;/h3&gt;
      &lt;p&gt;Email: {data.email}&lt;/p&gt;
    &lt;/div&gt;
  );
}
</code></pre>

</div>



<p>As you see the codebase above 80% of the codebase is same only the return statement have some changes but if we don’t know custom hooks we usually do stuff like this and this is i call garbage code.</p>

<p>Now let’s create a custom and see how this makes our life easier.<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight plaintext"><code>const useUserData =()=&gt;{
  const [data, setData] = useState(null);
  const [isLoading, setIsLoading] = useState(true);
  const [error, setError] = useState(null);

  useEffect(() =&gt; {
    async function fetchData() {
      setIsLoading(true);
      setError(null);
      try {
        const response = await axios.get(`/api/user`);
        setData(response.data);
      } catch (err) {
        setError(err);
      } finally {
        setIsLoading(false);
      }
    }
    fetchData();
  }, []); 

  return {
    data,
    isLoading,
    error
  }
}
</code></pre>

</div>



<p>Now that we have custom hook we can update the logic in both UserProfile and UserSettings and see how this optimized the workflow.</p>

<p>UserProfile.jsx<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight plaintext"><code>function UserProfile() {
  const { data, isLoading, error} = useUserData()

  if (isLoading) return &lt;p&gt;Loading user profile...&lt;/p&gt;;
  if (error) return &lt;p style={{ color: 'red' }}&gt;Error loading user: {error.message}&lt;/p&gt;;

  return (
    &lt;div&gt;
      &lt;h3&gt;{data.name}&lt;/h3&gt;
      &lt;p&gt;Email: {data.email}&lt;/p&gt;
      &lt;p&gt;Phone: {data.email}&lt;/p&gt;
    &lt;/div&gt;
  );
}    
</code></pre>

</div>



<p>UserSettings.jsx<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight plaintext"><code>function UserSettings() {
  const { data, isLoading, error} = useUserData()

  if (isLoading) return &lt;p&gt;Loading user profile...&lt;/p&gt;;
  if (error) return &lt;p style={{ color: 'red' }}&gt;Error loading user: {error.message}&lt;/p&gt;;

  return (
    &lt;div&gt;
      &lt;h3&gt;Settings&lt;/h3&gt;
      &lt;p&gt;Email: {data.email}&lt;/p&gt;
    &lt;/div&gt;
  );
}
</code></pre>

</div>



<p>Now that if we need to change anything in API we just have to update the custom hooks and all the components which are dependent on that hook will automatically reflect that change, So you don’t need to update each and every component from now on.</p>

<p><a href="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fd1du2bhlnpunstzmb7qp.png" class="article-body-image-wrapper"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fd1du2bhlnpunstzmb7qp.png" alt="Details" width="720" height="343"></a></p>

<blockquote>
<p>tip: You can return jsx / tsx statement from hooks and it will be used in different component. Let’s say you have a {data.name} component and you think that  component will later need some modifications so you can just return the  component and then later use that in Components like that {badge} and send that using return statement .. return { badge:… }.. like this.</p>
</blockquote>

<p>And there you have it.<br>
Hope this article made your life a little easy/better.</p>

<p>If you want my assistence please reach me out on <a href="https://github.com/ani-ali" rel="noopener noreferrer">Github </a>&amp; <a href="https://www.linkedin.com/in/anish-ali/" rel="noopener noreferrer">Linkedin</a>.</p>

