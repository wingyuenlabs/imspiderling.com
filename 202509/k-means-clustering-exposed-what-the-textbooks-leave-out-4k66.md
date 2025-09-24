---
Title: K-Means Clustering Exposed: What the Textbooks Leave Out
Description: 
Author: The Algorithm Age
Date: 2025-09-24T22:08:46.000Z
Robots: noindex,nofollow
Template: index
---
<h2>
  
  
  Social Media:You can’t escape it
</h2>

<p>In 2022, 14-year-old Molly Russell tragically took her own life. Her father, Ian Russell, described how his daughter was drawn into social media. Just two minutes before her death, she had saved an image on the platform that carried a depression-related slogan. Ian Russell said: “<em>It’s a world I don’t recognise. It’s a ghetto of the online world that once you fall into it, the algorithm means you can’t escape it and it keeps recommending more content. You can’t escape it.</em>”</p>

<p>At the inquest into her death, Elizabeth Lagone, Head of Health and Wellbeing Policy at Meta, the owner of Instagram and Facebookm, was questioned. The coroner noted that algorithms were used to provide content alongside adverts and included this in the list of contributing factors in Molly’s death. The coroner also ordered that Pinterest, Meta, Snap, Twitter, and others should act to prevent future deaths. The Daily Telegraph reported that Facebook “<em>has no data</em>” on how the “<em>dreadful</em>” posts shown to Molly were selected.</p>

<p>This tragedy occurred even after Meta had already added 8,750 staff dedicated to content moderation. These moderators had been recruited following a federal inquiry into Russia’s political advertising purchases on Facebook and election interference.<br>
Social media’s impact feels self-evident. It seems hard to argue that it does not affect people, unless, that is, you visit a computer science department and attend a lecture on algorithms. Algorithms are both part of the problem and possibly part of the solution.</p>

<p>The purpose of the algorithm age is to examine technology from both the outside and the inside. In doing so, we gain a new perspective on the entanglement of technology and society.<br>
Today, I want to talk about how, and why, social media bubbles form. We are going to approach this from an algorithmic point of view. Let’s start with one question.</p>

<h2>
  
  
  Do you know how TikTok works?
</h2>

<p>Not how you interact with it, , that’s human–computer interaction. Not how you become a social-media influencer; that’s more down to good teeth, media savvy, and marketing know-how. Do you know how TikTok works on the inside? I doubt anyone can honestly say yes to that. I mean, if Facebook knew how the algorithm pushed Molly, don’t you think they would tweak it to stop that happening? Why hire 8,750 human beings to moderate content if you could simply tweak the algorithm?</p>

<p>I’m going to let you in on a little secret: the reason Facebook, or any other social-media company, can’t stop things happening on their platforms is likely because they don’t know how to. After all, the people working there are not monsters. If they knew, they would fix it. The choices are limited: either they know and won’t fix it, or they don’t know and remain silent. For me, the “don’t know” option is the simpler explanation, it helps me sleep at night. I’m pretty confident about this because there is nothing in the recommender-systems literature that explains why it happens.<br>
Another way of putting it is this: if there were an algorithmic tweak that would prevent these harms, they would have made it. Hiring 8,750 people and working them in sweatshop conditions is clearly Plan B.</p>

<p>So how does TikTok work? Before I explain, I should say I’m focusing on TikTok because they have published papers on the subject, so we have some idea of what they are thinking about. The algorithm I’m about to describe is fairly common and is also used in data science, not to mention in scientific research more broadly.</p>

<h2>
  
  
  Your feed in crisis
</h2>

<p>"At the core of any social-media website lies the problem of what to show you next. There is a vast amount of content out there. Which piece should appear next on your screen?<br>
At the centre of this process is what computer scientists call recommendation. Godfather of new media and MIT guru Nicholas Negroponte explained it neatly: “<em>When I want to know what movie to go see, I phone my sister, because she knows movies and she knows me</em>.”</p>

<p>Personally, I always wondered how Negroponte’s sister felt about these phone calls, but I digress.</p>

<p>Now put yourself in the seat of social media. The system has about 50 milliseconds to find something that will engage the user. How does it do this? More importantly, how do you do this when you don’t understand the content you are showing? Remember, you’re a social-media algorithm. You can’t actually read the content of the messages. An algorithm won’t understand the semantic meaning of what is written.</p>

<p>Try this yourself. You’re working in an alien bookshop on Arcturus.You don’t read Arcturan just like a machine can't read videos.  These aliens have two qualities: they always lie, and they hate answering questions. An alien walks in and asks for a book they will like. What do you do? You can’t ask them, they will lie. They’ll usually say whatever they think you want to hear. To be honest, they don’t know themselves. So don't ask.  Meanwhile, the 50-millisecond clock is ticking.</p>

<p>Fortunately, you have a good memory. This particular alien is cyan-coloured. You recognise them: they have previously bought books from the right-hand side of the upper shelf. So you go to that shelf, pick a book you are sure they haven’t read, and show it to them. Forty-nine milliseconds gone. It’s a new book. They like it.  Success.</p>

<p>Another alien walks in. You don’t know their tastes, but you’re starting to see patterns. This one is also cyan-coloured, so you try books from the right-hand side of the upper shelf. A sale. Success again!</p>

<p>This is how TikTok, and recommendation more broadly, works. You group books together, and you group people together. If one group of people likes one group of books, then when someone similar walks in, you match them to the group and to the content they are likely to enjoy. Grouping like with like is the real problem.</p>

<h2>
  
  
  James MacQueen
</h2>

<p>So you might be asking: how does an algorithm do this? As always with algorithms, there are many possible approaches. The one we are talking about was set down by James MacQueen but, like most algorithms, it was built on a great deal of earlier work su as Hugo Steinhaus in 1956. MacQueen was a statistician at UCLA in the 1960s. </p>

<p>The MacQueen algorithm requires numbers, so stand by for a short diversion into converting alien books or text messages into numbers.</p>

<p>The first thing to acknowledge is that everything exists on a spectrum. By this I mean that it has measurable qualities. For example, we can measure how angry a message is. We might assign it a percentage from 0 to 100%. Researchers in computational linguistics have been developing sentiment analysis for years. This is a way for a machine to approximate emotions in language.<br>
Despite many uninformed comments to the contrary, machines are actually quite good at recognising emotional tone. Even before the current wave of AI, you could use a library to analyse a comment and determine its emotional flavour. For example, if it is written in all caps  <strong>THEN IT’S MAKING A VIVID EMOTIONAL POINT.</strong> There are angry words, sad words, funny words, happy words, sentimental words. From this, a message can be represented as a set of percentages across different emotional categories.</p>

<h2>
  
  
  Meta information
</h2>

<p>Beyond this, there are many mechanical bits of information. For example: when was the message sent? Was it a reply? During the Second World War, the British became highly skilled at examining the information attached to encrypted, unreadable messages. Just before an attack, for instance, they might notice an increase in the number of messages being sent. If there was a sudden surge of traffic in a given area, it was a sign to prepare for an attack. In a more complex example, when the Nazis sent a message it included a plain text destination address. By keeping extremely detailed dossiers on everyone and everything in the enemy army, the British could infer what was happening.</p>

<p>This is how Netflix and many other sites work. And it’s exactly what you were doing when you looked at the colour of the alien: you were using what is called meta-information.<br>
So, back to our main algorithm. Our aliens have two primary emotions: Aaarkness and Babambo. A message might be 25% A and 34% B, or 100% A and 0% B, or any combination in between. For any message, we can plot a point on a map according to its levels of Aaarkness and Babambo. It’s like latitude and longitude, or those pirate maps you made at school: so much across, so much down, the pin goes there.</p>

<p>Do this for many messages, and you begin to see clusters. It’s easy to spot with your eyes, but remember, we need an algorithm, they are blind. How do we “see” these groups with numbers?<br>
Here are the rules: each point (each pin) belongs to a group. The points hang like a swarm of gnats around a centre. To make things harder, the groups sometimes overlap. At the centre is a pure Arcturan idea, concept, or feeling. Take what we might call regret. If you read the messages with 90% A and 80% B, your local Arcturan friend will tell you these are books or messages that contain regret. Regret fiction is big on Arctura. No book is perfectly regret fiction, but anything close to 90% A and 80% B will be regret-ish. So these groups represent meaningful categories in Arcturan.</p>

<p>For a more practical example: looking at plants  suppose you measured leaf length and leaf width. The resulting groups would correspond to different species of plant.</p>

<h2>
  
  
  Get back to the point: how do we find groups?
</h2>

<p>In 1966, James MacQueen was thinking about exactly this. He wanted to identify the “clouds” of points and realised that meant finding the centre of each cloud. But he faced a problem: he didn’t know which item belonged to which group.</p>

<p>So he thought: suppose I know there are five groups. Then there must be five centres. At this point, he made a simple rule: each point (each pin on the map) belongs to the nearest centre. The only problem was—he didn’t know where those centres were. Then came the moment of inspiration: what if I don’t care? What if I just start with five random centres?</p>

<p>Here’s how it works. Start with five random centres. Apply the rule: for each pin, test which centre it is closest to. If the pin is nearest to Centre 5, then it gets added to Centre 5’s group of pin. Once that’s done, each centre has a list of pins that belong to it. Some groups will be larger than others. That’s fine. It’s even possible for a centre to end up with no pins at all, though usually there’s some division.</p>

<p>Next, for each group, calculate a new centre by finding the average of the points in that list. This is like finding the centroid of the group. Because the values are numbers, this is simple:</p>

<ul>
<li>take all the A values of the pins in the group and find their mean;</li>
<li>then take all the B values and find their mean.</li>
<li>The result is a new A–B coordinate: the updated centroid for that group. </li>
</ul>

<p><strong>Now comes the clever part</strong>: recompute which pins belong to which group, based on these new centres. Then rinse and repeat. Recalculate the means of the groups, move each centre to its new mean, and assign the pins to the nearest centre again. Keep going until the membership of the groups stops changing.<br>
The good news is that this usually settles down in about six to ten rounds.</p>

<h2>
  
  
  What works for two dimensions also works for a thousand dimensions
</h2>

<p>The magical part is that you don’t have to stop at just two emotions. Suppose you did everything in three dimensions—you can still measure the distance between two points. From this, you can work out which is closer, and with that you can decide which group to put it in. And you don’t need to stop at three; you could do four, five, six, a few hundred the sky is the limi. Let me give you a concrete example. We are going to add in the Arcturan emotion, Cagraki, for humans its like that desire people have to try and touch a pregnant woman's swollen belly and feel they have permission to do so. Time to fire up a spreadsheet. <br>
Let’s say we have a current centroid at (10A, 35B, 90C). We also have a point at (12A, 30B, 93C). How close is it? If the first values are 10A and 12A, the difference is –2 (10 – 12). Squared, that gives 4. For the next values, 35B and 30B, the difference is 5 (35 – 30). Squared, that gives 25. For the C values, 90 – 93 = –3, squared is 9. The sum of these squared differences is 4 + 25 + 9 = 38. The square root of this is about 6.16, so the distance between the two points in three dimensions is 6.16, as if measured with string. You can keep adding dimensions in exactly the same way.</p>

<p>Because you repeatedly find the means to define the new centres of the groups, and because you fix the number of groups (called k) in advance, you get the name k-means clustering. In our case, k = 5.<br>
In most textbooks or explanations of k-means clustering, the explanation stops here. They often go off on a tangent about how to determine k if you don’t know it in advance. (Spoiler: you try lots of different values of k.)</p>

<p><strong>But I want to stop and admire the algorithm</strong>. It’s quite simple: you can understand it, and if you know how to measure distances in a triangle, everything follows. And yet it does an amazing job. It doesn’t matter how messy your data is, you could have 1,000 dimensions, most of them irrelevant, and the method still digs through to the important ones and finds a good result.</p>

<p>The alternative, looking at every possible combination of assigning items to groups, is the black hole of efficiency known as NP-complete. k-means takes us from “almost impossible” to something that scales only with the number of data points. </p>

<p>I think I like it because it’s another one of those very graphic algorithms. If someone draws it on a whiteboard or a scrap of paper, bang, you get it.</p>

<h2>
  
  
  Where k-means goes wrong
</h2>

<p>This is about as much as your average data scientist knows about k-means clustering. What they don’t tell you in data-science school is that if you just follow the recipe, there’s a good chance it will give you the wrong result. We’ll see that below.</p>

<p>So let’s remember how we got here. Developers want to group people into “similar people.” The idea is that similar people share similar characteristics. The joy of k-means clustering is that you can just throw another dimension at it. </p>

<p>Take the case of books. If someone buys Squggle-Squiggle-slash—the third blue one on the top shelf (I said you can’t read Arcturan), then two people are considered similar if they’ve read the same book. So we create a dimension for that book—dimensions are cheap. Buying this book means you either end up with 100% if you own it, or 0% if you don’t. So people who prefer the same books are “closer” in our thousand-dimensional space. This makes k-means clustering very ignorance-proof, provided you stick to  rule of percentages for everything.</p>

<p>Can you see how the recommender works now? You cluster people together. The centre of the cluster will contain all the books the people in it share. So you just look for a book in the cluster centre that has, say, more than 90% ownership. If the person doesn’t already own that book, you suggest it.</p>

<p>You can go through the books in the cluster knowing there is something similar about them. This is how TikTok works: they put people into clusters based on messages, and they put messages into clusters based on people. If you’re in a cluster, you get shown content that other people in the cluster have responded to. You might ask what I mean by “has responded to”—and this is where different websites diverge. <strong>That’s the secret sauce.</strong></p>

<p>To be frank, the secret sauce keeps changing. People constantly game the system and find weaknesses, and that’s what gets patched. The recipe keeps shifting. They change it in the same way Coca-Cola keeps changing its recipe. (Yes, they do!) If you don’t believe me, go to the Coca-Cola museum in Atlanta: you can taste Coke from Egypt, which is sweeter, and Coke from Germany, which tastes different again, or compare them to Coke from the US. Both TikTok and Coke brew up new recipes and try them out on tasters. The only metric they really care about is: will people consume more? If not, forget the rollout.</p>

<h2>
  
  
  Cold start in November
</h2>

<p>Traditional books and research on recommendation often go off on a tangent about two things. The first is the so-called cold-start problem. How do you make recommendations when your start-up is brand new, has only a handful of users, and your vision is incomplete? It’s complicated, and over the years people have managed to make it even more complex.</p>

<p>The second issue is how to cope when you’re suddenly handling millions of tweets per day. These are performance problems. I’m not saying they’re minor, but they do show the myopic tendency of the field to hyper-focus on a narrow set of technical issues while often ignoring the broader social ones—including the big one we’re about to talk about.</p>

<h2>
  
  
  Snake-eyes experiment with Vivid-Solidary
</h2>

<p>Let’s try the following experiment—or simulation. Back to Arcturus. We’re building Arcturan TikTok. The aliens have two ways of expressing emotions (Aaarkness and Babambo), and there are seven types of Arcturan personalities. I can’t really describe them, but everyone agrees there are seven distinct personality types. I’ll give them human-style labels: aliens who express Vivid-Solidary like to see messages about cats; aliens who are Blue-Rain types prefer pictures of spades and spade-related news and opinions.</p>

<p>Of course, I can’t get data from Arcturus, so I’m going to do what all Silicon Valley tech-bros do: fake it till I make it.<br>
For this simulation, let’s generate fake data from known centres. This gives us a “gold standard” we can work back from to test whether the experiment works. We pick seven centres randomly. Then we generate points around each chosen centre using a bell curve. A bell curve creates more points close to the centre and fewer at the extremes. If you’ve ever played craps, rolling two dice, you know it’s more likely to add up to 7 than to 2 or 12—snake eyes. That’s the bell curve at work: the more dice you roll, the more the distribution smooths into a familiar hump.</p>

<p>Human traits often follow the same pattern: people’s heights form a bell curve, people’s weights form a bell curve, people’s intelligences form a bell curve. So faking data with a bell curve makes it look more like real data. In two dimensions, this creates a cloud (see figure 1).</p>

<p>This way, we can make a fake world where we know the correct answer. In the plots below, the magenta circle marks the true data centre—the one from which all the points were generated. Each data point is coloured according to its true generating centre. That gives us a gold standard against which we can test k-means clustering. A perfect system would find the true centres every time.</p>

<p>In practice, mistakes are inevitable. Some clusters overlap. Think of men’s and women’s heights. On average, men are taller than women. But we can all think of women taller than some men. The curves overlap, even though their peaks (the averages) differ. Height is informative for gender, but it’s not definitive.</p>

<p>From this, we can say the system will not always get it right. Sometimes it will misclassify individuals—“which cluster am I in?” But in reality, as we add more and more kinds of evidence, the system becomes more accurate. No single test is perfect, but with hundreds of factors, the accuracy improves dramatically.</p>

<p><a href="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fowa3ywrn3g5q6xmj61w9.png" class="article-body-image-wrapper"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fowa3ywrn3g5q6xmj61w9.png" alt="A plot in 2D space of the clusters on a black background" width="800" height="568"></a></p>

<p>** Figure 1. The clusters are they are generated ( Magenta points are the correct centres ) **</p>

<p>So let’s put in seven random points (figure 2). These are the squares with white borders. They represent the machine’s ‘guesses’ at the center points. Notice how the points (people) are now assigned to the nearest cluster. It’s wrong—but that’s fine, because it’s just a random guess.</p>

<p><a href="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2F554c7eoo6a87jei386ww.png" class="article-body-image-wrapper"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2F554c7eoo6a87jei386ww.png" alt="First  groups iteration" width="800" height="518"></a> ** Figure 2 – First random groups( squares).**</p>

<p>Next <br>
Let’s move forward one round:</p>

<ul>
<li>a) assign the points to the nearest cluster;</li>
<li>b) update the cluster centres.</li>
<li>The result is figure 3 </li>
</ul>

<p><a href="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fhjny79dxqhcv1m9j9fyk.png" class="article-body-image-wrapper"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fhjny79dxqhcv1m9j9fyk.png" alt="Update of clusters " width="800" height="589"></a><br>
** Figure 3 - firs iteration. Clusters move. **</p>

<p>This process continues the membership changes. In the good cases we move closer to the 'correct' origin.</p>

<p><a href="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2F8vcl18ukdpialrncrr2x.png" class="article-body-image-wrapper"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2F8vcl18ukdpialrncrr2x.png" alt="Final result" width="800" height="597"></a><br>
*<em>Figure 4 final clusters. With labels notice the difference between the 'found' clusters and the 'true' clusters. *</em></p>

<p>To quote Apollo 13: “Let’s start with what’s working, not what’s broken.” Ginger-Rage is what we generally want. Not perfect, but close enough. It’s still wrong—let’s just hope it’s not important. I see three groups. Notice that none of them are perfect. The algorithm has failed to capture the true essence of the groups. This may or may not be a problem depending on how the data is used. Bottom line: there is an error.</p>

<h2>
  
  
  Big errors
</h2>

<p>There are some serious problems. The broad categories suffer from split-ups, false unions, and fake groups.</p>

<h2>
  
  
  Split-ups – false diversity
</h2>

<p>The vivid-solitary group has been split into three subgroups. Their centres are nowhere near the true centre. This represents people being unfairly pushed into subcategories they might not feel represent them. So, for example, mature women could be divided into “post-menopausal delinquents” and “trad-mums.” Remember, these people will be shown more and more content that is supposedly “relevant” to them. Equally, they won’t be shown other, “cool” material they might actually have found engaging.</p>

<h2>
  
  
  False unions
</h2>

<p>The Blue-Rage and the Starbucks-Lovers have been merged. So people are being placed into a group they don’t really belong to. As a result, they’ll be shown material that is irrelevant to half the population.</p>

<h2>
  
  
  Synthetic bubbles
</h2>

<p>The final kind of error occurs when groups are not simply subsets of previous groups, or unions of two, but chimeras created from fragments of several groups. There isn’t a clear case of that in this version, but you can see one in the example below. Here, because we know the underlying pattern, we can recognise that the group is untrue and unrepresentative. In a scientific context, novelty might be valuable—but you would normally expect some kind of test to check whether the unexpected group is valid.<br>
This leads to a classic problem. It’s easy to dismiss people as stupid while assuming the machine is smart and that we were too dim to see it. We can be fooled into thinking novelty created by the machine is meaningful. We might even convince ourselves that “<em>the machine proves the existence of group X</em>.” It doesn’t.</p>

<h2>
  
  
  Solutions Kmeans++
</h2>

<p>There are really two solutions here—three, if you count the industry-standard approach of ignoring it.</p>

<p>First, there’s a method called k-means++. Instead of assigning centres randomly, you pick real data points: one at random, then the next one more likely to be further away from those already chosen. This reduces the problem but doesn’t eliminate it.</p>

<p>We get better results—more of the found centres are near the correct ones. But we still get errors.</p>

<p>So what happens in the real world? Each group of people is shown different content. What you’re seeing in the picture is the bubble the system builds around people. If you like gardening, you get more gardening messages.</p>

<p>But there are also errors. Large groups are sometimes split into two distinct subgroups. You’re into pets and children, but you only get shown pictures of pets—or only content about children. Groups can also be artificially merged. You’re worried about finding a girlfriend, and you’re worried about Mussolini’s reputation: in the synthetic space, one leads to the other.<br>
Perhaps—and I don’t have the real data to prove this—when people talk about conspiracy thinking leading into far-right thinking, what’s actually happening is this kind of merging of two groups.</p>

<h2>
  
  
  Increase diversity
</h2>

<p>Another “solution” is to run the whole k-means clustering algorithm, say, 20 times and pick the most common result. This is what happens if you use a standard package in Python this is what you get. </p>

<h2>
  
  
  The bottom line
</h2>

<p>The bottom line can be taken in two ways. From a business perspective, errors in k-means clustering mean that people are not being shown content they actually want to see. That creates an opening for businesses that can deliver better value.<br>
 From a human perspective, the groups in the machine do not reflect the groups in real life. Ask yourself: are you truly happy with the content you see? Turn the telescope around. If you sat down and listed everything you’d like to hear about on a daily basis, does your current social-media feed reflect that? Perhaps there’s an algorithmic reason it doesn’t.</p>

<h2>
  
  
  Do you feel you are in the right bubble?
</h2>

<p>Hopefully, you can see some of the problems. Remember: these aren’t dots, they are people. Perhaps you’re a bright teenager, going through the struggles that come with that age—finding yourself, having good moments but also natural lows. Perfectly normal. But in a busy house, those lows can feel lonely, isolating, hard to talk about. Click on something melancholic and the k-means clustering algorithm decides you’re closer to depression than to cats. The content shifts—and someone shifts with it.</p>

<p>Mainstream computer scientists often just see numbers and tables. They see 80 characters wide and 30 lines tall in the source code—a tiny letterbox view onto gigabytes of data. They make graphs and charts for their bosses. They have information but not insight. They have knowledge but not wisdom.</p>

<p>TikTok uses what is known as a latent space. This can guide the algorithms better, but the numbers are even more meaningless. The synthetic feelings are no different from Aaarkness and Babambo. We don’t know what they mean. We cannot relate to them.</p>

<p>On Facebook, Instagram, and all the other “socials,” the dots being pulled by algorithmic gravity along a path from melancholy to suicide are invisible. There are tens of thousands of these lonely dots, too many to see, and the algorithmic overlords are too blind to recognise them. All they can do is provide a “report this” button and hire humans to read the content. It’s like mowing a lawn one blade at a time. Poison weeds slip through, build up, and then one morning someone walks in to find their daughter dead on the bed.</p>

<p>This has been a long post. It was originally much longer. The second half introduces an experiment: if we acknowledge that the centres are used to select content, then the people themselves change and shift. That makes the errors of k-means clustering much worse.</p>

<p>Leave a comment if you’d like to see that second section. I also wrote the code from scratch to visualise the process and run the second experiment. If you’d like to see an interactive version, leave a comment and I’ll post a link.</p>

<p>That’s all for now. Stay tuned for more algorithmic content. Thank you and good night. <strong>AA.</strong> </p>

