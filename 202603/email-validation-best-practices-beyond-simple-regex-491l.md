---
Title: Email Validation Best Practices: Beyond Simple Regex
Description: 
Author: APIVerve
Date: 2026-03-04T21:36:31.000Z
Robots: noindex,nofollow
Template: index
---
<p>Email validation is deceptively complex. What seems like a simple task—checking whether an email address is valid—actually involves multiple layers of verification, edge cases that break simple approaches, and tradeoffs between strictness and user experience.</p>

<p>Getting email validation wrong has real costs. Reject valid addresses, and you lose potential customers who give up in frustration. Accept invalid addresses, and you waste resources on bounced emails, damage your sender reputation, and miss important communications with users.</p>

<p>This guide covers what proper email validation involves, common mistakes to avoid, and how to balance validation strictness with user experience.</p>

<h2>
  
  
  What Makes an Email Address Valid
</h2>

<p>The email address specification (defined in RFC 5321 and RFC 5322) is more permissive than most people realize. Valid email addresses can include:</p>

<p><strong>Dots in various positions</strong> - Both <code>first.last@example.com</code> and <code>f.i.r.s.t@example.com</code> are valid. Dots can appear almost anywhere in the local part (the part before the @).</p>

<p><strong>Plus signs for sub-addressing</strong> - <code>user+newsletter@gmail.com</code> is valid and widely used. Gmail and other providers use the plus sign to create aliases that deliver to the main address. Many users employ this for filtering and tracking which services share their email.</p>

<p><strong>Apostrophes and other special characters</strong> - <code>o'brien@company.ie</code> is perfectly valid. Irish names, French names, and others frequently include apostrophes. Hyphens, underscores, and various other characters are also permitted.</p>

<p><strong>Numeric local parts</strong> - <code>12345@example.com</code> is valid. Some organizations use numeric identifiers as email addresses.</p>

<p><strong>Long top-level domains</strong> - Modern TLDs go far beyond .com and .org. Addresses like <code>user@company.photography</code> or <code>contact@brand.engineering</code> are valid and increasingly common.</p>

<p><strong>International characters</strong> - The email specification now supports internationalized email addresses with non-ASCII characters. <code>用户@例え.jp</code> is a valid email address format.</p>

<p><strong>IP address domains</strong> - Technically, <code>user@[192.168.1.1]</code> is valid, though rarely used in practice.</p>

<p><strong>Quoted local parts</strong> - <code>"john doe"@example.com</code> with spaces inside quotes is valid per the specification.</p>

<p>Every restriction you add to email validation potentially rejects someone's real, working email address.</p>

<h2>
  
  
  Why Simple Regex Fails
</h2>

<p>The internet is full of email validation regex patterns, ranging from simple to absurdly complex. Most of them cause problems.</p>

<p><strong>Simple patterns reject valid addresses.</strong> A pattern that only allows alphanumeric characters, dots, and @ symbols will reject plus signs, apostrophes, and other valid characters. Users with these addresses can't sign up.</p>

<p><strong>Complex patterns are unmaintainable.</strong> The regex needed to fully match the email specification is hundreds of characters long and virtually impossible to debug. Even then, it only validates format—not whether the address actually works.</p>

<p><strong>All regex only validates format.</strong> Whether simple or complex, regex can only answer "does this string match a pattern?" It cannot answer "does this email address receive mail?" which is usually the question that actually matters.</p>

<p>Format validation catches obviously malformed input—missing @ symbols, empty strings, addresses that are clearly not emails. For anything beyond that, format validation alone is insufficient.</p>

<h2>
  
  
  Deliverability vs. Format Validity
</h2>

<p>Understanding the difference between format validity and deliverability is crucial for email validation.</p>

<p><strong>Format validity</strong> asks: Does this string conform to the email address specification? This is what regex checks.</p>

<p><strong>Deliverability</strong> asks: If I send an email to this address, will it arrive? This is what usually matters.</p>

<p>An email address can be perfectly formatted and completely undeliverable:</p>

<p><strong>Typos in the domain</strong> - <code>user@gmial.com</code> passes format validation. Gmail doesn't own gmial.com. The email will never arrive.</p>

<p><strong>Non-existent domains</strong> - <code>user@thisdoesnotexist12345.com</code> looks like an email address. But if the domain doesn't exist or has no mail servers configured, no email can be delivered.</p>

<p><strong>Non-existent mailboxes</strong> - <code>randomstring8472@gmail.com</code> has correct format and a valid domain. But if no one has registered that Gmail account, emails bounce.</p>

<p><strong>Disabled or full mailboxes</strong> - The address once worked but the account was closed, or the mailbox is full and rejecting new messages.</p>

<p><strong>Spam traps</strong> - Some email addresses exist specifically to catch spammers. Sending to them damages your reputation.</p>

<p>Format validation catches maybe 5% of email problems. Deliverability validation catches the rest.</p>

<p>A proper email validation API returns comprehensive results:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight javascript"><code><span class="kd">const</span> <span class="nx">response</span> <span class="o">=</span> <span class="k">await</span> <span class="nf">fetch</span><span class="p">(</span>
  <span class="dl">'</span><span class="s1">https://api.apiverve.com/v1/emailvalidator?email=user@example.com</span><span class="dl">'</span><span class="p">,</span>
  <span class="p">{</span> <span class="na">headers</span><span class="p">:</span> <span class="p">{</span> <span class="dl">'</span><span class="s1">x-api-key</span><span class="dl">'</span><span class="p">:</span> <span class="dl">'</span><span class="s1">YOUR_API_KEY</span><span class="dl">'</span> <span class="p">}</span> <span class="p">}</span>
<span class="p">);</span>
<span class="kd">const</span> <span class="p">{</span> <span class="nx">data</span> <span class="p">}</span> <span class="o">=</span> <span class="k">await</span> <span class="nx">response</span><span class="p">.</span><span class="nf">json</span><span class="p">();</span>

<span class="c1">// Check deliverability, not just format</span>
<span class="k">if </span><span class="p">(</span><span class="nx">data</span><span class="p">.</span><span class="nx">isValid</span> <span class="o">&amp;&amp;</span> <span class="nx">data</span><span class="p">.</span><span class="nx">isMxValid</span> <span class="o">&amp;&amp;</span> <span class="nx">data</span><span class="p">.</span><span class="nx">isSmtpValid</span><span class="p">)</span> <span class="p">{</span>
  <span class="c1">// Email is likely deliverable</span>
<span class="p">}</span>

<span class="c1">// data also includes:</span>
<span class="c1">// - isFreeEmail: true for Gmail, Yahoo, etc.</span>
<span class="c1">// - isCompanyEmail: true for business domains</span>
<span class="c1">// - hasTypo: true if domain looks like a typo (gmial.com)</span>
</code></pre>

</div>



<p>This tells you not just whether the format is correct, but whether the domain has mail servers, whether those servers accept connections, and whether it's a business or free email provider.</p>

<h2>
  
  
  Checking Email Deliverability
</h2>

<p>Real email validation goes beyond format checking to verify deliverability through multiple steps:</p>

<p><strong>DNS lookup</strong> - Does the domain exist? Every email domain must have DNS records. If the domain doesn't resolve, no email can be delivered.</p>

<p><strong>MX record check</strong> - Does the domain have mail servers configured? The MX (Mail Exchanger) records specify which servers handle email for a domain. No MX records usually means no email capability.</p>

<p><strong>SMTP verification</strong> - Can you connect to the mail server? Does it accept mail for this address? Some mail servers will tell you whether a specific mailbox exists. Others refuse to answer (to prevent address enumeration attacks).</p>

<p><strong>Reputation assessment</strong> - Is this domain associated with spam or fraud? Are deliverability rates historically low?</p>

<p>These checks require network requests and can't be done with client-side regex. They typically take 1-3 seconds per address, which is acceptable for form submission but too slow for real-time validation on every keystroke.</p>

<h2>
  
  
  Disposable Email Detection
</h2>

<p>Disposable email services provide temporary addresses that expire after minutes or hours. Popular services include Guerrilla Mail, 10 Minute Mail, Temp Mail, Mailinator, and hundreds of others.</p>

<p>People use disposable emails for various reasons:</p>

<p><strong>Avoiding spam</strong> - Signing up for services that might sell their email or send unwanted messages.</p>

<p><strong>One-time access</strong> - Downloading content or accessing gated material without providing a real address.</p>

<p><strong>Testing</strong> - Developers testing email flows without cluttering their real inbox.</p>

<p><strong>Abuse</strong> - Creating multiple accounts to exploit free trials, accumulate referral bonuses, evade bans, or engage in fraud.</p>

<p>Whether to block disposable emails depends on your use case:</p>

<p><strong>Trial signups</strong> - If you're offering a free trial and want a relationship with the user, blocking disposables makes sense. Users who won't provide a real email are unlikely to convert.</p>

<p><strong>Newsletter signups</strong> - Blocking might cost you some legitimate subscribers who are just cautious about spam.</p>

<p><strong>Account creation</strong> - For platforms where account value builds over time, requiring a permanent email address is reasonable.</p>

<p><strong>One-time downloads</strong> - Blocking disposables might be unnecessary friction. The transaction is complete; you may not need ongoing communication.</p>

<p>Disposable email detection requires maintaining an updated database of disposable domains. New services appear constantly, so static lists quickly become outdated. API-based detection stays current.</p>

<h2>
  
  
  Handling Common Typos
</h2>

<p>Beyond validation, detecting and suggesting corrections for common typos improves user experience and data quality.</p>

<p>The most frequently mistyped email domains include:</p>

<ul>
<li>
<code>gmial.com</code> instead of <code>gmail.com</code>
</li>
<li>
<code>gmal.com</code> instead of <code>gmail.com</code>
</li>
<li>
<code>gnail.com</code> instead of <code>gmail.com</code>
</li>
<li>
<code>hotmial.com</code> instead of <code>hotmail.com</code>
</li>
<li>
<code>yaho.com</code> instead of <code>yahoo.com</code>
</li>
<li>
<code>outlok.com</code> instead of <code>outlook.com</code>
</li>
<li>
<code>.con</code> instead of <code>.com</code>
</li>
</ul>

<p>When you detect a likely typo, showing "Did you mean gmail.com?" converts a future bounce into a successful signup. The user appreciates the help, and you get a working email address.</p>

<p>Typo detection should suggest, not auto-correct. Users might have legitimate addresses at unusual domains. Let them confirm the correction rather than silently changing what they typed.</p>

<h2>
  
  
  Validation Timing and User Experience
</h2>

<p>When and how you validate affects user experience as much as what you validate.</p>

<p><strong>Don't validate on every keystroke.</strong> Red error messages appearing while the user is still typing are distracting and frustrating. Wait until they've finished—on blur (when they click away from the field) or on form submission.</p>

<p><strong>Validate asynchronously when possible.</strong> Deep validation takes time. Start the validation when the user finishes typing the email, so results are ready by the time they submit the form.</p>

<p><strong>Provide specific, actionable feedback.</strong> "Invalid email" tells users nothing. "Please include an @ symbol" identifies the problem. "Did you mean gmail.com?" solves it.</p>

<p><strong>Distinguish between definite problems and warnings.</strong> A missing @ symbol is definitely wrong. A new domain you can't verify might be fine—warn but allow submission.</p>

<p><strong>Remember that you might be wrong.</strong> If your validation rejects <code>o'brien@company.ie</code>, you're wrong, not the user. Build in escape hatches for edge cases.</p>

<h2>
  
  
  Different Validation for Different Contexts
</h2>

<p>Not all email collection points need the same validation rigor.</p>

<p><strong>Account signup</strong> requires thorough validation. You need to send activation emails, password resets, and account notifications. An invalid email breaks the entire user experience. Full deliverability checking is justified.</p>

<p><strong>Checkout and transactions</strong> also warrant careful validation. Order confirmations, shipping notifications, and receipts need to reach the customer. The transaction's value justifies the validation overhead.</p>

<p><strong>Newsletter subscription</strong> can use lighter validation. A bad email just means one undelivered newsletter. The cost of false rejection (losing a subscriber) may exceed the cost of false acceptance (one bounce).</p>

<p><strong>Contact forms</strong> often need only basic validation. You're going to read and respond manually anyway. As long as the address looks plausible, you can handle problems individually.</p>

<p><strong>Profile updates</strong> should validate the new address thoroughly before replacing the old one. Consider requiring confirmation of both addresses—sending a verification to the new one and a notification to the old one.</p>

<h2>
  
  
  Role-Based and Group Addresses
</h2>

<p>Some email addresses are technically valid but serve different purposes than individual mailboxes:</p>

<p><strong>Role addresses</strong> like <code>info@</code>, <code>support@</code>, <code>admin@</code>, <code>sales@</code>, <code>webmaster@</code> go to teams or rotate among staff. They're valid for business communication but may not be ideal for individual user accounts.</p>

<p><strong>Group addresses</strong> deliver to multiple recipients. Perfectly valid, but again not individual accounts.</p>

<p><strong>Auto-responders</strong> reply automatically to incoming messages. Sending transactional emails to these addresses generates noise.</p>

<p>Whether these matter depends on context. For a B2B service, role addresses are normal and expected. For a consumer app expecting individual users, they might warrant gentle discouragement (not hard blocking).</p>

<h2>
  
  
  Free vs. Business Email Providers
</h2>

<p>Email validation can distinguish between free providers (Gmail, Yahoo, Outlook, etc.) and business domains (company-specific addresses).</p>

<p><strong>Free email addresses</strong> are perfectly legitimate for consumers. They're also slightly higher risk for fraud, since creating new accounts is easy.</p>

<p><strong>Business email addresses</strong> suggest professional context and are slightly harder to create in bulk. For B2B applications, business emails may indicate more serious prospects.</p>

<p>This distinction is useful for segmentation and prioritization, not for blocking. Plenty of legitimate business users use personal email addresses, especially for initial inquiries or small businesses.</p>

<h2>
  
  
  Handling Validation Failures Gracefully
</h2>

<p>When validation fails, how you communicate matters.</p>

<p><strong>Be specific about the problem.</strong> "We couldn't verify this email address exists. Please check for typos." is better than "Invalid email."</p>

<p><strong>Offer suggestions when possible.</strong> "Did you mean @gmail.com?" helps users fix the actual problem.</p>

<p><strong>Allow override with acknowledgment.</strong> "We couldn't verify this address. If you're sure it's correct, click Continue." respects user agency while flagging potential issues.</p>

<p><strong>Don't lecture or blame.</strong> "You entered an invalid email" sounds accusatory. "Please check your email address" is neutral.</p>

<p><strong>Make errors visible but not alarming.</strong> Red text is traditional for errors, but a subtle color change is often sufficient. Save the bold red for critical problems.</p>

<h2>
  
  
  The Business Case for Good Validation
</h2>

<p>Poor email validation has measurable costs:</p>

<p><strong>Bounced emails</strong> damage sender reputation. High bounce rates lead to deliverability problems where even valid emails land in spam folders.</p>

<p><strong>Invalid data</strong> pollutes your database. Fake addresses skew metrics and waste resources on campaigns that can't reach anyone.</p>

<p><strong>Lost customers</strong> leave when signup fails. A user whose valid email gets rejected will often abandon rather than contact support.</p>

<p><strong>Support burden</strong> increases when users can't sign up or receive emails. These tickets could be prevented by better validation.</p>

<p>Good validation pays for itself in cleaner data, better deliverability, fewer support tickets, and more successful user signups.</p>

<h2>
  
  
  Putting It All Together
</h2>

<p>Effective email validation is layered:</p>

<p><strong>First layer: Basic format check.</strong> Is there an @ symbol? Is there content on both sides? Does the domain have a dot? This catches obvious mistakes instantly with no external calls.</p>

<p><strong>Second layer: Deliverability verification.</strong> Does the domain exist? Does it have mail servers? Can you reach them? This catches typos, fake domains, and non-functional addresses.</p>

<p><strong>Third layer: Quality assessment.</strong> Is it disposable? Is it a role address? Is it from a free provider? This provides context for how to treat the address.</p>

<p><strong>Fourth layer: User experience.</strong> Can you suggest corrections? Can you explain problems clearly? Can you allow edge cases while flagging concerns?</p>

<p>Each layer builds on the previous. Skip a layer, and you miss a category of problems. Implement all layers thoughtfully, and you get clean data without frustrating legitimate users.</p>




<p>Validate email addresses comprehensively with the <a href="https://apiverve.com/marketplace/emailvalidator?utm_source=devto&amp;utm_medium=crosspost&amp;utm_campaign=why-your-email-validation-rejects-real-customers" rel="noopener noreferrer">Email Validator API</a>. Detect disposable addresses with the <a href="https://apiverve.com/marketplace/disposableemailchecker?utm_source=devto&amp;utm_medium=crosspost&amp;utm_campaign=why-your-email-validation-rejects-real-customers" rel="noopener noreferrer">Disposable Email Checker API</a>. Build signup flows that capture real email addresses without rejecting real customers.</p>




<p><em>Originally published at <a href="https://blog.apiverve.com/post/why-your-email-validation-rejects-real-customers" rel="noopener noreferrer">APIVerve Blog</a></em></p>

