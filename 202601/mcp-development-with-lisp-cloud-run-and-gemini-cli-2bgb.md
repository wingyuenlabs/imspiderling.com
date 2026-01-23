---
Title: MCP Development with Lisp, Cloud Run, and Gemini CLI
Description: 
Author: xbill
Date: 2026-01-23T21:39:41.000Z
Robots: noindex,nofollow
Template: index
---
<p>Leveraging Gemini CLI and the underlying Gemini LLM to build Model Context Protocol (MCP) AI applications with the Lisp language deployed to Google Cloud Run.</p>

<p><a href="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2F3dotnelyy3ibddhatrzv.jpeg" class="article-body-image-wrapper"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2F3dotnelyy3ibddhatrzv.jpeg" width="800" height="763"></a></p>

<h4>
  
  
  Why not just use Python?
</h4>

<p>Python has traditionally been the main coding language for ML and AI tools. One of the strengths of the MCP protocol is that the actual implementation details are independent of the development language. The reality is that not every project is coded in Python- and MCP allows you to use the latest AI approaches with other coding languages.</p>

<h4>
  
  
  LISP? That Alien Programming Language works with MCP?
</h4>

<p>The goal of this article is to provide a minimal viable basic working MCP HTTP transport server in LISP that can be run locally without any unneeded extra code or extensions.</p>

<h4>
  
  
  Not a fan of functional programming?
</h4>

<p>It takes all kinds. The bottom line is different strokes for different folks and the tools can meet you where you are. The Internet has gotten over the Emacs vs Vi flame wars.</p>

<h4>
  
  
  Lisp Native MCP Library
</h4>

<p>There are several available Lisp libraries in GitHub. The Lisp MCP library used in this demo/article is here:</p>

<p><a href="https://github.com/40ants/mcp" rel="noopener noreferrer">GitHub - 40ants/mcp: 40ANTS-MCP is a framework for building Model Context Protocol servers in Common Lisp</a></p>

<h4>
  
  
  What Is Lisp?
</h4>

<p>Lisp (historically LISP) is a family of programming languages that is the <strong>second-oldest high-level language still in widespread use</strong> (after Fortran), known for its unique syntax where both code and data are represented as lists. Its name is an acronym for “LISt Processing”.</p>

<h4>
  
  
  Official Common Lisp Site
</h4>

<p>There are a few sites with Lisp resources. This site is a good starting point:</p>

<p><a href="https://lisp-lang.org/" rel="noopener noreferrer">Common Lisp</a></p>

<h4>
  
  
  Installing Lisp
</h4>

<p>Lisp comes with a whole eco-system including tooling, utilities, and build management.</p>

<p>There are a few common interpreters. For ease of installation — the Steel Bank Common Lisp variant was used:</p>

<p><a href="https://www.sbcl.org/" rel="noopener noreferrer">Steel Bank Common Lisp</a></p>

<p>This tool comes standard in most Linux distributions. For a basic Debian 12 installation:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight plaintext"><code>sudo apt-get update
sudo apt-get install sbcl
</code></pre>

</div>



<p>Then check the installation:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight plaintext"><code>xbill@penguin:~$ sbcl --version
SBCL 2.2.9.debian
</code></pre>

</div>



<h4>
  
  
  Lisp Eco-System
</h4>

<p>The main components of the Lisp eco-system include:</p>

<p><strong>ASDF</strong></p>

<p>ASDF provides a basic build manager for Lisp:</p>

<p><a href="https://asdf.common-lisp.dev/" rel="noopener noreferrer">ASDF - Another System Definition Facility</a></p>

<p><strong>Quicklisp</strong></p>

<p>Quicklisp allows for automated setup and configuration for your Lisp packages:</p>

<p><a href="https://www.quicklisp.org/beta/" rel="noopener noreferrer">Quicklisp beta</a></p>

<h4>
  
  
  Gemini CLI
</h4>

<p>If not pre-installed you can download the Gemini CLI to interact with the source files and provide real-time assistance:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight plaintext"><code>npm install -g @google/gemini-cli
</code></pre>

</div>



<h4>
  
  
  Testing the Gemini CLI Environment
</h4>

<p>Once you have all the tools and the correct Node.js version in place- you can test the startup of Gemini CLI. You will need to authenticate with a Key or your Google Account:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight plaintext"><code>gemini
</code></pre>

</div>



<p><a href="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fjf9fz2qgv4bicut1fo3m.png" class="article-body-image-wrapper"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fjf9fz2qgv4bicut1fo3m.png" width="800" height="450"></a></p>

<h4>
  
  
  Node Version Management
</h4>

<p>Gemini CLI needs a consistent, up to date version of Node. The <strong>nvm</strong> command can be used to get a standard Node environment:</p>

<p><a href="https://github.com/nvm-sh/nvm" rel="noopener noreferrer">GitHub - nvm-sh/nvm: Node Version Manager - POSIX-compliant bash script to manage multiple active node.js versions</a></p>

<h4>
  
  
  Lisp MCP Documentation
</h4>

<p>The official MCP Lisp page provides samples and documentation for getting started:</p>

<p><a href="https://github.com/40ants/mcp/tree/master/docs" rel="noopener noreferrer">mcp/docs at master · 40ants/mcp</a></p>

<h4>
  
  
  Where do I start?
</h4>

<p>The strategy for starting MCP development is a incremental step by step approach.</p>

<p>First, the basic development environment is setup with the required system variables, and a working Gemini CLI configuration.</p>

<p>Then, a minimal Hello World Style Lisp MCP Server is built with HTTP transport. This server is validated with Gemini CLI in the local environment.</p>

<p>This setup validates the connection from Gemini CLI to the local process via MCP. The MCP client (Gemini CLI) and the MCP server both run in the same local environment.</p>

<p>Next- the basic MCP server is Containerized and deployed to Google Cloud Run. This remote MCP installation is validated with the local Gemini CLI running as a MCP client.</p>

<h4>
  
  
  Setup the Basic Environment
</h4>

<p>At this point you should have a working C build environment, Lisp tools, and a working Gemini CLI installation. The next step is to clone the GitHub samples repository with support scripts:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight plaintext"><code>cd ~
git clone https://github.com/xbill9/gemini-cli-codeassist
</code></pre>

</div>



<p>Then run <strong>init.sh</strong> from the cloned directory.</p>

<p>The script will attempt to determine your shell environment and set the correct variables:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight plaintext"><code>cd gemini-cli-codeassist
source init.sh
</code></pre>

</div>



<p>If your session times out or you need to re-authenticate- you can run the <strong>set_env.sh</strong> script to reset your environment variables:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight plaintext"><code>cd gemini-cli-codeassist
source set_env.sh
</code></pre>

</div>



<p>Variables like PROJECT_ID need to be setup for use in the various build scripts- so the set_env script can be used to reset the environment if you time-out.</p>

<h4>
  
  
  Hello World with HTTP Transport
</h4>

<p>One of the key features that the standard MCP libraries provide is abstracting various transport methods.</p>

<p>The high level MCP tool implementation is the same no matter what low level transport channel/method that the MCP Client uses to connect to a MCP Server.</p>

<p>The simplest transport that the SDK supports is the stdio (stdio/stdout) transport — which connects a locally running process. Both the MCP client and MCP Server must be running in the same environment.</p>

<p>The HTTP transport allows the MCP client and server to be on the same machine or distributed over the Internet.</p>

<p>The connection over HTTP will look similar to this:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight plaintext"><code>(in-package :40ants-mcp/http-transport)

(setf *sse-handler* (clack-sse:serve-sse 'sse-stream-writer))

(defmethod 40ants-mcp/transport/base:start-loop ((transport http-transport) message-handler)
  "Start the HTTP server and begin processing requests.
   Monkey-patched to listen on 0.0.0.0 for Cloud Run."
  (log:info "Starting HTTP transport on port" (transport-port transport))
  (setf (transport-message-handler transport) message-handler)

  ;; Start the server
  (setf (transport-server transport)
        (clack:clackup (transport-lack-app transport)
                       :server :hunchentoot
                       :address "0.0.0.0"
                       :port (transport-port transport)
                       :use-thread nil)))
</code></pre>

</div>



<h4>
  
  
  Lisp Package Information
</h4>

<p>The code depends on several standard libraries for MCP and logging:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight plaintext"><code>(defpackage :mcp-server
  (:use :cl)
  (:import-from :40ants-mcp/tools
                :define-tool)
  (:import-from :40ants-mcp/server/definition
                :start-server
                :mcp-server)
  (:export :main
           :get-greeting))
</code></pre>

</div>



<h4>
  
  
  Installing and Running the Code
</h4>

<p>Run the install make release target on the local system:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight plaintext"><code>mxbill@penguin:~/gemini-cli-codeassist/mcp-https-lisp$ make deps
2 dists to check.
You already have the latest version of "quicklisp": 2026-01-01.
You already have the latest version of "ultralisp": 20260118004500.
To load "jsonrpc":
  Load 1 ASDF system:
    jsonrpc
; Loading "jsonrpc"
.
To load "40ants-mcp":
  Load 1 ASDF system:
    40ants-mcp
; Loading "40ants-mcp"
..................................................
......
To load "serapeum":
  Load 1 ASDF system:
    serapeum
; Loading "serapeum"

To load "yason":
  Load 1 ASDF system:
    yason
; Loading "yason"

To load "local-time":
  Load 1 ASDF system:
    local-time
; Loading "local-time"

Dependencies installed successfully.
</code></pre>

</div>



<p>A binary is generated at the end of the process:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight plaintext"><code>xbill@penguin:~/gemini-cli-codeassist/mcp-https-lisp$ make
Building mcp-server...
To load "mcp-https-lisp":
  Load 1 ASDF system:
    mcp-https-lisp
; Loading "mcp-https-lisp"
..................................................
..................................................
[package mcp-server]...
xbill@penguin:~/gemini-cli-codeassist/mcp-https-lisp$ 
</code></pre>

</div>



<p>To test the code:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight plaintext"><code>xbill@penguin:~/gemini-cli-codeassist/mcp-https-lisp$ make test
To load "mcp-https-lisp/tests":
  Load 1 ASDF system:
    mcp-https-lisp/tests
; Loading "mcp-https-lisp/tests"
..................................................
..................................................
[package mcp-server-tests].

Testing System mcp-https-lisp/tests

;; testing 'mcp-server-tests'
test-greeting
  should return a valid greeting
    ✓ Expect (STRING= (GET-GREETING "World") "Hello, World!") to be true.
    ✓ Expect (STRING= (GET-GREETING "Lisp") "Hello, Lisp!") to be true.

✓ 1 test completed

Summary:
  All 1 test passed.
</code></pre>

</div>



<h4>
  
  
  Start the Local MCP Server
</h4>

<p>Open a terminal window and kick off the local MCP server:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight plaintext"><code>xbill@penguin:~/gemini-cli-codeassist/mcp-https-lisp$ make run
To load "mcp-https-lisp":
  Load 1 ASDF system:
    mcp-https-lisp
; Loading "mcp-https-lisp"
..................................................
........
{"timestamp":"2026-1-20T17:59:41.348940EST","level":"INFO","message":"Starting MCP server on port 8080..."}
 &lt;INFO&gt; [17:59:41] 40ants-mcp/server/definition definition.lisp (start-server) -
  Starting MCP server with 40ANTS-MCP/SERVER/DEFINITION::TRANSPORT: :HTTP
  transport and tools 40ANTS-MCP/SERVER/DEFINITION::TOOLS-COLLECTIONS: #&lt;MCP-SERVER::USER-TOOLS {1006C7A393}&gt;

 &lt;INFO&gt; [17:59:41] 40ants-mcp/http-transport main.lisp (start-loop http-transport) -
  Starting HTTP transport on port (40ANTS-MCP/HTTP-TRANSPORT:TRANSPORT-PORT
                                   40ANTS-MCP/HTTP-TRANSPORT::TRANSPORT): 8080

Hunchentoot server is going to start.
Listening on 0.0.0.0:8080.
NOTICE: Running in debug mode. Debugger will be invoked on errors.
  Specify ':debug nil' to turn it off on remote environments.
</code></pre>

</div>



<h4>
  
  
  Gemini CLI Settings for the Local HTTP server
</h4>

<p>The sample .gemini/settings.json.https has been pre-built with a local connection:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight plaintext"><code>{
  "mcpServers": {
    "mcp-https-lisp": {
      "url": "http://127.0.0.1:8080/mcp"
    }
  }
}
</code></pre>

</div>



<h4>
  
  
  MCP Client Connection Test
</h4>

<p>Next- open another window and start Gemini CLI. The local MCP connection over HTTP to the Lisp Code is validated and the full Gemini CLI session will start:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight plaintext"><code>&gt; /mcp list

Configured MCP servers:

🟢 mcp-https-lisp - Ready (2 tools)
  Tools:
  - add
  - greet

&gt; greet Weird Alien!

✦ I will call the greet tool for you.
╭──────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────╮
│ ✓ greet (mcp-https-lisp MCP Server) {"param":"Weird Alien!"} │
│ │
│ Hello, Weird Alien!! │
╰──────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────╯
✦ Hello, Weird Alien!!

&gt; sum the first two perfect numbers

✦ I will use the add tool to sum the first two perfect numbers (6 and 28).
╭──────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────╮
│ ✓ add (mcp-https-lisp MCP Server) {"b":28,"a":6} │
│ │
│ The sum of 6 and 28 is: 34 │
╰──────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────╯
✦ The sum of the first two perfect numbers (6 and 28) is 34.
</code></pre>

</div>



<p>And the Key Question:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight plaintext"><code>&gt; can I determine if all your bases belong to us via symbolic lisp programming?

✦ Yes, in Common Lisp, the ownership of bases can be symbolically verified with a simple predicate.

   1 (defun verify-ownership (bases)
   2 (declare (ignore bases))
   3 (format t "All your base are belong to us."))
   4 
   5 ;; Usage
   6 (verify-ownership 'all)

</code></pre>

</div>



<h4>
  
  
  Deploying to Cloud Run
</h4>

<p>After the HTTP version of the MCP server has been tested locally — it can be deployed remotely to Google Cloud Run.</p>

<p>First- switch to the directory with the HTTP MCP sample code:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight plaintext"><code>cd ~/gemini-cli-codeassist/mcp-https-lisp
</code></pre>

</div>



<p>Deploy the project to Google Cloud Run with the pre-built <strong>cloudbuild.yaml</strong> and <strong>Dockerfile:</strong><br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight plaintext"><code>cd ~/gemini-cli-codeassist/mcp-https-lisp
xbill@penguin:~/gemini-cli-codeassist/mcp-https-lisp$ make deploy
</code></pre>

</div>



<p>The Cloud Build will start:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight plaintext"><code>xbill@penguin:~/gemini-cli-codeassist/mcp-https-lisp$ make deploy
echo "Submitting build to Google Cloud Build..."
Submitting build to Google Cloud Build...
gcloud builds submit . --config cloudbuild.yaml
Creating temporary archive of 25 file(s) totalling 85.1 KiB before compression.
Some files were not included in the source upload.

Check the gcloud log [/home/xbill/.config/gcloud/logs/2026.01.20/18.18.42.784862.log] to see which files and the contents of the
default gcloudignore file used (see `$ gcloud topic gcloudignore` to learn
more).

Uploading tarball of [.] to [gs://comglitn_cloudbuild/source/1768951123.007661-372db85df2d14bd5b216df10248748fe.tgz]
Created [https://cloudbuild.googleapis.com/v1/projects/comglitn/locations/global/builds/28781173-a7bc-4de3-a9b5-3b5e04fb128b].
Logs are available at [https://console.cloud.google.com/cloud-build/builds/28781173-a7bc-4de3-a9b5-3b5e04fb128b?project=1056842563084].
Waiting for build to complete. Polling interval: 1 second(s).
</code></pre>

</div>



<p><strong>It can take 15–30 minutes to complete the build.</strong></p>

<p>The cloud build needs to pull in all the Lisp libraries in the build environment and generate the entire package from scratch.</p>

<p>When the build is complete- an endpoint will be returned:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight plaintext"><code>Starting Step #1
Step #1: Already have image (with digest): gcr.io/cloud-builders/gcloud
Step #1: Deploying container to Cloud Run service [mcp-https-lisp] in project [comglitn] region [us-central1]
Step #1: Deploying...
Step #1: Setting IAM Policy...........done
Step #1: Creating Revision........................................................................................done
Step #1: Routing traffic.....done
Step #1: Done.
Step #1: Service [mcp-https-lisp] revision [mcp-https-lisp-00006-wrj] has been deployed and is serving 100 percent of traffic.
Step #1: Service URL: https://mcp-https-lisp-1056842563084.us-central1.run.app
Finished Step #1
PUSH │
</code></pre>

</div>



<p>The service endpoint in this example is :<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight plaintext"><code>https://mcp-https-lisp-1056842563084.us-central1.run.app
</code></pre>

</div>



<p>The actual endpoint will vary based on your project settings.</p>

<h4>
  
  
  Review The Service in Cloud Run
</h4>

<p>Navigate to the Google Cloud console and search for Cloud Run -</p>

<p><a href="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Frr8y43nja7dm32qnk87m.png" class="article-body-image-wrapper"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Frr8y43nja7dm32qnk87m.png" width="800" height="450"></a></p>

<p>and then you can detailed information on the Cloud Run Service:</p>

<p><a href="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fyxsgjjxnxscp5ogr1u8d.png" class="article-body-image-wrapper"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fyxsgjjxnxscp5ogr1u8d.png" width="800" height="450"></a></p>

<h4>
  
  
  Cloud Logging
</h4>

<p>The remote server writes logs to <strong>stderr</strong> in standard JSON format. These logs are available from the deployed Cloud Run Service:</p>

<p><a href="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fhgbg6f5qxutd6i2wbzgi.png" class="article-body-image-wrapper"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fhgbg6f5qxutd6i2wbzgi.png" width="800" height="450"></a></p>

<h4>
  
  
  Validate HTTP connection
</h4>

<p>Once you have the Endpoint — you can attempt a connection- navigate to in your browser:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight plaintext"><code>https://mcp-https-lisp-1056842563084.us-central1.run.app
</code></pre>

</div>



<p>You will need to adjust the exact URL to match the URL returned from Cloud Build.</p>

<p>You will get an error- this connection is expecting a message in the MCP format:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight plaintext"><code>Page Not Found
</code></pre>

</div>



<h4>
  
  
  Gemini CLI settings.json.cloudrun
</h4>

<p>Replace the default Gemini CLI configuration file —  <strong>settings.json</strong> with a pre-configured sample- <strong>settings.json.cloudrun</strong> to use the Cloud Run version of the connection:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight plaintext"><code>{
  "mcpServers": {
    "mcp-cloudrun-lisp": {
      "url": "https://mcp-https-lisp-1056842563084.us-central1.run.app/mcp"
    }
  }
}
</code></pre>

</div>



<p>Copy the Cloud Run version of the Gemini CLI configuration file:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight plaintext"><code>xbill@penguin:~/gemini-cli-codeassist/mcp-https-lisp$ cd .gemini
cp settings.json.cloudrun settings.json
xbill@penguin:~/gemini-cli-codeassist/mcp-https-lisp/.gemini$
</code></pre>

</div>



<h4>
  
  
  Validation with Gemini CLI
</h4>

<p>The final connection test uses Gemini CLI as a MCP client with the deployed Cloud Run Service in C providing the MCP server. Startup Gemini CLI with the updated settings :<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight plaintext"><code>gemini
&gt; /mcp list

Configured MCP servers:

🟢 mcp-cloudrun-lisp - Ready (2 tools)
  Tools:
  - add
  - greet

&gt; greet List Processor in the Cloud

✦ I will call the greet tool to welcome you.

╭──────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────╮
│ ✓ greet (mcp-cloudrun-lisp MCP Server) {"param":"List Processor in the Cloud"} │
│ │
│ Hello, List Processor in the Cloud! │
╰──────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────╯
✦ Greetings! I'm ready to help you with your Common Lisp MCP server project. What's our first task?

&gt; sum the first 2 taxicab numbers

✦ I will calculate the sum of the first two taxicab numbers (2 and 1729) using the add tool.

╭──────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────╮
│ ✓ add (mcp-cloudrun-lisp MCP Server) {"b":1729,"a":2} │
│ │
│ The sum of 2 and 1729 is: 1731 │
╰──────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────╯
✦ The sum of the first two taxicab numbers ($Ta(1)=2$ and $Ta(2)=1729$) is 1731.
</code></pre>

</div>



<p>You can also validate the deployed code base:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight plaintext"><code>&gt; review the project

✦ The project review is complete. Here is a summary of the mcp-https-lisp project:

  Project Overview
  This is a Common Lisp MCP (Model Context Protocol) server designed to expose tools over HTTP using Server-Sent Events (SSE). It is built to
  run in a containerized environment (like Google Cloud Run).

  Key Components
   * Core Logic (`src/main.lisp`):
       * Implements the server entry point (main).
       * Registers two example tools: greet and add.
       * Monkey Patches: Contains significant "monkey patches" for 40ants-mcp, openrpc-server, and clack-sse libraries. these patches fix
         JSON-RPC field casing (e.g., isError, inputSchema), handle SSE headers correctly, and ensure the server binds to 0.0.0.0.
   * Logging (`src/logger.lisp`): Uses structured JSON logging (log-json), which is excellent for cloud observability.
   * Build System:
       * `mcp-https-lisp.asd`: Defines the system and dependencies (40ants-mcp, clack, yason, str, etc.).
       * `Makefile`: manages dependencies, builds the binary, and runs tests.
       * `Dockerfile`: Multistage build using fukamachi/sbcl and debian:bookworm-slim to create a deployment-ready image.
   * Tests: Minimal test coverage in tests/main.lisp (checks a basic greeting).

  Observations &amp; Recommendations
   1. Heavy Patching: The project relies on modifying external library behavior at runtime (monkey patching) to meet MCP specifications. This
      suggests the underlying libraries might need updates. Long-term goal: Upstream these fixes.
   2. Test Coverage: Tests are currently very basic. Adding integration tests that simulate MCP tool calls would ensure stability.
   3. Deployment Ready: The Dockerfile and cloudbuild.yaml indicate this is ready for Google Cloud deployment.
</code></pre>

</div>



<h4>
  
  
  Summary
</h4>

<p>MCP development with Lisp using Gemini CLI was validated with an incremental step by step approach.</p>

<p>A minimal streaming HTTP transport MCP Server was built from source code and validated with Gemini CLI running as a MCP client in the same local environment.</p>

<p>Then — the MCP server was wrapped in a container and submitted to Google Cloud Build for deployment to Google Cloud Run. The remote MCP server was validated with a standard browser, and Gemini CLI.</p>

<p>Finally- remote MCP operations were performed from the local Gemini CLI installation to the Lisp MCP server hosted in Google Cloud Run.</p>

<p>This approach can be extended to more complex deployments and Cloud based options.</p>

