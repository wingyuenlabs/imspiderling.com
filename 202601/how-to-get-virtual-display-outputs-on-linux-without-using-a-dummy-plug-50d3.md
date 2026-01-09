---
Title: How to get virtual display outputs on Linux without using a dummy plug
Description: 
Author: Fox
Date: 2026-01-09T21:53:22.000Z
Robots: noindex,nofollow
Template: index
---
<p>This guide is mainly for Xorg, on Wayland there are easier way of achieving this, for example KDE's virtual monitor implementation in Krfb: <a href="https://tldr.inbrowser.app/pages/linux/krfb-virtualmonitor" rel="noopener noreferrer">krfb-virtualmonitor | tldr InBrowser.App</a>.</p>

<p>A useful way to troubleshoot while working on this is to use OBS Studio to take a peek at the virtual output by adding a new source screen.</p>

<h2>
  
  
  Method #1: Sacrificing one of your existing outputs
</h2>

<p>Use Xrandr to check for free outputs on your system.</p>

<p>Follow <a href="https://www.azdanov.dev/articles/2025/how-to-create-a-virtual-display-for-sunshine-on-arch-linux" rel="noopener noreferrer">How to Create a Virtual Display for Sunshine on Arch Linux - Anton Ždanov</a> and stop before the Sunshine section:</p>

<ul>
<li>Regarding EDID files, you can add modes later using xrandr, so you don't need to choose one that matches your desired specifications.</li>
</ul>

<blockquote>
<h3>
  
  
  Step 1: Get an EDID File
</h3>

<p>An EDID (Extended Display Identification Data) file simulates a monitor. You can download pre-made EDID files from the <a href="https://git.linuxtv.org/v4l-utils.git/tree/utils/edid-decode/data" rel="noopener noreferrer">v4l-utils repository</a>. Choose one that matches your desired resolution and features (e.g., 4K, HDR).</p>
<h3>
  
  
  Step 2: Place the EDID File
</h3>

<ol>
<li>Create a directory for EDID files:</li>
</ol>


<pre class="highlight shell"><code><span class="nb">sudo mkdir</span> <span class="nt">-p</span> /usr/lib/firmware/edid
</code></pre>


<ol>
<li>Copy your chosen EDID file to this directory. For example:</li>
</ol>


<pre class="highlight shell"><code> <span class="nb">sudo cp </span>samsung-q800t-hdmi2.1 /usr/lib/firmware/edid/
</code></pre>

<h3>
  
  
  Step 3: Configure Kernel Parameters
</h3>

<ol>
<li>Identify your GPU's free HDMI or DP output:</li>
</ol>


<pre class="highlight shell"><code><span class="k">for </span>p <span class="k">in</span> /sys/class/drm/<span class="k">*</span>/status<span class="p">;</span> <span class="k">do </span><span class="nv">con</span><span class="o">=</span><span class="k">${</span><span class="nv">p</span><span class="p">%/status</span><span class="k">}</span><span class="p">;</span> <span class="nb">echo</span> <span class="nt">-n</span> <span class="s2">"</span><span class="k">${</span><span class="nv">con</span><span class="p">#*/card?-</span><span class="k">}</span><span class="s2">: "</span><span class="p">;</span> <span class="nb">cat</span> <span class="nv">$p</span><span class="p">;</span> <span class="k">done</span>
</code></pre>


<p>Replace <code>HDMI-A-1</code> in the next step with the appropriate output.</p>

<ol>
<li>Add the following kernel parameters (see <a href="https://wiki.archlinux.org/title/Kernel_parameters" rel="noopener noreferrer">arch wiki</a> or <a href="https://discovery.endeavouros.com/installation/systemd-boot/2022/12/" rel="noopener noreferrer">endeavour os discovery</a>):</li>
</ol>


<pre class="highlight conf"><code><span class="n">drm</span>.<span class="n">edid_firmware</span>=<span class="n">HDMI</span>-<span class="n">A</span>-<span class="m">1</span>:<span class="n">edid</span>/<span class="n">samsung</span>-<span class="n">q800t</span>-<span class="n">hdmi2</span>.<span class="m">1</span> <span class="n">video</span>=<span class="n">HDMI</span>-<span class="n">A</span>-<span class="m">1</span>:<span class="n">e</span>
</code></pre>


<ol>
<li>Update your mkinitcpio.conf file to include the EDID file:</li>
</ol>


<pre class="highlight shell"><code><span class="nv">FILES</span><span class="o">=(</span>/usr/lib/firmware/edid/samsung-q800t-hdmi2.1<span class="o">)</span>
</code></pre>


<ol>
<li>Regenerate the initramfs:</li>
</ol>


<pre class="highlight shell"><code><span class="nb">sudo </span>mkinitcpio <span class="nt">-P</span>
</code></pre>


<ol>
<li>Reboot your system.</li>
</ol>
</blockquote>

<p>You might afterwards need to enable the display in your Display Settings before it starts working.</p>

<h2>
  
  
  Method #2: Abusing the EVDI Linux driver <em>before</em> adding the output in kernel parameters
</h2>

<p>From <a href="https://github.com/DisplayLink/evdi" rel="noopener noreferrer">DisplayLink/evdi: Extensible Virtual Display Interface</a> on Github:</p>

<blockquote>
<p>The Extensible Virtual Display Interface (EVDI) is a Linux® kernel module that enables management of multiple screens, allowing user-space programs to take control over what happens with the image. It is essentially a virtual display you can add, remove and receive screen updates for, in an application that uses the <code>libevdi</code> library.</p>
</blockquote>

<ul>
<li>This driver is usually used for DisplayLink devices, which are basically external USB graphics cards used to obtain more display outputs from a system than it physically has, it even works for Macbooks. The cost being the software requirement and additional CPU overhead.</li>
</ul>

<p>In <a href="https://wiki.archlinux.org/title/Extreme_Multihead#Using_a_virtual_output" rel="noopener noreferrer">Using the EVDI driver - Extreme Multihead - ArchWiki</a>, it is suggested that this driver can be (ab)used to create virtual outputs without a DisplayLink dock. You will lose out on GPU acceleration that way, but the result is still usable for general tasks. The ArchWiki links to <a href="https://unix.stackexchange.com/questions/585069/unable-to-add-a-virtual-display-to-xorg" rel="noopener noreferrer">nvidia - Unable to add a VIRTUAL display to Xorg - Unix &amp; Linux Stack Exchange</a>, but the <a href="https://etam-software.eu/blog/2021-01-09-virtual-screen.html" rel="noopener noreferrer">Etam Software - Virtual screen</a> guide is clearer and lets you know that the current Xorg session will freeze during initial configuration. Although I've had luck deploying this solution for a long time on my machine, an update to EVDI suddenly now results in an unusable black screen, even tho this was supposedly patched in <a href="https://github.com/DisplayLink/evdi/issues/523" rel="noopener noreferrer">Monitor blank but detected as connected · Issue #523 · DisplayLink/evdi</a>, I was never able to get it working again just using EVDI.</p>

<p>Either way, the pure EVDI solution is less elegant the solution presented here, as it does not play nicely with Desktop Environment screen management (eg. having to disable the Screen Management background service KScreen2 in KDE Plasma using the Background Services GUI Program, rendering the Display Configuration page in KDE System Settings unusable.)</p>

<p>From <a href="https://etam-software.eu/blog/2021-01-09-virtual-screen.html" rel="noopener noreferrer">Etam Software - Virtual screen</a>:</p>

<ul>
<li>For Arch Linux the EVDI module is available in the AUR</li>
<li>You can get more than one extra output this way by changing the number for <code>initial_device_count</code>
</li>
</ul>

<blockquote>
<h3>
  
  
  Install evdi kernel module
</h3>


<pre class="highlight shell"><code>git clone https://github.com/DisplayLink/evdi.git
<span class="nb">cd </span>evdi/module

make <span class="o">&amp;&amp;</span> <span class="nb">sudo </span>make <span class="nb">install</span>
<span class="c"># or</span>
<span class="nb">sudo </span>make install_dkms
</code></pre>

<h3>
  
  
  Add <code>evdi</code> to <code>/etc/modules-load.d/local.conf</code>.
</h3>
<h3>
  
  
  Add <code>options evdi initial_device_count=1</code> to <code>/etc/modprobe.d/local.conf</code>.
</h3>

<p>WARNING! This next step will freeze your Xorg, so you might want to reboot instead or be clever about killing your Xorg or something.</p>


<pre class="highlight shell"><code><span class="nb">sudo </span>modprobe evdi
</code></pre>

</blockquote>

<p>After rebooting there should be new DVI outputs present in Xrandr</p>

<p>You will probably see that your new outputs don't have any modes listed in Xrandr, you should add those manually.</p>

<p>Now that you have extra outputs, you are able to use them to follow the steps in <strong>Method #1</strong>. </p>

<h2>
  
  
  Using your newly obtained virtual outputs
</h2>

<p>It is straightforward now to use VNC or other screen sharing solutions to get your virtual outputs on your desired devices, remember that by default VNC traffic is unencrypted.</p>

<p>If you're planning on using and Android device, consider using adb reversing for better latency and easier connection without having to set up the target IP:</p>

<p>From <a href="https://etam-software.eu/blog/2021-01-09-virtual-screen.html" rel="noopener noreferrer">Etam Software - Virtual screen</a>:</p>

<blockquote>
<p>4) (optional) Attach Android device through USB cable</p>


<pre class="highlight plaintext"><code>adb reverse tcp:5900 tcp:5900
</code></pre>


<p>Now you can point your VNC client application like <a href="https://f-droid.org/packages/com.coboltforge.dontmind.multivnc/" rel="noopener noreferrer">MultiVNC</a> to localhost:5900.</p>
</blockquote>

