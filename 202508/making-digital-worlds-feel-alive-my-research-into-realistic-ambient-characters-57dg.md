---
Title: Making Digital Worlds Feel Alive: My Research into Realistic Ambient Characters
Description: 
Author: Eric Buitrón López
Date: 2025-08-24T20:17:36.000Z
Robots: noindex,nofollow
Template: index
---
<h1>
  
  
  Overview
</h1>

<p>Have you ever had that odd feeling that we're all living in a simulation? If you have, you are not alone. Many people have thought about this, resulting not just in amazing sci-fi entertainment content but also in the <a href="https://simulation-argument.com/simulation.pdf" rel="noopener noreferrer">simulation theory</a> which is constantly being backed up by more people. Simulations need to be populated by digital characters to give some sense of realism. However, defining exactly what makes these characters realistic is very difficult since it can be very subjective. Nevertheless, it can be broadly divided into two main categories: how realistic their visual representation is (rendering and animations) and how realistic their behavior is.</p>

<p>That last category is the area which I’m more interested in, and what I’m currently researching as part of my master’s in computer science. In this blog post I will go over more details about my research topic and an overview of what I’m trying to accomplish.</p>

<h1>
  
  
  Video games as research domain
</h1>

<p>Simulations are used for many different things in a lot of fields. However, the area I’m most interested in is video games. Video games are probably the prime example of characters living in a simulation. Each game by itself creates a whole digital world (and sometimes universe) that you as a player get to interact and be part of. Games have a lot of different characters, and it would not be possible for me to focus on all of them during my master’s degree. Therefore, I decided that my research would focus on the behavior of ambient characters in video games. But why are video games particularly well-suited for this type of research?</p>

<p>Video games present unique challenges and opportunities for ambient character research. Unlike other simulations that might focus on specific behaviors or limited scenarios, games require characters to be believable across diverse situations (from peaceful market scenes to chaotic battles). Players also have unlimited time to observe these characters, making any flaws in behavior immediately apparent. This makes video games an ideal testing ground for developing robust ambient character systems that could eventually be applied to other simulation domains.</p>

<h1>
  
  
  What are ambient characters?
</h1>

<p>Characters in video games can be categorized based on their role, purpose, and level of interactivity. First, we have <strong>player characters</strong> which are controlled directly by human players. Then we have <strong>non-player characters (NPCs)</strong> which are any characters in the game world that are not controlled by human players. NPCs can be categorized based on their significance and functionality in a game. <strong>Main NPCs</strong> have significant narrative or gameplay importance in the game. They also tend to have important interactions with the player characters. Meanwhile, <strong>ambient NPCs</strong> (think of the pedestrians walking city streets or patrons sitting in taverns) populate the game world and create a sense of life and immersion. However, they tend to have very limited interactions or significance to the player characters.</p>

<p>You might be wondering, if they have very limited interactions or significance for players, why would I choose them as my research topic? Well, these are the characters that create an illusion of life within the game. They make the digital world feel alive and populated which increases the immersion of players. This is very important because immersion is what makes players believe, at least for some time, that they are actually part of the digital world the game offers.</p>

<p><a href="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fth.bing.com%2Fth%2Fid%2FR.426b2b63b9d79b132c9e131bc7986970%3Frik%3DvlV%252bbzFTaeH8bw%26riu%3Dhttp%253a%252f%252fcramgaming.com%252fwp-content%252fuploads%252f2013%252f09%252fgta-v-npcs.jpg%26ehk%3DHzimKdVgu0vRVUhMRO6za4YtMvYXmWL0z0kIF347zKQ%253d%26risl%3D%26pid%3DImgRaw%26r%3D0%2520align%3D" class="article-body-image-wrapper"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fth.bing.com%2Fth%2Fid%2FR.426b2b63b9d79b132c9e131bc7986970%3Frik%3DvlV%252bbzFTaeH8bw%26riu%3Dhttp%253a%252f%252fcramgaming.com%252fwp-content%252fuploads%252f2013%252f09%252fgta-v-npcs.jpg%26ehk%3DHzimKdVgu0vRVUhMRO6za4YtMvYXmWL0z0kIF347zKQ%253d%26risl%3D%26pid%3DImgRaw%26r%3D0%2520align%3D" alt="GTA V review - Cramgaming.com" width="800" height="450"></a></p>

<h1>
  
  
  Problems with current ambient characters
</h1>

<p>If you have played or seen someone play any game with ambient characters you might already be aware that the current state of these characters is not as realistic as we would hope. They lack believable persistence (characters don't remember previous interactions or maintain consistent states when players aren't looking) and exhibit repetitive behavior. Additionally, they also tend to have a poor understanding of their environment (they might not react appropriately to weather, crowds, or dramatic events happening around them) which makes them behave in a way that doesn’t make sense for what is happening in the digital world.</p>

<p>All of this causes what is called a break in realism for the players since they no longer believe that the digital world is “real”. For instance, you might see characters continuing their cheerful shopping routines while explosions happen just meters away, or watch the same character have an identical phone conversation every time you pass a certain corner. These moments instantly remind players they're in a digital simulation, breaking the carefully crafted immersion developers work so hard to create.</p>

<p>Some of the most recent open-world games (such as Red Dead Redemption 2) are implementing systems that address some of the limitations. For example, the ambient characters in this game have daily routines, can recognize the player from previous encounters, and react contextually to ongoing events in the world. However, these solutions tend to be very game-specific and the fact that ambient characters aren’t really the main focus of a video game will always remain true. Developers have to put most of their focus on other systems and on making sure that the game is fun (which should always be the priority).</p>

<p><a href="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fbb2km846epmdg637sc7c.png" class="article-body-image-wrapper"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fbb2km846epmdg637sc7c.png" alt="Valentine - Red Dead Redemption 2 Wiki" width="800" height="450"></a></p>

<h1>
  
  
  Ambient Character Behavior Framework
</h1>

<p>Given these significant limitations in current ambient character systems, there's clearly a need for a more sophisticated approach. This is where my research comes in. My goal with my research is to have more realistic ambient character behaviors. This can be seen as a focus on creating believable, varied, and contextually appropriate characters that enhance the player’s immersion of the game and makes them feel part of a living digital world.</p>

<p>My master's project focuses on proposing an ambient character behavior framework that addresses the current limitations. Rather than scripting countless behavioral variations, the framework enables characters to develop unique patterns based on their accumulated experiences and environmental awareness. Characters naturally avoid repetitive behaviors while maintaining meaningful persistence across player encounters - all without requiring individual programming for each character. While I can’t disclose the specifics of this framework yet, I can tell you that I’m currently working on implementing the first version of the framework and on creating a proof-of-concept demo. Hopefully, I’ll be able to show more progress about this soon.</p>

<h1>
  
  
  Looking forward
</h1>

<p>While I’m focusing my master’s project on the video games domain, I have a bigger vision for what I want for my framework. If everything works out, it would be very interesting to expand my research into other types of simulations that aren’t necessarily for entertainment. I believe that having more realistic ambient character behavior is the first step towards improving training, digital twins, emergency response, city planning and many other simulations. Not to mention, probably the most interesting application, enabling digital characters for the Metaverse.</p>

<p>With this in mind, I’ll keep working on my master’s project. Meanwhile, there are certain topics related to my research that I’m looking forward to writing about in other blog posts such as: the difference between game AI and academic AI, the importance of having user perspectives when defining the quality of character behavior and the challenges of cross-platform development in gaming. I hope you look forward to these future posts and more updates about my research in the coming months.</p>

<p>How do you imagine digital characters will be perceived in the next 5 years? I would love to read your thoughts on this and on my research in the comments. See you next time!</p>

