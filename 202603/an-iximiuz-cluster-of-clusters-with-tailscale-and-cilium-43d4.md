---
Title: An Iximiuz Cluster of Clusters with Tailscale and Cilium
Description: 
Author: Adam Leskis
Date: 2026-03-19T21:57:18.000Z
Robots: noindex,nofollow
Template: index
---
<blockquote>
<p>Editors note: This was originally much more ambitious, but for various reasons detailed below it had to be scaled back a bit. It's still pretty cool.</p>
</blockquote>

<p>The final k8s cluster project is <a href="https://github.com/lpmi-13/kubernetes-the-hard-way-iximiuz-plus-tailscale-cilium" rel="noopener noreferrer">here</a> and the thing I build to visualize it is <a href="https://github.com/lpmi-13/hubble-gazer" rel="noopener noreferrer">here</a>, but read on for the story of how I got there!</p>




<p>So I got hooked on the kubernetes-the-hard-way walkthroughs a couple years back and ran through them on various different public cloud providers. Since <a href="https://labs.iximiuz.com" rel="noopener noreferrer">iximiuz Labs</a> is my current favorite place to play, I thought I would give it a go there as well.</p>

<p>It turns out that somebody else <a href="https://labs.iximiuz.com/courses/kubernetes-the-very-hard-way-0cbfd997" rel="noopener noreferrer">already did this</a>, and it's way better than I would have done anyway, so I guess I might as well just forget about it.</p>

<p>...or maybe not!!!!</p>

<p><a href="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Faw9m53k65o7oidyou8fl.gif" class="article-body-image-wrapper"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Faw9m53k65o7oidyou8fl.gif" alt=" " width="480" height="480"></a></p>

<p> </p>

<p>What if, I thought to myself...just what IF instead of the traditional 1-3 controller nodes and 2-3 worker nodes (these numbers, I assume, are used in the typical k8s the hard way due to cost), I instead networked together not only more worker nodes, but different networks altogether?!?!</p>

<p>The <a href="https://github.com/kelseyhightower/kubernetes-the-hard-way" rel="noopener noreferrer">canonical walkthrough</a> uses only 4 machines (1 controller and 3 workers) all in the same subnet (this, as we shall see, MASSIVELY simplifies the networking). I decided that since I had so much fun playing with Tailscale in a <a href="https://dev.to/lpmi13/remote-homelab-admin-with-tailscale-1jp2">previous post</a>, I would use it a bit more to connect the various subnets across multiple iximiuz Labs playgrounds.</p>

<p><a href="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fin2qyq1n7hpk1k2yy5o5.png" class="article-body-image-wrapper"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fin2qyq1n7hpk1k2yy5o5.png" alt="cluster of clusters original topology" width="800" height="566"></a></p>

<p> <br>
 </p>

<p>So the original idea was to do all the inter-node routing via Tailscale, so that each controller and worker node would be able to address each other by hostname, which would map to the Tailscale overlay IP address (eg, <code>100.1.20.144</code>). So even though we were wiring together a bunch of different playgrounds/subnets, all of the VMs (and hence also the eventual pods) would think they were on a flat network.</p>

<p> <br>
 </p>

<h2>
  
  
  My Kingdom for Some Cool Viz
</h2>

<p>Then I decided that just a big ole kubernetes cluster is pretty cool all by itself, but it's a bit difficult to get a good picture of exactly <em>HOW</em> that differs from just a regular one (ie, all the other k8s the hard way walkthroughs).</p>

<p>So I thought, maybe we could have some sort of visual, and even better to have a live traffic graph UI that would make it very obvious when we scaled a deployment up/down, or even just had a traffic generator hit one of the services with a ton of traffic and actually <strong>see</strong> the difference between a 3 worker cluster and a 9 worker cluster.</p>

<p>And so after a bit of research, it seemed like the easiest way to do this would be with some sort of a service mesh (the pods don't care about it, and there's usually baked in telemetry, so a ready data source that I could consume).</p>

<p>The ones I was looking at were Istio/Envoy/Cilium, and since the iximiuz Labs VMs have a maximum of 4CPU and 8GiB RAM, I was trying to minimize the resource usage of the service mesh. Of those, not only is Cilium the clear winner, but it uses eBPF, which I though was really cool, and I wanted to play with it more.</p>

<p>Okay, so I decided on Cilium, now what?</p>

<p><a href="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2F0kb49yzd8szm3svf2qrc.jpg" class="article-body-image-wrapper"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2F0kb49yzd8szm3svf2qrc.jpg" alt=" " width="640" height="480"></a></p>

<p> </p>

<p>I started looking around for traffic visualization projects using Cilium (more specifically, hubble-observatory, which is Cilium's observability platform). The default view for hubble-ui, which is the included UI for hubble-observatory, just shows static traffic routes.</p>

<p><a href="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fi9wtel6ptdv1vs7j51hq.png" class="article-body-image-wrapper"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fi9wtel6ptdv1vs7j51hq.png" alt=" " width="800" height="603"></a></p>

<p> </p>

<p>Basically nothing I found was live traffic. There was a super old <a href="https://github.com/Netflix/vizceral" rel="noopener noreferrer">Netflix project</a>, but it wasn't maintained anymore.</p>

<p>Hmmmm...there's nothing that does what I need, and I've been spending the last year or so getting acquainted with agentic workflows...</p>

<p><a href="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fd14d0ryrw4u5qs744b1o.gif" class="article-body-image-wrapper"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fd14d0ryrw4u5qs744b1o.gif" alt=" " width="640" height="280"></a></p>

<p> <br>
 </p>

<h2>
  
  
  Let's Just Make The Thing We Need
</h2>

<p>Okay, so I need something running in the cluster that will grab data from hubble-observatory and I need a frontend that will open up and display something in a browser tab. Let's mix some Go and React and do the damned thing!</p>

<p><a href="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Ftayim89qv8efmjfvh4os.png" class="article-body-image-wrapper"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Ftayim89qv8efmjfvh4os.png" alt=" " width="800" height="396"></a></p>

<p> </p>

<p>I used codex for this, and I was actually surprised at how far it got on the first attempt. Most of the <del>work</del> prompting was getting a local dev example up and running with sample data so I could see what it looked like and iterate on stuff...</p>

<ul>
<li>no, those graph edges need to be thinner
 </li>
<li>...thinner!
 </li>
<li>alright, and stop resetting everything every 2 seconds, I said DECOUPLE the frontend and backend!
 </li>
<li>WHY DID YOU SUGGEST WEBSOCKETS AGAIN?!?!! THIS COULD EASILY BE SERVER SENT EVENTS!!!! (to be fair, it probably doesn't matter that much in this case, since it's just a throwaway demo, but the agent always starts out with websockets even when it's definitely a worse choice than SSE)
 </li>
<li>that's okay, but we need to auto-zoom when the topology changes
 </li>
<li>forget everything I just said, we need to also cater for a mobile UI (I'm obsessed with running iximiuz Labs stuff on my phone, so this is a must for any of my projects)</li>
</ul>

<p>...eventually, I had something that looked like I wanted, and I was ready to get it showing some actual live data.</p>

<p><a href="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Frzjan652bwdr9qkwgfju.png" class="article-body-image-wrapper"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Frzjan652bwdr9qkwgfju.png" alt=" " width="800" height="512"></a></p>

<p> <br>
 </p>

<h2>
  
  
  My MVP meets Live Cluster Reality
</h2>

<p>Okay, so this is where things went a bit pear-shaped, and I spent probably 75% of the total project time.</p>

<p>To start with, actually running a service inside kubernetes was going to require all kinds of extra stuff like RBAC, network policies, and pod specs for the deployment. No biggie, I've got an Intern to do that.</p>

<p>I'll also admit to heavy ulitization of the scripts in the project, since there's no way I was gonna "hard way" build it from scratch every day (the playgrounds have a max lifetime of 8 hours, and since it's a group of them, I <em>guess</em> we could have saved VM state and restarted them, but I also wanted to be sure the project would always come up from zero, so I went with the scripts), and I started to see some really weird failures and intermittent behavior:</p>

<ul>
<li><p>Everything would come up, but there would be no data coming through hubble-gazer</p></li>
<li><p>I could see initial data in hubble-gazer, but then every 2 seconds (the same interval as the SSE pushes to the frontend) it got less and less until it completely disappeared</p></li>
<li><p>sometimes the scripts would just choke and things wouldn't start up</p></li>
</ul>

<p>The short answer to "why?" is that not only was I using an overlay network (Cilium VXLANs) wrapped in another overlay network (Tailscale), but the topology was such that we had 5 different subnets and 12 kubernetes nodes all communicating with each other for reachability.</p>

<p>We ended up with a situation where some of the playgrounds would end up with the same public IP address but be on different private subnets, and so Tailscale would have to fall back to a relayed connection, which still works, but introduces latency.</p>

<p>So things I had initially wanted to try, like HPA scaling based on CPU from the metrics server to show a cool demo of dynamic pod growth, just wouldn't work. I couldn't even get just KEDA with queues in redis to work, since the jitter and general network conditions just wouldn't support it.</p>

<p>This was all very frustrating and a bit confusing to me, and I'm sure somebody with better networking knowledge could have sorted it out, but I had already learned a bunch and a deadline (I wanted this done by Kubecon EU 2026) was fast approaching.</p>

<p><a href="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fd6a1fs485k1hwmkao0ce.jpg" class="article-body-image-wrapper"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fd6a1fs485k1hwmkao0ce.jpg" alt=" " width="500" height="500"></a></p>

<p> <br>
 </p>

<p>Updating the topology so all of the worker nodes were in the same subnet basically solved everything, since now internode traffic didn't have to transit two layers of tunnelling, and we only used Tailscale for communication between the workers and the control plane.</p>

<p>So we ended up with a much simplified topology, but it also finally worked and I could finish aligning hubble-gazer to show the live traffic.</p>

<p>I had been hoping that the KEDA-based scaling I added would be more obvious, but oh well.</p>

<p>The final topology (3 playgrounds, 9 VMs) looks like this:</p>

<p><a href="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fyrjm6tyoxjhtnjiyidlr.png" class="article-body-image-wrapper"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fyrjm6tyoxjhtnjiyidlr.png" alt="cluster of clusters final topology" width="800" height="564"></a></p>

<p> <br>
 </p>

<p>And here are some tasty gifs of the live traffic being portforwarded from the jumpbox back to localhost to be accessible from the laptop I used to set up the cluster (that's why the browser URL bar says localhost:8888).</p>

<p><a href="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fqhk7q8s1e49ytbhj3pq9.gif" class="article-body-image-wrapper"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fqhk7q8s1e49ytbhj3pq9.gif" alt="All services and pods" width="480" height="253"></a><br>
All services and pods</p>

<p> <br>
 </p>

<p><a href="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2F2gb46kwoio59q37o6wtd.gif" class="article-body-image-wrapper"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2F2gb46kwoio59q37o6wtd.gif" alt="Application L4 traffic" width="600" height="316"></a><br>
Application L4 traffic, also grouped by worker node</p>

<p> <br>
 </p>

<p><a href="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2F3nxihzup92tu8h8vnqig.gif" class="article-body-image-wrapper"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2F3nxihzup92tu8h8vnqig.gif" alt="Application and System L7 traffic" width="720" height="380"></a></p>

<p>Application L7 traffic, as well as DNS queries</p>

