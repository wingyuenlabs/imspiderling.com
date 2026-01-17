---
Title: From Python to OpenGL: A Modern, Cross-Platform Survival Guide for OSU CS-450
Description: 
Author: Juan Guerrero
Date: 2026-01-17T21:56:35.000Z
Robots: noindex,nofollow
Template: index
---
<p><strong>By Juan Guerrero</strong></p>

<blockquote>
<p><strong>Disclaimer:</strong> This setup worked for me personally and is shared here to benefit others who might choose to follow it. This guide is based on my own experience in the class and is <strong>not</strong> an enforced standard or the "official" way to do things in CS-450. Always refer to the official class resources for the specific requirements of your assignment.</p>
</blockquote>

<h2>
  
  
  The Shock of the New (Old)
</h2>

<p>If you are an OSU student stepping into <strong>CS-450 (Introduction to Computer Graphics)</strong>, you might be coming from a background of Python, Java, or web development. You're used to <code>pip install</code>, useful error messages, and languages that manage memory for you.</p>

<p>Suddenly, you are dropped into a world of pointers, segmentation faults, linker errors, and manual memory management. You visit the class resources page and see three different zip files: one for Windows (Visual Studio), one for Mac, and one for Linux. Each has its own build system.</p>

<p><strong>There is a better way.</strong></p>

<p>This guide is designed to set up a <strong>unified, modern, cross-platform environment</strong> that works comfortably on Windows, Linux, and Mac. We will replace manual configuration with tools that automate the hard stuff, allowing you to focus on the graphics.</p>




<h2>
  
  
  Why Are We Doing This?
</h2>

<h3>
  
  
  1. The Language: Why C++?
</h3>

<p>You might ask, "Why can't I just use Python?"<br>
Graphics require performance. We need to talk directly to the GPU, manage memory down to the byte, and push millions of triangles per second. C++ gives us that control. </p>

<ul>
<li>  <strong>Memory:</strong> In Python, variables are just references. In C++, you decide if a variable lives on the <strong>stack</strong> (temporary, fast) or the <strong>heap</strong> (permanent, manual).</li>
<li>  <strong>Pointers:</strong> You will pass addresses of data (<code>&amp;data</code>) to OpenGL, not the data itself. This avoids copying massive 3D models around memory, keeping things fast.</li>
</ul>
<h3>
  
  
  2. The Problem: "It Runs on My Machine"
</h3>

<p>The default class setup relies on "project files" (like <code>.sln</code> for Visual Studio) or Makefiles. These are brittle. If you move a file, the project breaks. If you send your code to a partner on a Mac, it breaks.<br>
<strong>The Solution: CMake.</strong><br>
CMake is a meta-build system. You write a "recipe" (<code>CMakeLists.txt</code>) describing your project, and CMake <em>generates</em> the correct project files for whatever computer you are on. It creates the Visual Studio solution on Windows and the Makefile on Linux.</p>
<h3>
  
  
  3. The Nightmare: Dependency Management
</h3>

<p>In Python, you type <code>pip install numpy</code>.<br>
In C++, traditionally, you download a <code>.zip</code> from a random website, extract it, guess where to put the <code>.h</code> files, guess where to put the <code>.lib</code> files, and pray the linker finds them.</p>

<p><strong>The Solution:</strong></p>

<ul>
<li>  <strong>On Windows:</strong> We will use Microsoft's <strong>vcpkg</strong>, a command-line package manager that integrates with Visual Studio.</li>
<li>  <strong>On Linux/Mac:</strong> We will use system packages, but for tricky libraries (like outdated versions of GLUT), we can use <strong>CMake's FetchContent</strong> to download and compile them on the fly.</li>
</ul>


<h2>
  
  
  Part 1: The Environment Setup
</h2>
<h3>
  
  
  1. Windows: Taming the Beast
</h3>

<p>On Windows, we will use <strong>vcpkg</strong> to install our libraries and <strong>Visual Studio 2022</strong>.</p>

<ol>
<li> <strong>Install Git:</strong> <a href="https://git-scm.com/download/win" rel="noopener noreferrer">Download Git for Windows</a>. We need this to clone the vcpkg repository.</li>
<li> <strong>Install vcpkg:</strong>
Open PowerShell and run:
</li>
</ol>
<div class="highlight js-code-highlight">
<pre class="highlight powershell"><code><span class="w">    </span><span class="n">git</span><span class="w"> </span><span class="nx">clone</span><span class="w"> </span><span class="nx">https://github.com/microsoft/vcpkg.git</span><span class="w"> </span><span class="nx">C:\vcpkg</span><span class="w">
    </span><span class="n">cd</span><span class="w"> </span><span class="nx">C:\vcpkg</span><span class="w">
    </span><span class="o">.</span><span class="n">\bootstrap-vcpkg.bat</span><span class="w">
    </span><span class="o">.</span><span class="nx">\vcpkg</span><span class="w"> </span><span class="nx">integrate</span><span class="w"> </span><span class="nx">install</span><span class="w">
</span></code></pre>

</div>


<p><strong>Why?</strong> <code>bootstrap</code> compiles the package manager itself. <code>integrate install</code> tells Visual Studio "Hey, look in C:\vcpkg for libraries". This means you never have to manually add "Include Directories" again.</p>

<ol>
<li> <strong>Install Graphics Libraries:</strong>
</li>
</ol>
<div class="highlight js-code-highlight">
<pre class="highlight powershell"><code><span class="w">    </span><span class="o">.</span><span class="n">\vcpkg</span><span class="w"> </span><span class="nx">install</span><span class="w"> </span><span class="nx">glew</span><span class="w"> </span><span class="nx">freeglut</span><span class="w"> </span><span class="nx">opengl</span><span class="w">
</span></code></pre>

</div>

<h3>
  
  
  2. Linux &amp; macOS: The Native Way
</h3>

<p>On Unix systems, package managers are built-in. However, CMake assumes certain low-level libraries are present.</p>

<ul>
<li>  <strong>Linux (Fedora):</strong>
</li>
</ul>
<div class="highlight js-code-highlight">
<pre class="highlight shell"><code>    <span class="nb">sudo </span>dnf <span class="nb">install </span>cmake gcc-c++ glew-devel <span class="se">\</span>
    libXi-devel libXmu-devel libXext-devel libXrandr-devel libXxf86vm-devel
</code></pre>

</div>


<p><strong>Why?</strong> We install <code>glew-devel</code> and the X11 libraries, but we <strong>skip</strong> the system <code>freeglut</code>. As we'll see in Part 3.5, we will tell CMake to download and build a specific version of FreeGLUT to ensure it works perfectly with Wayland.</p>

<ul>
<li>  <strong>Linux (Ubuntu):</strong>
</li>
</ul>
<div class="highlight js-code-highlight">
<pre class="highlight shell"><code>    <span class="nb">sudo </span>apt <span class="nb">install </span>cmake g++ libglew-dev <span class="se">\</span>
    libxi-dev libxmu-dev libxext-dev libxrandr-dev
</code></pre>

</div>


<ul>
<li>  <strong>macOS:</strong>

<ol>
<li> Install Homebrew: <code>brew.sh</code>
</li>
<li> <code>brew install cmake glew freeglut</code>
</li>
</ol>
</li>
</ul>


<h2>
  
  
  Part 2: The Editor - Neovim
</h2>

<p>You can use Visual Studio 2022, but for a consistent experience across all OSs, I recommend <strong>Neovim</strong>. It's a terminal-based editor that, when configured, is faster and more powerful than VS Code.</p>
<h3>
  
  
  Why Neovim for C++?
</h3>

<ol>
<li> <strong>LSP (Language Server Protocol):</strong> We will hook Neovim up to specialized "Language Servers" that act like a brain for your editor. We need three specific ones for this class:

<ul>
<li>  <strong><code>clangd</code>:</strong> The brain for C/C++. It provides autocomplete and real-time error checking.</li>
<li>  <strong><code>cmake-language-server</code>:</strong> Helps you write <code>CMakeLists.txt</code> without typos.</li>
<li>  <strong><code>glsl_analyzer</code>:</strong> This is a life-saver for Projects 6 &amp; 7. It provides syntax checking and autocomplete for the GLSL shader language.</li>
</ul>
</li>
<li> <strong>Regex Block Editing:</strong> You will often need to edit blocks of coordinate data.

<ul>
<li>  <em>Example:</em> You have 50 lines that look like <code>v 1.0 2.0 3.0</code>. You want to change them to <code>vertex(1.0, 2.0, 3.0);</code>.</li>
<li>  In Neovim, you press <code>Ctrl-v</code> to select the column, shift-I to insert text on <em>all lines at once</em>, and escape to apply. It saves hours of typing.</li>
</ul>
</li>
<li> <strong>Speed:</strong> It opens instantly. No loading splash screens.</li>
</ol>
<h3>
  
  
  Quick Setup (LazyVim)
</h3>

<p>Configuring Vim from scratch is hard. Use <strong>LazyVim</strong>, a pre-made configuration.</p>

<ol>
<li> <strong>Install Neovim:</strong> (Check your OS package manager).</li>
<li> <strong>Install LazyVim:</strong>
</li>
</ol>
<div class="highlight js-code-highlight">
<pre class="highlight shell"><code>    git clone https://github.com/LazyVim/starter ~/.config/nvim
    <span class="nb">rm</span> <span class="nt">-rf</span> ~/.config/nvim/.git
    nvim
</code></pre>

</div>


<ol>
<li> <strong>Install C++ &amp; Shader Tools:</strong>
Inside Neovim, type the following command to install the brains:
<code>:MasonInstall clangd cmake-language-server glsl_analyzer</code>
</li>
</ol>


<h2>
  
  
  Part 3: Modernizing the Code
</h2>

<p>The class provides a file named <code>sample.cpp</code> inside <code>Sample2022.zip</code>. It works, but it's full of legacy <code>#ifdef</code> code to handle platform differences manually. We are going to clean this up using CMake.</p>
<h3>
  
  
  Step 1: The <code>CMakeLists.txt</code>
</h3>

<p>Create a file named <code>CMakeLists.txt</code> next to your <code>sample.cpp</code>. This file tells CMake how to build your program.<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight cmake"><code><span class="nb">cmake_minimum_required</span><span class="p">(</span>VERSION 3.15<span class="p">)</span>
<span class="nb">project</span><span class="p">(</span>cs450_project<span class="p">)</span>

<span class="nb">set</span><span class="p">(</span>CMAKE_CXX_STANDARD 17<span class="p">)</span>

<span class="c1"># 1. Find the libraries automatically</span>
<span class="c1"># CMake looks for standard locations (and vcpkg locations)</span>
<span class="nb">find_package</span><span class="p">(</span>OpenGL REQUIRED<span class="p">)</span>
<span class="nb">find_package</span><span class="p">(</span>GLEW REQUIRED<span class="p">)</span>
<span class="nb">find_package</span><span class="p">(</span>GLUT REQUIRED<span class="p">)</span>

<span class="c1"># 2. Add all your C++ files here.</span>
<span class="c1"># NOTE: If you uncomment 'bmptotexture.cpp' in the code, you MUST add it here too!</span>
<span class="nb">add_executable</span><span class="p">(</span>my_project
    sample.cpp
    <span class="c1"># bmptotexture.cpp</span>
    <span class="c1"># loadobjmtlfiles.cpp</span>
<span class="p">)</span>

<span class="c1"># 3. Tell the compiler where the .h files are</span>
<span class="nb">target_include_directories</span><span class="p">(</span>my_project PRIVATE
    <span class="si">${</span><span class="nv">OPENGL_INCLUDE_DIR</span><span class="si">}</span>
    <span class="si">${</span><span class="nv">GLEW_INCLUDE_DIRS</span><span class="si">}</span>
    <span class="si">${</span><span class="nv">GLUT_INCLUDE_DIRS</span><span class="si">}</span>
<span class="p">)</span>

<span class="c1"># 4. Link the libraries to the executable</span>
<span class="c1"># This connects the actual compiled code (.lib or .a files) to your program</span>
<span class="nb">target_link_libraries</span><span class="p">(</span>my_project PRIVATE
    <span class="si">${</span><span class="nv">OPENGL_LIBRARIES</span><span class="si">}</span>
    <span class="si">${</span><span class="nv">GLEW_LIBRARIES</span><span class="si">}</span>
    <span class="si">${</span><span class="nv">GLUT_LIBRARIES</span><span class="si">}</span>
<span class="p">)</span>
</code></pre>

</div>



<h3>
  
  
  Step 2: Fixing <code>sample.cpp</code> Includes
</h3>

<p>In C++, <code>#include</code> literally pastes the file contents into your code. The old <code>sample.cpp</code> uses local copies of headers (<code>"glew.h"</code>). We want to use the system installed ones (<code>&lt;GL/glew.h&gt;</code>).</p>

<p><strong>Find this block in <code>sample.cpp</code>:</strong><br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight cpp"><code><span class="cp">#ifdef WIN32
#include</span> <span class="cpf">&lt;windows.h&gt;</span><span class="cp">
#pragma warning(disable:4996)
#endif
</span>
<span class="cp">#include</span> <span class="cpf">"glew.h"</span><span class="cp">
#include</span> <span class="cpf">&lt;GL/gl.h&gt;</span><span class="cp">
#include</span> <span class="cpf">&lt;GL/glu.h&gt;</span><span class="cp">
#include</span> <span class="cpf">"glut.h"</span><span class="cp">
</span></code></pre>

</div>



<p><strong>Replace it with this:</strong><br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight cpp"><code><span class="cp">#include</span> <span class="cpf">&lt;stdio.h&gt;</span><span class="cp">
#include</span> <span class="cpf">&lt;stdlib.h&gt;</span><span class="cp">
#include</span> <span class="cpf">&lt;ctype.h&gt;</span><span class="cp">
#include</span> <span class="cpf">&lt;time.h&gt;</span><span class="cp">
#define _USE_MATH_DEFINES
#include</span> <span class="cpf">&lt;math.h&gt;</span><span class="cp">
</span>
<span class="c1">// Universal include block</span>
<span class="cp">#include</span> <span class="cpf">&lt;GL/glew.h&gt;</span><span class="cp">
#include</span> <span class="cpf">&lt;GL/freeglut.h&gt;</span><span class="c1"> </span><span class="cp">
</span><span class="c1">// Note: freeglut.h includes GL/gl.h and GL/glu.h automatically</span>
</code></pre>

</div>



<p><strong>Why?</strong><br>
We use <code>&lt; &gt;</code> to tell the compiler "look in the system folders" (where vcpkg/brew put the files). We removed the <code>#ifdef WIN32</code> because CMake handles platform differences for us.</p>
<h3>
  
  
  Step 3: Fixing Initialization
</h3>

<p>The <code>main()</code> function in <code>sample.cpp</code> has a specific check for Windows to initialize GLEW. <strong>This is a trap.</strong> Linux/Mac users using modern OpenGL profiles <em>also</em> need to initialize GLEW.</p>

<p><strong>Change the initialization code to:</strong><br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight cpp"><code>    <span class="c1">// Initialize GLEW on ALL platforms</span>
    <span class="n">GLenum</span> <span class="n">err</span> <span class="o">=</span> <span class="n">glewInit</span><span class="p">(</span> <span class="p">);</span>
    <span class="k">if</span><span class="p">(</span> <span class="n">err</span> <span class="o">!=</span> <span class="n">GLEW_OK</span> <span class="p">)</span>
    <span class="p">{</span>
        <span class="n">fprintf</span><span class="p">(</span> <span class="n">stderr</span><span class="p">,</span> <span class="s">"glewInit Error: %s</span><span class="se">\n</span><span class="s">"</span><span class="p">,</span> <span class="n">glewGetErrorString</span><span class="p">(</span><span class="n">err</span><span class="p">)</span> <span class="p">);</span>
    <span class="p">}</span>
</code></pre>

</div>



<p><strong>Why?</strong> <code>glewInit</code> loads the function pointers for OpenGL extensions. Without it, calling modern functions will cause a crash.</p>

<h3>
  
  
  Part 3.5: Advanced CMake - The Robust "Wayland-Ready" Recipe
</h3>

<p>The simple <code>CMakeLists.txt</code> above works for many, but on modern Linux (Fedora/Ubuntu) with Wayland, system libraries can break.</p>

<p><strong>The Solution:</strong> Use this robust <code>CMakeLists.txt</code> instead. It automatically downloads and patches a compatible version of FreeGLUT if you are on Linux, while using standard libraries on Windows/Mac.</p>

<p><strong>Copy-Paste this into your <code>CMakeLists.txt</code>:</strong><br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight cmake"><code><span class="nb">cmake_minimum_required</span><span class="p">(</span>VERSION 3.15<span class="p">)</span>
<span class="nb">project</span><span class="p">(</span>cs450_project<span class="p">)</span>

<span class="nb">set</span><span class="p">(</span>CMAKE_CXX_STANDARD 17<span class="p">)</span>

<span class="nb">find_package</span><span class="p">(</span>OpenGL REQUIRED<span class="p">)</span>
<span class="nb">find_package</span><span class="p">(</span>GLEW REQUIRED<span class="p">)</span>

<span class="c1"># --- Advanced Dependency Management ---</span>
<span class="nb">if</span><span class="p">(</span>UNIX <span class="nf">AND</span> <span class="p">(</span>NOT APPLE<span class="p">))</span>
  <span class="nb">include</span><span class="p">(</span>FetchContent<span class="p">)</span>
  <span class="nb">include</span><span class="p">(</span>ExternalProject<span class="p">)</span>

  <span class="c1"># Build FreeGLUT from source to ensure compatibility with modern Wayland desktops</span>
  <span class="c1"># and fix common header conflicts (Patch step)</span>
  <span class="nf">ExternalProject_Add</span><span class="p">(</span>
    freeglut
    GIT_REPOSITORY https://github.com/freeglut/freeglut
    GIT_TAG <span class="s2">"v3.6.0"</span>
    PATCH_COMMAND sh -c <span class="s2">"sed -i 's/^</span><span class="se">\\</span><span class="s2">(.*fgPlatformDestroy.*</span><span class="se">\\</span><span class="s2">)/</span><span class="se">\\</span><span class="s2">/</span><span class="se">\\</span><span class="s2">/</span><span class="se">\\</span><span class="s2">1/' &lt;SOURCE_DIR&gt;/src/egl/fg_init_egl.h"</span>
    CMAKE_ARGS -DFREEGLUT_BUILD_DEMOS=OFF -DFREEGLUT_BUILD_SHARED_LIBS=OFF -DCMAKE_POSITION_INDEPENDENT_CODE=ON -DCMAKE_INSTALL_PREFIX=&lt;INSTALL_DIR&gt; -DCMAKE_INSTALL_LIBDIR=lib
    BUILD_COMMAND cmake --build &lt;BINARY_DIR&gt;
    INSTALL_COMMAND cmake --install &lt;BINARY_DIR&gt;
  <span class="p">)</span>

  <span class="c1"># Link against the locally built version</span>
  <span class="nf">ExternalProject_Get_Property</span><span class="p">(</span>freeglut INSTALL_DIR<span class="p">)</span>
  <span class="nb">set</span><span class="p">(</span>PROGRAM_FREEGLUT_LINK_LIBRARY <span class="si">${</span><span class="nv">INSTALL_DIR</span><span class="si">}</span>/lib/libglut.a<span class="p">)</span>
  <span class="nb">include_directories</span><span class="p">(</span><span class="si">${</span><span class="nv">INSTALL_DIR</span><span class="si">}</span>/include<span class="p">)</span>
<span class="nb">else</span><span class="p">()</span>
  <span class="c1"># On Windows/Mac, use the system or vcpkg installed GLUT</span>
  <span class="nb">find_package</span><span class="p">(</span>GLUT REQUIRED<span class="p">)</span>
  <span class="nb">set</span><span class="p">(</span>PROGRAM_FREEGLUT_LINK_LIBRARY GLUT::GLUT<span class="p">)</span>
<span class="nb">endif</span><span class="p">()</span>

<span class="c1"># --- Project Files ---</span>
<span class="nb">add_executable</span><span class="p">(</span>my_project
    sample.cpp
    <span class="c1"># bmptotexture.cpp</span>
    <span class="c1"># loadobjmtlfiles.cpp</span>
<span class="p">)</span>

<span class="c1"># --- Include Directories ---</span>
<span class="nb">target_include_directories</span><span class="p">(</span>my_project PRIVATE
    <span class="si">${</span><span class="nv">OPENGL_INCLUDE_DIR</span><span class="si">}</span>
    <span class="si">${</span><span class="nv">GLEW_INCLUDE_DIRS</span><span class="si">}</span>
    <span class="si">${</span><span class="nv">GLUT_INCLUDE_DIRS</span><span class="si">}</span>
<span class="p">)</span>

<span class="c1"># --- Linking ---</span>
<span class="nb">target_link_libraries</span><span class="p">(</span>my_project PRIVATE
    <span class="si">${</span><span class="nv">OPENGL_LIBRARIES</span><span class="si">}</span>
    <span class="si">${</span><span class="nv">GLEW_LIBRARIES</span><span class="si">}</span>
    <span class="si">${</span><span class="nv">PROGRAM_FREEGLUT_LINK_LIBRARY</span><span class="si">}</span>
<span class="p">)</span>

<span class="c1"># Linux-specific: Link X11 libraries required by static FreeGLUT</span>
<span class="nb">if</span><span class="p">(</span>UNIX <span class="nf">AND</span> <span class="p">(</span>NOT APPLE<span class="p">))</span>
    <span class="nb">add_dependencies</span><span class="p">(</span>my_project freeglut<span class="p">)</span>
    <span class="nb">target_link_libraries</span><span class="p">(</span>my_project PRIVATE X11 Xi Xrandr Xxf86vm<span class="p">)</span>
<span class="nb">endif</span><span class="p">()</span>

<span class="c1"># Auto-copy assets to build folder (if they exist)</span>
<span class="nb">if</span><span class="p">(</span>EXISTS <span class="s2">"</span><span class="si">${</span><span class="nv">CMAKE_SOURCE_DIR</span><span class="si">}</span><span class="s2">/models"</span><span class="p">)</span>
    <span class="nb">file</span><span class="p">(</span>COPY <span class="s2">"</span><span class="si">${</span><span class="nv">CMAKE_SOURCE_DIR</span><span class="si">}</span><span class="s2">/models"</span> DESTINATION <span class="s2">"</span><span class="si">${</span><span class="nv">PROJECT_BINARY_DIR</span><span class="si">}</span><span class="s2">"</span><span class="p">)</span>
<span class="nb">endif</span><span class="p">()</span>
</code></pre>

</div>



<p><em>This script allows us to script complex compatibility layers, ensuring that "It works on my machine" translates to "It works on your Wayland machine too."</em></p>

<p>Now, the magic of CMake. Instead of memorizing different commands for every OS, we follow a standard process: <strong>Configure</strong>, <strong>Build</strong>, <strong>Run</strong>.</p>

<h3>
  
  
  Option A: The Terminal (Linux, macOS, Windows PowerShell)
</h3>

<p>This is the universal way to build C++ projects.</p>

<ol>
<li> <strong>Create a Build Directory:</strong>
We don't want to clutter our source code with temporary build files.
</li>
</ol>

<div class="highlight js-code-highlight">
<pre class="highlight shell"><code>    <span class="nb">mkdir </span>build
    <span class="nb">cd </span>build
</code></pre>

</div>



<ol>
<li> <strong>Configure (The "CMake" Step):</strong>
Run CMake to look at your <code>CMakeLists.txt</code> (in the parent directory, hence <code>..</code>) and generate the actual build files (Makefiles).
</li>
</ol>

<div class="highlight js-code-highlight">
<pre class="highlight shell"><code>    cmake ..
</code></pre>

</div>



<p><em>If this fails, it means CMake couldn't find a library. Check your <code>CMakeLists.txt</code> or install the missing package.</em></p>

<ol>
<li> <strong>Build:</strong>
Now we tell CMake to compile the code.
</li>
</ol>

<div class="highlight js-code-highlight">
<pre class="highlight shell"><code>    cmake <span class="nt">--build</span> <span class="nb">.</span>
</code></pre>

</div>



<p><em>The <code>.</code> tells CMake to build the project in the current directory.</em></p>

<ol>
<li> <strong>Run:</strong>
If the build succeeded, you will see an executable file.
<strong>Linux/Mac:</strong> You must prefix the command with <code>./</code> to tell the shell "run the program in <em>this</em> folder", not a system command.
</li>
</ol>

<div class="highlight js-code-highlight">
<pre class="highlight shell"><code>./my_project
</code></pre>

</div>



<p><strong>Windows (PowerShell):</strong><br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight powershell"><code><span class="o">.</span><span class="n">\Debug\my_project.exe</span><span class="w">
</span></code></pre>

</div>



<h3>
  
  
  Option B: Visual Studio 2022 (Windows)
</h3>

<p>Visual Studio now supports CMake natively.</p>

<ol>
<li><p><strong>Open the Folder:</strong><br>
Launch Visual Studio. Click <strong>"Open a Local Folder"</strong> and select the folder containing your <code>CMakeLists.txt</code>.</p></li>
<li><p><strong>Wait for Configuration:</strong><br>
Look at the "Output" window. You should see CMake scanning your system. It will find the libraries you installed with vcpkg automatically.</p></li>
<li><p><strong>Run:</strong><br>
In the top toolbar, select <code>my_project.exe</code> as the startup item and press <strong>F5</strong>.</p></li>
</ol>




<h2>
  
  
  Part 5: Managing Assets &amp; Subdirectories
</h2>

<p>In Python, you might just open <code>cat.jpg</code>. In C++, where the program runs matters.</p>

<h3>
  
  
  1. The "Current Working Directory" Trap
</h3>

<p>When you run a program from Visual Studio, the "working directory" (where it looks for files) might be different from where your source code is. This means <code>fopen("models/cat.obj")</code> might fail because the program is running inside <code>build/Debug/</code>.</p>

<p><strong>The CMake Fix:</strong><br>
Add this to the end of your <code>CMakeLists.txt</code> to automatically copy your assets folder to the build directory every time you compile:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight cmake"><code><span class="nb">file</span><span class="p">(</span>COPY models DESTINATION <span class="si">${</span><span class="nv">PROJECT_BINARY_DIR</span><span class="si">}</span><span class="p">)</span>
</code></pre>

</div>



<h3>
  
  
  2. Fixing <code>loadobjmtlfiles.cpp</code> for Subdirectories
</h3>

<p>The provided class code for loading OBJ files (<code>loadobjmtlfiles.cpp</code>) sometimes assumes textures are in the same folder as the executable. If your OBJ file refers to a texture <code>skin.bmp</code>, but that file is actually in <code>models/skin.bmp</code>, the loader will fail.</p>

<p>We can fix this by updating the loader to be "path aware".</p>

<p><strong>In <code>loadobjmtlfiles.cpp</code>, find where <code>map_kd</code> (diffuse texture) is parsed:</strong></p>

<p><em>Original Code:</em><br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight cpp"><code><span class="k">if</span><span class="p">(</span> <span class="n">strcmp</span><span class="p">(</span> <span class="n">tok</span><span class="p">,</span> <span class="s">"map_kd"</span> <span class="p">)</span> <span class="o">==</span> <span class="mi">0</span> <span class="p">)</span> <span class="p">{</span>
    <span class="n">tok</span> <span class="o">=</span> <span class="n">strtok</span><span class="p">(</span> <span class="nb">NULL</span><span class="p">,</span> <span class="n">DELIMS</span> <span class="p">);</span>
    <span class="c1">// calls BmpToTexture(tok) directly</span>
<span class="p">}</span>
</code></pre>

</div>



<p><em>Modernized C++ Fix:</em><br>
Use <code>std::filesystem</code> to prepend the parent directory of the OBJ file to the texture filename.<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight cpp"><code><span class="cp">#include</span> <span class="cpf">&lt;filesystem&gt;</span><span class="c1"> // Add this at the top</span><span class="cp">
</span>
<span class="c1">// ... inside the loop ...</span>
<span class="k">if</span> <span class="p">(</span><span class="n">strcmp</span><span class="p">(</span><span class="n">tok</span><span class="p">,</span> <span class="s">"map_kd"</span><span class="p">)</span> <span class="o">==</span> <span class="mi">0</span><span class="p">)</span> <span class="p">{</span>
    <span class="n">tok</span> <span class="o">=</span> <span class="n">strtok</span><span class="p">(</span><span class="nb">NULL</span><span class="p">,</span> <span class="n">DELIMS</span><span class="p">);</span>

    <span class="c1">// Construct the full path</span>
    <span class="n">std</span><span class="o">::</span><span class="n">filesystem</span><span class="o">::</span><span class="n">path</span> <span class="n">texturePath</span><span class="p">(</span><span class="n">tok</span><span class="p">);</span>
    <span class="k">if</span> <span class="p">(</span><span class="o">!</span><span class="n">texturePath</span><span class="p">.</span><span class="n">has_parent_path</span><span class="p">()</span> <span class="o">&amp;&amp;</span> <span class="o">!</span><span class="n">basePath</span><span class="p">.</span><span class="n">empty</span><span class="p">())</span> <span class="p">{</span>
        <span class="n">texturePath</span> <span class="o">=</span> <span class="n">basePath</span> <span class="o">/</span> <span class="n">texturePath</span><span class="p">;</span>
    <span class="p">}</span>

    <span class="c1">// Convert back to string for the old function</span>
    <span class="n">std</span><span class="o">::</span><span class="n">string</span> <span class="n">fullPath</span> <span class="o">=</span> <span class="n">texturePath</span><span class="p">.</span><span class="n">generic_string</span><span class="p">();</span>
    <span class="n">BmpToTexture</span><span class="p">(</span> <span class="p">(</span><span class="kt">char</span><span class="o">*</span><span class="p">)</span><span class="n">fullPath</span><span class="p">.</span><span class="n">c_str</span><span class="p">(),</span> <span class="p">...</span> <span class="p">);</span>
<span class="p">}</span>
</code></pre>

</div>






<h2>
  
  
  Part 6: Tips for the Python Developer
</h2>

<h3>
  
  
  1. Compilation vs Interpretation
</h3>

<p>In Python, if you have a syntax error on line 50, the program runs until line 49.<br>
In C++, the <strong>Compiler</strong> reads your whole file first. If there is one typo, <em>nothing</em> runs.<br>
Then, the <strong>Linker</strong> combines your code with libraries. If you get "Undefined Symbol", it means you declared a function but didn't give the Linker the <code>.cpp</code> file that contains it. Check your <code>add_executable</code> list in CMake!</p>

<h3>
  
  
  2. Segfaults (The Silent Killer)
</h3>

<p>If your program crashes instantly without an error message, you likely have a <strong>Segmentation Fault</strong>. This usually means you tried to access a list item that doesn't exist (like <code>arr[10]</code> in a list of size 5) or used a pointer that points to nothing (<code>NULL</code>).<br>
<strong>Tip:</strong> Use a debugger. In Visual Studio, it will pause exactly where the crash happens and show you the bad variable.</p>




<h2>
  
  
  Conclusion
</h2>

<p>Setting up C++ graphics environments used to be a rite of passage involving pain and suffering. With <strong>CMake</strong> and <strong>vcpkg</strong>, it becomes a repeatable, scriptable process. You have now graduated from "It runs on my machine" to "It runs anywhere."</p>

<p>Good luck with CS-450. May your framerates be high and your compile times low!</p>




<h2>
  
  
  References
</h2>

<ul>
<li>  <strong>Official CS-450 Class Resources Page:</strong> <a href="https://web.engr.oregonstate.edu/~mjb/cs450/" rel="noopener noreferrer">https://web.engr.oregonstate.edu/~mjb/cs450/</a>
</li>
</ul>

