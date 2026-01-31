---
Title: MVP architecture adapted for Unity
Description: 
Author: Ged Bashiri
Date: 2026-01-31T22:07:26.000Z
Robots: noindex,nofollow
Template: index
---
<h1>
  
  
  Intro
</h1>

<p>This article explains how we adopted an <strong>MVP (Model–View–Presenter)</strong> architecture for our Unity game. I’ll walk through the overall structure, explain the main modules, and then discuss the pros, cons, and how we handled some common challenges.</p>




<h1>
  
  
  Disclaimer
</h1>

<p>This is <strong>not</strong> a strict MVP handbook, nor a perfect one-to-one mapping of classic MVP into Unity.</p>

<p>What you’ll see here is <strong>an adaptation</strong> that works well for our game, which is closer to an app with lots of menus and many mini-games. It may or may not fit your project perfectly, but hopefully it gives you useful ideas.</p>

<p>MVP has been used in Unity before, and there are many tutorials out there (for example:<br>
<a href="https://learn.unity.com/tutorial/build-a-modular-codebase-with-mvc-and-mvp-programming-patterns" rel="noopener noreferrer">https://learn.unity.com/tutorial/build-a-modular-codebase-with-mvc-and-mvp-programming-patterns</a>).</p>

<p><strong>My main concern with many existing examples,</strong> is that they rely heavily on MonoBehaviour APIs everywhere. That undermines one of the biggest benefits of MVP: <strong>Testability</strong> (in my opinion of course)</p>


<h1>
  
  
  Our App
</h1>

<p>Before talking architecture, it’s important to understand the product. There is no “one architecture to rule them all.”</p>

<p><a href="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fmedia1.giphy.com%2Fmedia%2Fv1.Y2lkPTc5MGI3NjExYnJkOHNrazdpc2pkNnF4a2I4ZTJuOHhnYnN3YTRmMDgyeGJxdHg3ayZlcD12MV9pbnRlcm5hbF9naWZfYnlfaWQmY3Q9Zw%2FelUGwgiPOdq7e%2Fgiphy.gif" class="article-body-image-wrapper"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fmedia1.giphy.com%2Fmedia%2Fv1.Y2lkPTc5MGI3NjExYnJkOHNrazdpc2pkNnF4a2I4ZTJuOHhnYnN3YTRmMDgyeGJxdHg3ayZlcD12MV9pbnRlcm5hbF9naWZfYnlfaWQmY3Q9Zw%2FelUGwgiPOdq7e%2Fgiphy.gif" width="245" height="200"></a></p>
One ring to rule them all.



<p>Our app, <strong><a href="https://magrid.education/" rel="noopener noreferrer">Magrid</a></strong> is a learning solution for kids aged 3 to 10 (check it out if you have a kid or you are a 5 years old by any chance), designed to address two major goals:</p>

<ul>
<li>    It is <strong>language-free</strong>, so children around the world can use it</li>
<li>    It is <strong>inclusive</strong>, including support for children with disabilities</li>
</ul>

<p>From a technical perspective, the app contains:</p>

<ul>
<li>    Many <strong>menus</strong> (login/register, student management, performance review, curriculum management, etc.)</li>
<li>    Many <strong>mini-games</strong> (pattern recognition, visual perception, number comparison, and more)</li>
</ul>

<p>The main technical concern was <strong>separation of concerns</strong>, both for menu logic and for the mini-games themselves.</p>


<h1>
  
  
  Architecture
</h1>

<p>The architecture has 2 types of entities: <strong>MVP modules</strong> and <strong>Systems</strong>.</p>
<h2>
  
  
  MVP Modules
</h2>

<p>An MVP module has 3 components (duh!):</p>

<p><a href="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fsznf9jjorw5uiwq89srx.png" class="article-body-image-wrapper"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fsznf9jjorw5uiwq89srx.png" alt="Well, Who would have thought?" width="600" height="500"></a></p>
<h3>
  
  
  View:
</h3>

<ul>
<li>    Essentially a <code>MonoBehaviour</code>
</li>
<li>    Handles user input</li>
<li>    Displays data</li>
<li>    Talks directly to the Presenter</li>
</ul>
<h3>
  
  
  Presenter:
</h3>

<ul>
<li>    <strong>Not</strong> a <code>MonoBehaviour</code>
</li>
<li>    Contains all business logic</li>
<li>    Fetches data from Model and processes it</li>
<li>    Updates the <strong>View only through an interface</strong>
</li>
</ul>
<h3>
  
  
  Model:
</h3>

<ul>
<li>    Stores data needed by the module</li>
<li>    Separated for clarity</li>
</ul>
<h3>
  
  
  MVP Lifecycle
</h3>

<p>The <strong>View</strong> is the entry point of an MVP module.<br>
It can be initiated either by Unity event functions or by another MVP module.</p>

<ol>
<li>    The View creates its Presenter(s)</li>
<li>    A View may have <strong>more than one Presenter</strong> for reusability (covered in last section)</li>
<li>    The View passes its interface to the Presenter</li>
<li>    The Presenter creates its Model (if needed)</li>
<li>    The module is ready</li>
</ol>

<p>All <strong>business logic lives in the Presenter</strong>.</p>

<p>Imagine, if you were to replace the View with a command-line implementation, the application would still work. because the Presenter Uses <strong>no Unity APIs</strong> and Communicates only via interfaces</p>

<p><a href="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fb3bhmf4dex6fass0w4gj.gif" class="article-body-image-wrapper"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fb3bhmf4dex6fass0w4gj.gif" width="480" height="360"></a></p>
Presenter doesn’t need visuals to work. If your imagination is good enough.



<p>Here is simple example of each components:</p>
<h3>
  
  
  View:
</h3>


<div class="highlight js-code-highlight">
<pre class="highlight plaintext"><code>public class FooView : BaseScreen, IFooView
{
    [SerializeField] private TMP_Text exposedInExpectorExample;

    private FooPresenter _presenter;

    private void Start()
    {
        _presenter = new FooPresenter(this);
    }

    public void SetName(string barName)
    {
        exposedInExpectorExample.text = barName;
    }
}
</code></pre>

</div>

<h3>
  
  
  View interface:
</h3>


<div class="highlight js-code-highlight">
<pre class="highlight plaintext"><code>public interface IFooView : BaseMvpView&lt;FooPresenter&gt;
{
     public void SetName(string barName);
}
</code></pre>

</div>

<h3>
  
  
  Presenter:
</h3>


<div class="highlight js-code-highlight">
<pre class="highlight plaintext"><code>public class FooPresenter : BaseMvpPresenter
{
    private IFooView _view;
    private FooModel _model;

    public FooPresenter(IFooView view)
    {
        _view = view;
        _model = new FooModel();
    }
}
</code></pre>

</div>

<h3>
  
  
  Model:
</h3>


<div class="highlight js-code-highlight">
<pre class="highlight plaintext"><code>public class RegisterTeacherModel : BaseMvpModel
{
     public string BarName { get; set; }
}
</code></pre>

</div>


<p>In the example:<br>
— <code>BaseScreen</code>, which our view extends, is a <code>MonoBehaviour</code> responsible for canvas management(Open, Close, …)</p>

<p>— <code>BaseMvpView</code>, which our interface extends, is just a generic interface<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight plaintext"><code>public interface BaseMvpView&lt;T&gt; where T : BaseMvpPresenter
{
}
</code></pre>

</div>



<p>— <code>BaseMvpModel</code> also is just a simple interface<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight plaintext"><code>public interface BaseMvpModel
{
    public int Version { get; }
}
</code></pre>

</div>



<p>The <code>Version</code> field helps with data migration when loading persistent data after updates. Each module can manage its own migration logic.</p>

<h2>
  
  
  System
</h2>

<p>Systems are more or less an MVP module without View.<br>
<a href="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fmedia0.giphy.com%2Fmedia%2Fv1.Y2lkPTc5MGI3NjExc2R0a2h1dXVuNzA2Mm1mYzQwYWxubTYwemhnc3pkZDF4dzFkNXRoMyZlcD12MV9pbnRlcm5hbF9naWZfYnlfaWQmY3Q9Zw%2FuNE1fngZuYhIQ%2Fgiphy.gif" class="article-body-image-wrapper"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fmedia0.giphy.com%2Fmedia%2Fv1.Y2lkPTc5MGI3NjExc2R0a2h1dXVuNzA2Mm1mYzQwYWxubTYwemhnc3pkZDF4dzFkNXRoMyZlcD12MV9pbnRlcm5hbF9naWZfYnlfaWQmY3Q9Zw%2FuNE1fngZuYhIQ%2Fgiphy.gif" width="320" height="135"></a></p>
The systems are accessible via dependency injection.



<p>Their purpose is to serve all the MVP modules. They all will be initiated by a single MonoBehaviour (I called it AppManager) which is Accessible by MVPs (A Unity adapted singleton).<br>
This could also be implemented using dependency injection.<br>
I chose not to use <a href="https://github.com/modesttree/Zenject" rel="noopener noreferrer">Zenject</a> due to IL2CPP issues and instead built a simple dependency manager.\</p>

<h3>
  
  
  Key points:
</h3>

<ul>
<li>    Each system exposes an interface</li>
<li>    All systems can be mocked in tests</li>
<li>    Systems are initialized during app loading</li>
</ul>

<h3>
  
  
  Common Systems:
</h3>

<ul>
<li>    <strong>Profile System</strong> — persistent data storage</li>
<li>    <strong>API System</strong> — server communication</li>
<li>    <strong>Purchase System</strong> — in-app purchases</li>
<li>    <strong>Analytics System</strong> — analytics providers and events (Game analytics, Firebase, …)</li>
<li>    <strong>UI System</strong> — screens, popups, navigation, back actions</li>
</ul>

<h2>
  
  
  PlayInfo
</h2>

<p>Ok, there is another entity which I called it PlayInfo.</p>

<p><a href="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fwtk82x9qm5hlj7p54438.png" class="article-body-image-wrapper"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fwtk82x9qm5hlj7p54438.png" alt="Here me out, you can just ignore this if you are starting from scratch." width="360" height="186"></a></p>
Here me out, you can just ignore this if you are starting from scratch.



<p><code>PlayInfo</code> handles rare cases where MVP modules need to communicate.</p>

<p>It contains:</p>

<ul>
<li>    A set of <strong>events</strong> that modules can subscribe to</li>
<li>    Some <strong>shared variables</strong>
</li>
</ul>

<p>Originally, this existed mainly to help transition from a <strong>legacy codebase</strong>, where everything accessed everything else via static methods. <code>PlayInfo</code> acted as a <strong>mediator</strong> between refactored MVP modules and legacy code. In the end, we kept a few of its events because they turned out to be genuinely useful.</p>




<h1>
  
  
  Review
</h1>

<h2>
  
  
  Pros:
</h2>

<h3>
  
  
  Testability (Main Goal)
</h3>

<p>We use two types of tests:</p>

<h4>
  
  
  Unit tests
</h4>

<ul>
<li>    You can have excessive unit testes, covering presenters and systems. They have minimum dependencies which can be easily mocked.</li>
</ul>

<h4>
  
  
  Smoke / Integration tests
</h4>

<ul>
<li>    You can simulate user interactions and test multiple modules together.</li>
<li>    Simulating user interactions and checking UI can be done using UnityTest and Reflection which deserves its own article.</li>
</ul>

<h3>
  
  
  Encapsulation
</h3>

<p>Each module is fully isolated.<br>
Communication happens only via:</p>

<ul>
<li>    Interfaces</li>
<li>    (Rarely) events
This makes refactoring/changing safer and reduces unintended side effects.</li>
</ul>

<h2>
  
  
  Cons:
</h2>

<h3>
  
  
  Boilerplate:
</h3>

<p>Each module usually needs:</p>

<ul>
<li>    View</li>
<li>    View Interface</li>
<li>    Presenter</li>
<li>    Model
<img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fgvavdqbeuxym8pngmsm4.gif" width="244" height="170">
</li>
</ul>

<h3>
  
  
  Unity Edge Cases
</h3>

<p>At the end of the day, this is still a <strong>game</strong>. Sometimes you need unity APIs such as <code>Coroutines</code>, <code>Physics</code>.</p>

<h4>
  
  
  our solution:
</h4>

<ul>
<li>    To start <code>Coroutines</code>, use <code>AppManager</code> (Remember? <code>MonoBehaviour</code> Singleton)</li>
</ul>

<p>These cases were rare for us.<br>
If this becomes frequent in your project, it may be worth introducing a new entity.</p>

<h3>
  
  
  Reusability
</h3>

<p>Although, I’ll take <strong>maintainability</strong> over <strong>reusability</strong> any day, but repeating yourself still hurts.</p>

<p>Ways we addressed this:</p>

<h4>
  
  
  View Reuse
</h4>

<ul>
<li>Build generic UI components. For example a <strong>Student Profile UI element</strong> reused across multiple views. Not only you don’t repeat yourself over and over but also changing code/prefab once will update all screens everywhere</li>
</ul>

<h4>
  
  
  Logic Reuse
</h4>

<ul>
<li>Stateless logic goes into <strong>Utils</strong>.</li>
<li>For example email validation regex can be used in multiple places.</li>
</ul>

<h4>
  
  
  Presenter reuse
</h4>

<ul>
<li>A view can use multiple Presenters and pass proper interface to the presenter constructor. (Interface segregation)</li>
<li>Example: “Contact Support” logic already exists in Settings. Reuse that Presenter in the Shop screen instead of duplicating code</li>
</ul>

<p>If your situation doesn’t fit in these two cases, most probably it deserves to be a system and provide service to MVP modules in need.</p>




<h1>
  
  
  Conclusion
</h1>

<p>Sometimes Games and Apps are very similar and they are not very distinguishable. Architectures like <strong>MVP</strong> or <strong>MVVM</strong>, while not originally designed for games, can work extremely well for <strong>menu-heavy</strong>, <strong>logic-driven</strong> <strong>Unity projects</strong>. Here I tried adopt MVP architecture for Unity environment and benefited from:</p>

<ul>
<li>    Testability</li>
<li>    Separation of concerns</li>
<li>    Maintainability</li>
</ul>

<p>In the end, if you have questions or better idea to improve this approach, let me know. :D</p>

