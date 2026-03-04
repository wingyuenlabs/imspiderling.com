---
Title: Preventing Fake Signups: Registration Fraud Guide
Description: 
Author: APIVerve
Date: 2026-03-04T21:34:03.000Z
Robots: noindex,nofollow
Template: index
---
<p>Signup forms are targets. Every exposed registration form attracts automated bots, fraudsters creating fake accounts, competitors scraping behind login walls, and users abusing free trials with throwaway credentials.</p>

<p>The challenge is distinguishing fake signups from real users without creating friction that drives legitimate customers away. Overly strict validation loses real users. Insufficient validation lets abuse through.</p>

<p>Effective signup protection uses layered validation—multiple signals that together identify fake signups while keeping the door open for legitimate users.</p>

<h2>
  
  
  Understanding Fake Signup Patterns
</h2>

<p>Fake signups come from different sources with different characteristics:</p>

<p><strong>Automated bots</strong> submit registration forms programmatically. They're typically fast (submitting in milliseconds), don't execute JavaScript, don't move a mouse, and often have outdated or unusual user agents. Bots create accounts for spam, credential stuffing, or scraping.</p>

<p><strong>Disposable email users</strong> sign up with temporary email addresses that expire within minutes or hours. Some have legitimate reasons (privacy concerns), but many are abusing free trials, evading bans, or creating multiple accounts for fraud.</p>

<p><strong>Fraudulent accounts</strong> are created manually or semi-automatically with intent to abuse the system. Credit card testing, referral fraud, fake reviews, and similar schemes require accounts that look real enough to avoid immediate detection.</p>

<p><strong>Invalid or mistyped addresses</strong> come from real users who accidentally enter wrong information. These aren't malicious, but they create accounts that can never receive email—activation links, password resets, or important notifications fail silently.</p>

<p>Each category requires different detection approaches.</p>

<h2>
  
  
  Layer 1: Email Validation
</h2>

<p>Email validation is the foundation of signup protection. A thorough email check catches multiple problem categories:</p>

<p><strong>Format validation</strong> catches obviously malformed addresses—missing @ symbols, invalid characters, incomplete domains. This is the minimum any form should do.</p>

<p><strong>Domain validation</strong> verifies the domain exists and has DNS records. Typos like <code>gmial.com</code> pass format validation but fail domain checks because the domain doesn't exist.</p>

<p><strong>MX record verification</strong> confirms the domain is configured to receive email. A domain might exist but have no mail servers configured.</p>

<p><strong>Mailbox verification</strong> (where possible) checks whether the mail server accepts messages for the specific address. Some servers confirm this; others refuse to answer.</p>

<p><strong>Deliverability assessment</strong> considers factors like domain reputation, known bounce patterns, and historical deliverability data.</p>

<p>Each layer catches problems the previous layers miss. A phased approach validates progressively deeper:</p>

<ol>
<li>Check format instantly (client-side, free)</li>
<li>Check domain and MX on blur or submit (fast API call)</li>
<li>Full deliverability check for high-value signups (deeper API call)</li>
</ol>

<h2>
  
  
  Layer 2: Disposable Email Detection
</h2>

<p>Disposable email services provide temporary addresses for users who want to sign up without providing permanent contact information. Services like Guerrilla Mail, 10 Minute Mail, Temp Mail, Mailinator, and hundreds of others offer these throwaway addresses.</p>

<p>Disposable email detection maintains databases of known disposable domains, pattern-based detection for newly created services, and heuristics for identifying throwaway behavior.</p>

<p>When a disposable email is detected, applications can:</p>

<p><strong>Block registration</strong> - Simplest approach. Require a permanent email address.</p>

<p><strong>Allow with restrictions</strong> - Let users sign up but limit functionality until they provide a permanent address.</p>

<p><strong>Flag for monitoring</strong> - Allow registration but mark the account for closer scrutiny of behavior.</p>

<p><strong>Require phone verification</strong> - Add a second verification channel for users with disposable emails.</p>

<p>The right approach depends on what you're protecting. A free trial with significant value justifies stricter controls than a newsletter signup.</p>

<p>A disposable email check returns clear signals for decision-making:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight javascript"><code><span class="kd">const</span> <span class="nx">response</span> <span class="o">=</span> <span class="k">await</span> <span class="nf">fetch</span><span class="p">(</span>
  <span class="dl">'</span><span class="s1">https://api.apiverve.com/v1/disposableemailchecker?email=user@tempmail.com</span><span class="dl">'</span><span class="p">,</span>
  <span class="p">{</span> <span class="na">headers</span><span class="p">:</span> <span class="p">{</span> <span class="dl">'</span><span class="s1">x-api-key</span><span class="dl">'</span><span class="p">:</span> <span class="dl">'</span><span class="s1">YOUR_API_KEY</span><span class="dl">'</span> <span class="p">}</span> <span class="p">}</span>
<span class="p">);</span>
<span class="kd">const</span> <span class="p">{</span> <span class="nx">data</span> <span class="p">}</span> <span class="o">=</span> <span class="k">await</span> <span class="nx">response</span><span class="p">.</span><span class="nf">json</span><span class="p">();</span>

<span class="k">if </span><span class="p">(</span><span class="nx">data</span><span class="p">.</span><span class="nx">isDisposable</span><span class="p">)</span> <span class="p">{</span>
  <span class="c1">// Block signup or require phone verification</span>
  <span class="k">return</span> <span class="p">{</span> <span class="na">error</span><span class="p">:</span> <span class="dl">'</span><span class="s1">Please use a permanent email address</span><span class="dl">'</span> <span class="p">};</span>
<span class="p">}</span>
</code></pre>

</div>



<p>When <code>isDisposable</code> is true, your application can block, restrict, or flag the signup accordingly.</p>

<h2>
  
  
  Layer 3: Bot Detection
</h2>

<p>Automated bots exhibit behavioral patterns that differ from human users:</p>

<p><strong>Speed</strong> - Humans take seconds to fill forms. Bots submit instantly or with mechanical regularity. A form submitted 200 milliseconds after page load isn't a human.</p>

<p><strong>Interaction patterns</strong> - Humans move the mouse, scroll, make mistakes and correct them. Bots often interact minimally or not at all.</p>

<p><strong>JavaScript execution</strong> - Many bots don't fully render pages or execute JavaScript. Requiring client-side computation can identify basic bots.</p>

<p><strong>Honeypot fields</strong> - Hidden form fields that humans never see. Bots programmatically fill all fields, triggering the trap.</p>

<p><strong>Browser fingerprinting</strong> - Automated tools have identifiable characteristics—missing browser APIs, inconsistent reported capabilities, known bot browser signatures.</p>

<p><strong>User agent analysis</strong> - Bot frameworks have identifiable user agents. Outdated browsers are suspicious. Missing user agents are suspicious.</p>

<p><strong>IP reputation</strong> - IPs from known bot networks, data centers without legitimate traffic, or with high abuse history indicate automated traffic.</p>

<p>Bot detection typically combines multiple signals into a risk score rather than relying on any single indicator. A single bot signal might be a false positive; multiple signals together indicate high confidence.</p>

<h2>
  
  
  Layer 4: Phone Validation
</h2>

<p>For high-value signups—paid plans, financial services, enterprise accounts—phone validation adds another verification layer:</p>

<p><strong>Format validation</strong> confirms the phone number structure is valid for its country. This catches typos and made-up numbers.</p>

<p><strong>Type detection</strong> identifies whether a number is mobile, landline, or VoIP. Mobile numbers are harder to generate in bulk than VoIP numbers.</p>

<p><strong>Carrier identification</strong> shows which network the number belongs to. Known VoIP carriers may indicate higher risk for certain use cases.</p>

<p><strong>SMS verification</strong> sends a code the user must enter, confirming they control the phone number.</p>

<p>Phone validation adds friction, so it's typically reserved for contexts where account value justifies it. A free blog doesn't need phone verification; a financial services platform does.</p>

<p>VoIP numbers deserve consideration. Many legitimate users have Google Voice, Skype, or similar numbers. Blocking all VoIP numbers creates false positives. But VoIP numbers are also easier to generate for fraud. The right approach depends on your fraud risk profile.</p>

<h2>
  
  
  Layer 5: Pattern Detection
</h2>

<p>Individual signals have limited value. Patterns combining multiple signals are more powerful:</p>

<p><strong>Velocity patterns</strong> - Multiple signups from the same IP address, same device fingerprint, or same behavioral patterns in short time periods indicate bulk account creation.</p>

<p><strong>Sequential data</strong> - Email addresses like <code>test1@example.com</code>, <code>test2@example.com</code>, <code>test3@example.com</code> suggest automated generation.</p>

<p><strong>Cross-signal combinations</strong> - A new domain, plus disposable email, plus instant form submission, plus known bot user agent creates high confidence of fraud even if each signal alone wouldn't trigger blocking.</p>

<p><strong>Deviation from baselines</strong> - Understanding normal signup patterns helps identify abnormal ones. A sudden spike in signups from a particular country or with particular characteristics warrants investigation.</p>

<p>Pattern detection typically operates on aggregated data—looking at signup volume over time, geographic distribution, email domain distribution, and other aggregate metrics that reveal coordinated activity.</p>

<h2>
  
  
  Balancing Security and Conversion
</h2>

<p>Every validation check creates potential friction. A signup flow that rejects too many people—including legitimate users caught by false positives—has poor conversion.</p>

<p>Principles for balancing security with conversion:</p>

<p><strong>Validate progressively</strong> - Start with light validation. Apply heavier validation only when earlier signals indicate risk. Most legitimate users see minimal friction.</p>

<p><strong>Provide helpful feedback</strong> - When validation fails, help users fix the problem. "Did you mean gmail.com?" is better than "Invalid email."</p>

<p><strong>Allow override with verification</strong> - If validation is uncertain, let users proceed with additional verification rather than blocking outright.</p>

<p><strong>Measure false positives</strong> - Track when legitimate users are blocked. If the same validation rule causes repeated support tickets, the rule may be too aggressive.</p>

<p><strong>Segment by value</strong> - Apply stricter validation to higher-value actions. Creating an account might have light validation; starting a trial might have more; completing a purchase might have the most.</p>

<p><strong>Fail gracefully</strong> - If validation services are unavailable, have fallbacks. Rejecting all signups because the validation API is down is worse than allowing some risky signups through temporarily.</p>

<h2>
  
  
  User Experience Considerations
</h2>

<p>How validation feels matters as much as what it catches:</p>

<p><strong>Speed</strong> - Validation shouldn't noticeably slow down the signup process. Async validation that completes before users finish typing feels instant.</p>

<p><strong>Clarity</strong> - Error messages should explain what's wrong and how to fix it. "Please use a permanent email address" not "Error 403."</p>

<p><strong>Positioning</strong> - Show validation feedback near the relevant field. Don't make users hunt for error messages.</p>

<p><strong>Timing</strong> - Validate on blur (when users leave a field) rather than on every keystroke. Let users finish typing before showing errors.</p>

<p><strong>Persistence</strong> - Remember that validation passed. Don't re-validate on every page load or make users re-enter verified information.</p>

<h2>
  
  
  Post-Signup Monitoring
</h2>

<p>Validation at signup is one checkpoint. Ongoing monitoring catches accounts that passed initial validation but exhibit suspicious behavior later:</p>

<p><strong>Activation monitoring</strong> - Accounts that sign up but never activate may indicate fake signups that couldn't pass email verification.</p>

<p><strong>Behavior analysis</strong> - Normal users have varied behavior. Accounts that immediately scrape data, spam other users, or perform single-purpose actions warrant investigation.</p>

<p><strong>Velocity monitoring</strong> - Unusual activity levels—creating hundreds of items, sending hundreds of messages—indicate automation even if signup looked legitimate.</p>

<p><strong>Payment fraud signals</strong> - For accounts with payment, watch for card testing patterns, chargeback rates, and suspicious purchase behavior.</p>

<p>Post-signup monitoring catches sophisticated fraud that initial validation misses. The best fake accounts look legitimate at signup and reveal themselves through behavior.</p>

<h2>
  
  
  Measuring Effectiveness
</h2>

<p>Track metrics to understand whether your validation is working:</p>

<p><strong>Fake signup rate</strong> - What percentage of signups turn out to be fake (never activate, exhibit bot behavior, get caught in post-signup monitoring)?</p>

<p><strong>Activation rate</strong> - What percentage of signups complete activation? Low rates may indicate invalid emails getting through.</p>

<p><strong>Conversion rate</strong> - What percentage of signups become paying customers or engaged users? If conversion drops after adding validation, false positives may be the cause.</p>

<p><strong>Support ticket volume</strong> - Are users complaining about being unable to sign up? These represent potential false positives.</p>

<p><strong>Fraud losses</strong> - If accounts are used for fraud, what are the actual losses? This is the cost you're trying to minimize.</p>

<p>Track these metrics before and after changes to understand impact. A validation change that reduces fake signups 50% but also reduces legitimate signups 10% may not be worthwhile.</p>

<h2>
  
  
  Implementation Approaches
</h2>

<p><strong>Progressive implementation</strong> - Start with basic validation (format checking, obvious bot detection). Add layers as you understand your fraud patterns. Don't over-engineer initially.</p>

<p><strong>API-based validation</strong> - Build vs. buy decisions favor APIs for most validation tasks. Email deliverability databases, disposable email lists, and bot detection signatures require ongoing maintenance. APIs handle this.</p>

<p><strong>Risk scoring</strong> - Rather than binary accept/reject, assign risk scores. Low risk proceeds normally. Medium risk triggers additional verification. High risk blocks.</p>

<p><strong>Fallback handling</strong> - External validation services will occasionally be unavailable. Decide in advance whether to fail open (allow signups, validate later) or fail closed (block signups during outages).</p>

<h2>
  
  
  Compliance and Privacy
</h2>

<p>User validation intersects with privacy and data protection:</p>

<p><strong>Data minimization</strong> - Only collect information you need. Phone validation might be overkill for low-risk applications.</p>

<p><strong>Transparency</strong> - If you're checking users against databases, privacy policies should disclose this.</p>

<p><strong>Data retention</strong> - Validation data (IP addresses, device fingerprints, etc.) may be personal data with retention requirements.</p>

<p><strong>Regional variations</strong> - Validation approaches that work in one jurisdiction may face restrictions in others. Phone verification, IP geolocation, and device fingerprinting all have varying privacy implications.</p>

<p>Review validation approaches with privacy and legal requirements in mind.</p>

<h2>
  
  
  Starting Simple
</h2>

<p>For most applications, a reasonable starting point:</p>

<ol>
<li>
<strong>Format validation</strong> - Client-side, instant</li>
<li>
<strong>Email deliverability check</strong> - Server-side, on submit</li>
<li>
<strong>Disposable email detection</strong> - Server-side, on submit</li>
<li>
<strong>Basic bot detection</strong> - Honeypot fields, submission timing</li>
<li>
<strong>Rate limiting</strong> - Limit signups per IP per time period</li>
</ol>

<p>This combination catches the majority of fake signups with minimal friction for legitimate users. Add more sophisticated measures as you understand your specific fraud patterns.</p>

<p>The goal isn't to eliminate all fake signups—that's impossible without also eliminating many real users. The goal is reducing fake signups to manageable levels while maintaining healthy conversion rates for legitimate traffic.</p>




<p>Validate emails thoroughly with the <a href="https://apiverve.com/marketplace/emailvalidator?utm_source=devto&amp;utm_medium=crosspost&amp;utm_campaign=how-we-stopped-94-percent-of-fake-signups" rel="noopener noreferrer">Email Validator API</a>. Detect disposable addresses with the <a href="https://apiverve.com/marketplace/disposableemailchecker?utm_source=devto&amp;utm_medium=crosspost&amp;utm_campaign=how-we-stopped-94-percent-of-fake-signups" rel="noopener noreferrer">Disposable Email Checker API</a>. Verify phone numbers with the <a href="https://apiverve.com/marketplace/phonenumbervalidator?utm_source=devto&amp;utm_medium=crosspost&amp;utm_campaign=how-we-stopped-94-percent-of-fake-signups" rel="noopener noreferrer">Phone Number Validator API</a>. Identify bots with the <a href="https://apiverve.com/marketplace/botdetector?utm_source=devto&amp;utm_medium=crosspost&amp;utm_campaign=how-we-stopped-94-percent-of-fake-signups" rel="noopener noreferrer">Bot Detector API</a>. Build signup forms that keep the fakes out.</p>




<p><em>Originally published at <a href="https://blog.apiverve.com/post/how-we-stopped-94-percent-of-fake-signups" rel="noopener noreferrer">APIVerve Blog</a></em></p>

