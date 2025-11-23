---
Title: Nuxt Tutorial 6 - Vue.js Intermezzo
Description: 
Author: Alois Sečkár
Date: 2025-11-23T22:01:10.000Z
Robots: noindex,nofollow
Template: index
---
<p>In this inserted article, we take a closer look at selected fundamental concepts of the Vue.js framework that Nuxt builds upon. I believe it is not a good idea to use any tool without at least a rough understanding of how it works under the hood. Before we continue exploring more of Nuxt’s great features, we’ll go back to the roots without building anything specific here. But you will end up better prepared when you try it yourself later.</p>

<p>To begin, note the best available source of Vue information - the <a href="https://vuejs.org/" rel="noopener noreferrer">official documentation</a>. You will learn much more about everything mentioned in this article there, and you can always return to consult questions and issues.</p>

<p>Now for the promised overview of the basics. First, we return to components, which we <a href="https://dev.to/aloisseckar/nuxt-tutorial-2-components-pages-432g">briefly described</a> earlier, and go a bit deeper now.</p>

<h2>
  
  
  Components
</h2>

<p>The basic building block of a Vue application is a <strong>component</strong>. By composing and nesting them you create the required functionality. By far the most common (though not the only possible) way to define a component is the <strong>SFC – single file component</strong>. As the name suggests, it combines all UI parts and logic into a single file identifiable by the <code>.vue</code> extension.</p>

<p>Vue’s built-in compiler can interpret these files and, during the application build, translate them into fully functional HTML + JavaScript. This means that if we want to use them, we must include a <em>build phase</em> during which the transformation happens. I mention this because Vue can also be used statically via a simple <a href="https://vuejs.org/guide/extras/ways-of-using-vue.html#standalone-script" rel="noopener noreferrer">standalone script inclusion</a> in an HTML page. But this effectively limits you to use only bits of Vue here and there. However, I already tricked you into adopting the build step from the very beginning with the first <code>pnpm dev</code> command. That’s why we could use SFCs without worrying about anything.</p>

<p>An SFC consists of three main parts:</p>

<ul>
<li>
<strong>Template</strong> - an HTML‑like template that defines the final appearance of the rendered page</li>
<li>
<strong>Script</strong> - a section for JS/TS code that drives rendering logic, backend communication, and coordination with other parts of the app</li>
<li>
<strong>Style</strong> - a place for defining CSS styles</li>
</ul>

<p>It is not strictly necessary to include all three. A valid component can contain only <code>&lt;script&gt;</code> (a “renderless” or <em>functional</em> component) or only <code>&lt;template&gt;</code> if you don’t need more substantial data preparation or manipulation. A standalone <code>&lt;style&gt;</code> is not allowed because it wouldn’t be clear what the styles belong to.</p>

<p>I prefer having <code>&lt;template&gt;</code> first, <code>&lt;script&gt;</code> second and <code>&lt;style&gt;</code> last (if present). I found out I swim against a current with this habit, because you'll probably see <code>&lt;script&gt;</code> before <code>&lt;template&gt;</code> more often, but  it just feel more natural - first I want to see how the component will look like and only then what values will it display. Data without context seem less useful for me. Anyway, it is a matter of personal preference. The only guideline set is not to mix the order up within a project. Pick just one style and follow it everywhere.</p>

<p><em>It’s also theoretically possible to define arbitrary <a href="https://vuejs.org/api/sfc-spec.html#custom-blocks" rel="noopener noreferrer">custom blocks</a>. Vue lets you declare them, but you must handle them yourself. While I haven’t needed them in practice yet, a good example is an <a href="https://vue-i18n.intlify.dev/guide/advanced/sfc.html" rel="noopener noreferrer"><code>&lt;i18n&gt;</code> block</a> in the eponymous library for facilitating webapp localization.</em></p>

<p>SFCs are primarily a way to keep all logically connected parts of a certain functionality in one place. This is somewhat at odds with the more traditional (at least for me) separation of concerns. However, in my recent experience it makes much more sense and is much easier to work with than constantly jumping between multiple files. If that’s hard to accept, Vue has sort of a workaround - using the <code>src</code> attribute you can <a href="https://vuejs.org/api/sfc-spec.html#src-imports" rel="noopener noreferrer">import block contents from other files</a>. I still don’t recommend it. If your <code>.vue</code> files feel bloated, rather consider splitting them into several smaller components.</p>

<p>Now let’s look inside each base block:</p>

<h3>
  
  
  Template
</h3>

<p>Inside the <code>&lt;template&gt;</code> tag is the definition of the content of the future rendered (part of the) page. You write code very similar to HTML, but you can use certain special Vue syntactic features:</p>

<ul>
<li>Double curly braces (the “mustache” syntax) allow inserting JavaScript - <code>{{ msg }}</code>. During template compilation, the expression is evaluated and interpolated as <strong>plain text</strong>, and the evaluated current value appears in the rendered HTML. Typically, you want to evaluate only simple variable references or function calls. Technically you are not limited, but it’s considered a good practice to keep the template clean and push heavy JS work into the <code>&lt;script&gt;</code> section or other files - you already know Nuxt allows you to reference them directly thanks to auto‑imports.</li>
<li>A colon <code>:</code> before an attribute name (native or your own - see props section later) is shorthand for <code>v-bind:</code> and binds the attribute to a JavaScript expression. The same advice as above applies - don’t abuse templates for complex inline logic.</li>
<li>An <code>@</code> before an event name is shorthand for <code>v-on:</code> and binds a handler function. When the specified event is captured (native like <code>click</code> or <code>change</code>, or custom - see emits later), the given function is called. The function can be written inline, but it’s again better to reference a callback defined in the appropriate place.</li>
<li>Other <code>v-*</code> directives are pseudo‑attributes that give the compiler instructions for special handling. For example:

<ul>
<li>
<code>v-if</code> - <em>renders</em> the element if the condition <strong>is</strong> met</li>
<li>
<code>v-else</code> - <em>renders</em> the element if the condition <strong>is NOT</strong> met</li>
<li>
<code>v-show</code> - always <em>render</em>, but <em>show</em> the element only if the condition <strong>is</strong> met</li>
<li>
<code>v-for</code> - render multiple items from a list in a loop</li>
<li>
<code>v-html</code> - insert HTML‑styled content into an element (<strong>beware, potentially dangerous</strong>)</li>
<li>complete list is <a href="https://vuejs.org/api/built-in-directives.html" rel="noopener noreferrer">HERE</a>
</li>
<li><em>you can also define <a href="https://vuejs.org/guide/reusability/custom-directives.html" rel="noopener noreferrer">your own</a></em></li>
</ul>


</li>

</ul>

<p>Vue templates also have the ability to understand references to other components. Use the <code>.vue</code> filename in PascalCase notation as an identifier. In plain Vue, components must be <a href="https://vuejs.org/guide/components/registration.html" rel="noopener noreferrer">registered</a> before use. Nuxt handles this automatically as long as <code>.vue</code> files are in the <code>/app/components</code> folder <em>(or you <a href="https://nuxt.com/docs/4.x/directory-structure/app/components" rel="noopener noreferrer">configure directorsies</a> where auto‑import should happen)</em>.</p>

<p>Thanks to this, you can compose larger functional units from individual SFCs. Vue also fuels the simple, yet powerful two-way communication. We’ll show how it works in a moment.</p>

<h3>
  
  
  Script
</h3>

<p>Because Vue values flexibility, the following is not the only option, but in my opinion there is currently no better approach than wrapping your JavaScript/TypeScript logic using:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight typescript"><code><span class="o">&lt;</span><span class="nx">script</span> <span class="nx">setup</span> <span class="nx">lang</span><span class="o">=</span><span class="dl">"</span><span class="s2">ts</span><span class="dl">"</span><span class="o">&gt;</span>
<span class="c1">// your JS/TS logic</span>
<span class="o">&lt;</span><span class="sr">/script</span><span class="err">&gt;
</span></code></pre>

</div>



<p>The tag name is obvious. The <a href="https://vuejs.org/api/sfc-script-setup.html" rel="noopener noreferrer"><code>setup</code></a> attribute tells the compiler that we will use Vue’s <a href="https://vuejs.org/guide/extras/composition-api-faq.html" rel="noopener noreferrer">Composition API</a>. I’ve kept quiet until now that another variant exists, but the Options API is, in my view, an obsolete approach not worth learning first. You may come across guides, libraries and legacy codebases that still use it, but there’s no need to burden yourself until you actually need it. I’m confident you won’t regret using <code>&lt;script setup&gt;</code>. All the Vue/Nuxt features mentioned later implicitly assume using it.</p>

<p>The <code>lang="ts"</code> attribute denotes a preprocessor that enables TypeScript support. It isn’t mandatory, but without TypeScript and its static analysis I can’t really imagine my development. Properly configured TypeScript in your IDE immediately warns you about accessing undefined variables, passing wrong argument types, or calling functions that don’t exist - and much more. It’s worth investing a bit of effort up front to understand its principles.</p>

<p>Any JS variables and functions you define inside <code>&lt;script setup&gt;</code> are automatically available to the component template. They are also safely sealed off from the outside unless you explicitly expose them using <a href="https://vuejs.org/api/sfc-script-setup.html#defineexpose" rel="noopener noreferrer"><code>defineExpose</code></a> macro. I recommend keeping this block short - declare only elements tightly bound to the current component and refactor longer sequences into separate <code>utils</code> or <code>composables</code> (see the <a href="https://dev.to/aloisseckar/nuxt-tutorial-3-utils-composables-3o51">earlier tutorial part</a>). Once we introduce the last section, we’ll get to more concrete examples.</p>

<h3>
  
  
  Style
</h3>

<p>The <code>&lt;style&gt;</code> tag wraps the CSS definition. If you want more than plain CSS, you can use the <code>lang</code> attribute to define a preprocessor (e.g., <a href="https://sass-lang.com/" rel="noopener noreferrer">Sass</a>), but you must first <a href="https://nuxt.com/docs/4.x/getting-started/styling#using-preprocessors" rel="noopener noreferrer">configure it</a> in your project.</p>

<p>It’s a good practice to keep styles inside an SFC local to that component and avoid letting them <em>leaking</em> into the rest of the app. That can have unexpected consequences <a href="https://github.com/vuejs-translations/docs-cs/issues/267" rel="noopener noreferrer">elsewhere</a> and the source of such bugs is hard to find. Vue makes file‑level encapsulation easy - just add the <code>scoped</code> attribute: <code>&lt;style scoped&gt;</code> - and you’re done. Styles meant to apply globally should be defined at the top level in <code>app.vue</code>, or, if there are more of them, in a separate CSS file <a href="https://nuxt.com/docs/4.x/getting-started/styling#the-css-property" rel="noopener noreferrer">loaded via Nuxt config</a>.</p>

<p>You can define multiple <code>&lt;style&gt;</code> blocks in a single component, but I think it’s not very practical. I also haven’t used <a href="https://vuejs.org/api/sfc-css-features.html#css-modules" rel="noopener noreferrer">CSS Modules</a> much.</p>

<p>I would say you’ll likely start relying on this block first, but then you shift towards using it more sparingly - only for specific requirements. Appearance is often handled by UI libraries, and there’s no need to reinvent the wheel. Or if you reach for <strong>Tailwind CSS</strong> (there will be a later tutorial part about it), classic CSS almost disappears for you…</p>

<h2>
  
  
  Passing Data Between Components
</h2>

<p>Encapsulating parts of the app into separate components is great, but they must also communicate. Vue provides support for this indeed.</p>

<h3>
  
  
  Props
</h3>

<p>With the special <code>defineProps()</code> function, you can define a set of variables that can be passed into the component from outside. Conceptually this corresponds to public attributes of a class in object‑oriented languages. To be precise, this is not a real function but another <em>compile‑time macro</em> whose contents are transformed when translating the <code>.vue</code> file into real code.</p>

<p>The macro argument is an object of declared properties that Vue refers as <strong>props</strong>. There are several syntax variants. For a long time I liked so-called <em>object</em> syntax, because it looks natural:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight typescript"><code><span class="kd">const</span> <span class="nx">props</span> <span class="o">=</span> <span class="nf">defineProps</span><span class="p">({</span>
  <span class="na">foo</span><span class="p">:</span> <span class="p">{</span> <span class="na">type</span><span class="p">:</span> <span class="nb">String</span><span class="p">,</span> <span class="na">required</span><span class="p">:</span> <span class="kc">true</span> <span class="p">},</span>
  <span class="na">bar</span><span class="p">:</span> <span class="p">{</span> <span class="na">type</span><span class="p">:</span> <span class="nb">Number</span> <span class="p">},</span>
<span class="p">})</span>
</code></pre>

</div>



<p>The downside is that for more complex type you need to re-type it via generic Vue utility <code>PropType</code>:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight typescript"><code><span class="kd">const</span> <span class="nx">props</span> <span class="o">=</span> <span class="nf">defineProps</span><span class="p">({</span>
  <span class="na">foo</span><span class="p">:</span> <span class="p">{</span> <span class="na">type</span><span class="p">:</span> <span class="nb">String</span><span class="p">,</span> <span class="na">required</span><span class="p">:</span> <span class="kc">true</span> <span class="p">},</span>
  <span class="na">bar</span><span class="p">:</span> <span class="p">{</span> <span class="na">type</span><span class="p">:</span> <span class="nb">Number</span> <span class="p">},</span>
  <span class="na">ext</span><span class="p">:</span> <span class="p">{</span> <span class="na">type</span><span class="p">:</span> <span class="nb">Object</span> <span class="k">as</span> <span class="nx">PropType</span><span class="o">&lt;</span><span class="nx">CustomType</span><span class="o">&gt;</span><span class="p">},</span>
<span class="p">})</span>
</code></pre>

</div>



<p>This is not very intuitive and it is unnecessarily complex. Because of that I finally adopted newer <em>type based</em> syntax. It looks weird, but it is more suitable and much shorter:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight typescript"><code><span class="kd">const</span> <span class="nx">props</span> <span class="o">=</span> <span class="nx">defineProps</span><span class="o">&lt;</span><span class="p">{</span>
  <span class="na">foo</span><span class="p">:</span> <span class="kr">string</span><span class="p">,</span>
  <span class="nx">bar</span><span class="p">?:</span> <span class="kr">number</span><span class="p">,</span>
  <span class="nx">ext</span><span class="p">?:</span> <span class="nx">CustomType</span><span class="p">,</span>
<span class="p">}</span><span class="o">&gt;</span><span class="p">()</span>
</code></pre>

</div>



<p>You can then use the defined values inside <code>&lt;script&gt;</code> as <code>props.foo</code> and <code>props.bar</code>. In <code>&lt;template&gt;</code> expressions, you can refer to <code>foo</code> or <code>bar</code> directly; the compiler is smart enough to resolve that they are <strong>props</strong>.</p>

<p>Passing into the child from one level up looks like this:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight vue"><code><span class="nt">&lt;</span><span class="k">template</span><span class="nt">&gt;</span>
  <span class="nt">&lt;ChildComponent</span> <span class="na">foo=</span><span class="s">"foo"</span> <span class="na">:bar=</span><span class="s">"2"</span> <span class="nt">/&gt;</span>
<span class="nt">&lt;/</span><span class="k">template</span><span class="nt">&gt;</span>
</code></pre>

</div>



<p>Thanks to TypeScript, type checking works, so <code>foo</code> only accepts strings and <code>bar</code> only numbers. Note that text can be passed as a normal HTML attribute value, while all other types must be prefixed with a colon (shorthand for <code>v-bind:</code>).</p>

<p>Also remember that you should treat <strong>props</strong> as <em>read‑only</em> inside a component. If you mutate them, you tightly couple parent and child, which undermines encapsulation and reusability. You can use a <strong>prop</strong> value to initialize your own local variable (beware of <a href="https://medium.com/front-end-weekly/understanding-pass-by-value-and-pass-by-reference-in-javascript-8e2a0806b175" rel="noopener noreferrer">pass‑by‑reference</a> with objects). Usually it’s even better to use <code>v-model</code> or state management (see below).</p>

<h3>
  
  
  Emits
</h3>

<p>That covered passing data into a component; now we need the other direction. Use the <code>defineEmits()</code> macro to declare custom events that a component can <strong>emit</strong>. In the simplest case, pass an array of event names. In the template, trigger an event by passing its name to <code>$emit</code>:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight html"><code><span class="nt">&lt;button</span> <span class="err">@</span><span class="na">click=</span><span class="s">"$emit('event')"</span><span class="nt">&gt;&lt;/button&gt;</span>
</code></pre>

</div>



<p>In <code>&lt;script&gt;</code>, as with props, refer to the <code>defineEmits</code> result:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight typescript"><code><span class="kd">const</span> <span class="nx">emit</span> <span class="o">=</span> <span class="nf">defineEmits</span><span class="p">([</span><span class="dl">'</span><span class="s1">event</span><span class="dl">'</span><span class="p">])</span>

<span class="kd">function</span> <span class="nf">buttonClick</span><span class="p">()</span> <span class="p">{</span>
  <span class="nf">emit</span><span class="p">(</span><span class="dl">'</span><span class="s1">event</span><span class="dl">'</span><span class="p">)</span>
<span class="p">}</span>
</code></pre>

</div>



<p>In the parent component, listen with <code>@</code> (shorthand for <code>v-on:</code>) before the declared event name. When the child emits <code>event</code>, the parent’s <code>foo</code> method is called:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight vue"><code><span class="nt">&lt;</span><span class="k">template</span><span class="nt">&gt;</span>
  <span class="nt">&lt;ChildComponent</span> <span class="err">@</span><span class="na">event=</span><span class="s">"foo()"</span> <span class="nt">/&gt;</span>
<span class="nt">&lt;/</span><span class="k">template</span><span class="nt">&gt;</span>
</code></pre>

</div>



<p>To pass data upward along with the fact that something happened, events can declare payloads whose values are sent with them. Details here: <a href="https://vuejs.org/guide/typescript/composition-api.html#typing-component-emits" rel="noopener noreferrer">typing component emits</a>.</p>

<h3>
  
  
  Provide/Inject
</h3>

<p>Passing <strong>props</strong> works well when the component tree is flat. As components nest deeper, you may find you are passing some props only to forward them further down, until finally a deep child uses them. This can lead to the anti‑pattern known as <em>prop drilling</em>. The result is a tightly coupled structure that is hard to maintain because a change in internal implementation forces you to adjust prop definitions in many intermediate components.</p>

<p>Vue sidesteps this with <code>provide</code> and <code>inject</code>. Using <code>provide()</code> you can declare that a component exposes some data to all its descendants:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight typescript"><code><span class="nf">provide</span><span class="p">(</span><span class="dl">'</span><span class="s1">message</span><span class="dl">'</span><span class="p">,</span> <span class="dl">'</span><span class="s1">foo</span><span class="dl">'</span><span class="p">)</span>
</code></pre>

</div>



<p>Provided the component is in the ancestor chain, a descendant can request the data with <code>inject()</code>:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight typescript"><code><span class="kd">const</span> <span class="nx">message</span> <span class="o">=</span> <span class="nf">inject</span><span class="p">(</span><span class="dl">'</span><span class="s1">message</span><span class="dl">'</span><span class="p">)</span>
</code></pre>

</div>



<p>You can also provide data globally at the application level using <code>app.provide()</code>.</p>

<p>While this distribution mechanism may seem useful at first; in my experience I’ve rarely used it. In practice I find state‑management libraries work better (see below). It’s good to know the option exists, though.</p>

<p>We don’t have a similar problem with <strong>emits</strong> because events naturally bubble up the <strong>DOM tree</strong> and it’s up to you where to catch them. <em>Additionally, Vue lets you play with event behavior via <a href="https://vuejs.org/guide/essentials/event-handling.html#event-modifiers" rel="noopener noreferrer">modifiers</a>.</em></p>

<p>A brief comment on the <strong>DOM</strong>: it stands for <em>Document Object Model</em>, the internal representation of an HTML document - tags, their attributes, and content. By nature it’s a tree-like structure from the <em>root</em> element down to the deepest descendants. A lightweight JavaScript variant is the <strong>virtual DOM</strong>, and because manipulating its objects is much faster and more efficient than touching the real DOM, Vue and other frameworks primarily work with it and only then project the result into actual HTML for rendering.</p>

<h3>
  
  
  v-model
</h3>

<p>A common scenario in interactive apps is a component that accepts an initial value, lets the user work with it, and notifies the parent when it changes. A typical example is a form input.</p>

<p>The <code>v-model</code> directive in combination with the <code>defineModel</code> macro provides a more straightforward definition for this. You can create a simple <code>Child.vue</code> component like this:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight vue"><code><span class="nt">&lt;</span><span class="k">template</span><span class="nt">&gt;</span>
  <span class="nt">&lt;input</span> <span class="na">v-model=</span><span class="s">"model"</span> <span class="nt">/&gt;</span>
<span class="nt">&lt;/</span><span class="k">template</span><span class="nt">&gt;</span>

<span class="nt">&lt;</span><span class="k">script</span> <span class="na">setup</span><span class="nt">&gt;</span>
<span class="kd">const</span> <span class="nx">model</span> <span class="o">=</span> <span class="nf">defineModel</span><span class="p">()</span>
<span class="nt">&lt;/</span><span class="k">script</span><span class="nt">&gt;</span>
</code></pre>

</div>



<p>And call it just as simply from <code>Parent.vue</code>:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight vue"><code><span class="nt">&lt;</span><span class="k">template</span><span class="nt">&gt;</span>
  <span class="nt">&lt;Child</span> <span class="na">v-model=</span><span class="s">"foo"</span> <span class="nt">/&gt;</span>
<span class="nt">&lt;/</span><span class="k">template</span><span class="nt">&gt;</span>
</code></pre>

</div>



<p>Vue automatically ensures that user input updates <code>foo</code>; there’s no need to wire extra handlers. Another nice simplification of routine tasks.</p>

<h3>
  
  
  State management
</h3>

<p>The mechanisms above work fine for simpler apps. As the number of components and interactions grows, constantly thinking about where to wire connections becomes annoying. In such cases, raise the problem a level up and keep the state in one place across the entire application.</p>

<p>Nuxt provides the <a href="https://nuxt.com/docs/4.x/api/composables/use-state" rel="noopener noreferrer"><code>useState</code></a> composable, which is sufficient for less complex data on its own. The de facto standard for state management in Vue is currently <strong>Pinia</strong>, which makes it easy to create, maintain, and safely use global state anywhere in a Vue (Nuxt) app. More on this later in a separate article.</p>

<h2>
  
  
  Reactivity
</h2>

<p>Vue’s true power appears when you start working with dynamically changing data. For example, after clicking a button, you want the <em>“number of clicks”</em> value to update. You register a <code>click</code> listener, increment a variable… and then what? How to force a redraw on the screen? Should you reach into the DOM and edit an element’s text content? None of that - in Vue it just <em>“happens”</em> thanks to reactivity.</p>

<p>In the Vue world, reactivity means you can create special wrapper objects around data that automatically track value changes and <strong>react</strong> by propagating them to <strong>every place</strong> where the value is used. Different frameworks solve this general notification problem differently. Vue offers the <a href="https://vuejs.org/api/reactivity-core.html" rel="noopener noreferrer">Reactivity API</a> that makes it straightforward.</p>

<p>We won’t dive deep into the technical background here. If you’re interested, continue with <a href="https://vuejs.org/guide/extras/reactivity-in-depth.html" rel="noopener noreferrer">Reactivity in depth</a> in the docs. For now, three functions suffice:</p>

<ul>
<li>
<code>ref()</code> - takes a primitive/object/array and wraps it so the return value is reactive. Wherever you use it, it updates in the future whenever it changes. Such objects (<strong>refs</strong>) can be used in templates, passed as function arguments or even as <strong>props</strong> to components and carry their reactivity with them. The small cost is writing <code>name.value</code> when used in scripts to access the inner value. Inside <code>&lt;template&gt;</code>, <code>name</code> suffices because the compiler fills in <code>.value</code>.</li>
<li>
<code>computed()</code> - takes a callback function that automatically re‑runs if a reactive change is detected inside - i.e., when some value the system tracks changes. This defines dynamic computations. For example, for a simple adder, you can define two <code>ref()</code> values bound to user input, and their sum is <code>computed(() =&gt; a.value + b.value)</code>. Whenever the user changes one of the inputs, the result re‑computes immediately.</li>
<li>
<code>watch()</code> - allows observing a reactive value and performing a <em>side effect</em> whenever it changes. The first argument determines the value to watch; the second is the callback to run on change. For example, you could track the count of failed login attempts and, after exceeding a limit, set a variable that disables the <em>“Log in”</em> button.</li>
</ul>

<p>That was a whirlwind tour. The reactivity system is, of course, more complex; more functions exist and offer various options. You can read them as needed. Even with these three basics, you can cover many scenarios.</p>

<p>It’s more worth pointing out a pitfall now. In my experience, the ease of use tends to tempt overuse in places where it’s not needed. As an application grows, especially with <code>computed()</code> and <code>watch()</code>, more and more re‑evaluations happen and a seemingly small change can cause a cascade of calls, not to mention the subsequent manipulations of the DOM. Vue cleverly batches reactive updates and optimizes DOM work, but background load still grows. It also becomes difficult to trace update flows when debugging hidden changes that shouldn’t occur.</p>

<p>To make the problem worse, at first you might not even realize there is any. All reactivity operations run quietly in the background and, because Vue is well optimized for performance, you won’t notice anything wrong in simple apps. When it starts catching up with you, refactoring and optimizing can be quite hard already. It’s better to approach reactivity skeptically from the start - don’t ask <em>“what can I make reactive”</em>, but <em>“what doesn’t need to be reactive”</em>, because it won’t actually change, or it can be solved otherwise than adding <code>computed()/watch()</code>.</p>

<p>Still, it’s a very powerful tool and an integral part of a Vue developer’s arsenal. Just beware the <em>Man with a Hammer</em> syndrome.</p>

<h2>
  
  
  Component lifecycle
</h2>

<p>Reactive or not, a Vue component instance is not a static entity. It goes through several phases of its virtual life. Very briefly and simply:</p>

<p>1) When the compiler determines a new instance should be created, it first runs the code inside <code>&lt;script setup&gt;</code>. Variables are created and initial reactive state is set.<br>
2) The <strong>mount</strong> phase begins, when the necessary HTML structure is created inside the virtual DOM that Vue uses to control what ultimately gets rendered. The finished component is then displayed.<br>
3) The mounted component can enter the <strong>update</strong> phase if a relevant reactive state change is detected - values are recalculated, the virtual DOM is patched, and the HTML is re‑rendered.<br>
4) The component is <strong>unmounted</strong> because different content is rendered, navigation occurs, or the app exits.<br>
5) The component instance ceases to exist.</p>

<p>Before and after each <strong>mount</strong>, <strong>update</strong>, and <strong>unmount</strong> phase, you can register a callback that Vue runs every time it reaches that point. Use the special <em>lifecycle hooks</em> functions <code>onBeforeMount()</code>, <code>onMounted()</code>, etc. This can be useful if, for example, you want to show a message that page initialization completed or release resources after the component is done.</p>

<p>Beware of two things:</p>

<ul>
<li>Async functions are <strong>not awaited</strong> here. If needed, put <code>await</code> directly in <code>&lt;script setup&gt;</code>. An async function inside <code>onBeforeMount()</code> <strong>does not</strong> guarantee that DOM mounting starts only after it finishes.</li>
<li>Don’t place hook registration inside an asynchronous block (e.g., within <code>setTimeout</code>). As soon as <code>&lt;script setup&gt;</code> finishes synchronously, Vue loses the current component instance context, so there is nothing to attach to.</li>
</ul>

<p>More information, including a clear diagram, can be found <a href="https://vuejs.org/guide/essentials/lifecycle.html" rel="noopener noreferrer">HERE</a>.</p>

<h2>
  
  
  Summary
</h2>

<p>This article is already quite long, even though we only skimmed the surface of most topics and didn’t start many others. I repeat my tip from the beginning - read the <a href="https://vuejs.org/" rel="noopener noreferrer">Vue documentation</a>, where you’ll find much more. There is, of course, an extensive <a href="https://nuxt.com/" rel="noopener noreferrer">documentation for Nuxt</a> too.</p>

<p>At this point, you should be armed with enough knowledge about Vue and the Nuxt framework to start building larger, more realistic apps yourself. However, doing everything from scratch could be unnecessarily time‑consuming. In the next part of the tutorial, we’ll show how to integrate elements of selected UI libraries with little effort and start using them right away.</p>

