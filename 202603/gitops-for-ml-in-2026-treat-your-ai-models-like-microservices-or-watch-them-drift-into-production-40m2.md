---
Title: GitOps for ML in 2026: Treat Your AI Models Like Microservices (Or Watch Them Drift Into Production Chaos)
Description: 
Author: Mateen Anjum
Date: 2026-03-14T21:46:50.000Z
Robots: noindex,nofollow
Template: index
---
<p><strong>TL;DR:</strong> Apply the same GitOps discipline you use for application code to ML model deployments, and you get version history, rollback, and promotion gates that actually work, instead of the SSH-and-pray workflow most teams are still running.</p>

<h2>
  
  
  The Problem
</h2>

<p>There's a model running in production right now that nobody on your team can explain. It was trained six weeks ago, deployed by someone who's since moved to a different team, and the only record of what version it is lives in a Slack message that's been buried under 4,000 other messages.</p>

<p>When it starts making bad predictions, what's your rollback plan? If your answer involves SSHing into a server, editing a config file by hand, and hoping the right weights get loaded, you're in the majority. That doesn't make it less of a disaster.</p>

<p>I spent the better part of last year helping platform teams get their ML deployment story straight. The pattern I kept seeing: teams had decent model training pipelines, reasonable experiment tracking in MLflow, and then a complete gap between "model registered" and "model serving traffic." The gap got filled with shell scripts, manual steps, and a whole lot of tribal knowledge.</p>

<p>The fix isn't a new tool. It's applying discipline you already have from application deployments to the model deployment layer.</p>

<p>Before we moved to GitOps for model deployments, a typical promotion cycle looked like this. A data scientist trains a new version, registers it in MLflow, then files a ticket. A platform engineer picks up the ticket, SSH-es into the model server, updates the model path, restarts the serving process, and manually validates that predictions look reasonable. Start to finish: 4 to 6 hours on a good day, longer when the engineer is in meetings or the server is being weird.</p>

<p>Rollback? There was no rollback. The best-case scenario was that someone remembered what the previous model path was.</p>

<h2>
  
  
  What Most Teams Try First (And Why It Fails)
</h2>

<p>The first instinct is usually scripts. Someone writes a deploy.sh that takes a model version as an argument, connects to the serving infrastructure, and handles the update. This is better than pure manual steps, but it fails in a few predictable ways.</p>

<p>First, scripts don't have memory. You can run deploy.sh with model version 47, then run it again with version 51, and there's no audit trail of who ran what or why. When something goes wrong, you're back to grep-ing through logs and asking around.</p>

<p>Second, scripts don't handle promotion gates. You can't encode "this model can only go to production if it passed staging validation for 24 hours" in a shell script without it becoming a sprawling mess that nobody wants to maintain.</p>

<p>Third, and this one bites hardest: scripts assume the current state. If someone manually changes something on the serving infrastructure, your script has no way of detecting that drift. The next run might succeed or fail unpredictably depending on what changed and when.</p>

<p>MLflow solves the experiment tracking and model registry side well. You get version numbers, artifact storage in S3, stage transitions (Staging, Production), and a clean API. What MLflow doesn't give you is a Kubernetes-native way to declare "this cluster should be running model version 47 right now" and enforce that continuously.</p>

<p>That's where KServe and ArgoCD come in.</p>

<h2>
  
  
  The Architecture
</h2>

<p>The full stack has five layers working together.</p>

<p><a href="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fsla8yxyreadpi4g0h7zr.png" class="article-body-image-wrapper"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fsla8yxyreadpi4g0h7zr.png" alt=" " width="800" height="400"></a></p>

<p><strong>MLflow + S3</strong> handle model artifacts. Every trained model version gets registered with MLflow, which stores the artifact URI pointing to a path in S3. The URI looks something like <code>s3://ml-models-prod/fraud-detector/v47/model.pkl</code>. MLflow's registry gives you a version number and stage metadata. The actual weights live in S3.</p>

<p><strong>KServe InferenceService</strong> is the Kubernetes abstraction for serving. Instead of managing a Pod or Deployment by hand, you define an InferenceService custom resource that describes what model to load, from where, and how to scale. KServe handles the rest: downloading the artifact from S3, loading it into the serving framework (Triton, TorchServe, SKLearn Server), and exposing an HTTP endpoint.</p>

<p><strong>Git</strong> holds the desired state. A <code>values.yaml</code> file in your repository specifies which model version each environment should run. Promoting from staging to production is a PR that bumps a version number. The PR is the change review, the approval gate, and the audit trail all at once.</p>

<p><strong>ArgoCD</strong> reconciles the cluster to match what's in Git. When the PR merges, ArgoCD detects the change and applies the updated KServe InferenceService. If someone manually changes the InferenceService on the cluster, ArgoCD detects the drift and reverts it.</p>

<p><strong>Istio</strong> manages traffic splitting. During canary promotion, a VirtualService routes 10% of traffic to the new model version while 90% continues to the stable version. If metrics look good after a soak period, you update the weights and do a full cutover.</p>

<p><strong>Prometheus</strong> collects serving metrics. Latency (p99 in particular), throughput, and prediction distribution histograms give you the signals needed to decide whether a canary is healthy or needs to be rolled back.</p>

<h2>
  
  
  The Workflow
</h2>

<p>Here's how a model promotion actually works end to end.</p>

<p><a href="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Ftwb59lufwlvzqmev2ya1.png" class="article-body-image-wrapper"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Ftwb59lufwlvzqmev2ya1.png" alt=" " width="800" height="400"></a></p>

<p>A data scientist trains a new model, evaluates it against the validation set, and if it passes threshold, registers it in MLflow:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight python"><code><span class="kn">import</span> <span class="n">mlflow</span>

<span class="k">with</span> <span class="n">mlflow</span><span class="p">.</span><span class="nf">start_run</span><span class="p">():</span>
    <span class="n">mlflow</span><span class="p">.</span><span class="n">sklearn</span><span class="p">.</span><span class="nf">log_model</span><span class="p">(</span><span class="n">model</span><span class="p">,</span> <span class="sh">"</span><span class="s">model</span><span class="sh">"</span><span class="p">)</span>
    <span class="n">mlflow</span><span class="p">.</span><span class="nf">log_metrics</span><span class="p">({</span><span class="sh">"</span><span class="s">f1_score</span><span class="sh">"</span><span class="p">:</span> <span class="mf">0.94</span><span class="p">,</span> <span class="sh">"</span><span class="s">auc</span><span class="sh">"</span><span class="p">:</span> <span class="mf">0.97</span><span class="p">})</span>
    <span class="n">run_id</span> <span class="o">=</span> <span class="n">mlflow</span><span class="p">.</span><span class="nf">active_run</span><span class="p">().</span><span class="n">info</span><span class="p">.</span><span class="n">run_id</span>

<span class="n">client</span> <span class="o">=</span> <span class="n">mlflow</span><span class="p">.</span><span class="n">tracking</span><span class="p">.</span><span class="nc">MlflowClient</span><span class="p">()</span>
<span class="n">model_uri</span> <span class="o">=</span> <span class="sa">f</span><span class="sh">"</span><span class="s">runs:/</span><span class="si">{</span><span class="n">run_id</span><span class="si">}</span><span class="s">/model</span><span class="sh">"</span>
<span class="n">mv</span> <span class="o">=</span> <span class="n">client</span><span class="p">.</span><span class="nf">create_model_version</span><span class="p">(</span><span class="sh">"</span><span class="s">fraud-detector</span><span class="sh">"</span><span class="p">,</span> <span class="n">model_uri</span><span class="p">,</span> <span class="n">run_id</span><span class="p">)</span>
<span class="c1"># mv.version == "47"
</span></code></pre>

</div>



<p>That registration triggers a CI pipeline (GitHub Actions or Tekton, depending on your setup) that opens a pull request bumping the version in the dev environment's values file.</p>

<p><strong>values.yaml structure:</strong><br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight yaml"><code><span class="na">environments</span><span class="pi">:</span>
  <span class="na">dev</span><span class="pi">:</span>
    <span class="na">model</span><span class="pi">:</span>
      <span class="na">name</span><span class="pi">:</span> <span class="s">fraud-detector</span>
      <span class="na">version</span><span class="pi">:</span> <span class="s2">"</span><span class="s">47"</span>
      <span class="na">storageUri</span><span class="pi">:</span> <span class="s2">"</span><span class="s">s3://ml-models-prod/fraud-detector/v47"</span>
      <span class="na">resources</span><span class="pi">:</span>
        <span class="na">requests</span><span class="pi">:</span>
          <span class="na">cpu</span><span class="pi">:</span> <span class="s2">"</span><span class="s">2"</span>
          <span class="na">memory</span><span class="pi">:</span> <span class="s2">"</span><span class="s">4Gi"</span>
        <span class="na">limits</span><span class="pi">:</span>
          <span class="na">cpu</span><span class="pi">:</span> <span class="s2">"</span><span class="s">4"</span>
          <span class="na">memory</span><span class="pi">:</span> <span class="s2">"</span><span class="s">8Gi"</span>
      <span class="na">minReplicas</span><span class="pi">:</span> <span class="m">1</span>
      <span class="na">maxReplicas</span><span class="pi">:</span> <span class="m">3</span>

  <span class="na">staging</span><span class="pi">:</span>
    <span class="na">model</span><span class="pi">:</span>
      <span class="na">name</span><span class="pi">:</span> <span class="s">fraud-detector</span>
      <span class="na">version</span><span class="pi">:</span> <span class="s2">"</span><span class="s">45"</span>
      <span class="na">storageUri</span><span class="pi">:</span> <span class="s2">"</span><span class="s">s3://ml-models-prod/fraud-detector/v45"</span>
      <span class="na">resources</span><span class="pi">:</span>
        <span class="na">requests</span><span class="pi">:</span>
          <span class="na">cpu</span><span class="pi">:</span> <span class="s2">"</span><span class="s">2"</span>
          <span class="na">memory</span><span class="pi">:</span> <span class="s2">"</span><span class="s">4Gi"</span>
        <span class="na">limits</span><span class="pi">:</span>
          <span class="na">cpu</span><span class="pi">:</span> <span class="s2">"</span><span class="s">4"</span>
          <span class="na">memory</span><span class="pi">:</span> <span class="s2">"</span><span class="s">8Gi"</span>
      <span class="na">minReplicas</span><span class="pi">:</span> <span class="m">2</span>
      <span class="na">maxReplicas</span><span class="pi">:</span> <span class="m">5</span>

  <span class="na">prod</span><span class="pi">:</span>
    <span class="na">model</span><span class="pi">:</span>
      <span class="na">name</span><span class="pi">:</span> <span class="s">fraud-detector</span>
      <span class="na">version</span><span class="pi">:</span> <span class="s2">"</span><span class="s">43"</span>
      <span class="na">storageUri</span><span class="pi">:</span> <span class="s2">"</span><span class="s">s3://ml-models-prod/fraud-detector/v43"</span>
      <span class="na">resources</span><span class="pi">:</span>
        <span class="na">requests</span><span class="pi">:</span>
          <span class="na">cpu</span><span class="pi">:</span> <span class="s2">"</span><span class="s">4"</span>
          <span class="na">memory</span><span class="pi">:</span> <span class="s2">"</span><span class="s">8Gi"</span>
        <span class="na">limits</span><span class="pi">:</span>
          <span class="na">cpu</span><span class="pi">:</span> <span class="s2">"</span><span class="s">8"</span>
          <span class="na">memory</span><span class="pi">:</span> <span class="s2">"</span><span class="s">16Gi"</span>
      <span class="na">minReplicas</span><span class="pi">:</span> <span class="m">5</span>
      <span class="na">maxReplicas</span><span class="pi">:</span> <span class="m">20</span>
</code></pre>

</div>



<p><strong>KServe InferenceService (stable):</strong><br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight yaml"><code><span class="na">apiVersion</span><span class="pi">:</span> <span class="s">serving.kserve.io/v1beta1</span>
<span class="na">kind</span><span class="pi">:</span> <span class="s">InferenceService</span>
<span class="na">metadata</span><span class="pi">:</span>
  <span class="na">name</span><span class="pi">:</span> <span class="s">fraud-detector</span>
  <span class="na">namespace</span><span class="pi">:</span> <span class="s">ml-serving-prod</span>
  <span class="na">annotations</span><span class="pi">:</span>
    <span class="na">argocd.argoproj.io/sync-wave</span><span class="pi">:</span> <span class="s2">"</span><span class="s">1"</span>
<span class="na">spec</span><span class="pi">:</span>
  <span class="na">predictor</span><span class="pi">:</span>
    <span class="na">serviceAccountName</span><span class="pi">:</span> <span class="s">kserve-s3-sa</span>
    <span class="na">sklearn</span><span class="pi">:</span>
      <span class="na">storageUri</span><span class="pi">:</span> <span class="s2">"</span><span class="s">s3://ml-models-prod/fraud-detector/v43"</span>
      <span class="na">resources</span><span class="pi">:</span>
        <span class="na">requests</span><span class="pi">:</span>
          <span class="na">cpu</span><span class="pi">:</span> <span class="s2">"</span><span class="s">4"</span>
          <span class="na">memory</span><span class="pi">:</span> <span class="s2">"</span><span class="s">8Gi"</span>
        <span class="na">limits</span><span class="pi">:</span>
          <span class="na">cpu</span><span class="pi">:</span> <span class="s2">"</span><span class="s">8"</span>
          <span class="na">memory</span><span class="pi">:</span> <span class="s2">"</span><span class="s">16Gi"</span>
    <span class="na">minReplicas</span><span class="pi">:</span> <span class="m">5</span>
    <span class="na">maxReplicas</span><span class="pi">:</span> <span class="m">20</span>
    <span class="na">scaleTarget</span><span class="pi">:</span> <span class="m">80</span>
    <span class="na">scaleMetric</span><span class="pi">:</span> <span class="s">concurrency</span>
</code></pre>

</div>



<p><strong>KServe InferenceService (canary variant):</strong><br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight yaml"><code><span class="na">apiVersion</span><span class="pi">:</span> <span class="s">serving.kserve.io/v1beta1</span>
<span class="na">kind</span><span class="pi">:</span> <span class="s">InferenceService</span>
<span class="na">metadata</span><span class="pi">:</span>
  <span class="na">name</span><span class="pi">:</span> <span class="s">fraud-detector</span>
  <span class="na">namespace</span><span class="pi">:</span> <span class="s">ml-serving-prod</span>
  <span class="na">annotations</span><span class="pi">:</span>
    <span class="na">argocd.argoproj.io/sync-wave</span><span class="pi">:</span> <span class="s2">"</span><span class="s">1"</span>
<span class="na">spec</span><span class="pi">:</span>
  <span class="na">predictor</span><span class="pi">:</span>
    <span class="na">serviceAccountName</span><span class="pi">:</span> <span class="s">kserve-s3-sa</span>
    <span class="na">sklearn</span><span class="pi">:</span>
      <span class="na">storageUri</span><span class="pi">:</span> <span class="s2">"</span><span class="s">s3://ml-models-prod/fraud-detector/v47"</span>
      <span class="na">resources</span><span class="pi">:</span>
        <span class="na">requests</span><span class="pi">:</span>
          <span class="na">cpu</span><span class="pi">:</span> <span class="s2">"</span><span class="s">4"</span>
          <span class="na">memory</span><span class="pi">:</span> <span class="s2">"</span><span class="s">8Gi"</span>
        <span class="na">limits</span><span class="pi">:</span>
          <span class="na">cpu</span><span class="pi">:</span> <span class="s2">"</span><span class="s">8"</span>
          <span class="na">memory</span><span class="pi">:</span> <span class="s2">"</span><span class="s">16Gi"</span>
    <span class="na">minReplicas</span><span class="pi">:</span> <span class="m">1</span>
    <span class="na">maxReplicas</span><span class="pi">:</span> <span class="m">5</span>
    <span class="na">canaryTrafficPercent</span><span class="pi">:</span> <span class="m">10</span>
</code></pre>

</div>



<p><strong>ArgoCD ApplicationSet for multi-environment management:</strong><br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight yaml"><code><span class="na">apiVersion</span><span class="pi">:</span> <span class="s">argoproj.io/v1alpha1</span>
<span class="na">kind</span><span class="pi">:</span> <span class="s">ApplicationSet</span>
<span class="na">metadata</span><span class="pi">:</span>
  <span class="na">name</span><span class="pi">:</span> <span class="s">fraud-detector-serving</span>
  <span class="na">namespace</span><span class="pi">:</span> <span class="s">argocd</span>
<span class="na">spec</span><span class="pi">:</span>
  <span class="na">generators</span><span class="pi">:</span>
    <span class="pi">-</span> <span class="na">list</span><span class="pi">:</span>
        <span class="na">elements</span><span class="pi">:</span>
          <span class="pi">-</span> <span class="na">env</span><span class="pi">:</span> <span class="s">dev</span>
            <span class="na">cluster</span><span class="pi">:</span> <span class="s">dev-cluster</span>
            <span class="na">namespace</span><span class="pi">:</span> <span class="s">ml-serving-dev</span>
          <span class="pi">-</span> <span class="na">env</span><span class="pi">:</span> <span class="s">staging</span>
            <span class="na">cluster</span><span class="pi">:</span> <span class="s">staging-cluster</span>
            <span class="na">namespace</span><span class="pi">:</span> <span class="s">ml-serving-staging</span>
          <span class="pi">-</span> <span class="na">env</span><span class="pi">:</span> <span class="s">prod</span>
            <span class="na">cluster</span><span class="pi">:</span> <span class="s">prod-cluster</span>
            <span class="na">namespace</span><span class="pi">:</span> <span class="s">ml-serving-prod</span>
  <span class="na">template</span><span class="pi">:</span>
    <span class="na">metadata</span><span class="pi">:</span>
      <span class="na">name</span><span class="pi">:</span> <span class="s2">"</span><span class="s">fraud-detector-{{env}}"</span>
    <span class="na">spec</span><span class="pi">:</span>
      <span class="na">project</span><span class="pi">:</span> <span class="s">ml-serving</span>
      <span class="na">source</span><span class="pi">:</span>
        <span class="na">repoURL</span><span class="pi">:</span> <span class="s">https://github.com/org/ml-gitops</span>
        <span class="na">targetRevision</span><span class="pi">:</span> <span class="s">HEAD</span>
        <span class="na">path</span><span class="pi">:</span> <span class="s2">"</span><span class="s">environments/{{env}}"</span>
        <span class="na">helm</span><span class="pi">:</span>
          <span class="na">valueFiles</span><span class="pi">:</span>
            <span class="pi">-</span> <span class="s">values.yaml</span>
      <span class="na">destination</span><span class="pi">:</span>
        <span class="na">server</span><span class="pi">:</span> <span class="s2">"</span><span class="s">{{cluster}}"</span>
        <span class="na">namespace</span><span class="pi">:</span> <span class="s2">"</span><span class="s">{{namespace}}"</span>
      <span class="na">syncPolicy</span><span class="pi">:</span>
        <span class="na">automated</span><span class="pi">:</span>
          <span class="na">prune</span><span class="pi">:</span> <span class="kc">true</span>
          <span class="na">selfHeal</span><span class="pi">:</span> <span class="kc">true</span>
        <span class="na">syncOptions</span><span class="pi">:</span>
          <span class="pi">-</span> <span class="s">CreateNamespace=true</span>
          <span class="pi">-</span> <span class="s">RespectIgnoreDifferences=true</span>
</code></pre>

</div>



<p><strong>Istio VirtualService for canary traffic split:</strong><br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight yaml"><code><span class="na">apiVersion</span><span class="pi">:</span> <span class="s">networking.istio.io/v1beta1</span>
<span class="na">kind</span><span class="pi">:</span> <span class="s">VirtualService</span>
<span class="na">metadata</span><span class="pi">:</span>
  <span class="na">name</span><span class="pi">:</span> <span class="s">fraud-detector-vs</span>
  <span class="na">namespace</span><span class="pi">:</span> <span class="s">ml-serving-prod</span>
<span class="na">spec</span><span class="pi">:</span>
  <span class="na">hosts</span><span class="pi">:</span>
    <span class="pi">-</span> <span class="s">fraud-detector.ml-serving-prod.svc.cluster.local</span>
  <span class="na">http</span><span class="pi">:</span>
    <span class="pi">-</span> <span class="na">match</span><span class="pi">:</span>
        <span class="pi">-</span> <span class="na">headers</span><span class="pi">:</span>
            <span class="na">x-canary</span><span class="pi">:</span>
              <span class="na">exact</span><span class="pi">:</span> <span class="s2">"</span><span class="s">true"</span>
      <span class="na">route</span><span class="pi">:</span>
        <span class="pi">-</span> <span class="na">destination</span><span class="pi">:</span>
            <span class="na">host</span><span class="pi">:</span> <span class="s">fraud-detector-predictor-canary</span>
            <span class="na">port</span><span class="pi">:</span>
              <span class="na">number</span><span class="pi">:</span> <span class="m">80</span>
          <span class="na">weight</span><span class="pi">:</span> <span class="m">100</span>
    <span class="pi">-</span> <span class="na">route</span><span class="pi">:</span>
        <span class="pi">-</span> <span class="na">destination</span><span class="pi">:</span>
            <span class="na">host</span><span class="pi">:</span> <span class="s">fraud-detector-predictor-default</span>
            <span class="na">port</span><span class="pi">:</span>
              <span class="na">number</span><span class="pi">:</span> <span class="m">80</span>
          <span class="na">weight</span><span class="pi">:</span> <span class="m">90</span>
        <span class="pi">-</span> <span class="na">destination</span><span class="pi">:</span>
            <span class="na">host</span><span class="pi">:</span> <span class="s">fraud-detector-predictor-canary</span>
            <span class="na">port</span><span class="pi">:</span>
              <span class="na">number</span><span class="pi">:</span> <span class="m">80</span>
          <span class="na">weight</span><span class="pi">:</span> <span class="m">10</span>
</code></pre>

</div>



<p>After the PR merges to dev, ArgoCD picks up the change within 3 minutes (the default sync interval) and applies the updated InferenceService. The model downloads from S3, the serving pod comes up, and the endpoint starts responding. At this point you can run your automated evaluation suite against the dev endpoint.</p>

<p>Promoting to staging is another PR. A human reviews it, checks the dev evaluation results, and approves. Merge, ArgoCD syncs, done. Production promotion follows the same pattern but includes an additional step: the canary InferenceService gets deployed first with 10% traffic, and a GitHub Actions workflow monitors Prometheus metrics for a configured soak period (we use 2 hours for most models) before opening the full-cutover PR automatically.</p>

<h2>
  
  
  Drift Detection
</h2>

<p>Prediction drift is the sneaky failure mode. The model is technically serving, latency looks fine, but the distribution of predictions has shifted because the input data changed. You won't catch this with a liveness probe.</p>

<p><a href="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fya0whws7chpxtyg3q0me.png" class="article-body-image-wrapper"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fya0whws7chpxtyg3q0me.png" alt=" " width="800" height="622"></a></p>

<p>KServe's sklearn server exposes prediction histograms as Prometheus metrics out of the box. You define alerting rules that fire when the distribution deviates beyond a threshold from the baseline captured at deployment time.</p>

<p><strong>Prometheus PrometheusRule for drift alerting:</strong><br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight yaml"><code><span class="na">apiVersion</span><span class="pi">:</span> <span class="s">monitoring.coreos.com/v1</span>
<span class="na">kind</span><span class="pi">:</span> <span class="s">PrometheusRule</span>
<span class="na">metadata</span><span class="pi">:</span>
  <span class="na">name</span><span class="pi">:</span> <span class="s">fraud-detector-drift</span>
  <span class="na">namespace</span><span class="pi">:</span> <span class="s">ml-serving-prod</span>
  <span class="na">labels</span><span class="pi">:</span>
    <span class="na">prometheus</span><span class="pi">:</span> <span class="s">kube-prometheus</span>
    <span class="na">role</span><span class="pi">:</span> <span class="s">alert-rules</span>
<span class="na">spec</span><span class="pi">:</span>
  <span class="na">groups</span><span class="pi">:</span>
    <span class="pi">-</span> <span class="na">name</span><span class="pi">:</span> <span class="s">fraud-detector.drift</span>
      <span class="na">interval</span><span class="pi">:</span> <span class="s">2m</span>
      <span class="na">rules</span><span class="pi">:</span>
        <span class="pi">-</span> <span class="na">alert</span><span class="pi">:</span> <span class="s">PredictionDriftDetected</span>
          <span class="na">expr</span><span class="pi">:</span> <span class="pi">|</span>
            <span class="s">abs(</span>
              <span class="s">avg_over_time(fraud_detector_prediction_mean[10m])</span>
              <span class="s">- avg_over_time(fraud_detector_prediction_mean[60m] offset 1d)</span>
            <span class="s">) &gt; 0.15</span>
          <span class="na">for</span><span class="pi">:</span> <span class="s">10m</span>
          <span class="na">labels</span><span class="pi">:</span>
            <span class="na">severity</span><span class="pi">:</span> <span class="s">warning</span>
            <span class="na">model</span><span class="pi">:</span> <span class="s">fraud-detector</span>
            <span class="na">env</span><span class="pi">:</span> <span class="s">prod</span>
          <span class="na">annotations</span><span class="pi">:</span>
            <span class="na">summary</span><span class="pi">:</span> <span class="s2">"</span><span class="s">Prediction</span><span class="nv"> </span><span class="s">distribution</span><span class="nv"> </span><span class="s">shift</span><span class="nv"> </span><span class="s">detected</span><span class="nv"> </span><span class="s">for</span><span class="nv"> </span><span class="s">fraud-detector"</span>
            <span class="na">description</span><span class="pi">:</span> <span class="s2">"</span><span class="s">Mean</span><span class="nv"> </span><span class="s">prediction</span><span class="nv"> </span><span class="s">shifted</span><span class="nv"> </span><span class="s">by</span><span class="nv"> </span><span class="s">{{</span><span class="nv"> </span><span class="s">$value</span><span class="nv"> </span><span class="s">|</span><span class="nv"> </span><span class="s">humanizePercentage</span><span class="nv"> </span><span class="s">}}</span><span class="nv"> </span><span class="s">from</span><span class="nv"> </span><span class="s">yesterday's</span><span class="nv"> </span><span class="s">baseline.</span><span class="nv"> </span><span class="s">Check</span><span class="nv"> </span><span class="s">for</span><span class="nv"> </span><span class="s">input</span><span class="nv"> </span><span class="s">data</span><span class="nv"> </span><span class="s">schema</span><span class="nv"> </span><span class="s">changes."</span>

        <span class="pi">-</span> <span class="na">alert</span><span class="pi">:</span> <span class="s">ModelLatencyHigh</span>
          <span class="na">expr</span><span class="pi">:</span> <span class="pi">|</span>
            <span class="s">histogram_quantile(0.99,</span>
              <span class="s">sum(rate(fraud_detector_request_duration_seconds_bucket[5m])) by (le)</span>
            <span class="s">) &gt; 0.5</span>
          <span class="na">for</span><span class="pi">:</span> <span class="s">5m</span>
          <span class="na">labels</span><span class="pi">:</span>
            <span class="na">severity</span><span class="pi">:</span> <span class="s">critical</span>
            <span class="na">model</span><span class="pi">:</span> <span class="s">fraud-detector</span>
            <span class="na">env</span><span class="pi">:</span> <span class="s">prod</span>
          <span class="na">annotations</span><span class="pi">:</span>
            <span class="na">summary</span><span class="pi">:</span> <span class="s2">"</span><span class="s">p99</span><span class="nv"> </span><span class="s">latency</span><span class="nv"> </span><span class="s">above</span><span class="nv"> </span><span class="s">500ms</span><span class="nv"> </span><span class="s">for</span><span class="nv"> </span><span class="s">fraud-detector"</span>
            <span class="na">description</span><span class="pi">:</span> <span class="s2">"</span><span class="s">p99</span><span class="nv"> </span><span class="s">latency</span><span class="nv"> </span><span class="s">is</span><span class="nv"> </span><span class="s">{{</span><span class="nv"> </span><span class="s">$value</span><span class="nv"> </span><span class="s">}}s.</span><span class="nv"> </span><span class="s">SLA</span><span class="nv"> </span><span class="s">threshold</span><span class="nv"> </span><span class="s">is</span><span class="nv"> </span><span class="s">500ms."</span>

        <span class="pi">-</span> <span class="na">alert</span><span class="pi">:</span> <span class="s">ModelErrorRateHigh</span>
          <span class="na">expr</span><span class="pi">:</span> <span class="pi">|</span>
            <span class="s">rate(fraud_detector_request_total{status_code=~"5.."}[5m])</span>
            <span class="s">/</span>
            <span class="s">rate(fraud_detector_request_total[5m]) &gt; 0.01</span>
          <span class="na">for</span><span class="pi">:</span> <span class="s">5m</span>
          <span class="na">labels</span><span class="pi">:</span>
            <span class="na">severity</span><span class="pi">:</span> <span class="s">critical</span>
            <span class="na">model</span><span class="pi">:</span> <span class="s">fraud-detector</span>
            <span class="na">env</span><span class="pi">:</span> <span class="s">prod</span>
          <span class="na">annotations</span><span class="pi">:</span>
            <span class="na">summary</span><span class="pi">:</span> <span class="s2">"</span><span class="s">Error</span><span class="nv"> </span><span class="s">rate</span><span class="nv"> </span><span class="s">above</span><span class="nv"> </span><span class="s">1%</span><span class="nv"> </span><span class="s">for</span><span class="nv"> </span><span class="s">fraud-detector"</span>
</code></pre>

</div>



<p>When this alert fires, it sends to PagerDuty (or your alert routing of choice via AlertManager). The on-call engineer's first action is to check whether a canary is active. If it is, rolling back is a single command:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight shell"><code>git revert HEAD~1
git push origin main
</code></pre>

</div>



<p>ArgoCD detects the revert within 3 minutes and redeploys the previous InferenceService version. In practice, our rollbacks averaged 4 minutes from decision to stable serving.</p>

<h2>
  
  
  Results
</h2>

<div class="table-wrapper-paragraph"><table>
<thead>
<tr>
<th>Metric</th>
<th>Before</th>
<th>After</th>
</tr>
</thead>
<tbody>
<tr>
<td>Time to deploy new model version</td>
<td>4 to 6 hours</td>
<td>8 minutes to production canary</td>
</tr>
<tr>
<td>Rollback capability</td>
<td>None (manual rebuild)</td>
<td>
<code>git revert</code>, avg 4 minutes</td>
</tr>
<tr>
<td>Drift detection time</td>
<td>6 hours (user reports)</td>
<td>15 minutes (automated alert)</td>
</tr>
<tr>
<td>Deployment audit trail</td>
<td>Slack messages</td>
<td>Full Git history with PR reviews</td>
</tr>
<tr>
<td>Environment parity</td>
<td>Best effort</td>
<td>Enforced via ApplicationSet</td>
</tr>
<tr>
<td>Config drift prevention</td>
<td>None</td>
<td>ArgoCD selfHeal</td>
</tr>
</tbody>
</table></div>

<p>The number that surprised me most was the drift detection improvement. We caught a data schema change within 15 minutes on the new system. The same type of change previously went undetected for 6 hours before a user complaint surfaced it. That's not a monitoring win, it's a business outcome.</p>

<h2>
  
  
  Lessons Learned
</h2>

<p><strong>Start with the values.yaml contract.</strong> The shape of that file is the most important design decision you'll make. Get the team to agree on it before writing any ArgoCD config. Everything else follows from it.</p>

<p><strong>S3 artifact URIs in the InferenceService spec, not model names.</strong> MLflow stage names ("Production", "Staging") are mutable. If you reference a stage name in your InferenceService spec, two different model versions could map to the same stage name over time, and your Git history loses meaning. Reference the explicit S3 URI with the version number baked in.</p>

<p><strong>selfHeal is non-negotiable.</strong> Turn it on in your ArgoCD sync policy. Without selfHeal, a manual kubectl edit on the InferenceService will drift silently and nobody will notice until it matters.</p>

<p><strong>Canary soak time depends on your traffic volume.</strong> For a high-volume fraud model processing 50k requests per minute, 30 minutes of canary is enough to get statistically significant signal. For a low-volume model processing 100 requests per day, 2 hours of canary at 10% gives you 20 requests through the new version. Adjust accordingly, or route specific customers to the canary instead of random percentage splitting.</p>

<p><strong>Model cold start affects canary rollouts.</strong> Large models take time to download from S3 and load into memory. A 2GB model on a cold node might take 3 to 4 minutes before it's ready to serve. Account for this in your readiness probe timeouts and don't let your monitoring system flag the canary as failing during the startup window.</p>

<h2>
  
  
  Try It Yourself
</h2>

<p>The repository structure I've described looks like this:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight plaintext"><code>ml-gitops/
├── environments/
│   ├── dev/
│   │   ├── values.yaml
│   │   └── templates/
│   │       ├── inference-service.yaml
│   │       └── virtual-service.yaml
│   ├── staging/
│   │   ├── values.yaml
│   │   └── templates/
│   └── prod/
│       ├── values.yaml
│       └── templates/
├── base/
│   ├── inference-service-template.yaml
│   └── prometheus-rules.yaml
└── applicationset.yaml
</code></pre>

</div>



<p>Prerequisites before you start:</p>

<ul>
<li>Kubernetes cluster (1.28 or newer)</li>
<li>KServe 0.12 or newer installed</li>
<li>ArgoCD 2.9 or newer installed</li>
<li>Istio 1.20 or newer installed</li>
<li>MLflow tracking server accessible from the cluster</li>
<li>S3 bucket with appropriate IRSA or Workload Identity configured for KServe pods</li>
</ul>

<p>The ArgoCD ApplicationSet in this post assumes a Helm-based templating approach where each environment folder contains a values.yaml and a templates directory with the InferenceService and VirtualService manifests. You could also use Kustomize overlays. The concepts are identical.</p>

<p>Start with dev only. Get one model version deploying cleanly through ArgoCD before adding staging and prod. Add the canary workflow only after the basic promotion gate is working reliably.</p>

<p>The jump from "it works in dev" to "it's reliable in prod" is mostly about the Prometheus alerting and the canary soak automation. Those two pieces are what make the system trustworthy enough for the team to stop second-guessing every deployment.</p>




<p><strong>Resources:</strong></p>

<ul>
<li><a href="https://kserve.github.io/website/" rel="noopener noreferrer">KServe Documentation</a></li>
<li><a href="https://argo-cd.readthedocs.io/en/stable/user-guide/application-set/" rel="noopener noreferrer">ArgoCD ApplicationSets</a></li>
<li><a href="https://mlflow.org/docs/latest/model-registry.html" rel="noopener noreferrer">MLflow Model Registry</a></li>
<li><a href="https://istio.io/latest/docs/concepts/traffic-management/" rel="noopener noreferrer">Istio Traffic Management</a></li>
<li><a href="https://prometheus-operator.dev/docs/operator/api/" rel="noopener noreferrer">Prometheus Operator API</a></li>
</ul>

