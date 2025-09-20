---
Title: Setting Up Kubernetes on Windows with Minikube (Step-by-Step Guide)
Description: 
Author: Abhishek Korde
Date: 2025-09-20T22:00:25.000Z
Robots: noindex,nofollow
Template: index
---
<p><strong>🌀 Prerequisites: Kubernetes, Docker, Containers, Cloud</strong><br>
There are many tools for managing the cluster in kubernetes:</p>

<ul>
<li>Minikube → Local dev cluster (single-node).</li>
<li>Kind (Kubernetes in Docker) → Lightweight, fast local clusters in Docker containers.</li>
<li>k3s → Lightweight Kubernetes distribution for edge/IoT.</li>
<li>Kops → Production cluster provisioning (mostly AWS).</li>
<li>Kubadm → Official tool to bootstrap a Kubernetes cluster (often used for bare-metal or custom setup).</li>
<li>Rancher → GUI and management for multiple Kubernetes clusters.</li>
<li>OpenShift → Red Hat’s enterprise Kubernetes platform.</li>
<li>Managed services → EKS (AWS), GKE (Google Cloud), AKS (Azure).</li>
</ul>

<p>Some of them are used for production, while others are mainly for local development.<br>
Today, we are using minikub(obviously) for our local development.</p>

<p><strong>Step 1: Install Minikube</strong><br>
The first step is to install Minikube using the link below, or by searching “Minikube install” in your browser.<br>
link of installing the minikube:<a href="https://minikube.sigs.k8s.io/docs/start/?arch=%2Fwindows%2Fx86-64%2Fstable%2F.exe+download" rel="noopener noreferrer">Minikube Install</a></p>



<p><strong>Step 2: Install kubectl</strong><br>
The next step is to install kubectl. kubectl is the command-line tool for Kubernetes, and it allows us to interact directly with the cluster.<br>
To install kubectl you can use below link or directly search in browser as <em>kubectl install</em>.<br>
link for installing the kubectl:<a href="https://kubernetes.io/docs/tasks/tools/" rel="noopener noreferrer">Kubectl Install</a><br>
Choose the installation instructions for your platform (Linux, macOS, or Windows).</p>



<p><strong>Step 3: Install Docker Desktop</strong></p>

<ul>
<li>Minikube runs the Kubernetes cluster inside a Docker container (when you use the --driver=docker).</li>
<li>You can install Docker Desktop either from the official website (browser download) or from the Microsoft Store.</li>
<li>Docker Desktop is a free platform (but not fully open source).</li>
<li>If your system does not support Hyper-V, you can use WSL2 as the backend for Docker.</li>
<li>In Docker Desktop → Settings → Resources → WSL Integration → Clicking “Refetch distros” helps Docker detect available Linux distributions.</li>
<li>If no distro is listed, you need to install a Linux distro like Ubuntu from the Microsoft Store (or manually). 
<img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Faoixos5q6fpohdz01svk.png" alt=" " width="800" height="420">
</li>
</ul>



<p><strong>Step 4: Start Minikube</strong><br>
Now that the basic setup is complete, we can create the Kubernetes cluster using the command below:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight plaintext"><code>minikube start
</code></pre>

</div>



<p>This creates a single-node Kubernetes cluster.<br>
On Windows, instead of VirtualBox/Hyper-V, it runs inside Docker containers.</p>

<p>⚠️ Sometimes, Minikube fails because it pulls images from registry.k8s.io. Use a faster mirror:<br>
In this case, use the following command:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight plaintext"><code>minikube start --driver=docker --image-repository=registry.cn-hangzhou.aliyuncs.com/google_containers

</code></pre>

</div>



<p>This flag tells Minikube to use the Alibaba Cloud mirror instead, which is often faster and more reliable.</p>




<p><strong>Step 5: Verify the Cluster</strong><br>
To list all Pods in the cluster (including system Pods), use the command:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight plaintext"><code>kubectl get pods -A
</code></pre>

</div>



<p>It will show like this.<br>
<a href="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fjijpjw0otuegtc4qhx90.png" class="article-body-image-wrapper"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fjijpjw0otuegtc4qhx90.png" alt=" " width="800" height="183"></a><br>
These are the default system pods that Minikube runs inside the kube-system namespace:</p>

<ul>
<li>coredns → Handles DNS (service discovery inside the cluster).</li>
<li>etcd-minikube → Key-value database that stores the entire Kubernetes cluster state.</li>
<li>kube-apiserver-minikube → The main API server; all kubectl commands talk to this.</li>
<li>kube-controller-manager-minikube → Ensures desired state (like creating pods if one fails).</li>
<li>kube-proxy → Manages networking rules for pod-to-pod and pod-to-service communication.</li>
<li>kube-scheduler → Assigns new pods to nodes.</li>
<li>storage-provisioner → Automatically provisions storage volumes when needed.</li>
</ul>

<p>✅ All are in Running status → means your Minikube cluster is healthy and ready to run workloads.</p>

<p>To get all the nodes in your cluster and their status using below command:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight plaintext"><code>kubectl get nodes
</code></pre>

</div>



<p>The output of the above command looks like this:<br>
<a href="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2F8ry78ydy0hz52u3xo06q.png" class="article-body-image-wrapper"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2F8ry78ydy0hz52u3xo06q.png" alt=" " width="662" height="77"></a></p>

<ul>
<li>NAME → minikube → This is your Kubernetes node (a VM/Container created by Minikube).</li>
<li>STATUS → Ready → The node is healthy and can run pods.</li>
<li>ROLES → control-plane → This node acts as the master/control plane (manages the cluster).</li>
<li>AGE → 5m3s → Node has been running for about 5 minutes.</li>
<li>VERSION → v1.34.0 → The Kubernetes version running on this node.
✅ In short → Your Minikube cluster has 1 control-plane node, it’s healthy, and ready to schedule pods.
After confirming that our nodes are up and running.</li>
</ul>



<p><strong>Step 6: What is a Pod?</strong><br>
Now we have to install pod. Inside that pod our application will run.</p>

<p>First, let’s understand what a Pod is.?</p>

<ul>
<li>A Pod is the smallest deployable unit in Kubernetes.</li>
<li>A Pod is essentially a definition of how to run one or more containers.</li>
<li>The specification is written in a YAML manifest (like pod.yml), similar to how you give instructions in docker run.</li>
<li>Most Pods contain a single container, but sometimes Pods have:</li>
<li>Sidecar containers (for logging, monitoring, etc.).</li>
<li>Init containers (run before the main container).</li>
<li>Kubernetes assigns an IP address to the Pod, not to each individual container. All containers in the Pod share the same Pod IP and network namespace.</li>
</ul>



<p><strong>Step 7: Deploy Your First Application (Nginx)</strong><br>
We will now deploy our first Nginx application. The first step is to create a Pod by writing a pod.yml file.<br>
As I told earlier pod is basically yaml file so we will write first basic yaml file that will help to understand the structure.<br>
If you are on Windows, use the following command and paste the given YAML script:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight plaintext"><code>notepad pod.yml
</code></pre>

</div>



<p>because vi does not work on Windows (it is available only on Linux systems).<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight plaintext"><code>apiVersion: v1
kind: Pod
metadata:
  name: nginx
spec:
  containers:
  - name: nginx
    image: nginx:1.14.2
    ports:
    - containerPort: 80

</code></pre>

</div>



<p>🔎 In short:</p>

<p>You’re creating a Pod called nginx.<br>
Inside it, you run 1 container using the image nginx:1.14.2.<br>
That container exposes port 80 (default HTTP port).<br>
✅ This YAML defines the smallest deployable unit in Kubernetes (a Pod) running Nginx.<br>
After writing yml file we have to create application on pod using give command:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight plaintext"><code>kubectl create -f pod.yml
</code></pre>

</div>



<p>After creating the application, check whether it is running using the following command:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight plaintext"><code>kubectl get pods
</code></pre>

</div>



<p>and If we want to get entire details of pods with IP address of pod then use below command:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight plaintext"><code>kubectl get pods -o wide
</code></pre>

</div>



<p><a href="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2F43yohdd561vuk25l7prc.png" class="article-body-image-wrapper"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2F43yohdd561vuk25l7prc.png" alt=" " width="800" height="204"></a></p>




<p><strong>Step 8: Access the Minikube Cluster</strong><br>
If you want to log in to the cluster where the application is running, use the command:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight plaintext"><code>minikube ssh
</code></pre>

</div>



<ul>
<li>It opens an SSH session into the VM / Docker container that Minikube created to run your Kubernetes cluster.</li>
<li>This lets you log in directly inside the Minikube node (the machine acting as your Kubernetes cluster).</li>
</ul>

<p>🛠 What you can do inside:</p>

<ul>
<li>Run Linux commands (because Minikube runs on a Linux environment).</li>
<li>Check processes, logs, or system info.</li>
<li>Use tools like docker ps (to see the containers Kubernetes is running internally).</li>
<li>Troubleshoot networking or storage inside the cluster.
<img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fwa1d1wjzonqs0fthwiv0.png" alt=" " width="516" height="55">
After that, test if the Pod’s internal IP is reachable and serving traffic by using curl .
</li>
</ul>

<div class="highlight js-code-highlight">
<pre class="highlight plaintext"><code>curl &lt;POD_IP&gt;
</code></pre>

</div>



<p>This ip address of pod where your application is running.<br>
<a href="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2F5lz3keisbypp8yexzwdk.png" class="article-body-image-wrapper"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2F5lz3keisbypp8yexzwdk.png" alt=" " width="800" height="575"></a><br>
If you see the Nginx welcome page, 🎉 congratulations—your app is running successfully!</p>




<p><strong>✅ Conclusion</strong><br>
You now have:</p>

<ul>
<li>Minikube installed locally.</li>
<li>kubectl configured to interact with your cluster.</li>
<li>A working Nginx Pod running in Kubernetes.
This setup is perfect for learning and local development before moving on to production tools like Kops, EKS, GKE, or AKS.</li>
</ul>

<p>✨ Polished version:</p>

<p>To explore all commonly used kubectl commands, refer to the official Kubernetes cheatsheet:<br>
<a href="https://kubernetes.io/pt-br/docs/reference/kubectl/cheatsheet/" rel="noopener noreferrer">Kubectl sheet sheet</a></p>

