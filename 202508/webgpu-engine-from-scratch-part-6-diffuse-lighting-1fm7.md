---
Title: WebGPU Engine from Scratch Part 6: Diffuse Lighting
Description: 
Author: ndesmic
Date: 2025-08-25T19:00:00.000Z
Robots: noindex,nofollow
Template: index
---
<p>The scene is bland and could use some lighting so let's implement that.  First we start with the <code>Light</code> entity:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight typescript"><code><span class="c1">//light.js</span>
<span class="cm">/**
 * @typedef {"point" | "directional" | "spot" } LightType
 */</span>

<span class="k">export</span> <span class="kd">class</span> <span class="nc">Light</span> <span class="p">{</span>
    <span class="cm">/**@type {LightType} */</span>
    <span class="err">#</span><span class="kd">type</span><span class="p">;</span>
    <span class="err">#</span><span class="nx">position</span><span class="p">;</span>
    <span class="err">#</span><span class="nx">direction</span><span class="p">;</span>
    <span class="err">#</span><span class="nx">color</span><span class="p">;</span>

    <span class="nf">constructor</span><span class="p">(</span><span class="nx">light</span><span class="p">){</span>
        <span class="k">this</span><span class="p">.</span><span class="err">#</span><span class="kd">type</span> <span class="o">=</span> <span class="nx">light</span><span class="p">.</span><span class="kd">type</span> <span class="o">??</span> <span class="dl">"</span><span class="s2">point</span><span class="dl">"</span><span class="p">;</span>
        <span class="k">this</span><span class="p">.</span><span class="err">#</span><span class="nx">position</span> <span class="o">=</span> <span class="nx">light</span><span class="p">.</span><span class="nx">position</span> <span class="o">??</span> <span class="p">[</span><span class="mi">0</span><span class="p">,</span> <span class="mi">0</span><span class="p">,</span> <span class="mi">0</span><span class="p">,</span> <span class="mi">0</span><span class="p">];</span>
        <span class="k">this</span><span class="p">.</span><span class="err">#</span><span class="nx">direction</span> <span class="o">=</span> <span class="nx">light</span><span class="p">.</span><span class="nx">direction</span> <span class="o">??</span> <span class="p">[</span><span class="mi">0</span><span class="p">,</span> <span class="mi">0</span><span class="p">,</span> <span class="mi">0</span><span class="p">,</span> <span class="mi">0</span><span class="p">];</span>
        <span class="k">this</span><span class="p">.</span><span class="err">#</span><span class="nx">color</span> <span class="o">=</span> <span class="nx">light</span><span class="p">.</span><span class="nx">color</span> <span class="o">??</span> <span class="p">[</span><span class="mi">1</span><span class="p">,</span><span class="mi">1</span><span class="p">,</span><span class="mi">1</span><span class="p">,</span><span class="mi">1</span><span class="p">];</span>
    <span class="p">}</span>

    <span class="kd">set</span> <span class="nf">type</span><span class="p">(</span><span class="nx">val</span><span class="p">){</span>
        <span class="k">this</span><span class="p">.</span><span class="err">#</span><span class="kd">type</span> <span class="o">=</span> <span class="k">new</span> <span class="nc">Float32Array</span><span class="p">(</span><span class="nx">val</span><span class="p">);</span>
    <span class="p">}</span>
    <span class="kd">get</span> <span class="nf">type</span><span class="p">(){</span>
        <span class="k">return</span> <span class="k">this</span><span class="p">.</span><span class="err">#</span><span class="kd">type</span><span class="p">;</span>
    <span class="p">}</span>

    <span class="kd">set</span> <span class="nf">position</span><span class="p">(</span><span class="nx">val</span><span class="p">){</span>
        <span class="k">this</span><span class="p">.</span><span class="err">#</span><span class="nx">position</span> <span class="o">=</span> <span class="k">new</span> <span class="nc">Float32Array</span><span class="p">(</span><span class="nx">val</span><span class="p">);</span>
    <span class="p">}</span>
    <span class="kd">get</span> <span class="nf">position</span><span class="p">(){</span>
        <span class="k">return</span> <span class="k">this</span><span class="p">.</span><span class="err">#</span><span class="nx">position</span><span class="p">;</span>
    <span class="p">}</span>

    <span class="kd">set</span> <span class="nf">direction</span><span class="p">(</span><span class="nx">val</span><span class="p">){</span>
        <span class="k">this</span><span class="p">.</span><span class="err">#</span><span class="nx">direction</span> <span class="o">=</span> <span class="k">new</span> <span class="nc">Float32Array</span><span class="p">(</span><span class="nx">val</span><span class="p">);</span>
    <span class="p">}</span>
    <span class="kd">get</span> <span class="nf">direction</span><span class="p">(){</span>
        <span class="k">return</span> <span class="k">this</span><span class="p">.</span><span class="err">#</span><span class="nx">direction</span><span class="p">;</span>
    <span class="p">}</span>

    <span class="kd">set</span> <span class="nf">color</span><span class="p">(</span><span class="nx">val</span><span class="p">){</span>
        <span class="k">this</span><span class="p">.</span><span class="err">#</span><span class="nx">color</span> <span class="o">=</span> <span class="k">new</span> <span class="nc">Float32Array</span><span class="p">(</span><span class="nx">val</span><span class="p">);</span>
    <span class="p">}</span>
    <span class="kd">get</span> <span class="nf">color</span><span class="p">(){</span>
        <span class="k">return</span> <span class="k">this</span><span class="p">.</span><span class="err">#</span><span class="nx">color</span><span class="p">;</span>
    <span class="p">}</span>
<span class="p">}</span>
</code></pre>

</div>



<p>Similar to the WebGL version.  Then let's create a light<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight typescript"><code><span class="nf">initializeLights</span><span class="p">(){</span>
    <span class="k">this</span><span class="p">.</span><span class="err">#</span><span class="nx">lights</span><span class="p">.</span><span class="nf">set</span><span class="p">(</span><span class="dl">"</span><span class="s2">light</span><span class="dl">"</span><span class="p">,</span> <span class="k">new</span> <span class="nc">Light</span><span class="p">({</span>
        <span class="na">type</span><span class="p">:</span> <span class="dl">"</span><span class="s2">point</span><span class="dl">"</span><span class="p">,</span>
        <span class="na">position</span><span class="p">:</span> <span class="nf">sphericalToCartesian</span><span class="p">([</span> <span class="nb">Math</span><span class="p">.</span><span class="nx">PI</span> <span class="o">/</span> <span class="mi">4</span><span class="p">,</span> <span class="nb">Math</span><span class="p">.</span><span class="nx">PI</span> <span class="o">/</span> <span class="mi">4</span><span class="p">,</span> <span class="mi">2</span><span class="p">]),</span>
    <span class="p">}));</span>
<span class="p">}</span>
</code></pre>

</div>



<p>This is the same as the other initialization method and need <code>this.#lights</code> set as a <code>Map</code>.  Then we need to pass the light to the shader in a bindgroup.</p>

<p>I struggled for a while to get the correct size for the buffer and the correct offsets.  Lots of little mistakes.  I finally came up with a genericized way to pack structs:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight typescript"><code><span class="c1">//buffer-utils.js</span>
<span class="cm">/**
 * @typedef {[string,GpuType]} Prop
 * @typedef {Prop[]} Schema
 * @param {object} data 
 * @param {Schema} schema 
 */</span>
<span class="k">export</span> <span class="kd">function</span> <span class="nf">packStruct</span><span class="p">(</span><span class="nx">data</span><span class="p">,</span> <span class="nx">schema</span><span class="p">,</span> <span class="nx">minSize</span><span class="p">){</span>
    <span class="kd">const</span> <span class="p">{</span> <span class="nx">offsets</span><span class="p">,</span> <span class="nx">totalSize</span> <span class="p">}</span> <span class="o">=</span> <span class="nf">getAlignments</span><span class="p">(</span><span class="nx">schema</span><span class="p">.</span><span class="nf">map</span><span class="p">(</span><span class="nx">s</span> <span class="o">=&gt;</span> <span class="nx">s</span><span class="p">[</span><span class="mi">1</span><span class="p">]),</span> <span class="nx">minSize</span><span class="p">);</span>
    <span class="kd">const</span> <span class="nx">buffer</span> <span class="o">=</span> <span class="k">new</span> <span class="nc">ArrayBuffer</span><span class="p">(</span><span class="nx">totalSize</span><span class="p">);</span>
    <span class="kd">const</span> <span class="nx">dataView</span> <span class="o">=</span> <span class="k">new</span> <span class="nc">DataView</span><span class="p">(</span><span class="nx">buffer</span><span class="p">);</span>

    <span class="k">for</span><span class="p">(</span><span class="kd">let</span> <span class="nx">i</span> <span class="o">=</span> <span class="mi">0</span><span class="p">;</span> <span class="nx">i</span> <span class="o">&lt;</span> <span class="nx">schema</span><span class="p">.</span><span class="nx">length</span><span class="p">;</span> <span class="nx">i</span><span class="o">++</span><span class="p">){</span>
        <span class="kd">const</span> <span class="p">[</span><span class="nx">name</span><span class="p">,</span> <span class="kd">type</span><span class="p">]</span> <span class="o">=</span> <span class="nx">schema</span><span class="p">[</span><span class="nx">i</span><span class="p">];</span>
        <span class="kd">const</span> <span class="nx">value</span> <span class="o">=</span> <span class="nx">data</span><span class="p">[</span><span class="nx">name</span><span class="p">];</span>
        <span class="c1">//TODO: add other GPU Types</span>
        <span class="k">switch</span><span class="p">(</span><span class="kd">type</span><span class="p">){</span>
            <span class="k">case</span> <span class="dl">"</span><span class="s2">u32</span><span class="dl">"</span><span class="p">:</span> <span class="p">{</span>
                <span class="nx">dataView</span><span class="p">.</span><span class="nf">setUint32</span><span class="p">(</span><span class="nx">offsets</span><span class="p">[</span><span class="nx">i</span><span class="p">],</span> <span class="nx">value</span><span class="p">,</span> <span class="kc">true</span><span class="p">);</span>
                <span class="k">break</span><span class="p">;</span>
            <span class="p">}</span>
            <span class="k">case</span> <span class="dl">"</span><span class="s2">vec3f32</span><span class="dl">"</span><span class="p">:</span> <span class="p">{</span>
                <span class="nx">dataView</span><span class="p">.</span><span class="nf">setFloat32</span><span class="p">(</span><span class="nx">offsets</span><span class="p">[</span><span class="nx">i</span><span class="p">],</span> <span class="nx">value</span><span class="p">[</span><span class="mi">0</span><span class="p">],</span> <span class="kc">true</span><span class="p">);</span>
                <span class="nx">dataView</span><span class="p">.</span><span class="nf">setFloat32</span><span class="p">(</span><span class="nx">offsets</span><span class="p">[</span><span class="nx">i</span><span class="p">]</span> <span class="o">+</span> <span class="mi">4</span><span class="p">,</span> <span class="nx">value</span><span class="p">[</span><span class="mi">1</span><span class="p">],</span> <span class="kc">true</span><span class="p">);</span>
                <span class="nx">dataView</span><span class="p">.</span><span class="nf">setFloat32</span><span class="p">(</span><span class="nx">offsets</span><span class="p">[</span><span class="nx">i</span><span class="p">]</span> <span class="o">+</span> <span class="mi">8</span><span class="p">,</span> <span class="nx">value</span><span class="p">[</span><span class="mi">2</span><span class="p">],</span> <span class="kc">true</span><span class="p">);</span>
                <span class="k">break</span><span class="p">;</span>
            <span class="p">}</span>
            <span class="k">case</span> <span class="dl">"</span><span class="s2">vec4f32</span><span class="dl">"</span><span class="p">:</span> <span class="p">{</span>
                <span class="nx">dataView</span><span class="p">.</span><span class="nf">setFloat32</span><span class="p">(</span><span class="nx">offsets</span><span class="p">[</span><span class="nx">i</span><span class="p">],</span> <span class="nx">value</span><span class="p">[</span><span class="mi">0</span><span class="p">],</span> <span class="kc">true</span><span class="p">);</span>
                <span class="nx">dataView</span><span class="p">.</span><span class="nf">setFloat32</span><span class="p">(</span><span class="nx">offsets</span><span class="p">[</span><span class="nx">i</span><span class="p">]</span> <span class="o">+</span> <span class="mi">4</span><span class="p">,</span> <span class="nx">value</span><span class="p">[</span><span class="mi">1</span><span class="p">],</span> <span class="kc">true</span><span class="p">);</span>
                <span class="nx">dataView</span><span class="p">.</span><span class="nf">setFloat32</span><span class="p">(</span><span class="nx">offsets</span><span class="p">[</span><span class="nx">i</span><span class="p">]</span> <span class="o">+</span> <span class="mi">8</span><span class="p">,</span> <span class="nx">value</span><span class="p">[</span><span class="mi">2</span><span class="p">],</span> <span class="kc">true</span><span class="p">);</span>
                <span class="nx">dataView</span><span class="p">.</span><span class="nf">setFloat32</span><span class="p">(</span><span class="nx">offsets</span><span class="p">[</span><span class="nx">i</span><span class="p">]</span> <span class="o">+</span> <span class="mi">12</span><span class="p">,</span> <span class="nx">value</span><span class="p">[</span><span class="mi">3</span><span class="p">],</span> <span class="kc">true</span><span class="p">);</span>
                <span class="k">break</span><span class="p">;</span>
            <span class="p">}</span>
            <span class="nl">default</span><span class="p">:</span> <span class="p">{</span>
<span class="err"> </span> <span class="err"> </span> <span class="err"> </span> <span class="err"> </span> <span class="err"> </span> <span class="err"> </span> <span class="err"> </span> <span class="err"> </span> <span class="k">throw</span> <span class="k">new</span> <span class="nc">Error</span><span class="p">(</span><span class="s2">`Cannot pack type </span><span class="p">${</span><span class="kd">type</span><span class="p">}</span><span class="s2"> at prop index </span><span class="p">${</span><span class="nx">i</span><span class="p">}</span><span class="s2"> with value </span><span class="p">${</span><span class="nx">value</span><span class="p">}</span><span class="s2">`</span><span class="p">);</span>
<span class="err"> </span> <span class="err"> </span> <span class="err"> </span> <span class="err"> </span> <span class="err"> </span> <span class="err"> </span> <span class="p">}</span>
        <span class="p">}</span>
    <span class="p">}</span>

    <span class="k">return</span> <span class="nx">buffer</span><span class="p">;</span>
<span class="p">}</span>

</code></pre>

</div>



<p>This is only partially complete as it only covers the types that I need for the light right now but filling the rest in should be easy and an exercise to the reader (and my future self).  With that we can pack a light into a buffer.<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight typescript"><code><span class="c1">//buffer-utils.js</span>
<span class="cm">/**
 * 
 * @param {Light} light 
 */</span>
<span class="k">export</span> <span class="kd">function</span> <span class="nf">packLight</span><span class="p">(</span><span class="nx">light</span><span class="p">){</span>
    <span class="kd">const</span> <span class="nx">schema</span> <span class="o">=</span> <span class="p">[</span>
        <span class="p">[</span><span class="dl">"</span><span class="s2">typeInt</span><span class="dl">"</span><span class="p">,</span> <span class="dl">"</span><span class="s2">u32</span><span class="dl">"</span><span class="p">],</span>
        <span class="p">[</span><span class="dl">"</span><span class="s2">position</span><span class="dl">"</span><span class="p">,</span> <span class="dl">"</span><span class="s2">vec3f32</span><span class="dl">"</span><span class="p">],</span>
        <span class="p">[</span><span class="dl">"</span><span class="s2">direction</span><span class="dl">"</span><span class="p">,</span> <span class="dl">"</span><span class="s2">vec3f32</span><span class="dl">"</span><span class="p">],</span>
        <span class="p">[</span><span class="dl">"</span><span class="s2">color</span><span class="dl">"</span><span class="p">,</span> <span class="dl">"</span><span class="s2">vec4f32</span><span class="dl">"</span><span class="p">]</span>
    <span class="p">];</span>
    <span class="k">return</span> <span class="nf">packStruct</span><span class="p">(</span><span class="nx">light</span><span class="p">,</span> <span class="nx">schema</span><span class="p">,</span> <span class="mi">64</span><span class="p">);</span>
<span class="p">}</span>
</code></pre>

</div>



<p>This illustrates how to use <code>packStruct</code>.  It only works for shallow structs but you give tuples of name and type.  The name looks up the property in the object.  It's an array because I want to maintain strict ordering so there are no surprises.</p>

<p>We can then set the bind group.  I've made it a 3rd group and added it to <code>setMainBindGroups</code>.<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight typescript"><code><span class="nf">setMainLightBindGroup</span><span class="p">(</span><span class="nx">passEncoder</span><span class="p">,</span> <span class="nx">bindGroupLayouts</span><span class="p">,</span> <span class="nx">lights</span><span class="p">){</span>
    <span class="kd">const</span> <span class="nx">lightData</span> <span class="o">=</span> <span class="nf">packLight</span><span class="p">(</span><span class="nx">lights</span><span class="p">.</span><span class="nf">get</span><span class="p">(</span><span class="dl">"</span><span class="s2">light</span><span class="dl">"</span><span class="p">));</span>
    <span class="kd">const</span> <span class="nx">bufferSize</span> <span class="o">=</span> <span class="nx">lightData</span><span class="p">.</span><span class="nx">byteLength</span><span class="p">;</span>
    <span class="kd">const</span> <span class="nx">lightBuffer</span> <span class="o">=</span> <span class="k">this</span><span class="p">.</span><span class="err">#</span><span class="nx">device</span><span class="p">.</span><span class="nf">createBuffer</span><span class="p">({</span>
        <span class="na">size</span><span class="p">:</span> <span class="nx">bufferSize</span><span class="p">,</span>
        <span class="na">usage</span><span class="p">:</span> <span class="nx">GPUBufferUsage</span><span class="p">.</span><span class="nx">UNIFORM</span> <span class="o">|</span> <span class="nx">GPUBufferUsage</span><span class="p">.</span><span class="nx">COPY_DST</span> <span class="o">|</span> <span class="nx">GPUBufferUsage</span><span class="p">.</span><span class="nx">STORAGE</span><span class="p">,</span>
        <span class="na">label</span><span class="p">:</span> <span class="dl">"</span><span class="s2">main-light-buffer</span><span class="dl">"</span>
    <span class="p">});</span>
    <span class="k">this</span><span class="p">.</span><span class="err">#</span><span class="nx">device</span><span class="p">.</span><span class="nx">queue</span><span class="p">.</span><span class="nf">writeBuffer</span><span class="p">(</span><span class="nx">lightBuffer</span><span class="p">,</span> <span class="mi">0</span><span class="p">,</span> <span class="nx">lightData</span><span class="p">);</span>
    <span class="kd">const</span> <span class="nx">lightCountBuffer</span> <span class="o">=</span> <span class="k">this</span><span class="p">.</span><span class="err">#</span><span class="nx">device</span><span class="p">.</span><span class="nf">createBuffer</span><span class="p">({</span>
        <span class="na">size</span><span class="p">:</span> <span class="mi">4</span><span class="p">,</span>
        <span class="na">usage</span><span class="p">:</span> <span class="nx">GPUBufferUsage</span><span class="p">.</span><span class="nx">UNIFORM</span> <span class="o">|</span> <span class="nx">GPUBufferUsage</span><span class="p">.</span><span class="nx">COPY_DST</span><span class="p">,</span>
        <span class="na">label</span><span class="p">:</span> <span class="dl">"</span><span class="s2">main-light-count-buffer</span><span class="dl">"</span>
    <span class="p">});</span>
    <span class="k">this</span><span class="p">.</span><span class="err">#</span><span class="nx">device</span><span class="p">.</span><span class="nx">queue</span><span class="p">.</span><span class="nf">writeBuffer</span><span class="p">(</span><span class="nx">lightCountBuffer</span><span class="p">,</span> <span class="mi">0</span><span class="p">,</span> <span class="k">new</span> <span class="nc">Int32Array</span><span class="p">([</span><span class="nx">lights</span><span class="p">.</span><span class="nx">size</span><span class="p">]));</span>
    <span class="kd">const</span> <span class="nx">lightBindGroup</span> <span class="o">=</span> <span class="k">this</span><span class="p">.</span><span class="err">#</span><span class="nx">device</span><span class="p">.</span><span class="nf">createBindGroup</span><span class="p">({</span>
        <span class="na">label</span><span class="p">:</span> <span class="dl">"</span><span class="s2">main-light-bind-group</span><span class="dl">"</span><span class="p">,</span>
        <span class="na">layout</span><span class="p">:</span> <span class="nx">bindGroupLayouts</span><span class="p">.</span><span class="nf">get</span><span class="p">(</span><span class="dl">"</span><span class="s2">lights</span><span class="dl">"</span><span class="p">),</span>
        <span class="na">entries</span><span class="p">:</span> <span class="p">[</span>
            <span class="p">{</span>
                <span class="na">binding</span><span class="p">:</span> <span class="mi">0</span><span class="p">,</span>
                <span class="na">resource</span><span class="p">:</span> <span class="p">{</span>
                    <span class="na">buffer</span><span class="p">:</span> <span class="nx">lightBuffer</span><span class="p">,</span>
                    <span class="na">offset</span><span class="p">:</span> <span class="mi">0</span><span class="p">,</span>
                    <span class="na">size</span><span class="p">:</span> <span class="nx">bufferSize</span>
                <span class="p">}</span>
            <span class="p">},</span>
            <span class="p">{</span>
                <span class="na">binding</span><span class="p">:</span> <span class="mi">1</span><span class="p">,</span>
                <span class="na">resource</span><span class="p">:</span> <span class="p">{</span>
                    <span class="na">buffer</span><span class="p">:</span> <span class="nx">lightCountBuffer</span><span class="p">,</span>
                    <span class="na">offset</span><span class="p">:</span> <span class="mi">0</span><span class="p">,</span>
                    <span class="na">size</span><span class="p">:</span> <span class="mi">4</span>
                <span class="p">}</span>
            <span class="p">}</span>
        <span class="p">]</span>
    <span class="p">});</span>
    <span class="nx">passEncoder</span><span class="p">.</span><span class="nf">setBindGroup</span><span class="p">(</span><span class="mi">2</span><span class="p">,</span> <span class="nx">lightBindGroup</span><span class="p">);</span>
<span class="p">}</span>
</code></pre>

</div>



<p>This process should be well understood by now.  We are passing a light (we'll see soon that this can generalize to many lights) and the count of lights (the reason why explained below).  Also keep in mind that <code>lightBuffer</code> has usage <code>GPUBufferUsage.STORAGE</code>.  We'll test things out in a shader based on our previous one.<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight plaintext"><code>//textured-lit.wgsl
struct VertexOut {
    @builtin(position) position : vec4&lt;f32&gt;,
    @location(0) uv : vec2&lt;f32&gt;
};
struct Uniforms {
    view_matrix: mat4x4&lt;f32&gt;,
    projection_matrix: mat4x4&lt;f32&gt;,
    model_matrix: mat4x4&lt;f32&gt;,
    normal_matrix: mat3x3&lt;f32&gt;,
    camera_position: vec3&lt;f32&gt;
}
struct Light {
    light_type: u32,
    position: vec3&lt;f32&gt;,
    direction: vec3&lt;f32&gt;,
    color: vec4&lt;f32&gt;
}
struct LightCount {
    count: u32
}

@group(0) @binding(0) var&lt;uniform&gt; uniforms : Uniforms;
@group(1) @binding(0) var main_sampler: sampler;
@group(1) @binding(1) var texture: texture_2d&lt;f32&gt;;
@group(2) @binding(0) var&lt;storage, read&gt; lights: array&lt;Light&gt;;
@group(2) @binding(1) var&lt;uniform&gt; light_count: LightCount;

@vertex
fn vertex_main(@location(0) position: vec3&lt;f32&gt;, @location(1) uv: vec2&lt;f32&gt;) -&gt; VertexOut
{
    var output : VertexOut;
    output.position =  uniforms.projection_matrix * uniforms.view_matrix * uniforms.model_matrix * vec4&lt;f32&gt;(position, 1.0);
    output.uv = uv;
    return output;
}
@fragment
fn fragment_main(fragData: VertexOut) -&gt; @location(0) vec4&lt;f32&gt;
{
    var light = lights[0];
    var lc = light_count.count;

    var tex = textureSample(texture, main_sampler, fragData.uv);
    return light.color;
}
</code></pre>

</div>



<p>We need to define the struct for the <code>Light</code> which is pretty straight-forward.  What's a little more odd is <code>LightCount</code> which holds a single <code>u32</code>.  <strong><code>LightCount</code> must be a struct</strong>.  We cannot bind directly to scalar values for whatever reason so they always need to be boxed in a struct.  The other interesting part here is how we define an array of lights with <code>var&lt;storage, read&gt; lights: array&lt;Light&gt;;</code> This is cool because unlike WebGL we can actually pass arrays of dynamic length, however it is limited in that the length is set by the binding and we do not get a length in the shader value which is why we need to pass the count manually.  This array construct is also why the buffer usage was tagged with <code>GPUBufferUsage.STORAGE</code> because it's treated differently by the GPU.</p>

<p>In the fragment shader we're just doing a test to see if we can output the light color (white).  This will help ensure that we got all the data packing correct.  We need some dummy variables for light count and the texture otherwise they get erased for no-usage on compile and we'll get errors (I hate this about WebGPU).</p>

<p>If everything went well and we pray to the alignment gods we should see something like this.</p>

<p><a href="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fd10fyi07rjoxov32e7wi.png" class="article-body-image-wrapper"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fd10fyi07rjoxov32e7wi.png" alt="White teapot and rug" width="800" height="450"></a></p>

<p>Cool.</p>

<p>With that hooked up correctly now we can actually get to making our shader use normal diffuse lighting.<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight plaintext"><code>struct VertexOut {
    @builtin(position) frag_position : vec4&lt;f32&gt;,
    @location(0) world_position: vec4&lt;f32&gt;,
    @location(1) uv : vec2&lt;f32&gt;,
    @location(2) normal : vec3&lt;f32&gt;
};
struct Uniforms {
    view_matrix: mat4x4&lt;f32&gt;,
    projection_matrix: mat4x4&lt;f32&gt;,
    model_matrix: mat4x4&lt;f32&gt;,
    normal_matrix: mat3x3&lt;f32&gt;,
    camera_position: vec3&lt;f32&gt;
}
struct Light {
    light_type: u32,
    position: vec3&lt;f32&gt;,
    direction: vec3&lt;f32&gt;,
    color: vec4&lt;f32&gt;
}
struct LightCount {
    count: u32
}

@group(0) @binding(0) var&lt;uniform&gt; uniforms : Uniforms;
@group(1) @binding(0) var main_sampler: sampler;
@group(1) @binding(1) var texture: texture_2d&lt;f32&gt;;
@group(2) @binding(0) var&lt;storage, read&gt; lights: array&lt;Light&gt;;
@group(2) @binding(1) var&lt;uniform&gt; light_count: LightCount;

@vertex
fn vertex_main(@location(0) position: vec3&lt;f32&gt;, @location(1) uv: vec2&lt;f32&gt;, @location(2) normal: vec3&lt;f32&gt;) -&gt; VertexOut
{
    var output : VertexOut;
    output.frag_position =  uniforms.projection_matrix * uniforms.view_matrix * uniforms.model_matrix * vec4&lt;f32&gt;(position, 1.0);
    output.world_position = uniforms.model_matrix * vec4&lt;f32&gt;(position, 1.0);
    output.uv = uv;
    output.normal = normal;
    return output;
}
@fragment
fn fragment_main(frag_data: VertexOut) -&gt; @location(0) vec4&lt;f32&gt;
{   
    var surface_color = textureSample(texture, main_sampler, frag_data.uv);
    var diffuse = vec4(0.0);

    for(var i: u32 = 0; i &lt; light_count.count; i++){
        var light = lights[i];
        var light_position = vec4(light.position.xyz, 1.0);
        var to_light = normalize(light_position - frag_data.world_position);
        var light_intensity = max(dot(normalize(frag_data.normal), to_light.xyz), 0.0);
        diffuse += light.color * vec4(light_intensity, light_intensity, light_intensity, 1);
    }

    return surface_color * diffuse;
}
</code></pre>

</div>



<p>We need to modify the pipeline to pass in the normals and pass that through the vertex shader along with the world position of the fragment.  In the fragment shader we get the base color and apply each light in sequence.  We get the direction of the light and then dot that with the normal to figure out how much gets reflected (if the light is the same direction as the normal it all does).  Then we take the light color, surface color and intensity and multiply it together for the final color.  Repeat for each light.</p>

<p>This works with one light but what an actual array of lights?  We need a small modification.<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight typescript"><code><span class="c1">//buffer-utils.js</span>
<span class="k">export</span> <span class="kd">function</span> <span class="nf">packArray</span><span class="p">(</span><span class="nx">data</span><span class="p">,</span> <span class="nx">schema</span><span class="p">,</span> <span class="nx">minSize</span><span class="p">){</span>
    <span class="kd">const</span> <span class="p">{</span> <span class="na">totalSize</span><span class="p">:</span> <span class="nx">structSize</span> <span class="p">}</span> <span class="o">=</span> <span class="nf">getAlignments</span><span class="p">(</span><span class="nx">schema</span><span class="p">.</span><span class="nf">map</span><span class="p">(</span><span class="nx">s</span> <span class="o">=&gt;</span> <span class="nx">s</span><span class="p">[</span><span class="mi">1</span><span class="p">]),</span> <span class="nx">minSize</span><span class="p">);</span>
    <span class="kd">const</span> <span class="nx">totalSize</span> <span class="o">=</span> <span class="nx">structSize</span> <span class="o">*</span> <span class="nx">data</span><span class="p">.</span><span class="nx">length</span><span class="p">;</span>
    <span class="kd">const</span> <span class="nx">buffer</span> <span class="o">=</span> <span class="k">new</span> <span class="nc">ArrayBuffer</span><span class="p">(</span><span class="nx">totalSize</span><span class="p">);</span>
    <span class="k">for</span><span class="p">(</span><span class="kd">let</span> <span class="nx">i</span> <span class="o">=</span> <span class="mi">0</span><span class="p">;</span> <span class="nx">i</span> <span class="o">&lt;</span> <span class="nx">data</span><span class="p">.</span><span class="nx">length</span><span class="p">;</span> <span class="nx">i</span><span class="o">++</span><span class="p">){</span>
        <span class="nf">packStruct</span><span class="p">(</span><span class="nx">data</span><span class="p">[</span><span class="nx">i</span><span class="p">],</span> <span class="nx">schema</span><span class="p">,</span> <span class="nx">minSize</span><span class="p">,</span> <span class="nx">buffer</span><span class="p">,</span> <span class="nx">i</span> <span class="o">*</span> <span class="nx">structSize</span><span class="p">);</span>
    <span class="p">}</span>
    <span class="k">return</span> <span class="nx">buffer</span><span class="p">;</span>
<span class="p">}</span>
</code></pre>

</div>



<p>To pack an array we need to take the max align of the struct and make sure each element aligns on that boundary.  Luckily the padding from packing a struct is the same thing so we just need to put it into a single buffer.  To do this, I modified <code>packStruct</code> to optionally take a buffer and offset for where to place the packed bytes.  Not worth showing here but you can look at the final code.  Then we just modify <code>packLight</code> into <code>packLights</code> and use <code>packArray</code> instead.</p>

<p>When we apply it something weird happens.</p>

<p><a href="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fbq7cirxj3r14vmfzrz9y.png" class="article-body-image-wrapper"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fbq7cirxj3r14vmfzrz9y.png" alt="Side of teapot with red light. Rug is lit toward camera but teapot is lit from bottom" width="800" height="450"></a></p>

<p>Here I have a red light at <code>0, 0, -1</code>.  This should be slightly in front of the camera and on the surface mesh as seen from above (toward -Y) we can see that it is.</p>

<p><a href="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fsxc4ojz9y0ltrg4vmyu2.png" class="article-body-image-wrapper"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fsxc4ojz9y0ltrg4vmyu2.png" alt="Top down of teapot with red light on bottom of plane" width="800" height="450"></a></p>

<p>But the bottom of the teapot is light up, not the side.  The -X and +X work just fine though, but it appears as though Y and Z are flipped!  It took a while to figure out but it's because when we bake the transforms we didn't do the normals!  Recall that to properly transform normals we need to get the top left 3x3 of the model matrix and get the transpose of the inverse (in this case we do not need to multiply by the view matrix since it's in model space).  This also requires use to alter the matrix-vector multiplication because we had assumed it was always going to be a 4x4 times a 4, but now we need variable length.<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight typescript"><code><span class="c1">//vector.js</span>
<span class="cm">/**
 * 
 * @param {Float32Array} vector 
 * @param {Float32Array} matrix
 * @param {number} size
 * @returns 
 */</span>
<span class="k">export</span> <span class="kd">function</span> <span class="nf">multiplyMatrixVector</span><span class="p">(</span><span class="nx">vector</span><span class="p">,</span> <span class="nx">matrix</span><span class="p">,</span> <span class="nx">size</span><span class="p">)</span> <span class="p">{</span>
    <span class="kd">const</span> <span class="nx">newVector</span> <span class="o">=</span> <span class="k">new</span> <span class="nc">Float32Array</span><span class="p">(</span><span class="nx">size</span><span class="p">);</span>
    <span class="k">for</span><span class="p">(</span><span class="kd">let</span> <span class="nx">i</span> <span class="o">=</span> <span class="mi">0</span><span class="p">;</span> <span class="nx">i</span> <span class="o">&lt;</span> <span class="nx">size</span><span class="p">;</span> <span class="nx">i</span><span class="o">++</span><span class="p">){</span>
        <span class="kd">let</span> <span class="nx">sum</span> <span class="o">=</span> <span class="mi">0</span><span class="p">;</span>
        <span class="k">for</span><span class="p">(</span><span class="kd">let</span> <span class="nx">j</span> <span class="o">=</span> <span class="mi">0</span><span class="p">;</span> <span class="nx">j</span> <span class="o">&lt;</span> <span class="nx">size</span><span class="p">;</span> <span class="nx">j</span><span class="o">++</span><span class="p">){</span>
            <span class="nx">sum</span> <span class="o">+=</span> <span class="nx">vector</span><span class="p">[</span><span class="nx">j</span><span class="p">]</span> <span class="o">*</span> <span class="nx">matrix</span><span class="p">[</span><span class="nx">i</span> <span class="o">*</span> <span class="nx">size</span> <span class="o">+</span> <span class="nx">j</span><span class="p">]</span> 
        <span class="p">}</span>
        <span class="nx">newVector</span><span class="p">.</span><span class="nf">set</span><span class="p">([</span><span class="nx">sum</span><span class="p">],</span> <span class="nx">i</span><span class="p">);</span>
    <span class="p">}</span>

    <span class="k">return</span> <span class="nx">newVector</span><span class="p">;</span>
<span class="p">}</span>
</code></pre>

</div>



<p>The whole <code>bakeTransforms</code><br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight typescript"><code><span class="c1">//mesh.js</span>
<span class="nf">bakeTransforms</span><span class="p">(){</span>
    <span class="c1">//positions</span>
    <span class="kd">const</span> <span class="nx">modelMatrix</span> <span class="o">=</span> <span class="k">this</span><span class="p">.</span><span class="nf">getModelMatrix</span><span class="p">();</span>
    <span class="kd">const</span> <span class="nx">transformedPositions</span> <span class="o">=</span> <span class="nf">chunk</span><span class="p">(</span><span class="k">this</span><span class="p">.</span><span class="nx">positions</span><span class="p">,</span> <span class="k">this</span><span class="p">.</span><span class="nx">positionSize</span><span class="p">)</span>
        <span class="p">.</span><span class="nf">map</span><span class="p">(</span><span class="nx">values</span> <span class="o">=&gt;</span> <span class="p">{</span>
            <span class="kd">const</span> <span class="nx">lengthToPad</span> <span class="o">=</span> <span class="mi">4</span> <span class="o">-</span> <span class="nx">values</span><span class="p">.</span><span class="nx">length</span><span class="p">;</span>
            <span class="k">switch</span><span class="p">(</span><span class="nx">lengthToPad</span><span class="p">){</span>
                <span class="k">case</span> <span class="mi">1</span><span class="p">:{</span>
                    <span class="k">return</span> <span class="p">[...</span><span class="nx">values</span><span class="p">,</span> <span class="mf">1.0</span><span class="p">]</span>
                <span class="p">}</span>
                <span class="k">case</span> <span class="mi">2</span><span class="p">:{</span>
                    <span class="k">return</span> <span class="p">[...</span><span class="nx">values</span><span class="p">,</span> <span class="mf">0.0</span><span class="p">,</span> <span class="mf">1.0</span><span class="p">];</span>
                <span class="p">}</span>
                <span class="k">case</span> <span class="mi">3</span><span class="p">:</span> <span class="p">{</span>
                    <span class="k">return</span> <span class="p">[...</span><span class="nx">values</span><span class="p">,</span> <span class="mf">0.0</span><span class="p">,</span> <span class="mf">0.0</span><span class="p">,</span> <span class="mf">1.0</span><span class="p">];</span>
                <span class="p">}</span>
                <span class="nl">default</span><span class="p">:</span> <span class="p">{</span>
                    <span class="k">return</span> <span class="p">[</span><span class="mf">0.0</span><span class="p">,</span> <span class="mf">0.0</span><span class="p">,</span> <span class="mf">0.0</span><span class="p">,</span> <span class="mf">1.0</span><span class="p">];</span>
                <span class="p">}</span>
            <span class="p">}</span>
        <span class="p">})</span>
        <span class="p">.</span><span class="nf">map</span><span class="p">(</span><span class="nx">values</span> <span class="o">=&gt;</span> <span class="nf">multiplyMatrixVector</span><span class="p">(</span><span class="nx">values</span><span class="p">,</span> <span class="nx">modelMatrix</span><span class="p">,</span> <span class="k">this</span><span class="p">.</span><span class="nx">positionSize</span> <span class="o">+</span> <span class="mi">1</span><span class="p">))</span> <span class="c1">//need homogenous coordinates for positions</span>
        <span class="p">.</span><span class="nf">toArray</span><span class="p">();</span>
    <span class="kd">const</span> <span class="nx">normalMatrix</span> <span class="o">=</span> <span class="nf">getTranspose</span><span class="p">(</span>
        <span class="nf">getInverse</span><span class="p">(</span>
            <span class="nf">trimMatrix</span><span class="p">(</span><span class="nx">modelMatrix</span><span class="p">,</span> <span class="p">[</span><span class="mi">4</span><span class="p">,</span><span class="mi">4</span><span class="p">],</span> <span class="p">[</span><span class="k">this</span><span class="p">.</span><span class="nx">normalSize</span><span class="p">,</span><span class="k">this</span><span class="p">.</span><span class="nx">normalSize</span><span class="p">]),</span> 
        <span class="p">[</span><span class="k">this</span><span class="p">.</span><span class="nx">normalSize</span><span class="p">,</span><span class="k">this</span><span class="p">.</span><span class="nx">normalSize</span><span class="p">]),</span> 
    <span class="p">[</span><span class="k">this</span><span class="p">.</span><span class="nx">normalSize</span><span class="p">,</span><span class="k">this</span><span class="p">.</span><span class="nx">normalSize</span><span class="p">]);</span>
    <span class="kd">const</span> <span class="nx">transformedNormals</span> <span class="o">=</span> <span class="nf">chunk</span><span class="p">(</span><span class="k">this</span><span class="p">.</span><span class="nx">normals</span><span class="p">,</span> <span class="k">this</span><span class="p">.</span><span class="nx">normalSize</span><span class="p">)</span>
        <span class="p">.</span><span class="nf">map</span><span class="p">(</span><span class="nx">values</span> <span class="o">=&gt;</span> <span class="nf">multiplyMatrixVector</span><span class="p">(</span><span class="nx">values</span><span class="p">,</span> <span class="nx">normalMatrix</span><span class="p">,</span> <span class="k">this</span><span class="p">.</span><span class="nx">normalSize</span><span class="p">))</span>
        <span class="p">.</span><span class="nf">map</span><span class="p">(</span><span class="nx">values</span> <span class="o">=&gt;</span> <span class="nf">normalizeVector</span><span class="p">(</span><span class="nx">values</span><span class="p">))</span>
        <span class="p">.</span><span class="nf">toArray</span><span class="p">();</span>
    <span class="c1">//collect</span>
    <span class="kd">const</span> <span class="nx">newPositionsBuffer</span> <span class="o">=</span> <span class="k">new</span> <span class="nc">Float32Array</span><span class="p">(</span><span class="k">this</span><span class="p">.</span><span class="nx">vertexLength</span> <span class="o">*</span> <span class="k">this</span><span class="p">.</span><span class="nx">positionSize</span><span class="p">);</span>
    <span class="k">for</span><span class="p">(</span><span class="kd">let</span> <span class="nx">i</span> <span class="o">=</span> <span class="mi">0</span><span class="p">;</span> <span class="nx">i</span> <span class="o">&lt;</span> <span class="nx">transformedPositions</span><span class="p">.</span><span class="nx">length</span><span class="p">;</span> <span class="nx">i</span><span class="o">++</span><span class="p">){</span>
        <span class="nx">newPositionsBuffer</span><span class="p">.</span><span class="nf">set</span><span class="p">(</span><span class="nx">transformedPositions</span><span class="p">[</span><span class="nx">i</span><span class="p">].</span><span class="nf">slice</span><span class="p">(</span><span class="mi">0</span><span class="p">,</span> <span class="k">this</span><span class="p">.</span><span class="nx">positionSize</span><span class="p">),</span> <span class="nx">i</span> <span class="o">*</span> <span class="k">this</span><span class="p">.</span><span class="nx">positionSize</span><span class="p">)</span>
    <span class="p">}</span>
    <span class="kd">const</span> <span class="nx">newNormalsBuffer</span> <span class="o">=</span> <span class="k">new</span> <span class="nc">Float32Array</span><span class="p">(</span><span class="k">this</span><span class="p">.</span><span class="nx">vertexLength</span> <span class="o">*</span> <span class="k">this</span><span class="p">.</span><span class="nx">normalSize</span><span class="p">);</span>
    <span class="k">for </span><span class="p">(</span><span class="kd">let</span> <span class="nx">i</span> <span class="o">=</span> <span class="mi">0</span><span class="p">;</span> <span class="nx">i</span> <span class="o">&lt;</span> <span class="nx">transformedNormals</span><span class="p">.</span><span class="nx">length</span><span class="p">;</span> <span class="nx">i</span><span class="o">++</span><span class="p">)</span> <span class="p">{</span>
        <span class="nx">newNormalsBuffer</span><span class="p">.</span><span class="nf">set</span><span class="p">(</span><span class="nx">transformedNormals</span><span class="p">[</span><span class="nx">i</span><span class="p">].</span><span class="nf">slice</span><span class="p">(</span><span class="mi">0</span><span class="p">,</span> <span class="k">this</span><span class="p">.</span><span class="nx">normalSize</span><span class="p">),</span> <span class="nx">i</span> <span class="o">*</span> <span class="k">this</span><span class="p">.</span><span class="nx">normalSize</span><span class="p">)</span>
    <span class="p">}</span>
    <span class="k">this</span><span class="p">.</span><span class="nx">positions</span> <span class="o">=</span> <span class="nx">newPositionsBuffer</span><span class="p">;</span>
    <span class="k">this</span><span class="p">.</span><span class="nx">normals</span> <span class="o">=</span> <span class="nx">newNormalsBuffer</span><span class="p">;</span>
    <span class="k">this</span><span class="p">.</span><span class="nf">resetTransforms</span><span class="p">();</span>
    <span class="k">return</span> <span class="k">this</span><span class="p">;</span>
<span class="p">}</span>
</code></pre>

</div>



<p>Before we are done we need to double check a few things that tripped me up:</p>

<ul>
<li>The <code>normalMatrix</code> should not be multiplied by the view matrix, it should be in world space (if everything was view space it would work but it's more work).</li>
<li>Packing a <code>mat3x3</code> is unintuitively the same as packing a <code>mat4x4</code> with zeros at the end of each column.  Each column needs padding.  This took ages to debug because it's yet another alignment rule.</li>
<li>Matrices are column major meaning they index like <code>M[col][row]</code>.  This is flipped 90 from how we do it in js so you need to double check it's what your thing.</li>
</ul>

<p>I also when ahead and started using pack struct to pack the matrices and renamed "uniforms" to "scene" because it sounded better.  After sorting all the alignment bugs we can finally have 2 lights that properly work.</p>

<p><a href="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fs2jom582n7oup82p3pzf.png" class="article-body-image-wrapper"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fs2jom582n7oup82p3pzf.png" alt="teapot on rug with red and green lights" width="800" height="450"></a></p>

<h2>
  
  
  Code
</h2>

<p><a href="https://github.com/ndesmic/geo/releases/tag/v0.5" rel="noopener noreferrer">https://github.com/ndesmic/geo/releases/tag/v0.5</a></p>

