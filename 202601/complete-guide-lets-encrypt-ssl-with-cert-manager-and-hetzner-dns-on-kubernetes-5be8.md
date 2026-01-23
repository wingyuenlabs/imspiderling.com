---
Title: Complete Guide: Let's Encrypt SSL with cert-manager and Hetzner DNS on Kubernetes
Description: 
Author: giveitatry
Date: 2026-01-23T21:32:35.000Z
Robots: noindex,nofollow
Template: index
---
<h2>
  
  
  Introduction
</h2>

<p>This guide walks you through setting up automatic SSL certificates from Let's Encrypt on a Kubernetes cluster using cert-manager with Hetzner DNS for DNS-01 challenge validation. This is the most reliable method for issuing and renewing SSL certificates automatically, especially if you have multiple domains or subdomains.</p>

<p>By the end of this guide, you'll have:</p>

<ul>
<li>Automatic SSL certificate issuance via Let's Encrypt</li>
<li>DNS-based validation (no need to expose HTTP validation endpoints)</li>
<li>Automatic certificate renewal before expiration</li>
<li>HTTPS redirect middleware in Traefik</li>
</ul>

<h2>
  
  
  Prerequisites
</h2>

<ul>
<li>A Kubernetes cluster (k3s, kubeadm, EKS, etc.)</li>
<li>kubectl configured to access your cluster</li>
<li>A domain with DNS hosted at Hetzner DNS</li>
<li>Helm installed on your local machine</li>
<li>Traefik ingress controller (this guide uses Traefik)</li>
</ul>

<h2>
  
  
  Architecture Overview
</h2>

<p>Here's how the process works:</p>

<ol>
<li>You create a Certificate resource in Kubernetes</li>
<li>cert-manager reads this and contacts Let's Encrypt</li>
<li>Let's Encrypt returns a DNS challenge token</li>
<li>cert-manager calls the Hetzner webhook with this token</li>
<li>The Hetzner webhook uses the Hetzner DNS API to create a TXT record</li>
<li>Let's Encrypt queries the DNS and validates the TXT record</li>
<li>Let's Encrypt issues the certificate</li>
<li>cert-manager stores it in a Kubernetes Secret</li>
<li>Your Ingress uses this Secret for HTTPS</li>
</ol>

<h2>
  
  
  Step 1: Install cert-manager
</h2>

<p>cert-manager is the core component that manages certificate lifecycle.<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight shell"><code>kubectl apply <span class="nt">-f</span> https://github.com/cert-manager/cert-manager/releases/download/v1.19.2/cert-manager.yaml
</code></pre>

</div>



<p>Wait for it to be ready:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight shell"><code>kubectl <span class="nb">wait</span> <span class="nt">--for</span><span class="o">=</span><span class="nv">condition</span><span class="o">=</span>ready pod <span class="nt">-l</span> app.kubernetes.io/instance<span class="o">=</span>cert-manager <span class="nt">-n</span> cert-manager <span class="nt">--timeout</span><span class="o">=</span>300s
</code></pre>

</div>



<p>Verify installation:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight shell"><code>kubectl get pods <span class="nt">-n</span> cert-manager
</code></pre>

</div>



<p>You should see three pods running:</p>

<ul>
<li>cert-manager</li>
<li>cert-manager-cainjector</li>
<li>cert-manager-webhook</li>
</ul>

<h2>
  
  
  Step 2: Create Hetzner DNS API Token
</h2>

<ol>
<li>Log in to <a href="https://dns.hetzner.com" rel="noopener noreferrer">Hetzner DNS Control Panel</a>
</li>
<li>Navigate to "API Tokens" (usually under account settings)</li>
<li>Click "Create new token"</li>
<li>Give it a name like "cert-manager"</li>
<li>Copy the token immediately (you won't see it again)</li>
</ol>

<p><strong>Important:</strong> Keep this token secret. Anyone with this token can modify your DNS records.</p>

<h2>
  
  
  Step 3: Install the Hetzner Webhook for cert-manager
</h2>

<p>The Hetzner webhook is a cert-manager extension that knows how to talk to the Hetzner DNS API.</p>

<p>Add the Helm repository:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight shell"><code>helm repo add cert-manager-webhook-hetzner https://vadimkim.github.io/cert-manager-webhook-hetzner
helm repo update
</code></pre>

</div>



<p>Install the webhook:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight shell"><code>helm <span class="nb">install </span>cert-manager-webhook-hetzner <span class="se">\</span>
  <span class="nt">--namespace</span> cert-manager <span class="se">\</span>
  cert-manager-webhook-hetzner/cert-manager-webhook-hetzner
</code></pre>

</div>



<p>Verify the webhook pod is running:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight shell"><code>kubectl get pods <span class="nt">-n</span> cert-manager | <span class="nb">grep </span>hetzner
</code></pre>

</div>



<h2>
  
  
  Step 4: Create the Hetzner API Secret
</h2>

<p>Create a file called <code>hetzner-secret.yaml</code>:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight yaml"><code><span class="na">apiVersion</span><span class="pi">:</span> <span class="s">v1</span>
<span class="na">kind</span><span class="pi">:</span> <span class="s">Secret</span>
<span class="na">metadata</span><span class="pi">:</span>
  <span class="na">name</span><span class="pi">:</span> <span class="s">hcloud-api-token-secret</span>
  <span class="na">namespace</span><span class="pi">:</span> <span class="s">cert-manager</span>
<span class="na">type</span><span class="pi">:</span> <span class="s">Opaque</span>
<span class="na">stringData</span><span class="pi">:</span>
  <span class="na">api-key</span><span class="pi">:</span> <span class="s2">"</span><span class="s">your-hetzner-api-token-here"</span>
</code></pre>

</div>



<p><strong>Important:</strong> The key must be named <code>api-key</code> (not <code>token</code>). This is what the webhook expects.</p>

<p>Apply it:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight shell"><code>kubectl apply <span class="nt">-f</span> hetzner-secret.yaml
</code></pre>

</div>



<p>Verify the secret was created:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight shell"><code>kubectl get secret hcloud-api-token-secret <span class="nt">-n</span> cert-manager
</code></pre>

</div>



<h2>
  
  
  Step 5: Verify the Webhook's API Group Registration
</h2>

<p>Before creating the ClusterIssuer, you need to know the exact API group name that the webhook registered. This is crucial and often a source of problems.</p>

<p>Run this command:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight shell"><code>kubectl api-resources | <span class="nb">grep </span>hetzner
</code></pre>

</div>



<p>You should see output like:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight plaintext"><code>hetzner    hetzner.cert-mananger-webhook.noshoes.xyz/v1alpha1    false    ChallengePayload
</code></pre>

</div>



<p><strong>Note the exact API group name</strong> - in this example it's <code>hetzner.cert-mananger-webhook.noshoes.xyz</code>. This is what you'll use in your ClusterIssuer. Pay attention to the spelling - some versions have a typo: "mananger" instead of "manager".</p>

<h2>
  
  
  Step 6: Set Up RBAC Permissions
</h2>

<p>cert-manager needs permission to call the webhook, and the webhook needs permission to read the Hetzner API secret.</p>

<p>Create a file called <code>rbac.yaml</code>:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight yaml"><code><span class="c1"># Allow webhook to read the Hetzner API token</span>
<span class="na">apiVersion</span><span class="pi">:</span> <span class="s">rbac.authorization.k8s.io/v1</span>
<span class="na">kind</span><span class="pi">:</span> <span class="s">Role</span>
<span class="na">metadata</span><span class="pi">:</span>
  <span class="na">name</span><span class="pi">:</span> <span class="s">cert-manager-webhook-hetzner-secret-access</span>
  <span class="na">namespace</span><span class="pi">:</span> <span class="s">cert-manager</span>
<span class="na">rules</span><span class="pi">:</span>
<span class="pi">-</span> <span class="na">apiGroups</span><span class="pi">:</span> <span class="pi">[</span><span class="s2">"</span><span class="s">"</span><span class="pi">]</span>
  <span class="na">resources</span><span class="pi">:</span> <span class="pi">[</span><span class="s2">"</span><span class="s">secrets"</span><span class="pi">]</span>
  <span class="na">resourceNames</span><span class="pi">:</span> <span class="pi">[</span><span class="s2">"</span><span class="s">hcloud-api-token-secret"</span><span class="pi">]</span>
  <span class="na">verbs</span><span class="pi">:</span> <span class="pi">[</span><span class="s2">"</span><span class="s">get"</span><span class="pi">]</span>
<span class="nn">---</span>
<span class="na">apiVersion</span><span class="pi">:</span> <span class="s">rbac.authorization.k8s.io/v1</span>
<span class="na">kind</span><span class="pi">:</span> <span class="s">RoleBinding</span>
<span class="na">metadata</span><span class="pi">:</span>
  <span class="na">name</span><span class="pi">:</span> <span class="s">cert-manager-webhook-hetzner-secret-access</span>
  <span class="na">namespace</span><span class="pi">:</span> <span class="s">cert-manager</span>
<span class="na">roleRef</span><span class="pi">:</span>
  <span class="na">apiGroup</span><span class="pi">:</span> <span class="s">rbac.authorization.k8s.io</span>
  <span class="na">kind</span><span class="pi">:</span> <span class="s">Role</span>
  <span class="na">name</span><span class="pi">:</span> <span class="s">cert-manager-webhook-hetzner-secret-access</span>
<span class="na">subjects</span><span class="pi">:</span>
<span class="pi">-</span> <span class="na">kind</span><span class="pi">:</span> <span class="s">ServiceAccount</span>
  <span class="na">name</span><span class="pi">:</span> <span class="s">cert-manager-webhook-hetzner</span>
  <span class="na">namespace</span><span class="pi">:</span> <span class="s">cert-manager</span>
<span class="nn">---</span>
<span class="c1"># Allow cert-manager to call the webhook</span>
<span class="na">apiVersion</span><span class="pi">:</span> <span class="s">rbac.authorization.k8s.io/v1</span>
<span class="na">kind</span><span class="pi">:</span> <span class="s">ClusterRole</span>
<span class="na">metadata</span><span class="pi">:</span>
  <span class="na">name</span><span class="pi">:</span> <span class="s">cert-manager-webhook-hetzner</span>
<span class="na">rules</span><span class="pi">:</span>
<span class="pi">-</span> <span class="na">apiGroups</span><span class="pi">:</span>
  <span class="pi">-</span> <span class="s">hetzner.cert-mananger-webhook.noshoes.xyz</span>
  <span class="na">resources</span><span class="pi">:</span>
  <span class="pi">-</span> <span class="s">hetzner</span>
  <span class="na">verbs</span><span class="pi">:</span>
  <span class="pi">-</span> <span class="s">create</span>
<span class="nn">---</span>
<span class="na">apiVersion</span><span class="pi">:</span> <span class="s">rbac.authorization.k8s.io/v1</span>
<span class="na">kind</span><span class="pi">:</span> <span class="s">ClusterRoleBinding</span>
<span class="na">metadata</span><span class="pi">:</span>
  <span class="na">name</span><span class="pi">:</span> <span class="s">cert-manager-webhook-hetzner</span>
<span class="na">roleRef</span><span class="pi">:</span>
  <span class="na">apiGroup</span><span class="pi">:</span> <span class="s">rbac.authorization.k8s.io</span>
  <span class="na">kind</span><span class="pi">:</span> <span class="s">ClusterRole</span>
  <span class="na">name</span><span class="pi">:</span> <span class="s">cert-manager-webhook-hetzner</span>
<span class="na">subjects</span><span class="pi">:</span>
<span class="pi">-</span> <span class="na">kind</span><span class="pi">:</span> <span class="s">ServiceAccount</span>
  <span class="na">name</span><span class="pi">:</span> <span class="s">cert-manager</span>
  <span class="na">namespace</span><span class="pi">:</span> <span class="s">cert-manager</span>
</code></pre>

</div>



<p><strong>Important:</strong> Replace <code>hetzner.cert-mananger-webhook.noshoes.xyz</code> in the ClusterRole with whatever <code>kubectl api-resources | grep hetzner</code> showed you.</p>

<p>Apply it:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight shell"><code>kubectl apply <span class="nt">-f</span> rbac.yaml
</code></pre>

</div>



<h2>
  
  
  Step 7: Create the ClusterIssuer
</h2>

<p>The ClusterIssuer tells cert-manager how to request certificates from Let's Encrypt using Hetzner DNS.</p>

<p>Create a file called <code>cluster-issuer.yaml</code>:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight yaml"><code><span class="na">apiVersion</span><span class="pi">:</span> <span class="s">cert-manager.io/v1</span>
<span class="na">kind</span><span class="pi">:</span> <span class="s">ClusterIssuer</span>
<span class="na">metadata</span><span class="pi">:</span>
  <span class="na">name</span><span class="pi">:</span> <span class="s">letsencrypt-dns-hetzner</span>
<span class="na">spec</span><span class="pi">:</span>
  <span class="na">acme</span><span class="pi">:</span>
    <span class="na">server</span><span class="pi">:</span> <span class="s">https://acme-v02.api.letsencrypt.org/directory</span>
    <span class="na">email</span><span class="pi">:</span> <span class="s">admin@example.com</span>
    <span class="na">privateKeySecretRef</span><span class="pi">:</span>
      <span class="na">name</span><span class="pi">:</span> <span class="s">letsencrypt-dns-hetzner-key</span>
    <span class="na">solvers</span><span class="pi">:</span>
      <span class="pi">-</span> <span class="na">dns01</span><span class="pi">:</span>
          <span class="na">webhook</span><span class="pi">:</span>
            <span class="na">groupName</span><span class="pi">:</span> <span class="s">hetzner.cert-mananger-webhook.noshoes.xyz</span>
            <span class="na">solverName</span><span class="pi">:</span> <span class="s">hetzner</span>
            <span class="na">config</span><span class="pi">:</span>
              <span class="na">secretName</span><span class="pi">:</span> <span class="s">hcloud-api-token-secret</span>
              <span class="na">zoneName</span><span class="pi">:</span> <span class="s">example.com</span>
              <span class="na">apiUrl</span><span class="pi">:</span> <span class="s">https://dns.hetzner.com/api/v1</span>
</code></pre>

</div>



<p><strong>Replace:</strong></p>

<ul>
<li>
<code>admin@example.com</code> with your email</li>
<li>
<code>groupName</code> with the exact value from <code>kubectl api-resources | grep hetzner</code>
</li>
<li>
<code>zoneName</code> with your root domain (e.g., <code>example.com</code>)</li>
</ul>

<p>For testing/staging (to avoid Let's Encrypt rate limits), use this server instead:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight yaml"><code><span class="na">server</span><span class="pi">:</span> <span class="s">https://acme-staging-v02.api.letsencrypt.org/directory</span>
</code></pre>

</div>



<p>Apply it:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight shell"><code>kubectl apply <span class="nt">-f</span> cluster-issuer.yaml
</code></pre>

</div>



<p>Verify the issuer is ready:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight shell"><code>kubectl describe clusterissuer letsencrypt-dns-hetzner
</code></pre>

</div>



<p>Look for <code>Status: True</code> in the conditions.</p>

<h2>
  
  
  Step 8: Create Certificate Resources
</h2>

<p>Now create a Certificate resource for each domain you want to secure.</p>

<p>Create a file called <code>certificates.yaml</code>:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight yaml"><code><span class="na">apiVersion</span><span class="pi">:</span> <span class="s">cert-manager.io/v1</span>
<span class="na">kind</span><span class="pi">:</span> <span class="s">Certificate</span>
<span class="na">metadata</span><span class="pi">:</span>
  <span class="na">name</span><span class="pi">:</span> <span class="s">example-tls</span>
  <span class="na">namespace</span><span class="pi">:</span> <span class="s">default</span>
<span class="na">spec</span><span class="pi">:</span>
  <span class="na">secretName</span><span class="pi">:</span> <span class="s">example-tls</span>
  <span class="na">issuerRef</span><span class="pi">:</span>
    <span class="na">name</span><span class="pi">:</span> <span class="s">letsencrypt-dns-hetzner</span>
    <span class="na">kind</span><span class="pi">:</span> <span class="s">ClusterIssuer</span>
  <span class="na">commonName</span><span class="pi">:</span> <span class="s">example.com</span>
  <span class="na">dnsNames</span><span class="pi">:</span>
    <span class="pi">-</span> <span class="s">example.com</span>
    <span class="pi">-</span> <span class="s2">"</span><span class="s">*.example.com"</span>
<span class="nn">---</span>
<span class="na">apiVersion</span><span class="pi">:</span> <span class="s">cert-manager.io/v1</span>
<span class="na">kind</span><span class="pi">:</span> <span class="s">Certificate</span>
<span class="na">metadata</span><span class="pi">:</span>
  <span class="na">name</span><span class="pi">:</span> <span class="s">api-tls</span>
  <span class="na">namespace</span><span class="pi">:</span> <span class="s">default</span>
<span class="na">spec</span><span class="pi">:</span>
  <span class="na">secretName</span><span class="pi">:</span> <span class="s">api-tls</span>
  <span class="na">issuerRef</span><span class="pi">:</span>
    <span class="na">name</span><span class="pi">:</span> <span class="s">letsencrypt-dns-hetzner</span>
    <span class="na">kind</span><span class="pi">:</span> <span class="s">ClusterIssuer</span>
  <span class="na">commonName</span><span class="pi">:</span> <span class="s">api.example.com</span>
  <span class="na">dnsNames</span><span class="pi">:</span>
    <span class="pi">-</span> <span class="s">api.example.com</span>
</code></pre>

</div>



<p><strong>Replace:</strong></p>

<ul>
<li>Namespace with your application's namespace</li>
<li>Domain names with your actual domains</li>
<li>Certificate names as appropriate</li>
</ul>

<p>Apply it:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight shell"><code>kubectl apply <span class="nt">-f</span> certificates.yaml
</code></pre>

</div>



<h2>
  
  
  Step 9: Monitor Certificate Creation
</h2>

<p>Watch the certificates being created:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight shell"><code>kubectl get certificate <span class="nt">-A</span> <span class="nt">-w</span>
</code></pre>

</div>



<p>Check detailed status:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight shell"><code>kubectl describe certificate example-tls <span class="nt">-n</span> default
</code></pre>

</div>



<p>Watch the challenges:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight shell"><code>kubectl get challenge <span class="nt">-n</span> default <span class="nt">-w</span>
</code></pre>

</div>



<p>Check the webhook logs to see DNS records being created:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight shell"><code>kubectl logs <span class="nt">-n</span> cert-manager <span class="nt">-l</span> <span class="nv">app</span><span class="o">=</span>cert-manager-webhook-hetzner <span class="nt">-f</span> <span class="nt">--tail</span><span class="o">=</span>50
</code></pre>

</div>



<p>You should see messages like:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight plaintext"><code>Added TXT record result: {"record":{"id":"...","type":"TXT","name":"_acme-challenge.example",...}}
Presented txt record _acme-challenge.example.com.
Delete TXT record result: {"error":{}}
</code></pre>

</div>



<p>Once the certificate shows <code>Ready: True</code>, the secret is ready to use.</p>

<h2>
  
  
  Step 10: Use Certificates in Traefik Ingress
</h2>

<p>For standard Ingress with Traefik:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight yaml"><code><span class="na">apiVersion</span><span class="pi">:</span> <span class="s">networking.k8s.io/v1</span>
<span class="na">kind</span><span class="pi">:</span> <span class="s">Ingress</span>
<span class="na">metadata</span><span class="pi">:</span>
  <span class="na">name</span><span class="pi">:</span> <span class="s">example-ingress</span>
  <span class="na">namespace</span><span class="pi">:</span> <span class="s">default</span>
  <span class="na">annotations</span><span class="pi">:</span>
    <span class="na">cert-manager.io/cluster-issuer</span><span class="pi">:</span> <span class="s">letsencrypt-dns-hetzner</span>
<span class="na">spec</span><span class="pi">:</span>
  <span class="na">ingressClassName</span><span class="pi">:</span> <span class="s">traefik</span>
  <span class="na">tls</span><span class="pi">:</span>
    <span class="pi">-</span> <span class="na">hosts</span><span class="pi">:</span>
        <span class="pi">-</span> <span class="s">example.com</span>
      <span class="na">secretName</span><span class="pi">:</span> <span class="s">example-tls</span>
  <span class="na">rules</span><span class="pi">:</span>
    <span class="pi">-</span> <span class="na">host</span><span class="pi">:</span> <span class="s">example.com</span>
      <span class="na">http</span><span class="pi">:</span>
        <span class="na">paths</span><span class="pi">:</span>
          <span class="pi">-</span> <span class="na">path</span><span class="pi">:</span> <span class="s">/</span>
            <span class="na">pathType</span><span class="pi">:</span> <span class="s">Prefix</span>
            <span class="na">backend</span><span class="pi">:</span>
              <span class="na">service</span><span class="pi">:</span>
                <span class="na">name</span><span class="pi">:</span> <span class="s">my-service</span>
                <span class="na">port</span><span class="pi">:</span>
                  <span class="na">number</span><span class="pi">:</span> <span class="m">80</span>
</code></pre>

</div>



<p>For Traefik IngressRoute with HTTP-to-HTTPS redirect:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight yaml"><code><span class="na">apiVersion</span><span class="pi">:</span> <span class="s">traefik.io/v1alpha1</span>
<span class="na">kind</span><span class="pi">:</span> <span class="s">Middleware</span>
<span class="na">metadata</span><span class="pi">:</span>
  <span class="na">name</span><span class="pi">:</span> <span class="s">redirect-https</span>
  <span class="na">namespace</span><span class="pi">:</span> <span class="s">default</span>
<span class="na">spec</span><span class="pi">:</span>
  <span class="na">redirectScheme</span><span class="pi">:</span>
    <span class="na">scheme</span><span class="pi">:</span> <span class="s">https</span>
    <span class="na">permanent</span><span class="pi">:</span> <span class="kc">true</span>
<span class="nn">---</span>
<span class="na">apiVersion</span><span class="pi">:</span> <span class="s">traefik.io/v1alpha1</span>
<span class="na">kind</span><span class="pi">:</span> <span class="s">IngressRoute</span>
<span class="na">metadata</span><span class="pi">:</span>
  <span class="na">name</span><span class="pi">:</span> <span class="s">example-http</span>
  <span class="na">namespace</span><span class="pi">:</span> <span class="s">default</span>
<span class="na">spec</span><span class="pi">:</span>
  <span class="na">entryPoints</span><span class="pi">:</span>
    <span class="pi">-</span> <span class="s">web</span>
  <span class="na">routes</span><span class="pi">:</span>
    <span class="pi">-</span> <span class="na">match</span><span class="pi">:</span> <span class="s">Host(`example.com`)</span>
      <span class="na">kind</span><span class="pi">:</span> <span class="s">Rule</span>
      <span class="na">middlewares</span><span class="pi">:</span>
        <span class="pi">-</span> <span class="na">name</span><span class="pi">:</span> <span class="s">redirect-https</span>
          <span class="na">namespace</span><span class="pi">:</span> <span class="s">default</span>
      <span class="na">services</span><span class="pi">:</span>
        <span class="pi">-</span> <span class="na">name</span><span class="pi">:</span> <span class="s">my-service</span>
          <span class="na">port</span><span class="pi">:</span> <span class="m">80</span>
<span class="nn">---</span>
<span class="na">apiVersion</span><span class="pi">:</span> <span class="s">traefik.io/v1alpha1</span>
<span class="na">kind</span><span class="pi">:</span> <span class="s">IngressRoute</span>
<span class="na">metadata</span><span class="pi">:</span>
  <span class="na">name</span><span class="pi">:</span> <span class="s">example-https</span>
  <span class="na">namespace</span><span class="pi">:</span> <span class="s">default</span>
<span class="na">spec</span><span class="pi">:</span>
  <span class="na">entryPoints</span><span class="pi">:</span>
    <span class="pi">-</span> <span class="s">websecure</span>
  <span class="na">routes</span><span class="pi">:</span>
    <span class="pi">-</span> <span class="na">match</span><span class="pi">:</span> <span class="s">Host(`example.com`)</span>
      <span class="na">kind</span><span class="pi">:</span> <span class="s">Rule</span>
      <span class="na">services</span><span class="pi">:</span>
        <span class="pi">-</span> <span class="na">name</span><span class="pi">:</span> <span class="s">my-service</span>
          <span class="na">port</span><span class="pi">:</span> <span class="m">80</span>
  <span class="na">tls</span><span class="pi">:</span>
    <span class="na">secretName</span><span class="pi">:</span> <span class="s">example-tls</span>
</code></pre>

</div>



<h2>
  
  
  Troubleshooting Guide
</h2>

<h3>
  
  
  Problem 1: Certificate Stuck in "Pending" State
</h3>

<p><strong>Symptom:</strong> <code>kubectl get certificate</code> shows <code>READY: False</code> for hours</p>

<p><strong>Solution:</strong></p>

<p>Check the certificate details:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight shell"><code>kubectl describe certificate example-tls <span class="nt">-n</span> default
</code></pre>

</div>



<p>Look at the events section. The most common causes are listed below.</p>

<h3>
  
  
  Problem 2: "Unable to get secret" Error
</h3>

<p><strong>Error message:</strong><br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight plaintext"><code>unable to get secret `hcloud-api-token-secret/cert-manager`; secrets "hcloud-api-token-secret" is forbidden
</code></pre>

</div>



<p><strong>Cause:</strong> The RBAC permissions are not set up correctly. The webhook's service account can't read the Hetzner API token secret.</p>

<p><strong>Solution:</strong></p>

<ol>
<li>Verify the secret exists:
</li>
</ol>

<div class="highlight js-code-highlight">
<pre class="highlight shell"><code>kubectl get secret hcloud-api-token-secret <span class="nt">-n</span> cert-manager
</code></pre>

</div>



<ol>
<li>Reapply the RBAC rules:
</li>
</ol>

<div class="highlight js-code-highlight">
<pre class="highlight shell"><code>kubectl apply <span class="nt">-f</span> rbac.yaml
</code></pre>

</div>



<ol>
<li>Make sure the Role has the correct secret name:
</li>
</ol>

<div class="highlight js-code-highlight">
<pre class="highlight yaml"><code><span class="na">rules</span><span class="pi">:</span>
<span class="pi">-</span> <span class="na">apiGroups</span><span class="pi">:</span> <span class="pi">[</span><span class="s2">"</span><span class="s">"</span><span class="pi">]</span>
  <span class="na">resources</span><span class="pi">:</span> <span class="pi">[</span><span class="s2">"</span><span class="s">secrets"</span><span class="pi">]</span>
  <span class="na">resourceNames</span><span class="pi">:</span> <span class="pi">[</span><span class="s2">"</span><span class="s">hcloud-api-token-secret"</span><span class="pi">]</span>  <span class="c1"># Exact match required</span>
  <span class="na">verbs</span><span class="pi">:</span> <span class="pi">[</span><span class="s2">"</span><span class="s">get"</span><span class="pi">]</span>
</code></pre>

</div>



<h3>
  
  
  Problem 3: "Key 'api-key' not found in secret"
</h3>

<p><strong>Error message:</strong><br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight plaintext"><code>unable to get api-key from secret `hcloud-api-token-secret`; key "api-key" not found in secret data
</code></pre>

</div>



<p><strong>Cause:</strong> The secret key is named wrong. The webhook expects exactly <code>api-key</code>.</p>

<p><strong>Solution:</strong></p>

<p>Delete and recreate the secret with the correct key name:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight shell"><code>kubectl delete secret hcloud-api-token-secret <span class="nt">-n</span> cert-manager
</code></pre>

</div>



<p>Create it again with <code>api-key</code>:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight yaml"><code><span class="na">apiVersion</span><span class="pi">:</span> <span class="s">v1</span>
<span class="na">kind</span><span class="pi">:</span> <span class="s">Secret</span>
<span class="na">metadata</span><span class="pi">:</span>
  <span class="na">name</span><span class="pi">:</span> <span class="s">hcloud-api-token-secret</span>
  <span class="na">namespace</span><span class="pi">:</span> <span class="s">cert-manager</span>
<span class="na">type</span><span class="pi">:</span> <span class="s">Opaque</span>
<span class="na">stringData</span><span class="pi">:</span>
  <span class="na">api-key</span><span class="pi">:</span> <span class="s2">"</span><span class="s">your-token-here"</span>
</code></pre>

</div>



<h3>
  
  
  Problem 4: "cannot create resource 'hetzner' is forbidden"
</h3>

<p><strong>Error message:</strong><br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight plaintext"><code>User "system:serviceaccount:cert-manager:cert-manager" cannot create resource "hetzner" in API group "hetzner.cert-manager-webhook.noshoes.xyz"
</code></pre>

</div>



<p><strong>Cause:</strong> cert-manager service account doesn't have permission to call the webhook.</p>

<p><strong>Solution:</strong></p>

<p>Reapply the ClusterRole and ClusterRoleBinding:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight shell"><code>kubectl apply <span class="nt">-f</span> rbac.yaml
</code></pre>

</div>



<p>Verify the ClusterRoleBinding exists:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight shell"><code>kubectl get clusterrolebinding | <span class="nb">grep </span>hetzner
</code></pre>

</div>



<h3>
  
  
  Problem 5: "the server could not find the requested resource"
</h3>

<p><strong>Error message:</strong><br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight plaintext"><code>the server could not find the requested resource (post hetzner.cert-manager-webhook.noshoes.xyz)
</code></pre>

</div>



<p><strong>Cause:</strong> The <code>groupName</code> in your ClusterIssuer doesn't match what the webhook registered.</p>

<p><strong>Solution:</strong></p>

<p>Check the actual API group:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight shell"><code>kubectl api-resources | <span class="nb">grep </span>hetzner
</code></pre>

</div>



<p>This outputs something like:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight plaintext"><code>hetzner    hetzner.cert-mananger-webhook.noshoes.xyz/v1alpha1    false    ChallengePayload
</code></pre>

</div>



<p>Copy the exact API group (e.g., <code>hetzner.cert-mananger-webhook.noshoes.xyz</code>) and update your ClusterIssuer:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight yaml"><code><span class="na">solvers</span><span class="pi">:</span>
  <span class="pi">-</span> <span class="na">dns01</span><span class="pi">:</span>
      <span class="na">webhook</span><span class="pi">:</span>
        <span class="na">groupName</span><span class="pi">:</span> <span class="s">hetzner.cert-mananger-webhook.noshoes.xyz</span>  <span class="c1"># Use exact match</span>
        <span class="na">solverName</span><span class="pi">:</span> <span class="s">hetzner</span>
</code></pre>

</div>



<h3>
  
  
  Problem 6: The Webhook Has a Typo in the API Group Name
</h3>

<p><strong>Issue:</strong> The webhook registers as <code>hetzner.cert-mananger-webhook.noshoes.xyz</code> (with "mananger" - missing 'e')</p>

<p><strong>Cause:</strong> This is a known issue in some versions of the Hetzner webhook Helm chart.</p>

<p><strong>Solution:</strong></p>

<p>You have two options:</p>

<p><strong>Option A (Quickest):</strong> Just use the misspelled group name in your ClusterIssuer. It works fine despite the typo:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight yaml"><code><span class="na">groupName</span><span class="pi">:</span> <span class="s">hetzner.cert-mananger-webhook.noshoes.xyz</span>
</code></pre>

</div>



<p><strong>Option B (Proper fix):</strong> Reinstall the webhook with a corrected version or use a different webhook implementation that doesn't have this typo.</p>

<h3>
  
  
  Problem 7: Challenge Not Being Presented
</h3>

<p><strong>Symptom:</strong> Challenge status shows <code>Presented: false</code> but no errors in logs</p>

<p><strong>Cause:</strong> Webhook pod might have crashed or become unhealthy</p>

<p><strong>Solution:</strong></p>

<p>Check webhook health:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight shell"><code>kubectl get pods <span class="nt">-n</span> cert-manager | <span class="nb">grep </span>hetzner
</code></pre>

</div>



<p>Check pod logs:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight shell"><code>kubectl logs <span class="nt">-n</span> cert-manager <span class="nt">-l</span> <span class="nv">app</span><span class="o">=</span>cert-manager-webhook-hetzner
</code></pre>

</div>



<p>Restart the webhook:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight shell"><code>kubectl rollout restart deployment/cert-manager-webhook-hetzner <span class="nt">-n</span> cert-manager
</code></pre>

</div>



<p>Delete the challenge to trigger a retry:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight shell"><code>kubectl delete challenge <span class="nt">-n</span> default <span class="nt">--all</span>
</code></pre>

</div>



<h3>
  
  
  Problem 8: DNS Record Not Being Created in Hetzner
</h3>

<p><strong>Symptom:</strong> No TXT record appears in Hetzner DNS console</p>

<p><strong>Cause:</strong> Usually an invalid API token or zone name mismatch</p>

<p><strong>Solution:</strong></p>

<ol>
<li>Verify the API token is correct:
</li>
</ol>

<div class="highlight js-code-highlight">
<pre class="highlight shell"><code>kubectl get secret hcloud-api-token-secret <span class="nt">-n</span> cert-manager <span class="nt">-o</span> <span class="nv">jsonpath</span><span class="o">=</span><span class="s1">'{.data.api-key}'</span> | <span class="nb">base64</span> <span class="nt">-d</span>
</code></pre>

</div>



<ol>
<li>Verify the zone name matches your domain:
</li>
</ol>

<div class="highlight js-code-highlight">
<pre class="highlight yaml"><code><span class="na">zoneName</span><span class="pi">:</span> <span class="s">example.com</span>  <span class="c1"># Must match your actual domain</span>
</code></pre>

</div>



<ol>
<li>Check the webhook logs for errors:
</li>
</ol>

<div class="highlight js-code-highlight">
<pre class="highlight shell"><code>kubectl logs <span class="nt">-n</span> cert-manager <span class="nt">-l</span> <span class="nv">app</span><span class="o">=</span>cert-manager-webhook-hetzner <span class="nt">-f</span> | <span class="nb">grep</span> <span class="nt">-i</span> error
</code></pre>

</div>



<ol>
<li>Test the API token manually:
</li>
</ol>

<div class="highlight js-code-highlight">
<pre class="highlight shell"><code>curl <span class="nt">-H</span> <span class="s2">"Auth-API-Token: your-token"</span> https://dns.hetzner.com/api/v1/zones
</code></pre>

</div>



<h3>
  
  
  Problem 9: Certificate Ready but HTTPS Still Shows Old/Invalid Certificate
</h3>

<p><strong>Cause:</strong> Ingress is not using the new secret or old ingress resource is still active</p>

<p><strong>Solution:</strong></p>

<ol>
<li>Verify the secret exists:
</li>
</ol>

<div class="highlight js-code-highlight">
<pre class="highlight shell"><code>kubectl get secret example-tls <span class="nt">-n</span> default
</code></pre>

</div>



<ol>
<li>Check the Ingress is referencing it:
</li>
</ol>

<div class="highlight js-code-highlight">
<pre class="highlight shell"><code>kubectl describe ingress example-ingress <span class="nt">-n</span> default
</code></pre>

</div>



<ol>
<li>If using both Ingress and IngressRoute, delete the old Ingress:
</li>
</ol>

<div class="highlight js-code-highlight">
<pre class="highlight shell"><code>kubectl delete ingress example-ingress <span class="nt">-n</span> default
</code></pre>

</div>



<ol>
<li>Wait for the load balancer to update (usually 30 seconds to 2 minutes)</li>
</ol>

<h2>
  
  
  Verification Checklist
</h2>

<p>Once everything is set up, verify with this checklist:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight shell"><code><span class="c"># 1. cert-manager is running</span>
kubectl get pods <span class="nt">-n</span> cert-manager

<span class="c"># 2. Webhook is running</span>
kubectl get pods <span class="nt">-n</span> cert-manager | <span class="nb">grep </span>hetzner

<span class="c"># 3. ClusterIssuer is ready</span>
kubectl describe clusterissuer letsencrypt-dns-hetzner

<span class="c"># 4. Certificate is ready</span>
kubectl get certificate <span class="nt">-A</span>

<span class="c"># 5. Secret is created</span>
kubectl get secret example-tls <span class="nt">-n</span> default

<span class="c"># 6. Ingress/IngressRoute is configured</span>
kubectl get ingress,ingressroute <span class="nt">-A</span>

<span class="c"># 7. DNS records exist</span>
nslookup _acme-challenge.example.com

<span class="c"># 8. HTTPS works</span>
curl <span class="nt">-I</span> https://example.com
</code></pre>

</div>



<h2>
  
  
  Automatic Renewal
</h2>

<p>cert-manager automatically renews certificates 30 days before expiration. You don't need to do anything.</p>

<p>Monitor renewal status:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight shell"><code>kubectl describe certificate example-tls <span class="nt">-n</span> default
</code></pre>

</div>



<p>Check cert-manager logs for renewal activity:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight shell"><code>kubectl logs <span class="nt">-n</span> cert-manager <span class="nt">-l</span> <span class="nv">app</span><span class="o">=</span>cert-manager | <span class="nb">grep</span> <span class="nt">-i</span> renew
</code></pre>

</div>



<h2>
  
  
  Best Practices
</h2>

<ol>
<li>
<strong>Use ClusterIssuer, not Issuer</strong> - ClusterIssuer is namespace-agnostic and reusable</li>
<li>
<strong>Use production Let's Encrypt</strong> - Only use staging for testing to avoid rate limits</li>
<li>
<strong>Keep API token secret</strong> - Don't commit it to git, use sealed secrets or external secret management</li>
<li>
<strong>Monitor certificate expiration</strong> - Set up alerts for certificates not renewing</li>
<li>
<strong>Use separate secrets per domain</strong> - Makes it easier to rotate certs individually</li>
<li>
<strong>Test in staging first</strong> - Use the Let's Encrypt staging server before production</li>
<li>
<strong>Keep webhook updated</strong> - Regularly update the webhook Helm chart for security patches</li>
</ol>

<h2>
  
  
  Conclusion
</h2>

<p>You now have a fully automated SSL certificate system that:</p>

<ul>
<li>Issues certificates automatically from Let's Encrypt</li>
<li>Validates using DNS (no need for HTTP validation)</li>
<li>Renews certificates automatically before expiration</li>
<li>Works across all your subdomains under one root domain</li>
<li>Integrates seamlessly with Traefik</li>
</ul>

<p>If you encounter issues, the troubleshooting guide above covers the most common problems and their solutions.</p>

