---
Title: Adding a new feature to vscode-pets project
Description: 
Author: OleksandraKordonets
Date: 2025-10-17T21:07:41.000Z
Robots: noindex,nofollow
Template: index
---
<p>For my Hacktoberfest contribution, I worked on making the fetch ball in <a href="https://github.com/tonybaloney/vscode-pets" rel="noopener noreferrer">vscode-pets</a> configurable in a more user-friendly way. Originally, the ball color was hard-coded to a bright green, so every time you threw a ball it looked the same. <a href="https://github.com/tonybaloney/vscode-pets/issues/813" rel="noopener noreferrer">The issue</a> I picked up aimed to let users choose a color without interrupting their workflow, while keeping consistent with the project’s existing UX conventions.</p>

<p>I started by forking the repository, creating a small feature branch, and reading through the code to find where the ball was created and rendered. The key files I touched were the module that draws and manages the ball, and the extension’s <code>package.json</code>, where user settings are declared. My first prototype added a QuickPick prompt that appeared on each throw so the user could pick a color. That worked technically, but after repo owner's feedback in the comments under <a href="https://github.com/tonybaloney/vscode-pets/pull/815" rel="noopener noreferrer">my PR</a>, it became clear that prompting every time was tedious. The reviewer suggested mirroring the <code>petSize</code> behavior: choose a color once in Settings and use that value automatically for all throws. That suggestion made sense and matched the extension’s overall user experience.</p>

<p>Based on that feedback, I changed direction and added a new configuration property, <code>vscode-pets.ballColor</code>, to <code>package.json</code>, using <code>"format": "color"</code> so VS Code would display a color picker in the Settings UI. In the extension host, I introduced a helper function <code>getConfiguredBallColor()</code> that reads the setting and falls back to the default green color. I replaced the per-throw QuickPick in <code>throwBall()</code> with logic that posts the configured color to the webview and then triggers the throw action. On the webview side, I modified the drawing code so it no longer used a hard-coded value. Instead, the ball now uses a module-level <code>currentBallColor</code> variable and exposes <code>setNextBallColor(color)</code> so the host can update it by message. I also made sure the webview picks up the configured color on startup and responds to set-next-ball-color messages so that any changes in Settings take effect immediately.</p>

<p>This development flow taught me a few useful lessons. One of them is the importance of following a project’s existing UX patterns, they exist for a reason, and matching them usually leads to a better, more accepted change. Another takeaway is that small, incremental changes are easier to review and test. I kept this feature focused on one improvement rather than bundling it with extra UI or theme-contrast logic.</p>

<p>Working on this feature gave me a better understanding of how VS Code extensions manage configuration and communicate between the host and webview. It was a good balance of frontend and backend work, and seeing the color picker appear in Settings felt rewarding. If I were to extend it in the future, I’d like to explore making the ball color automatically adapt to different themes, but for now I’m happy with how clean and seamless the final result feels.</p>

