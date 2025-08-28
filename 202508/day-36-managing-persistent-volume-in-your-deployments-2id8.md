---
Title: Day 36: Managing Persistent volume in your deployments
Description: 
Author: Udoh Deborah
Date: 2025-08-28T21:46:58.000Z
Robots: noindex,nofollow
Template: index
---
<p>On Day 36 of your #90DaysOfDevOps journey, we’re  tackling an important part of Kubernetes: Persistent Volumes (PVs)</p>

<p>Let’s break this down and walk you through both tasks:</p>

<p>What are Persistent Volumes (PV) in Kubernetes?</p>

<p>Kubernetes Pods are ephemeral — which means data inside a Pod is lost when the Pod is deleted or restarted. To persist data beyond the lifecycle of a Pod, we use:<br>
    • Persistent Volumes (PV): Storage resource in the cluster, provisioned by admins or dynamically via StorageClass.<br>
    • Persistent Volume Claims (PVC): A request for storage by a user or app.<br>
    • Deployment + PVC: Your app (e.g., the ToDo app) can claim storage by referencing a PVC.</p>

<p>Task 1: Add Persistent Volume to Your Deployment</p>

<p>Let’s go step-by-step.</p>

<ol>
<li>Create a Persistent Volume (pv.yml)</li>
</ol>

<p>apiVersion: v1<br>
kind: PersistentVolume<br>
metadata:<br>
  name: todo-pv<br>
spec:<br>
  capacity:<br>
    storage: 1Gi<br>
  accessModes:<br>
    - ReadWriteOnce<br>
  hostPath:<br>
    path: /mnt/data</p>

<p>This creates a 1Gi volume using the hostPath /mnt/data on the node (local path for testing).</p>

<ol>
<li>Create a Persistent Volume Claim (pvc.yml)</li>
</ol>

<p>apiVersion: v1<br>
kind: PersistentVolumeClaim<br>
metadata:<br>
  name: todo-pvc<br>
spec:<br>
  accessModes:<br>
    - ReadWriteOnce<br>
  resources:<br>
    requests:<br>
      storage: 1Gi</p>

<p>This PVC will bind to the todo-pv if the specs match.</p>

<ol>
<li>Update your deployment.yml to use the PVC</li>
</ol>

<p>Here’s the updated Deployment with volume mounting:</p>

<p>apiVersion: apps/v1<br>
kind: Deployment<br>
metadata:<br>
  name: todo-app<br>
spec:<br>
  replicas: 1<br>
  selector:<br>
    matchLabels:<br>
      app: todo<br>
  template:<br>
    metadata:<br>
      labels:<br>
        app: todo<br>
    spec:<br>
      containers:<br>
        - name: todo-container<br>
          image: your-todo-app-image<br>
          volumeMounts:<br>
            - name: todo-storage<br>
              mountPath: /app/data<br>
      volumes:<br>
        - name: todo-storage<br>
          persistentVolumeClaim:<br>
            claimName: todo-pvc</p>

<p>Make sure:<br>
    •image: your-todo-app-image is replaced with your actual image.<br>
    •/app/data is where your app stores persistent data.</p>

<ol>
<li>Apply the files</li>
</ol>

<p>kubectl apply -f pv.yml<br>
kubectl apply -f pvc.yml<br>
kubectl apply -f deployment.yml</p>

<ol>
<li>Verify Everything</li>
</ol>

<p>kubectl get pv<br>
kubectl get pvc<br>
kubectl get pods</p>

<p>Look for STATUS: Bound on the PVC and Running pods.</p>

<p>Task 2: Access Data Inside the Pod</p>

<ol>
<li>Connect to the Pod</li>
</ol>

<p>kubectl get pods  # get pod name<br>
kubectl exec -it  -- /bin/bash</p>

<ol>
<li>Verify Data Access</li>
</ol>

<p>cd /app/data<br>
ls -l</p>

<h1>
  
  
  OR
</h1>

<p>cat </p>

<p>You should see files created or used by your app.</p>

<p>Optional: Test Persistence<br>
    1.  Delete the pod:</p>

<p>kubectl delete pod </p>

<div class="highlight js-code-highlight">
<pre class="highlight plaintext"><code>2.  A new pod will be created (Deployment ensures this).
3.  Check if data is still present by repeating the steps to access /app/data.
</code></pre>

</div>

<p>You have Created and attached a Persistent Volume.</p>

