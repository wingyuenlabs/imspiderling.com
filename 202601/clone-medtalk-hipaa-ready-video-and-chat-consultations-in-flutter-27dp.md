---
Title: Clone MedTalk: HIPAA-Ready Video and Chat Consultations in Flutter
Description: 
Author: Ekemini Samuel
Date: 2026-01-12T21:28:52.000Z
Robots: noindex,nofollow
Template: index
---
<p>Telehealth is transforming the way patients and providers connect, offering faster access to care and reducing barriers caused by distance or scheduling. A critical part of this experience is enabling secure, real-time video consultations alongside features like chat messaging for sharing updates, questions, and follow-ups.</p>

<p>With <a href="https://getstream.io/chat/solutions/healthcare/" rel="noopener noreferrer">Stream's healthcare chat solution</a>, developers can build HIPAA-ready communication features into their apps.</p>

<p>In this tutorial, you'll learn how to clone <a href="https://medtalk.co/" rel="noopener noreferrer">MedTalk</a>, a telehealth platform designed for seamless communication between doctors and patients. We'll use Flutter Web to deliver a responsive cross-platform experience and Stream's Chat &amp; Video SDKs to power HIPAA-ready messaging and video calls. </p>

<p><em>Here’s a demo video of the complete application:</em></p>

<p><iframe width="710" height="399" src="https://www.youtube.com/embed/qE58wXuTeN4">
</iframe>
</p>

<h2>
  
  
  Prerequisites
</h2>

<ul>
<li>A <a href="https://getstream.io/try-for-free/" rel="noopener noreferrer">free Stream account</a> and API key</li>
<li><a href="https://code.visualstudio.com/" rel="noopener noreferrer">VS Code</a></li>
<li><a href="https://developer.android.com/studio?gclsrc=aw.ds&amp;gad_source=1&amp;gad_campaignid=21831783744&amp;gclid=CjwKCAjw7fzDBhA7EiwAOqJkhybR1q73l_7c6U-WTqiP4ckb5sRc4NhY712UjxkQJy286r9gRSoheRoCzYwQAvD_BwE" rel="noopener noreferrer">Android Studio</a></li>
<li>Flutter SDK installed on your computer</li>
</ul>

<h2>
  
  
  Set Up Your Development Environment
</h2>

<p>To install Flutter on your computer, first open <a href="https://flutter.dev/" rel="noopener noreferrer">flutter.dev</a>.</p>

<p><a href="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fosxogwirhfiugymvpbpo.png" class="article-body-image-wrapper"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fosxogwirhfiugymvpbpo.png" alt="Flutter" width="800" height="346"></a></p>

<blockquote>
<p>Note: </p>
</blockquote>

<p><em>If you already have your development environment set up, you can clone the project and get started quickly.</em><br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight shell"><code>git clone &lt;https://github.com/Tabintel/medtech.git&gt;

<span class="nb">cd </span>medtech
</code></pre>

</div>



<p><em>Run this command to install the dependencies needed for the app:</em><br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight shell"><code>flutter pub get
</code></pre>

</div>



<p><em>If not, then continue with the steps below.</em></p>

<p>First, <a href="https://docs.flutter.dev/get-started/install?_gl=1*1z9esc*_ga*NTAxMTAxMjkzLjE3NTI5NTM5NDU.*_ga_04YGWK0175*czE3NTI5NTM5NDUkbzEkZzEkdDE3NTI5NTQwMzMkajYwJGwwJGgw" rel="noopener noreferrer">download</a> the version compatible with your computer’s operating system: Windows, macOS, Linux, or ChromeOS.</p>

<p><a href="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2F24otcxyrkngs19g8vmym.png" class="article-body-image-wrapper"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2F24otcxyrkngs19g8vmym.png" alt="App" width="800" height="217"></a></p>

<p>Then choose the type of app you’ll be building. For this tutorial, we’re building a Flutter web application, so select Web from the list.</p>

<p><a href="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fkvashvyltdgxws4r6z2u.png" class="article-body-image-wrapper"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fkvashvyltdgxws4r6z2u.png" alt="Code" width="800" height="201"></a></p>

<p>After selecting the type of app you’re building, the next step is setting up the requirements to begin building with Flutter. Follow the steps in the <a href="https://docs.flutter.dev/get-started/install/windows/web" rel="noopener noreferrer">Flutter documentation</a> to proceed. </p>

<blockquote>
<p>If you’re using another operating system, follow the <a href="https://docs.flutter.dev/get-started/install/macos" rel="noopener noreferrer">guide for your OS</a>.</p>
</blockquote>

<p>Specific requirements for a Windows OS are:</p>

<ul>
<li>Use <a href="https://gitforwindows.org/" rel="noopener noreferrer">Git for Windows</a> to manage Flutter versions and your source code versioning.</li>
<li>
<a href="https://www.google.com/chrome/dr/download/" rel="noopener noreferrer">Google Chrome</a> to debug JavaScript code for web apps.</li>
<li>
<a href="https://code.visualstudio.com/docs/setup/windows" rel="noopener noreferrer">Visual Studio Code</a> with the <a href="https://marketplace.visualstudio.com/items?itemName=Dart-Code.flutter" rel="noopener noreferrer">Flutter extension for VS Code</a>.</li>
<li>Also, note that we are installing Flutter with VS Code.</li>
<li>Once you meet all these requirements, you’re ready to start building.</li>
</ul>

<h2>
  
  
  Install the Flutter SDK
</h2>

<p>In your VS Code editor, run the Ctrl + Shift +p command.</p>

<p><a href="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fc343cgpe4d9ysjx3um1z.png" class="article-body-image-wrapper"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fc343cgpe4d9ysjx3um1z.png" alt="Flutter" width="800" height="330"></a></p>

<p>Then select <strong>Download</strong> when this window shows.</p>

<p>After downloading and installing the Flutter SDK, you will choose the type of Flutter template you want to install.</p>

<p>For this project, select <strong>Application</strong> as shown below.</p>

<p><a href="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Ffa89kl4uodlzn6z0w4oz.png" class="article-body-image-wrapper"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Ffa89kl4uodlzn6z0w4oz.png" alt="Code" width="800" height="417"></a></p>

<p>Now that we have the Flutter SDK, we can proceed to further develop the Virtual Health consultation platform with Stream and Flutter.</p>

<blockquote>
<p>If you need any clarification, <a href="https://docs.flutter.dev/install/with-vs-code" rel="noopener noreferrer">read the Flutter docs</a>.</p>
</blockquote>

<p>To confirm that you installed Flutter correctly, run flutter doctor -v in your command prompt terminal.</p>

<p>It should print out a similar response like so:</p>

<p><a href="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fo0563mxd0vc2saqnegd3.png" class="article-body-image-wrapper"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fo0563mxd0vc2saqnegd3.png" alt="Code" width="800" height="139"></a></p>

<p>Follow <a href="https://docs.flutter.dev/platform-integration/web/building" rel="noopener noreferrer">these steps</a> to set up the project for building with Flutter Web.</p>

<p>Create a medtech folder on your computer, and open it in the VS Code editor. In your command prompt, run:</p>

<p><code>flutter create . \--platforms web</code></p>

<p>The output then displays like so:</p>

<p><a href="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2F0f8yt2cxjw02ejnuz8ek.png" class="article-body-image-wrapper"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2F0f8yt2cxjw02ejnuz8ek.png" alt="project" width="672" height="324"></a></p>

<p>It creates the structure for the <a href="https://docs.flutter.dev/platform-integration/web/building" rel="noopener noreferrer">Flutter web</a> project. It also makes a web/ directory containing the web assets used to bootstrap and run your Flutter app, like so:</p>

<p><a href="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fz35b5mk2uw92y3nad7wp.png" class="article-body-image-wrapper"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fz35b5mk2uw92y3nad7wp.png" alt="App" width="443" height="407"></a></p>

<p>In the following steps, we will look at how to set up Stream for the virtual health consultation platform with Flutter SDK.</p>

<h2>
  
  
  Get Started with Stream
</h2>

<p>First, <a href="https://getstream.io/accounts/signup/" rel="noopener noreferrer">create an account</a> in the Stream dashboard.</p>

<p><a href="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fsan5r249b1kr78rhjh2v.png" class="article-body-image-wrapper"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fsan5r249b1kr78rhjh2v.png" alt="Stream" width="800" height="421"></a></p>

<p>Then click on <strong>Create App</strong> to create a new application.</p>

<p><a href="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Foeebe99em5dis9518bge.png" class="article-body-image-wrapper"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Foeebe99em5dis9518bge.png" alt="App" width="800" height="146"></a></p>

<p>Enter the details of the application.</p>

<p><a href="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Filt1tfcszhk7urr1gkx4.png" class="article-body-image-wrapper"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Filt1tfcszhk7urr1gkx4.png" alt="App" width="800" height="379"></a></p>

<p>After creating the Stream app, <a href="https://dashboard.getstream.io/?_gl=1*okfyq9*_gcl_au*MTk2NjYwMzM1LjE3NjEzMTY4NzM.*_ga*MjAxMDg2ODcxMS4xNzUzMjA1MjM0*_ga_FX552ZCLHK*czE3Njc5Njc1NTYkbzI3JGcxJHQxNzY3OTcwMzc5JGo2MCRsMCRoMTcxMjI1NzY2NA.." rel="noopener noreferrer">go to your dashboard</a> and click Chat Messaging in the top right. You will then see the App Access Keys displayed. </p>

<p><a href="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2F15i23h24gdk6n397p99f.png" class="article-body-image-wrapper"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2F15i23h24gdk6n397p99f.png" alt="App" width="800" height="339"></a></p>

<p>Create an <code>.env</code>file in your project’s root directory, copy the App key and secret, and save it like so:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight shell"><code>STREAM<span class="se">\_</span>API<span class="se">\_</span><span class="nv">KEY</span><span class="o">=</span>your-api-key  
STREAM<span class="se">\_</span>API<span class="se">\_</span><span class="nv">SECRET</span><span class="o">=</span>your-api-secret
</code></pre>

</div>



<p>Now you’re ready to build and integrate Stream using the <a href="https://getstream.io/chat/docs/flutter-dart/" rel="noopener noreferrer">Flutter SDK</a>.</p>

<p>In your terminal, run this command:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight shell"><code>flutter pub get
</code></pre>

</div>



<p>This command installs all the dependencies listed in your <code>pubspec.yaml</code>, making packages like <code>stream_chat_flutter</code>, <code>flutter_dotenv</code>, available to your app. It’s required after any change to <code>pubspec.yaml</code> before you can use new packages in your code.</p>

<p><a href="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fh85268qk7p74hbnwegxq.png" class="article-body-image-wrapper"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fh85268qk7p74hbnwegxq.png" alt="Code" width="800" height="429"></a></p>

<p>To integrate prebuilt UI components for chat in Flutter, use the stream_chat_flutter package. This package includes reusable and customizable UI components already integrated with Stream’s API.</p>

<p>Add the package to your dependencies like so:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight shell"><code>dependencies:  
  stream<span class="se">\_</span>chat<span class="se">\_</span>flutter: ^1.0.1-beta
</code></pre>

</div>



<p>This package provides full chat UI widgets you can use directly in your app for fast integration.</p>

<blockquote>
<p>For video, UI components to Stream Video in Flutter, check out <a href="https://getstream.io/video/docs/flutter/ui-components-overview/" rel="noopener noreferrer">the documentation</a>. </p>
</blockquote>

<h2>
  
  
  Build the Role Selection Screen
</h2>

<p>As we develop the app, the <a href="https://github.com/Tabintel/medtech" rel="noopener noreferrer">codebase</a> will increase to what is in the image below.</p>

<p><a href="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Frtypph5eizi0zda586fu.png" class="article-body-image-wrapper"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Frtypph5eizi0zda586fu.png" alt="App" width="800" height="488"></a></p>

<p>In the <code>lib/screen/auth</code> directory, create a file called <code>role_selection_screen.dart</code> and enter the code below:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight dart"><code><span class="kn">import</span> <span class="s">'package:flutter/material.dart'</span><span class="o">;</span>  
<span class="kn">import</span> <span class="s">'../../stream</span><span class="err">\</span><span class="s">_client.dart'</span><span class="o">;</span>  
<span class="kd">class</span> <span class="nc">RoleSelectionScreen</span> <span class="kd">extends</span> <span class="n">StatefulWidget</span> <span class="p">{</span>  
  <span class="kd">const</span> <span class="n">RoleSelectionScreen</span><span class="p">({</span><span class="k">super</span><span class="o">.</span><span class="na">key</span><span class="p">});</span>  
  <span class="nd">@override</span>  
  <span class="n">State</span><span class="err">\</span><span class="p">&lt;</span><span class="n">RoleSelectionScreen</span><span class="err">\</span><span class="p">&gt;</span> <span class="n">createState</span><span class="p">()</span> <span class="err">\</span><span class="o">=</span><span class="err">\</span><span class="p">&gt;</span> <span class="err">\</span><span class="n">_RoleSelectionScreenState</span><span class="p">();</span>  
<span class="p">}</span>  
<span class="kd">class</span> <span class="err">\</span><span class="nc">_RoleSelectionScreenState</span> <span class="kd">extends</span> <span class="n">State</span><span class="err">\</span><span class="p">&lt;</span><span class="n">RoleSelectionScreen</span><span class="err">\</span><span class="p">&gt;</span> <span class="p">{</span>  
  <span class="kt">bool</span> <span class="err">\</span><span class="n">_loading</span> <span class="err">\</span><span class="o">=</span> <span class="kc">false</span><span class="p">;</span>  
  <span class="n">Future</span><span class="err">\</span><span class="p">&lt;</span><span class="kt">void</span><span class="err">\</span><span class="p">&gt;</span> <span class="err">\</span><span class="n">_connectAndNavigate</span><span class="p">(</span><span class="kt">String</span> <span class="n">userId</span><span class="p">,</span> <span class="kt">String</span> <span class="n">route</span><span class="p">,</span> <span class="p">{</span><span class="kt">String</span><span class="o">?</span> <span class="n">name</span><span class="p">})</span> <span class="kd">async</span> <span class="p">{</span>  
    <span class="n">setState</span><span class="p">(()</span> <span class="err">\</span><span class="o">=</span><span class="err">\</span><span class="p">&gt;</span> <span class="err">\</span><span class="n">_loading</span> <span class="err">\</span><span class="o">=</span> <span class="kc">true</span><span class="p">);</span>  
    <span class="k">await</span> <span class="n">StreamClientProvider</span><span class="o">.</span><span class="na">connectUser</span><span class="p">(</span><span class="n">userId</span><span class="p">,</span> <span class="nl">name:</span> <span class="n">name</span><span class="p">);</span>  
    <span class="n">setState</span><span class="p">(()</span> <span class="err">\</span><span class="o">=</span><span class="err">\</span><span class="p">&gt;</span> <span class="err">\</span><span class="n">_loading</span> <span class="err">\</span><span class="o">=</span> <span class="kc">false</span><span class="p">);</span>  
    <span class="n">Navigator</span><span class="o">.</span><span class="na">pushNamed</span><span class="p">(</span><span class="n">context</span><span class="p">,</span> <span class="n">route</span><span class="p">);</span>  
  <span class="p">}</span>  
  <span class="nd">@override</span>  
  <span class="n">Widget</span> <span class="n">build</span><span class="p">(</span><span class="n">BuildContext</span> <span class="n">context</span><span class="p">)</span> <span class="p">{</span>  
    <span class="k">return</span> <span class="n">Scaffold</span><span class="p">(</span>  
      <span class="nl">backgroundColor:</span> <span class="kd">const</span> <span class="n">Color</span><span class="p">(</span><span class="mh">0xFFF7F9FC</span><span class="p">),</span>  
      <span class="nl">body:</span> <span class="n">Center</span><span class="p">(</span>  
        <span class="nl">child:</span> <span class="n">Padding</span><span class="p">(</span>  
          <span class="nl">padding:</span> <span class="kd">const</span> <span class="n">EdgeInsets</span><span class="o">.</span><span class="na">all</span><span class="p">(</span><span class="mf">24.0</span><span class="p">),</span>  
          <span class="nl">child:</span> <span class="n">Column</span><span class="p">(</span>  
            <span class="nl">mainAxisAlignment:</span> <span class="n">MainAxisAlignment</span><span class="o">.</span><span class="na">center</span><span class="p">,</span>  
            <span class="nl">children:</span> <span class="err">\</span><span class="p">[</span>  
              <span class="n">Image</span><span class="o">.</span><span class="na">asset</span><span class="p">(</span>  
                <span class="s">'assets/images/medtalk</span><span class="err">\</span><span class="s">_logo.png'</span><span class="p">,</span>  
                <span class="nl">height:</span> <span class="mi">120</span><span class="p">,</span>  
              <span class="p">),</span>  
              <span class="kd">const</span> <span class="n">SizedBox</span><span class="p">(</span><span class="nl">height:</span> <span class="mi">32</span><span class="p">),</span>  
              <span class="kd">const</span> <span class="n">Text</span><span class="p">(</span>  
                <span class="s">'MedTalk'</span><span class="p">,</span>  
                <span class="nl">style:</span> <span class="n">TextStyle</span><span class="p">(</span><span class="nl">fontSize:</span> <span class="mi">32</span><span class="p">,</span> <span class="nl">fontWeight:</span> <span class="n">FontWeight</span><span class="o">.</span><span class="na">bold</span><span class="p">,</span> <span class="nl">color:</span> <span class="n">Color</span><span class="p">(</span><span class="mh">0xFF2A4D9B</span><span class="p">)),</span>  
              <span class="p">),</span>  
              <span class="kd">const</span> <span class="n">SizedBox</span><span class="p">(</span><span class="nl">height:</span> <span class="mi">8</span><span class="p">),</span>  
              <span class="kd">const</span> <span class="n">Text</span><span class="p">(</span>  
                <span class="s">'Talk to a doctor anytime, anywhere'</span><span class="p">,</span>  
                <span class="nl">style:</span> <span class="n">TextStyle</span><span class="p">(</span><span class="nl">fontSize:</span> <span class="mi">16</span><span class="p">,</span> <span class="nl">color:</span> <span class="n">Color</span><span class="p">(</span><span class="mh">0xFF4A4A4A</span><span class="p">)),</span>  
                <span class="nl">textAlign:</span> <span class="n">TextAlign</span><span class="o">.</span><span class="na">center</span><span class="p">,</span>  
              <span class="p">),</span>  
              <span class="kd">const</span> <span class="n">SizedBox</span><span class="p">(</span><span class="nl">height:</span> <span class="mi">48</span><span class="p">),</span>  
              <span class="err">\</span><span class="n">_loading</span>  
                  <span class="o">?</span> <span class="kd">const</span> <span class="n">CircularProgressIndicator</span><span class="p">()</span>  
                  <span class="o">:</span> <span class="n">Column</span><span class="p">(</span>  
                      <span class="nl">children:</span> <span class="err">\</span><span class="p">[</span>  
                        <span class="n">ElevatedButton</span><span class="p">(</span>  
                          <span class="nl">onPressed:</span> <span class="p">()</span> <span class="err">\</span><span class="o">=</span><span class="err">\</span><span class="p">&gt;</span> <span class="err">\</span><span class="n">_connectAndNavigate</span><span class="p">(</span><span class="s">'doctor'</span><span class="p">,</span> <span class="s">'/doctor'</span><span class="p">,</span> <span class="nl">name:</span> <span class="s">'Dr. Sarah Lee'</span><span class="p">),</span>  
                          <span class="nl">style:</span> <span class="n">ElevatedButton</span><span class="o">.</span><span class="na">styleFrom</span><span class="p">(</span>  
                            <span class="nl">minimumSize:</span> <span class="kd">const</span> <span class="n">Size</span><span class="o">.</span><span class="na">fromHeight</span><span class="p">(</span><span class="mi">48</span><span class="p">),</span>  
                            <span class="nl">backgroundColor:</span> <span class="n">Color</span><span class="p">(</span><span class="mh">0xFF2A4D9B</span><span class="p">),</span>  
                            <span class="nl">foregroundColor:</span> <span class="n">Colors</span><span class="o">.</span><span class="na">white</span><span class="p">,</span>  
                            <span class="nl">textStyle:</span> <span class="kd">const</span> <span class="n">TextStyle</span><span class="p">(</span><span class="nl">fontWeight:</span> <span class="n">FontWeight</span><span class="o">.</span><span class="na">bold</span><span class="p">),</span>  
                            <span class="nl">shape:</span> <span class="n">RoundedRectangleBorder</span><span class="p">(</span>  
                              <span class="nl">borderRadius:</span> <span class="n">BorderRadius</span><span class="o">.</span><span class="na">circular</span><span class="p">(</span><span class="mi">12</span><span class="p">),</span>  
                            <span class="p">),</span>  
                          <span class="p">),</span>  
                          <span class="nl">child:</span> <span class="kd">const</span> <span class="n">Text</span><span class="p">(</span><span class="s">'Continue as Doctor'</span><span class="p">),</span>  
                        <span class="p">),</span>  
                        <span class="kd">const</span> <span class="n">SizedBox</span><span class="p">(</span><span class="nl">height:</span> <span class="mi">16</span><span class="p">),</span>  
                        <span class="n">OutlinedButton</span><span class="p">(</span>  
                          <span class="nl">onPressed:</span> <span class="p">()</span> <span class="err">\</span><span class="o">=</span><span class="err">\</span><span class="p">&gt;</span> <span class="err">\</span><span class="n">_connectAndNavigate</span><span class="p">(</span><span class="s">'patient'</span><span class="p">,</span> <span class="s">'/patient'</span><span class="p">,</span> <span class="nl">name:</span> <span class="s">'Patient'</span><span class="p">),</span>  
                          <span class="nl">style:</span> <span class="n">OutlinedButton</span><span class="o">.</span><span class="na">styleFrom</span><span class="p">(</span>  
                            <span class="nl">minimumSize:</span> <span class="kd">const</span> <span class="n">Size</span><span class="o">.</span><span class="na">fromHeight</span><span class="p">(</span><span class="mi">48</span><span class="p">),</span>  
                            <span class="nl">foregroundColor:</span> <span class="n">Color</span><span class="p">(</span><span class="mh">0xFF2A4D9B</span><span class="p">),</span>  
                            <span class="nl">textStyle:</span> <span class="kd">const</span> <span class="n">TextStyle</span><span class="p">(</span><span class="nl">fontWeight:</span> <span class="n">FontWeight</span><span class="o">.</span><span class="na">bold</span><span class="p">),</span>  
                            <span class="nl">side:</span> <span class="kd">const</span> <span class="n">BorderSide</span><span class="p">(</span><span class="nl">color:</span> <span class="n">Color</span><span class="p">(</span><span class="mh">0xFF2A4D9B</span><span class="p">),</span> <span class="nl">width:</span> <span class="mi">2</span><span class="p">),</span>  
                            <span class="nl">shape:</span> <span class="n">RoundedRectangleBorder</span><span class="p">(</span>  
                              <span class="nl">borderRadius:</span> <span class="n">BorderRadius</span><span class="o">.</span><span class="na">circular</span><span class="p">(</span><span class="mi">12</span><span class="p">),</span>  
                            <span class="p">),</span>  
                          <span class="p">),</span>  
                          <span class="nl">child:</span> <span class="kd">const</span> <span class="n">Text</span><span class="p">(</span><span class="s">'Continue as Patient'</span><span class="p">),</span>  
                        <span class="p">),</span>  
                      <span class="err">\</span><span class="p">],</span>  
                    <span class="p">),</span>  
            <span class="err">\</span><span class="p">],</span>  
          <span class="p">),</span>  
        <span class="p">),</span>  
      <span class="p">),</span>  
    <span class="p">);</span>  
  <span class="p">}</span>  
<span class="p">}</span>
</code></pre>

</div>



<blockquote>
<p>Note: You can get the asset in <code>assets/images/medtalk_logo.png</code> from this <a href="https://github.com/Tabintel/medtech/tree/master/assets/images" rel="noopener noreferrer">GitHub repository</a>. Download the <a href="https://github.com/Tabintel/medtech/blob/master/assets/images/medtalk_logo.png" rel="noopener noreferrer">medtalk_logo.png</a> file and add it to the <code>/assets/images</code> directory.</p>
</blockquote>

<p>The role selection screen is the entry point to the virtual health consultation application. It allows users to choose between <strong>doctor and patient</strong> roles. This screen sets the user's identity and routes to the appropriate dashboard.</p>

<p>The state management uses Flutter's built-in <code>StatefulWidget</code> to manage UI states and tracks the loading state with a <code>_loading</code> boolean to show/hide the loading indicator.<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight javascript"><code><span class="kd">class</span> <span class="err">\</span><span class="nc">_RoleSelectionScreenState</span> <span class="kd">extends</span> <span class="nc">State</span><span class="err">\</span><span class="o">&lt;</span><span class="nx">RoleSelectionScreen</span><span class="err">\</span><span class="o">&gt;</span> <span class="p">{</span>  
  <span class="nx">bool</span> <span class="err">\</span><span class="nx">_loading</span> <span class="err">\</span><span class="o">=</span> <span class="kc">false</span><span class="p">;</span>  
  <span class="c1">// ...  </span>
<span class="p">}</span>
</code></pre>

</div>



<p>We import <code>flutter/material.dart</code> for UI components and <code>../../stream_client.dart</code> to manage connections to Stream Chat using a user’s ID and name. </p>

<blockquote>
<p>The screen is built as a <a href="https://api.flutter.dev/flutter/widgets/StatefulWidget-class.html" rel="noopener noreferrer">stateful widget</a> to handle dynamic changes, such as displaying a loading spinner during Stream connection, with a _loading boolean (shown in the code snippet above) that tracks this connection status.</p>
</blockquote>

<p><code>_connectAndNavigate</code> function sets <code>_loading</code> to true, calls <code>StreamClientProvider.connectUser(userId, name: name)</code> to connect the user, then navigates to either <em>/doctor</em> or <em>/patient</em> once connected.</p>

<p>The UI has an off-white background, the MedTalk logo, and a tagline. If <code>_loading</code> is true, a <code>CircularProgressIndicator</code> is shown; otherwise, two buttons appear — “Continue as Doctor” (blue, connects with Dr. Sarah Lee) and “Continue as Patient” (outlined, connects with Patient).</p>

<p>This screen is the entry point to the virtual health consultation platform, guiding users with a personalized experience based on their role; <em>Doctor or Patient</em>.</p>

<h2>
  
  
  Build the Doctor’s Home Screen
</h2>

<p>After users select their role and connect to <strong>Stream</strong>, doctors are taken to the <code>DoctorHomeScreen</code>. This is the main dashboard for doctors, where they can see demo patients, start chats, and view existing conversations.</p>

<p>In <code>lib/screens/doctor/</code> create a file: <code>doctor_home_screen.dart</code>, and enter the code below:</p>

<blockquote>
<p>The full code is in the <a href="https://github.com/Tabintel/medtech/blob/master/lib/screens/doctor/doctor_home_screen.dart" rel="noopener noreferrer">GitHub repository</a>.<br>
</p>
</blockquote>

<div class="highlight js-code-highlight">
<pre class="highlight dart"><code><span class="kn">import</span> <span class="s">'package:flutter/material.dart'</span><span class="o">;</span>  
<span class="kn">import</span> <span class="s">'package:stream</span><span class="err">\</span><span class="s">_chat</span><span class="err">\</span><span class="s">_flutter/stream</span><span class="err">\</span><span class="s">_chat</span><span class="err">\</span><span class="s">_flutter.dart'</span><span class="o">;</span>  
<span class="kn">import</span> <span class="s">'../../stream</span><span class="err">\</span><span class="s">_client.dart'</span><span class="o">;</span>  
<span class="kn">import</span> <span class="s">'../../demo</span><span class="err">\</span><span class="s">_users.dart'</span><span class="o">;</span>  
<span class="kd">class</span> <span class="nc">DoctorHomeScreen</span> <span class="kd">extends</span> <span class="n">StatefulWidget</span> <span class="p">{</span>  
  <span class="kd">const</span> <span class="n">DoctorHomeScreen</span><span class="p">({</span><span class="n">Key</span><span class="o">?</span> <span class="n">key</span><span class="p">})</span> <span class="o">:</span> <span class="k">super</span><span class="p">(</span><span class="nl">key:</span> <span class="n">key</span><span class="p">);</span>  
  <span class="nd">@override</span>  
  <span class="n">State</span><span class="err">\</span><span class="p">&lt;</span><span class="n">DoctorHomeScreen</span><span class="err">\</span><span class="p">&gt;</span> <span class="n">createState</span><span class="p">()</span> <span class="err">\</span><span class="o">=</span><span class="err">\</span><span class="p">&gt;</span> <span class="err">\</span><span class="n">_DoctorHomeScreenState</span><span class="p">();</span>  
<span class="p">}</span>  
<span class="kd">class</span> <span class="err">\</span><span class="nc">_DoctorHomeScreenState</span> <span class="kd">extends</span> <span class="n">State</span><span class="err">\</span><span class="p">&lt;</span><span class="n">DoctorHomeScreen</span><span class="err">\</span><span class="p">&gt;</span> <span class="p">{</span>  
  <span class="kd">late</span> <span class="kd">final</span> <span class="n">StreamChannelListController</span> <span class="err">\</span><span class="n">_channelListController</span><span class="p">;</span>  
  <span class="nd">@override</span>  
  <span class="kt">void</span> <span class="n">initState</span><span class="p">()</span> <span class="p">{</span>  
    <span class="k">super</span><span class="o">.</span><span class="na">initState</span><span class="p">();</span>  
    <span class="err">\</span><span class="n">_channelListController</span> <span class="err">\</span><span class="o">=</span> <span class="n">StreamChannelListController</span><span class="p">(</span>  
      <span class="nl">client:</span> <span class="n">StreamClientProvider</span><span class="o">.</span><span class="na">client</span><span class="p">,</span>  
      <span class="nl">filter:</span> <span class="n">Filter</span><span class="o">.</span><span class="na">in</span><span class="err">\</span><span class="n">_</span><span class="p">(</span><span class="s">'members'</span><span class="p">,</span> <span class="err">\</span><span class="p">[</span><span class="n">StreamClientProvider</span><span class="o">.</span><span class="na">client</span><span class="o">.</span><span class="na">state</span><span class="o">.</span><span class="na">currentUser</span><span class="err">\</span><span class="o">!.</span><span class="na">id</span><span class="err">\</span><span class="p">]),</span>  
      <span class="nl">limit:</span> <span class="mi">20</span><span class="p">,</span>  
    <span class="p">);</span>  
  <span class="p">}</span>  
  <span class="nd">@override</span>  
  <span class="n">Widget</span> <span class="n">build</span><span class="p">(</span><span class="n">BuildContext</span> <span class="n">context</span><span class="p">)</span> <span class="p">{</span>  
    <span class="kd">final</span> <span class="n">currentUser</span> <span class="err">\</span><span class="o">=</span> <span class="n">StreamClientProvider</span><span class="o">.</span><span class="na">client</span><span class="o">.</span><span class="na">state</span><span class="o">.</span><span class="na">currentUser</span><span class="p">;</span>  
    <span class="k">return</span> <span class="n">Scaffold</span><span class="p">(</span>  
      <span class="nl">backgroundColor:</span> <span class="kd">const</span> <span class="n">Color</span><span class="p">(</span><span class="mh">0xFFF7F9FC</span><span class="p">),</span>  
      <span class="nl">appBar:</span> <span class="n">AppBar</span><span class="p">(</span>  
        <span class="nl">title:</span> <span class="kd">const</span> <span class="n">Text</span><span class="p">(</span><span class="s">'Doctor'</span><span class="p">,</span> <span class="nl">style:</span> <span class="n">TextStyle</span><span class="p">(</span><span class="nl">color:</span> <span class="n">Color</span><span class="p">(</span><span class="mh">0xFF2A4D9B</span><span class="p">),</span> <span class="nl">fontWeight:</span> <span class="n">FontWeight</span><span class="o">.</span><span class="na">bold</span><span class="p">)),</span>  
        <span class="nl">backgroundColor:</span> <span class="n">Colors</span><span class="o">.</span><span class="na">white</span><span class="p">,</span>  
        <span class="nl">iconTheme:</span> <span class="kd">const</span> <span class="n">IconThemeData</span><span class="p">(</span><span class="nl">color:</span> <span class="n">Color</span><span class="p">(</span><span class="mh">0xFF2A4D9B</span><span class="p">)),</span>  
        <span class="nl">elevation:</span> <span class="mi">1</span><span class="p">,</span>  
      <span class="p">),</span>  
      <span class="nl">body:</span> <span class="n">Padding</span><span class="p">(</span>  
        <span class="nl">padding:</span> <span class="kd">const</span> <span class="n">EdgeInsets</span><span class="o">.</span><span class="na">all</span><span class="p">(</span><span class="mf">8.0</span><span class="p">),</span>  
        <span class="nl">child:</span> <span class="n">Column</span><span class="p">(</span>  
          <span class="nl">children:</span> <span class="err">\</span><span class="p">[</span>  
            <span class="c1">// Patient list  </span>
            <span class="n">SizedBox</span><span class="p">(</span>  
              <span class="nl">height:</span> <span class="mi">120</span><span class="p">,</span>  
              <span class="nl">child:</span> <span class="n">ListView</span><span class="o">.</span><span class="na">separated</span><span class="p">(</span>  
                <span class="nl">scrollDirection:</span> <span class="n">Axis</span><span class="o">.</span><span class="na">horizontal</span><span class="p">,</span>  
                <span class="nl">itemCount:</span> <span class="n">demoPatients</span><span class="o">.</span><span class="na">length</span><span class="p">,</span>  
                <span class="nl">separatorBuilder:</span> <span class="p">(</span><span class="err">\</span><span class="n">_</span><span class="p">,</span> <span class="err">\</span><span class="n">_</span><span class="err">\</span><span class="n">_</span><span class="p">)</span> <span class="err">\</span><span class="o">=</span><span class="err">\</span><span class="p">&gt;</span> <span class="kd">const</span> <span class="n">SizedBox</span><span class="p">(</span><span class="nl">width:</span> <span class="mi">16</span><span class="p">),</span>  
                <span class="nl">itemBuilder:</span> <span class="p">(</span><span class="n">context</span><span class="p">,</span> <span class="n">index</span><span class="p">)</span> <span class="p">{</span>  
                  <span class="kd">final</span> <span class="n">patient</span> <span class="err">\</span><span class="o">=</span> <span class="n">demoPatients</span><span class="err">\</span><span class="p">[</span><span class="n">index</span><span class="err">\</span><span class="p">];</span>  
                  <span class="k">return</span> <span class="n">Card</span><span class="p">(</span>  
                    <span class="nl">elevation:</span> <span class="mi">2</span><span class="p">,</span>  
                    <span class="nl">shape:</span> <span class="n">RoundedRectangleBorder</span><span class="p">(</span><span class="nl">borderRadius:</span> <span class="n">BorderRadius</span><span class="o">.</span><span class="na">circular</span><span class="p">(</span><span class="mi">12</span><span class="p">)),</span>  
                    <span class="nl">child:</span> <span class="n">Container</span><span class="p">(</span>  
                      <span class="nl">width:</span> <span class="mi">200</span><span class="p">,</span>  
                      <span class="nl">padding:</span> <span class="kd">const</span> <span class="n">EdgeInsets</span><span class="o">.</span><span class="na">all</span><span class="p">(</span><span class="mi">12</span><span class="p">),</span>  
                      <span class="nl">child:</span> <span class="n">Column</span><span class="p">(</span>  
                        <span class="nl">mainAxisAlignment:</span> <span class="n">MainAxisAlignment</span><span class="o">.</span><span class="na">center</span><span class="p">,</span>  
                        <span class="nl">children:</span> <span class="err">\</span><span class="p">[</span>  
                          <span class="n">Text</span><span class="p">(</span><span class="n">patient</span><span class="o">.</span><span class="na">name</span><span class="p">,</span> <span class="nl">style:</span> <span class="kd">const</span> <span class="n">TextStyle</span><span class="p">(</span><span class="nl">fontWeight:</span> <span class="n">FontWeight</span><span class="o">.</span><span class="na">bold</span><span class="p">)),</span>  
                          <span class="kd">const</span> <span class="n">SizedBox</span><span class="p">(</span><span class="nl">height:</span> <span class="mi">8</span><span class="p">),</span>  
                          <span class="n">ElevatedButton</span><span class="p">(</span>  
                            <span class="nl">onPressed:</span> <span class="p">()</span> <span class="kd">async</span> <span class="p">{</span>  
                              <span class="kd">final</span> <span class="n">client</span> <span class="err">\</span><span class="o">=</span> <span class="n">StreamClientProvider</span><span class="o">.</span><span class="na">client</span><span class="p">;</span>  
                              <span class="kd">final</span> <span class="n">currentUser</span> <span class="err">\</span><span class="o">=</span> <span class="n">client</span><span class="o">.</span><span class="na">state</span><span class="o">.</span><span class="na">currentUser</span><span class="p">;</span>  

                              <span class="k">if</span> <span class="p">(</span><span class="n">currentUser</span> <span class="err">\</span><span class="o">==</span> <span class="kc">null</span><span class="p">)</span> <span class="p">{</span>  
                                <span class="n">ScaffoldMessenger</span><span class="o">.</span><span class="na">of</span><span class="p">(</span><span class="n">context</span><span class="p">)</span><span class="o">.</span><span class="na">showSnackBar</span><span class="p">(</span>  
                                  <span class="n">SnackBar</span><span class="p">(</span><span class="nl">content:</span> <span class="n">Text</span><span class="p">(</span><span class="s">'Current user not found. Please log in again.'</span><span class="p">)),</span>  
                                <span class="p">);</span>  
                                <span class="k">return</span><span class="p">;</span>  
                              <span class="p">}</span>  
                              <span class="k">try</span> <span class="p">{</span>  
                                <span class="c1">// Create the channel  </span>
                                <span class="kd">final</span> <span class="n">channel</span> <span class="err">\</span><span class="o">=</span> <span class="n">client</span><span class="o">.</span><span class="na">channel</span><span class="p">(</span>  
                                  <span class="s">'messaging'</span><span class="p">,</span>  
                                  <span class="nl">extraData:</span> <span class="p">{</span>  
                                    <span class="s">'members'</span><span class="o">:</span> <span class="err">\</span><span class="p">[</span><span class="n">currentUser</span><span class="o">.</span><span class="na">id</span><span class="p">,</span> <span class="n">patient</span><span class="o">.</span><span class="na">id</span><span class="err">\</span><span class="p">],</span>  
                                    <span class="s">'created</span><span class="err">\</span><span class="s">_by</span><span class="err">\</span><span class="s">_id'</span><span class="o">:</span> <span class="n">currentUser</span><span class="o">.</span><span class="na">id</span><span class="p">,</span>  
                                  <span class="p">},</span>  
                                <span class="p">);</span>  

                                <span class="c1">// Create the channel on Stream  </span>
                                <span class="k">await</span> <span class="n">channel</span><span class="o">.</span><span class="na">create</span><span class="p">();</span>  

                                <span class="c1">// Navigate to the chat  </span>
                                <span class="k">if</span> <span class="p">(</span><span class="err">\</span><span class="o">!</span><span class="n">mounted</span><span class="p">)</span> <span class="k">return</span><span class="p">;</span>  
                                <span class="n">Navigator</span><span class="o">.</span><span class="na">push</span><span class="p">(</span>  
                                  <span class="n">context</span><span class="p">,</span>  
                                  <span class="n">MaterialPageRoute</span><span class="p">(</span>  
                                    <span class="nl">builder:</span> <span class="p">(</span><span class="n">context</span><span class="p">)</span> <span class="err">\</span><span class="o">=</span><span class="err">\</span><span class="p">&gt;</span> <span class="n">StreamChannel</span><span class="p">(</span>  
                                      <span class="nl">channel:</span> <span class="n">channel</span><span class="p">,</span>  
                                      <span class="nl">child:</span> <span class="kd">const</span> <span class="n">StreamChannelPage</span><span class="p">(),</span>  
                                    <span class="p">),</span>  
                                  <span class="p">),</span>  
                                <span class="p">);</span>  
                              <span class="p">}</span> <span class="k">catch</span> <span class="p">(</span><span class="n">e</span><span class="p">)</span> <span class="p">{</span>  
                                <span class="k">if</span> <span class="p">(</span><span class="err">\</span><span class="o">!</span><span class="n">mounted</span><span class="p">)</span> <span class="k">return</span><span class="p">;</span>  
                                <span class="n">ScaffoldMessenger</span><span class="o">.</span><span class="na">of</span><span class="p">(</span><span class="n">context</span><span class="p">)</span><span class="o">.</span><span class="na">showSnackBar</span><span class="p">(</span>  
                                  <span class="n">SnackBar</span><span class="p">(</span><span class="nl">content:</span> <span class="n">Text</span><span class="p">(</span><span class="s">'Error creating chat: </span><span class="si">${e.toString()}</span><span class="s">'</span><span class="p">)),</span>  
                                <span class="p">);</span>  
                              <span class="p">}</span>  
                            <span class="p">},</span>  
                            <span class="nl">child:</span> <span class="kd">const</span> <span class="n">Text</span><span class="p">(</span><span class="s">'Start Chat'</span><span class="p">),</span>  
                          <span class="p">),</span>  
                        <span class="err">\</span><span class="p">],</span>  
                      <span class="p">),</span>  
                    <span class="p">),</span>  
                  <span class="p">);</span>  
                <span class="p">},</span>
</code></pre>

</div>



<p>We import four files:</p>

<ul>
<li>
<code>flutter/material.dart</code> for UI components.</li>
<li>
<code>stream_chat_flutter.dart</code> for prebuilt Stream Chat widgets and controllers.</li>
<li>
<code>stream_client.dart</code> for the connected Stream client instance.</li>
<li>
<code>demo_users.dart</code> for sample patient data, so the demo works without a real backend.</li>
</ul>

<p><code>DoctorHomeScreen</code> is a stateful widget because it manages a <code>StreamChannelListController</code>, which fetches channels based on filters. In <code>initState()</code>, the controller is created with:</p>

<ul>
<li>client: StreamClientProvider.client — the active Stream connection.</li>
<li>filter: Filter.in_('members', [currentUser.id]) — returns only channels that include the logged-in doctor.</li>
<li>limit: 20 — loads the latest 20 channels.</li>
</ul>

<p>In the build method, we get the currentUser to display their info and use it when creating new chats. The UI has two main sections:</p>

<p><strong>1. Patient List (top horizontal list)</strong><br>
A scrollable row of demo patients from demoPatients. Each patient card shows their name and a <strong>Start Chat</strong> button. It then:</p>

<ul>
<li>Creates a messaging channel with members [doctorId, patientId].</li>
<li>Calls await <code>channel.create()</code> to set up the conversation in Stream.</li>
<li>Navigates to <code>StreamChannelPage</code>, which shows the chat UI if there’s an error, and a SnackBar displays the message.</li>
</ul>

<p><strong>2. Chat List (main section)</strong><br>
A <code>StreamChannelListView</code> that automatically lists all the doctor’s existing chats based on the controller’s filter. Tapping a channel opens it in StreamChannelPage.</p>

<p><strong>The StreamChannelPage</strong><br>
This uses three widgets from <code>stream_chat_flutter</code>:</p>

<ul>
<li> StreamChannelHeader(): shows the other user’s name and a back button.</li>
<li> StreamMessageListView(): displays real-time messages.</li>
<li> StreamMessageInput(): lets users send new messages with emoji and attachments.</li>
</ul>

<blockquote>
<p><a href="https://getstream.io/chat/docs/sdk/flutter/" rel="noopener noreferrer">Stream Chat’s Flutter SDK</a> handles message syncing, typing indicators, history, and UI updates, so we don’t build these features manually.</p>
</blockquote>

<p>This setup allows doctors to see their patients, start conversations, view existing chats, and message in real time. Because this is Flutter Web, you can even open two browser tabs: one as a doctor and one as a patient.</p>

<p>Next, we’ll switch to the patient’s view in <code>lib/screens/patient/patient_home_screen.dart</code> to see how the experience mirrors the doctor’s while keeping roles separate.</p>
<h2>
  
  
  Build the Patient’s Home Screen
</h2>

<p>Just like the doctor’s view, our patient home screen gives users two main capabilities:</p>

<ul>
<li>See available doctors.</li>
<li>Start or continue chats with them.</li>
</ul>

<p>Create a <code>patient_home_screen.dart</code> file in the <code>lib/screens/patient</code> directory and enter the code below:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight dart"><code><span class="kn">import</span> <span class="s">'package:flutter/material.dart'</span><span class="o">;</span>  
<span class="kn">import</span> <span class="s">'package:stream</span><span class="err">\</span><span class="s">_chat</span><span class="err">\</span><span class="s">_flutter/stream</span><span class="err">\</span><span class="s">_chat</span><span class="err">\</span><span class="s">_flutter.dart'</span><span class="o">;</span>  
<span class="kn">import</span> <span class="s">'../../stream</span><span class="err">\</span><span class="s">_client.dart'</span><span class="o">;</span>  
<span class="kn">import</span> <span class="s">'../../demo</span><span class="err">\</span><span class="s">_users.dart'</span><span class="o">;</span>  
<span class="kd">class</span> <span class="nc">PatientHomeScreen</span> <span class="kd">extends</span> <span class="n">StatefulWidget</span> <span class="p">{</span>  
  <span class="kd">const</span> <span class="n">PatientHomeScreen</span><span class="p">({</span><span class="n">Key</span><span class="o">?</span> <span class="n">key</span><span class="p">})</span> <span class="o">:</span> <span class="k">super</span><span class="p">(</span><span class="nl">key:</span> <span class="n">key</span><span class="p">);</span>  
  <span class="nd">@override</span>  
  <span class="n">State</span><span class="err">\</span><span class="p">&lt;</span><span class="n">PatientHomeScreen</span><span class="err">\</span><span class="p">&gt;</span> <span class="n">createState</span><span class="p">()</span> <span class="err">\</span><span class="o">=</span><span class="err">\</span><span class="p">&gt;</span> <span class="err">\</span><span class="n">_PatientHomeScreenState</span><span class="p">();</span>  
<span class="p">}</span>  
<span class="kd">class</span> <span class="err">\</span><span class="nc">_PatientHomeScreenState</span> <span class="kd">extends</span> <span class="n">State</span><span class="err">\</span><span class="p">&lt;</span><span class="n">PatientHomeScreen</span><span class="err">\</span><span class="p">&gt;</span> <span class="p">{</span>  
  <span class="kd">late</span> <span class="kd">final</span> <span class="n">StreamChannelListController</span> <span class="err">\</span><span class="n">_channelListController</span><span class="p">;</span>  
  <span class="nd">@override</span>  
  <span class="kt">void</span> <span class="n">initState</span><span class="p">()</span> <span class="p">{</span>  
    <span class="k">super</span><span class="o">.</span><span class="na">initState</span><span class="p">();</span>  
    <span class="err">\</span><span class="n">_channelListController</span> <span class="err">\</span><span class="o">=</span> <span class="n">StreamChannelListController</span><span class="p">(</span>  
      <span class="nl">client:</span> <span class="n">StreamClientProvider</span><span class="o">.</span><span class="na">client</span><span class="p">,</span>  
      <span class="nl">filter:</span> <span class="n">Filter</span><span class="o">.</span><span class="na">in</span><span class="err">\</span><span class="n">_</span><span class="p">(</span><span class="s">'members'</span><span class="p">,</span> <span class="err">\</span><span class="p">[</span><span class="n">StreamClientProvider</span><span class="o">.</span><span class="na">client</span><span class="o">.</span><span class="na">state</span><span class="o">.</span><span class="na">currentUser</span><span class="err">\</span><span class="o">!.</span><span class="na">id</span><span class="err">\</span><span class="p">]),</span>  
      <span class="nl">limit:</span> <span class="mi">20</span><span class="p">,</span>  
    <span class="p">);</span>  
  <span class="p">}</span>  
  <span class="nd">@override</span>  
  <span class="n">Widget</span> <span class="n">build</span><span class="p">(</span><span class="n">BuildContext</span> <span class="n">context</span><span class="p">)</span> <span class="p">{</span>  
    <span class="kd">final</span> <span class="n">currentUser</span> <span class="err">\</span><span class="o">=</span> <span class="n">StreamClientProvider</span><span class="o">.</span><span class="na">client</span><span class="o">.</span><span class="na">state</span><span class="o">.</span><span class="na">currentUser</span><span class="p">;</span>  
    <span class="k">return</span> <span class="n">Scaffold</span><span class="p">(</span>  
      <span class="nl">backgroundColor:</span> <span class="kd">const</span> <span class="n">Color</span><span class="p">(</span><span class="mh">0xFFF7F9FC</span><span class="p">),</span>  
      <span class="nl">appBar:</span> <span class="n">AppBar</span><span class="p">(</span>  
        <span class="nl">title:</span> <span class="kd">const</span> <span class="n">Text</span><span class="p">(</span><span class="s">'Patient'</span><span class="p">,</span> <span class="nl">style:</span> <span class="n">TextStyle</span><span class="p">(</span><span class="nl">color:</span> <span class="n">Color</span><span class="p">(</span><span class="mh">0xFF2A4D9B</span><span class="p">),</span> <span class="nl">fontWeight:</span> <span class="n">FontWeight</span><span class="o">.</span><span class="na">bold</span><span class="p">)),</span>  
        <span class="nl">backgroundColor:</span> <span class="n">Colors</span><span class="o">.</span><span class="na">white</span><span class="p">,</span>  
        <span class="nl">iconTheme:</span> <span class="kd">const</span> <span class="n">IconThemeData</span><span class="p">(</span><span class="nl">color:</span> <span class="n">Color</span><span class="p">(</span><span class="mh">0xFF2A4D9B</span><span class="p">)),</span>  
        <span class="nl">elevation:</span> <span class="mi">1</span><span class="p">,</span>  
      <span class="p">),</span>  
      <span class="nl">body:</span> <span class="n">Padding</span><span class="p">(</span>  
        <span class="nl">padding:</span> <span class="kd">const</span> <span class="n">EdgeInsets</span><span class="o">.</span><span class="na">all</span><span class="p">(</span><span class="mf">8.0</span><span class="p">),</span>  
        <span class="nl">child:</span> <span class="n">Column</span><span class="p">(</span>  
          <span class="nl">children:</span> <span class="err">\</span><span class="p">[</span>  
            <span class="c1">// Doctor list  </span>
            <span class="n">SizedBox</span><span class="p">(</span>  
              <span class="nl">height:</span> <span class="mi">120</span><span class="p">,</span>  
              <span class="nl">child:</span> <span class="n">ListView</span><span class="o">.</span><span class="na">separated</span><span class="p">(</span>  
                <span class="nl">scrollDirection:</span> <span class="n">Axis</span><span class="o">.</span><span class="na">horizontal</span><span class="p">,</span>  
                <span class="nl">itemCount:</span> <span class="n">demoDoctors</span><span class="o">.</span><span class="na">length</span><span class="p">,</span>  
                <span class="nl">separatorBuilder:</span> <span class="p">(</span><span class="err">\</span><span class="n">_</span><span class="p">,</span> <span class="err">\</span><span class="n">_</span><span class="err">\</span><span class="n">_</span><span class="p">)</span> <span class="err">\</span><span class="o">=</span><span class="err">\</span><span class="p">&gt;</span> <span class="kd">const</span> <span class="n">SizedBox</span><span class="p">(</span><span class="nl">width:</span> <span class="mi">16</span><span class="p">),</span>  
                <span class="nl">itemBuilder:</span> <span class="p">(</span><span class="n">context</span><span class="p">,</span> <span class="n">index</span><span class="p">)</span> <span class="p">{</span>  
                  <span class="kd">final</span> <span class="n">doctor</span> <span class="err">\</span><span class="o">=</span> <span class="n">demoDoctors</span><span class="err">\</span><span class="p">[</span><span class="n">index</span><span class="err">\</span><span class="p">];</span>  
                  <span class="k">return</span> <span class="n">Card</span><span class="p">(</span>  
                    <span class="nl">elevation:</span> <span class="mi">2</span><span class="p">,</span>  
                    <span class="nl">shape:</span> <span class="n">RoundedRectangleBorder</span><span class="p">(</span><span class="nl">borderRadius:</span> <span class="n">BorderRadius</span><span class="o">.</span><span class="na">circular</span><span class="p">(</span><span class="mi">12</span><span class="p">)),</span>  
                    <span class="nl">child:</span> <span class="n">Container</span><span class="p">(</span>  
                      <span class="nl">width:</span> <span class="mi">200</span><span class="p">,</span>  
                      <span class="nl">padding:</span> <span class="kd">const</span> <span class="n">EdgeInsets</span><span class="o">.</span><span class="na">all</span><span class="p">(</span><span class="mi">12</span><span class="p">),</span>  
                      <span class="nl">child:</span> <span class="n">Column</span><span class="p">(</span>  
                        <span class="nl">mainAxisAlignment:</span> <span class="n">MainAxisAlignment</span><span class="o">.</span><span class="na">center</span><span class="p">,</span>  
                        <span class="nl">children:</span> <span class="err">\</span><span class="p">[</span>  
                          <span class="n">Text</span><span class="p">(</span><span class="n">doctor</span><span class="o">.</span><span class="na">name</span><span class="p">,</span> <span class="nl">style:</span> <span class="kd">const</span> <span class="n">TextStyle</span><span class="p">(</span><span class="nl">fontWeight:</span> <span class="n">FontWeight</span><span class="o">.</span><span class="na">bold</span><span class="p">)),</span>  
                          <span class="kd">const</span> <span class="n">SizedBox</span><span class="p">(</span><span class="nl">height:</span> <span class="mi">8</span><span class="p">),</span>  
                          <span class="n">ElevatedButton</span><span class="p">(</span>  
                            <span class="nl">onPressed:</span> <span class="p">()</span> <span class="kd">async</span> <span class="p">{</span>  
                              <span class="kd">final</span> <span class="n">client</span> <span class="err">\</span><span class="o">=</span> <span class="n">StreamClientProvider</span><span class="o">.</span><span class="na">client</span><span class="p">;</span>  
                              <span class="kd">final</span> <span class="n">currentUser</span> <span class="err">\</span><span class="o">=</span> <span class="n">client</span><span class="o">.</span><span class="na">state</span><span class="o">.</span><span class="na">currentUser</span><span class="p">;</span>  

                              <span class="k">if</span> <span class="p">(</span><span class="n">currentUser</span> <span class="err">\</span><span class="o">==</span> <span class="kc">null</span><span class="p">)</span> <span class="p">{</span>  
                                <span class="k">if</span> <span class="p">(</span><span class="err">\</span><span class="o">!</span><span class="n">mounted</span><span class="p">)</span> <span class="k">return</span><span class="p">;</span>  
                                <span class="n">ScaffoldMessenger</span><span class="o">.</span><span class="na">of</span><span class="p">(</span><span class="n">context</span><span class="p">)</span><span class="o">.</span><span class="na">showSnackBar</span><span class="p">(</span>  
                                  <span class="kd">const</span> <span class="n">SnackBar</span><span class="p">(</span><span class="nl">content:</span> <span class="n">Text</span><span class="p">(</span><span class="s">'Error: User not logged in'</span><span class="p">)),</span>  
                                <span class="p">);</span>  
                                <span class="k">return</span><span class="p">;</span>  
                              <span class="p">}</span>  

                              <span class="k">try</span> <span class="p">{</span>  
                                <span class="c1">// Create the channel  </span>
                                <span class="kd">final</span> <span class="n">channel</span> <span class="err">\</span><span class="o">=</span> <span class="n">client</span><span class="o">.</span><span class="na">channel</span><span class="p">(</span>  
                                  <span class="s">'messaging'</span><span class="p">,</span>  
                                  <span class="nl">extraData:</span> <span class="p">{</span>  
                                    <span class="s">'members'</span><span class="o">:</span> <span class="err">\</span><span class="p">[</span><span class="n">currentUser</span><span class="o">.</span><span class="na">id</span><span class="p">,</span> <span class="n">doctor</span><span class="o">.</span><span class="na">id</span><span class="err">\</span><span class="p">],</span>  
                                    <span class="s">'created</span><span class="err">\</span><span class="s">_by</span><span class="err">\</span><span class="s">_id'</span><span class="o">:</span> <span class="n">currentUser</span><span class="o">.</span><span class="na">id</span><span class="p">,</span>  
                                  <span class="p">},</span>  
                                <span class="p">);</span>  

                                <span class="c1">// Create the channel on Stream  </span>
                                <span class="k">await</span> <span class="n">channel</span><span class="o">.</span><span class="na">create</span><span class="p">();</span>  

                                <span class="c1">// Navigate to the chat  </span>
                                <span class="k">if</span> <span class="p">(</span><span class="err">\</span><span class="o">!</span><span class="n">mounted</span><span class="p">)</span> <span class="k">return</span><span class="p">;</span>  
                                <span class="n">Navigator</span><span class="o">.</span><span class="na">push</span><span class="p">(</span>  
                                  <span class="n">context</span><span class="p">,</span>  
                                  <span class="n">MaterialPageRoute</span><span class="p">(</span>  
                                    <span class="nl">builder:</span> <span class="p">(</span><span class="n">context</span><span class="p">)</span> <span class="err">\</span><span class="o">=</span><span class="err">\</span><span class="p">&gt;</span> <span class="n">StreamChannel</span><span class="p">(</span>  
                                      <span class="nl">channel:</span> <span class="n">channel</span><span class="p">,</span>  
                                      <span class="nl">child:</span> <span class="kd">const</span> <span class="n">StreamChannelPage</span><span class="p">(),</span>  
                                    <span class="p">),</span>  
                                  <span class="p">),</span>  
                                <span class="p">);</span>  
                              <span class="p">}</span> <span class="k">catch</span> <span class="p">(</span><span class="n">e</span><span class="p">)</span> <span class="p">{</span>  
                                <span class="k">if</span> <span class="p">(</span><span class="err">\</span><span class="o">!</span><span class="n">mounted</span><span class="p">)</span> <span class="k">return</span><span class="p">;</span>  
                                <span class="n">ScaffoldMessenger</span><span class="o">.</span><span class="na">of</span><span class="p">(</span><span class="n">context</span><span class="p">)</span><span class="o">.</span><span class="na">showSnackBar</span><span class="p">(</span>  
                                  <span class="n">SnackBar</span><span class="p">(</span><span class="nl">content:</span> <span class="n">Text</span><span class="p">(</span><span class="s">'Error creating chat: </span><span class="si">${e.toString()}</span><span class="s">'</span><span class="p">)),</span>  
                                <span class="p">);</span>  
                              <span class="p">}</span>  
                            <span class="p">},</span>  
                            <span class="nl">child:</span> <span class="kd">const</span> <span class="n">Text</span><span class="p">(</span><span class="s">'Start Chat'</span><span class="p">),</span>  
                          <span class="p">),</span>
</code></pre>

</div>



<blockquote>
<p>Get the <a href="https://github.com/Tabintel/medtech/tree/master/lib/screens/patient" rel="noopener noreferrer">full code in GitHub</a>.</p>
</blockquote>

<p>When the screen loads, <code>initState()</code> creates a <code>StreamChannelListController</code> that acts as the inbox manager. It connects to <code>StreamClientProvider.client</code> (set up in role_selection_screen.dart) and filters for any channels where the logged-in patient is a member:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight dart"><code><span class="nl">filter:</span> <span class="n">Filter</span><span class="o">.</span><span class="na">in</span><span class="err">\</span><span class="n">_</span><span class="p">(</span><span class="s">'members'</span><span class="p">,</span> <span class="err">\</span><span class="p">[</span><span class="n">StreamClientProvider</span><span class="o">.</span><span class="na">client</span><span class="o">.</span><span class="na">state</span><span class="o">.</span><span class="na">currentUser</span><span class="err">\</span><span class="o">!.</span><span class="na">id</span><span class="err">\</span><span class="p">]),</span>
</code></pre>

</div>



<p>This works because the patient is connected to Stream through getstream-token-server.js. (We will explain this later in the article.)</p>

<p>The Node server takes the patient’s ID ("patient"), uses the Stream API key and secret from .env, generates a user token, and registers them with Stream. Without that step, currentUser would be null and chats wouldn’t work.</p>

<p>The UI starts with a horizontal list of doctors pulled from demoDoctors. </p>

<p>When a patient taps <strong>Start Chat</strong>:</p>

<ol>
<li>We create or retrieve a messaging channel with both patient and doctor IDs.</li>
<li>If it doesn’t exist, Stream creates it automatically.</li>
<li>We call await channel.create() to finalize it.</li>
<li>We navigate to StreamChannelPage, which uses
StreamMessageListView() for real-time messages and    StreamMessageInput() for sending new ones.</li>
</ol>

<p>At the bottom, <code>StreamChannelListView</code> displays all active chats, including ones started by doctors. Stream’s SDK updates this list instantly, so new messages appear in real time.</p>

<p>With both doctor and patient home screens complete, we have a two-way chat system using <a href="https://flutter.dev/multi-platform/web" rel="noopener noreferrer">Flutter Web</a>, <a href="https://getstream.io/chat/docs/sdk/flutter/" rel="noopener noreferrer">Stream Flutter SDK</a>, and a <a href="https://github.com/Tabintel/medtech/blob/master/getstream-token-server.js" rel="noopener noreferrer">Node token server</a>. The next step is to add a shared chat screen functionality.</p>

<h2>
  
  
  Build the Shared Chat Screen
</h2>

<p>Both doctors and patients use the shared chat screen. It provides a reusable UI for a single channel, powered by the Stream Chat Flutter SDK. You pass in a channelId and userId, and the screen connects to that channel, showing real-time messages with Stream’s prebuilt widgets.</p>

<p>Create a <code>chat_screen.dart</code> file in the <code>lib/screens/shared</code> directory and enter the code below:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight dart"><code><span class="kn">import</span> <span class="s">'package:flutter/material.dart'</span><span class="o">;</span>  
<span class="kn">import</span> <span class="s">'package:stream</span><span class="err">\</span><span class="s">_chat</span><span class="err">\</span><span class="s">_flutter/stream</span><span class="err">\</span><span class="s">_chat</span><span class="err">\</span><span class="s">_flutter.dart'</span><span class="o">;</span>  
<span class="kn">import</span> <span class="s">'../../stream</span><span class="err">\</span><span class="s">_client.dart'</span><span class="o">;</span>  
<span class="kd">class</span> <span class="nc">ChatScreen</span> <span class="kd">extends</span> <span class="n">StatefulWidget</span> <span class="p">{</span>  
  <span class="kd">final</span> <span class="kt">String</span> <span class="n">channelId</span><span class="p">;</span>  
  <span class="kd">final</span> <span class="kt">String</span> <span class="n">userId</span><span class="p">;</span>  
  <span class="kd">const</span> <span class="n">ChatScreen</span><span class="p">({</span><span class="k">super</span><span class="o">.</span><span class="na">key</span><span class="p">,</span> <span class="kd">required</span> <span class="k">this</span><span class="o">.</span><span class="na">channelId</span><span class="p">,</span> <span class="kd">required</span> <span class="k">this</span><span class="o">.</span><span class="na">userId</span><span class="p">});</span>  
  <span class="nd">@override</span>  
  <span class="n">State</span><span class="err">\</span><span class="p">&lt;</span><span class="n">ChatScreen</span><span class="err">\</span><span class="p">&gt;</span> <span class="n">createState</span><span class="p">()</span> <span class="err">\</span><span class="o">=</span><span class="err">\</span><span class="p">&gt;</span> <span class="err">\</span><span class="n">_ChatScreenState</span><span class="p">();</span>  
<span class="p">}</span>  
<span class="kd">class</span> <span class="err">\</span><span class="nc">_ChatScreenState</span> <span class="kd">extends</span> <span class="n">State</span><span class="err">\</span><span class="p">&lt;</span><span class="n">ChatScreen</span><span class="err">\</span><span class="p">&gt;</span> <span class="p">{</span>  
  <span class="kd">late</span> <span class="n">Channel</span> <span class="n">channel</span><span class="p">;</span>  
  <span class="nd">@override</span>  
  <span class="kt">void</span> <span class="n">initState</span><span class="p">()</span> <span class="p">{</span>  
    <span class="k">super</span><span class="o">.</span><span class="na">initState</span><span class="p">();</span>  
    <span class="n">channel</span> <span class="err">\</span><span class="o">=</span> <span class="n">StreamClientProvider</span><span class="o">.</span><span class="na">client</span><span class="o">.</span><span class="na">channel</span><span class="p">(</span>  
      <span class="s">'messaging'</span><span class="p">,</span>  
      <span class="nl">id:</span> <span class="n">widget</span><span class="o">.</span><span class="na">channelId</span><span class="p">,</span>  
      <span class="nl">extraData:</span> <span class="p">{</span><span class="s">'members'</span><span class="o">:</span> <span class="err">\</span><span class="p">[</span><span class="n">widget</span><span class="o">.</span><span class="na">userId</span><span class="err">\</span><span class="p">]},</span>  
    <span class="p">);</span>  
    <span class="n">channel</span><span class="o">.</span><span class="na">watch</span><span class="p">();</span>  
  <span class="p">}</span>  
  <span class="nd">@override</span>  
  <span class="n">Widget</span> <span class="n">build</span><span class="p">(</span><span class="n">BuildContext</span> <span class="n">context</span><span class="p">)</span> <span class="p">{</span>  
    <span class="k">return</span> <span class="n">StreamChannel</span><span class="p">(</span>  
      <span class="nl">channel:</span> <span class="n">channel</span><span class="p">,</span>  
      <span class="nl">child:</span> <span class="n">Scaffold</span><span class="p">(</span>  
        <span class="nl">appBar:</span> <span class="n">StreamChannelHeader</span><span class="p">(),</span>  
        <span class="nl">body:</span> <span class="n">Column</span><span class="p">(</span>  
          <span class="nl">children:</span> <span class="err">\</span><span class="p">[</span>  
            <span class="n">Expanded</span><span class="p">(</span><span class="nl">child:</span> <span class="n">StreamMessageListView</span><span class="p">()),</span>  
            <span class="n">StreamMessageInput</span><span class="p">(),</span>  
          <span class="err">\</span><span class="p">],</span>  
        <span class="p">),</span>  
      <span class="p">),</span>  
    <span class="p">);</span>  
  <span class="p">}</span>  
<span class="p">}</span>
</code></pre>

</div>



<p>When the widget loads, <code>initState()</code> creates a Channel from the global client in StreamClientProvider.</p>

<p>It calls:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight dart"><code><span class="n">client</span><span class="o">.</span><span class="na">channel</span><span class="p">(</span>  
  <span class="s">'messaging'</span><span class="p">,</span>  
  <span class="nl">id:</span> <span class="n">widget</span><span class="o">.</span><span class="na">channelId</span><span class="p">,</span>  
  <span class="nl">extraData:</span> <span class="p">{</span><span class="s">'members'</span><span class="o">:</span> <span class="err">\</span><span class="p">[</span><span class="n">widget</span><span class="o">.</span><span class="na">userId</span><span class="err">\</span><span class="p">]},</span>  
<span class="p">)</span>
</code></pre>

</div>



<p>This communicates with Stream to: </p>

<p><em>“Get or create this messaging channel with these members.”</em></p>

<p>If the channel doesn’t exist yet, Stream will create it when <code>channel.create()</code> is called earlier in the flow. The <code>extraData</code> helps identify channel members and supports filtering and admin logic.</p>

<p>After creating the channel object, the code calls channel.watch(). This starts streaming channel state and events—messages, typing indicators, presence updates, and reactions—over a WebSocket connection. Updates arrive instantly without polling.</p>

<p>In the <code>build()</code> method, the UI is wrapped in <code>StreamChannel(channel: channel, child: ...)</code>. StreamChannel is a provider widget that makes the channel available to its children. Inside, the prebuilt widgets handle the chat experience:</p>

<ul>
<li>StreamChannelHeader() → shows the channel name and back button.</li>
<li>StreamMessageListView() → displays the live message feed with avatars, ordering, and read receipts.</li>
<li>StreamMessageInput() → sends messages, attachments, and reactions.</li>
</ul>

<p>These widgets handle sending and receiving messages, rendering updates, retry logic, and syncing with Stream’s servers.</p>

<p>This screen assumes the user is already connected to Stream through StreamClientProvider—a step that happens earlier (for example, in role_selection_screen.dart). That connection involves calling your Node-based token server (getstream-token-server.js), which uses your API secret to generate a user token. The Flutter app then calls client.connectUser(User(id: ...), token) to authenticate.</p>

<p><a href="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fwt0izqmzyvndac44nnne.png" class="article-body-image-wrapper"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fwt0izqmzyvndac44nnne.png" alt="App" width="800" height="342"></a></p>

<p>When a doctor or patient clicks “Start Chat” from their home screen, the app either creates the channel there or navigates into this ChatScreen with the existing channelId. As long as both members are included in the channel’s members list, Stream will route messages correctly and store them in your <a href="https://dashboard.getstream.io/?_gl=1*1hqky2c*_gcl_au*MTk2NjYwMzM1LjE3NjEzMTY4NzM.*_ga*MjAxMDg2ODcxMS4xNzUzMjA1MjM0*_ga_FX552ZCLHK*czE3NjgyMzY1OTgkbzI5JGcxJHQxNzY4MjQzNzg0JGo2MCRsMCRoMzc0OTI5ODgz" rel="noopener noreferrer">dashboard</a>.</p>

<h2>
  
  
  Connect to Stream Client
</h2>

<p>To connect the Flutter app to the Stream client, create a stream.dart file in the lib/stream directory and enter the code below:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight dart"><code><span class="kn">import</span> <span class="s">'package:stream</span><span class="err">\</span><span class="s">_chat</span><span class="err">\</span><span class="s">_flutter/stream</span><span class="err">\</span><span class="s">_chat</span><span class="err">\</span><span class="s">_flutter.dart'</span><span class="o">;</span>  
<span class="kn">import</span> <span class="s">'package:flutter</span><span class="err">\</span><span class="s">_dotenv/flutter</span><span class="err">\</span><span class="s">_dotenv.dart'</span><span class="o">;</span>  
<span class="kn">import</span> <span class="s">'dart:convert'</span><span class="o">;</span>  
<span class="kn">import</span> <span class="s">'package:http/http.dart'</span> <span class="k">as</span> <span class="n">http</span><span class="o">;</span>  
<span class="kd">class</span> <span class="nc">StreamClientProvider</span> <span class="p">{</span>  
  <span class="kd">static</span> <span class="kd">final</span> <span class="n">StreamChatClient</span> <span class="n">client</span> <span class="err">\</span><span class="o">=</span> <span class="n">StreamChatClient</span><span class="p">(</span>  
    <span class="n">dotenv</span><span class="o">.</span><span class="na">env</span><span class="err">\</span><span class="p">[</span><span class="s">'STREAM</span><span class="err">\</span><span class="s">_API</span><span class="err">\</span><span class="s">_KEY'</span><span class="err">\</span><span class="p">]</span><span class="err">\</span><span class="o">!</span><span class="p">,</span>  
    <span class="nl">logLevel:</span> <span class="n">Level</span><span class="o">.</span><span class="na">INFO</span><span class="p">,</span>  
  <span class="p">);</span>  
  <span class="c1">/// Connect a user to the Stream Chat client (production, fetch token from backend)  </span>
  <span class="kd">static</span> <span class="n">Future</span><span class="err">\</span><span class="p">&lt;</span><span class="kt">void</span><span class="err">\</span><span class="p">&gt;</span> <span class="n">connectUser</span><span class="p">(</span><span class="kt">String</span> <span class="n">userId</span><span class="p">,</span> <span class="p">{</span><span class="kt">String</span><span class="o">?</span> <span class="n">name</span><span class="p">})</span> <span class="kd">async</span> <span class="p">{</span>  
    <span class="k">if</span> <span class="p">(</span><span class="n">client</span><span class="o">.</span><span class="na">state</span><span class="o">.</span><span class="na">currentUser</span><span class="o">?.</span><span class="na">id</span> <span class="err">\</span><span class="o">==</span> <span class="n">userId</span><span class="p">)</span> <span class="k">return</span><span class="p">;</span>  
    <span class="k">await</span> <span class="n">client</span><span class="o">.</span><span class="na">disconnectUser</span><span class="p">();</span>  
    <span class="c1">// Fetch token from backend  </span>
    <span class="kd">final</span> <span class="n">tokenEndpoint</span> <span class="err">\</span><span class="o">=</span> <span class="n">dotenv</span><span class="o">.</span><span class="na">env</span><span class="err">\</span><span class="p">[</span><span class="s">'TOKEN</span><span class="err">\</span><span class="s">_ENDPOINT'</span><span class="err">\</span><span class="p">]</span> <span class="o">??</span> <span class="s">'http://localhost:3000/token'</span><span class="p">;</span>  
    <span class="kd">final</span> <span class="n">response</span> <span class="err">\</span><span class="o">=</span> <span class="k">await</span> <span class="n">http</span><span class="o">.</span><span class="na">get</span><span class="p">(</span><span class="kt">Uri</span><span class="o">.</span><span class="na">parse</span><span class="p">(</span><span class="s">'</span><span class="si">$tokenEndpoint</span><span class="s">?user</span><span class="err">\</span><span class="s">_id=</span><span class="si">$userId</span><span class="s">'</span><span class="p">));</span>  
    <span class="k">if</span> <span class="p">(</span><span class="n">response</span><span class="o">.</span><span class="na">statusCode</span> <span class="err">\</span><span class="o">!=</span> <span class="mi">200</span><span class="err">\</span><span class="p">)</span> <span class="p">{</span>  
      <span class="k">throw</span> <span class="n">Exception</span><span class="p">(</span><span class="s">'Failed to fetch Stream Chat token: </span><span class="si">${response.body}</span><span class="s">'</span><span class="p">);</span>  
    <span class="p">}</span>  
    <span class="kd">final</span> <span class="n">token</span> <span class="err">\</span><span class="o">=</span> <span class="n">jsonDecode</span><span class="p">(</span><span class="n">response</span><span class="o">.</span><span class="na">body</span><span class="p">)</span><span class="err">\</span><span class="p">[</span><span class="s">'token'</span><span class="err">\</span><span class="p">];</span>  
    <span class="k">await</span> <span class="n">client</span><span class="o">.</span><span class="na">connectUser</span><span class="p">(</span>  
      <span class="n">User</span><span class="p">(</span><span class="nl">id:</span> <span class="n">userId</span><span class="p">,</span> <span class="nl">name:</span> <span class="n">name</span> <span class="o">??</span> <span class="n">userId</span><span class="p">),</span>  
      <span class="n">token</span><span class="p">,</span>  
    <span class="p">);</span>  
  <span class="p">}</span>  
<span class="p">}</span>
</code></pre>

</div>



<blockquote>
<p>Building your own app? Get early access to our <a href="https://getstream.io/video/livestreaming/" rel="noopener noreferrer">Livestream</a> or <a href="https://getstream.io/video/video-calling/" rel="noopener noreferrer">Video Calling API</a> and launch in days!</p>
</blockquote>

<p>This connects the Flutter app and the Stream Chat API. It loads your API key from .env, initializes a single Stream client for the whole app, and connects users by fetching their token from your backend.</p>

<p>At the top, we import:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight dart"><code><span class="kn">import</span> <span class="s">'package:stream</span><span class="err">\</span><span class="s">_chat</span><span class="err">\</span><span class="s">_flutter/stream</span><span class="err">\</span><span class="s">_chat</span><span class="err">\</span><span class="s">_flutter.dart'</span><span class="o">;</span>  
<span class="kn">import</span> <span class="s">'package:flutter</span><span class="err">\</span><span class="s">_dotenv/flutter</span><span class="err">\</span><span class="s">_dotenv.dart'</span><span class="o">;</span>  
<span class="kn">import</span> <span class="s">'dart:convert'</span><span class="o">;</span>  
<span class="kn">import</span> <span class="s">'package:http/http.dart'</span> <span class="k">as</span> <span class="n">http</span><span class="o">;</span>
</code></pre>

</div>



<ul>
<li>
<strong>stream_chat_flutter</strong> – The official Flutter SDK for Stream Chat.</li>
<li>
<strong>flutter_dotenv</strong> – Loads environment variables from .env, such as your API key and backend URL.</li>
<li>
<strong>dart:convert</strong> – Decodes JSON responses from the server.</li>
<li>
<strong>http</strong> – Makes HTTP requests to your backend to get the chat token.</li>
</ul>

<p>The main class is StreamClientProvider, which creates one StreamChatClient for the entire app:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight dart"><code><span class="kd">static</span> <span class="kd">final</span> <span class="n">StreamChatClient</span> <span class="n">client</span> <span class="err">\</span><span class="o">=</span> <span class="n">StreamChatClient</span><span class="p">(</span>  
  <span class="n">dotenv</span><span class="o">.</span><span class="na">env</span><span class="err">\</span><span class="p">[</span><span class="s">'STREAM</span><span class="err">\</span><span class="s">_API</span><span class="err">\</span><span class="s">_KEY'</span><span class="err">\</span><span class="p">]</span><span class="err">\</span><span class="o">!</span><span class="p">,</span>  
  <span class="nl">logLevel:</span> <span class="n">Level</span><span class="o">.</span><span class="na">INFO</span><span class="p">,</span>  
<span class="p">);</span>
</code></pre>

</div>



<p>Here, the API key comes from .env, and the log level is set to INFO so you can see helpful debug messages about chat events.</p>

<p>The <code>connectUser</code> method is where the actual login happens:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight dart"><code><span class="kd">static</span> <span class="n">Future</span><span class="err">\</span><span class="p">&lt;</span><span class="kt">void</span><span class="err">\</span><span class="p">&gt;</span> <span class="n">connectUser</span><span class="p">(</span><span class="kt">String</span> <span class="n">userId</span><span class="p">,</span> <span class="p">{</span><span class="kt">String</span><span class="o">?</span> <span class="n">name</span><span class="p">})</span> <span class="kd">async</span> <span class="p">{</span>  
  <span class="k">if</span> <span class="p">(</span><span class="n">client</span><span class="o">.</span><span class="na">state</span><span class="o">.</span><span class="na">currentUser</span><span class="o">?.</span><span class="na">id</span> <span class="err">\</span><span class="o">==</span> <span class="n">userId</span><span class="p">)</span> <span class="k">return</span><span class="p">;</span>  
  <span class="k">await</span> <span class="n">client</span><span class="o">.</span><span class="na">disconnectUser</span><span class="p">();</span>
</code></pre>

</div>



<p>First, it checks if the user is already connected with the same ID. If so, it skips the rest. If not, it disconnects any existing users to avoid conflicts.</p>

<p>Next, it fetches the token from your backend:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight dart"><code><span class="kd">final</span> <span class="n">tokenEndpoint</span> <span class="o">=</span> <span class="n">dotenv</span><span class="o">.</span><span class="na">env</span><span class="p">[</span><span class="s">'TOKEN_ENDPOINT'</span><span class="p">]</span> <span class="o">??</span> <span class="s">'http://localhost:3000/token'</span><span class="p">;</span>

<span class="kd">final</span> <span class="n">response</span> <span class="o">=</span> <span class="k">await</span> <span class="n">http</span><span class="o">.</span><span class="na">get</span><span class="p">(</span><span class="kt">Uri</span><span class="o">.</span><span class="na">parse</span><span class="p">(</span><span class="s">'</span><span class="si">$tokenEndpoint</span><span class="s">?user_id=</span><span class="si">$userId</span><span class="s">'</span><span class="p">));</span>
</code></pre>

</div>



<p>The token endpoint is read from .env (falling back to localhost for development). The request includes the user ID as a query parameter.</p>

<p>If the server responds with anything other than 200 OK, it throws an error. Otherwise, it extracts the token from the JSON:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight dart"><code><span class="k">if</span> <span class="p">(</span><span class="n">response</span><span class="o">.</span><span class="na">statusCode</span> <span class="o">!=</span> <span class="mi">200</span><span class="p">)</span> <span class="p">{</span>
  <span class="k">throw</span> <span class="n">Exception</span><span class="p">(</span>
    <span class="s">'Failed to fetch Stream Chat token: </span><span class="si">${response.body}</span><span class="s">'</span><span class="p">,</span>
  <span class="p">);</span>
<span class="p">}</span>
<span class="kd">final</span> <span class="n">token</span> <span class="o">=</span> <span class="n">jsonDecode</span><span class="p">(</span><span class="n">response</span><span class="o">.</span><span class="na">body</span><span class="p">)[</span><span class="s">'token'</span><span class="p">];</span>
</code></pre>

</div>



<p>Finally, it connects the user to Stream:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight dart"><code><span class="k">await</span> <span class="n">client</span><span class="o">.</span><span class="na">connectUser</span><span class="p">(</span>  
  <span class="n">User</span><span class="p">(</span><span class="nl">id:</span> <span class="n">userId</span><span class="p">,</span> <span class="nl">name:</span> <span class="n">name</span> <span class="o">??</span> <span class="n">userId</span><span class="p">),</span>  
  <span class="n">token</span><span class="p">,</span>  
<span class="p">);</span>
</code></pre>

</div>



<p>The SDK uses this ID, optional display name, and the JWT token from your backend to log in securely. The token is generated server-side using your API key and secret, so those sensitive credentials never touch the Flutter app.</p>

<p>Basically, the stream_client.dart file loads your API key from .env, requests a secure token from the server, and connects the user to Stream Chat so they can send and receive messages in real time.</p>

<p><a href="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fwmaagfpzeue2ix7yfb2c.png" class="article-body-image-wrapper"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fwmaagfpzeue2ix7yfb2c.png" alt="App" width="800" height="344"></a></p>

<h2>
  
  
  Set Up the Main App Entry Point
</h2>

<p>In the lib directory, create a main.dart file. This will serve as the entry point for your Flutter application.<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight dart"><code><span class="kn">import</span> <span class="s">'package:flutter/material.dart'</span><span class="o">;</span>  
<span class="kn">import</span> <span class="s">'package:flutter</span><span class="err">\</span><span class="s">_dotenv/flutter</span><span class="err">\</span><span class="s">_dotenv.dart'</span><span class="o">;</span>  
<span class="kn">import</span> <span class="s">'package:stream</span><span class="err">\</span><span class="s">_chat</span><span class="err">\</span><span class="s">_flutter/stream</span><span class="err">\</span><span class="s">_chat</span><span class="err">\</span><span class="s">_flutter.dart'</span><span class="o">;</span>  
<span class="kn">import</span> <span class="s">'package:flutter</span><span class="err">\</span><span class="s">_localizations/flutter</span><span class="err">\</span><span class="s">_localizations.dart'</span><span class="o">;</span>  
<span class="kn">import</span> <span class="s">'stream</span><span class="err">\</span><span class="s">_client.dart'</span><span class="o">;</span>  
<span class="kn">import</span> <span class="s">'screens/auth/role</span><span class="err">\</span><span class="s">_selection</span><span class="err">\</span><span class="s">_screen.dart'</span><span class="o">;</span>  
<span class="kn">import</span> <span class="s">'screens/doctor/doctor</span><span class="err">\</span><span class="s">_home</span><span class="err">\</span><span class="s">_screen.dart'</span><span class="o">;</span>  
<span class="kn">import</span> <span class="s">'screens/patient/patient</span><span class="err">\</span><span class="s">_home</span><span class="err">\</span><span class="s">_screen.dart'</span><span class="o">;</span>  
<span class="kn">import</span> <span class="s">'screens/shared/video</span><span class="err">\</span><span class="s">_call</span><span class="err">\</span><span class="s">_screen.dart'</span><span class="o">;</span>  
<span class="n">Future</span><span class="err">\</span><span class="p">&lt;</span><span class="kt">void</span><span class="err">\</span><span class="p">&gt;</span> <span class="n">main</span><span class="p">()</span> <span class="kd">async</span> <span class="p">{</span>  
  <span class="n">WidgetsFlutterBinding</span><span class="o">.</span><span class="na">ensureInitialized</span><span class="p">();</span>  
  <span class="k">await</span> <span class="n">dotenv</span><span class="o">.</span><span class="na">load</span><span class="p">(</span><span class="nl">fileName:</span> <span class="s">'.env'</span><span class="p">);</span>  
  <span class="n">runApp</span><span class="p">(</span><span class="n">MedTalkApp</span><span class="p">());</span>  
<span class="p">}</span>  
<span class="kd">class</span> <span class="nc">MedTalkApp</span> <span class="kd">extends</span> <span class="n">StatelessWidget</span> <span class="p">{</span>  
  <span class="n">MedTalkApp</span><span class="p">({</span><span class="k">super</span><span class="o">.</span><span class="na">key</span><span class="p">});</span>  
  <span class="kd">final</span> <span class="n">StreamChatClient</span> <span class="n">client</span> <span class="err">\</span><span class="o">=</span> <span class="n">StreamClientProvider</span><span class="o">.</span><span class="na">client</span><span class="p">;</span>  
  <span class="nd">@override</span>  
  <span class="n">Widget</span> <span class="n">build</span><span class="p">(</span><span class="n">BuildContext</span> <span class="n">context</span><span class="p">)</span> <span class="p">{</span>  
    <span class="k">return</span> <span class="n">MaterialApp</span><span class="p">(</span>  
      <span class="nl">title:</span> <span class="s">'MedTalk'</span><span class="p">,</span>  
      <span class="nl">theme:</span> <span class="n">ThemeData</span><span class="p">(</span>  
        <span class="nl">colorScheme:</span> <span class="n">ColorScheme</span><span class="o">.</span><span class="na">fromSeed</span><span class="p">(</span><span class="nl">seedColor:</span> <span class="n">Colors</span><span class="o">.</span><span class="na">blue</span><span class="p">),</span>  
        <span class="nl">useMaterial3:</span> <span class="kc">true</span><span class="p">,</span>  
      <span class="p">),</span>  
      <span class="nl">localizationsDelegates:</span> <span class="err">\</span><span class="p">[</span>  
        <span class="n">GlobalMaterialLocalizations</span><span class="o">.</span><span class="na">delegate</span><span class="p">,</span>  
        <span class="n">GlobalWidgetsLocalizations</span><span class="o">.</span><span class="na">delegate</span><span class="p">,</span>  
        <span class="n">GlobalCupertinoLocalizations</span><span class="o">.</span><span class="na">delegate</span><span class="p">,</span>  
      <span class="err">\</span><span class="p">],</span>  
      <span class="nl">supportedLocales:</span> <span class="err">\</span><span class="p">[</span>  
        <span class="n">Locale</span><span class="p">(</span><span class="s">'en'</span><span class="p">),</span>  
      <span class="err">\</span><span class="p">],</span>  
      <span class="nl">initialRoute:</span> <span class="s">'/'</span><span class="p">,</span>  
      <span class="nl">routes:</span> <span class="p">{</span>  
        <span class="s">'/'</span><span class="o">:</span> <span class="p">(</span><span class="n">context</span><span class="p">)</span> <span class="err">\</span><span class="o">=</span><span class="err">\</span><span class="p">&gt;</span> <span class="kd">const</span> <span class="n">RoleSelectionScreen</span><span class="p">(),</span>  
        <span class="s">'/doctor'</span><span class="o">:</span> <span class="p">(</span><span class="n">context</span><span class="p">)</span> <span class="err">\</span><span class="o">=</span><span class="err">\</span><span class="p">&gt;</span> <span class="kd">const</span> <span class="n">DoctorHomeScreen</span><span class="p">(),</span>  
        <span class="s">'/patient'</span><span class="o">:</span> <span class="p">(</span><span class="n">context</span><span class="p">)</span> <span class="err">\</span><span class="o">=</span><span class="err">\</span><span class="p">&gt;</span> <span class="kd">const</span> <span class="n">PatientHomeScreen</span><span class="p">(),</span>  
        <span class="s">'/video'</span><span class="o">:</span> <span class="p">(</span><span class="n">context</span><span class="p">)</span> <span class="err">\</span><span class="o">=</span><span class="err">\</span><span class="p">&gt;</span> <span class="kd">const</span> <span class="n">VideoCallScreen</span><span class="p">(</span><span class="nl">participantName:</span> <span class="s">'Dr. Sarah Lee'</span><span class="p">),</span>  
      <span class="p">},</span>  
      <span class="nl">builder:</span> <span class="p">(</span><span class="n">context</span><span class="p">,</span> <span class="n">child</span><span class="p">)</span> <span class="err">\</span><span class="o">=</span><span class="err">\</span><span class="p">&gt;</span> <span class="n">StreamChat</span><span class="p">(</span>  
        <span class="nl">client:</span> <span class="n">client</span><span class="p">,</span>  
        <span class="nl">child:</span> <span class="n">child</span><span class="err">\</span><span class="o">!</span><span class="p">,</span>  
      <span class="p">),</span>  
    <span class="p">);</span>  
  <span class="p">}</span>  
<span class="p">}</span>
</code></pre>

</div>



<p>We start by importing the required packages:</p>

<ul>
<li>flutter/material.dart for core Flutter UI components</li>
<li>flutter_dotenv to load environment variables from the .env file</li>
<li>stream_chat_flutter for integrating the Stream Chat SDK</li>
<li>flutter_localizations to add localization support</li>
<li>stream_client.dart` (local file) and screen widgets for different roles and features.</li>
</ul>

<p>Next, we set up the main() function, which ensures Flutter’s bindings are initialized, loads our environment variables, and finally runs the MedTalkApp.</p>

<p>The MedTalkApp widget extends StatelessWidget and defines the app’s overall configuration. It creates a StreamChatClient instance (from StreamClientProvider.client) and sets up the app theme, localization delegates, and supported locales.</p>

<p>For navigation, we define our initial route ('/'), which takes the user to the <strong>Role Selection Screen</strong>, and register other routes for the Doctor Home, Patient Home, and Video Call screens. This makes navigating between different app parts easy, depending on the user’s role and actions.</p>

<p>The builder property of MaterialApp wraps the child widget with the StreamChat widget, passing it on to the client. This ensures the Stream Chat SDK is available throughout the app’s widget tree, enabling messaging and chat features in any screen.</p>

<p>With the app’s entry point ready, the next step is to create a node server to generate Stream Chat tokens.</p>

<h2>
  
  
  Set Up the Node.js Server
</h2>

<p>Create a getstream-token-server.js file in the root directory and enter the code in the <a href="https://github.com/Tabintel/medtech/blob/master/getstream-token-server.js" rel="noopener noreferrer">GitHub repository</a>.</p>

<p>This runs a Node.js + Express server that generates secure JWT tokens for authenticating users with the Stream Chat API.</p>

<blockquote>
<p><strong>Note:</strong> In production, do not hardcode your Stream API secret directly in your Flutter app. Instead, the server/backend generates a timed token for the authenticated user, and your Flutter app uses that token to connect to Stream. This is exactly what this Node token server does in this context.</p>
</blockquote>

<p>We start by importing the required packages:</p>

<ul>
<li>express to create a simple HTTP server</li>
<li>stream-chat to talk to Stream’s API and create tokens</li>
<li>cors so our Flutter web app can call the server without CORS issues</li>
<li>dotenv so we can store our API key and secret safely in a <code>.env</code> file</li>
</ul>

<p>Next, we load the environment variables and check if the STREAM_API_KEY and STREAM_API_SECRET are set. The server will stop immediately if they're missing; we can’t run without them.</p>

<p>We then initialize a StreamChat server client using the API key and secret. This client will be able to create tokens, seed users, and perform admin operations.</p>

<p>The server exposes two main endpoints:</p>

<ul>
<li>/token?user_id=USER_ID – This generates a JWT token for the given user_id. Flutter will call this when connecting to Stream.</li>
<li>/ – A simple HTML page just to verify that the server is running.</li>
</ul>

<p>We also added:</p>

<ul>
<li>Request logging so you can see when tokens are requested.</li>
<li>A *<em>user seeding script *</em>(seedUsers) that adds demo doctors and patients to Stream. These IDs and names match the ones in our Flutter app, so you can test right away.</li>
<li>Graceful shutdown handling so the server closes cleanly when you stop it.</li>
</ul>

<p>Open a terminal and start the server with this command:</p>

<p><code>node getstream-token-server.js</code></p>

<p>Now your Flutter app can request tokens securely and connect to Stream Chat.</p>

<h2>
  
  
  Set Up Demo Users for MedTalk
</h2>

<p>To keep development simple, we’ll define a set of demo doctors and patients that our MedTalk app can use for testing. This will allow us to quickly switch between roles (doctor or patient).</p>

<p>In the lib directory, create a new file called demo_users.dart and add the following code from the <a href="https://github.com/Tabintel/medtech/blob/master/lib/demo_users.dart" rel="noopener noreferrer">GitHub repository</a>.</p>

<p>With this setup, we can easily pull a list of doctors or patients in the UI, making it much faster to test the application.</p>

<p>Now the full Flutter app is set up, and we can begin testing it!</p>

<h2>
  
  
  Test the Virtual Health Consultation App
</h2>

<blockquote>
<p>Select <a href="https://www.google.com/chrome/" rel="noopener noreferrer">Chrome</a> as your app's target device to run and debug a <a href="https://docs.flutter.dev/platform-integration/web/building" rel="noopener noreferrer">Flutter web app</a>:</p>
</blockquote>

<p><code></code><code>bash<br>
flutter run \-d chrome<br>
</code><code></code></p>

<p>You can also choose Chrome as a target device in your IDE.</p>

<p>If you prefer, you can use the edge device type on Windows, or use web-server to navigate to a local URL in the browser of your choice.</p>

<h2>
  
  
  Run From the Command Line
</h2>

<p>If you use Flutter run from the command line, you can now run hot reload on the web with the following command:</p>

<p><code></code><code>bash<br>
flutter run \-d chrome \--web-experimental-hot-reload<br>
</code><code></code><br>
When hot reload is enabled, you can restart your application by pressing "r" in the running terminal or "R".</p>

<p>The Flutter web app loads and opens this URL in the Chrome browser:</p>

<p><a href="http://localhost:60767/" rel="noopener noreferrer">http://localhost:60767/</a></p>

<p>This is the application's landing page.</p>

<p><a href="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fcoflam65psdse7tb4txf.png" class="article-body-image-wrapper"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fcoflam65psdse7tb4txf.png" alt="MedTalk" width="800" height="360"></a></p>

<p>When you click on either <strong>Continue as Doctor</strong> or <strong>Continue as Patient</strong>, it then redirects to a loading screen for a few seconds, like so:</p>

<p><a href="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Ferqptff69ae9xqwjkxrw.png" class="article-body-image-wrapper"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Ferqptff69ae9xqwjkxrw.png" alt="App" width="800" height="363"></a></p>

<p>Which then redirects to the chat page, with this URL (if you selected Patient role), like so:</p>

<p><a href="http://localhost:57331/#/patient" rel="noopener noreferrer">http://localhost:57331//#/patient</a></p>

<p><a href="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fx5z8zgcsd4cchf0qgfm4.png" class="article-body-image-wrapper"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fx5z8zgcsd4cchf0qgfm4.png" alt="Chat" width="" height=""></a></p>

<p>At the moment, there are no active chats yet.</p>

<p>The users then show:</p>

<p><a href="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fe6yphtqaqsxyg5m6a1kq.png" class="article-body-image-wrapper"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fe6yphtqaqsxyg5m6a1kq.png" alt="App" width="800" height="364"></a></p>

<p>Then the patients:</p>

<p><a href="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2F48y098w5gjcagwuwdgje.png" class="article-body-image-wrapper"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2F48y098w5gjcagwuwdgje.png" alt="App" width="800" height="365"></a></p>

<p>And the Patient/Doctor chat loads:</p>

<p><a href="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fspa1h8cbidb5tdol9woq.png" class="article-body-image-wrapper"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fspa1h8cbidb5tdol9woq.png" alt="App" width="800" height="359"></a></p>

<p>From the Stream dashboard, check the chat logs to confirm the application is integrated with Stream through the <a href="https://getstream.io/chat/docs/flutter-dart/" rel="noopener noreferrer">Flutter SDK</a>.</p>

<p><a href="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2F00frvrq3f9xsshf20l2j.png" class="article-body-image-wrapper"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2F00frvrq3f9xsshf20l2j.png" alt="Flutter" width="800" height="344"></a></p>

<blockquote>
<p><strong>Note</strong>: We built a Node.js server for handling the Stream tokens for chat/video.</p>
</blockquote>

<p><a href="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2F7adb0m1oz99063vbbuan.png" class="article-body-image-wrapper"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2F7adb0m1oz99063vbbuan.png" alt="App" width="800" height="211"></a></p>

<p>Test the endpoint in your browser or with curl: </p>

<p><a href="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fkadgj19l0srdo5e8av4k.png" class="article-body-image-wrapper"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fkadgj19l0srdo5e8av4k.png" alt="App" width="800" height="145"></a></p>

<p>You should get a JSON response like:</p>

<p><a href="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fh37j2i2tmnpd2hqjplc4.png" class="article-body-image-wrapper"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fh37j2i2tmnpd2hqjplc4.png" alt="JSON" width="800" height="128"></a></p>

<h2>
  
  
  How to Run the Flutter Web App
</h2>

<p>To connect the Flutter web app with the Stream Flutter SDK, we set up a server using Node.js​​, as shown in the step above. This server generates tokens from Stream Chat using the Stream API, which is connected through the API key set up in the .env file at the root of the project directory codebase.</p>

<p>To do this, we first need to run the Node.js server, seed some users (Doctors and Patients), and assign Stream tokens to each user. This enables users to chat asynchronously on the platform.</p>

<p>To get started running the app, first run the Node.js server in a different terminal.</p>

<blockquote>
<p><strong>Note:</strong> We use two command prompts in a terminal to run this application.</p>
</blockquote>

<p>In the first terminal, run this command:</p>

<p><code></code><code>bash<br>
node getstream-token-server.js<br>
</code><code></code></p>

<p>This starts the node server and shows an output like so:</p>

<p><a href="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fw1acfxably8znjorryaa.png" class="article-body-image-wrapper"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fw1acfxably8znjorryaa.png" alt="Node" width="800" height="286"></a></p>

<p>To test an endpoint, open this URL in your browser: <a href="http://localhost:3000/token?user_id=doctor_anna" rel="noopener noreferrer">http://localhost:3000/token?user_id=doctor_anna</a></p>

<p>It then loads this HTML page, and when you click Get token for doctor anna.</p>

<p><a href="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fe41lv2rtxasoxyedzkhb.png" class="article-body-image-wrapper"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fe41lv2rtxasoxyedzkhb.png" alt="App" width="657" height="304"></a></p>

<p>It generates a token like so:</p>

<p><a href="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fgtaj5qj6u92061bu8xes.png" class="article-body-image-wrapper"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fgtaj5qj6u92061bu8xes.png" alt="Node" width="800" height="340"></a></p>

<p>You can also view the server logs in the Node.js terminal.</p>

<p><a href="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fgdlmyc0xwtv69nfv5dse.png" class="article-body-image-wrapper"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fgdlmyc0xwtv69nfv5dse.png" alt="App" width="729" height="162"></a></p>

<p>The token server is now up and running perfectly with this setup:</p>

<p>Server is listening on <a href="http://localhost:3000" rel="noopener noreferrer">http://localhost:3000</a><br>
 It has successfully generated a token for a Doctor user, doctor_anna<br>
 All demo users are seeded in Stream Chat</p>

<p>Next, let's test the Flutter app, which should now be able to:</p>

<ul>
<li>Connect to the token server</li>
<li>Authenticate users</li>
<li>Create chat channels</li>
</ul>

<p>Run the Flutter app with this command:</p>

<p><code>flutter run \-d chrome</code></p>

<p>The Flutter app begins to start and shows these logs in your terminal. If you look closely at the log messages, you will see that the Flutter app is connected to the Stream API.</p>

<p><a href="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2F881lbc03llhhkjut69ii.png" class="article-body-image-wrapper"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2F881lbc03llhhkjut69ii.png" alt="App" width="800" height="599"></a></p>

<p>The Flutter app then loads and opens in your Chrome browser.</p>

<p><a href="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2F2k0ckukst4paqte09iev.png" class="article-body-image-wrapper"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2F2k0ckukst4paqte09iev.png" alt="App" width="800" height="360"></a></p>

<p>Click on either Continue as Doctor or Patient, and then try starting a new chat. If you select the Doctor role, the chat loads, showing the history of previous chats with Patients.</p>

<p><a href="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fbeas4ryh3hf23lx3tlc5.png" class="article-body-image-wrapper"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fbeas4ryh3hf23lx3tlc5.png" alt="App" width="800" height="357"></a></p>

<p>Click on any user, say Bob Brown, to start a chat</p>

<p><a href="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fpe0mxl7odswisuaie6o6.png" class="article-body-image-wrapper"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fpe0mxl7odswisuaie6o6.png" alt="App" width="800" height="330"></a></p>

<blockquote>
<p>You can also confirm that the messages are sent through Stream by <a href="https://dashboard.getstream.io/app/1407599/chat/apilogs?_gl=1*ttx6av*_gcl_au*MTk2NjYwMzM1LjE3NjEzMTY4NzM.*_ga*MjAxMDg2ODcxMS4xNzUzMjA1MjM0*_ga_FX552ZCLHK*czE3NjgyMzY1OTgkbzI5JGcxJHQxNzY4MjUwMTc0JGo2MCRsMCRoMzc0OTI5ODgz" rel="noopener noreferrer">viewing the real-time chat logs</a> in your dashboard.</p>
</blockquote>

<p><a href="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fydf0qnabxkkcv8wac6jk.png" class="article-body-image-wrapper"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fydf0qnabxkkcv8wac6jk.png" alt="App" width="800" height="344"></a></p>

<p>Now let’s try out the Patient's chat!</p>

<p>Navigate to the homepage of the application with <a href="http://localhost:64150" rel="noopener noreferrer">http://localhost:64150</a>, and then click on Continue as Patient, which then loads the view as shown below:</p>

<blockquote>
<p>The direct URL to the patient view of the app is: <a href="http://localhost:64150/#/patient" rel="noopener noreferrer">http://localhost:64150/#/patient</a></p>
</blockquote>

<p><a href="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fyrak5b20l7d6l42drng5.png" class="article-body-image-wrapper"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fyrak5b20l7d6l42drng5.png" alt="App" width="800" height="323"></a></p>

<p>Click on any of the Doctors, say Doctor Emily, to start a conversation:</p>

<p><a href="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2F596e4fch94om0830fwd1.png" class="article-body-image-wrapper"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2F596e4fch94om0830fwd1.png" alt="App" width="800" height="328"></a></p>

<p><strong>1. As a Doctor:</strong></p>

<ul>
<li> Click "Start Chat" with a patient</li>
<li> Send a message in the chat</li>
<li> The chat should now appear in your chat list</li>
</ul>

<p><strong>2. As a Patient:</strong></p>

<ul>
<li>Log in as the patient you chatted with</li>
<li>You should see the chat in your chat list</li>
<li>You can reply, and the doctor will see it immediately</li>
</ul>

<p><strong>3. Verify Real-time Sync:</strong></p>

<ul>
<li>Open the same chat on two devices (or two different user accounts)</li>
<li>Messages should appear instantly on both devices when sent from either side</li>
</ul>

<p>You can view the total number of messages on the Stream chat dashboard.</p>

<p><a href="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fvdp7lyucoy9gnpblkt4q.png" class="article-body-image-wrapper"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fvdp7lyucoy9gnpblkt4q.png" alt="App" width="800" height="164"></a></p>

<p>To run the app locally, clone the GitHub repository with this command.</p>

<p><code>git clone</code></p>

<blockquote>
<p>Note: As mentioned in the article earlier, make sure you have Flutter set up and installed on your computer. </p>
</blockquote>

<p>Navigate to the project directory, then run this command to install the dependencies needed for the app.</p>

<p><code>flutter pub get</code></p>

<p>Also, ensure your Stream API keys are added to an .env file in the project directory.</p>

<p>Then run the application with:</p>

<p><code>flutter run \-d chrome</code></p>

<p>And that’s it! You have a working virtual consultation app, fully set up.</p>

<h2>
  
  
  Conclusion
</h2>

<p>In this tutorial, you built a MedTalk clone, a <a href="https://getstream.io/blog/hipaa-video-conferencing/" rel="noopener noreferrer">HIPAA-compliant telehealth application</a> powered by Flutter and Stream. By integrating Stream Chat and Stream Video, you enabled secure messaging and real-time video consultations between doctors and patients. This prototype shows how quickly you can deliver compliant, modern healthcare communication experiences with Stream’s SDKs.</p>

