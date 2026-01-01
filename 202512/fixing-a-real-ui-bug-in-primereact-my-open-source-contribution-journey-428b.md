---
Title: 🛠 Fixing a Real UI Bug in PrimeReact: My Open Source Contribution Journey
Description: 
Author: Rohan Satkar | Coderxrohan
Date: 2025-12-31T20:52:51.000Z
Robots: noindex,nofollow
Template: index
---
<p>Recently, I contributed a fix to PrimeReact’s OrganizationChart component, addressing a visual bug that affected real-world use cases. I wanted to share the full journey — from identifying the issue to implementing a maintainable fix and navigating CI/style checks — because this is what real open-source work looks like.</p>

<p>🐞 The Bug</p>

<p>The issue occurred in OrganizationChart when a node had a large number of child nodes.</p>

<p><strong>What users were seeing:</strong></p>

<ul>
<li>Connector lines between parent and children appeared broken or misaligned</li>
<li>Some child nodes looked visually disconnected</li>
<li>The hierarchy became difficult to understand</li>
</ul>

<p><strong>Important detail:</strong></p>

<ul>
<li>The data was correct</li>
<li>The component did not crash</li>
<li>The issue was purely visual, but highly misleading</li>
</ul>

<p>This bug was already reported and labeled as a Bug by the PrimeReact team.</p>

<p><strong>📊 Impact Assessment</strong></p>

<p>I’d rate the severity of this bug as 7/10.</p>

<p><strong>Why it mattered:</strong></p>

<ul>
<li>Organization charts are meant to communicate hierarchy clearly</li>
<li>Broken connectors lead to incorrect interpretation</li>
<li>The issue appeared in real-world org charts with large teams, not just edge cases</li>
</ul>

<p><strong>Why it wasn’t critical:</strong></p>

<ul>
<li>No runtime errors</li>
<li>Small and medium charts worked fine</li>
<li>No data loss or crashes</li>
</ul>

<p>Still, for teams using OrganizationChart at scale, this was a real UX problem.</p>




<p>🔍 Root Cause Analysis</p>

<p>After digging into the component’s implementation, I found the core issue:</p>

<ul>
<li>All child nodes of a parent were rendered in a single table row</li>
<li>Connector lines were drawn using CSS borders</li>
<li>When the number of children increased:</li>
</ul>

<ol>
<li>-    The row became extremely wide</li>
<li>-    Browser layout rounding and CSS limitations kicked in</li>
<li>-    Connector lines no longer aligned correctly</li>
</ol>

<p>This wasn’t a styling mistake or incorrect data — it was a structural layout limitation.</p>

<p>🧠 The Fix Strategy</p>

<p>The key was to <strong>fix the layout without breaking existing behavior.</strong></p>

<p><strong>Constraints:</strong></p>

<ul>
<li>No public API changes</li>
<li>No new props</li>
<li>No CSS hacks</li>
<li>Maintain backward compatibility</li>
</ul>

<p>The approach:</p>

<ol>
<li>Split large sibling groups into multiple rows instead of one wide row</li>
<li>Keep each row’s width within reasonable bounds</li>
<li>Render connector lines per row</li>
<li>Preserve the existing connector logic and styling</li>
<li>Ensure expand/collapse and selection still worked as before</li>
</ol>

<p>This way, small charts behaved exactly the same, while large charts became readable and stable.</p>

<p>🧩 Implementation Details</p>

<ul>
<li>Introduced internal chunking logic to batch child nodes</li>
<li>Calculated colSpan per row instead of globally</li>
<li>Grouped connector rendering (linesDown, linesMiddle, child nodes) per row</li>
<li>Avoided DOM measurements or SVG rewrites to keep risk low</li>
</ul>

<p>The result:</p>

<ul>
<li>Clean hierarchy</li>
<li>Correct connector alignment</li>
<li>No breaking changes</li>
</ul>

<p>🧪 CI, Formatting &amp; ESLint Challenges (Real OSS Life)</p>

<p>Fixing the bug was only half the job.</p>

<p>PrimeReact has strict CI checks, including:</p>

<ul>
<li>Prettier</li>
<li>ESLint (with padding-line-between-statements)</li>
<li>Style consistency rules</li>
</ul>

<p>I had to:</p>

<ul>
<li>Normalize formatting</li>
<li>Add required blank lines between statements</li>
<li>Separate logic commits from style-only commits</li>
<li>Ensure everything passed locally before pushing</li>
</ul>

<p>This part may look small, but it’s critical in large OSS projects.</p>




<p><strong>✅ Final Result</strong><br>
<strong>Before:</strong></p>

<p>Large org charts looked broken</p>

<p>Connector lines misled users</p>

<p>Visual hierarchy was unreliable</p>

<p><strong>After:</strong></p>

<p>Large sibling groups render across multiple rows</p>

<p>Connector lines remain aligned</p>

<p>Hierarchy is clear and readable</p>

<p>Existing behavior remains unchanged</p>

<p>The fix is now linked to the original issue and ready for maintainers to review.</p>

<p><strong>📚 What I Learned</strong></p>

<ul>
<li>UI bugs can be high impact even without errors</li>
<li>CSS-based layouts have scaling limits</li>
<li>Good OSS fixes are:</li>
<li>- Minimal</li>
<li>- Backward-compatible</li>
<li>- Easy to review</li>
<li>CI failures are part of the process — not a failure of the solution</li>
<li>Clear communication with maintainers matters as much as code
__________________________________________________________________</li>
</ul>

<p><strong>🙌 Why This Matters</strong></p>

<p>This contribution wasn’t about adding features — it was about *<em>making existing functionality reliable at scale.<br>
*</em><br>
That’s the kind of work that quietly improves developer experience and user trust.</p>

<ul>
<li>If you’re thinking about contributing to open source:</li>
<li>Don’t wait for “perfect” issues</li>
<li>Pick a real bug</li>
<li>Fix it cleanly</li>
<li>Learn from the review process</li>
</ul>

<p>That’s where the real growth happens.</p>

