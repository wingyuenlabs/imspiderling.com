---
Title: [SUI] Formularios (Form)
Description: 
Author: GoyesDev
Date: 2026-01-30T21:42:29.000Z
Robots: noindex,nofollow
Template: index
---
<p><a href="https://developer.apple.com/documentation/swiftui/form" rel="noopener noreferrer"><code>Form</code></a> es un contenedor para agrupar controles principalmente usados para configuración de alguna funcionalidad.</p>

<p><code>Form</code> presenta los controles en un <a href="https://developer.apple.com/documentation/swiftui/list" rel="noopener noreferrer"><code>List</code></a> con estilo <a href="https://developer.apple.com/documentation/swiftui/liststyle/grouped" rel="noopener noreferrer"><code>grouped</code></a>, con relleno alrededor.</p>

<p>Notar que, así como el componente <code>List</code>, <code>Form</code> también puede separar el contenido por secciones usando <a href="https://developer.apple.com/documentation/SwiftUI/Section" rel="noopener noreferrer"><code>Section</code></a>.<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight swift"><code><span class="kd">enum</span> <span class="kt">NotifyMeAboutType</span> <span class="p">{</span>
  <span class="k">case</span> <span class="n">directMessages</span><span class="p">,</span> <span class="n">mentions</span><span class="p">,</span> <span class="n">anything</span>
<span class="p">}</span>

<span class="kd">enum</span> <span class="kt">ProfileImageSize</span> <span class="p">{</span>
  <span class="k">case</span> <span class="n">large</span><span class="p">,</span> <span class="n">medium</span><span class="p">,</span> <span class="n">small</span>
<span class="p">}</span>

<span class="kd">struct</span> <span class="kt">ContentView</span><span class="p">:</span> <span class="kt">View</span> <span class="p">{</span>
  <span class="kd">@State</span> <span class="kd">private</span> <span class="k">var</span> <span class="nv">notifyMeAbout</span><span class="p">:</span> <span class="kt">NotifyMeAboutType</span> <span class="o">=</span> <span class="o">.</span><span class="n">anything</span>
  <span class="kd">@State</span> <span class="kd">private</span> <span class="k">var</span> <span class="nv">playNotificationSounds</span> <span class="o">=</span> <span class="kc">false</span>
  <span class="kd">@State</span> <span class="kd">private</span> <span class="k">var</span> <span class="nv">sendReadReceipts</span> <span class="o">=</span> <span class="kc">false</span>
  <span class="kd">@State</span> <span class="kd">private</span> <span class="k">var</span> <span class="nv">profileImageSize</span><span class="p">:</span> <span class="kt">ProfileImageSize</span> <span class="o">=</span> <span class="o">.</span><span class="n">small</span>

  <span class="k">var</span> <span class="nv">body</span><span class="p">:</span> <span class="kd">some</span> <span class="kt">View</span> <span class="p">{</span>
    <span class="kt">NavigationView</span> <span class="p">{</span>
      <span class="kt">Form</span> <span class="p">{</span>
        <span class="kt">Section</span><span class="p">(</span><span class="nv">header</span><span class="p">:</span> <span class="kt">Text</span><span class="p">(</span><span class="s">"Notifications"</span><span class="p">),</span> <span class="nv">footer</span><span class="p">:</span> <span class="kt">Text</span><span class="p">(</span><span class="s">"¿Qué quieres que te notifique?"</span><span class="p">))</span> <span class="p">{</span>
          <span class="kt">Picker</span><span class="p">(</span><span class="s">"Notify Me About"</span><span class="p">,</span> <span class="nv">selection</span><span class="p">:</span> <span class="n">$notifyMeAbout</span><span class="p">)</span> <span class="p">{</span>
            <span class="kt">Text</span><span class="p">(</span><span class="s">"Direct Messages"</span><span class="p">)</span><span class="o">.</span><span class="nf">tag</span><span class="p">(</span><span class="kt">NotifyMeAboutType</span><span class="o">.</span><span class="n">directMessages</span><span class="p">)</span>
            <span class="kt">Text</span><span class="p">(</span><span class="s">"Mentions"</span><span class="p">)</span><span class="o">.</span><span class="nf">tag</span><span class="p">(</span><span class="kt">NotifyMeAboutType</span><span class="o">.</span><span class="n">mentions</span><span class="p">)</span>
            <span class="kt">Text</span><span class="p">(</span><span class="s">"Anything"</span><span class="p">)</span><span class="o">.</span><span class="nf">tag</span><span class="p">(</span><span class="kt">NotifyMeAboutType</span><span class="o">.</span><span class="n">anything</span><span class="p">)</span>
          <span class="p">}</span>
          <span class="kt">Toggle</span><span class="p">(</span><span class="s">"Play notification sounds"</span><span class="p">,</span> <span class="nv">isOn</span><span class="p">:</span> <span class="n">$playNotificationSounds</span><span class="p">)</span>
          <span class="kt">Toggle</span><span class="p">(</span><span class="s">"Send read receipts"</span><span class="p">,</span> <span class="nv">isOn</span><span class="p">:</span> <span class="n">$sendReadReceipts</span><span class="p">)</span>
        <span class="p">}</span>
        <span class="kt">Section</span><span class="p">(</span><span class="nv">header</span><span class="p">:</span> <span class="kt">Text</span><span class="p">(</span><span class="s">"User Profiles"</span><span class="p">))</span> <span class="p">{</span>
          <span class="kt">Picker</span><span class="p">(</span><span class="s">"Profile Image Size"</span><span class="p">,</span> <span class="nv">selection</span><span class="p">:</span> <span class="n">$profileImageSize</span><span class="p">)</span> <span class="p">{</span>
            <span class="kt">Text</span><span class="p">(</span><span class="s">"Large"</span><span class="p">)</span><span class="o">.</span><span class="nf">tag</span><span class="p">(</span><span class="kt">ProfileImageSize</span><span class="o">.</span><span class="n">large</span><span class="p">)</span>
            <span class="kt">Text</span><span class="p">(</span><span class="s">"Medium"</span><span class="p">)</span><span class="o">.</span><span class="nf">tag</span><span class="p">(</span><span class="kt">ProfileImageSize</span><span class="o">.</span><span class="n">medium</span><span class="p">)</span>
            <span class="kt">Text</span><span class="p">(</span><span class="s">"Small"</span><span class="p">)</span><span class="o">.</span><span class="nf">tag</span><span class="p">(</span><span class="kt">ProfileImageSize</span><span class="o">.</span><span class="n">small</span><span class="p">)</span>
          <span class="p">}</span>
          <span class="kt">Button</span><span class="p">(</span><span class="s">"Clear Image Cache"</span><span class="p">)</span> <span class="p">{}</span>
        <span class="p">}</span>
      <span class="p">}</span>
    <span class="p">}</span>
  <span class="p">}</span>
<span class="p">}</span>
</code></pre>

</div>



<p><a href="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2F7dwo8srj3x14ysot7svl.png" class="article-body-image-wrapper"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2F7dwo8srj3x14ysot7svl.png" alt=" " width="339" height="280"></a></p>

