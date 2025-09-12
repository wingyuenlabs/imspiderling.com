---
Title: Middleware in .Net
Description: 
Author: Ahmet muhsinoğlu
Date: 2025-09-12T21:22:41.000Z
Robots: noindex,nofollow
Template: index
---
<p>Middleware allows us to shape the request structures and response structures from the client.</p>

<ol>
<li>Routing</li>
<li>Authentication</li>
<li>Authorization</li>
<li>Logging</li>
</ol>

<p>These items can be made middleware constructs.</p>

<p>For example;</p>

<h3>
  
  
  Middleware Imp.
</h3>

<p>We have an Api project.</p>

<p>Our aim is to check whether there are Client-Key, Client-Id, key-value parameters in the header section of the request models that will come to this api and to write middleware for the response that returns according to the presence of the parameters.</p>

<p>HeaderCheckMiddleware class is created.<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight plaintext"><code>public class HeaderCheckMiddleware
{
    private readonly RequestDelegate _next;
    public HeaderCheckMiddleware(RequestDelegate next)
    {
        _next = next;
    }

    public async Task Invoke(HttpContext httpContext)
    {
        var key1 = httpContext.Request.Headers.Keys.Contains("Client-Key");
        var key2 = httpContext.Request.Headers.Keys.Contains("Device-Id");

        if (!key1 || !key2)
        {
            httpContext.Response.StatusCode = 400;
            await httpContext.Response.WriteAsync("Missing requeired keys !");
            return;
        }
        else
        {
            //todo
        }
        await _next.Invoke(httpContext);
    }
}
</code></pre>

</div>



<p>The Invoke method allows us to intervene in the incoming request and response. In this scenario, if there are no parameters we want in the header of the incoming request model, we return 400 bad request.</p>

<p>In step 2, we write extension method for HeaderCheckMiddleware class. We are writing a static method because we want to add it to program.cs and we want the middleware to run when the system runs.<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight plaintext"><code>public static class MiddlewareExtension
{
    public static IApplicationBuilder UseHeaderCheckMiddleware
(this IApplicationBuilder builder)
    {
        return builder.UseMiddleware&lt;HeaderCheckMiddleware&gt;();
    }
}
</code></pre>

</div>



<p>In step 3, we use this static method in program.cs.</p>

<p>And let’s we see the responses.</p>

<h4>
  
  
  400 — Bad Request
</h4>

<p><a href="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fby9amtoon62r1tmhsowl.jpg" class="article-body-image-wrapper"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fby9amtoon62r1tmhsowl.jpg" alt=" " width="800" height="308"></a></p>

<h4>
  
  
  200 — OK
</h4>

<p><a href="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fzo83efcb7mesz1vrz7dx.jpg" class="article-body-image-wrapper"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fzo83efcb7mesz1vrz7dx.jpg" alt=" " width="800" height="367"></a></p>

<p>Thanks for reading…</p>

<p>Ahmet Muhsinoglu.</p>

