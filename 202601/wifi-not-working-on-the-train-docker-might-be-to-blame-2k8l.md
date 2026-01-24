---
Title: WiFi Not Working On The Train? Docker Might Be To Blame
Description: 
Author: Christian Schärf
Date: 2026-01-24T20:57:23.000Z
Robots: noindex,nofollow
Template: index
---
<p>A while ago I was travelling on the ICE high-speed train of Germany’s national railway operator Deutsche Bahn and tried to use the on-board WiFi, but, alas, to no avail.<br>
Now, it is not <em>exactly</em> unheard of that the Internet connection does not work, but in this case, I could not even reach the captive portal.<sup id="fnref1">1</sup></p>

<blockquote>
<p>[!note]- Docker network types<br>
When Docker networks are being referenced in this article, I am referring to networks of the <code>bridge</code> type.</p>

<p>[!TIP]- Resolution<br>
The onboard WiFi on the ICE has the IP address range <code>172.18.0.0/16</code> which was already occupied by a previously created Docker network.<br>
Thus, all packages meant for the onboard WiFi were routed into the Docker network.</p>
</blockquote>
<h2>
  
  
  Analysis
</h2>

<p>Previously, I was working with Docker and had created a network of type <code>bridge</code>.<br>
With this type, all containers connected to the network as well as the host get IP addresses assigned with which they can be reached.<br>
Per default, Docker uses a <code>/16</code> block of the IP address range <code>172.16.0.0/12</code><sup id="fnref2">2</sup> for each network.<br>
This range is reserved for private networking and thus not part of the global Internet.<sup id="fnref3">3</sup></p>

<p>The first address block <code>172.16.0.0/16</code> is skipped used since it is commonly used for local networks.<sup id="fnref4">4</sup><br>
The next block <code>172.17.0.0/16</code> is the default address range of the default <code>bridge</code> network:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight plaintext"><code>❯ docker network inspect bridge 
[
    {
        …
        "IPAM": {
            …
            "Config": [
                {
                    "Subnet": "172.17.0.0/16",
                    "IPRange": "",
                    "Gateway": "172.17.0.1"
                }
            ]
        },
        …
    }
]
</code></pre>

</div>



<p>All further blocks are used for networks created with <code>docker network create</code>, in ascending order. The first one (here named <code>demo</code>) is assigned:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight plaintext"><code>❯ docker network inspect demo 
[
    {
        …
        "IPAM": {
            …
            "Config": [
                {
                    "Subnet": "172.18.0.0/16",
                    "IPRange": "",
                    "Gateway": "172.18.0.1"
                }
            ]
        },
        …
    }
]
</code></pre>

</div>



<p>As previously mentioned, the block <code>172.16.0.0/16</code> is skipped because it is commonly used for local networks.<br>
All other ranges are usually unused. One consequential exception the the ICE’s onboard WiFi: It also uses the <code>172.18.0.0/16</code> range,<br>
the same as the first user-created Docker network:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight plaintext"><code>❯ ip a
…
2: wlp2s0: &lt;BROADCAST,MULTICAST,UP,LOWER_UP&gt; mtu 1440 qdisc noqueue state UP group default qlen 1000
    link/ether 01:23:45:67:89:ab brd ff:ff:ff:ff:ff:ff
    altname wlx74d83e3e31a4
    inet 172.18.0.1/16 brd 172.18.255.255 scope global dynamic noprefixroute wlp2s0
       valid_lft 598sec preferred_lft 598sec
    inet6 fe80::cdd8:684:4c94:dc3d/64 scope link noprefixroute 
       valid_lft forever preferred_lft forever
…
12: br-d9ec15037321: &lt;NO-CARRIER,BROADCAST,MULTICAST,UP&gt; mtu 1500 qdisc noqueue state DOWN group default 
    link/ether 76:8c:41:da:45:99 brd ff:ff:ff:ff:ff:ff
    inet 172.18.0.1/16 brd 172.18.255.255 scope global br-d9ec15037321
       valid_lft forever preferred_lft forever
</code></pre>

</div>



<p>Consequently, all packages that are meant for the onboard WiFi are routed to the Docker network and get lost.</p>

<h2>
  
  
  Workaround
</h2>

<p>For a quick workaround it is sufficient to delete the offending Docker network with <code>docker network rm</code> and to recreate it with <code>docker network create</code>.<br>
Afterwards, the containers have to be reconnected if required.</p>

<blockquote>
<p>[!WARNING]<br>
Before, all (including non-running) containers connected to the network should be disconnected manually.</p>
</blockquote>

<p>This works because the Docker skips already used IP address ranges when (re)creating networks.<br>
Understandably, Docker cannot know if a specific IP address ranges will maybe be needed at some point in the future.</p>
<h2>
  
  
  Solution
</h2>

<p>The IP address ranges used by the Docker engine can be configured in <code>/etc/docker/daemon.json</code>.<sup id="fnref5">5</sup><br>
Here, usable address ranges are defined and the network sizes (number of prefix bits) are assigned.</p>
<h3>
  
  
  IPv4
</h3>

<p>For IPv4 it is reasonable to use a part of the private networking IP address range <code>10.0.0.0/8</code><sup id="fnref3">3</sup>, for example:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight json"><code><span class="p">{</span><span class="w">
    </span><span class="nl">"default-address-pools"</span><span class="p">:</span><span class="w"> </span><span class="p">[</span><span class="w">
        </span><span class="p">{</span><span class="w">
            </span><span class="nl">"base"</span><span class="p">:</span><span class="w"> </span><span class="s2">"10.128.0.0/9"</span><span class="p">,</span><span class="w">
            </span><span class="nl">"size"</span><span class="p">:</span><span class="w"> </span><span class="mi">16</span><span class="w">
        </span><span class="p">}</span><span class="w">
    </span><span class="p">]</span><span class="w">
</span><span class="p">}</span><span class="w">
</span></code></pre>

</div>



<p>Here the second half of the previously mentioned range stating at <code>10.128.0.0</code> is used. Each Docker network will have a <code>/16</code> bit prefix.<br>
Thus, 128 networks with 2¹⁶ = 65536 addresses each are available.</p>

<p>Keep in mind, however, that this IP address range can also collide with other local networks, especially public WiFis and corporate networks.</p>
<h3>
  
  
  IPv6
</h3>

<p>Since version 27 of the Docker engine, IPv6 support is no longer experimental.<sup id="fnref6">6</sup> If no IPv6 address ranges are specified<br>
(in <code>/etc/docker/daemon.json</code> or via the <code>--subnet</code> command line option) Docker assigns a subrange of the <em>unique local addresses</em>.<sup id="fnref7">7</sup></p>

<p>When creating a network, you can specify to use IPv6 only:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight shell"><code>docker network create <span class="nt">--ipv6</span> <span class="nt">--ipv4</span><span class="o">=</span><span class="nb">false </span>ipv6only
</code></pre>

</div>



<p>For Compose, you can also set these settings:<sup id="fnref8">8</sup><br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight yaml"><code><span class="na">networks</span><span class="pi">:</span>
  <span class="na">ipv6only</span><span class="pi">:</span>
    <span class="na">enable_ipv4</span><span class="pi">:</span> <span class="kc">false</span>
    <span class="na">enable_ipv6</span><span class="pi">:</span> <span class="kc">true</span>
</code></pre>

</div>



<p>Unfortunately, there are still some limitations at the time of writing:</p>

<ul>
<li>Forwarding an IPv4 host port to the container is apparently not possible.<sup id="fnref9">9</sup>. This means that the container can be reached by IPv6 only from the outside.
For operating a server, this is probably not acceptable in most cases, but it should be fine for local development.
&gt; [!note]
&gt; This limitation is not present in the reverse case: A host IPv6 port can be forwarded to a container having only an IPv4 address.</li>
<li>It is apparently not possible to set IPv6-only as default for new networks.</li>
<li>Also, it is apparently impossible to configure the predefined <code>bridge</code> network as IPv6-only.</li>
</ul>

<h2>
  
  
  Closing Remarks
</h2>

<p>None of the involved parties made a mistake. Chosing the <code>172.18.0.0/16</code> address range for the onboard WiFi cannot be criticised as this is the inteded purpose.<br>
Docker has to take its IP addresses from somewhere as well, using ranges designated for private networking is a sensible choice.<br>
Even in the larger address range <code>10.0.0.0/8</code> (which Podman uses per default for this purpose) collisions are, in principle, possible.<br>
There are no IPv4 address ranges reserved for ‘host-local’ networks, after all.</p>

<p>Only the much larger address range of the <em>unique local addresses</em> of IPv6 presents a conclusive solution.<br>
However, IPv6 is not used by Docker per default and is still subject to some limitations.<br>
Apparantly, IPv6 is still <em>Neuland</em> (uncharted territory) for Docker.</p>

<p>Here, the number after the slash specifies, how many bits of the IP address are part of the network prefix.<br>
Thus, the network <code>172.16.0.0/12</code> contains the IP addresses from <code>172.16.0.0</code> to <code>172.31.255.255</code> because they all share the same leading 12 bits.</p>




<ol>

<li id="fn1">
<p>The captive portal is the page that public networks redirect you to for you to accept the terms of use. ↩</p>
</li>

<li id="fn2">
<p>We use the <a href="https://de.wikipedia.org/wiki/Classless_Inter-Domain_Routing" rel="noopener noreferrer">CIDR notation</a>. ↩</p>
</li>

<li id="fn3">
<p><a href="https://en.wikipedia.org/wiki/List_of_reserved_IP_addresses#IPv4" rel="noopener noreferrer">https://en.wikipedia.org/wiki/List_of_reserved_IP_addresses#IPv4</a> ↩</p>
</li>

<li id="fn4">
<p><a href="https://docs.docker.com/engine/network/#automatic-subnet-allocation" rel="noopener noreferrer">https://docs.docker.com/engine/network/#automatic-subnet-allocation</a> ↩</p>
</li>

<li id="fn5">
<p>Assuming you use a reasonable operating system ↩</p>
</li>

<li id="fn6">
<p><a href="https://docs.docker.com/engine/release-notes/27/#ipv6" rel="noopener noreferrer">https://docs.docker.com/engine/release-notes/27/#ipv6</a> ↩</p>
</li>

<li id="fn7">
<p><a href="https://docs.docker.com/engine/daemon/ipv6/#dynamic-ipv6-subnet-allocation" rel="noopener noreferrer">https://docs.docker.com/engine/daemon/ipv6/#dynamic-ipv6-subnet-allocation</a> ↩</p>
</li>

<li id="fn8">
<p><a href="https://docs.docker.com/reference/compose-file/networks/#enable_ipv4" rel="noopener noreferrer">https://docs.docker.com/reference/compose-file/networks/#enable_ipv4</a> ↩</p>
</li>

<li id="fn9">
<p><a href="https://github.com/moby/moby/issues/49617" rel="noopener noreferrer">https://github.com/moby/moby/issues/49617</a> ↩</p>
</li>

</ol>

