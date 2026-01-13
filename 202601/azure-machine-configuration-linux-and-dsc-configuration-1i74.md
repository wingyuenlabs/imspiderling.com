---
Title: Azure Machine Configuration, Linux, and DSC Configuration
Description: 
Author: Olivier Miossec
Date: 2026-01-13T22:03:23.000Z
Robots: noindex,nofollow
Template: index
---
<p>I recently made a post about Azure Machine Configuration and PowerShell DSC, and how to deploy VM configurations as infrastructure as Code, just like the rest of your infrastructure.</p>

<p>But in this post, I focused mainly on Windows configuration and wrote almost nothing about Linux.</p>

<p>However, Azure Machine Configuration is also available for Linux VMs, where you can use DSC or Chef InSpec.</p>

<p>To run DSC configurations, VMs need to have the AzurePolicyForLinux extension and a managed identity.</p>

<p>But unlike Windows, most Linux VMs don’t come with PowerShell installed. To solve this problem, the AzurePolicyForLinux extension installs PowerShell in a Folder but doesn’t modify the Path of the VM. PowerShell can only be used by the policy. </p>

<p>The next step is to write a DSC configuration. There are many DSC resources for managing Windows, but to manage a Linux server, you will need a compatible DSC resource. The most advanced is the <a href="https://github.com/azure/nxtools" rel="noopener noreferrer">NxTools module</a>. It is the recommended version to use with the Azure Machine configuration. </p>

<p>The NxTools module is a POSIX wrapper for several Linux commands and includes several DSC resources. </p>

<ul>
<li>nxFiles, to manage files and folders</li>
<li>nxGroup, to manage groups and group membership</li>
<li>nxUser, to manage users</li>
<li>nxPackages, to manage packages (support only apt)</li>
<li>nxService, to manage service (only system)</li>
<li>nxScript, to execute scripts in DSC configurations</li>
</ul>

<p>To demonstrate DSC on Linux with Azure Machine Configuration, let's ensure the user devTo is created and added to the group publisher, and a file with specified content is created using a script resource.<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight powershell"><code><span class="kr">configuration</span><span class="w"> </span><span class="n">demoDSCLinux</span><span class="w">
</span><span class="p">{</span><span class="w">

    </span><span class="n">Import-DscResource</span><span class="w"> </span><span class="nt">-ModuleName</span><span class="w"> </span><span class="s1">'nxtools'</span><span class="w">


        </span><span class="n">nxUser</span><span class="w"> </span><span class="nx">ensureDevToUser</span><span class="w"> </span><span class="p">{</span><span class="w">

            </span><span class="n">Ensure</span><span class="w"> </span><span class="o">=</span><span class="w">  </span><span class="s1">'Present'</span><span class="w">
            </span><span class="n">UserName</span><span class="w"> </span><span class="o">=</span><span class="w">  </span><span class="s1">'DevTo'</span><span class="w">
            </span><span class="n">FullName</span><span class="w"> </span><span class="o">=</span><span class="w"> </span><span class="s1">'Dev To Demo user'</span><span class="w">
            </span><span class="n">HomeDirectory</span><span class="w"> </span><span class="o">=</span><span class="w"> </span><span class="s1">'/home/DevTo'</span><span class="w">
            </span><span class="n">Description</span><span class="w"> </span><span class="o">=</span><span class="w"> </span><span class="s1">'Ensure that DevTo user is present on the system'</span><span class="w">
        </span><span class="p">}</span><span class="w">

        </span><span class="n">nxGroup</span><span class="w"> </span><span class="nx">ensurePublishersGroup</span><span class="w"> </span><span class="p">{</span><span class="w">
            </span><span class="c"># the group must be present and have root as only member</span><span class="w">
            </span><span class="n">Ensure</span><span class="w"> </span><span class="o">=</span><span class="w">  </span><span class="s1">'Present'</span><span class="w">
            </span><span class="n">GroupName</span><span class="w"> </span><span class="o">=</span><span class="w">  </span><span class="s1">'publishers'</span><span class="w">
            </span><span class="n">Members</span><span class="w"> </span><span class="o">=</span><span class="w"> </span><span class="p">@(</span><span class="s1">'DevTo'</span><span class="p">)</span><span class="w">
        </span><span class="p">}</span><span class="w">

        </span><span class="n">nxScript</span><span class="w"> </span><span class="nx">ensurePublisherfilePresent</span><span class="w"> </span><span class="p">{</span><span class="w">
        </span><span class="n">GetScript</span><span class="w"> </span><span class="o">=</span><span class="w"> </span><span class="p">{</span><span class="w">
            </span><span class="nv">$Reason</span><span class="w"> </span><span class="o">=</span><span class="w"> </span><span class="p">[</span><span class="n">Reason</span><span class="p">]::</span><span class="n">new</span><span class="p">()</span><span class="w">
            </span><span class="nv">$Reason</span><span class="o">.</span><span class="nf">Code</span><span class="w"> </span><span class="o">=</span><span class="w"> </span><span class="s2">"Script:Script:FileMissing"</span><span class="w">
            </span><span class="nv">$Reason</span><span class="o">.</span><span class="nf">Phrase</span><span class="w"> </span><span class="o">=</span><span class="w"> </span><span class="s2">"File does not exist"</span><span class="w">

            </span><span class="kr">if</span><span class="w"> </span><span class="p">(</span><span class="n">Test-Path</span><span class="w"> </span><span class="nt">-Path</span><span class="w"> </span><span class="s2">"/home/DevTo/publisher.txt"</span><span class="p">)</span><span class="w">
            </span><span class="p">{</span><span class="w">
                    </span><span class="nv">$Reason</span><span class="o">.</span><span class="nf">Code</span><span class="w"> </span><span class="o">=</span><span class="w"> </span><span class="s2">"Script:Script:Success"</span><span class="w">
                    </span><span class="nv">$Reason</span><span class="o">.</span><span class="nf">Phrase</span><span class="w"> </span><span class="o">=</span><span class="w"> </span><span class="s2">"File exists"</span><span class="w">
            </span><span class="p">}</span><span class="w">

            </span><span class="kr">return</span><span class="w"> </span><span class="p">@{</span><span class="w">
                </span><span class="nx">Reasons</span><span class="w"> </span><span class="o">=</span><span class="w"> </span><span class="p">@(</span><span class="nv">$Reason</span><span class="p">)</span><span class="w">
            </span><span class="p">}</span><span class="w">
        </span><span class="p">}</span><span class="w">
        </span><span class="n">TestScript</span><span class="w"> </span><span class="o">=</span><span class="w"> </span><span class="p">{</span><span class="w">
            </span><span class="kr">if</span><span class="w"> </span><span class="p">(</span><span class="n">Test-Path</span><span class="w"> </span><span class="nt">-Path</span><span class="w"> </span><span class="s2">"/home/DevTo/publisher.txt"</span><span class="p">)</span><span class="w">
            </span><span class="p">{</span><span class="w">
                </span><span class="kr">return</span><span class="w"> </span><span class="bp">$true</span><span class="w">
            </span><span class="p">}</span><span class="w">
            </span><span class="kr">else</span><span class="w">
            </span><span class="p">{</span><span class="w">
                </span><span class="kr">return</span><span class="w"> </span><span class="bp">$false</span><span class="w">
            </span><span class="p">}</span><span class="w">
        </span><span class="p">}</span><span class="w">
        </span><span class="n">SetScript</span><span class="w"> </span><span class="o">=</span><span class="w"> </span><span class="p">{</span><span class="w">
            </span><span class="bp">$null</span><span class="w"> </span><span class="o">=</span><span class="w"> </span><span class="n">New-Item</span><span class="w"> </span><span class="nt">-Path</span><span class="w"> </span><span class="s2">"/home/DevTo/publisher.txt"</span><span class="w"> </span><span class="nt">-ItemType</span><span class="w"> </span><span class="s2">"File"</span><span class="w"> </span><span class="nt">-Force</span><span class="w">
        </span><span class="p">}</span><span class="w">
        </span><span class="p">}</span><span class="w">

</span><span class="p">}</span><span class="w">
</span></code></pre>

</div>



<p>The first two resources create the DevTo user and create the Publishers group, and add the DevTo user. </p>

<p>The last one ensures that the file "/home/DevTo/publisher.txt" is present on the system using a nxScript resource. The GetScript block of the resource needs to return a Hashtable with a Reason object to work. The reason.phrase will be shown in the Azure Portal in case of noncompliance.</p>

<p>If you try to compile this configuration on your Windows or macOS machine, you will get an error; you need a Linux VM with PowerShell. </p>

<p>You can set up a dev VM for that, but you can also use a CI/CD pipeline to compile and package the configuration for Linux. You can also compile and package the DSC configuration in GitHub Actions. </p>

<p>First, you will need a script for compiling the configuration and creating the Azure Machine Configuration package.<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight powershell"><code><span class="c"># Check if the host is a Linux system</span><span class="w">

</span><span class="kr">if</span><span class="w"> </span><span class="p">(</span><span class="bp">$IsLinux</span><span class="w"> </span><span class="o">-eq</span><span class="w"> </span><span class="bp">$false</span><span class="p">)</span><span class="w"> </span><span class="p">{</span><span class="w">
    </span><span class="n">Write-Error</span><span class="w"> </span><span class="s2">"This script must be run on a Linux system."</span><span class="w">
    </span><span class="kr">exit</span><span class="w"> </span><span class="mi">1</span><span class="w">
</span><span class="p">}</span><span class="w">

</span><span class="c"># install modules </span><span class="w">
</span><span class="n">Install-Module</span><span class="w"> </span><span class="nt">-Name</span><span class="w"> </span><span class="nx">nxtools</span><span class="w"> </span><span class="nt">-Force</span><span class="w">
</span><span class="n">install-module</span><span class="w"> </span><span class="nt">-name</span><span class="w"> </span><span class="nx">PSDesiredStateConfiguration</span><span class="w"> </span><span class="nt">-RequiredVersion</span><span class="w"> </span><span class="nx">3.0.0-beta1</span><span class="w"> </span><span class="nt">-Force</span><span class="w"> </span><span class="nt">-AllowPrerelease</span><span class="w">
</span><span class="n">install-module</span><span class="w"> </span><span class="nt">-name</span><span class="w"> </span><span class="nx">GuestConfiguration</span><span class="w"> </span><span class="nt">-Force</span><span class="w"> </span><span class="nt">-RequiredVersion</span><span class="w"> </span><span class="nx">4.1.0</span><span class="w"> 

</span><span class="c"># compile the demo configuration</span><span class="w">

</span><span class="o">.</span><span class="w"> </span><span class="o">.</span><span class="n">/DSC-Linux/demolinux.dsc.ps1</span><span class="w">

</span><span class="n">demoDSCLinux</span><span class="w">

</span><span class="c"># rename the MOF file </span><span class="w">

</span><span class="n">Rename-Item</span><span class="w"> </span><span class="nt">-Path</span><span class="w"> </span><span class="o">.</span><span class="nx">\demoDSCLinux\localhost.mof</span><span class="w"> </span><span class="nt">-NewName</span><span class="w"> </span><span class="nx">demoDSCLinux.mof</span><span class="w"> </span><span class="nt">-Force</span><span class="w"> 

</span><span class="c"># create the package for Azure Machine Configuration</span><span class="w">

</span><span class="n">New-GuestConfigurationPackage</span><span class="w"> </span><span class="nt">-name</span><span class="w"> </span><span class="s2">"demoDSCLinux"</span><span class="w"> </span><span class="nt">-Type</span><span class="w"> </span><span class="nx">AuditAndSet</span><span class="w"> </span><span class="nt">-Configuration</span><span class="w"> </span><span class="o">.</span><span class="nx">\demoDSCLinux\demoDSCLinux.mof</span><span class="w"> </span><span class="nt">-Force</span><span class="w"> </span><span class="bp">$true</span><span class="w">
</span></code></pre>

</div>



<p>The first line of the script tests if the host is a Linux machine. The configuration can only be created on Linux. </p>

<p>Then the script imports all the necessary modules, nxtools for the DSC resources, PSDesiredStateConfiguration for DSC, and GuestConfiguration to package the configuration. </p>

<p>It then compiles the configuration and packages it. The package can then be deployed to a Linux VM with Azure Machine configuration.</p>

