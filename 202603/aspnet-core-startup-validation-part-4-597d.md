---
Title: ASP.NET Core startup validation part 4
Description: 
Author: Karen Payne
Date: 2026-03-22T22:10:30.000Z
Robots: noindex,nofollow
Template: index
---
<h2>
  
  
  Introduction
</h2>

<p>Learn how to use a class that implements <a href="https://learn.microsoft.com/en-us/dotnet/api/microsoft.extensions.options.ivalidateoptions-1?view=net-10.0-pp" rel="noopener noreferrer">IValidateOptions</a> to validate that sections exist with the required keys in the appsettings.json file.</p>

<p><a href="https://github.com/karenpayneoregon/learning-topics/tree/master/GetSettingFromAppSettings" class="crayons-btn crayons-btn--primary" rel="noopener noreferrer">Source code</a>
</p>

<p>Source code requires Microsoft Visual Studion 2022 or higher.</p>

<h2>
  
  
  appsettings.json
</h2>



<div class="highlight js-code-highlight">
<pre class="highlight json"><code><span class="p">{</span><span class="w">
  </span><span class="nl">"Logging"</span><span class="p">:</span><span class="w"> </span><span class="p">{</span><span class="w">
    </span><span class="nl">"LogLevel"</span><span class="p">:</span><span class="w"> </span><span class="p">{</span><span class="w">
      </span><span class="nl">"Microsoft.EntityFrameworkCore.Database.Command"</span><span class="p">:</span><span class="w"> </span><span class="s2">"Information"</span><span class="p">,</span><span class="w">
      </span><span class="nl">"Default"</span><span class="p">:</span><span class="w"> </span><span class="s2">"Information"</span><span class="p">,</span><span class="w">
      </span><span class="nl">"Microsoft.AspNetCore"</span><span class="p">:</span><span class="w"> </span><span class="s2">"Warning"</span><span class="w">
    </span><span class="p">}</span><span class="w">
  </span><span class="p">},</span><span class="w">
  </span><span class="nl">"AllowedHosts"</span><span class="p">:</span><span class="w"> </span><span class="s2">"*"</span><span class="p">,</span><span class="w">
  </span><span class="nl">"Helpdesk"</span><span class="p">:</span><span class="w"> </span><span class="p">{</span><span class="w">
    </span><span class="nl">"Phone"</span><span class="p">:</span><span class="w"> </span><span class="s2">"123-456-7890"</span><span class="p">,</span><span class="w">
    </span><span class="nl">"Email"</span><span class="p">:</span><span class="w"> </span><span class="s2">"helpdesk@example.com"</span><span class="w">
  </span><span class="p">}</span><span class="w">
</span><span class="p">}</span><span class="w">
</span></code></pre>

</div>



<p>In the file, we want to validate that both Logging and HelpDesk are validated.</p>

<h3>
  
  
  Required classes
</h3>

<p>💡 properties must be nullable for validation code to work.<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight csharp"><code><span class="k">public</span> <span class="k">class</span> <span class="nc">LogLevelSettings</span>
<span class="p">{</span>
    <span class="k">public</span> <span class="kt">string</span><span class="p">?</span> <span class="n">Default</span> <span class="p">{</span> <span class="k">get</span><span class="p">;</span> <span class="k">set</span><span class="p">;</span> <span class="p">}</span> <span class="p">=</span> <span class="kt">string</span><span class="p">.</span><span class="n">Empty</span><span class="p">;</span>

    <span class="p">[</span><span class="nf">ConfigurationKeyName</span><span class="p">(</span><span class="s">"Microsoft.AspNetCore"</span><span class="p">)]</span>
    <span class="k">public</span> <span class="kt">string</span><span class="p">?</span> <span class="n">MicrosoftAspNetCore</span> <span class="p">{</span> <span class="k">get</span><span class="p">;</span> <span class="k">set</span><span class="p">;</span> <span class="p">}</span> <span class="p">=</span> <span class="kt">string</span><span class="p">.</span><span class="n">Empty</span><span class="p">;</span>

    <span class="p">[</span><span class="nf">ConfigurationKeyName</span><span class="p">(</span><span class="s">"Microsoft.EntityFrameworkCore.Database.Command"</span><span class="p">)]</span>
    <span class="k">public</span> <span class="kt">string</span><span class="p">?</span> <span class="n">MicrosoftEntityFrameworkCoreDatabaseCommand</span> <span class="p">{</span> <span class="k">get</span><span class="p">;</span> <span class="k">set</span><span class="p">;</span> <span class="p">}</span> <span class="p">=</span> <span class="kt">string</span><span class="p">.</span><span class="n">Empty</span><span class="p">;</span>
<span class="p">}</span>

<span class="k">public</span> <span class="k">class</span> <span class="nc">HelpDesk</span>
<span class="p">{</span>
    <span class="k">public</span> <span class="kt">string</span><span class="p">?</span> <span class="n">Phone</span> <span class="p">{</span> <span class="k">get</span><span class="p">;</span> <span class="k">set</span><span class="p">;</span> <span class="p">}</span>
    <span class="k">public</span> <span class="kt">string</span><span class="p">?</span> <span class="n">Email</span> <span class="p">{</span> <span class="k">get</span><span class="p">;</span> <span class="k">set</span><span class="p">;</span> <span class="p">}</span>
<span class="p">}</span>
</code></pre>

</div>



<h2>
  
  
  _Layout.cshtml
</h2>

<p>Note that the following lines are needed for displaying HelpDesk values<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight html"><code>@using GetSettingFromAppSettings.Models
@using Microsoft.Extensions.Options
@inject IOptions<span class="nt">&lt;HelpDesk&gt;</span> HelpdeskOptions
</code></pre>

</div>



<h2>
  
  
  Validation class
</h2>

<p>Responsible for the validation of a section in the above appsettings file.</p>

<ul>
<li>The <strong>Validate</strong> method checks:

<ul>
<li>For required sections</li>
<li>Calls <strong>ValidateValue</strong> to check that the required keys exist</li>
</ul>


</li>

</ul>

<p>If errors list is not empty indicates failure.<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight csharp"><code><span class="k">public</span> <span class="k">sealed</span> <span class="k">class</span> <span class="nc">LoggingSettingsValidation</span><span class="p">(</span><span class="n">IConfiguration</span> <span class="n">configuration</span><span class="p">)</span> <span class="p">:</span> 
    <span class="n">IValidateOptions</span><span class="p">&lt;</span><span class="n">LoggingSettings</span><span class="p">&gt;</span>
<span class="p">{</span>

    <span class="k">public</span> <span class="n">ValidateOptionsResult</span> <span class="nf">Validate</span><span class="p">(</span><span class="kt">string</span><span class="p">?</span> <span class="n">name</span><span class="p">,</span> <span class="n">LoggingSettings</span> <span class="n">options</span><span class="p">)</span>
    <span class="p">{</span>
        <span class="n">List</span><span class="p">&lt;</span><span class="kt">string</span><span class="p">&gt;</span> <span class="n">errors</span> <span class="p">=</span> <span class="p">[];</span>

        <span class="kt">var</span> <span class="n">loggingSection</span> <span class="p">=</span> <span class="n">configuration</span><span class="p">.</span><span class="nf">GetSection</span><span class="p">(</span><span class="s">"Logging"</span><span class="p">);</span>
        <span class="k">if</span> <span class="p">(!</span><span class="n">loggingSection</span><span class="p">.</span><span class="nf">Exists</span><span class="p">())</span>
        <span class="p">{</span>
            <span class="n">errors</span><span class="p">.</span><span class="nf">Add</span><span class="p">(</span><span class="s">"Configuration section 'Logging' is missing."</span><span class="p">);</span>
            <span class="k">return</span> <span class="n">ValidateOptionsResult</span><span class="p">.</span><span class="nf">Fail</span><span class="p">(</span><span class="n">errors</span><span class="p">);</span>
        <span class="p">}</span>

        <span class="kt">var</span> <span class="n">logLevelSection</span> <span class="p">=</span> <span class="n">configuration</span><span class="p">.</span><span class="nf">GetSection</span><span class="p">(</span><span class="s">"Logging:LogLevel"</span><span class="p">);</span>
        <span class="k">if</span> <span class="p">(!</span><span class="n">logLevelSection</span><span class="p">.</span><span class="nf">Exists</span><span class="p">())</span>
        <span class="p">{</span>
            <span class="n">errors</span><span class="p">.</span><span class="nf">Add</span><span class="p">(</span><span class="s">"Configuration section 'Logging:LogLevel' is missing."</span><span class="p">);</span>
            <span class="k">return</span> <span class="n">ValidateOptionsResult</span><span class="p">.</span><span class="nf">Fail</span><span class="p">(</span><span class="n">errors</span><span class="p">);</span>
        <span class="p">}</span>

        <span class="nf">ValidateValue</span><span class="p">(</span><span class="s">"Logging:LogLevel:Default"</span><span class="p">,</span> <span class="n">errors</span><span class="p">);</span>
        <span class="nf">ValidateValue</span><span class="p">(</span><span class="s">"Logging:LogLevel:Microsoft.AspNetCore"</span><span class="p">,</span> <span class="n">errors</span><span class="p">);</span>
        <span class="nf">ValidateValue</span><span class="p">(</span><span class="s">"Logging:LogLevel:Microsoft.EntityFrameworkCore.Database.Command"</span><span class="p">,</span> <span class="n">errors</span><span class="p">);</span>

        <span class="k">return</span> <span class="n">errors</span><span class="p">.</span><span class="n">Count</span> <span class="p">&gt;</span> <span class="m">0</span>
            <span class="p">?</span> <span class="n">ValidateOptionsResult</span><span class="p">.</span><span class="nf">Fail</span><span class="p">(</span><span class="n">errors</span><span class="p">)</span>
            <span class="p">:</span> <span class="n">ValidateOptionsResult</span><span class="p">.</span><span class="n">Success</span><span class="p">;</span>
    <span class="p">}</span>


    <span class="k">private</span> <span class="k">void</span> <span class="nf">ValidateValue</span><span class="p">(</span><span class="kt">string</span> <span class="n">key</span><span class="p">,</span> <span class="n">List</span><span class="p">&lt;</span><span class="kt">string</span><span class="p">&gt;</span> <span class="n">errors</span><span class="p">)</span>
    <span class="p">{</span>
        <span class="kt">var</span> <span class="k">value</span> <span class="p">=</span> <span class="n">configuration</span><span class="p">[</span><span class="n">key</span><span class="p">];</span>

        <span class="k">if</span> <span class="p">(</span><span class="kt">string</span><span class="p">.</span><span class="nf">IsNullOrWhiteSpace</span><span class="p">(</span><span class="k">value</span><span class="p">))</span>
        <span class="p">{</span>
            <span class="n">errors</span><span class="p">.</span><span class="nf">Add</span><span class="p">(</span><span class="s">$"Configuration value '</span><span class="p">{</span><span class="n">key</span><span class="p">}</span><span class="s">' is missing or empty."</span><span class="p">);</span>
        <span class="p">}</span>
    <span class="p">}</span>
<span class="p">}</span>
</code></pre>

</div>



<h2>
  
  
  Program.cs code
</h2>

<p>The following code is used to invoke <strong>LoggingSettingsValidation</strong> assertions.<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight csharp"><code><span class="n">builder</span><span class="p">.</span><span class="n">Services</span>
    <span class="p">.</span><span class="n">AddOptions</span><span class="p">&lt;</span><span class="n">LoggingSettings</span><span class="p">&gt;()</span>
    <span class="p">.</span><span class="nf">Bind</span><span class="p">(</span><span class="n">builder</span><span class="p">.</span><span class="n">Configuration</span><span class="p">.</span><span class="nf">GetSection</span><span class="p">(</span><span class="s">"Logging"</span><span class="p">))</span>
    <span class="p">.</span><span class="nf">ValidateOnStart</span><span class="p">();</span>

<span class="n">builder</span><span class="p">.</span><span class="n">Services</span><span class="p">.</span><span class="n">AddSingleton</span><span class="p">&lt;</span><span class="n">IValidateOptions</span><span class="p">&lt;</span><span class="n">LoggingSettings</span><span class="p">&gt;,</span> <span class="n">LoggingSettingsValidation</span><span class="p">&gt;();</span>
</code></pre>

</div>



<h2>
  
  
  See also
</h2>

<ul>
<li><a href="https://dev.to/karenpayneoregon/aspnet-core-startup-validation-20e7">Part 1</a></li>
<li><a href="https://dev.to/karenpayneoregon/aspnet-core-startup-validation-part-2-3f7m">Part 2</a></li>
<li><a href="https://dev.to/karenpayneoregon/asp-net-core-startup-validation-part-3-jh9">Part 3</a></li>
</ul>

<h2>
  
  
  Summary
</h2>

<p>Some developers may think the code presented is great, while others might consider the validation presented overkill. It depends on how the code is tested and if there are steps to manually check that sections and keys exist. To keep a web application running correctly, test and use the provided code.</p>

<h3>
  
  
  Optional
</h3>

<p>If more than one web application requires appsetting validation, consider moving the code to a class project compiled to a local NuGet package for consistency and maintenance.</p>

