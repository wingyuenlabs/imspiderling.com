---
Title: Normal Forms and MongoDB
Description: 
Author: Franck Pachot
Date: 2026-02-07T21:53:57.000Z
Robots: noindex,nofollow
Template: index
---
<p>I learned about normal forms when databases were designed before the applications that used them. At that time, relational data models focused on enterprise-wide entities, defined before access patterns were known, so future applications could share a stable, normalized schema.</p>

<p>Today, we design databases for specific applications or bounded domains. Instead of defining a full model up front, we add features incrementally, gather feedback, and let the schema evolve with the application.</p>

<p>Normal forms aren't just relational theory—they describe real data dependencies. MongoDB's document model doesn't exempt you from thinking about normalization—it gives you more flexibility in how you apply it.</p>

<h2>
  
  
  Example: Pizzerias
</h2>

<p>We're starting a new business: a large network of pizzerias across many areas with a wide variety of pizzas. But let's start small.</p>

<h2>
  
  
  Tabular: One Pizza, One Area
</h2>

<p>As a minimal viable product (MVP), each pizzeria has one manager, sells only one variety, and delivers to one area. You can choose any database for this: key-value, relational, document, or even a spreadsheet. The choice will matter only when your product evolves.</p>

<p>Here is our first pizzeria:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight javascript"><code><span class="p">{</span>
  <span class="nl">name</span><span class="p">:</span> <span class="dl">"</span><span class="s2">A1 Pizza</span><span class="dl">"</span><span class="p">,</span>
  <span class="nx">manager</span><span class="p">:</span> <span class="dl">"</span><span class="s2">Bob</span><span class="dl">"</span><span class="p">,</span>
  <span class="nx">variety</span><span class="p">:</span> <span class="dl">"</span><span class="s2">Thick Crust</span><span class="dl">"</span><span class="p">,</span>
  <span class="nx">area</span><span class="p">:</span> <span class="dl">"</span><span class="s2">Springfield</span><span class="dl">"</span>
<span class="p">}</span>
</code></pre>

</div>



<p>With no repeating groups or multi-valued attributes, the model is already in First Normal Form (1NF). Because the MVP data model is simple—one value per attribute and a single key—there are no dependencies that would violate higher normal forms.</p>

<p>Many database designs start out fully normalized, not because the designer worked through every normal form, but because the initial dataset is too simple for complex dependencies to exist.</p>

<p>Normalization becomes necessary later, as business rules evolve and new varieties, areas, and independent attributes introduce dependencies that higher normal forms address.</p>

<h2>
  
  
  1NF: More Menu Options
</h2>

<p>The business started quite well and evolves. A pizzeria can now offer several varieties.</p>

<p>The following, adding multiple varieties in a single field, would violate 1NF:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight javascript"><code><span class="p">{</span>
  <span class="nl">name</span><span class="p">:</span> <span class="dl">"</span><span class="s2">A1 Pizza</span><span class="dl">"</span><span class="p">,</span>
  <span class="nx">manager</span><span class="p">:</span> <span class="dl">"</span><span class="s2">Bob</span><span class="dl">"</span><span class="p">,</span>
  <span class="nx">varieties</span><span class="p">:</span> <span class="dl">"</span><span class="s2">Thick Crust, Stuffed Crust</span><span class="dl">"</span><span class="p">,</span>
  <span class="nx">area</span><span class="p">:</span> <span class="dl">"</span><span class="s2">Springfield</span><span class="dl">"</span>
<span class="p">}</span>
</code></pre>

</div>



<p>1NF requires atomic values—each field should hold one indivisible piece of data. A comma-separated string breaks this rule: you can't easily query, index, or update individual varieties. You can manipulate it as a character string, but you can't treat each entry as a distinct pizza variety, and you can't index it efficiently.</p>

<p>SQL and NoSQL databases avoid this pattern for different reasons. In a relational database, the logical model must be independent of cardinalities and access patterns. Because the relational model doesn't know whether there are two or one million pizza varieties, it treats every one-to-many relationship as unbounded and stores it in a separate table as a set of pizzeria–variety relationships rather than embedding varieties within the pizzeria entity.</p>

<p>Once we understand the application domain, we can set realistic bounds. Thousands of pizza varieties in the menu would be impractical from a business perspective well before hitting database limits, so storing the varieties together can be acceptable. When object-oriented applications use richer structures than two-dimensional tables, it's better to represent such lists as arrays rather than comma-separated strings:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight javascript"><code><span class="p">{</span>
  <span class="nl">name</span><span class="p">:</span> <span class="dl">"</span><span class="s2">A1 Pizza</span><span class="dl">"</span><span class="p">,</span>
  <span class="nx">manager</span><span class="p">:</span> <span class="dl">"</span><span class="s2">Bob</span><span class="dl">"</span><span class="p">,</span>
  <span class="nx">email</span><span class="p">:</span> <span class="dl">"</span><span class="s2">bob@a1-pizza.it</span><span class="dl">"</span><span class="p">,</span>
  <span class="nx">varieties</span><span class="p">:</span> <span class="p">[</span><span class="dl">"</span><span class="s2">Thick Crust</span><span class="dl">"</span><span class="p">,</span> <span class="dl">"</span><span class="s2">Stuffed Crust</span><span class="dl">"</span><span class="p">]</span>
<span class="p">}</span>
</code></pre>

</div>



<p>Arrays of atomic values satisfy a document-oriented equivalent of 1NF—each element is atomic and independently addressable—even though the document model isn't bound by the relational requirement of flat tuples. While SQL databases provide abstraction and logical-physical data independence, MongoDB keeps data colocated down to the storage and CPU caches for more predictable performance.</p>

<p>Normal form definitions assume keys for each 1NF relation. In a document model, multiple relations can appear as embedded sub-documents or arrays. Treating the parent key and the array element together as a composite key lets us apply higher normal forms to analyze partial and transitive dependencies within a single document.</p>

<h2>
  
  
  2NF: Pizza Pricing
</h2>

<p>We want to add the price of the pizzas to our database. If each pizzeria defines its own base price, it can be added to the varieties items:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight javascript"><code><span class="p">{</span>
  <span class="nl">name</span><span class="p">:</span> <span class="dl">"</span><span class="s2">A1 Pizza</span><span class="dl">"</span><span class="p">,</span>
  <span class="nx">manager</span><span class="p">:</span> <span class="dl">"</span><span class="s2">Bob</span><span class="dl">"</span><span class="p">,</span>
  <span class="nx">email</span><span class="p">:</span> <span class="dl">"</span><span class="s2">bob@a1-pizza.it</span><span class="dl">"</span><span class="p">,</span>
  <span class="nx">varieties</span><span class="p">:</span> <span class="p">[</span>
    <span class="p">{</span> <span class="na">name</span><span class="p">:</span> <span class="dl">"</span><span class="s2">Thick Crust</span><span class="dl">"</span><span class="p">,</span> <span class="na">basePrice</span><span class="p">:</span> <span class="mi">10</span> <span class="p">},</span>
    <span class="p">{</span> <span class="na">name</span><span class="p">:</span> <span class="dl">"</span><span class="s2">Stuffed Crust</span><span class="dl">"</span><span class="p">,</span> <span class="na">basePrice</span><span class="p">:</span> <span class="mi">12</span> <span class="p">}</span>
  <span class="p">]</span>
<span class="p">}</span>
</code></pre>

</div>



<p>Second Normal Form (2NF) builds on 1NF by requiring that every non-key attribute depends on the entire primary key, not just part of it. This only becomes relevant when dealing with composite keys.</p>

<p>In our embedded model, consider the composite key ("pizzeria", "variety") for each item in the varieties array. If the price depends on the pizzeria and variety together—meaning different pizzerias can set different prices for the same variety—then "basePrice" depends on the full composite key, and we satisfy 2NF.</p>

<p>However, if prices are standardized across all pizzerias—the same variety costs the same everywhere—then a partial dependency exists: "basePrice" depends only on "variety", not on the full ("pizzeria", "variety") key. This violates 2NF.</p>

<p>To resolve this, we define pricing in a separate collection where the base price depends only on the pizza variety:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight javascript"><code><span class="p">{</span> <span class="nl">variety</span><span class="p">:</span> <span class="dl">"</span><span class="s2">Thick Crust</span><span class="dl">"</span><span class="p">,</span> <span class="nx">basePrice</span><span class="p">:</span> <span class="mi">10</span> <span class="p">}</span>
<span class="p">{</span> <span class="na">variety</span><span class="p">:</span> <span class="dl">"</span><span class="s2">Stuffed Crust</span><span class="dl">"</span><span class="p">,</span> <span class="na">basePrice</span><span class="p">:</span> <span class="mi">12</span> <span class="p">}</span>
</code></pre>

</div>



<p>We can remove the base price from the pizzeria's varieties array and retrieve it from the pricing collection at query time:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight javascript"><code><span class="nx">db</span><span class="p">.</span><span class="nf">createView</span><span class="p">(</span>
  <span class="dl">"</span><span class="s2">pizzeriasWithPrices</span><span class="dl">"</span><span class="p">,</span>
  <span class="dl">"</span><span class="s2">pizzerias</span><span class="dl">"</span><span class="p">,</span>
  <span class="p">[</span>
    <span class="p">{</span> <span class="na">$unwind</span><span class="p">:</span> <span class="dl">"</span><span class="s2">$varieties</span><span class="dl">"</span> <span class="p">},</span>
    <span class="p">{</span>
      <span class="na">$lookup</span><span class="p">:</span> <span class="p">{</span>
        <span class="na">from</span><span class="p">:</span> <span class="dl">"</span><span class="s2">pricing</span><span class="dl">"</span><span class="p">,</span>
        <span class="na">localField</span><span class="p">:</span> <span class="dl">"</span><span class="s2">varieties.name</span><span class="dl">"</span><span class="p">,</span>
        <span class="na">foreignField</span><span class="p">:</span> <span class="dl">"</span><span class="s2">variety</span><span class="dl">"</span><span class="p">,</span>
        <span class="na">as</span><span class="p">:</span> <span class="dl">"</span><span class="s2">priceInfo</span><span class="dl">"</span>
      <span class="p">}</span>
    <span class="p">},</span>
    <span class="p">{</span> <span class="na">$unwind</span><span class="p">:</span> <span class="dl">"</span><span class="s2">$priceInfo</span><span class="dl">"</span> <span class="p">},</span>
    <span class="p">{</span> <span class="na">$addFields</span><span class="p">:</span> <span class="p">{</span> <span class="dl">"</span><span class="s2">varieties.basePrice</span><span class="dl">"</span><span class="p">:</span> <span class="dl">"</span><span class="s2">$priceInfo.basePrice</span><span class="dl">"</span> <span class="p">}</span> <span class="p">},</span>
    <span class="p">{</span> <span class="na">$project</span><span class="p">:</span> <span class="p">{</span> <span class="na">priceInfo</span><span class="p">:</span> <span class="mi">0</span> <span class="p">}</span> <span class="p">}</span>
  <span class="p">]</span>
<span class="p">);</span>
</code></pre>

</div>



<p>Alternatively, we can use the pricing collection as a reference, where the application retrieves the price and stores it in the pizzeria document for faster reads.</p>

<p>To avoid update anomalies, the application updates all affected documents when a variety's price changes:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight javascript"><code><span class="kd">const</span> <span class="nx">session</span> <span class="o">=</span> <span class="nx">db</span><span class="p">.</span><span class="nf">getMongo</span><span class="p">().</span><span class="nf">startSession</span><span class="p">();</span>
<span class="kd">const</span> <span class="nx">sessionDB</span> <span class="o">=</span> <span class="nx">session</span><span class="p">.</span><span class="nf">getDatabase</span><span class="p">(</span><span class="nx">db</span><span class="p">.</span><span class="nf">getName</span><span class="p">());</span>
<span class="nx">session</span><span class="p">.</span><span class="nf">startTransaction</span><span class="p">();</span>

<span class="nx">sessionDB</span><span class="p">.</span><span class="nf">getCollection</span><span class="p">(</span><span class="dl">"</span><span class="s2">pricing</span><span class="dl">"</span><span class="p">).</span><span class="nf">updateOne</span><span class="p">(</span>
  <span class="p">{</span> <span class="na">variety</span><span class="p">:</span> <span class="dl">"</span><span class="s2">Thick Crust</span><span class="dl">"</span> <span class="p">},</span>
  <span class="p">{</span> <span class="na">$set</span><span class="p">:</span> <span class="p">{</span> <span class="na">basePrice</span><span class="p">:</span> <span class="mi">11</span> <span class="p">}</span> <span class="p">}</span>
<span class="p">);</span>

<span class="nx">sessionDB</span><span class="p">.</span><span class="nf">getCollection</span><span class="p">(</span><span class="dl">"</span><span class="s2">pizzerias</span><span class="dl">"</span><span class="p">).</span><span class="nf">updateMany</span><span class="p">(</span>
  <span class="p">{</span> <span class="dl">"</span><span class="s2">varieties.name</span><span class="dl">"</span><span class="p">:</span> <span class="dl">"</span><span class="s2">Thick Crust</span><span class="dl">"</span> <span class="p">},</span>
  <span class="p">{</span> <span class="na">$set</span><span class="p">:</span> <span class="p">{</span> <span class="dl">"</span><span class="s2">varieties.$[v].basePrice</span><span class="dl">"</span><span class="p">:</span> <span class="mi">11</span> <span class="p">}</span> <span class="p">},</span>
  <span class="p">{</span> <span class="na">arrayFilters</span><span class="p">:</span> <span class="p">[{</span> <span class="dl">"</span><span class="s2">v.name</span><span class="dl">"</span><span class="p">:</span> <span class="dl">"</span><span class="s2">Thick Crust</span><span class="dl">"</span> <span class="p">}]</span> <span class="p">}</span>
<span class="p">);</span>

<span class="nx">session</span><span class="p">.</span><span class="nf">commitTransaction</span><span class="p">();</span>
</code></pre>

</div>



<p>SQL databases avoid such multiple updates because they're designed for direct end-user access, sometimes bypassing the application layer. Without applying normal forms to break dependencies into multiple tables, there's a risk of overlooking replicated data. A document database is updated by an application service responsible for maintaining consistency.</p>

<p>While normalizing to 2NF is possible, it may not always be the best choice in a domain-driven design. Keeping the price embedded in each pizzeria allows asynchronous updates and supports future requirements where some pizzerias may offer different prices—without breaking integrity, as the application enforces updates atomically.</p>

<p>In practice, many applications accept this controlled duplication when price changes are infrequent and prefer fast single-document reads over perfectly normalized writes.</p>

<h2>
  
  
  3NF: Manager's Contacts
</h2>

<p>When we started, each pizzeria had a single email contact:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight javascript"><code><span class="p">{</span>
  <span class="nl">name</span><span class="p">:</span> <span class="dl">"</span><span class="s2">A1 Pizza</span><span class="dl">"</span><span class="p">,</span>
  <span class="nx">manager</span><span class="p">:</span> <span class="dl">"</span><span class="s2">Bob</span><span class="dl">"</span><span class="p">,</span>
  <span class="nx">email</span><span class="p">:</span> <span class="dl">"</span><span class="s2">bob@a1-pizza.it</span><span class="dl">"</span><span class="p">,</span>
  <span class="nx">varieties</span><span class="p">:</span> <span class="p">[</span>
    <span class="p">{</span> <span class="na">name</span><span class="p">:</span> <span class="dl">"</span><span class="s2">Thick Crust</span><span class="dl">"</span><span class="p">,</span> <span class="na">basePrice</span><span class="p">:</span> <span class="mi">10</span> <span class="p">},</span>
    <span class="p">{</span> <span class="na">name</span><span class="p">:</span> <span class="dl">"</span><span class="s2">Stuffed Crust</span><span class="dl">"</span><span class="p">,</span> <span class="na">basePrice</span><span class="p">:</span> <span class="mi">12</span> <span class="p">}</span>
  <span class="p">]</span>
<span class="p">}</span>
</code></pre>

</div>



<p>Third Normal Form (3NF) builds on 2NF by requiring that non-key attributes depend only on the primary key, not on other non-key attributes. When a non-key attribute depends on another non-key attribute, we have a transitive dependency.</p>

<p>Here, the email actually belongs to the manager, not the pizzeria directly. This creates a transitive dependency: "pizzeria" → "manager" → "email". Since "email" depends on "manager" (a non-key attribute) rather than directly on the pizzeria, this violates 3NF.</p>

<p>We can normalize this by grouping the manager's attributes into an embedded subdocument:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight javascript"><code><span class="p">{</span>
  <span class="nl">name</span><span class="p">:</span> <span class="dl">"</span><span class="s2">A1 Pizza</span><span class="dl">"</span><span class="p">,</span>
  <span class="nx">manager</span><span class="p">:</span> <span class="p">{</span> <span class="nl">name</span><span class="p">:</span> <span class="dl">"</span><span class="s2">Bob</span><span class="dl">"</span><span class="p">,</span> <span class="nx">email</span><span class="p">:</span> <span class="dl">"</span><span class="s2">bob@a1-pizza.it</span><span class="dl">"</span> <span class="p">},</span>
  <span class="nx">varieties</span><span class="p">:</span> <span class="p">[</span>
    <span class="p">{</span> <span class="na">name</span><span class="p">:</span> <span class="dl">"</span><span class="s2">Thick Crust</span><span class="dl">"</span><span class="p">,</span> <span class="na">basePrice</span><span class="p">:</span> <span class="mi">10</span> <span class="p">},</span>
    <span class="p">{</span> <span class="na">name</span><span class="p">:</span> <span class="dl">"</span><span class="s2">Stuffed Crust</span><span class="dl">"</span><span class="p">,</span> <span class="na">basePrice</span><span class="p">:</span> <span class="mi">12</span> <span class="p">}</span>
  <span class="p">]</span>
<span class="p">}</span>
</code></pre>

</div>



<p>Now the email is clearly an attribute of the manager entity embedded within the pizzeria. If a pizzeria has multiple managers, we can simply use an array of subdocuments without creating new collections or changing index definitions.</p>

<p>A generic relational model would probably split this into multiple tables, with manager being a foreign key to a "contacts" table. However, in our business domain, we don't manage contacts outside of pizzerias. Even if the same person manages multiple pizzerias, they're recorded as separate manager entries. Bob may have multiple emails and use different ones for each of his pizzerias.</p>

<h2>
  
  
  4NF: Delivery Areas
</h2>

<p>We want to record the areas where a pizzeria can deliver its pizza varieties:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight javascript"><code><span class="p">{</span>
  <span class="nl">name</span><span class="p">:</span> <span class="dl">"</span><span class="s2">A1 Pizza</span><span class="dl">"</span><span class="p">,</span>
  <span class="nx">manager</span><span class="p">:</span> <span class="p">{</span> <span class="nl">name</span><span class="p">:</span> <span class="dl">"</span><span class="s2">Bob</span><span class="dl">"</span><span class="p">,</span> <span class="nx">email</span><span class="p">:</span> <span class="dl">"</span><span class="s2">bob@a1-pizza.it</span><span class="dl">"</span> <span class="p">},</span>
  <span class="nx">offerings</span><span class="p">:</span> <span class="p">[</span>
    <span class="p">{</span> <span class="na">variety</span><span class="p">:</span> <span class="p">{</span> <span class="na">name</span><span class="p">:</span> <span class="dl">"</span><span class="s2">Thick Crust</span><span class="dl">"</span><span class="p">,</span> <span class="na">basePrice</span><span class="p">:</span> <span class="mi">10</span> <span class="p">},</span> <span class="na">area</span><span class="p">:</span> <span class="dl">"</span><span class="s2">Springfield</span><span class="dl">"</span> <span class="p">},</span>
    <span class="p">{</span> <span class="na">variety</span><span class="p">:</span> <span class="p">{</span> <span class="na">name</span><span class="p">:</span> <span class="dl">"</span><span class="s2">Thick Crust</span><span class="dl">"</span><span class="p">,</span> <span class="na">basePrice</span><span class="p">:</span> <span class="mi">10</span> <span class="p">},</span> <span class="na">area</span><span class="p">:</span> <span class="dl">"</span><span class="s2">Shelbyville</span><span class="dl">"</span> <span class="p">}</span>
  <span class="p">]</span>
<span class="p">}</span>
</code></pre>

</div>



<p>Fourth Normal Form (4NF) addresses multi-valued dependencies. A multi-valued dependency exists when one attribute determines a set of values for another attribute, independent of all other attributes. 4NF requires that a relation have no non-trivial multi-valued dependencies except on superkeys.</p>

<p>If varieties and areas were dependent—for example, if certain varieties were only available in certain areas—then storing ("variety", "area") combinations would represent a single multi-valued fact, and there would be no 4NF violation.</p>

<p>However, since our pizzerias deliver all varieties to all areas, these are independent multi-valued dependencies: "pizzeria" →→ "variety" and "pizzeria →→ area". Storing all combinations creates redundancy—if we add a new area, we must add entries for every variety.</p>

<p>We normalize by storing each independent fact in a separate array:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight javascript"><code><span class="p">{</span>
  <span class="nl">name</span><span class="p">:</span> <span class="dl">"</span><span class="s2">A1 Pizza</span><span class="dl">"</span><span class="p">,</span>
  <span class="nx">manager</span><span class="p">:</span> <span class="p">{</span> <span class="nl">name</span><span class="p">:</span> <span class="dl">"</span><span class="s2">Bob</span><span class="dl">"</span><span class="p">,</span> <span class="nx">email</span><span class="p">:</span> <span class="dl">"</span><span class="s2">bob@a1-pizza.it</span><span class="dl">"</span> <span class="p">},</span>
  <span class="nx">varieties</span><span class="p">:</span> <span class="p">[</span>
    <span class="p">{</span> <span class="na">name</span><span class="p">:</span> <span class="dl">"</span><span class="s2">Thick Crust</span><span class="dl">"</span><span class="p">,</span> <span class="na">basePrice</span><span class="p">:</span> <span class="mi">10</span> <span class="p">},</span>
    <span class="p">{</span> <span class="na">name</span><span class="p">:</span> <span class="dl">"</span><span class="s2">Stuffed Crust</span><span class="dl">"</span><span class="p">,</span> <span class="na">basePrice</span><span class="p">:</span> <span class="mi">12</span> <span class="p">}</span>
  <span class="p">],</span>
  <span class="nx">deliveryAreas</span><span class="p">:</span> <span class="p">[</span><span class="dl">"</span><span class="s2">Springfield</span><span class="dl">"</span><span class="p">,</span> <span class="dl">"</span><span class="s2">Shelbyville</span><span class="dl">"</span><span class="p">]</span>
<span class="p">}</span>
</code></pre>

</div>



<p>With this schema, we avoid violating 4NF because delivery areas and varieties are stored independently—even though the document model allows us to embed them together.</p>

<h2>
  
  
  BCNF: Per-Area Pricing
</h2>

<p>Our network grows further. Some pizzerias now charge different prices depending on the delivery area—distant areas cost more:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight javascript"><code><span class="p">{</span>
  <span class="nl">name</span><span class="p">:</span> <span class="dl">"</span><span class="s2">A1 Pizza</span><span class="dl">"</span><span class="p">,</span>
  <span class="nx">manager</span><span class="p">:</span> <span class="p">{</span> <span class="nl">name</span><span class="p">:</span> <span class="dl">"</span><span class="s2">Bob</span><span class="dl">"</span><span class="p">,</span> <span class="nx">email</span><span class="p">:</span> <span class="dl">"</span><span class="s2">bob@a1-pizza.it</span><span class="dl">"</span> <span class="p">},</span>
  <span class="nx">offerings</span><span class="p">:</span> <span class="p">[</span>
    <span class="p">{</span> <span class="na">variety</span><span class="p">:</span> <span class="dl">"</span><span class="s2">Thick Crust</span><span class="dl">"</span><span class="p">,</span> <span class="na">area</span><span class="p">:</span> <span class="dl">"</span><span class="s2">Springfield</span><span class="dl">"</span><span class="p">,</span> <span class="na">price</span><span class="p">:</span> <span class="mi">10</span> <span class="p">},</span>
    <span class="p">{</span> <span class="na">variety</span><span class="p">:</span> <span class="dl">"</span><span class="s2">Thick Crust</span><span class="dl">"</span><span class="p">,</span> <span class="na">area</span><span class="p">:</span> <span class="dl">"</span><span class="s2">Shelbyville</span><span class="dl">"</span><span class="p">,</span> <span class="na">price</span><span class="p">:</span> <span class="mi">11</span> <span class="p">},</span>
    <span class="p">{</span> <span class="na">variety</span><span class="p">:</span> <span class="dl">"</span><span class="s2">Stuffed Crust</span><span class="dl">"</span><span class="p">,</span> <span class="na">area</span><span class="p">:</span> <span class="dl">"</span><span class="s2">Springfield</span><span class="dl">"</span><span class="p">,</span> <span class="na">price</span><span class="p">:</span> <span class="mi">12</span> <span class="p">},</span>
    <span class="p">{</span> <span class="na">variety</span><span class="p">:</span> <span class="dl">"</span><span class="s2">Stuffed Crust</span><span class="dl">"</span><span class="p">,</span> <span class="na">area</span><span class="p">:</span> <span class="dl">"</span><span class="s2">Shelbyville</span><span class="dl">"</span><span class="p">,</span> <span class="na">price</span><span class="p">:</span> <span class="mi">13</span> <span class="p">}</span>
  <span class="p">]</span>
<span class="p">}</span>
</code></pre>

</div>



<p>The composite key for each offering is ("pizzeria", "variety", "area"). The price depends on the full key, satisfying 2NF and 3NF.</p>

<p>Now our franchise assigns an area manager to each area—one manager per area, regardless of pizzeria. We add it to our offerings:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight javascript"><code><span class="nx">offerings</span><span class="p">:</span> <span class="p">[</span>
  <span class="p">{</span> <span class="na">variety</span><span class="p">:</span> <span class="dl">"</span><span class="s2">Thick Crust</span><span class="dl">"</span><span class="p">,</span> <span class="na">area</span><span class="p">:</span> <span class="dl">"</span><span class="s2">Springfield</span><span class="dl">"</span><span class="p">,</span> <span class="na">price</span><span class="p">:</span> <span class="mi">10</span><span class="p">,</span> <span class="na">areaManager</span><span class="p">:</span> <span class="dl">"</span><span class="s2">Alice</span><span class="dl">"</span> <span class="p">},</span>
  <span class="p">{</span> <span class="na">variety</span><span class="p">:</span> <span class="dl">"</span><span class="s2">Stuffed Crust</span><span class="dl">"</span><span class="p">,</span> <span class="na">area</span><span class="p">:</span> <span class="dl">"</span><span class="s2">Springfield</span><span class="dl">"</span><span class="p">,</span> <span class="na">price</span><span class="p">:</span> <span class="mi">12</span><span class="p">,</span> <span class="na">areaManager</span><span class="p">:</span> <span class="dl">"</span><span class="s2">Alice</span><span class="dl">"</span> <span class="p">},</span>
  <span class="p">{</span> <span class="na">variety</span><span class="p">:</span> <span class="dl">"</span><span class="s2">Thick Crust</span><span class="dl">"</span><span class="p">,</span> <span class="na">area</span><span class="p">:</span> <span class="dl">"</span><span class="s2">Shelbyville</span><span class="dl">"</span><span class="p">,</span> <span class="na">price</span><span class="p">:</span> <span class="mi">11</span><span class="p">,</span> <span class="na">areaManager</span><span class="p">:</span> <span class="dl">"</span><span class="s2">Eve</span><span class="dl">"</span> <span class="p">},</span>
  <span class="p">{</span> <span class="na">variety</span><span class="p">:</span> <span class="dl">"</span><span class="s2">Stuffed Crust</span><span class="dl">"</span><span class="p">,</span> <span class="na">area</span><span class="p">:</span> <span class="dl">"</span><span class="s2">Shelbyville</span><span class="dl">"</span><span class="p">,</span> <span class="na">price</span><span class="p">:</span> <span class="mi">13</span><span class="p">,</span> <span class="na">areaManager</span><span class="p">:</span> <span class="dl">"</span><span class="s2">Eve</span><span class="dl">"</span> <span class="p">}</span>
<span class="p">]</span>
</code></pre>

</div>



<p>Boyce-Codd Normal Form (BCNF) is a stricter version of 3NF. It requires that for every non-trivial functional dependency <code>X → Y</code>, the determinant <code>X</code> must be a superkey. Unlike 3NF, BCNF doesn't make an exception for dependencies where the dependent attribute is part of a candidate key.</p>

<p>This model passes 3NF but fails BCNF: the dependency "area" → "areaManager" has a determinant ("area") that is not a superkey of the offerings relation. The area alone doesn't uniquely identify an offering—you need the full ("pizzeria", "variety", "area") key for that.</p>

<p>The practical problem: if Alice is replaced by Carol for Springfield, we must update every offering for that area across every pizzeria. The relational solution is to extract area managers to a separate table.</p>

<p>In MongoDB, we can keep the embedded structure and handle updates explicitly:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight javascript"><code><span class="nx">db</span><span class="p">.</span><span class="nx">pizzerias</span><span class="p">.</span><span class="nf">updateMany</span><span class="p">(</span>
  <span class="p">{</span> <span class="dl">"</span><span class="s2">offerings.area</span><span class="dl">"</span><span class="p">:</span> <span class="dl">"</span><span class="s2">Springfield</span><span class="dl">"</span> <span class="p">},</span>
  <span class="p">{</span> <span class="na">$set</span><span class="p">:</span> <span class="p">{</span> <span class="dl">"</span><span class="s2">offerings.$[o].areaManager</span><span class="dl">"</span><span class="p">:</span> <span class="dl">"</span><span class="s2">Carol</span><span class="dl">"</span> <span class="p">}</span> <span class="p">},</span>
  <span class="p">{</span> <span class="na">arrayFilters</span><span class="p">:</span> <span class="p">[{</span> <span class="dl">"</span><span class="s2">o.area</span><span class="dl">"</span><span class="p">:</span> <span class="dl">"</span><span class="s2">Springfield</span><span class="dl">"</span> <span class="p">}]</span> <span class="p">}</span>
<span class="p">)</span>
</code></pre>

</div>



<p>This trades strict BCNF compliance for simpler queries and faster reads. The application ensures consistency during updates.</p>

<h2>
  
  
  5NF: Adding Pizza Sizes
</h2>

<p>We now offer multiple sizes (Small, Medium, Large). Sizes, varieties, and delivery areas are all independent—any combination is valid.</p>

<p>Storing every combination explodes quickly:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight javascript"><code><span class="nx">offerings</span><span class="p">:</span> <span class="p">[</span>
  <span class="p">{</span> <span class="na">variety</span><span class="p">:</span> <span class="dl">"</span><span class="s2">Thick Crust</span><span class="dl">"</span><span class="p">,</span> <span class="na">size</span><span class="p">:</span> <span class="dl">"</span><span class="s2">Large</span><span class="dl">"</span><span class="p">,</span> <span class="na">area</span><span class="p">:</span> <span class="dl">"</span><span class="s2">Springfield</span><span class="dl">"</span> <span class="p">},</span>
  <span class="p">{</span> <span class="na">variety</span><span class="p">:</span> <span class="dl">"</span><span class="s2">Thick Crust</span><span class="dl">"</span><span class="p">,</span> <span class="na">size</span><span class="p">:</span> <span class="dl">"</span><span class="s2">Large</span><span class="dl">"</span><span class="p">,</span> <span class="na">area</span><span class="p">:</span> <span class="dl">"</span><span class="s2">Shelbyville</span><span class="dl">"</span> <span class="p">},</span>
  <span class="p">{</span> <span class="na">variety</span><span class="p">:</span> <span class="dl">"</span><span class="s2">Thick Crust</span><span class="dl">"</span><span class="p">,</span> <span class="na">size</span><span class="p">:</span> <span class="dl">"</span><span class="s2">Medium</span><span class="dl">"</span><span class="p">,</span> <span class="na">area</span><span class="p">:</span> <span class="dl">"</span><span class="s2">Springfield</span><span class="dl">"</span> <span class="p">},</span>
  <span class="c1">// ... 150 entries for 5 varieties × 3 sizes × 10 areas</span>
<span class="p">]</span>
</code></pre>

</div>



<p>Fifth Normal Form (5NF), also called Project-Join Normal Form, addresses join dependencies. A relation is in 5NF if it cannot be decomposed into smaller relations that, when joined, reconstruct the original—without losing information or introducing spurious tuples.</p>

<p>When valid combinations can be reconstructed from independent sets (the Cartesian product of varieties, sizes, and areas), storing all combinations explicitly creates redundancy and risks inconsistency. This violates 5NF.</p>

<p>The fix stores each independent fact separately:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight javascript"><code><span class="p">{</span>
  <span class="nl">name</span><span class="p">:</span> <span class="dl">"</span><span class="s2">A1 Pizza</span><span class="dl">"</span><span class="p">,</span>
  <span class="nx">varieties</span><span class="p">:</span> <span class="p">[</span><span class="dl">"</span><span class="s2">Thick Crust</span><span class="dl">"</span><span class="p">,</span> <span class="dl">"</span><span class="s2">Stuffed Crust</span><span class="dl">"</span><span class="p">],</span>
  <span class="nx">sizes</span><span class="p">:</span> <span class="p">[</span><span class="dl">"</span><span class="s2">Large</span><span class="dl">"</span><span class="p">,</span> <span class="dl">"</span><span class="s2">Medium</span><span class="dl">"</span><span class="p">],</span>
  <span class="nx">deliveryAreas</span><span class="p">:</span> <span class="p">[</span><span class="dl">"</span><span class="s2">Springfield</span><span class="dl">"</span><span class="p">,</span> <span class="dl">"</span><span class="s2">Shelbyville</span><span class="dl">"</span><span class="p">]</span>
<span class="p">}</span>
</code></pre>

</div>



<p>Adding a new size requires updating one array—not hundreds of entries. The application or query logic reconstructs valid combinations when needed.</p>

<h2>
  
  
  6NF: Tracking Price History
</h2>

<p>Our finance team needs to track price changes over time. We could embed the history:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight javascript"><code><span class="nx">offerings</span><span class="p">:</span> <span class="p">[</span>
  <span class="p">{</span>
    <span class="na">variety</span><span class="p">:</span> <span class="dl">"</span><span class="s2">Thick Crust</span><span class="dl">"</span><span class="p">,</span>
    <span class="na">area</span><span class="p">:</span> <span class="dl">"</span><span class="s2">Springfield</span><span class="dl">"</span><span class="p">,</span>
    <span class="na">currentPrice</span><span class="p">:</span> <span class="mi">12</span><span class="p">,</span>
    <span class="na">priceHistory</span><span class="p">:</span> <span class="p">[</span>
      <span class="p">{</span> <span class="na">price</span><span class="p">:</span> <span class="mi">10</span><span class="p">,</span> <span class="na">effectiveDate</span><span class="p">:</span> <span class="nc">ISODate</span><span class="p">(</span><span class="dl">"</span><span class="s2">2024-01-01</span><span class="dl">"</span><span class="p">)</span> <span class="p">},</span>
      <span class="p">{</span> <span class="na">price</span><span class="p">:</span> <span class="mi">11</span><span class="p">,</span> <span class="na">effectiveDate</span><span class="p">:</span> <span class="nc">ISODate</span><span class="p">(</span><span class="dl">"</span><span class="s2">2024-03-15</span><span class="dl">"</span><span class="p">)</span> <span class="p">},</span>
      <span class="p">{</span> <span class="na">price</span><span class="p">:</span> <span class="mi">12</span><span class="p">,</span> <span class="na">effectiveDate</span><span class="p">:</span> <span class="nc">ISODate</span><span class="p">(</span><span class="dl">"</span><span class="s2">2024-06-01</span><span class="dl">"</span><span class="p">)</span> <span class="p">}</span>
    <span class="p">]</span>
  <span class="p">}</span>
<span class="p">]</span>
</code></pre>

</div>



<p>This works for moderate history but grows unboundedly over time.</p>

<p>Sixth Normal Form (6NF) decomposes relations so that each stores a single non-key attribute along with its time dimension. Every row represents one fact at one point in time:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight javascript"><code><span class="c1">// price_history collection</span>
<span class="p">{</span> <span class="nl">pizzeria</span><span class="p">:</span> <span class="dl">"</span><span class="s2">A1 Pizza</span><span class="dl">"</span><span class="p">,</span> <span class="nx">variety</span><span class="p">:</span> <span class="dl">"</span><span class="s2">Thick Crust</span><span class="dl">"</span><span class="p">,</span> <span class="nx">area</span><span class="p">:</span> <span class="dl">"</span><span class="s2">Springfield</span><span class="dl">"</span><span class="p">,</span> <span class="nx">price</span><span class="p">:</span> <span class="mi">10</span><span class="p">,</span> <span class="nx">effectiveDate</span><span class="p">:</span> <span class="nc">ISODate</span><span class="p">(</span><span class="dl">"</span><span class="s2">2024-01-01</span><span class="dl">"</span><span class="p">)</span> <span class="p">}</span>
<span class="p">{</span> <span class="na">pizzeria</span><span class="p">:</span> <span class="dl">"</span><span class="s2">A1 Pizza</span><span class="dl">"</span><span class="p">,</span> <span class="na">variety</span><span class="p">:</span> <span class="dl">"</span><span class="s2">Thick Crust</span><span class="dl">"</span><span class="p">,</span> <span class="na">area</span><span class="p">:</span> <span class="dl">"</span><span class="s2">Springfield</span><span class="dl">"</span><span class="p">,</span> <span class="na">price</span><span class="p">:</span> <span class="mi">11</span><span class="p">,</span> <span class="na">effectiveDate</span><span class="p">:</span> <span class="nc">ISODate</span><span class="p">(</span><span class="dl">"</span><span class="s2">2024-03-15</span><span class="dl">"</span><span class="p">)</span> <span class="p">}</span>
<span class="p">{</span> <span class="na">pizzeria</span><span class="p">:</span> <span class="dl">"</span><span class="s2">A1 Pizza</span><span class="dl">"</span><span class="p">,</span> <span class="na">variety</span><span class="p">:</span> <span class="dl">"</span><span class="s2">Thick Crust</span><span class="dl">"</span><span class="p">,</span> <span class="na">area</span><span class="p">:</span> <span class="dl">"</span><span class="s2">Springfield</span><span class="dl">"</span><span class="p">,</span> <span class="na">price</span><span class="p">:</span> <span class="mi">12</span><span class="p">,</span> <span class="na">effectiveDate</span><span class="p">:</span> <span class="nc">ISODate</span><span class="p">(</span><span class="dl">"</span><span class="s2">2024-06-01</span><span class="dl">"</span><span class="p">)</span> <span class="p">}</span>
</code></pre>

</div>



<p>6NF is rarely used for operational data because it requires extensive joins for common queries. However, for auditing, analytics, and temporal queries—where you need to answer "what was the price on March 10th?"—it provides a clean model for tracking changes over time.</p>

<h2>
  
  
  Summary
</h2>

<p>Normal forms are not a relic of relational theory. They describe fundamental data dependencies present in any system, regardless of storage technology. MongoDB’s document model does not remove the need to consider normalization. Instead, it lets you decide where, when, and how strictly to apply it, based on domain boundaries and access patterns.</p>

<p>In relational/SQL databases, schemas are usually designed as enterprise-wide information models. Many applications and users share the same database, accepting ad hoc SQL. To avoid update, insertion, and deletion anomalies in this shared environment, the schema must enforce functional dependencies, making higher normal forms essential. Because the database is the system of record, normalization centralizes integrity rules in the data model.</p>

<p>Modern architectures, by contrast, often follow Domain-Driven Design (DDD). Each bounded context owns its data model, which evolves with the application. With CQRS and microservices, each aggregate is updated only through a single application service that encapsulates business rules. Here, the database is not a shared integration point but a private persistence detail of the service.</p>

<p>MongoDB fits this style well:</p>

<ul>
<li>Documents model aggregates as they exist in the domain</li>
<li>Arrays capture bounded one-to-many relationships</li>
<li>Denormalization and controlled duplication improve read performance and scalability</li>
<li>Consistency is enforced by application logic, not global database constraints</li>
</ul>

<p>Because one service owns all updates, violating higher normal forms can be acceptable—and sometimes beneficial—provided the service preserves its invariants. Normalization becomes a design tool, not a rigid checklist.</p>

<p>In short:</p>

<ul>
<li>Use relational normalization when the database is a shared, queryable system of record accessed by many applications and users via SQL.</li>
<li>Use document modeling with selective denormalization when building domain-aligned services with clear ownership, CQRS, and microservices.</li>
</ul>

<p>Normal forms still matter—but in MongoDB, they guide your choices instead of dictating your schema.</p>

