---
Title: I Built a Side Project That Works in 4 Languages — Here's What I Learned
Description: 
Author: david
Date: 2026-03-02T21:36:11.000Z
Robots: noindex,nofollow
Template: index
---
<p>When I started building k4pi, I thought the hardest part would be the AI. Spoiler: it wasn't.</p>

<p>The hardest part was realizing that "marketplace" means completely different things depending on where your user lives.</p>




<h2>
  
  
  The Accidental Multilingual Product
</h2>

<p>I didn't plan to support four languages. I planned to build a Telegram bot for buying and selling stuff — simple, useful, done. But Telegram users don't care about your roadmap. They come from everywhere: Russia, India, Latin America, the US, Europe. Within the first few weeks of testing, I got messages in Russian, English, Spanish, and Hindi. So I had a choice: ignore them, or adapt.</p>

<p>I adapted. And that decision turned into one of the most educational rabbit holes I've ever fallen into.</p>

<p><a href="https://t.me/k4pi_bot" rel="noopener noreferrer">k4pi (@k4pi_bot)</a> is an AI-powered marketplace bot that runs entirely inside Telegram. No app to download, no website to register on. You post what you're selling, the AI categorizes it, filters inappropriate content, indexes it for search — and buyers find it instantly. The whole thing is powered by vector search and image recognition. But none of that matters if the bot can't understand what language someone is speaking.</p>




<h2>
  
  
  The Technical Side of i18n (It's Not Just Translation)
</h2>

<p>Let me be honest: the first version of multilingual support was embarrassing. I used a dictionary of hardcoded strings, ran them through a translation service, and called it done. It worked for about five minutes.</p>

<p>The problem with translation-first thinking is that languages aren't just different words — they're different sentence structures. Russian is heavily inflected, which means the same word changes form depending on grammatical case. "Buy" becomes "купить", "купил", "куплю", "купленный" — and if you're searching for products, you want to match all of them. This is why I ended up integrating a morphological analyzer (pymorphy3) specifically for Russian-language search. Without it, searching "телефон" wouldn't find listings that said "телефона" or "телефоны".</p>

<p>Hindi has a similar issue with Devanagari script — compound words and morphological complexity that naive string matching completely misses. Spanish is more forgiving by comparison, but compound nouns and regional vocabulary still trip up basic search.</p>

<p><strong>The real lesson:</strong> don't translate your product. <em>Localize</em> it. These are fundamentally different things.</p>

<p>For auto-detecting language, I ended up using a combination of Telegram's built-in <code>language_code</code> from the user object and runtime detection on message content. The Telegram language code is a starting point, but it's unreliable — plenty of Russian speakers have their Telegram set to English because they learned the interface that way. Actual message content detection (even something lightweight) is more accurate.</p>




<h2>
  
  
  Cultural Differences I Did Not Expect
</h2>

<p>This is the part nobody writes about in i18n guides.</p>

<p><strong>Russian-speaking markets:</strong> Sellers here are incredibly detail-oriented in their listings. You get precise measurements, condition grades, original packaging details. They also expect negotiation — a fixed price is almost an insult. The bot's "make an offer" flow gets used constantly in Russian-language listings.</p>

<p><strong>English-speaking markets:</strong> Much more terse. "iPhone 13, good condition, $400, DM me." That's it. They trust the platform more than the seller's description. They want the AI to surface trust signals — photo quality, listing completeness, seller history.</p>

<p><strong>Spanish-speaking markets (Latin America specifically):</strong> Wildly more social. The buying process often starts with "Hola!" before any negotiation. Users wanted to chat <em>around</em> the product before committing. I had to rethink how the conversation flow worked — the bot felt too transactional and cold for this audience initially.</p>

<p><strong>Hindi-speaking markets:</strong> Price sensitivity is extremely high, and category specificity matters enormously. A "clothing" category doesn't cut it — you need "sarees", "kurtas", "western wear" as distinct buckets. I ended up expanding the taxonomy significantly after looking at what people were actually posting.</p>

<p>None of this was in any i18n guide I read. You learn it by watching real users interact with your product.</p>




<h2>
  
  
  The Search Problem Across Languages
</h2>

<p>Building a search engine that works across four languages is genuinely hard. I'm using Elasticsearch with language-specific analyzers, plus vector search via Qdrant for semantic similarity (powered by a SigLIP model that I quantized down to about 270MB so it actually fits in RAM on a budget VPS).</p>

<p>The vector search is the great equalizer — because image embeddings don't care what language the listing is in. A picture of a couch is a picture of a couch whether the description is in Russian or Spanish. So cross-language product discovery actually works pretty well at the image level.</p>

<p>But text search? That's where things get messy. You need separate analyzers, separate tokenization logic, and ideally morphological analysis for heavily inflected languages. The Elasticsearch <code>language</code> analyzer helps, but it doesn't solve everything. For Russian I added pymorphy3 for lemmatization. For other languages I'm leaning more heavily on the vector search as a fallback.</p>

<p>The architecture that ended up working:</p>

<ol>
<li>BM25 text search with language-specific analyzers (precision)</li>
<li>Vector search on text embeddings (recall and cross-language)</li>
<li>Image vector search (language-agnostic discovery)</li>
<li>Reciprocal rank fusion to combine all three</li>
</ol>




<h2>
  
  
  What Actually Surprised Me
</h2>

<p>I expected the AI part to be the challenge. Instead, the hardest problems were:</p>

<p><strong>1. Date formats.</strong> Trivial, right? Not when you have "expired listing" logic. Russia uses DD.MM.YYYY. The US uses MM/DD/YYYY. India often uses DD-MM-YYYY. One wrong assumption and your "remove old listings" job starts deleting active ones.</p>

<p><strong>2. Currency and price display.</strong> I show prices as entered by sellers. No conversion, no formatting assumption. The bot just stores the raw string. This sounds lazy but it's actually correct — a seller knows their market, and converting currencies adds a trust problem you don't need.</p>

<p><strong>3. Moderation across cultures.</strong> My NSFW filter is trained on mostly English-language concepts of what's inappropriate. It over-flags certain cultural items that are completely normal in some markets. This is an ongoing calibration problem I haven't fully solved.</p>

<p><strong>4. Right-to-left languages.</strong> I don't support Arabic or Hebrew yet, but when I looked at Telegram's user base, I realized this will come up eventually. The entire UI layout assumption changes. Adding this to the backlog now before it becomes painful later.</p>




<h2>
  
  
  The #BuildInPublic Part
</h2>

<p>I've shipped a lot of side projects that nobody used. This one got real users, in multiple countries, doing real transactions, within the first month. I attribute that almost entirely to not limiting the language from day one.</p>

<p>The mistake I see a lot of indie hackers make: building for "English speakers" and planning to add languages "later". Later never comes. The code debt accumulates. The translation architecture you hacked in at the start becomes a prison.</p>

<p>My advice: decide on your language strategy before you write your first string. Use a proper i18n library. Store user language preference in your DB from day one. Even if you only support one language initially, the scaffolding matters.</p>

<p>And if your product is on Telegram — which has 950 million monthly active users spread across the entire globe — defaulting to English-only is leaving most of your potential users on the table.</p>




<h2>
  
  
  Try It Now
</h2>

<p>If you want to see what a multilingual AI marketplace feels like in practice, the bot is live and free to use.</p>

<p>Find something to sell. Take a photo. Send it to <a href="https://t.me/k4pi_bot" rel="noopener noreferrer">@k4pi_bot</a>. The AI will handle categorization, description, and indexing. Buyers can search by text or photo. Everything runs inside Telegram — no account, no app, no friction.</p>

<p><strong><a href="https://t.me/k4pi_bot" rel="noopener noreferrer">Open @k4pi_bot on Telegram</a></strong></p>

<p>I'm building this in the open, learning as I go. If you've solved any of these multilingual problems in your own projects, I'd genuinely love to hear how. Drop a comment below.</p>

