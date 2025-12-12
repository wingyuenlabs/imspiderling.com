---
Title: Write Powerful System Prompts | Prompt Engineering - Build AI Platforms From Scratch #3
Description: 
Author: Nick Goldstein
Date: 2025-12-12T21:30:21.000Z
Robots: noindex,nofollow
Template: index
---
<h1>
  
  
  Prompt Engineering
</h1>

<p>Learn effective prompt engineering techniques for building AI platforms.</p>

<p><strong>📺 <a href="https://nicholasmgoldstein.com/course/build-ai-platforms/module/3" rel="noopener noreferrer">View this module with video &amp; slides</a></strong></p>




<h2>
  
  
  What is Prompt Engineering?
</h2>

<p><em>And how does it make or break AI platforms?</em></p>

<p>Prompt engineering is the discipline of designing instructions that control AI behavior. Your prompt is the blueprint for how an AI interprets requests and generates responses.</p>

<p>In AI platforms, prompts aren't just suggestions, they're the entire control mechanism.</p>

<ul>
<li>
<strong>Bad prompts</strong> = unpredictable outputs, hallucinations, parsing failures, user frustration.</li>
<li>
<strong>Good prompts</strong> = consistent behavior, reliable outputs, maintainable systems, scalable platforms.</li>
</ul>

<p>The difference between a working AI product and a broken one often comes down to prompt quality:</p>

<ul>
<li>Vague prompts → AI invents its own interpretations.</li>
<li>Contradictory prompts → AI picks randomly between conflicting instructions.</li>
<li>Well-structured prompts → AI follows rules consistently.</li>
</ul>

<p>Prompts are architectural decisions that determine whether your AI layer functions reliably or fails under edge cases.</p>

<h2>
  
  
  An Example from My Project
</h2>

<p><em>Short example from Emstrata</em></p>

<p>The World Builder is a foundational prompt in Emstrata that allows users to create custom narrative worlds based on their inputs, complete with character, locations, items, and a reality that maintains.</p>

<p>My World Builder system prompt in Emstrata takes in a number of input elements (pieces of data that I clearly define within the system prompt):</p>

<ul>
<li><code>user-msg</code></li>
<li><code>title</code></li>
<li><code>prefs</code></li>
<li><code>genre</code></li>
<li><code>arc</code></li>
</ul>

<p>From these input elements, the World Builder outputs:</p>

<ul>
<li><code>prose("")</code></li>
<li><code>basis("")</code></li>
<li><code>char("name", "desc", "state")</code></li>
<li><code>Item("name", "desc", "state")</code></li>
<li><code>location("name", "desc", "state")</code></li>
</ul>

<h2>
  
  
  How to Structure a System Prompt
</h2>

<p><em>Achieving maximum effectiveness with your prompts</em></p>

<p>Split your prompt into modules. Each one handles a distinct concern:</p>

<ol>
<li>
<strong>Core Identity</strong> - what is this AI (a chatbot, research agent, etc.) and what does it do</li>
<li>
<strong>Platform Specifics</strong> - context about where/how it operates, if that relates to the output</li>
<li>
<strong>Understanding Role</strong> - scope, responsibilities, boundaries</li>
<li>
<strong>Dissecting Requests</strong> - how to parse incoming data</li>
<li>
<strong>Response Expectations</strong> - exact output format with function calls</li>
<li>
<strong>Quality Standards</strong> - non-negotiable benchmarks for output</li>
</ol>

<h2>
  
  
  Request Structure
</h2>

<p><em>What I suggest</em></p>

<p>Requests on projects should come in key-value pairs that are easily understandable and clearly defined.</p>

<p>An example given is <code>user-input: "This is what the user said!"</code></p>

<p>Another example is <code>convo-summary: "This is a breakdown of what was said previously"</code></p>

<p>A number of these key-value pairs can be listed, and the input can be split up as much as needed, as long as it remains logical and the AI will understand the intent after defining the input elements.</p>

<h2>
  
  
  Response Structure
</h2>

<p><em>What I suggest</em></p>

<p><strong>Structured Output:</strong> Returning predefined functions instead of unstructured output aids in parsing, logical coherence, AI understanding, and prevents anomalous responses or AI hallucinations.</p>

<p><strong>Argument Definition:</strong> It's crucial to define arguments beforehand and ensure the AI maintains the correct order and argument types (e.g., string, number, likelihood/1000).</p>

<p>Examples of predefined functions:</p>

<ul>
<li>
<code>try("Outcome 1", "Outcome 2", 100/1000)</code> - This means "10% chance outcome 1 is successful, 90% chance outcome 2 happens."</li>
<li>
<code>attack(damage, 20)</code> - This means "the attack does 20 points of damage." Note that "damage" is a keyword, not a string, in this context.</li>
<li>
<code>speak("this is some example dialogue")</code> - This represents "Example dialogue coming from the AI."</li>
</ul>

<h2>
  
  
  What is a Function
</h2>

<p><em>For non-coders</em></p>

<p>A function is a preset command that can hold arguments which determine specific outcomes.</p>

<p><strong>Anatomy:</strong> <code>functionName(argument1, argument2, argument3)</code></p>

<ul>
<li>
<strong>Function Name:</strong> The command type (speak, create, attack, move)</li>
<li>
<strong>Parentheses ()</strong> - Container holding the arguments</li>
<li>
<strong>Arguments</strong> - The values that determine what actually happens, separated by commas, in a specific order</li>
</ul>

<p><strong>Example:</strong> <code>speak("Hello there", cheerful)</code></p>

<ul>
<li>
<code>speak</code> = command type</li>
<li>
<code>"Hello there"</code> = what gets said</li>
<li>
<code>cheerful</code> = how it's said</li>
</ul>

<h2>
  
  
  Modularity
</h2>

<p><em>The benefits of breaking system prompts down</em></p>

<p>Modules are independently updatable. Fix one without rewriting everything.</p>

<p>Think of it like code architecture. There's a separation of concerns that makes everything cleaner and more maintainable.</p>

<p>Modular architecture gives you:</p>

<ul>
<li>
<strong>Independent updates</strong> - fix one module without touching others</li>
<li>
<strong>Reusability</strong> - drop the same module into different prompts</li>
<li>
<strong>Clarity</strong> - each module has one job, easy to understand</li>
<li>
<strong>Collaboration</strong> - team members can work on different modules simultaneously</li>
<li>
<strong>Debugging</strong> - isolate issues to specific modules instead of hunting through a massive block of text</li>
</ul>

<h2>
  
  
  Inputs and Outputs
</h2>

<p><em>How these make or break a system prompt</em></p>

<ul>
<li>Input only what you absolutely need to get an expected output. Additional info may deprioritize more important inputs.</li>
<li>Setup a prioritization hierarchy. i.e. <code>user-input &gt; convo-history &gt; saved-prefs</code>
</li>
<li>Similarly, output only the necessities as well. The more elements being generated, the higher the chance of confusion or deprioritization.</li>
<li>Take your time cycling through building =&gt; reducing =&gt; building =&gt; reducing. You will likely see that you were overthinking what you need in both parts.</li>
</ul>

<h2>
  
  
  Defining Rules
</h2>

<p><em>Ensuring rules are stressed and enforced</em></p>

<ul>
<li>Critical rules need emphasis. Use ALL CAPS, repetition, strategic placement at the beginning or end of modules.</li>
<li>Eliminate contradictory instructions. If you give conflicting instructions (e.g., "be concise" in one place and "provide extensive detail" in another), the AI will pick one arbitrarily.</li>
<li>Be explicit about constraints. What the AI cannot do is as important as what it can do.</li>
<li>Define argument types strictly and repeat them. Examples: string <code>"text in quotes"</code>, num without quotes, and likelihood <code>num/1000</code>.</li>
</ul>

<p><strong>Example of bad rule:</strong> "Respond appropriately"</p>

<p><strong>Example of good rule:</strong> "You must respond using only the functions defined in Response Expectations Module. Do not invent new functions or arguments."</p>

<h2>
  
  
  Preventing Hallucinations
</h2>

<p><em>Techniques to keep AIs on track</em></p>

<ul>
<li>Restricting AIs to a specific formatted response is one of the best ways to ensure flawed responses don't happen i.e. <code>response("this is an example of a response", "This is the second argument to this function")</code>
</li>
<li>If you choose this method, reiterate to the AI that they cannot make up their own functions or arguments, they will likely try.</li>
<li>Be hyper-specific about requirements, identify and eliminate contradictory prompting that may be confusing the AI.</li>
<li>All caps can be an effective method to stress an aspect to the AI, if necessary.</li>
</ul>

<h2>
  
  
  Trial &amp; Error
</h2>

<p><em>Iteration is absolutely necessary to achieve good results</em></p>

<ul>
<li>All LLMs will react differently to certain types of prompting. Some will follow rules and understand certain instructions better than others.</li>
<li>Testing, iterating, and saving updates will be essential to getting good results from your AIs.</li>
<li>Consider the size of the model in proportion to the size of the task. A massive response with complex logic should be handled by a bigger (probably more expensive) model.</li>
</ul>

<h2>
  
  
  Major Takeaways
</h2>

<p><em>What to remember</em></p>

<ul>
<li>Modular architecture beats monolithic blocks every time. Easier to debug, reuse, and collaborate on.</li>
<li>Define clear input/output structures using key-value pairs and preset functions. Ambiguity kills AI performance.</li>
<li>Minimize both inputs and outputs to absolute essentials. More complexity = more confusion = worse results.</li>
<li>Formatted responses (function calls with strict argument types) are your best defense against hallucinations.</li>
<li>Test, reduce, test again. Your first draft will always need refinement.</li>
<li>Be hyper-specific. Eliminate contradictions. Stress critical rules with ALL CAPS if necessary.</li>
</ul>

<h2>
  
  
  System Prompt Generator Tool
</h2>

<p><em>Easiest way to get started</em></p>

<p>Available now on <a href="https://nicholasmgoldstein.com/system-prompt-generator" rel="noopener noreferrer">https://nicholasmgoldstein.com/system-prompt-generator</a></p>

<ul>
<li>Prebuilt modular system prompt skeleton that can give you a basis to build upon</li>
<li>Feel free to copy/paste this into Notion, Google Docs, Microsoft Word, or whatever you plan to use and add your own modules/rulesets and logic</li>
</ul>

<h2>
  
  
  Exemplary System Prompt
</h2>

<p><em>A complete example demonstrating all the concepts from this module</em></p>

<h3>
  
  
  Core Identity Module
</h3>

<p><em>The following statement defines your fundamental identity and primary purpose. All your actions, responses, and behaviors must align with this core identity.</em></p>

<p>You are SupportBot, a customer service assistant for ShopEase, an online retail platform specializing in electronics and home goods.</p>

<h3>
  
  
  Platform Specifics Module
</h3>

<p><em>The following information describes the platform, environment, or system you operate within. Consider these details when formulating responses and making decisions.</em></p>

<p>You operate within a live chat widget on the ShopEase website. Users can message you 24/7. The platform has access to order databases, tracking information, and product catalogs. Sessions remain active for 15 minutes of inactivity before auto-closing.</p>

<h3>
  
  
  Quality Standards
</h3>

<p><em>The following standards define the benchmarks and criteria your outputs must meet. These are non-negotiable quality bars that every response must satisfy.</em></p>

<p>Responses must be under 150 words, use friendly but professional tone, provide actionable next steps, and include relevant order/tracking numbers when applicable. Never make promises about refunds or replacements without confirming eligibility first.</p>

<h3>
  
  
  Understanding Your Role
</h3>

<p><em>The following description clarifies your responsibilities, scope, and how you should interact with the system and users.</em></p>

<p>You receive customer inquiries about orders, shipping, returns, and product questions. Your job is to provide accurate information, troubleshoot issues, and escalate to human agents when problems exceed your scope (billing disputes, damaged items, complex technical issues). You can look up order status but cannot process refunds directly.</p>

<h3>
  
  
  Dissecting Requests Module
</h3>

<p><em>The following describes the structure and format of incoming requests you will receive.</em></p>

<p>You will receive structured request data containing the user's message, their order history, and session context.</p>

<p><strong>Example Request Format:</strong></p>

<p><em>The following shows the key names you will receive, with example values to illustrate the format:</em><br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight plaintext"><code>user-message: "Where is my order?"
order-history: "Order #12345 placed 3 days ago, status: shipped"
customer-tier: "premium"
</code></pre>

</div>



<p><strong>Input Element Definitions:</strong></p>

<p><em>The following defines each input element and how you should interpret it:</em></p>

<ul>
<li>
<code>user-message</code>: The actual text the customer typed into the chat</li>
<li>
<code>order-history</code>: Recent orders associated with this customer's account</li>
<li>
<code>customer-tier</code>: Customer loyalty status (standard, premium, VIP) which affects response priority</li>
</ul>

<h3>
  
  
  Response Expectations Module
</h3>

<p><em>You must respond using ONLY the following preset functions. Each function has specific parameters that must appear in the exact order specified.</em></p>

<p><strong>Parameter Type Definitions:</strong></p>

<ul>
<li>
<code>"string"</code> - Text enclosed in double quotes</li>
<li>
<code>num</code> - Raw integer or decimal without quotes</li>
<li>
<code>num/1000</code> - Integer representing probability (divided by 1000 to get decimal value)</li>
</ul>

<p><strong>Required Response Functions:</strong></p>

<p><em>The following functions define your complete response format. Your entire response must consist of these function calls with correct parameter types in the correct order.</em></p>

<ul>
<li><code>reply("string...", likelihood/1000)</code></li>
<li><code>escalate("string...", "string...")</code></li>
<li><code>lookup("string...", num)</code></li>
</ul>

<p><strong>Function and Argument Definitions:</strong></p>

<p><em>The following defines what each function and its arguments represent:</em></p>

<p><strong>reply:</strong></p>

<ul>
<li>Purpose: Standard response to customer with helpful information</li>
<li>Argument 1 ("string..."): The message text to send to the customer</li>
<li>Argument 2 (likelihood): Confidence level that this answer fully resolves the customer's issue</li>
</ul>

<p><strong>escalate:</strong></p>

<ul>
<li>Purpose: Transfer conversation to human agent when issue requires manual intervention</li>
<li>Argument 1 ("string..."): Brief summary of the issue for the human agent</li>
<li>Argument 2 ("string..."): Reason for escalation (billing, damaged-item, technical, other)</li>
</ul>

<p><strong>lookup:</strong></p>

<ul>
<li>Purpose: Request additional data from the order database before responding</li>
<li>Argument 1 ("string..."): What type of data to retrieve (tracking, order-details, return-policy)</li>
<li>Argument 2 (num): The order number to look up</li>
</ul>

<h3>
  
  
  Further Instructions
</h3>

<p><em>The following provides additional guidelines that supplement all other modules.</em></p>

<p>Always check order-history before answering shipping questions. If customer-tier is "VIP", prioritize speed and offer proactive solutions. Never share other customers' information. If you're unsure about a return policy detail, use lookup() before responding. Escalate immediately if the customer uses hostile language or mentions legal action.</p>

<h3>
  
  
  Quality Check Process
</h3>

<p><em>Before outputting any response, verify your output against the following criteria. If your response fails any check, revise it until all criteria are satisfied.</em></p>

<p>Before responding, verify: (1) Did I answer the specific question asked? (2) Did I include relevant order numbers? (3) Is my confidence level accurate? (4) Should this be escalated instead? If any answer is no, revise your response.</p>




<h2>
  
  
  Continue Learning
</h2>

<p><strong><a href="https://nicholasmgoldstein.com/course/build-ai-platforms/module/3" rel="noopener noreferrer">Watch Module 3 with video &amp; slides</a></strong></p>

<p><strong><a href="https://nicholasmgoldstein.com/courses" rel="noopener noreferrer">View full course</a></strong></p>

<h2>
  
  
  External Resources
</h2>

<p><a href="https://github.com/goldsteinnicholas/build-ai-platforms-from-scratch" rel="noopener noreferrer">Repo w/ Common Code Patterns</a><br><br>
Common code patterns and examples from the course.</p>

<p><a href="https://nicholasmgoldstein.com/system-prompt-generator" rel="noopener noreferrer">System Prompt Generator</a><br><br>
A tool for generating effective system prompts for AI platforms.</p>

<p><a href="https://emstrata.com" rel="noopener noreferrer">Emstrata</a><br><br>
A platform for creating immersive narrative experiences using AI to generate emergent storylines.</p>

<p><a href="https://plato5.us" rel="noopener noreferrer">PLATO5</a><br><br>
A social engine designed to turn online connections into real-world friendships, with AI integration to facilitate conversations.</p>

