---
Title: Docker Compose - SFTP: Managing files securely
Description: 
Author: Alexandre Fernandes dos Santos
Date: 2026-01-25T21:29:40.000Z
Robots: noindex,nofollow
Template: index
---
<p>Good morning everyone! I hope this finds you well. Today I want to share with you almost a version 2.0 of my old article <a href="">Docker Compose - Server FTP</a>. This text was born from my desire to bring some security improvements and best practice tips.</p>

<p>Don't get me wrong, the previous text has its charm. If you want to spin up a basic and fast FTP server, that's your go-to.</p>

<p>Well, in case you don't know the <strong>SFTP</strong> protocol (<em>Secure File Transfer Protocol</em> or <em>SSH File Transfer Protocol</em>), it is a network file transfer protocol like FTP, but it uses SSH to encrypt commands and data (during transmission).</p>

<h2>
  
  
  Who can, can; who can't, won't
</h2>

<p>I advise creating a directory with two subdirectories: one for files and another to persist credentials.<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight plaintext"><code>sftp
|-- files
|-- creds
</code></pre>

</div>



<p>To avoid permission problems, we should guarantee the correct access rights to prevent issues when erasing or editing files created by the Docker container.</p>

<p>To do this, I advise finding out which user will be used and adding the directories to that user's group, as well as setting permissions via <code>chmod</code>.<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight plaintext"><code>Note: If the user who created the directories is the same one who will execute the docker container, this step might not be necessary, though it is recommended.
</code></pre>

</div>



<p>For simplicity's sake, I'll assume that the user being used in the shell is the same one running Docker, so let's go!</p>

<p>1 - Discover the current user using <code>whoami</code>:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight plaintext"><code>➜ whoami
alexandre
</code></pre>

</div>



<p>2 - In my case, the user is <em>alexandre</em>. Now let's check the user's ID:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight plaintext"><code>➜ id
uid=1000(alexandre) gid=1000(alexandre) grupos=1000(alexandre),4(adm),24(cdrom),27(sudo),30(dip),46(plugdev),100(users),116(lpadmin),987(docker)
</code></pre>

</div>



<p>3 - Do you see the <code>uid</code> value preceding our username in parentheses? The value inside it is what we want. In my case, it's <em>1000</em>. We'll need this when we create the users with access to SFTP.</p>

<p>4 - Add the necessary directories to the group using the <em>chown</em> command on the folder we are working in, with the user and group in the following format: <em>user:group</em>.<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight plaintext"><code>chown -R alexandre:alexandre ./sftp
</code></pre>

</div>



<p>5 - Adding permissions: the creator of the file and the group can <em>read</em>, <em>edit</em>, and <em>delete</em> data, while others can only execute read operations (774).<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight plaintext"><code>chmod 774 -R ./sftp
</code></pre>

</div>



<h2>
  
  
  Creating the users file
</h2>

<p>The application we are using today allows users to be passed via a file. I think this method is easier to manage than via command line or env vars, so let's create this file.</p>

<p>Just create a file named <em>users.conf</em> and add the user information in the following format <code>username:password:uid</code>. I will explain each field:</p>

<ul>
<li>
<strong>username:</strong> Can be any username, including your current user (my case).</li>
<li>
<strong>password:</strong> I will leave it blank because we'll use an <em>SSH</em> key to authenticate.</li>
<li>
<strong>uid:</strong> The UID we got in the previous step <em>(if you're in a hurry, try 1000 or 1001)</em>.</li>
</ul>

<p>My <strong>users.conf</strong> content will look like this:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight plaintext"><code>alexandre::1000
</code></pre>

</div>



<h2>
  
  
  Generating SSH keys
</h2>

<p>To generate the keys, I will use <em>ssh-keygen</em>, which usually comes installed on Ubuntu.</p>

<p>1 - Enter the <em>creds</em> directory inside <em>/sftp</em>.</p>

<p>2 - Use the command <strong>ssh-keygen -t rsa</strong> and type the filename. It can be any name; I used <em>key</em>.</p>

<p>3 - After that, it asks if you want to add a passphrase. It's optional. I usually don't set one, remembering that if added, it'll be requested upon every access.</p>

<h2>
  
  
  After the Odyssey, Ulysses returns home
</h2>

<p>Now for the best part. Here is the docker compose of happiness:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight plaintext"><code>services:
  sftp:
    image: atmoz/sftp
    restart: 'unless-stopped' 
    volumes:
        - ./files:/home/alexandre/upload                          # The files will be maintained here
        - ./creds/key.pub:/home/alexandre/.ssh/keys/key.pub:ro    # Sharing keys with the service
        - ./users.conf:/etc/sftp/users.conf:ro                    # Adding users
    ports:
        - 2222:22                                                 # The exposed port will be 2222
</code></pre>

</div>



<p>And to spin it up, you just need to execute:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight plaintext"><code>docker compose up -d
</code></pre>

</div>



<h2>
  
  
  How to connect
</h2>

<p>If you want a tip for a client to connect to your freshly created service, I recommend the classic FileZilla. See below how to use it:</p>

<p>1 - Open the <em>Site Manager</em> in the File tab:</p>

<p><a href="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fewm782fkbyn5ez2prpw4.png" class="article-body-image-wrapper"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fewm782fkbyn5ez2prpw4.png" alt="file/site manager" width="800" height="335"></a></p>

<p>2 - Follow the suggested config:</p>

<p><a href="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2F82vnhmlwce6fb74b8sp4.png" class="article-body-image-wrapper"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2F82vnhmlwce6fb74b8sp4.png" alt="site manager/config" width="800" height="342"></a></p>

<ul>
<li>
<strong>Protocol:</strong> SFTP</li>
<li>
<strong>Host:</strong> localhost</li>
<li>
<strong>Port:</strong> 2222 - Normally port 22 is used, but I redirected it to 2222 to avoid conflicts with other ports.</li>
<li>
<strong>Logon Type:</strong> Key file</li>
<li>
<strong>User:</strong> alexandre - Add the user from the <em>users.conf</em> file.</li>
<li>
<strong>Key file:</strong> Use the private key (without .pub) we created in the step [Generating SSH keys].</li>
<li>
<strong>Background color:</strong> It depends on your taste.</li>
</ul>

<p>3 - Voilá! Now just enjoy making your dubious deployments or keeping your "green steam" files on your favorite SFTP (a father's and mother's love knows no bounds).</p>

<h2>
  
  
  That's all, folks!
</h2>

<p>I hope this text is useful for you. It became a little longer than I expected, but I wanted to explain everything thoroughly (I hope I achieved that 😅).</p>

<p>However, if you have any questions, just leave a comment below and I'll be happy to answer them!</p>

