---
Title: Harmonizing shared state with local state in React
Description: 
Author: axtk
Date: 2025-10-24T21:49:44.000Z
Robots: noindex,nofollow
Template: index
---
<p>The split between the mental models of local and shared state management in React apps has been around since time immemorial. Here's an example of how to do these conceptually similar things in a very similar way:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight diff"><code><span class="gi">+ import { Store, useStore } from "@t8/react-store";
+
+ let counterStore = new Store(0);
</span><span class="err">
</span>  let Counter = () =&gt; {
<span class="gd">-   let [counter, setCounter] = useState(0);
</span><span class="gi">+   let [counter, setCounter] = useStore(counterStore);
</span><span class="err">
</span>    let handleClick = () =&gt; {
      setCounter(value =&gt; value + 1);
    };
<span class="err">
</span>    return &lt;button onClick={handleClick}&gt;+ {counter}&lt;/button&gt;;
  };
<span class="err">
</span>  let ResetButton = () =&gt; {
<span class="gd">-   let [, setCounter] = useState(0);
</span><span class="gi">+   let [, setCounter] = useStore(counterStore, false);
</span><span class="err">
</span>    let handleClick = () =&gt; {
      setCounter(0);
    };
<span class="err">
</span>    return &lt;button onClick={handleClick}&gt;×&lt;/button&gt;;
  };
<span class="err">
</span>  let App = () =&gt; &lt;&gt;&lt;Counter/&gt;{" "}&lt;ResetButton/&gt;&lt;/&gt;;
</code></pre>

</div>



<p>This is a full-fledged shared state setup. Similar handling of local and shared state means a short migration path from one to another without tedious refactors.</p>

<p>More details on this package can be found in <a href="https://github.com/t8js/react-store#readme" rel="noopener noreferrer">its concise overview</a>.</p>

