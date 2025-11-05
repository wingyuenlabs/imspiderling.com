---
Title: sshOuroboros multiplayer game in goLang with ssh, and the story.
Description: 
Author: Misha Shelemetyev
Date: 2025-11-05T21:44:56.000Z
Robots: noindex,nofollow
Template: index
---
<h2>
  
  
  GIMME THE MEAT
</h2>

<p><a href="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fraw.githubusercontent.com%2FMShel%2FsshOuroboros%2Fmain%2Fstatic%2FgameRecord.gif" class="article-body-image-wrapper"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fraw.githubusercontent.com%2FMShel%2FsshOuroboros%2Fmain%2Fstatic%2FgameRecord.gif" width="560" height="420"></a></p>

<p>sshOurboros -- multiplayer game where snake eats itself and grows</p>

<p><strong>To play go here <code>ssh web2u.org -p6996</code></strong></p>

<p><em>The Rules:</em></p>

<ol>
<li><p>The goal is to claim the most space</p></li>
<li><p>Secondary goal is to kill as many other Ourboroses as you can<br>
<em>The how:</em></p></li>
<li><p>To claim space you need to either eat your own tail or reach tiles you've already claimed,<br>
tiles that are enclosed when you do so become yours!</p></li>
<li><p>To kill other snakes you hit their tails<br>
<em>To watch out:</em></p></li>
<li><p>Other players can kill you by hitting your exposed tail</p></li>
<li><p>Other players can take your tiles.</p></li>
</ol>

<p>The source: <a href="https://github.com/MShel/sshOuroboros/tree/main" rel="noopener noreferrer">https://github.com/MShel/sshOuroboros/tree/main</a></p>

<h2>
  
  
  Walk through on how it works and why
</h2>

<p>When you Sign Up to the game you are presented with 254 Ansi colors<a href="https://github.com/MShel/sshOuroboros/blob/db5cd9fdc0d1e674d6fc56a9b3fc3ad0a863a8ef/internal/ui/SetupForm.go#L117-L124" rel="noopener noreferrer">or maybe less if there are active players in the game right now</a>. <br>
Why 254? glad you asked! Thats what <a href="https://en.wikipedia.org/wiki/ANSI_escape_code" rel="noopener noreferrer">we have in terminal</a> - (void color + wall color).</p>

<p>But now that you've signed up there are players there! What the heck? all the colors were available. Well that's bots -- we try to keep all 254 colors busy at all times sshOuroboroses never sleep as <del>cool</del> kids say.</p>

<p>You joining the world of sshOuroboros actually already caused drama and we had to kick the <a href="https://github.com/MShel/sshOuroboros/blob/db5cd9fdc0d1e674d6fc56a9b3fc3ad0a863a8ef/internal/game/GameManager.go#L228-L230" rel="noopener noreferrer">bot with your color out</a></p>

<p>Alright we are in and passing commands back and fro through your <a href="https://github.com/MShel/sshOuroboros/blob/db5cd9fdc0d1e674d6fc56a9b3fc3ad0a863a8ef/internal/game/GameManager.go#L100-L102" rel="noopener noreferrer">players channel</a><br>
And that's fantastic but now we are getting to the meat of the game which is - how do we claim the territory? Well one AI would say its been solved bazzillion times you do flood fill by defining boundaries and calculating out of boundary tiles and swapping color for others....One would say that and it would not work? why-- because we have 254 bots that potentially need to scan <a href="https://github.com/MShel/sshOuroboros/blob/main/internal/game/Config.go" rel="noopener noreferrer">350_000 tiles</a> or cells on every 100ms move. That's going to be too slow.</p>

<p>So here comes my concurrent simplistic implementation:<br>
Imagine your ouroboros it can hit a tail or base from only 4 different sides and most of the time from one side it would be the area to claim and from other- the area you don't need to touch, so we need to explore only those 2(its not exactly this logic but ~90% of it). Now we throw 2 goroutines if one of them exited before finding the wall we kill another and we are done! <a href="https://github.com/MShel/sshOuroboros/blob/db5cd9fdc0d1e674d6fc56a9b3fc3ad0a863a8ef/internal/game/SpaceFiller.go#L53" rel="noopener noreferrer">link to code</a></p>

<p>The bots, oh those pesky bots in my first implementation I had a separate ticker for the bot master and it would be responsible for moving them around, that worked great till I started testing with 250 of them now it did not work at all =D the amount of kills of players that were killed and claims for tiles by bots who were killed was astonishing so I've scrapped that and now we assign strategy directly to player and run bots strategies right in the main loop. <a href="https://github.com/MShel/sshOuroboros/blob/db5cd9fdc0d1e674d6fc56a9b3fc3ad0a863a8ef/internal/game/GameManager.go#L213-L218" rel="noopener noreferrer">link</a></p>

<p>Talking about bots when I started I had this plan of multiple strategies and random selection of strategy based on bot situation........but ain't got time for that so I have this little Default strategy file that is Actual If conditions all around with some <a href="https://github.com/MShel/sshOuroboros/blob/db5cd9fdc0d1e674d6fc56a9b3fc3ad0a863a8ef/internal/game/DefaultStrategy.go#L9" rel="noopener noreferrer">ridiculous logic</a><br>
Now the nice part it is an implementation of interface that is used in <code>*Player</code> which theoretically unblocks me from saying later -- yeahyeah its bad but I can throw it away and rewrite. Also that is one of the areas I see as future major change in this project(see whats next? section below)</p>

<p>We have this 350_000 tile thing going on with 254 players on it how the heck do do we show it on the terminal you ask? well we cant -- not on most terminals, we need viewport of limited width and height and hopefully bother bubbletea(the rendering lib) update only with small portion of it. [here] we ask for chunk of map (<a href="https://github.com/MShel/sshOuroboros/blob/db5cd9fdc0d1e674d6fc56a9b3fc3ad0a863a8ef/internal/game/GameManager.go#L304-L331" rel="noopener noreferrer">https://github.com/MShel/sshOuroboros/blob/db5cd9fdc0d1e674d6fc56a9b3fc3ad0a863a8ef/internal/game/GameManager.go#L304-L331</a>) And <a href="https://github.com/MShel/sshOuroboros/blob/db5cd9fdc0d1e674d6fc56a9b3fc3ad0a863a8ef/internal/ui/GameView.go#L233-L270" rel="noopener noreferrer">here</a> we render it.<br>
This whole view function is kinda nasty and was a huge pain to write but it works most time anyway. </p>

<p>So here you go that's the key parts of the game, there are  a lot more areas to cover but those are the most interesting ones IMHO(ask me about anything else and answer I will)</p>

<h2>
  
  
  Whats In the Future
</h2>

<p>Well here I need your dear Reader advice, I've got following ideas that I'm planning to think about next, but if something completely different comes to your mind(hopefully about the game) do let me know. Anyway, help me pick!</p>

<ul>
<li>New game play as Bots strategy (add lua VM and allow strategy to be small executable lua script to be submitted by user)</li>
<li>Variable ouroboros speed press <code>x</code> to go real Fast type of situation</li>
<li>More games -- we have one with 256 players we can easily provision many with 256, though my server will have to be improved(its 10$ on linode and its meh)</li>
</ul>

<h2>
  
  
  Why did I write it?
</h2>

<p>Well Ai and <code>terminal.shop</code>... Ai is to blame for it, I <del>was</del> am slightly depressed by on slot of claims that AI will take over everything I like and make me write documentation and that is my 2'd least favorite thing in this career(1't are useless meetings that only end up in more useless meetings).</p>

<p>So hearing all that and watching Prime on youtube talking about how awesome ssh coffee stuff is I was like "lets try to write an ssh game cause its fun and be open minded about AI".</p>

<p>Now the good -- Ai wont take my job for now and that's not only because I tend to be very openly opinionated about things but also because:</p>

<ol>
<li>Flood fill -- it could not figure it out, it did provide classical algorithms that did not scale but it was of no help at the end</li>
<li>Concurrency issues -- It can not figure those out like at all, the game had a lot of those in my original iterations, ai could not suggest any sort of fix that would address performance bottlenecks</li>
</ol>

<p>Also good'ish? -- Ai was good at helping me with code I did not want to write like view files, I just dont care about them enough<br>
Or the HighScore sqlite crud thingy, that also was pretty much all AI</p>

<p>Now the bad -- I can write 250 more articles about how AI cant really replace anyone but it wont help the narrative been pushed by those weird AI CEO's :-(</p>

