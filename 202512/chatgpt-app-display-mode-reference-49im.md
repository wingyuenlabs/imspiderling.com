---
Title: ChatGPT App Display Mode Reference
Description: 
Author: Abe Wheeler
Date: 2025-12-10T21:49:21.000Z
Robots: noindex,nofollow
Template: index
---
<p>The <a href="https://developers.openai.com/apps-sdk/concepts/ui-guidelines" rel="noopener noreferrer">ChatGPT Apps SDK</a> doesn’t offer a comprehensive breakdown of app display behavior on all Display Modes &amp; screen widths, so I figured I’d do so here.</p>

<h2>
  
  
  Inline
</h2>

<p><a href="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2F5imf3mwa7d926jzn4r35.png" class="article-body-image-wrapper"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2F5imf3mwa7d926jzn4r35.png" alt="Inline Display Mode" width="800" height="449"></a></p>

<p>Inline display mode inserts your resource in the flow of the conversation. Your App iframe is inserted in a div that looks like the following:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight jsx"><code><span class="p">&lt;</span><span class="nt">div</span> <span class="na">class</span><span class="p">=</span><span class="s">"no-scrollbar relative mb-2 @w-sm/main:w-full mx-0 max-sm:-mx-(--thread-content-margin) max-sm:w-[100cqw] max-sm:overflow-hidden overflow-visible"</span><span class="p">&gt;</span>
    <span class="p">&lt;</span><span class="nt">div</span> <span class="na">class</span><span class="p">=</span><span class="s">"relative overflow-hidden h-full"</span> <span class="na">style</span><span class="p">=</span><span class="s">"height: 270px;"</span><span class="p">&gt;</span>
     <span class="p">&lt;</span><span class="nt">iframe</span> <span class="na">class</span><span class="p">=</span><span class="s">"h-full w-full max-w-full"</span><span class="p">&gt;</span>
         <span class="p">&lt;</span><span class="err">!--</span> <span class="na">Your</span> <span class="na">App</span> <span class="err">--</span><span class="p">&gt;</span>
     <span class="p">&lt;/</span><span class="nt">iframe</span><span class="p">&gt;</span>
    <span class="p">&lt;/</span><span class="nt">div</span><span class="p">&gt;</span>
<span class="p">&lt;/</span><span class="nt">div</span><span class="p">&gt;</span>
</code></pre>

</div>



<p>The height of the div is fixed to the height of your Resource, and your Resource can be as tall as you want (I tested up to 20k px). The <code>window.openai.maxHeight</code> global (aka <code>useMaxHeight</code> hook) has been undefined by ChatGPT in all of my tests, and seems to be unused for this display mode.</p>

<h2>
  
  
  Fullscreen
</h2>

<p><a href="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fm6uw8tn9p05h5g6jh0ud.png" class="article-body-image-wrapper"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fm6uw8tn9p05h5g6jh0ud.png" alt="Fullscreen display mode" width="800" height="450"></a></p>

<p>Fullscreen display mode takes up the full conversation space, below the ChatGPT header/nav. This nav converts to the title of your application centered with the X button to exit fullscreen aligned left. Your App iframe is inserted in a div that looks like the following:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight jsx"><code><span class="p">&lt;</span><span class="nt">div</span> <span class="na">class</span><span class="p">=</span><span class="s">"no-scrollbar fixed start-0 end-0 top-0 bottom-0 z-50 mx-auto flex w-auto flex-col overflow-hidden"</span><span class="p">&gt;</span>
    <span class="p">&lt;</span><span class="nt">div</span> <span class="na">class</span><span class="p">=</span><span class="s">"border-token-border-secondary bg-token-bg-primary sm:bg-token-bg-primary z-10 grid h-(--header-height) grid-cols-[1fr_auto_1fr] border-b px-2"</span><span class="p">&gt;</span>
        <span class="p">&lt;</span><span class="err">!--</span> <span class="na">ChatGPT</span> <span class="na">header</span> <span class="err">/</span> <span class="na">nav</span> <span class="err">--</span><span class="p">&gt;</span>
    <span class="p">&lt;/</span><span class="nt">div</span><span class="p">&gt;</span>
    <span class="p">&lt;</span><span class="nt">div</span> <span class="na">class</span><span class="p">=</span><span class="s">"relative overflow-hidden flex-1"</span><span class="p">&gt;</span>
        <span class="p">&lt;</span><span class="nt">iframe</span> <span class="na">class</span><span class="p">=</span><span class="s">"h-full w-full max-w-full"</span><span class="p">&gt;</span>
         <span class="p">&lt;</span><span class="err">!--</span> <span class="na">Your</span> <span class="na">App</span> <span class="err">--</span><span class="p">&gt;</span>
        <span class="p">&lt;/</span><span class="nt">iframe</span><span class="p">&gt;</span>
    <span class="p">&lt;/</span><span class="nt">div</span><span class="p">&gt;</span>
<span class="p">&lt;/</span><span class="nt">div</span><span class="p">&gt;</span>
</code></pre>

</div>



<p>As with inline mode, your Resource can be as tall as you want (I tested up to 20k px). The <code>window.openai.maxHeight</code> global (aka <code>useMaxHeight</code> hook) has been undefined by ChatGPT in all of my tests, and seems to be unused for this display mode as well.</p>

<h2>
  
  
  Picture-in-Picture (PiP)
</h2>

<p><a href="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fgrcuzyn1o4z97wvoorqx.png" class="article-body-image-wrapper"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fgrcuzyn1o4z97wvoorqx.png" alt="PiP_wide.png" width="800" height="450"></a></p>

<p>PiP display mode inserts your resource absolutely, above the conversation. Your App iframe is inserted in a div that looks like the following:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight jsx"><code><span class="p">&lt;</span><span class="nt">div</span> <span class="na">class</span><span class="p">=</span><span class="s">"no-scrollbar @w-xl/main:top-4 fixed start-4 end-4 top-4 z-50 mx-auto max-w-(--thread-content-max-width) sm:start-0 sm:end-0 sm:top-(--header-height) sm:w-full overflow-visible"</span> <span class="na">style</span><span class="p">=</span><span class="s">"max-height: 480.5px;"</span><span class="p">&gt;</span>
    <span class="p">&lt;</span><span class="nt">div</span> <span class="na">class</span><span class="p">=</span><span class="s">"relative overflow-hidden h-full rounded-2xl sm:rounded-3xl shadow-[0px_0px_0px_1px_var(--border-heavy),0px_6px_20px_rgba(0,0,0,0.1)] md:-mx-4"</span> <span class="na">style</span><span class="p">=</span><span class="s">"height: 270px;"</span><span class="p">&gt;</span>
     <span class="p">&lt;</span><span class="nt">iframe</span> <span class="na">class</span><span class="p">=</span><span class="s">"h-full w-full max-w-full"</span><span class="p">&gt;</span>
         <span class="p">&lt;</span><span class="err">!--</span> <span class="na">Your</span> <span class="na">App</span> <span class="err">--</span><span class="p">&gt;</span>
     <span class="p">&lt;/</span><span class="nt">iframe</span><span class="p">&gt;</span>
    <span class="p">&lt;/</span><span class="nt">div</span><span class="p">&gt;</span>
<span class="p">&lt;/</span><span class="nt">div</span><span class="p">&gt;</span>
</code></pre>

</div>



<p>This is the only display mode that uses the <code>window.openai.maxHeight</code> global (aka <code>useMaxHeight</code> hook). Your iframe can assume any height it likes, but content will be scrollable past the <code>maxHeight</code> setting, and the PiP window will not expand beyond that height.</p>

<p>Further, note that PiP is not supported on mobile screen widths and instead coerces to the fullscreen display mode.</p>

<h2>
  
  
  Wrapping Up
</h2>

<p>Practically speaking, each display mode acts like a different client, and your App will have to respond accordingly. The good news is that the only required display mode is inline, which makes our lives easier.</p>

<p>For interactive visuals of each display mode, check out the <a href="https://sunpeak.ai/#simulator" rel="noopener noreferrer">sunpeak ChatGPT simulator</a>!</p>

<p>To get started building ChatGPT Apps with the sunpeak framework, check out the <a href="https://docs.sunpeak.ai/" rel="noopener noreferrer">sunpeak documentation</a>.</p>

<p>If you found this helpful, please <a href="https://github.com/Sunpeak-AI/sunpeak" rel="noopener noreferrer"><strong>star us on GitHub</strong></a>!The <a href="https://developers.openai.com/apps-sdk/concepts/ui-guidelines" rel="noopener noreferrer">ChatGPT Apps SDK</a> doesn’t offer a comprehensive breakdown of app display behavior on all Display Modes &amp; screen widths, so I figured I’d do so here.</p>

