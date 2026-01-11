---
Title: Linux File System
Description: 
Author: Shivakumar
Date: 2026-01-11T21:22:10.000Z
Robots: noindex,nofollow
Template: index
---
<h3>
  
  
  1. Linux File System Basics
</h3>

<p>In Linux, everything is a file. The file system controls how data is stored and retrieved. Unlike Windows (which uses drive letters like <code>C:</code>, <code>D:</code>), Linux uses a single hierarchical tree structure starting from the root directory (<code>/</code>).</p>

<ul>
<li>
<strong>Hierarchy:</strong> The structure looks like an inverted tree.</li>
<li>
<code>/</code> (Root): The starting point.</li>
<li>
<code>/home</code>: User personal data.</li>
<li>
<code>/etc</code>: Configuration files.</li>
<li>
<code>/var</code>: Variable data (logs, websites).</li>
<li><p><code>/bin</code> &amp; <code>/usr/bin</code>: Executable binaries/programs.</p></li>
<li><p><strong>Common File System Types:</strong></p></li>
<li><p><strong>ext4:</strong> The most common standard Linux file system. Reliable and stable.</p></li>
<li><p><strong>XFS:</strong> High-performance, great for handling very large files (often used in enterprise RHEL/CentOS systems).</p></li>
<li><p><strong>Btrfs:</strong> Advanced features like snapshots and self-healing.</p></li>
</ul>




<h3>
  
  
  2. Inodes (Index Nodes)
</h3>

<p>An Inode is a data structure on the file system that stores all information about a file <strong>except its name and actual data</strong>.</p>

<ul>
<li>
<strong>What it stores:</strong> File size, owner, permissions, timestamps, and pointers to the disk blocks where the actual data resides.</li>
<li><p><em>Analogy:</em> Think of an Inode as a library catalog card. It tells you where the book (data) is on the shelf, who wrote it, and when it was bought, but it isn't the book itself.</p></li>
<li><p><strong>Inode Number:</strong> Every file has a unique inode number.</p></li>
<li><p><strong>Checking Inodes:</strong> Use the command <code>ls -i</code> to see the inode number of a file.</p></li>
</ul>

<blockquote>
<p><strong>Note:</strong> If you run out of Inodes (by creating millions of tiny files), you cannot create new files, even if you have free disk space.</p>
</blockquote>

<p>Command,Description,Example<br>
df -h,"Shows disk space usage in ""human-readable"" format (GB, MB).",df -h<br>
df -T,"Shows the file system type (ext4, xfs, etc.).",df -Th<br>
du -sh [path],"""Disk Usage"". Shows the size of a specific directory.",du -sh /var/log<br>
lsblk,Lists all block devices (disks/partitions) in a tree view.,lsblk<br>
mount / umount,Attaches or detaches a file system.,mount /dev/sdb1 /mnt/data<br>
ls -i,Lists files with their Inode numbers.,ls -i filename<br>
df -i,Checks total Inodes available vs. used (useful if disk is full but df -h shows space).,df -i</p>
<h2>
  
  
  stat [file],"Displays detailed Inode info (access time, modify time, Inode #).",stat index.html
</h2>
<h3>
  
  
  3. Hard Links &amp; Soft Links
</h3>

<p>Hard Links<br>
A Hard Link is essentially a mirror copy or a second name for the exact same file.</p>

<p>It points directly to the Inode of the original file.</p>

<p>If you change the content of the hard link, the original file changes (because they are physically the same data).</p>

<p>If you delete the original file, the hard link still works because the inode and data blocks still exist.</p>

<p>Soft Links (Symbolic Links / Symlinks)<br>
A Soft Link is like a shortcut in Windows.</p>

<p>It points to the file path (name) of the original file, not the inode.</p>

<p>It has its own unique inode.</p>

<p>If you delete the original file, the soft link becomes "broken" (dangling) because the path it points to no longer exists.</p>

<p>Links are pointers to files. Linux has two distinct types.</p>

<div class="table-wrapper-paragraph"><table>
<thead>
<tr>
<th>Feature</th>
<th>Hard Link</th>
<th>Soft Link (Symbolic Link)</th>
</tr>
</thead>
<tbody>
<tr>
<td><strong>Definition</strong></td>
<td>A direct mirror copy. It points to the <strong>same Inode</strong> as the original file.</td>
<td>A shortcut. It is a special file that points to the <strong>path/name</strong> of another file.</td>
</tr>
<tr>
<td><strong>Inode</strong></td>
<td>Shares the <strong>same</strong> Inode number as the original.</td>
<td>Has its <strong>own unique</strong> Inode number.</td>
</tr>
<tr>
<td><strong>Deletion</strong></td>
<td>If you delete the original file, the hard link <strong>still works</strong> (data remains).</td>
<td>If you delete the original file, the soft link becomes <strong>broken</strong> (dangling).</td>
</tr>
<tr>
<td><strong>Cross-filesystem</strong></td>
<td>Cannot link across different partitions/drives.</td>
<td>Can link across different partitions/drives.</td>
</tr>
<tr>
<td><strong>Directory</strong></td>
<td>Generally cannot link to directories.</td>
<td>Can link to directories.</td>
</tr>
<tr>
<td><strong>Command</strong></td>
<td><code>ln target link_name</code></td>
<td><code>ln -s target link_name</code></td>
</tr>
</tbody>
</table></div>

<p>Commands for Hard &amp; Soft Links<br>
How to create and identify links using the ln command.</p>

<p>Creating a Hard Link</p>

<p>Syntax: ln [original_file] [link_name]</p>

<p>Example: ln script.py hard_script.py</p>

<p>Check: If you run ls -i, both files will share the same Inode number.</p>

<p>Creating a Soft (Symbolic) Link</p>

<p>Syntax: ln -s [original_path] [link_name]</p>

<p>Example: ln -s /var/www/html/index.php my_link</p>
<h2>
  
  
  Check: ls -l will show an arrow: my_link -&gt; /var/www/html/index.php. The Inode numbers will be different.
</h2>
<h3>
  
  
  4. LVM (Logical Volume Manager)
</h3>

<p>LVM allows for flexible disk management. Instead of dealing with fixed physical partitions (like <code>/dev/sda1</code>), LVM creates an abstraction layer. This allows you to resize volumes on the fly.</p>

<p><strong>The LVM Architecture:</strong></p>

<ol>
<li>
<strong>Physical Volume (PV):</strong> The actual raw disk or partition (e.g., <code>/dev/sdb</code>).</li>
<li>
<strong>Volume Group (VG):</strong> A pool of storage created by combining one or more PVs. Think of this as a "storage bucket."</li>
<li>
<strong>Logical Volume (LV):</strong> A virtual partition carved out of the VG. This is what you actually format and mount (e.g., <code>/dev/mapper/vg_data-lv_logs</code>).</li>
</ol>

<p><strong>Why use LVM?</strong></p>

<ul>
<li>
<strong>Resizing:</strong> You can extend a Logical Volume if it runs out of space (provided the Volume Group has space) without rebooting.</li>
<li>
<strong>Spanning:</strong> A single file system can span across multiple physical hard drives.</li>
<li>
<strong>Snapshots:</strong> You can take a frozen copy of the file system for backups.</li>
</ul>

<p>LVM (Logical Volume Manager) Commands<br>
LVM involves three steps: Physical Volumes (PV) → Volume Groups (VG) → Logical Volumes (LV).</p>

<p>Step A: Physical Volume (PV)<br>
pvcreate /dev/sdb: Initialize a disk or partition for LVM use.</p>

<p>pvs: Short summary of physical volumes.</p>

<p>pvdisplay: Detailed info about PVs.</p>

<p>Step B: Volume Group (VG)<br>
vgcreate my_vg /dev/sdb: Create a group named "my_vg" using the PV.</p>

<p>vgs: Short summary of volume groups.</p>

<p>vgextend my_vg /dev/sdc: Add a new disk to an existing group to increase capacity.</p>

<p>Step C: Logical Volume (LV)<br>
lvcreate -L 10G -n my_lv my_vg: Create a 10GB logical partition named "my_lv" inside "my_vg".</p>

<p>lvs: Short summary of logical volumes.</p>

<p>lvextend -L +5G /dev/my_vg/my_lv: Increase the size of the logical volume by 5GB.</p>
<h2>
  
  
  resize2fs /dev/my_vg/my_lv: Crucial step. Resizes the file system to use the newly added space (for ext4).
</h2>
<h3>
  
  
  5. RAID Levels (Redundant Array of Independent Disks)
</h3>

<p>RAID combines multiple physical disk drive components into one or more logical units for data redundancy, performance improvement, or both.</p>

<ul>
<li><strong>RAID 0 (Striping):</strong></li>
<li>
<strong>Concept:</strong> Splits data evenly across two or more disks.</li>
<li>
<strong>Pros:</strong> Very high performance (read/write speed).</li>
<li><p><strong>Cons:</strong> <strong>Zero redundancy.</strong> If one disk fails, <em>all</em> data is lost.</p></li>
<li><p><strong>RAID 1 (Mirroring):</strong></p></li>
<li><p><strong>Concept:</strong> Writes the exact same data to two disks simultaneously.</p></li>
<li><p><strong>Pros:</strong> High redundancy. If one drive fails, the other has a full copy.</p></li>
<li><p><strong>Cons:</strong> You lose 50% of your storage capacity (2TB + 2TB drives = 2TB usable space).</p></li>
<li><p><strong>RAID 5 (Striping with Parity):</strong></p></li>
<li><p><strong>Concept:</strong> Stripes data and parity (error correction) information across 3 or more disks.</p></li>
<li><p><strong>Pros:</strong> Good balance of performance and redundancy. Can survive 1 disk failure.</p></li>
<li><p><strong>Cons:</strong> Write performance is slower due to parity calculations.</p></li>
<li><p><strong>RAID 10 (1+0):</strong></p></li>
<li><p><strong>Concept:</strong> A stripe of mirrors. It combines the speed of RAID 0 with the redundancy of RAID 1.</p></li>
<li><p><strong>Pros:</strong> High speed and high redundancy.</p></li>
<li><p><strong>Cons:</strong> Expensive (requires at least 4 disks and loses 50% capacity).<br>
RAID Commands<br>
Software RAID in Linux is typically managed using the mdadm tool.</p></li>
</ul>

<p>cat /proc/mdstat: The quickest way to check the status of active RAID arrays.</p>

<p>mdadm --detail /dev/md0: specific details about a RAID array (replace /dev/md0 with your array).</p>

<p>mdadm --create /dev/md0 --level=1 --raid-devices=2 /dev/sdb /dev/sdc: Example command to create a RAID 1 (Mirror) array using two disks.</p>

<p>Here is a safe, executable Bash script that simulates these concepts. You can copy this code, save it as a file (e.g., practice_links.sh), and run it in your terminal.</p>

<p>This script will create a temporary directory, generate files, create links, and display the Inode numbers so you can see the differences clearly.</p>

<p>The Practice Script<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight plaintext"><code>#!/bin/bash

# 1. Setup a sandbox directory so we don't mess up your actual files
mkdir -p link_practice
cd link_practice
echo "--- Starting Inode &amp; Link Practice ---"
echo ""

# 2. Create an original file
echo "This is the original content." &gt; original.txt
echo "[1] Created 'original.txt'."
ls -li original.txt
echo "--------------------------------------"

# 3. Create a Hard Link
ln original.txt hard_link.txt
echo "[2] Created Hard Link named 'hard_link.txt'."
echo "Check the Inode numbers (1st column) below. They should be identical:"
ls -li original.txt hard_link.txt
echo "--------------------------------------"

# 4. Create a Soft Link
ln -s original.txt soft_link.txt
echo "[3] Created Soft Link named 'soft_link.txt'."
echo "Check the Inode numbers below. The Soft Link has a DIFFERENT Inode:"
ls -li original.txt soft_link.txt
echo "--------------------------------------"

# 5. Modify the Hard Link
echo " [Update] Appending text to 'hard_link.txt'..."
echo " (Added via hard link)" &gt;&gt; hard_link.txt
echo "Reading 'original.txt':"
cat original.txt
echo "--&gt; Notice: The original file changed because they point to the same data."
echo "--------------------------------------"

# 6. Delete the Original File
rm original.txt
echo "[4] DELETED 'original.txt'."
echo ""

# 7. Check the results
echo "Checking Hard Link content:"
if cat hard_link.txt; then
    echo "--&gt; SUCCESS: Hard link still works (Data still exists on disk)."
else
    echo "--&gt; FAIL."
fi
echo ""

echo "Checking Soft Link content:"
if cat soft_link.txt 2&gt;/dev/null; then
    echo "--&gt; SUCCESS."
else
    echo "--&gt; FAILED: Soft link is broken (Dangling link). It points to a filename that is gone."
fi

# Cleanup
cd ..
rm -rf link_practice
echo ""
echo "--- End of Simulation (Cleaned up temporary files) ---"
</code></pre>

</div>



<p>The LVM Practice Script<br>
Save this as lvm_practice.sh, make it executable (chmod +x lvm_practice.sh), and run it with sudo ./lvm_practice.sh.<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight plaintext"><code>#!/bin/bash

# Ensure script is run as root
if [[ $EUID -ne 0 ]]; then
   echo "This script must be run as root (use sudo)." 
   exit 1
fi

echo "--- Starting LVM Simulation ---"
echo "We will create 'fake' hard drives using files to practice LVM safely."
echo ""

# 1. Create fake physical disks (100MB each)
echo "[1] Creating 2 fake disk images (100MB each)..."
dd if=/dev/zero of=disk1.img bs=1M count=100 status=none
dd if=/dev/zero of=disk2.img bs=1M count=100 status=none

# Attach them to loop devices so Linux sees them as block devices
LOOP1=$(losetup --find --show disk1.img)
LOOP2=$(losetup --find --show disk2.img)
echo "--&gt; Created fake disks at: $LOOP1 and $LOOP2"
echo "--------------------------------------"

# 2. Initialize Physical Volumes (PV)
echo "[2] Initializing Physical Volumes (pvcreate)..."
pvcreate $LOOP1 $LOOP2
echo "--&gt; PVs created."
echo "--------------------------------------"

# 3. Create a Volume Group (VG)
echo "[3] Creating Volume Group 'demo_vg' using the first disk..."
vgcreate demo_vg $LOOP1
vgs
echo "--&gt; Volume Group 'demo_vg' created."
echo "--------------------------------------"

# 4. Create a Logical Volume (LV)
echo "[4] Creating a 50MB Logical Volume 'demo_lv'..."
lvcreate -L 50M -n demo_lv demo_vg
lvs
echo "--&gt; Logical Volume 'demo_lv' created."
echo "--------------------------------------"

# 5. Format and Mount
echo "[5] Formatting with ext4 and mounting..."
mkfs.ext4 /dev/demo_vg/demo_lv &gt; /dev/null
mkdir -p /mnt/lvm_demo
mount /dev/demo_vg/demo_lv /mnt/lvm_demo

# Check space
echo "--&gt; Current Disk Usage:"
df -h /mnt/lvm_demo
echo "--------------------------------------"

# 6. EXTENDING the Storage (The Magic of LVM)
echo "[6] SCENARIO: We are running out of space!"
echo "    We will add the second disk to the group and extend the volume."
echo ""

echo "Step A: Extend Volume Group (add disk2)..."
vgextend demo_vg $LOOP2

echo "Step B: Extend Logical Volume (add 80MB)..."
lvextend -L +80M /dev/demo_vg/demo_lv

echo "Step C: Resize Filesystem (make the OS see the new space)..."
resize2fs /dev/demo_vg/demo_lv &gt; /dev/null

echo ""
echo "--&gt; New Disk Usage (Notice the size increase!):"
df -h /mnt/lvm_demo
echo "--------------------------------------"

# 7. CLEANUP (Undo everything)
echo "[7] Cleaning up..."
umount /mnt/lvm_demo
lvremove -y /dev/demo_vg/demo_lv
vgremove -y demo_vg
pvremove -y $LOOP1 $LOOP2
losetup -d $LOOP1
losetup -d $LOOP2
rm disk1.img disk2.img
rmdir /mnt/lvm_demo

echo "--- Simulation Complete. System returned to normal. ---"
</code></pre>

</div>



<p>What you will learn from this script:<br>
pvcreate: You'll see how raw "disks" are prepped.</p>

<p>vgextend: You'll see how easy it is to add a second "disk" to your storage pool.</p>

<p>lvextend + resize2fs: You'll witness the storage capacity of a mounted folder jump from ~50MB to ~130MB instantly without losing data.</p>

<p>Troubleshooting<br>
If the script fails halfway (e.g., if you cancel it with Ctrl+C), you might have leftover loop devices. You can clean them manually:</p>

<p>sudo losetup -D (Detaches all loop devices)</p>

<p>sudo rm disk1.img disk2.img</p>

<p>Here is the breakdown of <strong>Linux File Permissions</strong>. This is one of the most critical security concepts in Linux. If you master this, you control who can see, touch, or run anything on your server.</p>


<h2>
  
  
  1. The Anatomy of a Permission
</h2>

<p>When you run <code>ls -l</code>, you see a string like this:<br>
<code>drwxr-xr--</code></p>

<p>This string is split into 4 parts:</p>

<ol>
<li>
<strong>File Type:</strong> The first character.</li>
<li>
<code>-</code> = Normal File</li>
<li>
<code>d</code> = Directory</li>
<li><p><code>l</code> = Link</p></li>
<li><p><strong>User (u):</strong> The <strong>Owner</strong> of the file. (Next 3 chars)</p></li>
<li><p><strong>Group (g):</strong> The <strong>Group</strong> assigned to the file. (Next 3 chars)</p></li>
<li><p><strong>Others (o):</strong> Everyone else (the public). (Last 3 chars)</p></li>
</ol>


<h2>
  
  
  2. The "What": Read, Write, Execute
</h2>

<p>Linux uses a point system (Octal) for permissions. You just add the numbers up to get the permission level.</p>

<div class="table-wrapper-paragraph"><table>
<thead>
<tr>
<th>Permission</th>
<th>Letter</th>
<th>Number Value</th>
<th>Description</th>
</tr>
</thead>
<tbody>
<tr>
<td><strong>Read</strong></td>
<td><strong>r</strong></td>
<td><strong>4</strong></td>
<td>Can open and view the file content.</td>
</tr>
<tr>
<td><strong>Write</strong></td>
<td><strong>w</strong></td>
<td><strong>2</strong></td>
<td>Can modify or delete the file content.</td>
</tr>
<tr>
<td><strong>Execute</strong></td>
<td><strong>x</strong></td>
<td><strong>1</strong></td>
<td>Can run the file as a program/script.</td>
</tr>
<tr>
<td><strong>None</strong></td>
<td><strong>-</strong></td>
<td><strong>0</strong></td>
<td>No access.</td>
</tr>
</tbody>
</table></div>
<h3>
  
  
  The Math (Common Examples)
</h3>

<ul>
<li>
<strong>7</strong> (<code>rwx</code>): 4 + 2 + 1 (Read + Write + Execute) → <em>Full Control</em>
</li>
<li>
<strong>6</strong> (<code>rw-</code>): 4 + 2 (Read + Write) → <em>Standard for files</em>
</li>
<li>
<strong>5</strong> (<code>r-x</code>): 4 + 1 (Read + Execute) → <em>Standard for programs you don't want edited</em>
</li>
<li>
<strong>4</strong> (<code>r--</code>): 4 (Read only) → <em>Strict read-only</em>
</li>
</ul>


<h2>
  
  
  3. The Commands
</h2>
<h3>
  
  
  A. <code>chmod</code> (Change Mode)
</h3>

<p>This command changes the permissions (Who can do What).</p>

<p><strong>Method 1: The Numeric Way (Most Common)</strong><br>
You provide 3 numbers: One for User, one for Group, one for Others.</p>

<ul>
<li><code>chmod 777 file.txt</code></li>
<li>User: 7 (rwx), Group: 7 (rwx), Others: 7 (rwx).</li>
<li><p><strong>Danger:</strong> Everyone can delete this file.</p></li>
<li><p><code>chmod 644 file.txt</code></p></li>
<li><p>User: 6 (rw-), Group: 4 (r--), Others: 4 (r--).</p></li>
<li><p><strong>Standard:</strong> Owner can read/write; everyone else can only read.</p></li>
<li><p><code>chmod 755 script.sh</code></p></li>
<li><p>User: 7 (rwx), Group: 5 (r-x), Others: 5 (r-x).</p></li>
<li><p><strong>Standard for Scripts:</strong> Everyone can run it, but only the owner can change the code.</p></li>
</ul>

<p><strong>Method 2: The Symbolic Way</strong><br>
Useful for quick tweaks without doing math.</p>

<ul>
<li>
<code>u</code> (User), <code>g</code> (Group), <code>o</code> (Others), <code>a</code> (All)</li>
<li>
<code>+</code> (Add), <code>-</code> (Remove)</li>
<li>
<code>chmod +x script.sh</code> → Makes it executable for <strong>everyone</strong>.</li>
<li>
<code>chmod u+x script.sh</code> → Makes it executable <strong>only for the owner</strong>.</li>
<li>
<code>chmod o-r secret.txt</code> → Removes read permission for <strong>others</strong> (privacy).</li>
</ul>


<h3>
  
  
  B. <code>chown</code> (Change Owner)
</h3>

<p>This command changes <strong>who</strong> owns the file and which <strong>group</strong> it belongs to. You usually need <code>sudo</code> for this.</p>

<ul>
<li>
<strong>Syntax:</strong> <code>chown [user]:[group] [file]</code>
</li>
</ul>

<p><strong>Examples:</strong></p>

<ul>
<li><code>chown steve file.txt</code></li>
<li><p>Changes the owner to "steve".</p></li>
<li><p><code>chown steve:developers file.txt</code></p></li>
<li><p>Changes owner to "steve" and group to "developers".</p></li>
<li><p><code>chown -R steve:developers /var/www/html</code></p></li>
<li><p><strong>Recursive:</strong> Changes owner/group for the folder and <em>everything inside it</em>.</p></li>
</ul>


<h2>
  
  
  4. Practice Simulation
</h2>

<p>Here is a safe script to practice permissions. It will create files and let you try to "break" the security rules to see what happens.</p>

<p><strong>Save as:</strong> <code>perm_practice.sh</code><br>
<strong>Run:</strong> <code>chmod +x perm_practice.sh</code> then <code>./perm_practice.sh</code><br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight shell"><code><span class="c">#!/bin/bash</span>

<span class="c"># Setup</span>
<span class="nb">mkdir</span> <span class="nt">-p</span> perm_lab
<span class="nb">cd </span>perm_lab
<span class="nb">echo</span> <span class="s2">"This is a Top Secret File"</span> <span class="o">&gt;</span> secret.txt

<span class="nb">echo</span> <span class="s2">"--- Permission Practice ---"</span>

<span class="c"># 1. Standard Permissions</span>
<span class="nb">echo</span> <span class="s2">"[1] Current permissions for secret.txt:"</span>
<span class="nb">ls</span> <span class="nt">-l</span> secret.txt
<span class="nb">echo</span> <span class="s2">"-----------------------------------"</span>

<span class="c"># 2. Locking it down (Read Only)</span>
<span class="nb">echo</span> <span class="s2">"[2] Running: chmod 400 secret.txt (Read-only for User, Nothing for others)"</span>
<span class="nb">chmod </span>400 secret.txt
<span class="nb">ls</span> <span class="nt">-l</span> secret.txt

<span class="nb">echo</span> <span class="s2">"--&gt; Attempting to READ the file:"</span>
<span class="nb">cat </span>secret.txt
<span class="nb">echo</span> <span class="s2">"    (Success!)"</span>

<span class="nb">echo</span> <span class="s2">"--&gt; Attempting to WRITE to the file:"</span>
<span class="c"># We use a trick to catch the error message safely</span>
<span class="k">if </span><span class="nb">echo</span> <span class="s2">"Hacker was here"</span> <span class="o">&gt;&gt;</span> secret.txt 2&gt;/dev/null<span class="p">;</span> <span class="k">then
    </span><span class="nb">echo</span> <span class="s2">"    Written successfully."</span>
<span class="k">else
    </span><span class="nb">echo</span> <span class="s2">"    FAILED: Permission Denied! (As expected)"</span>
<span class="k">fi
</span><span class="nb">echo</span> <span class="s2">"-----------------------------------"</span>

<span class="c"># 3. Making it Executable</span>
<span class="nb">echo</span> <span class="s2">"[3] We created a script called 'hello.sh'"</span>
<span class="nb">echo</span> <span class="s1">'#!/bin/bash'</span> <span class="o">&gt;</span> hello.sh
<span class="nb">echo</span> <span class="s1">'echo "Hello World!"'</span> <span class="o">&gt;&gt;</span> hello.sh

<span class="nb">echo</span> <span class="s2">"--&gt; Attempting to run it ./hello.sh"</span>
<span class="c"># Try to run it before giving permission</span>
<span class="k">if</span> ./hello.sh 2&gt;/dev/null<span class="p">;</span> <span class="k">then
    </span><span class="nb">echo</span> <span class="s2">"    Ran successfully."</span>
<span class="k">else
    </span><span class="nb">echo</span> <span class="s2">"    FAILED: Permission Denied! (It lacks the 'x' flag)"</span>
<span class="k">fi

</span><span class="nb">echo</span> <span class="s2">"--&gt; Fix: Running chmod u+x hello.sh"</span>
<span class="nb">chmod </span>u+x hello.sh
<span class="nb">ls</span> <span class="nt">-l</span> hello.sh
./hello.sh
<span class="nb">echo</span> <span class="s2">"-----------------------------------"</span>

<span class="c"># Cleanup</span>
<span class="nb">cd</span> ..
<span class="nb">rm</span> <span class="nt">-rf</span> perm_lab
<span class="nb">echo</span> <span class="s2">""</span>
<span class="nb">echo</span> <span class="s2">"--- Lab Complete ---"</span>

</code></pre>

</div>



<p>Here are the <strong>Special Permissions</strong> in Linux.</p>

<p>In standard permissions (rwx), if I create a file, I own it. If I run a script, it runs with <em>my</em> level of power. Special permissions break these rules to allow for system administration and team collaboration.</p>




<h3>
  
  
  1. SUID (Set User ID)
</h3>

<p><strong>"The Temporary Root Badge"</strong></p>

<ul>
<li>
<strong>Function:</strong> When a file with SUID is executed, it runs with the permissions of the <strong>file owner</strong>, not the user who ran it.</li>
<li>
<strong>Classic Example:</strong> The <code>passwd</code> command.</li>
<li>Normal users need to change their passwords, which requires writing to <code>/etc/shadow</code>.</li>
<li>Normal users <em>cannot</em> write to <code>/etc/shadow</code>.</li>
<li><p>The <code>passwd</code> binary is owned by <strong>root</strong> and has the <strong>SUID</strong> bit set. When you run it, you temporarily "become" root for that specific task.</p></li>
<li><p><strong>Symbol:</strong> Look for an <strong><code>s</code></strong> in the <strong>User</strong> execute slot.</p></li>
<li><p><code>rwsr-xr-x</code></p></li>
<li><p><strong>Numeric Value:</strong> 4000</p></li>
</ul>

<blockquote>
<p><strong>⚠️ DevOps Security Warning:</strong> SUID is dangerous. If a hacker finds a vulnerable SUID file (like a script that lets them spawn a shell), they can become root instantly. Always audit your system for unknown SUID files: <code>find / -perm /4000</code></p>
</blockquote>




<h3>
  
  
  2. SGID (Set Group ID)
</h3>

<p><strong>"The Collaboration Tool"</strong></p>

<p>SGID works differently depending on whether it is on a file or a directory.</p>

<p><strong>A. On a File:</strong></p>

<ul>
<li>Similar to SUID, but the program runs with the permissions of the <strong>File Group</strong>.</li>
</ul>

<p><strong>B. On a Directory (Most Common DevOps Use Case):</strong></p>

<ul>
<li>
<strong>Function:</strong> Any new file created inside this directory will inherit the <strong>Group ownership of the directory</strong>, not the User's primary group.</li>
<li>
<strong>Why use it?</strong> If Developers Alice and Bob share a folder <code>/var/www/project</code>.</li>
<li>Without SGID: Alice creates a file, it belongs to <code>alice:alice</code>. Bob cannot edit it.</li>
<li><p>With SGID: Alice creates a file, it belongs to <code>alice:developers</code>. Bob (part of <code>developers</code> group) can edit it.</p></li>
<li><p><strong>Symbol:</strong> Look for an <strong><code>s</code></strong> in the <strong>Group</strong> execute slot.</p></li>
<li><p><code>rwxr-sr-x</code></p></li>
<li><p><strong>Numeric Value:</strong> 2000</p></li>
</ul>




<h3>
  
  
  3. Sticky Bit
</h3>

<p><strong>"The Anti-Sabotage Rule"</strong></p>

<ul>
<li>
<strong>Function:</strong> Used on shared directories. It ensures that <strong>only the owner</strong> of a file (or root) can delete or rename it. Even if others have "Write" permission on the folder, they cannot delete <em>your</em> files.</li>
<li>
<strong>Classic Example:</strong> The <code>/tmp</code> directory.</li>
<li>Every user and process writes temporary files here.</li>
<li><p>Without the Sticky Bit, User A could delete User B's temp files, crashing User B's programs.</p></li>
<li><p><strong>Symbol:</strong> Look for a <strong><code>t</code></strong> in the <strong>Others</strong> execute slot.</p></li>
<li><p><code>rwxrwxrwt</code></p></li>
<li><p><strong>Numeric Value:</strong> 1000</p></li>
</ul>




<h3>
  
  
  Summary Table
</h3>

<div class="table-wrapper-paragraph"><table>
<thead>
<tr>
<th>Permission</th>
<th>Acronym</th>
<th>On File</th>
<th>On Directory</th>
<th>Numeric</th>
<th>Symbol</th>
</tr>
</thead>
<tbody>
<tr>
<td><strong>Set UID</strong></td>
<td><strong>SUID</strong></td>
<td>Runs as <strong>File Owner</strong>
</td>
<td>(Ignored)</td>
<td>4000</td>
<td><code>rws------</code></td>
</tr>
<tr>
<td><strong>Set GID</strong></td>
<td><strong>SGID</strong></td>
<td>Runs as <strong>File Group</strong>
</td>
<td>Files inherit <strong>Dir Group</strong>
</td>
<td>2000</td>
<td><code>---rws---</code></td>
</tr>
<tr>
<td><strong>Sticky Bit</strong></td>
<td><strong>---</strong></td>
<td>(Ignored)</td>
<td>Only <strong>Owner</strong> deletes</td>
<td>1000</td>
<td><code>------rwt</code></td>
</tr>
</tbody>
</table></div>

<h3>
  
  
  Commands to Set Special Permissions
</h3>

<p>You can add these bits using the symbolic method (easiest) or numeric method.</p>

<p><strong>1. Setting SUID:</strong></p>

<ul>
<li><code>chmod u+s filename</code></li>
<li>
<code>chmod 4755 filename</code> (4=SUID, 7=User, 5=Group, 5=Others)</li>
</ul>

<p><strong>2. Setting SGID:</strong></p>

<ul>
<li><code>chmod g+s directory_name</code></li>
<li><code>chmod 2775 directory_name</code></li>
</ul>

<p><strong>3. Setting Sticky Bit:</strong></p>

<ul>
<li><code>chmod +t directory_name</code></li>
<li>
<code>chmod 1777 directory_name</code> (Standard for <code>/tmp</code>)</li>
</ul>




<h3>
  
  
  Quick Practice Check
</h3>

<p>Run this on your terminal to see the Sticky Bit in action on your own system:</p>

<p><code>ls -ld /tmp</code></p>

<p>You should see something like: <code>drwxrwxrw</code><strong><code>t</code></strong> ... root root ... /tmp</p>




<p>This is a complete, hands-on "Master Class" tutorial. It combines every concept we have discussed into a single workflow.</p>

<p><strong>Scenario:</strong> You are a Junior DevOps Engineer setting up a new server. You need to manage storage efficiently and secure files for a development team.</p>

<p><strong>Prerequisites:</strong> A Linux terminal (Ubuntu/CentOS/WSL) and <code>sudo</code> access.</p>




<h1>
  
  
  Phase 1: Storage Internals (Inodes &amp; Links)
</h1>

<p><strong>Goal:</strong> Understand how Linux stores files and how to create shortcuts safely.</p>

<h3>
  
  
  Step 1: Check your File System
</h3>

<p>First, let's see what storage you currently have.<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight shell"><code><span class="nb">df</span> <span class="nt">-hT</span>

</code></pre>

</div>



<ul>
<li>
<strong>Look for:</strong> The <code>Type</code> column (usually ext4 or xfs) and the <code>Avail</code> column (free space).</li>
</ul>

<h3>
  
  
  Step 2: Inode Investigation
</h3>

<p>Create a sandbox directory and a file.<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight shell"><code><span class="nb">mkdir </span>storage_lab
<span class="nb">cd </span>storage_lab
<span class="nb">echo</span> <span class="s2">"This is my data"</span> <span class="o">&gt;</span> original.txt

</code></pre>

</div>



<p>Check the <strong>Inode number</strong> (the file's unique ID):<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight shell"><code><span class="nb">ls</span> <span class="nt">-i</span> original.txt

</code></pre>

</div>



<ul>
<li><em>Note the number returned (e.g., 123456).</em></li>
</ul>

<h3>
  
  
  Step 3: Hard vs. Soft Links
</h3>

<p>Now, let's create both types of links.</p>

<ol>
<li>
<strong>Create a Hard Link</strong> (A mirror copy):
</li>
</ol>

<div class="highlight js-code-highlight">
<pre class="highlight shell"><code><span class="nb">ln </span>original.txt hard_link.txt

</code></pre>

</div>



<ol>
<li>
<strong>Create a Soft Link</strong> (A shortcut):
</li>
</ol>

<div class="highlight js-code-highlight">
<pre class="highlight shell"><code><span class="nb">ln</span> <span class="nt">-s</span> original.txt soft_link.txt

</code></pre>

</div>



<ol>
<li>
<strong>Compare them:</strong>
</li>
</ol>

<div class="highlight js-code-highlight">
<pre class="highlight shell"><code><span class="nb">ls</span> <span class="nt">-li</span>

</code></pre>

</div>



<ul>
<li>
<strong>Observation:</strong> <code>original.txt</code> and <code>hard_link.txt</code> share the <strong>same</strong> Inode number. <code>soft_link.txt</code> has a <strong>different</strong> Inode number.</li>
</ul>

<ol>
<li>
<strong>The "Destruction" Test:</strong>
Delete the original file and see what survives.
</li>
</ol>

<div class="highlight js-code-highlight">
<pre class="highlight shell"><code><span class="nb">rm </span>original.txt
<span class="nb">cat </span>hard_link.txt   <span class="c"># This works (Data is safe)</span>
<span class="nb">cat </span>soft_link.txt   <span class="c"># This fails (Broken link)</span>

</code></pre>

</div>






<h1>
  
  
  Phase 2: Logical Volume Manager (LVM)
</h1>

<p><strong>Goal:</strong> Simulate adding a new hard drive and expanding storage without downtime. This is crucial for resizing AWS EBS volumes or DigitalOcean Droplets.</p>

<h3>
  
  
  Step 1: Create "Fake" Hard Drives
</h3>

<p>Since we can't physically insert a drive, we will create two 100MB files that act like drives.<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight shell"><code><span class="nb">sudo dd </span><span class="k">if</span><span class="o">=</span>/dev/zero <span class="nv">of</span><span class="o">=</span>disk1.img <span class="nv">bs</span><span class="o">=</span>1M <span class="nv">count</span><span class="o">=</span>100
<span class="nb">sudo dd </span><span class="k">if</span><span class="o">=</span>/dev/zero <span class="nv">of</span><span class="o">=</span>disk2.img <span class="nv">bs</span><span class="o">=</span>1M <span class="nv">count</span><span class="o">=</span>100
<span class="c"># Attach them as loop devices</span>
<span class="nb">sudo </span>losetup /dev/loop10 disk1.img
<span class="nb">sudo </span>losetup /dev/loop11 disk2.img

</code></pre>

</div>



<h3>
  
  
  Step 2: Initialize Physical Volumes (PV)
</h3>

<p>Tell LVM to use these disks.<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight shell"><code><span class="nb">sudo </span>pvcreate /dev/loop10 /dev/loop11
<span class="nb">sudo </span>pvs

</code></pre>

</div>



<h3>
  
  
  Step 3: Create a Volume Group (VG)
</h3>

<p>Combine the disks into a storage pool named <code>app_data</code>.<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight shell"><code><span class="nb">sudo </span>vgcreate app_data /dev/loop10
<span class="nb">sudo </span>vgs

</code></pre>

</div>



<h3>
  
  
  Step 4: Create a Logical Volume (LV)
</h3>

<p>Carve out a 50MB partition from that pool.<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight shell"><code><span class="nb">sudo </span>lvcreate <span class="nt">-L</span> 50M <span class="nt">-n</span> logs_lv app_data
<span class="nb">sudo </span>lvs

</code></pre>

</div>



<h3>
  
  
  Step 5: Format and Mount
</h3>

<p>Make it usable for the OS.<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight shell"><code><span class="c"># Format as ext4</span>
<span class="nb">sudo </span>mkfs.ext4 /dev/app_data/logs_lv
<span class="c"># Create a mount point and mount it</span>
<span class="nb">sudo mkdir</span> /mnt/logs
<span class="nb">sudo </span>mount /dev/app_data/logs_lv /mnt/logs

</code></pre>

</div>



<p>Check the size:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight shell"><code><span class="nb">df</span> <span class="nt">-h</span> /mnt/logs

</code></pre>

</div>



<p><em>(You should see roughly 45-50MB available).</em></p>

<h3>
  
  
  Step 6: The "Oh No, Disk Full!" Scenario
</h3>

<p>Imagine your logs filled up the partition. Let's extend it on the fly using the second disk.<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight shell"><code><span class="c"># 1. Add the second disk to the group</span>
<span class="nb">sudo </span>vgextend app_data /dev/loop11

<span class="c"># 2. Extend the Logical Volume by 80MB</span>
<span class="nb">sudo </span>lvextend <span class="nt">-L</span> +80M /dev/app_data/logs_lv

<span class="c"># 3. Resize the file system (The most important step!)</span>
<span class="nb">sudo </span>resize2fs /dev/app_data/logs_lv

<span class="c"># 4. Check the new size</span>
<span class="nb">df</span> <span class="nt">-h</span> /mnt/logs

</code></pre>

</div>



<p><em>(You should now see roughly 120-130MB. You just resized storage without rebooting!)</em></p>




<h1>
  
  
  Phase 3: Understanding RAID
</h1>

<p><strong>Goal:</strong> Conceptual check. (We won't build a RAID array as it conflicts with the LVM lab on a single machine, but we will check for it).</p>

<ol>
<li>
<strong>Check RAID Status:</strong>
Run this command to see if your current system uses Software RAID.
</li>
</ol>

<div class="highlight js-code-highlight">
<pre class="highlight shell"><code><span class="nb">cat</span> /proc/mdstat

</code></pre>

</div>



<ul>
<li>If you see <code>unused devices: &lt;none&gt;</code>, you are not running RAID.</li>
<li>In a real server, you might use <code>mdadm</code> to pair disks (e.g., RAID 1 for mirroring).</li>
</ul>




<h1>
  
  
  Phase 4: Basic Permissions
</h1>

<p><strong>Goal:</strong> Control who can read/write your files.</p>

<h3>
  
  
  Step 1: Create a "Secret" File
</h3>



<div class="highlight js-code-highlight">
<pre class="highlight shell"><code><span class="nb">cd</span> /mnt/logs
<span class="nb">sudo touch </span>secret_config.txt

</code></pre>

</div>



<h3>
  
  
  Step 2: Lock it down
</h3>

<p>Remove all permissions for "Group" and "Others".<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight shell"><code><span class="nb">sudo chmod </span>600 secret_config.txt
<span class="nb">ls</span> <span class="nt">-l</span> secret_config.txt

</code></pre>

</div>



<ul>
<li>
<strong>Result:</strong> Only the owner (root) can read/write. If you switch users, you cannot see this file.</li>
</ul>

<h3>
  
  
  Step 3: Change Ownership (chown)
</h3>

<p>Give this file to your current user so you can edit it without sudo.<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight shell"><code><span class="c"># Replace 'your_user' with your actual username (run 'whoami' to check)</span>
<span class="nb">sudo chown</span> <span class="nv">$USER</span>:<span class="nv">$USER</span> secret_config.txt
<span class="nb">ls</span> <span class="nt">-l</span> secret_config.txt

</code></pre>

</div>






<h1>
  
  
  Phase 5: Advanced Permissions (The Pro Stuff)
</h1>

<p><strong>Goal:</strong> Use SGID for team collaboration and Sticky Bit for safety.</p>

<h3>
  
  
  Step 1: SGID (Shared Group Collaboration)
</h3>

<p>This is how multiple developers work in one folder (like <code>/var/www/html</code> for WordPress).</p>

<ol>
<li>Create a shared directory.
</li>
</ol>

<div class="highlight js-code-highlight">
<pre class="highlight shell"><code><span class="nb">sudo mkdir</span> /mnt/logs/team_project

</code></pre>

</div>



<ol>
<li>Set the <strong>SGID</strong> bit (the <code>2</code> in <code>2777</code>).
</li>
</ol>

<div class="highlight js-code-highlight">
<pre class="highlight shell"><code><span class="nb">sudo chmod </span>2777 /mnt/logs/team_project

</code></pre>

</div>



<ol>
<li>Check the permission (Look for the <code>s</code> in the group column).
</li>
</ol>

<div class="highlight js-code-highlight">
<pre class="highlight shell"><code><span class="nb">ls</span> <span class="nt">-ld</span> /mnt/logs/team_project

</code></pre>

</div>



<ul>
<li>
<em>Result:</em> Any file created inside this folder will automatically inherit the group ownership of the folder, ensuring all team members can access it.</li>
</ul>

<h3>
  
  
  Step 2: Sticky Bit (Anti-Delete)
</h3>

<ol>
<li>Set the Sticky Bit (the <code>1</code> in <code>1777</code>).
</li>
</ol>

<div class="highlight js-code-highlight">
<pre class="highlight shell"><code><span class="nb">sudo chmod </span>1777 /mnt/logs/team_project

</code></pre>

</div>



<ol>
<li>Check the permission (Look for the <code>t</code> at the end).
</li>
</ol>

<div class="highlight js-code-highlight">
<pre class="highlight shell"><code><span class="nb">ls</span> <span class="nt">-ld</span> /mnt/logs/team_project

</code></pre>

</div>



<ul>
<li>
<em>Result:</em> Users can write files here, but <strong>only the person who created the file</strong> can delete it. This prevents team members from accidentally deleting each other's work.</li>
</ul>




<h1>
  
  
  Cleanup
</h1>

<p>Since we created fake drives and mount points, let's clean up your system so nothing is left behind.<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight shell"><code><span class="c"># 1. Unmount the partition</span>
<span class="nb">sudo </span>umount /mnt/logs

<span class="c"># 2. Remove LVM components</span>
<span class="nb">sudo </span>lvremove <span class="nt">-y</span> /dev/app_data/logs_lv
<span class="nb">sudo </span>vgremove <span class="nt">-y</span> app_data
<span class="nb">sudo </span>pvremove <span class="nt">-y</span> /dev/loop10 /dev/loop11

<span class="c"># 3. Detach loop devices and delete fake files</span>
<span class="nb">sudo </span>losetup <span class="nt">-d</span> /dev/loop10
<span class="nb">sudo </span>losetup <span class="nt">-d</span> /dev/loop11
<span class="nb">rm</span> ~/storage_lab/disk1.img ~/storage_lab/disk2.img
<span class="nb">rm</span> <span class="nt">-rf</span> ~/storage_lab
<span class="nb">sudo rmdir</span> /mnt/logs

</code></pre>

</div>



<h3>
  
  
  Tutorial Complete
</h3>

<p>You have just performed the daily tasks of a Systems Engineer: managing inodes, resizing volumes with LVM, and securing files with advanced permissions!</p>

