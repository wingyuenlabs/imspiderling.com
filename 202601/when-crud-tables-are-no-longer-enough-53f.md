---
Title: When CRUD Tables Are No Longer Enough
Description: 
Author: giuliopanda
Date: 2026-01-09T21:43:50.000Z
Robots: noindex,nofollow
Template: index
---
<p>Anyone who has built an admin panel in PHP knows how this story usually ends. At the beginning, everything is simple: a table, a form, a couple of filters. Then real requirements arrive — relationships between data, more complex flows, partial updates — and the classic “table + form” CRUD starts to crack.</p>

<p>It’s not CRUD’s fault. The problem is that admin interfaces are not just CRUD.<br>
A real admin needs to see data in different contexts, navigate relationships without losing state, and edit related entities quickly.</p>

<p>These reflections led me to explore different approaches, until I decided to build a system from scratch, outside of the most popular frameworks. This is where I ended up.</p>

<p><strong>A Different Way to Think About the Problem</strong><br>
Instead of thinking “I need to build a page”, I started thinking “I need to describe a view of this data”.<br>
This led me to builders: PHP objects that describe what to display and how. Not frontend components, not magic widgets — just PHP structures that generate HTML, handle queries, and return either HTML or JSON responses.<br>
The code stays PHP. The flow stays request → processing → response.</p>

<p><strong>A Concrete Example: Posts</strong><br>
Let’s start with the simplest case: a module to manage posts with a title and content.</p>

<p><strong>The Model defines the data structure</strong><br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight plaintext"><code>class PostsModel extends AbstractModel
{
    protected function configure($rule): void
    {
        $rule-&gt;table('#__posts')
            -&gt;id()
            -&gt;string('title')-&gt;index()
            -&gt;text('content')-&gt;formType('editor');
    }

    #[Validate('title')]
    public function validateTitle($current_record_obj): string {
        $value = $current_record_obj-&gt;title;
        if (strlen($value) &lt; 5) {
            return 'Title must be at least 5 characters long';
        }
        return '';
    }
} 
</code></pre>

</div>



<p>The model is the source of truth. It declares the table, the fields, how they should appear in forms, and where validation belongs — in the model, where it actually makes sense.</p>

<p><strong>The Controller uses builders to generate views</strong><br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight plaintext"><code>class PostsController extends AbstractController
{ 
    #[RequestAction('home')]
    public function postsList() {
        $tableBuilder = TableBuilder::create($this-&gt;model, 'idTablePosts')
            -&gt;field('content')-&gt;truncate(50)
            -&gt;field('title')-&gt;link('?page=posts&amp;action=edit&amp;id=%id%')
            -&gt;setDefaultActions();

        $response = array_merge($this-&gt;getCommonData(), $tableBuilder-&gt;getResponse());
        Response::render(MILK_DIR . '/Theme/SharedViews/list_page.php', $response);
    }

    #[RequestAction('edit')]
    public function postEdit() {
        $response = $this-&gt;getCommonData();
        $response['form'] = FormBuilder::create($this-&gt;model, $this-&gt;page)-&gt;getForm();
        Response::render(MILK_DIR . '/Theme/SharedViews/edit_page.php', $response);
    }
} 
</code></pre>

</div>



<p>TableBuilder already knows how to handle search, pagination, and sorting. FormBuilder knows which fields to show and how to validate them. You describe what you want — the builders generate the rest.</p>

<p><a href="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fysq235qhfa5ogfxh6isw.gif" class="article-body-image-wrapper"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fysq235qhfa5ogfxh6isw.gif" alt=" " width="800" height="512"></a></p>

<p><a href="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fk43ackzozhuukbc0ifv6.gif" class="article-body-image-wrapper"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fk43ackzozhuukbc0ifv6.gif" alt=" " width="800" height="515"></a></p>

<p>Views are plain PHP templates — no templating engines, no special syntax. If you need to change something, you open the file and edit the HTML directly.</p>

<h2>
  
  
  Where Things Get Complicated: Relationships
</h2>

<p>Simple CRUD works everywhere. Problems start when entities are related.<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight plaintext"><code>Recipes
 └─ Comments
</code></pre>

</div>



<p>When relationships become interactive, the usual options are either: splitting everything into multiple pages (losing fluidity), or relying on heavy frameworks that manage state for you (losing transparency and control).</p>

<p>Tools like Laravel Nova or Filament are powerful and work well in structured teams. But when flows become very specific, it can be hard to understand what is really happening between backend and frontend.</p>

<p>I wanted a different approach: keep everything in PHP, use fetch to avoid page reloads, but without hiding what’s going on.</p>

<h2>
  
  
  Recipes: CRUD with Relationships, Fully in Fetch
</h2>

<p>This is the full example: recipes with comments, two interface levels, no page reloads.</p>

<p>Recipe model<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight plaintext"><code>class RecipeModel extends AbstractModel
{
   protected function configure($rule): void
   {
        $rule-&gt;table('#__recipes')
            -&gt;id()
            -&gt;hasMany('comments', RecipeCommentsModel::class, 'recipe_id')
            -&gt;image('image')
            -&gt;title('name')-&gt;index()
            -&gt;text('ingredients')-&gt;formType('textarea')
            -&gt;select('difficulty', ['Easy', 'Medium', 'Hard']);
   }
} 
</code></pre>

</div>



<p>In just a few lines, it defines the table, the one-to-many relationship, and the field types. The builders automatically know how to handle uploads, selects, and text areas.</p>

<p><strong>Comments model</strong><br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight plaintext"><code>class RecipeCommentsModel extends AbstractModel
{
    protected function configure($rule): void
    {
        $rule-&gt;table('#__recipe_comments')
            -&gt;id()
            -&gt;int('recipe_id')-&gt;formType('hidden')
            -&gt;text('comment');
    }
} 
</code></pre>

</div>



<p>Everything else — lists, offcanvas forms, modals, automatic refresh — is built using the same builders and the same mental model.</p>

<p><a href="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fqo6baiayycqiqj0ieg34.gif" class="article-body-image-wrapper"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fqo6baiayycqiqj0ieg34.gif" alt=" " width="700" height="474"></a></p>

<p>Why This Approach<br>
This is not another framework. It’s a way of thinking about admin panels.<br>
Builders reduce boilerplate, but the code remains readable PHP. If a builder doesn’t do what you need, you can always drop down to plain PHP.<br>
JSON as a contract between backend and frontend is easy to debug. No hidden state, no complex lifecycles. PHP sends instructions, the browser executes them.<br>
Most importantly, relationships are handled with the same tools as basic CRUD. There’s no separate system to learn just for relations.</p>

<p>Who This Is For<br>
If you work with Laravel and you’re happy with it, this is probably not for you.</p>

<p>But if you find yourself in one of these situations:</p>

<p>You maintain existing PHP projects that can’t be migrated<br>
You work on internal tools, CRMs, or management systems<br>
You need to understand the code quickly, even months later<br>
You don’t want to chase complex ecosystems<br>
You want PHP that looks and feels like PHP<br>
Then it might be worth rethinking how we build admin interfaces.</p>

<p>Admin panels are not public websites. They are work tools. And the best tools are those that do their job well, remain understandable over time, and don’t force you to rewrite everything when requirements change.</p>

<p>These reflections come from the development of MilkAdmin, a PHP admin panel system I actively maintain. Code, demos, and documentation are available at <a href="//milkadmin.org">milkadmin.org</a>. MIT licensed.</p>

<p><a href="https://github.com/giuliopanda/milk-admin" rel="noopener noreferrer">https://github.com/giuliopanda/milk-admin</a></p>

