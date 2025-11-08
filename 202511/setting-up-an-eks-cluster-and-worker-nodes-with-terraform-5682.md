---
Title: Setting up an EKS cluster and worker nodes with Terraform
Description: 
Author: AnamsKen
Date: 2025-11-08T20:46:12.000Z
Robots: noindex,nofollow
Template: index
---
<p>Kubernetes can be difficult even for the most experienced DevOps engineers; that stuff is a crazy monster! We will explore how to set up an Elastic Kubernetes cluster (EKS) and connect worker nodes using Terraform. <br>
 <br>
 <br>
First things first: Configure your AWS command line interface. This will give you access to run CLI commands against your AWS account, and this configuration will also allow Terraform to leverage your account to spin up infrastructures in your account. <br>
 <br>
You can check out this documentation on how to do that. <br>
 <br>
Alright, let's jump in and write some code.<br>
Create 3 major files :</p>

<p><code>touch main.tf variable.tf</code></p>

<p>In the main.tf, create the provider block and add the resource needed to create EKS cluster. As part of the requirement, we need to create the required role that EKS needs to interact with other AWS resources. We will call this eks-cluster-role</p>
<h1>
  
  
  Setting up the terraform block
</h1>


<div class="highlight js-code-highlight">
<pre class="highlight plaintext"><code>terraform {
  required_providers {
    aws = {
      source  = "hashicorp/aws"
      version = "~&gt; 6.0"
    }
  }
}

# Configure the AWS Provider
provider "aws" {
  region = "us-east-1"
}

</code></pre>

</div>


<p>Create the eks-cluster-role and attach the AmazonEKSClusterPolicy to this role<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight plaintext"><code>resource "aws_iam_role" "eks-cluster-role" {
  name = "eeks-cluster-role"
  assume_role_policy = jsonencode({
    Version = "2012-10-17"
    Statement = [
      {
        Action = [
          "sts:AssumeRole",
          "sts:TagSession"
        ]
        Effect = "Allow"
        Principal = {
          Service = "eks.amazonaws.com"
        }
      },
    ]
  })
}

#Attaching  policy to role

resource "aws_iam_role_policy_attachment" "cluster_AmazonEKSClusterPolicy" {
  policy_arn = "arn:aws:iam::aws:policy/AmazonEKSClusterPolicy"
  role       = aws_iam_role.eks-cluster-role.name
}

Create the EKS resource code
resource "aws_eks_cluster" "my-eks" {
  name = "my-eks"

  access_config {
    authentication_mode = "API"
  }

#Attaching the role we created to EKS
  role_arn = aws_iam_role.eks-cluster-role.arn
  version  = "1.31"

#Include your subnet ids here
  vpc_config {
    subnet_ids = [
      var.subnet1,
      var.subnet2,

    ]
  }

# Using API for authentication mode

access_config {

 authentication_mode = "API"
 bootstrap_cluster_creator_admin_permissions = true
}

#Setting this to true to enable core addons
bootstrap_self_managed_addons = true

  # Ensure that IAM Role permissions are created before and deleted

  depends_on = [
    aws_iam_role_policy_attachment.cluster_AmazonEKSClusterPolicy,
  ]
}
</code></pre>

</div>



<p>If you observed, we enabled some configuration to true, such as <code>bootstrap_self_managed_addons</code>. Setting this to true will install the core addons we need to run Kubernetes, such as<code>aws-cni</code>, <code>kube-proxy</code> and <code>CoreDns</code>. Ignoring this will mean we have to make these installations manually, which is not the goal of automation. <br>
 <br>
 Now let's move to setting up worker nodes using node groups. As usual, we will start by creating the roles we need for the worker nodes</p>

<p>We will be attaching three policies to this role;<br>
 <code>AmazonEKSWorkerNodePolicy</code>, <code>AmazonEKS_CNI_Policy</code> and <code>AmazonEC2ContainerRegistryReadOnly</code><br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight plaintext"><code>#Creating Node-group

resource "aws_iam_role" "node-group-role" {
  name = "eks-node-group"

  assume_role_policy = jsonencode({
    Statement = [{
      Action = "sts:AssumeRole"
      Effect = "Allow"
      Principal = {
        Service = "ec2.amazonaws.com"
      }
    }]
    Version = "2012-10-17"
  })
}

#Attaching needed policies to role

resource "aws_iam_role_policy_attachment" "node-AmazonEKSWorkerNodePolicy" {
  policy_arn = "arn:aws:iam::aws:policy/AmazonEKSWorkerNodePolicy"
  role       = aws_iam_role.node-group-role.name
}

resource "aws_iam_role_policy_attachment" "node-AmazonEKS_CNI_Policy" {
  policy_arn = "arn:aws:iam::aws:policy/AmazonEKS_CNI_Policy"
  role       = aws_iam_role.node-group-role.name
}

resource "aws_iam_role_policy_attachment" "node-AmazonEC2ContainerRegistryReadOnly" {
  policy_arn = "arn:aws:iam::aws:policy/AmazonEC2ContainerRegistryReadOnly"
  role       = aws_iam_role.node-group-role.name
}

Now we can go ahead and create our Node group
resource "aws_eks_node_group" "node-group" {
  cluster_name    = aws_eks_cluster.my-eks.name
  node_group_name = "my-node-group"
  node_role_arn   = aws_iam_role.node-group-role.arn
  subnet_ids      = [var.subnet1, var.subnet2] #You can add more subnets
  scaling_config {
    desired_size = 1
    max_size     = 2
    min_size     = 1
  }

  update_config {
    max_unavailable = 1
  }

#If you need remote access to your node groups
remote_access {
 #want to add custom security groups
 source_security_group_ids = [var.security_group_id] 
 ec2_ssh_key = "kube-demo" #create this ssh key in the account

}

  # Ensure that IAM Role permissions are created before and deleted after EKS Node Group handling.
  # Otherwise, EKS will not be able to properly delete EC2 Instances and Elastic Network Interfaces.
  depends_on = [
    aws_iam_role_policy_attachment.node-AmazonEKSWorkerNodePolicy,
    aws_iam_role_policy_attachment.node-AmazonEKS_CNI_Policy,
    aws_iam_role_policy_attachment.node-AmazonEC2ContainerRegistryReadOnly,
  ]
}
</code></pre>

</div>



<p>Now we have our <code>main.tf</code> file is complete, ensure the variables are created in the <code>variable.tf</code> file and then proceed to run your <code>terraform apply</code> command. This resource creation usually takes time, so get a cup of coffee and give it some minutes.</p>

<p>Connect To Control Node</p>

<p>Once your command is successful, it is now time to connect to your control plane node, using this command.<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight plaintext"><code>aws eks update-kubeconfig - name &lt;cluster name&gt; - region us-east-1

#get the node info
kubectl get nodes
</code></pre>

</div>



<p>You are good to go!</p>

