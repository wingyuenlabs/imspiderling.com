---
Title: How to Add a Form Wizard to Your Website (React, Angular, Vue, plain JS)
Description: 
Author: SurveyJS
Date: 2026-03-12T21:13:31.000Z
Robots: noindex,nofollow
Template: index
---
<p>The quickest way to add a form wizard to a web application is to use a JavaScript form library that already supports multi-step flows. Instead of building page navigation, validation, and conditional logic from scratch, you define the form as JSON and let the rendering library handle the runtime behavior.</p>

<p>SurveyJS is a good fit for this. It lets you model forms as JSON schemas, configure validation and logic declaratively, and render the same form in React, Angular, Vue, or plain JavaScript. Survey Creator allows you to generate that schema visually, and Form Library renders it in your application.</p>

<p>This article shows how to create a multi-step form wizard with SurveyJS and embed it in different JavaScript stacks.</p>

<h2>
  
  
  What is a Form Wizard
</h2>

<p>A form wizard is a form navigation pattern that splits a long or complex form into a sequence of smaller steps. Instead of showing every input at once, the UI presents one section, question, or field at a time and guides the user through the flow with navigation controls and progress indicators. This approach improves usability for long forms such as the sample background check consent form shown below:</p>

<p><a href="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2F0hgfbbfhhibgjtl1u1lu.png" class="article-body-image-wrapper"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2F0hgfbbfhhibgjtl1u1lu.png" alt="Multi-step form wizard" width="800" height="421"></a></p>

<h2>
  
  
  Why Use SurveyJS for Multi-Step Forms
</h2>

<p>SurveyJS separates form design from rendering.</p>

<p>You define the form structure, validation rules, navigation, and logic in JSON. Then the Form Library renders that schema in the framework of your choice. <a href="https://surveyjs.io/form-library/documentation/overview" rel="noopener noreferrer">SurveyJS Form Library</a> is framework-specific at the UI layer, while <code>survey-core</code> contains the shared model and runtime logic, so you can reuse the same form definition across React, Angular, Vue, and plain JavaScript.</p>

<p>To learn more about SurveyJS architecture, please refer to the <a href="https://surveyjs.io/documentation/surveyjs-architecture" rel="noopener noreferrer">SurveyJS Architecture</a> guide.</p>

<p><a href="https://surveyjs.io/survey-creator/documentation/overview" rel="noopener noreferrer">Survey Creator</a> is the visual form builder used to generate and edit form schemas. It supports drag-and-drop form design, a JSON editor, validation rules, and conditional logic configuration. The generated JSON can then be rendered by SurveyJS Form Library.</p>

<p>In practice, that gives you:</p>

<ul>
<li>visual form authoring</li>
<li>JSON-based configuration</li>
<li>built-in validation</li>
<li>conditional logic</li>
<li>reusable rendering across frameworks</li>
</ul>

<h2>
  
  
  SurveyJS Rendering Packages
</h2>

<p>SurveyJS ships framework-specific rendering packages on top of <code>survey-core</code>. Using these packages, you can embed multi-step forms into any<br>
website. </p>

<div class="table-wrapper-paragraph"><table>
<thead>
<tr>
<th>Framework</th>
<th>NPM Package</th>
<th>Get Started guide</th>
</tr>
</thead>
<tbody>
<tr>
<td>React</td>
<td><a href="https://www.npmjs.com/package/survey-react-ui" rel="noopener noreferrer"><code>survey-react-ui</code></a></td>
<td><a href="https://surveyjs.io/form-library/documentation/get-started-react" rel="noopener noreferrer">React Get Started Guide</a></td>
</tr>
<tr>
<td>Angular</td>
<td><a href="https://www.npmjs.com/package/survey-angular-ui" rel="noopener noreferrer"><code>survey-angular-ui</code></a></td>
<td><a href="https://surveyjs.io/form-library/documentation/get-started-angular" rel="noopener noreferrer">Angular Get Started Guide</a></td>
</tr>
<tr>
<td>Vue.js</td>
<td><a href="https://www.npmjs.com/package/survey-vue3-ui" rel="noopener noreferrer"><code>survey-vue3-ui</code></a></td>
<td><a href="https://surveyjs.io/form-library/documentation/get-started-vue" rel="noopener noreferrer">Vue Get Started Guide</a></td>
</tr>
<tr>
<td>Plain JavaScript</td>
<td><a href="https://www.npmjs.com/package/survey-js-ui" rel="noopener noreferrer"><code>survey-js-ui</code></a></td>
<td><a href="https://surveyjs.io/form-library/documentation/get-started-html-css-javascript" rel="noopener noreferrer">Plain JavaScript Get Started Guide</a></td>
</tr>
</tbody>
</table></div>

<p>All of them render the same JSON schema, which means the form definition stays the same even if the frontend stack changes.</p>
<h2>
  
  
  Define the Wizard as JSON
</h2>

<p>In SurveyJS, a form is described as a JSON schema. At the top level, you define survey metadata and runtime options. Inside pages, you define the questions shown in each step.</p>

<p>Here is a simplified example of a background check consent form:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight javascript"><code><span class="kd">const</span> <span class="nx">surveyJson</span> <span class="o">=</span> <span class="p">{</span>
  <span class="dl">"</span><span class="s2">title</span><span class="dl">"</span><span class="p">:</span> <span class="dl">"</span><span class="s2">Background check consent form</span><span class="dl">"</span><span class="p">,</span>
  <span class="dl">"</span><span class="s2">pages</span><span class="dl">"</span><span class="p">:</span> <span class="p">[</span>
    <span class="p">{</span>
      <span class="dl">"</span><span class="s2">name</span><span class="dl">"</span><span class="p">:</span> <span class="dl">"</span><span class="s2">personal-details</span><span class="dl">"</span><span class="p">,</span>
      <span class="dl">"</span><span class="s2">title</span><span class="dl">"</span><span class="p">:</span> <span class="dl">"</span><span class="s2">Personal Details</span><span class="dl">"</span><span class="p">,</span>
      <span class="dl">"</span><span class="s2">elements</span><span class="dl">"</span><span class="p">:</span> <span class="p">[</span>
        <span class="p">{</span>
          <span class="dl">"</span><span class="s2">type</span><span class="dl">"</span><span class="p">:</span> <span class="dl">"</span><span class="s2">text</span><span class="dl">"</span><span class="p">,</span>
          <span class="dl">"</span><span class="s2">name</span><span class="dl">"</span><span class="p">:</span> <span class="dl">"</span><span class="s2">surname</span><span class="dl">"</span><span class="p">,</span>
          <span class="dl">"</span><span class="s2">title</span><span class="dl">"</span><span class="p">:</span> <span class="dl">"</span><span class="s2">Surname</span><span class="dl">"</span><span class="p">,</span>
          <span class="dl">"</span><span class="s2">maxLength</span><span class="dl">"</span><span class="p">:</span> <span class="mi">35</span>
        <span class="p">},</span>
        <span class="p">{</span>
          <span class="dl">"</span><span class="s2">type</span><span class="dl">"</span><span class="p">:</span> <span class="dl">"</span><span class="s2">text</span><span class="dl">"</span><span class="p">,</span>
          <span class="dl">"</span><span class="s2">name</span><span class="dl">"</span><span class="p">:</span> <span class="dl">"</span><span class="s2">surname-at-birth</span><span class="dl">"</span><span class="p">,</span>
          <span class="dl">"</span><span class="s2">startWithNewLine</span><span class="dl">"</span><span class="p">:</span> <span class="kc">false</span><span class="p">,</span>
          <span class="dl">"</span><span class="s2">title</span><span class="dl">"</span><span class="p">:</span> <span class="dl">"</span><span class="s2">Surname at birth</span><span class="dl">"</span><span class="p">,</span>
          <span class="dl">"</span><span class="s2">maxLength</span><span class="dl">"</span><span class="p">:</span> <span class="mi">35</span>
        <span class="p">},</span>
        <span class="p">{</span>
          <span class="dl">"</span><span class="s2">type</span><span class="dl">"</span><span class="p">:</span> <span class="dl">"</span><span class="s2">text</span><span class="dl">"</span><span class="p">,</span>
          <span class="dl">"</span><span class="s2">name</span><span class="dl">"</span><span class="p">:</span> <span class="dl">"</span><span class="s2">dob</span><span class="dl">"</span><span class="p">,</span>
          <span class="dl">"</span><span class="s2">title</span><span class="dl">"</span><span class="p">:</span> <span class="dl">"</span><span class="s2">Date of Birth</span><span class="dl">"</span><span class="p">,</span>
          <span class="dl">"</span><span class="s2">validators</span><span class="dl">"</span><span class="p">:</span> <span class="p">[</span>
            <span class="p">{</span>
              <span class="dl">"</span><span class="s2">type</span><span class="dl">"</span><span class="p">:</span> <span class="dl">"</span><span class="s2">expression</span><span class="dl">"</span><span class="p">,</span>
              <span class="dl">"</span><span class="s2">text</span><span class="dl">"</span><span class="p">:</span> <span class="dl">"</span><span class="s2">You must be at least 18 y.o. to submit the form.</span><span class="dl">"</span><span class="p">,</span>
              <span class="dl">"</span><span class="s2">expression</span><span class="dl">"</span><span class="p">:</span> <span class="dl">"</span><span class="s2">age({dob}) &gt;= 18</span><span class="dl">"</span>
            <span class="p">}</span>
          <span class="p">],</span>
          <span class="dl">"</span><span class="s2">inputType</span><span class="dl">"</span><span class="p">:</span> <span class="dl">"</span><span class="s2">date</span><span class="dl">"</span><span class="p">,</span>
        <span class="p">},</span>
                <span class="p">{</span>
          <span class="dl">"</span><span class="s2">type</span><span class="dl">"</span><span class="p">:</span> <span class="dl">"</span><span class="s2">radiogroup</span><span class="dl">"</span><span class="p">,</span>
          <span class="dl">"</span><span class="s2">name</span><span class="dl">"</span><span class="p">:</span> <span class="dl">"</span><span class="s2">gender</span><span class="dl">"</span><span class="p">,</span>
          <span class="dl">"</span><span class="s2">maxWidth</span><span class="dl">"</span><span class="p">:</span> <span class="dl">"</span><span class="s2">50%</span><span class="dl">"</span><span class="p">,</span>
          <span class="dl">"</span><span class="s2">title</span><span class="dl">"</span><span class="p">:</span> <span class="dl">"</span><span class="s2">Gender</span><span class="dl">"</span><span class="p">,</span>
          <span class="dl">"</span><span class="s2">commentText</span><span class="dl">"</span><span class="p">:</span> <span class="dl">"</span><span class="s2">Other (describe)</span><span class="dl">"</span><span class="p">,</span>
          <span class="dl">"</span><span class="s2">choices</span><span class="dl">"</span><span class="p">:</span> <span class="p">[</span>
            <span class="p">{</span>
              <span class="dl">"</span><span class="s2">value</span><span class="dl">"</span><span class="p">:</span> <span class="dl">"</span><span class="s2">female</span><span class="dl">"</span><span class="p">,</span>
              <span class="dl">"</span><span class="s2">text</span><span class="dl">"</span><span class="p">:</span> <span class="dl">"</span><span class="s2">Female</span><span class="dl">"</span>
            <span class="p">},</span>
            <span class="p">{</span>
              <span class="dl">"</span><span class="s2">value</span><span class="dl">"</span><span class="p">:</span> <span class="dl">"</span><span class="s2">male</span><span class="dl">"</span><span class="p">,</span>
              <span class="dl">"</span><span class="s2">text</span><span class="dl">"</span><span class="p">:</span> <span class="dl">"</span><span class="s2">Male</span><span class="dl">"</span>
            <span class="p">},</span>
            <span class="p">{</span>
              <span class="dl">"</span><span class="s2">value</span><span class="dl">"</span><span class="p">:</span> <span class="dl">"</span><span class="s2">other</span><span class="dl">"</span><span class="p">,</span>
              <span class="dl">"</span><span class="s2">text</span><span class="dl">"</span><span class="p">:</span> <span class="dl">"</span><span class="s2">Other</span><span class="dl">"</span>
            <span class="p">}</span>
          <span class="p">],</span>
          <span class="dl">"</span><span class="s2">otherText</span><span class="dl">"</span><span class="p">:</span> <span class="dl">"</span><span class="s2">Other (please specify)</span><span class="dl">"</span><span class="p">,</span>
          <span class="dl">"</span><span class="s2">otherErrorText</span><span class="dl">"</span><span class="p">:</span> <span class="dl">"</span><span class="s2">Response required: please specify your               gender.</span><span class="dl">"</span><span class="p">,</span>
          <span class="dl">"</span><span class="s2">colCount</span><span class="dl">"</span><span class="p">:</span> <span class="mi">3</span>
        <span class="p">},</span>
      <span class="p">]</span>
    <span class="p">}</span>
  <span class="p">],</span>
  <span class="dl">"</span><span class="s2">showProgressBar</span><span class="dl">"</span><span class="p">:</span> <span class="kc">true</span><span class="p">,</span>
  <span class="dl">"</span><span class="s2">progressBarLocation</span><span class="dl">"</span><span class="p">:</span> <span class="dl">"</span><span class="s2">belowheader</span><span class="dl">"</span><span class="p">,</span>
  <span class="dl">"</span><span class="s2">progressBarType</span><span class="dl">"</span><span class="p">:</span> <span class="dl">"</span><span class="s2">questions</span><span class="dl">"</span><span class="p">,</span>
  <span class="dl">"</span><span class="s2">completeText</span><span class="dl">"</span><span class="p">:</span> <span class="dl">"</span><span class="s2">Submit Form</span><span class="dl">"</span><span class="p">,</span>
  <span class="dl">"</span><span class="s2">questionsOnPageMode</span><span class="dl">"</span><span class="p">:</span> <span class="dl">"</span><span class="s2">questionPerPage</span><span class="dl">"</span>
<span class="p">}</span>
</code></pre>

</div>



<p>The <a href="https://surveyjs.io/form-library/documentation/api-reference/survey-data-model#questionsOnPageMode" rel="noopener noreferrer"><code>questionsOnPageMode</code></a> is a survey-level setting that controls how the runtime treats steps. It works as the switch that determines whether navigation happens by page, by question, or by input.</p>

<p>The property accepts the following values:</p>

<ul>
<li>
<code>singlePage</code> – Combines all survey pages into a single page.</li>
<li>
<code>questionPerPage</code> – Displays each question on a separate page.</li>
<li>
<code>inputPerPage</code> – Displays each input field on a separate page. Complex questions—such as Single-Select Matrix, Multi-Select Matrix, Dynamic Matrix, Dynamic Panel, and Multiple Textboxes—are split so that each input field appears on its own page.</li>
<li>
<code>standard</code> (default) – Retains the original single- or multi-page structure.</li>
</ul>

<p><code>questionsOnPageMode: "questionPerPage"</code> turns the form into a question-by-question wizard, regardless of how many elements are on a page. If you want each declared page to behave as a wizard step, use the default page structure instead of question-per-page mode.</p>

<h2>
  
  
  Configure Wizard Behavior in Survey Creator
</h2>

<p>You can define the JSON manually, but it is faster to configure the form visually in Survey Creator.</p>

<p>Survey Creator produces a JSON configuration that Form Library can render later, and the survey-level settings include layout and navigation configuration. To configure a multi-step form in the builder UI:</p>

<ol>
<li>At the top of the Property Grid, select <strong>Survey</strong> to switch to the survey-level settings.</li>
<li>Under <strong>Navigation</strong>, locate the <strong>Survey layout</strong> setting.</li>
<li>Select the option that suits your use case.</li>
</ol>

<p><a href="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fi1jbut8iniwfvo49ysle.png" class="article-body-image-wrapper"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fi1jbut8iniwfvo49ysle.png" alt="Form wizard layout options" width="800" height="421"></a> </p>

<h2>
  
  
  Render the Form Wizard
</h2>

<p>Once the JSON schema is ready, pass it to a <a href="https://surveyjs.io/form-library/documentation/api-reference/survey-data-model" rel="noopener noreferrer"><code>Model</code></a> from <code>survey-core</code> and render it with the package for your framework.</p>

<h3>
  
  
  React
</h3>



<div class="highlight js-code-highlight">
<pre class="highlight typescript"><code><span class="dl">"</span><span class="s2">use client</span><span class="dl">"</span><span class="p">;</span>

<span class="k">import</span> <span class="p">{</span> <span class="nx">useMemo</span> <span class="p">}</span> <span class="k">from</span> <span class="dl">"</span><span class="s2">react</span><span class="dl">"</span><span class="p">;</span>
<span class="k">import</span> <span class="dl">"</span><span class="s2">survey-core/survey-core.css</span><span class="dl">"</span><span class="p">;</span>
<span class="k">import</span> <span class="p">{</span> <span class="nx">Model</span> <span class="p">}</span> <span class="k">from</span> <span class="dl">"</span><span class="s2">survey-core</span><span class="dl">"</span><span class="p">;</span>
<span class="k">import</span> <span class="p">{</span> <span class="nx">Survey</span> <span class="p">}</span> <span class="k">from</span> <span class="dl">"</span><span class="s2">survey-react-ui</span><span class="dl">"</span><span class="p">;</span>

<span class="kd">const</span> <span class="nx">surveyJson</span> <span class="o">=</span> <span class="p">{</span>
  <span class="cm">/* ... */</span>
<span class="p">};</span>

<span class="k">export</span> <span class="k">default</span> <span class="kd">function</span> <span class="nf">SurveyWizard</span><span class="p">()</span> <span class="p">{</span>
  <span class="kd">const</span> <span class="nx">survey</span> <span class="o">=</span> <span class="nf">useMemo</span><span class="p">(()</span> <span class="o">=&gt;</span> <span class="k">new</span> <span class="nc">Model</span><span class="p">(</span><span class="nx">surveyJson</span><span class="p">),</span> <span class="p">[]);</span>

  <span class="k">return</span> <span class="o">&lt;</span><span class="nx">Survey</span> <span class="nx">model</span><span class="o">=</span><span class="p">{</span><span class="nx">survey</span><span class="p">}</span> <span class="sr">/&gt;</span><span class="err">;
</span><span class="p">}</span>
</code></pre>

</div>



<h3>
  
  
  Angular
</h3>



<div class="highlight js-code-highlight">
<pre class="highlight typescript"><code><span class="k">import</span> <span class="p">{</span> <span class="nx">Component</span> <span class="p">}</span> <span class="k">from</span> <span class="dl">"</span><span class="s2">@angular/core</span><span class="dl">"</span><span class="p">;</span>
<span class="k">import</span> <span class="p">{</span> <span class="nx">Model</span> <span class="p">}</span> <span class="k">from</span> <span class="dl">"</span><span class="s2">survey-core</span><span class="dl">"</span><span class="p">;</span>

<span class="p">@</span><span class="nd">Component</span><span class="p">({</span>
  <span class="na">selector</span><span class="p">:</span> <span class="dl">"</span><span class="s2">app-root</span><span class="dl">"</span><span class="p">,</span>
  <span class="na">template</span><span class="p">:</span> <span class="s2">`&lt;survey [model]="survey"&gt;&lt;/survey&gt;`</span>
<span class="p">})</span>
<span class="k">export</span> <span class="kd">class</span> <span class="nc">AppComponent</span> <span class="p">{</span>
  <span class="nx">survey</span> <span class="o">=</span> <span class="k">new</span> <span class="nc">Model</span><span class="p">(</span><span class="nx">surveyJson</span><span class="p">);</span>
<span class="p">}</span>
</code></pre>

</div>



<p>And in your Angular module:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight typescript"><code><span class="k">import</span> <span class="p">{</span> <span class="nx">NgModule</span> <span class="p">}</span> <span class="k">from</span> <span class="dl">"</span><span class="s2">@angular/core</span><span class="dl">"</span><span class="p">;</span>
<span class="k">import</span> <span class="p">{</span> <span class="nx">BrowserModule</span> <span class="p">}</span> <span class="k">from</span> <span class="dl">"</span><span class="s2">@angular/platform-browser</span><span class="dl">"</span><span class="p">;</span>
<span class="k">import</span> <span class="p">{</span> <span class="nx">SurveyModule</span> <span class="p">}</span> <span class="k">from</span> <span class="dl">"</span><span class="s2">survey-angular-ui</span><span class="dl">"</span><span class="p">;</span>

<span class="k">import</span> <span class="p">{</span> <span class="nx">AppComponent</span> <span class="p">}</span> <span class="k">from</span> <span class="dl">"</span><span class="s2">./app.component</span><span class="dl">"</span><span class="p">;</span>

<span class="p">@</span><span class="nd">NgModule</span><span class="p">({</span>
  <span class="na">declarations</span><span class="p">:</span> <span class="p">[</span><span class="nx">AppComponent</span><span class="p">],</span>
  <span class="na">imports</span><span class="p">:</span> <span class="p">[</span><span class="nx">BrowserModule</span><span class="p">,</span> <span class="nx">SurveyModule</span><span class="p">],</span>
  <span class="na">bootstrap</span><span class="p">:</span> <span class="p">[</span><span class="nx">AppComponent</span><span class="p">]</span>
<span class="p">})</span>
<span class="k">export</span> <span class="kd">class</span> <span class="nc">AppModule</span> <span class="p">{}</span>
</code></pre>

</div>



<h3>
  
  
  Vue
</h3>



<div class="highlight js-code-highlight">
<pre class="highlight javascript"><code><span class="o">&lt;</span><span class="nx">script</span> <span class="nx">setup</span> <span class="nx">lang</span><span class="o">=</span><span class="dl">"</span><span class="s2">ts</span><span class="dl">"</span><span class="o">&gt;</span>
<span class="k">import</span> <span class="dl">"</span><span class="s2">survey-core/survey-core.css</span><span class="dl">"</span><span class="p">;</span>
<span class="k">import</span> <span class="p">{</span> <span class="nx">Model</span> <span class="p">}</span> <span class="k">from</span> <span class="dl">"</span><span class="s2">survey-core</span><span class="dl">"</span><span class="p">;</span>
<span class="k">import</span> <span class="p">{</span> <span class="nx">SurveyComponent</span> <span class="p">}</span> <span class="k">from</span> <span class="dl">"</span><span class="s2">survey-vue3-ui</span><span class="dl">"</span><span class="p">;</span>

<span class="kd">const</span> <span class="nx">survey</span> <span class="o">=</span> <span class="k">new</span> <span class="nc">Model</span><span class="p">(</span><span class="nx">surveyJson</span><span class="p">);</span>
<span class="o">&lt;</span><span class="sr">/script</span><span class="err">&gt;
</span>
<span class="o">&lt;</span><span class="nx">template</span><span class="o">&gt;</span>
  <span class="o">&lt;</span><span class="nx">SurveyComponent</span> <span class="p">:</span><span class="nx">model</span><span class="o">=</span><span class="dl">"</span><span class="s2">survey</span><span class="dl">"</span> <span class="o">/&gt;</span>
<span class="o">&lt;</span><span class="sr">/template</span><span class="err">&gt;
</span></code></pre>

</div>



<h3>
  
  
  Plain JavaScript
</h3>



<div class="highlight js-code-highlight">
<pre class="highlight html"><code><span class="cp">&lt;!DOCTYPE html&gt;</span>
<span class="nt">&lt;html</span> <span class="na">lang=</span><span class="s">"en"</span><span class="nt">&gt;</span>
<span class="nt">&lt;head&gt;</span>
  <span class="nt">&lt;meta</span> <span class="na">charset=</span><span class="s">"utf-8"</span> <span class="nt">/&gt;</span>
  <span class="nt">&lt;title&gt;</span>SurveyJS Form Wizard<span class="nt">&lt;/title&gt;</span>

  <span class="nt">&lt;link</span>
    <span class="na">rel=</span><span class="s">"stylesheet"</span>
    <span class="na">href=</span><span class="s">"https://unpkg.com/survey-core/survey-core.min.css"</span>
  <span class="nt">/&gt;</span>
  <span class="nt">&lt;script </span><span class="na">src=</span><span class="s">"https://unpkg.com/survey-core/survey.core.min.js"</span><span class="nt">&gt;&lt;/script&gt;</span>
  <span class="nt">&lt;script </span><span class="na">src=</span><span class="s">"https://unpkg.com/survey-js-ui/survey-js-ui.min.js"</span><span class="nt">&gt;&lt;/script&gt;</span>
<span class="nt">&lt;/head&gt;</span>
<span class="nt">&lt;body&gt;</span>
  <span class="nt">&lt;div</span> <span class="na">id=</span><span class="s">"surveyContainer"</span><span class="nt">&gt;&lt;/div&gt;</span>

  <span class="nt">&lt;script&gt;</span>
    <span class="kd">const</span> <span class="nx">surveyJson</span> <span class="o">=</span> <span class="p">{</span>
      <span class="cm">/* ... */</span>
    <span class="p">};</span>

    <span class="kd">const</span> <span class="nx">survey</span> <span class="o">=</span> <span class="k">new</span> <span class="nx">Survey</span><span class="p">.</span><span class="nc">Model</span><span class="p">(</span><span class="nx">surveyJson</span><span class="p">);</span>

    <span class="nb">document</span><span class="p">.</span><span class="nf">addEventListener</span><span class="p">(</span><span class="dl">"</span><span class="s2">DOMContentLoaded</span><span class="dl">"</span><span class="p">,</span> <span class="nf">function </span><span class="p">()</span> <span class="p">{</span>
      <span class="nx">survey</span><span class="p">.</span><span class="nf">render</span><span class="p">(</span><span class="nb">document</span><span class="p">.</span><span class="nf">getElementById</span><span class="p">(</span><span class="dl">"</span><span class="s2">surveyContainer</span><span class="dl">"</span><span class="p">));</span>
    <span class="p">});</span>
  <span class="nt">&lt;/script&gt;</span>
<span class="nt">&lt;/body&gt;</span>
<span class="nt">&lt;/html&gt;</span>
</code></pre>

</div>



<h2>
  
  
  Final thoughts
</h2>

<p>A form wizard is not just a UI convenience. It is a runtime model for managing long-form input: sequence, validation, conditional branching, and completion. SurveyJS is useful here because those concerns live in the schema instead of being hand-coded into each frontend implementation.</p>

<p>That design has two practical benefits. First, the form definition becomes portable across frameworks. Second, the logic is easier to review, version, and maintain because it is expressed declaratively in JSON rather than spread across multiple components and event handlers.</p>

<p>If your goal is to ship a multi-step form quickly without losing control over validation and logic, that is a solid architecture.</p>

