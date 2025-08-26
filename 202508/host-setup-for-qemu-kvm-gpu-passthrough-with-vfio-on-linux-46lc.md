---
Title: Host Setup for Qemu KVM GPU Passthrough with VFIO on Linux
Description: 
Author: Dmitry Trifonov
Date: 2025-08-26T21:27:08.000Z
Robots: noindex,nofollow
Template: index
---
<h3>
  
  
  From “black magic” to reproducible results
</h3>

<p>GPU passthrough shouldn't feel like sorcery. If you've ever lost a weekend to half-working configs, random resets, or a guest that only boots when the moon is right, this guide is for you. I have pulled lots of hair while hardening the <a href="https://cloudrift.ai" rel="noopener noreferrer">CloudRift</a><br>
VM service for a variety of consumer (RTX 4090, 5090, PRO 6000) and data center (H100, B200) GPUs, so writing this guide to help you avoid common pitfalls.</p>

<p>I'll focus specifically on the host node configuration for GPU passthrough. Thus, this guide is relevant regardless of whether you're using Proxmox or plain libvirt/QEMU. The provided instructions have been tested on Ubuntu 22.04 and 24.04 with various NVIDIA GPUs.</p>

<p>To keep this guide manageable, I won't delve into lower-level details, such as specific domain XML tricks, Linux kernel builds, or GPU firmware flashing. In most cases, you don't need to fiddle with those.</p>
<h2>
  
  
  1. Remove NVIDIA drivers
</h2>

<p>The first step is to remove the NVIDIA drivers. It is not required, but NVIDIA drivers tend to cause issues with passthrough in one way or another, so it's better to remove them altogether.</p>

<blockquote>
<p>If you're configuring your own work PC with multiple GPUs, skip this step as without NVIDIA drivers you won't be able to run UI applications. In this case, the passthrough robustness is likely not a priority for you. However, I strongly recommend removing NVIDIA drivers on headless servers.</p>
</blockquote>

<p>If the NVIDIA driver is installed from the repository, you can remove it using the following commands:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight shell"><code><span class="nb">sudo </span>apt-get remove <span class="nt">--purge</span> <span class="s1">'^nvidia-.*'</span>
<span class="nb">sudo </span>apt autoremove
</code></pre>

</div>



<p>If you've installed the driver using the RUN file, remove it using:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight shell"><code><span class="nb">sudo</span> /usr/bin/nvidia-uninstall
</code></pre>

</div>



<p>Remove configs if any.<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight shell"><code><span class="nb">sudo rm</span> <span class="nt">-rf</span> /etc/X11/xorg.conf
<span class="nb">sudo rm</span> <span class="nt">-rf</span> /etc/modprobe.d/nvidia<span class="k">*</span>.conf
<span class="nb">sudo rm</span> <span class="nt">-rf</span> /lib/modprobe.d/nvidia<span class="k">*</span>.conf
</code></pre>

</div>



<p>Reboot the system after driver removal<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight shell"><code><span class="nb">sudo </span>reboot
</code></pre>

</div>



<h2>
  
  
  2. Check BIOS, IOMMU Support and IOMMU Group Assignment
</h2>

<p>The next step is to check virtualization and IOMMU support. We need to check four things:</p>

<ol>
<li>Virtualization is enabled (AMD-Vi / Intel VT-D options are enabled in bios). If present, enable "Above 4G decoding" and "Resizable BAR (ReBAR)" options in BIOS as well.</li>
<li>IOMMU is active (groups exist).</li>
<li>Each GPU and its audio function are isolated in their own IOMMU group.</li>
<li>GPU groups contain only GPU/video-audio functions and PCI bridges — no NICs, NVMe, SATA, etc.</li>
</ol>

<p><a href="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fr2boo48ogsvmbugducvy.png" class="article-body-image-wrapper"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fr2boo48ogsvmbugducvy.png" alt=" " width="797" height="594"></a></p>

<p>You can use the following handy-dandy script to check those preconditions.</p>

<blockquote>
<p>AI goes overboard when generating helper scripts, doesn't it? I can't complain, though. It provides a lot of useful information.<br>
</p>
</blockquote>

<div class="highlight js-code-highlight">
<pre class="highlight shell"><code><span class="c">#!/usr/bin/env bash</span>
<span class="c"># VFIO host sanity check: IOMMU support + GPU-containing groups</span>

<span class="nb">set</span> <span class="nt">-u</span>  <span class="c"># don't use -e so greps that find nothing don't abort</span>

<span class="c"># --- helpers ---------------------------------------------------------------</span>
have<span class="o">()</span> <span class="o">{</span> <span class="nb">command</span> <span class="nt">-v</span> <span class="s2">"</span><span class="nv">$1</span><span class="s2">"</span> <span class="o">&gt;</span>/dev/null 2&gt;&amp;1<span class="p">;</span> <span class="o">}</span>

read_klog<span class="o">()</span> <span class="o">{</span>
  <span class="k">if </span>have journalctl<span class="p">;</span> <span class="k">then </span>journalctl <span class="nt">-k</span> <span class="nt">-b</span> 0 2&gt;/dev/null
  <span class="k">else </span>dmesg 2&gt;/dev/null
  <span class="k">fi</span>
<span class="o">}</span>

trim<span class="o">()</span> <span class="o">{</span> <span class="nb">sed</span> <span class="nt">-e</span> <span class="s1">'s/^[[:space:]]*//'</span> <span class="nt">-e</span> <span class="s1">'s/[[:space:]]*$//'</span><span class="p">;</span> <span class="o">}</span>

<span class="c"># --- 1) CPU vendor + boot flags -------------------------------------------</span>
<span class="nv">CPU_VENDOR</span><span class="o">=</span><span class="s2">"</span><span class="si">$(</span>
  <span class="o">(</span>lscpu 2&gt;/dev/null | <span class="nb">awk</span> <span class="nt">-F</span>: <span class="s1">'/Vendor ID/{print $2}'</span> | trim<span class="o">)</span> <span class="o">||</span>
  <span class="o">(</span><span class="nb">grep</span> <span class="nt">-m1</span> <span class="s1">'vendor_id'</span> /proc/cpuinfo 2&gt;/dev/null | <span class="nb">awk</span> <span class="s1">'{print $3}'</span><span class="o">)</span>
<span class="si">)</span><span class="s2">"</span>
<span class="o">[</span> <span class="nt">-z</span> <span class="s2">"</span><span class="k">${</span><span class="nv">CPU_VENDOR</span><span class="k">}</span><span class="s2">"</span> <span class="o">]</span> <span class="o">&amp;&amp;</span> <span class="nv">CPU_VENDOR</span><span class="o">=</span><span class="s2">"(unknown)"</span>

<span class="nv">CMDLINE</span><span class="o">=</span><span class="s2">"</span><span class="si">$(</span><span class="nb">cat</span> /proc/cmdline 2&gt;/dev/null <span class="o">||</span> <span class="nb">echo</span> <span class="s1">''</span><span class="si">)</span><span class="s2">"</span>
<span class="nv">HAS_INTEL_FLAG</span><span class="o">=</span><span class="si">$(</span><span class="nb">echo</span> <span class="s2">"</span><span class="nv">$CMDLINE</span><span class="s2">"</span> | <span class="nb">grep</span> <span class="nt">-q</span> <span class="s1">'intel_iommu=on'</span> <span class="o">&amp;&amp;</span> <span class="nb">echo yes</span> <span class="o">||</span> <span class="nb">echo </span>no<span class="si">)</span>
<span class="nv">HAS_AMD_FLAG</span><span class="o">=</span><span class="si">$(</span><span class="nb">echo</span> <span class="s2">"</span><span class="nv">$CMDLINE</span><span class="s2">"</span> | <span class="nb">grep</span> <span class="nt">-q</span> <span class="s1">'amd_iommu=on'</span> <span class="o">&amp;&amp;</span> <span class="nb">echo yes</span> <span class="o">||</span> <span class="nb">echo </span>no<span class="si">)</span>
<span class="nv">HAS_PT_FLAG</span><span class="o">=</span><span class="si">$(</span><span class="nb">echo</span> <span class="s2">"</span><span class="nv">$CMDLINE</span><span class="s2">"</span> | <span class="nb">grep</span> <span class="nt">-q</span> <span class="s1">'iommu=pt'</span> <span class="o">&amp;&amp;</span> <span class="nb">echo yes</span> <span class="o">||</span> <span class="nb">echo </span>no<span class="si">)</span>

<span class="c"># --- 2) Kernel log signals ------------------------------------------------</span>
<span class="nv">KLOG</span><span class="o">=</span><span class="s2">"</span><span class="si">$(</span>read_klog<span class="si">)</span><span class="s2">"</span>

<span class="nv">DISABLED_MSG</span><span class="o">=</span><span class="si">$(</span><span class="nb">echo</span> <span class="s2">"</span><span class="nv">$KLOG</span><span class="s2">"</span> | egrep <span class="nt">-i</span> <span class="s1">'IOMMU.*disabled by BIOS|DMAR:.*disabled|AMD-Vi:.*disabled'</span> <span class="o">||</span> <span class="nb">true</span><span class="si">)</span>
<span class="nv">ENABLED_MSG</span><span class="o">=</span><span class="si">$(</span><span class="nb">echo</span> <span class="s2">"</span><span class="nv">$KLOG</span><span class="s2">"</span> | egrep <span class="nt">-i</span> <span class="s1">'DMAR: IOMMU enabled|AMD-Vi:.*IOMMU.*enabled|IOMMU: .*enabled'</span> <span class="o">||</span> <span class="nb">true</span><span class="si">)</span>
<span class="nv">IR_MSG</span><span class="o">=</span><span class="si">$(</span><span class="nb">echo</span> <span class="s2">"</span><span class="nv">$KLOG</span><span class="s2">"</span> | egrep <span class="nt">-i</span> <span class="s1">'Interrupt remapping enabled'</span> <span class="o">||</span> <span class="nb">true</span><span class="si">)</span>

<span class="c"># --- 3) IOMMU groups presence --------------------------------------------</span>
<span class="nv">GROUPS_DIR</span><span class="o">=</span><span class="s2">"/sys/kernel/iommu_groups"</span>
<span class="nv">GROUP_COUNT</span><span class="o">=</span>0
<span class="k">if</span> <span class="o">[</span> <span class="nt">-d</span> <span class="s2">"</span><span class="nv">$GROUPS_DIR</span><span class="s2">"</span> <span class="o">]</span><span class="p">;</span> <span class="k">then
  </span><span class="nv">GROUP_COUNT</span><span class="o">=</span><span class="si">$(</span>find <span class="s2">"</span><span class="nv">$GROUPS_DIR</span><span class="s2">"</span> <span class="nt">-mindepth</span> 1 <span class="nt">-maxdepth</span> 1 <span class="nt">-type</span> d 2&gt;/dev/null | <span class="nb">wc</span> <span class="nt">-l</span> | <span class="nb">awk</span> <span class="s1">'{print $1}'</span><span class="si">)</span>
<span class="k">fi</span>

<span class="c"># Heuristic: active if groups exist (&gt;0). Logs help explain state.</span>
<span class="nv">IOMMU_ACTIVE</span><span class="o">=</span><span class="s2">"no"</span>
<span class="o">[</span> <span class="s2">"</span><span class="nv">$GROUP_COUNT</span><span class="s2">"</span> <span class="nt">-gt</span> 0 <span class="o">]</span> <span class="o">&amp;&amp;</span> <span class="nv">IOMMU_ACTIVE</span><span class="o">=</span><span class="s2">"yes"</span>

<span class="c"># --- 4) Report summary ----------------------------------------------------</span>
<span class="nb">echo</span> <span class="s2">"=== IOMMU Summary ==="</span>
<span class="nb">echo</span> <span class="s2">"CPU vendor           : </span><span class="nv">$CPU_VENDOR</span><span class="s2">"</span>
<span class="nb">echo</span> <span class="s2">"Kernel cmdline       : </span><span class="nv">$CMDLINE</span><span class="s2">"</span>
<span class="nb">echo</span> <span class="s2">"Boot flags           : intel_iommu=</span><span class="nv">$HAS_INTEL_FLAG</span><span class="s2">  amd_iommu=</span><span class="nv">$HAS_AMD_FLAG</span><span class="s2">  iommu=pt=</span><span class="nv">$HAS_PT_FLAG</span><span class="s2">"</span>
<span class="nb">echo</span> <span class="s2">"Groups directory     : </span><span class="nv">$GROUPS_DIR</span><span class="s2">  (exists: </span><span class="si">$(</span><span class="o">[</span> <span class="nt">-d</span> <span class="s2">"</span><span class="nv">$GROUPS_DIR</span><span class="s2">"</span> <span class="o">]</span> <span class="o">&amp;&amp;</span> <span class="nb">echo yes</span> <span class="o">||</span> <span class="nb">echo </span>no<span class="si">)</span><span class="s2">)"</span>
<span class="nb">echo</span> <span class="s2">"IOMMU group count    : </span><span class="nv">$GROUP_COUNT</span><span class="s2">"</span>
<span class="nb">echo</span> <span class="s2">"Kernel says enabled  : </span><span class="si">$(</span><span class="o">[</span> <span class="nt">-n</span> <span class="s2">"</span><span class="nv">$ENABLED_MSG</span><span class="s2">"</span> <span class="o">]</span> <span class="o">&amp;&amp;</span> <span class="nb">echo yes</span> <span class="o">||</span> <span class="nb">echo </span>no<span class="si">)</span><span class="s2">"</span>
<span class="nb">echo</span> <span class="s2">"Interrupt remapping  : </span><span class="si">$(</span><span class="o">[</span> <span class="nt">-n</span> <span class="s2">"</span><span class="nv">$IR_MSG</span><span class="s2">"</span> <span class="o">]</span> <span class="o">&amp;&amp;</span> <span class="nb">echo yes</span> <span class="o">||</span> <span class="nb">echo </span>no<span class="si">)</span><span class="s2">"</span>
<span class="nb">echo</span> <span class="s2">"Kernel says disabled : </span><span class="si">$(</span><span class="o">[</span> <span class="nt">-n</span> <span class="s2">"</span><span class="nv">$DISABLED_MSG</span><span class="s2">"</span> <span class="o">]</span> <span class="o">&amp;&amp;</span> <span class="nb">echo yes</span> <span class="o">||</span> <span class="nb">echo </span>no<span class="si">)</span><span class="s2">"</span>
<span class="nb">echo</span> <span class="s2">"IOMMU ACTIVE?        : </span><span class="nv">$IOMMU_ACTIVE</span><span class="s2">"</span>
<span class="nb">echo

</span><span class="k">if</span> <span class="o">[</span> <span class="nt">-n</span> <span class="s2">"</span><span class="nv">$ENABLED_MSG</span><span class="s2">"</span> <span class="o">]</span><span class="p">;</span> <span class="k">then
  </span><span class="nb">echo</span> <span class="s2">"--- Kernel enable lines ---"</span>
  <span class="nb">echo</span> <span class="s2">"</span><span class="nv">$ENABLED_MSG</span><span class="s2">"</span>
  <span class="nb">echo
</span><span class="k">fi
if</span> <span class="o">[</span> <span class="nt">-n</span> <span class="s2">"</span><span class="nv">$DISABLED_MSG</span><span class="s2">"</span> <span class="o">]</span><span class="p">;</span> <span class="k">then
  </span><span class="nb">echo</span> <span class="s2">"--- Kernel disable lines ---"</span>
  <span class="nb">echo</span> <span class="s2">"</span><span class="nv">$DISABLED_MSG</span><span class="s2">"</span>
  <span class="nb">echo
</span><span class="k">fi</span>

<span class="c"># --- 5) Original: list only GPU-containing groups -------------------------</span>
<span class="nb">echo</span> <span class="s2">"=== GPU-Containing IOMMU Groups ==="</span>
<span class="k">if</span> <span class="o">[</span> <span class="o">!</span> <span class="nt">-d</span> <span class="s2">"</span><span class="nv">$GROUPS_DIR</span><span class="s2">"</span> <span class="o">]</span> <span class="o">||</span> <span class="o">[</span> <span class="s2">"</span><span class="nv">$GROUP_COUNT</span><span class="s2">"</span> <span class="nt">-eq</span> 0 <span class="o">]</span><span class="p">;</span> <span class="k">then
  </span><span class="nb">echo</span> <span class="s2">"(no IOMMU groups found)"</span>
<span class="k">else
  </span><span class="nb">declare</span> <span class="nt">-A</span> <span class="nv">GPU_COUNT_BY_GROUP</span><span class="o">=()</span>
  <span class="nv">group_warnings</span><span class="o">=()</span>

  <span class="k">for </span>g <span class="k">in</span> <span class="s2">"</span><span class="nv">$GROUPS_DIR</span><span class="s2">"</span>/<span class="k">*</span><span class="p">;</span> <span class="k">do</span>
    <span class="o">[</span> <span class="nt">-d</span> <span class="s2">"</span><span class="nv">$g</span><span class="s2">"</span> <span class="o">]</span> <span class="o">||</span> <span class="k">continue
    </span><span class="nv">group_num</span><span class="o">=</span><span class="si">$(</span><span class="nb">basename</span> <span class="s2">"</span><span class="nv">$g</span><span class="s2">"</span><span class="si">)</span>
    <span class="nv">gpu_found</span><span class="o">=</span><span class="nb">false
    </span><span class="nv">device_lines</span><span class="o">=</span><span class="s2">""</span>
    <span class="nv">non_gpu_non_bridge</span><span class="o">=</span><span class="nb">false
    </span><span class="nv">gpu_count_in_this_group</span><span class="o">=</span>0

    <span class="k">for </span>d <span class="k">in</span> <span class="s2">"</span><span class="nv">$g</span><span class="s2">"</span>/devices/<span class="k">*</span><span class="p">;</span> <span class="k">do</span>
      <span class="o">[</span> <span class="nt">-e</span> <span class="s2">"</span><span class="nv">$d</span><span class="s2">"</span> <span class="o">]</span> <span class="o">||</span> <span class="k">continue
      </span><span class="nv">pci_addr</span><span class="o">=</span><span class="si">$(</span><span class="nb">basename</span> <span class="s2">"</span><span class="nv">$d</span><span class="s2">"</span><span class="si">)</span>
      <span class="c"># -nns prints class code [XXXX] and vendor:device [vvvv:dddd]</span>
      <span class="nv">line</span><span class="o">=</span><span class="si">$(</span>lspci <span class="nt">-nns</span> <span class="s2">"</span><span class="nv">$pci_addr</span><span class="s2">"</span> 2&gt;/dev/null <span class="o">||</span> <span class="nb">echo</span> <span class="s2">"</span><span class="nv">$pci_addr</span><span class="s2"> (unlisted)"</span><span class="si">)</span>
      device_lines+<span class="o">=</span><span class="s2">"</span><span class="nv">$line</span><span class="s2">"</span><span class="s1">$'</span><span class="se">\n</span><span class="s1">'</span>

      <span class="c"># Extract first [...] which is the class code, e.g. 0300, 0302, 0403, 0604, 0600</span>
      <span class="nv">class_code</span><span class="o">=</span><span class="si">$(</span><span class="nb">echo</span> <span class="s2">"</span><span class="nv">$line</span><span class="s2">"</span> | <span class="nb">awk</span> <span class="nt">-F</span><span class="s1">'[][]'</span> <span class="s1">'{print $2}'</span><span class="si">)</span>

      <span class="c"># Detect GPUs / 3D controllers and their HDA audio functions</span>
      <span class="k">if </span><span class="nb">echo</span> <span class="s2">"</span><span class="nv">$line</span><span class="s2">"</span> | <span class="nb">grep</span> <span class="nt">-qE</span> <span class="s1">'VGA compatible controller|3D controller'</span><span class="p">;</span> <span class="k">then
        </span><span class="nv">gpu_found</span><span class="o">=</span><span class="nb">true
        </span><span class="nv">gpu_count_in_this_group</span><span class="o">=</span><span class="k">$((</span>gpu_count_in_this_group+1<span class="k">))</span>
      <span class="k">fi</span>

      <span class="c"># Allowlist: 0300(VGA), 0302(3D), 0403(HDA audio), 0600(host bridge), 0604(PCI bridge)</span>
      <span class="k">case</span> <span class="s2">"</span><span class="nv">$class_code</span><span class="s2">"</span> <span class="k">in
        </span>0300|0302|0403|0600|0604<span class="p">)</span> : <span class="p">;;</span>
        <span class="k">*</span><span class="p">)</span> <span class="nv">non_gpu_non_bridge</span><span class="o">=</span><span class="nb">true</span> <span class="p">;;</span>
      <span class="k">esac</span>
    <span class="k">done

    if</span> <span class="nv">$gpu_found</span><span class="p">;</span> <span class="k">then
      </span><span class="nb">echo</span> <span class="s2">"IOMMU Group </span><span class="nv">$group_num</span><span class="s2">:"</span>
      <span class="nb">echo</span> <span class="s2">"</span><span class="nv">$device_lines</span><span class="s2">"</span>

      <span class="c"># Track GPUs per group</span>
      GPU_COUNT_BY_GROUP[<span class="s2">"</span><span class="nv">$group_num</span><span class="s2">"</span><span class="o">]=</span><span class="nv">$gpu_count_in_this_group</span>

      <span class="c"># Warn if unexpected devices share the group with the GPU</span>
      <span class="k">if</span> <span class="nv">$non_gpu_non_bridge</span><span class="p">;</span> <span class="k">then
        </span>group_warnings+<span class="o">=(</span><span class="s2">"WARN: Group </span><span class="nv">$group_num</span><span class="s2"> contains non-GPU, non-audio, non-bridge devices (consider different slot/CPU root complex or ACS)."</span><span class="o">)</span>
      <span class="k">fi
    fi
  done</span>

  <span class="c"># Post-checks</span>
  <span class="c"># 1) Each GPU should be alone (one GPU per group)</span>
  <span class="nv">shared_groups</span><span class="o">=()</span>
  <span class="k">for </span>gnum <span class="k">in</span> <span class="s2">"</span><span class="k">${</span><span class="p">!GPU_COUNT_BY_GROUP[@]</span><span class="k">}</span><span class="s2">"</span><span class="p">;</span> <span class="k">do
    if</span> <span class="o">[</span> <span class="s2">"</span><span class="k">${</span><span class="nv">GPU_COUNT_BY_GROUP</span><span class="p">[</span><span class="nv">$gnum</span><span class="p">]</span><span class="k">}</span><span class="s2">"</span> <span class="nt">-gt</span> 1 <span class="o">]</span><span class="p">;</span> <span class="k">then
      </span>shared_groups+<span class="o">=(</span><span class="s2">"</span><span class="nv">$gnum</span><span class="s2">"</span><span class="o">)</span>
    <span class="k">fi
  done

  if</span> <span class="o">[</span> <span class="s2">"</span><span class="k">${#</span><span class="nv">shared_groups</span><span class="p">[@]</span><span class="k">}</span><span class="s2">"</span> <span class="nt">-gt</span> 0 <span class="o">]</span><span class="p">;</span> <span class="k">then
    </span><span class="nb">echo
    echo</span> <span class="s2">"WARN: Multiple GPUs share these IOMMU groups: </span><span class="k">${</span><span class="nv">shared_groups</span><span class="p">[*]</span><span class="k">}</span><span class="s2"> (prefer one GPU per group for VFIO)."</span>
  <span class="k">fi</span>

  <span class="c"># 2) Any non-bridge co-residents?</span>
  <span class="k">if</span> <span class="o">[</span> <span class="s2">"</span><span class="k">${#</span><span class="nv">group_warnings</span><span class="p">[@]</span><span class="k">}</span><span class="s2">"</span> <span class="nt">-gt</span> 0 <span class="o">]</span><span class="p">;</span> <span class="k">then
    </span><span class="nb">echo
    printf</span> <span class="s2">"%s</span><span class="se">\n</span><span class="s2">"</span> <span class="s2">"</span><span class="k">${</span><span class="nv">group_warnings</span><span class="p">[@]</span><span class="k">}</span><span class="s2">"</span>
  <span class="k">fi
fi</span>
</code></pre>

</div>



<p>Here is what a good summary should look like:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight shell"><code><span class="o">===</span> IOMMU Summary <span class="o">===</span>
CPU vendor           : AuthenticAMD
Kernel cmdline       : <span class="nv">BOOT_IMAGE</span><span class="o">=</span>/boot/vmlinuz-6.8.0-71-generic <span class="nv">root</span><span class="o">=</span>/dev/mapper/vgroot-lvroot ro systemd.unified_cgroup_hierarchy<span class="o">=</span><span class="nb">false </span><span class="nv">default_hugepagesz</span><span class="o">=</span>1G <span class="nv">hugepages</span><span class="o">=</span>576 <span class="nv">hugepagesz</span><span class="o">=</span>1G nomodeset <span class="nv">video</span><span class="o">=</span>efifb:off <span class="nv">iommu</span><span class="o">=</span>pt <span class="nv">pci</span><span class="o">=</span>realloc <span class="nv">pcie_aspm</span><span class="o">=</span>off <span class="nv">amd_iommu</span><span class="o">=</span>on vfio-pci.ids<span class="o">=</span>10de:0000,10de:204b,10de:22e8,10de:2bb1 modprobe.blacklist<span class="o">=</span>nouveau,nvidia,nvidiafb,snd_hda_intel
Boot flags           : <span class="nv">intel_iommu</span><span class="o">=</span>no  <span class="nv">amd_iommu</span><span class="o">=</span><span class="nb">yes  </span><span class="nv">iommu</span><span class="o">=</span><span class="nv">pt</span><span class="o">=</span><span class="nb">yes
</span>Groups directory     : /sys/kernel/iommu_groups  <span class="o">(</span>exists: <span class="nb">yes</span><span class="o">)</span>
IOMMU group count    : 57
Kernel says enabled  : no
Interrupt remapping  : no
Kernel says disabled : no
IOMMU ACTIVE?        : <span class="nb">yes</span>

<span class="o">===</span> GPU-Containing IOMMU Groups <span class="o">===</span>
IOMMU Group 13:
c1:00.0 VGA compatible controller <span class="o">[</span>0300]: NVIDIA Corporation Device <span class="o">[</span>10de:2bb1] <span class="o">(</span>rev a1<span class="o">)</span>
c1:00.1 Audio device <span class="o">[</span>0403]: NVIDIA Corporation Device <span class="o">[</span>10de:22e8] <span class="o">(</span>rev a1<span class="o">)</span>

IOMMU Group 16:
c6:00.0 PCI bridge <span class="o">[</span>0604]: ASPEED Technology, Inc. AST1150 PCI-to-PCI Bridge <span class="o">[</span>1a03:1150] <span class="o">(</span>rev 06<span class="o">)</span>
c7:00.0 VGA compatible controller <span class="o">[</span>0300]: ASPEED Technology, Inc. ASPEED Graphics Family <span class="o">[</span>1a03:2000] <span class="o">(</span>rev 52<span class="o">)</span>

IOMMU Group 27:
81:00.0 VGA compatible controller <span class="o">[</span>0300]: NVIDIA Corporation Device <span class="o">[</span>10de:2bb1] <span class="o">(</span>rev a1<span class="o">)</span>
81:00.1 Audio device <span class="o">[</span>0403]: NVIDIA Corporation Device <span class="o">[</span>10de:22e8] <span class="o">(</span>rev a1<span class="o">)</span>

IOMMU Group 42:
01:00.0 VGA compatible controller <span class="o">[</span>0300]: NVIDIA Corporation Device <span class="o">[</span>10de:2bb1] <span class="o">(</span>rev a1<span class="o">)</span>
01:00.1 Audio device <span class="o">[</span>0403]: NVIDIA Corporation Device <span class="o">[</span>10de:22e8] <span class="o">(</span>rev a1<span class="o">)</span>

IOMMU Group 54:
41:00.0 VGA compatible controller <span class="o">[</span>0300]: NVIDIA Corporation Device <span class="o">[</span>10de:2bb1] <span class="o">(</span>rev a1<span class="o">)</span>
41:00.1 Audio device <span class="o">[</span>0403]: NVIDIA Corporation Device <span class="o">[</span>10de:22e8] <span class="o">(</span>rev a1<span class="o">)</span>
</code></pre>

</div>



<p>As we can see, IOMMU support is enabled, and all GPUs and their corresponding audio devices are in separate IOMMU groups.</p>

<p>Sometimes you may see PCI bridges in the GPU IOMMU group. This is normal.<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight shell"><code><span class="o">===</span> GPU-Containing IOMMU Groups <span class="o">===</span>
IOMMU Group 13:
40:01.0 Host bridge <span class="o">[</span>0600]: Advanced Micro Devices, Inc. <span class="o">[</span>AMD] Starship/Matisse PCIe Dummy Host Bridge <span class="o">[</span>1022:1482]
40:01.1 PCI bridge <span class="o">[</span>0604]: Advanced Micro Devices, Inc. <span class="o">[</span>AMD] Starship/Matisse GPP Bridge <span class="o">[</span>1022:1483]
41:00.0 VGA compatible controller <span class="o">[</span>0300]: NVIDIA Corporation Device <span class="o">[</span>10de:2b85] <span class="o">(</span>rev a1<span class="o">)</span>
41:00.1 Audio device <span class="o">[</span>0403]: NVIDIA Corporation Device <span class="o">[</span>10de:22e8] <span class="o">(</span>rev a1<span class="o">)</span>

IOMMU Group 32:
20:03.0 Host bridge <span class="o">[</span>0600]: Advanced Micro Devices, Inc. <span class="o">[</span>AMD] Starship/Matisse PCIe Dummy Host Bridge <span class="o">[</span>1022:1482]
20:03.1 PCI bridge <span class="o">[</span>0604]: Advanced Micro Devices, Inc. <span class="o">[</span>AMD] Starship/Matisse GPP Bridge <span class="o">[</span>1022:1483]
25:00.0 VGA compatible controller <span class="o">[</span>0300]: NVIDIA Corporation Device <span class="o">[</span>10de:2b85] <span class="o">(</span>rev a1<span class="o">)</span>
25:00.1 Audio device <span class="o">[</span>0403]: NVIDIA Corporation Device <span class="o">[</span>10de:22e8] <span class="o">(</span>rev a1<span class="o">)</span>
</code></pre>

</div>



<h2>
  
  
  3. Leverage 1G Huge Pages
</h2>

<p>This step is optional. However, if you have more than 512GB of RAM on your system, it is highly-encouraged. From experience, aside from providing performance benefit, 1GB huge pages make the VM startup much more reliable on high-memory systems.</p>

<p><strong>Rule of thumb</strong></p>

<ul>
<li>
<strong>&lt; 128 GB RAM</strong>: usually skip (benefit is small).</li>
<li>
<strong>128–512 GB</strong>: optional; can reduce latency jitter.</li>
<li>
<strong>&gt; 512 GB</strong>: recommended for reliability and predictable performance.</li>
</ul>

<p><strong>Why 1 GiB pages help</strong></p>

<ul>
<li>Fewer page-table walks → fewer TLB misses.</li>
<li>Lower page management overhead.</li>
<li>More predictable VM start times on large RAM allocations.</li>
</ul>

<h3>
  
  
  3.1 Check Huge Page Support
</h3>

<p>To confirm 1G huge page support on your system, check the pdpe1gb CPU flag.<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight shell"><code><span class="nb">grep</span> <span class="nt">-m1</span> pdpe1gb /proc/cpuinfo <span class="o">&gt;</span>/dev/null <span class="o">&amp;&amp;</span> <span class="nb">echo</span> <span class="s2">"✓ CPU supports 1GiB pages"</span> <span class="o">||</span> <span class="nb">echo</span> <span class="s2">"✗ No 1GiB page support"</span>
</code></pre>

</div>



<h3>
  
  
  3.2 Allocate Huge Pages
</h3>

<p>Determine how much memory you want to reserve for the VMs. You need to reserve that much memory for huge pages plus a buffer.</p>

<blockquote>
<p>Note that the memory reserved for huge pages will not be usable on the host system.</p>
</blockquote>

<p>For example, if you want to dedicate <code>2000GB</code> to virtual machines with a <code>80 GB</code> buffer, you would need <code>2080</code> huge pages.</p>

<p>I use the following empirically validated table to determine the huge page configuration on a high-memory multi-GPU system.</p>

<div class="table-wrapper-paragraph"><table>
<thead>
<tr>
<th>Total System RAM</th>
<th>VM Allocation</th>
<th>Buffer</th>
<th>Huge Pages</th>
<th>Left for System</th>
</tr>
</thead>
<tbody>
<tr>
<td>768 GB</td>
<td>640 (8x80) GB</td>
<td>60 GB</td>
<td>700</td>
<td>68 GB</td>
</tr>
<tr>
<td>1024 GB</td>
<td>800 (8x100) GB</td>
<td>80 GB</td>
<td>880</td>
<td>144 GB</td>
</tr>
<tr>
<td>1256 GB</td>
<td>1040 (8x130) GB</td>
<td>100 GB</td>
<td>1140</td>
<td>116 GB</td>
</tr>
<tr>
<td>1512 GB</td>
<td>1280 (8x160) GB</td>
<td>120 GB</td>
<td>1300</td>
<td>212 GB</td>
</tr>
<tr>
<td>2048 GB</td>
<td>1760 (8x220) GB</td>
<td>160 GB</td>
<td>1920</td>
<td>128 GB</td>
</tr>
<tr>
<td>4096 GB</td>
<td>3680 (8*460) GB</td>
<td>200 GB</td>
<td>3880</td>
<td>216 GB</td>
</tr>
</tbody>
</table></div>

<blockquote>
<p>Is there a reliable formula to determine the huge page buffer size? Good question. If you know one, let me know in the comments. It makes sense that we need to leave some memory for the system, but it feels that the gap between the memory dedicated to VM allocation and the number of huge pages is unnecessary. After VM startup, we'll see that the system has allocated the exact number of requested huge pages, so why do we need a buffer, and how big should it be? Is it because of the fragmentation? Empirically, I've confirmed that it is needed. Without a buffer, I occasionally encountered OOM errors.</p>
</blockquote>

<p>Run the following command to allocate 2000 pages (it will take a while):<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight shell"><code><span class="nb">echo </span>2000 | <span class="nb">sudo tee</span> /sys/kernel/mm/hugepages/hugepages-1048576kB/nr_hugepages
</code></pre>

</div>



<p>To check that huge pages were allocated, run <code>grep -i huge /proc/meminfo</code>. Look at <code>Hugepagesize</code> and <code>Hugetlb</code> values. They tell the huge page size and the total amount of RAM allocated for huge pages. You should see output like this:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight shell"><code>AnonHugePages:     79872 kB
ShmemHugePages:        0 kB
FileHugePages:         0 kB
HugePages_Total:    2080
HugePages_Free:     1580
HugePages_Rsvd:        0
HugePages_Surp:        0
Hugepagesize:    1048576 kB
Hugetlb:        2181038080 kB
</code></pre>

</div>



<p>To deallocate, invoke:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight shell"><code><span class="nb">echo </span>0 | <span class="nb">sudo tee</span> /sys/kernel/mm/hugepages/hugepages-1048576kB/nr_hugepages
</code></pre>

</div>



<h3>
  
  
  3.3 Make Huge Pages Persistent
</h3>

<p>Edit the <code>/etc/default/grub</code> file and modify the line containing <code>GRUB_CMDLINE_LINUX</code>.</p>

<p>Add <code>default_hugepagesz=1G hugepagesz=1G hugepages=&lt;num&gt;</code> to the <code>GRUB_CMDLINE_LINUX</code> options. The <code>&lt;num&gt;</code> is the number of huge pages to allocate. For example:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight shell"><code><span class="nv">GRUB_CMDLINE_LINUX</span><span class="o">=</span><span class="s2">"... default_hugepagesz=1G hugepagesz=1G hugepages=200"</span>
</code></pre>

</div>



<blockquote>
<p><strong>Be careful. If you specify more huge pages than the system can allocate, the machine will not boot.</strong></p>
</blockquote>

<p>Update the GRUB changes, reboot, and verify that huge pages are allocated (or do this in the end).<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight shell"><code><span class="nb">sudo </span>update-grub
<span class="nb">sudo </span>reboot
</code></pre>

</div>



<h3>
  
  
  3.4 (Optional) Mount Huge Page Table
</h3>

<p>Many systems already have <code>/dev/hugepages</code>. If not, or if you want a dedicated mount:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight shell"><code><span class="nb">sudo mkdir</span> <span class="nt">-p</span> /mnt/hugepages-1G
<span class="nb">sudo </span>mount <span class="nt">-t</span> hugetlbfs <span class="nt">-o</span> <span class="nv">pagesize</span><span class="o">=</span>1G none /mnt/hugepages-1G
</code></pre>

</div>



<p>Check that the mount point is present by running <code>grep hugetlbfs /proc/mounts</code>.</p>

<p>You should see something like:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight shell"><code>hugetlbfs /dev/hugepages hugetlbfs rw,nosuid,nodev,relatime,pagesize<span class="o">=</span>1024M 0 0
hugetlbfs /mnt/hugepages-1G hugetlbfs rw,relatime,pagesize<span class="o">=</span>1024M 0 0
</code></pre>

</div>



<p>To persist - invoke:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight shell"><code><span class="nb">echo</span> <span class="s2">"none /mnt/hugepages-1G hugetlbfs pagesize=1G 0 0"</span> | <span class="nb">sudo tee</span> <span class="nt">-a</span> /etc/fstab
</code></pre>

</div>



<h3>
  
  
  3.5 Configure your Virtualization Software to use Huge Pages
</h3>

<p>Neither Proxmox nor libvirt is using huge pages by default.</p>

<p>To use them in libvirt, you need to add the following section to the domain XML<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight xml"><code><span class="nt">&lt;memoryBacking&gt;</span>
  <span class="nt">&lt;hugepages&gt;</span>
    <span class="nt">&lt;page</span> <span class="na">size=</span><span class="s">'1048576'</span> <span class="na">unit=</span><span class="s">'KiB'</span><span class="nt">/&gt;</span>
  <span class="nt">&lt;/hugepages&gt;</span>
  <span class="nt">&lt;locked/&gt;</span>
<span class="nt">&lt;/memoryBacking&gt;</span>
</code></pre>

</div>



<p>In Proxmox CLI you do it as follows:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight shell"><code>qm <span class="nb">set</span> &lt;vmid&gt; <span class="nt">--hugepages</span> 1024   <span class="c"># use 1GiB pages</span>
qm <span class="nb">set</span> &lt;vmid&gt; <span class="nt">--keephugepages</span> 1  <span class="c"># optional: keep reserved after shutdown</span>
</code></pre>

</div>



<h2>
  
  
  4. Bind to VFIO Early
</h2>

<p>For maximum stability, have VFIO claim the GPU at boot so no runtime driver swaps occur (Proxmox/libvirt will otherwise bind/unbind around VM start/stop).</p>

<h3>
  
  
  4.1 Identify the PCI IDs to bind
</h3>

<p>First, you need to determine the PCI vendor ID and device ID for your GPUs.</p>

<p>List all NVIDIA functions (display + audio, and any auxiliary functions):<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight shell"><code>lspci <span class="nt">-nn</span> | <span class="nb">grep</span> <span class="nt">-i</span> nvidia
</code></pre>

</div>



<p>Example (RTX 5090):<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight shell"><code>01:00.0 VGA compatible controller <span class="o">[</span>0300]: NVIDIA Corporation Device <span class="o">[</span>10de:2b85] <span class="o">(</span>rev a1<span class="o">)</span>
01:00.1 Audio device <span class="o">[</span>0403]: NVIDIA Corporation Device <span class="o">[</span>10de:22e8] <span class="o">(</span>rev a1<span class="o">)</span>
</code></pre>

</div>



<h3>
  
  
  4.2 Give VFIO first claim
</h3>

<p>Add the following lines to <code>GRUB_CMDLINE_LINUX_DEFAULT</code> in <code>/etc/default/grub</code>, replacing the PCI vendor ID and device ID with the appropriate values. Keep other options if needed.<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight plaintext"><code>GRUB_CMDLINE_LINUX_DEFAULT="modprobe.blacklist=nouveau,nvidia,nvidiafb,snd_hda_intel vfio-pci.ids=10de:2b85,10de:22e8 ..."
</code></pre>

</div>



<blockquote>
<p>Proxmox is likely using systemd-boot by default instead of GRUB. <a href="https://pve.proxmox.com/pve-docs/pve-admin-guide.html#sysboot" rel="noopener noreferrer">Check the bootloader</a> you're using and adjust the kernel command line accordingly.</p>

<p>Many online manuals suggest adding VFIO modules to <code>/etc/modprobe.d/vfio.conf</code>, but this approach has not always worked for me. I recommend early binding via the kernel command line.</p>
</blockquote>

<h3>
  
  
  4.3 Ensure VFIO is in the initramfs
</h3>

<p>We need to make sure that vfio modules are loaded early in the boot process. To achieve this, we include them in the <code>initramfs</code><br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight shell"><code><span class="nb">sudo tee</span> <span class="nt">-a</span> /etc/initramfs-tools/modules <span class="o">&gt;</span>/dev/null <span class="o">&lt;&lt;</span><span class="sh">'</span><span class="no">EOF</span><span class="sh">'
vfio
vfio_iommu_type1
vfio_pci
vfio_virqfd
</span><span class="no">EOF
</span></code></pre>

</div>



<h3>
  
  
  4.4 Reboot and verify
</h3>

<p>Update grub, initramfs and reboot.<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight shell"><code><span class="nb">sudo </span>update-initramfs <span class="nt">-u</span> <span class="nt">-k</span> all
<span class="nb">sudo </span>update-grub
<span class="nb">sudo </span>reboot
</code></pre>

</div>



<p>After reboot check that VFIO drivers are in use. You can use <code>lspci -k | grep -A 2 -i nvidia</code> command<br>
and should see <code>vfio</code> drivers in use:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight shell"><code>81:00.0 VGA compatible controller: NVIDIA Corporation Device 2b85 <span class="o">(</span>rev a1<span class="o">)</span>
    Subsystem: Gigabyte Technology Co., Ltd Device 416f
    Kernel driver <span class="k">in </span>use: vfio-pci
    Kernel modules: nvidiafb, nouveau
81:00.1 Audio device: NVIDIA Corporation Device 22e8 <span class="o">(</span>rev a1<span class="o">)</span>
    Subsystem: NVIDIA Corporation Device 0000
    Kernel driver <span class="k">in </span>use: vfio-pci
    Kernel modules: snd_hda_intel
</code></pre>

</div>



<blockquote>
<p>To be fair, there was one machine where this technique to bind VFIO failed. The system was aggressively binding<br>
  snd_hda_intel driver to the GPU audio function. However, this method worked for me in all other cases.</p>
</blockquote>

<h2>
  
  
  5. Other GRUB Options
</h2>

<p>Here is a summary of other kernel command line options that you may want to consider, along with my thoughts on each.</p>

<ul>
<li>
<code>pci=realloc</code>: Reallocate PCI resources forces the kernel to reassign PCI bus resources (MMIO/IOBARs) from scratch, ignoring what the firmware/BIOS assigned. It helps avoid issues when the BIOS didn't allocate enough space for devices (common with large GPUs or multiple devices). Fixes “BAR can't be assigned” or “resource busy” errors. <em>This option is helpful. I like to include it into the guest OS kernel params as well. It occasionally helps to work around BAR allocation issues. However, there is no need to list it unless the system has PCI device enumeration issues.</em>
</li>
<li>
<code>iommu=pt</code>: IOMMU passthrough mode tells the kernel to enable the IOMMU but use pass-through mode for DMA mappings by default. For VFIO GPU passthrough — allows the device to access physical memory directly with minimal performance penalty. <em>I haven't had a chance to test the performance gains, so I can just say that this option didn't break anything.</em>
</li>
<li>
<code>pcie_aspm=off</code>: Disable PCIe Active State Power Management, which is a power-saving feature that reduces PCIe link power in idle states. Some PCIe devices (especially GPUs) have trouble retraining links or waking from ASPM low-power states, leading to hangs or device inaccessible errors. This option was introduced to my configs after losing a lot of time on the <a href="https://dev.to/blog/bug-bounty-nvidia-reset-bug">Reset Bug</a>. <em>It didn't help. I don't consider this option helpful at the moment, but I am still evaluating it.</em>
</li>
<li>
<code>nomodeset</code>: Disable kernel mode setting (KMS) for all GPUs; prevents DRM drivers from taking over the console. <strong>This option is intended for use with headless servers only. It can break desktop/console output. I typically use it since we're working with headless servers.</strong>
</li>
<li>
<code>video=efifb:off</code>: Disables the firmware EFI framebuffer so simpledrm/efifb won’t grab the boot GPU before VFIO claims it. <em>This option is outdated and has no effect on systems with modern kernels. I list it for completeness.</em>
</li>
<li>
<code>intel_iommu=on</code> / <code>amd_iommu=on</code>: Enable IOMMU support for Intel and AMD. <em>These are enabled by default, so there is no need to add them to kernel parameters</em>.</li>
</ul>

<p>Here is how the typical kernel command line should look on a headless server with over 500GB of RAM.<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight shell"><code>nomodeset
modprobe.blacklist<span class="o">=</span>nouveau,nvidia,nvidiafb,snd_hda_intel
vfio-pci.ids<span class="o">=</span>10de:2b85,10de:22e8
<span class="nv">default_hugepagesz</span><span class="o">=</span>1G <span class="nv">hugepagesz</span><span class="o">=</span>1G <span class="nv">hugepages</span><span class="o">=</span>400
</code></pre>

</div>



<h2>
  
  
  Conclusion
</h2>

<p>The VFIO GPU passthrough is a finicky process. It is sensitive to host hardware and software configuration. However, with enough diligence, you can make it robust and reliable. <strong>I strongly believe in this approach and rely on VFIO GPU passthrough as the primary tool for our GPU rental service at <a href="https://cloudrift.ai/" rel="noopener noreferrer">cloudrift.ai</a></strong>.</p>

<p>I hope this guide helped you to improve your homelab or data center setup. If you notice any inaccuracies or have suggestions, please don't hesitate to let me know so we can improve the workflow together.</p>

<p>Final host checklist:</p>

<ul>
<li>Enable IOMMU, Above 4G, and (where applicable) ReBAR in the BIOS.</li>
<li>Verify clean IOMMU groups; each GPU (+ audio) isolated.</li>
<li>Bind to vfio-pci early.</li>
<li>Size huge pages (1 GiB on high-RAM hosts) and confirm in /proc/meminfo.</li>
<li>Configure other kernel command-line options as needed.</li>
</ul>

