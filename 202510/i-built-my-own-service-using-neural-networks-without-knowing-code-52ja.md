---
Title: I Built My Own Service Using Neural Networks Without Knowing Code
Description: 
Author: Paul Molyanov
Date: 2025-10-17T20:57:59.000Z
Robots: noindex,nofollow
Template: index
---
<p>I've been following neural networks for a long time—back in 2016, I wrote an article about how these things would do all the work for copywriters and editors. Back then, AI was very dumb, and only geeks played with it, detecting cat faces in photos of random stuff.</p>

<p>I seriously dove into the technology around 2023 when more or less adequate ChatGPT and Midjourney models appeared. Since then, I started figuring out neural networks, created Neurozeh, and now use AI every day in work and daily life.</p>

<p><strong>I Wanted to Create a Service Using Only Neural Networks</strong></p>

<p>I already have a running business that's doing well without my constant involvement. I wanted to launch something for myself—a service that would generate additional income. I didn't want to involve developers or spend months on development, so I decided to make something simple. The main criteria were:</p>

<ul>
<li>There's already demand for the service—something similar is being searched for in search engines. I definitely didn't want to reinvent the wheel</li>
<li>The service can be quickly built and tested. And if it "doesn't take off," just move on to building a new one</li>
<li>In the future, the service can be promoted through SEO and potentially monetized</li>
</ul>

<p>A browser extension fit these parameters best. It's easy to code and doesn't require complex maintenance. And if you make it for Chrome, Google will give the extension priority in search.</p>

<p><strong>What I Did and How, Step by Step</strong></p>

<p><strong>Found an idea using ChatGPT.</strong> In Deep Research mode, I asked the neural network to find ideas for extensions that already exist and are in demand, meaning they have proven demand. I set these criteria:</p>

<ul>
<li>Already exist on the market: meaning the idea isn't "pulled from thin air" but confirmed by real users</li>
<li>At least 10,000 users—this was the demand threshold</li>
<li>Professional tool: oriented not toward mass entertainment but toward specialists. For example, SEO specialists, developers, product managers</li>
<li>Reviews present, to exclude manipulation and ensure the app is actually used</li>
<li>Ease of implementation—ChatGPT should assess complexity from a junior's perspective or a conditionally competent user</li>
<li>No server-side component—so there's no need to deploy backend and infrastructure</li>
</ul>

<p><strong>Prompt:</strong></p>

<p><em>I want to develop my own Chrome extension and am looking for ideas. Your task is to find 30 ideas for me.</em><br>
<em>As ideas, we'll take already existing extensions—meaning ones for which there's obvious demand. So you need to find 30 already existing extensions that have users.</em><br>
<em>Select by these criteria:</em><br>
<em>- More than 10,000 users in chromewebstore</em><br>
<em>- One simple function. Don't need complex overloaded services, need something simple</em><br>
<em>- Easy to implement. I'm not a great programmer and won't be able to make something complex. Need something that can be hacked together in 2 weeks</em><br>
<em>- Developer is not a corporation. No point competing with Adobe or Microsoft, zero chance of beating them</em><br>
<em>- Target audience of the extension is specialists. That is, it's a tool that helps people do their work, save time, etc. Entertainment, games, wallpapers, services "for everyone" like downloading videos from YouTube—don't fit</em><br>
<em>- There's SEO demand for the service's functionality. It does something that people search for on Google</em><br>
<em>Give answers in this format:</em><br>
<em>Extension name</em><br>
<em>Link to chromewebstore</em><br>
<em>Number of users</em><br>
<em>Brief description of what it does</em><br>
<em>Number of reviews</em><br>
<em>Average rating</em><br>
<em>Your comment on why you believe in this idea</em></p>

<p>Then I asked it to find fifty applications that fit my criteria and started manually reviewing them. I clicked around, looked at interfaces, documentation, target audience. Then checked queries through Semrush to make sure there was search traffic on the topic.</p>

<p>In the final, 3-4 ideas made it through, one of which was a broken link checking service. I postponed some of the rest for later, and eventually rejected one. It was an idea for integrating a neural network with Google Calendar—but that's already being done in Gemini.</p>

<p><strong>Built the first version of the service in ChatGPT.</strong> I described the extension idea to the neural network and gave basic requirements like: "make a service that finds all links on a page and checks them." The process began.</p>

<p>ChatGPT generated code that worked but looked bad and was difficult to use. Plus, coding through this neural network is very hard—the bot outputs responses as an archive that needs to be unpacked on the computer. It takes long and tedious, but importantly, it works. This reassured me, and I decided to take the process seriously.</p>

<p><strong>Manually drew the interface in Figma.</strong> I ran through future competitors and tested their services. I screenshotted and wrote down what I liked, but importantly—what needed to be fixed in my future application:</p>

<ul>
<li>Someone had an overloaded interface</li>
<li>Someone had too many buttons</li>
<li>Someone hid links too deep</li>
<li>Someone simply showed HTTP codes—and that's it</li>
</ul>

<p>In general, I fixed all the interface flaws of competitors and got my own design. Exported the file from Figma and took it to another service—Cursor.</p>

<p><strong>Started coding in Cursor with the Claude 4 Sonnet model.</strong> Although I studied to be a developer, my experience is only at university graduate level. In theory, I understand something, but I couldn't write a full-fledged service. So my communication with the neural network looked like this:</p>

<p><em>— do this and that, it should work like this</em><br>
<em>— [neural network outputs code]</em><br>
<em>— fix this and that so it works like this...</em></p>

<p>I simply described to the neural network how the application should work and reported if it wasn't working correctly. I fit the entire process into several sequential steps:</p>

<p>First, I asked to create a basic Chrome extension that simply opens a popup with my interface. That is, literally a window with what I drew in Figma. As soon as the neural network succeeded, I started asking it to roll out functionality: so buttons became buttons, lists became lists. Gradually I brought the HTML and CSS in order.</p>

<p><strong>Compiled detailed requirements for the neural network:</strong> I described in it exactly how the service should work. Explained what each button is responsible for and loaded it all into the neural network.</p>

<p><strong>Gradually in Cursor started working through each option</strong> for the application in the format "one step = one feature." My approximate order was:</p>

<ul>
<li>Collecting all links on the open page—automatic parsing</li>
<li>Sending HTTP requests to each link to check status: 200, 404, 502, etc.</li>
<li>Saving results and visual indication: green = ok, red = error</li>
<li>Filtering links by status so you can quickly find needed ones</li>
<li>Highlighting problematic links right on the page</li>
<li>Settings system—choosing display and processing mode</li>
<li>Link counter—total, working, broken, and other statuses</li>
</ul>

<p>But this is easy in text, while in reality it required many iterations. You write one sentence, it makes it, you click in the browser, doesn't work. Then you write: "dude, doesn't work, here's a screenshot"—it keeps working on it. So you run back and forth.</p>

<p>And Cursor can suddenly break everything and stubbornly not understand where to look for the error. So fairly quickly I realized I needed to make backups—then the program can be rolled back to the previous version when everything worked. During development, I rolled back dozens of times.</p>

<p>It may happen that some bugs can't be fixed at all. In this case, I recommend going to ChatGPT with approximately this request: "I'm making application X, I'm having problem Y. Think about what the problem might be and how it can be solved." With web search enabled, ChatGPT will start googling and looking for how other people solved similar problems.</p>

<p>When you get ChatGPT's answer, you can send it to Cursor with this prompt: "An independent expert suggested this solution to the problem [ChatGPT's advice]." High probability that after this everything will work.</p>

<p><strong>The service development took me about eight hours</strong>—from drawing in Figma to a fully working version of the extension. At the same time, I didn't write a single line of code myself—and the neural network would clearly have done better =)</p>

<p><strong>Created SEO description using Claude.</strong> I exported keywords for SEO using Semrush, wrote the service name, and created its short and long descriptions using the neural network. Honestly, I sat all evening making this stupid description. I think if I'd written it myself—it would have been faster. But it was important to me to do everything with neural networks.</p>

<p><strong>Adapted the extension for different languages.</strong> I initially planned to promote through SEO, without paid traffic. So to increase reach, I needed to properly adapt everything for foreign markets.</p>

<p>I had a ready SEO description in English—but needed to translate it to other popular languages, 52 in total. I did translations through ChatGPT. But not just like that, but so it would collect key phrases by which similar services are searched for in that language. The neural network downloaded descriptions of other extensions and added keywords to my descriptions.</p>

<p>Translations took one evening of intensive work. Probably, without neural networks, I would have spent more time just looking for contractors.</p>

<p><strong>The Application Is Already Published in the Store</strong></p>

<p>Now it can be installed in the Chrome extension market, it's called Broken Link Checker — <a href="https://chromewebstore.google.com/detail/broken-link-checker/cipdlgmjblnniiicohcmcafcncippbha" rel="noopener noreferrer">https://chromewebstore.google.com/detail/broken-link-checker/cipdlgmjblnniiicohcmcafcncippbha</a></p>

<p><strong>Development took about 30 hours.</strong> Most of them went to finding an idea and SEO, while the actual "programming" required about 8 hours. I think a working product in just a week is a good result.</p>

<p><strong>The extension already has first users.</strong> In total, 1,000 people have installed it, and over 600 of them launch it at least once a week. The audience is gradually growing, which means my plan is working—SEO traffic really brings people to the product =)</p>

<p><strong>Recently launched on Product Hunt, collected 24 upvotes. The extension has 10 reviews.</strong></p>

<p><strong>Investment in the application turned out small.</strong> ChatGPT and Claude cost me about $20 each—but I pay for them every month anyway. Claude Sonnet API, which I used through Cursor—another $90. Development, design, copywriting, and translations collectively would have cost over one hundred thousand rubles. Can't say I saved money—I spent my time too. But it was an interesting experiment.</p>

<p>Most importantly—with neural networks I was actually able to develop this application. Without them, I would have had to dig through documentation, watch all kinds of Python guides, and much more. In general, it would have taken much more time. With neural networks, I assembled a working project on my lap, in a few evenings of intensive work and without bringing in outside specialists.</p>

<p><strong>Preparing the Extension for Monetization</strong></p>

<p>Now the service is gradually being connected by new users. I plan to grow the audience to 10,000 people through SEO, without paid traffic. And after that, launch monetization.</p>

<p>Most likely, monetization will be through subscription or additional features that can be implemented on a paid plan. Similar products are already selling on the market, and they cost around $50. Even if I just compete on price and set it at $40—I'll still be in the plus.</p>

<p>I think this isn't my last application. I have several more ideas in my plans that I'd like to implement. If everything works out, I'll start stamping out such services once a quarter—maybe one of them will take off.</p>

