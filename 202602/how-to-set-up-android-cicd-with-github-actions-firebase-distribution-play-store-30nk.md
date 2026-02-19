---
Title: How to Set Up Android CI/CD with GitHub Actions — Firebase Distribution & Play Store
Description: 
Author: Aleksandr Ilinskiy
Date: 2026-02-19T21:35:24.000Z
Robots: noindex,nofollow
Template: index
---
<p>Setting up CI/CD for Android apps on GitHub Actions is straightforward once you know the gotchas. This guide covers everything: building signed APKs/AABs, caching Gradle, deploying to Firebase Distribution for testers, and publishing to Play Store.</p>

<h2>
  
  
  Prerequisites
</h2>

<ul>
<li>An Android project with Gradle (Groovy or Kotlin DSL)</li>
<li>A GitHub repository</li>
<li>For Firebase: a Firebase project with your app added</li>
<li>For Play Store: a Google Play Console account with your app set up</li>
</ul>

<h2>
  
  
  Step 1: Make Gradlew Executable
</h2>

<p>This trips up almost everyone on their first CI run. Your <code>gradlew</code> file might not be executable in Git:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight shell"><code>git update-index <span class="nt">--chmod</span><span class="o">=</span>+x gradlew
git commit <span class="nt">-m</span> <span class="s2">"Make gradlew executable"</span>
</code></pre>

</div>



<p>Or add this step to your workflow (we'll include it below).</p>

<h2>
  
  
  Step 2: Create a Signing Keystore
</h2>

<p>If you don't have one yet:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight shell"><code>keytool <span class="nt">-genkeypair</span> <span class="nt">-v</span> <span class="se">\</span>
  <span class="nt">-keystore</span> release.jks <span class="se">\</span>
  <span class="nt">-keyalg</span> RSA <span class="nt">-keysize</span> 2048 <span class="se">\</span>
  <span class="nt">-validity</span> 10000 <span class="se">\</span>
  <span class="nt">-alias</span> release <span class="se">\</span>
  <span class="nt">-storepass</span> YOUR_STORE_PASSWORD <span class="se">\</span>
  <span class="nt">-keypass</span> YOUR_KEY_PASSWORD <span class="se">\</span>
  <span class="nt">-dname</span> <span class="s2">"CN=Your Name, O=Your Org"</span>
</code></pre>

</div>



<p><strong>Never commit the keystore to Git.</strong> Instead, base64-encode it:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight shell"><code><span class="nb">base64</span> <span class="nt">-i</span> release.jks | pbcopy   <span class="c"># macOS</span>
<span class="nb">base64</span> <span class="nt">-w</span> 0 release.jks          <span class="c"># Linux</span>
</code></pre>

</div>



<h2>
  
  
  Step 3: Add GitHub Secrets
</h2>

<p>Go to your repo → <strong>Settings → Secrets and variables → Actions</strong> and create:</p>

<div class="table-wrapper-paragraph"><table>
<thead>
<tr>
<th>Secret</th>
<th>Value</th>
</tr>
</thead>
<tbody>
<tr>
<td><code>KEYSTORE_BASE64</code></td>
<td>Base64-encoded .jks file</td>
</tr>
<tr>
<td><code>KEYSTORE_PASSWORD</code></td>
<td>Your keystore password</td>
</tr>
<tr>
<td><code>KEY_ALIAS</code></td>
<td>Your key alias (e.g. "release")</td>
</tr>
<tr>
<td><code>KEY_PASSWORD</code></td>
<td>Your key password</td>
</tr>
</tbody>
</table></div>

<p>For <strong>Firebase Distribution</strong>, also add:</p>

<div class="table-wrapper-paragraph"><table>
<thead>
<tr>
<th>Secret</th>
<th>Value</th>
</tr>
</thead>
<tbody>
<tr>
<td><code>FIREBASE_APP_ID</code></td>
<td>Firebase Console → Project Settings → Your Android app ID</td>
</tr>
<tr>
<td><code>FIREBASE_SERVICE_ACCOUNT</code></td>
<td>JSON content of a Firebase service account key</td>
</tr>
</tbody>
</table></div>

<p>For <strong>Play Store</strong>, add:</p>

<div class="table-wrapper-paragraph"><table>
<thead>
<tr>
<th>Secret</th>
<th>Value</th>
</tr>
</thead>
<tbody>
<tr>
<td><code>PLAY_SERVICE_ACCOUNT_JSON</code></td>
<td>Google Play Console → API access → Service account JSON</td>
</tr>
</tbody>
</table></div>

<h2>
  
  
  Step 4: The Workflow — Firebase Distribution
</h2>

<p>Create <code>.github/workflows/android-firebase.yml</code>:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight yaml"><code><span class="na">name</span><span class="pi">:</span> <span class="s">Android — Build &amp; Deploy to Firebase</span>

<span class="na">on</span><span class="pi">:</span>
  <span class="na">push</span><span class="pi">:</span>
    <span class="na">branches</span><span class="pi">:</span> <span class="pi">[</span><span class="nv">main</span><span class="pi">]</span>

<span class="na">concurrency</span><span class="pi">:</span>
  <span class="na">group</span><span class="pi">:</span> <span class="s">${{ github.workflow }}-${{ github.ref }}</span>
  <span class="na">cancel-in-progress</span><span class="pi">:</span> <span class="kc">true</span>

<span class="na">jobs</span><span class="pi">:</span>
  <span class="na">build</span><span class="pi">:</span>
    <span class="na">name</span><span class="pi">:</span> <span class="s">Build &amp; Upload to Firebase</span>
    <span class="na">runs-on</span><span class="pi">:</span> <span class="s">ubuntu-latest</span>

    <span class="na">steps</span><span class="pi">:</span>
      <span class="pi">-</span> <span class="na">name</span><span class="pi">:</span> <span class="s">Checkout</span>
        <span class="na">uses</span><span class="pi">:</span> <span class="s">actions/checkout@v4</span>

      <span class="pi">-</span> <span class="na">name</span><span class="pi">:</span> <span class="s">Set up JDK </span><span class="m">17</span>
        <span class="na">uses</span><span class="pi">:</span> <span class="s">actions/setup-java@v4</span>
        <span class="na">with</span><span class="pi">:</span>
          <span class="na">java-version</span><span class="pi">:</span> <span class="s1">'</span><span class="s">17'</span>
          <span class="na">distribution</span><span class="pi">:</span> <span class="s1">'</span><span class="s">temurin'</span>

      <span class="pi">-</span> <span class="na">name</span><span class="pi">:</span> <span class="s">Cache Gradle</span>
        <span class="na">uses</span><span class="pi">:</span> <span class="s">actions/cache@v4</span>
        <span class="na">with</span><span class="pi">:</span>
          <span class="na">path</span><span class="pi">:</span> <span class="pi">|</span>
            <span class="s">~/.gradle/caches</span>
            <span class="s">~/.gradle/wrapper</span>
          <span class="na">key</span><span class="pi">:</span> <span class="s">${{ runner.os }}-gradle-${{ hashFiles('**/*.gradle*', '**/gradle-wrapper.properties') }}</span>
          <span class="na">restore-keys</span><span class="pi">:</span> <span class="pi">|</span>
            <span class="s">${{ runner.os }}-gradle-</span>

      <span class="pi">-</span> <span class="na">name</span><span class="pi">:</span> <span class="s">Make gradlew executable</span>
        <span class="na">run</span><span class="pi">:</span> <span class="s">chmod +x ./gradlew</span>

      <span class="pi">-</span> <span class="na">name</span><span class="pi">:</span> <span class="s">Decode Keystore</span>
        <span class="na">run</span><span class="pi">:</span> <span class="s">echo "${{ secrets.KEYSTORE_BASE64 }}" | base64 -d &gt; app/release.jks</span>

      <span class="pi">-</span> <span class="na">name</span><span class="pi">:</span> <span class="s">Build Release APK</span>
        <span class="na">run</span><span class="pi">:</span> <span class="s">./gradlew assembleRelease</span>
        <span class="na">env</span><span class="pi">:</span>
          <span class="na">KEYSTORE_PASSWORD</span><span class="pi">:</span> <span class="s">${{ secrets.KEYSTORE_PASSWORD }}</span>
          <span class="na">KEY_ALIAS</span><span class="pi">:</span> <span class="s">${{ secrets.KEY_ALIAS }}</span>
          <span class="na">KEY_PASSWORD</span><span class="pi">:</span> <span class="s">${{ secrets.KEY_PASSWORD }}</span>

      <span class="pi">-</span> <span class="na">name</span><span class="pi">:</span> <span class="s">Upload to Firebase Distribution</span>
        <span class="na">uses</span><span class="pi">:</span> <span class="s">wzieba/Firebase-Distribution-Github-Action@v1</span>
        <span class="na">with</span><span class="pi">:</span>
          <span class="na">appId</span><span class="pi">:</span> <span class="s">${{ secrets.FIREBASE_APP_ID }}</span>
          <span class="na">serviceCredentialsFileContent</span><span class="pi">:</span> <span class="s">${{ secrets.FIREBASE_SERVICE_ACCOUNT }}</span>
          <span class="na">groups</span><span class="pi">:</span> <span class="s">testers</span>
          <span class="na">file</span><span class="pi">:</span> <span class="s">app/build/outputs/apk/release/app-release.apk</span>
</code></pre>

</div>



<h2>
  
  
  Step 5: The Workflow — Play Store
</h2>

<p>For Play Store deployment, the workflow is similar but uses AAB (Android App Bundle) instead of APK:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight yaml"><code><span class="na">name</span><span class="pi">:</span> <span class="s">Android — Build &amp; Publish to Play Store</span>

<span class="na">on</span><span class="pi">:</span>
  <span class="na">push</span><span class="pi">:</span>
    <span class="na">tags</span><span class="pi">:</span>
      <span class="pi">-</span> <span class="s1">'</span><span class="s">v*'</span>

<span class="na">concurrency</span><span class="pi">:</span>
  <span class="na">group</span><span class="pi">:</span> <span class="s">${{ github.workflow }}-${{ github.ref }}</span>
  <span class="na">cancel-in-progress</span><span class="pi">:</span> <span class="kc">true</span>

<span class="na">jobs</span><span class="pi">:</span>
  <span class="na">build</span><span class="pi">:</span>
    <span class="na">name</span><span class="pi">:</span> <span class="s">Build &amp; Publish to Play Store</span>
    <span class="na">runs-on</span><span class="pi">:</span> <span class="s">ubuntu-latest</span>

    <span class="na">steps</span><span class="pi">:</span>
      <span class="pi">-</span> <span class="na">name</span><span class="pi">:</span> <span class="s">Checkout</span>
        <span class="na">uses</span><span class="pi">:</span> <span class="s">actions/checkout@v4</span>

      <span class="pi">-</span> <span class="na">name</span><span class="pi">:</span> <span class="s">Set up JDK </span><span class="m">17</span>
        <span class="na">uses</span><span class="pi">:</span> <span class="s">actions/setup-java@v4</span>
        <span class="na">with</span><span class="pi">:</span>
          <span class="na">java-version</span><span class="pi">:</span> <span class="s1">'</span><span class="s">17'</span>
          <span class="na">distribution</span><span class="pi">:</span> <span class="s1">'</span><span class="s">temurin'</span>

      <span class="pi">-</span> <span class="na">name</span><span class="pi">:</span> <span class="s">Cache Gradle</span>
        <span class="na">uses</span><span class="pi">:</span> <span class="s">actions/cache@v4</span>
        <span class="na">with</span><span class="pi">:</span>
          <span class="na">path</span><span class="pi">:</span> <span class="pi">|</span>
            <span class="s">~/.gradle/caches</span>
            <span class="s">~/.gradle/wrapper</span>
          <span class="na">key</span><span class="pi">:</span> <span class="s">${{ runner.os }}-gradle-${{ hashFiles('**/*.gradle*', '**/gradle-wrapper.properties') }}</span>
          <span class="na">restore-keys</span><span class="pi">:</span> <span class="pi">|</span>
            <span class="s">${{ runner.os }}-gradle-</span>

      <span class="pi">-</span> <span class="na">name</span><span class="pi">:</span> <span class="s">Make gradlew executable</span>
        <span class="na">run</span><span class="pi">:</span> <span class="s">chmod +x ./gradlew</span>

      <span class="pi">-</span> <span class="na">name</span><span class="pi">:</span> <span class="s">Decode Keystore</span>
        <span class="na">run</span><span class="pi">:</span> <span class="s">echo "${{ secrets.KEYSTORE_BASE64 }}" | base64 -d &gt; app/release.jks</span>

      <span class="pi">-</span> <span class="na">name</span><span class="pi">:</span> <span class="s">Build Release AAB</span>
        <span class="na">run</span><span class="pi">:</span> <span class="s">./gradlew bundleRelease</span>
        <span class="na">env</span><span class="pi">:</span>
          <span class="na">KEYSTORE_PASSWORD</span><span class="pi">:</span> <span class="s">${{ secrets.KEYSTORE_PASSWORD }}</span>
          <span class="na">KEY_ALIAS</span><span class="pi">:</span> <span class="s">${{ secrets.KEY_ALIAS }}</span>
          <span class="na">KEY_PASSWORD</span><span class="pi">:</span> <span class="s">${{ secrets.KEY_PASSWORD }}</span>

      <span class="pi">-</span> <span class="na">name</span><span class="pi">:</span> <span class="s">Upload to Play Store</span>
        <span class="na">uses</span><span class="pi">:</span> <span class="s">r0adkll/upload-google-play@v1</span>
        <span class="na">with</span><span class="pi">:</span>
          <span class="na">serviceAccountJsonPlainText</span><span class="pi">:</span> <span class="s">${{ secrets.PLAY_SERVICE_ACCOUNT_JSON }}</span>
          <span class="na">packageName</span><span class="pi">:</span> <span class="s">com.yourcompany.yourapp</span>
          <span class="na">releaseFiles</span><span class="pi">:</span> <span class="s">app/build/outputs/bundle/release/app-release.aab</span>
          <span class="na">track</span><span class="pi">:</span> <span class="s">internal</span>
</code></pre>

</div>



<p><strong>Note:</strong> Replace <code>com.yourcompany.yourapp</code> with your actual package name. The <code>track</code> can be <code>internal</code>, <code>alpha</code>, <code>beta</code>, or <code>production</code>.</p>

<h2>
  
  
  Step 6: Configure build.gradle for CI Signing
</h2>

<p>Your <code>app/build.gradle.kts</code> needs to read signing config from environment variables:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight kotlin"><code><span class="nf">android</span> <span class="p">{</span>
    <span class="nf">signingConfigs</span> <span class="p">{</span>
        <span class="nf">create</span><span class="p">(</span><span class="s">"release"</span><span class="p">)</span> <span class="p">{</span>
            <span class="n">storeFile</span> <span class="p">=</span> <span class="nf">file</span><span class="p">(</span><span class="s">"release.jks"</span><span class="p">)</span>
            <span class="n">storePassword</span> <span class="p">=</span> <span class="nc">System</span><span class="p">.</span><span class="nf">getenv</span><span class="p">(</span><span class="s">"KEYSTORE_PASSWORD"</span><span class="p">)</span>
            <span class="n">keyAlias</span> <span class="p">=</span> <span class="nc">System</span><span class="p">.</span><span class="nf">getenv</span><span class="p">(</span><span class="s">"KEY_ALIAS"</span><span class="p">)</span>
            <span class="n">keyPassword</span> <span class="p">=</span> <span class="nc">System</span><span class="p">.</span><span class="nf">getenv</span><span class="p">(</span><span class="s">"KEY_PASSWORD"</span><span class="p">)</span>
        <span class="p">}</span>
    <span class="p">}</span>

    <span class="nf">buildTypes</span> <span class="p">{</span>
        <span class="nf">release</span> <span class="p">{</span>
            <span class="n">signingConfig</span> <span class="p">=</span> <span class="n">signingConfigs</span><span class="p">.</span><span class="nf">getByName</span><span class="p">(</span><span class="s">"release"</span><span class="p">)</span>
            <span class="n">isMinifyEnabled</span> <span class="p">=</span> <span class="k">true</span>
            <span class="nf">proguardFiles</span><span class="p">(</span>
                <span class="nf">getDefaultProguardFile</span><span class="p">(</span><span class="s">"proguard-android-optimize.txt"</span><span class="p">),</span>
                <span class="s">"proguard-rules.pro"</span>
            <span class="p">)</span>
        <span class="p">}</span>
    <span class="p">}</span>
<span class="p">}</span>
</code></pre>

</div>



<h2>
  
  
  Common Issues
</h2>

<p><strong>"Permission denied: gradlew"</strong> — Either run <code>chmod +x ./gradlew</code> in your workflow or fix it in Git with <code>git update-index --chmod=+x gradlew</code>.</p>

<p><strong>"JAVA_HOME is not set"</strong> — Make sure the <code>setup-java</code> step comes before any Gradle commands.</p>

<p><strong>Build takes too long</strong> — The Gradle cache step should cut build times significantly after the first run. First build downloads all dependencies and can take 5-10 minutes.</p>

<p><strong>"No key with alias found in keystore"</strong> — Double-check your <code>KEY_ALIAS</code> secret matches exactly what you used when creating the keystore.</p>

<p><strong>"Failed to read key from store"</strong> — Your base64 encoding might be corrupted. Re-encode the keystore and update the secret.</p>

<h2>
  
  
  Pro Tips
</h2>

<p><strong>Concurrency groups</strong> prevent wasted CI minutes. If you push twice quickly, the first run cancels automatically.</p>

<p><strong>Tag-based triggers</strong> for Play Store are ideal — push a <code>v1.0.0</code> tag when you're ready to release, and CI handles the rest.</p>

<p><strong>Firebase Distribution</strong> is great for internal testing — testers get a notification with each new build, no Play Store review required.</p>

<h2>
  
  
  The Easy Way
</h2>

<p>Don't want to write YAML by hand? I built <a href="https://runlane.dev" rel="noopener noreferrer">Run Lane</a> — a free visual configurator that generates GitHub Actions workflows for Android and iOS. Pick your platform, choose your distribution target, and download a ready-to-use workflow. No account needed.</p>




<p><em>Tags: android, github, cicd, kotlin</em></p>

