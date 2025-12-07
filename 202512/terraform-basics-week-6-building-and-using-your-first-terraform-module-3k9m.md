---
Title: Terraform Basics – Week 6: Building and Using Your First Terraform Module
Description: 
Author: Ozan Guner
Date: 2025-12-07T21:41:29.000Z
Robots: noindex,nofollow
Template: index
---
<p><strong>Table of Contents</strong></p>

<p><strong>1. Recap of Previous Weeks</strong></p>

<p><strong>2. What are Terraform Modules and Why Do We Use Modules in Terraform?</strong></p>

<p><strong>3. Understanding Module Inputs and Outputs</strong></p>

<p><strong>4. Creating Our First Terraform Module</strong></p>

<p><strong>5. Using the Module in Our Root Configuration</strong></p>

<p><strong>6. Deploying to Azure</strong></p>

<p><strong>7. Wrap-Up</strong></p>

<p><strong>1. Recap of Previous Weeks</strong></p>

<p>Over the past five weeks, we built a small but realistic Azure environment while learning the core Terraform concepts. We started by deploying our first VM, then introduced variables and tfvars files to make the configuration more flexible. We added security with NSGs and dynamic blocks, and finally exposed useful information through output values.</p>

<p>Here's the visual representation of the infrastructure we have so far:</p>

<p><a href="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fal2v4gta28ifhvs9oai0.jpg" class="article-body-image-wrapper"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fal2v4gta28ifhvs9oai0.jpg" alt="Infrastructure" width="800" height="593"></a></p>

<p>Here's what .tf files we have under our Project Folder in Visual Studio Code:</p>

<p><a href="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fw6qlvl9xkk9qj1r8gbem.png" class="article-body-image-wrapper"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fw6qlvl9xkk9qj1r8gbem.png" alt="Folder Structure" width="292" height="293"></a></p>

<p><strong>nsg.tf</strong> &gt; Contains the Network Security Group and the NSG association.<br>
<strong>outputs.tf</strong> &gt; Contains the output values.<br>
<strong>providers.tf</strong> &gt; Contains the provider block (AzureRM)<br>
<strong>resource-group.tf</strong> &gt; Contains the resource group resource block.<br>
<strong>terraform.tfvars</strong> &gt; Contains the values for our variables for the project.<br>
<strong>variables.tf</strong> &gt; Contains the definition of all variables we used for our project.<br>
<strong>virtual-machine.tf</strong> &gt; Contains the virtual machine and Network Interface Card (NIC) resources.<br>
<strong>virtual-network.tf</strong> &gt; Contains the Virtual Network and subnet resources.</p>

<p>At this point, we have a fully working VM deployment that is parameterized, secure and able to surface important data. As the configuration grows, we are beginning to see repeated patterns, and managing multiple similar resources would quickly become difficult.</p>

<p>This is the perfect time to introduce Terraform modules.</p>

<p>If you missed the previous week's posts and want to do a deeper dive, here are the links to them :</p>

<p><a href="https://dev.to/ozanguner/terraform-basics-week-1-deploying-your-first-azure-vm-1f86">Week 1</a><br>
 <a href="https://dev.to/ozanguner/terraform-basics-week-2-variables-and-reusability-2c15">Week 2 </a><br>
 <a href="https://dev.to/ozanguner/terraform-basics-week-3-managing-variables-with-tfvars-files-j8i">Week 3</a> <br>
 <a href="https://dev.to/ozanguner/terraform-basics-week-4-securing-your-azure-vms-with-nsgs-and-dynamic-blocks-3mh0">Week 4</a><br>
 <a href="https://dev.to/ozanguner/terraform-basics-week-5-exposing-infrastructure-data-with-outputs-5h6n">Week 5</a></p>

<p><a href="https://github.com/OzangunerGH/Terraform-Basics-Series" rel="noopener noreferrer">GitHub</a> repository link for this series, where you can find the full .tf files used for every week including this week.</p>

<p>Now let's get started with Terraform Modules!</p>

<p><strong>2. What are Terraform Modules and Why Do We Use Modules in Terraform?</strong></p>

<p>A Terraform module is simply a collection of resources grouped together so they can be reused as a single unit. Instead of rewriting the same VM, NIC or NSG blocks every time we deploy a new virtual machine, we define them once inside a module and call that module whenever we need another instance.</p>

<p>Before we continue, it helps to understand how Terraform organizes configuration. The folder where your main Terraform files live is called the root module. This is where Terraform starts. Any module you place in a subfolder becomes a child module, and it cannot run on its own. The root module provides the inputs, and the child module returns outputs. This relationship is what allows us to reuse infrastructure patterns cleanly without duplicating code.</p>

<p>Modules become especially important in real-world environments. Imagine a team managing ten VMs across development, staging and production. Without modules, each environment ends up with its own copy of the same resource blocks. A simple change—like updating tags or adjusting an NSG rule—must now be repeated in multiple places. Over time, small differences creep in, files become harder to manage and consistency across environments suffers.</p>

<p>Modules solve this problem by centralizing the logic. We define the VM pattern once and deploy it anywhere by simply passing different input values. This keeps the configuration consistent, reduces duplication and makes it much easier to maintain and scale the infrastructure as it grows.</p>

<p><strong>3. Understanding Module Inputs and Outputs</strong></p>

<p>Before we begin creating our first module, it helps to understand how information moves between the root module and a child module. A child module cannot directly read variables or resources from the root module. Instead, Terraform requires us to explicitly pass values into the module and explicitly return values from it. This makes modules predictable, reusable and easier to maintain.</p>

<p><strong>Module Inputs</strong><br>
Inputs are the values a module needs in order to create its resources. These inputs are defined as variables inside the module, and the root module supplies the actual values when calling the module. In our case, the VM module will need things like the resource group name, resource group location, subnet ID, VM size, admin credentials, NIC settings and allowed ports.</p>

<p><strong>Module Outputs</strong><br>
Outputs are values that the module exposes back to the root module. This allows the root to access information created inside the module without reaching into its internal resources. For example, the module can return the VM’s ID, NIC name or private IP address so they can be used elsewhere in the configuration or displayed after deployment.</p>

<p>With this idea of inputs and outputs in mind, we can now begin refactoring our VM, NIC and NSG resources into a dedicated Terraform module.</p>

<p><strong>4. Creating Our First Terraform Module</strong></p>

<p>Disclaimer: This is more of "refactoring" our previous Terraform code into a module rather than creating a module from scratch. This is very useful in real-world scenarios where someone created a set of resources that will be re-used repeatedly and you want to turn that Terraform code into a reusable module.</p>

<p>In this step, we are going to move the VM, NIC and NSG resources into a modules/vm folder and wire them up so they still use the same variables as before.</p>

<p>The resource group, virtual network and subnet will stay in the root configuration. Our existing variables.tf, outputs.tf and terraform.tfvars will also remain in the root. We will keep using those files to provide values and simply pass them into the new module.</p>

<p>First, create the folder for the VM module, and the new .tf files like so :</p>

<p><a href="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Ftuzjqi7op5grukjjxjxk.png" class="article-body-image-wrapper"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Ftuzjqi7op5grukjjxjxk.png" alt="File Structure" width="294" height="166"></a></p>

<p>The main.tf file we create will hold the resource blocks for the module, which will be the Virtual Machine, Virtual Machine NIC, NSG, and the NSG association resources. Copy and paste those resources from virtual-machine.tf and nsg.tf files to the main.tf, like so :</p>

<p><a href="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fk326tpb44ocm3e0whglq.png" class="article-body-image-wrapper"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fk326tpb44ocm3e0whglq.png" alt="Main.tf" width="589" height="462"></a></p>

<p>After copying the code, you can remove the virtual-machine.tf and nsg.tf files.</p>

<p>There is one important change we need to make at this point. In the previous weeks, the VM, NIC and NSG resources in virtual-machine.tf and nsg.tf were reading location and resource_group_name directly from the resource group in the root, because all resources were at the root module level. An example of this is referring to resource group name and location attributes like this:</p>

<p><a href="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2F0rv3ehwb36t6yqy0q31w.png" class="article-body-image-wrapper"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2F0rv3ehwb36t6yqy0q31w.png" alt="reference" width="518" height="110"></a></p>

<p>Now that we are inside a child module, we can no longer access root resources this way. Instead, the module should receive the resource group name and location as input variables.</p>

<p>So, while you are in modules/vm/main.tf, replace any usage of azurerm_resource_group.prod.name with var.rg_name, and any usage of azurerm_resource_group.prod.location with var.rg_location, and usage of azurerm_subnet_prod.id with var.subnet_id, like so:</p>

<p><a href="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Ftguc3v2arcwvq7ddtv0z.png" class="article-body-image-wrapper"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Ftguc3v2arcwvq7ddtv0z.png" alt=" " width="800" height="198"></a></p>

<p>Next, create modules/vm/variables.tf and define the variables that the module expects. These should match the names you are already using in main.tf for the VM and NSG, like so :</p>

<p><a href="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2F0rouoxvhxnj0wme8wjpe.png" class="article-body-image-wrapper"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2F0rouoxvhxnj0wme8wjpe.png" alt=" " width="800" height="473"></a></p>

<p>This should address the errors the Visual Studio Code were showing in this manner:</p>

<p><a href="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fatagr2lxddfp79kpzqhl.png" class="article-body-image-wrapper"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fatagr2lxddfp79kpzqhl.png" alt=" " width="589" height="462"></a></p>

<p>Finally, create modules/vm/outputs.tf. Here we define the values that we want to expose back to the root configuration. These are the same kinds of values you exposed in Week 5, but now they come from the module instead of directly from the VM resource.</p>

<p><a href="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2F8w99bacwt1qw8zcrjig7.png" class="article-body-image-wrapper"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2F8w99bacwt1qw8zcrjig7.png" alt="modules/vm/Outputs.tf" width="800" height="267"></a></p>

<p>As you can see, we only took the outputs that are specific to resources under our main.tf file, and left out the resource group output that still is in the root and not in the module.</p>

<p>In order to refer to the outputs in our VM module, we must now use module.module_name.output_name syntax at the beginning of our outputs we have defined in our root outputs.tf file, like so:</p>

<p><a href="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fef3nq04kesnpztg3b4l2.png" class="article-body-image-wrapper"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fef3nq04kesnpztg3b4l2.png" alt=" " width="717" height="329"></a></p>

<p>Notice how the resource group output is still the same, as that is not a resource we included in the VM module.</p>

<p><strong>5. Using the Module in Our Root Configuration</strong></p>

<p>Now that we have created our VM child module and moved all VM-related resources into it, the next step is to connect this module to our root configuration. This is where we pass in the variables from our existing variables.tf and continue using terraform.tfvars just like in previous weeks.</p>

<p>Create a new main.tf under the root directory (Azure), like so:</p>

<p><a href="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2F6ctm0jifay4j1ogdm83g.png" class="article-body-image-wrapper"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2F6ctm0jifay4j1ogdm83g.png" alt=" " width="356" height="209"></a></p>

<p>Here, we will add a module block to actually call the child module we created under modules/vm and create the VM resource. The syntax to create a module looks like this:</p>

<p><a href="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fha89u4z6hnkaapf140ry.png" class="article-body-image-wrapper"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fha89u4z6hnkaapf140ry.png" alt=" " width="460" height="323"></a></p>

<p><strong>Source</strong> is where the module is relative to the root directory.<br>
The rest of the variables are the <strong>Inputs</strong> that the Module expects to be able to create the module. So what happens when we are calling the VM module is we are passing the variable values in terraform.tfvars file as inputs when the VM is being created using the child module.</p>

<p>Notice how azurerm_subnet.prod.id is not a variable. It’s an attribute that exists only after the virtual network and subnet are created. Because the VM module depends on the subnet being created first, referencing the subnet ID here creates an implicit dependency. Terraform automatically ensures that the subnet is created before the VM module runs, simply because the module cannot receive this value until the subnet exists.</p>

<p>With this module block in place, the root configuration is now responsible for providing inputs (via variables and resource outputs), while the child module handles the actual creation of the VM, NIC and NSG. This separation keeps the root clean and makes the VM definition fully reusable for future deployments.</p>

<p><strong>6. Deploying to Azure</strong><br>
Okay, that was a lot of editing and moving things around. This week, we'll add some additional commands to make sure everything is in tact.</p>

<p>Let's start with initializing Terraform by issuing terraform init command:</p>

<p><a href="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fe718r9makwzjsg0d4dqz.png" class="article-body-image-wrapper"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fe718r9makwzjsg0d4dqz.png" alt=" " width="800" height="415"></a></p>

<p>Since we moved a lot of code around, the formatting in our .tf files may no longer be consistent. Terraform provides a built-in tool for this. Running the terraform fmt command will automatically clean up and standardize the formatting across all your Terraform files so everything is properly aligned and easier to read. Since we have a hierarchical folder structure, in order to apply terraform fmt command to all files in all directories, we will append -recursive flag at the end of terraform fmt command.</p>

<p><a href="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Ftzvgbq9j9xk43e3hlpmy.png" class="article-body-image-wrapper"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Ftzvgbq9j9xk43e3hlpmy.png" alt=" " width="800" height="413"></a></p>

<p>Great, looks like i had multiple files that required formatting which are listed above.</p>

<p>Before applying any changes, it is a good idea to run the terraform validate command. This command checks whether your Terraform files are written correctly and whether the configuration is structurally sound. It does not contact Azure or create any resources. It simply reads your files and confirms that the syntax, references and module structure are valid. Running this helps catch issues early before running a plan or apply:</p>

<p><a href="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fojn3lp3d02m9hnwoki77.png" class="article-body-image-wrapper"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fojn3lp3d02m9hnwoki77.png" alt=" " width="800" height="228"></a></p>

<p>Now we are going to run terraform plan as usual:</p>

<p><a href="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2F3npkq2aze3qc2oqkx61y.png" class="article-body-image-wrapper"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2F3npkq2aze3qc2oqkx61y.png" alt=" " width="534" height="205"></a></p>

<p>Great! Looks like it's still creating 7 resources (VM, VM NIC, NSG, NSG Association, Virtual Network, Subnet , Resource Group), and outputs are still the same as well. Looks like our module configuration is working.</p>

<p>Next, we run terraform apply -auto-approve command:</p>

<p><a href="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fs5ygl9z80cqymnzim8cj.png" class="article-body-image-wrapper"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fs5ygl9z80cqymnzim8cj.png" alt=" " width="800" height="440"></a></p>

<p>Perfect, all resources are created successfully, and at the end of our apply, we got our outputs printed out.</p>

<p>Checking the Azure Portal, I can also see the VM created successfully with the values expected:</p>

<p><a href="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fxdu09th4ish48znwsuk6.png" class="article-body-image-wrapper"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fxdu09th4ish48znwsuk6.png" alt=" " width="800" height="208"></a></p>

<p>Finally, don't forget to issue terraform destroy -auto-approve command to prevent incurred costs:</p>

<p><a href="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fcxrtk5wyz25ve4ffc97s.png" class="article-body-image-wrapper"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fcxrtk5wyz25ve4ffc97s.png" alt=" " width="800" height="210"></a></p>

<p><strong>7. Wrap-Up</strong></p>

<p>By introducing our first Terraform module, we transformed our configuration from a simple, single-VM setup into a more modular and reusable design. Modules are how Terraform scales—whether you're deploying five VMs or fifty, the pattern stays the same. This week we learned how to move existing resources into a module, pass data into it from the root and expose outputs back out so other parts of the configuration can use them.</p>

<p>From here, you’ll start to see why teams rely on modules in every real Terraform project. In the upcoming posts, we’ll expand this module, deploy multiple instances and continue turning our growing configuration into something that mirrors real infrastructure practices.</p>

<p>Thanks for reading and stay tuned for more!</p>

