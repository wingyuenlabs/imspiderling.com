---
Title: Kubernetes 1.35: In-Place Pod Resize is GA — Scale Vertically Without Restarting
Description: 
Author: Grego
Date: 2026-01-06T22:07:26.000Z
Robots: noindex,nofollow
Template: index
---
<h1>
  
  
  Kubernetes 1.35: In-Place Pod Resize is GA — Scale Vertically Without Restarting
</h1>

<p><strong>Update CPU and memory of running Pods without recreating them</strong></p>




<h2>
  
  
  <strong>The Problem: “Simple” Changes That Cause Disruptions</strong>
</h2>

<p>A production service shows CPU throttling. P95 latency is rising. The solution is obvious: increase the CPU limit from 500m to 700m.</p>

<p>In earlier versions of Kubernetes, this “simple change” triggered a cascade of unwanted events:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight plaintext"><code># Change this...
resources:
  limits:
    cpu: "500m"

# ...to this
resources:
  limits:
    cpu: "700m"

</code></pre>

</div>



<p><strong>Consequences in K8s ≤1.34:</strong></p>

<ul>
<li>Pod completely recreated</li>
<li>Active connections terminated</li>
<li>In-memory cache lost</li>
<li>Local state deleted</li>
<li>Jobs in progress interrupted</li>
</ul>

<p>No code changed. No behavior changed. Just a number adjustment. But the system treated that adjustment as a full deployment.</p>

<p><strong>Kubernetes 1.35 changes this fundamentally.</strong></p>




<h2>
  
  
  <strong>The Solution: In-Place Resource Update (GA)</strong>
</h2>

<p>Starting with Kubernetes 1.35,<strong>in-place Pod resource updates are GA</strong>(Generally Available). This means CPU and memory can be modified in running Pods, and the node applies the new values without the automatic recreation cycle.</p>

<p><a href="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fworkplace.yodev.dev%2Fapi%2Fworkspaces%2F433d4ad6-a633-40cd-8a14-e0d45e4d5b2f%2Fblobs%2FG59sp2KYe5BZU5fm41jTSvisQ57P6j3B6oYWghQAbJ8%3D%3Ftoken%3D433d4ad6-a633-40cd-8a14-e0d45e4d5b2f%253AG59sp2KYe5BZU5fm41jTSvisQ57P6j3B6oYWghQAbJ8%253D%253A1767740845807%252CMEQCIHEmdXhoBur093C63FEhHRPHMR3CqI3fL6r%252BS%252BE2KUamAiB0eMm08LTeM2ndiGi76NqN59dlDUCZIEjLpzmdxVYqmA%253D%253D" class="article-body-image-wrapper"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fworkplace.yodev.dev%2Fapi%2Fworkspaces%2F433d4ad6-a633-40cd-8a14-e0d45e4d5b2f%2Fblobs%2FG59sp2KYe5BZU5fm41jTSvisQ57P6j3B6oYWghQAbJ8%3D%3Ftoken%3D433d4ad6-a633-40cd-8a14-e0d45e4d5b2f%253AG59sp2KYe5BZU5fm41jTSvisQ57P6j3B6oYWghQAbJ8%253D%253A1767740845807%252CMEQCIHEmdXhoBur093C63FEhHRPHMR3CqI3fL6r%252BS%252BE2KUamAiB0eMm08LTeM2ndiGi76NqN59dlDUCZIEjLpzmdxVYqmA%253D%253D" width="1477" height="1319"></a></p>

<p><strong>Key features:</strong></p>

<ul>
<li>CPU changes applied hot (without restart)</li>
<li>Memory changes configurable (with or without container restart)</li>
<li>Pod is never recreated — only cgroups are adjusted</li>
<li>State, connections, and cache preserved</li>
<li>Observable via Pod status and conditions</li>
</ul>




<h2>
  
  
  <strong>Architecture: How Resize Works</strong>
</h2>

<p>The process involves several Kubernetes components working in coordination:</p>

<p><a href="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fworkplace.yodev.dev%2Fapi%2Fworkspaces%2F433d4ad6-a633-40cd-8a14-e0d45e4d5b2f%2Fblobs%2FaOrceoGbsqMou5DhhTV1TbpcS2lLB6NvElGrrzwXpq8%3D%3Ftoken%3D433d4ad6-a633-40cd-8a14-e0d45e4d5b2f%253AaOrceoGbsqMou5DhhTV1TbpcS2lLB6NvElGrrzwXpq8%253D%253A1767740845807%252CMEUCIBdP2JGU0ZLp9mtWngD6igyJT%252F%252Bg1vkDv1ETEDKlzx%252BlAiEA1YG8CWuB%252FKQhjmIJYheqPYQ9lKirqFHkY7OVpcQhiO4%253D" class="article-body-image-wrapper"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fworkplace.yodev.dev%2Fapi%2Fworkspaces%2F433d4ad6-a633-40cd-8a14-e0d45e4d5b2f%2Fblobs%2FaOrceoGbsqMou5DhhTV1TbpcS2lLB6NvElGrrzwXpq8%3D%3Ftoken%3D433d4ad6-a633-40cd-8a14-e0d45e4d5b2f%253AaOrceoGbsqMou5DhhTV1TbpcS2lLB6NvElGrrzwXpq8%253D%253A1767740845807%252CMEUCIBdP2JGU0ZLp9mtWngD6igyJT%252F%252Bg1vkDv1ETEDKlzx%252BlAiEA1YG8CWuB%252FKQhjmIJYheqPYQ9lKirqFHkY7OVpcQhiO4%253D" width="767" height="2365"></a></p>

<h3>
  
  
  <strong>The New Mental Model: Desired vs Actual vs Allocated</strong>
</h3>

<p>Kubernetes 1.35 introduces an important distinction in how resources are reported:</p>

<div class="table-wrapper-paragraph"><table>
<thead>
<tr>
<th><strong>Field</strong></th>
<th><strong>Description</strong></th>
</tr>
</thead>
<tbody>
<tr>
<td><code>spec.containers[].resources</code></td>
<td>What the operator*<em>desires</em>*</td>
</tr>
<tr>
<td><code>status.containerStatuses[].allocatedResources</code></td>
<td>What the node*<em>reserved</em>*</td>
</tr>
<tr>
<td><code>status.containerStatuses[].resources</code></td>
<td>What the container*<em>is using</em>*</td>
</tr>
<tr>
<td><code>status.conditions</code></td>
<td>State of the resize (Pending, InProgress)</td>
</tr>
<tr>
<td><code>status.observedGeneration</code></td>
<td>Confirmation that kubelet processed the change</td>
</tr>
</tbody>
</table></div>

<p>This visibility allows you to know exactly what state the resize is in at any moment.</p>




<h2>
  
  
  <strong>Configuration: resizePolicy</strong>
</h2>

<p>The resize behavior is configured per resource type using<code>resizePolicy</code>in the container spec:</p>

<p><a href="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fworkplace.yodev.dev%2Fapi%2Fworkspaces%2F433d4ad6-a633-40cd-8a14-e0d45e4d5b2f%2Fblobs%2F9GNRrfhU2Fcs6zP2-9kiS5uBT8vbPbgywvkcpwvOejk%3D%3Ftoken%3D433d4ad6-a633-40cd-8a14-e0d45e4d5b2f%253A9GNRrfhU2Fcs6zP2-9kiS5uBT8vbPbgywvkcpwvOejk%253D%253A1767740845807%252CMEUCIQD1A5%252Fl5OSNwKb1s3wx513GHVuJkYerzG1%252FsVdYmnncrQIgSzDqUOWtE3W9dRkGTIB0ioGJjQAKk4yjvbZmVNtAPnA%253D" class="article-body-image-wrapper"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fworkplace.yodev.dev%2Fapi%2Fworkspaces%2F433d4ad6-a633-40cd-8a14-e0d45e4d5b2f%2Fblobs%2F9GNRrfhU2Fcs6zP2-9kiS5uBT8vbPbgywvkcpwvOejk%3D%3Ftoken%3D433d4ad6-a633-40cd-8a14-e0d45e4d5b2f%253A9GNRrfhU2Fcs6zP2-9kiS5uBT8vbPbgywvkcpwvOejk%253D%253A1767740845807%252CMEUCIQD1A5%252Fl5OSNwKb1s3wx513GHVuJkYerzG1%252FsVdYmnncrQIgSzDqUOWtE3W9dRkGTIB0ioGJjQAKk4yjvbZmVNtAPnA%253D" width="1587" height="1360"></a></p>

<h3>
  
  
  <strong>Pod Spec with Resize Policy</strong>
</h3>



<div class="highlight js-code-highlight">
<pre class="highlight plaintext"><code>apiVersion: v1
kind: Pod
metadata:
  name: app-con-resize
spec:
  containers:
    - name: app
      image: mi-app:latest
      ports:
        - containerPort: 8080
      resizePolicy:
        - resourceName: cpu
          restartPolicy: NotRequired      # CPU hot
        - resourceName: memory
          restartPolicy: RestartContainer # Memory with restart
      resources:
        requests:
          cpu: "300m"
          memory: "256Mi"
        limits:
          cpu: "300m"
          memory: "256Mi"

</code></pre>

</div>



<p><strong>Recommendation for production:</strong></p>

<ul>
<li>
<strong>CPU:</strong><code>NotRequired</code>— CPU changes are safe to apply hot</li>
<li>
<strong>Memory:</strong><code>RestartContainer</code>— More predictable than waiting for the app to free memory</li>
</ul>




<h2>
  
  
  <strong>Demo: Verifying that Resize Works</strong>
</h2>

<p>To demonstrate that resize really works without restart, you can create a simple server that exposes its PID and current cgroup limits.</p>

<h3>
  
  
  <strong>Demo Server (Go)</strong>
</h3>



<div class="highlight js-code-highlight">
<pre class="highlight plaintext"><code>package main

import (
    "fmt"
    "io"
    "net/http"
    "os"
    "strings"
)

funcread(path string) string {
    b, err := os.ReadFile(path)
    if err != nil {
        return fmt.Sprintf("unavailable (%v)", err)
    }
    return strings.TrimSpace(string(b))
}

funchandler(w http.ResponseWriter, r *http.Request) {
    pid := os.Getpid()

    // cgroup v2 paths
    cpuMax := read("/sys/fs/cgroup/cpu.max")
    memMax := read("/sys/fs/cgroup/memory.max")

    io.WriteString(w, fmt.Sprintf("pid=%d\n", pid))
    io.WriteString(w, fmt.Sprintf("cpu.max=%s\n", cpuMax))
    io.WriteString(w, fmt.Sprintf("memory.max=%s\n", memMax))
}

funcmain() {
    http.HandleFunc("/", handler)
    fmt.Println("listening on :8080")
    http.ListenAndServe(":8080", nil)
}

</code></pre>

</div>



<h3>
  
  
  <strong>Dockerfile</strong>
</h3>



<div class="highlight js-code-highlight">
<pre class="highlight plaintext"><code>FROM golang:1.23-alpine AS build
WORKDIR /src
COPY . .
RUN go build -o app .

FROM alpine:3.20
WORKDIR /app
COPY --from=build /src/app /app/app
EXPOSE 8080
ENTRYPOINT ["/app/app"]

</code></pre>

</div>



<h3>
  
  
  <strong>Deploy and Verify</strong>
</h3>



<div class="highlight js-code-highlight">
<pre class="highlight plaintext"><code># Create the Pod
kubectl apply -f pod-resize-demo.yaml

# Port-forward to access
kubectl port-forward pod/app-con-resize 8080:8080 &amp;

# Verify initial state
curl localhost:8080
# pid=7
# cpu.max=30000 100000
# memory.max=268435456

</code></pre>

</div>






<h2>
  
  
  <strong>Running the Resize</strong>
</h2>

<p>In Kubernetes 1.35, resize is executed against the Pod’s<code>resize</code>subresource:</p>

<h3>
  
  
  <strong>Increase CPU (Without Restart)</strong>
</h3>



<div class="highlight js-code-highlight">
<pre class="highlight plaintext"><code>kubectl patch pod app-con-resize --subresource resize --type merge -p ' { "spec": { "containers": [ { "name": "app", "resources": { "requests": { "cpu": "700m" }, "limits": { "cpu": "700m" } } } ] } }'

</code></pre>

</div>



<h3>
  
  
  <strong>Verify It Worked</strong>
</h3>



<div class="highlight js-code-highlight">
<pre class="highlight plaintext"><code># The endpoint should show:
# - Same PID (no restart)
# - New cpu.max value
curl localhost:8080
# pid=7 &lt;- SAME PID
# cpu.max=70000 100000 &lt;- NEW LIMIT
# memory.max=268435456

# Confirm there was no restart
kubectl get pod app-con-resize -o jsonpath='{.status.containerStatuses[0].restartCount}'
# 0

</code></pre>

</div>



<p><strong>If the PID remains the same and cpu.max changed, the in-place resize worked correctly.</strong></p>




<h2>
  
  
  <strong>Increase Memory (With Container Restart)</strong>
</h2>

<p>With the<code>RestartContainer</code>policy for memory:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight plaintext"><code>kubectl patch pod app-con-resize --subresource resize --type merge -p ' { "spec": { "containers": [ { "name": "app", "resources": { "requests": { "memory": "512Mi" }, "limits": { "memory": "512Mi" } } } ] } }'

</code></pre>

</div>



<p>In this case:</p>

<ul>
<li>
<code>restartCount</code>will increment</li>
<li>The PID will change</li>
<li>
<strong>But the Pod will NOT be recreated</strong>— volumes and networking are preserved</li>
</ul>




<h2>
  
  
  <strong>Observability During Resize</strong>
</h2>

<p>Kubernetes 1.35 provides visibility of the resize state via conditions:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight plaintext"><code>kubectl describe pod app-con-resize

</code></pre>

</div>



<p><strong>Relevant conditions:</strong></p>

<ul>
<li>
<code>PodResizePending</code>— The resize was requested but not applied yet</li>
<li>
<code>PodResizeInProgress</code>— The kubelet is applying the change
</li>
</ul>

<div class="highlight js-code-highlight">
<pre class="highlight plaintext"><code># See the detailed state
kubectl get pod app-con-resize -o jsonpath='{.status.conditions}' | jq
```

This eliminates uncertainty. There's no longer any guessing whether the change was applied — the system reports it explicitly. --- ## Limitations to Consider The feature is powerful but has clear limits: | Limitation | Description | |------------|-------------| | **QoS Class** | Cannot change post-creation (Guaranteed/Burstable/BestEffort) | | **Init containers** | Do not support resize | | **Ephemeral containers** | Do not support resize | | **Sidecars** | Yes, support resize | | **Windows Pods** | Not supported | | **Memory decrease** | Best-effort without restart (the app must free memory) | | **Node constraints** | Some CPU/Memory managers may block changes | These limitations are part of what makes the feature safe. A feature that promises everything becomes dangerous. A feature that declares its limits is operable. --- ## Scheduler Protection A valid concern: what happens if the resize is pending but the scheduler assumes it's already applied?

Kubernetes prevents this by being conservative during incomplete resizes. When scheduling, it considers the **maximum** between:
- What is requested (desired)
- What is allocated (allocated)
- What is applied (actual)

This prevents overcommit based on changes that haven't yet completed. --- ## Operational Impact The most significant change is not technical — it's cultural.

**Before K8s 1.35:**
- Teams avoided resize until it was urgent
- Engineers over-provisioned to avoid touching resources afterward
- "Right-sizing" was a project, not a habit
- On-call delayed simple fixes out of fear of disruption

**With K8s 1.35:**
- CPU corrections without restart cost
- Faster iteration over resource configuration
- Response to throttling without maintenance window
- Resize becomes a normal operation, not an event

---

## Command Summary



```bash
# Apply CPU resize
kubectl patch pod POD_NAME --subresource resize --type merge -p ' { "spec": { "containers": [{ "name": "CONTAINER_NAME", "resources": { "requests": { "cpu": "NEW_VALUE" }, "limits": { "cpu": "NEW_VALUE" } } }] } }'

# Check resize status
kubectl describe pod POD_NAME | grep -A5 Conditions

# View current vs desired resources
kubectl get pod POD_NAME -o jsonpath='{.status.containerStatuses[0].resources}'

# Confirm no restart occurred
kubectl get pod POD_NAME -o jsonpath='{.status.containerStatuses[0].restartCount}'

```

`

***

## **Conclusion**

Kubernetes 1.35 solves a problem that should never have existed: the need to restart a process just because a resource limit was adjusted.

With in-place resize GA:

* **CPU**can be adjusted without any restart
* **Memory**can be configured for container restart (not Pod)
* **Full observability**of resize state
* **Protection**against overcommit during pending changes

Vertical scaling finally behaves like an adjustment, not a deployment.

***

## **Resources**

* [KEP-1287: In-Place Pod Vertical Scaling](https://github.com/kubernetes/enhancements/tree/master/keps/sig-node/1287-in-place-update-pod-resources)
* [Kubernetes 1.35 Release Notes](https://kubernetes.io/blog/)
* [Pod Resource Management](https://kubernetes.io/docs/concepts/configuration/manage-resources-containers/)

***

*Published on yoDEV.dev — The Latin American developer community*
</code></pre>

</div>

