---
Title: Automating .NET Localization: From Code Scan to Auto-Translate to Pull Request
Description: 
Author: Nikolaos Protopapas
Date: 2025-11-14T21:34:56.000Z
Robots: noindex,nofollow
Template: index
---
<p>A cross-platform CLI/TUI for managing .resx files with free AI translation, code scanning, and CI/CD automation</p>

<p><a href="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fu66x5lf8cm5v7sng6qqu.gif" class="article-body-image-wrapper"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fu66x5lf8cm5v7sng6qqu.gif" alt=" " width="1028" height="608"></a></p>

<h2>
  
  
  The Problem Every .NET Developer Faces
</h2>

<p>You've built a great .NET application. Now you need to support multiple languages. You create your first <code>.resx</code> file, add some keys, and everything works beautifully in English. Then the requests come in: "Can we support French? German? Greek?"</p>

<p>You create <code>Resources.fr.resx</code>, copy all the keys, send them to a translator, paste the translations back... and your build breaks. An XML tag got corrupted. A key is missing. A duplicate slipped in. You're now manually diff-ing XML files trying to figure out what went wrong.</p>

<p>If you're on Linux or macOS, it's even worse. Visual Studio's ResX Resource Manager? Windows-only. Editing XML by hand? Error-prone and painful. You end up writing bash scripts that barely work, or worse, you spin up a Windows VM just to manage resource files.</p>

<p><strong>There had to be a better way.</strong></p>

<h2>
  
  
  Understanding .NET Localization: The Foundation
</h2>

<p>Before we dive into the solution, let's understand how .NET actually handles localization. If you've ever wondered what those <code>.resx</code> files really are and how they work at runtime, this section is for you.</p>

<h3>
  
  
  What Are .resx Files?
</h3>

<p>Resource files (<code>.resx</code>) are XML-based files that store localized strings, images, and other resources. Here's what a typical resource file looks like:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight xml"><code><span class="cp">&lt;?xml version="1.0" encoding="utf-8"?&gt;</span>
<span class="nt">&lt;root&gt;</span>
  <span class="nt">&lt;data</span> <span class="na">name=</span><span class="s">"WelcomeMessage"</span> <span class="na">xml:space=</span><span class="s">"preserve"</span><span class="nt">&gt;</span>
    <span class="nt">&lt;value&gt;</span>Welcome to our application!<span class="nt">&lt;/value&gt;</span>
    <span class="nt">&lt;comment&gt;</span>Shown on the home page<span class="nt">&lt;/comment&gt;</span>
  <span class="nt">&lt;/data&gt;</span>
  <span class="nt">&lt;data</span> <span class="na">name=</span><span class="s">"SaveButton"</span> <span class="na">xml:space=</span><span class="s">"preserve"</span><span class="nt">&gt;</span>
    <span class="nt">&lt;value&gt;</span>Save<span class="nt">&lt;/value&gt;</span>
  <span class="nt">&lt;/data&gt;</span>
  <span class="nt">&lt;data</span> <span class="na">name=</span><span class="s">"CancelButton"</span> <span class="na">xml:space=</span><span class="s">"preserve"</span><span class="nt">&gt;</span>
    <span class="nt">&lt;value&gt;</span>Cancel<span class="nt">&lt;/value&gt;</span>
  <span class="nt">&lt;/data&gt;</span>
<span class="nt">&lt;/root&gt;</span>
</code></pre>

</div>



<p>Each <code>&lt;data&gt;</code> element contains:</p>

<ul>
<li>
<strong>name</strong>: The key you use in code</li>
<li>
<strong>value</strong>: The translated text</li>
<li>
<strong>comment</strong> (optional): Context for translators</li>
</ul>

<h3>
  
  
  The Resource File Naming Convention
</h3>

<p>.NET uses a specific naming convention to organize translations:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight plaintext"><code>Resources/
├── SharedResources.resx          ← Default/invariant culture (usually English)
├── SharedResources.fr.resx       ← French
├── SharedResources.de.resx       ← German
├── SharedResources.el.resx       ← Greek
└── SharedResources.fr-CA.resx    ← French Canadian (specific culture)
</code></pre>

</div>



<p>The pattern is simple: <code>{BaseName}.{culture-code}.resx</code></p>

<h3>
  
  
  How the ResourceManager Works
</h3>

<p>When your application runs, .NET's <code>ResourceManager</code> automatically loads the correct resource file based on the current culture:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight plaintext"><code>User's Culture: fr-CA (French Canadian)
                  │
                  ↓
         Look for: SharedResources.fr-CA.resx
                  │
                  ↓ (not found)
         Fall back to: SharedResources.fr.resx
                  │
                  ↓ (not found)
         Fall back to: SharedResources.resx (default)
</code></pre>

</div>



<p>This fallback mechanism ensures users always see <em>something</em>, even if a specific translation is missing. But it also means missing translations can go unnoticed until production.</p>

<h3>
  
  
  The Culture Fallback Chain
</h3>



<div class="highlight js-code-highlight">
<pre class="highlight plaintext"><code>┌─────────────────────────────────────────────────────────┐
│  Application Runtime                                    │
│                                                         │
│  CurrentUICulture: "el-GR" (Greek - Greece)             │
└────────────────────┬────────────────────────────────────┘
                     │
                     ↓
        ┌─────────────────────────────────────────────────┐
        │ ResourceManager.GetString("WelcomeMessage")     │
        └────────────┬────────────────────────────────────┘
                     │
                     ↓
        ┌─────────────────────────────────────────────────┐
        │ 1. Try: Resources.el-GR.resx                    │
        │     File not found                              │
        └────────────┬────────────────────────────────────┘
                     │
                     ↓
        ┌─────────────────────────────────────────────────┐
        │ 2. Try: Resources.el.resx                       │
        │     Found! Return Greek translation             │
        └─────────────────────────────────────────────────┘
                     │
                     ↓ (if not found)
        ┌─────────────────────────────────────────────────┐
        │ 3. Fallback: Resources.resx                     │
        │     Return default (English)                    │
        └─────────────────────────────────────────────────┘
</code></pre>

</div>



<h3>
  
  
  Common Pain Points in .NET Localization
</h3>

<p>Through building LocalizationManager, I encountered (and solved) these recurring issues:</p>

<ol>
<li>
<strong>XML Corruption</strong>: One misplaced <code>&lt;</code> or <code>&amp;</code> breaks your entire build</li>
<li>
<strong>Missing Translations</strong>: Keys exist in the default file but not in translated files</li>
<li>
<strong>Duplicate Keys</strong>: Same key defined twice = build error</li>
<li>
<strong>Empty Values</strong>: Is an empty <code>&lt;value&gt;&lt;/value&gt;</code> intentional or a mistake?</li>
<li>
<strong>Sync Issues</strong>: You add a key to <code>Resources.resx</code> but forget to add it to all language files</li>
<li>
<strong>No Validation</strong>: You only discover issues when the build fails</li>
<li>
<strong>Manual Translation</strong>: Expensive, slow, and doesn't scale</li>
</ol>

<h3>
  
  
  Why Existing Tools Fall Short
</h3>

<ul>
<li>
<strong>Visual Studio</strong>: Windows-only, requires full IDE installation</li>
<li>
<strong>ResX Resource Manager</strong>: Excellent tool, but also Windows-only</li>
<li>
<strong>Manual XML Editing</strong>: Works everywhere, but error-prone and lacks validation</li>
<li>
<strong>Online Tools</strong>: Require uploading sensitive data, no local workflow integration</li>
<li>
<strong>Custom Scripts</strong>: Everyone reinvents the wheel poorly</li>
</ul>

<p><strong>The gap was clear</strong>: We needed a cross-platform, command-line-first tool that could handle the entire localization workflow from validation to automated translation to CI/CD integration.</p>

<h2>
  
  
  The Solution: LocalizationManager
</h2>

<p>I built <a href="https://github.com/nickprotop/LocalizationManager" rel="noopener noreferrer">LocalizationManager (lrm)</a> to be the tool I wished existed when I started working with .NET localization on Linux.</p>

<h3>
  
  
  Core Philosophy
</h3>

<ol>
<li>
<strong>Cross-platform first</strong>: Native binaries for Linux, Windows, macOS, including ARM64</li>
<li>
<strong>CLI-first, with a bonus TUI</strong>: Perfect for automation and CI/CD, but also great for interactive use</li>
<li>
<strong>Complete workflow</strong>: Not just a translator, but the entire pipeline</li>
<li>
<strong>Modern approach</strong>: AI translation with caching, smart batch processing</li>
<li>
<strong>Developer-friendly</strong>: JSON output, exit codes, extensive documentation</li>
</ol>

<h3>
  
  
  Key Features at a Glance
</h3>

<ul>
<li>✅ <strong>Validation</strong>: Detect missing translations, duplicates, XML errors</li>
<li>📊 <strong>Statistics</strong>: Translation coverage with progress bars</li>
<li>🔍 <strong>Code Scanning</strong>: Find unused keys and missing references in your source code</li>
<li>🌐 <strong>Auto-Translation</strong>: 7 providers including Google, DeepL, OpenAI, Claude, and local Ollama</li>
<li>📺 <strong>Interactive TUI</strong>: Visual editing with keyboard shortcuts</li>
<li>🚀 <strong>CI/CD Ready</strong>: JSON output, exit codes, GitHub Actions integration</li>
<li>💾 <strong>Import/Export</strong>: CSV format for working with translators</li>
<li>📝 <strong>Full CLI</strong>: 15+ commands for every localization task</li>
</ul>

<h2>
  
  
  Architecture &amp; Technical Design
</h2>

<p>Let me share some of the key architectural decisions that make LocalizationManager work well.</p>

<h3>
  
  
  Technology Stack
</h3>

<ul>
<li>
<strong>Language</strong>: C# / .NET 9.0</li>
<li>
<strong>TUI Framework</strong>: Spectre.Console (amazing for rich terminal interfaces)</li>
<li>
<strong>CLI Framework</strong>: Spectre.Console.Cli (command routing and parsing)</li>
<li>
<strong>Translation APIs</strong>: Native HTTP clients (no heavy SDK dependencies)</li>
<li>
<strong>Caching</strong>: SQLite for translation cache</li>
<li>
<strong>Packaging</strong>: Self-contained single-file executables</li>
</ul>

<h3>
  
  
  Translation Provider Architecture
</h3>

<p>One of the most important design decisions was creating an abstraction for translation providers. This allows users to choose the best provider for their needs while keeping the code maintainable.<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight plaintext"><code>┌─────────────────────────────────────────────────────────────┐
│                   TranslationProviderFactory                │
│                                                             │
│  Create(providerName) → ITranslationProvider                │
└───────────────────┬─────────────────────────────────────────┘
                    │
        ┌───────────┴───────────┬──────────────┬──────────────┐
        ↓                       ↓              ↓              ↓
┌───────────────┐    ┌───────────────┐  ┌──────────┐  ┌──────────┐
│ GoogleProvider│    │  DeepLProvider│  │ Ollama   │  │ OpenAI   │
│               │    │               │  │ Provider │  │ Provider │
│ - Translate   │    │ - Translate   │  │          │  │          │
│ - Batch       │    │ - Batch       │  │ - Local  │  │ - GPT-4  │
│ - Cache       │    │ - Cache       │  │ - Free   │  │ - Claude │
└───────────────┘    └───────────────┘  └──────────┘  └──────────┘
        │                       │              │              │
        └───────────────────────┴──────────────┴──────────────┘
                                │
                   ┌────────────▼─────────────┐
                   │    TranslationCache      │
                   │    (SQLite, 30-day)      │
                   │                          │
                   │  - Reduces API costs     │
                   │  - Provider-specific     │
                   │  - SHA-256 cache keys    │
                   └──────────────────────────┘
</code></pre>

</div>



<p><strong>Key Benefits:</strong></p>

<ul>
<li>Add new providers without changing existing code</li>
<li>Each provider handles its own rate limiting</li>
<li>Centralized caching reduces costs by 80%+</li>
<li>Providers can be traditional (Google) or AI-powered (Claude)</li>
</ul>

<h3>
  
  
  Using the Translation System
</h3>

<p>Switch between providers and control translation behavior:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight shell"><code><span class="c"># Translate all missing keys</span>
lrm translate <span class="nt">--only-missing</span> <span class="nt">--provider</span> google

<span class="c"># Use specific provider</span>
lrm translate <span class="nt">--provider</span> ollama  <span class="c"># Free local translation</span>
lrm translate <span class="nt">--provider</span> deepl   <span class="c"># High quality</span>

<span class="c"># Target specific languages only</span>
lrm translate <span class="nt">--target-languages</span> fr,de,es <span class="nt">--provider</span> google

<span class="c"># Re-translate specific keys</span>
lrm translate <span class="s2">"Error*"</span> <span class="nt">--provider</span> deepl <span class="nt">--overwrite</span>

<span class="c"># Preview before saving</span>
lrm translate <span class="nt">--only-missing</span> <span class="nt">--dry-run</span>

<span class="c"># Batch control for rate limiting</span>
lrm translate <span class="nt">--batch-size</span> 5 <span class="nt">--provider</span> google
</code></pre>

</div>



<p><strong>Key options:</strong></p>

<ul>
<li>
<code>--provider</code> - google, deepl, openai, claude, ollama, libretranslate, azureopenai, azuretranslator</li>
<li>
<code>--only-missing</code> - Skip keys with existing translations</li>
<li>
<code>--target-languages</code> - Comma-separated language codes (fr,de,es)</li>
<li>
<code>--overwrite</code> - Allow re-translating existing values (use with key pattern)</li>
<li>
<code>--dry-run</code> - Preview without saving</li>
<li>
<code>--batch-size</code> - Control concurrent API calls</li>
<li>
<code>--no-cache</code> - Bypass the 30-day SQLite cache</li>
</ul>

<p><strong>Caching:</strong> All translations cached 30 days in <code>~/.local/share/LocalizationManager/translations.db</code>, reducing API costs by 80%+.</p>

<h3>
  
  
  The AI Translation Revolution
</h3>

<p>Here's something exciting: we added support for modern AI translation providers alongside traditional machine translation services.</p>

<p><strong>Provider Comparison:</strong></p>

<div class="table-wrapper-paragraph"><table>
<thead>
<tr>
<th>Provider</th>
<th>Type</th>
<th>Quality</th>
<th>Privacy</th>
<th>API Key Required</th>
</tr>
</thead>
<tbody>
<tr>
<td>Google Translate</td>
<td>Traditional MT</td>
<td>High</td>
<td>Cloud</td>
<td>Yes</td>
</tr>
<tr>
<td>DeepL</td>
<td>Traditional MT</td>
<td>Highest (EU)</td>
<td>Cloud</td>
<td>Yes</td>
</tr>
<tr>
<td>LibreTranslate</td>
<td>Traditional MT</td>
<td>Good</td>
<td>Self-host</td>
<td>Optional</td>
</tr>
<tr>
<td>OpenAI GPT</td>
<td>AI (LLM)</td>
<td>Excellent</td>
<td>Cloud</td>
<td>Yes</td>
</tr>
<tr>
<td>Claude</td>
<td>AI (LLM)</td>
<td>Excellent</td>
<td>Cloud</td>
<td>Yes</td>
</tr>
<tr>
<td>Ollama</td>
<td>AI (Local LLM)</td>
<td>Very Good</td>
<td><strong>Local</strong></td>
<td><strong>No</strong></td>
</tr>
<tr>
<td>Azure OpenAI</td>
<td>AI (LLM)</td>
<td>Excellent</td>
<td>Enterprise</td>
<td>Yes</td>
</tr>
</tbody>
</table></div>

<p><strong>Why AI Translation Matters:</strong></p>

<ol>
<li>
<strong>Context Awareness</strong>: LLMs understand context better than traditional MT</li>
<li>
<strong>Technical Terminology</strong>: Better at preserving technical terms and code snippets</li>
<li>
<strong>Tone Preservation</strong>: Maintains the original tone (formal, casual, technical)</li>
<li>
<strong>Free Option</strong>: Ollama runs locally for free with surprisingly good results</li>
<li>
<strong>Privacy</strong>: With Ollama, your data never leaves your machine</li>
</ol>

<h3>
  
  
  Code Scanning: Keeping Code and Resources in Sync
</h3>

<p>One unique feature is the code scanner that analyzes your source code to find localization key usage:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight shell"><code><span class="c"># Scan C#, Razor, XAML files for key usage</span>
lrm scan <span class="nt">--source-path</span> ./src <span class="nt">--format</span> json

<span class="c"># Output shows:</span>
<span class="c"># ✓ Scanned 245 files</span>
<span class="c"># Found 156 key references</span>
<span class="c"># ⚠ 3 keys used in code but missing from .resx</span>
<span class="c"># ⚠ 12 keys in .resx but never used</span>
</code></pre>

</div>



<p>The scanner detects patterns like:</p>

<ul>
<li><code>Resources.WelcomeMessage</code></li>
<li><code>GetString("SaveButton")</code></li>
<li><code>Translate("ErrorText")</code></li>
<li>
<code>{x:Static res:Strings.PageTitle}</code> (XAML)</li>
<li>
<code>@Resources.Message</code> (Razor)</li>
</ul>

<p>The scan command <strong>reports</strong> the issues - you then use other commands to fix them:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight shell"><code><span class="c"># 1. Scan and save results</span>
lrm scan <span class="nt">--format</span> json <span class="o">&gt;</span> scan-results.json

<span class="c"># 2. Add missing keys interactively (prompts for proper values)</span>
jq <span class="nt">-r</span> <span class="s1">'.missingKeys[]?.key // empty'</span> scan-results.json | <span class="k">while </span><span class="nb">read </span>key<span class="p">;</span> <span class="k">do
  </span>lrm add <span class="s2">"</span><span class="nv">$key</span><span class="s2">"</span> <span class="nt">-i</span>  <span class="c"># Interactive mode - prompts for values in each language</span>
<span class="k">done</span>

<span class="c"># OR: Add as placeholders for manual review later</span>
jq <span class="nt">-r</span> <span class="s1">'.missingKeys[]?.key // empty'</span> scan-results.json | <span class="k">while </span><span class="nb">read </span>key<span class="p">;</span> <span class="k">do
  </span>lrm add <span class="s2">"</span><span class="nv">$key</span><span class="s2">"</span> <span class="nt">--lang</span> default:<span class="s2">"</span><span class="nv">$key</span><span class="s2">"</span> <span class="nt">--comment</span> <span class="s2">"TODO: Add proper text (from code scan)"</span>
<span class="k">done</span>
<span class="c"># ⚠️ Important: Edit .resx files to add proper English text before running translate!</span>

<span class="c"># 3. After adding proper values, then translate</span>
lrm translate <span class="nt">--only-missing</span>
</code></pre>

</div>



<p>This helps prevent:</p>

<ul>
<li>Dead keys in resource files (unused keys)</li>
<li>Missing keys that will fail at runtime</li>
<li>Localization drift over time</li>
</ul>

<h2>
  
  
  The Complete Workflow: From Chaos to Automation
</h2>

<p>Here's the workflow that LocalizationManager enables, from manual pain to full automation:</p>

<h3>
  
  
  Before: The Manual Nightmare
</h3>



<div class="highlight js-code-highlight">
<pre class="highlight plaintext"><code>Developer adds English string
    ↓
Export .resx to Excel/CSV somehow
    ↓
Email to translator (wait 3 days)
    ↓
Receive translated Excel/CSV
    ↓
Manually copy-paste into .resx files
    ↓
Build fails (XML corruption)
    ↓
Fix XML manually
    ↓
Build succeeds, but French is missing 3 keys
    ↓
Repeat entire process for those 3 keys
    ↓
Week wasted
</code></pre>

</div>



<h3>
  
  
  After: The Automated Flow
</h3>



<div class="highlight js-code-highlight">
<pre class="highlight plaintext"><code>┌──────────────────────────────────────────────────────────────┐
│                                                              │
│  Step 1: VALIDATE                                            │
│  ├─ Check XML syntax                                         │
│  ├─ Find duplicate keys                                      │
│  └─ Detect empty values                                      │
│                                                              │
└────────────────┬─────────────────────────────────────────────┘
                 ↓
┌──────────────────────────────────────────────────────────────┐
│                                                              │
│  Step 2: SCAN CODE (optional)                                │
│  ├─ Find keys used in source files                           │
│  ├─ Detect missing keys (in code but not in .resx)           │
│  ├─ Detect unused keys (in .resx but not in code)            │
│  └─ Generate JSON report for processing                      │
│                                                              │
└────────────────┬─────────────────────────────────────────────┘
                 ↓
┌──────────────────────────────────────────────────────────────┐
│                                                              │
│  Step 2a: ADD MISSING KEYS (if scan found any)               │
│  ├─ Parse scan results JSON                                  │
│  ├─ Use 'lrm add' command for each missing key               │
│  └─ Keys now added to default .resx file                     │
│                                                              │
└────────────────┬─────────────────────────────────────────────┘
                 ↓
┌──────────────────────────────────────────────────────────────┐
│                                                              │
│  Step 3: CHECK MISSING                                       │
│  ├─ Find untranslated keys                                   │
│  ├─ Generate report per language                             │
│  └─ Count: 47 keys need translation                          │
│                                                              │
└────────────────┬─────────────────────────────────────────────┘
                 ↓
┌──────────────────────────────────────────────────────────────┐
│                                                              │
│  Step 4: AUTO-TRANSLATE                                      │
│  ├─ Batch translate missing keys                             │
│  ├─ Use cache for repeated strings                           │
│  ├─ Apply rate limiting                                      │
│  └─ Result: 47/47 translations completed                     │
│                                                              │
└────────────────┬─────────────────────────────────────────────┘
                 ↓
┌──────────────────────────────────────────────────────────────┐
│                                                              │
│  Step 5: RE-VALIDATE                                         │
│  ├─ Verify all XML is still valid                            │
│  ├─ Confirm no keys were corrupted                           │
│  └─ Status: All checks passed                                │
│                                                              │
└────────────────┬─────────────────────────────────────────────┘
                 ↓
┌──────────────────────────────────────────────────────────────┐
│                                                              │
│  Step 6: COMMIT &amp; PR                                         │
│  ├─ Git commit with detailed message                         │
│  ├─ Create pull request                                      │
│  └─ Time elapsed: 2 minutes                                  │
│                                                              │
└──────────────────────────────────────────────────────────────┘
</code></pre>

</div>



<h3>
  
  
  Running the Workflow: Five Ways
</h3>

<p>Choose the approach that matches your team's needs:</p>

<ol>
<li>
<strong>Manual CLI</strong> - Step-by-step learning and exploration</li>
<li>
<strong>Bash Script A</strong> - Auto-translate existing keys with proper values</li>
<li>
<strong>Bash Script B</strong> - Scan code and add placeholders for review</li>
<li>
<strong>Ollama (Free!)</strong> - Private local translation, zero cost 🆓</li>
<li>
<strong>Duplicate Cleanup</strong> - Merge or delete duplicate keys</li>
</ol>




<p><strong>1. Manual CLI (step by step):</strong><br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight shell"><code><span class="c"># Step 1: Validate</span>
lrm validate <span class="nt">--path</span> ./Resources

<span class="c"># Step 2: Check what's missing</span>
lrm validate <span class="nt">--missing-only</span> <span class="nt">--format</span> json <span class="o">&gt;</span> missing.json

<span class="c"># Step 3: Translate only what's missing</span>
lrm translate <span class="nt">--only-missing</span> <span class="nt">--provider</span> openai

<span class="c"># Step 4: Verify</span>
lrm validate
</code></pre>

</div>



<p><strong>2. Automated Script (Option A - Translate Existing Keys):</strong><br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight shell"><code><span class="c">#!/bin/bash</span>
<span class="c"># Workflow for auto-translating keys that already have proper English text</span>
<span class="nb">set</span> <span class="nt">-e</span>

<span class="c"># Validate</span>
lrm validate

<span class="c"># Translate missing translations</span>
lrm translate <span class="nt">--only-missing</span> <span class="nt">--provider</span> google

<span class="c"># Final validation</span>
lrm validate

<span class="c"># Commit and create PR</span>
git add Resources/
git commit <span class="nt">-m</span> <span class="s2">"🌐 Auto-translate missing localization keys"</span>
gh <span class="nb">pr </span>create <span class="nt">--title</span> <span class="s2">"Auto-translate localizations"</span>
</code></pre>

</div>



<p><strong>2b. Automated Script (Option B - Scan and Add Keys for Review):</strong><br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight shell"><code><span class="c">#!/bin/bash</span>
<span class="c"># Workflow for scanning code and adding placeholder keys that need manual review</span>
<span class="nb">set</span> <span class="nt">-e</span>

<span class="c"># Validate</span>
lrm validate

<span class="c"># Scan code and add missing keys</span>
lrm scan <span class="nt">--source-path</span> ./src <span class="nt">--format</span> json <span class="o">&gt;</span> scan-results.json
<span class="k">if</span> <span class="o">[</span> <span class="nt">-s</span> scan-results.json <span class="o">]</span><span class="p">;</span> <span class="k">then
  </span>jq <span class="nt">-r</span> <span class="s1">'.missingKeys[]?.key // empty'</span> scan-results.json | <span class="k">while </span><span class="nb">read </span>key<span class="p">;</span> <span class="k">do</span>
    <span class="o">[</span> <span class="nt">-n</span> <span class="s2">"</span><span class="nv">$key</span><span class="s2">"</span> <span class="o">]</span> <span class="o">&amp;&amp;</span> lrm add <span class="nt">--key</span> <span class="s2">"</span><span class="nv">$key</span><span class="s2">"</span> <span class="nt">--value</span> <span class="s2">"</span><span class="nv">$key</span><span class="s2">"</span> <span class="nt">--comment</span> <span class="s2">"TODO: Add proper English text (from code scan)"</span>
  <span class="k">done
fi</span>

<span class="c"># Final validation</span>
lrm validate

<span class="c"># Commit and create PR for manual review</span>
git add Resources/
git commit <span class="nt">-m</span> <span class="s2">"📋 Add placeholder keys from code scan - manual review required"</span>
gh <span class="nb">pr </span>create <span class="nt">--title</span> <span class="s2">"Review and add text for new localization keys"</span> <span class="nt">--body</span> <span class="s2">"⚠️ These keys were added with placeholder values. Please edit the .resx files to add proper English text before running auto-translation."</span>
</code></pre>

</div>



<blockquote>
<p><strong>⚠️ Important</strong>: Option B adds keys with placeholder values (key name as the value). Do NOT run <code>lrm translate</code> until after manually editing the .resx files to add proper English text. Otherwise, it will translate the placeholder "WelcomeMessage" literally instead of your intended English message.</p>
</blockquote>

<p><strong>3a. GitHub Actions - Translate Existing Keys:</strong><br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight yaml"><code><span class="na">name</span><span class="pi">:</span> <span class="s">Auto-Translate Existing Keys</span>

<span class="na">on</span><span class="pi">:</span>
  <span class="na">push</span><span class="pi">:</span>
    <span class="na">paths</span><span class="pi">:</span> <span class="pi">[</span><span class="s1">'</span><span class="s">Resources/**/*.resx'</span><span class="pi">]</span>  <span class="c1"># Only .resx changes</span>

<span class="na">jobs</span><span class="pi">:</span>
  <span class="na">translate</span><span class="pi">:</span>
    <span class="na">runs-on</span><span class="pi">:</span> <span class="s">ubuntu-latest</span>
    <span class="na">steps</span><span class="pi">:</span>
      <span class="pi">-</span> <span class="na">uses</span><span class="pi">:</span> <span class="s">actions/checkout@v4</span>

      <span class="pi">-</span> <span class="na">name</span><span class="pi">:</span> <span class="s">Setup LRM</span>
        <span class="na">run</span><span class="pi">:</span> <span class="pi">|</span>
          <span class="s">wget https://github.com/nickprotop/LocalizationManager/releases/latest/download/lrm-linux-x64.tar.gz</span>
          <span class="s">tar -xzf lrm-linux-x64.tar.gz</span>
          <span class="s">chmod +x linux-x64/lrm</span>
          <span class="s">echo "$PWD/linux-x64" &gt;&gt; $GITHUB_PATH</span>

      <span class="pi">-</span> <span class="na">name</span><span class="pi">:</span> <span class="s">Validate → Translate → Validate</span>
        <span class="na">env</span><span class="pi">:</span>
          <span class="na">LRM_OPENAI_API_KEY</span><span class="pi">:</span> <span class="s">${{ secrets.OPENAI_API_KEY }}</span>
        <span class="na">run</span><span class="pi">:</span> <span class="pi">|</span>
          <span class="s"># Validate</span>
          <span class="s">lrm validate</span>

          <span class="s"># Translate all missing translations</span>
          <span class="s">lrm translate --only-missing --provider openai</span>

          <span class="s"># Final validation</span>
          <span class="s">lrm validate</span>

      <span class="pi">-</span> <span class="na">name</span><span class="pi">:</span> <span class="s">Commit Changes</span>
        <span class="na">run</span><span class="pi">:</span> <span class="pi">|</span>
          <span class="s">git config user.name "github-actions[bot]"</span>
          <span class="s">git config user.email "github-actions[bot]@users.noreply.github.com"</span>
          <span class="s">git add Resources/</span>
          <span class="s">git commit -m "🌐 Auto-translate missing localization keys" || echo "No changes"</span>
          <span class="s">git push</span>
</code></pre>

</div>



<p><strong>3b. GitHub Actions - Scan and Add Placeholders:</strong><br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight yaml"><code><span class="na">name</span><span class="pi">:</span> <span class="s">Scan Code and Add Placeholders</span>

<span class="na">on</span><span class="pi">:</span>
  <span class="na">push</span><span class="pi">:</span>
    <span class="na">paths</span><span class="pi">:</span> <span class="pi">[</span><span class="s1">'</span><span class="s">src/**/*.cs'</span><span class="pi">]</span>  <span class="c1"># Only code changes</span>

<span class="na">jobs</span><span class="pi">:</span>
  <span class="na">scan</span><span class="pi">:</span>
    <span class="na">runs-on</span><span class="pi">:</span> <span class="s">ubuntu-latest</span>
    <span class="na">permissions</span><span class="pi">:</span>
      <span class="na">contents</span><span class="pi">:</span> <span class="s">write</span>
      <span class="na">pull-requests</span><span class="pi">:</span> <span class="s">write</span>
    <span class="na">steps</span><span class="pi">:</span>
      <span class="pi">-</span> <span class="na">uses</span><span class="pi">:</span> <span class="s">actions/checkout@v4</span>

      <span class="pi">-</span> <span class="na">name</span><span class="pi">:</span> <span class="s">Setup LRM</span>
        <span class="na">run</span><span class="pi">:</span> <span class="pi">|</span>
          <span class="s">wget https://github.com/nickprotop/LocalizationManager/releases/latest/download/lrm-linux-x64.tar.gz</span>
          <span class="s">tar -xzf lrm-linux-x64.tar.gz</span>
          <span class="s">chmod +x linux-x64/lrm</span>
          <span class="s">echo "$PWD/linux-x64" &gt;&gt; $GITHUB_PATH</span>

      <span class="pi">-</span> <span class="na">name</span><span class="pi">:</span> <span class="s">Scan → Add → Create PR</span>
        <span class="na">run</span><span class="pi">:</span> <span class="pi">|</span>
          <span class="s"># Validate</span>
          <span class="s">lrm validate</span>

          <span class="s"># Scan code for missing keys</span>
          <span class="s">lrm scan --source-path ./src --format json &gt; scan-results.json</span>

          <span class="s"># Add missing keys found in code</span>
          <span class="s">jq -r '.missingKeys[]?.key // empty' scan-results.json | while read key; do</span>
            <span class="s">if [ -n "$key" ]; then</span>
              <span class="s">lrm add --key "$key" --value "$key" --comment "TODO: Add proper English text (from code scan)"</span>
            <span class="s">fi</span>
          <span class="s">done</span>

          <span class="s"># Validate additions</span>
          <span class="s">lrm validate</span>

      <span class="pi">-</span> <span class="na">name</span><span class="pi">:</span> <span class="s">Create PR for Review</span>
        <span class="na">env</span><span class="pi">:</span>
          <span class="na">GH_TOKEN</span><span class="pi">:</span> <span class="s">${{ secrets.GITHUB_TOKEN }}</span>
        <span class="na">run</span><span class="pi">:</span> <span class="pi">|</span>
          <span class="s">git config user.name "github-actions[bot]"</span>
          <span class="s">git config user.email "github-actions[bot]@users.noreply.github.com"</span>
          <span class="s">git checkout -b scan/add-keys-$(date +%s)</span>
          <span class="s">git add Resources/</span>
          <span class="s">git commit -m "📋 Add placeholder keys from code scan" || exit 0</span>
          <span class="s">git push --set-upstream origin HEAD</span>

          <span class="s">gh pr create \</span>
            <span class="s">--title "Review: Add proper text for new localization keys" \</span>
            <span class="s">--body "⚠️ **Manual Review Required**</span>

          <span class="s">These keys were added with placeholder values (key name as value).</span>

          <span class="s">**Next steps:**</span>
          <span class="s">1. Edit the .resx files to replace placeholder values with proper English text</span>
          <span class="s">2. After merging, the auto-translate workflow will handle other languages"</span>
</code></pre>

</div>



<blockquote>
<p><strong>⚠️ Important</strong>: These are two separate workflows. Workflow 3a handles translation of keys that already have proper English text. Workflow 3b adds new keys found in code but does NOT translate them - it creates a PR for manual review first.</p>
</blockquote>

<p><strong>4. Privacy-First Local Translation (Zero Cost with Ollama):</strong></p>

<p>The best translation is free! Use Ollama to translate locally without cloud APIs:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight shell"><code><span class="c">#!/bin/bash</span>
<span class="c"># One-time setup: Install Ollama and pull a model</span>
curl <span class="nt">-fsSL</span> https://ollama.com/install.sh | sh
ollama pull llama3.2

<span class="c"># Validate resources</span>
lrm validate

<span class="c"># Translate locally - no cloud, no cost, no data leaving your machine</span>
lrm translate <span class="nt">--only-missing</span> <span class="nt">--provider</span> ollama

<span class="c"># Validate</span>
lrm validate
</code></pre>

</div>



<p><strong>Why Ollama is amazing:</strong></p>

<ul>
<li>💰 <strong>$0 cost</strong> - Unlimited translations forever</li>
<li>🔒 <strong>Complete privacy</strong> - Data never leaves your machine</li>
<li>✈️ <strong>Works offline</strong> - No internet required after model download</li>
<li>🔑 <strong>No API keys</strong> - One less secret to manage</li>
<li>🎯 <strong>Good quality</strong> - llama3.2 handles technical UI text well</li>
<li>⚖️ <strong>GDPR/HIPAA friendly</strong> - Perfect for sensitive data</li>
</ul>

<p><strong>Perfect for:</strong></p>

<ul>
<li>Medical records UI (HIPAA compliance)</li>
<li>Banking applications (PCI/SOX compliance)</li>
<li>Internal tools (security policies)</li>
<li>Bootstrapped projects (zero budget)</li>
<li>High-volume needs (no quota limits)</li>
</ul>

<p><strong>5. Clean Up Duplicate Keys (Merge or Delete):</strong></p>

<p>Resource files can accumulate duplicate keys from merges, imports, or accidents. LRM helps clean them up:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight shell"><code><span class="c">#!/bin/bash</span>
<span class="c"># Step 1: Detect duplicates</span>
lrm validate <span class="nt">--format</span> json <span class="o">&gt;</span> validation.json

<span class="nv">duplicate_count</span><span class="o">=</span><span class="si">$(</span>jq <span class="s1">'[.issues[]? | select(.type=="Duplicate")] | length'</span> validation.json<span class="si">)</span>

<span class="k">if</span> <span class="o">[</span> <span class="s2">"</span><span class="nv">$duplicate_count</span><span class="s2">"</span> <span class="nt">-gt</span> 0 <span class="o">]</span><span class="p">;</span> <span class="k">then
  </span><span class="nb">echo</span> <span class="s2">"⚠️  Found </span><span class="nv">$duplicate_count</span><span class="s2"> duplicate keys"</span>

  <span class="c"># Step 2: View duplicates</span>
  lrm validate  <span class="c"># Shows table with duplicate warnings</span>

  <span class="c"># Step 3: Choose your cleanup strategy</span>

  <span class="c"># Option A: Merge interactively (choose which value to keep per language)</span>
  lrm merge-duplicates ErrorMessage
  <span class="c"># Shows:</span>
  <span class="c"># ErrorMessage [1]: "An error occurred"</span>
  <span class="c"># ErrorMessage [2]: "Error occurred"</span>
  <span class="c"># Which to keep for default language? [1/2]: 1</span>
  <span class="c"># (repeats for each language)</span>

  <span class="c"># Option B: Auto-merge keeping first occurrence</span>
  lrm merge-duplicates ErrorMessage <span class="nt">--auto-first</span> <span class="nt">-y</span>

  <span class="c"># Option C: Merge ALL duplicates at once (keeps first of each)</span>
  lrm merge-duplicates <span class="nt">--all</span> <span class="nt">--auto-first</span> <span class="nt">-y</span>

  <span class="c"># Option D: Delete all occurrences entirely</span>
  lrm delete ErrorMessage <span class="nt">--all-duplicates</span> <span class="nt">-y</span>

  <span class="c"># Step 4: Validate cleanup</span>
  lrm validate
  <span class="nb">echo</span> <span class="s2">"✅ Duplicates cleaned up"</span>
<span class="k">fi</span>
</code></pre>

</div>



<p><strong>Common scenarios that create duplicates:</strong></p>

<ul>
<li>❌ Git merge conflicts in .resx files</li>
<li>❌ Importing CSV files with existing keys</li>
<li>❌ Manual XML editing mistakes</li>
<li>❌ Merging resource files from different branches</li>
</ul>

<p><strong>Interactive merge example:</strong><br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight shell"><code><span class="nv">$ </span>lrm merge-duplicates WelcomeMessage

Found 2 occurrences of <span class="s1">'WelcomeMessage'</span>:

Default language:
  <span class="o">[</span>1] <span class="s2">"Welcome to our app!"</span>
  <span class="o">[</span>2] <span class="s2">"Welcome to our application!"</span>

Which to keep? <span class="o">[</span>1/2]: 1

French <span class="o">(</span>fr<span class="o">)</span>:
  <span class="o">[</span>1] <span class="s2">"Bienvenue dans notre app !"</span>
  <span class="o">[</span>2] <span class="s2">"Bienvenue dans notre application !"</span>

Which to keep? <span class="o">[</span>1/2]: 2

✓ Merged WelcomeMessage: kept occurrence 1 <span class="k">for </span>default, occurrence 2 <span class="k">for </span>fr
</code></pre>

</div>



<p><strong>Pro tip:</strong> Always validate after cleanup to ensure no issues remain!</p>

<h2>
  
  
  Real-World Usage Examples
</h2>

<p>Let's see LocalizationManager in action with real examples.</p>

<h3>
  
  
  Installation (One Command)
</h3>



<div class="highlight js-code-highlight">
<pre class="highlight shell"><code><span class="c"># Linux/macOS</span>
curl <span class="nt">-sSL</span> https://raw.githubusercontent.com/nickprotop/LocalizationManager/main/install-lrm.sh | bash

<span class="c"># Windows (PowerShell)</span>
iwr <span class="nt">-useb</span> https://raw.githubusercontent.com/nickprotop/LocalizationManager/main/install-lrm.ps1 | iex
</code></pre>

</div>



<p>That's it. No dependencies, no .NET SDK required. Self-contained binary.</p>

<h3>
  
  
  Basic Workflow
</h3>



<div class="highlight js-code-highlight">
<pre class="highlight shell"><code><span class="c"># Navigate to your Resources folder</span>
<span class="nb">cd </span>MyApp/Resources

<span class="c"># Check current status</span>
lrm stats

<span class="c"># Output:</span>
<span class="c"># ┌──────────────────┬───────┬──────────┬───────────┐</span>
<span class="c"># │ Language         │ Keys  │ Coverage │ File Size │</span>
<span class="c"># ├──────────────────┼───────┼──────────┼───────────┤</span>
<span class="c"># │ English (en)     │ 252   │ 100.0%   │ 45.2 KB   │</span>
<span class="c"># │ French (fr)      │ 245   │  97.2%   │ 43.1 KB   │</span>
<span class="c"># │ German (de)      │ 238   │  94.4%   │ 41.8 KB   │</span>
<span class="c"># │ Greek (el)       │ 252   │ 100.0%   │ 46.3 KB   │</span>
<span class="c"># └──────────────────┴───────┴──────────┴───────────┘</span>

<span class="c"># Translate missing French and German keys</span>
lrm translate <span class="nt">--only-missing</span> <span class="nt">--target-languages</span> fr,de <span class="nt">--provider</span> openai

<span class="c"># Output:</span>
<span class="c"># 🌐 Translating 14 keys to 2 languages...</span>
<span class="c"># ✓ fr: 7/7 keys translated</span>
<span class="c"># ✓ de: 7/7 keys translated</span>
<span class="c"># ✓ Translations saved</span>
<span class="c"># ⚡ Used cache for 3 translations</span>
</code></pre>

</div>



<h3>
  
  
  Interactive TUI Mode
</h3>



<div class="highlight js-code-highlight">
<pre class="highlight shell"><code><span class="c"># Launch the Terminal UI</span>
lrm edit
</code></pre>

</div>





<div class="highlight js-code-highlight">
<pre class="highlight plaintext"><code>╔═══════════════════════════════════════════════════════════╗
║  LocalizationManager - Interactive Editor                 ║
╠═══════════════════════════════════════════════════════════╣
║                                                           ║
║  Key: WelcomeMessage                                      ║
║  ┌─────────────────────────────────────────────────────┐  ║
║  │ Default (en): Welcome to our application!           │  ║
║  │ French (fr):  Bienvenue dans notre application!     │  ║
║  │ German (de):  Willkommen in unserer Anwendung!      │  ║
║  │ Greek (el):   Καλώς ήρθατε στην εφαρμογή μας!       │  ║
║  └─────────────────────────────────────────────────────┘  ║
║                                                           ║
║  [Ctrl+T] Translate  [Ctrl+S] Save  [Ctrl+Q] Quit         ║
╚═══════════════════════════════════════════════════════════╝
</code></pre>

</div>



<h3>
  
  
  AI Translation Comparison
</h3>

<p>Let's translate a technical string with different providers:</p>

<p><strong>Source (English):</strong><br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight plaintext"><code>"Click the Save button to persist your changes to the database."
</code></pre>

</div>



<p><strong>Traditional MT (Google Translate) → French:</strong><br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight plaintext"><code>"Cliquez sur le bouton Enregistrer pour conserver vos modifications dans la base de données."
</code></pre>

</div>



<p><em>Accurate but mechanical</em></p>

<p><strong>AI (GPT-4o-mini) → French:</strong><br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight plaintext"><code>"Cliquez sur le bouton Enregistrer pour sauvegarder vos modifications dans la base de données."
</code></pre>

</div>



<p><em>More natural, preserves "Save" as "Enregistrer" (common UI term)</em></p>

<p><strong>AI (Claude) → French:</strong><br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight plaintext"><code>"Cliquez sur le bouton Enregistrer pour sauvegarder définitivement vos modifications dans la base de données."
</code></pre>

</div>



<p><em>Adds "définitivement" (permanently) - excellent context awareness</em></p>

<p><strong>Provider Features:</strong></p>

<ul>
<li>Traditional APIs: Consistent quality, proven reliability</li>
<li>AI (Cloud): Excellent context awareness, requires API key</li>
<li>
<strong>Ollama (Local): Free to use, runs on your machine, no API key needed</strong> ⭐</li>
</ul>

<h3>
  
  
  Configuration for AI Providers
</h3>

<p>Create <code>lrm.json</code> in your Resources folder:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight json"><code><span class="p">{</span><span class="w">
  </span><span class="nl">"defaultLanguageCode"</span><span class="p">:</span><span class="w"> </span><span class="s2">"en"</span><span class="p">,</span><span class="w">
  </span><span class="nl">"translation"</span><span class="p">:</span><span class="w"> </span><span class="p">{</span><span class="w">
    </span><span class="nl">"defaultProvider"</span><span class="p">:</span><span class="w"> </span><span class="s2">"openai"</span><span class="p">,</span><span class="w">
    </span><span class="nl">"aiProviders"</span><span class="p">:</span><span class="w"> </span><span class="p">{</span><span class="w">
      </span><span class="nl">"openai"</span><span class="p">:</span><span class="w"> </span><span class="p">{</span><span class="w">
        </span><span class="nl">"model"</span><span class="p">:</span><span class="w"> </span><span class="s2">"gpt-4o-mini"</span><span class="p">,</span><span class="w">
        </span><span class="nl">"customSystemPrompt"</span><span class="p">:</span><span class="w"> </span><span class="s2">"Translate UI text naturally, preserve technical terms"</span><span class="p">,</span><span class="w">
        </span><span class="nl">"rateLimitPerMinute"</span><span class="p">:</span><span class="w"> </span><span class="mi">60</span><span class="w">
      </span><span class="p">},</span><span class="w">
      </span><span class="nl">"ollama"</span><span class="p">:</span><span class="w"> </span><span class="p">{</span><span class="w">
        </span><span class="nl">"apiUrl"</span><span class="p">:</span><span class="w"> </span><span class="s2">"http://localhost:11434"</span><span class="p">,</span><span class="w">
        </span><span class="nl">"model"</span><span class="p">:</span><span class="w"> </span><span class="s2">"llama3.2"</span><span class="p">,</span><span class="w">
        </span><span class="nl">"rateLimitPerMinute"</span><span class="p">:</span><span class="w"> </span><span class="mi">10</span><span class="w">
      </span><span class="p">}</span><span class="w">
    </span><span class="p">}</span><span class="w">
  </span><span class="p">}</span><span class="w">
</span><span class="p">}</span><span class="w">
</span></code></pre>

</div>



<p>Set your API keys:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight shell"><code><span class="c"># Via environment variables (recommended)</span>
<span class="nb">export </span><span class="nv">LRM_OPENAI_API_KEY</span><span class="o">=</span><span class="s2">"sk-..."</span>
<span class="nb">export </span><span class="nv">LRM_CLAUDE_API_KEY</span><span class="o">=</span><span class="s2">"sk-ant-..."</span>

<span class="c"># Or via secure credential store</span>
lrm config set-api-key <span class="nt">--provider</span> openai <span class="nt">--key</span> <span class="s2">"sk-..."</span>

<span class="c"># Ollama needs no API key!</span>
</code></pre>

</div>



<h2>
  
  
  Architecture Highlights: Why It Works
</h2>

<h3>
  
  
  1. Self-Contained Binaries
</h3>

<p>Using .NET's publishing options, LocalizationManager compiles to a single, self-contained executable:</p>

<ul>
<li>No .NET runtime installation required</li>
<li>No dependencies to manage</li>
<li>~50MB binary includes everything</li>
<li>Fast startup (~100ms)</li>
</ul>

<h3>
  
  
  2. Cross-Platform Terminal UI
</h3>

<p>Spectre.Console makes the TUI work beautifully on all platforms:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight plaintext"><code>Windows Command Prompt  ✓
Windows Terminal       ✓
PowerShell            ✓
Linux Terminal        ✓
macOS Terminal        ✓
SSH Sessions          ✓
</code></pre>

</div>



<h3>
  
  
  3. JSON Output for Automation
</h3>

<p>Every command supports <code>--format json</code> for easy parsing:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight shell"><code>lrm validate <span class="nt">--format</span> json | jq <span class="s1">'.isValid'</span>
<span class="c"># true</span>

lrm stats <span class="nt">--format</span> json | jq <span class="s1">'.statistics[].coverage'</span>
<span class="c"># 100.0</span>
<span class="c"># 97.2</span>
<span class="c"># 94.4</span>
</code></pre>

</div>



<p>Perfect for:</p>

<ul>
<li>CI/CD pipelines</li>
<li>Custom dashboards</li>
<li>Monitoring and alerts</li>
<li>Integration with other tools</li>
</ul>

<h3>
  
  
  4. Intelligent Rate Limiting
</h3>

<p>Each translation provider has configurable rate limits to avoid hitting API quotas:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight csharp"><code><span class="c1">// Automatic rate limiting</span>
<span class="k">public</span> <span class="k">class</span> <span class="nc">RateLimiter</span>
<span class="p">{</span>
    <span class="k">private</span> <span class="k">readonly</span> <span class="kt">int</span> <span class="n">_requestsPerMinute</span><span class="p">;</span>

    <span class="k">public</span> <span class="k">async</span> <span class="n">Task</span> <span class="nf">WaitAsync</span><span class="p">()</span>
    <span class="p">{</span>
        <span class="c1">// Automatically throttles requests</span>
        <span class="c1">// Tracks request times in sliding window</span>
        <span class="c1">// Prevents API rate limit errors</span>
    <span class="p">}</span>
<span class="p">}</span>
</code></pre>

</div>



<h3>
  
  
  5. Batch Processing with Fallback
</h3>

<p>Translations are processed in batches when providers support it:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight csharp"><code><span class="c1">// Try batch translation first</span>
<span class="kt">var</span> <span class="n">batchResult</span> <span class="p">=</span> <span class="k">await</span> <span class="nf">TranslateBatchAsync</span><span class="p">(</span><span class="n">requests</span><span class="p">);</span>

<span class="c1">// Fall back to individual translation if batch fails</span>
<span class="k">if</span> <span class="p">(</span><span class="n">batchResult</span><span class="p">.</span><span class="n">HasErrors</span><span class="p">)</span>
<span class="p">{</span>
    <span class="k">foreach</span> <span class="p">(</span><span class="kt">var</span> <span class="n">request</span> <span class="k">in</span> <span class="n">failedRequests</span><span class="p">)</span>
    <span class="p">{</span>
        <span class="k">await</span> <span class="nf">TranslateAsync</span><span class="p">(</span><span class="n">request</span><span class="p">);</span>
    <span class="p">}</span>
<span class="p">}</span>
</code></pre>

</div>



<h2>
  
  
  Getting Started
</h2>

<h3>
  
  
  Try It Now
</h3>



<div class="highlight js-code-highlight">
<pre class="highlight shell"><code><span class="c"># Install</span>
curl <span class="nt">-sSL</span> https://raw.githubusercontent.com/nickprotop/LocalizationManager/main/install-lrm.sh | bash

<span class="c"># Go to your Resources folder</span>
<span class="nb">cd </span>YourProject/Resources

<span class="c"># Check status</span>
lrm stats

<span class="c"># Validate everything</span>
lrm validate

<span class="c"># Start interactive mode</span>
lrm edit
</code></pre>

</div>



<h3>
  
  
  Next Steps
</h3>

<ol>
<li>
<strong>Read the docs</strong>: <a href="https://github.com/nickprotop/LocalizationManager/docs" rel="noopener noreferrer">Full documentation</a>
</li>
<li>
<strong>Star on GitHub</strong>: <a href="https://github.com/nickprotop/LocalizationManager" rel="noopener noreferrer">GitHub Repository</a>
</li>
<li>
<strong>Try the workflow</strong>: Follow the Quick Start guide</li>
<li>
<strong>Set up CI/CD</strong>: Check out the <a href="https://github.com/nickprotop/LocalizationManager/docs/CICD.md" rel="noopener noreferrer">CI/CD examples</a>
</li>
<li>
<strong>Join the community</strong>: Report issues, request features, contribute!</li>
</ol>

<h3>
  
  
  Contributing
</h3>

<p>LocalizationManager is MIT licensed and welcomes contributions:</p>

<ul>
<li>🐛 <strong>Bug reports</strong>: Found an issue? Let us know!</li>
<li>✨ <strong>Feature requests</strong>: Have an idea? Open a discussion!</li>
<li>🔧 <strong>Pull requests</strong>: Code contributions welcome!</li>
<li>📖 <strong>Documentation</strong>: Help improve the docs!</li>
</ul>

<h2>
  
  
  Conclusion: Making .NET Localization Painless
</h2>

<p>LocalizationManager started as a personal itch I needed to scratch: managing .resx files on Linux without Windows tools. It evolved into a comprehensive solution that:</p>

<p>✅ Works on <strong>any platform</strong> (Linux, Windows, macOS, ARM64)<br>
✅ Automates the <strong>complete workflow</strong> (validate → translate → commit → PR)<br>
✅ Integrates <strong>modern AI translation</strong> (OpenAI, Claude, Ollama)<br>
✅ Smart caching reduces redundant API calls<br>
✅ Provides both <strong>CLI and TUI</strong> for automation and interactive use<br>
✅ Scans your <strong>source code</strong> to keep resources in sync<br>
✅ Integrates seamlessly with <strong>CI/CD pipelines</strong></p>

<p>Whether you're a solo developer managing translations for a side project or a team handling enterprise-scale localization, LocalizationManager provides the tools you need to work efficiently.</p>

<p><strong>The best part?</strong> It's completely free and open source. No subscriptions, no locked features, no vendor lock-in. Just a tool that works.</p>




<h2>
  
  
  Links &amp; Resources
</h2>

<ul>
<li>
<strong>GitHub Repository</strong>: <a href="https://github.com/nickprotop/LocalizationManager" rel="noopener noreferrer">https://github.com/nickprotop/LocalizationManager</a>
</li>
<li>
<strong>Documentation</strong>: <a href="https://github.com/nickprotop/LocalizationManager/tree/main/docs" rel="noopener noreferrer">https://github.com/nickprotop/LocalizationManager/tree/main/docs</a>
</li>
<li>
<strong>Installation Guide</strong>: <a href="https://github.com/nickprotop/LocalizationManager/docs/INSTALLATION.md" rel="noopener noreferrer">https://github.com/nickprotop/LocalizationManager/docs/INSTALLATION.md</a>
</li>
<li>
<strong>CI/CD Examples</strong>: <a href="https://github.com/nickprotop/LocalizationManager/docs/CICD.md" rel="noopener noreferrer">https://github.com/nickprotop/LocalizationManager/docs/CICD.md</a>
</li>
<li>
<strong>Translation Guide</strong>: <a href="https://github.com/nickprotop/LocalizationManager/docs/TRANSLATION.md" rel="noopener noreferrer">https://github.com/nickprotop/LocalizationManager/docs/TRANSLATION.md</a>
</li>
<li>
<strong>Issue Tracker</strong>: <a href="https://github.com/nickprotop/LocalizationManager/issues" rel="noopener noreferrer">https://github.com/nickprotop/LocalizationManager/issues</a>
</li>
</ul>




<p><em>Built with ❤️ by <a href="https://github.com/nickprotop" rel="noopener noreferrer">Nikolaos Protopapas</a></em><br>
<em>Licensed under MIT - Free to use, modify, and distribute</em></p>




<p><strong>Did you find this article helpful? Please share it with your fellow .NET developers!</strong></p>

<p><strong>Tags</strong>: #dotnet #devops #opensource #localization #ai #linux #translation #csharp #cli #automation</p>

