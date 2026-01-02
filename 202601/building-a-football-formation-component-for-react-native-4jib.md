---
Title: Building a Football Formation Component for React Native ⚽
Description: 
Author: Arbab Rafiq
Date: 2026-01-01T21:32:45.000Z
Robots: noindex,nofollow
Template: index
---
<p>As a football enthusiast and React Native developer, I've always been fascinated by the tactical side of the beautiful game. Whether it's analyzing how Guardiola's 4-3-3 differs from Klopp's setup or understanding why a 3-5-2 might counter a 4-2-3-1, formations tell a story.</p>

<p>But when I needed to display team lineups in a mobile app, I hit a wall. There was no ready-made React Native component that could elegantly render football formations with player statistics, custom styling, and proper TypeScript support.</p>

<p>So I built one. And today, I'm sharing the journey of creating <strong>react-native-football-formation</strong> — a package that's now available on npm for the entire React Native community.</p>




<h2>
  
  
  🎯 The Problem: Visualizing Tactical Complexity
</h2>

<p>If you've ever tried to build a football app, you know the challenge. A team formation isn't just a static image, it's a dynamic visualization that needs to:</p>

<ul>
<li>✅ <strong>Position 11 players</strong> accurately based on tactical formation (4-3-3, 4-2-3-1, 5-3-2, etc.)</li>
<li>✅ <strong>Display player statistics</strong> (goals, assists, cards, substitutions)</li>
<li>✅ <strong>Handle multiple formations</strong> (there are dozens of tactical setups)</li>
<li>✅ <strong>Support customization</strong> (team colors, fonts, icons)</li>
<li>✅ <strong>Work on different screen sizes</strong> and orientations</li>
<li>✅ <strong>Provide excellent TypeScript support</strong> for developer experience</li>
</ul>

<p>The existing solutions were either too rigid, poorly maintained, or simply didn't exist for React Native. Most developers were either embedding static images or building one-off custom solutions for each project.</p>

<p>I wanted something better, a flexible, production-ready component that any developer could drop into their app.</p>

<p><a href="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fxjpwi6f3g5grn1xg7676.png" class="article-body-image-wrapper"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fxjpwi6f3g5grn1xg7676.png" alt="demo" width="800" height="820"></a></p>




<h2>
  
  
  💡 The Solution: A Highly Customizable Formation Component
</h2>

<p>After weeks of development, testing, and iteration, here's what I built:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight tsx"><code><span class="k">import</span> <span class="p">{</span> <span class="nx">FormationField</span> <span class="p">}</span> <span class="k">from</span> <span class="dl">'</span><span class="s1">react-native-football-formation</span><span class="dl">'</span><span class="p">;</span>

<span class="kd">function</span> <span class="nf">MatchLineup</span><span class="p">()</span> <span class="p">{</span>
  <span class="kd">const</span> <span class="nx">lineup</span> <span class="o">=</span> <span class="p">{</span>
    <span class="na">players</span><span class="p">:</span> <span class="p">[</span>
      <span class="p">{</span>
        <span class="na">playerId</span><span class="p">:</span> <span class="dl">'</span><span class="s1">1</span><span class="dl">'</span><span class="p">,</span>
        <span class="na">matchName</span><span class="p">:</span> <span class="dl">'</span><span class="s1">De Gea</span><span class="dl">'</span><span class="p">,</span>
        <span class="na">shirtNumber</span><span class="p">:</span> <span class="mi">1</span><span class="p">,</span>
        <span class="na">rating</span><span class="p">:</span> <span class="dl">'</span><span class="s1">7.5</span><span class="dl">'</span><span class="p">,</span>
        <span class="na">position</span><span class="p">:</span> <span class="dl">'</span><span class="s1">Goalkeeper</span><span class="dl">'</span><span class="p">,</span>
        <span class="na">formationPlace</span><span class="p">:</span> <span class="dl">'</span><span class="s1">1</span><span class="dl">'</span><span class="p">,</span>
        <span class="na">stats</span><span class="p">:</span> <span class="p">[</span>
          <span class="p">{</span> <span class="na">type</span><span class="p">:</span> <span class="dl">'</span><span class="s1">goals</span><span class="dl">'</span><span class="p">,</span> <span class="na">value</span><span class="p">:</span> <span class="mi">1</span> <span class="p">},</span>
          <span class="p">{</span> <span class="na">type</span><span class="p">:</span> <span class="dl">'</span><span class="s1">yellowCard</span><span class="dl">'</span><span class="p">,</span> <span class="na">value</span><span class="p">:</span> <span class="dl">'</span><span class="s1">1</span><span class="dl">'</span> <span class="p">},</span>
        <span class="p">],</span>
      <span class="p">},</span>
      <span class="c1">// ... 10 more players</span>
    <span class="p">],</span>
    <span class="na">formationUsed</span><span class="p">:</span> <span class="dl">'</span><span class="s1">433</span><span class="dl">'</span><span class="p">,</span>
  <span class="p">};</span>

  <span class="k">return </span><span class="p">(</span>
    <span class="p">&lt;</span><span class="nc">FormationField</span>
      <span class="na">lineup</span><span class="p">=</span><span class="si">{</span><span class="nx">lineup</span><span class="si">}</span>
      <span class="na">getPlayerPhotoUrl</span><span class="p">=</span><span class="si">{</span><span class="p">(</span><span class="nx">playerId</span><span class="p">)</span> <span class="o">=&gt;</span>
        <span class="s2">`https://api.example.com/players/</span><span class="p">${</span><span class="nx">playerId</span><span class="p">}</span><span class="s2">.png`</span>
      <span class="si">}</span>
      <span class="na">onPlayerPress</span><span class="p">=</span><span class="si">{</span><span class="p">(</span><span class="nx">player</span><span class="p">)</span> <span class="o">=&gt;</span> <span class="p">{</span>
        <span class="c1">// Navigate to player details</span>
        <span class="nx">console</span><span class="p">.</span><span class="nf">log</span><span class="p">(</span><span class="dl">'</span><span class="s1">Tapped:</span><span class="dl">'</span><span class="p">,</span> <span class="nx">player</span><span class="p">.</span><span class="nx">matchName</span><span class="p">);</span>
      <span class="p">}</span><span class="si">}</span>
    <span class="p">/&gt;</span>
  <span class="p">);</span>
<span class="p">}</span>
</code></pre>

</div>


<p><strong>Simple API. Powerful features.</strong> That's what I aimed for.</p>


<h2>
  
  
  🚀 Key Features That Make It Stand Out
</h2>
<h3>
  
  
  1. Support for 24 Tactical Formations
</h3>

<p>From the classic 4-4-2 to modern variations like the 3-2-4-1 or false 9 (4-2-4-0), the component handles them all. Each formation has carefully calculated coordinate mappings to position players realistically on the field.</p>

<p>The coordinate system I developed allows for precise positioning:<br>
</p>
<div class="highlight js-code-highlight">
<pre class="highlight typescript"><code><span class="kd">const</span> <span class="nx">playerX</span> <span class="o">=</span> <span class="p">(</span><span class="nx">player</span><span class="p">.</span><span class="nx">x</span> <span class="o">/</span> <span class="mi">100</span><span class="p">)</span> <span class="o">*</span> <span class="nx">fieldWidth</span> <span class="o">-</span> <span class="nx">cardWidth</span> <span class="o">/</span> <span class="mi">2</span><span class="p">;</span>
<span class="kd">const</span> <span class="nx">playerY</span> <span class="o">=</span> <span class="p">(</span><span class="nx">player</span><span class="p">.</span><span class="nx">y</span> <span class="o">/</span> <span class="mi">100</span><span class="p">)</span> <span class="o">*</span> <span class="nx">fieldHeight</span> <span class="o">-</span> <span class="nx">cardHeight</span> <span class="o">/</span> <span class="mi">2</span><span class="p">;</span>
</code></pre>

</div>


<p>This percentage-based approach ensures formations scale perfectly across different device sizes.</p>
<h3>
  
  
  2. Comprehensive Player Statistics
</h3>

<p>The component automatically renders player stats with intuitive visual indicators:</p>

<ul>
<li>⚽ Goals scored (with count for multiple goals)</li>
<li>👟 Assists provided</li>
<li>🟨 Yellow cards</li>
<li>🟥 Red cards</li>
<li>🔄 Substitutions</li>
<li>⚽🔴 Own goals</li>
</ul>

<p>All icons are customizable, and stats gracefully handle null values and zero counts.</p>
<h3>
  
  
  3. Complete Theme Customization
</h3>

<p>Every visual aspect is customizable through a comprehensive theme system:<br>
</p>
<div class="highlight js-code-highlight">
<pre class="highlight tsx"><code><span class="p">&lt;</span><span class="nc">FormationField</span>
  <span class="na">lineup</span><span class="p">=</span><span class="si">{</span><span class="nx">lineup</span><span class="si">}</span>
  <span class="na">theme</span><span class="p">=</span><span class="si">{</span><span class="p">{</span>
    <span class="na">colors</span><span class="p">:</span> <span class="p">{</span>
      <span class="na">primary</span><span class="p">:</span> <span class="dl">'</span><span class="s1">#FF0000</span><span class="dl">'</span><span class="p">,</span>
      <span class="na">playerName</span><span class="p">:</span> <span class="dl">'</span><span class="s1">#FFFFFF</span><span class="dl">'</span><span class="p">,</span>
      <span class="na">success</span><span class="p">:</span> <span class="dl">'</span><span class="s1">#00FF00</span><span class="dl">'</span><span class="p">,</span>
      <span class="na">warning</span><span class="p">:</span> <span class="dl">'</span><span class="s1">#FFAA00</span><span class="dl">'</span><span class="p">,</span>
    <span class="p">},</span>
    <span class="na">typography</span><span class="p">:</span> <span class="p">{</span>
      <span class="na">fontFamily</span><span class="p">:</span> <span class="dl">'</span><span class="s1">MyCustomFont-Regular</span><span class="dl">'</span><span class="p">,</span>
      <span class="na">fontFamilyBold</span><span class="p">:</span> <span class="dl">'</span><span class="s1">MyCustomFont-Bold</span><span class="dl">'</span><span class="p">,</span>
      <span class="na">playerNameSize</span><span class="p">:</span> <span class="mi">14</span><span class="p">,</span>
    <span class="p">},</span>
    <span class="na">spacing</span><span class="p">:</span> <span class="p">{</span>
      <span class="na">playerCardWidth</span><span class="p">:</span> <span class="mi">80</span><span class="p">,</span>
      <span class="na">playerImageSize</span><span class="p">:</span> <span class="mi">50</span><span class="p">,</span>
    <span class="p">},</span>
  <span class="p">}</span><span class="si">}</span>
<span class="p">/&gt;</span>
</code></pre>

</div>

<h3>
  
  
  4. RTL Language Support 🌍
</h3>

<p>Being conscious of the global football community, I built in full RTL (right-to-left) support using React Native's <code>I18nManager</code>. The component automatically adapts icon positioning and text direction for Arabic, Hebrew, and other RTL languages.</p>
<h3>
  
  
  5. Custom Rendering Hooks
</h3>

<p>For advanced use cases, developers can override the default player cards or footer:<br>
</p>
<div class="highlight js-code-highlight">
<pre class="highlight tsx"><code><span class="p">&lt;</span><span class="nc">FormationField</span>
  <span class="na">lineup</span><span class="p">=</span><span class="si">{</span><span class="nx">lineup</span><span class="si">}</span>
  <span class="na">renderPlayerCard</span><span class="p">=</span><span class="si">{</span><span class="p">(</span><span class="nx">player</span><span class="p">,</span> <span class="nx">width</span><span class="p">,</span> <span class="nx">height</span><span class="p">)</span> <span class="o">=&gt;</span> <span class="p">(</span>
    <span class="p">&lt;</span><span class="nc">CustomPlayerCard</span> <span class="na">player</span><span class="p">=</span><span class="si">{</span><span class="nx">player</span><span class="si">}</span> <span class="p">/&gt;</span>
  <span class="p">)</span><span class="si">}</span>
  <span class="na">renderFooter</span><span class="p">=</span><span class="si">{</span><span class="p">(</span><span class="nx">formation</span><span class="p">)</span> <span class="o">=&gt;</span> <span class="p">(</span>
    <span class="p">&lt;</span><span class="nc">CustomFooter</span> <span class="na">formation</span><span class="p">=</span><span class="si">{</span><span class="nx">formation</span><span class="si">}</span> <span class="p">/&gt;</span>
  <span class="p">)</span><span class="si">}</span>
<span class="p">/&gt;</span>
</code></pre>

</div>


<p>This makes the component incredibly flexible while maintaining ease of use for simple cases.</p>


<h2>
  
  
  🔧 Technical Decisions &amp; Challenges
</h2>
<h3>
  
  
  TypeScript First
</h3>

<p>I wrote the entire package in TypeScript. This was non-negotiable. The type safety not only caught countless bugs during development but also provides an excellent developer experience with full IntelliSense support.<br>
</p>
<div class="highlight js-code-highlight">
<pre class="highlight typescript"><code><span class="kr">interface</span> <span class="nx">Player</span> <span class="p">{</span>
  <span class="nl">rating</span><span class="p">:</span> <span class="kr">string</span><span class="p">;</span>
  <span class="nl">playerId</span><span class="p">:</span> <span class="kr">string</span><span class="p">;</span>
  <span class="nl">position</span><span class="p">:</span> <span class="kr">string</span><span class="p">;</span>
  <span class="nl">matchName</span><span class="p">:</span> <span class="kr">string</span><span class="p">;</span>
  <span class="nl">shirtNumber</span><span class="p">:</span> <span class="kr">number</span><span class="p">;</span>
  <span class="nl">formationPlace</span><span class="p">?:</span> <span class="kr">string</span><span class="p">;</span>
  <span class="nl">stats</span><span class="p">:</span> <span class="p">(</span><span class="nx">PlayerStats</span> <span class="o">|</span> <span class="kc">null</span><span class="p">)[];</span>
<span class="p">}</span>

<span class="kr">interface</span> <span class="nx">PlayerStats</span> <span class="p">{</span>
  <span class="nl">type</span><span class="p">:</span> <span class="dl">'</span><span class="s1">goals</span><span class="dl">'</span> <span class="o">|</span> <span class="dl">'</span><span class="s1">yellowCard</span><span class="dl">'</span> <span class="o">|</span> <span class="dl">'</span><span class="s1">redCard</span><span class="dl">'</span> <span class="o">|</span> <span class="dl">'</span><span class="s1">goalAssist</span><span class="dl">'</span> <span class="o">|</span> <span class="dl">'</span><span class="s1">totalSubOff</span><span class="dl">'</span> <span class="o">|</span> <span class="dl">'</span><span class="s1">ownGoals</span><span class="dl">'</span><span class="p">;</span>
  <span class="nl">value</span><span class="p">:</span> <span class="kr">string</span> <span class="o">|</span> <span class="kr">number</span><span class="p">;</span>
<span class="p">}</span>
</code></pre>

</div>

<h3>
  
  
  Coordinate Calculation Mathematics 📐
</h3>

<p>Calculating accurate player positions for 24 different formations was mathematically intensive. Each formation needed manual coordinate mapping based on realistic tactical positioning:<br>
</p>
<div class="highlight js-code-highlight">
<pre class="highlight typescript"><code><span class="k">export</span> <span class="kd">const</span> <span class="nx">FORMATION_COORDINATES_BY_PLACE</span> <span class="o">=</span> <span class="p">{</span>
  <span class="dl">'</span><span class="s1">4-3-3</span><span class="dl">'</span><span class="p">:</span> <span class="p">{</span>
    <span class="dl">'</span><span class="s1">1</span><span class="dl">'</span><span class="p">:</span> <span class="p">{</span> <span class="na">x</span><span class="p">:</span> <span class="mi">50</span><span class="p">,</span> <span class="na">y</span><span class="p">:</span> <span class="mi">90</span> <span class="p">},</span>  <span class="c1">// Goalkeeper</span>
    <span class="dl">'</span><span class="s1">2</span><span class="dl">'</span><span class="p">:</span> <span class="p">{</span> <span class="na">x</span><span class="p">:</span> <span class="mi">20</span><span class="p">,</span> <span class="na">y</span><span class="p">:</span> <span class="mi">70</span> <span class="p">},</span>  <span class="c1">// Right Back</span>
    <span class="dl">'</span><span class="s1">3</span><span class="dl">'</span><span class="p">:</span> <span class="p">{</span> <span class="na">x</span><span class="p">:</span> <span class="mi">40</span><span class="p">,</span> <span class="na">y</span><span class="p">:</span> <span class="mi">75</span> <span class="p">},</span>  <span class="c1">// Center Back</span>
    <span class="c1">// ... positions 4-11</span>
  <span class="p">},</span>
  <span class="dl">'</span><span class="s1">4-2-3-1</span><span class="dl">'</span><span class="p">:</span> <span class="p">{</span>
    <span class="c1">// Different coordinate mapping</span>
  <span class="p">},</span>
  <span class="c1">// ... 22 more formations</span>
<span class="p">};</span>
</code></pre>

</div>


<p>I spent hours watching match footage and analyzing tactical diagrams to get these positions right.</p>
<h3>
  
  
  Performance Optimization ⚡
</h3>

<p>With 11 player cards, each containing multiple conditional renderings (stats icons, cards, etc.), performance was critical. I implemented:</p>

<ul>
<li>
<strong>React.memo</strong> for PlayerCard component to prevent unnecessary re-renders</li>
<li>
<strong>Conditional rendering</strong> to only show stat icons when they exist</li>
<li>
<strong>Optimized image loading</strong> with proper resize modes</li>
<li>
<strong>Efficient coordinate calculations</strong> cached at the component level</li>
</ul>


<h2>
  
  
  💼 Real-World Use Cases
</h2>

<p>The package is versatile enough for various applications:</p>

<ul>
<li>
<strong>Fantasy Football Apps</strong>: Display user lineups with live player stats</li>
<li>
<strong>Sports News Apps</strong>: Visualize team formations in match previews</li>
<li>
<strong>Tactical Analysis Tools</strong>: Allow coaches to create and share formations</li>
<li>
<strong>Match Statistics Apps</strong>: Show starting elevens with performance metrics</li>
<li>
<strong>Football Games</strong>: Display team selection screens</li>
</ul>


<h2>
  
  
  🎮 Try It Yourself
</h2>

<p>The package is available now on npm:<br>
</p>
<div class="highlight js-code-highlight">
<pre class="highlight shell"><code>npm <span class="nb">install </span>react-native-football-formation
</code></pre>

</div>


<p>Or if you prefer yarn:<br>
</p>
<div class="highlight js-code-highlight">
<pre class="highlight shell"><code>yarn add react-native-football-formation
</code></pre>

</div>



<div class="ltag-github-readme-tag">
  <div class="readme-overview">
    <h2>
      <img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fassets.dev.to%2Fassets%2Fgithub-logo-5a155e1f9a670af7944dd5e12375bc76ed542ea80224905ecaf878b9157cdefc.svg" alt="GitHub logo">
      <a href="https://github.com/arbab-io" rel="noopener noreferrer">
        arbab-io
      </a> / <a href="https://github.com/arbab-io/react-native-football-formation" rel="noopener noreferrer">
        react-native-football-formation
      </a>
    </h2>
    <h3>
      
    </h3>
  </div>
  <div class="ltag-github-body">
    
<div id="readme" class="md">
<div class="markdown-heading">
<h1 class="heading-element">React Native Football Formation</h1>
</div>

<p>A highly customizable React Native component for displaying football/soccer team formations with player positions, stats, and match information.</p>

<p><a rel="noopener noreferrer nofollow" href="https://camo.githubusercontent.com/9938df3ef70a0a9c56c9ec51146c96486cacb628440af08c0bab6066f5f2420b/68747470733a2f2f696d672e736869656c64732e696f2f62616467652f52656163742532304e61746976652d253345253344302e37342e302d626c7565"><img src="https://camo.githubusercontent.com/9938df3ef70a0a9c56c9ec51146c96486cacb628440af08c0bab6066f5f2420b/68747470733a2f2f696d672e736869656c64732e696f2f62616467652f52656163742532304e61746976652d253345253344302e37342e302d626c7565" alt="React Native"></a>
<a rel="noopener noreferrer nofollow" href="https://camo.githubusercontent.com/fb6bab0fd0f66cf073aff507746d1b51fe079b74e7024a9bc695d0345c46df97/68747470733a2f2f696d672e736869656c64732e696f2f62616467652f547970655363726970742d352e302d626c7565"><img src="https://camo.githubusercontent.com/fb6bab0fd0f66cf073aff507746d1b51fe079b74e7024a9bc695d0345c46df97/68747470733a2f2f696d672e736869656c64732e696f2f62616467652f547970655363726970742d352e302d626c7565" alt="TypeScript"></a>
<a rel="noopener noreferrer nofollow" href="https://camo.githubusercontent.com/f8df3091bbe1149f398a5369b2c39e896766f9f6efba3477c63e9b4aa940ef14/68747470733a2f2f696d672e736869656c64732e696f2f62616467652f6c6963656e73652d4d49542d677265656e"><img src="https://camo.githubusercontent.com/f8df3091bbe1149f398a5369b2c39e896766f9f6efba3477c63e9b4aa940ef14/68747470733a2f2f696d672e736869656c64732e696f2f62616467652f6c6963656e73652d4d49542d677265656e" alt="License"></a></p>
<div class="markdown-heading">
<h2 class="heading-element">Status</h2>
</div>
<blockquote>
<p><strong>Note:</strong> This package is currently in active development and is provided for testing and evaluation purposes only. The API is subject to change and stability is not guaranteed. It is not recommended for use in production environments at this time.</p>
</blockquote>
<div class="markdown-heading">
<h2 class="heading-element">Demo</h2>
</div>
<p>
  <a rel="noopener noreferrer" href="https://github.com/arbab-io/react-native-football-formation/formation-demo.png"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fgithub.com%2Farbab-io%2Freact-native-football-formation%2Fformation-demo.png" alt="Football Formation Demo" width="400"></a>
</p>

<div class="markdown-heading">
<h2 class="heading-element">Features</h2>

</div>

<p>✅ <strong>24 Supported Formations</strong> - All major tactical formations (4-3-3, 4-2-3-1, 3-5-2, etc.)
✅ <strong>Player Statistics</strong> - Goals, assists, cards, substitutions, own goals
✅ <strong>Highly Customizable</strong> - Theme system for colors, fonts, spacing
✅ <strong>RTL Support</strong> - Built-in support for right-to-left languages
✅ <strong>TypeScript</strong> - Full type safety and IntelliSense
✅ <strong>Expo &amp; Bare RN</strong> - Compatible with both Expo and bare React Native projects
✅ <strong>Asset Override</strong> - Use your own field backgrounds and icons
✅ <strong>Component Override</strong> - Custom rendering for player cards…</p>
</div>
  </div>
  <div class="gh-btn-container"><a class="gh-btn" href="https://github.com/arbab-io/react-native-football-formation" rel="noopener noreferrer">View on GitHub</a></div>
</div>





<h2>
  
  
  💭 Final Thoughts
</h2>

<p>Building this package taught me that great developer tools require:</p>

<ul>
<li>
<strong>Clear problem definition</strong> (what specific pain are you solving?)</li>
<li>
<strong>Simple API design</strong> (make the easy things easy, hard things possible)</li>
<li>
<strong>Comprehensive documentation</strong> (code is only half the product)</li>
<li>
<strong>Community engagement</strong> (listen, iterate, improve)</li>
<li>
<strong>Production quality</strong> (TypeScript, error handling, edge cases)</li>
</ul>

<p>Whether you're building a football app or any other specialized component, I hope this journey inspires you to contribute to the open-source ecosystem. The React Native community thrives on developers solving real problems and sharing solutions.</p>




<h2>
  
  
  👋 Let's Connect!
</h2>

<p>If you found this useful, give the repository a ⭐ on GitHub! I'd love to hear:</p>

<ul>
<li>What features would you like to see?</li>
<li>Are you building a football/sports app?</li>
<li>What challenges have you faced with React Native components?</li>
</ul>

<p>Drop a comment below and let's chat! 💬</p>

<p>⚽ Happy coding, and may your formations always be tactically sound!</p>




<p><strong>About Me</strong>: I'm Arbab Rafiq, a React Native developer passionate about building developer tools and sports applications. Find me on <a href="https://github.com/arbab-io" rel="noopener noreferrer">GitHub</a>.</p>

