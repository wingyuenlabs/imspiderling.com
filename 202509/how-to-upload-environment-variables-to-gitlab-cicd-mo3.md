---
Title: 🚀 How to Upload Environment Variables to GitLab CI/CD
Description: 
Author: Ruben Alvarado
Date: 2025-09-13T21:28:24.000Z
Robots: noindex,nofollow
Template: index
---
<p>Managing secrets and configuration in GitLab pipelines doesn’t have to be messy. Here’s a quick guide to securely upload your environment variables and integrate them into your .gitlab-ci.yml workflow.</p>

<ul>
<li><p>In Gitlab goto left side bar, Settings, CI/CD</p></li>
<li><p>Go to variables section, click add variable</p></li>
</ul>

<p><a href="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fbeta.appflowy.cloud%2Fapi%2Ffile_storage%2F67b91e9f-1e54-4861-9909-2038e29a9b3d%2Fv1%2Fblob%2F1b81d85c-ec6d-406f-9903-82cf598f9cb0%2Fgdp_i-zbG263nXSlHGFRk9JnEP5XlLNq4oyqo1TWa4k%3D." class="article-body-image-wrapper"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fbeta.appflowy.cloud%2Fapi%2Ffile_storage%2F67b91e9f-1e54-4861-9909-2038e29a9b3d%2Fv1%2Fblob%2F1b81d85c-ec6d-406f-9903-82cf598f9cb0%2Fgdp_i-zbG263nXSlHGFRk9JnEP5XlLNq4oyqo1TWa4k%3D." alt="Variables Menu" width="1249" height="297"></a></p>

<ul>
<li><p>From right side bar select select <code>visible</code> and deselect <code>protect variable</code><br>
<a href="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fa3v7irfjl2429f1v33ta.png" class="article-body-image-wrapper"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fa3v7irfjl2429f1v33ta.png" alt="Add Variable Menu" width="391" height="933"></a></p></li>
<li><p>In Key write the name of the file that will store all the variables</p></li>
<li><p>In value enter the key and values of your variables (values are written without ")</p></li>
<li><p>Make sure that your yml script adds the env vars file<br>
<code>.gitlab-ci.yml</code><br>
</p></li>
</ul>

<div class="highlight js-code-highlight">
<pre class="highlight plaintext"><code>Autotest:
  stage: test
  image: node:22
  before_script:
    - cat "$ENV_VARS" | tr -d '\r' &gt; .env
</code></pre>

</div>



<p>You’ve now securely injected environment variables into your GitLab CI/CD pipeline. No more hardcoding secrets or juggling config files—just clean, maintainable automation.</p>

