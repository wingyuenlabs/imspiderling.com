---
Title: My Solo GameDev Toolkit
Description: 
Author: Trent Polack
Date: 2025-08-21T20:24:26.000Z
Robots: noindex,nofollow
Template: index
---
<p>Ever since I picked up solo dev again, I’ve been far more active in analyzing how I work and more meticulous in reading the news outside my core circle and finding new ways to work smarter or more efficiently. This is the result of that work thus far and it is, I’m sure, far from complete.</p>

<p>It’s worth noting in advance: the “solo” part of this article’s title is very purposeful. I won’t claim to be truly multidiscipline but I do work across the gamut of gamedev tasks which means I’m context-switching a lot and having to learn a lot of new programs to keep up. This takes an active effort, but it’s one I think is worthwhile because not only does it afford opportunities to find clever workflows and automations but, hey, it’s also fun. It’s also worth noting that I make no attempt to be a minimalist in my toolkit; if I find a piece of software that fulfills a use, however infrequent, it gets added to the taskbar for a rainy day.</p>

<p>Anyway, let’s start with the heavy hitters and work our way down. I don’t plan on this to be an exhaustive treatise on each piece of software, more of just an awareness-boosting post.</p>

<h2>
  
  
  The All-Star List
</h2>

<h3>
  
  
  Visual Studio
</h3>

<p>It’s been critical to the development process as long as I’ve been in games and nothing about that has changed except that it no longer costs $650 for a copy regardless of student, professional, etc. Gone are the days of the educational version where any console program ended with a note that it was built with the educational copy of Visual Studio. </p>

<p>But Visual Studio is borderline omnipresent. It has integrations everywhere, top-tier extensions (Visual Assist X), all major game engines support it, and it’s just a generally improved piece of software than it’s ever been in the past and that trend looks to continue going forward.</p>

<h3>
  
  
  Houdini
</h3>

<p>Houdini has seen an absolute renaissance over the last few years and it all kicked off with the Houdini Engine and SideFX’s renewed commitment to making the software useful for game developers (it was previously aimed at the film industry). Houdini is one of the most complex pieces of software I’ve ever used but with that complexity comes some freedom to create the most powerful tools and extensions in the industry that cover:</p>

<ul>
<li>Polygonal Modeling — The base polygonal modeling and modification tools are good but can be further enhanced with MOPS, MOPS+, and qLib.</li>
<li>Axiom — Creates high-fidelity fluid sims on the GPU.</li>
<li>Redshift — Sometimes it’s just necessary to make beautiful renders.</li>
<li>
<strong>Houdini Engine</strong> — This is the big one: Houdini has plugins for [most] major game engines called the Houdini Engine and it allows for creating digital assets in Houdini that are parameterized at creation time and then execute custom logic (along with allowing further customization) once imported into your engine of choice.</li>
</ul>

<h3>
  
  
  Gaea 2
</h3>

<p>As far as I’m aware, procedural generation of terrain datasets really hit the big time with World Machine a long while back. There was a slight problem with World Machine in that it was deeply unpredictable with a lengthy turnaround between iterations. A lot of the woes with World Machine were remedied with an extension called GeoGlyph, developed by a company called QuadSpinner.</p>

<p>Well, QuadSpinner wasn’t content to leave things at that and they came out the door with their own competitor to World Machine: Gaea. At the time one of the major hooks was that the generation process could live on the GPU and each step along the generation could be analyzed in detail without having to walk away for an hour and hope for the best. Gaea succeeded on its high-level goals, but it wasn’t a killer program.</p>

<p>Gaea 2 changed that; it overhauled a lot of the core interface, it sped up the response rate of everything, it provided more layout options, it added more new nodes that neither World Machine nor GeoGlyph had, and it provided almost all the flexibility in a heightmap-based terrain editor you could ask for. It also has solid plugins and integrations with software apps (Houdini) and Unity and Unreal.</p>

<h3>
  
  
  World Creator
</h3>

<p>Gaea 2 is step one for creating new landscapes and World Creator is the immediate next step where you can really put all of the disparate output from the generation process together into a cohesive scene (see below).</p>

<p><a href="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fm3bplbq7oy7svaz3n0bc.png" class="article-body-image-wrapper"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fm3bplbq7oy7svaz3n0bc.png" alt="The Americana landscape in Gaea 2." width="800" height="354"></a></p>

<p><a href="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2F80cx2eh40lm9pplwovwi.jpg" class="article-body-image-wrapper"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2F80cx2eh40lm9pplwovwi.jpg" alt="The Shape Designer in World Creator." width="800" height="472"></a></p>

<p><a href="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2F94d0hbs5j5jfifycnrit.jpg" class="article-body-image-wrapper"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2F94d0hbs5j5jfifycnrit.jpg" alt="A rendered view of the Americana landscape in World Creator." width="800" height="472"></a></p>

<p>Whereas everything in Gaea 2 tends to be a destructive operation, World Creator lets you assemble all of the pieces of data and content you want progressively and iterate as much as you want along the way. You can then use the World Creator Bridge to sync the state of that project with the engine of your choice.</p>

<p>With World Machine, you’re also not locked into the data that gets exported from a program like Gaea 2; you can apply new Noise filters, run object and world feature simulations, apply new masks and define new stamps/sculpts, etc. It’s essentially a tremendously powerful postprocessing step for one of the most complex features that’ll end up in your game world (so the time is worth it).</p>

<p>There are issues with the Houdini plugin at the moment, but there’s also a step for Houdini in the Gaea/World Creator process as well that uses Houdini Engine.</p>

<h3>
  
  
  Photoshop
</h3>

<p>Photoshop, because, yeah. Obviously. Though one recommendation is getting the EXR-Io plugin, especially when working with high detail images (e.g. terrain heightmaps).</p>

<h3>
  
  
  Substance Designer
</h3>

<p>A procedural material/texture creation tool is all someone like me could ever ask for in this world. It lets me create some basic textures that I don’t have the skills in Photoshop (or some such) to do in any traditional way.</p>

<p>And, much like Houdini’s Digital Assets, it’s great setting up a Substance file and relying on plugins into software and game engines to do runtime customization of parameters to tweak the output texture file. Though, and I can’t stress this direly enough, the Unity plugin needs work.</p>

<h2>
  
  
  Critical Additions
</h2>

<h3>
  
  
  Visual Studio Code
</h3>

<p>I was very tempted to include VS Code as an all-star program but, really, there is a lot of overlapping functionality to the point that minimalists don’t need both and Visual Studio has more critical integrations.</p>

<p>That said, VS Code is the runner-up; it’s lightweight, it has a mindblowing extension library, it supports all major modern languages and frameworks, and it can be anything from a light VS replacement (emphasis on light; it’s no replacement for a proper Solution) to just a really handy series of files to keep open at all times.</p>

<h3>
  
  
  Kitbash3D/Cargo
</h3>

<p>This is a hard fit for the list since it’s more of an asset marketplace than software but the quality of its offerings is so high and it’s so well-integrated with game engines and software that it’s impossible to not recommend. The volume and variety of content that’s been assembled since the service launched is also absolutely astounding. </p>

<h3>
  
  
  Marmoset Toolbag
</h3>

<p>InstaMAT Studio (below) is more user-friendly and I think I can actually use it properly but Marmoset Toolbag is my model-viewer of choice. It offers a lot of great rendering options, it has good material customization, it has a variety of import/export options, and it’s nice and speedy.</p>

<p>Now, I rarely use any sort of High-to-Low Poly Bake process because I’m never in a position to a properly UV-wrapped mesh much less a high-poly sculpt to use just to get optimal data. Since that’s one of the areas where Toolbag specializes in, it’s worth considering whether or not the uses I outlined above would be enough to warrant a purchase if you’re not going to use the baking much.</p>

<h3>
  
  
  InstaMAT Studio
</h3>

<p>This program is new to me and it’s something I discovered while trying to figure out a good low poly pipeline; it’s a nice, powerful, simple material creation package similar to something you’d find in Marmoset Toolbag but a bit more user-friendly. </p>

<p><a href="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fi3msazzj3mbp4esu8x86.webp" class="article-body-image-wrapper"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fi3msazzj3mbp4esu8x86.webp" alt="InstaMAT with a low-poly tree." width="800" height="481"></a></p>

<h3>
  
  
  Windows Terminal
</h3>

<p>Windows Terminal is a highly customizable terminal that allows you to choose from multiple shell types in a single session and keep them sharing a common aesthetic and properties. My recommendations:</p>

<ul>
<li>Windows Terminal</li>
<li>Powershell — No particular build in mind, I just think Powershell is ace.</li>
<li>oh-my-posh extension</li>
<li>Starship — I didn’t end up personally liking this but I can see the appeal.</li>
<li>Mega Collection of Powershell Scripts</li>
</ul>

<h2>
  
  
  Honorable Mentions
</h2>

<p>None of these applications are going to make or break your workflow but I’ve found them invaluable in moment-to-moment work:</p>

<ul>
<li>ShareX — A great open-source screen shot and video clip gathering tool.</li>
<li>Numi — A “smart scratchpad” for doing math equations in written form and keeping a history going; basically, it’s Notepad but smart.</li>
<li>WakaTime — A series of plugins that keeps track of your development (<a href="https://1drv.ms/b/c/53570b69628e20c4/IQSksSIxmw2JTqscTlHrRxlaAZBFQ5T94qwIbwkfVuSU1jE" rel="noopener noreferrer">here’s a link to my recently-output PDF</a>) and where the time goes. It’s not 100% accurate, but I love metrics and I’ve found that WakaTime keeps me responsible for my time a bit more than I would on my own.</li>
<li>GitKraken — I use Git from the command line about 85% of the time but, sometimes, I just want something visual when I’m working with my repos and GitKraken has been the best Git client that I’ve found so far. It has some latency issues but, for the most part, it’s reliable and feature-rich.</li>
<li>The JangaFX Element Suite — EmberGen, LiquiGen, GeoGen, and Illugen are all great software that each fulfill a unique service in a user-friendly way. They’re currently held back by still being relatively early in development across the board but the variety of functionality, quality of output, and price make them a great solution for VFX.</li>
<li>Daily.Dev — I just discovered this yesterday but it’s already my favorite news aggregator. I love it.</li>
<li>PopcornFX 2 — I was a fierce advocate of PopcornFX 1; I think its script-based approach to particle emitter creation was incredibly strong, clever, and it let someone like me mesh the left and right side of my brain to create actual good emitters. PopcornFX 2, however, is a complete overhaul that went in an entirely node graph editor-based approach and changed the workflow paradigm entirely and I’ve… yet to warm up to it. I’ve still got hope though.</li>
</ul>

