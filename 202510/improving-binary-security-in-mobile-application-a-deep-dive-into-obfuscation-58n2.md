---
Title: Improving Binary Security in Mobile Application: A Deep Dive into Obfuscation
Description: 
Author: Yusuff Jokanola O.
Date: 2025-10-17T21:13:23.000Z
Robots: noindex,nofollow
Template: index
---
<h2>
  
  
  Introduction
</h2>

<p>Mobile applications increasingly serve as the gateway to critical business operations, making them high-value targets for reverse engineering and code tampering. This threat is formally recognized in the <strong><a href="https://owasp.org/www-project-mobile-top-10/" rel="noopener noreferrer">OWASP Mobile Top 10 (2024) risks</a></strong>. This category highlights <strong><a href="https://owasp.org/www-project-mobile-top-10/2023-risks/m7-insufficient-binary-protection.html" rel="noopener noreferrer">M7: Insufficient Binary Protection</a></strong> enable attackers to extract secrets, reverse engineer proprietary logic, and repackage apps for malicious use.</p>

<p>In the context of React Native, this threat is especially relevant because the application’s core logic is bundled in JavaScript and then compiled into Hermes bytecode for performance optimization. While the use of Hermes offers a foundational layer of obfuscation through bytecode compilation, it is not a foolproof defense. Its output can still be decompiled or analyzed by determined adversaries (though advanced analysis techniques are beyond the scope of this article). Crucially, React Native apps also include several native files (Java/Kotlin on Android) and metadata that can reveal insights about the app’s internal structure, such as all the third-party packages in use. This information, when combined with the decompiled Hermes bytecode, can help attackers better understand the app’s architecture, identify potential weaknesses, and target specific vulnerabilities.</p>

<p>In this article, we’ll explore how Android reverse engineering affects React Native applications, why it remains a top-tier security concern, and outline key mitigation strategies—with a focus on implementing code obfuscation as an essential defense layer.</p>

<h3>
  
  
  Prerequisites for Analysis
</h3>

<p>To practically understand and test the security posture of your application against reverse engineering, you must have the production-ready Android Package Kit (APK) file.</p>

<h3>
  
  
  For React Native (Bare Workflow)
</h3>

<p>You will typically generate the APK directly using Android Studio or the command line.</p>

<h3>
  
  
  For Expo (Managed Workflow)
</h3>

<p>The .apk file is produced after a release build (eas build --platform android). If you receive an Android App Bundle (.aab) file from your build service or Google Play Console, you must use the Google-provided bundletool to extract a Universal or device-specific .apk for analysis.</p>

<blockquote>
<p>bundletool can be used to Converts .aab to .apk files for testing.</p>
</blockquote>

<h3>
  
  
  How Attackers/CyberSecurity Analyst Decompile Your App
</h3>

<p>Disclaimer: This section is for educational purposes only, intended to help developers understand and mitigate attack vectors. The techniques described are mimic standard practices used by security analysts and penetration testers to assess application security.</p>

<p>The standard process for reverse engineering an Android application involves several key steps, primarily centered around a tool called Apktool.</p>

<h3>
  
  
  The Decompilation Process
</h3>

<p>Apktool is the go-to utility for unpacking and repacking Android APKs. It extracts the resources and decompiles the compiled code into a more readable format called Smali.</p>

<p><strong>APK Acquisition</strong>: The attacker obtains the APK from an app store or directly from a compromised device.</p>

<p><strong>Decompilation Command</strong>: After installing apktool run below command in your preferred terminal</p>

<p><a href="https://apktool.org/docs/install/" rel="noopener noreferrer">Apktool Installation</a><br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight plaintext"><code>apktool d app-release.apk
</code></pre>

</div>



<p>This creates a new directory containing the decompiled files.</p>

<p><strong>Code Inspection</strong>: The attacker then navigates and analyzes the resulting files, which include the app's native code and resource structure.</p>

<h2>
  
  
  Anatomy of a Decompiled App (Before Obfuscation)
</h2>

<p>In an unprotected (or minimally protected) React Native or Expo application, the directory structure and file contents offer clear signposts to an attacker.</p>

<p>Example Decompiled app directory</p>

<p><a href="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2F97voiv3bv2zimat5a9ji.png" class="article-body-image-wrapper"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2F97voiv3bv2zimat5a9ji.png" alt="Decompiled App with Minimal Obfuscation" width="555" height="837"></a></p>

<p>smali/ directory Contains .smali files, which are a human-readable assembly-like language derived from the app's native Java/Kotlin bytecode.   Proprietary Logic: Function names, variable names, class structures, and API endpoint references are often intact and descriptive as shown in the image above.</p>

<h2>
  
  
  How to Improve Obfuscation in Expo Apps
</h2>

<p>For React Native apps, particularly those built with the Expo Managed Workflow, you can leverage the powerful built-in tooling for advanced binary protection. The key is to enable and fine-tune ProGuard (or R8, the modern equivalent) through the expo-build-properties plugin in your app.json or app.config.js.</p>

<h2>
  
  
  Configuration Steps
</h2>

<p>By default, Expo enables basic resource shrinking. For stronger protection, you must explicitly enable code obfuscation and resource compression for release builds, you can customize your app.config.js or app.json, by add expo-build-properties in your plugins<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight plaintext"><code>[
  "expo-build-properties",
  {
    "android": {
      // 1. **Crucial:** Enables R8/ProGuard for code obfuscation and shrinking.
      "enableProguardInReleaseBuilds": true,

      // 2. Optimizes the size of the final APK by compressing bundled assets.
      "enableBundleCompression": true, 

      // 3. Removes unused resources (like images or unused strings) to shrink size.
      "enableShrinkResourcesInReleaseBuilds": true,

      // 4. Advanced: Allows custom ProGuard rules for fine-tuning obfuscation.
      "extraProguardRules": "-dontwarn com.squareup.okhttp3.**\n-keep class com.myapp.secure.** { *; }"
    }
  }
],
</code></pre>

</div>



<p><strong>Implementing extraProguardRules</strong><br>
<strong>The extraProguardRules</strong> property is powerful for fine-tuning the <br>
protection: You can write specific rules to further obfuscate proprietary code or to prevent obfuscation on specific native modules that rely on introspection (reflection) to function correctly.</p>

<p>Best Practice: Always use a -keep rule for classes that are accessed dynamically (e.g., via reflection, JNI, or third-party SDKs) to prevent R8/ProGuard from renaming or stripping them, which would cause runtime crashes.</p>

<h2>
  
  
  Anatomy of a Decompiled App (After Obfuscation)
</h2>

<p>After implementing robust obfuscation, the same directories and files become significantly less useful to the attacker, drastically increasing the time and effort required for reverse engineering.</p>

<p><a href="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fheyzq6i3fhsfqc1w3058.png" class="article-body-image-wrapper"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fheyzq6i3fhsfqc1w3058.png" alt="Decompiled App with Custom Obfuscation" width="800" height="352"></a></p>

<p>After implementing robust obfuscation, the same directories and files become significantly less useful to the attacker, drastically increasing the time and effort required for reverse engineering.</p>

<p>Directory/File  Post-Obfuscation State  Impact on Attacker<br>
smali/ directory File and function names are replaced with meaningless, single-letter or short, randomized strings (e.g., a1.smali, b.smali). <br>
Deterrence: The attacker loses context. A file named a.smali provides no clues about its function, forcing manual, tedious analysis of the bytecode's flow and content.</p>

<p><strong>NB</strong><br>
⚠️ <strong>Warning and Testing</strong>: You must thoroughly test your application after enabling or tightening obfuscation. Since some native packages (especially those using reflection) may be affected by renamed classes, rigorous regression testing across all functionalities is non-negotiable before deploying the secure build.</p>

<h2>
  
  
  Final Thoughts
</h2>

<p>While basic, default obfuscation provides a minimal barrier, it may not deter a determined attacker targeting a high-value application. As the threat landscape evolves, and with attackers increasingly leveraging <strong>Large Language Models (LLMs)</strong> to accelerate the decoding of default binary encoding provided by frameworks, developers must respond with sufficient binary protection.</p>

<p>Obfuscation is a layer of defense, not a complete solution. When combined with other critical controls—such as <strong>Root/Jailbreak Detection, Runtime Application Self-Protection (RASP), Certificate Pinning, and proper Secret Management</strong>—a comprehensive strategy will significantly increase the cost and effort required for a successful reverse engineering attack, stretching the attacker's resources to their limit. For serious, commercially-sensitive projects, insufficient binary protection is a risk you simply cannot afford.</p>

