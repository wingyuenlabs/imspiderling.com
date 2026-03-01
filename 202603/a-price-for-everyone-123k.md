---
Title: 🤑 A Price for Everyone 🤑
Description: 
Author: John A Madrigal
Date: 2026-03-01T21:25:33.000Z
Robots: noindex,nofollow
Template: index
---
<p><em>This is a submission for the <a href="https://dev.to/challenges/weekend-2026-02-28">DEV Weekend Challenge: Community</a></em></p>

<h2>
  
  
  The Community 👥
</h2>

<p>I'm an avid card player. You know, the kind that used to play once a week, and now maybe 5 times a year 🤭 I started playing Magic the Gathering back in High School when Invasion was dropping. We would go to the top of the gym, and use gymnastic stacked mats to play Magic all the time, crack new packs, and have the thrill of getting a foil. Cracking packs of Magic used to be so fun, knowing you might get a rare foil, or some interesting mythic. Recently Magic has kind have let my friends and I down. With them releasing IP's like TMNT, SpiderMan, and others, and with foils being in every pack, cracking a pack does not feel special anymore.</p>

<p>Come Sorcery: Contested Realm, a card game that has that exact same thrill of opening packs and getting a foil maybe 1 out of 6 packs. What makes cracking packs so fun, is getting that single rare card, or a curiosa, that could have HUGE value. This game is in it's early stages too, but it has cards valued at over $1,000~ 🤑 (Curiousas, Rare Foils) in the game already!</p>

<p>Most people who collect TCG cards of value, often know about TCGPlayer.com. This site holds the standard when it comes to priced TCG cards on the market.  Sorcery: Contested Realm price data is already on this website. Sorcery also has it's own website for card information, deck building, and collection management. So what's the issue?</p>

<p>Let's go with S:CR (Sorcery: Contested Realm) for the rest of this entry. S:CR's website Curiosa.io is an incredible website. But some friends and I have always wanted to be able to price our collection and decks. I mean how cool would that be?! To be able to check our collection and see how "rich" 😆 we are. But in all honesty, I always thought it would be cool if curiousa.io built a way to check the price of each deck and keep up with your collection value.</p>



<h2>
  
  
  What I Built 👾
</h2>



<p>Big brain thought. Why not just do exactly this ourselves?! The ability to have price information and total value information on the website would be absolutely incredible. This site has been out 2+ years and no one has created anything to allow this. </p>

<p>So, let's go through the "app" I built (😵‍💫,😤,😂). I've been trying to think of ways I could implement this into our community of Sorcery card players and wanted this tool for everyone. Since TCGPlayer and Curiousa.io both run on react / virtual DOM's, the way it loads information makes it almost impossible to scrape. So how can we scrape price data information on TCGPlayer?</p>

<p>In comes <strong>MutationObserver</strong>. I had never thought to use this. After an several attempts on my own trying to figure out how to scrape the data, I had CoPilot help me understand the process of how TCGPlayer loads it's data and how I could grab that information. With <strong>MutationObserver</strong>, it observes the DOM and waits for certain data to appear before starting itself.</p>

<p>Now that we got how we're going to scrape the data, how are we going to create this as something that will scrape the data, match data on a page, and inject data into that page? A Chrome Extension!! And my very first time creating an extension in Chrome. </p>

<p>In building the Chrome Extension, I learned how to create a manifest for Chrome Extension environments, and how a file structure works for Extensions. I think one of the coolest things about Extensions, is how you can just refresh the extension and it grab the newest bit of code with new versions. So let's break down the versions.</p>




<p>1.0.1 - Users have to scrape TCGPlayer for data, built inside the extension, link for scraping and all. Can see price data only for decks on Curiousa.io, card names are matching and price injection is happening. But at the moment, it's not accurate. We are just pulling the first card Name we see, not considering foils and Alpha/Beta same cards. </p>




<p>1.0.2-5 - Can see price data now accurate for versions "Alpha/Beta". Let me tell you, and excuse my french, but that was a bitch to handle. So the way Curiousa.io works is by tRPC Batch Requests. With the extension I had to intercept the fetch request, create the same exact request for card information, and match the data for the set it's under. The set is shown when you click on a card, so I also had to setup the code to automate the clicks, give a loading screen, then inject the price data correctly.</p>




<p>1.0.6 - Now we have foils pulling in, very nicely as well. They have this nice yellow badge on the card name along with the price and all cards are now injecting correct set pricing. </p>




<p>1.0.7 - Added Total cards and Total value of each deck to the top of the deck.</p>




<p>1.0.8 - Created a collection value and used the same name match and price injection to show correct information within the collection.</p>




<p>1.0.9 - Created a link to TCGPlayer.com on the price badge someone can click on to go to the card price information on TCGPlayer.com</p>




<p>1.0.10 - Finessed the wording for the plugin, tested to make sure all price data was matching correctly in both decks and collection, and optimized the code some.</p>




<p>1.1.0 - Created a .crx file extension for testing. </p>

<p>Note: So each user has to scrape the data manually each time to get updated prices and card information from TCGPlayer. I was afraid to hit TCG with tons of scrape data daily from each extension being loaded in Chrome, so I made the scraping happen manually.</p>



<h2>
  
  
  Demo 🎥
</h2>



<p>This is a video demo of the Sorcery Extension being live used.<br>
<iframe width="710" height="399" src="https://www.youtube.com/embed/NCxqKleLHp0">
</iframe>
</p>



<h2>
  
  
  Code 🧬
</h2>




<div class="ltag-github-readme-tag">
  <div class="readme-overview">
    <h2>
      <img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fassets.dev.to%2Fassets%2Fgithub-logo-5a155e1f9a670af7944dd5e12375bc76ed542ea80224905ecaf878b9157cdefc.svg" alt="GitHub logo">
      <a href="https://github.com/trickell" rel="noopener noreferrer">
        trickell
      </a> / <a href="https://github.com/trickell/sorcery_data_scraper" rel="noopener noreferrer">
        sorcery_data_scraper
      </a>
    </h2>
    <h3>
      
    </h3>
  </div>
  <div class="ltag-github-body">
    
<div id="readme" class="md">
<div class="markdown-heading">
<h1 class="heading-element">🔮 Sorcery Price Tracker — Chrome Extension</h1>
</div>
<p>Automatically displays TCGPlayer market prices for <strong>Sorcery: Contested Realm</strong> cards on <a href="https://curiosa.io" rel="nofollow noopener noreferrer">Curiosa.io</a>, including deck totals.</p>

<div class="markdown-heading">
<h2 class="heading-element">File Structure</h2>
</div>
<div class="snippet-clipboard-content notranslate position-relative overflow-auto"><pre class="notranslate"><code>sorcery-price-tracker/
├── manifest.json          # Extension config + permissions
├── background.js          # Service worker: storage, canonical names, messaging
├── content_tcgplayer.js   # Scraper: runs on TCGPlayer, collects card prices
├── content_curiosa.js     # Injector: runs on Curiosa.io, displays prices
├── popup.html             # Extension popup UI
├── popup.js               # Popup logic
└── icons/                 # Add icon16.png, icon48.png, icon128.png here
</code></pre></div>

<div class="markdown-heading">
<h2 class="heading-element">Setup</h2>
</div>
<ol>
<li>Open Chrome and go to <code>chrome://extensions</code>
</li>
<li>Enable <strong>Developer Mode</strong> (top right toggle)</li>
<li>Click <strong>Load unpacked</strong> and select this folder</li>
<li>The extension icon will appear in your toolbar</li>
</ol>

<div class="markdown-heading">
<h2 class="heading-element">How to Use</h2>

</div>
<div class="markdown-heading">
<h3 class="heading-element">Step 1 — Scrape Prices from TCGPlayer</h3>

</div>
<ol>
<li>Click the extension icon → <strong>"Open TCGPlayer to Scrape"</strong>
</li>
<li>The TCGPlayer Sorcery listings page will open</li>
<li>Scroll through the results — prices are saved automatically as cards load</li>
<li>Navigate to page 2…</li>
</ol>
</div>
  </div>
  <div class="gh-btn-container"><a class="gh-btn" href="https://github.com/trickell/sorcery_data_scraper" rel="noopener noreferrer">View on GitHub</a></div>
</div>


<p>Here is a sample of the code I used to scrape TCGPlayer.com website for card data.</p>

<p><em>content_tcgplayer.js</em><br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight plaintext"><code>function observeDOM() {
    const observer = new MutationObserver(() =&gt; {
      clearTimeout(debounceTimer);
      debounceTimer = setTimeout(tryScrape, DEBOUNCE_MS);
    });
    observer.observe(document.body, { childList: true, subtree: true });
  }

  function tryScrape() {
    const results = scrapeCurrentPage();
    const count = Object.keys(results).length;
    if (count === 0) return;

    scrapedOnThisPage = true;
    const slug = getSetSlugFromUrl();
    updateBanner(`Scraped ${count} entries from ${getSetDisplayName(slug)} — saving...`);

    chrome.runtime.sendMessage(
      { type: 'SAVE_PRICES', payload: results },
      (response) =&gt; {
        if (response?.success) {
          updateBanner(`${getSetDisplayName(slug)}: ${count} entries saved (${response.count} total). Open another set's price guide to continue.`);
        }
      }
    );
  }

  // ─────────────────────────────────────────────
  // Scrape the price guide table.
  // Captures card name, price, AND the TCGPlayer
  // url from the card.
  // ─────────────────────────────────────────────
  function scrapeCurrentPage() {
    const prices = {};
    const slug = getSetSlugFromUrl();
    const setDisplayName = getSetDisplayName(slug);

    const rows = document.querySelectorAll('.tcg-table-body tr');
    if (rows.length === 0) return prices;

    rows.forEach((row) =&gt; {
      const nameEl = row.querySelector('.pdp-url');
      const cells  = row.querySelectorAll('.tcg-table-body__cell');

      if (!nameEl || cells.length &lt;= PRICE_COLUMN_INDEX) return;

      const rawName  = nameEl.innerText?.trim();
      const rawPrice = cells[PRICE_COLUMN_INDEX]?.innerText?.trim();

      if (!rawName || !rawPrice) return;
      if (rawPrice === '-' || rawPrice === '' || rawPrice === 'N/A') return;

      const priceFloat = parseFloat(rawPrice.replace(/[^0-9.]/g, ''));
      if (isNaN(priceFloat) || priceFloat === 0) return;

      // Capture the TCGPlayer PDP URL from the anchor tag
      const href = nameEl.href || nameEl.closest('a')?.href || null;
      const pdpUrl = href &amp;&amp; href.startsWith('http') ? href : null;

      const foil = isFoilName(rawName);
      const baseName = foil ? stripFoilSuffix(rawName) : rawName;

      const priceObj = {
        display:   rawPrice,
        value:     priceFloat,
        set:       setDisplayName,
        setSlug:   slug,
        foil:      foil,
        url:       pdpUrl,
        source:    'tcgplayer',
        scrapedAt: Date.now(),
      };
</code></pre>

</div>



<ol>
<li>observeDOM() - This will open a new MutationObserver to watch the body of the page.</li>
<li>tryScrape() - This will create a small banner modal on the bottom of the screen, updating it's value, show the data being scraped, and showing the exact amount of entries stored.</li>
<li>scrapeCurrentPage() - When on TCGPlayer.com it will attempt to scrape the data base on this method and if the classes begin to match it'll auto-scrape the page if the extension is on.</li>
</ol>

<p>So there's a better understanding of the file structure here is how I created the file structure for this extension:</p>

<ul>
<li>manifest.json = Loads the extension data for Chrome</li>
<li>background.js = Runs in background, using SorceryAPI, to pull card data.</li>
<li>content_curiosa.js = Runs when on Curiosa.io website</li>
<li>content_tcgplayer.js = Runs when on TCGPlayer.com</li>
</ul>

<h2>
  
  
  How I Built It 🛠️
</h2>

<p>The way I created the extension was simply with Javascript. It's the only thing it needed. I also used Claude Sonnet in browser to help me understand the way TCGPlayer loaded it's data and the way Curiousa.io was fetching it's card information through tRCP requests.</p>

