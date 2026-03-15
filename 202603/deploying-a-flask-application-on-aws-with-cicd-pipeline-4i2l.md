---
Title: Deploying a Flask Application On AWS with CI/CD Pipeline
Description: 
Author: lmprojectdevto
Date: 2026-03-15T21:36:38.000Z
Robots: noindex,nofollow
Template: index
---
<h2>
  
  
  <strong>Python Flask Application</strong>
</h2>

<p>This project was to create a web application, that's a daily movie quiz. </p>

<p>This quiz will first display the year of release and genres of a movie alongside the Rotten Tomatoes critic and audience score. The user will enter a guess, if they are incorrect, the next clue given will be a snippet of a review provided by Rotten Tomatoes.<br>
The user will be given a number of attempts and reviews before being told the answer and the next day a new set of clue will be provided for a new movie.</p>

<p><a href="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fsxkc95cysxuuwghrlilr.png" class="article-body-image-wrapper"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fsxkc95cysxuuwghrlilr.png" alt=" " width="800" height="487"></a></p>

<p>To create this web application I used Python Flask.</p>

<p>Python Flask is a web application framework, allowing for the creation of web applications and APIs. </p>

<p>This project uses this framework to create a web page and API that sends a GET request to a PostgreSQL database, and display the results of this request on the HTML web page.</p>

<h2>
  
  
  <strong>JavaScript</strong>
</h2>

<p>To dynamically display the returned data from the GET request, JavaScript is used to take advantage of DOM Manipulation.<br>
DOM Manipulation is used to access, and change the structure, content, and style of the Web Pages HTML and CSS after it has loaded.</p>

<p>For example when this application is initially loaded a GET request is sent by the Python Flask backend. The values returned for that days Film title, synopsis, genres and scores are then added to the HTML elements. </p>

<p>After a unsuccessful guess, a new container is created on the page and the values returned from the GET request are dynamically added to the elements inside it, depending on the number of incorrect guesses they have had so far. </p>

<h2>
  
  
  <strong>Gunicorn</strong>
</h2>

<p>Gunicorn is a WSGI, Web Server Gateway Interface, application Server that is designed to work with Python web applications. </p>

<p>This application server serves and loads the web app, and creates multiple workers that are used to handle numerous requests simultaneously. In this application, I created a config file which I call when ran. This Python config file dynamically identifies the number of worker processes to generate by looking at the number of logical CPUs available. This allows for the application to scale dynamically if more resources in the future are provided. </p>

<h2>
  
  
  <strong>GitHub CI/CD pipeline</strong>
</h2>

<p>A private GitHub repository was created, allowing one for version control of the web application being developed, and two create a CI/CD pipeline to AWS.</p>

<p>This CI/CD pipeline to AWS pushes a new container image to ECR, Elastic Container Registry, which can then be used to easily deploy on AWS services. The .yml file first configures AWS credentials which are first provided as secrets in GitHub. </p>

<p>After this, it logs into Amazon ECR. It then builds the docker image of the application, if it is successful it pushes the image to ECR. </p>

<p><a href="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fpjdsejbqvsivuz8a4b4s.png" class="article-body-image-wrapper"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fpjdsejbqvsivuz8a4b4s.png" alt=" " width="800" height="464"></a><br>
The AWS configuration that will be built.</p>

<h2>
  
  
  <strong>EC2</strong>
</h2>

<p>Elastic Compute Cloud provides virtual servers/instances in the cloud. These are on demand instances which allow for us to rent compute capacity without having to manage the physical infrastructure. </p>

<p>An EC2 instance was chosen to host this application. As this a small web application not expecting much traffic, a t2.micro instance is chosen. This will also keep costs low. <br>
Ports 80, 443 and 22 are open. Allowing for web traffic to the instance and allowing for myself to SSH into the instance when needed, for maintenance and pulling images from ECR. </p>

<p>A IAM Role is added to this EC2 instance, with a policy allowing for it to  read the containers uploaded to ECR.<br>
Once updates on the instance are ran, the container is pulled from ECR. The docker image is then built and ran on the EC2 instance. </p>

<p>At this moment this Python Flask web application is ran behind on the application http server Gunicorn on port 8000. <br>
A web server was now needed to serve as a reverse proxy server and better handle client connections. </p>

<h2>
  
  
  <strong>Nginx</strong>
</h2>

<p>Nginx is a web server that acts as a reverse proxy for my Python Flask application. This also allows for multiple client connections and load balancing making it highly scalable and flexible. </p>

<p>Using Certbot on EC2 also allows you to obtain, install and renew SSL/TLS certificates. When ran this automatically updates the Nginx config file, allowing for a secure connection to the web application over HTTPS. </p>

<h2>
  
  
  <strong>RDS</strong>
</h2>

<p>During development the web application connects to a PostgreSQL database created locally on my machine. </p>

<p>In the AWS production environment an RDS instance has been created. <br>
Relational Database Service is a service provided by AWS, allowing for relational databases to be easily created, and administrative operations such as patching and backups are done automatically. </p>

<p>The RDS instance chosen was a db.t4g.micro. This again was chosen as to keep costs low, and because not much traffic is expected for the web application.<br>
This is then connected to the EC2 instance, in the same VPC in the same Availability Zone.</p>

<h2>
  
  
  <strong>Route 53</strong>
</h2>

<p>Route 53 is a DNS, Domain Name System, that allows for domains to be easily routed to IP addresses used by AWS infrastructure. Route 53 was used to route the domain <a href="//guessthatmovie.today">guessthatmovie.today</a> to the elastic IP address assigned to the EC2 instance hosting the web app.</p>

<h2>
  
  
  <strong>Conclusion - What I learnt &amp; Future improvements</strong>
</h2>

<p>Taking on this project allowed me to get more hands on with AWS services after completing the Certified Cloud Practitioner certification. </p>

<p>One important thing I had learnt was the importance of setting up CI/CD pipelines and AWS infrastructure earlier in development. This would of helped avoid headaches I encountered when finally trying to identify how the application would be deployed onto AWS, after getting the flask app to a stage I was happy with. </p>

<p>In regard to future improvements to AWS infrastructure. I would like to look at using a infrastructure that uses the Elastic Load Balancer, allowing for EC2 instances to be automatically generated when needed in an new AZ. The Web Application Firewall could also be used in conjunction with this.</p>

<p>I will also look to familiarise myself with Terraform. To easily create AWS infrastructure via code. </p>

<p>I would look to incorporate Redis cache to the flask application. This would be with a view to limit the amount of GET requests to the RDS instance. </p>

<p>The Flask app would also need a number of improvements to the look of the application itself. This would include making a clear indication on the number of attempts left, a list of the previous answers given by the user and an update on the styling to make it easier to look at for mobile users.</p>

