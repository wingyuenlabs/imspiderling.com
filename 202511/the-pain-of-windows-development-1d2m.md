---
Title: The pain of Windows development
Description: 
Author: Simeon Norris
Date: 2025-11-19T22:02:41.000Z
Robots: noindex,nofollow
Template: index
---
<p>When joining a mixed reality (XR) company I wanted to help with the scripting and we were all on Windows at the time. So, I was introduced to PowerShell and system management. I quickly found out that Windows makes everything difficult. Networked system communication, software versioning and an overall bloated OS to name a few pain points. </p>

<p>One of the core functions of our dashboard was to run PowerShell scripts from Unreal. I discovered when running 20 scripts at once would just max out the CPU. Each instance of PowerShell launches a .NET instance in the background, couple that with anti malware software scanning it or a machine left on all the time and you've got a bad time. Even basic ping scripts would be unreliable, Windows limits network resources so they cannot be abused (ICMP throttling). </p>

<p>We found ways to mitigate the cost of running PowerShell scripts by essentially queuing the scripts and running them in batches. This was a real pain for us as a company, as when scaling the amount of devices introduced more performance problems. </p>

<p>Now the dashboard v2 is going to be on Linux and the performance difference is night and day. I heard the other day that we can run 100 scripts with the same scheduling system and only reach 20-30% on CPU, on a SteamDeck. This is my first experience with Linux and as a developer it truly is amazing to work with. It is a steep learning curve but everything I said was a pain in Windows is much simpler and everything is faster. One example is using SSH, yes you can use SSH on windows, but the setup and usage on Linux is much more pain free.</p>

<p>One major difference to Windows is on Linux everything is a file descriptor, a file, process or script. A file descriptor points to a data blob called a iNode, that is referenced in a lookup table. Learning Linux is beneficial because unlike Windows Linux wont change. Learning the cmd line tools makes using your machine everyday more headache free. Can't find a file? No worries just use the find command 'find . -name "file.txt"'. </p>

<p>I will be posting more about Linux on my profile, thanks for reading. </p>

