---
Title: I Built a Self-Updating ML Model That Handles Traffic With Ease - Here’s How
Description: 
Author: Matia Rašetina
Date: 2025-09-15T21:48:29.000Z
Robots: noindex,nofollow
Template: index
---
<p>Most ML models die in notebooks. I built one that retrains itself, scales to hundreds of users, and deploys in minutes — here’s the exact AWS + Docker + CI/CD stack I used.</p>

<p>In this project, to keep the costs as low as possible, the Machine Learning model was trained on the local machine then uploaded the artifacts to the AWS platform. This project also demonstrates an entry-level MLOps workflow — retraining and redeployment are automated through CI/CD, but advanced MLOps features like data versioning, model registry, and monitoring are not yet implemented.</p>

<p>There are a few prerequisites for this project and they are the following:</p>

<ul>
<li>Docker</li>
<li>Python</li>
<li>CI/CD pipelines</li>
<li>basic usage of GitHub</li>
<li>basic knowledge of AWS services</li>
</ul>

<p>Link to the code is <a href="https://github.com/mate329/student-results-prediction-sagemaker-api" rel="noopener noreferrer">here</a> — let’s get started!</p>

<h2>
  
  
  <strong>Part 1: XGBoost Model Training with Optuna</strong>
</h2>

<p>The dataset for this project was taken from the following Kaggle dataset — <a href="https://www.kaggle.com/datasets/lainguyn123/student-performance-factors/data" rel="noopener noreferrer">link to dataset</a> — this dataset is a great example of a simple regression problem, with clear features and data ready to be used for training a Machine Learning model. We are going to use XGBoost model and there are several reasons:</p>

<ul>
<li>gradient boosting trees, particularly XGBoost, consistently outperforms deep learning models on tabular data with evidence that it won most of the Kaggle challenges in 2015 — <a href="https://arxiv.org/pdf/1603.02754" rel="noopener noreferrer">link</a>
</li>
<li>XGBoost implements L1 and L2 regularization on tree weights, which reduces overfitting</li>
</ul>

<p>Other model structures, like deep learning models, might seem more appealing, but with so little data and only 15 features, XGBoost typically outperforms deep learning models on tabular data this size.</p>

<p>This makes it a great candidate for this project. There are other models which are also worth trying out, but won’t be implemented in this blog post.</p>

<p>To ensure we are deploying the best model possible with the available data, we need to optimize our model’s hyperparameters. As in the previous blog post, we are going to use Python library called Optuna.</p>

<p>Optuna is a hyperparameter optimization library and it has become the go-to library for quick and easy Machine Learning workflows.</p>

<p>At the end of model optimization and training, the training script will output 2 files - <code>final_model.pkl</code> and <code>label_encoders.pkl</code> , which will be put inside a tarball and uploaded to S3.</p>

<p>We chose local training since our training dataset is very small and it only takes a couple of minutes to train the XGBoost model. AWS Sagemaker does provide a platform for training your models, which makes it very useful if you have big datasets or need a more powerful machine to train your models, but for this use case, local training saves us the money by keeping us inside the AWS Free Tier.</p>

<p>Here is the output of the training process:</p>

<p><a href="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fbjytounv7gtizca81led.png" class="article-body-image-wrapper"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fbjytounv7gtizca81led.png" alt="XGBoost + Optuna training output" width="800" height="181"></a></p>

<p>In the image itself, you can see that we’ve logged the metrics like Mean Square Error (MSE), R^2 score, Root Mean Squared Error (RMSE) and Mean Absolute Error (MAE). These metrics are very important to assess model performance. For a more detailed explanation why do we use these metrics for assessing model performance, please check out <a href="https://medium.com/analytics-vidhya/complete-guide-to-machine-learning-evaluation-metrics-615c2864d916" rel="noopener noreferrer">this Medium article</a>.</p>

<h2>
  
  
  <strong>Part 2: Custom Container Strategy</strong>
</h2>

<p>Here’s where AWS defaults fall short.</p>

<p>We are going to use a custom Docker container instead of public Sagemaker Docker images provided by AWS. The reason is that we want to use the latest libraries instead of the outdated ones which are inside the pre-built AWS Docker containers (which you can see here - <a href="https://docs.aws.amazon.com/sagemaker/latest/dg-ecr-paths/ecr-eu-central-1.html#xgboost-eu-central-1" rel="noopener noreferrer">https://docs.aws.amazon.com/sagemaker/latest/dg-ecr-paths/ecr-eu-central-1.html#xgboost-eu-central-1</a>). At the time of writing this blog, the latest package version for the XGBoost library is 1.7.4, while on PIP, latest available version is 3.0.5.</p>

<p>Choosing Sagemaker’s pre-built Docker images would handle deployment automatically, but we lose the control over the dependencies and it takes away the ability of testing locally. The usage of custom containers, like in this project, adds complexity but if you were doing this on your job, this approach would enable you a more reproducible environment with faster iteration cycles.</p>

<p>Our architectural decision to create our custom Docker image has a drawback - we will need to write additional logic to handle 2 HTTP API calls which Sagemaker does by design, those are <code>/invocations</code> and <code>/ping</code> . </p>

<p>When a resource wants the model to return a value, it will provide the necessary payload to the <code>/invocations</code> endpoint, while the <code>/ping</code> endpoint is used as a health-check endpoint - Sagemaker will periodically send a GET HTTP request to it to verify that everything is up and running - that’s why we’ll need to install Flask - a lightweight Python library for creating servers in Python. </p>

<p>If we used the pre-built Docker image, we would just need to provide the model artifacts and the <code>interefece.py</code> file, which contains the logic to get the prediction from the model. On the other hand, there is a great benefit to this approach - we are able to test it locally by running the Docker image and we are going to take a look at testing the model locally in the next few steps.</p>

<p>The following code block is our Dockerfile and let’s explain it step by step:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight docker"><code><span class="c"># Set the linux/amd64 platform as a base for this Docker Image</span>
<span class="c"># so even if you have an ARM processor, it won't impact the runtime</span>
<span class="c"># Common mistake here is a mismatch of the installed libraries because of the</span>
<span class="c"># host machine and the machine which runs the image - e.g. if you build the image</span>
<span class="c"># on an ARM processor, it will only work on machines with that architecture CPU</span>
<span class="c"># and not work on regular x86 processors.</span>
<span class="k">ARG</span><span class="s"> TARGETPLATFORM=linux/amd64</span>
<span class="k">FROM</span><span class="s"> --platform=$TARGETPLATFORM python:3.12-slim</span>

<span class="c"># Set the working directory in the container</span>
<span class="k">WORKDIR</span><span class="s"> /app</span>

<span class="c"># Copy the dependencies file to the working directory</span>
<span class="k">COPY</span><span class="s"> requirements.txt .</span>

<span class="c"># Install Python dependencies</span>
<span class="k">RUN </span>pip <span class="nb">install</span> <span class="nt">--no-cache-dir</span> <span class="nt">-r</span> requirements.txt

<span class="c"># Copy the Flask application code into the container</span>
<span class="k">COPY</span><span class="s"> . .</span>

<span class="c"># Set the environment variable for the SageMaker model directory</span>
<span class="k">ENV</span><span class="s"> SM_MODEL_DIR=/opt/ml/model</span>

<span class="c"># Expose port 8080 to allow external access to the Flask application</span>
<span class="k">EXPOSE</span><span class="s"> 8080</span>

<span class="c"># Define the entry point for running the Flask application</span>
<span class="k">ENTRYPOINT</span><span class="s"> ["gunicorn", "-b", "0.0.0.0:8080", "app:app"]</span>
</code></pre>

</div>



<p>Here are the following commands which you need to run in the <code>aws</code> folder to build the image and run the ML model locally:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight shell"><code><span class="c"># Build the image locally</span>
<span class="nb">cd </span>aws/docker
docker build <span class="nt">-t</span> sagemaker-demo <span class="nb">.</span>

<span class="c"># Run the Docker image to create a container for local testing</span>
docker run <span class="nt">-p</span> 8080:8080 <span class="nt">-v</span> ./model.tar.gz:/opt/ml/model/model.tar.gz sagemaker-demo
</code></pre>

</div>



<p>This way, we are simulating the Sagemaker environment and we can use API testing tools, like Postman, to see if our model is working as expected.</p>

<p>I’ve used Postman as the API testing tool to verify that the Flask server is working by sending the GET payload to the <code>/ping</code>, that the model is loaded and returning the expected student score based on the received payload when the POST <code>/invocations</code> endpoint is called.</p>

<p><a href="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fuu5egejt4minpn9xilv9.png" class="article-body-image-wrapper"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fuu5egejt4minpn9xilv9.png" width="800" height="447"></a></p>

<p><a href="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fxjccqwy4h2ui3fxwlxaa.png" class="article-body-image-wrapper"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fxjccqwy4h2ui3fxwlxaa.png" width="800" height="506"></a></p>

<h2>
  
  
  <strong>Part 3: AWS Infrastructure as Code</strong>
</h2>

<p>AWS CDK generates the Infrastructure resources automatically and handles the complex resource dependencies. If you did this manually in the AWS console, it would be way more error-prone and would take much more time. This way, we are leveraging our Python knowledge by creating these templates. Another usage of IaC in practice is that we can deploy the same stack in multiple environments (e.g. <code>testing</code> and <code>production</code> environments) knowing that all resources are the deployed in the same way.</p>

<p>We are going to create multiple resources to be able to open up our Sagemaker model to the Internet:</p>

<ul>
<li>AWS Lambda which takes the request and forwards it to the model</li>
<li>API Gateway opens up our Lambda to the Internet</li>
<li>Docker Image asset and model training artifacts containing our Docker image with model dependencies and model artifacts respectively</li>
<li>Sagemaker Model and Endpoint using our defined Docker Image and model artifacts</li>
</ul>

<p>To visualize our infrastructure, here is an architecture diagram:</p>

<p><a href="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fvju4mlanzhcdxpvac57q.png" class="article-body-image-wrapper"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fvju4mlanzhcdxpvac57q.png" alt="Architecture Diagram of the project" width="650" height="361"></a></p>

<p>There are multiple resources which need to be defined inside the CDK file:</p>

<ol>
<li>define the path to the Dockerimage file to build our Docker image locally
</li>
</ol>

<div class="highlight js-code-highlight">
<pre class="highlight python"><code><span class="c1"># 1. Build &amp; push Docker image from local ./container folder
</span><span class="n">image_asset</span> <span class="o">=</span> <span class="n">ecr_assets</span><span class="p">.</span><span class="nc">DockerImageAsset</span><span class="p">(</span>
    <span class="n">self</span><span class="p">,</span>
    <span class="sh">"</span><span class="s">XGBImage</span><span class="sh">"</span><span class="p">,</span>
    <span class="n">directory</span><span class="o">=</span><span class="sh">"</span><span class="s">./docker</span><span class="sh">"</span><span class="p">,</span>
<span class="p">)</span>
</code></pre>

</div>



<ol>
<li>define the path to our <code>model.tar.gz</code> which contains necessary artifacts
</li>
</ol>

<div class="highlight js-code-highlight">
<pre class="highlight python"><code><span class="c1"># 2. Upload model.tar.gz to S3
</span><span class="n">model_asset</span> <span class="o">=</span> <span class="n">s3_assets</span><span class="p">.</span><span class="nc">Asset</span><span class="p">(</span>
    <span class="n">self</span><span class="p">,</span>
    <span class="sh">"</span><span class="s">XGBModel</span><span class="sh">"</span><span class="p">,</span>
    <span class="n">path</span><span class="o">=</span><span class="sh">"</span><span class="s">./model.tar.gz</span><span class="sh">"</span><span class="p">,</span>
<span class="p">)</span>
</code></pre>

</div>



<ol>
<li>define the Sagemaker model to connect the Docker image and model artifacts into one entity
</li>
</ol>

<div class="highlight js-code-highlight">
<pre class="highlight python"><code><span class="c1"># 3. Create SageMaker Model
</span><span class="n">sm_model</span> <span class="o">=</span> <span class="n">sagemaker</span><span class="p">.</span><span class="nc">CfnModel</span><span class="p">(</span>
    <span class="n">self</span><span class="p">,</span>
    <span class="sh">"</span><span class="s">XGBModelResource</span><span class="sh">"</span><span class="p">,</span>
    <span class="n">execution_role_arn</span><span class="o">=</span><span class="n">role</span><span class="p">.</span><span class="n">role_arn</span><span class="p">,</span>
    <span class="n">primary_container</span><span class="o">=</span><span class="n">sagemaker</span><span class="p">.</span><span class="n">CfnModel</span><span class="p">.</span><span class="nc">ContainerDefinitionProperty</span><span class="p">(</span>
        <span class="n">image</span><span class="o">=</span><span class="n">image_asset</span><span class="p">.</span><span class="n">image_uri</span><span class="p">,</span>
        <span class="n">model_data_url</span><span class="o">=</span><span class="n">model_asset</span><span class="p">.</span><span class="n">s3_object_url</span><span class="p">,</span>
    <span class="p">),</span>
<span class="p">)</span>

<span class="c1"># Ensure model.tar.gz upload finishes first
</span><span class="n">sm_model</span><span class="p">.</span><span class="n">node</span><span class="p">.</span><span class="nf">add_dependency</span><span class="p">(</span><span class="n">model_asset</span><span class="p">)</span>
</code></pre>

</div>



<ol>
<li>deploy the model and open it up as an endpoint - very important here is that we are using a Serverless inference configuration to lower cost
</li>
</ol>

<div class="highlight js-code-highlight">
<pre class="highlight python"><code><span class="c1"># 4. Create EndpointConfig using Serverless Inference
</span><span class="n">endpoint_config</span> <span class="o">=</span> <span class="n">sagemaker</span><span class="p">.</span><span class="nc">CfnEndpointConfig</span><span class="p">(</span>
    <span class="n">self</span><span class="p">,</span>
    <span class="sh">"</span><span class="s">XGBEndpointConfig</span><span class="sh">"</span><span class="p">,</span>
    <span class="n">production_variants</span><span class="o">=</span><span class="p">[</span>
        <span class="n">sagemaker</span><span class="p">.</span><span class="n">CfnEndpointConfig</span><span class="p">.</span><span class="nc">ProductionVariantProperty</span><span class="p">(</span>
            <span class="n">model_name</span><span class="o">=</span><span class="n">sm_model</span><span class="p">.</span><span class="n">attr_model_name</span><span class="p">,</span>
            <span class="n">initial_variant_weight</span><span class="o">=</span><span class="mf">1.0</span><span class="p">,</span>
            <span class="n">variant_name</span><span class="o">=</span><span class="sh">"</span><span class="s">AllTraffic</span><span class="sh">"</span><span class="p">,</span>
            <span class="c1"># Serverless config replaces instance_type/initial_instance_count
</span>            <span class="n">serverless_config</span><span class="o">=</span><span class="n">sagemaker</span><span class="p">.</span><span class="n">CfnEndpointConfig</span><span class="p">.</span><span class="nc">ServerlessConfigProperty</span><span class="p">(</span>
                <span class="n">memory_size_in_mb</span><span class="o">=</span><span class="mi">1024</span><span class="p">,</span>
                <span class="n">max_concurrency</span><span class="o">=</span><span class="mi">5</span><span class="p">,</span>
            <span class="p">),</span>
        <span class="p">)</span>
    <span class="p">],</span>
<span class="p">)</span>

<span class="c1"># and create the endpoint
</span><span class="n">sm_endpoint</span> <span class="o">=</span> <span class="n">sagemaker</span><span class="p">.</span><span class="nc">CfnEndpoint</span><span class="p">(</span>
    <span class="n">self</span><span class="p">,</span>
    <span class="sh">"</span><span class="s">XGBEndpoint</span><span class="sh">"</span><span class="p">,</span>
    <span class="n">endpoint_config_name</span><span class="o">=</span><span class="n">endpoint_config</span><span class="p">.</span><span class="n">attr_endpoint_config_name</span><span class="p">,</span>
<span class="p">)</span>
</code></pre>

</div>



<ol>
<li>lastly, create the Lambda which will be opened up to the Internet, so the users can access the ML model, without having to sign a request to access the model
</li>
</ol>

<div class="highlight js-code-highlight">
<pre class="highlight python"><code><span class="c1"># 5. Create the Lambda and open it to public (API Gateway)
</span><span class="n">predict_student_score_lambda</span> <span class="o">=</span> <span class="n">_lambda</span><span class="p">.</span><span class="nc">Function</span><span class="p">(</span>
    <span class="n">self</span><span class="p">,</span>
    <span class="sh">"</span><span class="s">PredictStudentScoreLambda</span><span class="sh">"</span><span class="p">,</span>
    <span class="n">runtime</span><span class="o">=</span><span class="n">_lambda</span><span class="p">.</span><span class="n">Runtime</span><span class="p">.</span><span class="n">PYTHON_3_12</span><span class="p">,</span>
    <span class="n">handler</span><span class="o">=</span><span class="sh">"</span><span class="s">lambda_handler.lambda_handler</span><span class="sh">"</span><span class="p">,</span>
    <span class="n">code</span><span class="o">=</span><span class="n">_lambda</span><span class="p">.</span><span class="n">Code</span><span class="p">.</span><span class="nf">from_asset</span><span class="p">(</span>
    <span class="sh">"</span><span class="s">./PredictStudentScore</span><span class="sh">"</span><span class="p">,</span>
    <span class="n">bundling</span><span class="o">=</span><span class="p">{</span>
        <span class="sh">"</span><span class="s">image</span><span class="sh">"</span><span class="p">:</span> <span class="n">_lambda</span><span class="p">.</span><span class="n">Runtime</span><span class="p">.</span><span class="n">PYTHON_3_12</span><span class="p">.</span><span class="n">bundling_image</span><span class="p">,</span>
        <span class="sh">"</span><span class="s">command</span><span class="sh">"</span><span class="p">:</span> <span class="p">[</span><span class="sh">"</span><span class="s">bash</span><span class="sh">"</span><span class="p">,</span> <span class="sh">"</span><span class="s">-c</span><span class="sh">"</span><span class="p">,</span> <span class="sh">"</span><span class="s">pip install aws-lambda-powertools -t /asset-output &amp;&amp; cp -r . /asset-output</span><span class="sh">"</span><span class="p">],</span>
    <span class="p">},</span>
    <span class="p">),</span>
    <span class="n">environment</span><span class="o">=</span><span class="p">{</span>
        <span class="sh">"</span><span class="s">SAGEMAKER_ENDPOINT_NAME</span><span class="sh">"</span><span class="p">:</span> <span class="n">sm_endpoint</span><span class="p">.</span><span class="n">attr_endpoint_name</span>
    <span class="p">},</span>
    <span class="n">timeout</span><span class="o">=</span><span class="n">cdk</span><span class="p">.</span><span class="n">Duration</span><span class="p">.</span><span class="nf">seconds</span><span class="p">(</span><span class="mi">30</span><span class="p">)</span>
<span class="p">)</span>

<span class="c1"># Allow Lambda to invoke SageMaker endpoint
</span><span class="n">predict_student_score_lambda</span><span class="p">.</span><span class="nf">add_to_role_policy</span><span class="p">(</span>
    <span class="n">iam</span><span class="p">.</span><span class="nc">PolicyStatement</span><span class="p">(</span>
        <span class="n">actions</span><span class="o">=</span><span class="p">[</span><span class="sh">"</span><span class="s">sagemaker:InvokeEndpoint</span><span class="sh">"</span><span class="p">],</span>
        <span class="n">resources</span><span class="o">=</span><span class="p">[</span>
            <span class="sa">f</span><span class="sh">"</span><span class="s">arn:aws:sagemaker:</span><span class="si">{</span><span class="n">self</span><span class="p">.</span><span class="n">region</span><span class="si">}</span><span class="s">:</span><span class="si">{</span><span class="n">self</span><span class="p">.</span><span class="n">account</span><span class="si">}</span><span class="s">:endpoint/</span><span class="si">{</span><span class="n">sm_endpoint</span><span class="p">.</span><span class="n">attr_endpoint_name</span><span class="si">}</span><span class="sh">"</span>
        <span class="p">]</span>
    <span class="p">)</span>
<span class="p">)</span>

<span class="c1"># API Gateway
</span><span class="n">api</span> <span class="o">=</span> <span class="n">apigw</span><span class="p">.</span><span class="nc">RestApi</span><span class="p">(</span>
    <span class="n">self</span><span class="p">,</span>
    <span class="sh">"</span><span class="s">StudentScoreApi</span><span class="sh">"</span><span class="p">,</span>
    <span class="n">rest_api_name</span><span class="o">=</span><span class="sh">"</span><span class="s">StudentScore API</span><span class="sh">"</span><span class="p">,</span>
    <span class="n">description</span><span class="o">=</span><span class="sh">"</span><span class="s">StudentScore Services API</span><span class="sh">"</span><span class="p">,</span>
    <span class="n">deploy</span><span class="o">=</span><span class="bp">True</span><span class="p">,</span>
    <span class="n">deploy_options</span><span class="o">=</span><span class="n">apigw</span><span class="p">.</span><span class="nc">StageOptions</span><span class="p">(</span>
        <span class="n">stage_name</span><span class="o">=</span><span class="sh">"</span><span class="s">score</span><span class="sh">"</span>
    <span class="p">),</span>
    <span class="n">default_cors_preflight_options</span><span class="o">=</span><span class="n">apigw</span><span class="p">.</span><span class="nc">CorsOptions</span><span class="p">(</span>
        <span class="n">allow_origins</span><span class="o">=</span><span class="n">apigw</span><span class="p">.</span><span class="n">Cors</span><span class="p">.</span><span class="n">ALL_ORIGINS</span><span class="p">,</span>
        <span class="n">allow_methods</span><span class="o">=</span><span class="n">apigw</span><span class="p">.</span><span class="n">Cors</span><span class="p">.</span><span class="n">ALL_METHODS</span><span class="p">,</span>
        <span class="n">allow_headers</span><span class="o">=</span><span class="n">apigw</span><span class="p">.</span><span class="n">Cors</span><span class="p">.</span><span class="n">DEFAULT_HEADERS</span><span class="p">,</span>
    <span class="p">),</span>
<span class="p">)</span>

<span class="c1"># API Gateway Integrations
</span><span class="n">predict_score_integration</span> <span class="o">=</span> <span class="n">apigw</span><span class="p">.</span><span class="nc">LambdaIntegration</span><span class="p">(</span><span class="n">predict_student_score_lambda</span><span class="p">)</span>

<span class="c1"># API Gateway Resources and Methods
</span><span class="n">api</span><span class="p">.</span><span class="n">root</span><span class="p">.</span><span class="nf">add_resource</span><span class="p">(</span><span class="sh">"</span><span class="s">predict-student-score</span><span class="sh">"</span><span class="p">).</span><span class="nf">add_method</span><span class="p">(</span><span class="sh">"</span><span class="s">POST</span><span class="sh">"</span><span class="p">,</span> <span class="n">predict_score_integration</span><span class="p">)</span>
</code></pre>

</div>



<p>And that’s it! Our stack is now ready for deployment. You can deploy the stack by simply running <code>cdk deploy</code>.</p>

<h2>
  
  
  <strong>Part 4: CI/CD Pipeline for Dynamic Model Updates</strong>
</h2>

<p>Another great feature of this project is implementing a CI/CD solution - we’ve implemented a Github Actions pipeline, which on dataset change, triggers the training process automatically and updates the Sagemaker endpoint in minutes! But first, let’s clarify why are we choosing Github Actions instead of AWS CodePipeline for example.</p>

<p>GitHub Actions integrates directly with our code repository and offers 2,000 free minutes monthly. AWS CodePipeline would cost $1 per pipeline per month plus compute costs.</p>

<p>Now, let’s go over the CI/CD workflow, step by step.</p>

<h3>
  
  
  Connecting our pipeline with AWS
</h3>

<p>To connect the pipeline with AWS, I found this tutorial inside the Github docs which I think is a great starting point, the link is <a href="https://docs.github.com/en/actions/how-tos/secure-your-work/security-harden-deployments/oidc-in-aws" rel="noopener noreferrer">https://docs.github.com/en/actions/how-tos/secure-your-work/security-harden-deployments/oidc-in-aws</a>. However to make it easier, I’ve created a <code>deploy_bootstrap.py</code> script inside the <code>aws/bootstrap</code> folder which will create OIDC for you as well if you don’t have it set up, together with the necessary IAM role which our pipeline will use.</p>

<p>After the <code>aws/bootstrap/deploy_bootstrap.py</code> script creates the IAM role, you will get the output from the script itself. That IAM role ARN will be used inside your Github Actions configuration file, so have it ready for the next steps. One more thing, this step needs to be done manually. The output should look like, I’ve redacted some things which are related to my AWS account:</p>

<p><a href="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fpljmb8mnh315mbr3vitf.png" class="article-body-image-wrapper"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fpljmb8mnh315mbr3vitf.png" width="800" height="411"></a></p>

<p>I’ve left steps to implement the role ARN to implement inside the Github Actions pipeline, however here is the visualization of the process itself:</p>

<ol>
<li>go to your Github repo settings</li>
</ol>

<p><a href="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fj3r4emnkn94k5cp4az0z.png" class="article-body-image-wrapper"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fj3r4emnkn94k5cp4az0z.png" width="800" height="44"></a></p>

<ol>
<li>under Security, click on “Secrets and variables” then on “Actions”</li>
</ol>

<p><a href="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fzbsz6xll4invc9helqit.png" class="article-body-image-wrapper"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fzbsz6xll4invc9helqit.png" width="674" height="483"></a></p>

<ol>
<li>click on “New Repository Secret”, enter the secret name and secret itself - in my example <code>AWS_ROLE_ARN</code> and the secret ARN itself</li>
</ol>

<p><a href="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2For79otml2do0g79j1zjn.png" class="article-body-image-wrapper"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2For79otml2do0g79j1zjn.png" width="800" height="424"></a></p>

<p>Now you can use the secret inside your pipeline configuration! We’ll input it in later steps.</p>

<h3>
  
  
  Name the workflow and define when to run it
</h3>

<p>Now, let’s define the pipeline configuration file!</p>

<p>The <code>name</code> property is self-explanatory - the name of your workflow.</p>

<p>After that, you can define on which actions to run your pipeline. In this example, the pipeline will be started in two scenarios:</p>

<ol>
<li>if the CSV dataset inside the <code>data</code> folder changes and new data is added</li>
<li>or by manually starting the pipeline</li>
</ol>

<p>YAML code which addresses the mentioned fields is in the next code block:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight yaml"><code><span class="na">name</span><span class="pi">:</span> <span class="s">Train Model</span>

<span class="na">on</span><span class="pi">:</span>
  <span class="na">push</span><span class="pi">:</span>
    <span class="na">paths</span><span class="pi">:</span>
      <span class="pi">-</span> <span class="s1">'</span><span class="s">data/**'</span>
    <span class="na">branches</span><span class="pi">:</span> <span class="pi">[</span><span class="nv">main</span><span class="pi">]</span>
  <span class="na">workflow_dispatch</span><span class="pi">:</span>
    <span class="na">inputs</span><span class="pi">:</span>
      <span class="na">model_version</span><span class="pi">:</span>
        <span class="na">description</span><span class="pi">:</span> <span class="s1">'</span><span class="s">Model</span><span class="nv"> </span><span class="s">version</span><span class="nv"> </span><span class="s">tag</span><span class="nv"> </span><span class="s">(optional)'</span>
        <span class="na">required</span><span class="pi">:</span> <span class="kc">false</span>
        <span class="na">default</span><span class="pi">:</span> <span class="s1">'</span><span class="s">latest'</span>
        <span class="na">type</span><span class="pi">:</span> <span class="s">string</span>
</code></pre>

</div>



<h3>
  
  
  Job configuration and environment setup
</h3>

<p>Now, let’s define the pipeline environment and download the necessary dependencies:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight yaml"><code><span class="na">jobs</span><span class="pi">:</span>
  <span class="na">train-model</span><span class="pi">:</span>


    <span class="na">name</span><span class="pi">:</span> <span class="s">Train Model</span>
    <span class="na">runs-on</span><span class="pi">:</span> <span class="s">ubuntu-latest</span>
    <span class="na">permissions</span><span class="pi">:</span>
      <span class="na">id-token</span><span class="pi">:</span> <span class="s">write</span>          <span class="c1"># Required for AWS OIDC authentication</span>
      <span class="na">contents</span><span class="pi">:</span> <span class="s">read</span>           <span class="c1"># Required to read repository files</span>
        <span class="na">steps</span><span class="pi">:</span>
            <span class="c1"># Checkout the code and setup Python, NPM and necessary libraries for</span>
            <span class="c1"># training the ML model</span>
          <span class="pi">-</span> <span class="na">name</span><span class="pi">:</span> <span class="s">Check out code</span>
            <span class="na">uses</span><span class="pi">:</span> <span class="s">actions/checkout@v4</span>
          <span class="pi">-</span> <span class="na">name</span><span class="pi">:</span> <span class="s">Set up Python</span>
            <span class="na">uses</span><span class="pi">:</span> <span class="s">actions/setup-python@v4</span>
            <span class="na">with</span><span class="pi">:</span>
              <span class="na">python-version</span><span class="pi">:</span> <span class="s1">'</span><span class="s">3.12'</span>
          <span class="pi">-</span> <span class="na">name</span><span class="pi">:</span> <span class="s">Set up Node.js</span>
            <span class="na">uses</span><span class="pi">:</span> <span class="s">actions/setup-node@v4</span>
            <span class="na">with</span><span class="pi">:</span>
              <span class="na">node-version</span><span class="pi">:</span> <span class="s1">'</span><span class="s">22'</span>
          <span class="pi">-</span> <span class="na">name</span><span class="pi">:</span> <span class="s">Configure AWS credentials</span>
        <span class="na">uses</span><span class="pi">:</span> <span class="s">aws-actions/configure-aws-credentials@v4</span>
        <span class="na">with</span><span class="pi">:</span>
          <span class="na">role-to-assume</span><span class="pi">:</span> <span class="s">${{ secrets.AWS_ROLE_ARN }}</span>
          <span class="na">aws-region</span><span class="pi">:</span> <span class="s">&lt;your-AWS-region&gt;</span>
          <span class="pi">-</span> <span class="na">name</span><span class="pi">:</span> <span class="s">Install dependencies</span>
              <span class="s">run</span><span class="err">:</span> <span class="pi">|</span>
                <span class="s">pip install -r requirements.txt    # Python ML libraries</span>
                <span class="s">npm install -g aws-cdk            # CDK CLI tool</span>
</code></pre>

</div>



<h3>
  
  
  Training the model and deploying to AWS via CDK
</h3>



<div class="highlight js-code-highlight">
<pre class="highlight yaml"><code><span class="pi">-</span> <span class="na">name</span><span class="pi">:</span> <span class="s">Run the train_script script</span>
  <span class="na">run</span><span class="pi">:</span> <span class="pi">|</span>
    <span class="s">python train_model.py</span>
<span class="pi">-</span> <span class="na">name</span><span class="pi">:</span> <span class="s">Deploy model with CDK</span>
  <span class="na">run</span><span class="pi">:</span> <span class="pi">|</span>
    <span class="s">cdk deploy --require-approval never</span>
  <span class="na">working-directory</span><span class="pi">:</span> <span class="s">aws</span>
  <span class="na">env</span><span class="pi">:</span>
    <span class="na">AWS_DEFAULT_REGION</span><span class="pi">:</span> <span class="s">&lt;your-AWS-region&gt;</span>

</code></pre>

</div>



<p>And that’s our pipeline configuration file!</p>

<p>The end result should look like this - first image is the complete overview of the whole job, while the second image is the process of deployment to AWS from our pipeline.</p>

<p><a href="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Ftqa2yrv5w7h3l5hcv8oq.png" class="article-body-image-wrapper"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Ftqa2yrv5w7h3l5hcv8oq.png" width="800" height="386"></a></p>

<p><a href="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fuhwrmugak78j64cokfmh.png" class="article-body-image-wrapper"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fuhwrmugak78j64cokfmh.png" width="800" height="419"></a></p>

<h1>
  
  
  Part 5: Testing the API
</h1>

<p>By using integrated Postman testing, we’ve used the following configuration to test our deployed API:</p>

<ul>
<li>“Ramp up” as our load profile</li>
<li>30 virtual users</li>
</ul>

<p>Here are the test results:</p>

<p><a href="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2F4wer8c04l34p1j9oq335.png" class="article-body-image-wrapper"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2F4wer8c04l34p1j9oq335.png" width="800" height="500"></a></p>

<p>From this test run, we can conclude the following:</p>

<ul>
<li>initial spike of latency, between 500-600ms, was caused by Lambda cold start - can be easily fixed by enabling Provisioned Concurrency, however that would increase cost</li>
<li>after Lambda cold start was resolved, our infrastructure handled traffic without any issue, returning rock-solid 100ms consistently, even when number of requests per second increased</li>
<li>no error rates, confirming that our code can handle low traffic without issues</li>
</ul>

<p>There are a couple of things which could additionally be tested:</p>

<ul>
<li>more requests with a more aggressive spike instead of ramp up</li>
<li>what is the amount of requests when our infrastructure and configuration starts throttling?</li>
</ul>

<p>For production workloads expecting higher traffic, consider testing throttling scenarios and implementing retry logic.</p>

<p>These are up to you if you want to test them, they won’t be covered in this blog post. At the time of writing this, the test cost me ~$0.05USD.</p>

<h2>
  
  
  <strong>Conclusion: Lessons Learned</strong>
</h2>

<p>In this blog post, we’ve learned many very useful skills and ways of using AWS:</p>

<ul>
<li>creating, training and optimizing our Machine Learning model based on very simple regression data

<ul>
<li>a great intro for a beginner Data Engineer</li>
</ul>


</li>

<li>using a “Custom Container” strategy to create our own environment for our ML model

<ul>
<li>we own the environment for our model - we can do whatever we want</li>
</ul>


</li>

<li>using Docker and AWS CDK with Python to build, test and deploy our application

<ul>
<li>you’ve deepened your knowledge about AWS CDK and how to test your code which is deployed</li>
</ul>


</li>

<li>connecting our AWS account with Github Actions CI/CD pipeline to have an automized way of deploying the latest version of our ML model

<ul>
<li>an easy intro to MLOps and CI/CD, which is always a great field to understand</li>
</ul>


</li>

</ul>

<p>Thank you for reading! See you in the next blog post!</p>

