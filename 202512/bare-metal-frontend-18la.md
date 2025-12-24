---
Title: Bare-metal frontend
Description: 
Author: Stanislav Yaranov
Date: 2025-12-24T21:07:27.000Z
Robots: noindex,nofollow
Template: index
---
<h1>
  
  
  Bare-metal frontend
</h1>

<h2>
  
  
  Introduction
</h2>

<p>Modern frontend applications have become very rich, complex and sophisticated. They are not just simple UIs polling data. They have complex business logic, various flows, real-time updates, offline support, caching, and so on. All this complexity needs to be managed and organized properly to keep the code maintainable, testable, and extensible.</p>

<p>There are many ways to architect and develop such applications.<br>
Also today there are a number of libraries that help with organizing different flows, state management, side effects handling, data fetching, caching, etc.<br>
Often times applications become very dependent of these libraries, and testing features becomes complicated because business logic and UI become intertwined, not easy to mock dependencies or even have to add another library to do it, UI dictates how the business logic should flow, even worse the business logic becomes just a side effect of UI.</p>

<p>I'm very happy if you don't have these issues and your structure is well thought from the beginning with clear separation of concerns, but I'm sure there are still plenty of developers who face these challenges, and I hope this article will help get more insights in future projects.</p>

<p>Describing a problem. You participate in React project which has grown into something big, and you're spending significant amount of time understanding how various things happen, you find yourself tangled in a web of prop drilling, state management complexities, component re-renders, while all this is tightly connected with business logic processing. You see a button, this button has a hook, the hook dispatches an action, that action updates some state, the state change triggers another action to trigger a side effect, it fetches some data, that data updates some other state, and so on. And when you need to add a new feature, it turns out that adding a test for it is even more complicated than implementing the feature itself. Then you have to include a whole libraries for trivial things like caching of requests, orchestrating flows, mocking/intercepting the added libraries. And I want to point out that we're still talking about making the project work, not about adding a value to the product.<br>
So how to avoid this in the future or maybe try to improve the existing projects?</p>

<p>The key problem here is how we treat React in general. For some reason many developers consider React as a framework for building applications, and the project development starts with React components because we naturally want to see results as soon as possible, then extending their behaviour and adding business logic inside hooks, then in actions, sagas, queries, you name it. The funny thing is that if you open <a href="https://react.dev/" rel="noopener noreferrer">React</a> home page, you'll find how React identifies itself:</p>

<blockquote>
<p>The library for web and native user interfaces</p>
</blockquote>

<p>It's a library. Thus, it should only be used as a tool, and not be a central piece of the application.</p>

<p>We just need to change the perspective.<br>
There is an application doing business logic in the center, and there is UI that can be implemented with React, Vue, plain JS, or anything else. The UI is just a way to present the application state and capabilities to the user and get user input to trigger these capabilities.</p>

<p>I think it should be much easier to describe my thinkings with a practical example. We'll build a simple but not trivial application with React UI. I should confess that while preparing it for the article I started with business logic, not sure if this is because of my backend background, but then I realized that most of frontend developers most likely would start with UI. So let's try to do the same while maintaining the same perspective. Of course, the approach we're going to discuss here doesn't actually match the complexity of the task, this is demo after all, at the same time even small applications may grow into something huge.</p>
<h2>
  
  
  Input
</h2>

<p>Let's say we have the following requirements for our application:</p>

<p>A simple travel app where users can search for travel options, view travel details, and purchase travels. A travel option includes information about departure and destination locations, departure date, travel class, price, airline.</p>

<p>Once a user opens the app, they see a Home page. On the Home page they see a form to set up travel preferences (from location, to location, travel classes), a list of previously searched preferences (if any), a list of last minute deals (if any).</p>

<p>Clicking a recent search preference triggers a search with those criteria. Once the user submits the form, they see a list of travel cards (under last minute deals) with information about each travel option. The search form is visible on every page to refine the search.</p>

<p>Each card has a button to purchase a travel. When the user clicks on the purchase button, if they are not signed in, they see a Sign In pop-up. After signing in they need to click purchase again. After purchasing, they can see the travel on the Profile page, and a congratulatory message shows up. The purchased travel can have a name, and the user can change it.</p>

<p>On every page there is a header with a home button, a sign in or profile button.</p>

<p>Also, if a new last minute deal appears in the system, a notification shows up with a link to the Home page with last minute deals list.</p>
<h2>
  
  
  UI
</h2>
<h3>
  
  
  Analysis
</h3>

<p>Now let's think about how to structure it, starting with UI.</p>

<p>From the description we can identify there are two pages:</p>

<ul>
<li>
<strong>Home page</strong> - contains the following containers:

<ul>
<li>
<strong>Recent searches list</strong> - to show previously searched preferences</li>
<li>
<strong>Last minute deals list</strong> - to show last minute deals</li>
<li>
<strong>Travel cards list</strong> - to show search results</li>
</ul>
</li>
<li>
<strong>Profile page</strong> - contains the following containers:

<ul>
<li>
<strong>User info</strong> - to show user information</li>
<li>
<strong>Purchased travels list</strong> - to show travels purchased by the user</li>
</ul>
</li>
</ul>

<p>Also there are several modals:</p>

<ul>
<li>
<strong>Sign In modal</strong> - to login or register</li>
<li>
<strong>Purchase congratulation modal</strong> - to show a success message after purchase</li>
<li>
<strong>Rename travel modal</strong> - to rename a purchased travel on the profile page</li>
</ul>

<p>And finally there is one more dynamic element:</p>

<ul>
<li>
<strong>Notification pop-up</strong> - to notify the user about events and errors</li>
</ul>

<p>The UI will need at least one layout and since it is allowed to search travels regardless of authentication status, and search form is visible on every page, we can have only one layout:</p>

<ul>
<li>
<strong>Session layout</strong> - contains the following containers:

<ul>
<li>
<strong>Header</strong> - header with title and action links</li>
<li>
<strong>Search form</strong> - to set up travel preferences</li>
</ul>
</li>
</ul>

<p>These are the UI components that deliver the required experience.</p>

<p>Then we need to think about what data and actions the UI will need from the application to maintain this experience.</p>

<p>From reading the description again, we have the following entities to work with:</p>

<ul>
<li>
<strong>Session</strong> - represents the user session, contains user information and probably other session related data</li>
<li>
<strong>Travel card</strong> - represents a travel option with all its details</li>
<li>
<strong>Search criteria</strong> - represents the search preferences set by the user</li>
<li>
<strong>Last minute deal</strong> - represents a last minute deal option</li>
<li>
<strong>Purchased travel</strong> - represents a travel that the user has purchased</li>
</ul>

<p>The UI expects to be supplied with hooks to get these entities.</p>

<p>And here's the list of actions that we can identify at this stage:</p>

<ul>
<li>
<strong>Login</strong> - to log the user in</li>
<li>
<strong>Register</strong> - to register a new user</li>
<li>
<strong>Search travels</strong> - to search for travel options based on criteria</li>
<li>
<strong>Get last minute deals</strong> - to get current last minute deals existing in the system</li>
<li>
<strong>Get new last minute deals</strong> - to get new last minute deals which are not shown yet</li>
<li>
<strong>Get recent searches</strong> - to get the list of recent search criteria</li>
<li>
<strong>Purchase travel</strong> - to purchase a travel option</li>
<li>
<strong>Purchase last minute deal</strong> - to purchase a last minute deal</li>
<li>
<strong>Rename purchased travel</strong> - to rename a purchased travel option</li>
</ul>

<p>Nice. We can add more later. Now as we have the overall picture of the UI, we can move on to implementation.</p>
<h3>
  
  
  Implementation
</h3>

<p>First of all, please find the code repository here: <a href="https://github.com/afewlinesofcode/frontend-app/" rel="noopener noreferrer">https://github.com/afewlinesofcode/frontend-app/</a></p>

<p>And use it as a reference while reading the article. I didn't want to bloat the article with many code snippets, so I'll try to just describe what I've done in each step, and you can check the code in the repository.</p>

<p>For this project we decided to use <a href="https://react.dev/" rel="noopener noreferrer">React</a> with <a href="https://www.typescriptlang.org/" rel="noopener noreferrer">TypeScript</a> and <a href="https://ui.mantine.dev/" rel="noopener noreferrer">Mantine UI</a> library for components and styling.</p>

<p>Let's use <a href="https://vite.dev/" rel="noopener noreferrer">Vite</a> this time to create a React + TypeScript project. Then install Mantine UI library and other required dependencies. In the <code>src/</code> folder create <code>ui/</code> folder for UI components and <code>app/</code> folder for future business logic application.</p>

<p>My strategy is the following. First, I define the contracts specifying which objects I'm going to work with and how I can get them. Second, I define application React context that will provide means to access these contracts and to connect with the application to access its API. And then I can create all necesary UI components.</p>

<p>Now starting implementation.</p>
<h4>
  
  
  Contracts
</h4>

<p>First I want to define UI expectations in <code>ui/contracts/</code> folder. The application will have to comply these expectations. TypeScript is not strictly typed like C++ or Java so we'll leverage from using it as a plug and a socket (interfaces with same shape are compatible).</p>

<p>For the entites I do it in two files: <code>auth.ts</code> and <code>travel.ts</code>. Why? Let's take a look at our entities and actions in the <em>Analysis</em> stage again. We can find that they can be split into two main contexts (or two bounded contexts if speaking DDD terms): Auth context and Travel context. So let's break it for better maintainability.</p>

<p>Entity types defined in <code>auth.ts</code>:</p>

<ul>
<li>
<code>Session</code> - contains name and email of the user</li>
<li>
<code>AuthStatus</code> - contains loading state for authentication actions</li>
</ul>

<p>Entity types defined in <code>travel.ts</code>:</p>

<ul>
<li>
<code>TravelCard</code> - contains id, from and to locations, departure date, price, airline, travel class</li>
<li>
<code>SearchCriteria</code> - contains from, to, and travel class</li>
<li>
<code>LastMinuteDeal</code> - contains same fields as in travel card as a denormalized view, plus id of the travel card this deal is related to, and description</li>
<li>
<code>PurchasedTravel</code> - contains same fields as in travel card, plus id of the related travel, purchased date, and configurable name</li>
<li>
<code>TravelStatus</code> - contains loading states for travel related actions</li>
</ul>

<p>Please check these structs in the repository.</p>
<h4>
  
  
  Application context
</h4>

<p>In the <code>ui/contracts/app.ts</code> file I define interface for application's React context <code>AppContext&lt;Api&gt;</code> that will provide all necessary hooks to get entities per bounded context as well as the <code>api</code> reference that will provide the API to push the flow forward.<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight typescript"><code><span class="kr">interface</span> <span class="nx">AuthHooksContext</span> <span class="p">{</span>
  <span class="nl">useSession</span><span class="p">:</span> <span class="p">()</span> <span class="o">=&gt;</span> <span class="nx">Session</span> <span class="o">|</span> <span class="kc">null</span>
  <span class="nx">useStatus</span><span class="p">:</span> <span class="p">(</span><span class="nx">key</span><span class="p">:</span> <span class="kr">keyof</span> <span class="nx">AuthStatus</span><span class="p">)</span> <span class="o">=&gt;</span> <span class="nx">AuthStatus</span><span class="p">[</span><span class="k">typeof</span> <span class="nx">key</span><span class="p">]</span>
<span class="p">}</span>

<span class="kr">interface</span> <span class="nx">TravelHooksContext</span> <span class="p">{</span>
  <span class="nl">useSearchCriteria</span><span class="p">:</span> <span class="p">()</span> <span class="o">=&gt;</span> <span class="nx">SearchCriteria</span> <span class="o">|</span> <span class="kc">null</span>
  <span class="nx">useTravelCards</span><span class="p">:</span> <span class="p">()</span> <span class="o">=&gt;</span> <span class="nx">TravelCard</span><span class="p">[]</span>
  <span class="nx">usePurchasedTravels</span><span class="p">:</span> <span class="p">()</span> <span class="o">=&gt;</span> <span class="nx">PurchasedTravel</span><span class="p">[]</span>
  <span class="nx">useRecentSearches</span><span class="p">:</span> <span class="p">()</span> <span class="o">=&gt;</span> <span class="nx">SearchCriteria</span><span class="p">[]</span>
  <span class="nx">useLastMinuteDeals</span><span class="p">:</span> <span class="p">()</span> <span class="o">=&gt;</span> <span class="nx">LastMinuteDeal</span><span class="p">[]</span>
  <span class="nx">useStatus</span><span class="p">:</span> <span class="p">(</span><span class="nx">key</span><span class="p">:</span> <span class="kr">keyof</span> <span class="nx">TravelStatus</span><span class="p">)</span> <span class="o">=&gt;</span> <span class="nx">TravelStatus</span><span class="p">[</span><span class="k">typeof</span> <span class="nx">key</span><span class="p">]</span>
<span class="p">}</span>

<span class="k">export</span> <span class="kr">interface</span> <span class="nx">AppContext</span><span class="o">&lt;</span><span class="nx">Api</span><span class="o">&gt;</span> <span class="p">{</span>
  <span class="na">api</span><span class="p">:</span> <span class="nx">Api</span>
  <span class="na">auth</span><span class="p">:</span> <span class="nx">AuthHooksContext</span>
  <span class="na">travel</span><span class="p">:</span> <span class="nx">TravelHooksContext</span>
<span class="p">}</span>
</code></pre>

</div>



<p>You've probably noticed that I've used generic <code>Api</code> type here. The idea is that there's no application at this stage. So while working on UI we can have some demo application implementation not necessarily with the same API as will be in the real application (however it's better to define it to avoid the necessity to adjust API calls later), but enough to provide the required UI experience (like adding a travel card, triggering last minute deal appearance). Also I want to point out that having an application context wrapping all UI components is totally safe because all props in it are stable references, so no re-renders will be triggered because of context value changes.</p>

<p>We have entities representing data structures to work with and hooks providing these data structures. It is time to create React context to provide all this to components. Let's do it in <code>ui/context/app.ts</code> file. And if you've checked it already in the repository, you can notice that I also implemented a very simple demo application to use while working on the UI. Let's take a look at it.</p>

<h4>
  
  
  Demo application
</h4>

<p><code>ui/context/demo/app.ts</code> file provides composition root for the demo application with <code>api</code> property providing <code>auth</code> and <code>travel</code> contexts' APIs, and <code>react</code> property providing hooks from these contexts.<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight typescript"><code><span class="p">{</span>
    <span class="nl">api</span><span class="p">:</span> <span class="p">{</span>
      <span class="na">auth</span><span class="p">:</span> <span class="nx">auth</span><span class="p">.</span><span class="nx">app</span><span class="p">,</span>
      <span class="na">travel</span><span class="p">:</span> <span class="nx">travel</span><span class="p">.</span><span class="nx">app</span><span class="p">,</span>
    <span class="p">},</span>
    <span class="nx">react</span><span class="p">:</span> <span class="p">{</span>
      <span class="nl">auth</span><span class="p">:</span> <span class="nx">auth</span><span class="p">.</span><span class="nx">react</span><span class="p">,</span>
      <span class="nx">travel</span><span class="p">:</span> <span class="nx">travel</span><span class="p">.</span><span class="nx">react</span><span class="p">,</span>
    <span class="p">},</span>
  <span class="p">}</span>
</code></pre>

</div>



<p>For example, let's analyze <code>demo/auth.ts</code> file.<br>
I didn't want to add any event management or state management libraries, so created a primitive helper in <code>demo/reactive.ts</code> that would allow getting reactions on value changes that I can use in hooks. Having it, I've just defined auth state object variable like this:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight typescript"><code><span class="kd">const</span> <span class="nx">state</span> <span class="o">=</span> <span class="p">{</span>
  <span class="na">session</span><span class="p">:</span> <span class="k">new</span> <span class="nc">Reactive</span><span class="p">(</span><span class="kc">null</span> <span class="k">as</span> <span class="nx">Session</span> <span class="o">|</span> <span class="kc">null</span><span class="p">),</span>
  <span class="na">authStatus</span><span class="p">:</span> <span class="k">new</span> <span class="nc">Reactive</span><span class="p">({</span> <span class="na">isLoading</span><span class="p">:</span> <span class="kc">false</span> <span class="p">}</span> <span class="k">as</span> <span class="nx">AuthStatus</span><span class="p">),</span>
<span class="p">}</span>
</code></pre>

</div>



<p>Now <code>state.session.value</code> would return the session value, and by calling <code>state.session.subscribe(callback)</code> I can get notified when the session value gets changed by <code>state.session.value = &lt;new value&gt;</code> expression. And hooks effectively use it via <code>useReactiveValue</code> helper hook defined in the same file. Now to create a session hook I just define a function:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight typescript"><code><span class="kd">function</span> <span class="nf">useSession</span><span class="p">()</span> <span class="p">{</span>
  <span class="k">return</span> <span class="nf">useReactiveValue</span><span class="p">(</span><span class="nx">state</span><span class="p">.</span><span class="nx">session</span><span class="p">)</span>
<span class="p">}</span>
</code></pre>

</div>



<p>And a function to perform login action:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight typescript"><code><span class="k">async</span> <span class="kd">function</span> <span class="nf">login</span><span class="p">(</span><span class="nx">command</span><span class="p">:</span> <span class="p">{</span> <span class="nl">email</span><span class="p">:</span> <span class="kr">string</span><span class="p">;</span> <span class="nl">password</span><span class="p">:</span> <span class="kr">string</span> <span class="p">})</span> <span class="p">{</span>
  <span class="c1">// emulate loading</span>
  <span class="nx">state</span><span class="p">.</span><span class="nx">authStatus</span><span class="p">.</span><span class="nx">value</span> <span class="o">=</span> <span class="p">{</span> <span class="p">...</span><span class="nx">state</span><span class="p">.</span><span class="nx">authStatus</span><span class="p">.</span><span class="nx">value</span><span class="p">,</span> <span class="na">isLoading</span><span class="p">:</span> <span class="kc">true</span> <span class="p">}</span>
  <span class="k">await</span> <span class="nf">sleep</span><span class="p">(</span><span class="mi">1000</span><span class="p">)</span>
  <span class="nx">state</span><span class="p">.</span><span class="nx">authStatus</span><span class="p">.</span><span class="nx">value</span> <span class="o">=</span> <span class="p">{</span> <span class="p">...</span><span class="nx">state</span><span class="p">.</span><span class="nx">authStatus</span><span class="p">.</span><span class="nx">value</span><span class="p">,</span> <span class="na">isLoading</span><span class="p">:</span> <span class="kc">false</span> <span class="p">}</span>
  <span class="c1">// set session</span>
  <span class="nx">state</span><span class="p">.</span><span class="nx">session</span><span class="p">.</span><span class="nx">value</span> <span class="o">=</span> <span class="p">{</span>
    <span class="na">email</span><span class="p">:</span> <span class="nx">command</span><span class="p">.</span><span class="nx">email</span><span class="p">,</span>
    <span class="na">name</span><span class="p">:</span> <span class="dl">'</span><span class="s1">Test User</span><span class="dl">'</span><span class="p">,</span>
  <span class="p">}</span>
<span class="p">}</span>
</code></pre>

</div>



<p>Finally, <code>composeAuth</code> combines these functions and returns <code>api</code> with an API method to log in with loading simulation and an API method to register which throws an error as handling of errors also needs to be implemented in the UI, and <code>react</code> with hooks to get current session and auth status.</p>

<p>Similarly, <code>demo/travel.ts</code> file implements travel context with its own state and API methods and hooks.</p>

<p><code>demo/app.ts</code> composes these two contexts into one demo application.</p>

<h4>
  
  
  Back to the context
</h4>

<p>So the context is created as:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight typescript"><code><span class="kd">type</span> <span class="nx">Api</span> <span class="o">=</span> <span class="nb">ReturnType</span><span class="o">&lt;</span><span class="k">typeof</span> <span class="nx">composeDemo</span><span class="o">&gt;</span><span class="p">[</span><span class="dl">'</span><span class="s1">api</span><span class="dl">'</span><span class="p">]</span>
<span class="k">export</span> <span class="kd">const</span> <span class="nx">appContext</span> <span class="o">=</span> <span class="nx">createContext</span><span class="o">&lt;</span><span class="nx">AppContext</span><span class="o">&lt;</span><span class="nx">Api</span><span class="o">&gt;</span> <span class="o">|</span> <span class="kc">null</span><span class="o">&gt;</span><span class="p">(</span><span class="kc">null</span><span class="p">)</span>
</code></pre>

</div>



<p>And <code>ui/context/provider/app.tsx</code> file completes the application context implementation with provider component.</p>

<p>When the real application becomes available, we can plug it in by changing the <code>Api</code> type, and composing it in the provider, also will probably need to adjust API calls (as mentioned earlier) in the UI components, but the overall structure will remain the same.</p>

<p>After wrapping everything with <code>AppProvider</code> in <code>ui/main.tsx</code>, anywhere in the components tree accessing application data can be as simple as:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight typescript"><code><span class="kd">const</span> <span class="p">{</span> <span class="nx">auth</span> <span class="p">}</span> <span class="o">=</span> <span class="nf">useAppContext</span><span class="p">()</span>
<span class="kd">const</span> <span class="nx">session</span> <span class="o">=</span> <span class="nx">auth</span><span class="p">.</span><span class="nf">useSession</span><span class="p">()</span>
</code></pre>

</div>



<p>And if we need to trigger an action:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight typescript"><code><span class="kd">const</span> <span class="p">{</span> <span class="nx">api</span> <span class="p">}</span> <span class="o">=</span> <span class="nf">useAppContext</span><span class="p">()</span>

<span class="kd">const</span> <span class="nx">onSearch</span> <span class="o">=</span> <span class="nf">useCallback</span><span class="p">(</span>
  <span class="p">(</span><span class="nx">criteria</span><span class="p">:</span> <span class="nx">SearchCriteria</span><span class="p">)</span> <span class="o">=&gt;</span> <span class="p">{</span>
    <span class="nx">api</span><span class="p">.</span><span class="nx">travel</span><span class="p">.</span><span class="nf">searchTravels</span><span class="p">(</span><span class="nx">criteria</span><span class="p">)</span>
  <span class="p">},</span>
  <span class="p">[</span><span class="nx">api</span><span class="p">]</span>
<span class="p">)</span>
</code></pre>

</div>



<p>One nice thing that you'll appreciate in the future is the <code>window.api</code> that you can expose from the provider component. It allows you to access the application API from the browser console to manipulate the application state while working on the UI.</p>

<p>Seems like we have everything to start building UI components.</p>

<h4>
  
  
  Components
</h4>

<p>Need to build a tree of UI components delivering all required experience.<br>
Let's make a general structure to quickly overview what we're doing.</p>

<ul>
<li>Session layout

<ul>
<li>Search form</li>
<li>Links: Home, Profile/Login</li>
</ul>
</li>
<li>Home page

<ul>
<li>Recent searches list</li>
<li>Last minute deals list</li>
<li>Travel cards list</li>
</ul>
</li>
<li>Profile page

<ul>
<li>User info</li>
<li>Purchased travels list</li>
</ul>
</li>
<li>Dynamic elements

<ul>
<li>Sign In modal</li>
<li>Purchase congratulation modal</li>
<li>Rename travel modal</li>
<li>Notification pop-up</li>
</ul>
</li>
</ul>

<p>Nice.</p>

<p>Creating React components is not in the scope of this article, please check the repository for their implementation. Most of them are built from examples provided on <a href="https://ui.mantine.dev/" rel="noopener noreferrer">Mantine UI</a> website. The main idea here is that components only use the application context to get data and trigger actions, no logic except very simple solely UI oriented like modals visibility, resetting forms, etc.</p>

<p>I want to draw your attention to one item that is created for UI purposes - UI context in <code>ui/contexts/ui.ts</code> file. It provides hooks to manage UI state like modals visibility, notifications, etc. It contains only UI logic, absolutely nothing that may be related to business logic. This implementation with React context isn't very efficient because it will trigger unnecessary re-renders on modal actions making context value change, but I used it for simplicity.</p>

<p>Let's quickly overview what we have with components. It starts in <code>ui/main.tsx</code> rendering root component with all providers and the main app component inside.<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight tsx"><code><span class="nf">createRoot</span><span class="p">(</span><span class="nb">document</span><span class="p">.</span><span class="nf">getElementById</span><span class="p">(</span><span class="dl">'</span><span class="s1">root</span><span class="dl">'</span><span class="p">)</span><span class="o">!</span><span class="p">).</span><span class="nf">render</span><span class="p">(</span>
  <span class="p">&lt;</span><span class="nc">AppProvider</span><span class="p">&gt;</span>
    <span class="p">&lt;</span><span class="nc">UIProvider</span><span class="p">&gt;</span>
      <span class="p">&lt;</span><span class="nc">BrowserRouter</span><span class="p">&gt;</span>
        <span class="p">&lt;</span><span class="nc">MantineProvider</span><span class="p">&gt;</span>
          <span class="p">&lt;</span><span class="nc">Notifications</span> <span class="na">position</span><span class="p">=</span><span class="s">"top-center"</span> <span class="p">/&gt;</span>
          <span class="p">&lt;</span><span class="nc">AppComponent</span> <span class="p">/&gt;</span>
        <span class="p">&lt;/</span><span class="nc">MantineProvider</span><span class="p">&gt;</span>
      <span class="p">&lt;/</span><span class="nc">BrowserRouter</span><span class="p">&gt;</span>
    <span class="p">&lt;/</span><span class="nc">UIProvider</span><span class="p">&gt;</span>
  <span class="p">&lt;/</span><span class="nc">AppProvider</span><span class="p">&gt;</span>
<span class="p">)</span>
</code></pre>

</div>



<p>The App component is in <code>ui/containers/App.tsx</code> file that sets up routing per page, layout, and adds modals. Also, it calls <code>useLastMinuteDealsWatch()</code> hook that reacts on new last minute deals and shows notifications.<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight tsx"><code><span class="k">export</span> <span class="k">default</span> <span class="kd">function</span> <span class="nf">App</span><span class="p">()</span> <span class="p">{</span>
  <span class="nf">useLastMinuteDealsWatch</span><span class="p">()</span>

  <span class="k">return </span><span class="p">(</span>
    <span class="p">&lt;</span><span class="nc">SessionLayout</span><span class="p">&gt;</span>
      <span class="p">&lt;</span><span class="nc">Routes</span><span class="p">&gt;</span>
        <span class="p">&lt;</span><span class="nc">Route</span> <span class="na">index</span> <span class="na">element</span><span class="p">=</span><span class="si">{</span><span class="p">&lt;</span><span class="nc">HomePage</span> <span class="p">/&gt;</span><span class="si">}</span> <span class="p">/&gt;</span>
        <span class="p">&lt;</span><span class="nc">Route</span> <span class="na">path</span><span class="p">=</span><span class="s">"/profile"</span> <span class="na">element</span><span class="p">=</span><span class="si">{</span><span class="p">&lt;</span><span class="nc">ProfilePage</span> <span class="p">/&gt;</span><span class="si">}</span> <span class="p">/&gt;</span>
      <span class="p">&lt;/</span><span class="nc">Routes</span><span class="p">&gt;</span>

      <span class="p">&lt;</span><span class="nc">LoginModal</span> <span class="p">/&gt;</span>
      <span class="p">&lt;</span><span class="nc">RenameModal</span> <span class="p">/&gt;</span>
      <span class="p">&lt;</span><span class="nc">CongratulationsModal</span> <span class="p">/&gt;</span>
    <span class="p">&lt;/</span><span class="nc">SessionLayout</span><span class="p">&gt;</span>
  <span class="p">)</span>
<span class="p">}</span>
</code></pre>

</div>



<p>Every page simply renders its containers, containers use components and provide them with callbacks to trigger actions wrapped with <code>try/catch</code> to show error notifications in case of failures.<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight tsx"><code><span class="kd">function</span> <span class="nf">useSearchCallback</span><span class="p">()</span> <span class="p">{</span>
  <span class="kd">const</span> <span class="p">{</span> <span class="nx">api</span> <span class="p">}</span> <span class="o">=</span> <span class="nf">useAppContext</span><span class="p">()</span>
  <span class="kd">const</span> <span class="p">{</span> <span class="nx">notify</span> <span class="p">}</span> <span class="o">=</span> <span class="nf">useUIContext</span><span class="p">()</span>
  <span class="kd">const</span> <span class="nx">navigate</span> <span class="o">=</span> <span class="nf">useNavigate</span><span class="p">()</span>

  <span class="k">return</span> <span class="nf">useCallback</span><span class="p">(</span>
    <span class="k">async </span><span class="p">(</span><span class="nx">criteria</span><span class="p">:</span> <span class="nx">SearchCriteria</span><span class="p">)</span> <span class="o">=&gt;</span> <span class="p">{</span>
      <span class="k">try</span> <span class="p">{</span>
        <span class="k">await</span> <span class="nx">api</span><span class="p">.</span><span class="nx">travel</span><span class="p">.</span><span class="nf">searchTravels</span><span class="p">(</span><span class="nx">criteria</span><span class="p">)</span>
        <span class="nf">navigate</span><span class="p">(</span><span class="dl">'</span><span class="s1">/</span><span class="dl">'</span><span class="p">)</span>
      <span class="p">}</span> <span class="k">catch </span><span class="p">(</span><span class="nx">e</span><span class="p">)</span> <span class="p">{</span>
        <span class="nf">notify</span><span class="p">({</span>
          <span class="na">color</span><span class="p">:</span> <span class="dl">'</span><span class="s1">red</span><span class="dl">'</span><span class="p">,</span>
          <span class="na">title</span><span class="p">:</span> <span class="dl">'</span><span class="s1">Search error</span><span class="dl">'</span><span class="p">,</span>
          <span class="na">message</span><span class="p">:</span> <span class="p">(</span><span class="nx">e</span> <span class="k">as</span> <span class="nb">Error</span><span class="p">).</span><span class="nx">message</span><span class="p">,</span>
        <span class="p">})</span>
      <span class="p">}</span>
    <span class="p">},</span>
    <span class="p">[</span><span class="nx">app</span><span class="p">,</span> <span class="nx">navigate</span><span class="p">,</span> <span class="nx">notify</span><span class="p">]</span>
  <span class="p">)</span>
<span class="p">}</span>

<span class="k">export</span> <span class="k">default</span> <span class="kd">function</span> <span class="nf">RecentSearches</span><span class="p">()</span> <span class="p">{</span>
  <span class="kd">const</span> <span class="p">{</span> <span class="nx">travel</span> <span class="p">}</span> <span class="o">=</span> <span class="nf">useAppContext</span><span class="p">()</span>
  <span class="kd">const</span> <span class="nx">recentSearches</span> <span class="o">=</span> <span class="nx">travel</span><span class="p">.</span><span class="nf">useRecentSearches</span><span class="p">()</span>
  <span class="kd">const</span> <span class="nx">search</span> <span class="o">=</span> <span class="nf">useSearchCallback</span><span class="p">()</span>

  <span class="k">return </span><span class="p">(</span>
    <span class="p">&lt;</span><span class="nt">div</span><span class="p">&gt;</span>
      <span class="si">{</span><span class="nx">recentSearches</span><span class="p">.</span><span class="nx">length</span> <span class="o">&gt;</span> <span class="mi">0</span> <span class="o">&amp;&amp;</span> <span class="p">&lt;</span><span class="nt">h2</span><span class="p">&gt;</span>Recent Searches<span class="p">&lt;/</span><span class="nt">h2</span><span class="p">&gt;</span><span class="si">}</span>
      <span class="p">&lt;</span><span class="nc">Grid</span> <span class="na">className</span><span class="p">=</span><span class="si">{</span><span class="nx">classes</span><span class="p">.</span><span class="nx">grid</span><span class="si">}</span> <span class="na">gutter</span><span class="p">=</span><span class="s">"md"</span><span class="p">&gt;</span>
        <span class="si">{</span><span class="nx">recentSearches</span><span class="p">.</span><span class="nf">map</span><span class="p">((</span><span class="nx">recentSearch</span><span class="p">,</span> <span class="nx">index</span><span class="p">)</span> <span class="o">=&gt;</span> <span class="p">(</span>
          <span class="p">&lt;</span><span class="nc">Grid</span><span class="p">.</span><span class="nc">Col</span> <span class="na">key</span><span class="p">=</span><span class="si">{</span><span class="nx">index</span><span class="si">}</span> <span class="na">span</span><span class="p">=</span><span class="si">{</span><span class="p">{</span> <span class="na">base</span><span class="p">:</span> <span class="mi">12</span><span class="p">,</span> <span class="na">xs</span><span class="p">:</span> <span class="mi">6</span><span class="p">,</span> <span class="na">md</span><span class="p">:</span> <span class="mi">3</span> <span class="p">}</span><span class="si">}</span><span class="p">&gt;</span>
            <span class="p">&lt;</span><span class="nc">RecentSearch</span>
              <span class="na">value</span><span class="p">=</span><span class="si">{</span><span class="nx">recentSearch</span><span class="si">}</span>
              <span class="na">onClick</span><span class="p">=</span><span class="si">{</span><span class="p">()</span> <span class="o">=&gt;</span> <span class="nf">search</span><span class="p">(</span><span class="nx">recentSearch</span><span class="p">)</span><span class="si">}</span>
            <span class="p">/&gt;</span>
          <span class="p">&lt;/</span><span class="nc">Grid</span><span class="p">.</span><span class="nc">Col</span><span class="p">&gt;</span>
        <span class="p">))</span><span class="si">}</span>
      <span class="p">&lt;/</span><span class="nc">Grid</span><span class="p">&gt;</span>
    <span class="p">&lt;/</span><span class="nt">div</span><span class="p">&gt;</span>
  <span class="p">)</span>
<span class="p">}</span>
</code></pre>

</div>



<p>As you see it just performs a usual call to <code>api.travel.searchTravels()</code> method like you can do in the browser console as well, no magic. The container with travel cards will eventually get cards via <code>useTravelCards</code> hook.</p>

<p>Now you can run the app with <code>npm run dev</code> command (don't forget <code>npm install</code> if not installed yet) and see the result. Open browser console and try typing <code>api.travel.addLastMinuteDeal()</code> to see how notification appears.</p>

<h2>
  
  
  Application
</h2>

<p>In a big application where a lot needs to be processed and calculated in the background and a lot of sophisticated UI needs to be delivered there can be separate developers for both parts.<br>
They only need to agree on the contracts. And UI have already provided us with something. Very nice when application can do the same.</p>

<p>Ok. So now we're application developers. We have nothing to do with the UI, at this stage we don't even know what library is to be used. All we need is to provide API.</p>

<p>When creating application we are not limited by any library constraints and can use any architecture we want. Just remember about clear boundaries between different parts of the application. I suggest to create a DDD inspired implementation.</p>

<p>We have absolutely the same task description as before, and by the same logic we decide what bounded contexts we need:</p>

<ul>
<li>
<strong>Auth context</strong> - to manage user authentication and session</li>
<li>
<strong>Travel context</strong> - to manage travel options, searches, purchases, and last minute deals</li>
</ul>

<p>But since we're not going to have any backend let's also add one more context:</p>

<ul>
<li>
<strong>Admin context</strong> - to manage travel data</li>
</ul>

<p>Create <code>src/app/</code> folder for our application, and the following folders inside it:</p>

<ul>
<li>
<code>contexts/</code> - for the mentioned bounded contexts</li>
<li>
<code>shared/</code> - for things shared between contexts</li>
</ul>

<p>And I want to start with <strong>Admin context</strong> as it will provide data for the rest of the application.</p>
<h3>
  
  
  Admin context
</h3>

<p>Create <code>src/app/contexts/admin/</code> folder that will be home for the context.</p>

<p>In the new context folder immediately create the following sub-folders:</p>

<ul>
<li>
<code>domain/</code> - to put objects and entities defining the domain</li>
<li>
<code>application/</code> - to put application services and use cases orchestrating workflows</li>
<li>
<code>infrastructure/</code> - to put infrastructure implementations and different API integrations</li>
<li>
<code>interface/</code> - for providing context interface API</li>
</ul>
<h4>
  
  
  Domain layer
</h4>

<p>In the admin context I decided to define the following domain objects:</p>

<ul>
<li>
<code>Offer</code> - an entity that includes id and other travel card details</li>
<li>
<code>SpecialOffer</code> - an entity that includes id, special price, description, and reference to the related offer</li>
</ul>

<p>Also I've added the following domain object classes:</p>

<ul>
<li>
<code>OfferDraft</code> - to represent a draft of an offer to be created</li>
<li>
<code>SpecialOfferDraft</code> - to represent a draft of a special offer to be created</li>
</ul>

<p>The draft classes are bases for the existing entities classes.<br>
To create a new offer draft a factory method <code>OfferDraft::create(props)</code> is defined and for an existing offer there is <code>Offer::rehydrate(props)</code> method. Same with special offers.</p>

<p><code>SpecialOfferDraft</code>:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight typescript"><code><span class="k">export</span> <span class="kd">type</span> <span class="nx">SpecialOfferDraftProps</span> <span class="o">=</span> <span class="p">{</span>
  <span class="na">offerId</span><span class="p">:</span> <span class="kr">string</span>
  <span class="na">specialPrice</span><span class="p">:</span> <span class="kr">number</span>
  <span class="na">description</span><span class="p">:</span> <span class="kr">string</span>
<span class="p">}</span>

<span class="k">export</span> <span class="k">default</span> <span class="kd">class</span> <span class="nc">SpecialOfferDraft</span><span class="o">&lt;</span>
  <span class="nx">T</span> <span class="kd">extends</span> <span class="nx">SpecialOfferDraftProps</span> <span class="o">=</span> <span class="nx">SpecialOfferDraftProps</span><span class="p">,</span>
<span class="o">&gt;</span> <span class="p">{</span>
  <span class="k">protected</span> <span class="na">_props</span><span class="p">:</span> <span class="nx">T</span>

  <span class="k">public</span> <span class="kd">get</span> <span class="nf">offerId</span><span class="p">()</span> <span class="p">{</span>
    <span class="k">return</span> <span class="k">this</span><span class="p">.</span><span class="nx">_props</span><span class="p">.</span><span class="nx">offerId</span>
  <span class="p">}</span>

  <span class="k">public</span> <span class="kd">get</span> <span class="nf">specialPrice</span><span class="p">()</span> <span class="p">{</span>
    <span class="k">return</span> <span class="k">this</span><span class="p">.</span><span class="nx">_props</span><span class="p">.</span><span class="nx">specialPrice</span>
  <span class="p">}</span>

  <span class="k">public</span> <span class="kd">get</span> <span class="nf">description</span><span class="p">()</span> <span class="p">{</span>
    <span class="k">return</span> <span class="k">this</span><span class="p">.</span><span class="nx">_props</span><span class="p">.</span><span class="nx">description</span>
  <span class="p">}</span>
  <span class="k">protected</span> <span class="nf">constructor</span><span class="p">(</span><span class="na">props</span><span class="p">:</span> <span class="nx">T</span><span class="p">)</span> <span class="p">{</span>
    <span class="k">this</span><span class="p">.</span><span class="nx">_props</span> <span class="o">=</span> <span class="p">{</span> <span class="p">...</span><span class="nx">props</span> <span class="p">}</span>
  <span class="p">}</span>
  <span class="k">public</span> <span class="k">static</span> <span class="nf">create</span><span class="p">(</span><span class="na">props</span><span class="p">:</span> <span class="nx">SpecialOfferDraftProps</span><span class="p">)</span> <span class="p">{</span>
    <span class="k">return</span> <span class="k">new</span> <span class="nc">SpecialOfferDraft</span><span class="p">({</span> <span class="p">...</span><span class="nx">props</span> <span class="p">})</span>
  <span class="p">}</span>
<span class="p">}</span>
</code></pre>

</div>



<p><code>SpecialOffer</code>:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight typescript"><code><span class="k">export</span> <span class="kd">type</span> <span class="nx">SpecialOfferProps</span> <span class="o">=</span> <span class="nx">SpecialOfferDraftProps</span> <span class="o">&amp;</span> <span class="p">{</span>
  <span class="na">id</span><span class="p">:</span> <span class="kr">string</span>
<span class="p">}</span>

<span class="k">export</span> <span class="k">default</span> <span class="kd">class</span> <span class="nc">SpecialOffer</span> <span class="kd">extends</span> <span class="nc">SpecialOfferDraft</span><span class="o">&lt;</span><span class="nx">SpecialOfferProps</span><span class="o">&gt;</span> <span class="p">{</span>
  <span class="k">public</span> <span class="kd">get</span> <span class="nf">id</span><span class="p">()</span> <span class="p">{</span>
    <span class="k">return</span> <span class="k">this</span><span class="p">.</span><span class="nx">_props</span><span class="p">.</span><span class="nx">id</span>
  <span class="p">}</span>

  <span class="k">public</span> <span class="nf">patch</span><span class="p">(</span><span class="na">props</span><span class="p">:</span> <span class="nb">Partial</span><span class="o">&lt;</span><span class="nb">Omit</span><span class="o">&lt;</span><span class="nx">SpecialOfferProps</span><span class="p">,</span> <span class="dl">'</span><span class="s1">id</span><span class="dl">'</span><span class="o">&gt;&gt;</span><span class="p">)</span> <span class="p">{</span>
    <span class="k">this</span><span class="p">.</span><span class="nx">_props</span> <span class="o">=</span> <span class="p">{</span> <span class="p">...</span><span class="k">this</span><span class="p">.</span><span class="nx">_props</span><span class="p">,</span> <span class="p">...</span><span class="nx">props</span> <span class="p">}</span>
  <span class="p">}</span>

  <span class="k">public</span> <span class="k">static</span> <span class="nf">rehydrate</span><span class="p">(</span><span class="na">props</span><span class="p">:</span> <span class="nx">SpecialOfferProps</span><span class="p">)</span> <span class="p">{</span>
    <span class="k">return</span> <span class="k">new</span> <span class="nc">SpecialOffer</span><span class="p">(</span><span class="nx">props</span><span class="p">)</span>
  <span class="p">}</span>
<span class="p">}</span>
</code></pre>

</div>



<p>Why classes and not just interfaces? Because I want to add behaviour to these objects. They can have updaters, validators, various policies. And this way I can be sure that my domain objects are always valid and consistent.</p>

<p>Now as the domain is ready, we can switch to the application layer and define what our context is capable of.</p>

<h4>
  
  
  Application layer
</h4>

<p>In the <code>application/services/</code> folder I've created two files:</p>

<ul>
<li>
<code>offers-service.ts</code> - with <code>OffersService</code> class to manage offers</li>
<li>
<code>special-offers-service.ts</code> - with <code>SpecialOffersService</code> class to manage special offers</li>
</ul>

<p>Let's take a look at <code>OffersService</code> class.</p>

<p>I decided to allow the following operations:</p>

<ul>
<li>Get all offers</li>
<li>Get offer by ID</li>
<li>Add a new offer</li>
<li>Update an existing offer</li>
</ul>

<p>This is why <code>OffersService</code> class provides the following methods:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight typescript"><code><span class="kd">class</span> <span class="nc">OffersService</span> <span class="p">{</span>
  <span class="k">public</span> <span class="k">async</span> <span class="nf">getAll</span><span class="p">():</span> <span class="nb">Promise</span><span class="o">&lt;</span><span class="nx">OfferView</span><span class="p">[]</span><span class="o">&gt;</span> <span class="p">{</span>
    <span class="cm">/*...*/</span>
  <span class="p">}</span>
  <span class="k">public</span> <span class="k">async</span> <span class="nf">getById</span><span class="p">(</span><span class="nx">id</span><span class="p">:</span> <span class="kr">string</span><span class="p">):</span> <span class="nb">Promise</span><span class="o">&lt;</span><span class="nx">OfferView</span> <span class="o">|</span> <span class="kc">null</span><span class="o">&gt;</span> <span class="p">{</span>
    <span class="cm">/*...*/</span>
  <span class="p">}</span>
  <span class="k">public</span> <span class="k">async</span> <span class="nf">add</span><span class="p">(</span><span class="nx">command</span><span class="p">:</span> <span class="nx">AddOfferCommand</span><span class="p">):</span> <span class="nb">Promise</span><span class="o">&lt;</span><span class="nx">OfferView</span><span class="o">&gt;</span> <span class="p">{</span>
    <span class="cm">/*...*/</span>
  <span class="p">}</span>
  <span class="k">public</span> <span class="k">async</span> <span class="nf">update</span><span class="p">(</span><span class="nx">command</span><span class="p">:</span> <span class="nx">UpdateOfferCommand</span><span class="p">):</span> <span class="nb">Promise</span><span class="o">&lt;</span><span class="nx">OfferView</span><span class="o">&gt;</span> <span class="p">{</span>
    <span class="cm">/*...*/</span>
  <span class="p">}</span>
<span class="p">}</span>
</code></pre>

</div>



<p>And to implement all these methods I need a repository to store and manage offers. So I've created a port in <code>application/ports/offers-repository.ts</code> file with <code>OffersRepository</code> interface defining methods for managing offers in a store:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight typescript"><code><span class="k">export</span> <span class="k">default</span> <span class="kr">interface</span> <span class="nx">OffersRepository</span> <span class="p">{</span>
  <span class="nf">findAll</span><span class="p">():</span> <span class="nb">Promise</span><span class="o">&lt;</span><span class="nx">Offer</span><span class="p">[]</span><span class="o">&gt;</span>
  <span class="nf">findById</span><span class="p">(</span><span class="nx">id</span><span class="p">:</span> <span class="kr">string</span><span class="p">):</span> <span class="nb">Promise</span><span class="o">&lt;</span><span class="nx">Offer</span> <span class="o">|</span> <span class="kc">null</span><span class="o">&gt;</span>
  <span class="nf">findByIds</span><span class="p">(</span><span class="nx">ids</span><span class="p">:</span> <span class="kr">string</span><span class="p">[]):</span> <span class="nb">Promise</span><span class="o">&lt;</span><span class="nx">Offer</span><span class="p">[]</span><span class="o">&gt;</span>
  <span class="nf">add</span><span class="p">(</span><span class="nx">offer</span><span class="p">:</span> <span class="nx">OfferDraft</span><span class="p">):</span> <span class="nb">Promise</span><span class="o">&lt;</span><span class="nx">Offer</span><span class="o">&gt;</span>
  <span class="nf">update</span><span class="p">(</span><span class="nx">offer</span><span class="p">:</span> <span class="nx">Offer</span><span class="p">):</span> <span class="nb">Promise</span><span class="o">&lt;</span><span class="nx">Offer</span><span class="o">&gt;</span>
  <span class="nf">clear</span><span class="p">():</span> <span class="nb">Promise</span><span class="o">&lt;</span><span class="k">void</span><span class="o">&gt;</span>
<span class="p">}</span>
</code></pre>

</div>



<p>And injected this dependency into the service constructor.<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight typescript"><code><span class="k">export</span> <span class="k">default</span> <span class="kd">class</span> <span class="nc">OffersService</span> <span class="p">{</span>
  <span class="k">public</span> <span class="nf">constructor</span><span class="p">(</span><span class="k">private</span> <span class="nx">offersRepository</span><span class="p">:</span> <span class="nx">OffersRepository</span><span class="p">)</span> <span class="p">{}</span>
  <span class="c1">//...</span>
<span class="p">}</span>
</code></pre>

</div>



<p>Now the service can be implemented.</p>

<p>Actually there are many debates on where repositories should live, application or domain, but to my understanding persistence is not part of domain. I prefer to keep them in the application layer next to other ports like providers and gateways.</p>

<h4>
  
  
  ACL
</h4>

<p>One more important element is the Anti-Corruption Layer (ACL), which acts as a protective barrier between layers.</p>

<p>I'm sure you've noticed <code>OfferView</code> in <code>OffersService</code>. This interface is defined in <code>application/acl/</code>. Why not simply <code>Offer</code> which is defined in the domain?<br>
This is an example of separation of concerns. Domain layer should always stay isolated from outside world, feeling cozy and warm. <code>OfferView</code> provides an API-friendly representation of <code>Offer</code> built from simple types.</p>

<p>The client uses exposed API from application layer and provides instances of <code>OfferView</code> to the API, not caring about domain details, then application layer needs instances of <code>Offer</code> for domain logic and ports, and to orchestrate the flow. After some processing application layer having instances of <code>Offer</code> needs to return instances of <code>OfferView</code> back to the client. This is where ACL comes into play. It may simply convert plain view objects to domain class instances and back, also it can handle more complex processing with validations and throwing errors.</p>

<p>And in the application we usually have ACL for application-domain and for domain-infrastructure mapping.</p>
<h4>
  
  
  Infrastructure layer
</h4>

<p>And again, why not just a normal class <code>OffersRepository</code> doing all the work with local storage? The answer is: because I want to decouple the business logic from storage implementation details. This way I'm free to use my <code>OffersService</code> with local storage, in-memory storage for testing, backend API based on REST or GraphQL, or anything else. And I add implementations in the infrastructure layer in <code>infrastructure/</code> folder. I've put local storage adapter in <code>infrastructure/local-storage/offers-repository.ts</code> file.</p>

<p>And here we find ACL too in <code>infrastructure/local-storage/acl/</code>. The only change is in the terminology as we now have DTOs (Data Transfer Objects). Need to map domain object <code>Offer</code> to a value that adapter's storage can consume - <code>OfferDTO</code>.</p>
<h4>
  
  
  D
</h4>

<p>Another thing worth mentioning is how the local storage adapter is implemented. Although it's intended to work with concrete local storage, we still inject a generic storage interface dependency. This way it's much easier to test the class even when no local storage is available.</p>
<h4>
  
  
  Errors
</h4>

<p>I don't like returning error codes in various flows and prefer using exceptions. I think it's the cleanest way to inform about errors, to know about errors, and to clean up after errors. Errors related to application flow are defined in <code>application/errors/</code> folder.</p>
<h4>
  
  
  Interface
</h4>

<p>Finally, to provide the context interface API we need to create <code>interface/api.ts</code> file that will compose API from application objects that we want to expose.</p>
<h4>
  
  
  Tests
</h4>

<p>To make sure everything works fine while working on the context implementation I created a number of tests in <code>tests/app/contexts/admin/</code> folder. Please check them and run them.</p>
<h3>
  
  
  Auth context
</h3>

<p>Now we can move to <strong>Auth context</strong> in <code>src/app/contexts/auth/</code> folder. Similarly create the same structure of sub-folders.</p>

<p>But after analyzing we find out there's no domain in this context. And this is fine because this context only relies on various gateways and providers APIs. Nothing to own.</p>

<p>This time I decided to implement use-cases instead of services. They are defined in <code>application/use-cases/</code> folder. They are different from services in that they are single purpose classes that perform one action only. Let's take a look at the file <code>restore-user.ts</code>.<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight typescript"><code><span class="k">export</span> <span class="k">default</span> <span class="kd">class</span> <span class="nc">RestoreUser</span> <span class="p">{</span>
  <span class="k">public</span> <span class="nf">constructor</span><span class="p">(</span>
    <span class="k">private</span> <span class="nx">sessionProvider</span><span class="p">:</span> <span class="nx">SessionProvider</span><span class="p">,</span>
    <span class="k">private</span> <span class="nx">sessionStore</span><span class="p">:</span> <span class="nx">SessionStore</span>
  <span class="p">)</span> <span class="p">{}</span>

  <span class="k">public</span> <span class="k">async</span> <span class="nf">execute</span><span class="p">()</span> <span class="p">{</span>
    <span class="kd">const</span> <span class="nx">session</span> <span class="o">=</span> <span class="k">await</span> <span class="k">this</span><span class="p">.</span><span class="nx">sessionProvider</span><span class="p">.</span><span class="nf">restore</span><span class="p">()</span>

    <span class="k">if </span><span class="p">(</span><span class="nx">session</span><span class="p">)</span> <span class="p">{</span>
      <span class="k">this</span><span class="p">.</span><span class="nx">sessionStore</span><span class="p">.</span><span class="nf">setSession</span><span class="p">({</span>
        <span class="na">email</span><span class="p">:</span> <span class="nx">session</span><span class="p">.</span><span class="nx">email</span><span class="p">,</span>
        <span class="na">name</span><span class="p">:</span> <span class="nx">session</span><span class="p">.</span><span class="nx">name</span><span class="p">,</span>
      <span class="p">})</span>
    <span class="p">}</span> <span class="k">else</span> <span class="p">{</span>
      <span class="k">this</span><span class="p">.</span><span class="nx">sessionStore</span><span class="p">.</span><span class="nf">setSession</span><span class="p">(</span><span class="kc">null</span><span class="p">)</span>
    <span class="p">}</span>
  <span class="p">}</span>
<span class="p">}</span>
</code></pre>

</div>



<p>It contains <code>RestoreUser</code> use-case class with <code>execute()</code> method only which performs the only task to restore user session. All dependencies are injected via constructor and they are all abstract. This way we again have a clear separation of concerns, and each use-case is easy to test.</p>

<p>Of course, <code>execute</code> method can have parameters same as methods in services, and also this is the place where all validation of input data should happen. For example, in <code>LoginUser</code> use-case:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight typescript"><code><span class="k">public</span> <span class="k">async</span> <span class="nf">execute</span><span class="p">(</span><span class="nx">command</span><span class="p">:</span> <span class="nx">LoginUserCommand</span><span class="p">)</span> <span class="p">{</span>
    <span class="nf">assertLoginUserCommand</span><span class="p">(</span><span class="nx">command</span><span class="p">)</span>
    <span class="c1">// ...</span>
<span class="p">}</span>
</code></pre>

</div>



<p>Here <code>assertLoginUserCommand</code> function is used to check both <code>email</code> and <code>password</code> fields are provided and not empty.</p>

<p>In this context we needed to add two ports in <code>application/ports/</code> folder:</p>

<ul>
<li>
<code>SessionProvider</code> - to save and restore sessions between reloads</li>
<li>
<code>AuthGateway</code> - to communicate with auth API</li>
</ul>

<p>Also added <code>application/contracts/</code> folder to define types and commands that the context uses with external APIs. There's no need to add ACL here, nothing to isolate.</p>

<p>Similarly to the admin context, in the infrastructure layer we provide implementations for the defined ports.</p>

<p>Finally, created <code>interface/api.ts</code> file to compose the context API.</p>

<p>And of course tests are in <code>tests/app/contexts/auth/</code> folder.</p>

<h3>
  
  
  Travel context
</h3>

<p>Now the most interesting context in our demo - <strong>Travel context</strong> in <code>src/app/contexts/travel/</code> folder. Again, create the same structure of sub-folders.</p>

<p>This context has something to look at in the domain layer.</p>

<h4>
  
  
  Domain layer
</h4>

<p>We have <code>SearchCriteria</code> value object. And for this object we added policies making sure the created search criteria is valid.</p>

<p>The base policy interface:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight typescript"><code><span class="k">export</span> <span class="k">default</span> <span class="kr">interface</span> <span class="nx">SearchCriteriaPolicy</span> <span class="p">{</span>
  <span class="nf">validate</span><span class="p">(</span><span class="nx">props</span><span class="p">:</span> <span class="nx">SearchCriteriaProps</span><span class="p">):</span> <span class="k">void</span>
<span class="p">}</span>
</code></pre>

</div>



<p>and implemented <code>DistinctOriginDestinationPolicy</code> to check <code>from</code> and <code>to</code> locations are different. The policy runs when <code>SearchCriteria</code> is created:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight typescript"><code><span class="k">public</span> <span class="k">static</span> <span class="nf">create</span><span class="p">(</span>
  <span class="nx">props</span><span class="p">:</span> <span class="nx">SearchCriteriaProps</span><span class="p">,</span>
  <span class="nx">policy</span><span class="p">:</span> <span class="nx">SearchCriteriaPolicy</span> <span class="o">=</span> <span class="k">new</span> <span class="nc">DistinctOriginDestinationPolicy</span><span class="p">()</span>
<span class="p">)</span> <span class="p">{</span>
  <span class="nx">policy</span><span class="p">.</span><span class="nf">validate</span><span class="p">(</span><span class="nx">props</span><span class="p">)</span>
  <span class="k">return</span> <span class="k">new</span> <span class="nc">SearchCriteria</span><span class="p">({</span> <span class="p">...</span><span class="nx">props</span> <span class="p">})</span>
<span class="p">}</span>
</code></pre>

</div>



<p>We also have <code>TravelInfo</code> value object to contain info about a travel, and <code>PurchasedTravel</code> entity with <code>info: TravelInfo</code> property containing information about the purchased travel.</p>

<h4>
  
  
  Application layer
</h4>

<p>In the application layer we have the following use-cases:</p>

<ul>
<li>
<code>SearchTravels</code> - searches for travels based on criteria, and gets travel cards</li>
<li>
<code>GetRecentSearches</code> - gets all recent searches</li>
<li>
<code>GetLastMinuteDeals</code> - gets all last-minute deals</li>
<li>
<code>PurchaseTravel</code> - handles the purchase of a travel</li>
<li>
<code>PurchaseLastMinuteDeal</code> - handles the purchase of a last-minute deal</li>
<li>
<code>RenamePurchasedTravel</code> - renames a purchased travel</li>
<li>
<code>GetPurchasedTravels</code> - gets all purchased travels</li>
</ul>

<p>Also added a service <code>LastMinuteDealsWatch</code>. The purpose of the service is to periodically check <code>TravelsProvider</code> for new last-minute deals.</p>

<p>Please also check <code>application/ports/</code> folder for the defined ports:</p>

<ul>
<li>
<code>BookingProvider</code> - to purchase travels and last-minute deals</li>
<li>
<code>TravelsProvider</code> - to get information about travels, last-minute deals, and recent searches</li>
<li>
<code>PurchasedTravelsRepository</code> - to manage purchased travels</li>
</ul>

<p>And also check how ACL is implemented. <code>PurchasedTravel</code> is a domain entity with nested <code>TravelInfo</code> value object, and is mapped to/from a plain object, very convenient for API clients, who don't need any domain details.</p>

<h4>
  
  
  Infrastructure layer
</h4>

<p>Please check <code>infrastructure/</code> folder for all implementations.<br>
There's nothing special.<br>
Only one thing that I want to highlight is how the data is stored.<br>
We know nothing about Admin context and this is great, because contexts should know nothing about each other. This allows even to extract a context into a separate package.<br>
And for this reason to retrieve data about offers and special offers in <code>StorageDataProvider</code> I directly access storage by the same keys we used in the Admin context. And if you say this is so vulnerable, I would totally agree with you. Of course, should've been better to have shared repositories for managing offers and special offers injected in concretes used in Admin and Travel contexts, so if you like the topic you can try implementing it.</p>
<h4>
  
  
  Interface
</h4>

<p>The API is exposed in <code>interface/api.ts</code> file.</p>
<h3>
  
  
  Composition root
</h3>

<p>All our services and use-cases in contexts depend only on abstractions. And now it's time to wire everything together and build complete API in the composition root.</p>

<p>For this purpose create <code>root/web-app.ts</code> file. And yes, in <code>root/</code> you can create different composition roots using different adapter for different purposes.</p>

<p>Please check <code>web-app.ts</code> in the repository to see how the API is composed.</p>

<p>First, we instantiate all infrastructure components, and then we compose the API using these components to make abstract real.</p>
<h3>
  
  
  State persistence
</h3>

<p>At this stage we have only API. So the client of the API should store received data somewhere else.<br>
This actually can work with our Travel App if we add a thick bridge, say with Redux, create slices, add thunks, and store received data in the slice's state.</p>

<p>But I want to make the application stateful. So I can have more control over access policies and state management.</p>

<p>Let's add stores to the contexts.</p>

<p>Auth context needs <code>SessionStore</code>:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight typescript"><code><span class="k">export</span> <span class="k">default</span> <span class="kr">interface</span> <span class="nx">SessionStore</span> <span class="p">{</span>
  <span class="nl">session</span><span class="p">:</span> <span class="nx">Session</span> <span class="o">|</span> <span class="kc">null</span>
  <span class="nx">status</span><span class="p">:</span> <span class="nx">AuthStatus</span>
  <span class="nf">setSession</span><span class="p">(</span><span class="nx">session</span><span class="p">:</span> <span class="nx">Session</span> <span class="o">|</span> <span class="kc">null</span><span class="p">):</span> <span class="k">void</span>
  <span class="nf">setStatus</span><span class="p">(</span><span class="nx">key</span><span class="p">:</span> <span class="kr">keyof</span> <span class="nx">AuthStatus</span><span class="p">,</span> <span class="nx">value</span><span class="p">:</span> <span class="nx">AuthStatus</span><span class="p">[</span><span class="k">typeof</span> <span class="nx">key</span><span class="p">]):</span> <span class="k">void</span>
<span class="p">}</span>
</code></pre>

</div>



<p>This way we can allow different parts of the code to know about the session state via providers, middlewares, and other means.</p>

<p>Similarly Travel context needs <code>TravelStore</code>, and <code>LastMinuteDealsStore</code> which I decided to separate from <code>TravelStore</code> to add some logic with new last-minute deals.</p>

<p>Cool. Now we can add these as dependencies to use-cases and services to allow them to update the state accordingly.</p>

<p>You should already have seen them in the code. For examples in <code>SearchTravels</code> use-case in the Travel context:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight typescript"><code><span class="k">this</span><span class="p">.</span><span class="nx">travelStore</span><span class="p">.</span><span class="nf">setStatus</span><span class="p">(</span><span class="dl">'</span><span class="s1">isLoadingCards</span><span class="dl">'</span><span class="p">,</span> <span class="kc">true</span><span class="p">)</span>
<span class="c1">// ...</span>
<span class="k">this</span><span class="p">.</span><span class="nx">travelStore</span><span class="p">.</span><span class="nf">setTravelCards</span><span class="p">(</span><span class="nx">travelCards</span><span class="p">)</span>
<span class="c1">// ...</span>
</code></pre>

</div>



<p>The only thing left is to implement these stores in the infrastructure layer. And before implementing them we need something to allow users of these stores to get informed about changes.</p>

<p>Let's add events mechanism. Events is a good way to orchestrate flows in the application and we'll do one orchestration as well. Also, looking ahead we'll use them to connect stores and hooks.</p>

<h3>
  
  
  Events
</h3>

<p>A simple event bus sufficient for the application can be defined as:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight typescript"><code><span class="k">export</span> <span class="kd">type</span> <span class="nx">Event</span> <span class="o">=</span> <span class="nx">object</span>

<span class="k">export</span> <span class="kd">type</span> <span class="nx">EventConstructor</span><span class="o">&lt;</span><span class="nx">T</span> <span class="kd">extends</span> <span class="nx">Event</span> <span class="o">=</span> <span class="nx">Event</span><span class="o">&gt;</span> <span class="o">=</span> <span class="p">(</span><span class="k">new </span><span class="p">(</span>
  <span class="p">...</span><span class="nx">args</span><span class="p">:</span> <span class="nx">never</span><span class="p">[]</span>
<span class="p">)</span> <span class="o">=&gt;</span> <span class="nx">T</span><span class="p">)</span> <span class="o">&amp;</span> <span class="p">{</span> <span class="na">id</span><span class="p">:</span> <span class="kr">string</span> <span class="p">}</span>

<span class="k">export</span> <span class="k">default</span> <span class="kr">interface</span> <span class="nx">EventBus</span> <span class="p">{</span>
  <span class="nf">publish</span><span class="p">(</span><span class="nx">event</span><span class="p">:</span> <span class="nx">Event</span><span class="p">):</span> <span class="k">void</span>

  <span class="nx">subscribe</span><span class="o">&lt;</span><span class="nx">T</span> <span class="kd">extends</span> <span class="nx">Event</span><span class="o">&gt;</span><span class="p">(</span>
    <span class="nx">eventClass</span><span class="p">:</span> <span class="nx">EventConstructor</span><span class="o">&lt;</span><span class="nx">T</span><span class="o">&gt;</span><span class="p">,</span>
    <span class="nx">listener</span><span class="p">:</span> <span class="p">(</span><span class="nx">event</span><span class="p">:</span> <span class="nb">InstanceType</span><span class="o">&lt;</span><span class="nx">EventConstructor</span><span class="o">&lt;</span><span class="nx">T</span><span class="o">&gt;&gt;</span><span class="p">)</span> <span class="o">=&gt;</span> <span class="k">void</span>
  <span class="p">):</span> <span class="p">()</span> <span class="o">=&gt;</span> <span class="k">void</span>
<span class="p">}</span>
</code></pre>

</div>



<p>This is an interface from <code>shared/ports/event-bus.ts</code>. And is implemented in <code>shared/infrastructure/event-emitter-event-bus.ts</code> using standard <code>EventEmitter</code> from <code>events</code>.</p>

<p>Now we can define an event in <code>application/events/</code> as a simple class with static <code>id: string</code> property identifying event type:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight typescript"><code><span class="k">export</span> <span class="k">default</span> <span class="kd">class</span> <span class="nc">PurchasedTravelAdded</span> <span class="p">{</span>
  <span class="k">public</span> <span class="k">static</span> <span class="nx">id</span> <span class="o">=</span> <span class="dl">'</span><span class="s1">Travel.PurchasedTravelAdded</span><span class="dl">'</span>
  <span class="k">public</span> <span class="nf">constructor</span><span class="p">(</span><span class="k">public</span> <span class="k">readonly</span> <span class="nx">purchasedTravel</span><span class="p">:</span> <span class="nx">PurchasedTravelView</span><span class="p">)</span> <span class="p">{}</span>
<span class="p">}</span>
</code></pre>

</div>



<p>then publish/emit it to the event bus:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight typescript"><code><span class="nx">eventBus</span><span class="p">.</span><span class="nf">publish</span><span class="p">(</span><span class="k">new</span> <span class="nc">PurchasedTravelAdded</span><span class="p">(</span><span class="nx">purchasedTravel</span><span class="p">))</span>
</code></pre>

</div>



<p>And to subscribe:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight typescript"><code><span class="nx">eventBus</span><span class="p">.</span><span class="nf">subscribe</span><span class="p">(</span><span class="nx">PurchasedTravelAdded</span><span class="p">,</span> <span class="p">(</span><span class="nx">event</span><span class="p">)</span> <span class="o">=&gt;</span> <span class="p">{</span>
  <span class="nx">console</span><span class="p">.</span><span class="nf">log</span><span class="p">(</span><span class="dl">'</span><span class="s1">Purchased travel added:</span><span class="dl">'</span><span class="p">,</span> <span class="nx">event</span><span class="p">.</span><span class="nx">purchasedTravel</span><span class="p">)</span>
<span class="p">})</span>
</code></pre>

</div>



<p>I prefer this over using string literals for events because by using application classes you get better type safety and better refactoring support.</p>

<p>Now let's create related events in Auth and Travel contexts and publish them when stores change like in the following store:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight typescript"><code><span class="k">export</span> <span class="k">default</span> <span class="kd">class</span> <span class="nc">InMemorySessionStore</span> <span class="k">implements</span> <span class="nx">SessionStore</span> <span class="p">{</span>
  <span class="k">private</span> <span class="nx">_session</span><span class="p">:</span> <span class="nx">Session</span> <span class="o">|</span> <span class="kc">null</span> <span class="o">=</span> <span class="kc">null</span>

  <span class="k">public</span> <span class="nf">constructor</span><span class="p">(</span><span class="k">private</span> <span class="nx">eventBus</span><span class="p">:</span> <span class="nx">EventBus</span><span class="p">)</span> <span class="p">{}</span>

  <span class="k">public</span> <span class="kd">get</span> <span class="nf">session</span><span class="p">():</span> <span class="nx">Session</span> <span class="o">|</span> <span class="kc">null</span> <span class="p">{</span>
    <span class="k">return</span> <span class="k">this</span><span class="p">.</span><span class="nx">_session</span>
  <span class="p">}</span>

  <span class="k">public</span> <span class="nf">setSession</span><span class="p">(</span><span class="nx">session</span><span class="p">:</span> <span class="nx">Session</span> <span class="o">|</span> <span class="kc">null</span><span class="p">):</span> <span class="k">void</span> <span class="p">{</span>
    <span class="k">this</span><span class="p">.</span><span class="nx">_session</span> <span class="o">=</span> <span class="nx">session</span>
    <span class="k">this</span><span class="p">.</span><span class="nx">eventBus</span><span class="p">.</span><span class="nf">publish</span><span class="p">(</span><span class="k">new</span> <span class="nc">SessionChanged</span><span class="p">(</span><span class="nx">session</span><span class="p">))</span>
  <span class="p">}</span>
<span class="p">}</span>
</code></pre>

</div>



<h3>
  
  
  Hooks
</h3>

<p>I think now you know how we're going to implement React hooks. Whenever the store changes, the event bus will notify the hooks, allowing them to update their React state accordingly.<br>
And for this pattern let's create a hook factory function:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight typescript"><code><span class="k">export</span> <span class="kd">function</span> <span class="nf">makeEventHook</span><span class="o">&lt;</span>
  <span class="nx">Args</span> <span class="kd">extends</span> <span class="nx">unknown</span><span class="p">[],</span>
  <span class="nx">T</span><span class="p">,</span>
  <span class="nx">Events</span> <span class="kd">extends</span> <span class="nx">EventConstructor</span><span class="o">&lt;</span><span class="nx">object</span><span class="o">&gt;</span><span class="p">[],</span>
<span class="o">&gt;</span><span class="p">(</span>
  <span class="nx">selector</span><span class="p">:</span> <span class="p">(</span><span class="nx">eventArg</span><span class="p">:</span> <span class="nb">InstanceType</span><span class="o">&lt;</span><span class="nx">Events</span><span class="p">[</span><span class="kr">number</span><span class="p">]</span><span class="o">&gt;</span> <span class="o">|</span> <span class="kc">null</span><span class="p">,</span> <span class="p">...</span><span class="nx">args</span><span class="p">:</span> <span class="nx">Args</span><span class="p">)</span> <span class="o">=&gt;</span> <span class="nx">T</span><span class="p">,</span>
  <span class="nx">eventBus</span><span class="p">:</span> <span class="nx">EventBus</span><span class="p">,</span>
  <span class="nx">events</span><span class="p">:</span> <span class="nx">Events</span>
<span class="p">)</span> <span class="p">{</span>
  <span class="k">return</span> <span class="nf">function </span><span class="p">(...</span><span class="nx">args</span><span class="p">:</span> <span class="nx">Args</span><span class="p">)</span> <span class="p">{</span>
    <span class="kd">const</span> <span class="p">[</span><span class="nx">state</span><span class="p">,</span> <span class="nx">setState</span><span class="p">]</span> <span class="o">=</span> <span class="nf">useState</span><span class="p">(</span><span class="nf">selector</span><span class="p">(</span><span class="kc">null</span><span class="p">,</span> <span class="p">...</span><span class="nx">args</span><span class="p">))</span>

    <span class="nf">useEffect</span><span class="p">(()</span> <span class="o">=&gt;</span> <span class="p">{</span>
      <span class="kd">const</span> <span class="nx">unsubscribe</span> <span class="o">=</span> <span class="nx">events</span><span class="p">.</span><span class="nf">map</span><span class="p">((</span><span class="nx">event</span><span class="p">)</span> <span class="o">=&gt;</span>
        <span class="nx">eventBus</span><span class="p">.</span><span class="nf">subscribe</span><span class="p">(</span><span class="nx">event</span><span class="p">,</span> <span class="p">(</span><span class="nx">eventArg</span><span class="p">)</span> <span class="o">=&gt;</span> <span class="p">{</span>
          <span class="nf">setState</span><span class="p">(</span><span class="nf">selector</span><span class="p">(</span><span class="nx">eventArg</span><span class="p">,</span> <span class="p">...</span><span class="nx">args</span><span class="p">))</span>
        <span class="p">})</span>
      <span class="p">)</span>

      <span class="k">return </span><span class="p">()</span> <span class="o">=&gt;</span> <span class="p">{</span>
        <span class="nx">unsubscribe</span><span class="p">.</span><span class="nf">forEach</span><span class="p">((</span><span class="nx">fn</span><span class="p">)</span> <span class="o">=&gt;</span> <span class="nf">fn</span><span class="p">())</span>
      <span class="p">}</span>
    <span class="p">},</span> <span class="p">[</span><span class="nx">setState</span><span class="p">,</span> <span class="p">...</span><span class="nx">args</span><span class="p">])</span>

    <span class="k">return</span> <span class="nx">state</span>
  <span class="p">}</span>
<span class="p">}</span>
</code></pre>

</div>



<p>It may look creepy at first. But if you watch a little longer, you'll find it is actually quite straightforward. We need to provide three arguments to this function:</p>

<ul>
<li>
<code>selector</code> - a function returning a value for the hook's state, it receives the triggering event object as the first argument or null when initializing the state, and may also receive a number of arguments if a hook is parameterized</li>
<li>
<code>eventBus</code> - the EventBus instance to subscribe to events from</li>
<li>
<code>events</code> - an array of event classes to listen for</li>
</ul>

<p>The returned hook will subscribe to the events, update the state accordingly, and unsubscribe when the component unmounts.</p>

<p>There are also two variants of this hook factory for array and object types. The reason for this is objects in a usual app never change their references, but React is based on immutability, this is why we need to create shallow copies for arrays and objects.</p>

<p>Now use these hook factories to compose API for React UI. And this is where we need to remember about the contracts from UI. We do it same way as for exposing context API in the <code>interface/</code> folder.</p>

<p>In <code>auth/interface/react/hooks.ts</code>:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight typescript"><code><span class="k">export</span> <span class="k">default</span> <span class="kd">function</span> <span class="nf">composeAuthHooks</span><span class="p">(</span>
  <span class="nx">sessionStore</span><span class="p">:</span> <span class="nx">SessionStore</span><span class="p">,</span>
  <span class="nx">eventBus</span><span class="p">:</span> <span class="nx">EventBus</span>
<span class="p">)</span> <span class="p">{</span>
  <span class="k">return</span> <span class="p">{</span>
    <span class="na">useSession</span><span class="p">:</span> <span class="nf">makeEventHookForObject</span><span class="p">(()</span> <span class="o">=&gt;</span> <span class="nx">sessionStore</span><span class="p">.</span><span class="nx">session</span><span class="p">,</span> <span class="nx">eventBus</span><span class="p">,</span> <span class="p">[</span>
      <span class="nx">SessionChanged</span><span class="p">,</span>
    <span class="p">]),</span>
    <span class="na">useStatus</span><span class="p">:</span> <span class="nf">makeEventHook</span><span class="p">(</span>
      <span class="p">(</span><span class="nx">_event</span><span class="p">,</span> <span class="na">key</span><span class="p">:</span> <span class="kr">keyof</span> <span class="nx">AuthStatus</span><span class="p">)</span> <span class="o">=&gt;</span> <span class="nx">sessionStore</span><span class="p">.</span><span class="nx">status</span><span class="p">[</span><span class="nx">key</span><span class="p">],</span>
      <span class="nx">eventBus</span><span class="p">,</span>
      <span class="p">[</span><span class="nx">SessionChanged</span><span class="p">]</span>
    <span class="p">),</span>
  <span class="p">}</span>
<span class="p">}</span>
</code></pre>

</div>



<p>And in <code>travel/interface/react/hooks.ts</code>:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight typescript"><code><span class="k">export</span> <span class="k">default</span> <span class="kd">function</span> <span class="nf">composeTravelHooks</span><span class="p">(</span>
  <span class="nx">travelStore</span><span class="p">:</span> <span class="nx">TravelStore</span><span class="p">,</span>
  <span class="nx">lastMinuteDealsStore</span><span class="p">:</span> <span class="nx">LastMinuteDealsStore</span><span class="p">,</span>
  <span class="nx">eventBus</span><span class="p">:</span> <span class="nx">EventBus</span>
<span class="p">)</span> <span class="p">{</span>
  <span class="k">return</span> <span class="p">{</span>
    <span class="na">useSearchCriteria</span><span class="p">:</span> <span class="nf">makeEventHookForObject</span><span class="p">(</span>
      <span class="p">()</span> <span class="o">=&gt;</span> <span class="nx">travelStore</span><span class="p">.</span><span class="nx">searchCriteria</span><span class="p">,</span>
      <span class="nx">eventBus</span><span class="p">,</span>
      <span class="p">[</span><span class="nx">SearchCriteriaChanged</span><span class="p">]</span>
    <span class="p">),</span>
    <span class="na">useTravelCards</span><span class="p">:</span> <span class="nf">makeEventHookForArray</span><span class="p">(</span>
      <span class="p">()</span> <span class="o">=&gt;</span> <span class="nx">travelStore</span><span class="p">.</span><span class="nx">travelCards</span><span class="p">,</span>
      <span class="nx">eventBus</span><span class="p">,</span>
      <span class="p">[</span><span class="nx">TravelCardsChanged</span><span class="p">]</span>
    <span class="p">),</span>
    <span class="na">usePurchasedTravels</span><span class="p">:</span> <span class="nf">makeEventHookForArray</span><span class="p">(</span>
      <span class="p">()</span> <span class="o">=&gt;</span> <span class="nx">travelStore</span><span class="p">.</span><span class="nx">purchasedTravels</span><span class="p">,</span>
      <span class="nx">eventBus</span><span class="p">,</span>
      <span class="p">[</span><span class="nx">PurchasedTravelsChanged</span><span class="p">,</span> <span class="nx">PurchasedTravelAdded</span><span class="p">,</span> <span class="nx">PurchasedTravelUpdated</span><span class="p">]</span>
    <span class="p">),</span>
    <span class="na">useRecentSearches</span><span class="p">:</span> <span class="nf">makeEventHookForArray</span><span class="p">(</span>
      <span class="p">()</span> <span class="o">=&gt;</span> <span class="nx">travelStore</span><span class="p">.</span><span class="nx">recentSearches</span><span class="p">,</span>
      <span class="nx">eventBus</span><span class="p">,</span>
      <span class="p">[</span><span class="nx">RecentSearchesChanged</span><span class="p">]</span>
    <span class="p">),</span>
    <span class="na">useLastMinuteDeals</span><span class="p">:</span> <span class="nf">makeEventHookForArray</span><span class="p">(</span>
      <span class="p">()</span> <span class="o">=&gt;</span> <span class="nx">lastMinuteDealsStore</span><span class="p">.</span><span class="nx">deals</span><span class="p">,</span>
      <span class="nx">eventBus</span><span class="p">,</span>
      <span class="p">[</span><span class="nx">LastMinuteDealsChanged</span><span class="p">,</span> <span class="nx">LastMinuteDealsAdded</span><span class="p">]</span>
    <span class="p">),</span>
    <span class="na">useNewLastMinuteDeals</span><span class="p">:</span> <span class="nf">makeEventHookForArray</span><span class="p">(</span>
      <span class="p">(</span><span class="nx">event</span><span class="p">)</span> <span class="o">=&gt;</span> <span class="nx">event</span><span class="p">?.</span><span class="nx">lastMinuteDeals</span> <span class="o">||</span> <span class="p">[],</span>
      <span class="nx">eventBus</span><span class="p">,</span>
      <span class="p">[</span><span class="nx">LastMinuteDealsAdded</span><span class="p">]</span>
    <span class="p">),</span>
    <span class="na">useStatus</span><span class="p">:</span> <span class="nf">makeEventHook</span><span class="p">(</span>
      <span class="p">(</span><span class="nx">_event</span><span class="p">,</span> <span class="na">key</span><span class="p">:</span> <span class="kr">keyof</span> <span class="nx">TravelStatus</span><span class="p">)</span> <span class="o">=&gt;</span> <span class="nx">travelStore</span><span class="p">.</span><span class="nx">status</span><span class="p">[</span><span class="nx">key</span><span class="p">],</span>
      <span class="nx">eventBus</span><span class="p">,</span>
      <span class="p">[</span><span class="nx">TravelStatusChanged</span><span class="p">]</span>
    <span class="p">),</span>
  <span class="p">}</span>
<span class="p">}</span>
</code></pre>

</div>



<p>Notice how <code>useNewLastMinuteDeals</code> hook is implemented. All components using it will receive state change only when an event is published. If another component starts using it after recent event has been published, it will get only default value. A nice way to subscribe to events inside components.</p>

<p>These two composers are included in the composition root in <code>web-app.ts</code>:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight typescript"><code><span class="nx">react</span><span class="p">:</span> <span class="p">{</span>
  <span class="nl">auth</span><span class="p">:</span> <span class="nf">composeAuthHooks</span><span class="p">(</span><span class="nx">sessionStore</span><span class="p">,</span> <span class="nx">eventBus</span><span class="p">),</span>
  <span class="nx">travel</span><span class="p">:</span> <span class="nf">composeTravelHooks</span><span class="p">(</span><span class="nx">travelStore</span><span class="p">,</span> <span class="nx">lastMinuteDealsStore</span><span class="p">,</span> <span class="nx">eventBus</span><span class="p">),</span>
<span class="p">}</span>
</code></pre>

</div>



<p>This is not the only way to create a bridge between application and UI. You can also use Redux or anything else. I've created a boilerplate code in <code>contexts/travel/interface/redux/</code> that you can use as a starting point. The only problem I see in this approach is you have to duplicate application state in Redux.</p>

<h3>
  
  
  Orchestrators
</h3>

<p>Often times we need to listen to events and coordinate flows across different contexts. And for these purposes we can create orchestrators. Let's put them in <code>processes/</code> folder.</p>

<p>In the current application we have the following orchestrators:</p>

<ul>
<li>
<code>InitProcess</code> - runs application initialization (restore auth session, start checking new last minute deals)</li>
<li>
<code>InitSessionProcess</code> - subscribes to <code>SessionChanged</code> event to update stores with related data on session change</li>
</ul>

<p>They are also initialized and started in the composition root, which is a very beautiful place where you can see how this little virtual world is built.</p>

<h3>
  
  
  Middlewares
</h3>

<p>Middlewares are a powerful way to extend the functionality of our application by adding additional processing steps, guards, and transformations to the API. I think all libraries have them. Let's add them too.</p>

<p>In our application we have the following middlewares:</p>

<ul>
<li>
<code>RequireAuthMiddleware</code> - checks if the user is authenticated before allowing access to certain use-cases</li>
<li>
<code>QueryCacheMiddleware</code> - caches the results of expensive queries to improve performance for a specific duration</li>
</ul>

<p>Let's take a look at <code>contexts/auth/middleware/require-auth.ts</code>:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight typescript"><code><span class="k">export</span> <span class="k">default</span> <span class="kd">function</span> <span class="nf">makeRequireAuthMiddleware</span><span class="p">(</span><span class="nx">sessionStore</span><span class="p">:</span> <span class="nx">SessionStore</span><span class="p">)</span> <span class="p">{</span>
  <span class="k">return</span> <span class="kd">function</span> <span class="nf">requireAuth</span><span class="o">&lt;</span><span class="nx">TArgs</span> <span class="kd">extends</span> <span class="nx">unknown</span><span class="p">[],</span> <span class="nx">TResult</span><span class="o">&gt;</span><span class="p">(</span>
    <span class="nx">useCase</span><span class="p">:</span> <span class="nx">UseCase</span><span class="o">&lt;</span><span class="nx">TArgs</span><span class="p">,</span> <span class="nx">TResult</span><span class="o">&gt;</span>
  <span class="p">)</span> <span class="p">{</span>
    <span class="k">return</span> <span class="p">{</span>
      <span class="k">async</span> <span class="nf">execute</span><span class="p">(...</span><span class="na">args</span><span class="p">:</span> <span class="nx">TArgs</span><span class="p">):</span> <span class="nb">Promise</span><span class="o">&lt;</span><span class="nx">TResult</span><span class="o">&gt;</span> <span class="p">{</span>
        <span class="k">if </span><span class="p">(</span><span class="o">!</span><span class="nx">sessionStore</span><span class="p">.</span><span class="nx">session</span><span class="p">)</span> <span class="p">{</span>
          <span class="k">throw</span> <span class="k">new</span> <span class="nc">UnauthorizedError</span><span class="p">(</span><span class="dl">'</span><span class="s1">User is not authenticated</span><span class="dl">'</span><span class="p">)</span>
        <span class="p">}</span>

        <span class="k">return</span> <span class="nx">useCase</span><span class="p">.</span><span class="nf">execute</span><span class="p">(...</span><span class="nx">args</span><span class="p">)</span>
      <span class="p">},</span>
    <span class="p">}</span>
  <span class="p">}</span>
<span class="p">}</span>
</code></pre>

</div>



<p>Pretty simple, right? A middleware factory receives a <code>sessionStore</code> instance and returns a middleware function that can wrap a use-case, becomes a proxy. The middleware checks if the user is authenticated before allowing the use-case to execute, otherwise it throws an error. So simple and so effective.</p>

<p>Please check also a middleware in <code>shared/middleware/query-cache.ts</code>. It allows to reuse use-case's results per use-case command during a specific time window and avoid unnecessary re-fetching.</p>

<p>What I want to point out is everything becomes so much simpler to implement if not putting initially ourself in some library's constraints especially considering that the library didn't want us to do it.</p>

<h2>
  
  
  Result
</h2>

<p>If you've cloned the code, now open <code>ui/contexts/app.ts</code> and switch from Demo to WebApp, there should be prepared commented out lines. And do the same in <code>ui/contexts/provider/app.tsx</code>. It shouldn't require you to update API calls in components as I tried to keep demo's API same as WebApp's. Then run <code>npm run dev</code>.</p>

<p>Try logging in, registering. We haven't implemented logging out, but you can do it yourself. Also, you can delete <code>session</code> key in Local Storage in DevTools. In the browser console enter <code>await api.admin.seed()</code> (a use-case in Admin context) to populate your local storage with some travel data. Now you should be able to find and purchase a travel of your dreams between the following cities:</p>

<ul>
<li>New York</li>
<li>Los Angeles</li>
<li>Chicago</li>
<li>Houston</li>
<li>Miami</li>
<li>San Francisco</li>
<li>Seattle</li>
<li>Boston</li>
<li>Denver</li>
<li>Atlanta</li>
</ul>

<p>You may also copy <strong>Offer ID</strong> from some travel card, and then enter the following in the console:</p>

<p><code>localStorage.setItem('specialOffers', JSON.stringify([...JSON.parse(localStorage.getItem('specialOffers')), { offer_id: '&lt;OFFER_ID&gt;', special_price: 99.99, description: 'Test offer', id: 'my_id' }]))</code></p>

<p>And see a new last-minute deal appeared in the list.</p>

<h2>
  
  
  Conclusion
</h2>

<p>In this article we have built a very nice UI which doesn't perform any side effect by itself, and for this reason it is very easy to develop and maintain. Responsibility for various logic processing and decision making has moved to the application - the central thing that should orchestrate everything, and UI becomes only a presentation of this. We've touched DDD, cared about separation of concerns, didn't forget about SOLID, and implemented a nice layered architecture that allows for easy testing and maintenance, and exported a very convenient API. We've also built a nice bridge between application and UI via hooks and events.</p>

<p>The code may look very verbose, more like a framework, and seems to be overkill. Well, for this particular application - yes. But only now, while the project is small. As the application grows the simpler you start the more mess you have in the project later, because when you have a lot of building blocks in the application, you need more rooms for them, and these rooms need a solid foundation.</p>

<p>So I hope you found this article helpful and some ideas will inspire you to implement something new in your projects.<br>
Anyway, what do you think?</p>

<p>P.S. It is actually not that bare-metal since I used React for UI, Mantine UI for components, and EventEmitter for event bus. But I thinks that's ok. When you need to make something not very trivial bare-metal for STM32 you still have to use a few basic libraries.</p>

<p>P.P.S. Sorry for my English, I didn't want to process it via AI tools to not lose my identity.</p>




<p>Originally published on <a href="https://github.com/afewlinesofcode/frontend-arch/blob/main/README.md" rel="noopener noreferrer">GitHub</a>.</p>

<p>© 2025 Stanislav Yaranov<br>
Licensed under <a href="https://creativecommons.org/licenses/by/4.0/" rel="noopener noreferrer">CC BY 4.0</a>.</p>

