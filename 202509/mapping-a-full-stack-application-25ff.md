---
Title: Mapping a Full Stack Application
Description: 
Author: Kaden Wildauer
Date: 2025-09-09T21:04:15.000Z
Robots: noindex,nofollow
Template: index
---
<p>This article is going to map a full stack system: the what, where, and why. This isn't really system design but going over a pre designed system and aimed at beginner to lower-intermediate web developers. There are going to be three sections. First is serving code; this will go over what happens when you visit a full stack application. The second is logging in, this is a smaller section, but in most full stack applications, being logged in is fundamental. The third section is common operations, this section is going to go over what happens when someone logs in and is using the application. Where does JWT fit? Cookies? CORS? HTTPS? API calls? And other things that are important to understand when building a full stack application.</p>

<h2>
  
  
  Serving the website
</h2>

<p><a href="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fc3gdfj4lyuy3ig6bhdqw.png" class="article-body-image-wrapper"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fc3gdfj4lyuy3ig6bhdqw.png" alt="A graph showing the website serving process" width="800" height="436"></a></p>

<h3>
  
  
  I. Initiation
</h3>

<p>As shown in the image, the first step is simply typing the domain in. This is important to mention because we are going to be going over domain names and how they work.</p>

<h3>
  
  
  II. DNS Lookup
</h3>

<p>This is the second step and leads into how the internet works. And the internet is all about endpoints and IP addresses. You see, google.com or <a href="http://www.kadenwildauer.com" rel="noopener noreferrer">www.kadenwildauer.com</a> don't really exist, they are just human-readable names that point to an IP address. The true name is just a random string of numbers that point toward the location of the server/endpoint. So let’s go over how and what a DNS is. DNS is a bit complex. In truth, first, in some cases, they aren’t even used because you have already visited the site and your browser has cached(saved) the IP address. But in the case that you have not visited it, your browser's memory will look through a DNS server to find the endpoint of your domain name. Every device or tech that connects to the internet has an IP address, and the DNS server is a database that maps domain names to IP addresses.</p>

<h3>
  
  
  III. TCP/TLS Handshake
</h3>

<p>First, TCP stands for Transmission Control Protocol. Using a metaphor of what this does, it’s like a pipe that connects both the client and server otherwise, we are just spraying water and hoping it gets to the other side. More directly, it ensures all data packets are sent and received in the correct order, a connection exists before data is sent, data integrity is verified and protects devices against to much data aka data flow. And that’s what it does and why it's used. How it’s set up is a three-way handshake. You can research this more on your own, but basically, we request a connection, the server says yes, and then we say yes back.</p>

<p>TLS is a wrapper for TCP that adds encryption to the data being sent and received. And this is used everywhere a website with https:// uses TLS and is using high level and impactful encryption sometimes marketed as 'military grade'. Now, this is rather simple to know why it's used, but let’s go over how it initiates, which is arguably more important than understanding TCP specifics, because you might actually need to know specifics of how TLS works, while TCP is more automated and you don’t need to know the specifics. First, the client initiates a connection with a 'hello' message, which includes the TLS version, cipher suites, and a random number. The server responds with its own 'hello' message, which includes its chosen TLS version, cipher suites, random number, and SSL certificate. The client then verifies the server's certificate against a list of certificate authorities or, more commonly, the exact authority that issued the certificate. This is to ensure the server is who it says it is. Then the client sends a pre-master secret encrypted with the server's public key and can only be decrypted by the server's private key. And the client gets the server's public key from the SSL certificate. Then the server decrypts it with its private key, and both client and server generate a session key using the pre-master secret and the random numbers exchanged earlier. The key should be the same on both sides. They exchange a 'finished' message encrypted with the session key to confirm the handshake is complete. And now both client and server can communicate securely using the session key.</p>

<p>SSL certificates are an important thing I want to add to a bit more. First, SSL is the old name for TLS technically it’s a TLS certificate and you can just think of it as a base64 encoded file that contains things like the public key, the domain name, the certificate authority that issued it, and the expiration date. It’s not some special certificate, just some key-value pairs that the client uses to verify information and acquire information.</p>

<h3>
  
  
  IV. HTTP Request
</h3>

<p>The HTTP request is the next step and probably one of the most basic steps in the process. We are just sending a request to get the compiled frontend code for the website. This step is practically the same as any other HTTP request you would make to a server, and also, in most cases, automated by the browser.</p>

<h3>
  
  
  V. HTTP Handling
</h3>

<p>Now here, there is a wide variety of how a response is received and sent back. In many cases, you do nothing and some other service handles the response for you for example, Nginx, Apache, Caddy, or Serve. So most all the time, this is something automated but necessary to understand.</p>

<h3>
  
  
  VI. Response
</h3>

<p>This response is the final step in most cases. We send back the compiled frontend code, and the browser renders it. This is the end of getting a website on your screen.</p>

<h2>
  
  
  Logging In
</h2>

<p>This is meant to be a short section that's important for understanding how a full stack application works, and that’s logging in and how it ties into the last section, which is mapping common operations. We need to know where the data is stored in this stateless application.</p>

<p><a href="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fvutlk37ieq4fjlbd7s4l.png" class="article-body-image-wrapper"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fvutlk37ieq4fjlbd7s4l.png" alt="A image showing the login process" width="800" height="436"></a></p>

<h3>
  
  
  I. User Submits Login Form
</h3>

<p>This is rather simple and self-explanatory the user is sending login or signup information to the server.</p>

<h3>
  
  
  II. Server Validates Credentials and JWT Explanation
</h3>

<p>The server checks the credentials against a database, checking if it exists, if the information is correct, etc. And wraps the information with a JWT. We don't store this on the backend to keep things stateless I will link a resource at the end about stateless apps. But the JWT, while having nuances, is a kind of wrapper for information. It holds 3 important pieces of information. The first is the header, which is the algorithm used to sign the token. The second is the data or information we are wrapping. And the third and most important is the signature, which is a combination of the header and payload that is signed with the secret key. The idea is: Header algo + Payload + Secret key = Signature this is calculated when we sign (aka create) the JWT. And when the data gets back, we run the same calculation and compare it to the signature in the JWT. Someone tampering with the data does not have the secret key, so they can’t create a valid signature. And this is how we verify the JWT and its data.</p>

<h3>
  
  
  III. Storing JWT in Cookies and Cookies Explanation
</h3>

<p>First, the JWT is sent back with the important information and, in this case, stored/sent in a cookie. Cookies are very similar to local storage but with key differences. The first is that they are automatically sent with every request, which promotes security and statelessness. The second is size limitations it’s vastly smaller than local storage and has a limit of 4KB per cookie, and most browser's have a max cookie limit. The third, and most important, is flags that can be set on a cookie like HttpOnly, Secure, SameSite, Domain, and time. These all have meanings and reasons, but here I am going to talk about time. The rest is a little bit of research you can do on your own but each protect against a type of attack. Time is like a JWT a cookie has a storage expiration time. Depending on the time and type of application, this can be a few minutes to a few months.</p>

<h2>
  
  
  Mapping Common Operations
</h2>

<p>This section is all about mapping common operations in a fullstack application. The definitely really cool, and well designed looking diagram below will be explained in numerical order and will be a walkthrough over each step and important pieces of information...</p>

<p><a href="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fea933uej77kjvfpddf11.png" class="article-body-image-wrapper"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fea933uej77kjvfpddf11.png" alt="A image showing common full stack operations" width="800" height="436"></a></p>

<h3>
  
  
  I. HTTPS
</h3>

<p>Here, especially after we had that long explanation of TLS and SSL, what is the point? What does https:// do? Well, it does one really important thing and that is encrypt the data being sent and received. But only if the other endpoint is also using HTTPS. If not, it's a normal naked HTTP request. So, as shown on the diagram, the backend is hosted on, for example, <a href="https://kadenwildauer.com/api" rel="noopener noreferrer">https://kadenwildauer.com/api</a>, so it's still protected by TLS and SSL. While our communications to the SQL database are not encrypted. It's not really needed because it's a private network and not exposed to the internet. But that's HTTPS.</p>

<h3>
  
  
  II. Cookies
</h3>

<p>Now, I already went over this, but it's still something to note that since this is a stateless application, we need all the information to be sent with every request so we can query based on that data. Now, of the available cookies we can use a certain amount which is different for each browser we are using only one, which is wrapped in a JWT.</p>

<h3>
  
  
  III. API Calls
</h3>

<p>The API calls are the next step. Let's go over requesting data:<br>
First, we send a cookie and information related to the backend function we are calling. If we pass certain checks decided by the backend like for example authentication, CORS, and other things I will go over later, we get a response back with the included data. This is generally the same for all API calls and is the most basic operation in a full stack application. But it's important to note the amount of control the backend has over the frontend. It's ideal for that to be a lot of control because the frontend is just the client. And they can manipulate it in any way they want. So depending on the app, the backend can have a lot of control over the frontend.</p>

<h3>
  
  
  IV. Pages and Routing
</h3>

<p>This is a part that I debated on whether to include, but it feels integral enough to an application given that most full stack applications are not single page and have routes and pages. Now, a route is just a path that can be easily created by a programmer and is a way to organize the application. When you click on a button, for example, we go to another route and unload the previous page and load a new one a page being code like html, CSS, etc. It's easy for us to implement and use. Some websites don't have this, and that's fine, but most do.</p>

<p>Now, what about backend routing? Well, it's kind of similar, but not really. It's a way to organize requests and responses. For example, to log in we might go to <a href="https://kadenwildauer.com/api/login" rel="noopener noreferrer">https://kadenwildauer.com/api/login</a>, and this route expects certain information to be sent with the request and runs certain functions based on the route. Commonly, this is something called a controller directory that holds controllers functions that handle the request and response. And this is how we organize our backend code.</p>

<h3>
  
  
  V. Database
</h3>

<p>Now this is an integral part of a website. Most everything on a website specific to you, the user, is stored in a database. We store things like login information, user data, posts, comments, and other things. This is a very important part of a full stack application. There are many types of databases or methods of storing data. But most commonly, we use a SQL database like MySQL, PostgreSQL, or SQLite. We also most commonly use an ORM, or Object Relational Mapper, to interact with the database. This is a way to interact with the database using code instead of writing raw SQL queries. It’s more flexible, fast, and easier to use than writing raw SQL queries. Although it can be slower and less efficient in that case, you still need to write raw SQL queries. I won’t go into too much detail, but as for security, it consists mostly of a few things. Such as sanitizing inputs, not exposing the database to the internet, using .env files to connect and store the database connection information, limiting types of connections allowed, permission monitoring, and using a TLS connection to the database.</p>

<h3>
  
  
  VI. Backend Protections
</h3>

<p>We have covered most everything integral to a basic full stack application the more external stuff and here we are going over backend protections. This is very important and often overlooked by beginner developers I know I did. But it’s important because if you're doing things correctly, then the backend is the gatekeeper for portions of frontend code and data. So we need to protect it. The first is CORS, or Cross Origin Resource Sharing. Think of it like a wall that protects from request types that you don’t allow for example, from a different domain or origin, a non HTTP request, or a request without credentials. The second and most important is middleware, often called auth or something similar. This is a function that checks a request's credentials. Do they have a valid JWT? Is it expired? Is it a valid user? And other requests that a programmer might want to check. If I visit a site and go to a protected route, something like an auth middleware will check and, in this case, not allow me to access any HTTP request or even load the component depending on the developer. Although, as we went over, frontend code does not really do much if someone is determined to get the data. There are other protections like rate limiting, input sanitization, and other things that are important to know about but not really needed to understand a full stack application.</p>

<h2>
  
  
  Puzzle Pieces
</h2>

<p>This is a basic high level overview of a full stack application and how it works. I did not go over the why of everything because that goes into system design, and that is not what this article is about.<br>
This is about if someone is building a full stack application what are the pieces they need to know about and how they fit together.<br>
Any of these things that I mentioned can be moved around, manipulated, changed, and I have no doubt that some research into any of these topics will lead you to ways of improving your full stack application.<br>
But no matter what, almost all full stack applications will have these pieces, and it's important to understand how they fit together.<br>
It gives more freedom of thought and creativity for a developer. And if you know this, abstractions? Are easy. SaaS products? You’ve already made a full stack application manually or understand what’s actually happening. <br>
So an abstraction? That’s light work. If you understand this stuff, maybe implement it yourself. Then all full stack applications are just these pieces implemented in different ways.</p>

<h2>
  
  
  Conclusion
</h2>

<p>I hope you gained something from this article and learned something new. Apart from my bad diagram skills, I hope you learned something about how a full stack application works and how the pieces fit together. <br>
Regardless, have a good day. I am going to include some resources below for steps that I had trouble with or that I think are important to understand and as for some proof two of my<br>
full stack apps used a modified version of what I just showed you with an ec2 you can see them in the skills section.</p>

<ul>
<li><a href="https://blog.purestorage.com/purely-educational/stateful-vs-stateless-applications-whats-the-difference/" rel="noopener noreferrer">Serverless</a></li>
<li><a href="https://www.cloudflare.com/learning/dns/what-is-dns/" rel="noopener noreferrer">DNS</a></li>
<li><a href="https://www.cloudflare.com/learning/ssl/transport-layer-security-tls/" rel="noopener noreferrer">TLS/Https</a></li>
<li><a href="https://jwt.io/introduction/" rel="noopener noreferrer">JWT</a></li>
</ul>

