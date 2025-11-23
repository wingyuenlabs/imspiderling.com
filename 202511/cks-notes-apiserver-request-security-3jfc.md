---
Title: CKS Notes - Apiserver request security
Description: 
Author: Cheedge Lee
Date: 2025-11-23T21:17:54.000Z
Robots: noindex,nofollow
Template: index
---
<h1>
  
  
  1. Identity = kubeconfig
</h1>

<p>Here the identity equals kubeconfig file</p>

<p>Every kubeconfig file specifies:</p>

<ul>
<li><p>the <strong>certificate</strong></p></li>
<li><p>the <strong>client key</strong></p></li>
<li><p>the <strong>username (CN)</strong></p></li>
<li><p>the <strong>group (O)</strong></p></li>
<li><p>the <strong>cluster endpoint</strong></p></li>
</ul>

<p>This determines:</p>

<ul>
<li><p><strong>Who you are</strong> (identity)</p></li>
<li><p><strong>What you can do</strong> (RBAC permissions)</p></li>
</ul>

<p>Now let’s first distinguish 2 types of identity: <strong>admin cluster identity</strong> vs <strong>node identity</strong>.</p>

<h2>
  
  
  1.1 kubelet.conf
</h2>

<p>Location:</p>

<p>each nodes <code>/etc/kubernetes/kubelet.conf</code></p>

<p>Identity inside it:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight shell"><code>user <span class="o">=</span> system:node:&lt;nodeName&gt;
group <span class="o">=</span> system:nodes
</code></pre>

</div>



<p>Meaning:</p>

<ul>
<li>This identity belongs to <strong>the kubelet on that node</strong>
</li>
</ul>

<p>Permissions (through RBAC):</p>

<ul>
<li><p>can update its <strong>own</strong> node object</p></li>
<li><p>can update <strong>pod status</strong> for pods assigned to it</p></li>
<li><p>cannot <code>get</code>/<code>delete</code>/<code>list</code> arbitrary pods</p></li>
<li><p>cannot label other nodes</p></li>
</ul>

<p><strong><em>This identity is heavily restricted by NodeRestriction.</em></strong></p>




<h2>
  
  
  1.2 <code>$HOME/.kube/config</code> <strong>(</strong><code>admin.conf</code> <strong>copy)</strong>
</h2>

<p>Location:</p>

<p>on control plane node</p>

<p>Identity:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight shell"><code>user <span class="o">=</span> kubernetes-admin
group <span class="o">=</span> system:masters
</code></pre>

</div>



<p>Meaning:</p>

<ul>
<li><p>This is the <strong>cluster admin identity</strong></p></li>
<li><p>Full power (cluster-admin cluster role)</p></li>
</ul>

<p>Permissions:</p>

<ul>
<li>can do “everything” — (<code>get</code>/<code>delete</code>/<code>list</code>/<code>create</code>/<code>patch</code> ANY resource)</li>
</ul>

<p><strong><em>NodeRestriction does NOT apply to this identity.</em></strong></p>

<h1>
  
  
  2. Workflows
</h1>

<p>there is one section about <a href="https://kubernetes.io/docs/reference/access-authn-authz/admission-controllers/#admission-control-phases" rel="noopener noreferrer">Admission control phase</a>, but it may not easy to understand. Here is my previous post for the workflow (<a href="https://dev.to/cheedge_lee/kubernets-work-flow-5da8">Kubernetes workflow</a>).</p>

<h2>
  
  
  2.1 Basic concepts
</h2>

<p>Let’s first have a overlook of some basic components.</p>

<p><code>kubelet</code></p>

<ul>
<li><p>runs the containers</p></li>
<li><p>updates pod/node status</p></li>
</ul>

<p><code>kubectl</code></p>

<ul>
<li>send requests</li>
</ul>

<p><code>kubelet.conf</code> = identity of kubelet</p>

<ul>
<li>Used by kubelet when it talks to API server, (NodeRestriction)</li>
</ul>

<p><code>admin.conf / ~/.kube/config</code> = identity of cluster admin</p>

<ul>
<li>Used by kubectl</li>
</ul>

<h2>
  
  
  2.2 Diff workflows
</h2>

<p>Now let’s see the workflow for diff process</p>

<h3>
  
  
  API request:
</h3>

<p><code>kubectl → API server → etcd</code></p>

<h3>
  
  
  Pod scheduling:
</h3>

<p><code>scheduler → API server → etcd</code></p>

<h3>
  
  
  Pod execution:
</h3>

<p><code>kubelet → containerd → pod runs</code></p>

<h3>
  
  
  Status updates:
</h3>

<p><code>kubelet → API server → etcd</code></p>

<h3>
  
  
  Controllers maintain desired state:
</h3>

<p><code>controller manager → API server → etcd</code></p>

<h1>
  
  
  3. Security
</h1>

<p>the basic security flow is Authentication - Authorization - (Admission Control)</p>

<h2>
  
  
  3.1 Authentication (Identity)
</h2>

<p>Here we see when using these 2 type (node identity, cluster identity) config file their workflow:</p>

<ol>
<li>When human/admin runs <code>kubectl</code> (<strong>on controlplane</strong>)
</li>
</ol>

<div class="highlight js-code-highlight">
<pre class="highlight shell"><code><span class="c"># eg. on controlplane</span>
k get node
k label node node01 <span class="nb">test</span>/thislabel<span class="o">=</span>123
</code></pre>

</div>





<div class="highlight js-code-highlight">
<pre class="highlight shell"><code>0. ~/.kube/config

1. kubectl sends HTTPS request with admin cert

2. API server authenticates admin identity
</code></pre>

</div>



<ol>
<li>When kubelet talks to API server (<strong>on each node</strong>)
</li>
</ol>

<div class="highlight js-code-highlight">
<pre class="highlight shell"><code><span class="c"># eg. on node01</span>
k label node node01 <span class="nb">test</span>/thislabel<span class="o">=</span>123 <span class="nt">--kubeconfig</span><span class="o">=</span>/etc/kubernetes/kubelet.conf
</code></pre>

</div>





<div class="highlight js-code-highlight">
<pre class="highlight shell"><code>0. /etc/kubernetes/kubelet.conf  <span class="o">(</span><span class="nb">local </span>to that node<span class="o">)</span>

1. kubelet loads cert/key

2. kubelet sends HTTPS request to API server

3. API server authenticates <span class="s2">"system:node:NODENAME"</span>
</code></pre>

</div>



<blockquote>
<p>notice:</p>

<ul>
<li><p><code>kubectl</code> sends requests for user operations</p></li>
<li><p><code>kubelet</code> sends requests for node/pod management</p></li>
</ul>
</blockquote>

<h2>
  
  
  3.2 Authorization (RBAC)
</h2>

<ul>
<li><p>Uses CN and O from the certificate</p></li>
<li><p>Checks “is this identity allowed to do this verb on this resource?”</p></li>
</ul>

<h3>
  
  
  RBAC uses this identity to check:
</h3>

<ol>
<li>Which User?  eg. <code>system:node:node01</code>
</li>
<li>Which Groups? eg. <code>system:nodes</code> (all kubelets), <code>system:authenticated</code>
</li>
<li>Which verb/resource? 

<ul>
<li><code>update node/worker1</code></li>
<li><code>patch node/worker1</code></li>
<li><code>get pods</code></li>
</ul>
</li>
<li>RBAC rules that apply to those groups 

<ul>
<li>eg. Kubernetes has a built-in ClusterRole</li>
</ul>
</li>
</ol>

<p>more details can refer to official doc or previous post <a href="https://dev.to/cheedge_lee/a-glance-of-kubernetes-rbac-547a">here</a>.</p>

<h2>
  
  
  3.3 Admission Control (NodeRestriction)
</h2>

<p>Check the data/operations that try to modify the resource in the arriving request. Prevents kubelets from doing dangerous modifications:</p>

<ul>
<li><p>Node labels</p></li>
<li><p>Node annotations</p></li>
<li><p>Node taints</p></li>
<li><p>Pods on other nodes</p></li>
</ul>

<p>More details can check in official doc “<a href="https://kubernetes.io/docs/reference/access-authn-authz/admission-controllers" rel="noopener noreferrer">Admission Control in Kubernetes</a>” and “<a href="https://kubernetes.io/docs/reference/access-authn-authz/node/" rel="noopener noreferrer">Using Node Authorization</a>”</p>

