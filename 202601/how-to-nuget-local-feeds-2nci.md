---
Title: How to: NuGet local feeds
Description: 
Author: Karen Payne
Date: 2026-01-10T21:49:04.000Z
Robots: noindex,nofollow
Template: index
---
<h2>
  
  
  Introduction
</h2>

<p><a href="https://learn.microsoft.com/en-us/nuget/what-is-nuget" rel="noopener noreferrer">NuGet packages</a> are the standard way .NET developers share, version, and <a href="https://learn.microsoft.com/en-us/nuget/consume-packages/overview-and-workflow" rel="noopener noreferrer">consume</a> reusable code, whether it’s core libraries, third-party tools, or internal company components. A NuGet package is essentially a zipped bundle that contains compiled assemblies, metadata, and dependency information, which allows Visual Studio and the .NET CLI to automatically restore the correct versions your project needs. This system has been around for years, and it works because it enforces consistency: you no longer copy DLLs by hand; instead, you declare what you depend on and let NuGet manage updates, conflicts, and builds in a predictable and repeatable way.</p>

<p>In this article, learn how to write code to publish to a local source.</p>

<h2>
  
  
  Individual accounts on NuGet.org
</h2>

<p>You must create an <a href="https://learn.microsoft.com/en-us/nuget/nuget-org/individual-accounts" rel="noopener noreferrer">individual account</a> to publish and manage packages on NuGet.org.</p>

<h2>
  
  
  Local feed/source
</h2>

<p>A NuGet local feed is simply a folder on your machine or network that NuGet treats as a package source, allowing you to create, store, and consume NuGet packages without publishing them to a remote repository like nuget.org.</p>

<p>Local feeds are fast, offline-friendly, and ideal for in-house or experimental packages; however, they lack the governance, versioning discipline, and auditability that you get from a proper package server.</p>

<h2>
  
  
  When to create local packages
</h2>

<ul>
<li>A developer uses the same code over and over again.</li>
<li>The code is proprietary to a developer or an organization.</li>
<li>Easy to maintain and allows multiple versions to be available.</li>
</ul>

<h3>
  
  
  Starter example 1
</h3>

<p>A  developer or team of developers needs to create a type handler for <a href="https://www.nuget.org/packages/Dapper" rel="noopener noreferrer">Dapper</a> to understand <a href="https://learn.microsoft.com/en-us/dotnet/api/system.dateonly?view=net-10.0" rel="noopener noreferrer">DateOnly</a> and <a href="https://learn.microsoft.com/en-us/dotnet/api/system.timeonly?view=net-10.0" rel="noopener noreferrer">TimeOnly</a> structs. This would be a good example which in this case could be local or public as <a href="https://www.nuget.org/packages/kp.Dapper.Handlers/" rel="noopener noreferrer">shown here</a>.<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight csharp"><code><span class="k">public</span> <span class="k">class</span> <span class="nc">SqlDateOnlyTypeHandler</span> <span class="p">:</span> <span class="n">SqlMapper</span><span class="p">.</span><span class="n">TypeHandler</span><span class="p">&lt;</span><span class="n">DateOnly</span><span class="p">&gt;</span>
<span class="p">{</span>
    <span class="k">public</span> <span class="k">override</span> <span class="k">void</span> <span class="nf">SetValue</span><span class="p">(</span><span class="n">IDbDataParameter</span> <span class="n">parameter</span><span class="p">,</span> <span class="n">DateOnly</span> <span class="n">date</span><span class="p">)</span> 
        <span class="p">=&gt;</span> <span class="n">parameter</span><span class="p">.</span><span class="n">Value</span> <span class="p">=</span> <span class="n">date</span><span class="p">.</span><span class="nf">ToDateTime</span><span class="p">(</span><span class="k">new</span> <span class="nf">TimeOnly</span><span class="p">(</span><span class="m">0</span><span class="p">,</span> <span class="m">0</span><span class="p">));</span>
    <span class="k">public</span> <span class="k">override</span> <span class="n">DateOnly</span> <span class="nf">Parse</span><span class="p">(</span><span class="kt">object</span> <span class="k">value</span><span class="p">)</span> <span class="p">=&gt;</span> <span class="n">DateOnly</span><span class="p">.</span><span class="nf">FromDateTime</span><span class="p">((</span><span class="n">DateTime</span><span class="p">)</span><span class="k">value</span><span class="p">);</span>
<span class="p">}</span>

<span class="k">public</span> <span class="k">class</span> <span class="nc">SqlTimeOnlyTypeHandler</span> <span class="p">:</span> <span class="n">SqlMapper</span><span class="p">.</span><span class="n">TypeHandler</span><span class="p">&lt;</span><span class="n">TimeOnly</span><span class="p">&gt;</span>
<span class="p">{</span>
    <span class="k">public</span> <span class="k">override</span> <span class="k">void</span> <span class="nf">SetValue</span><span class="p">(</span><span class="n">IDbDataParameter</span> <span class="n">parameter</span><span class="p">,</span> <span class="n">TimeOnly</span> <span class="n">time</span><span class="p">)</span>
    <span class="p">{</span>
        <span class="n">parameter</span><span class="p">.</span><span class="n">Value</span> <span class="p">=</span> <span class="n">time</span><span class="p">.</span><span class="nf">ToString</span><span class="p">();</span>
    <span class="p">}</span>

    <span class="k">public</span> <span class="k">override</span> <span class="n">TimeOnly</span> <span class="nf">Parse</span><span class="p">(</span><span class="kt">object</span> <span class="k">value</span><span class="p">)</span> <span class="p">=&gt;</span> <span class="n">TimeOnly</span><span class="p">.</span><span class="nf">FromTimeSpan</span><span class="p">((</span><span class="n">TimeSpan</span><span class="p">)</span><span class="k">value</span><span class="p">);</span>
<span class="p">}</span>
</code></pre>

</div>



<h3>
  
  
  Starter example 2
</h3>

<p>A developer or a team develops a proprietary API that is solely used internally. These packages need to be protected as a seasoned developer, or a hacker can easily get to the package source code. A package file is just as easy to hack as a .DLL file.</p>

<h2>
  
  
  Publishing to Local feed/source tool
</h2>

<p>The purpose of a tool is to copy a package file to a local feed folder for personal use or for a team of developers.</p>

<p><a href="https://github.com/karenpayneoregon/learning-topics/tree/master/Publisher" class="ltag_cta ltag_cta--branded" rel="noopener noreferrer">Source code</a>
</p>

<p><strong>Screen shot</strong></p>

<p><a href="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fdjlto5vx9quzgsthjegb.png" class="article-body-image-wrapper"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fdjlto5vx9quzgsthjegb.png" alt="The tool" width="469" height="344"></a></p>

<h3>
  
  
  Using the tool
</h3>

<p>Select a <code>package</code>, then select a <code>version</code> of the package. Click the <code>Publish button</code>, which presents a confirmation dialog to continue. If a package exists, it will be overwritten when clicking yes in the confirmation dialog.</p>

<h3>
  
  
  Configuration
</h3>

<p>To configure the tool: open <code>appsettings.json</code></p>

<p><strong>PackageLocation</strong> points to the folder where your local NuGet packages are stored for Microsoft Visual Studio.</p>

<p>Visual Studio options dialog for NuGet package sources<br>
<a href="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fz44mqsj0pbp42vb9w1z8.png" class="article-body-image-wrapper"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fz44mqsj0pbp42vb9w1z8.png" alt="Visual Studio options dialog for NuGet package sources" width="743" height="486"></a></p>

<p><strong>ClassProjects section</strong>: In this section, add one or more paths to project folders that contain one or more <code>.nupkg files</code>.</p>

<ul>
<li>A path points to a project folder not the path to nupkg files.</li>
<li>Paths can be in one or more Visual Solutions.</li>
</ul>

<p>Example appsettings.json file with five paths; each path in <code>ClassProjects</code> points to a project under different Visual Studio solutions.<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight json"><code><span class="p">{</span><span class="w">
  </span><span class="nl">"PackageLocation"</span><span class="p">:</span><span class="w"> </span><span class="s2">"C:</span><span class="se">\\</span><span class="s2">NuGetLocal"</span><span class="p">,</span><span class="w">
  </span><span class="nl">"ProjectItems"</span><span class="p">:</span><span class="w"> </span><span class="p">[</span><span class="w">
    </span><span class="p">{</span><span class="w">
      </span><span class="nl">"Path"</span><span class="p">:</span><span class="w"> </span><span class="s2">"C:</span><span class="se">\\</span><span class="s2">DotnetLand</span><span class="se">\\</span><span class="s2">VS2022</span><span class="se">\\</span><span class="s2">ProjectTemplates2025</span><span class="se">\\</span><span class="s2">SpectreConsoleLibrary"</span><span class="w">
    </span><span class="p">},</span><span class="w">
    </span><span class="p">{</span><span class="w">
      </span><span class="nl">"Path"</span><span class="p">:</span><span class="w"> </span><span class="s2">"C:</span><span class="se">\\</span><span class="s2">DotnetLand</span><span class="se">\\</span><span class="s2">VS2022</span><span class="se">\\</span><span class="s2">BogusTrainingSolution</span><span class="se">\\</span><span class="s2">BogusLibrary"</span><span class="w">
    </span><span class="p">},</span><span class="w">
    </span><span class="p">{</span><span class="w">
      </span><span class="nl">"Path"</span><span class="p">:</span><span class="w"> </span><span class="s2">"C:</span><span class="se">\\</span><span class="s2">DotnetLand</span><span class="se">\\</span><span class="s2">VS2022</span><span class="se">\\</span><span class="s2">SerilogSolution</span><span class="se">\\</span><span class="s2">SeriLogThemesLibrary"</span><span class="w">
    </span><span class="p">},</span><span class="w">
    </span><span class="p">{</span><span class="w">
      </span><span class="nl">"Path"</span><span class="p">:</span><span class="w"> </span><span class="s2">"C:</span><span class="se">\\</span><span class="s2">DotnetLand</span><span class="se">\\</span><span class="s2">VS2022</span><span class="se">\\</span><span class="s2">ConsoleHelpersSolution</span><span class="se">\\</span><span class="s2">ConsoleConfigurationLibrary"</span><span class="w">
    </span><span class="p">},</span><span class="w">
    </span><span class="p">{</span><span class="w">
      </span><span class="nl">"Path"</span><span class="p">:</span><span class="w"> </span><span class="s2">"c:</span><span class="se">\\</span><span class="s2">DotnetLand</span><span class="se">\\</span><span class="s2">VS2019</span><span class="se">\\</span><span class="s2">dapper-libraries</span><span class="se">\\</span><span class="s2">Dapper.Handlers"</span><span class="w">
    </span><span class="p">}</span><span class="w">
  </span><span class="p">]</span><span class="w">
</span><span class="p">}</span><span class="w">
</span></code></pre>

</div>



<h3>
  
  
  Important
</h3>

<p>💡 The method <code>DirectoryHelper.ProjectItems</code> traverses folders specified from <code>ClassProjects</code> in <code>appsettings.json</code>. Only paths that exist and contain at least one .nupkg will be displayed in the tool.</p>

<h2>
  
  
  Summary
</h2>

<p>Instructions and why a developer or team of developers may want to use local NuGet packages along with a tool to copy packages to a local feed folder in this article to assist with working with local NuGet packages.</p>

