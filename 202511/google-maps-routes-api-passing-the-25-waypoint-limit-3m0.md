---
Title: Google Maps Routes API: Passing the 25 Waypoint Limit
Description: 
Author: Danny Hodge
Date: 2025-11-11T21:11:46.000Z
Robots: noindex,nofollow
Template: index
---
<p>If you've worked on the new Google Maps Routes API, and are working on a complex solution, you've probably been hampered by the 25 Waypoint limit. I've recently been working on a solution where the number of Waypoints could be in the hundreds! But not to worry, there's a solution that's simple and scalable. So pull up a chair, and let's take a look.</p>

<h2>
  
  
  Getting Started
</h2>

<p>To help me along, I've used the 'vis.gl/react-google-maps' library, which provides React components that wrap around the Google Maps JavaScript API features. There's a couple of options for this (none fully featured unfortunately), but this one is Google sponsored (<a href="https://mapsplatform.google.com/resources/blog/introducing-react-components-for-the-maps-javascript-api/" rel="noopener noreferrer">See their blob post here</a>), so probably the best option. In my example, I'll be using version 1.7.1. </p>

<h2>
  
  
  How it Works
</h2>

<p>The solution to this is actually quite simple. If you have a route with 50  Waypoints, you have a couple of options. </p>

<ol>
<li>Send the request with 50 Waypoints, and have it be rejected.</li>
<li>Split the request into two 25 Waypoint requests.</li>
<li>Split the request into several requests, for example with up to 10 Waypoints.</li>
</ol>

<p>The reason I chose option 3 is that it seems to be the cheapest, due to the pricing tier system Google employes. A request with 11 to 25 Intermediate Waypoints will jump up a cost tier (<a href="https://developers.google.com/maps/documentation/routes/intermed_waypoints" rel="noopener noreferrer">shown here in the Intermediate Waypoints documentation</a>), making it slightly more expensive. Feel free to play around with the numbers yourself though, as this could change: </p>

<p><a href="https://mapsplatform.google.com/pricing/#pricing-calculator" rel="noopener noreferrer">https://mapsplatform.google.com/pricing/#pricing-calculator</a></p>

<p>Once you've got the data back, you can simply render a Polyline for each of your requests, and it will be presented as one continuous line. </p>

<p>This solution will technically scale indefinitely, or at least until user's run into performance issues.</p>

<h2>
  
  
  The Code
</h2>

<p>I'll link the full CodeSandbox below for this solution <br>
<a href="https://codesandbox.io/p/sandbox/dhdjkg?file=%2Fsrc%2FApp.tsx" rel="noopener noreferrer">here</a>, but if you're not as experienced with the Google Maps JavaScript API, I suggest following along with the walkthrough to get a better understanding. You'll also need to get your own API Key to be able to run this demo.</p>

<h3>
  
  
  The Components
</h3>

<p>The code for the Polyline component was taken from <a href="https://github.com/visgl/react-google-maps/blob/main/examples/geometry/src/components/polyline.tsx" rel="noopener noreferrer">the vis.gl documentation</a>. For some reason it isn't exposed as a component, but we can pull the component from their documentation regardless.</p>

<p>To give a short summary, this component takes in the customizable Polyline properties and events as optional parameters, uses them to instantiate the Polyline and renders it on the map. It's also responsible for bundling all this up into a React component for you to use in your application.</p>

<h3>
  
  
  The Solution
</h3>

<p>This is where it gets a little more complicated. Let's break it down.</p>

<p>We'll start with the API request. We begin by iterating through our points, as shown in the code block below:</p>

<p><a href="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2F40z3vkeie1mj3estyadb.png" class="article-body-image-wrapper"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2F40z3vkeie1mj3estyadb.png" alt="Code block showing loop to set up API Request" width="629" height="340"></a></p>

<p>FetchRoute takes the start and end index of the current segment, passed in by a piece of code we'll see later. It gets the list of intermediate values (indices that will become Intermediate Waypoints) by taking a splice of a subset of the array (ignoring first and last point as these will be the Origin and Destination points), and then mapping through each of these values to format our data in the way the Google Routes API is expecting it, using the 'LatLng' type.</p>

<p>The next segment of our code is merely structuring the data for the API request. Take care to include the 'X-Goog-Api-Key' and 'X-Goog-FieldMask' headers, and arrange your data how you see fit. <br>
Importantly, nothing I have included here will take this request out of the cheapest tier of pricing (as it stands in late 2025). However, Google's tier system means many of the optional parameters available to this Routes API Endpoint will significantly increase your cost, so tinker with caution!</p>

<p><a href="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fx2s0acj71ma6y5htpdai.png" class="article-body-image-wrapper"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fx2s0acj71ma6y5htpdai.png" alt=" " width="654" height="859"></a></p>

<p>At the end of the above screenshot, you'll see that we concatenate the route segments (encoded Polylines) together to form our array of routes, which we'll be iterating through later on when we come to render the full Route.</p>

<p>And we can see right after how we call the above function, iterating through our list of points, and passing the start/end points of each segment in as parameters. You'll see a ternary operator towards the end, which is there to ensure that the end index isn't larger than the size of the array.</p>

<p><a href="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Ff93stdf9xhpn9uzgy58f.png" class="article-body-image-wrapper"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Ff93stdf9xhpn9uzgy58f.png" alt="Code block showing API Request" width="599" height="188"></a></p>

<p>Finally, we reach the end of the component, where we render our map and route (plus the markers to help visualise what's going on). Despite it being a large chunk of the code, there's really not much to explain here. The APIProvider and Map are basically passed in as they're found from any of the vis.gl docs, with a bit of styling added to keep it centred. After that, it's just a matter of looping through the array of Routes we set in state earlier, and rendering a Polyline for each of them. </p>

<p><a href="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2F22qs31km9hz981pjj005.png" class="article-body-image-wrapper"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2F22qs31km9hz981pjj005.png" alt="Code block showing rendering of Map and Polylines" width="665" height="682"></a></p>

<p>I hope this has been helpful, please let me know if there are any more Google Maps guides/demos you'd like me to create.</p>

