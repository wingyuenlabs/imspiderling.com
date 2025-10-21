---
Title: Implementing Dark Mode in a React App
Description: 
Author: Samantha Laine
Date: 2025-10-21T22:12:26.000Z
Robots: noindex,nofollow
Template: index
---
<p><strong>Author:</strong> Samantha Laine King<br><br>
<strong>Last Updated:</strong> October 2025  </p>


<h2>
  
  
  🪄 Overview
</h2>

<p>This guide walks through adding a clean, accessible <strong>Dark Mode toggle</strong> to a React application using <strong>Tailwind CSS</strong> and <strong>localStorage</strong>.<br><br>
It covers setup, theme persistence, and best practices for accessibility.</p>


<h2>
  
  
  ⚙️ Step 1: Set Up Tailwind for Theming
</h2>

<p>Tailwind provides built-in dark mode support.<br><br>
In your <code>tailwind.config.js</code> file, enable <code>class</code> mode so users can manually toggle it:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight javascript"><code><span class="c1">// tailwind.config.js</span>
<span class="k">export</span> <span class="k">default</span> <span class="p">{</span>
  <span class="na">darkMode</span><span class="p">:</span> <span class="dl">'</span><span class="s1">class</span><span class="dl">'</span><span class="p">,</span>
  <span class="na">theme</span><span class="p">:</span> <span class="p">{</span>
    <span class="na">extend</span><span class="p">:</span> <span class="p">{},</span>
  <span class="p">},</span>
  <span class="na">plugins</span><span class="p">:</span> <span class="p">[],</span>
<span class="p">}</span>
</code></pre>

</div>



<p>This allows you to switch themes by adding or removing the dark class on the root element.</p>

<h2>
  
  
  🧩 Step 2: Create a Custom Hook for Theme Management
</h2>

<p>We’ll use a custom React hook to manage theme state and persist it in localStorage.<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight plaintext"><code>// useDarkMode.ts
import { useEffect, useState } from 'react'

export default function useDarkMode() {
  const [theme, setTheme] = useState(localStorage.theme || 'light')

  useEffect(() =&gt; {
    const root = window.document.documentElement
    const isDark = theme === 'dark'

    root.classList.toggle('dark', isDark)
    localStorage.setItem('theme', theme)
  }, [theme])

  return { theme, setTheme }
}
</code></pre>

</div>



<h2>
  
  
  💡 Step 3: Add the Toggle Button
</h2>

<p>Here’s how to create a reusable toggle button that switches between light and dark modes.<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight plaintext"><code>// ThemeToggle.tsx
import { Moon, Sun } from 'lucide-react'
import useDarkMode from './useDarkMode'

export default function ThemeToggle() {
  const { theme, setTheme } = useDarkMode()

  return (
    &lt;button
      onClick={() =&gt; setTheme(theme === 'dark' ? 'light' : 'dark')}
      className="p-2 rounded-full border text-gray-700 dark:text-gray-100 hover:bg-gray-100 dark:hover:bg-gray-700 transition"
      aria-label="Toggle dark mode"
    &gt;
      {theme === 'dark' ? &lt;Sun size={18} /&gt; : &lt;Moon size={18} /&gt;}
    &lt;/button&gt;
  )
}
</code></pre>

</div>



<h2>
  
  
  🎨 Step 4: Test Your Dark Mode
</h2>

<p>Wrap your app in a container that uses Tailwind’s dark variant:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight plaintext"><code>// App.tsx
import ThemeToggle from './ThemeToggle'

export default function App() {
  return (
    &lt;div className="min-h-screen bg-white text-gray-900 dark:bg-gray-900 dark:text-gray-100 transition-colors"&gt;
      &lt;header className="flex justify-between p-4"&gt;
        &lt;h1 className="text-xl font-semibold"&gt;Dark Mode Example&lt;/h1&gt;
        &lt;ThemeToggle /&gt;
      &lt;/header&gt;
      &lt;main className="p-4"&gt;
        &lt;p&gt;Try toggling dark mode and refresh the page to see it persist!&lt;/p&gt;
      &lt;/main&gt;
    &lt;/div&gt;
  )
}
</code></pre>

</div>



<h2>
  
  
  ✅ Step 5: Best Practices &amp; Accessibility Tips
</h2>

<p>Step 5: Best Practices &amp; Accessibility Tips</p>

<ul>
<li><p>Provide sufficient color contrast (use tools like contrast-ratio.com).</p></li>
<li><p>Add an aria-label to the toggle button for screen readers.</p></li>
<li><p>Ensure motion preferences are respected when using transitions.</p></li>
<li><p>Store the user’s preference in localStorage for a consistent experience.</p></li>
</ul>

<h2>
  
  
  ✨ Summary
</h2>

<p>This implementation gives users control over their theme while maintaining accessibility and performance.<br>
The setup is lightweight, scalable, and easy to extend for future design systems.</p>

<p>Thanks for reading!</p>

<p>Written with care by Samantha Laine King — Frontend Developer passionate about clean UI, accessibility, and developer education.</p>

