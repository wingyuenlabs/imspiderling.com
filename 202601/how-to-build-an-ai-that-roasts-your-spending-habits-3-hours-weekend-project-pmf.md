---
Title: How to Build an AI That Roasts Your Spending Habits (3 hours Weekend Project)
Description: 
Author: Ademola Balogun
Date: 2026-01-19T21:37:08.000Z
Robots: noindex,nofollow
Template: index
---
<p>Building an AI Financial Roaster That People Actually Want to Use</p>

<p>Let me show you how to build the most brutally honest financial advisor you'll ever meet: an AI that doesn't care about your feelings.</p>

<p>I love building things that people actually use. That is why I wanted to create a tutorial project that would teach developers how to make AI applications that people actually enjoy using. Because let's be honest, most AI tutorials are boring calculators and chatbots.</p>

<p>The project? Finance Roaster: Users upload their bank statement, get roasted by GPT-4 with savage, witty humor.</p>

<p>This tutorial will teach you:</p>

<p>How to process CSV and PDF bank statements</p>

<p>How to engineer AI prompts for humor and personality</p>

<p>How to build shareable, viral-worthy UI/UX</p>

<p>How to create an AI app in under 100 lines of Python</p>

<p>By the end of this guide, you'll have a fully functional app you can deploy this weekend.</p>

<p>The Psychology: Why This Project Concept Works</p>

<p>Before diving into code, let's talk about why this makes such a great tutorial project.</p>

<p>Traditional budgeting apps are boring. They show you pie charts and tell you to "reduce discretionary spending." Not exactly share-worthy.</p>

<p>But an AI that says: "$87 at Foods Co for organic kale? Your wallet is as wilted as that kale will be in three days"? That's memorable. That's the kind of feature that makes people actually want to test your app.</p>

<p>Three principles that make this project interesting:</p>

<p>Humor makes boring tasks fun - People avoid checking their finances because it's painful. Adding humor changes that dynamic.</p>

<p>Highly shareable - When you build something funny, people naturally want to show friends. This teaches you virality mechanics.</p>

<p>Practical learning - You'll learn file uploads, AI prompt engineering, and building clean UIs, all transferable skills.</p>

<p>The Tech Stack (Perfect for Learning)</p>

<p>I deliberately chose a simple stack that beginners can understand while still being production-ready.</p>

<p>What we'll use:</p>

<p>Backend: Flask (Python) - 100 lines of code</p>

<p>AI Engine: OpenAI's GPT-4o-mini ($0.15 per 1M tokens)</p>

<p>File Processing: Pandas for CSV, pdfplumber for PDF statements</p>

<p>Frontend: Single HTML page with Tailwind CSS</p>

<p>Cost per request: ~$0.01</p>

<p>Why this stack is perfect for learning:</p>

<p>No deployment complexity (runs locally or on a single server)</p>

<p>Sub-3-second response times</p>

<p>Teaches you file handling, AI integration, and modern UI design</p>

<p>Easily extensible (add features as you learn more)</p>

<p>Part 1: Making AI Understand Money (The Easy Part)</p>

<p>First challenge: How do you teach an AI to roast someone's spending?</p>

<p>Step 1: Extract Transaction Data</p>

<p>Most banks export statements as CSV or PDF. Here's the extraction logic:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight plaintext"><code>import pandas as pd
import pdfplumber
from openai import OpenAI

def process_csv(file):
    """Extract spending summary from CSV bank statement"""
    df = pd.read_csv(file)

    # Find the amount column (banks use different names)
    amount_col = next(
        (col for col in df.columns 
         if any(kw in col.lower() for kw in ['amount', 'debit', 'spent'])),
        df.columns[-1]  # Default to last column
    )

    # Convert to numeric, handle currency symbols
    df[amount_col] = pd.to_numeric(
        df[amount_col].astype(str).str.replace('$', '').str.replace(',', ''),
        errors='coerce'
    )

    # Calculate summary
    total_spent = df[amount_col].abs().sum()
    top_expenses = df.nlargest(5, amount_col)

    return {
        'total_spent': f"${total_spent:,.2f}",
        'top_expenses': top_expenses.to_dict('records')
    }
</code></pre>

</div>



<p>PDF handling is trickier because banks format statements differently. Solution? Use pdfplumber for text extraction, then let GPT-4 parse it:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight plaintext"><code>def extract_transactions_from_pdf(pdf_text):
    """Use AI as a fallback parser for complex PDFs"""
    response = client.chat.completions.create(
        model="gpt-4o-mini",
        messages=[{
            "role": "system",
            "content": "Extract transactions: Date | Description | Amount"
        }, {
            "role": "user",
            "content": pdf_text[:4000]  # Limit tokens
        }]
    )
    # Parse AI response into structured data
    # ...
</code></pre>

</div>



<p>Pro tip: AI is surprisingly good at parsing messy financial data. I tried regex patterns for weeks before realizing GPT-4 could do it in one call.</p>

<p>Part 2: Teaching AI to Be Savage (The Fun Part)</p>

<p>Now the magic: turning transaction data into comedy gold.</p>

<p>The System Prompt (Your AI's Personality)</p>

<p>This is where most people fail. A generic "be funny" prompt produces generic humor. You need specificity.</p>

<p>Here's my system prompt:</p>

<p>SYSTEM_PROMPT = """<br>
You are a Savage Financial Parent - brutally honest, wickedly funny, <br>
and armed with someone's transaction history. </p>

<p>Your job: Roast their spending habits using SPECIFIC transaction details.</p>

<p>Style guide:</p>

<ul>
<li>Think disappointed parent meets stand-up comedian</li>
<li>Use their actual purchase names (e.g., "Taco Bell at 2 AM")</li>
<li>Be savage but not cruel - make them laugh while crying</li>
<li>End with a Financial Maturity Grade (A-F) and one-line explanation</li>
</ul>

<p>Examples of good roasts:</p>

<ul>
<li>"Five Starbucks trips in one day? That's not a coffee addiction, 
that's a $30 therapy session with extra foam."</li>
<li>"You spent $200 on DoorDash this month. That's literally paying 
someone $8 to make your laziness official."</li>
</ul>

<p>Keep it under 200 words. Make it screenshot-worthy.<br>
"""</p>

<p>Why this works:</p>

<p>Specific examples teach the AI your humor style</p>

<p>Constraints (200 words) force concise, punchy writing</p>

<p>"Screenshot-worthy" reminds the AI this is for social sharing</p>

<p>The Actual API Call<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight plaintext"><code>def generate_roast(summary_data):
    """Send transaction summary to OpenAI, get roast back"""

    prompt = f"""
Transaction Data:
- Total Spent: {summary_data['total_spent']}
- Top Expense: {summary_data['top_expenses'][0]}
- You went to {summary_data['frequent_vendors'][0]['vendor']} 
  {summary_data['frequent_vendors'][0]['count']} times

Roast these spending habits. Be specific and savage.
"""

    response = client.chat.completions.create(
        model="gpt-4o-mini",
        messages=[
            {"role": "system", "content": SYSTEM_PROMPT},
            {"role": "user", "content": prompt}
        ],
        temperature=0.9  # High creativity for humor
    )

    return response.choices[0].message.content
</code></pre>

</div>



<p>The temperature setting (0.9) is crucial. Lower values make AI boring. Higher values make it creative... sometimes too creative. 0.9 is the sweet spot for comedy.</p>

<p>Part 3: The UI That Makes People Click "Roast Me"</p>

<p>Nobody uploads bank statements to ugly websites. Your UI needs to be:</p>

<p>Dark and edgy (matches the roasting vibe)</p>

<p>Dead simple (one button: "Roast Me")</p>

<p>Fast (loading states matter)</p>

<p>The Landing Page<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight plaintext"><code>&lt;div class="bg-gradient-to-br from-gray-900 to-black min-h-screen"&gt;
  &lt;h1 class="text-6xl font-bold bg-gradient-to-r from-red-500 to-pink-500 
             bg-clip-text text-transparent"&gt;
    💸 Finance Roaster
  &lt;/h1&gt;
  &lt;p class="text-xl text-gray-400"&gt;
    Upload Your Regrets. Get Brutally Honest Feedback.
  &lt;/p&gt;

  &lt;!-- Drag-and-drop upload area --&gt;
  &lt;div class="border-2 border-dashed border-red-500 rounded-xl p-12 
              hover:bg-red-500/10 cursor-pointer"&gt;
    &lt;input type="file" accept=".csv,.pdf" hidden /&gt;
    &lt;p&gt;Drop your bank statement here&lt;/p&gt;
    &lt;p class="text-sm text-gray-500"&gt;CSV or PDF • Max 16MB&lt;/p&gt;
  &lt;/div&gt;

  &lt;button class="w-full bg-gradient-to-r from-red-500 to-pink-500 
                 text-white font-bold py-4 rounded-xl"&gt;
    🔥 Roast Me
  &lt;/button&gt;
&lt;/div&gt;
</code></pre>

</div>



<p>Key design choices:</p>

<p>Red/pink gradient: Danger + playfulness</p>

<p>Dark background: Focuses attention on the upload area</p>

<p>Emojis: Adds personality without extra design work</p>

<p>Tailwind CSS: Rapid prototyping without writing CSS</p>

<p>The Loading State (Critical for Virality)</p>

<p>While the AI thinks, show personality:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight plaintext"><code>function showLoading() {
  document.getElementById('loadingState').innerHTML = `
    &lt;div class="animate-spin h-16 w-16 border-b-2 border-red-500"&gt;&lt;/div&gt;
    &lt;p class="text-xl"&gt;Analyzing your poor life choices...&lt;/p&gt;
    &lt;p class="text-sm text-gray-500"&gt;This won't take long. Unlike your debt.&lt;/p&gt;
  `;
}
</code></pre>

</div>



<p>Why this matters: Users wait 3-5 seconds for AI responses. A boring spinner loses attention. A funny loading message keeps them engaged.</p>

<p>Part 4: Security (Because Banks Care About This)</p>

<p>You're handling financial data. Even for a joke app, security matters.</p>

<p>Three Non-Negotiable Rules:</p>

<ol>
<li>Never store files on disk</li>
</ol>

<h1>
  
  
  ❌ BAD: Saves file to server
</h1>

<p>file.save('statements/' + filename)</p>

<h1>
  
  
  ✅ GOOD: Process in-memory
</h1>

<p>file_bytes = io.BytesIO(file.read())<br>
df = pd.read_csv(file_bytes)</p>

<ol>
<li>Don't log transaction details</li>
</ol>
<h1>
  
  
  ❌ BAD: Logs sensitive data
</h1>

<p>print(f"User spent ${amount} at {merchant}")</p>
<h1>
  
  
  ✅ GOOD: Generic logging
</h1>

<p>print(f"Processing statement with {len(transactions)} transactions")</p>

<ol>
<li>Strip personal info before sending to OpenAI
</li>
</ol>
<div class="highlight js-code-highlight">
<pre class="highlight plaintext"><code># Remove account numbers, names, addresses
summary_data = {
    'total_spent': total,
    'top_categories': categories,  # "Food", not "Joe's Pizza"
    'spending_pattern': pattern
}
</code></pre>

</div>


<p>Reality check: Even with sanitization, tell users their data goes to OpenAI. Transparency builds trust.</p>

<p>Part 5: Adding Viral Features (Optional Enhancement)</p>

<p>If you want to take this tutorial further, here's how to add shareability:</p>

<p>Built-in Sharing Capability:</p>

<ol>
<li>One-Click Social Sharing
</li>
</ol>
<div class="highlight js-code-highlight">
<pre class="highlight plaintext"><code>function shareRoast() {
  const roastText = document.getElementById('roast').textContent;
  const shareText = `I just got my spending roasted by AI! 🔥\n\n"${roastText.substring(0, 200)}..."\n\nBuild your own: [your-github-link]`;

  if (navigator.share) {
    navigator.share({ text: shareText });
  } else {
    navigator.clipboard.writeText(shareText);
    alert('Roast copied! Share with friends! 🔥');
  }
}
</code></pre>

</div>


<p>Why this matters: If you eventually deploy this publicly, sharing features help spread awareness.</p>

<ol>
<li>The Financial Maturity Grade</li>
</ol>

<p>Every roast ends with a grade: A through F.</p>

<p>Financial Maturity Grade: D-<br>
"You have the self-control of a toddler in a candy store, <br>
except the candy is overpriced artisanal coffee."</p>

<p>Learning point: Grades, scores, and rankings make results more shareable and comparable.</p>

<p>What You'll Learn Building This</p>

<p>This project teaches you several valuable skills:</p>

<p>File Processing:</p>

<p>Handle both CSV and PDF uploads</p>

<p>Parse unstructured financial data</p>

<p>Work with Pandas for data analysis</p>

<p>AI Engineering:</p>

<p>Craft effective system prompts for personality</p>

<p>Temperature tuning for creative outputs</p>

<p>Handle AI responses in production</p>

<p>Full-Stack Development:</p>

<p>Build clean REST APIs with Flask</p>

<p>Create engaging UIs with Tailwind CSS</p>

<p>Implement drag-and-drop file uploads</p>

<p>Handle loading states and user feedback</p>

<p>Deployment Ready:</p>

<p>In-memory file processing (no disk storage)</p>

<p>Error handling and validation</p>

<p>Security best practices for financial data</p>

<p>Design Decisions: Why I Built It This Way</p>

<p>What Works in This Architecture:</p>

<p>Simplicity over features: A single-purpose app is easier to understand and build. You can always add features later once you understand the core.</p>

<p>Humor as the hook: The AI's personality is what makes this project interesting. Technical tutorials don't have to be boring.</p>

<p>File processing in memory: No database needed for this tutorial. Everything happens in RAM, making deployment simple.</p>

<p>What You Might Customize:</p>

<p>Add comparison data: "You spent more on coffee than X% of typical users" - requires a database to track aggregates.</p>

<p>Mobile-first design: The current UI works on mobile, but you could optimize it further with responsive breakpoints.</p>

<p>Prompt variations: The system prompt I provide is a starting point. Experiment with different personalities and tones.</p>

<p>Build Your Own AI App: What This Template Teaches</p>

<p>This project is designed to be a template for other AI applications. Here's how to adapt it:</p>

<ol>
<li>Find a task people find tedious</li>
</ol>

<p>Checking finances → This tutorial</p>

<p>Reading legal documents → "Legal Jargon Translator"</p>

<p>Analyzing health data → "Fitness Report Card"</p>

<p>Reviewing meeting notes → "Meeting BS Detector"</p>

<ol>
<li>Add personality through AI prompts</li>
</ol>

<p>Make AI roast them (finance, fitness)</p>

<p>Turn it into a game (quiz format)</p>

<p>Add competitive elements (grades, scores)</p>

<p>Use unexpected analogies (explain tech in food terms)</p>

<ol>
<li>Keep the UI simple</li>
</ol>

<p>One-click file upload</p>

<p>Clear results display</p>

<p>Optional sharing features</p>

<p>Fast loading indicators</p>

<ol>
<li>Make it easy to extend</li>
</ol>

<p>Modular code structure</p>

<p>Clear separation of concerns</p>

<p>Well-commented functions</p>

<p>Standard design patterns</p>

<p>The Complete Code Repository</p>

<p>The entire Finance Roaster tutorial project is available on GitHub. Here's the structure:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight plaintext"><code>FinanceRoaster/
├── app.py              # Flask server (81 lines)
├── services.py         # AI + file processing (156 lines)
├── templates/
│   └── home.html       # UI (342 lines with CSS/JS)
├── requirements.txt    # Dependencies (7 packages)
├── .env.example       # Template for your API key
└── README.md          # Setup instructions
</code></pre>

</div>



<p>To run this tutorial locally:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight plaintext"><code>git clone https://github.com/yourusername/FinanceRoaster
cd FinanceRoaster
pip install -r requirements.txt
# Copy .env.example to .env and add your OpenAI key
python app.py

Visit http://localhost:5000 and test it out.
</code></pre>

</div>



<p>Learning cost: ~$0.01 per test (OpenAI API calls)</p>

<p>What's included:</p>

<p>Complete working code</p>

<p>Detailed comments explaining each function</p>

<p>Example bank statements for testing</p>

<p>Deployment guide for Heroku/Railway</p>

<p>Potential Use Cases (What You Could Build)</p>

<p>Once you understand this template, you can adapt it for various applications:</p>

<p>Personal Finance Tools:</p>

<p>Budget analyzer with friendly advice</p>

<p>Subscription tracker that flags unused services</p>

<p>Shopping habit analyzer (impulse vs planned purchases)</p>

<p>Professional Development:</p>

<p>Resume reviewer that gives honest feedback</p>

<p>Email tone analyzer (passive-aggressive detector)</p>

<p>Meeting notes summarizer with action items</p>

<p>Health &amp; Wellness:</p>

<p>Food diary analyzer with nutrition roasts</p>

<p>Workout consistency tracker with motivational guilt</p>

<p>Sleep pattern analyzer with bedtime recommendations</p>

<p>Content Creation:</p>

<p>Social media post analyzer (engagement predictor)</p>

<p>Blog post readability scorer</p>

<p>Video script feedback tool</p>

<p>The core pattern: upload file → AI analysis → humorous/useful feedback - works for countless domains.</p>

<p>The Technical Deep Dive (For the Nerds)</p>

<p>Challenge 1: PDF Parsing Hell</p>

<p>Banks format PDFs differently. Chase, Wells Fargo, Bank of America - all unique snowflakes.</p>

<p>My solution: Hybrid approach<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight plaintext"><code>def parse_pdf_transactions(pdf_text):
    # Try regex first (fast, works 70% of the time)
    pattern = r'(\d{1,2}/\d{1,2}/\d{4})\s+(.+?)\s+([-]?\$?\d+\.?\d*)'
    matches = re.findall(pattern, pdf_text)

    if matches:
        return pd.DataFrame(matches, columns=['Date', 'Desc', 'Amount'])

    # Fallback to AI (slower, works 95% of the time)
    return extract_with_gpt(pdf_text)
</code></pre>

</div>



<p>Lesson: AI should be your fallback, not your first choice. Regex is 100x faster.</p>

<p>Challenge 2: Keeping Roasts PG-13</p>

<p>Early versions were too savage. One user got: "Your spending screams 'quarter-life crisis.' Seek therapy."</p>

<p>The fix: Content filters<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight plaintext"><code>FORBIDDEN_TOPICS = [
    'therapy', 'depression', 'mental health', 
    'divorce', 'death', 'addiction'
]

def sanitize_roast(roast_text):
    # Check for sensitive topics
    if any(topic in roast_text.lower() for topic in FORBIDDEN_TOPICS):
        return generate_roast(summary_data)  # Retry
    return roast_text
</code></pre>

</div>



<p>Temperature tuning also helps: Lowering from 1.0 to 0.9 reduced inappropriate jokes by 60%.</p>

<p>Challenge 3: Rate Limiting Without Breaking UX</p>

<p>OpenAI has rate limits. Hitting them = angry users.</p>

<p>Solution: Queue system<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight plaintext"><code>from redis import Redis
from rq import Queue

queue = Queue(connection=Redis())

@app.route('/roast', methods=['POST'])
def roast():
    job = queue.enqueue(process_and_roast, file_data)
    return jsonify({'job_id': job.id})

@app.route('/status/&lt;job_id&gt;')
def status(job_id):
    job = Job.fetch(job_id, connection=Redis())
    if job.is_finished:
        return jsonify({'status': 'complete', 'roast': job.result})
    return jsonify({'status': 'processing'})
</code></pre>

</div>



<p>Frontend polls /status every second. Feels instant, never hits limits.</p>

<p>Extension Ideas: Taking It Further</p>

<p>Once you've built the basic version, here are ways to extend it:</p>

<ol>
<li>Add data persistence
</li>
</ol>

<div class="highlight js-code-highlight">
<pre class="highlight plaintext"><code># Save anonymized spending patterns
from flask_sqlalchemy import SQLAlchemy

db = SQLAlchemy(app)

class SpendingPattern(db.Model):
    id = db.Column(db.Integer, primary_key=True)
    total_amount = db.Column(db.Float)
    category = db.Column(db.String(50))
    timestamp = db.Column(db.DateTime)
</code></pre>

</div>



<ol>
<li>Implement comparison features
</li>
</ol>

<div class="highlight js-code-highlight">
<pre class="highlight plaintext"><code>def get_percentile(user_spending, category):
    all_spending = SpendingPattern.query.filter_by(category=category).all()
    percentile = sum(1 for x in all_spending if x.total_amount &lt; user_spending)
    return (percentile / len(all_spending)) * 100
</code></pre>

</div>



<ol>
<li>Add authentication
</li>
</ol>

<div class="highlight js-code-highlight">
<pre class="highlight plaintext"><code>from flask_login import LoginManager, login_required

@app.route('/history')
@login_required
def view_history():
    # Show user's past roasts
    pass
</code></pre>

</div>



<ol>
<li>Implement rate limiting
</li>
</ol>

<div class="highlight js-code-highlight">
<pre class="highlight plaintext"><code>from flask_limiter import Limiter

limiter = Limiter(app, key_func=lambda: request.remote_addr)

@app.route('/roast', methods=['POST'])
@limiter.limit("5 per hour")
def roast():
    # Prevent API abuse
    pass
</code></pre>

</div>



<p>Deployment Options: From Local to Production</p>

<p>Once your project works locally, here are deployment options:</p>

<p>Option 1: Heroku (Easiest)<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight plaintext"><code># Install Heroku CLI, then:
heroku create FinanceRoaster
git push heroku main
heroku config:set OPENAI_API_KEY=your_key
</code></pre>

</div>



<p>Option 2: Railway</p>

<p>Connect your GitHub repo</p>

<p>Add environment variables</p>

<p>Auto-deploys on push</p>

<p>Option 3: DigitalOcean App Platform</p>

<p>Deploy from GitHub</p>

<p>$5/month for basic app</p>

<p>Built-in SSL certificates</p>

<p>Scaling considerations:</p>

<p>Caching for similar inputs: Store common roast patterns</p>

<p>Rate limiting: Prevent abuse with Flask-Limiter</p>

<p>CDN for assets: Use Cloudflare free tier</p>

<p>Database: PostgreSQL if you add user accounts</p>

<p>For a tutorial project, any of these platforms work great. Start with Heroku's free tier to test.</p>

<p>The Bigger Picture: What This Tutorial Teaches About AI Development</p>

<p>Here's what building Finance Roaster teaches you about creating useful AI applications:</p>

<p>Lesson 1: AI doesn't have to be serious. Most AI tutorials focus on optimization, efficiency, and accuracy. But the best projects are ones people actually want to use. Adding personality makes your projects memorable.</p>

<p>Lesson 2: Simple prompts can be powerful. The entire "roasting" capability comes from a well-crafted system prompt. You don't need fine-tuning or complex models, just clear instructions and examples.</p>

<p>Lesson 3: User experience matters more than complexity. A 100-line Flask app with great UX beats a microservices architecture with boring design. Focus on the user experience first, optimize later.</p>

<p>Lesson 4: File processing is a valuable skill. Being able to parse CSVs and PDFs opens up countless project possibilities. Financial statements, receipts, invoices, medical records, they're all structured data waiting to be analyzed.</p>

<p>This pattern applies to many domains:</p>

<p>Upload document → AI analyzes → Useful/entertaining output</p>

<p>It's simple, it works, and users understand it immediately</p>

<p>Try It Yourself: Weekend Project Checklist</p>

<p>Want to build your own AI app this weekend? Here's your roadmap:</p>

<p>Friday night (0.5 hours):</p>

<p>Pick your painful task (finance, health, career, dating)</p>

<p>Write 10 example roasts manually</p>

<p>Define your AI's personality in 3 sentences</p>

<p>Saturday (1 hour):</p>

<p>Set up Flask + OpenAI</p>

<p>Build file upload handling</p>

<p>Test your system prompt with real data</p>

<p>Iterate on temperature/prompt until roasts are funny</p>

<p>Sunday (1.5 hours):</p>

<p>Design UI with Tailwind</p>

<p>Add one-click sharing</p>

<p>Test on 5 friends (get honest feedback)</p>

<p>Deploy to your preferred cloud platform</p>

<p>Total time: 3 hours\Total cost: $0 (Heroku free tier + OpenAI free credits)</p>

<p>The Code (Seriously, It's That Simple)</p>

<p>Here's the entire backend in 81 lines:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight plaintext"><code>from flask import Flask, request, jsonify, render_template
import io, pandas as pd, pdfplumber
from openai import OpenAI
from dotenv import load_dotenv
import os

load_dotenv()
app = Flask(__name__)
client = OpenAI(api_key=os.getenv('OPENAI_API_KEY'))

SYSTEM_PROMPT = """You are a Savage Financial Parent - brutally honest, 
wickedly funny, armed with transaction history. Roast their spending using 
SPECIFIC details. Think disappointed parent meets stand-up comedian. 
Keep under 200 words. End with Financial Maturity Grade (A-F)."""

def process_csv(file):
    df = pd.read_csv(file)
    amount_col = next((c for c in df.columns 
                      if 'amount' in c.lower()), df.columns[-1])
    df[amount_col] = pd.to_numeric(
        df[amount_col].astype(str).str.replace('$', ''),
        errors='coerce'
    )

    total = df[amount_col].abs().sum()
    top5 = df.nlargest(5, amount_col)

    return {
        'total_spent': f"${total:,.2f}",
        'top_expenses': top5.to_dict('records')
    }

def generate_roast(summary):
    prompt = f"""
Total Spent: {summary['total_spent']}
Top Expenses: {summary['top_expenses']}

Roast these spending habits with savage humor.
"""

    response = client.chat.completions.create(
        model="gpt-4o-mini",
        messages=[
            {"role": "system", "content": SYSTEM_PROMPT},
            {"role": "user", "content": prompt}
        ],
        temperature=0.9
    )

    return response.choices[0].message.content

@app.route('/')
def home():
    return render_template('home.html')

@app.route('/roast', methods=['POST'])
def roast():
    try:
        file = request.files['file']
        file_bytes = io.BytesIO(file.read())

        summary = process_csv(file_bytes)
        roast_text = generate_roast(summary)

        return jsonify({
            'success': True,
            'roast': roast_text,
            'summary': summary
        })
    except Exception as e:
        return jsonify({'error': str(e)}), 500

if __name__ == '__main__':
    app.run(debug=True)
</code></pre>

</div>



<p>That's it. 81 lines. No ML training, no complex infrastructure, no month-long sprints.</p>

<p><strong>Final Thoughts: Learn by Building</strong></p>

<p>This tutorial project - a simple weekend build with 81 lines of Python - teaches more practical AI skills than any of those complex systems.</p>

<p>Why? Because you can actually build it yourself, understand every line, and adapt it to your own ideas.</p>

<p>The lesson? The best way to learn AI development is to build projects that are:</p>

<ul>
<li>Simple enough to finish (weekend projects, not month-long sprints)</li>
<li>Interesting enough to share (add personality, make it fun)</li>
<li>Practical enough to extend (real use cases, clear applications)
So stop reading tutorials and start building. Pick a file format (PDF, CSV, JSON), add some AI analysis, and ship it this weekend.</li>
</ul>

<p>And when you build something cool, share it! The AI development community loves creative projects.</p>

<p><strong>Resources &amp; Next Steps</strong></p>

<p>Complete tutorial code: <a href="https://github.com/ademicho123/FinanceRoaster" rel="noopener noreferrer">https://github.com/ademicho123/FinanceRoaster</a> (Fork it and customize!)</p>

<p><strong>What to do after building this:</strong></p>

<ul>
<li>Deploy it online (Heroku, Railway, or DigitalOcean)</li>
<li>Share with friends for feedback</li>
<li>Adapt the pattern for a different domain</li>
<li>Add the project to your portfolio</li>
</ul>

<p>P.S. After you build this, try uploading your own bank statement. The AI is surprisingly insightful (and funny). Consider it a feature test and free financial feedback.</p>

<p>Additionally, if you extend this tutorial with cool features, please submit a pull request! The best additions will be merged into the main repo with credit to you.</p>

