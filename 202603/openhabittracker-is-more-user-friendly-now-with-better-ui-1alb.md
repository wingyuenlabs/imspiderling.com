---
Title: OpenHabitTracker is more user friendly now with better UI
Description: 
Author: Urban
Date: 2026-03-21T21:37:15.000Z
Robots: noindex,nofollow
Template: index
---
<p>OpenHabitTracker is a free, open source app for taking Markdown notes, planning tasks, and tracking habits. No account required, no ads, no subscription - all your data stays on your device.</p>

<p>It runs on Windows, Linux, Android, iOS, macOS, and in the browser as a PWA.</p>




<h3>
  
  
  A cleaner, more configurable interface
</h3>

<p>The app now lets you hide the fields you don't use. Don't care about priorities? Turn them off. Don't use categories? Hide them. The settings are simple toggles and they make a real difference to how much visual noise you're dealing with day to day.</p>

<p>Notes, tasks, and habits each have their own background color, so you can tell at a glance what type of item you're looking at without reading the label.</p>

<p>There's also a filter to hide completed tasks, which keeps the list focused on what's still pending.</p>




<h3>
  
  
  Weekly summary
</h3>

<p>Tasks and habits both have an optional stats panel showing your week at a glance - how many you completed, how much time you spent, and whether you're keeping up with your habits or falling behind.</p>




<h3>
  
  
  Keyboard navigation
</h3>

<p>The menu sidebar follows the ARIA menu pattern - arrow keys move between items, no mouse needed. The habit calendar supports the full ARIA grid pattern: arrow keys to move between days, Home and End for the start and end of a week, Page Up and Page Down to switch months.</p>

<p>When you open a sidebar, focus moves into it automatically. When you close it, focus returns to where you were. It sounds like a small thing but it's the difference between an app that feels under control and one that constantly loses your place.</p>




<h3>
  
  
  Accessibility
</h3>

<p>Every interactive element has a proper ARIA label. The layout uses semantic HTML landmarks - <code>&lt;nav&gt;</code> and <code>&lt;main&gt;</code> - so screen readers can navigate the page structure without hunting through the content. Decorative icons are hidden from assistive technology. Toggle buttons correctly reflect their state with <code>aria-expanded</code>. The active page in the navigation is marked with <code>aria-current</code>. The app is fully usable without a mouse.</p>




<h3>
  
  
  20 languages, 26 themes
</h3>

<p>Besides English, German, Spanish, and Slovenian, the app is now also available in French, Portuguese, Italian, Japanese, Chinese, Korean, Dutch, Danish, Norwegian, Swedish, Finnish, Polish, Czech, Slovak, Croatian, and Serbian.</p>

<p>It ships with 26 themes in both dark and light modes.</p>




<p>Data can be exported and imported as JSON, YAML, TSV, or Markdown. Google Keep notes can be imported from a Google Takeout ZIP.</p>

<p>For the self-hosting community, there's a Docker image that runs a Blazor Server instance. Besides accessing it in the browser, any native version of the app - Windows, Linux, Android, iOS, macOS - can log in to your instance and sync, so your data follows you across devices without going through anyone else's servers.</p>

<p>Web: <a href="https://openhabittracker.net" rel="noopener noreferrer">openhabittracker.net</a><br>
PWA: <a href="https://pwa.openhabittracker.net" rel="noopener noreferrer">pwa.openhabittracker.net</a><br>
Source: <a href="https://github.com/Jinjinov/OpenHabitTracker" rel="noopener noreferrer">github.com/Jinjinov/OpenHabitTracker</a></p>

