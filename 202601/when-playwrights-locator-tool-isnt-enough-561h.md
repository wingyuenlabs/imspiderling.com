---
Title: When Playwright’s Locator Tool Isn’t Enough
Description: 
Author: Phil
Date: 2026-01-19T21:47:09.000Z
Robots: noindex,nofollow
Template: index
---
<p>Playwright’s built-in locator tool works fine most of the time, but once you begin dealing with real-world component libraries it can start to miss. It often suggests <code>getByText</code> or <code>getByRole</code> against elements that are not truly semantic controls, which makes those locators flaky or unusable in practice.</p>

<p>Checkboxes are a good example. In most modern apps, vanilla input <code>[type="checkbox"]</code> elements are rarely seen. What usually exists are custom components built with divs, spans, hidden inputs, aria attributes, and classes that represent state. Because of that, Playwright assertions like <code>toBeChecked</code> or <code>toBeDisabled</code> often cannot be trusted. The checkbox looks checked in the UI, but the underlying HTML does not expose the state in the way those helpers expect.</p>

<p>In these cases you need to own the locator yourself. The most reliable starting point is usually some stable text on the screen. Once you anchor to that text, you can walk up and down the DOM to find the real checkbox element.</p>

<p>Playwright has a small but very useful API for this: <code>locator('..')</code>. It moves one level up in the DOM, and you can chain it as many times as you need. It is much cleaner than <code>xpath=../..</code> and a lot easier to remember. From there, you can navigate back down into the exact node that represents the state you care about.</p>

<p>For example, a locator chain for a checkbox in our app might look like this:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight plaintext"><code>const checkboxLocator = page
  .locator('p.user-select-none', { hasText: 'School-Pupil Activity Bus' })
  .locator('..')
  .locator('..')
  .locator('.special-requirement-checkbox')
  .locator('input[type="checkbox"][aria-checked="true"][disabled="disabled"]')

await expect(checkboxLocator).toBeVisible()
</code></pre>

</div>



<p>Here the anchor is the visible label text. From there, the locator walks up to a shared container, then back down into a wrapper with a class you know is stable enough, and finally onto the actual input element. Because the checkbox is custom, you assert on <code>aria-checked</code> and <code>disabled</code> instead of relying on <code>toBeChecked</code> or <code>toBeDisabled</code>. A simple <code>toBeVisible</code> plus the right attributes ends up being more concrete than the higher level assertion API.</p>

<p>All of this is also to say why I am a little skeptical of AI testing tools that promise automatic locators and assertions. Real applications rarely use simple, semantic HTML controls. There is a lot of custom markup, hidden inputs, and framework specific structure that you have to understand and navigate. For now, a human who can anchor on the right text, walk the DOM, and assert on real attributes is still the most reliable way to write strong Playwright tests.</p>

