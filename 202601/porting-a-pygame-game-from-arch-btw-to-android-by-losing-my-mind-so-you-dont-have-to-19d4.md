---
Title: Porting a Pygame Game from Arch (BTW) to Android (By losing my mind so you don’t have to)
Description: 
Author: IamAndelib
Date: 2026-01-20T21:32:31.000Z
Robots: noindex,nofollow
Template: index
---
<h3>
  
  
  <strong>Introduction</strong>
</h3>

<p>If you have ever tried to run <em>buildozer</em> android debug on a rolling-release distribution like Arch Linux, you probably know the pain. It usually starts with a simple command and ends with you staring at <em>gcc</em> compiler logs at 3 AM, wondering why <em>longintrepr.h</em> is missing.</p>

<h4>
  
  
  Table of Contents
</h4>

<ul>
<li>The Golden Stack (TL;DR)</li>
<li>Step 1: System Prerequisites</li>
<li>Step 2: The Virtual Environment</li>
<li>Step 3: The "Flag Sanitization" Patch</li>
<li>Step 4: Building the APK</li>
<li>Troubleshooting Cheatsheet</li>
<li>Final Thoughts</li>
</ul>

<h4>
  
  
  <strong>The Golden Stack (TL;DR)</strong>
</h4>

<p>Android cross-compilation is fragile. If you mix the wrong versions of <em>Python, Cython,</em> and _Java, _the build fails. Here is the exact combination that works in 2026:</p>

<div class="table-wrapper-paragraph"><table>
<thead>
<tr>
<th>Tool</th>
<th>Version</th>
<th>The Why</th>
</tr>
</thead>
<tbody>
<tr>
<td><strong>Python</strong></td>
<td><code>3.11</code></td>
<td>3.12+ removed <code>distutils</code>, breaking many Android recipes.</td>
</tr>
<tr>
<td><strong>Cython</strong></td>
<td><code>&lt; 3.0.0</code></td>
<td>Cython 3.0 removed <code>longintrepr.h</code>, breaking Kivy/Pygame recipes.</td>
</tr>
<tr>
<td><strong>Java</strong></td>
<td><code>JDK 17</code></td>
<td>Required by the latest Android SDK/Gradle.</td>
</tr>
</tbody>
</table></div>

<h4>
  
  
  Step 1: System Prerequisites
</h4>

<p>Arch is "bare bones," so we need to manually fetch the headers that <em><code>python-for-android</code> (p4a)</em> needs to compile the recipes.</p>

<p>Run the following in your terminal to install the build toolchain and headers:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight plaintext"><code># Update everything first
sudo pacman -Syu


# Install the build toolchain and headers
# We need jdk17-openjdk for Gradle and base-devel for gcc/make
sudo pacman -S git zip unzip jdk17-openjdk python pip python-virtualenv \
    base-devel autoconf libtool pkg-config zlib ncurses cmake libffi openssl
</code></pre>

</div>



<h4>
  
  
  Step 2: The Virtual Environment (DO NOT SKIP THIS)
</h4>

<p>Never install <em>Buildozer</em> or <em>Cython</em> globally on Arch. System updates will break your build toolchain. We need an isolated environment where we can pin older versions of <em>Cython</em> without messing up the system.<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight plaintext"><code># 1. Create the venv with Python 3.11
python3.11 -m venv .venv


# 2. Activate it
source .venv/bin/activate


# 3. Install the specific versions
# We pin Cython &lt; 3.0 to fix the "longintrepr.h" error
pip install "Cython&lt;3.0" "setuptools&lt;71.0.0" buildozer
</code></pre>

</div>



<h4>
  
  
  Step 3: The "Flag Sanitization" Patch
</h4>

<p>This was the hardest bug to track down.</p>

<p>On Arch Linux, the system compilers are configured with aggressive security and optimization flags like <code>-fcf-protection</code> (Intel Control-flow enforcement) and <code>-march=x86-64</code>.</p>

<p>When Buildozer runs, it sometimes leaks these Host Flags into the Target (Android) build process. The Android NDK compiler sees <code>-march=x86-64</code>, panics because it's trying to build for ARM, and crashes with cryptic errors.</p>

<p><strong>The Fix:</strong> We need to "sanitize" the environment variables in the recipe before the build starts.</p>

<ul>
<li>Navigate to the pygame recipe inside your hidden .buildozer directory. It is usually located here: <code>.buildozer/android/platform/python-for-android/pythonforandroid/recipes/pygame/__init__.py</code>
</li>
<li>Locate the <code>build_armeabi_v7a</code> method and inject this sanitation logic:
</li>
</ul>

<div class="highlight js-code-highlight">
<pre class="highlight plaintext"><code>import re


# ... inside the build_armeabi_v7a method ...
env = self.get_recipe_env(arch)


# 🧹 THE SANITIZER
# Strip flags that break ARM builds (like x86 specific protections)
bad_flags = [
    r'-m(arch|tune|cpu)=\S+',      # e.g., -march=x86-64
    r'-fcf-protection(=\S+)?',     # Intel specific
    r'-fno-plt'                    # PLT hardening
]


for var in ['CFLAGS', 'LDFLAGS']:
    if var in env:
        for pattern in bad_flags:
            env[var] = re.sub(pattern, '', env[var])


# ... proceed with the build ...
</code></pre>

</div>



<p>This simple script strips the x86 flags before they reach the NDK.</p>

<h4>
  
  
  Step 4: Building the APK
</h4>

<p>Now that our environment is clean and our flags are sanitized, we can build.</p>

<p><strong>CRITICAL:</strong> You must export your local <code>venv</code> path before building. If you don't, <em>Buildozer</em> might use the system <em>Cython (v3.0+)</em> and fail.<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight plaintext"><code># Force the build to use our local venv tools
export PATH="$(pwd)/.venv/bin:$PATH"


# Build the debug APK
buildozer android debug
</code></pre>

</div>



<p>This will:</p>

<ul>
<li>Download the Android SDK/NDK (takes a while the first time).</li>
<li>Compile Python 3.11 for ARM.</li>
<li>Package your Pygame app.</li>
</ul>

<p>If everything goes well, you’ll see the glorious <code>Application pushed.</code> message.</p>

<h3>
  
  
  Troubleshooting Cheatsheet
</h3>

<p>If (when) things break, here is your rescue kit.</p>

<h4>
  
  
  1. The <strong>"longintrepr.h not found"</strong> Error
</h4>

<ul>
<li>
<strong>Cause:</strong> You are using <em>Cython 3.0+</em>.</li>
<li>
<strong>Fix:</strong> Run <code>pip install "Cython&lt;3.0"</code> and check your <code>PATH</code>.</li>
</ul>

<h4>
  
  
  2. The App Crashes Immediately
</h4>

<p>Buildozer hides the Python errors. You need <code>logcat</code> to see why your game crashed.<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight plaintext"><code># Filter specifically for Python crashes
adb logcat -s python studio.flappybird.game
</code></pre>

</div>



<h4>
  
  
  3. "Get Android SDK" Hangs
</h4>

<ul>
<li>Cause: You need to accept a license agreement.</li>
<li>Fix: Run the command in the foreground first, or check your internet connection.</li>
</ul>

<h3>
  
  
  Final Thoughts
</h3>

<p>Porting to Android doesn't have to be a nightmare, but on Linux (Arch BTW), you have to respect the toolchain. By pinning your dependencies and sanitizing your compiler flags, you can build reliable APKs even on a bleeding-edge distro like Arch.</p>

<p><strong>Happy Coding!</strong></p>

