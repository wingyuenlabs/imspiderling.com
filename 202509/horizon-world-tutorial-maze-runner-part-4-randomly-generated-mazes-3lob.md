---
Title: Horizon World Tutorial - Maze Runner - Part 4 - Randomly generated mazes
Description: 
Author: LNATION
Date: 2025-09-02T21:56:11.000Z
Robots: noindex,nofollow
Template: index
---
<p>In the earlier parts of this tutorial series, we established the foundational world, built out the main gameplay features, and introduced a custom UI timer and background music. Now, our focus shifts to generating the maze itself. While there are many maze generation algorithms to choose from, we'll be using a straightforward depth-first search (DFS) <code>carve</code> algorithm for this guide. This method works by recursively carving out random paths in a grid to create the maze structure. One important note: the carve implementation I will use today require both the width and height of the grid to be odd numbers to ensure the maze renders properly.</p>

<p>To begin, create a new script dedicated to maze generation. Open your world in the desktop editor and use the UI to add a new script.</p>

<p><a href="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fdwi63rz676satz6l60jw.png" class="article-body-image-wrapper"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fdwi63rz676satz6l60jw.png" alt="Maze Script" width="377" height="197"></a></p>

<p>After creating the script, open it. You should see the familiar boilerplate code as a starting point.<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight typescript"><code><span class="k">import</span> <span class="o">*</span> <span class="k">as</span> <span class="nx">hz</span> <span class="k">from</span> <span class="dl">'</span><span class="s1">horizon/core</span><span class="dl">'</span><span class="p">;</span>

<span class="kd">class</span> <span class="nc">Maze</span> <span class="kd">extends</span> <span class="nc">hz</span><span class="p">.</span><span class="nx">Component</span><span class="o">&lt;</span><span class="k">typeof</span> <span class="nx">Maze</span><span class="o">&gt;</span> <span class="p">{</span>
  <span class="k">static</span> <span class="nx">propsDefinition</span> <span class="o">=</span> <span class="p">{};</span>

  <span class="nf">start</span><span class="p">()</span> <span class="p">{</span>

  <span class="p">}</span>
<span class="p">}</span>
<span class="nx">hz</span><span class="p">.</span><span class="nx">Component</span><span class="p">.</span><span class="nf">register</span><span class="p">(</span><span class="nx">Maze</span><span class="p">);</span>
</code></pre>

</div>



<p>Like with the other files we must first import the <code>Events</code> and <code>GameState</code> from our <code>GameUtils</code> script. Add the import line before the class definition.<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight typescript"><code><span class="k">import</span> <span class="p">{</span> <span class="nx">Events</span><span class="p">,</span> <span class="nx">GameState</span> <span class="p">}</span> <span class="k">from</span> <span class="dl">'</span><span class="s1">GameUtils</span><span class="dl">'</span><span class="p">;</span>
</code></pre>

</div>



<p>Next we will plan our <code>propsDefinition</code> for the <code>Maze</code> component. This will include any properties we want to expose for customisation. We will make our maze size configurable and made up of two main properties: <code>width</code> and <code>height</code>, both of which will be integers representing the dimensions of the maze grid. For the maze walls we will use <code>assets</code> we will have two type of walls a simple wall and a detailed wall, the walls we will be using today will be 4x10x4. Also because we will make our maze dynamic we will need to position the start and end zones within the maze, so we will need properties to store a reference to them also. Extend the <code>propsDefinition</code> object in your class to the following:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight typescript"><code>    <span class="k">static</span> <span class="nx">propsDefinition</span> <span class="o">=</span> <span class="p">{</span>
        <span class="na">wallDetailed</span><span class="p">:</span> <span class="p">{</span> <span class="na">type</span><span class="p">:</span> <span class="nx">hz</span><span class="p">.</span><span class="nx">PropTypes</span><span class="p">.</span><span class="nx">Asset</span><span class="p">,</span> <span class="na">required</span><span class="p">:</span> <span class="kc">true</span> <span class="p">},</span>
        <span class="na">wallSimple</span><span class="p">:</span> <span class="p">{</span> <span class="na">type</span><span class="p">:</span> <span class="nx">hz</span><span class="p">.</span><span class="nx">PropTypes</span><span class="p">.</span><span class="nx">Asset</span><span class="p">,</span> <span class="na">required</span><span class="p">:</span> <span class="kc">true</span> <span class="p">},</span>
        <span class="na">wallWidth</span><span class="p">:</span> <span class="p">{</span> <span class="na">type</span><span class="p">:</span> <span class="nx">hz</span><span class="p">.</span><span class="nx">PropTypes</span><span class="p">.</span><span class="nb">Number</span><span class="p">,</span> <span class="na">default</span><span class="p">:</span> <span class="mi">4</span> <span class="p">},</span>
        <span class="na">wallHeight</span><span class="p">:</span> <span class="p">{</span> <span class="na">type</span><span class="p">:</span> <span class="nx">hz</span><span class="p">.</span><span class="nx">PropTypes</span><span class="p">.</span><span class="nb">Number</span><span class="p">,</span> <span class="na">default</span><span class="p">:</span> <span class="mi">10</span> <span class="p">},</span>
        <span class="na">width</span><span class="p">:</span> <span class="p">{</span> <span class="na">type</span><span class="p">:</span> <span class="nx">hz</span><span class="p">.</span><span class="nx">PropTypes</span><span class="p">.</span><span class="nb">Number</span><span class="p">,</span> <span class="na">default</span><span class="p">:</span> <span class="mi">19</span> <span class="p">},</span>
        <span class="na">height</span><span class="p">:</span> <span class="p">{</span> <span class="na">type</span><span class="p">:</span> <span class="nx">hz</span><span class="p">.</span><span class="nx">PropTypes</span><span class="p">.</span><span class="nb">Number</span><span class="p">,</span> <span class="na">default</span><span class="p">:</span> <span class="mi">19</span> <span class="p">},</span>
        <span class="na">startPosition</span><span class="p">:</span> <span class="p">{</span> <span class="na">type</span><span class="p">:</span> <span class="nx">hz</span><span class="p">.</span><span class="nx">PropTypes</span><span class="p">.</span><span class="nx">Entity</span><span class="p">,</span> <span class="na">required</span><span class="p">:</span> <span class="kc">true</span> <span class="p">},</span>
        <span class="na">finishPosition</span><span class="p">:</span> <span class="p">{</span> <span class="na">type</span><span class="p">:</span> <span class="nx">hz</span><span class="p">.</span><span class="nx">PropTypes</span><span class="p">.</span><span class="nx">Entity</span><span class="p">,</span> <span class="na">required</span><span class="p">:</span> <span class="kc">true</span> <span class="p">},</span>
    <span class="p">};</span>
</code></pre>

</div>



<p>With that in place, save the file and return to the desktop editor. Create a new object, name this <code>Maze</code>, and attach the <code>Maze</code> script to it. This will then allow you to attach the necessary assets and configure the maze properties directly from the editor. For the maze walls in this tutorial we will use two assets that I have made public, search in the <code>Assets</code> panel for <code>Maze Runner</code> and then add the first wall asset to your world. After that, rename the object to <code>WallDetailed</code> and set the <code>Motion</code> to <code>animated</code>.</p>

<p><a href="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Frzn67wlcur4z9rka60c7.png" class="article-body-image-wrapper"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Frzn67wlcur4z9rka60c7.png" alt="Wall Detailed" width="800" height="558"></a></p>

<p>Once you have done that you next need to save the asset to your library, right click on the asset and select 'create asset', save it into your local asset directory. After that, repeat the process for the second wall asset, renaming it to <code>WallSimple</code> and remembering to set the <code>Motion</code> to <code>animated</code> as well.</p>

<p><a href="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fr3jd6glvdlq2i1hjh2hn.png" class="article-body-image-wrapper"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fr3jd6glvdlq2i1hjh2hn.png" alt="Create Asset" width="366" height="396"></a></p>

<p><a href="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2F03k80c4cwfdzv0pzqacp.png" class="article-body-image-wrapper"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2F03k80c4cwfdzv0pzqacp.png" alt="Save Asset" width="679" height="658"></a></p>

<p>Then with both assets saved you can delete the original wall objects from your scene, leaving only the newly created assets in your library. Navigate to the <code>Maze</code> object and when you click on <code>WallDetailed</code> you will be able to search for the relevant asset. Leave the wallWidth as 4, but set the maze width and height properties with the value of 9 and attach the <code>GameSpawnPoint</code> to the <code>startPosition</code> and <code>Finish</code> to <code>finishPosition</code>.</p>

<p><a href="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fwrk1zed9822q7ndwa13y.png" class="article-body-image-wrapper"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fwrk1zed9822q7ndwa13y.png" alt="Wall Simpler" width="453" height="166"></a></p>

<p><a href="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fccyk5h2dviagmc9jnold.png" class="article-body-image-wrapper"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fccyk5h2dviagmc9jnold.png" alt="Maze Script Props" width="376" height="319"></a></p>

<p>Now with the relevant properties set we in the desktop editor you can return to your code editor, and open <code>Maze.ts</code>. Next we will add a private property which we will use to store our maze grid. Beneath the <code>propsDefinition</code> add a line to define a private attribute that is an array of arrays of hashes. In typescript you need to explicitly define the hash that will be stored for type checking, in our hash we will have keys for the <code>wall</code> asset, grid <code>type</code> and <code>x</code>, <code>y</code>, and <code>z</code> positions.<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight typescript"><code>    <span class="k">private</span> <span class="nx">walls</span><span class="p">:</span> <span class="p">{</span> <span class="nl">wall</span><span class="p">:</span> <span class="nx">hz</span><span class="p">.</span><span class="nx">Entity</span> <span class="o">|</span> <span class="kc">undefined</span><span class="p">,</span> <span class="kd">type</span><span class="p">:</span> <span class="kr">string</span><span class="p">,</span> <span class="nx">x</span><span class="p">:</span> <span class="kr">number</span><span class="p">,</span> <span class="nx">y</span><span class="p">:</span> <span class="kr">number</span><span class="p">,</span> <span class="nx">z</span><span class="p">:</span> <span class="kr">number</span> <span class="p">}[][]</span> <span class="o">=</span> <span class="p">[];</span>
</code></pre>

</div>



<p>The next step is to generate the maze grid, this will involve creating a 2D array that represents the maze layout. Each cell in the grid will be an object that contains information about the wall asset to use, its position, and the type of wall which we will default to 'W' for wall when generating the grid. We will hook into the <code>start</code> function to populate this grid, we will do this via a callback function <code>generateMazeGrid</code>. Update the <code>start</code> function in <code>Maze.ts</code> to the following:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight typescript"><code>    <span class="nf">start</span><span class="p">()</span> <span class="p">{</span>
        <span class="k">if </span><span class="p">(</span><span class="o">!</span><span class="k">this</span><span class="p">.</span><span class="nx">walls</span><span class="p">.</span><span class="nx">length</span><span class="p">)</span> <span class="p">{</span>
            <span class="k">this</span><span class="p">.</span><span class="nf">generateMazeGrid</span><span class="p">();</span>
        <span class="p">}</span>
    <span class="p">}</span>
</code></pre>

</div>



<p>Then to implement the <code>generateMazeGrid</code> function, we will first need to calculate the dimensions of the maze based on the wall width and the number of walls. This will give us the overall width and height of the maze.<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight typescript"><code>    <span class="k">private</span> <span class="nf">generateMazeGrid</span><span class="p">()</span> <span class="p">{</span>
        <span class="kd">let</span> <span class="nx">width</span> <span class="o">=</span> <span class="k">this</span><span class="p">.</span><span class="nx">props</span><span class="p">.</span><span class="nx">width</span> <span class="o">*</span> <span class="k">this</span><span class="p">.</span><span class="nx">props</span><span class="p">.</span><span class="nx">wallWidth</span><span class="p">;</span>
        <span class="kd">let</span> <span class="nx">height</span> <span class="o">=</span> <span class="k">this</span><span class="p">.</span><span class="nx">props</span><span class="p">.</span><span class="nx">height</span> <span class="o">*</span> <span class="k">this</span><span class="p">.</span><span class="nx">props</span><span class="p">.</span><span class="nx">wallWidth</span><span class="p">;</span>

    <span class="p">}</span>
</code></pre>

</div>



<p>Next we will need to calculate the start x, y and z positions for our grid, the y will always be half our wall height for our assets. The x will be half the full width of the maze and the z will be zero minus half the full height of the maze. This will ensure we start drawing the grid from the south west corner as our game is designed.<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight typescript"><code>        <span class="kd">let</span> <span class="nx">x</span> <span class="o">=</span> <span class="nx">width</span> <span class="o">/</span> <span class="mi">2</span><span class="p">;</span>
        <span class="kd">let</span> <span class="nx">y</span> <span class="o">=</span> <span class="k">this</span><span class="p">.</span><span class="nx">props</span><span class="p">.</span><span class="nx">wallHeight</span> <span class="o">/</span> <span class="mi">2</span><span class="p">;</span>
        <span class="kd">let</span> <span class="nx">z</span> <span class="o">=</span> <span class="mi">0</span> <span class="o">-</span> <span class="p">(</span><span class="nx">height</span> <span class="o">/</span> <span class="mi">2</span><span class="p">);</span>
</code></pre>

</div>



<p>Now we are going to iterate over the height and width of the maze to create the wall entities. We will need to recurse either the X or Z so we must store the start position for atleast one, in this example we will keep track of x, so next declare a <code>startX</code> variable.<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight typescript"><code>        <span class="kd">let</span> <span class="nx">startX</span> <span class="o">=</span> <span class="nx">x</span><span class="p">;</span>
</code></pre>

</div>



<p>To add variety to our maze, we'll randomly rotate the detailed wall assets so that different sides are visible each time the maze is generated. Since the simple wall looks the same on all sides, we can use a fixed rotation for it. Let's introduce a <code>simpleRotation</code> variable for the simple wall, while the detailed wall will use a random rotation.<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight typescript"><code>        <span class="kd">let</span> <span class="nx">simpleRotation</span> <span class="o">=</span> <span class="nx">hz</span><span class="p">.</span><span class="nx">Quaternion</span><span class="p">.</span><span class="nf">fromEuler</span><span class="p">(</span><span class="k">new</span> <span class="nx">hz</span><span class="p">.</span><span class="nc">Vec3</span><span class="p">(</span><span class="mi">270</span><span class="p">,</span> <span class="mi">0</span><span class="p">,</span> <span class="mi">0</span><span class="p">));</span>
</code></pre>

</div>



<p>We also need to create a local 2d array to hold our wall entities before we set the global property. Use the same type definition as the <code>walls</code> property.<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight typescript"><code>        <span class="kd">let</span> <span class="nx">walls</span><span class="p">:</span> <span class="p">{</span> <span class="nl">wall</span><span class="p">:</span> <span class="nx">hz</span><span class="p">.</span><span class="nx">Entity</span> <span class="o">|</span> <span class="kc">undefined</span><span class="p">,</span> <span class="kd">type</span><span class="p">:</span> <span class="kr">string</span><span class="p">,</span> <span class="nx">x</span><span class="p">:</span> <span class="kr">number</span><span class="p">,</span> <span class="nx">y</span><span class="p">:</span> <span class="kr">number</span><span class="p">,</span> <span class="nx">z</span><span class="p">:</span> <span class="kr">number</span> <span class="p">}[][]</span> <span class="o">=</span> <span class="p">[];</span>
</code></pre>

</div>



<p>Next we need to do something a little strange, we need to declare a variable that is casted to either 'Entity' or 'undefined' and then set this to undefined. We will use this as a placeholder while our assets are loaded into entities.<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight typescript"><code>        <span class="kd">let</span> <span class="nx">undefined_entity</span><span class="p">:</span> <span class="nx">hz</span><span class="p">.</span><span class="nx">Entity</span> <span class="o">|</span> <span class="kc">undefined</span> <span class="o">=</span> <span class="kc">undefined</span><span class="p">;</span>
</code></pre>

</div>



<p>The final variable we need to declare is something I call <code>at_the_races</code>, this variable will by used to keep track of the number of wall entities that are still being spawned. There is more than one way to handle this race condition, but a simple approach is to use a counter that increments when a wall entity is spawned and decrements when it is finished.<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight typescript"><code>        <span class="kd">let</span> <span class="nx">at_the_races</span> <span class="o">=</span> <span class="mi">0</span><span class="p">;</span>
</code></pre>

</div>



<p>Now to implement the iteration needed to generate our grid, we will iterate over the height and width of the maze to create the wall entities. Add the following after the <code>at_the_races</code> declaration:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight typescript"><code>        <span class="k">for </span><span class="p">(</span><span class="kd">let</span> <span class="nx">i</span> <span class="o">=</span> <span class="mi">0</span><span class="p">;</span> <span class="nx">i</span> <span class="o">&lt;</span> <span class="nx">height</span><span class="p">;</span> <span class="nx">i</span> <span class="o">+=</span> <span class="k">this</span><span class="p">.</span><span class="nx">props</span><span class="p">.</span><span class="nx">wallWidth</span><span class="p">)</span> <span class="p">{</span>
            <span class="kd">let</span> <span class="nx">row</span> <span class="o">=</span> <span class="p">[];</span>
            <span class="k">for </span><span class="p">(</span><span class="kd">let</span> <span class="nx">j</span> <span class="o">=</span> <span class="mi">0</span><span class="p">;</span> <span class="nx">j</span> <span class="o">&lt;</span> <span class="nx">width</span><span class="p">;</span> <span class="nx">j</span> <span class="o">+=</span> <span class="k">this</span><span class="p">.</span><span class="nx">props</span><span class="p">.</span><span class="nx">wallWidth</span><span class="p">)</span> <span class="p">{</span>

            <span class="p">}</span>

        <span class="p">}</span>
</code></pre>

</div>



<p>Then we need to first calculate the start x, y and z position for our grid wall. This must be a Vec3 and at this point should just be the x, y and z variables we have already defined, we will increment these later in our loops.<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight typescript"><code>                <span class="kd">let</span> <span class="nx">position</span> <span class="o">=</span> <span class="k">new</span> <span class="nx">hz</span><span class="p">.</span><span class="nc">Vec3</span><span class="p">(</span>
                    <span class="nx">x</span><span class="p">,</span>
                    <span class="nx">y</span><span class="p">,</span>
                    <span class="nx">z</span>
                <span class="p">);</span>
</code></pre>

</div>



<p>Next, create a grid cell using the same type definition as the <code>walls</code> property. Initialize each property with the values you have at this stage, setting the <code>type</code> to <code>W</code> by default since all cells start as walls.<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight typescript"><code>                <span class="kd">let</span> <span class="nx">cell</span><span class="p">:</span> <span class="p">{</span> <span class="nl">wall</span><span class="p">:</span> <span class="nx">hz</span><span class="p">.</span><span class="nx">Entity</span> <span class="o">|</span> <span class="kc">undefined</span><span class="p">,</span> <span class="kd">type</span><span class="p">:</span> <span class="kr">string</span><span class="p">,</span> <span class="nx">x</span><span class="p">:</span> <span class="kr">number</span><span class="p">,</span> <span class="nx">y</span><span class="p">:</span> <span class="kr">number</span><span class="p">,</span> <span class="nx">z</span><span class="p">:</span> <span class="kr">number</span> <span class="p">}</span> <span class="o">=</span> <span class="p">{</span> <span class="na">wall</span><span class="p">:</span> <span class="nx">undefined_entity</span><span class="p">,</span> <span class="na">type</span><span class="p">:</span> <span class="dl">'</span><span class="s1">W</span><span class="dl">'</span><span class="p">,</span> <span class="na">x</span><span class="p">:</span> <span class="nx">x</span><span class="p">,</span> <span class="na">y</span><span class="p">:</span> <span class="nx">y</span><span class="p">,</span> <span class="na">z</span><span class="p">:</span> <span class="nx">z</span> <span class="p">};</span>
                <span class="nx">row</span><span class="p">.</span><span class="nf">push</span><span class="p">(</span><span class="nx">cell</span><span class="p">);</span>
</code></pre>

</div>



<p>Now we need to decide which wall asset to spawn. For now we will implement simple logic that will make the third wall in each row a detailed wall. When we identify that we will spawn a detailed wall, we will also randomly rotate it. Add the following after 'row.push(cell)'.<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight typescript"><code>                <span class="kd">let</span> <span class="p">[</span><span class="nx">asset</span><span class="p">,</span> <span class="nx">rotation</span><span class="p">]</span> <span class="o">=</span> <span class="p">(</span><span class="nx">j</span> <span class="o">+</span> <span class="nx">i</span> <span class="o">/</span> <span class="k">this</span><span class="p">.</span><span class="nx">props</span><span class="p">.</span><span class="nx">wallWidth</span><span class="p">)</span> <span class="o">%</span> <span class="mi">3</span> <span class="o">===</span> <span class="mi">0</span>
                    <span class="p">?</span> <span class="p">[</span><span class="k">this</span><span class="p">.</span><span class="nx">props</span><span class="p">.</span><span class="nx">wallDetailed</span><span class="p">,</span> <span class="nx">hz</span><span class="p">.</span><span class="nx">Quaternion</span><span class="p">.</span><span class="nf">fromEuler</span><span class="p">(</span><span class="k">new</span> <span class="nx">hz</span><span class="p">.</span><span class="nc">Vec3</span><span class="p">(</span><span class="mi">270</span><span class="p">,</span> <span class="mi">0</span><span class="p">,</span> <span class="mi">90</span> <span class="o">*</span> <span class="nb">Math</span><span class="p">.</span><span class="nf">floor</span><span class="p">(</span><span class="nb">Math</span><span class="p">.</span><span class="nf">random</span><span class="p">()</span> <span class="o">*</span> <span class="mi">4</span><span class="p">)))]</span>
                    <span class="p">:</span> <span class="p">[</span><span class="k">this</span><span class="p">.</span><span class="nx">props</span><span class="p">.</span><span class="nx">wallSimple</span><span class="p">,</span> <span class="nx">simpleRotation</span><span class="p">];</span>
</code></pre>

</div>



<p>Then increment <code>at_the_races</code> as a wall entity is about to be spawned via a promise.<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight typescript"><code>                <span class="nx">at_the_races</span><span class="o">++</span><span class="p">;</span>
</code></pre>

</div>



<p>To spawn the asset we must first check if the <code>asset</code> variable is defined, then we can use <code>hz.Entity.spawn(asset, position, rotation)</code> to create the wall entity. When the promise resolves, we can set the cell walls entity property and decrement the <code>at_the_races</code> variable.<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight typescript"><code>                <span class="k">if </span><span class="p">(</span><span class="nx">asset</span><span class="p">)</span> <span class="k">this</span><span class="p">.</span><span class="nx">world</span><span class="p">.</span><span class="nf">spawnAsset</span><span class="p">(</span><span class="nx">asset</span><span class="p">,</span> <span class="nx">position</span><span class="p">,</span> <span class="nx">rotation</span><span class="p">).</span><span class="nf">then</span><span class="p">(</span><span class="nx">spawnedObjects</span> <span class="o">=&gt;</span> <span class="p">{</span>
                    <span class="nx">spawnedObjects</span><span class="p">.</span><span class="nf">forEach</span><span class="p">(</span><span class="nx">obj</span> <span class="o">=&gt;</span> <span class="p">{</span>
                        <span class="nx">cell</span><span class="p">.</span><span class="nx">wall</span> <span class="o">=</span> <span class="nx">obj</span><span class="p">;</span>
                        <span class="nx">at_the_races</span><span class="o">--</span><span class="p">;</span>
                    <span class="p">});</span>
                <span class="p">});</span>
</code></pre>

</div>



<p>The final piece of code needed in the width iteration is to increment the x variable so that the next loop draws the next grid cell.<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight typescript"><code>                <span class="nx">x</span> <span class="o">-=</span> <span class="k">this</span><span class="p">.</span><span class="nx">props</span><span class="p">.</span><span class="nx">wallWidth</span><span class="p">;</span>
</code></pre>

</div>



<p>Now that is our grid width row is being drawn correctly, next is the height iteration. After you exit the width iteration, you will need to push the row into the walls array, reset the x variable and increment the z variable so that the next loop draws the next grid row.<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight typescript"><code>            <span class="nx">walls</span><span class="p">.</span><span class="nf">push</span><span class="p">(</span> <span class="nx">row</span> <span class="p">);</span>
            <span class="nx">x</span> <span class="o">=</span> <span class="nx">startX</span><span class="p">;</span>
            <span class="nx">z</span> <span class="o">+=</span> <span class="k">this</span><span class="p">.</span><span class="nx">props</span><span class="p">.</span><span class="nx">wallWidth</span><span class="p">;</span>
</code></pre>

</div>



<p>With that in place we have only our race condition to handle, after the height itteration add the following:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight typescript"><code>        <span class="kd">let</span> <span class="nx">racer</span> <span class="o">=</span> <span class="mi">0</span><span class="p">;</span>
        <span class="nx">racer</span> <span class="o">=</span> <span class="k">this</span><span class="p">.</span><span class="k">async</span><span class="p">.</span><span class="nf">setInterval</span><span class="p">(()</span> <span class="o">=&gt;</span> <span class="p">{</span>
            <span class="k">if </span><span class="p">(</span><span class="nx">at_the_races</span> <span class="o">===</span> <span class="mi">0</span><span class="p">)</span> <span class="p">{</span>
                <span class="k">this</span><span class="p">.</span><span class="nx">walls</span> <span class="o">=</span> <span class="nx">walls</span><span class="p">;</span>
                <span class="k">this</span><span class="p">.</span><span class="nf">sendLocalBroadcastEvent</span><span class="p">(</span><span class="nx">Events</span><span class="p">.</span><span class="nx">setGameState</span><span class="p">,</span> <span class="p">{</span> <span class="na">state</span><span class="p">:</span> <span class="nx">GameState</span><span class="p">.</span><span class="nx">Ready</span><span class="p">,</span> <span class="na">winner</span><span class="p">:</span> <span class="kc">undefined</span> <span class="p">});</span>
                <span class="k">this</span><span class="p">.</span><span class="k">async</span><span class="p">.</span><span class="nf">clearInterval</span><span class="p">(</span><span class="nx">racer</span><span class="p">);</span>
            <span class="p">}</span>
        <span class="p">},</span> <span class="mi">100</span><span class="p">);</span>
</code></pre>

</div>



<p>This code sets up a <code>setInterval</code> that checks if all wall entities have been spawned. Once all walls are in place, it sets the walls property with the final grid, updates the game state to "Ready" via a <code>setGameState</code> event, and clears the interval to stop the checks.</p>

<p>Your current <code>generateMazeGrid</code> function should look like this:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight typescript"><code>    <span class="k">private</span> <span class="nf">generateMazeGrid</span><span class="p">()</span> <span class="p">{</span>
        <span class="kd">let</span> <span class="nx">width</span> <span class="o">=</span> <span class="k">this</span><span class="p">.</span><span class="nx">props</span><span class="p">.</span><span class="nx">width</span> <span class="o">*</span> <span class="k">this</span><span class="p">.</span><span class="nx">props</span><span class="p">.</span><span class="nx">wallWidth</span><span class="p">;</span>
        <span class="kd">let</span> <span class="nx">height</span> <span class="o">=</span> <span class="k">this</span><span class="p">.</span><span class="nx">props</span><span class="p">.</span><span class="nx">height</span> <span class="o">*</span> <span class="k">this</span><span class="p">.</span><span class="nx">props</span><span class="p">.</span><span class="nx">wallWidth</span><span class="p">;</span>
        <span class="kd">let</span> <span class="nx">x</span> <span class="o">=</span> <span class="nx">width</span> <span class="o">/</span> <span class="mi">2</span><span class="p">;</span>
        <span class="kd">let</span> <span class="nx">y</span> <span class="o">=</span> <span class="k">this</span><span class="p">.</span><span class="nx">props</span><span class="p">.</span><span class="nx">wallHeight</span> <span class="o">/</span> <span class="mi">2</span><span class="p">;</span>
        <span class="kd">let</span> <span class="nx">z</span> <span class="o">=</span> <span class="mi">0</span> <span class="o">-</span> <span class="p">(</span><span class="nx">height</span> <span class="o">/</span> <span class="mi">2</span><span class="p">);</span>
        <span class="kd">let</span> <span class="nx">startX</span> <span class="o">=</span> <span class="nx">x</span><span class="p">;</span>
        <span class="kd">let</span> <span class="nx">simpleRotation</span> <span class="o">=</span> <span class="nx">hz</span><span class="p">.</span><span class="nx">Quaternion</span><span class="p">.</span><span class="nf">fromEuler</span><span class="p">(</span><span class="k">new</span> <span class="nx">hz</span><span class="p">.</span><span class="nc">Vec3</span><span class="p">(</span><span class="mi">270</span><span class="p">,</span> <span class="mi">0</span><span class="p">,</span> <span class="mi">0</span><span class="p">));</span>
        <span class="kd">let</span> <span class="nx">walls</span><span class="p">:</span> <span class="p">{</span> <span class="nl">wall</span><span class="p">:</span> <span class="nx">hz</span><span class="p">.</span><span class="nx">Entity</span> <span class="o">|</span> <span class="kc">undefined</span><span class="p">,</span> <span class="kd">type</span><span class="p">:</span> <span class="kr">string</span><span class="p">,</span> <span class="nx">x</span><span class="p">:</span> <span class="kr">number</span><span class="p">,</span> <span class="nx">y</span><span class="p">:</span> <span class="kr">number</span><span class="p">,</span> <span class="nx">z</span><span class="p">:</span> <span class="kr">number</span> <span class="p">}[][]</span> <span class="o">=</span> <span class="p">[];</span>
        <span class="kd">let</span> <span class="nx">undefined_entity</span><span class="p">:</span> <span class="nx">hz</span><span class="p">.</span><span class="nx">Entity</span> <span class="o">|</span> <span class="kc">undefined</span> <span class="o">=</span> <span class="kc">undefined</span><span class="p">;</span>
        <span class="kd">let</span> <span class="nx">at_the_races</span> <span class="o">=</span> <span class="mi">0</span><span class="p">;</span>

        <span class="k">for </span><span class="p">(</span><span class="kd">let</span> <span class="nx">i</span> <span class="o">=</span> <span class="mi">0</span><span class="p">;</span> <span class="nx">i</span> <span class="o">&lt;</span> <span class="nx">height</span><span class="p">;</span> <span class="nx">i</span> <span class="o">+=</span> <span class="k">this</span><span class="p">.</span><span class="nx">props</span><span class="p">.</span><span class="nx">wallWidth</span><span class="p">)</span> <span class="p">{</span>
            <span class="kd">let</span> <span class="nx">row</span> <span class="o">=</span> <span class="p">[];</span>
            <span class="k">for </span><span class="p">(</span><span class="kd">let</span> <span class="nx">j</span> <span class="o">=</span> <span class="mi">0</span><span class="p">;</span> <span class="nx">j</span> <span class="o">&lt;</span> <span class="nx">width</span><span class="p">;</span> <span class="nx">j</span> <span class="o">+=</span> <span class="k">this</span><span class="p">.</span><span class="nx">props</span><span class="p">.</span><span class="nx">wallWidth</span><span class="p">)</span> <span class="p">{</span>
                <span class="kd">let</span> <span class="nx">position</span> <span class="o">=</span> <span class="k">new</span> <span class="nx">hz</span><span class="p">.</span><span class="nc">Vec3</span><span class="p">(</span>
                    <span class="nx">x</span><span class="p">,</span>
                    <span class="nx">y</span><span class="p">,</span>
                    <span class="nx">z</span>
                <span class="p">);</span>
                <span class="kd">let</span> <span class="nx">cell</span><span class="p">:</span> <span class="p">{</span> <span class="nl">wall</span><span class="p">:</span> <span class="nx">hz</span><span class="p">.</span><span class="nx">Entity</span> <span class="o">|</span> <span class="kc">undefined</span><span class="p">,</span> <span class="kd">type</span><span class="p">:</span> <span class="kr">string</span><span class="p">,</span> <span class="nx">x</span><span class="p">:</span> <span class="kr">number</span><span class="p">,</span> <span class="nx">y</span><span class="p">:</span> <span class="kr">number</span><span class="p">,</span> <span class="nx">z</span><span class="p">:</span> <span class="kr">number</span> <span class="p">}</span> <span class="o">=</span> <span class="p">{</span> <span class="na">wall</span><span class="p">:</span> <span class="nx">undefined_entity</span><span class="p">,</span> <span class="na">type</span><span class="p">:</span> <span class="dl">'</span><span class="s1">W</span><span class="dl">'</span><span class="p">,</span> <span class="na">x</span><span class="p">:</span> <span class="nx">x</span><span class="p">,</span> <span class="na">y</span><span class="p">:</span> <span class="nx">y</span><span class="p">,</span> <span class="na">z</span><span class="p">:</span> <span class="nx">z</span> <span class="p">};</span>
                <span class="nx">row</span><span class="p">.</span><span class="nf">push</span><span class="p">(</span><span class="nx">cell</span><span class="p">);</span>
                <span class="nx">at_the_races</span><span class="o">++</span><span class="p">;</span>
                <span class="kd">let</span> <span class="p">[</span><span class="nx">asset</span><span class="p">,</span> <span class="nx">rotation</span><span class="p">]</span> <span class="o">=</span> <span class="p">(</span><span class="nx">j</span> <span class="o">+</span> <span class="nx">i</span> <span class="o">/</span> <span class="k">this</span><span class="p">.</span><span class="nx">props</span><span class="p">.</span><span class="nx">wallWidth</span><span class="p">)</span> <span class="o">%</span> <span class="mi">3</span> <span class="o">===</span> <span class="mi">0</span>
                    <span class="p">?</span> <span class="p">[</span><span class="k">this</span><span class="p">.</span><span class="nx">props</span><span class="p">.</span><span class="nx">wallDetailed</span><span class="p">,</span> <span class="nx">hz</span><span class="p">.</span><span class="nx">Quaternion</span><span class="p">.</span><span class="nf">fromEuler</span><span class="p">(</span><span class="k">new</span> <span class="nx">hz</span><span class="p">.</span><span class="nc">Vec3</span><span class="p">(</span><span class="mi">270</span><span class="p">,</span> <span class="mi">0</span><span class="p">,</span> <span class="mi">90</span> <span class="o">*</span> <span class="nb">Math</span><span class="p">.</span><span class="nf">floor</span><span class="p">(</span><span class="nb">Math</span><span class="p">.</span><span class="nf">random</span><span class="p">()</span> <span class="o">*</span> <span class="mi">4</span><span class="p">)))]</span>
                    <span class="p">:</span> <span class="p">[</span><span class="k">this</span><span class="p">.</span><span class="nx">props</span><span class="p">.</span><span class="nx">wallSimple</span><span class="p">,</span> <span class="nx">simpleRotation</span><span class="p">];</span>
                <span class="k">if </span><span class="p">(</span><span class="nx">asset</span><span class="p">)</span> <span class="k">this</span><span class="p">.</span><span class="nx">world</span><span class="p">.</span><span class="nf">spawnAsset</span><span class="p">(</span><span class="nx">asset</span><span class="p">,</span> <span class="nx">position</span><span class="p">,</span> <span class="nx">rotation</span><span class="p">).</span><span class="nf">then</span><span class="p">(</span><span class="nx">spawnedObjects</span> <span class="o">=&gt;</span> <span class="p">{</span>
                    <span class="nx">spawnedObjects</span><span class="p">.</span><span class="nf">forEach</span><span class="p">(</span><span class="nx">obj</span> <span class="o">=&gt;</span> <span class="p">{</span>
                        <span class="nx">cell</span><span class="p">.</span><span class="nx">wall</span> <span class="o">=</span> <span class="nx">obj</span><span class="p">;</span>
                        <span class="nx">at_the_races</span><span class="o">--</span><span class="p">;</span>
                    <span class="p">});</span>
                <span class="p">});</span>

                <span class="nx">x</span> <span class="o">-=</span> <span class="k">this</span><span class="p">.</span><span class="nx">props</span><span class="p">.</span><span class="nx">wallWidth</span><span class="p">;</span>
            <span class="p">}</span>
            <span class="nx">walls</span><span class="p">.</span><span class="nf">push</span><span class="p">(</span> <span class="nx">row</span> <span class="p">);</span>
            <span class="nx">x</span> <span class="o">=</span> <span class="nx">startX</span><span class="p">;</span>
            <span class="nx">z</span> <span class="o">+=</span> <span class="k">this</span><span class="p">.</span><span class="nx">props</span><span class="p">.</span><span class="nx">wallWidth</span><span class="p">;</span>
        <span class="p">}</span>

        <span class="kd">let</span> <span class="nx">racer</span> <span class="o">=</span> <span class="mi">0</span><span class="p">;</span>
        <span class="nx">racer</span> <span class="o">=</span> <span class="k">this</span><span class="p">.</span><span class="k">async</span><span class="p">.</span><span class="nf">setInterval</span><span class="p">(()</span> <span class="o">=&gt;</span> <span class="p">{</span>
            <span class="k">if </span><span class="p">(</span><span class="nx">at_the_races</span> <span class="o">===</span> <span class="mi">0</span><span class="p">)</span> <span class="p">{</span>
                <span class="k">this</span><span class="p">.</span><span class="nx">walls</span> <span class="o">=</span> <span class="nx">walls</span><span class="p">;</span>
                <span class="k">this</span><span class="p">.</span><span class="nf">sendLocalBroadcastEvent</span><span class="p">(</span><span class="nx">Events</span><span class="p">.</span><span class="nx">setGameState</span><span class="p">,</span> <span class="p">{</span> <span class="na">state</span><span class="p">:</span> <span class="nx">GameState</span><span class="p">.</span><span class="nx">Ready</span><span class="p">,</span> <span class="na">winner</span><span class="p">:</span> <span class="kc">undefined</span> <span class="p">});</span>
                <span class="k">this</span><span class="p">.</span><span class="k">async</span><span class="p">.</span><span class="nf">clearInterval</span><span class="p">(</span><span class="nx">racer</span><span class="p">);</span>
            <span class="p">}</span>
        <span class="p">},</span> <span class="mi">100</span><span class="p">);</span>
    <span class="p">}</span>
</code></pre>

</div>



<p>You will now be able to save, and enter preview mode in your desktop editor. You should see a 9x9 grid of walls in the center of the game area.</p>

<p><a href="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fo2ldedisoh0ta8lxxsqf.png" class="article-body-image-wrapper"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fo2ldedisoh0ta8lxxsqf.png" alt="9x9 Grid" width="800" height="322"></a></p>

<p>Next we need to setup our event listener so that we can randomly generate the maze via a <code>carve</code> function when the game state changes to <code>Starting</code>, we will also want to reset all grid cells to their initial state when the game state changes to <code>Finished</code>. We will implement this listener inside of a <code>preStart</code> function. Add the following code before the <code>start</code> function.<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight typescript"><code>    <span class="nf">preStart</span><span class="p">()</span> <span class="p">{</span>
        <span class="k">if </span><span class="p">(</span><span class="k">this</span><span class="p">.</span><span class="nx">props</span><span class="p">.</span><span class="nx">width</span> <span class="o">%</span> <span class="mi">2</span> <span class="o">===</span> <span class="mi">0</span><span class="p">)</span> <span class="k">throw</span> <span class="k">new</span> <span class="nc">Error</span><span class="p">(</span><span class="dl">"</span><span class="s2">Width must be odd</span><span class="dl">"</span><span class="p">);</span>
        <span class="k">if </span><span class="p">(</span><span class="k">this</span><span class="p">.</span><span class="nx">props</span><span class="p">.</span><span class="nx">height</span> <span class="o">%</span> <span class="mi">2</span> <span class="o">===</span> <span class="mi">0</span><span class="p">)</span> <span class="k">throw</span> <span class="k">new</span> <span class="nc">Error</span><span class="p">(</span><span class="dl">"</span><span class="s2">Height must be odd</span><span class="dl">"</span><span class="p">);</span>
        <span class="k">this</span><span class="p">.</span><span class="nf">connectLocalBroadcastEvent</span><span class="p">(</span><span class="nx">Events</span><span class="p">.</span><span class="nx">gameStateChanged</span><span class="p">,</span> <span class="p">(</span><span class="nx">data</span><span class="p">)</span> <span class="o">=&gt;</span> <span class="p">{</span>
            <span class="k">if </span><span class="p">(</span><span class="nx">data</span><span class="p">.</span><span class="nx">toState</span> <span class="o">==</span> <span class="nx">GameState</span><span class="p">.</span><span class="nx">Starting</span><span class="p">)</span> <span class="p">{</span>
                <span class="k">this</span><span class="p">.</span><span class="nf">carve</span><span class="p">(</span><span class="mi">1</span><span class="p">,</span> <span class="mi">1</span><span class="p">);</span>
            <span class="p">}</span> <span class="k">else</span> <span class="k">if </span><span class="p">(</span><span class="nx">data</span><span class="p">.</span><span class="nx">toState</span> <span class="o">==</span> <span class="nx">GameState</span><span class="p">.</span><span class="nx">Finished</span><span class="p">)</span> <span class="p">{</span>
                <span class="k">this</span><span class="p">.</span><span class="nf">resetWalls</span><span class="p">();</span>
            <span class="p">}</span>
        <span class="p">});</span>
    <span class="p">}</span>
</code></pre>

</div>



<p>Note how we have added some validation to ensure the maze dimensions are odd numbers, this is to handle the earlier mentioned caveat with the algorithm we are about to implement which relies on a grid structure that is always odd in both dimensions to render perfectly. </p>

<p>To implement the carve function start with this boilerplate code:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight typescript"><code>    <span class="k">private</span> <span class="nf">carve</span><span class="p">(</span><span class="nx">x</span><span class="p">:</span> <span class="kr">number</span><span class="p">,</span> <span class="nx">z</span><span class="p">:</span> <span class="kr">number</span><span class="p">,</span> <span class="nx">visited</span><span class="p">?:</span> <span class="nb">Set</span><span class="o">&lt;</span><span class="kr">string</span><span class="o">&gt;</span><span class="p">):</span> <span class="k">void</span> <span class="p">{</span>

    <span class="p">}</span>
</code></pre>

</div>



<p>We have defined x and z as the coordinates of the current cell being processed in the maze. The carve function will use these coordinates to manipulate the maze data structure and create paths. The visited parameter is a Set that keeps track of all the cells that have been processed so far to avoid infinite loops. (even though this algorithm would never go into an infinite loop this is necessary for typescript).</p>

<p>Now we need to define some variables, first height and width which we can get from our properties and then a reference to walls.<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight typescript"><code>        <span class="kd">let</span> <span class="nx">height</span> <span class="o">=</span> <span class="k">this</span><span class="p">.</span><span class="nx">props</span><span class="p">.</span><span class="nx">height</span><span class="p">;</span>
        <span class="kd">let</span> <span class="nx">width</span> <span class="o">=</span> <span class="k">this</span><span class="p">.</span><span class="nx">props</span><span class="p">.</span><span class="nx">width</span><span class="p">;</span>
        <span class="kd">let</span> <span class="nx">maze</span> <span class="o">=</span> <span class="k">this</span><span class="p">.</span><span class="nx">walls</span><span class="p">;</span>
</code></pre>

</div>



<p>Then the <code>visited</code> variable if it is not defined.<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight typescript"><code>        <span class="k">if </span><span class="p">(</span><span class="o">!</span><span class="nx">visited</span><span class="p">)</span> <span class="nx">visited</span> <span class="o">=</span> <span class="k">new</span> <span class="nb">Set</span><span class="o">&lt;</span><span class="kr">string</span><span class="o">&gt;</span><span class="p">();</span>
</code></pre>

</div>



<p>Next we need to check whether we have visited this cell, and if so return early. (Not really necessary, but good for clarity).<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight typescript"><code>        <span class="kd">const</span> <span class="nx">key</span> <span class="o">=</span> <span class="s2">`</span><span class="p">${</span><span class="nx">x</span><span class="p">}</span><span class="s2">,</span><span class="p">${</span><span class="nx">z</span><span class="p">}</span><span class="s2">`</span><span class="p">;</span>
        <span class="k">if </span><span class="p">(</span><span class="nx">visited</span><span class="p">.</span><span class="nf">has</span><span class="p">(</span><span class="nx">key</span><span class="p">))</span> <span class="k">return</span><span class="p">;</span>
        <span class="nx">visited</span><span class="p">.</span><span class="nf">add</span><span class="p">(</span><span class="nx">key</span><span class="p">);</span>
</code></pre>

</div>



<p>Now we need to explore the neighboring cells. We will use a randomized depth-first search (DFS) approach to carve out the maze. First, we need to define the possible directions we can move in the maze.<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight typescript"><code>        <span class="kd">let</span> <span class="nx">dirs</span> <span class="o">=</span> <span class="p">[[</span><span class="mi">0</span><span class="p">,</span> <span class="o">-</span><span class="mi">1</span><span class="p">],</span> <span class="p">[</span><span class="mi">1</span><span class="p">,</span> <span class="mi">0</span><span class="p">],</span> <span class="p">[</span><span class="mi">0</span><span class="p">,</span> <span class="mi">1</span><span class="p">],</span> <span class="p">[</span><span class="o">-</span><span class="mi">1</span><span class="p">,</span> <span class="mi">0</span><span class="p">]];</span>
</code></pre>

</div>



<p>Then we need to randomize the order of the directions to ensure that the maze is carved in a random pattern.<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight typescript"><code>        <span class="k">for </span><span class="p">(</span><span class="kd">let</span> <span class="nx">i</span> <span class="o">=</span> <span class="mi">0</span><span class="p">;</span> <span class="nx">i</span> <span class="o">&lt;</span> <span class="mi">4</span><span class="p">;</span> <span class="o">++</span><span class="nx">i</span><span class="p">)</span> <span class="p">{</span>
            <span class="kd">let</span> <span class="nx">j</span> <span class="o">=</span> <span class="nb">Math</span><span class="p">.</span><span class="nf">floor</span><span class="p">(</span><span class="nb">Math</span><span class="p">.</span><span class="nf">random</span><span class="p">()</span> <span class="o">*</span> <span class="mi">4</span><span class="p">);</span>
            <span class="kd">let</span> <span class="nx">tmp0</span> <span class="o">=</span> <span class="nx">dirs</span><span class="p">[</span><span class="nx">i</span><span class="p">][</span><span class="mi">0</span><span class="p">],</span> <span class="nx">tmp1</span> <span class="o">=</span> <span class="nx">dirs</span><span class="p">[</span><span class="nx">i</span><span class="p">][</span><span class="mi">1</span><span class="p">];</span>
            <span class="nx">dirs</span><span class="p">[</span><span class="nx">i</span><span class="p">][</span><span class="mi">0</span><span class="p">]</span> <span class="o">=</span> <span class="nx">dirs</span><span class="p">[</span><span class="nx">j</span><span class="p">][</span><span class="mi">0</span><span class="p">];</span>
            <span class="nx">dirs</span><span class="p">[</span><span class="nx">i</span><span class="p">][</span><span class="mi">1</span><span class="p">]</span> <span class="o">=</span> <span class="nx">dirs</span><span class="p">[</span><span class="nx">j</span><span class="p">][</span><span class="mi">1</span><span class="p">];</span>
            <span class="nx">dirs</span><span class="p">[</span><span class="nx">j</span><span class="p">][</span><span class="mi">0</span><span class="p">]</span> <span class="o">=</span> <span class="nx">tmp0</span><span class="p">;</span>
            <span class="nx">dirs</span><span class="p">[</span><span class="nx">j</span><span class="p">][</span><span class="mi">1</span><span class="p">]</span> <span class="o">=</span> <span class="nx">tmp1</span><span class="p">;</span>
        <span class="p">}</span>
</code></pre>

</div>



<p>After we mark the current cell as part of the path.<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight typescript"><code>        <span class="k">if </span><span class="p">(</span><span class="nx">maze</span><span class="p">[</span><span class="nx">z</span><span class="p">]</span> <span class="o">&amp;&amp;</span> <span class="nx">maze</span><span class="p">[</span><span class="nx">z</span><span class="p">][</span><span class="nx">x</span><span class="p">])</span> <span class="p">{</span>
            <span class="k">if </span><span class="p">(</span><span class="nx">maze</span><span class="p">[</span><span class="nx">z</span><span class="p">][</span><span class="nx">x</span><span class="p">].</span><span class="kd">type</span> <span class="o">===</span> <span class="dl">'</span><span class="s1">W</span><span class="dl">'</span><span class="p">)</span> <span class="p">{</span>
                <span class="nx">maze</span><span class="p">[</span><span class="nx">z</span><span class="p">][</span><span class="nx">x</span><span class="p">].</span><span class="kd">type</span> <span class="o">=</span> <span class="dl">'</span><span class="s1">P</span><span class="dl">'</span><span class="p">;</span>
                <span class="nx">maze</span><span class="p">[</span><span class="nx">z</span><span class="p">][</span><span class="nx">x</span><span class="p">].</span><span class="nx">wall</span><span class="p">?.</span><span class="nx">position</span><span class="p">.</span><span class="nf">set</span><span class="p">(</span><span class="k">new</span> <span class="nx">hz</span><span class="p">.</span><span class="nc">Vec3</span><span class="p">(</span><span class="nx">maze</span><span class="p">[</span><span class="nx">z</span><span class="p">][</span><span class="nx">x</span><span class="p">].</span><span class="nx">x</span><span class="p">,</span> <span class="nx">maze</span><span class="p">[</span><span class="nx">z</span><span class="p">][</span><span class="nx">x</span><span class="p">].</span><span class="nx">y</span> <span class="o">-</span> <span class="k">this</span><span class="p">.</span><span class="nx">props</span><span class="p">.</span><span class="nx">wallHeight</span><span class="p">,</span> <span class="nx">maze</span><span class="p">[</span><span class="nx">z</span><span class="p">][</span><span class="nx">x</span><span class="p">].</span><span class="nx">z</span><span class="p">));</span>
            <span class="p">}</span>
        <span class="p">}</span>
</code></pre>

</div>



<p>Then we are ready to explore the neighbouring cells. We will use the randomised directions we created earlier to move to each neighbouring cell. Checking first whether we have hit an outer wall. If we have, we need to stop the carving process in that direction. We continue this process recursively until we have carved out the entire maze. Extend with the following code:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight typescript"><code>        <span class="k">for </span><span class="p">(</span><span class="kd">let</span> <span class="nx">i</span> <span class="o">=</span> <span class="mi">0</span><span class="p">;</span> <span class="nx">i</span> <span class="o">&lt;</span> <span class="mi">4</span><span class="p">;</span> <span class="o">++</span><span class="nx">i</span><span class="p">)</span> <span class="p">{</span>
            <span class="kd">let</span> <span class="nx">dx</span> <span class="o">=</span> <span class="nx">dirs</span><span class="p">[</span><span class="nx">i</span><span class="p">][</span><span class="mi">0</span><span class="p">],</span> <span class="nx">dz</span> <span class="o">=</span> <span class="nx">dirs</span><span class="p">[</span><span class="nx">i</span><span class="p">][</span><span class="mi">1</span><span class="p">];</span>
            <span class="kd">let</span> <span class="nx">nx</span> <span class="o">=</span> <span class="nx">x</span> <span class="o">+</span> <span class="mi">2</span><span class="o">*</span><span class="nx">dx</span><span class="p">,</span> <span class="nx">nz</span> <span class="o">=</span> <span class="nx">z</span> <span class="o">+</span> <span class="mi">2</span><span class="o">*</span><span class="nx">dz</span><span class="p">;</span>
            <span class="k">if </span><span class="p">(</span><span class="nx">nx</span> <span class="o">&gt;</span> <span class="mi">0</span> <span class="o">&amp;&amp;</span> <span class="nx">nx</span> <span class="o">&lt;</span> <span class="nx">width</span><span class="o">-</span><span class="mi">1</span> <span class="o">&amp;&amp;</span> <span class="nx">nz</span> <span class="o">&gt;</span> <span class="mi">0</span> <span class="o">&amp;&amp;</span> <span class="nx">nz</span> <span class="o">&lt;</span> <span class="nx">height</span><span class="o">-</span><span class="mi">1</span><span class="p">)</span> <span class="p">{</span>
                <span class="k">if </span><span class="p">(</span><span class="nx">maze</span><span class="p">[</span><span class="nx">nz</span><span class="p">]</span> <span class="o">&amp;&amp;</span> <span class="nx">maze</span><span class="p">[</span><span class="nx">nz</span><span class="p">][</span><span class="nx">nx</span><span class="p">]</span> <span class="o">&amp;&amp;</span> <span class="nx">maze</span><span class="p">[</span><span class="nx">nz</span><span class="p">][</span><span class="nx">nx</span><span class="p">].</span><span class="kd">type</span> <span class="o">===</span> <span class="dl">'</span><span class="s1">W</span><span class="dl">'</span><span class="p">)</span> <span class="p">{</span>
                    <span class="nx">maze</span><span class="p">[</span><span class="nx">z</span><span class="o">+</span><span class="nx">dz</span><span class="p">][</span><span class="nx">x</span><span class="o">+</span><span class="nx">dx</span><span class="p">].</span><span class="kd">type</span> <span class="o">=</span> <span class="dl">'</span><span class="s1">P</span><span class="dl">'</span><span class="p">;</span>
                    <span class="nx">maze</span><span class="p">[</span><span class="nx">z</span><span class="o">+</span><span class="nx">dz</span><span class="p">][</span><span class="nx">x</span><span class="o">+</span><span class="nx">dx</span><span class="p">].</span><span class="nx">wall</span><span class="p">?.</span><span class="nx">position</span><span class="p">.</span><span class="nf">set</span><span class="p">(</span><span class="k">new</span> <span class="nx">hz</span><span class="p">.</span><span class="nc">Vec3</span><span class="p">(</span><span class="nx">maze</span><span class="p">[</span><span class="nx">z</span><span class="o">+</span><span class="nx">dz</span><span class="p">][</span><span class="nx">x</span><span class="o">+</span><span class="nx">dx</span><span class="p">].</span><span class="nx">x</span><span class="p">,</span> <span class="nx">maze</span><span class="p">[</span><span class="nx">z</span><span class="o">+</span><span class="nx">dz</span><span class="p">][</span><span class="nx">x</span><span class="o">+</span><span class="nx">dx</span><span class="p">].</span><span class="nx">y</span> <span class="o">-</span> <span class="k">this</span><span class="p">.</span><span class="nx">props</span><span class="p">.</span><span class="nx">wallHeight</span><span class="p">,</span> <span class="nx">maze</span><span class="p">[</span><span class="nx">z</span><span class="o">+</span><span class="nx">dz</span><span class="p">][</span><span class="nx">x</span><span class="o">+</span><span class="nx">dx</span><span class="p">].</span><span class="nx">z</span><span class="p">));</span>
                    <span class="k">this</span><span class="p">.</span><span class="nf">carve</span><span class="p">(</span><span class="nx">nx</span><span class="p">,</span> <span class="nx">nz</span><span class="p">,</span> <span class="nx">visited</span><span class="p">);</span>
                <span class="p">}</span>
            <span class="p">}</span>
        <span class="p">}</span>
</code></pre>

</div>



<p>Your final implementation of the carve function should look like this:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight typescript"><code>    <span class="k">private</span> <span class="nf">carve</span><span class="p">(</span><span class="nx">x</span><span class="p">:</span> <span class="kr">number</span><span class="p">,</span> <span class="nx">z</span><span class="p">:</span> <span class="kr">number</span><span class="p">,</span> <span class="nx">visited</span><span class="p">?:</span> <span class="nb">Set</span><span class="o">&lt;</span><span class="kr">string</span><span class="o">&gt;</span><span class="p">):</span> <span class="k">void</span> <span class="p">{</span>
        <span class="kd">let</span> <span class="nx">height</span> <span class="o">=</span> <span class="k">this</span><span class="p">.</span><span class="nx">props</span><span class="p">.</span><span class="nx">height</span><span class="p">;</span>
        <span class="kd">let</span> <span class="nx">width</span> <span class="o">=</span> <span class="k">this</span><span class="p">.</span><span class="nx">props</span><span class="p">.</span><span class="nx">width</span><span class="p">;</span>
        <span class="kd">let</span> <span class="nx">maze</span> <span class="o">=</span> <span class="k">this</span><span class="p">.</span><span class="nx">walls</span><span class="p">;</span>

        <span class="k">if </span><span class="p">(</span><span class="o">!</span><span class="nx">visited</span><span class="p">)</span> <span class="nx">visited</span> <span class="o">=</span> <span class="k">new</span> <span class="nb">Set</span><span class="o">&lt;</span><span class="kr">string</span><span class="o">&gt;</span><span class="p">();</span>
        <span class="kd">const</span> <span class="nx">key</span> <span class="o">=</span> <span class="s2">`</span><span class="p">${</span><span class="nx">x</span><span class="p">}</span><span class="s2">,</span><span class="p">${</span><span class="nx">z</span><span class="p">}</span><span class="s2">`</span><span class="p">;</span>
        <span class="k">if </span><span class="p">(</span><span class="nx">visited</span><span class="p">.</span><span class="nf">has</span><span class="p">(</span><span class="nx">key</span><span class="p">))</span> <span class="k">return</span><span class="p">;</span>
        <span class="nx">visited</span><span class="p">.</span><span class="nf">add</span><span class="p">(</span><span class="nx">key</span><span class="p">);</span>

        <span class="kd">let</span> <span class="nx">dirs</span> <span class="o">=</span> <span class="p">[[</span><span class="mi">0</span><span class="p">,</span> <span class="o">-</span><span class="mi">1</span><span class="p">],</span> <span class="p">[</span><span class="mi">1</span><span class="p">,</span> <span class="mi">0</span><span class="p">],</span> <span class="p">[</span><span class="mi">0</span><span class="p">,</span> <span class="mi">1</span><span class="p">],</span> <span class="p">[</span><span class="o">-</span><span class="mi">1</span><span class="p">,</span> <span class="mi">0</span><span class="p">]];</span>
        <span class="k">for </span><span class="p">(</span><span class="kd">let</span> <span class="nx">i</span> <span class="o">=</span> <span class="mi">0</span><span class="p">;</span> <span class="nx">i</span> <span class="o">&lt;</span> <span class="mi">4</span><span class="p">;</span> <span class="o">++</span><span class="nx">i</span><span class="p">)</span> <span class="p">{</span>
            <span class="kd">let</span> <span class="nx">j</span> <span class="o">=</span> <span class="nb">Math</span><span class="p">.</span><span class="nf">floor</span><span class="p">(</span><span class="nb">Math</span><span class="p">.</span><span class="nf">random</span><span class="p">()</span> <span class="o">*</span> <span class="mi">4</span><span class="p">);</span>
            <span class="kd">let</span> <span class="nx">tmp0</span> <span class="o">=</span> <span class="nx">dirs</span><span class="p">[</span><span class="nx">i</span><span class="p">][</span><span class="mi">0</span><span class="p">],</span> <span class="nx">tmp1</span> <span class="o">=</span> <span class="nx">dirs</span><span class="p">[</span><span class="nx">i</span><span class="p">][</span><span class="mi">1</span><span class="p">];</span>
            <span class="nx">dirs</span><span class="p">[</span><span class="nx">i</span><span class="p">][</span><span class="mi">0</span><span class="p">]</span> <span class="o">=</span> <span class="nx">dirs</span><span class="p">[</span><span class="nx">j</span><span class="p">][</span><span class="mi">0</span><span class="p">];</span>
            <span class="nx">dirs</span><span class="p">[</span><span class="nx">i</span><span class="p">][</span><span class="mi">1</span><span class="p">]</span> <span class="o">=</span> <span class="nx">dirs</span><span class="p">[</span><span class="nx">j</span><span class="p">][</span><span class="mi">1</span><span class="p">];</span>
            <span class="nx">dirs</span><span class="p">[</span><span class="nx">j</span><span class="p">][</span><span class="mi">0</span><span class="p">]</span> <span class="o">=</span> <span class="nx">tmp0</span><span class="p">;</span>
            <span class="nx">dirs</span><span class="p">[</span><span class="nx">j</span><span class="p">][</span><span class="mi">1</span><span class="p">]</span> <span class="o">=</span> <span class="nx">tmp1</span><span class="p">;</span>
        <span class="p">}</span>

        <span class="k">if </span><span class="p">(</span><span class="nx">maze</span><span class="p">[</span><span class="nx">z</span><span class="p">]</span> <span class="o">&amp;&amp;</span> <span class="nx">maze</span><span class="p">[</span><span class="nx">z</span><span class="p">][</span><span class="nx">x</span><span class="p">])</span> <span class="p">{</span>
            <span class="k">if </span><span class="p">(</span><span class="nx">maze</span><span class="p">[</span><span class="nx">z</span><span class="p">][</span><span class="nx">x</span><span class="p">].</span><span class="kd">type</span> <span class="o">===</span> <span class="dl">'</span><span class="s1">W</span><span class="dl">'</span><span class="p">)</span> <span class="p">{</span>
                <span class="nx">maze</span><span class="p">[</span><span class="nx">z</span><span class="p">][</span><span class="nx">x</span><span class="p">].</span><span class="kd">type</span> <span class="o">=</span> <span class="dl">'</span><span class="s1">P</span><span class="dl">'</span><span class="p">;</span>
                <span class="nx">maze</span><span class="p">[</span><span class="nx">z</span><span class="p">][</span><span class="nx">x</span><span class="p">].</span><span class="nx">wall</span><span class="p">?.</span><span class="nx">position</span><span class="p">.</span><span class="nf">set</span><span class="p">(</span><span class="k">new</span> <span class="nx">hz</span><span class="p">.</span><span class="nc">Vec3</span><span class="p">(</span><span class="nx">maze</span><span class="p">[</span><span class="nx">z</span><span class="p">][</span><span class="nx">x</span><span class="p">].</span><span class="nx">x</span><span class="p">,</span> <span class="nx">maze</span><span class="p">[</span><span class="nx">z</span><span class="p">][</span><span class="nx">x</span><span class="p">].</span><span class="nx">y</span> <span class="o">-</span> <span class="k">this</span><span class="p">.</span><span class="nx">props</span><span class="p">.</span><span class="nx">wallHeight</span><span class="p">,</span> <span class="nx">maze</span><span class="p">[</span><span class="nx">z</span><span class="p">][</span><span class="nx">x</span><span class="p">].</span><span class="nx">z</span><span class="p">));</span>
            <span class="p">}</span>
        <span class="p">}</span>

        <span class="k">for </span><span class="p">(</span><span class="kd">let</span> <span class="nx">i</span> <span class="o">=</span> <span class="mi">0</span><span class="p">;</span> <span class="nx">i</span> <span class="o">&lt;</span> <span class="mi">4</span><span class="p">;</span> <span class="o">++</span><span class="nx">i</span><span class="p">)</span> <span class="p">{</span>
            <span class="kd">let</span> <span class="nx">dx</span> <span class="o">=</span> <span class="nx">dirs</span><span class="p">[</span><span class="nx">i</span><span class="p">][</span><span class="mi">0</span><span class="p">],</span> <span class="nx">dz</span> <span class="o">=</span> <span class="nx">dirs</span><span class="p">[</span><span class="nx">i</span><span class="p">][</span><span class="mi">1</span><span class="p">];</span>
            <span class="kd">let</span> <span class="nx">nx</span> <span class="o">=</span> <span class="nx">x</span> <span class="o">+</span> <span class="mi">2</span><span class="o">*</span><span class="nx">dx</span><span class="p">,</span> <span class="nx">nz</span> <span class="o">=</span> <span class="nx">z</span> <span class="o">+</span> <span class="mi">2</span><span class="o">*</span><span class="nx">dz</span><span class="p">;</span>
            <span class="k">if </span><span class="p">(</span><span class="nx">nx</span> <span class="o">&gt;</span> <span class="mi">0</span> <span class="o">&amp;&amp;</span> <span class="nx">nx</span> <span class="o">&lt;</span> <span class="nx">width</span><span class="o">-</span><span class="mi">1</span> <span class="o">&amp;&amp;</span> <span class="nx">nz</span> <span class="o">&gt;</span> <span class="mi">0</span> <span class="o">&amp;&amp;</span> <span class="nx">nz</span> <span class="o">&lt;</span> <span class="nx">height</span><span class="o">-</span><span class="mi">1</span><span class="p">)</span> <span class="p">{</span>
                <span class="k">if </span><span class="p">(</span><span class="nx">maze</span><span class="p">[</span><span class="nx">nz</span><span class="p">]</span> <span class="o">&amp;&amp;</span> <span class="nx">maze</span><span class="p">[</span><span class="nx">nz</span><span class="p">][</span><span class="nx">nx</span><span class="p">]</span> <span class="o">&amp;&amp;</span> <span class="nx">maze</span><span class="p">[</span><span class="nx">nz</span><span class="p">][</span><span class="nx">nx</span><span class="p">].</span><span class="kd">type</span> <span class="o">===</span> <span class="dl">'</span><span class="s1">W</span><span class="dl">'</span><span class="p">)</span> <span class="p">{</span>
                    <span class="nx">maze</span><span class="p">[</span><span class="nx">z</span><span class="o">+</span><span class="nx">dz</span><span class="p">][</span><span class="nx">x</span><span class="o">+</span><span class="nx">dx</span><span class="p">].</span><span class="kd">type</span> <span class="o">=</span> <span class="dl">'</span><span class="s1">P</span><span class="dl">'</span><span class="p">;</span>
                    <span class="nx">maze</span><span class="p">[</span><span class="nx">z</span><span class="o">+</span><span class="nx">dz</span><span class="p">][</span><span class="nx">x</span><span class="o">+</span><span class="nx">dx</span><span class="p">].</span><span class="nx">wall</span><span class="p">?.</span><span class="nx">position</span><span class="p">.</span><span class="nf">set</span><span class="p">(</span><span class="k">new</span> <span class="nx">hz</span><span class="p">.</span><span class="nc">Vec3</span><span class="p">(</span><span class="nx">maze</span><span class="p">[</span><span class="nx">z</span><span class="o">+</span><span class="nx">dz</span><span class="p">][</span><span class="nx">x</span><span class="o">+</span><span class="nx">dx</span><span class="p">].</span><span class="nx">x</span><span class="p">,</span> <span class="nx">maze</span><span class="p">[</span><span class="nx">z</span><span class="o">+</span><span class="nx">dz</span><span class="p">][</span><span class="nx">x</span><span class="o">+</span><span class="nx">dx</span><span class="p">].</span><span class="nx">y</span> <span class="o">-</span> <span class="k">this</span><span class="p">.</span><span class="nx">props</span><span class="p">.</span><span class="nx">wallHeight</span><span class="p">,</span> <span class="nx">maze</span><span class="p">[</span><span class="nx">z</span><span class="o">+</span><span class="nx">dz</span><span class="p">][</span><span class="nx">x</span><span class="o">+</span><span class="nx">dx</span><span class="p">].</span><span class="nx">z</span><span class="p">));</span>
                    <span class="k">this</span><span class="p">.</span><span class="nf">carve</span><span class="p">(</span><span class="nx">nx</span><span class="p">,</span> <span class="nx">nz</span><span class="p">,</span> <span class="nx">visited</span><span class="p">);</span>
                <span class="p">}</span>
            <span class="p">}</span>
        <span class="p">}</span>
    <span class="p">}</span>
</code></pre>

</div>



<p>Finally for our script to run we need to quickly add a <code>resetWalls</code> function, this function will just iterate the walls variable and set all grid cells to <code>W</code> resetting their positions.<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight typescript"><code>    <span class="k">private</span> <span class="nf">resetWalls</span><span class="p">():</span> <span class="k">void</span> <span class="p">{</span>
        <span class="k">for </span><span class="p">(</span><span class="kd">let</span> <span class="nx">row</span> <span class="k">of</span> <span class="k">this</span><span class="p">.</span><span class="nx">walls</span><span class="p">)</span> <span class="p">{</span>
            <span class="k">for </span><span class="p">(</span><span class="kd">let</span> <span class="nx">cell</span> <span class="k">of</span> <span class="nx">row</span><span class="p">)</span> <span class="p">{</span>
                <span class="k">if </span><span class="p">(</span><span class="nx">cell</span><span class="p">.</span><span class="kd">type</span> <span class="o">===</span> <span class="dl">'</span><span class="s1">P</span><span class="dl">'</span><span class="p">)</span> <span class="p">{</span>
                    <span class="nx">cell</span><span class="p">.</span><span class="nx">wall</span><span class="p">?.</span><span class="nx">position</span><span class="p">.</span><span class="nf">set</span><span class="p">(</span><span class="k">new</span> <span class="nx">hz</span><span class="p">.</span><span class="nc">Vec3</span><span class="p">(</span><span class="nx">cell</span><span class="p">.</span><span class="nx">x</span><span class="p">,</span> <span class="nx">cell</span><span class="p">.</span><span class="nx">y</span><span class="p">,</span> <span class="nx">cell</span><span class="p">.</span><span class="nx">z</span><span class="p">));</span>
                    <span class="nx">cell</span><span class="p">.</span><span class="kd">type</span> <span class="o">=</span> <span class="dl">'</span><span class="s1">W</span><span class="dl">'</span><span class="p">;</span> <span class="c1">// Reset type to wall</span>
                <span class="p">}</span>
            <span class="p">}</span>
        <span class="p">}</span>
    <span class="p">}</span>
</code></pre>

</div>



<p>Now if you return to your desktop editor and run preview mode, after you have waited for the maze to fully load you should be able to press the button and see the path drawn in the maze. You will however not spawn into the maze as we will implement that part next.</p>

<p><a href="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fg5h8ivt2mooy2p46ccy8.png" class="article-body-image-wrapper"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fg5h8ivt2mooy2p46ccy8.png" alt="Maze Carved" width="800" height="323"></a></p>

<p>To ensure the players spawn into the maze correctly and the finish line is accessible, we need to position our entities based upon the grid that has been generated. Inside the <code>Maze</code> class update the <code>racer</code> interval function to set the start and finish positions based upon the grid cells:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight typescript"><code>        <span class="nx">racer</span> <span class="o">=</span> <span class="k">this</span><span class="p">.</span><span class="k">async</span><span class="p">.</span><span class="nf">setInterval</span><span class="p">(()</span> <span class="o">=&gt;</span> <span class="p">{</span>
            <span class="k">if </span><span class="p">(</span><span class="nx">at_the_races</span> <span class="o">===</span> <span class="mi">0</span><span class="p">)</span> <span class="p">{</span>
                <span class="k">this</span><span class="p">.</span><span class="nx">walls</span> <span class="o">=</span> <span class="nx">walls</span><span class="p">;</span>
                <span class="k">this</span><span class="p">.</span><span class="nx">props</span><span class="p">.</span><span class="nx">startPosition</span><span class="p">?.</span><span class="nx">position</span><span class="p">.</span><span class="nf">set</span><span class="p">(</span><span class="k">new</span> <span class="nx">hz</span><span class="p">.</span><span class="nc">Vec3</span><span class="p">(</span><span class="k">this</span><span class="p">.</span><span class="nx">walls</span><span class="p">[</span><span class="mi">1</span><span class="p">][</span><span class="mi">1</span><span class="p">].</span><span class="nx">x</span><span class="p">,</span> <span class="mi">0</span><span class="p">,</span> <span class="k">this</span><span class="p">.</span><span class="nx">walls</span><span class="p">[</span><span class="mi">1</span><span class="p">][</span><span class="mi">1</span><span class="p">].</span><span class="nx">z</span><span class="p">));</span>
                <span class="k">this</span><span class="p">.</span><span class="nx">props</span><span class="p">.</span><span class="nx">finishPosition</span><span class="p">?.</span><span class="nx">position</span><span class="p">.</span><span class="nf">set</span><span class="p">(</span><span class="k">new</span> <span class="nx">hz</span><span class="p">.</span><span class="nc">Vec3</span><span class="p">(</span><span class="k">this</span><span class="p">.</span><span class="nx">walls</span><span class="p">[</span><span class="k">this</span><span class="p">.</span><span class="nx">walls</span><span class="p">.</span><span class="nx">length</span> <span class="o">-</span> <span class="mi">2</span><span class="p">][</span><span class="k">this</span><span class="p">.</span><span class="nx">walls</span><span class="p">[</span><span class="mi">0</span><span class="p">].</span><span class="nx">length</span> <span class="o">-</span> <span class="mi">2</span><span class="p">].</span><span class="nx">x</span><span class="p">,</span> <span class="k">this</span><span class="p">.</span><span class="nx">props</span><span class="p">.</span><span class="nx">wallHeight</span> <span class="o">/</span> <span class="mi">2</span><span class="p">,</span> <span class="k">this</span><span class="p">.</span><span class="nx">walls</span><span class="p">[</span><span class="k">this</span><span class="p">.</span><span class="nx">walls</span><span class="p">.</span><span class="nx">length</span> <span class="o">-</span> <span class="mi">2</span><span class="p">][</span><span class="k">this</span><span class="p">.</span><span class="nx">walls</span><span class="p">[</span><span class="mi">0</span><span class="p">].</span><span class="nx">length</span> <span class="o">-</span> <span class="mi">2</span><span class="p">].</span><span class="nx">z</span><span class="p">));</span>
                <span class="k">this</span><span class="p">.</span><span class="nf">sendLocalBroadcastEvent</span><span class="p">(</span><span class="nx">Events</span><span class="p">.</span><span class="nx">setGameState</span><span class="p">,</span> <span class="p">{</span> <span class="na">state</span><span class="p">:</span> <span class="nx">GameState</span><span class="p">.</span><span class="nx">Ready</span><span class="p">,</span> <span class="na">winner</span><span class="p">:</span> <span class="kc">undefined</span> <span class="p">});</span>
                <span class="k">this</span><span class="p">.</span><span class="k">async</span><span class="p">.</span><span class="nf">clearInterval</span><span class="p">(</span><span class="nx">racer</span><span class="p">);</span>
            <span class="p">}</span>
        <span class="p">},</span> <span class="mi">100</span><span class="p">);</span>

</code></pre>

</div>



<p>Now before you try out this code, lets quickly fix the <code>Loading</code> phase of our game engine, if you remember we temporarily added some code in our <code>GameController</code> to automatically move into the <code>Ready</code> state but now we have the maze rendering and the event written we can remove this code. Open <code>GameController.ts</code> and remove the following line from the <code>start</code> function:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight typescript"><code>        <span class="k">this</span><span class="p">.</span><span class="nf">setGameState</span><span class="p">(</span><span class="nx">GameState</span><span class="p">.</span><span class="nx">Ready</span><span class="p">,</span> <span class="kc">undefined</span><span class="p">);</span>
</code></pre>

</div>



<p>Save your files, return to your desktop editor and allow for the scripts to compile. When you now enter preview mode, you will see the text will display 'Loading the Maze...' while you wait for the maze grid to be generated. Once generated you can press the button and you will be teleported into the maze, once you reach the finish line you will be declared the winner. We have a working Maze running game.</p>

<p><a href="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Flmw5eyqlgmkh88gnzr9o.png" class="article-body-image-wrapper"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Flmw5eyqlgmkh88gnzr9o.png" alt="In Game Area" width="800" height="319"></a></p>

<p>Before we finish for this instalment, lets now tidy up our game world. We no longer need the outer walls or floor for the game area so these objects can be removed. You may also want to move the finish object so it is not in view from the lobby while the maze is rendering.</p>

<p>In the end your game world should now look something like this:</p>

<p><a href="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Ftec6ph1tc3a3o8tfufn0.png" class="article-body-image-wrapper"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Ftec6ph1tc3a3o8tfufn0.png" alt="Final Preview" width="800" height="323"></a></p>

<p>This concludes this part of the tutorial. You now have a fully functional maze game with a randomly generated maze each time you play. In the final instalment of this series, we'll look at adding NPCs to race against within your maze.</p>

