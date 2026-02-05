---
Title: Binary Search Trees: Why They’re Great in Memory but Terrible on Disk
Description: 
Author: Quame Jnr
Date: 2026-02-05T22:05:13.000Z
Robots: noindex,nofollow
Template: index
---
<p>Binary search trees (BST) are in-memory sorted data for efficient lookups. They have 2 children nodes, ergo the name binary. For each node, all values of nodes to the left should be less than the node and all values of nodes to the right should be greater than the node.</p>

<p><a href="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2F3297uw96gmro31t6agoa.png" class="article-body-image-wrapper"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2F3297uw96gmro31t6agoa.png" alt="Binary Search Tree" width="576" height="780"></a></p>

<p>This is good because it allows you to do O(log N) search since the tree has O(log N) height. For a tree like ours above with 4 elements, in the worst case you traverse O(log 4) = 2 to find your target since the height is 2. However, BSTs can also be at the mercy of the insertion order, thus we need to constantly rebalance the tree.</p>

<h2>
  
  
  Tree Balancing
</h2>

<p>If we insert numbers [1, 2, 3] then without rebalancing, insertions will determine the tree structure leading us to Fig A. This means O(N) searches, thus defeating the purpose of BST. A BST is seen as unbalanced if the difference between the heights of the two subtrees is greater than 1. We can rebalance by doing a rotation pivot to get Fig B.</p>

<p><a href="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Falr7stixwer0tsij4n3j.png" class="article-body-image-wrapper"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Falr7stixwer0tsij4n3j.png" alt="Rebalancing of BST" width="800" height="603"></a></p>

<h2>
  
  
  Why does it Suck as Disk-Based Storage?
</h2>

<h3>
  
  
  1. Random Insertion Order
</h3>

<p>Each new node is allocated any free disk space and the pointer from its parent This means a node can be far away from its parent node, thus no benefits of locality.</p>

<p>So here's the thing: when the OS is loading data into memory, it doesn't just load the target data, it loads in certain sizes, let's say 16KB. This contiguous segment of memory will contain your target data but also neighboring data. Ergo, if the next data the CPU has to operate on is closer to your target data, it might already be loaded in memory. Since BST puts nodes in any free space, it means you don't benefit from this as your child node might be in another segment of the disk.</p>

<h3>
  
  
  2. Fanout of 2
</h3>

<p>BST having binary nodes means a higher number of elements leads to higher height, and higher height means a higher number of traversals during search. So given 1 million elements, you will have a height of ~20, which means 20 traversals. Coupled with the data being in random disk locations means you have to do a disk seek for each node, read the data to find the pointer to the child node if it's not your target element, and then do another disk seek, another page transfer to read the data, over and over again.</p>

<p>Also, keep in mind that the tree may rebalance during insertions, thus you can't cache pathways to a node as they may change.</p>

<h2>
  
  
  Conclusion
</h2>

<p>BSTs are elegant for in-memory operations; O(log N) searches with simple pointer-chasing works great when everything's in RAM. But the moment you move to disk, those same properties become liabilities. Random allocation kills locality, binary fanout means excessive tree height, and every pointer dereference potentially triggers a disk seek.</p>

<p>This is why disk-based databases don't use binary search trees. Instead, they use B-trees and B+ trees with high fanout (hundreds of children per node instead of 2) and nodes sized to match disk pages. A B-tree with the same 1 million elements? Height of ~3 instead of ~20. That's the difference between 3 disk reads and 20.</p>

<p>Sometimes data structures that work brilliantly in one context can be completely wrong for another.</p>

