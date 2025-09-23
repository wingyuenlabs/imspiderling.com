---
Title: Graph-Based Movie Recommendations: A Practical Guide with Python
Description: 
Author: IvanDev
Date: 2025-09-23T21:48:29.000Z
Robots: noindex,nofollow
Template: index
---
<h1>
  
  
  Introduction
</h1>

<p>I built a small Python project where I use a well-known algorithm called BFS (Breadth-First Search), which I employ to recommend movies based on the closest match by genre. The BFS algorithm, for example, searches for movies related within N degrees of separation, for example:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight python"><code><span class="n">movie_graph</span> <span class="o">=</span> <span class="p">{</span>
<span class="sh">"</span><span class="s">The Matrix</span><span class="sh">"</span> <span class="p">[</span><span class="sh">"</span><span class="s">Inception</span><span class="sh">"</span><span class="p">,</span> <span class="sh">"</span><span class="s">Blade Runner</span><span class="sh">"</span><span class="p">,</span> <span class="sh">"</span><span class="s">Dark City</span><span class="sh">"</span><span class="p">],</span>
<span class="sh">"</span><span class="s">Inception</span><span class="sh">"</span><span class="p">:</span> <span class="p">[</span><span class="sh">"</span><span class="s">The Matrix</span><span class="sh">"</span><span class="p">,</span> <span class="sh">"</span><span class="s">Interstellar</span><span class="sh">"</span><span class="p">,</span> <span class="sh">"</span><span class="s">Memento</span><span class="sh">"</span><span class="p">],</span>
<span class="c1"># ... More connections
</span><span class="p">}</span>
</code></pre>

</div>



<p>As you can see or try to understand, The Matrix is very similar to Blade Runner or Dark City (to some extent), and another film in roughly the same neighborhood would be: Inception is similar to or has connections to The Matrix, Interstellar, and, finally, Memento.</p>

<p>For the structure, I went with something basic that’s always used for these small projects, so you can understand the prototypes whether it’s for a big or small idea.</p>

<p>Here we find the README file, the data folder or database in a JSON file, the entry point <code>main.py</code>, and the <code>src</code> directory where we put all the files for the program’s logic.<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight shell"><code>movie_recommender/
├── README.md
├── data
│ └── movies_sample.json
├── main.py └── src
└── src
└── movie_data.py
└── search.py
</code></pre>

</div>



<p>Of course, we need to make our interface as easy and obvious as possible in this case, so the CLI would be the way to go—to keep things simple and not difficult to understand, which is also fun (well, for me anyway :) ).</p>

<p>So we start with a welcome message, just like we would in any game. After that, we ask what to do—that is, we give options. This is usually done in almost any language with a while loop, which is the easiest way to discard options as they’re chosen, and the game or program continues with a series of questions until we’ve answered everything and the program ends.</p>

<p><strong>This is what I mean:</strong><br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight shell"><code>Welcome to Movie Recommender!

Enter a movie title <span class="o">(</span>or <span class="nb">type </span>to search<span class="o">)</span>: inc
Suggestions: <span class="o">[</span><span class="s1">'Inception'</span>, <span class="s1">'The Incredibles'</span>, <span class="s1">'Incendies'</span><span class="o">]</span>

Selected: Inception Inception

What would you like to <span class="k">do</span>?
1. Find similar movies Find similar movies
2. See top-rated titles <span class="k">in </span>the same genre
3. Explore by actor/director Explore by actor/director

Choice: 1

Movies similar to Inception:
- Interstellar <span class="o">(</span>same director, sci-fi<span class="o">)</span>
- The Matrix <span class="o">(</span>Mind-bending, action<span class="o">)</span>
- Memento <span class="o">(</span>Christopher Nolan, thriller<span class="o">)</span>
</code></pre>

</div>



<p>But anyway, just to refresh your memory, the BFS algorithm searches all nodes at one level, and when it’s done with that level, it moves on to the next level until it’s finished. That’s basically it: it visits each node and keeps track of which ones it’s visited until it’s done. Consequently, it uses a Queue (FIFO) data structure → BFS = Goes level by level, meaning the first one in is the first one out, just like a line. Well, this topic is quite extensive, and I’d have to write another article since it takes time to understand how this type of data structure works.</p>

<p><strong>An example</strong></p>

<p>Tree Structure:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight shell"><code>      A
     / <span class="se">\</span>
    C   B
   / <span class="se">\ </span>/ <span class="se">\</span>
  G  F E  D
</code></pre>

</div>



<p>The BFS = [A] Root</p>

<p>First level = [B, C]</p>

<p>Second level = [D, E, F, G]</p>

<p>BFS Order: A → B → C → D → E → F → G</p>

<p>What this highly popular algorithm does is:</p>

<ul>
<li><p>Depth 1: Direct connections (very highly similar movies)</p></li>
<li><p>Depth 2: Extended recommendation (movies similar to other similar movies)</p></li>
</ul>

<p>As you can imagine, this algorithm is basically everyplace on the internet, from movie recommenders to the products we buy on any e-commerce site—like related items—it’s really quite popular.</p>

<p>Okay, all this talk is a shame, my friend. Let’s get down to business—briefly:</p>

<p>First, we create movies_sample.json, which is our small internal or lite database:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight javascript"><code><span class="dl">"</span><span class="s2">movies</span><span class="dl">"</span><span class="p">:</span> <span class="p">[</span>
<span class="dl">"</span><span class="s2">id</span><span class="dl">"</span><span class="p">:</span> <span class="mi">1</span><span class="p">,</span>
<span class="dl">"</span><span class="s2">title</span><span class="dl">"</span><span class="p">:</span> <span class="dl">"</span><span class="s2">Inception</span><span class="dl">"</span><span class="p">,</span>
<span class="dl">"</span><span class="s2">year</span><span class="dl">"</span><span class="p">:</span> <span class="mi">2010</span><span class="p">,</span>
<span class="dl">"</span><span class="s2">genres</span><span class="dl">"</span><span class="p">:</span> <span class="p">[</span><span class="dl">"</span><span class="s2">Sci-Fi</span><span class="dl">"</span><span class="p">,</span> <span class="dl">"</span><span class="s2">Thriller</span><span class="dl">"</span><span class="p">],</span>
<span class="dl">"</span><span class="s2">rating</span><span class="dl">"</span><span class="p">:</span> <span class="mf">8.8</span>
<span class="p">},</span>
<span class="p">{</span> <span class="dl">"</span><span class="s2">id</span><span class="dl">"</span><span class="p">:</span> <span class="mi">2</span><span class="p">,</span> <span class="dl">"</span><span class="s2">English</span><span class="dl">"</span><span class="p">:</span>
<span class="dl">"</span><span class="s2">id</span><span class="dl">"</span><span class="p">:</span> <span class="mi">2</span><span class="p">,</span>
<span class="dl">"</span><span class="s2">title</span><span class="dl">"</span><span class="p">:</span> <span class="dl">"</span><span class="s2">28 Days Later</span><span class="dl">"</span>
<span class="dl">"</span><span class="s2">year</span><span class="dl">"</span><span class="p">:</span> <span class="mi">2002</span><span class="p">,</span>
<span class="dl">"</span><span class="s2">genres</span><span class="dl">"</span><span class="p">:</span> <span class="p">[</span><span class="dl">"</span><span class="s2">Sci-Fi</span><span class="dl">"</span><span class="p">,</span> <span class="dl">"</span><span class="s2">Drama</span><span class="dl">"</span><span class="p">,</span> <span class="dl">"</span><span class="s2">Thriller</span><span class="dl">"</span><span class="p">],</span>
<span class="dl">"</span><span class="s2">rating</span><span class="dl">"</span><span class="p">:</span> <span class="mf">7.3</span>
<span class="p">},</span>

<span class="nx">and</span> <span class="mi">8</span> <span class="nx">more</span>

<span class="p">...</span>
</code></pre>

</div>



<p>After that, we define the MovieDatabase class:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight python"><code><span class="kn">import</span> <span class="n">json</span>

<span class="k">class</span> <span class="nc">MovieDatabase</span><span class="p">:</span>
    <span class="k">def</span> <span class="nf">__init__</span><span class="p">(</span><span class="n">self</span><span class="p">):</span>
        <span class="n">self</span><span class="p">.</span><span class="n">movies</span> <span class="o">=</span> <span class="p">{}</span>  <span class="c1"># id -&gt; movie info
</span>        <span class="n">self</span><span class="p">.</span><span class="n">connections</span> <span class="o">=</span> <span class="p">{}</span>  <span class="c1"># id  -&gt; list of connected ids
</span>
    <span class="k">def</span> <span class="nf">load_data</span><span class="p">(</span><span class="n">self</span><span class="p">,</span> <span class="n">filepath</span><span class="p">):</span>
        <span class="sh">"""</span><span class="s">Load movie data from JSON file</span><span class="sh">"""</span>
        <span class="k">with</span> <span class="nf">open</span><span class="p">(</span><span class="n">filepath</span><span class="p">,</span> <span class="sh">'</span><span class="s">r</span><span class="sh">'</span><span class="p">)</span> <span class="k">as</span> <span class="n">f</span><span class="p">:</span>
            <span class="n">data</span> <span class="o">=</span> <span class="n">json</span><span class="p">.</span><span class="nf">load</span><span class="p">(</span><span class="n">f</span><span class="p">)</span>

            <span class="c1"># Extract movies and store them
</span>            <span class="k">for</span> <span class="n">movie</span> <span class="ow">in</span> <span class="n">data</span><span class="p">[</span><span class="sh">'</span><span class="s">movies</span><span class="sh">'</span><span class="p">]:</span>
                <span class="n">self</span><span class="p">.</span><span class="n">movies</span><span class="p">[</span><span class="n">movie</span><span class="p">[</span><span class="sh">'</span><span class="s">id</span><span class="sh">'</span><span class="p">]]</span> <span class="o">=</span> <span class="n">movie</span> 

            <span class="c1"># Extract connections
</span>            <span class="k">for</span> <span class="n">connection</span> <span class="ow">in</span> <span class="n">data</span><span class="p">[</span><span class="sh">'</span><span class="s">connections</span><span class="sh">'</span><span class="p">]:</span>
                <span class="n">self</span><span class="p">.</span><span class="n">connections</span><span class="p">[</span><span class="n">connection</span><span class="p">[</span><span class="sh">'</span><span class="s">movie_id</span><span class="sh">'</span><span class="p">]]</span> <span class="o">=</span> <span class="n">connection</span><span class="p">[</span><span class="sh">'</span><span class="s">similar_to</span><span class="sh">'</span><span class="p">]</span>

    <span class="k">def</span> <span class="nf">get_movie</span><span class="p">(</span><span class="n">self</span><span class="p">,</span> <span class="n">movie_id</span><span class="p">):</span>
        <span class="sh">"""</span><span class="s">Return movie info by ID</span><span class="sh">"""</span>
        <span class="k">return</span> <span class="n">self</span><span class="p">.</span><span class="n">movies</span><span class="p">.</span><span class="nf">get</span><span class="p">(</span><span class="n">movie_id</span><span class="p">,</span> <span class="bp">None</span><span class="p">)</span>

    <span class="k">def</span> <span class="nf">get_all_movies</span><span class="p">(</span><span class="n">self</span><span class="p">):</span>
        <span class="sh">"""</span><span class="s">Return list of all movies</span><span class="sh">"""</span>
        <span class="k">return</span> <span class="nf">list</span><span class="p">(</span><span class="n">self</span><span class="p">.</span><span class="n">movies</span><span class="p">.</span><span class="nf">values</span><span class="p">())</span>
</code></pre>

</div>



<p>First import json module to extract data, then we initialize the constructor with its variables instantiated. Now that we have the above, we can define methods so that our main class, <code>MovieRecommenderCLI</code>, can interact with them. The first method is for loading data from the JSON file, from which we extract the data so that we can manipulate it once it’s been extracted. The other two methods are the getters <code>get_movie</code> and <code>get_all_movies</code>; as you can see, they are for reading data.<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight python"><code><span class="kn">import</span> <span class="n">json</span>

<span class="k">class</span> <span class="nc">MovieSearcher</span><span class="p">:</span>
    <span class="k">def</span> <span class="nf">__init__</span><span class="p">(</span><span class="n">self</span><span class="p">,</span> <span class="n">database</span><span class="p">):</span>
        <span class="sh">"""</span><span class="s">
        Initialize with a MovieDatabase instance

        Parameters:
            database: MovieDatabase object that contains our movie data
        </span><span class="sh">"""</span>
        <span class="n">self</span><span class="p">.</span><span class="n">db</span> <span class="o">=</span> <span class="n">database</span>
</code></pre>

</div>



<p>In the next step, we will proceed to create the <code>MovieSearcher</code> class. This is where we find several algorithms, such as the <code>search_by_title</code> method:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight python"><code><span class="k">def</span> <span class="nf">search_by_title</span><span class="p">(</span><span class="n">self</span><span class="p">,</span> <span class="n">query</span><span class="p">):</span>
        <span class="sh">"""</span><span class="s">
        Search movies by partial title match (case-insensitive)

        Parameters:
            query: String to search for in movie titles

        Returns:
            List of matching movie dictionaries
        </span><span class="sh">"""</span>
        <span class="n">results</span> <span class="o">=</span> <span class="p">[]</span>
        <span class="n">query_lower</span> <span class="o">=</span> <span class="n">query</span><span class="p">.</span><span class="nf">lower</span><span class="p">()</span>  <span class="c1"># Convert to lowercase for case-insensitive search
</span>
        <span class="c1"># Iterate through all movies in database
</span>        <span class="k">for</span> <span class="n">movie_id</span><span class="p">,</span> <span class="n">movie</span> <span class="ow">in</span> <span class="n">self</span><span class="p">.</span><span class="n">db</span><span class="p">.</span><span class="n">movies</span><span class="p">.</span><span class="nf">items</span><span class="p">():</span>
            <span class="c1"># Check if query appears anywhere in the title
</span>            <span class="k">if</span> <span class="n">query_lower</span> <span class="ow">in</span> <span class="n">movie</span><span class="p">[</span><span class="sh">'</span><span class="s">title</span><span class="sh">'</span><span class="p">].</span><span class="nf">lower</span><span class="p">():</span>
                <span class="n">results</span><span class="p">.</span><span class="nf">append</span><span class="p">(</span><span class="n">movie</span><span class="p">)</span>

        <span class="k">return</span> <span class="n">results</span>
</code></pre>

</div>



<p>In which we convert the query to lowercase, iterate through each movie in the database, check if the query appears in the movie’s title (also converted to lowercase), and add the movies to the results list.<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight python"><code><span class="k">def</span> <span class="nf">search_by_genre</span><span class="p">(</span><span class="n">self</span><span class="p">,</span> <span class="n">genre</span><span class="p">):</span>
        <span class="sh">"""</span><span class="s">
        Find all movies that have a specific genre

        Parameters:
            genre: String genre to search for

        Returns:
            List of movies that contain this genre
        </span><span class="sh">"""</span>
        <span class="n">results</span> <span class="o">=</span> <span class="p">[]</span>
        <span class="n">genre_lower</span> <span class="o">=</span> <span class="n">genre</span><span class="p">.</span><span class="nf">lower</span><span class="p">()</span>

        <span class="k">for</span> <span class="n">movie_id</span><span class="p">,</span> <span class="n">movie</span> <span class="ow">in</span> <span class="n">self</span><span class="p">.</span><span class="n">db</span><span class="p">.</span><span class="n">movies</span><span class="p">.</span><span class="nf">items</span><span class="p">():</span>
            <span class="c1"># Convert all genres to lowercase for comparison
</span>            <span class="n">movie_genres_lower</span> <span class="o">=</span> <span class="p">[</span><span class="n">g</span><span class="p">.</span><span class="nf">lower</span><span class="p">()</span> <span class="k">for</span> <span class="n">g</span> <span class="ow">in</span> <span class="n">movie</span><span class="p">[</span><span class="sh">'</span><span class="s">genres</span><span class="sh">'</span><span class="p">]]</span>

            <span class="c1"># Check if the genre appears in any of the movie's genres
</span>            <span class="k">for</span> <span class="n">movie_genre</span> <span class="ow">in</span> <span class="n">movie_genres_lower</span><span class="p">:</span>
                <span class="k">if</span> <span class="n">genre_lower</span> <span class="ow">in</span> <span class="n">movie_genre</span><span class="p">:</span>  <span class="c1"># Partial match
</span>                    <span class="n">results</span><span class="p">.</span><span class="nf">append</span><span class="p">(</span><span class="n">movie</span><span class="p">)</span>
                    <span class="k">break</span>  <span class="c1"># Don't add the same movie twice
</span>
        <span class="k">return</span> <span class="n">results</span>
</code></pre>

</div>



<p>After this, we continue with the next method, <code>search_by_genre</code>, which does the following: Convert the string parameter <code>genre</code> to lowercase, iterate through each movie, and check all its genres. We use a partial comparison, such as ("Horror" matches "Slasher Horror"), and finally break after finding the first match to prevent duplicates.<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight python"><code><span class="k">def</span> <span class="nf">find_recommendations</span><span class="p">(</span><span class="n">self</span><span class="p">,</span> <span class="n">movie_id</span><span class="p">,</span> <span class="n">depth</span><span class="o">=</span><span class="mi">1</span><span class="p">):</span>
        <span class="sh">"""</span><span class="s">
        Use BFS to find related movies up to a certain depth

        Parameters:
            movie_id: ID of the movie to find recommendations for
            depth: How many </span><span class="sh">"</span><span class="s">hops</span><span class="sh">"</span><span class="s"> away to search (1 = direct connections only)

        Returns:
            List of recommended movies with their connection distance
        </span><span class="sh">"""</span>
        <span class="k">if</span> <span class="n">movie_id</span> <span class="ow">not</span> <span class="ow">in</span> <span class="n">self</span><span class="p">.</span><span class="n">db</span><span class="p">.</span><span class="n">movies</span><span class="p">:</span>
            <span class="k">return</span> <span class="p">[]</span>

        <span class="n">visited</span> <span class="o">=</span> <span class="nf">set</span><span class="p">()</span>  <span class="c1"># Track visited movies
</span>        <span class="n">queue</span> <span class="o">=</span> <span class="p">[(</span><span class="n">movie_id</span><span class="p">,</span> <span class="mi">0</span><span class="p">)]</span>  <span class="c1"># (movie_id, current_depth)
</span>        <span class="n">recommendations</span> <span class="o">=</span> <span class="p">[]</span>

        <span class="k">while</span> <span class="n">queue</span><span class="p">:</span>
            <span class="n">current_id</span><span class="p">,</span> <span class="n">current_depth</span> <span class="o">=</span> <span class="n">queue</span><span class="p">.</span><span class="nf">pop</span><span class="p">(</span><span class="mi">0</span><span class="p">)</span>  <span class="c1"># FIFO for BFS
</span>
            <span class="c1"># Skip if we've seen this movie or exceeded depth
</span>            <span class="k">if</span> <span class="n">current_id</span> <span class="ow">in</span> <span class="n">visited</span> <span class="ow">or</span> <span class="n">current_depth</span> <span class="o">&gt;</span> <span class="n">depth</span><span class="p">:</span>
                <span class="k">continue</span>

            <span class="n">visited</span><span class="p">.</span><span class="nf">add</span><span class="p">(</span><span class="n">current_id</span><span class="p">)</span>

            <span class="c1"># Don't include the original movie in recommendations
</span>            <span class="k">if</span> <span class="n">current_id</span> <span class="o">!=</span> <span class="n">movie_id</span><span class="p">:</span>
                <span class="n">movie</span> <span class="o">=</span> <span class="n">self</span><span class="p">.</span><span class="n">db</span><span class="p">.</span><span class="n">movies</span><span class="p">.</span><span class="nf">get</span><span class="p">(</span><span class="n">current_id</span><span class="p">)</span>
                <span class="k">if</span> <span class="n">movie</span><span class="p">:</span>
                    <span class="n">recommendations</span><span class="p">.</span><span class="nf">append</span><span class="p">({</span>
                        <span class="sh">'</span><span class="s">movie</span><span class="sh">'</span><span class="p">:</span> <span class="n">movie</span><span class="p">,</span>
                        <span class="sh">'</span><span class="s">distance</span><span class="sh">'</span><span class="p">:</span> <span class="n">current_depth</span>
                    <span class="p">})</span>

            <span class="c1"># Add connected movies to queue (if they exist)
</span>            <span class="k">if</span> <span class="n">current_id</span> <span class="ow">in</span> <span class="n">self</span><span class="p">.</span><span class="n">db</span><span class="p">.</span><span class="n">connections</span><span class="p">:</span>
                <span class="k">for</span> <span class="n">connected_id</span> <span class="ow">in</span> <span class="n">self</span><span class="p">.</span><span class="n">db</span><span class="p">.</span><span class="n">connections</span><span class="p">[</span><span class="n">current_id</span><span class="p">]:</span>
                    <span class="k">if</span> <span class="n">connected_id</span> <span class="ow">not</span> <span class="ow">in</span> <span class="n">visited</span><span class="p">:</span>
                        <span class="n">queue</span><span class="p">.</span><span class="nf">append</span><span class="p">((</span><span class="n">connected_id</span><span class="p">,</span> <span class="n">current_depth</span> <span class="o">+</span> <span class="mi">1</span><span class="p">))</span>

        <span class="k">return</span> <span class="n">recommendations</span>
</code></pre>

</div>



<p>The last method is where we apply the BFS algorithm <code>find_recommendations</code> method, and it works like this:<br>
We start with the movie source at depth 0, then use a queue (FIFO) to process the movies level by level. As explained above, we mark the movies that have been visited (i.e., the nodes) to prevent cycles. From here, we add all unvisited connections to the queue with depth + 1, or another level. And finally, we stop when we have exceeded the specified depth.</p>

<p>We now have practically the entire program, but the most important element of the project is still the <code>MovieRecommenderCLI</code> class. This is the brain, where actions or options are thought out, because from there we extract user input as the Command Line Interface of <code>CLI</code>, which all developers are so proud of for making these types of programs. And tell me, guys? And if you're just starting out, you're going to feel super special for making these <code>CLI</code>s.<br>
Returning to the topic at hand, here above we see the code and walk through it very briefly as follows:<br>
The CLI interface methods:<br>
It shows us the menu with all the options, handles user input with error checking, formats the output appropriately with its structure, allows searching by title, genre, and obtaining recommendations.<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight python"><code><span class="k">def</span> <span class="nf">search_by_title_interface</span><span class="p">(</span><span class="n">self</span><span class="p">):</span>
        <span class="sh">"""</span><span class="s">Handle title search interaction</span><span class="sh">"""</span>
        <span class="n">query</span> <span class="o">=</span> <span class="nf">input</span><span class="p">(</span><span class="sh">"</span><span class="se">\n</span><span class="s">Enter movie title to search: </span><span class="sh">"</span><span class="p">).</span><span class="nf">strip</span><span class="p">()</span>
        <span class="n">results</span> <span class="o">=</span> <span class="n">self</span><span class="p">.</span><span class="n">searcher</span><span class="p">.</span><span class="nf">search_by_title</span><span class="p">(</span><span class="n">query</span><span class="p">)</span>

        <span class="k">if</span> <span class="n">results</span><span class="p">:</span>
            <span class="nf">print</span><span class="p">(</span><span class="sa">f</span><span class="sh">"</span><span class="se">\n</span><span class="s">Found </span><span class="si">{</span><span class="nf">len</span><span class="p">(</span><span class="n">results</span><span class="p">)</span><span class="si">}</span><span class="s"> movie(s):</span><span class="sh">"</span><span class="p">)</span>
            <span class="nf">print</span><span class="p">(</span><span class="sh">"</span><span class="s">-</span><span class="sh">"</span><span class="o">*</span><span class="mi">40</span><span class="p">)</span>
            <span class="k">for</span> <span class="n">movie</span> <span class="ow">in</span> <span class="n">results</span><span class="p">:</span>
                <span class="n">self</span><span class="p">.</span><span class="nf">display_movie</span><span class="p">(</span><span class="n">movie</span><span class="p">)</span>
        <span class="k">else</span><span class="p">:</span>
            <span class="nf">print</span><span class="p">(</span><span class="sa">f</span><span class="sh">"</span><span class="se">\n</span><span class="s">No movies found with </span><span class="sh">'</span><span class="si">{</span><span class="n">query</span><span class="si">}</span><span class="sh">'</span><span class="s"> in the title</span><span class="sh">"</span><span class="p">)</span>

    <span class="k">def</span> <span class="nf">search_by_genre_interface</span><span class="p">(</span><span class="n">self</span><span class="p">):</span>
        <span class="sh">"""</span><span class="s">Handle genre search interaction</span><span class="sh">"""</span>
        <span class="c1"># Show avaialable genres
</span>        <span class="n">all_genres</span> <span class="o">=</span> <span class="nf">set</span><span class="p">()</span>
        <span class="k">for</span> <span class="n">movie</span> <span class="ow">in</span> <span class="n">self</span><span class="p">.</span><span class="n">db</span><span class="p">.</span><span class="n">movies</span><span class="p">.</span><span class="nf">values</span><span class="p">():</span>
            <span class="k">for</span> <span class="n">genre</span> <span class="ow">in</span> <span class="n">movie</span><span class="p">[</span><span class="sh">'</span><span class="s">genres</span><span class="sh">'</span><span class="p">]:</span>
                <span class="n">all_genres</span><span class="p">.</span><span class="nf">add</span><span class="p">(</span><span class="n">genre</span><span class="p">)</span>

        <span class="nf">print</span><span class="p">(</span><span class="sh">"</span><span class="se">\n</span><span class="s">Avalilable genres</span><span class="sh">"</span><span class="p">)</span>
        <span class="k">for</span> <span class="n">genre</span> <span class="ow">in</span> <span class="nf">sorted</span><span class="p">(</span><span class="n">all_genres</span><span class="p">):</span>
            <span class="nf">print</span><span class="p">(</span><span class="sa">f</span><span class="sh">"</span><span class="s">    . </span><span class="si">{</span><span class="n">genre</span><span class="si">}</span><span class="sh">"</span><span class="p">)</span>

        <span class="n">query</span> <span class="o">=</span> <span class="nf">input</span><span class="p">(</span><span class="sh">"</span><span class="se">\n</span><span class="s">Enter genre to search: </span><span class="sh">"</span><span class="p">).</span><span class="nf">strip</span><span class="p">()</span>
        <span class="n">results</span> <span class="o">=</span> <span class="n">self</span><span class="p">.</span><span class="n">searcher</span><span class="p">.</span><span class="nf">search_by_genre</span><span class="p">(</span><span class="n">query</span><span class="p">)</span>

        <span class="k">if</span> <span class="n">results</span><span class="p">:</span>
            <span class="nf">print</span><span class="p">(</span><span class="sa">f</span><span class="sh">"</span><span class="se">\n</span><span class="s">Found </span><span class="si">{</span><span class="nf">len</span><span class="p">(</span><span class="n">results</span><span class="p">)</span><span class="si">}</span><span class="s"> movies(s) in </span><span class="sh">'</span><span class="si">{</span><span class="n">query</span><span class="si">}</span><span class="sh">'</span><span class="s"> genre</span><span class="sh">"</span><span class="p">)</span>
            <span class="nf">print</span><span class="p">(</span><span class="sh">"</span><span class="s">-</span><span class="sh">"</span><span class="o">*</span><span class="mi">40</span><span class="p">)</span>
            <span class="k">for</span> <span class="n">movie</span> <span class="ow">in</span> <span class="n">results</span><span class="p">:</span>
                <span class="n">self</span><span class="p">.</span><span class="nf">display_movie</span><span class="p">(</span><span class="n">movie</span><span class="p">)</span>
        <span class="k">else</span><span class="p">:</span>
            <span class="nf">print</span><span class="p">(</span><span class="sa">f</span><span class="sh">"</span><span class="se">\n</span><span class="s">No movies found in </span><span class="sh">'</span><span class="si">{</span><span class="n">query</span><span class="si">}</span><span class="sh">'</span><span class="s"> genre</span><span class="sh">"</span><span class="p">)</span>
</code></pre>

</div>



<p>The <code>search_by_title_interface</code> and <code>search_by_genre_interface</code> methods basically handle the interactions for searching for movies. With these, we collect user input, call the other related methods of the <code>searcher</code> object, and display the formatted results. The genre search has an extra feature: it displays all the genres before prompting the input, making it more user-friendly.<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight python"><code><span class="k">def</span> <span class="nf">get_recommendations_interface</span><span class="p">(</span><span class="n">self</span><span class="p">):</span>
        <span class="sh">"""</span><span class="s">Handle recommendation interaction</span><span class="sh">"""</span>
        <span class="c1"># Show all movies with IDs
</span>        <span class="nf">print</span><span class="p">(</span><span class="sh">"</span><span class="se">\n</span><span class="s">Select a movie to get recommendations</span><span class="sh">"</span><span class="p">)</span>
        <span class="nf">print</span><span class="p">(</span><span class="sh">"</span><span class="s">-</span><span class="sh">"</span><span class="o">*</span><span class="mi">40</span><span class="p">)</span>
        <span class="k">for</span> <span class="n">movie_id</span><span class="p">,</span> <span class="n">movie</span> <span class="ow">in</span> <span class="n">self</span><span class="p">.</span><span class="n">db</span><span class="p">.</span><span class="n">movies</span><span class="p">.</span><span class="nf">items</span><span class="p">():</span>
            <span class="nf">print</span><span class="p">(</span><span class="sa">f</span><span class="sh">"</span><span class="s">    [</span><span class="si">{</span><span class="n">movie_id</span><span class="si">}</span><span class="s">] </span><span class="si">{</span><span class="n">movie</span><span class="p">[</span><span class="sh">'</span><span class="s">title</span><span class="sh">'</span><span class="p">]</span><span class="si">}</span><span class="s"> (</span><span class="si">{</span><span class="n">movie</span><span class="p">[</span><span class="sh">'</span><span class="s">year</span><span class="sh">'</span><span class="p">]</span><span class="si">}</span><span class="s">)</span><span class="sh">"</span><span class="p">)</span>

        <span class="k">try</span><span class="p">:</span>
            <span class="n">movie_id</span> <span class="o">=</span> <span class="nf">int</span><span class="p">(</span><span class="nf">input</span><span class="p">(</span><span class="sh">"</span><span class="se">\n</span><span class="s">Enter movie ID: </span><span class="sh">"</span><span class="p">))</span>    

            <span class="k">if</span> <span class="n">movie_id</span> <span class="ow">not</span> <span class="ow">in</span> <span class="n">self</span><span class="p">.</span><span class="n">db</span><span class="p">.</span><span class="n">movies</span><span class="p">:</span>
                <span class="nf">print</span><span class="p">(</span><span class="sa">f</span><span class="sh">"</span><span class="se">\n</span><span class="s">Movie with ID </span><span class="si">{</span><span class="n">movie_id</span><span class="si">}</span><span class="s"> not found</span><span class="sh">"</span><span class="p">)</span>
                <span class="k">return</span> 

            <span class="nf">print</span><span class="p">(</span><span class="sh">"</span><span class="se">\n</span><span class="s">Recommendation type:</span><span class="sh">"</span><span class="p">)</span>
            <span class="nf">print</span><span class="p">(</span><span class="sh">"</span><span class="s">1. Similar movies (closest matches)</span><span class="sh">"</span><span class="p">)</span>
            <span class="nf">print</span><span class="p">(</span><span class="sh">"</span><span class="s">2. Extended recommendations (includes related to similar)</span><span class="sh">"</span><span class="p">)</span>
            <span class="n">choice</span> <span class="o">=</span> <span class="nf">input</span><span class="p">(</span><span class="sh">"</span><span class="s">Choose (1 o 2) [default=1]: </span><span class="sh">"</span><span class="p">).</span><span class="nf">strip</span><span class="p">()</span>
            <span class="n">depth</span> <span class="o">=</span> <span class="mi">2</span> <span class="k">if</span> <span class="n">choice</span> <span class="o">==</span> <span class="sh">'</span><span class="s">2</span><span class="sh">'</span> <span class="k">else</span> <span class="mi">1</span>


            <span class="n">source_movie</span> <span class="o">=</span> <span class="n">self</span><span class="p">.</span><span class="n">db</span><span class="p">.</span><span class="n">movies</span><span class="p">[</span><span class="n">movie_id</span><span class="p">]</span>
            <span class="nf">print</span><span class="p">(</span><span class="sa">f</span><span class="sh">"</span><span class="se">\n</span><span class="s">Movies similar to: </span><span class="si">{</span><span class="n">source_movie</span><span class="p">[</span><span class="sh">'</span><span class="s">title</span><span class="sh">'</span><span class="p">]</span><span class="si">}</span><span class="sh">"</span><span class="p">)</span>
            <span class="nf">print</span><span class="p">(</span><span class="sh">"</span><span class="s">=</span><span class="sh">"</span><span class="o">*</span><span class="mi">50</span><span class="p">)</span>

            <span class="n">recommendations</span> <span class="o">=</span> <span class="n">self</span><span class="p">.</span><span class="n">searcher</span><span class="p">.</span><span class="nf">find_recommendations</span><span class="p">(</span><span class="n">movie_id</span><span class="p">,</span> <span class="n">depth</span><span class="p">)</span>

            <span class="k">if</span> <span class="n">recommendations</span><span class="p">:</span>
                <span class="c1"># Sort by distance
</span>                <span class="n">recommendations</span><span class="p">.</span><span class="nf">sort</span><span class="p">(</span><span class="n">key</span><span class="o">=</span><span class="k">lambda</span> <span class="n">x</span><span class="p">:</span> <span class="n">x</span><span class="p">[</span><span class="sh">'</span><span class="s">distance</span><span class="sh">'</span><span class="p">])</span>

                <span class="k">for</span> <span class="n">rec</span> <span class="ow">in</span> <span class="n">recommendations</span><span class="p">:</span>
                    <span class="k">if</span> <span class="n">rec</span><span class="p">[</span><span class="sh">'</span><span class="s">distance</span><span class="sh">'</span><span class="p">]</span> <span class="o">==</span> <span class="mi">1</span><span class="p">:</span>
                       <span class="n">similarity_label</span> <span class="o">=</span> <span class="sh">"</span><span class="s">Highly Similar</span><span class="sh">"</span>
                    <span class="k">else</span><span class="p">:</span>
                       <span class="n">similarity_label</span> <span class="o">=</span> <span class="sh">"</span><span class="s">Also Recommended</span><span class="sh">"</span>

                    <span class="nf">print</span><span class="p">(</span><span class="sa">f</span><span class="sh">"</span><span class="se">\n</span><span class="s">  </span><span class="si">{</span><span class="n">similarity_label</span><span class="si">}</span><span class="sh">"</span><span class="p">)</span>
                    <span class="n">self</span><span class="p">.</span><span class="nf">display_movie</span><span class="p">(</span><span class="n">rec</span><span class="p">[</span><span class="sh">'</span><span class="s">movie</span><span class="sh">'</span><span class="p">],</span> <span class="sh">"</span><span class="s">  </span><span class="sh">"</span><span class="p">)</span>

                    <span class="c1"># Add reason if it's a direct connection
</span>                    <span class="k">if</span> <span class="n">rec</span><span class="p">[</span><span class="sh">'</span><span class="s">distance</span><span class="sh">'</span><span class="p">]</span> <span class="o">==</span> <span class="mi">1</span> <span class="ow">and</span> <span class="n">movie_id</span> <span class="ow">in</span> <span class="n">self</span><span class="p">.</span><span class="n">db</span><span class="p">.</span><span class="n">connections</span><span class="p">:</span>
                        <span class="c1"># Find the connection reason from the original JSON if available
</span>                        <span class="nf">print</span><span class="p">(</span><span class="sa">f</span><span class="sh">"</span><span class="s">    Why: Similar themes and style</span><span class="sh">"</span><span class="p">)</span>
            <span class="k">else</span><span class="p">:</span>
                <span class="nf">print</span><span class="p">(</span><span class="sh">"</span><span class="se">\n</span><span class="s">No recommendations found</span><span class="sh">"</span><span class="p">)</span>

        <span class="k">except</span> <span class="nb">ValueError</span><span class="p">:</span>
            <span class="nf">print</span><span class="p">(</span><span class="sh">"</span><span class="se">\n</span><span class="s">No recommendations found</span><span class="sh">"</span><span class="p">)</span>
</code></pre>

</div>



<p>The <code>get_recommendation_interface</code> method is where we also use the BFS algorithm. When we enter <code>depth=2</code> for its depth to make "extended recommendations," it implements a two-level BFS traversal through your movie's connection graph. This algorithm first finds movies directly connected to the source (distance=1), and then finds movies connected to it (distance=2). The results are sorted by distance and displayed with labels such as "Highly Similar" for direct connections and "Also Recommended" for second-degree connections. The <code>show_all_movies</code> method simply displays all movies sorted by rating in descending order.<br>
A very important piece of information, at the end of this brief explanation of the code, is that in: <code>self.searcher.find_recommendations(movie_id, depth)</code>. With <code>depth=2</code>, it explores the graph level by level, first visiting all immediate neighbors (similar movies), then their neighbors (extended recommendations), thus creating a tree recommendation system that expands beyond the selected movie.</p>

<h2>
  
  
  Wrappping Up: The Main Application Loop
</h2>

<p>The final pieces bring everything togheter into a working application. The  <code>show_all_movies</code>method provides a simple but useful feature, it retrieves all movies from the database, sorts them by ratin in descending order (highest-rated first), and displays them in a formatted list. This gives user a quick overview of the best movies in the system.</p>

<p>The core also of the application it's the <code>run</code> method main event loop, orchestrating the entire user experinece. It displays a welcome message, then enters an infinite while loop that continuouly presents the meny and processes user choices as any CLI. Based on the selected option (1-5), it routes to the appropriate interface method, whether that's searching by title, genre, getting recommendations, showing all movies, or exiting the program. Lastly, it pauses after we did all operations with "Press Enter to continue" to let users review results before returning to the main menu.<br>
Finally, the entry point at the bottom <code>if __name__ == "__main__"</code> is where the magic begins. When the script runs directly, it creates an instance of our <code>MovieRecommenderCLI</code> class and call its <code>run()</code> method, launching the interactive command-line interface. Well amazing job, I deserve a I deserve a pat on the back, well done Ivan :)</p>

<p>Now you can test this final project please check the provided link to clone the project from Github:</p>

<p><a href="https://github.com/ivansing/movie-recommender" rel="noopener noreferrer">Github repository project</a></p>

<h2>
  
  
  Conclusion
</h2>

<p>As you could see, this was a great little project because it really showed how the BFS algorithm works, more or less. Although I didn’t go into great depth, it’s not difficult, but it does require attention. We were able to create several classes and their methods for searching and recommending using this very useful algorithm, by the way. Well, as homework :) see how this type of algorithm is used all over the internet—it’s really impressive. Just think that this is only one of hundreds, if not thousands, of algorithms out there.</p>

<p>Thank you for reading this article. I hope you enjoyed it, great community. I want to keep writing more regularly, and I hope I can, so I can share my thoughts on this great topic of computer science.</p>

<h2>
  
  
  About the Author
</h2>

<p>Ivan Duarte is a full-stack developer, Node.js and Python developer, content writer, entrepreneur, and founder of ByteUp LLC, a software development company.</p>

