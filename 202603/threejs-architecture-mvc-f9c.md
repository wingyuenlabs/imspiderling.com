---
Title: Three.js Architecture: MVC
Description: 
Author: Ivan Babkov
Date: 2026-03-18T21:59:07.000Z
Robots: noindex,nofollow
Template: index
---
<p>It's easy to start a <a href="https://github.com/mrdoob/three.js/" rel="noopener noreferrer">Three.js</a> project, but keeping it maintainable as features grow is a different story. Many developers find themselves with massive files where input, physics, and rendering are all mixed together. When your logic is this tangled, even small changes can break parts of the app that seem unrelated.</p>

<p>We often use structured patterns like <a href="https://en.wikipedia.org/wiki/Model%E2%80%93view%E2%80%93controller" rel="noopener noreferrer">MVC</a> or <a href="https://www.oreilly.com/library/view/clean-architecture-a/9780134494272/" rel="noopener noreferrer">Clean Architecture</a> in web and mobile apps, but for some reason, we forget them once we enter a 3D scene. Instead of clean separation, we default to global variables or deeply nested classes. In my <a href="https://dev.to/i_babkov/threejs-architecture-ecs-3fg2">previous article on ECS</a>, we explored how a data-driven approach can help. However, if you are more comfortable with traditional patterns, MVC is an excellent way to organize a Three.js codebase.</p>

<p>In this article, we'll build a Three.js application using the Model-View-Controller pattern. We'll separate our application state from the 3D visuals. This will keep the logic predictable, maintainable, and easier to test.</p>

<p>🔗 The Final Result: <a href="https://github.com/ibabkov/threejs-mvc-example" rel="noopener noreferrer">GitHub</a> • <a href="https://ibabkov.github.io/threejs-mvc-example/" rel="noopener noreferrer">Demo</a></p>




<h2>
  
  
  Three.js Architecture Series
</h2>

<ul>
<li><a href="https://dev.to/i_babkov/threejs-architecture-ecs-3fg2">Three.js Architecture: ECS</a></li>
<li><a href="https://dev.to/i_babkov/threejs-architecture-mvc-f9c/">Three.js Architecture: MVC</a></li>
<li><a href="http://ibabkov.com/blog" rel="noopener noreferrer">All Articles</a></li>
</ul>




<h2>
  
  
  The MVC Pattern
</h2>

<p>MVC splits the application into three roles. This separation keeps the app state independent from the 3D visuals.</p>

<p><a href="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Flz4ibjws4xdn2sqqct26.png" class="article-body-image-wrapper"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Flz4ibjws4xdn2sqqct26.png" alt="Diagram of MVC's interactions" width="500" height="550"></a></p>

<p>• <strong>Controllers</strong>: They process the logic. A Controller listens for user input and updates the Model. It doesn't touch the View directly. It just changes the data, which then triggers a View update.</p>

<p>• <strong>Models</strong>: They hold the data. A Model stores properties like position or color, but knows nothing about Three.js meshes. It maintains the state and notifies other parts when it changes.</p>

<p>• <strong>Views</strong>: They handle the visual side. A View creates Three.js objects (meshes, lights, or cameras) and adds them to the scene. It watches the Model and updates the 3D objects to match the current data.</p>

<p>This keeps the scene graph relatively clean. You don't have to hunt for logic buried inside 3D objects. You can check if your logic works by looking at Model values, even without a rendering loop. So it simplifies testing as well as maintenance.</p>




<h2>
  
  
  Bootstrapping our app
</h2>

<p>We start with a standard Three.js boilerplate, but instead of putting everything into one function, we split it across models, views, and a controller from the start. This gives us a rendering loop, a camera, some lights, and an environment texture. I won't go too deep into Three.js itself because we are here to learn about MVC.</p>

<h3>
  
  
  Controllers
</h3>

<p>First, we need a rendering loop. The <code>RenderController</code> takes a callback and calls it every frame using <code>requestAnimationFrame</code>. It just makes sure we keep rendering.<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight typescript"><code><span class="c1">// src/controllers/RenderController.ts</span>
<span class="k">export</span> <span class="kd">class</span> <span class="nc">RenderController</span> <span class="p">{</span>
  <span class="k">private</span> <span class="nx">onRender</span><span class="p">:</span> <span class="p">()</span> <span class="o">=&gt;</span> <span class="k">void</span><span class="p">;</span>

  <span class="nf">constructor</span><span class="p">(</span><span class="nx">onRender</span><span class="p">:</span> <span class="p">()</span> <span class="o">=&gt;</span> <span class="k">void</span><span class="p">)</span> <span class="p">{</span>
    <span class="k">this</span><span class="p">.</span><span class="nx">onRender</span> <span class="o">=</span> <span class="nx">onRender</span><span class="p">;</span>
  <span class="p">}</span>

  <span class="k">private</span> <span class="nx">update</span> <span class="o">=</span> <span class="p">()</span> <span class="o">=&gt;</span> <span class="p">{</span>
    <span class="k">this</span><span class="p">.</span><span class="nf">onRender</span><span class="p">();</span>
    <span class="nf">requestAnimationFrame</span><span class="p">(</span><span class="k">this</span><span class="p">.</span><span class="nx">update</span><span class="p">);</span>
  <span class="p">}</span>

  <span class="k">public</span> <span class="nf">start</span><span class="p">()</span> <span class="p">{</span>
    <span class="k">this</span><span class="p">.</span><span class="nf">update</span><span class="p">();</span>
  <span class="p">}</span>
<span class="p">}</span>

</code></pre>

</div>



<h3>
  
  
  Models
</h3>

<p>All of our models extend a common <code>BaseModel</code>. It uses Three.js's <code>EventDispatcher</code>, which gives us a way to notify other parts of the app when data changes.<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight typescript"><code><span class="c1">// src/models/BaseModel.ts</span>
<span class="k">import</span> <span class="p">{</span> <span class="nx">EventDispatcher</span> <span class="p">}</span> <span class="k">from</span> <span class="dl">'</span><span class="s1">three</span><span class="dl">'</span><span class="p">;</span>

<span class="k">export</span> <span class="kd">class</span> <span class="nc">BaseModel</span><span class="o">&lt;</span><span class="nx">TEventMap</span> <span class="kd">extends</span> <span class="p">{}</span> <span class="o">=</span> <span class="p">{}</span><span class="o">&gt;</span> <span class="kd">extends</span> <span class="nx">EventDispatcher</span><span class="o">&lt;</span><span class="nx">TEventMap</span><span class="o">&gt;</span> <span class="p">{</span>
  <span class="nf">constructor</span><span class="p">()</span> <span class="p">{</span> <span class="k">super</span><span class="p">();</span> <span class="p">}</span>
<span class="p">}</span>

</code></pre>

</div>



<p>The <code>SceneModel</code> keeps track of objects in the scene. It stores them in a private <code>_objects</code> map and exposes <code>add</code>, <code>get</code>, and <code>getAll</code> methods. We use a private map instead of a public array to prevent outside code from mutating the collection directly. This ensures that all modifications happen through controlled methods. Later, when we add events, the <code>add</code> method is where we'll notify the rest of the app that a new object appeared in the scene.<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight typescript"><code><span class="c1">// src/models/SceneModel.ts</span>
<span class="k">import</span> <span class="kd">type</span> <span class="p">{</span> <span class="nx">SceneObjectModel</span> <span class="p">}</span> <span class="k">from</span> <span class="dl">'</span><span class="s1">./SceneObjectModel</span><span class="dl">'</span><span class="p">;</span>
<span class="k">import</span> <span class="p">{</span> <span class="nx">BaseModel</span> <span class="p">}</span> <span class="k">from</span> <span class="dl">'</span><span class="s1">./BaseModel</span><span class="dl">'</span><span class="p">;</span>

<span class="k">export</span> <span class="kd">class</span> <span class="nc">SceneModel</span> <span class="kd">extends</span> <span class="nc">BaseModel</span> <span class="p">{</span>
  <span class="k">private</span> <span class="nx">_objects</span><span class="p">:</span> <span class="nb">Map</span><span class="o">&lt;</span><span class="kr">string</span><span class="p">,</span> <span class="nx">SceneObjectModel</span><span class="o">&gt;</span> <span class="o">=</span> <span class="k">new</span> <span class="nc">Map</span><span class="p">();</span>

  <span class="nf">constructor</span><span class="p">()</span> <span class="p">{</span> <span class="k">super</span><span class="p">();</span> <span class="p">}</span>

  <span class="nf">add</span><span class="p">(</span><span class="nx">object</span><span class="p">:</span> <span class="nx">SceneObjectModel</span><span class="p">,</span> <span class="nx">parent</span><span class="p">?:</span> <span class="nx">SceneObjectModel</span><span class="p">)</span> <span class="p">{</span>
    <span class="k">this</span><span class="p">.</span><span class="nx">_objects</span><span class="p">.</span><span class="nf">set</span><span class="p">(</span><span class="nx">object</span><span class="p">.</span><span class="nx">id</span><span class="p">,</span> <span class="nx">object</span><span class="p">);</span>
  <span class="p">}</span>

  <span class="nf">get</span><span class="p">(</span><span class="nx">id</span><span class="p">:</span> <span class="kr">string</span><span class="p">):</span> <span class="nx">SceneObjectModel</span> <span class="o">|</span> <span class="kc">undefined</span> <span class="p">{</span>
    <span class="k">return</span> <span class="k">this</span><span class="p">.</span><span class="nx">_objects</span><span class="p">.</span><span class="nf">get</span><span class="p">(</span><span class="nx">id</span><span class="p">);</span>
  <span class="p">}</span>

  <span class="nf">getAll</span><span class="p">():</span> <span class="nx">SceneObjectModel</span><span class="p">[]</span> <span class="p">{</span>
    <span class="k">return</span> <span class="nb">Array</span><span class="p">.</span><span class="k">from</span><span class="p">(</span><span class="k">this</span><span class="p">.</span><span class="nx">_objects</span><span class="p">.</span><span class="nf">values</span><span class="p">());</span>
  <span class="p">}</span>
<span class="p">}</span>

</code></pre>

</div>



<p>Now we need models for the camera, lights, and environment. Same idea as <code>_objects</code> above: private fields, public getters. No outside code can silently overwrite a camera's FOV or a light's color. When we need to change those values later, we'll add setter methods that validate input and dispatch change events. For now, the getters are read-only, which is all our boilerplate needs.<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight typescript"><code><span class="c1">// src/models/CameraModel.ts</span>
<span class="k">import</span> <span class="p">{</span> <span class="nx">Vector3</span> <span class="p">}</span> <span class="k">from</span> <span class="dl">'</span><span class="s1">three</span><span class="dl">'</span><span class="p">;</span>

<span class="k">import</span> <span class="p">{</span> <span class="nx">BaseModel</span> <span class="p">}</span> <span class="k">from</span> <span class="dl">'</span><span class="s1">./BaseModel</span><span class="dl">'</span><span class="p">;</span>

<span class="k">export</span> <span class="kd">class</span> <span class="nc">CameraModel</span> <span class="kd">extends</span> <span class="nc">BaseModel</span> <span class="p">{</span>
  <span class="k">private</span> <span class="nx">_position</span><span class="p">:</span> <span class="nx">Vector3</span>  <span class="o">=</span> <span class="k">new</span> <span class="nc">Vector3</span><span class="p">(</span><span class="mi">0</span><span class="p">,</span> <span class="mi">0</span><span class="p">,</span> <span class="mi">5</span><span class="p">);</span>
  <span class="k">private</span> <span class="nx">_fov</span><span class="p">:</span> <span class="kr">number</span> <span class="o">=</span> <span class="mi">70</span><span class="p">;</span>
  <span class="k">private</span> <span class="nx">_near</span><span class="p">:</span> <span class="kr">number</span> <span class="o">=</span> <span class="mf">0.1</span><span class="p">;</span>
  <span class="k">private</span> <span class="nx">_far</span><span class="p">:</span> <span class="kr">number</span> <span class="o">=</span> <span class="mi">10</span><span class="p">;</span>

  <span class="nf">constructor</span><span class="p">()</span> <span class="p">{</span> <span class="k">super</span><span class="p">();</span> <span class="p">}</span>

  <span class="k">public</span> <span class="kd">get</span> <span class="nf">position</span><span class="p">()</span> <span class="p">{</span> <span class="k">return</span> <span class="k">this</span><span class="p">.</span><span class="nx">_position</span><span class="p">;</span> <span class="p">}</span>

  <span class="k">public</span> <span class="kd">get</span> <span class="nf">fov</span><span class="p">()</span> <span class="p">{</span> <span class="k">return</span> <span class="k">this</span><span class="p">.</span><span class="nx">_fov</span><span class="p">;</span> <span class="p">}</span>

  <span class="k">public</span> <span class="kd">get</span> <span class="nf">near</span><span class="p">()</span> <span class="p">{</span> <span class="k">return</span> <span class="k">this</span><span class="p">.</span><span class="nx">_near</span><span class="p">;</span> <span class="p">}</span>

  <span class="k">public</span> <span class="kd">get</span> <span class="nf">far</span><span class="p">()</span> <span class="p">{</span> <span class="k">return</span> <span class="k">this</span><span class="p">.</span><span class="nx">_far</span><span class="p">;</span> <span class="p">}</span>
<span class="p">}</span>

</code></pre>

</div>





<div class="highlight js-code-highlight">
<pre class="highlight typescript"><code><span class="c1">// src/models/LightsModel.ts</span>
<span class="k">import</span> <span class="p">{</span> <span class="nx">Vector3</span> <span class="p">}</span> <span class="k">from</span> <span class="dl">'</span><span class="s1">three</span><span class="dl">'</span><span class="p">;</span>

<span class="k">import</span> <span class="p">{</span> <span class="nx">BaseModel</span> <span class="p">}</span> <span class="k">from</span> <span class="dl">'</span><span class="s1">./BaseModel</span><span class="dl">'</span><span class="p">;</span>

<span class="k">export</span> <span class="kd">class</span> <span class="nc">LightsModel</span> <span class="kd">extends</span> <span class="nc">BaseModel</span> <span class="p">{</span>
  <span class="k">private</span> <span class="nx">_ambientLightColor</span> <span class="o">=</span> <span class="dl">'</span><span class="s1">#ffffff</span><span class="dl">'</span><span class="p">;</span>
  <span class="k">private</span> <span class="nx">_ambientLightIntensity</span> <span class="o">=</span> <span class="mi">2</span><span class="p">;</span>
  <span class="k">private</span> <span class="nx">_directionalLightColor</span> <span class="o">=</span> <span class="dl">'</span><span class="s1">#ffffff</span><span class="dl">'</span><span class="p">;</span>
  <span class="k">private</span> <span class="nx">_directionalLightIntensity</span> <span class="o">=</span> <span class="mi">2</span><span class="p">;</span>
  <span class="k">private</span> <span class="nx">_directionalLightPosition</span> <span class="o">=</span> <span class="k">new</span> <span class="nc">Vector3</span><span class="p">(</span><span class="mi">2</span><span class="p">,</span> <span class="mi">2</span><span class="p">,</span> <span class="mi">2</span><span class="p">);</span>

  <span class="nf">constructor</span><span class="p">()</span> <span class="p">{</span> <span class="k">super</span><span class="p">();</span> <span class="p">}</span>

  <span class="k">public</span> <span class="kd">get</span> <span class="nf">ambientLightColor</span><span class="p">()</span> <span class="p">{</span>
      <span class="k">return</span> <span class="k">this</span><span class="p">.</span><span class="nx">_ambientLightColor</span><span class="p">;</span>
  <span class="p">}</span>

  <span class="k">public</span> <span class="kd">get</span> <span class="nf">ambientLightIntensity</span><span class="p">()</span> <span class="p">{</span>
      <span class="k">return</span> <span class="k">this</span><span class="p">.</span><span class="nx">_ambientLightIntensity</span><span class="p">;</span>
  <span class="p">}</span>

  <span class="k">public</span> <span class="kd">get</span> <span class="nf">directionalLightColor</span><span class="p">()</span> <span class="p">{</span>
      <span class="k">return</span> <span class="k">this</span><span class="p">.</span><span class="nx">_directionalLightColor</span><span class="p">;</span>
  <span class="p">}</span>

  <span class="k">public</span> <span class="kd">get</span> <span class="nf">directionalLightIntensity</span><span class="p">()</span> <span class="p">{</span>
      <span class="k">return</span> <span class="k">this</span><span class="p">.</span><span class="nx">_directionalLightIntensity</span><span class="p">;</span>
  <span class="p">}</span>

  <span class="k">public</span> <span class="kd">get</span> <span class="nf">directionalLightPosition</span><span class="p">()</span> <span class="p">{</span>
      <span class="k">return</span> <span class="k">this</span><span class="p">.</span><span class="nx">_directionalLightPosition</span><span class="p">;</span>
  <span class="p">}</span>
<span class="p">}</span>

</code></pre>

</div>





<div class="highlight js-code-highlight">
<pre class="highlight typescript"><code><span class="c1">// src/models/EnvironmentModel.ts</span>
<span class="k">import</span> <span class="p">{</span> <span class="nx">BaseModel</span> <span class="p">}</span> <span class="k">from</span> <span class="dl">'</span><span class="s1">./BaseModel</span><span class="dl">'</span><span class="p">;</span>

<span class="k">export</span> <span class="kd">class</span> <span class="nc">EnvironmentModel</span> <span class="kd">extends</span> <span class="nc">BaseModel</span> <span class="p">{</span>
    <span class="k">private</span> <span class="nx">_backgroundColor</span> <span class="o">=</span> <span class="dl">'</span><span class="s1">#0d1117</span><span class="dl">'</span><span class="p">;</span>
    <span class="k">private</span> <span class="nx">_textureURL</span> <span class="o">=</span> <span class="k">new</span> <span class="nc">URL</span><span class="p">(</span>
        <span class="dl">'</span><span class="s1">../assets/env_1k.hdr</span><span class="dl">'</span><span class="p">,</span>
        <span class="k">import</span><span class="p">.</span><span class="nx">meta</span><span class="p">.</span><span class="nx">url</span><span class="p">,</span>
    <span class="p">);</span>

    <span class="nf">constructor</span><span class="p">()</span> <span class="p">{</span> <span class="k">super</span><span class="p">();</span> <span class="p">}</span>

    <span class="k">public</span> <span class="kd">get</span> <span class="nf">backgroundColor</span><span class="p">()</span> <span class="p">{</span> <span class="k">return</span> <span class="k">this</span><span class="p">.</span><span class="nx">_backgroundColor</span><span class="p">;</span> <span class="p">}</span>

    <span class="k">public</span> <span class="kd">get</span> <span class="nf">textureURL</span><span class="p">()</span> <span class="p">{</span> <span class="k">return</span> <span class="k">this</span><span class="p">.</span><span class="nx">_textureURL</span><span class="p">.</span><span class="nf">toString</span><span class="p">();</span> <span class="p">}</span>
<span class="p">}</span>

</code></pre>

</div>



<p>Storing these values in models rather than hardcoding them in the Three.js setup allows us to update the application state at runtime. The views can then react to these changes automatically. The models act as the source of truth, and the views read from them.</p>

<h3>
  
  
  Views
</h3>

<p>Now we can create our first view to represent the data we store in the models. Wrapping the <code>PerspectiveCamera</code> in a <code>CameraView</code> keeps the Three.js rendering logic separate from our application data. <code>PerspectiveCamera</code> is a Three.js object that belongs to the rendering side. The <code>CameraModel</code> holds the data (FOV, position, clipping planes), while <code>CameraView</code> turns that data into an actual camera. This setup makes it easier to change how the camera is built or to react to model updates without searching through the whole project.<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight typescript"><code><span class="c1">// src/views/CameraView.ts</span>
<span class="k">import</span> <span class="p">{</span> <span class="nx">PerspectiveCamera</span> <span class="p">}</span> <span class="k">from</span> <span class="dl">'</span><span class="s1">three</span><span class="dl">'</span><span class="p">;</span>

<span class="k">import</span> <span class="kd">type</span> <span class="p">{</span> <span class="nx">CameraModel</span> <span class="p">}</span> <span class="k">from</span> <span class="dl">'</span><span class="s1">../models/CameraModel</span><span class="dl">'</span><span class="p">;</span>

<span class="k">export</span> <span class="kd">class</span> <span class="nc">CameraView</span> <span class="p">{</span>
    <span class="k">private</span> <span class="nx">_camera</span><span class="p">:</span> <span class="nx">PerspectiveCamera</span><span class="p">;</span>
    <span class="k">private</span> <span class="nx">_model</span><span class="p">:</span> <span class="nx">CameraModel</span>

    <span class="nf">constructor</span><span class="p">(</span><span class="nx">model</span><span class="p">:</span> <span class="nx">CameraModel</span><span class="p">)</span> <span class="p">{</span>
        <span class="k">this</span><span class="p">.</span><span class="nx">_model</span> <span class="o">=</span> <span class="nx">model</span><span class="p">;</span>
        <span class="k">this</span><span class="p">.</span><span class="nx">_camera</span> <span class="o">=</span> <span class="k">new</span> <span class="nc">PerspectiveCamera</span><span class="p">(</span>
            <span class="k">this</span><span class="p">.</span><span class="nx">_model</span><span class="p">.</span><span class="nx">fov</span><span class="p">,</span>
            <span class="nb">window</span><span class="p">.</span><span class="nx">innerWidth</span> <span class="o">/</span> <span class="nb">window</span><span class="p">.</span><span class="nx">innerHeight</span><span class="p">,</span>
            <span class="k">this</span><span class="p">.</span><span class="nx">_model</span><span class="p">.</span><span class="nx">near</span><span class="p">,</span>
            <span class="k">this</span><span class="p">.</span><span class="nx">_model</span><span class="p">.</span><span class="nx">far</span>
        <span class="p">);</span>

        <span class="k">this</span><span class="p">.</span><span class="nx">_camera</span><span class="p">.</span><span class="nx">position</span><span class="p">.</span><span class="nf">copy</span><span class="p">(</span><span class="k">this</span><span class="p">.</span><span class="nx">_model</span><span class="p">.</span><span class="nx">position</span><span class="p">);</span>
    <span class="p">}</span>

    <span class="k">public</span> <span class="nf">getCamera</span><span class="p">():</span> <span class="nx">PerspectiveCamera</span> <span class="p">{</span>
        <span class="k">return</span> <span class="k">this</span><span class="p">.</span><span class="nx">_camera</span><span class="p">;</span>
    <span class="p">}</span>
<span class="p">}</span>

</code></pre>

</div>



<p><code>SceneView</code> is where the actual 3D scene lives. It owns the <code>WebGLRenderer</code> and the Three.js <code>Scene</code>. The constructor reads from <code>LightsModel</code> and <code>EnvironmentModel</code> to set up lights and the HDR environment texture, and it keeps a reference to <code>CameraView</code> for rendering. It also appends the renderer's canvas to the DOM. The <code>update</code> method gets called every frame and tells the renderer to draw the scene from the camera's point of view.<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight typescript"><code><span class="c1">// src/views/SceneView.ts</span>
<span class="k">import</span> <span class="p">{</span>
    <span class="nx">Scene</span><span class="p">,</span>
    <span class="nx">WebGLRenderer</span><span class="p">,</span>
    <span class="nx">Color</span><span class="p">,</span>
    <span class="nx">AmbientLight</span><span class="p">,</span>
    <span class="nx">DirectionalLight</span><span class="p">,</span>
    <span class="nx">EquirectangularReflectionMapping</span><span class="p">,</span>
<span class="p">}</span> <span class="k">from</span> <span class="dl">'</span><span class="s1">three</span><span class="dl">'</span><span class="p">;</span>
<span class="k">import</span> <span class="p">{</span> <span class="nx">HDRLoader</span> <span class="p">}</span> <span class="k">from</span> <span class="dl">'</span><span class="s1">three/examples/jsm/loaders/HDRLoader</span><span class="dl">'</span><span class="p">;</span>

<span class="k">import</span> <span class="kd">type</span> <span class="p">{</span> <span class="nx">CameraView</span> <span class="p">}</span> <span class="k">from</span> <span class="dl">'</span><span class="s1">./CameraView</span><span class="dl">'</span><span class="p">;</span>
<span class="k">import</span> <span class="kd">type</span> <span class="p">{</span> <span class="nx">SceneModel</span> <span class="p">}</span> <span class="k">from</span> <span class="dl">'</span><span class="s1">../models/SceneModel</span><span class="dl">'</span><span class="p">;</span>
<span class="k">import</span> <span class="kd">type</span> <span class="p">{</span> <span class="nx">LightsModel</span> <span class="p">}</span> <span class="k">from</span> <span class="dl">'</span><span class="s1">../models/LightsModel</span><span class="dl">'</span><span class="p">;</span>
<span class="k">import</span> <span class="kd">type</span> <span class="p">{</span> <span class="nx">EnvironmentModel</span> <span class="p">}</span> <span class="k">from</span> <span class="dl">'</span><span class="s1">../models/EnvironmentModel</span><span class="dl">'</span><span class="p">;</span>

<span class="k">export</span> <span class="kd">class</span> <span class="nc">SceneView</span> <span class="p">{</span>
    <span class="nl">scene</span><span class="p">:</span> <span class="nx">Scene</span><span class="p">;</span>
    <span class="k">private</span> <span class="nx">renderer</span><span class="p">:</span> <span class="nx">WebGLRenderer</span><span class="p">;</span>
    <span class="k">private</span> <span class="nx">sceneModel</span><span class="p">:</span> <span class="nx">SceneModel</span><span class="p">;</span>
    <span class="k">private</span> <span class="nx">cameraView</span><span class="p">:</span> <span class="nx">CameraView</span><span class="p">;</span>

    <span class="nf">constructor</span><span class="p">(</span>
        <span class="nx">sceneModel</span><span class="p">:</span> <span class="nx">SceneModel</span><span class="p">,</span>
        <span class="nx">lightsModel</span><span class="p">:</span> <span class="nx">LightsModel</span><span class="p">,</span>
        <span class="nx">environmentModel</span><span class="p">:</span> <span class="nx">EnvironmentModel</span><span class="p">,</span>
        <span class="nx">cameraView</span><span class="p">:</span> <span class="nx">CameraView</span>
    <span class="p">)</span> <span class="p">{</span>
        <span class="k">this</span><span class="p">.</span><span class="nx">sceneModel</span> <span class="o">=</span> <span class="nx">sceneModel</span><span class="p">;</span>
        <span class="k">this</span><span class="p">.</span><span class="nx">scene</span> <span class="o">=</span> <span class="k">new</span> <span class="nc">Scene</span><span class="p">();</span>
        <span class="k">this</span><span class="p">.</span><span class="nx">cameraView</span> <span class="o">=</span> <span class="nx">cameraView</span><span class="p">;</span>
        <span class="k">this</span><span class="p">.</span><span class="nx">renderer</span> <span class="o">=</span> <span class="k">this</span><span class="p">.</span><span class="nf">setupRenderer</span><span class="p">();</span>
        <span class="k">this</span><span class="p">.</span><span class="nx">scene</span> <span class="o">=</span> <span class="k">this</span><span class="p">.</span><span class="nf">setupScene</span><span class="p">(</span><span class="nx">environmentModel</span><span class="p">);</span>
        <span class="k">this</span><span class="p">.</span><span class="nf">setupLights</span><span class="p">(</span><span class="nx">lightsModel</span><span class="p">);</span>
        <span class="k">this</span><span class="p">.</span><span class="nf">setupEnvironment</span><span class="p">(</span><span class="nx">environmentModel</span><span class="p">);</span>

        <span class="nb">document</span>
            <span class="p">.</span><span class="nf">querySelector</span><span class="p">(</span><span class="dl">'</span><span class="s1">#app</span><span class="dl">'</span><span class="p">)</span><span class="o">!</span>
            <span class="p">.</span><span class="nf">appendChild</span><span class="p">(</span><span class="k">this</span><span class="p">.</span><span class="nx">renderer</span><span class="p">.</span><span class="nx">domElement</span><span class="p">);</span>
    <span class="p">}</span>

    <span class="k">private</span> <span class="nf">setupRenderer</span><span class="p">():</span> <span class="nx">WebGLRenderer</span> <span class="p">{</span>
        <span class="kd">const</span> <span class="nx">renderer</span> <span class="o">=</span> <span class="k">new</span> <span class="nc">WebGLRenderer</span><span class="p">({</span>
            <span class="na">antialias</span><span class="p">:</span> <span class="kc">true</span><span class="p">,</span>
            <span class="na">powerPreference</span><span class="p">:</span> <span class="dl">'</span><span class="s1">high-performance</span><span class="dl">'</span>
        <span class="p">});</span>

        <span class="nx">renderer</span><span class="p">.</span><span class="nf">setSize</span><span class="p">(</span><span class="nb">window</span><span class="p">.</span><span class="nx">innerWidth</span><span class="p">,</span> <span class="nb">window</span><span class="p">.</span><span class="nx">innerHeight</span><span class="p">);</span>
        <span class="nx">renderer</span><span class="p">.</span><span class="nf">setPixelRatio</span><span class="p">(</span><span class="nb">Math</span><span class="p">.</span><span class="nf">min</span><span class="p">(</span><span class="nb">window</span><span class="p">.</span><span class="nx">devicePixelRatio</span><span class="p">,</span> <span class="mi">2</span><span class="p">));</span>

        <span class="k">return</span> <span class="nx">renderer</span><span class="p">;</span>
    <span class="p">}</span>

    <span class="k">private</span> <span class="nf">setupScene</span><span class="p">(</span><span class="nx">environmentModel</span><span class="p">:</span> <span class="nx">EnvironmentModel</span><span class="p">):</span> <span class="nx">Scene</span> <span class="p">{</span>
        <span class="kd">const</span> <span class="nx">scene</span> <span class="o">=</span> <span class="k">new</span> <span class="nc">Scene</span><span class="p">();</span>
        <span class="nx">scene</span><span class="p">.</span><span class="nx">background</span> <span class="o">=</span> <span class="k">new</span> <span class="nc">Color</span><span class="p">(</span><span class="nx">environmentModel</span><span class="p">.</span><span class="nx">backgroundColor</span><span class="p">);</span>

        <span class="k">return</span> <span class="nx">scene</span><span class="p">;</span>
    <span class="p">}</span>

    <span class="k">private</span> <span class="nf">setupLights</span><span class="p">(</span><span class="nx">lightsModel</span><span class="p">:</span> <span class="nx">LightsModel</span><span class="p">)</span> <span class="p">{</span>
        <span class="kd">const</span> <span class="nx">ambient</span> <span class="o">=</span> <span class="k">new</span> <span class="nc">AmbientLight</span><span class="p">(</span>
            <span class="nx">lightsModel</span><span class="p">.</span><span class="nx">ambientLightColor</span><span class="p">,</span>
            <span class="nx">lightsModel</span><span class="p">.</span><span class="nx">ambientLightIntensity</span><span class="p">,</span>
        <span class="p">);</span>
        <span class="k">this</span><span class="p">.</span><span class="nx">scene</span><span class="p">.</span><span class="nf">add</span><span class="p">(</span><span class="nx">ambient</span><span class="p">);</span>

        <span class="kd">const</span> <span class="nx">directional</span> <span class="o">=</span> <span class="k">new</span> <span class="nc">DirectionalLight</span><span class="p">(</span>
            <span class="nx">lightsModel</span><span class="p">.</span><span class="nx">directionalLightColor</span><span class="p">,</span>
            <span class="nx">lightsModel</span><span class="p">.</span><span class="nx">directionalLightIntensity</span><span class="p">,</span>
        <span class="p">);</span>
        <span class="nx">directional</span><span class="p">.</span><span class="nx">position</span><span class="p">.</span><span class="nf">copy</span><span class="p">(</span><span class="nx">lightsModel</span><span class="p">.</span><span class="nx">directionalLightPosition</span><span class="p">);</span>
        <span class="k">this</span><span class="p">.</span><span class="nx">scene</span><span class="p">.</span><span class="nf">add</span><span class="p">(</span><span class="nx">directional</span><span class="p">);</span>
    <span class="p">}</span>

    <span class="k">private</span> <span class="nf">setupEnvironment</span><span class="p">(</span><span class="nx">environmentModel</span><span class="p">:</span> <span class="nx">EnvironmentModel</span><span class="p">)</span> <span class="p">{</span>
        <span class="k">new</span> <span class="nc">HDRLoader</span><span class="p">().</span><span class="nf">load</span><span class="p">(</span><span class="nx">environmentModel</span><span class="p">.</span><span class="nx">textureURL</span><span class="p">,</span> <span class="p">(</span><span class="nx">texture</span><span class="p">)</span> <span class="o">=&gt;</span> <span class="p">{</span>
            <span class="nx">texture</span><span class="p">.</span><span class="nx">mapping</span> <span class="o">=</span> <span class="nx">EquirectangularReflectionMapping</span><span class="p">;</span>
            <span class="k">this</span><span class="p">.</span><span class="nx">scene</span><span class="p">.</span><span class="nx">environment</span> <span class="o">=</span> <span class="nx">texture</span><span class="p">;</span>
        <span class="p">});</span>
    <span class="p">}</span>

    <span class="k">public</span> <span class="nf">update</span><span class="p">()</span> <span class="p">{</span>
        <span class="k">this</span><span class="p">.</span><span class="nx">renderer</span><span class="p">.</span><span class="nf">render</span><span class="p">(</span><span class="k">this</span><span class="p">.</span><span class="nx">scene</span><span class="p">,</span> <span class="k">this</span><span class="p">.</span><span class="nx">cameraView</span><span class="p">.</span><span class="nf">getCamera</span><span class="p">());</span>
    <span class="p">}</span>

    <span class="k">public</span> <span class="kd">get</span> <span class="nf">domElement</span><span class="p">()</span> <span class="p">{</span> <span class="k">return</span> <span class="k">this</span><span class="p">.</span><span class="nx">renderer</span><span class="p">.</span><span class="nx">domElement</span><span class="p">;</span> <span class="p">}</span>
<span class="p">}</span>

</code></pre>

</div>



<p>Now the <code>Application</code> class in our entry point wires everything together. The <code>constructor</code> first creates all the models: <code>SceneModel</code>, <code>CameraModel</code>, <code>LightsModel</code>, <code>EnvironmentModel</code>. At this point, there are no Three.js meshes yet, just data. Then it creates the views. <code>CameraView</code> gets the <code>CameraModel</code> and builds a <code>PerspectiveCamera</code> from it. <code>SceneView</code> gets the rest and sets up the renderer, scene, lights, and environment. Last, it creates the <code>RenderController</code> and passes it the <code>update</code> method as a per-frame callback. <code>update</code> calls <code>this.sceneView.update()</code>, which does a single render pass. <code>start</code> kicks off the loop through <code>renderController.start()</code>.<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight typescript"><code><span class="c1">// src/index.ts</span>
<span class="k">import</span> <span class="p">{</span> <span class="nx">SceneModel</span> <span class="p">}</span> <span class="k">from</span> <span class="dl">'</span><span class="s1">./models/SceneModel</span><span class="dl">'</span><span class="p">;</span>
<span class="k">import</span> <span class="p">{</span> <span class="nx">CameraModel</span> <span class="p">}</span> <span class="k">from</span> <span class="dl">'</span><span class="s1">./models/CameraModel</span><span class="dl">'</span><span class="p">;</span>
<span class="k">import</span> <span class="p">{</span> <span class="nx">LightsModel</span> <span class="p">}</span> <span class="k">from</span> <span class="dl">'</span><span class="s1">./models/LightsModel</span><span class="dl">'</span><span class="p">;</span>
<span class="k">import</span> <span class="p">{</span> <span class="nx">EnvironmentModel</span> <span class="p">}</span> <span class="k">from</span> <span class="dl">'</span><span class="s1">./models/EnvironmentModel</span><span class="dl">'</span><span class="p">;</span>

<span class="k">import</span> <span class="p">{</span> <span class="nx">SceneView</span> <span class="p">}</span> <span class="k">from</span> <span class="dl">'</span><span class="s1">./views/SceneView</span><span class="dl">'</span><span class="p">;</span>
<span class="k">import</span> <span class="p">{</span> <span class="nx">CameraView</span> <span class="p">}</span> <span class="k">from</span> <span class="dl">'</span><span class="s1">./views/CameraView</span><span class="dl">'</span><span class="p">;</span>

<span class="k">import</span> <span class="p">{</span> <span class="nx">RenderController</span> <span class="p">}</span> <span class="k">from</span> <span class="dl">'</span><span class="s1">./controllers/RenderController</span><span class="dl">'</span><span class="p">;</span>

<span class="k">export</span> <span class="kd">class</span> <span class="nc">Application</span> <span class="p">{</span>
    <span class="k">private</span> <span class="nx">sceneModel</span><span class="p">:</span> <span class="nx">SceneModel</span><span class="p">;</span>
    <span class="k">private</span> <span class="nx">cameraModel</span><span class="p">:</span> <span class="nx">CameraModel</span><span class="p">;</span>
    <span class="k">private</span> <span class="nx">lightsModel</span><span class="p">:</span> <span class="nx">LightsModel</span><span class="p">;</span>
    <span class="k">private</span> <span class="nx">environmentModel</span><span class="p">:</span> <span class="nx">EnvironmentModel</span><span class="p">;</span>

    <span class="k">private</span> <span class="nx">sceneView</span><span class="p">:</span> <span class="nx">SceneView</span><span class="p">;</span>
    <span class="k">private</span> <span class="nx">cameraView</span><span class="p">:</span> <span class="nx">CameraView</span><span class="p">;</span>

    <span class="k">private</span> <span class="nx">renderController</span><span class="p">:</span> <span class="nx">RenderController</span><span class="p">;</span>

    <span class="nf">constructor</span><span class="p">()</span> <span class="p">{</span>
        <span class="c1">// Create models</span>
        <span class="k">this</span><span class="p">.</span><span class="nx">sceneModel</span> <span class="o">=</span> <span class="k">new</span> <span class="nc">SceneModel</span><span class="p">();</span>
        <span class="k">this</span><span class="p">.</span><span class="nx">cameraModel</span> <span class="o">=</span> <span class="k">new</span> <span class="nc">CameraModel</span><span class="p">();</span>
        <span class="k">this</span><span class="p">.</span><span class="nx">lightsModel</span> <span class="o">=</span> <span class="k">new</span> <span class="nc">LightsModel</span><span class="p">();</span>
        <span class="k">this</span><span class="p">.</span><span class="nx">environmentModel</span> <span class="o">=</span> <span class="k">new</span> <span class="nc">EnvironmentModel</span><span class="p">();</span>

        <span class="c1">// Create views</span>
        <span class="k">this</span><span class="p">.</span><span class="nx">cameraView</span> <span class="o">=</span> <span class="k">new</span> <span class="nc">CameraView</span><span class="p">(</span><span class="k">this</span><span class="p">.</span><span class="nx">cameraModel</span><span class="p">);</span>
        <span class="k">this</span><span class="p">.</span><span class="nx">sceneView</span> <span class="o">=</span> <span class="k">new</span> <span class="nc">SceneView</span><span class="p">(</span>
            <span class="k">this</span><span class="p">.</span><span class="nx">sceneModel</span><span class="p">,</span>
            <span class="k">this</span><span class="p">.</span><span class="nx">lightsModel</span><span class="p">,</span>
            <span class="k">this</span><span class="p">.</span><span class="nx">environmentModel</span><span class="p">,</span>
            <span class="k">this</span><span class="p">.</span><span class="nx">cameraView</span><span class="p">,</span>
        <span class="p">);</span>

        <span class="c1">// Create controllers</span>
        <span class="k">this</span><span class="p">.</span><span class="nx">renderController</span> <span class="o">=</span> <span class="k">new</span> <span class="nc">RenderController</span><span class="p">(</span><span class="k">this</span><span class="p">.</span><span class="nx">update</span><span class="p">);</span>
    <span class="p">}</span>

    <span class="k">private</span> <span class="nx">update</span> <span class="o">=</span> <span class="p">()</span> <span class="o">=&gt;</span> <span class="p">{</span>
        <span class="k">this</span><span class="p">.</span><span class="nx">sceneView</span><span class="p">.</span><span class="nf">update</span><span class="p">();</span>
    <span class="p">}</span>

    <span class="k">public</span> <span class="nf">start</span><span class="p">()</span> <span class="p">{</span>
        <span class="k">this</span><span class="p">.</span><span class="nx">renderController</span><span class="p">.</span><span class="nf">start</span><span class="p">();</span>
    <span class="p">}</span>
<span class="p">}</span>

<span class="kd">const</span> <span class="nx">app</span> <span class="o">=</span> <span class="k">new</span> <span class="nc">Application</span><span class="p">();</span>

<span class="nx">app</span><span class="p">.</span><span class="nf">start</span><span class="p">();</span>

</code></pre>

</div>






<h2>
  
  
  Adding 3D objects
</h2>

<p>In a typical Three.js project, you'd create a mesh, set its properties, and add it to the scene all in one place. With MVC, we split that process: models define what the object is, and views turn that definition into something Three.js can render. More boilerplate? Yes. But once the project has dozens of objects, you'll be glad the data and the rendering code don't live in the same place.</p>

<h3>
  
  
  Events
</h3>

<p>Before we add any objects, we need a way for models to tell views that something happened. When a new object gets added to the scene model, the scene view needs to know about it so it can create the matching mesh. We'll define an event type for this. The <code>parent</code> field is optional because not every object has a parent in the hierarchy.<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight typescript"><code><span class="c1">// src/types/events.ts</span>
<span class="k">import</span> <span class="kd">type</span> <span class="p">{</span> <span class="nx">SceneObjectModel</span> <span class="p">}</span> <span class="k">from</span> <span class="dl">'</span><span class="s1">../models/SceneObjectModel</span><span class="dl">'</span><span class="p">;</span>

<span class="k">export</span> <span class="kd">type</span> <span class="nx">SceneModelEvents</span> <span class="o">=</span> <span class="p">{</span>
    <span class="dl">'</span><span class="s1">object-added</span><span class="dl">'</span><span class="p">:</span> <span class="p">{</span>
        <span class="na">object</span><span class="p">:</span> <span class="nx">SceneObjectModel</span><span class="p">,</span>
        <span class="nx">parent</span><span class="p">?:</span> <span class="nx">SceneObjectModel</span><span class="p">,</span>
    <span class="p">};</span>
<span class="p">}</span>

</code></pre>

</div>



<h3>
  
  
  Models
</h3>

<p>Every 3D object in our scene needs position, rotation, scale, and material data. Instead of duplicating these fields across every model, we put them in a <code>SceneObjectModel</code> base class. It takes an <code>id</code> string and an optional material config. The concrete models will extend it with their own geometry parameters.<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight typescript"><code><span class="c1">// src/models/SceneObjectModel.ts</span>
<span class="k">import</span> <span class="p">{</span> <span class="nx">Euler</span><span class="p">,</span> <span class="nx">Vector3</span> <span class="p">}</span> <span class="k">from</span> <span class="dl">'</span><span class="s1">three</span><span class="dl">'</span><span class="p">;</span>

<span class="k">import</span> <span class="p">{</span> <span class="nx">BaseModel</span> <span class="p">}</span> <span class="k">from</span> <span class="dl">'</span><span class="s1">./BaseModel</span><span class="dl">'</span><span class="p">;</span>
<span class="k">import</span> <span class="kd">type</span> <span class="p">{</span> <span class="nx">MapColorPropertiesToColorRepresentations</span> <span class="p">}</span> <span class="k">from</span> <span class="dl">'</span><span class="s1">three/src/materials/Material</span><span class="dl">'</span><span class="p">;</span>


<span class="k">export</span> <span class="kd">class</span> <span class="nc">SceneObjectModel</span><span class="o">&lt;</span><span class="nx">T</span> <span class="o">=</span> <span class="nb">Partial</span><span class="o">&lt;</span><span class="nx">MapColorPropertiesToColorRepresentations</span><span class="o">&lt;</span><span class="p">{}</span><span class="o">&gt;&gt;&gt;</span> <span class="kd">extends</span> <span class="nx">BaseModel</span> <span class="p">{</span>
    <span class="k">public</span> <span class="k">readonly</span> <span class="nx">id</span><span class="p">;</span>
    <span class="nl">_position</span><span class="p">:</span> <span class="nx">Vector3</span><span class="p">;</span>
    <span class="nl">_rotation</span><span class="p">:</span> <span class="nx">Euler</span><span class="p">;</span>
    <span class="nl">_scale</span><span class="p">:</span> <span class="nx">Vector3</span><span class="p">;</span>
    <span class="nl">_material</span><span class="p">:</span> <span class="nb">Partial</span><span class="o">&lt;</span><span class="nx">MapColorPropertiesToColorRepresentations</span><span class="o">&lt;</span><span class="nx">T</span><span class="o">&gt;&gt;</span><span class="p">;</span>

    <span class="nf">constructor</span><span class="p">(</span>
        <span class="nx">id</span><span class="p">:</span> <span class="kr">string</span><span class="p">,</span>
        <span class="nx">material</span><span class="p">:</span> <span class="nb">Partial</span><span class="o">&lt;</span><span class="nx">MapColorPropertiesToColorRepresentations</span><span class="o">&lt;</span><span class="nx">T</span><span class="o">&gt;&gt;</span> <span class="o">=</span> <span class="p">{}</span>
    <span class="p">)</span> <span class="p">{</span>
        <span class="k">super</span><span class="p">();</span>
        <span class="k">this</span><span class="p">.</span><span class="nx">id</span> <span class="o">=</span> <span class="nx">id</span><span class="p">;</span>
        <span class="k">this</span><span class="p">.</span><span class="nx">_position</span> <span class="o">=</span> <span class="k">new</span> <span class="nc">Vector3</span><span class="p">(</span><span class="mi">0</span><span class="p">,</span> <span class="mi">0</span><span class="p">,</span> <span class="mi">0</span><span class="p">);</span>
        <span class="k">this</span><span class="p">.</span><span class="nx">_rotation</span> <span class="o">=</span> <span class="k">new</span> <span class="nc">Euler</span><span class="p">(</span><span class="mi">0</span><span class="p">,</span> <span class="mi">0</span><span class="p">,</span> <span class="mi">0</span><span class="p">);</span>
        <span class="k">this</span><span class="p">.</span><span class="nx">_scale</span> <span class="o">=</span> <span class="k">new</span> <span class="nc">Vector3</span><span class="p">(</span><span class="mi">1</span><span class="p">,</span> <span class="mi">1</span><span class="p">,</span> <span class="mi">1</span><span class="p">);</span>
        <span class="k">this</span><span class="p">.</span><span class="nx">_material</span> <span class="o">=</span> <span class="p">{</span> <span class="p">...</span><span class="nx">material</span> <span class="p">};</span>
    <span class="p">}</span>

    <span class="k">public</span> <span class="kd">get</span> <span class="nf">position</span><span class="p">():</span> <span class="nx">Vector3</span> <span class="p">{</span> <span class="k">return</span> <span class="k">this</span><span class="p">.</span><span class="nx">_position</span><span class="p">;</span> <span class="p">}</span>

    <span class="k">public</span> <span class="kd">get</span> <span class="nf">rotation</span><span class="p">():</span> <span class="nx">Euler</span> <span class="p">{</span> <span class="k">return</span> <span class="k">this</span><span class="p">.</span><span class="nx">_rotation</span><span class="p">;</span> <span class="p">}</span>

    <span class="k">public</span> <span class="kd">get</span> <span class="nf">scale</span><span class="p">():</span> <span class="nx">Vector3</span> <span class="p">{</span> <span class="k">return</span> <span class="k">this</span><span class="p">.</span><span class="nx">_scale</span><span class="p">;</span> <span class="p">}</span>

    <span class="k">public</span> <span class="kd">get</span> <span class="nf">material</span><span class="p">():</span> <span class="nb">Partial</span><span class="o">&lt;</span><span class="nx">MapColorPropertiesToColorRepresentations</span><span class="o">&lt;</span><span class="nx">T</span><span class="o">&gt;&gt;</span> <span class="p">{</span>
        <span class="k">return</span> <span class="k">this</span><span class="p">.</span><span class="nx">_material</span><span class="p">;</span>
    <span class="p">}</span>
<span class="p">}</span>

</code></pre>

</div>



<p><code>CubeModel</code> extends <code>SceneObjectModel</code> with a <code>size</code> field and sets up a wireframe material. All the geometry and material data live here in the model, not in the view.<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight typescript"><code><span class="c1">// src/models/CubeModel.ts</span>
<span class="k">import</span> <span class="p">{</span><span class="nx">MathUtils</span><span class="p">,</span> <span class="nx">Vector3</span><span class="p">}</span> <span class="k">from</span> <span class="dl">'</span><span class="s1">three</span><span class="dl">'</span><span class="p">;</span>
<span class="k">import</span> <span class="kd">type</span> <span class="p">{</span> <span class="nx">MeshBasicMaterialProperties</span> <span class="p">}</span> <span class="k">from</span> <span class="dl">'</span><span class="s1">three</span><span class="dl">'</span><span class="p">;</span>

<span class="k">import</span> <span class="p">{</span> <span class="nx">SceneObjectModel</span> <span class="p">}</span> <span class="k">from</span> <span class="dl">'</span><span class="s1">./SceneObjectModel</span><span class="dl">'</span><span class="p">;</span>

<span class="k">export</span> <span class="kd">class</span> <span class="nc">CubeModel</span> <span class="kd">extends</span> <span class="nc">SceneObjectModel</span><span class="o">&lt;</span><span class="nx">MeshBasicMaterialProperties</span><span class="o">&gt;</span> <span class="p">{</span>
    <span class="k">private</span> <span class="nx">_size</span> <span class="o">=</span> <span class="mf">1.5</span><span class="p">;</span>

    <span class="nf">constructor</span><span class="p">(</span><span class="na">id</span><span class="p">:</span> <span class="kr">string</span><span class="p">)</span> <span class="p">{</span>
        <span class="k">super</span><span class="p">(</span><span class="nx">id</span><span class="p">,</span> <span class="p">{</span>
            <span class="na">color</span><span class="p">:</span> <span class="dl">'</span><span class="s1">#44aaff</span><span class="dl">'</span><span class="p">,</span>
            <span class="na">wireframe</span><span class="p">:</span> <span class="kc">true</span>
        <span class="p">});</span>

        <span class="k">this</span><span class="p">.</span><span class="nx">rotation</span><span class="p">.</span><span class="nx">y</span> <span class="o">=</span> <span class="nx">MathUtils</span><span class="p">.</span><span class="nf">degToRad</span><span class="p">(</span><span class="mi">45</span><span class="p">);</span>
    <span class="p">}</span>

    <span class="kd">get</span> <span class="nf">size</span><span class="p">()</span> <span class="p">{</span> <span class="k">return</span> <span class="k">this</span><span class="p">.</span><span class="nx">_size</span><span class="p">;</span> <span class="p">}</span>
<span class="p">}</span>

</code></pre>

</div>



<p><code>TorusModel</code> follows the same idea but with parameters specific to a torus knot geometry: radius, tube thickness, and segment counts. The material here is a <code>MeshStandardMaterial</code> with metalness and roughness, so we'll get reflections from the HDR environment we set up earlier.<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight typescript"><code><span class="c1">// src/models/TorusModel.ts</span>
<span class="k">import</span> <span class="kd">type</span> <span class="p">{</span> <span class="nx">MeshStandardMaterial</span> <span class="p">}</span> <span class="k">from</span> <span class="dl">'</span><span class="s1">three</span><span class="dl">'</span><span class="p">;</span>

<span class="k">import</span> <span class="p">{</span> <span class="nx">SceneObjectModel</span> <span class="p">}</span> <span class="k">from</span> <span class="dl">'</span><span class="s1">./SceneObjectModel</span><span class="dl">'</span><span class="p">;</span>

<span class="k">export</span> <span class="kd">class</span> <span class="nc">TorusModel</span> <span class="kd">extends</span> <span class="nc">SceneObjectModel</span><span class="o">&lt;</span><span class="nx">MeshStandardMaterial</span><span class="o">&gt;</span> <span class="p">{</span>
    <span class="k">private</span> <span class="na">_radius</span><span class="p">:</span> <span class="kr">number</span> <span class="o">=</span> <span class="mf">0.3</span><span class="p">;</span>
    <span class="k">private</span> <span class="na">_tube</span><span class="p">:</span> <span class="kr">number</span> <span class="o">=</span> <span class="mf">0.08</span><span class="p">;</span>
    <span class="k">private</span> <span class="na">_tubularSegments</span><span class="p">:</span> <span class="kr">number</span> <span class="o">=</span> <span class="mi">128</span><span class="p">;</span>
    <span class="k">private</span> <span class="na">_radialSegments</span><span class="p">:</span> <span class="kr">number</span> <span class="o">=</span> <span class="mi">16</span><span class="p">;</span>

    <span class="nf">constructor</span><span class="p">(</span><span class="na">id</span><span class="p">:</span> <span class="kr">string</span><span class="p">)</span> <span class="p">{</span>
        <span class="k">super</span><span class="p">(</span><span class="nx">id</span><span class="p">,</span> <span class="p">{</span>
            <span class="na">color</span><span class="p">:</span> <span class="dl">'</span><span class="s1">#4e8ab3</span><span class="dl">'</span><span class="p">,</span>
            <span class="na">metalness</span><span class="p">:</span> <span class="mi">1</span><span class="p">,</span>
            <span class="na">roughness</span><span class="p">:</span> <span class="mf">0.1</span>
        <span class="p">});</span>
    <span class="p">}</span>

    <span class="k">public</span> <span class="kd">get</span> <span class="nf">radius</span><span class="p">()</span> <span class="p">{</span> <span class="k">return</span> <span class="k">this</span><span class="p">.</span><span class="nx">_radius</span><span class="p">;</span> <span class="p">}</span>

    <span class="k">public</span> <span class="kd">get</span> <span class="nf">tube</span><span class="p">()</span> <span class="p">{</span> <span class="k">return</span> <span class="k">this</span><span class="p">.</span><span class="nx">_tube</span><span class="p">;</span> <span class="p">}</span>

    <span class="k">public</span> <span class="kd">get</span> <span class="nf">tubularSegments</span><span class="p">()</span> <span class="p">{</span> <span class="k">return</span> <span class="k">this</span><span class="p">.</span><span class="nx">_tubularSegments</span><span class="p">;</span> <span class="p">}</span>

    <span class="k">public</span> <span class="kd">get</span> <span class="nf">radialSegments</span><span class="p">()</span> <span class="p">{</span> <span class="k">return</span> <span class="k">this</span><span class="p">.</span><span class="nx">_radialSegments</span><span class="p">;</span> <span class="p">}</span>
<span class="p">}</span>

</code></pre>

</div>



<h3>
  
  
  Views
</h3>

<p>On the view side, we need the same kind of base class. <code>SceneObjectView</code> is abstract. It takes a model, calls <code>createMesh</code> (which subclasses implement), and syncs the mesh's transform to match the model's position, rotation, and scale. This way, every object view starts in the right place without repeating the same copy logic.<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight typescript"><code><span class="c1">// src/views/SceneObjectView.ts</span>
<span class="k">import</span> <span class="kd">type</span> <span class="p">{</span> <span class="nx">Mesh</span> <span class="p">}</span> <span class="k">from</span> <span class="dl">'</span><span class="s1">three</span><span class="dl">'</span><span class="p">;</span>

<span class="k">import</span> <span class="p">{</span> <span class="nx">SceneObjectModel</span> <span class="p">}</span> <span class="k">from</span> <span class="dl">'</span><span class="s1">../models/SceneObjectModel</span><span class="dl">'</span><span class="p">;</span>

<span class="k">export</span> <span class="kd">abstract</span> <span class="kd">class</span> <span class="nc">SceneObjectView</span><span class="o">&lt;</span><span class="nx">T</span> <span class="kd">extends</span> <span class="nx">SceneObjectModel</span> <span class="o">=</span> <span class="nx">SceneObjectModel</span><span class="o">&gt;</span> <span class="p">{</span>
    <span class="k">private</span> <span class="nx">_mesh</span><span class="o">!</span><span class="p">:</span> <span class="nx">Mesh</span><span class="p">;</span>
    <span class="k">private</span> <span class="na">_model</span><span class="p">:</span> <span class="nx">T</span><span class="p">;</span>

    <span class="nf">constructor</span><span class="p">(</span><span class="na">model</span><span class="p">:</span> <span class="nx">T</span><span class="p">)</span> <span class="p">{</span>
        <span class="k">this</span><span class="p">.</span><span class="nx">_model</span> <span class="o">=</span> <span class="nx">model</span><span class="p">;</span>
        <span class="k">this</span><span class="p">.</span><span class="nf">createMesh</span><span class="p">();</span>
        <span class="k">this</span><span class="p">.</span><span class="nf">syncTransform</span><span class="p">();</span>
    <span class="p">}</span>

    <span class="k">protected</span> <span class="kd">abstract</span> <span class="nf">createMesh</span><span class="p">():</span> <span class="k">void</span><span class="p">;</span>

    <span class="kd">get</span> <span class="nf">mesh</span><span class="p">():</span> <span class="nx">Mesh</span> <span class="p">{</span> <span class="k">return</span> <span class="k">this</span><span class="p">.</span><span class="nx">_mesh</span><span class="p">;</span> <span class="p">}</span>

    <span class="kd">set</span> <span class="nf">mesh</span><span class="p">(</span><span class="na">mesh</span><span class="p">:</span> <span class="nx">Mesh</span><span class="p">)</span> <span class="p">{</span> <span class="k">this</span><span class="p">.</span><span class="nx">_mesh</span> <span class="o">=</span> <span class="nx">mesh</span><span class="p">;</span> <span class="p">}</span>

    <span class="kd">get</span> <span class="nf">model</span><span class="p">():</span> <span class="nx">T</span> <span class="p">{</span> <span class="k">return</span> <span class="k">this</span><span class="p">.</span><span class="nx">_model</span><span class="p">;</span> <span class="p">}</span>

    <span class="k">private</span> <span class="nf">syncTransform</span><span class="p">()</span> <span class="p">{</span>
        <span class="k">this</span><span class="p">.</span><span class="nx">mesh</span><span class="p">.</span><span class="nx">position</span><span class="p">.</span><span class="nf">copy</span><span class="p">(</span><span class="k">this</span><span class="p">.</span><span class="nx">_model</span><span class="p">.</span><span class="nx">position</span><span class="p">);</span>
        <span class="k">this</span><span class="p">.</span><span class="nx">mesh</span><span class="p">.</span><span class="nx">rotation</span><span class="p">.</span><span class="nf">copy</span><span class="p">(</span><span class="k">this</span><span class="p">.</span><span class="nx">_model</span><span class="p">.</span><span class="nx">rotation</span><span class="p">);</span>
        <span class="k">this</span><span class="p">.</span><span class="nx">mesh</span><span class="p">.</span><span class="nx">scale</span><span class="p">.</span><span class="nf">copy</span><span class="p">(</span><span class="k">this</span><span class="p">.</span><span class="nx">_model</span><span class="p">.</span><span class="nx">scale</span><span class="p">);</span>
    <span class="p">}</span>
<span class="p">}</span>

</code></pre>

</div>



<p><code>CubeView</code> implements <code>createMesh</code> by reading the model's size and material properties to build a <code>BoxGeometry</code> and <code>MeshBasicMaterial</code>. The view doesn't decide what color the cube is or how big it should be. It just reads from the model.<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight typescript"><code><span class="c1">// src/views/CubeView.ts</span>
<span class="k">import</span> <span class="p">{</span> <span class="nx">BoxGeometry</span><span class="p">,</span> <span class="nx">MeshBasicMaterial</span><span class="p">,</span> <span class="nx">Mesh</span> <span class="p">}</span> <span class="k">from</span> <span class="dl">'</span><span class="s1">three</span><span class="dl">'</span><span class="p">;</span>

<span class="k">import</span> <span class="p">{</span> <span class="nx">SceneObjectView</span> <span class="p">}</span> <span class="k">from</span> <span class="dl">'</span><span class="s1">./SceneObjectView</span><span class="dl">'</span><span class="p">;</span>
<span class="k">import</span> <span class="p">{</span> <span class="nx">CubeModel</span> <span class="p">}</span> <span class="k">from</span> <span class="dl">'</span><span class="s1">../models/CubeModel</span><span class="dl">'</span><span class="p">;</span>

<span class="k">export</span> <span class="kd">class</span> <span class="nc">CubeView</span> <span class="kd">extends</span> <span class="nc">SceneObjectView</span><span class="o">&lt;</span><span class="nx">CubeModel</span><span class="o">&gt;</span> <span class="p">{</span>
    <span class="nf">constructor</span><span class="p">(</span><span class="na">model</span><span class="p">:</span> <span class="nx">CubeModel</span><span class="p">)</span> <span class="p">{</span>
        <span class="k">super</span><span class="p">(</span><span class="nx">model</span><span class="p">);</span>
    <span class="p">}</span>

    <span class="k">protected</span> <span class="nf">createMesh</span><span class="p">()</span> <span class="p">{</span>
        <span class="kd">const</span> <span class="nx">geometry</span> <span class="o">=</span> <span class="k">new</span> <span class="nc">BoxGeometry</span><span class="p">(</span>
            <span class="k">this</span><span class="p">.</span><span class="nx">model</span><span class="p">.</span><span class="nx">size</span><span class="p">,</span>
            <span class="k">this</span><span class="p">.</span><span class="nx">model</span><span class="p">.</span><span class="nx">size</span><span class="p">,</span>
            <span class="k">this</span><span class="p">.</span><span class="nx">model</span><span class="p">.</span><span class="nx">size</span><span class="p">,</span>
        <span class="p">);</span>
        <span class="kd">const</span> <span class="nx">material</span> <span class="o">=</span> <span class="k">new</span> <span class="nc">MeshBasicMaterial</span><span class="p">({</span>
            <span class="na">color</span><span class="p">:</span> <span class="k">this</span><span class="p">.</span><span class="nx">model</span><span class="p">.</span><span class="nx">material</span><span class="p">.</span><span class="nx">color</span><span class="p">,</span>
            <span class="na">wireframe</span><span class="p">:</span> <span class="k">this</span><span class="p">.</span><span class="nx">model</span><span class="p">.</span><span class="nx">material</span><span class="p">.</span><span class="nx">wireframe</span><span class="p">,</span>
        <span class="p">});</span>
        <span class="k">this</span><span class="p">.</span><span class="nx">mesh</span> <span class="o">=</span> <span class="k">new</span> <span class="nc">Mesh</span><span class="p">(</span><span class="nx">geometry</span><span class="p">,</span> <span class="nx">material</span><span class="p">);</span>
    <span class="p">}</span>
<span class="p">}</span>

</code></pre>

</div>



<p><code>TorusView</code> does the same thing for a torus knot. Different geometry, different material type, but the same pattern: read from the model, build the mesh.<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight typescript"><code><span class="c1">// src/views/TorusView.ts</span>
<span class="k">import</span> <span class="p">{</span> <span class="nx">TorusKnotGeometry</span><span class="p">,</span> <span class="nx">MeshStandardMaterial</span><span class="p">,</span> <span class="nx">Mesh</span> <span class="p">}</span> <span class="k">from</span> <span class="dl">'</span><span class="s1">three</span><span class="dl">'</span><span class="p">;</span>

<span class="k">import</span> <span class="p">{</span> <span class="nx">SceneObjectView</span> <span class="p">}</span> <span class="k">from</span> <span class="dl">'</span><span class="s1">./SceneObjectView</span><span class="dl">'</span><span class="p">;</span>
<span class="k">import</span> <span class="p">{</span> <span class="nx">TorusModel</span> <span class="p">}</span> <span class="k">from</span> <span class="dl">'</span><span class="s1">../models/TorusModel</span><span class="dl">'</span><span class="p">;</span>

<span class="k">export</span> <span class="kd">class</span> <span class="nc">TorusView</span> <span class="kd">extends</span> <span class="nc">SceneObjectView</span><span class="o">&lt;</span><span class="nx">TorusModel</span><span class="o">&gt;</span> <span class="p">{</span>
    <span class="nf">constructor</span><span class="p">(</span><span class="na">model</span><span class="p">:</span> <span class="nx">TorusModel</span><span class="p">)</span> <span class="p">{</span>
        <span class="k">super</span><span class="p">(</span><span class="nx">model</span><span class="p">);</span>
    <span class="p">}</span>

    <span class="k">protected</span> <span class="nf">createMesh</span><span class="p">()</span> <span class="p">{</span>
        <span class="kd">const</span> <span class="nx">geometry</span> <span class="o">=</span> <span class="k">new</span> <span class="nc">TorusKnotGeometry</span><span class="p">(</span>
            <span class="k">this</span><span class="p">.</span><span class="nx">model</span><span class="p">.</span><span class="nx">radius</span><span class="p">,</span>
            <span class="k">this</span><span class="p">.</span><span class="nx">model</span><span class="p">.</span><span class="nx">tube</span><span class="p">,</span>
            <span class="k">this</span><span class="p">.</span><span class="nx">model</span><span class="p">.</span><span class="nx">tubularSegments</span><span class="p">,</span>
            <span class="k">this</span><span class="p">.</span><span class="nx">model</span><span class="p">.</span><span class="nx">radialSegments</span><span class="p">,</span>
        <span class="p">);</span>
        <span class="kd">const</span> <span class="nx">material</span> <span class="o">=</span> <span class="k">new</span> <span class="nc">MeshStandardMaterial</span><span class="p">({</span>
            <span class="na">color</span><span class="p">:</span> <span class="k">this</span><span class="p">.</span><span class="nx">model</span><span class="p">.</span><span class="nx">material</span><span class="p">.</span><span class="nx">color</span><span class="p">,</span>
            <span class="na">metalness</span><span class="p">:</span> <span class="k">this</span><span class="p">.</span><span class="nx">model</span><span class="p">.</span><span class="nx">material</span><span class="p">.</span><span class="nx">metalness</span><span class="p">,</span>
            <span class="na">roughness</span><span class="p">:</span> <span class="k">this</span><span class="p">.</span><span class="nx">model</span><span class="p">.</span><span class="nx">material</span><span class="p">.</span><span class="nx">roughness</span>
        <span class="p">});</span>
        <span class="k">this</span><span class="p">.</span><span class="nx">mesh</span> <span class="o">=</span> <span class="k">new</span> <span class="nc">Mesh</span><span class="p">(</span><span class="nx">geometry</span><span class="p">,</span> <span class="nx">material</span><span class="p">);</span>
    <span class="p">}</span>
<span class="p">}</span>

</code></pre>

</div>



<h3>
  
  
  Wiring it up
</h3>

<p>Now we need to connect these pieces. First, <code>SceneModel</code> gets updated to dispatch an <code>'object-added'</code> event whenever a new object is registered. This is the bridge between models and views: the model doesn't know about views at all, it just announces that something changed.<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight typescript"><code><span class="c1">// src/models/SceneModel.ts</span>
<span class="k">import</span> <span class="kd">type</span> <span class="p">{</span> <span class="nx">SceneObjectModel</span> <span class="p">}</span> <span class="k">from</span> <span class="dl">'</span><span class="s1">./SceneObjectModel</span><span class="dl">'</span><span class="p">;</span>
<span class="k">import</span> <span class="p">{</span> <span class="nx">BaseModel</span> <span class="p">}</span> <span class="k">from</span> <span class="dl">'</span><span class="s1">./BaseModel</span><span class="dl">'</span><span class="p">;</span>
<span class="k">import</span> <span class="kd">type</span> <span class="p">{</span> <span class="nx">SceneModelEvents</span> <span class="p">}</span> <span class="k">from</span> <span class="dl">'</span><span class="s1">../types/events</span><span class="dl">'</span><span class="p">;</span>

<span class="k">export</span> <span class="kd">class</span> <span class="nc">SceneModel</span> <span class="kd">extends</span> <span class="nc">BaseModel</span><span class="o">&lt;</span><span class="nx">SceneModelEvents</span><span class="o">&gt;</span> <span class="p">{</span>
    <span class="c1">// ... _objects map, get, and getAll remain the same</span>

    <span class="nf">add</span><span class="p">(</span><span class="na">object</span><span class="p">:</span> <span class="nx">SceneObjectModel</span><span class="p">,</span> <span class="nx">parent</span><span class="p">?:</span> <span class="nx">SceneObjectModel</span><span class="p">)</span> <span class="p">{</span>
        <span class="k">this</span><span class="p">.</span><span class="nx">_objects</span><span class="p">.</span><span class="nf">set</span><span class="p">(</span><span class="nx">object</span><span class="p">.</span><span class="nx">id</span><span class="p">,</span> <span class="nx">object</span><span class="p">);</span>
        <span class="k">this</span><span class="p">.</span><span class="nf">dispatchEvent</span><span class="p">({</span> <span class="na">type</span><span class="p">:</span> <span class="dl">'</span><span class="s1">object-added</span><span class="dl">'</span><span class="p">,</span> <span class="nx">object</span><span class="p">,</span> <span class="nx">parent</span> <span class="p">});</span>
    <span class="p">}</span>
<span class="p">}</span>

</code></pre>

</div>



<p><code>SceneView</code> listens for that event. When it fires, the view checks what kind of model it got and creates the right view class for it. It also handles parent-child hierarchy: if the object has a parent, the mesh gets added to the parent's mesh instead of the scene root. The <code>objectViews</code> map keeps track of all created views so we can look up parents.<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight typescript"><code><span class="c1">// src/views/SceneView.ts</span>
<span class="c1">// ... existing three.js imports remain the same</span>
<span class="k">import</span> <span class="p">{</span> <span class="nx">CubeView</span> <span class="p">}</span> <span class="k">from</span> <span class="dl">'</span><span class="s1">./CubeView</span><span class="dl">'</span><span class="p">;</span>
<span class="k">import</span> <span class="p">{</span> <span class="nx">TorusView</span> <span class="p">}</span> <span class="k">from</span> <span class="dl">'</span><span class="s1">./TorusView</span><span class="dl">'</span><span class="p">;</span>
<span class="k">import</span> <span class="kd">type</span> <span class="p">{</span> <span class="nx">CameraView</span> <span class="p">}</span> <span class="k">from</span> <span class="dl">'</span><span class="s1">./CameraView</span><span class="dl">'</span><span class="p">;</span>
<span class="k">import</span> <span class="kd">type</span> <span class="p">{</span> <span class="nx">SceneObjectView</span> <span class="p">}</span> <span class="k">from</span> <span class="dl">'</span><span class="s1">./SceneObjectView</span><span class="dl">'</span><span class="p">;</span>
<span class="k">import</span> <span class="kd">type</span> <span class="p">{</span> <span class="nx">SceneModel</span> <span class="p">}</span> <span class="k">from</span> <span class="dl">'</span><span class="s1">../models/SceneModel</span><span class="dl">'</span><span class="p">;</span>
<span class="k">import</span> <span class="kd">type</span> <span class="p">{</span> <span class="nx">LightsModel</span> <span class="p">}</span> <span class="k">from</span> <span class="dl">'</span><span class="s1">../models/LightsModel</span><span class="dl">'</span><span class="p">;</span>
<span class="k">import</span> <span class="kd">type</span> <span class="p">{</span> <span class="nx">EnvironmentModel</span> <span class="p">}</span> <span class="k">from</span> <span class="dl">'</span><span class="s1">../models/EnvironmentModel</span><span class="dl">'</span><span class="p">;</span>
<span class="k">import</span> <span class="p">{</span> <span class="nx">CubeModel</span> <span class="p">}</span> <span class="k">from</span> <span class="dl">'</span><span class="s1">../models/CubeModel</span><span class="dl">'</span><span class="p">;</span>
<span class="k">import</span> <span class="p">{</span> <span class="nx">TorusModel</span> <span class="p">}</span> <span class="k">from</span> <span class="dl">'</span><span class="s1">../models/TorusModel</span><span class="dl">'</span><span class="p">;</span>

<span class="k">export</span> <span class="kd">class</span> <span class="nc">SceneView</span> <span class="p">{</span>
    <span class="c1">// ... scene, renderer, sceneModel, cameraView remain the same</span>
    <span class="k">private</span> <span class="nx">objectViews</span><span class="p">:</span> <span class="nb">Map</span><span class="o">&lt;</span><span class="kr">string</span><span class="p">,</span> <span class="nx">SceneObjectView</span><span class="o">&gt;</span> <span class="o">=</span> <span class="k">new</span> <span class="nc">Map</span><span class="p">();</span>

    <span class="nf">constructor</span><span class="p">(</span>
        <span class="nx">sceneModel</span><span class="p">:</span> <span class="nx">SceneModel</span><span class="p">,</span>
        <span class="nx">lightsModel</span><span class="p">:</span> <span class="nx">LightsModel</span><span class="p">,</span>
        <span class="nx">environmentModel</span><span class="p">:</span> <span class="nx">EnvironmentModel</span><span class="p">,</span>
        <span class="nx">cameraView</span><span class="p">:</span> <span class="nx">CameraView</span>
    <span class="p">)</span> <span class="p">{</span>
        <span class="c1">// ... existing setup remains the same</span>
        <span class="k">this</span><span class="p">.</span><span class="nf">addEventListeners</span><span class="p">();</span>
    <span class="p">}</span>

    <span class="c1">// ... setupRenderer, setupScene, setupLights,</span>
    <span class="c1">// setupEnvironment, update, domElement remain the same</span>

    <span class="k">private</span> <span class="nf">addEventListeners</span><span class="p">()</span> <span class="p">{</span>
        <span class="k">this</span><span class="p">.</span><span class="nx">sceneModel</span><span class="p">.</span><span class="nf">addEventListener</span><span class="p">(</span><span class="dl">'</span><span class="s1">object-added</span><span class="dl">'</span><span class="p">,</span> <span class="p">(</span><span class="nx">event</span><span class="p">)</span> <span class="o">=&gt;</span> <span class="p">{</span>
            <span class="kd">let</span> <span class="na">view</span><span class="p">:</span> <span class="nx">SceneObjectView</span><span class="p">;</span>

            <span class="k">if </span><span class="p">(</span><span class="nx">event</span><span class="p">.</span><span class="nx">object</span> <span class="k">instanceof</span> <span class="nx">CubeModel</span><span class="p">)</span> <span class="p">{</span>
                <span class="nx">view</span> <span class="o">=</span> <span class="k">new</span> <span class="nc">CubeView</span><span class="p">(</span><span class="nx">event</span><span class="p">.</span><span class="nx">object</span><span class="p">);</span>
            <span class="p">}</span> <span class="k">else</span> <span class="k">if </span><span class="p">(</span><span class="nx">event</span><span class="p">.</span><span class="nx">object</span> <span class="k">instanceof</span> <span class="nx">TorusModel</span><span class="p">)</span> <span class="p">{</span>
                <span class="nx">view</span> <span class="o">=</span> <span class="k">new</span> <span class="nc">TorusView</span><span class="p">(</span><span class="nx">event</span><span class="p">.</span><span class="nx">object</span><span class="p">);</span>
            <span class="p">}</span> <span class="k">else</span> <span class="p">{</span>
                <span class="k">return</span><span class="p">;</span>
            <span class="p">}</span>

            <span class="k">this</span><span class="p">.</span><span class="nx">objectViews</span><span class="p">.</span><span class="nf">set</span><span class="p">(</span><span class="nx">event</span><span class="p">.</span><span class="nx">object</span><span class="p">.</span><span class="nx">id</span><span class="p">,</span> <span class="nx">view</span><span class="p">);</span>
            <span class="kd">const</span> <span class="nx">parentView</span> <span class="o">=</span> <span class="nx">event</span><span class="p">.</span><span class="nx">parent</span>
                <span class="o">&amp;&amp;</span> <span class="k">this</span><span class="p">.</span><span class="nx">objectViews</span><span class="p">.</span><span class="nf">get</span><span class="p">(</span><span class="nx">event</span><span class="p">.</span><span class="nx">parent</span><span class="p">.</span><span class="nx">id</span><span class="p">);</span>
            <span class="kd">const</span> <span class="nx">parent</span> <span class="o">=</span> <span class="nx">parentView</span><span class="p">?.</span><span class="nx">mesh</span> <span class="o">||</span> <span class="k">this</span><span class="p">.</span><span class="nx">scene</span><span class="p">;</span>
            <span class="nx">parent</span><span class="p">.</span><span class="nf">add</span><span class="p">(</span><span class="nx">view</span><span class="p">.</span><span class="nx">mesh</span><span class="p">);</span>
        <span class="p">});</span>
    <span class="p">}</span>
<span class="p">}</span>

</code></pre>

</div>



<p>The last piece is spawning the actual objects. We add a <code>spawnInitialObjects</code> method to the <code>Application</code> class. It creates a <code>CubeModel</code> and a <code>TorusModel</code>, then adds them to the <code>SceneModel</code>. Because we pass <code>cube</code> as the parent of the torus, the torus mesh will be nested inside the cube mesh in the Three.js scene graph. The event system takes care of the rest: <code>SceneModel.add</code> fires <code>'object-added'</code>, <code>SceneView</code> picks it up, and the meshes appear.<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight typescript"><code><span class="c1">// src/index.ts</span>
<span class="c1">// ... existing imports remain the same</span>
<span class="k">import</span> <span class="p">{</span> <span class="nx">CubeModel</span> <span class="p">}</span> <span class="k">from</span> <span class="dl">'</span><span class="s1">./models/CubeModel</span><span class="dl">'</span><span class="p">;</span>
<span class="k">import</span> <span class="p">{</span> <span class="nx">TorusModel</span> <span class="p">}</span> <span class="k">from</span> <span class="dl">'</span><span class="s1">./models/TorusModel</span><span class="dl">'</span><span class="p">;</span>

<span class="k">export</span> <span class="kd">class</span> <span class="nc">Application</span> <span class="p">{</span>
    <span class="c1">// ... existing model, view, and controller fields remain the same</span>

    <span class="nf">constructor</span><span class="p">()</span> <span class="p">{</span>
        <span class="c1">// ... existing model, view,</span>
        <span class="c1">// and controller creation remains the same</span>
        <span class="k">this</span><span class="p">.</span><span class="nf">spawnInitialObjects</span><span class="p">();</span>
    <span class="p">}</span>

    <span class="c1">// ... update and start methods remain the same</span>

    <span class="k">private</span> <span class="nf">spawnInitialObjects</span><span class="p">()</span> <span class="p">{</span>
        <span class="kd">const</span> <span class="nx">cube</span> <span class="o">=</span> <span class="k">new</span> <span class="nc">CubeModel</span><span class="p">(</span><span class="dl">'</span><span class="s1">cube-1</span><span class="dl">'</span><span class="p">);</span>
        <span class="kd">const</span> <span class="nx">torus</span> <span class="o">=</span> <span class="k">new</span> <span class="nc">TorusModel</span><span class="p">(</span><span class="dl">'</span><span class="s1">torus-1</span><span class="dl">'</span><span class="p">);</span>

        <span class="k">this</span><span class="p">.</span><span class="nx">sceneModel</span><span class="p">.</span><span class="nf">add</span><span class="p">(</span><span class="nx">cube</span><span class="p">);</span>
        <span class="k">this</span><span class="p">.</span><span class="nx">sceneModel</span><span class="p">.</span><span class="nf">add</span><span class="p">(</span><span class="nx">torus</span><span class="p">,</span> <span class="nx">cube</span><span class="p">);</span>
    <span class="p">}</span>
<span class="p">}</span>

<span class="c1">// ... app creation and start remain the same</span>

</code></pre>

</div>



<p>At this point, the wireframe cube and the metallic torus knot are visible in the scene:<br>
<a href="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fy47tlfy11gf1dbw1b9lu.png" class="article-body-image-wrapper"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fy47tlfy11gf1dbw1b9lu.png" alt="Three.js Architecture: ECS / Basic 3D Visualization" width="800" height="600"></a></p>

<p>Data flows in one direction: models hold the state, views read from it. Neither side knows about the other's internals because events are the only thing connecting them.</p>


<h2>
  
  
  Bringing Interactivity
</h2>

<p>We have objects on screen, but they sit there. The whole point of splitting models and views is that we can change data at runtime and have the visuals follow. In MVC, user input doesn't touch the views directly. It goes through a controller, which updates a model, which fires an event, which the view picks up. That sounds like a long chain for "move the mouse and rotate a cube," but the rotation logic never imports anything from Three.js, and the rendering code never sees a pointer event.</p>
<h3>
  
  
  Events
</h3>

<p>We need two more event types. <code>SceneObjectEvents</code> carries a <code>rotation-changed</code> payload so views can react when a model's rotation updates. <code>InputModelEvents</code> carries normalized pointer coordinates whenever the user moves the mouse.<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight typescript"><code><span class="c1">// src/types/events.ts</span>
<span class="k">import</span> <span class="kd">type</span> <span class="p">{</span> <span class="nx">Vector2</span><span class="p">,</span> <span class="nx">Euler</span> <span class="p">}</span> <span class="k">from</span> <span class="dl">'</span><span class="s1">three</span><span class="dl">'</span><span class="p">;</span>

<span class="k">import</span> <span class="kd">type</span> <span class="p">{</span> <span class="nx">SceneObjectModel</span> <span class="p">}</span> <span class="k">from</span> <span class="dl">'</span><span class="s1">../models/SceneObjectModel</span><span class="dl">'</span><span class="p">;</span>

<span class="k">export</span> <span class="kd">type</span> <span class="nx">SceneObjectEvents</span> <span class="o">=</span> <span class="p">{</span>
    <span class="dl">'</span><span class="s1">rotation-changed</span><span class="dl">'</span><span class="p">:</span> <span class="p">{</span> <span class="na">rotation</span><span class="p">:</span> <span class="nx">Euler</span> <span class="p">};</span>
<span class="p">}</span>

<span class="k">export</span> <span class="kd">type</span> <span class="nx">InputModelEvents</span> <span class="o">=</span> <span class="p">{</span>
    <span class="dl">'</span><span class="s1">input-changed</span><span class="dl">'</span><span class="p">:</span> <span class="p">{</span> <span class="na">position</span><span class="p">:</span> <span class="nx">Vector2</span> <span class="p">};</span>
<span class="p">}</span>

<span class="c1">// ... SceneModelEvents remains the same</span>

</code></pre>

</div>



<h3>
  
  
  Models
</h3>

<p><code>SceneObjectModel</code> needs to use the new <code>SceneObjectEvents</code> type so it can dispatch <code>rotation-changed</code>. The only change from before is the generic parameter passed to <code>BaseModel</code>.<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight typescript"><code><span class="c1">// src/models/SceneObjectModel.ts</span>
<span class="c1">// ... existing imports remain the same</span>
<span class="k">import</span> <span class="kd">type</span> <span class="p">{</span> <span class="nx">SceneObjectEvents</span> <span class="p">}</span> <span class="k">from</span> <span class="dl">'</span><span class="s1">../types/events</span><span class="dl">'</span><span class="p">;</span>


<span class="k">export</span> <span class="kd">class</span> <span class="nc">SceneObjectModel</span><span class="o">&lt;</span><span class="nx">T</span> <span class="o">=</span> <span class="nb">Partial</span><span class="o">&lt;</span><span class="nx">MapColorPropertiesToColorRepresentations</span><span class="o">&lt;</span><span class="p">{}</span><span class="o">&gt;&gt;&gt;</span> <span class="kd">extends</span> <span class="nx">BaseModel</span><span class="o">&lt;</span><span class="nx">SceneObjectEvents</span><span class="o">&gt;</span> <span class="p">{</span>
    <span class="c1">// ... everything else remains the same</span>
<span class="p">}</span>

</code></pre>

</div>



<p><code>InputModel</code> is small. It stores a 2D position vector and exposes an <code>update</code> method. When called, it sets the new coordinates and fires <code>'input-changed'</code>. That's it. No DOM references and no knowledge of what will consume the event.<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight typescript"><code><span class="c1">// src/models/InputModel.ts</span>
<span class="k">import</span> <span class="p">{</span> <span class="nx">Vector2</span> <span class="p">}</span> <span class="k">from</span> <span class="dl">'</span><span class="s1">three</span><span class="dl">'</span><span class="p">;</span>

<span class="k">import</span> <span class="p">{</span> <span class="nx">BaseModel</span> <span class="p">}</span> <span class="k">from</span> <span class="dl">'</span><span class="s1">./BaseModel</span><span class="dl">'</span><span class="p">;</span>
<span class="k">import</span> <span class="p">{</span> <span class="nx">InputModelEvents</span> <span class="p">}</span> <span class="k">from</span> <span class="dl">'</span><span class="s1">../types/events</span><span class="dl">'</span><span class="p">;</span>

<span class="k">export</span> <span class="kd">class</span> <span class="nc">InputModel</span> <span class="kd">extends</span> <span class="nc">BaseModel</span><span class="o">&lt;</span><span class="nx">InputModelEvents</span><span class="o">&gt;</span> <span class="p">{</span>
    <span class="k">private</span> <span class="na">_position</span><span class="p">:</span> <span class="nx">Vector2</span><span class="p">;</span>

    <span class="nf">constructor</span><span class="p">()</span> <span class="p">{</span>
        <span class="k">super</span><span class="p">();</span>
        <span class="k">this</span><span class="p">.</span><span class="nx">_position</span> <span class="o">=</span> <span class="k">new</span> <span class="nc">Vector2</span><span class="p">(</span><span class="mi">0</span><span class="p">,</span> <span class="mi">0</span><span class="p">);</span>
    <span class="p">}</span>

    <span class="k">public</span> <span class="nf">update</span><span class="p">(</span><span class="na">x</span><span class="p">:</span> <span class="kr">number</span><span class="p">,</span> <span class="na">y</span><span class="p">:</span> <span class="kr">number</span><span class="p">)</span> <span class="p">{</span>
        <span class="k">this</span><span class="p">.</span><span class="nx">_position</span><span class="p">.</span><span class="nf">set</span><span class="p">(</span><span class="nx">x</span><span class="p">,</span> <span class="nx">y</span><span class="p">);</span>
        <span class="k">this</span><span class="p">.</span><span class="nf">dispatchEvent</span><span class="p">({</span>
            <span class="na">type</span><span class="p">:</span> <span class="dl">'</span><span class="s1">input-changed</span><span class="dl">'</span><span class="p">,</span>
            <span class="na">position</span><span class="p">:</span> <span class="k">this</span><span class="p">.</span><span class="nx">_position</span><span class="p">,</span>
        <span class="p">});</span>
    <span class="p">}</span>
<span class="p">}</span>

</code></pre>

</div>



<h3>
  
  
  Views
</h3>

<p><code>SceneObjectView</code> gets one new method: <code>addEventListeners</code>. It subscribes to <code>'rotation-changed'</code> on its model and copies the new rotation onto the mesh. The constructor now calls this method after <code>syncTransform</code>.<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight typescript"><code><span class="c1">// src/views/SceneObjectView.ts</span>
<span class="k">import</span> <span class="kd">type</span> <span class="p">{</span> <span class="nx">Mesh</span> <span class="p">}</span> <span class="k">from</span> <span class="dl">'</span><span class="s1">three</span><span class="dl">'</span><span class="p">;</span>

<span class="k">import</span> <span class="p">{</span> <span class="nx">SceneObjectModel</span> <span class="p">}</span> <span class="k">from</span> <span class="dl">'</span><span class="s1">../models/SceneObjectModel</span><span class="dl">'</span><span class="p">;</span>

<span class="k">export</span> <span class="kd">abstract</span> <span class="kd">class</span> <span class="nc">SceneObjectView</span><span class="o">&lt;</span><span class="nx">T</span> <span class="kd">extends</span> <span class="nx">SceneObjectModel</span> <span class="o">=</span> <span class="nx">SceneObjectModel</span><span class="o">&gt;</span> <span class="p">{</span>
    <span class="k">private</span> <span class="nx">_mesh</span><span class="o">!</span><span class="p">:</span> <span class="nx">Mesh</span><span class="p">;</span>
    <span class="k">private</span> <span class="na">_model</span><span class="p">:</span> <span class="nx">T</span><span class="p">;</span>

    <span class="nf">constructor</span><span class="p">(</span><span class="na">model</span><span class="p">:</span> <span class="nx">T</span><span class="p">)</span> <span class="p">{</span>
        <span class="k">this</span><span class="p">.</span><span class="nx">_model</span> <span class="o">=</span> <span class="nx">model</span><span class="p">;</span>
        <span class="k">this</span><span class="p">.</span><span class="nf">createMesh</span><span class="p">();</span>
        <span class="k">this</span><span class="p">.</span><span class="nf">syncTransform</span><span class="p">();</span>
        <span class="k">this</span><span class="p">.</span><span class="nf">addEventListeners</span><span class="p">();</span>
    <span class="p">}</span>

    <span class="k">protected</span> <span class="kd">abstract</span> <span class="nf">createMesh</span><span class="p">():</span> <span class="k">void</span><span class="p">;</span>

    <span class="kd">get</span> <span class="nf">mesh</span><span class="p">():</span> <span class="nx">Mesh</span> <span class="p">{</span> <span class="k">return</span> <span class="k">this</span><span class="p">.</span><span class="nx">_mesh</span><span class="p">;</span> <span class="p">}</span>

    <span class="kd">set</span> <span class="nf">mesh</span><span class="p">(</span><span class="na">mesh</span><span class="p">:</span> <span class="nx">Mesh</span><span class="p">)</span> <span class="p">{</span> <span class="k">this</span><span class="p">.</span><span class="nx">_mesh</span> <span class="o">=</span> <span class="nx">mesh</span><span class="p">;</span> <span class="p">}</span>

    <span class="kd">get</span> <span class="nf">model</span><span class="p">():</span> <span class="nx">T</span> <span class="p">{</span> <span class="k">return</span> <span class="k">this</span><span class="p">.</span><span class="nx">_model</span><span class="p">;</span> <span class="p">}</span>

    <span class="k">private</span> <span class="nf">syncTransform</span><span class="p">()</span> <span class="p">{</span>
        <span class="k">this</span><span class="p">.</span><span class="nx">mesh</span><span class="p">.</span><span class="nx">position</span><span class="p">.</span><span class="nf">copy</span><span class="p">(</span><span class="k">this</span><span class="p">.</span><span class="nx">_model</span><span class="p">.</span><span class="nx">position</span><span class="p">);</span>
        <span class="k">this</span><span class="p">.</span><span class="nx">mesh</span><span class="p">.</span><span class="nx">rotation</span><span class="p">.</span><span class="nf">copy</span><span class="p">(</span><span class="k">this</span><span class="p">.</span><span class="nx">_model</span><span class="p">.</span><span class="nx">rotation</span><span class="p">);</span>
        <span class="k">this</span><span class="p">.</span><span class="nx">mesh</span><span class="p">.</span><span class="nx">scale</span><span class="p">.</span><span class="nf">copy</span><span class="p">(</span><span class="k">this</span><span class="p">.</span><span class="nx">_model</span><span class="p">.</span><span class="nx">scale</span><span class="p">);</span>
    <span class="p">}</span>

    <span class="k">private</span> <span class="nf">addEventListeners</span><span class="p">()</span> <span class="p">{</span>
        <span class="k">this</span><span class="p">.</span><span class="nx">_model</span><span class="p">.</span><span class="nf">addEventListener</span><span class="p">(</span><span class="dl">'</span><span class="s1">rotation-changed</span><span class="dl">'</span><span class="p">,</span> <span class="p">(</span><span class="nx">event</span><span class="p">)</span> <span class="o">=&gt;</span> <span class="p">{</span>
            <span class="k">this</span><span class="p">.</span><span class="nx">mesh</span><span class="p">.</span><span class="nx">rotation</span><span class="p">.</span><span class="nf">set</span><span class="p">(</span>
                <span class="nx">event</span><span class="p">.</span><span class="nx">rotation</span><span class="p">.</span><span class="nx">x</span><span class="p">,</span> 
                <span class="nx">event</span><span class="p">.</span><span class="nx">rotation</span><span class="p">.</span><span class="nx">y</span><span class="p">,</span>
                <span class="nx">event</span><span class="p">.</span><span class="nx">rotation</span><span class="p">.</span><span class="nx">z</span><span class="p">,</span>
                <span class="nx">event</span><span class="p">.</span><span class="nx">rotation</span><span class="p">.</span><span class="nx">order</span><span class="p">,</span>
            <span class="p">);</span>
        <span class="p">});</span>
    <span class="p">}</span>
<span class="p">}</span>

</code></pre>

</div>



<h3>
  
  
  Controllers
</h3>

<p>Controllers sit between user input and models. They read events from one model and write changes to another. No rendering code, no DOM manipulation beyond the initial listener setup.</p>

<p><code>InputController</code> listens for <code>pointermove</code> and <code>pointerdown</code> on the renderer's canvas. When a pointer event fires, it normalizes the coordinates to a -1 to 1 range and calls <code>inputModel.update</code>. The controller doesn't care what happens after that.<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight typescript"><code><span class="c1">// src/controllers/InputController.ts</span>
<span class="k">import</span> <span class="kd">type</span> <span class="p">{</span> <span class="nx">InputModel</span> <span class="p">}</span> <span class="k">from</span> <span class="dl">'</span><span class="s1">../models/InputModel</span><span class="dl">'</span><span class="p">;</span>

<span class="k">export</span> <span class="kd">class</span> <span class="nc">InputController</span> <span class="p">{</span>
    <span class="k">private</span> <span class="nx">inputModel</span><span class="p">:</span> <span class="nx">InputModel</span><span class="p">;</span>
    <span class="k">private</span> <span class="nx">domElement</span><span class="p">:</span> <span class="nx">HTMLElement</span><span class="p">;</span>

    <span class="nf">constructor</span><span class="p">(</span><span class="nx">inputModel</span><span class="p">:</span> <span class="nx">InputModel</span><span class="p">,</span> <span class="nx">domElement</span><span class="p">:</span> <span class="nx">HTMLElement</span><span class="p">)</span> <span class="p">{</span>
        <span class="k">this</span><span class="p">.</span><span class="nx">inputModel</span> <span class="o">=</span> <span class="nx">inputModel</span><span class="p">;</span>
        <span class="k">this</span><span class="p">.</span><span class="nx">domElement</span> <span class="o">=</span> <span class="nx">domElement</span><span class="p">;</span>

        <span class="k">this</span><span class="p">.</span><span class="nf">setupListeners</span><span class="p">();</span>
    <span class="p">}</span>

    <span class="k">private</span> <span class="nx">handlePointer</span> <span class="o">=</span> <span class="p">(</span><span class="nx">e</span><span class="p">:</span> <span class="nx">PointerEvent</span><span class="p">)</span> <span class="o">=&gt;</span> <span class="p">{</span>
        <span class="k">if </span><span class="p">(</span>
            <span class="o">!</span><span class="nx">e</span><span class="p">.</span><span class="nx">isPrimary</span> <span class="o">||</span>
            <span class="o">!</span><span class="p">(</span><span class="nx">e</span><span class="p">.</span><span class="nx">currentTarget</span> <span class="k">instanceof</span> <span class="nx">HTMLElement</span><span class="p">)</span>
        <span class="p">)</span> <span class="k">return</span><span class="p">;</span>

        <span class="kd">const</span> <span class="nx">rect</span> <span class="o">=</span> <span class="nx">e</span><span class="p">.</span><span class="nx">currentTarget</span><span class="p">.</span><span class="nf">getBoundingClientRect</span><span class="p">();</span>
        <span class="kd">const</span> <span class="nx">x</span> <span class="o">=</span> <span class="p">((</span><span class="nx">e</span><span class="p">.</span><span class="nx">clientX</span> <span class="o">-</span> <span class="nx">rect</span><span class="p">.</span><span class="nx">left</span><span class="p">)</span> <span class="o">/</span> <span class="nx">rect</span><span class="p">.</span><span class="nx">width</span><span class="p">)</span> <span class="o">*</span> <span class="mi">2</span> <span class="o">-</span> <span class="mi">1</span><span class="p">;</span>
        <span class="kd">const</span> <span class="nx">y</span> <span class="o">=</span> <span class="o">-</span><span class="p">(((</span><span class="nx">e</span><span class="p">.</span><span class="nx">clientY</span> <span class="o">-</span> <span class="nx">rect</span><span class="p">.</span><span class="nx">top</span><span class="p">)</span> <span class="o">/</span> <span class="nx">rect</span><span class="p">.</span><span class="nx">height</span><span class="p">)</span> <span class="o">*</span> <span class="mi">2</span> <span class="o">-</span> <span class="mi">1</span><span class="p">);</span>

        <span class="k">this</span><span class="p">.</span><span class="nx">inputModel</span><span class="p">.</span><span class="nf">update</span><span class="p">(</span><span class="nx">x</span><span class="p">,</span> <span class="nx">y</span><span class="p">);</span>
    <span class="p">}</span>

    <span class="k">private</span> <span class="nf">setupListeners</span><span class="p">()</span> <span class="p">{</span>
        <span class="k">this</span><span class="p">.</span><span class="nx">domElement</span><span class="p">.</span><span class="nf">addEventListener</span><span class="p">(</span><span class="dl">'</span><span class="s1">pointermove</span><span class="dl">'</span><span class="p">,</span> <span class="k">this</span><span class="p">.</span><span class="nx">handlePointer</span><span class="p">);</span>
        <span class="k">this</span><span class="p">.</span><span class="nx">domElement</span><span class="p">.</span><span class="nf">addEventListener</span><span class="p">(</span><span class="dl">'</span><span class="s1">pointerdown</span><span class="dl">'</span><span class="p">,</span> <span class="k">this</span><span class="p">.</span><span class="nx">handlePointer</span><span class="p">);</span>
    <span class="p">}</span>
<span class="p">}</span>

</code></pre>

</div>



<p><code>TransformController</code> connects the input to the scene objects. It listens for <code>'input-changed'</code> on the <code>InputModel</code>, iterates over all objects in the <code>SceneModel</code>, and dispatches <code>'rotation-changed'</code> on each one. The cube rotates on the Y axis based on horizontal pointer position, and the torus rotates on the X axis based on vertical position. The controller decides the mapping, but the model owns the data, and the view owns the rendering.<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight typescript"><code><span class="c1">// src/controllers/TransformController.ts</span>
<span class="k">import</span> <span class="kd">type</span> <span class="p">{</span> <span class="nx">SceneModel</span> <span class="p">}</span> <span class="k">from</span> <span class="dl">'</span><span class="s1">../models/SceneModel</span><span class="dl">'</span><span class="p">;</span>
<span class="k">import</span> <span class="kd">type</span> <span class="p">{</span> <span class="nx">InputModel</span> <span class="p">}</span> <span class="k">from</span> <span class="dl">'</span><span class="s1">../models/InputModel</span><span class="dl">'</span><span class="p">;</span>
<span class="k">import</span> <span class="p">{</span> <span class="nx">CubeModel</span> <span class="p">}</span> <span class="k">from</span> <span class="dl">'</span><span class="s1">../models/CubeModel</span><span class="dl">'</span><span class="p">;</span>
<span class="k">import</span> <span class="p">{</span> <span class="nx">TorusModel</span> <span class="p">}</span> <span class="k">from</span> <span class="dl">'</span><span class="s1">../models/TorusModel</span><span class="dl">'</span><span class="p">;</span>

<span class="k">export</span> <span class="kd">class</span> <span class="nc">TransformController</span> <span class="p">{</span>
    <span class="k">private</span> <span class="nx">sceneModel</span><span class="p">:</span> <span class="nx">SceneModel</span><span class="p">;</span>
    <span class="k">private</span> <span class="nx">inputModel</span><span class="p">:</span> <span class="nx">InputModel</span><span class="p">;</span>

    <span class="nf">constructor</span><span class="p">(</span><span class="nx">sceneModel</span><span class="p">:</span> <span class="nx">SceneModel</span><span class="p">,</span> <span class="nx">inputModel</span><span class="p">:</span> <span class="nx">InputModel</span><span class="p">)</span> <span class="p">{</span>
        <span class="k">this</span><span class="p">.</span><span class="nx">sceneModel</span> <span class="o">=</span> <span class="nx">sceneModel</span><span class="p">;</span>
        <span class="k">this</span><span class="p">.</span><span class="nx">inputModel</span> <span class="o">=</span> <span class="nx">inputModel</span><span class="p">;</span>

        <span class="k">this</span><span class="p">.</span><span class="nf">setupListeners</span><span class="p">();</span>
    <span class="p">}</span>

    <span class="k">private</span> <span class="nf">setupListeners</span><span class="p">()</span> <span class="p">{</span>
        <span class="k">this</span><span class="p">.</span><span class="nx">inputModel</span><span class="p">.</span><span class="nf">addEventListener</span><span class="p">(</span>
            <span class="dl">'</span><span class="s1">input-changed</span><span class="dl">'</span><span class="p">,</span>
            <span class="p">({</span> <span class="nx">position</span> <span class="p">})</span> <span class="o">=&gt;</span> <span class="p">{</span>
                <span class="kd">const</span> <span class="nx">objects</span> <span class="o">=</span> <span class="k">this</span><span class="p">.</span><span class="nx">sceneModel</span><span class="p">.</span><span class="nf">getAll</span><span class="p">();</span>
                <span class="nx">objects</span><span class="p">.</span><span class="nf">forEach</span><span class="p">((</span><span class="nx">obj</span><span class="p">)</span> <span class="o">=&gt;</span> <span class="p">{</span>
                    <span class="kd">const</span> <span class="nx">rotation</span> <span class="o">=</span> <span class="nx">obj</span><span class="p">.</span><span class="nx">rotation</span><span class="p">;</span>
                    <span class="k">if </span><span class="p">(</span><span class="nx">obj</span> <span class="k">instanceof</span> <span class="nx">CubeModel</span><span class="p">)</span> <span class="p">{</span>
                        <span class="nx">rotation</span><span class="p">.</span><span class="nx">y</span> <span class="o">=</span> <span class="nx">position</span><span class="p">.</span><span class="nx">x</span> <span class="o">*</span> <span class="nb">Math</span><span class="p">.</span><span class="nx">PI</span><span class="p">;</span>
                        <span class="nx">obj</span><span class="p">.</span><span class="nf">dispatchEvent</span><span class="p">({</span>
                            <span class="na">type</span><span class="p">:</span> <span class="dl">'</span><span class="s1">rotation-changed</span><span class="dl">'</span><span class="p">,</span>
                            <span class="nx">rotation</span><span class="p">,</span>
                        <span class="p">});</span>
                    <span class="p">}</span> <span class="k">else</span> <span class="k">if </span><span class="p">(</span><span class="nx">obj</span> <span class="k">instanceof</span> <span class="nx">TorusModel</span><span class="p">)</span> <span class="p">{</span>
                        <span class="nx">rotation</span><span class="p">.</span><span class="nx">x</span> <span class="o">=</span> <span class="nx">position</span><span class="p">.</span><span class="nx">y</span> <span class="o">*</span> <span class="nb">Math</span><span class="p">.</span><span class="nx">PI</span><span class="p">;</span>
                        <span class="nx">obj</span><span class="p">.</span><span class="nf">dispatchEvent</span><span class="p">({</span>
                            <span class="na">type</span><span class="p">:</span> <span class="dl">'</span><span class="s1">rotation-changed</span><span class="dl">'</span><span class="p">,</span>
                            <span class="nx">rotation</span><span class="p">,</span>
                        <span class="p">});</span>
                    <span class="p">}</span>
                <span class="p">});</span>
            <span class="p">}</span>
        <span class="p">);</span>
    <span class="p">}</span>
<span class="p">}</span>

</code></pre>

</div>



<h3>
  
  
  Wiring it all together
</h3>

<p>The <code>Application</code> class now creates all three controllers and wires them to the right models and views. <code>InputController</code> needs the canvas element from <code>SceneView</code> for pointer events. <code>TransformController</code> sits between <code>SceneModel</code> and <code>InputModel</code>, translating pointer coordinates into rotation updates. <code>RenderController</code> still calls <code>SceneView.update()</code> every frame.<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight typescript"><code><span class="c1">// src/index.ts</span>
<span class="k">import</span> <span class="p">{</span> <span class="nx">SceneModel</span> <span class="p">}</span> <span class="k">from</span> <span class="dl">'</span><span class="s1">./models/SceneModel</span><span class="dl">'</span><span class="p">;</span>
<span class="k">import</span> <span class="p">{</span> <span class="nx">CubeModel</span> <span class="p">}</span> <span class="k">from</span> <span class="dl">'</span><span class="s1">./models/CubeModel</span><span class="dl">'</span><span class="p">;</span>
<span class="k">import</span> <span class="p">{</span> <span class="nx">TorusModel</span> <span class="p">}</span> <span class="k">from</span> <span class="dl">'</span><span class="s1">./models/TorusModel</span><span class="dl">'</span><span class="p">;</span>
<span class="k">import</span> <span class="p">{</span> <span class="nx">CameraModel</span> <span class="p">}</span> <span class="k">from</span> <span class="dl">'</span><span class="s1">./models/CameraModel</span><span class="dl">'</span><span class="p">;</span>
<span class="k">import</span> <span class="p">{</span> <span class="nx">InputModel</span> <span class="p">}</span> <span class="k">from</span> <span class="dl">'</span><span class="s1">./models/InputModel</span><span class="dl">'</span><span class="p">;</span>
<span class="k">import</span> <span class="p">{</span> <span class="nx">LightsModel</span> <span class="p">}</span> <span class="k">from</span> <span class="dl">'</span><span class="s1">./models/LightsModel</span><span class="dl">'</span><span class="p">;</span>

<span class="k">import</span> <span class="p">{</span> <span class="nx">SceneView</span> <span class="p">}</span> <span class="k">from</span> <span class="dl">'</span><span class="s1">./views/SceneView</span><span class="dl">'</span><span class="p">;</span>
<span class="k">import</span> <span class="p">{</span> <span class="nx">CameraView</span> <span class="p">}</span> <span class="k">from</span> <span class="dl">'</span><span class="s1">./views/CameraView</span><span class="dl">'</span><span class="p">;</span>

<span class="k">import</span> <span class="p">{</span> <span class="nx">InputController</span> <span class="p">}</span> <span class="k">from</span> <span class="dl">'</span><span class="s1">./controllers/InputController</span><span class="dl">'</span><span class="p">;</span>
<span class="k">import</span> <span class="p">{</span> <span class="nx">TransformController</span> <span class="p">}</span> <span class="k">from</span> <span class="dl">'</span><span class="s1">./controllers/TransformController</span><span class="dl">'</span><span class="p">;</span>
<span class="k">import</span> <span class="p">{</span> <span class="nx">RenderController</span> <span class="p">}</span> <span class="k">from</span> <span class="dl">'</span><span class="s1">./controllers/RenderController</span><span class="dl">'</span><span class="p">;</span>
<span class="k">import</span> <span class="p">{</span> <span class="nx">EnvironmentModel</span> <span class="p">}</span> <span class="k">from</span> <span class="dl">'</span><span class="s1">./models/EnvironmentModel</span><span class="dl">'</span><span class="p">;</span>


<span class="k">export</span> <span class="kd">class</span> <span class="nc">Application</span> <span class="p">{</span>
    <span class="k">private</span> <span class="nx">sceneModel</span><span class="p">:</span> <span class="nx">SceneModel</span><span class="p">;</span>
    <span class="k">private</span> <span class="nx">cameraModel</span><span class="p">:</span> <span class="nx">CameraModel</span><span class="p">;</span>
    <span class="k">private</span> <span class="nx">inputModel</span><span class="p">:</span> <span class="nx">InputModel</span><span class="p">;</span>
    <span class="k">private</span> <span class="nx">lightsModel</span><span class="p">:</span> <span class="nx">LightsModel</span><span class="p">;</span>
    <span class="k">private</span> <span class="nx">environmentModel</span><span class="p">:</span> <span class="nx">EnvironmentModel</span><span class="p">;</span>

    <span class="k">private</span> <span class="nx">sceneView</span><span class="p">:</span> <span class="nx">SceneView</span><span class="p">;</span>
    <span class="k">private</span> <span class="nx">cameraView</span><span class="p">:</span> <span class="nx">CameraView</span><span class="p">;</span>

    <span class="k">private</span> <span class="nx">inputController</span><span class="p">:</span> <span class="nx">InputController</span><span class="p">;</span>
    <span class="k">private</span> <span class="nx">transformController</span><span class="p">:</span> <span class="nx">TransformController</span><span class="p">;</span>
    <span class="k">private</span> <span class="nx">renderController</span><span class="p">:</span> <span class="nx">RenderController</span><span class="p">;</span>


    <span class="nf">constructor</span><span class="p">()</span> <span class="p">{</span>
        <span class="c1">// Create models</span>
        <span class="k">this</span><span class="p">.</span><span class="nx">sceneModel</span> <span class="o">=</span> <span class="k">new</span> <span class="nc">SceneModel</span><span class="p">();</span>
        <span class="k">this</span><span class="p">.</span><span class="nx">cameraModel</span> <span class="o">=</span> <span class="k">new</span> <span class="nc">CameraModel</span><span class="p">();</span>
        <span class="k">this</span><span class="p">.</span><span class="nx">inputModel</span> <span class="o">=</span> <span class="k">new</span> <span class="nc">InputModel</span><span class="p">();</span>
        <span class="k">this</span><span class="p">.</span><span class="nx">lightsModel</span> <span class="o">=</span> <span class="k">new</span> <span class="nc">LightsModel</span><span class="p">();</span>
        <span class="k">this</span><span class="p">.</span><span class="nx">environmentModel</span> <span class="o">=</span> <span class="k">new</span> <span class="nc">EnvironmentModel</span><span class="p">();</span>

        <span class="c1">// Create views</span>
        <span class="k">this</span><span class="p">.</span><span class="nx">cameraView</span> <span class="o">=</span> <span class="k">new</span> <span class="nc">CameraView</span><span class="p">(</span><span class="k">this</span><span class="p">.</span><span class="nx">cameraModel</span><span class="p">);</span>
        <span class="k">this</span><span class="p">.</span><span class="nx">sceneView</span> <span class="o">=</span> <span class="k">new</span> <span class="nc">SceneView</span><span class="p">(</span>
            <span class="k">this</span><span class="p">.</span><span class="nx">sceneModel</span><span class="p">,</span>
            <span class="k">this</span><span class="p">.</span><span class="nx">lightsModel</span><span class="p">,</span>
            <span class="k">this</span><span class="p">.</span><span class="nx">environmentModel</span><span class="p">,</span>
            <span class="k">this</span><span class="p">.</span><span class="nx">cameraView</span><span class="p">,</span>
        <span class="p">);</span>

        <span class="c1">// Create controllers</span>
        <span class="k">this</span><span class="p">.</span><span class="nx">renderController</span> <span class="o">=</span> <span class="k">new</span> <span class="nc">RenderController</span><span class="p">(</span><span class="k">this</span><span class="p">.</span><span class="nx">update</span><span class="p">);</span>
        <span class="k">this</span><span class="p">.</span><span class="nx">inputController</span> <span class="o">=</span> <span class="k">new</span> <span class="nc">InputController</span><span class="p">(</span>
            <span class="k">this</span><span class="p">.</span><span class="nx">inputModel</span><span class="p">,</span>
            <span class="k">this</span><span class="p">.</span><span class="nx">sceneView</span><span class="p">.</span><span class="nx">domElement</span>
        <span class="p">);</span>
        <span class="k">this</span><span class="p">.</span><span class="nx">transformController</span> <span class="o">=</span> <span class="k">new</span> <span class="nc">TransformController</span><span class="p">(</span>
            <span class="k">this</span><span class="p">.</span><span class="nx">sceneModel</span><span class="p">,</span>
            <span class="k">this</span><span class="p">.</span><span class="nx">inputModel</span><span class="p">,</span>
        <span class="p">);</span>

        <span class="k">this</span><span class="p">.</span><span class="nf">spawnInitialObjects</span><span class="p">();</span>
    <span class="p">}</span>

    <span class="k">private</span> <span class="nx">update</span> <span class="o">=</span> <span class="p">()</span> <span class="o">=&gt;</span> <span class="p">{</span>
        <span class="k">this</span><span class="p">.</span><span class="nx">sceneView</span><span class="p">.</span><span class="nf">update</span><span class="p">();</span>
    <span class="p">}</span>

    <span class="k">private</span> <span class="nf">spawnInitialObjects</span><span class="p">()</span> <span class="p">{</span>
        <span class="kd">const</span> <span class="nx">cube</span> <span class="o">=</span> <span class="k">new</span> <span class="nc">CubeModel</span><span class="p">(</span><span class="dl">'</span><span class="s1">cube-1</span><span class="dl">'</span><span class="p">);</span>
        <span class="kd">const</span> <span class="nx">torus</span> <span class="o">=</span> <span class="k">new</span> <span class="nc">TorusModel</span><span class="p">(</span><span class="dl">'</span><span class="s1">torus-1</span><span class="dl">'</span><span class="p">);</span>

        <span class="k">this</span><span class="p">.</span><span class="nx">sceneModel</span><span class="p">.</span><span class="nf">add</span><span class="p">(</span><span class="nx">cube</span><span class="p">);</span>
        <span class="k">this</span><span class="p">.</span><span class="nx">sceneModel</span><span class="p">.</span><span class="nf">add</span><span class="p">(</span><span class="nx">torus</span><span class="p">,</span> <span class="nx">cube</span><span class="p">);</span>
    <span class="p">}</span>

    <span class="k">public</span> <span class="nf">start</span><span class="p">()</span> <span class="p">{</span>
        <span class="k">this</span><span class="p">.</span><span class="nx">renderController</span><span class="p">.</span><span class="nf">start</span><span class="p">();</span>
    <span class="p">}</span>
<span class="p">}</span>

<span class="kd">const</span> <span class="nx">app</span> <span class="o">=</span> <span class="k">new</span> <span class="nc">Application</span><span class="p">();</span>

<span class="nx">app</span><span class="p">.</span><span class="nf">start</span><span class="p">();</span>

</code></pre>

</div>



<p>The data flow for a single pointer move looks like this: <code>PointerEvent</code> → <code>InputController</code> → <code>InputModel</code> (fires <code>'input-changed'</code>) → <code>TransformController</code> → <code>CubeModel</code>/<code>TorusModel</code> (fires <code>'rotation-changed'</code>) → <code>SceneObjectView</code> (updates mesh rotation). Every link in the chain only knows about the interface it talks to, not the full pipeline.</p>

<p>Now, let's check the final result:</p>

<p><a href="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Frh8bjp9ni1fur50bxp7l.gif" class="article-body-image-wrapper"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Frh8bjp9ni1fur50bxp7l.gif" alt="Three.js Architecture: MVC / Final 3D Visualization" width="560" height="315"></a></p>

<p>🔗 <a href="https://github.com/ibabkov/threejs-koota-ecs-example" rel="noopener noreferrer">GitHub</a> • <a href="https://ibabkov.github.io/threejs-koota-ecs-example/" rel="noopener noreferrer">Demo</a></p>




<h2>
  
  
  Final File Structure
</h2>



<div class="highlight js-code-highlight">
<pre class="highlight plaintext"><code>src/
├── controllers/
│   ├── InputController.ts
│   ├── RenderController.ts
│   └── TransformController.ts
├── models/
│   ├── CameraModel.ts
│   ├── CubeModel.ts
│   ├── EnvironmentModel.ts
│   ├── InputModel.ts
│   ├── LightsModel.ts
│   ├── SceneModel.ts
│   ├── SceneObjectModel.ts
│   └── TorusModel.ts
├── types/
│   └── events.ts
├── views/
│   ├── CameraView.ts
│   ├── CubeView.ts
│   ├── SceneObjectView.ts
│   ├── SceneView.ts
│   └── TorusView.ts
├── index.ts

</code></pre>

</div>






<h2>
  
  
  Benchmarks
</h2>

<p>Now let's validate the results for different numbers of objects. For this purpose, I have to use an InstancedMesh instead of the regular Mesh,<br>
which is outside the scope of the article. The architecture remains the same. Performance profiling over a 10-second interval on a Mac M1 Pro (Safari) yields the following results:</p>

<div class="table-wrapper-paragraph"><table>
<thead>
<tr>
<th>Metric</th>
<th>1 torus + <br>1 cube</th>
<th>10 tori + <br>10 cubes</th>
<th>100 tori + <br>100 cubes</th>
</tr>
</thead>
<tbody>
<tr>
<td>FPS (Avg/Min/Max)</td>
<td>120</td>
<td>120</td>
<td>120</td>
</tr>
<tr>
<td>Input &gt; Render Latency (Avg)</td>
<td>0.67 ms</td>
<td>0.69 ms</td>
<td>0.71</td>
</tr>
<tr>
<td>Input &gt; Render Latency (Max)</td>
<td>2.2 ms</td>
<td>2.4ms</td>
<td>2.8ms</td>
</tr>
<tr>
<td>Memory Footprint</td>
<td>16 MB</td>
<td>16 MB</td>
<td>16 MB</td>
</tr>
</tbody>
</table></div>

<p>The results confirm that the overhead associated with the MVC abstraction layer is negligible at this scale. The application maintains a stable 120 FPS with sub-millisecond input latency. The memory footprint remains a flat 16 MB across 1, 20, and 200 entities due to InstancedMesh implementation.</p>


<h2>
  
  
  Beyond Basic MVC
</h2>
<h3>
  
  
  Centralizing event communication
</h3>

<p>Our models and views currently rely on direct event attachments. We can replace this by routing all messages through a single central dispatcher. Controllers and models send updates to this central hub, and views subscribe only to the specific data they need. You should do this when your app gets large. It prevents you from having to debug complex webs of direct component subscriptions.</p>
<h3>
  
  
  Choosing between MVVM and MVP
</h3>

<p>Many of us need to build a React or Vue interface on top of a 3D scene. Passing data between traditional MVC controllers and a reactive UI often creates duplicate state. We can shift to a <a href="https://en.wikipedia.org/wiki/Model%E2%80%93view%E2%80%93viewmodel" rel="noopener noreferrer">Model-View-ViewModel</a> pattern to solve this. The view model transforms the 3D data into a format the DOM easily understands. We should use this pattern when the application relies heavily on HTML configuration panels.<br>
Alternatively, MVC and MVVM can cause frame drops if thousands of objects recalculate positions every frame. We can restructure into a <a href="https://en.wikipedia.org/wiki/Model%E2%80%93view%E2%80%93presenter" rel="noopener noreferrer">Model-View-Presenter</a> layout to protect performance. The view becomes completely passive in this pattern. The presenter calculates the math for all entities and manually batches the WebGL updates. This approach is necessary when the scene is massive and a stable frame rate is our absolute top priority.</p>
<h3>
  
  
  SOLID
</h3>

<p>While MVC organizes the app, <a href="https://en.wikipedia.org/wiki/SOLID" rel="noopener noreferrer">SOLID</a> principles keep it extensible. Without them, controllers often end up importing concrete classes, and views begin branching on object types. This eventually breaks the clean layering we established.</p>

<p>In <code>TransformController</code>, we see this tight coupling:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight typescript"><code><span class="c1">// src/controllers/TransformController.ts</span>

<span class="k">if </span><span class="p">(</span><span class="nx">obj</span> <span class="k">instanceof</span> <span class="nx">CubeModel</span><span class="p">)</span> <span class="p">{</span>
    <span class="nx">rotation</span><span class="p">.</span><span class="nx">y</span> <span class="o">=</span> <span class="nx">position</span><span class="p">.</span><span class="nx">x</span> <span class="o">*</span> <span class="nb">Math</span><span class="p">.</span><span class="nx">PI</span><span class="p">;</span>
    <span class="nx">obj</span><span class="p">.</span><span class="nf">dispatchEvent</span><span class="p">({</span> <span class="na">type</span><span class="p">:</span> <span class="dl">'</span><span class="s1">rotation-changed</span><span class="dl">'</span><span class="p">,</span> <span class="nx">rotation</span> <span class="p">});</span>
<span class="p">}</span> <span class="k">else</span> <span class="k">if </span><span class="p">(</span><span class="nx">obj</span> <span class="k">instanceof</span> <span class="nx">TorusModel</span><span class="p">)</span> <span class="p">{</span>
    <span class="nx">rotation</span><span class="p">.</span><span class="nx">x</span> <span class="o">=</span> <span class="nx">position</span><span class="p">.</span><span class="nx">y</span> <span class="o">*</span> <span class="nb">Math</span><span class="p">.</span><span class="nx">PI</span><span class="p">;</span>
    <span class="nx">obj</span><span class="p">.</span><span class="nf">dispatchEvent</span><span class="p">({</span> <span class="na">type</span><span class="p">:</span> <span class="dl">'</span><span class="s1">rotation-changed</span><span class="dl">'</span><span class="p">,</span> <span class="nx">rotation</span> <span class="p">});</span>
<span class="p">}</span>
</code></pre>

</div>



<p>And <code>SceneView.addEventListeners</code>:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight typescript"><code><span class="c1">// src/views/SceneView.ts</span>

<span class="k">if </span><span class="p">(</span><span class="nx">event</span><span class="p">.</span><span class="nx">object</span> <span class="k">instanceof</span> <span class="nx">CubeModel</span><span class="p">)</span> <span class="p">{</span>
    <span class="nx">view</span> <span class="o">=</span> <span class="k">new</span> <span class="nc">CubeView</span><span class="p">(</span><span class="nx">event</span><span class="p">.</span><span class="nx">object</span><span class="p">);</span>
<span class="p">}</span> <span class="k">else</span> <span class="k">if </span><span class="p">(</span><span class="nx">event</span><span class="p">.</span><span class="nx">object</span> <span class="k">instanceof</span> <span class="nx">TorusModel</span><span class="p">)</span> <span class="p">{</span>
    <span class="nx">view</span> <span class="o">=</span> <span class="k">new</span> <span class="nc">TorusView</span><span class="p">(</span><span class="nx">event</span><span class="p">.</span><span class="nx">object</span><span class="p">);</span>
<span class="p">}</span>
</code></pre>

</div>



<p>These <code>instanceof</code> chains demonstrate exactly where the coupling occurs. I used them here to keep the data flow obvious, but they violate the Open/Closed Principle, adding a new object type would require modifying both classes. They also break the Dependency Inversion Principle because high-level controllers and views should not depend on concrete model implementations. In a larger project, the rotation logic should be moved into the models, and a view factory should be used to keep the architecture extensible and decoupled.</p>




<h2>
  
  
  Conclusion
</h2>

<p>Using an MVC architecture separates core logic from rendering, making both easier to test and maintain. The benchmarks show that this structure has minimal impact on performance, with stable frame rates and low latency.</p>

<h3>
  
  
  Pros
</h3>

<p><strong>Predictability</strong>: The strict separation means your data is isolated from the rendering loop. You can modify your application state without accidentally breaking the WebGL context.</p>

<p><strong>Testability</strong>: You can run unit tests on your controllers and models in a standard Node environment. You do not need to mock a canvas or a WebGL renderer to verify your logic.</p>

<p><strong>Performance</strong>: The event-driven architecture has a very low footprint. Our tests maintained a stable frame rate and a flat 16 MB memory usage profile across different entity counts.</p>

<h3>
  
  
  Cons
</h3>

<p><strong>Boilerplate</strong>: You have to write significantly more code to get the first object on the screen. Creating a model, a view, and event listeners takes more time than simply adding a mesh to a scene.</p>

<p><strong>Data duplication</strong>: You often need to keep properties synchronized between your models and the actual Three.js objects. This means maintaining position and rotation data in two places at once.</p>

<p><strong>Strict discipline</strong>: You have to strictly maintain the MVC pattern and SOLID principles as the application grows. Letting logic slip into views or allowing controllers to bypass models will break the entire architecture and defeat the purpose of the setup.</p>

<h3>
  
  
  What's Next
</h3>

<p>We proved MVC works well for our immediate needs, but it is just one way to organize a Three.js codebase. In upcoming posts, I'll walk through a few other patterns, so we can look at the practical trade-offs they bring to web-based 3D.</p>




<p><strong>About the Author</strong><br>
Ivan Babkov - 2D/3D Graphics Software Engineer from Canada, building interactive web experiences | 3D, WebGL, GLTF, Three.js</p>

<p><a href="https://www.linkedin.com/in/ivan-babkov/" rel="noopener noreferrer">LinkedIn</a> | <a href="https://www.ibabkov.com/projects/" rel="noopener noreferrer">Portfolio</a> | <a href="https://github.com/ibabkov" rel="noopener noreferrer">GitHub</a></p>

