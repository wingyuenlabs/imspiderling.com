---
Title: Low-Noise EC2 Benchmarking: A Practical Guide
Description: 
Author: Kien Do
Date: 2026-01-26T21:35:18.000Z
Robots: noindex,nofollow
Template: index
---
<h2>
  
  
  Introduction
</h2>

<p>Running reliable performance benchmarks in the cloud is notoriously difficult. EC2 instances have inherent performance variability—CPU frequency scaling, power-saving states, NUMA balancing, swap activity, and shared I/O—that can mask real code regressions or trigger false alarms. When your benchmark shows a 15% regression, is it your code or just EC2 having a bad day?</p>

<p>I initially encountered this issue while benchmarking Rust production code with the <a href="https://bheisler.github.io/criterion.rs/book/criterion_rs.html" rel="noopener noreferrer">Criterion</a> crate, and observing wildly varied wall-clock results between runs. As it turns out, this is expected behaviour and has been documented in Criterion's <a href="https://bheisler.github.io/criterion.rs/book/faq.html#how-should-i-run-criterionrs-benchmarks-in-a-ci-pipeline" rel="noopener noreferrer">FAQ - How Should I Run Criterion.rs Benchmarks In A CI Pipeline?</a> (short answer: don't). The problem isn't Rust-specific, it's the underlying hardware that affects all code benchmarks.</p>

<p>After implementing optimizations based on MongoDB's extensive research into cloud performance testing, I reduced benchmark variance from 20–30% down to under 5%. This guide provides the practical scripts and step-by-step instructions to do the same for your CI benchmarking infrastructure.</p>

<p><strong>What you'll learn:</strong></p>

<ol>
<li>How to configure an EC2 instance for low-noise performance testing</li>
<li>How to register the instance as a GitHub Actions self-hosted runner</li>
<li>How to create a CI workflow for automated benchmark regression detection (with Rust Criterion examples)</li>
</ol>

<h3>
  
  
  Why Dedicated EC2?
</h3>

<p>With dynamically provisioned shared runners (e.g. GitHub-hosted runners), you have no control over CPU frequency scaling, other workloads on the same physical host, memory pressure, or I/O contention. With a dedicated EC2 that you manage yourself, you control the configuration.</p>

<p>MongoDB's performance team spent years researching this problem. Their key insight: <strong>prioritize repeatability over peak performance</strong>. A benchmark that's consistently 10% slower but stable is far more useful than one that's sometimes fast but varies by 30%.</p>

<h3>
  
  
  Why not bare metal?
</h3>

<p>Yes, bare metal would be ideal, as there is no hypervisor, no noisy neighbours, and predictable performance. But when I asked the DevOps team about provisioning physical machines, the answer was an immediate "no". Too expensive, too difficult to manage and secure, and nobody does that anymore. Cloud VMs are the reality now. Shared, ephemeral resources are how modern infrastructure works. So rather than fighting this trend, the goal becomes: how do we make cloud instances behave <em>as close to bare metal as possible</em> for the duration of our benchmarks?</p>

<p>That's what this guide addresses.</p>

<h3>
  
  
  MongoDB Research References
</h3>

<ul>
<li><a href="https://www.mongodb.com/resources/products/capabilities/reducing-variability-performance-tests-ec2-setup-key-results" rel="noopener noreferrer">Reducing Variability in Performance Tests on EC2: Setup and Key Results</a></li>
<li><a href="https://www.mongodb.com/resources/products/capabilities/repeatable-performance-tests-ec2-instances-neither-good-nor-bad" rel="noopener noreferrer">Repeatable Performance Tests: EC2 Instances are Neither Good Nor Bad</a></li>
<li><a href="https://www.mongodb.com/company/blog/repeadtable-performance-tests-ebs-instances-stable-option" rel="noopener noreferrer">Repeatable Performance Tests: EBS Instances are the Stable Option</a></li>
<li><a href="https://www.mongodb.com/resources/products/capabilities/repeatable-performance-tests-cpu-options-best-disabled" rel="noopener noreferrer">Repeatable Performance Tests: CPU Options Are Best Disabled</a></li>
</ul>




<h2>
  
  
  Step 1: Create an EC2 Instance
</h2>

<p>Launch an EC2 instance with the following specifications:</p>

<div class="table-wrapper-paragraph"><table>
<thead>
<tr>
<th>Component</th>
<th>Recommended</th>
<th>Alternative</th>
</tr>
</thead>
<tbody>
<tr>
<td><strong>Instance Type</strong></td>
<td>
<code>c6i.8xlarge</code> (Nitro)</td>
<td>
<code>c3.8xlarge</code> (Xen)</td>
</tr>
<tr>
<td><strong>vCPUs / RAM</strong></td>
<td>32 cores, 64 GiB</td>
<td>32 cores, 60 GiB</td>
</tr>
<tr>
<td><strong>OS</strong></td>
<td>Ubuntu 24.04 LTS</td>
<td>Ubuntu 22.04 LTS</td>
</tr>
<tr>
<td><strong>Root Volume</strong></td>
<td>30 GiB gp3, 3000 IOPS</td>
<td>—</td>
</tr>
<tr>
<td><strong>EBS Data Volume</strong></td>
<td>100+ GiB gp3, 4000 IOPS</td>
<td>100+ GiB io2, 5000+ IOPS</td>
</tr>
</tbody>
</table></div>

<p><strong>Why these specs?</strong></p>

<ul>
<li>
<strong>32 vCPUs:</strong> Enables disabling hyperthreading (32 → 16 physical cores), eliminating interference between logical cores sharing execution units.</li>
<li>
<strong>Dedicated EBS:</strong> MongoDB's research showed EBS with provisioned IOPS dramatically outperforms ephemeral SSDs by eliminating shared I/O contention. This is counterintuitive but well-documented.</li>
<li>
<strong>c3.8xlarge advantage:</strong> Runs at a fixed optimal frequency automatically—less configuration needed.</li>
</ul>

<p><strong>EBS cost comparison (100 GiB):</strong></p>

<div class="table-wrapper-paragraph"><table>
<thead>
<tr>
<th>Option</th>
<th>Type</th>
<th>IOPS</th>
<th>Monthly Cost</th>
</tr>
</thead>
<tbody>
<tr>
<td>Recommended</td>
<td>gp3</td>
<td>4000</td>
<td>~$38 USD</td>
</tr>
<tr>
<td>MongoDB exact</td>
<td>io2</td>
<td>5000+</td>
<td>~$338 USD</td>
</tr>
</tbody>
</table></div>

<p>For most workloads, gp3 provides 90% of the benefit at 11% of the cost.</p>




<h2>
  
  
  Step 2: Run the Setup Scripts
</h2>

<p>Create a folder called <code>ec2-setup</code> and add the scripts from the Scripts section. Then:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight shell"><code><span class="c"># SSH into your instance</span>
ssh ubuntu@&lt;instance-ip&gt;

<span class="c"># Make scripts executable</span>
<span class="nb">chmod</span> +x ./ec2-setup/<span class="k">*</span>.sh

<span class="c"># Identify your EBS data volume</span>
lsblk
<span class="c"># Nitro instances: /dev/nvme1n1 or /dev/nvme2n1</span>
<span class="c"># Xen instances: /dev/xvdb, /dev/xvdc</span>

<span class="c"># Run setup (adjust device path as needed)</span>
<span class="nb">sudo</span> ./ec2-setup/setup-ec2-instance.sh /dev/nvme1n1
</code></pre>

</div>






<h2>
  
  
  Step 3: Reboot and Apply CPU Optimizations
</h2>



<div class="highlight js-code-highlight">
<pre class="highlight shell"><code><span class="c"># Reboot to apply kernel parameters</span>
<span class="nb">sudo </span>reboot

<span class="c"># After reboot, apply CPU optimizations</span>
<span class="nb">sudo</span> /usr/local/bin/disable-hyperthreading.sh
<span class="nb">sudo</span> /usr/local/bin/set-cpu-frequency.sh

<span class="c"># Verify the system is ready</span>
run-canary-tests
benchmark-status
</code></pre>

</div>






<h2>
  
  
  Step 4: Register as a GitHub Actions Runner
</h2>

<ol>
<li><p>Navigate to <strong>Repository → Settings → Actions → Runners → New self-hosted runner</strong></p></li>
<li><p>Download and configure the runner:<br>
</p></li>
</ol>

<div class="highlight js-code-highlight">
<pre class="highlight shell"><code><span class="nb">mkdir </span>actions-runner <span class="o">&amp;&amp;</span> <span class="nb">cd </span>actions-runner

<span class="c"># Download (check GitHub for current version)</span>
curl <span class="nt">-o</span> actions-runner-linux-x64-2.311.0.tar.gz <span class="nt">-L</span> <span class="se">\</span>
    https://github.com/actions/runner/releases/download/v2.311.0/actions-runner-linux-x64-2.311.0.tar.gz
<span class="nb">tar </span>xzf ./actions-runner-linux-x64-2.311.0.tar.gz

<span class="c"># Configure with your repository token</span>
./config.sh <span class="nt">--url</span> https://github.com/YOUR_ORG/YOUR_REPO <span class="nt">--token</span> YOUR_TOKEN
</code></pre>

</div>



<ol>
<li><p>When prompted, add a descriptive label like <code>benchmark-runner</code></p></li>
<li><p>Install and start as a service:<br>
</p></li>
</ol>

<div class="highlight js-code-highlight">
<pre class="highlight shell"><code><span class="nb">sudo</span> ./svc.sh <span class="nb">install
sudo</span> ./svc.sh start
</code></pre>

</div>



<p>For complete documentation, see <a href="https://docs.github.com/en/actions/hosting-your-own-runners/managing-self-hosted-runners/adding-self-hosted-runners" rel="noopener noreferrer">GitHub Docs: Adding self-hosted runners</a>.</p>

<p><strong>Corporate environments:</strong> If you have GitHub Enterprise, proxies, or firewalls, consult your DevOps team for additional configuration.</p>




<h2>
  
  
  Step 5: Verify Your Setup
</h2>

<p>Run through this checklist:</p>

<ul>
<li>[ ] CPU governor shows <code>performance</code> (or N/A on Xen)</li>
<li>[ ] Hyperthreading disabled (16 active cores from 32)</li>
<li>[ ] Swap completely disabled (<code>free -h</code> shows 0 swap)</li>
<li>[ ] NUMA balancing disabled</li>
<li>[ ] EBS storage mounted at <code>/opt/benchmark-data</code>
</li>
<li>[ ] Canary tests passing consistently</li>
<li>[ ] GitHub runner showing online in repository settings
</li>
</ul>

<div class="highlight js-code-highlight">
<pre class="highlight shell"><code><span class="c"># Quick verification commands</span>
<span class="nb">cat</span> /sys/devices/system/cpu/cpu0/cpufreq/scaling_governor
<span class="nb">nproc</span>  <span class="c"># Should show 16</span>
free <span class="nt">-h</span>
<span class="nb">cat</span> /proc/sys/kernel/numa_balancing  <span class="c"># Should show 0</span>
<span class="nb">df</span> <span class="nt">-h</span> /opt/benchmark-data
</code></pre>

</div>



<p>Your EC2 instance is now configured as a GitHub Actions self-hosted runner! You can create and run automated benchmarks on it as part of your CI workflow.</p>

<p>The next section provides an example workflow using Rust's <a href="https://bheisler.github.io/criterion.rs/book/criterion_rs.html" rel="noopener noreferrer">Criterion</a> crate, but the same approach works for any language's benchmarking framework.</p>




<h2>
  
  
  Example CI Workflow - Rust Criterion Benchmarks
</h2>

<p>This workflow implements a <strong>baseline comparison strategy</strong>: run benchmarks on main, then on your feature branch, and compare. This provides meaningful regression detection in virtualized environments where absolute timing values are unreliable. The example uses Criterion, but the pattern applies to any benchmarking tool.</p>

<p>Create <code>.github/workflows/benchmark.yaml</code>:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight yaml"><code><span class="na">name</span><span class="pi">:</span> <span class="s">Performance Benchmarks</span>

<span class="na">on</span><span class="pi">:</span>
  <span class="na">push</span><span class="pi">:</span>
    <span class="na">branches</span><span class="pi">:</span> <span class="pi">[</span><span class="nv">main</span><span class="pi">]</span>
  <span class="na">pull_request</span><span class="pi">:</span>
    <span class="na">types</span><span class="pi">:</span> <span class="pi">[</span><span class="nv">labeled</span><span class="pi">]</span>

<span class="na">permissions</span><span class="pi">:</span>
  <span class="na">contents</span><span class="pi">:</span> <span class="s">read</span>
  <span class="na">pull-requests</span><span class="pi">:</span> <span class="s">read</span>

<span class="na">jobs</span><span class="pi">:</span>
  <span class="na">benchmark</span><span class="pi">:</span>
    <span class="na">name</span><span class="pi">:</span> <span class="s">Run Criterion Benchmarks</span>
    <span class="na">if</span><span class="pi">:</span> <span class="pi">&gt;</span>
      <span class="s">github.event_name == 'push' ||</span>
      <span class="s">github.event.label.name == 'run-benchmark' ||</span>
      <span class="s">github.event.label.name == 'full-benchmark'</span>
    <span class="na">runs-on</span><span class="pi">:</span> <span class="pi">[</span><span class="nv">self-hosted</span><span class="pi">,</span> <span class="nv">Linux</span><span class="pi">,</span> <span class="nv">X64</span><span class="pi">,</span> <span class="nv">benchmark-runner</span><span class="pi">]</span>

    <span class="na">steps</span><span class="pi">:</span>
      <span class="pi">-</span> <span class="na">name</span><span class="pi">:</span> <span class="s">Checkout code</span>
        <span class="na">uses</span><span class="pi">:</span> <span class="s">actions/checkout@v4</span>
        <span class="na">with</span><span class="pi">:</span>
          <span class="na">fetch-depth</span><span class="pi">:</span> <span class="m">0</span>

      <span class="pi">-</span> <span class="na">name</span><span class="pi">:</span> <span class="s">Install Rust toolchain</span>
        <span class="na">run</span><span class="pi">:</span> <span class="pi">|</span>
          <span class="s">curl --proto '=https' --tlsv1.2 -sSf https://sh.rustup.rs | sh -s -- -y</span>
          <span class="s">echo "$HOME/.cargo/bin" &gt;&gt; $GITHUB_PATH</span>

      <span class="pi">-</span> <span class="na">name</span><span class="pi">:</span> <span class="s">Check benchmark scope</span>
        <span class="na">id</span><span class="pi">:</span> <span class="s">check-labels</span>
        <span class="na">run</span><span class="pi">:</span> <span class="pi">|</span>
          <span class="s">if [ "${{ github.event.label.name }}" = "full-benchmark" ]; then</span>
            <span class="s">echo "feature_flag=--features full-benchmark" &gt;&gt; $GITHUB_OUTPUT</span>
          <span class="s">else</span>
            <span class="s">echo "feature_flag=" &gt;&gt; $GITHUB_OUTPUT</span>
          <span class="s">fi</span>

      <span class="pi">-</span> <span class="na">name</span><span class="pi">:</span> <span class="s">Run benchmarks with baseline comparison</span>
        <span class="na">run</span><span class="pi">:</span> <span class="pi">|</span>
          <span class="s">cd src/your-project  # Adjust to your project path</span>
          <span class="s">mkdir -p benchmark_results</span>

          <span class="s"># Save current commit</span>
          <span class="s">CURRENT_COMMIT=$(git rev-parse HEAD)</span>

          <span class="s"># Run baseline on main</span>
          <span class="s">git fetch origin main:refs/remotes/origin/main --force</span>
          <span class="s">git checkout origin/main</span>
          <span class="s">cargo bench ${{ steps.check-labels.outputs.feature_flag }} -- --verbose \</span>
            <span class="s">&gt; benchmark_results/baseline.txt 2&gt;&amp;1</span>

          <span class="s"># Run on feature branch</span>
          <span class="s">git checkout "$CURRENT_COMMIT"</span>
          <span class="s">cargo bench ${{ steps.check-labels.outputs.feature_flag }} -- --verbose \</span>
            <span class="s">&gt; benchmark_results/feature.txt 2&gt;&amp;1</span>

          <span class="s"># Check for regressions</span>
          <span class="s">if grep -q "Performance has regressed" benchmark_results/feature.txt; then</span>
            <span class="s">echo "::error::Performance regression detected!"</span>
            <span class="s">grep -A5 "Performance has regressed" benchmark_results/feature.txt</span>
            <span class="s">exit 1</span>
          <span class="s">fi</span>
          <span class="s">echo "No regressions detected."</span>

      <span class="pi">-</span> <span class="na">name</span><span class="pi">:</span> <span class="s">Upload results</span>
        <span class="na">if</span><span class="pi">:</span> <span class="s">always()</span>
        <span class="na">uses</span><span class="pi">:</span> <span class="s">actions/upload-artifact@v4</span>
        <span class="na">with</span><span class="pi">:</span>
          <span class="na">name</span><span class="pi">:</span> <span class="s">benchmark-results</span>
          <span class="na">path</span><span class="pi">:</span> <span class="s">src/your-project/benchmark_results/</span>
          <span class="na">retention-days</span><span class="pi">:</span> <span class="m">30</span>

      <span class="pi">-</span> <span class="na">name</span><span class="pi">:</span> <span class="s">Cleanup</span>
        <span class="na">if</span><span class="pi">:</span> <span class="s">always()</span>
        <span class="na">run</span><span class="pi">:</span> <span class="pi">|</span>
          <span class="s">cd src/your-project</span>
          <span class="s">cargo clean || true</span>
          <span class="s">rm -rf benchmark_results || true</span>
</code></pre>

</div>



<h3>
  
  
  Controlling Benchmark Scope with Labels
</h3>

<div class="table-wrapper-paragraph"><table>
<thead>
<tr>
<th>Label</th>
<th>Effect</th>
</tr>
</thead>
<tbody>
<tr>
<td><code>run-benchmark</code></td>
<td>Run standard benchmarks</td>
</tr>
<tr>
<td><code>full-benchmark</code></td>
<td>Run extended benchmarks with larger datasets</td>
</tr>
</tbody>
</table></div>

<p>In your <code>Cargo.toml</code>:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight toml"><code><span class="nn">[features]</span>
<span class="py">full-benchmark</span> <span class="p">=</span> <span class="p">[]</span>
</code></pre>

</div>



<p>In your benchmark code:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight rust"><code><span class="nd">#[cfg(not(feature</span> <span class="nd">=</span> <span class="s">"full-benchmark"</span><span class="nd">))]</span>
<span class="k">const</span> <span class="n">TEST_SIZES</span><span class="p">:</span> <span class="o">&amp;</span><span class="p">[</span><span class="nb">usize</span><span class="p">]</span> <span class="o">=</span> <span class="o">&amp;</span><span class="p">[</span><span class="mi">100</span><span class="p">,</span> <span class="mi">1_000</span><span class="p">];</span>

<span class="nd">#[cfg(feature</span> <span class="nd">=</span> <span class="s">"full-benchmark"</span><span class="nd">)]</span>
<span class="k">const</span> <span class="n">TEST_SIZES</span><span class="p">:</span> <span class="o">&amp;</span><span class="p">[</span><span class="nb">usize</span><span class="p">]</span> <span class="o">=</span> <span class="o">&amp;</span><span class="p">[</span><span class="mi">100</span><span class="p">,</span> <span class="mi">1_000</span><span class="p">,</span> <span class="mi">10_000</span><span class="p">,</span> <span class="mi">100_000</span><span class="p">];</span>
</code></pre>

</div>






<h2>
  
  
  Running Benchmarks Manually
</h2>



<div class="highlight js-code-highlight">
<pre class="highlight shell"><code><span class="c"># Set target directory to EBS for faster builds</span>
<span class="nb">export </span><span class="nv">CARGO_TARGET_DIR</span><span class="o">=</span>/opt/benchmark-data/workdir

<span class="c"># Pin to specific cores for isolation</span>
taskset <span class="nt">-c</span> 0-3 cargo bench

<span class="c"># With NUMA binding (multi-socket systems)</span>
numactl <span class="nt">--cpunodebind</span><span class="o">=</span>0 <span class="nt">--membind</span><span class="o">=</span>0 cargo bench

<span class="c"># Full benchmark suite</span>
cargo bench <span class="nt">--features</span> full-benchmark
</code></pre>

</div>






<h2>
  
  
  What the Scripts Do
</h2>

<h3>
  
  
  CPU Optimization (<code>01-system-tuning.sh</code>)
</h3>

<p><strong>Goal:</strong> Eliminate frequency scaling and power-saving states.</p>

<ul>
<li>
<strong>Performance governor:</strong> Locks CPU to maximum frequency</li>
<li>
<strong>Disable C-states:</strong> Prevents power-saving delays via kernel parameters: <code>processor.max_cstate=1 intel_idle.max_cstate=1 intel_pstate=disable idle=poll</code>
</li>
<li>
<strong>Disable hyperthreading:</strong> Two logical cores on one physical core share execution units, caches, and branch predictors. Disabling hyperthreading eliminates this interference.</li>
</ul>

<h3>
  
  
  Memory Optimization
</h3>

<p><strong>Goal:</strong> Eliminate swap and NUMA balancing.</p>

<ul>
<li>
<strong>Disable swap:</strong> Even with plenty of RAM, Linux may swap pages. A 10ms disk access in your hot loop destroys timing data.</li>
<li>
<strong>Disable NUMA balancing:</strong> The kernel may migrate memory between NUMA nodes, causing unpredictable stalls.</li>
</ul>

<h3>
  
  
  Storage Optimization (<code>02-storage-setup.sh</code>)
</h3>

<p><strong>Goal:</strong> Consistent I/O performance.</p>

<ul>
<li>
<strong>ext4 without journaling:</strong> Eliminates journal write overhead</li>
<li>
<strong>Mount options:</strong> <code>noatime,nodiratime,nobarrier,discard</code>
</li>
<li>
<strong>noop I/O scheduler:</strong> Best for SSDs</li>
</ul>

<h3>
  
  
  Canary Tests (<code>03-system-validation.sh</code>)
</h3>

<p><strong>Goal:</strong> Distinguish EC2 issues from code regressions.</p>

<p>Run a known workload before your benchmarks. If the canary regresses, the problem is EC2, not your code.<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight shell"><code>run-canary-tests run       <span class="c"># Run all tests</span>
run-canary-tests readiness <span class="c"># Check system configuration</span>
</code></pre>

</div>






<h2>
  
  
  Expected Results
</h2>

<div class="table-wrapper-paragraph"><table>
<thead>
<tr>
<th>Metric</th>
<th>Before</th>
<th>After</th>
</tr>
</thead>
<tbody>
<tr>
<td>Benchmark variance</td>
<td>20–30%</td>
<td>&lt;5%</td>
</tr>
<tr>
<td>False positive rate</td>
<td>High</td>
<td>Near zero</td>
</tr>
<tr>
<td>Regression detection</td>
<td>Unreliable</td>
<td>Catches 5% regressions</td>
</tr>
</tbody>
</table></div>




<h2>
  
  
  Scripts
</h2>

<p>Download these scripts and place them in an <code>ec2-setup</code> folder.</p>

<h3>
  
  
  setup-ec2-instance.sh
</h3>



<div class="highlight js-code-highlight">
<pre class="highlight shell"><code><span class="c">#!/bin/bash</span>
<span class="nb">set</span> <span class="nt">-euo</span> pipefail

<span class="nv">SCRIPT_DIR</span><span class="o">=</span><span class="s2">"</span><span class="si">$(</span><span class="nb">cd</span> <span class="s2">"</span><span class="si">$(</span><span class="nb">dirname</span> <span class="s2">"</span><span class="k">${</span><span class="nv">BASH_SOURCE</span><span class="p">[0]</span><span class="k">}</span><span class="s2">"</span><span class="si">)</span><span class="s2">"</span> <span class="o">&amp;&amp;</span> <span class="nb">pwd</span><span class="si">)</span><span class="s2">"</span>
<span class="nv">LOG_FILE</span><span class="o">=</span><span class="s2">"/var/log/ec2-benchmark-setup.log"</span>

print_header<span class="o">()</span> <span class="o">{</span> <span class="nb">echo</span> <span class="nt">-e</span> <span class="s2">"</span><span class="se">\e</span><span class="s2">[1;34m=== </span><span class="nv">$1</span><span class="s2"> ===</span><span class="se">\e</span><span class="s2">[0m"</span><span class="p">;</span> <span class="o">}</span>
print_status<span class="o">()</span> <span class="o">{</span> <span class="nb">echo</span> <span class="nt">-e</span> <span class="s2">"</span><span class="se">\e</span><span class="s2">[1;32m[</span><span class="si">$(</span><span class="nb">date</span> <span class="s1">'+%H:%M:%S'</span><span class="si">)</span><span class="s2">] </span><span class="nv">$1</span><span class="se">\e</span><span class="s2">[0m"</span> | <span class="nb">tee</span> <span class="nt">-a</span> <span class="s2">"</span><span class="nv">$LOG_FILE</span><span class="s2">"</span><span class="p">;</span> <span class="o">}</span>
print_error<span class="o">()</span> <span class="o">{</span> <span class="nb">echo</span> <span class="nt">-e</span> <span class="s2">"</span><span class="se">\e</span><span class="s2">[1;31m[</span><span class="si">$(</span><span class="nb">date</span> <span class="s1">'+%H:%M:%S'</span><span class="si">)</span><span class="s2">] ERROR: </span><span class="nv">$1</span><span class="se">\e</span><span class="s2">[0m"</span> | <span class="nb">tee</span> <span class="nt">-a</span> <span class="s2">"</span><span class="nv">$LOG_FILE</span><span class="s2">"</span><span class="p">;</span> <span class="o">}</span>
print_warning<span class="o">()</span> <span class="o">{</span> <span class="nb">echo</span> <span class="nt">-e</span> <span class="s2">"</span><span class="se">\e</span><span class="s2">[1;33m[</span><span class="si">$(</span><span class="nb">date</span> <span class="s1">'+%H:%M:%S'</span><span class="si">)</span><span class="s2">] WARNING: </span><span class="nv">$1</span><span class="se">\e</span><span class="s2">[0m"</span> | <span class="nb">tee</span> <span class="nt">-a</span> <span class="s2">"</span><span class="nv">$LOG_FILE</span><span class="s2">"</span><span class="p">;</span> <span class="o">}</span>

check_root<span class="o">()</span> <span class="o">{</span>
    <span class="k">if</span> <span class="o">[</span> <span class="s2">"</span><span class="nv">$EUID</span><span class="s2">"</span> <span class="nt">-ne</span> 0 <span class="o">]</span><span class="p">;</span> <span class="k">then
        </span>print_error <span class="s2">"This script must be run as root"</span>
        <span class="nb">exit </span>1
    <span class="k">fi</span>
<span class="o">}</span>

detect_instance_info<span class="o">()</span> <span class="o">{</span>
    print_status <span class="s2">"Detecting EC2 instance information..."</span>
    <span class="nv">INSTANCE_TYPE</span><span class="o">=</span><span class="si">$(</span>curl <span class="nt">-s</span> http://169.254.169.254/latest/meta-data/instance-type 2&gt;/dev/null <span class="o">||</span> <span class="nb">echo</span> <span class="s2">"unknown"</span><span class="si">)</span>
    print_status <span class="s2">"Instance Type: </span><span class="nv">$INSTANCE_TYPE</span><span class="s2">"</span>
    print_status <span class="s2">"CPU Cores: </span><span class="si">$(</span><span class="nb">nproc</span><span class="si">)</span><span class="s2">"</span>
    print_status <span class="s2">"Memory: </span><span class="si">$(</span>free <span class="nt">-g</span> | <span class="nb">awk</span> <span class="s1">'/^Mem:/{print $2}'</span><span class="si">)</span><span class="s2">GB"</span>
<span class="o">}</span>

update_system<span class="o">()</span> <span class="o">{</span>
    print_header <span class="s2">"Updating System Packages"</span>
    <span class="nb">export </span><span class="nv">DEBIAN_FRONTEND</span><span class="o">=</span>noninteractive
    apt-get update <span class="o">&amp;&amp;</span> apt-get upgrade <span class="nt">-y</span>
    apt-get <span class="nb">install</span> <span class="nt">-y</span> <span class="se">\</span>
        htop iotop sysstat cpufrequtils linux-tools-common <span class="se">\</span>
        stress-ng bc curl wget jq git build-essential <span class="se">\</span>
        fio nvme-cli smartmontools python3 unzip
<span class="o">}</span>

install_rust<span class="o">()</span> <span class="o">{</span>
    print_header <span class="s2">"Installing Rust Toolchain"</span>
    <span class="k">if </span><span class="nb">sudo</span> <span class="nt">-u</span> <span class="s2">"</span><span class="nv">$SUDO_USER</span><span class="s2">"</span> bash <span class="nt">-c</span> <span class="s1">'command -v cargo'</span> &amp;&gt; /dev/null<span class="p">;</span> <span class="k">then
        </span>print_status <span class="s2">"Rust already installed"</span>
        <span class="k">return </span>0
    <span class="k">fi
    </span><span class="nb">sudo</span> <span class="nt">-u</span> <span class="s2">"</span><span class="nv">$SUDO_USER</span><span class="s2">"</span> bash <span class="nt">-c</span> <span class="s1">'curl --proto "=https" --tlsv1.2 -sSf https://sh.rustup.rs | sh -s -- -y'</span>
    <span class="nb">echo</span> <span class="s1">'export PATH="$HOME/.cargo/bin:$PATH"'</span> <span class="o">&gt;&gt;</span> <span class="s2">"/home/</span><span class="nv">$SUDO_USER</span><span class="s2">/.bashrc"</span>
<span class="o">}</span>

setup_directories<span class="o">()</span> <span class="o">{</span>
    <span class="nb">mkdir</span> <span class="nt">-p</span> /usr/local/bin/benchmark-scripts
    <span class="nb">cp</span> <span class="s2">"</span><span class="nv">$SCRIPT_DIR</span><span class="s2">"</span>/<span class="k">*</span>.sh /usr/local/bin/benchmark-scripts/
    <span class="nb">chmod</span> +x /usr/local/bin/benchmark-scripts/<span class="k">*</span>.sh
<span class="o">}</span>

create_convenience_scripts<span class="o">()</span> <span class="o">{</span>
    <span class="nb">cat</span> <span class="o">&gt;</span> /usr/local/bin/run-canary-tests <span class="o">&lt;&lt;</span> <span class="sh">'</span><span class="no">EOF</span><span class="sh">'
#!/bin/bash
exec /usr/local/bin/benchmark-scripts/03-system-validation.sh "</span><span class="nv">$@</span><span class="sh">"
</span><span class="no">EOF
</span>    <span class="nb">chmod</span> +x /usr/local/bin/run-canary-tests

    <span class="nb">cat</span> <span class="o">&gt;</span> /usr/local/bin/benchmark-status <span class="o">&lt;&lt;</span> <span class="sh">'</span><span class="no">EOF</span><span class="sh">'
#!/bin/bash
echo "=== EC2 Benchmark Status ==="
echo "Instance: </span><span class="si">$(</span>curl <span class="nt">-s</span> http://169.254.169.254/latest/meta-data/instance-type 2&gt;/dev/null <span class="o">||</span> <span class="nb">echo</span> <span class="s1">'unknown'</span><span class="si">)</span><span class="sh">"
echo "Cores: </span><span class="si">$(</span><span class="nb">nproc</span><span class="si">)</span><span class="sh">"
echo "Governor: </span><span class="si">$(</span><span class="nb">cat</span> /sys/devices/system/cpu/cpu0/cpufreq/scaling_governor 2&gt;/dev/null <span class="o">||</span> <span class="nb">echo</span> <span class="s1">'N/A'</span><span class="si">)</span><span class="sh">"
echo ""
free -h
echo ""
df -h | grep -E '(/opt/benchmark-data|/</span><span class="nv">$)</span><span class="sh">'
</span><span class="no">EOF
</span>    <span class="nb">chmod</span> +x /usr/local/bin/benchmark-status
<span class="o">}</span>

setup_monitoring<span class="o">()</span> <span class="o">{</span>
    <span class="nb">cat</span> <span class="o">&gt;</span> /etc/cron.d/benchmark-canary <span class="o">&lt;&lt;</span> <span class="sh">'</span><span class="no">EOF</span><span class="sh">'
0 * * * * root /usr/local/bin/run-canary-tests run &gt;&gt; /var/log/canary-cron.log 2&gt;&amp;1
</span><span class="no">EOF
</span><span class="o">}</span>

main<span class="o">()</span> <span class="o">{</span>
    <span class="nb">local </span><span class="nv">ebs_device</span><span class="o">=</span><span class="s2">"</span><span class="k">${</span><span class="nv">1</span><span class="k">:-}</span><span class="s2">"</span>
    check_root
    detect_instance_info
    update_system
    install_rust
    setup_directories

    bash <span class="s2">"</span><span class="nv">$SCRIPT_DIR</span><span class="s2">/01-system-tuning.sh"</span>

    <span class="k">if</span> <span class="o">[</span> <span class="nt">-n</span> <span class="s2">"</span><span class="nv">$ebs_device</span><span class="s2">"</span> <span class="o">]</span><span class="p">;</span> <span class="k">then
        </span>bash <span class="s2">"</span><span class="nv">$SCRIPT_DIR</span><span class="s2">/02-storage-setup.sh"</span> <span class="s2">"</span><span class="nv">$ebs_device</span><span class="s2">"</span>
    <span class="k">else
        </span>print_warning <span class="s2">"No EBS device specified—skipping storage setup"</span>
        <span class="nb">mkdir</span> <span class="nt">-p</span> /opt/benchmark-data/<span class="o">{</span>workdir,results,tmp<span class="o">}</span>
        <span class="nb">chown</span> <span class="nt">-R</span> <span class="s2">"</span><span class="nv">$SUDO_USER</span><span class="s2">:</span><span class="nv">$SUDO_USER</span><span class="s2">"</span> /opt/benchmark-data
    <span class="k">fi

    </span>create_convenience_scripts
    setup_monitoring

    print_header <span class="s2">"Setup Complete"</span>
    print_warning <span class="s2">"REBOOT REQUIRED to apply kernel parameters"</span>
    print_status <span class="s2">"After reboot, run:"</span>
    print_status <span class="s2">"  sudo /usr/local/bin/disable-hyperthreading.sh"</span>
    print_status <span class="s2">"  sudo /usr/local/bin/set-cpu-frequency.sh"</span>
<span class="o">}</span>

main <span class="s2">"</span><span class="nv">$@</span><span class="s2">"</span>
</code></pre>

</div>



<h3>
  
  
  01-system-tuning.sh
</h3>



<div class="highlight js-code-highlight">
<pre class="highlight shell"><code><span class="c">#!/bin/bash</span>
<span class="nb">set</span> <span class="nt">-euo</span> pipefail

<span class="nb">echo</span> <span class="s2">"=== Applying System Tuning ==="</span>

<span class="c"># CPU governor</span>
<span class="k">if</span> <span class="o">[</span> <span class="nt">-d</span> /sys/devices/system/cpu/cpu0/cpufreq <span class="o">]</span><span class="p">;</span> <span class="k">then
    </span><span class="nb">echo</span> <span class="s1">'performance'</span> | <span class="nb">tee</span> /sys/devices/system/cpu/cpu<span class="k">*</span>/cpufreq/scaling_governor
    <span class="nb">cat</span> <span class="o">&gt;</span> /etc/systemd/system/cpu-performance.service <span class="o">&lt;&lt;</span> <span class="sh">'</span><span class="no">EOF</span><span class="sh">'
[Unit]
Description=Set CPU governor to performance
After=multi-user.target

[Service]
Type=oneshot
ExecStart=/bin/bash -c 'echo performance | tee /sys/devices/system/cpu/cpu*/cpufreq/scaling_governor'
RemainAfterExit=yes

[Install]
WantedBy=multi-user.target
</span><span class="no">EOF
</span>    systemctl <span class="nb">enable </span>cpu-performance.service
<span class="k">else
    </span><span class="nb">echo</span> <span class="s2">"CPU frequency scaling not exposed (typical for Xen instances)"</span>
<span class="k">fi</span>

<span class="c"># Kernel parameters</span>
<span class="nb">sed</span> <span class="nt">-i</span> <span class="s1">'s/GRUB_CMDLINE_LINUX_DEFAULT="\(.*\)"/GRUB_CMDLINE_LINUX_DEFAULT="\1 processor.max_cstate=1 intel_idle.max_cstate=1 intel_pstate=disable idle=poll"/'</span> /etc/default/grub
update-grub

<span class="c"># Memory</span>
swapoff <span class="nt">-a</span>
<span class="nb">sed</span> <span class="nt">-i</span> <span class="s1">'/swap/d'</span> /etc/fstab

<span class="nb">cat</span> <span class="o">&gt;</span> /etc/sysctl.d/99-benchmark-tuning.conf <span class="o">&lt;&lt;</span> <span class="sh">'</span><span class="no">EOF</span><span class="sh">'
kernel.numa_balancing=0
vm.swappiness=1
vm.dirty_ratio=15
vm.dirty_background_ratio=5
vm.vfs_cache_pressure=50
</span><span class="no">EOF
</span>sysctl <span class="nt">-p</span> /etc/sysctl.d/99-benchmark-tuning.conf

<span class="c"># Disable unnecessary services</span>
<span class="k">for </span>service <span class="k">in </span>bluetooth cups avahi-daemon ModemManager<span class="p">;</span> <span class="k">do
    </span>systemctl disable <span class="s2">"</span><span class="nv">$service</span><span class="s2">"</span> 2&gt;/dev/null <span class="o">||</span> <span class="nb">true
    </span>systemctl stop <span class="s2">"</span><span class="nv">$service</span><span class="s2">"</span> 2&gt;/dev/null <span class="o">||</span> <span class="nb">true
</span><span class="k">done</span>

<span class="c"># Hyperthreading disable script</span>
<span class="nb">cat</span> <span class="o">&gt;</span> /usr/local/bin/disable-hyperthreading.sh <span class="o">&lt;&lt;</span> <span class="sh">'</span><span class="no">EOF</span><span class="sh">'
#!/bin/bash
CPU_COUNT=</span><span class="si">$(</span><span class="nb">nproc</span><span class="si">)</span><span class="sh">
if [ "</span><span class="nv">$CPU_COUNT</span><span class="sh">" -eq 32 ]; then
    echo "Disabling hyperthreading (32 → 16 cores)..."
    for core in </span><span class="si">$(</span><span class="nb">seq </span>16 31<span class="si">)</span><span class="sh">; do
        echo 0 &gt; /sys/devices/system/cpu/cpu</span><span class="nv">$core</span><span class="sh">/online
    done
    echo "Active cores: </span><span class="si">$(</span><span class="nb">nproc</span><span class="si">)</span><span class="sh">"
fi
</span><span class="no">EOF
</span><span class="nb">chmod</span> +x /usr/local/bin/disable-hyperthreading.sh

<span class="c"># CPU frequency script</span>
<span class="nb">cat</span> <span class="o">&gt;</span> /usr/local/bin/set-cpu-frequency.sh <span class="o">&lt;&lt;</span> <span class="sh">'</span><span class="no">EOF</span><span class="sh">'
#!/bin/bash
if [ -d /sys/devices/system/cpu/cpu0/cpufreq ]; then
    for cpu in /sys/devices/system/cpu/cpu*/cpufreq/scaling_setspeed; do
        [ -f "</span><span class="nv">$cpu</span><span class="sh">" ] &amp;&amp; echo "</span><span class="si">$(</span><span class="nb">cat</span> <span class="s2">"</span><span class="k">${</span><span class="nv">cpu</span><span class="p">%/*</span><span class="k">}</span><span class="s2">/scaling_max_freq"</span><span class="si">)</span><span class="sh">" &gt; "</span><span class="nv">$cpu</span><span class="sh">" 2&gt;/dev/null || true
    done
    echo "CPU frequency locked to maximum"
else
    echo "CPU frequency scaling not available"
fi
</span><span class="no">EOF
</span><span class="nb">chmod</span> +x /usr/local/bin/set-cpu-frequency.sh

<span class="nb">echo</span> <span class="s2">"=== System tuning complete. Reboot required. ==="</span>
</code></pre>

</div>



<h3>
  
  
  02-storage-setup.sh
</h3>



<div class="highlight js-code-highlight">
<pre class="highlight shell"><code><span class="c">#!/bin/bash</span>
<span class="nb">set</span> <span class="nt">-euo</span> pipefail

<span class="nb">echo</span> <span class="s2">"=== Configuring EBS Storage ==="</span>

<span class="nv">EBS_DEVICE</span><span class="o">=</span><span class="s2">"</span><span class="k">${</span><span class="nv">1</span><span class="k">:-}</span><span class="s2">"</span>
<span class="nv">MOUNT_POINT</span><span class="o">=</span><span class="s2">"/opt/benchmark-data"</span>

<span class="k">if</span> <span class="o">[</span> <span class="nt">-z</span> <span class="s2">"</span><span class="nv">$EBS_DEVICE</span><span class="s2">"</span> <span class="o">]</span><span class="p">;</span> <span class="k">then
    </span><span class="nb">echo</span> <span class="s2">"Auto-detecting EBS device..."</span>
    <span class="nv">ROOT_DEVICE</span><span class="o">=</span><span class="si">$(</span>lsblk <span class="nt">-no</span> PKNAME <span class="s2">"</span><span class="si">$(</span>findmnt <span class="nt">-n</span> <span class="nt">-o</span> SOURCE /<span class="si">)</span><span class="s2">"</span> | <span class="nb">head</span> <span class="nt">-1</span><span class="si">)</span>
    <span class="nv">EBS_DEVICE</span><span class="o">=</span><span class="s2">"/dev/</span><span class="si">$(</span>lsblk <span class="nt">-dnr</span> <span class="nt">-o</span> NAME | <span class="nb">grep</span> <span class="nt">-E</span> <span class="s1">'(nvme|xvd)'</span> | <span class="nb">grep</span> <span class="nt">-v</span> <span class="s2">"</span><span class="nv">$ROOT_DEVICE</span><span class="s2">"</span> | <span class="nb">grep</span> <span class="nt">-v</span> <span class="s1">'xvda'</span> | <span class="nb">head</span> <span class="nt">-1</span><span class="si">)</span><span class="s2">"</span>
    <span class="nb">echo</span> <span class="s2">"Detected: </span><span class="nv">$EBS_DEVICE</span><span class="s2">"</span>
<span class="k">fi</span>

<span class="o">[</span> <span class="o">!</span> <span class="nt">-b</span> <span class="s2">"</span><span class="nv">$EBS_DEVICE</span><span class="s2">"</span> <span class="o">]</span> <span class="o">&amp;&amp;</span> <span class="nb">echo</span> <span class="s2">"ERROR: Device </span><span class="nv">$EBS_DEVICE</span><span class="s2"> not found!"</span> <span class="o">&amp;&amp;</span> <span class="nb">exit </span>1

<span class="c"># Create filesystem</span>
mkfs.ext4 <span class="nt">-F</span> <span class="nt">-O</span> ^has_journal <span class="nt">-E</span> <span class="nv">stride</span><span class="o">=</span>32,stripe-width<span class="o">=</span>32 <span class="nt">-b</span> 4096 <span class="nt">-m</span> 0 <span class="s2">"</span><span class="nv">$EBS_DEVICE</span><span class="s2">"</span>

<span class="nb">mkdir</span> <span class="nt">-p</span> <span class="s2">"</span><span class="nv">$MOUNT_POINT</span><span class="s2">"</span>
<span class="nv">UUID</span><span class="o">=</span><span class="si">$(</span>blkid <span class="nt">-s</span> UUID <span class="nt">-o</span> value <span class="s2">"</span><span class="nv">$EBS_DEVICE</span><span class="s2">"</span><span class="si">)</span>
<span class="nb">sed</span> <span class="nt">-i</span> <span class="s2">"</span><span class="se">\|</span><span class="nv">$MOUNT_POINT</span><span class="s2">|d"</span> /etc/fstab
<span class="nb">echo</span> <span class="s2">"UUID=</span><span class="nv">$UUID</span><span class="s2"> </span><span class="nv">$MOUNT_POINT</span><span class="s2"> ext4 noatime,nodiratime,nobarrier,discard 0 2"</span> <span class="o">&gt;&gt;</span> /etc/fstab
mount <span class="s2">"</span><span class="nv">$MOUNT_POINT</span><span class="s2">"</span>

<span class="nb">mkdir</span> <span class="nt">-p</span> <span class="s2">"</span><span class="nv">$MOUNT_POINT</span><span class="s2">"</span>/<span class="o">{</span>workdir,results,tmp<span class="o">}</span>
<span class="nb">chmod </span>755 <span class="s2">"</span><span class="nv">$MOUNT_POINT</span><span class="s2">"</span>/<span class="o">{</span>workdir,results,tmp<span class="o">}</span>

<span class="c"># I/O scheduler</span>
<span class="nv">DEVICE_NAME</span><span class="o">=</span><span class="si">$(</span><span class="nb">basename</span> <span class="s2">"</span><span class="nv">$EBS_DEVICE</span><span class="s2">"</span><span class="si">)</span>
<span class="nb">echo </span>noop <span class="o">&gt;</span> <span class="s2">"/sys/block/</span><span class="nv">$DEVICE_NAME</span><span class="s2">/queue/scheduler"</span> 2&gt;/dev/null <span class="o">||</span> <span class="nb">true
echo </span>32 <span class="o">&gt;</span> <span class="s2">"/sys/block/</span><span class="nv">$DEVICE_NAME</span><span class="s2">/queue/nr_requests"</span> 2&gt;/dev/null <span class="o">||</span> <span class="nb">true

chown</span> <span class="nt">-R</span> <span class="s2">"</span><span class="si">$(</span><span class="nb">logname</span><span class="si">)</span><span class="s2">"</span>:<span class="s2">"</span><span class="si">$(</span><span class="nb">logname</span><span class="si">)</span><span class="s2">"</span> <span class="s2">"</span><span class="nv">$MOUNT_POINT</span><span class="s2">"</span> 2&gt;/dev/null <span class="o">||</span> <span class="nb">true

echo</span> <span class="s2">"=== EBS configured at </span><span class="nv">$MOUNT_POINT</span><span class="s2"> ==="</span>
</code></pre>

</div>



<h3>
  
  
  03-system-validation.sh
</h3>



<div class="highlight js-code-highlight">
<pre class="highlight shell"><code><span class="c">#!/bin/bash</span>
<span class="nb">set</span> <span class="nt">-euo</span> pipefail

<span class="nv">RESULTS_DIR</span><span class="o">=</span><span class="s2">"/opt/benchmark-data/results/canary"</span>
<span class="nv">LOG_FILE</span><span class="o">=</span><span class="s2">"/var/log/ec2-canary-tests.log"</span>

log<span class="o">()</span> <span class="o">{</span> <span class="nb">echo</span> <span class="s2">"[</span><span class="si">$(</span><span class="nb">date</span> <span class="s1">'+%Y-%m-%d %H:%M:%S'</span><span class="si">)</span><span class="s2">] </span><span class="nv">$1</span><span class="s2">"</span> | <span class="nb">tee</span> <span class="nt">-a</span> <span class="s2">"</span><span class="nv">$LOG_FILE</span><span class="s2">"</span><span class="p">;</span> <span class="o">}</span>

run_cpu_canary<span class="o">()</span> <span class="o">{</span>
    log <span class="s2">"Running CPU canary..."</span>
    <span class="nb">mkdir</span> <span class="nt">-p</span> <span class="s2">"</span><span class="nv">$RESULTS_DIR</span><span class="s2">/cpu"</span>
    <span class="nb">local </span><span class="nv">start</span><span class="o">=</span><span class="si">$(</span><span class="nb">date</span> +%s.%N<span class="si">)</span>
    <span class="nb">echo</span> <span class="s2">"scale=1000; 4*a(1)"</span> | <span class="nb">timeout </span>5s bc <span class="nt">-l</span> <span class="o">&gt;</span> /dev/null 2&gt;&amp;1 <span class="o">||</span> <span class="nb">true
    local </span><span class="nv">duration</span><span class="o">=</span><span class="si">$(</span><span class="nb">echo</span> <span class="s2">"</span><span class="si">$(</span><span class="nb">date</span> +%s.%N<span class="si">)</span><span class="s2"> - </span><span class="nv">$start</span><span class="s2">"</span> | bc <span class="nt">-l</span><span class="si">)</span>
    <span class="nb">echo</span> <span class="s2">"</span><span class="nv">$duration</span><span class="s2">"</span> <span class="o">&gt;</span> <span class="s2">"</span><span class="nv">$RESULTS_DIR</span><span class="s2">/cpu/pi_</span><span class="si">$(</span><span class="nb">date</span> +%s<span class="si">)</span><span class="s2">.txt"</span>
    log <span class="s2">"CPU canary: </span><span class="k">${</span><span class="nv">duration</span><span class="k">}</span><span class="s2">s"</span>
<span class="o">}</span>

run_io_canary<span class="o">()</span> <span class="o">{</span>
    log <span class="s2">"Running I/O canary..."</span>
    <span class="nb">mkdir</span> <span class="nt">-p</span> <span class="s2">"</span><span class="nv">$RESULTS_DIR</span><span class="s2">/io"</span>
    mountpoint <span class="nt">-q</span> <span class="s2">"/opt/benchmark-data"</span> <span class="o">||</span> <span class="o">{</span> log <span class="s2">"ERROR: Storage not mounted"</span><span class="p">;</span> <span class="k">return </span>1<span class="p">;</span> <span class="o">}</span>

    <span class="nb">local </span><span class="nv">test_file</span><span class="o">=</span><span class="s2">"/opt/benchmark-data/tmp/canary.dat"</span>
    <span class="nb">local </span><span class="nv">start</span><span class="o">=</span><span class="si">$(</span><span class="nb">date</span> +%s.%N<span class="si">)</span>
    <span class="nb">dd </span><span class="k">if</span><span class="o">=</span>/dev/zero <span class="nv">of</span><span class="o">=</span><span class="s2">"</span><span class="nv">$test_file</span><span class="s2">"</span> <span class="nv">bs</span><span class="o">=</span>4k <span class="nv">count</span><span class="o">=</span>25600 <span class="nv">oflag</span><span class="o">=</span>direct 2&gt;/dev/null
    <span class="nb">local </span><span class="nv">mbps</span><span class="o">=</span><span class="si">$(</span><span class="nb">echo</span> <span class="s2">"scale=2; (25600 * 4) / 1024 / (</span><span class="si">$(</span><span class="nb">date</span> +%s.%N<span class="si">)</span><span class="s2"> - </span><span class="nv">$start</span><span class="s2">)"</span> | bc <span class="nt">-l</span><span class="si">)</span>
    <span class="nb">rm</span> <span class="nt">-f</span> <span class="s2">"</span><span class="nv">$test_file</span><span class="s2">"</span>
    <span class="nb">echo</span> <span class="s2">"</span><span class="nv">$mbps</span><span class="s2">"</span> <span class="o">&gt;</span> <span class="s2">"</span><span class="nv">$RESULTS_DIR</span><span class="s2">/io/write_</span><span class="si">$(</span><span class="nb">date</span> +%s<span class="si">)</span><span class="s2">.txt"</span>
    log <span class="s2">"I/O canary: </span><span class="k">${</span><span class="nv">mbps</span><span class="k">}</span><span class="s2"> MB/s"</span>
<span class="o">}</span>

check_readiness<span class="o">()</span> <span class="o">{</span>
    log <span class="s2">"Checking readiness..."</span>
    <span class="nb">local </span><span class="nv">ready</span><span class="o">=</span><span class="nb">true
    local </span><span class="nv">gov</span><span class="o">=</span><span class="si">$(</span><span class="nb">cat</span> /sys/devices/system/cpu/cpu0/cpufreq/scaling_governor 2&gt;/dev/null <span class="o">||</span> <span class="nb">echo</span> <span class="s2">"unknown"</span><span class="si">)</span>
    <span class="o">[</span> <span class="s2">"</span><span class="nv">$gov</span><span class="s2">"</span> <span class="o">!=</span> <span class="s2">"performance"</span> <span class="o">]</span> <span class="o">&amp;&amp;</span> <span class="o">[</span> <span class="s2">"</span><span class="nv">$gov</span><span class="s2">"</span> <span class="o">!=</span> <span class="s2">"unknown"</span> <span class="o">]</span> <span class="o">&amp;&amp;</span> log <span class="s2">"WARNING: Governor is '</span><span class="nv">$gov</span><span class="s2">'"</span> <span class="o">&amp;&amp;</span> <span class="nv">ready</span><span class="o">=</span><span class="nb">false
    </span>mountpoint <span class="nt">-q</span> <span class="s2">"/opt/benchmark-data"</span> <span class="o">||</span> <span class="o">{</span> log <span class="s2">"WARNING: Storage not mounted"</span><span class="p">;</span> <span class="nv">ready</span><span class="o">=</span><span class="nb">false</span><span class="p">;</span> <span class="o">}</span>
    <span class="o">[</span> <span class="s2">"</span><span class="nv">$ready</span><span class="s2">"</span> <span class="o">=</span> <span class="nb">true</span> <span class="o">]</span> <span class="o">&amp;&amp;</span> log <span class="s2">"System ready"</span> <span class="o">||</span> log <span class="s2">"Configuration issues detected"</span>
<span class="o">}</span>

<span class="k">case</span> <span class="s2">"</span><span class="k">${</span><span class="nv">1</span><span class="k">:-</span><span class="nv">run</span><span class="k">}</span><span class="s2">"</span> <span class="k">in
    </span>run<span class="p">)</span> <span class="nb">mkdir</span> <span class="nt">-p</span> <span class="s2">"</span><span class="nv">$RESULTS_DIR</span><span class="s2">"</span><span class="p">;</span> run_cpu_canary<span class="p">;</span> run_io_canary<span class="p">;</span> log <span class="s2">"Canary tests complete"</span> <span class="p">;;</span>
    readiness<span class="p">)</span> check_readiness <span class="p">;;</span>
    <span class="k">*</span><span class="p">)</span> <span class="nb">echo</span> <span class="s2">"Usage: </span><span class="nv">$0</span><span class="s2"> [run|readiness]"</span> <span class="p">;;</span>
<span class="k">esac</span>
</code></pre>

</div>



<h3>
  
  
  revert-all-optimizations.sh
</h3>



<div class="highlight js-code-highlight">
<pre class="highlight shell"><code><span class="c">#!/bin/bash</span>
<span class="nb">set</span> <span class="nt">-euo</span> pipefail

<span class="o">[</span> <span class="s2">"</span><span class="nv">$EUID</span><span class="s2">"</span> <span class="nt">-ne</span> 0 <span class="o">]</span> <span class="o">&amp;&amp;</span> <span class="nb">echo</span> <span class="s2">"Must be run as root"</span> <span class="o">&amp;&amp;</span> <span class="nb">exit </span>1

<span class="nb">echo</span> <span class="s2">"=== Reverting Optimizations ==="</span>

<span class="c"># Re-enable all cores</span>
<span class="k">for </span>core <span class="k">in</span> <span class="si">$(</span><span class="nb">seq </span>0 31<span class="si">)</span><span class="p">;</span> <span class="k">do
    </span><span class="nb">echo </span>1 <span class="o">&gt;</span> <span class="s2">"/sys/devices/system/cpu/cpu</span><span class="nv">$core</span><span class="s2">/online"</span> 2&gt;/dev/null <span class="o">||</span> <span class="nb">true
</span><span class="k">done</span>

<span class="c"># Restore governor</span>
<span class="k">for </span>gov <span class="k">in</span> /sys/devices/system/cpu/cpu<span class="k">*</span>/cpufreq/scaling_governor<span class="p">;</span> <span class="k">do
    </span><span class="nb">echo </span>ondemand <span class="o">&gt;</span> <span class="s2">"</span><span class="nv">$gov</span><span class="s2">"</span> 2&gt;/dev/null <span class="o">||</span> <span class="nb">true
</span><span class="k">done</span>

<span class="c"># Unmount storage</span>
umount <span class="s2">"/opt/benchmark-data"</span> 2&gt;/dev/null <span class="o">||</span> <span class="nb">true
sed</span> <span class="nt">-i</span> <span class="s1">'\|/opt/benchmark-data|d'</span> /etc/fstab

<span class="c"># Remove services and scripts</span>
systemctl disable cpu-performance.service 2&gt;/dev/null <span class="o">||</span> <span class="nb">true
rm</span> <span class="nt">-f</span> /etc/systemd/system/cpu-performance.service
<span class="nb">rm</span> <span class="nt">-f</span> /etc/sysctl.d/99-benchmark-tuning.conf
<span class="nb">rm</span> <span class="nt">-f</span> /usr/local/bin/<span class="o">{</span>run-canary-tests,benchmark-status,disable-hyperthreading.sh,set-cpu-frequency.sh<span class="o">}</span>
<span class="nb">rm</span> <span class="nt">-rf</span> /usr/local/bin/benchmark-scripts
<span class="nb">rm</span> <span class="nt">-f</span> /etc/cron.d/benchmark-<span class="o">{</span>canary,stats<span class="o">}</span>

<span class="c"># Restore services</span>
<span class="k">for </span>svc <span class="k">in </span>bluetooth cups avahi-daemon<span class="p">;</span> <span class="k">do
    </span>systemctl <span class="nb">enable</span> <span class="s2">"</span><span class="nv">$svc</span><span class="s2">"</span> 2&gt;/dev/null <span class="o">||</span> <span class="nb">true
</span><span class="k">done

</span>systemctl daemon-reload
<span class="nb">echo</span> <span class="s2">"=== Revert complete. Reboot required. ==="</span>
</code></pre>

</div>






<h2>
  
  
  Key Takeaways
</h2>

<ol>
<li><p><strong>Repeatability over performance.</strong> A slower but stable benchmark is infinitely more useful than a fast but noisy one.</p></li>
<li><p><strong>Measure, don't assume.</strong> MongoDB's team was surprised that ephemeral SSDs performed worse than EBS—the opposite of conventional wisdom.</p></li>
<li><p><strong>Canary tests are essential.</strong> They distinguish "the cloud is being weird" from "my code got slower."</p></li>
<li><p><strong>Disable, don't tune.</strong> For benchmarking, disabling dynamic features entirely beats trying to tune them.</p></li>
<li><p><strong>Baseline comparison works.</strong> Comparing feature branches against main provides meaningful regression detection even in virtualized environments.</p></li>
</ol>




<p><em>If you found this useful, I'd love to hear about your experience applying these optimizations to your own benchmarking infrastructure.</em></p>

<p><em>Special thanks to the MongoDB performance engineering team for their extensive research on EC2 benchmarking variability, which formed the foundation for this guide.</em></p>

