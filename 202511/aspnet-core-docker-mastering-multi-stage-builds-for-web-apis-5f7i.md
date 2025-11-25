---
Title: ASP.NET Core + Docker: Mastering Multi-Stage Builds for Web APIs
Description: 
Author: Cristian Sifuentes
Date: 2025-11-25T21:02:39.000Z
Robots: noindex,nofollow
Template: index
---
<p><a href="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fu3glxr70yq5m0k0231ng.png" class="article-body-image-wrapper"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fu3glxr70yq5m0k0231ng.png" alt="ASP.NET Core + Docker: Mastering Multi‑Stage Builds for Web APIs" width="800" height="800"></a></p>

<h2>
  
  
  ASP.NET Core + Docker: Mastering Multi‑Stage Builds for Web APIs
</h2>

<blockquote>
<p>“Visual Studio gave me this Dockerfile… but what is it actually doing?”</p>
</blockquote>

<p>If you’ve ever right‑clicked <strong>“Add &gt; Docker Support”</strong> in an ASP.NET project, you’ve probably seen a fairly complex <strong>multi‑stage Dockerfile</strong> appear in your repo.</p>

<p>It looks smart. It builds. It even runs in Debug.<br><br>
But:</p>

<ul>
<li>What does each <strong>stage</strong> really do?</li>
<li>What do you actually need installed to make it work?</li>
<li>Why is there a mysterious <code>USER $APP_UID</code> line?</li>
<li>How do you safely build and run the final image in your own environment?</li>
</ul>

<p>This guide takes a <strong>real Dockerfile</strong> for an ASP.NET Core Web API and turns it into a <strong>clear mental model</strong> you can reuse in any .NET + Docker project.</p>


<h2>
  
  
  TL;DR — What You’ll Learn
</h2>

<p>✅ How a multi‑stage Dockerfile for ASP.NET Core is structured (base → build → publish → final)<br><br>
✅ What the <code>USER $APP_UID</code> line does and how to avoid permission problems<br><br>
✅ What you <em>actually</em> need installed to build and run this image<br><br>
✅ How to build and run the container step by step<br><br>
✅ Why aligning <code>aspnet:9.0</code> and <code>sdk:10.0</code> versions matters<br><br>
✅ A checklist to verify “yes, I can run this Dockerfile in my environment”</p>

<p>Copy‑paste friendly commands included. Let’s dissect this thing. 🪓</p>


<h2>
  
  
  1. The Dockerfile We’re Analyzing (Big Picture)
</h2>

<p>Here’s the multi‑stage Dockerfile, slightly formatted:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight docker"><code><span class="c"># Base runtime stage (used for running the app)</span>
<span class="k">FROM</span><span class="w"> </span><span class="s">mcr.microsoft.com/dotnet/aspnet:9.0</span><span class="w"> </span><span class="k">AS</span><span class="w"> </span><span class="s">base</span>
<span class="k">USER</span><span class="s"> $APP_UID</span>
<span class="k">WORKDIR</span><span class="s"> /app</span>
<span class="k">EXPOSE</span><span class="s"> 8080</span>
<span class="k">EXPOSE</span><span class="s"> 8081</span>

<span class="c"># Build stage (used to compile the project)</span>
<span class="k">FROM</span><span class="w"> </span><span class="s">mcr.microsoft.com/dotnet/sdk:10.0</span><span class="w"> </span><span class="k">AS</span><span class="w"> </span><span class="s">build</span>
<span class="k">ARG</span><span class="s"> BUILD_CONFIGURATION=Release</span>
<span class="k">WORKDIR</span><span class="s"> /src</span>
<span class="k">COPY</span><span class="s"> ["Directory.Packages.props", "."]</span>
<span class="k">COPY</span><span class="s"> ["Directory.Build.props", "."]</span>
<span class="k">COPY</span><span class="s"> ["Web.Api/Web.Api.csproj", "Web.Api/"]</span>
<span class="k">RUN </span>dotnet restore <span class="s2">"./Web.Api/Web.Api.csproj"</span>
<span class="k">COPY</span><span class="s"> . .</span>
<span class="k">WORKDIR</span><span class="s"> "/src/Web.Api"</span>
<span class="k">RUN </span>dotnet build <span class="s2">"./Web.Api.csproj"</span> <span class="nt">-c</span> <span class="nv">$BUILD_CONFIGURATION</span> <span class="nt">-o</span> /app/build

<span class="c"># Publish stage (produces the final published output)</span>
<span class="k">FROM</span><span class="w"> </span><span class="s">build</span><span class="w"> </span><span class="k">AS</span><span class="w"> </span><span class="s">publish</span>
<span class="k">ARG</span><span class="s"> BUILD_CONFIGURATION=Release</span>
<span class="k">RUN </span>dotnet publish <span class="s2">"./Web.Api.csproj"</span> <span class="nt">-c</span> <span class="nv">$BUILD_CONFIGURATION</span> <span class="nt">-o</span> /app/publish /p:UseAppHost<span class="o">=</span><span class="nb">false</span>

<span class="c"># Final runtime stage (what actually runs in prod)</span>
<span class="k">FROM</span><span class="w"> </span><span class="s">base</span><span class="w"> </span><span class="k">AS</span><span class="w"> </span><span class="s">final</span>
<span class="k">WORKDIR</span><span class="s"> /app</span>
<span class="k">COPY</span><span class="s"> --from=publish /app/publish .</span>
<span class="k">ENTRYPOINT</span><span class="s"> ["dotnet", "Web.Api.dll"]</span>
</code></pre>

</div>



<p>At a high level, this is a <strong>classic multi‑stage Dockerfile</strong>:</p>

<ol>
<li>
<strong><code>base</code></strong> → ASP.NET runtime, non‑root user, ports exposed
</li>
<li>
<strong><code>build</code></strong> → full .NET SDK, restores &amp; compiles your Web API
</li>
<li>
<strong><code>publish</code></strong> → takes the build output and publishes a trimmed app
</li>
<li>
<strong><code>final</code></strong> → runtime image + published app + clean entrypoint</li>
</ol>

<p>Let’s go stage by stage.</p>




<h2>
  
  
  2. Stage‑by‑Stage Breakdown
</h2>

<h3>
  
  
  2.1 Base Stage — Runtime Image
</h3>



<div class="highlight js-code-highlight">
<pre class="highlight docker"><code><span class="k">FROM</span><span class="w"> </span><span class="s">mcr.microsoft.com/dotnet/aspnet:9.0</span><span class="w"> </span><span class="k">AS</span><span class="w"> </span><span class="s">base</span>
<span class="k">USER</span><span class="s"> $APP_UID</span>
<span class="k">WORKDIR</span><span class="s"> /app</span>
<span class="k">EXPOSE</span><span class="s"> 8080</span>
<span class="k">EXPOSE</span><span class="s"> 8081</span>
</code></pre>

</div>



<p>What this does:</p>

<ul>
<li>Uses <strong>ASP.NET Core 9.0 runtime</strong> (Linux container).</li>
<li>Switches to a <strong>non‑root</strong> user via <code>USER $APP_UID</code>.</li>
<li>Sets <code>/app</code> as the working directory.</li>
<li>Exposes ports <strong>8080</strong> and <strong>8081</strong> (for HTTP/HTTPS or multiple endpoints).</li>
</ul>

<blockquote>
<p>Mental model:<br><br>
This is the <strong>“slim, production‑ready base”</strong> where your app will run. No SDK, just runtime + your files.</p>
</blockquote>




<h3>
  
  
  2.2 Build Stage — SDK Image for Compiling
</h3>



<div class="highlight js-code-highlight">
<pre class="highlight docker"><code><span class="k">FROM</span><span class="w"> </span><span class="s">mcr.microsoft.com/dotnet/sdk:10.0</span><span class="w"> </span><span class="k">AS</span><span class="w"> </span><span class="s">build</span>
<span class="k">ARG</span><span class="s"> BUILD_CONFIGURATION=Release</span>
<span class="k">WORKDIR</span><span class="s"> /src</span>
<span class="k">COPY</span><span class="s"> ["Directory.Packages.props", "."]</span>
<span class="k">COPY</span><span class="s"> ["Directory.Build.props", "."]</span>
<span class="k">COPY</span><span class="s"> ["Web.Api/Web.Api.csproj", "Web.Api/"]</span>
<span class="k">RUN </span>dotnet restore <span class="s2">"./Web.Api/Web.Api.csproj"</span>
<span class="k">COPY</span><span class="s"> . .</span>
<span class="k">WORKDIR</span><span class="s"> "/src/Web.Api"</span>
<span class="k">RUN </span>dotnet build <span class="s2">"./Web.Api.csproj"</span> <span class="nt">-c</span> <span class="nv">$BUILD_CONFIGURATION</span> <span class="nt">-o</span> /app/build
</code></pre>

</div>



<p>Key points:</p>

<ul>
<li>Uses <strong>.NET SDK 10.0</strong> (preview/future version) to <strong>restore and build</strong> the project.</li>
<li>Assumes these files exist in the <strong>build context</strong> (folder where you call <code>docker build</code>):
</li>
</ul>

<div class="highlight js-code-highlight">
<pre class="highlight plaintext"><code>  Directory.Packages.props
  Directory.Build.props
  Web.Api/Web.Api.csproj
  Web.Api/...
</code></pre>

</div>



<ul>
<li>Flow:

<ol>
<li>Copy minimal files (<code>.props</code> + <code>.csproj</code>) for faster restore caching.</li>
<li>
<code>dotnet restore</code> downloads all NuGet packages.</li>
<li>Copy the rest of the source (<code>COPY . .</code>).</li>
<li>Build the Web API project into <code>/app/build</code>.</li>
</ol>
</li>
</ul>

<blockquote>
<p>Mental model:<br><br>
This stage is your <strong>“build server inside a container”</strong>. It contains the full SDK and compiles your code, but it won’t be shipped as‑is to production.</p>
</blockquote>




<h3>
  
  
  2.3 Publish Stage — Final Output
</h3>



<div class="highlight js-code-highlight">
<pre class="highlight docker"><code><span class="k">FROM</span><span class="w"> </span><span class="s">build</span><span class="w"> </span><span class="k">AS</span><span class="w"> </span><span class="s">publish</span>
<span class="k">ARG</span><span class="s"> BUILD_CONFIGURATION=Release</span>
<span class="k">RUN </span>dotnet publish <span class="s2">"./Web.Api.csproj"</span> <span class="nt">-c</span> <span class="nv">$BUILD_CONFIGURATION</span> <span class="nt">-o</span> /app/publish /p:UseAppHost<span class="o">=</span><span class="nb">false</span>
</code></pre>

</div>



<p>What happens here:</p>

<ul>
<li>Reuses the <strong>build stage</strong> (SDK + source + dependencies).</li>
<li>Runs <code>dotnet publish</code> to generate an optimized output into <code>/app/publish</code>.</li>
<li>Uses <code>/p:UseAppHost=false</code> to avoid bundling a platform‑specific executable; you’ll run with <code>dotnet Web.Api.dll</code>.</li>
</ul>

<blockquote>
<p>Mental model:<br><br>
This stage transforms your compiled app into the <strong>final published bundle</strong> that will be copied into the runtime image.</p>
</blockquote>




<h3>
  
  
  2.4 Final Stage — What Actually Runs in Production
</h3>



<div class="highlight js-code-highlight">
<pre class="highlight docker"><code><span class="k">FROM</span><span class="w"> </span><span class="s">base</span><span class="w"> </span><span class="k">AS</span><span class="w"> </span><span class="s">final</span>
<span class="k">WORKDIR</span><span class="s"> /app</span>
<span class="k">COPY</span><span class="s"> --from=publish /app/publish .</span>
<span class="k">ENTRYPOINT</span><span class="s"> ["dotnet", "Web.Api.dll"]</span>
</code></pre>

</div>



<p>This is the stage that creates the <strong>image you actually run</strong>:</p>

<ul>
<li>Starts from the <code>base</code> runtime image (ASP.NET 9.0, non‑root user, ports exposed).</li>
<li>Copies the published output from the <code>publish</code> stage.</li>
<li>Sets the entrypoint to:
</li>
</ul>

<div class="highlight js-code-highlight">
<pre class="highlight shell"><code>  dotnet Web.Api.dll
</code></pre>

</div>



<blockquote>
<p>Mental model:<br><br>
Final image = <strong>runtime + published app</strong>. Small, clean, production‑oriented.</p>
</blockquote>




<h2>
  
  
  3. What You Must Have Installed to Build This Image
</h2>

<p>The good news: <strong>you don’t need .NET SDK installed on your host</strong> to build this image. The Dockerfile uses SDK images <em>inside</em> the container.</p>

<h3>
  
  
  Mandatory
</h3>

<ol>
<li>
<p><strong>Docker Engine / Docker Desktop</strong></p>

<ul>
<li>Windows, macOS, or Linux, with <strong>Linux containers</strong> enabled.</li>
</ul>
</li>
<li><p><strong>Correct project layout matching the Dockerfile</strong><br><br>
In the directory where you run <code>docker build</code>, you should see something like:<br>
</p></li>
</ol>

<div class="highlight js-code-highlight">
<pre class="highlight plaintext"><code>   Directory.Packages.props
   Directory.Build.props
   Web.Api/
       Web.Api.csproj
       Program.cs
       appsettings.json
       ...
   Dockerfile
</code></pre>

</div>



<ol>
<li>
<strong>Internet access (at least for the first build)</strong>
Docker must be able to pull:</li>
</ol>

<ul>
<li><code>mcr.microsoft.com/dotnet/aspnet:9.0</code></li>
<li>
<code>mcr.microsoft.com/dotnet/sdk:10.0</code> (or 9.0 if you align versions)</li>
<li>All NuGet packages during <code>dotnet restore</code>.</li>
</ul>

<h3>
  
  
  Optional (Nice to Have)
</h3>

<ul>
<li>
<strong>.NET SDK on your host</strong>
Only needed if you also want to run:
</li>
</ul>

<div class="highlight js-code-highlight">
<pre class="highlight shell"><code>  dotnet run
  dotnet <span class="nb">test</span>
</code></pre>

</div>



<p>directly on your machine. The Docker build itself doesn’t require it.</p>




<h2>
  
  
  4. The <code>USER $APP_UID</code> Trap (And How to Fix It)
</h2>

<p>This line lives in the base stage:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight docker"><code><span class="k">FROM</span><span class="w"> </span><span class="s">mcr.microsoft.com/dotnet/aspnet:9.0</span><span class="w"> </span><span class="k">AS</span><span class="w"> </span><span class="s">base</span>
<span class="k">USER</span><span class="s"> $APP_UID</span>
</code></pre>

</div>



<p>It tries to ensure your app <strong>does not run as root</strong> inside the container. Great for security.<br><br>
But there’s a catch:</p>

<p>For this to work correctly:</p>

<ul>
<li>The environment variable <strong><code>APP_UID</code></strong> must be set at build or runtime, <strong>and</strong>
</li>
<li>That UID must map to a <strong>valid user inside the container</strong>.</li>
</ul>

<p>If not, you can get:</p>

<ul>
<li>Permission errors
</li>
<li>“No such user” problems
</li>
<li>Confusing runtime failures</li>
</ul>
<h3>
  
  
  ✅ Option 1 — Easiest for Local Dev: Comment it Out
</h3>

<p>For local testing only, you can temporarily remove or comment the line:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight docker"><code><span class="c"># USER $APP_UID</span>
</code></pre>

</div>



<p>Your app will run as root inside the container, which is fine for <strong>dev</strong>, but not ideal for <strong>production security</strong>.</p>

<h3>
  
  
  ✅ Option 2 — Define and Create a Non‑Root User
</h3>

<p>Hardened, production‑friendly version:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight docker"><code><span class="k">FROM</span><span class="w"> </span><span class="s">mcr.microsoft.com/dotnet/aspnet:9.0</span><span class="w"> </span><span class="k">AS</span><span class="w"> </span><span class="s">base</span>

<span class="k">ARG</span><span class="s"> APP_UID=1000</span>
<span class="k">ARG</span><span class="s"> APP_GID=1000</span>

<span class="k">RUN </span>groupadd <span class="nt">-g</span> <span class="nv">$APP_GID</span> appgroup     <span class="o">&amp;&amp;</span> useradd <span class="nt">-u</span> <span class="nv">$APP_UID</span> <span class="nt">-g</span> <span class="nv">$APP_GID</span> <span class="nt">-m</span> appuser

<span class="k">USER</span><span class="s"> appuser</span>
<span class="k">WORKDIR</span><span class="s"> /app</span>
<span class="k">EXPOSE</span><span class="s"> 8080</span>
<span class="k">EXPOSE</span><span class="s"> 8081</span>
</code></pre>

</div>



<p>Now you can optionally override the UID/GID at build time:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight shell"><code>docker build <span class="nt">-t</span> my-webapi <span class="nt">--build-arg</span> <span class="nv">APP_UID</span><span class="o">=</span>1001 <span class="nt">--build-arg</span> <span class="nv">APP_GID</span><span class="o">=</span>1001 <span class="nb">.</span>
</code></pre>

</div>



<h3>
  
  
  ✅ Option 3 — Use a Predefined Non‑Root User (If Image Provides It)
</h3>

<p>Some images define built‑in users like <code>dotnet</code> or <code>app</code>. In that case, you might see:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight docker"><code><span class="k">USER</span><span class="s"> app</span>
</code></pre>

</div>



<p>But that depends on the <strong>specific tag</strong> and image; check the official docs for the image you’re using.</p>

<blockquote>
<p>Recommended strategy:  </p>

<ul>
<li>Dev: comment <code>USER $APP_UID</code> if it’s blocking you.
</li>
<li>Prod: properly define and create the non‑root user as shown above.</li>
</ul>
</blockquote>




<h2>
  
  
  5. How to Build the Image (Step by Step)
</h2>

<p>In the folder where your <code>Dockerfile</code> and <code>Web.Api</code> project live, run:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight shell"><code><span class="c"># Basic build (uses default Release configuration)</span>
docker build <span class="nt">-t</span> my-webapi <span class="nb">.</span>
</code></pre>

</div>



<p>Want to be explicit about configuration?<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight shell"><code>docker build <span class="nt">-t</span> my-webapi <span class="nt">--build-arg</span> <span class="nv">BUILD_CONFIGURATION</span><span class="o">=</span>Release <span class="nb">.</span>
</code></pre>

</div>



<p>What Docker will do:</p>

<ol>
<li>Pull <code>mcr.microsoft.com/dotnet/sdk:10.0</code> (or from cache)
</li>
<li>Restore NuGet packages for <code>Web.Api.csproj</code>
</li>
<li>Build the project to <code>/app/build</code>
</li>
<li>Publish the project to <code>/app/publish</code>
</li>
<li>Create a final runtime image from <code>aspnet:9.0</code> with <code>/app/publish</code> copied in</li>
</ol>

<p>If the build fails, check:</p>

<ul>
<li>Are <code>Directory.Packages.props</code> and <code>Directory.Build.props</code> really in the context?
</li>
<li>Is the project folder exactly <code>Web.Api</code> and the file exactly <code>Web.Api.csproj</code>?
</li>
<li>Do you need to fix or remove <code>USER $APP_UID</code>?</li>
</ul>




<h2>
  
  
  6. How to Run the Container
</h2>

<p>Once the image builds successfully:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight shell"><code>docker run <span class="nt">--rm</span> <span class="nt">-p</span> 8080:8080 <span class="nt">--name</span> my-webapi my-webapi
</code></pre>

</div>



<p>What this means:</p>

<ul>
<li>
<code>--rm</code> → removes the container when it stops
</li>
<li>
<code>-p 8080:8080</code> → <strong>host port 8080 → container port 8080</strong>
</li>
<li>
<code>--name my-webapi</code> → gives the container a readable name
</li>
<li>
<code>my-webapi</code> → the image you built</li>
</ul>

<p>Now browse to:</p>

<ul>
<li><code>http://localhost:8080</code></li>
<li>Maybe <code>http://localhost:8080/swagger</code> depending on your API setup.</li>
</ul>

<h3>
  
  
  What if your app listens on port 80 inside the container?
</h3>

<p>Sometimes ASP.NET is configured to listen on <code>http://+:80</code> inside the container. In that case, change the mapping:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight shell"><code>docker run <span class="nt">--rm</span> <span class="nt">-p</span> 8080:80 my-webapi
</code></pre>

</div>



<ul>
<li>Host 8080 → Container 80</li>
</ul>

<blockquote>
<p>Tip:<br><br>
Always confirm your <strong>Kestrel configuration</strong> (<code>ASPNETCORE_URLS</code>, <code>appsettings.json</code>, or <code>Program.cs</code>) to map ports correctly.</p>
</blockquote>




<h2>
  
  
  7. Version Alignment: aspnet:9.0 vs sdk:10.0
</h2>

<p>Right now the Dockerfile uses:</p>

<ul>
<li>Runtime: <code>mcr.microsoft.com/dotnet/aspnet:9.0</code>
</li>
<li>SDK: <code>mcr.microsoft.com/dotnet/sdk:10.0</code>
</li>
</ul>

<p>This can work (SDK 10 building a .NET 9 app), but in most real‑world setups you want <strong>matching major versions</strong>, e.g.:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight docker"><code><span class="k">FROM</span><span class="w"> </span><span class="s">mcr.microsoft.com/dotnet/aspnet:9.0</span><span class="w"> </span><span class="k">AS</span><span class="w"> </span><span class="s">base</span>
<span class="k">FROM</span><span class="w"> </span><span class="s">mcr.microsoft.com/dotnet/sdk:9.0</span><span class="w"> </span><span class="k">AS</span><span class="w"> </span><span class="s">build</span>
</code></pre>

</div>



<p>Why align versions?</p>

<ul>
<li>Less surprise when APIs change between SDK versions
</li>
<li>Consistent behavior between build and runtime
</li>
<li>Easier upgrades: bump both to <code>10.0</code> at once later</li>
</ul>

<blockquote>
<p>Rule of thumb:<br><br>
Unless you explicitly know why you need a newer SDK, keep <strong>runtime and SDK on the same major version</strong>.</p>
</blockquote>




<h2>
  
  
  8. Quick Checklist: “Can I Run This Dockerfile?”
</h2>

<p>Use this as a quick validation list before you start debugging in circles:</p>

<h3>
  
  
  Environment
</h3>

<ul>
<li>[ ] Docker Desktop / Engine is installed and running
</li>
<li>[ ] You are using <strong>Linux containers</strong>, not Windows containers</li>
</ul>

<h3>
  
  
  Project Layout
</h3>

<ul>
<li>[ ] <code>Dockerfile</code> is at the root of the solution (where you intend to build)
</li>
<li>[ ] <code>Directory.Packages.props</code> and <code>Directory.Build.props</code> exist in that directory
</li>
<li>[ ] There is a <code>Web.Api</code> folder with <code>Web.Api.csproj</code> and the API source files</li>
</ul>

<h3>
  
  
  Security/User
</h3>

<ul>
<li>[ ] Either:

<ul>
<li>[ ] <code>USER $APP_UID</code> is temporarily commented out for dev, <strong>or</strong>
</li>
<li>[ ] A valid non‑root user is created and <code>APP_UID</code>/<code>APP_GID</code> are configured</li>
</ul>


</li>

</ul>

<h3>
  
  
  Build
</h3>

<ul>
<li>[ ] <code>docker build -t my-webapi .</code> completes successfully
</li>
<li>[ ] No <code>dotnet restore</code> errors (NuGet sources reachable, correct TFMs, etc.)</li>
</ul>

<h3>
  
  
  Run
</h3>

<ul>
<li>[ ] <code>docker run -p 8080:8080 my-webapi</code> starts the container
</li>
<li>[ ] The app responds at <code>http://localhost:8080</code> (or mapped port/route)
</li>
<li>[ ] Logs show the app listening on the expected URL/port</li>
</ul>

<p>If all items are checked, you’re in a solid place to start iterating and hardening.</p>




<h2>
  
  
  Final Thoughts
</h2>

<p>Visual Studio’s generated Dockerfile isn’t magic — it’s a <strong>clean example of a multi‑stage build</strong>:</p>

<ul>
<li>
<code>base</code> → runtime foundation
</li>
<li>
<code>build</code> → SDK and compilation
</li>
<li>
<code>publish</code> → final app output
</li>
<li>
<code>final</code> → minimal runtime image</li>
</ul>

<p>Once you fully understand a Dockerfile like this, you can:</p>

<ul>
<li>Tweak it for <strong>different projects</strong> (other Web APIs, gRPC, background workers)
</li>
<li>Enforce <strong>non‑root users</strong> correctly in production
</li>
<li>Align SDK/runtime versions with intent
</li>
<li>Plug the same image into <strong>Kubernetes, Azure Container Apps, ECS, Cloud Run</strong>, etc.</li>
</ul>

<p>If you’d like a follow‑up article, here are some natural next steps:</p>

<ul>
<li>Multi‑stage builds with <strong>Node + .NET</strong> (SPA + API in one image)
</li>
<li>Using <strong>multi‑arch images</strong> for ARM (Apple Silicon, Raspberry Pi)
</li>
<li>Dockerfile patterns for <strong>minimal images</strong> (Alpine, distroless)
</li>
<li>Integrating this image into <strong>CI/CD pipelines</strong> (GitHub Actions, Azure DevOps, GitLab CI)</li>
</ul>

<p>✍️ Written for engineers who don’t just want Docker to “work”, but want to <strong>understand</strong> what’s happening in every layer.</p>

