---
Title: Setting my own rules
Description: 
Author: Xauntasia Mabry
Date: 2026-02-07T21:45:21.000Z
Robots: noindex,nofollow
Template: index
---
<p>First, let me explain the title. In my normal work, I infrequently get the opportunity to build with AWS services that have not been approved for use within my company. So that's why, when I use services like AWS Amplify in my personal project knowing it's not approved in my 9-5, it feels a little scandalous. Like I'm jumping out the window to go to a party I've been told I can't attend. I didn't know this kind of feeling would be part of the journey creating my own stuff, but it's hilarious to finally experience setting my own rules in how I want to use technology. Like new levels unlocked I didn't see coming. I like it here.</p>

<p>Now that you understand that, I'd take a moment to double-click on AWS Amplify from <a href="https://dev.to/xmabry/my-problem-has-a-frontend-2fd5">this previous post</a> and how I've gotten a chance to learn about it in the midst of my frontend development efforts. Amplify lives up to the expectation of being able to deploy something quickly. Just as fast as I got Copilot to help me create a frontend app using React framework, Amplify immediately deployed it. I had the Amplify application set up to auto-detect branches and to auto-build so from this point on as long as I pushed new changes to the app that compiled, the build and deployment was completely handled.</p>

<p>What I am choosing not to do is to directly integrate the Amplify with functions, storage and data options. That seems to be a major perk in having Amplify be able to interact with other AWS services on your behalf, but for this I'd like to have a little more control. I've chosen to use API Gateway to control access to my Lambdas so that I can implement granular validation and error handling for requests with API Gateway. I also want to protect with a managed authorizer the Lambdas that will be calling Bedrock Models for generating content and downloading content from S3. Last reason is because I'm not willing to write the backend in TypeScript lol. React Frontend is enough learning for me in 2026. Backend will be in a language I'm more comfortable with.</p>

<p>In a <a href="https://dev.to/xmabry/to-terraform-orcloudformation-3c5o">previous post</a>, I asked how others chose to deploy their resources for personal projects. This post is an indication I chose to create TF after manual deployment just for lack of familiarity with the Amplify resources in the provider. With that, I'm testing myself to see if I can quickly come up with the Terraform necessary to deploy the same resource and configured the way I did it in console. </p>

<p>This is actually one of the points of friction I've heard from development teams in my 9-5. Coming up with Terraform to deploy the resources hasn't always been an easy dance. I can attest to this from my own previous experience, but this particular experience with Amplify highlighted this further. The current AWS Terraform provider doesn't give you resources to manage or customize the monitoring or alerting for the application, so that still needs to be done in console.</p>

<p>Here's a code snippet of the way I initially configured Amplify to deploy my frontend app:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight plaintext"><code>resource "aws_amplify_app" "homeschool_app" {
  name       = "homeschool-app"
  repository = "https://github.com/homeschool-app" # Replace with your GH repository

  build_spec = &lt;&lt;-EOT
    version: 1
    frontend:
    phases:
        preBuild:
        commands:
            - npm ci
        build:
        commands:
            - npm run build
    artifacts:
        baseDirectory: build
        files:
        - '**/*'
    cache:
        paths:
        - node_modules/**/*

  EOT

  enable_auto_branch_creation = true
  auto_branch_creation_patterns = [
    "*",
    "*/**",
  ]
  auto_branch_creation_config {
    enable_auto_build = true
  }

  environment_variables = {

# Cognito User Pool Domain (without the .auth.region.amazoncognito.com part). This is important
VITE_COGNITO_DOMAIN=your-cognito-domain

# Cognito User Pool App Client ID
VITE_COGNITO_CLIENT_ID=your-cognito-client-id
# Example Usage: VITE_COGNITO_CLIENT_ID=1a2b3c4d5e6f7g8h9i0j1k2l3m 

# AWS Region (optional - defaults to us-east-1)
VITE_AWS_REGION=us-east-1

# Redirect URI after successful login (optional - defaults to current origin)
VITE_COGNITO_REDIRECT_URI=http://localhost:3000 # Local development redirect 
# For production: VITE_COGNITO_REDIRECT_URI=https://your-domain.com

# API Configuration
# Main API Gateway URL for backend endpoints
VITE_API_ENDPOINT=https://your-api-gateway.execute-api.us-east-1.amazonaws.com/prod

  }
}
</code></pre>

</div>



