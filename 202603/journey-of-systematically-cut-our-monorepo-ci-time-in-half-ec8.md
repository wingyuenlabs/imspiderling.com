---
Title: Journey of Systematically Cut Our Monorepo CI Time in Half
Description: 
Author: Jimmy Yeung
Date: 2026-03-26T21:51:34.000Z
Robots: noindex,nofollow
Template: index
---
<p>The engineering team maintains a Python/TypeScript monorepo with 22+ Django microservices, a React frontend, and shared libraries — all running through GitHub Actions CI. Over time, CI had become the productivity bottleneck. PRs routinely waited 10-15 minutes for green checks, and the slowest pipeline consistently exceeded 10 minutes. Engineers were context-switching while waiting for builds, or worse, stacking PRs on top of unverified ones.</p>

<p>I ran a focused initiative to systematically identify and eliminate CI bottlenecks. This post is a record of how I approached it.</p>




<h2>
  
  
  1. Finding the Bottleneck
</h2>

<p>You can't optimize what you can't see. So my first instinct was to write a script to pull GitHub Actions workflow run data from the API and compute aggregate statistics — average, P50, P75, P90 — for both wall clock time and per-job durations across any date range.</p>

<p>This immediately told me two things:</p>

<ol>
<li>
<strong>The largest service's unit-test job was the critical path</strong> — at P50 of ~10 minutes, it single-handedly determined the CI wall clock time.</li>
<li>
<strong>Other services stayed around 8-9 minutes</strong> — they were running multiple jobs in parallel so further investigation was needed.</li>
</ol>

<p>Having this data let me prioritize ruthlessly. Instead of optimizing everything at once, I focused on the jobs that actually moved the wall clock needle.</p>

<p>And it's always satisfying to compare the before vs. after statistics.</p>




<h2>
  
  
  2. Low-Hanging Fruit — Modernizing the Toolchain
</h2>

<p>My first lever was swapping slow tools for faster alternatives. The Rust-based ecosystem has matured to the point where several tools are genuine drop-in replacements, so this felt like a no-brainer:</p>

<ul>
<li>Yarn to Bun (not Rust-written, but famous for its speed and maturity)</li>
<li>Webpack to Rspack</li>
<li>storybook-rsbuild</li>
<li>Adopt <code>eslint-plugin-oxlint</code>, disabling ESLint rules that oxlint now handles</li>
</ul>

<p>Each swap reduced CI time per job by 30-40%.</p>




<h2>
  
  
  3. Low-Hanging Fruit — Docker Build Optimization
</h2>

<p>Docker builds were another time sink. I attacked from multiple angles:</p>

<ul>
<li>
<strong>Enabled <code>.dockerignore</code></strong> — builds weren't using ignore files because the build context was outside their scope</li>
<li>
<strong>Optimized <code>uv sync</code> in Dockerfiles</strong> — eliminated unnecessary dependency installation loops; <code>uv build --package</code> uses build isolation and never touches the workspace venv</li>
<li>
<strong>Cleaned up stale cross-service dependencies</strong> — after migrating to uv workspaces, some services had phantom dependencies on unrelated services, triggering unnecessary rebuilds</li>
</ul>

<p>This reduced Docker build CI time by about 50%.</p>




<h2>
  
  
  4. Test Duration Enforcement &amp; Slow Test Fixes
</h2>

<p>After tackling the low-hanging fruit, I turned to the tests themselves. The repo already had a <code>check-test-durations</code> composite action that parses JUnit XML and reports the top 10 slowest tests — but it was only wired up in 2 of 22 services.</p>

<h3>
  
  
  Rolling Out Visibility
</h3>

<p>I added the duration check to all workspace CI workflows with <code>continue-on-error: true</code> — visibility without blocking builds.</p>

<p>This immediately surfaced tests taking 10-30+ seconds across services that had never been profiled.</p>

<h3>
  
  
  Fixing the Worst Offenders
</h3>

<p>With data in hand, I targeted the slowest individual tests. Some were making unnecessary network calls, others were creating excessive test data, and a few were simply doing too much in a single test. I fixed tests exceeding 10 seconds across the two slowest services, bringing meaningful improvements before touching any CI infrastructure.</p>

<p>(One fun find: somewhere the application code called <code>time.sleep(10)</code>, and the test didn't mock it — so the test took at least 10 seconds for free. Sometimes reducing test duration is that simple.)</p>

<h3>
  
  
  Enforcing the Threshold
</h3>

<p>After the initial round of fixes, I made the duration check a hard enforcement — any individual test exceeding 10 seconds fails the build. This prevents slow tests from creeping back in, turning a one-time fix into a sustainable practice.</p>




<h2>
  
  
  5. Caching: Every Layer Counts
</h2>

<p>GitHub Actions provides a 10 GB cache quota per repository. In a monorepo with 22+ services, that's not much — so every cache needs to earn its space. I identified and enabled caching at four layers of the CI pipeline.</p>

<h3>
  
  
  <code>.git</code> Cache
</h3>

<p>In a monorepo, <code>git clone</code> with <code>fetch-depth: 0</code> (needed for change detection and visual regression diff baselines) is expensive — the repo has significant history. I set up a scheduled workflow that runs every 6 hours to pre-populate the <code>.git</code> directory cache on master.</p>

<h3>
  
  
  <code>uv</code> Cache
</h3>

<p>I enabled <code>setup-uv</code>'s built-in cache across all CI workflow files.</p>

<p>The cache is keyed on the <code>uv.lock</code> hash, so it invalidates exactly when dependencies change. Impact: ~5 seconds saved per job on the "Install uv" step (8s → 3s average). Small per-job, just a few hundred KB.</p>

<h3>
  
  
  <code>node_modules</code> Cache
</h3>

<p>The frontend workflows cache <code>node_modules</code> after <code>bun install</code>. The cache key was previously overloaded — some callers passed a branch name (<code>${{ github.head_ref }}</code>), others a static date string. Branch-keyed caches created redundant entries for the same lockfile, wasting quota.</p>

<p>I simplified to a single lockfile-based key using <code>bun.lock</code>.</p>

<p>GitHub Actions already scopes cache access by branch (PRs can read from the base branch's cache but not other PRs'), so per-branch keys were pure overhead. This change eliminated redundant cache entries and improved hit rates.</p>

<h3>
  
  
  ESLint Cache
</h3>

<p>ESLint is one of the slower steps in frontend validation. I enabled its persistent cache by restoring <code>.eslintcache</code> between runs.</p>

<p>On cache hit, ESLint only re-lints files that changed since the last run, skipping the majority of the codebase.</p>




<h2>
  
  
  6. Test Parallelization (with Cost in Mind)
</h2>

<p>Parallelization is the most effective lever for reducing wall clock time — but in GitHub Actions, more parallel jobs means more billable minutes. Every matrix shard spins up a fresh runner, installs dependencies, and tears down. The setup overhead isn't free. I approached this deliberately, targeting parallelization where the payoff justified the cost.</p>

<h3>
  
  
  pytest-xdist: Free Parallelism
</h3>

<p>The lowest-cost optimization is utilizing all CPUs within a single runner. I enabled <code>pytest-xdist</code> with <code>-n auto</code> in services that were running tests serially.</p>

<p>For one service, this cut pytest execution from ~4m30s to ~2m07s — roughly 2x faster with zero additional runner cost.</p>

<h3>
  
  
  Matrix Sharding: Deliberate Trade-off
</h3>

<p>For the largest service (8,674 tests, ~10 minute P50), single-runner parallelism wasn't enough. I restructured its CI into 7 matrix shards to keep each under 5 minutes.</p>

<p><strong>The cost trade-off</strong>: 7 shards means 7x the setup overhead (dependency installation, database creation). Total billable minutes actually increased. But the wall clock time — what engineers wait on — dropped from ~10 minutes to ~5 minutes. For a team making dozens of PRs daily, the productivity gain far outweighs the compute cost.</p>




<h2>
  
  
  Takeaways
</h2>

<ol>
<li><p><strong>Measure first, always.</strong> A simple Python script using the <code>gh</code> CLI gave me reproducible P50/P75/P90 data across any date range. Way more reliable than eyeballing the Actions UI — and it makes before/after comparisons trivial.</p></li>
<li><p><strong>Modernize your toolchain — it's free speed.</strong> Bun, Rspack, and oxlint are drop-in replacements that delivered 30-40% speedups per job with minimal migration effort. Highest ROI work I did.</p></li>
<li><p><strong>Instrument test durations, then enforce them.</strong> A 30-second test hiding in a 6-minute suite is invisible until you look. These are often the easiest wins — and a hard time budget prevents regression.</p></li>
<li><p><strong>In a monorepo, cache quota is a shared resource.</strong> GitHub Actions' 10 GB limit is shared across all workflows. A poorly-keyed cache doesn't just waste space — it actively evicts caches that matter.</p></li>
<li><p><strong>Parallelism costs money — spend it wisely.</strong> pytest-xdist within a single runner is free performance. Matrix sharding trades billable minutes for wall clock time. Make that trade deliberately, and know the numbers before and after.</p></li>
</ol>

