---
Title: Deploy .NET 8 App to Azure Kubernetes Service (AKS)
Description: 
Author: SUBAIR NURUDEEN ADEWALE
Date: 2026-02-10T22:09:02.000Z
Robots: noindex,nofollow
Template: index
---
<p><strong>Introduction</strong></p>

<p>Modern software delivery increasingly relies on cloud-native technologies such as containers, Kubernetes, and automated deployment pipelines. This project demonstrates the end-to-end process of building, containerizing, and deploying a .NET 8 Web API to Azure Kubernetes Service (AKS) using industry best practices.</p>

<p>By developing a Weather Forecast API, the project simulates a real-world DevOps workflow where applications are tested locally, packaged with Docker, stored in Azure Container Registry, deployed to Kubernetes, and automatically updated through CI/CD. The goal is to gain practical experience in deploying scalable and production-ready applications in a cloud environment.</p>

<p><strong>Architecture Flow</strong></p>

<p><a href="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fki6xxvzh5l8enu5mmuiv.png" class="article-body-image-wrapper"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fki6xxvzh5l8enu5mmuiv.png" alt=" " width="690" height="1306"></a></p>

<p><strong>Prerequisites</strong></p>

<p>Required Software<br>
Install the following tools in this order:</p>

<ul>
<li>Visual Studio Code</li>
<li>.NET 8 SDK</li>
<li>Git</li>
<li>Docker Desktop,Start Docker Desktop after installation</li>
<li>Azure CLI</li>
<li>kubectl</li>
<li>GitHub CLI (optional but recommended)</li>
<li>Required Accounts</li>
<li>GitHub account (free)</li>
<li>Azure account (free tier available)</li>
</ul>

<p><strong>Verify Installations</strong><br>
Run the following commands:</p>

<ul>
<li>dotnet --version</li>
<li>git --version</li>
<li>docker --version</li>
<li>az --version</li>
<li>kubectl version --client</li>
</ul>

<p>If any command fails, fix it before proceeding.</p>

<p><strong>Step 1: Create and Test the .NET Application Locally</strong><br>
Why This Step Matters<br>
You never deploy untested code.</p>

<p>Testing locally allows you to:</p>

<ul>
<li>Catch errors early</li>
<li>Validate endpoints</li>
<li>Avoid expensive cloud debugging</li>
</ul>

<p><strong>1.1 Create Project Structure</strong></p>

<p>Use the command to create directory and to enter into the directory</p>

<p><code>mkdir weather-app-demo</code><br>
<code>cd weather-app-demo</code><br>
<code>mkdir WeatherApp</code><br>
<code>cd WeatherApp</code></p>

<p><a href="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Ffk13ly8zve81duykl26k.png" class="article-body-image-wrapper"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Ffk13ly8zve81duykl26k.png" alt=" " width="800" height="425"></a></p>

<p><strong>1.2 Initialize the .NET Web API</strong></p>

<p><code>dotnet new webapi -minimal</code></p>

<p>This creates a minimal API with fewer files and faster startup.</p>

<p><a href="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2F9cupretli7smdjkagx82.png" class="article-body-image-wrapper"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2F9cupretli7smdjkagx82.png" alt=" " width="800" height="425"></a></p>

<p><strong>1.3 Add Required Packages</strong></p>

<p>This command adds the Health Checks library to your .NET project.</p>

<p>In simple terms:<br>
It installs a package that allows your app to report whether it is healthy or not.</p>

<p><code>dotnet add package Microsoft.Extensions.Diagnostics.HealthChecks</code></p>

<p><a href="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2F1s4pvmx9gij7t52zji3n.png" class="article-body-image-wrapper"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2F1s4pvmx9gij7t52zji3n.png" alt=" " width="800" height="425"></a></p>

<p><code>dotnet add package Swashbuckle.AspNetCore</code><br>
This installs Swagger support into your .NET Web API project.</p>

<p>Swashbuckle is the library that automatically generates:</p>

<ul>
<li>API documentation</li>
<li>Interactive testing UI
In Simple Terms:
It gives you a web page where you can see and test all your API endpoints.</li>
</ul>

<p><a href="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fvd80ihd6nyctvg0543mk.png" class="article-body-image-wrapper"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fvd80ihd6nyctvg0543mk.png" alt=" " width="800" height="425"></a></p>

<p><code>Code .</code>  </p>

<p>It opens the current folder in Visual Studio Code.</p>

<p><a href="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fxl7p5ma2y79f12z8jdzu.png" class="article-body-image-wrapper"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fxl7p5ma2y79f12z8jdzu.png" alt=" " width="800" height="425"></a></p>

<p><strong>1.4 Replace Program.cs</strong><br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight plaintext"><code>var builder = WebApplication.CreateBuilder(args);

// Services
builder.Services.AddHealthChecks();
builder.Services.AddEndpointsApiExplorer();
builder.Services.AddSwaggerGen();

// Kestrel config for containers
builder.WebHost.ConfigureKestrel(options =&gt;
{
    options.ListenAnyIP(8080);
});

var app = builder.Build();

// Middleware
app.UseSwagger();
app.UseSwaggerUI();

// Endpoints
app.MapGet("/", () =&gt; new
{
    Message = "Welcome to the Weather App!",
    Version = "1.0.0",
    Timestamp = DateTime.UtcNow
});

app.MapGet("/weather", () =&gt;
{
    var summaries = new[]
    {
        "Freezing", "Bracing", "Chilly", "Cool",
        "Mild", "Warm", "Hot", "Scorching"
    };

    return Enumerable.Range(1, 5).Select(i =&gt; new
    {
        Date = DateOnly.FromDateTime(DateTime.Now.AddDays(i)),
        TemperatureC = Random.Shared.Next(-20, 40),
        Summary = summaries[Random.Shared.Next(summaries.Length)]
    });
});

// Health endpoint for Kubernetes
app.MapHealthChecks("/health");

app.Run();

</code></pre>

</div>



<p><a href="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fc3g39gawxewm6a14ia1t.png" class="article-body-image-wrapper"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fc3g39gawxewm6a14ia1t.png" alt=" " width="800" height="425"></a></p>

<p><strong>1.5 Test Locally</strong></p>

<p><code>dotnet run</code></p>

<p>Test endpoints:</p>

<ul>
<li><strong><a href="http://localhost:8080/" rel="noopener noreferrer">http://localhost:8080/</a></strong></li>
</ul>

<p><a href="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2F87tjs3xu6wsajgpsjasb.png" class="article-body-image-wrapper"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2F87tjs3xu6wsajgpsjasb.png" alt=" " width="800" height="425"></a></p>

<ul>
<li><strong><a href="http://localhost:8080/weather" rel="noopener noreferrer">http://localhost:8080/weather</a></strong></li>
</ul>

<p><a href="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fkeh8252jaqyur6c56vlb.png" class="article-body-image-wrapper"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fkeh8252jaqyur6c56vlb.png" alt=" " width="800" height="425"></a></p>

<ul>
<li><strong><a href="http://localhost:8080/swagger" rel="noopener noreferrer">http://localhost:8080/swagger</a></strong></li>
</ul>

<p><a href="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fvwpoqfp2tic2509yb2io.png" class="article-body-image-wrapper"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fvwpoqfp2tic2509yb2io.png" alt=" " width="800" height="425"></a></p>

<ul>
<li><strong><a href="http://localhost:8080/health" rel="noopener noreferrer">http://localhost:8080/health</a></strong></li>
</ul>

<p><a href="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fefhsoup1q6zaqgjifcuo.png" class="article-body-image-wrapper"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fefhsoup1q6zaqgjifcuo.png" alt=" " width="800" height="425"></a></p>

<p>If this works, your app is ready for containerization.</p>

<p><strong>Step 2: Containerize the Application with Docker</strong></p>

<p>Why Containerize?<br>
Containers ensure:</p>

<ul>
<li>Same runtime everywhere</li>
<li>No “works on my machine” issues</li>
<li>Smooth Kubernetes deployment</li>
</ul>

<p><strong>2.1 Create Dockerfile</strong></p>

<p>Use the command <code>touch or New-item Dockerfile</code></p>

<p><a href="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fpw3vq4axqh32wlv67gvk.png" class="article-body-image-wrapper"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fpw3vq4axqh32wlv67gvk.png" alt=" " width="800" height="425"></a></p>

<p>Copy the docker instruction into the dockerfile<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight plaintext"><code># Runtime
FROM mcr.microsoft.com/dotnet/aspnet:8.0 AS base
WORKDIR /app
EXPOSE 8080

# Build
FROM mcr.microsoft.com/dotnet/sdk:8.0 AS build
WORKDIR /src
COPY weatherApp.csproj .
RUN dotnet restore
COPY . .
RUN dotnet publish -c Release -o /app/publish

# Final image
FROM base
WORKDIR /app
COPY --from=build /app/publish .
ENTRYPOINT ["dotnet", "weatherApp.dll"]
</code></pre>

</div>



<p><a href="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fy3gc2c23c1mkca1vmfg5.png" class="article-body-image-wrapper"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fy3gc2c23c1mkca1vmfg5.png" alt=" " width="800" height="425"></a></p>

<p><strong>2.2 Create .dockerignore</strong><br>
Use the command <code>touch or New-item .dockerignore</code></p>

<p><a href="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fxxv6fsoxtyacvi2xd94x.png" class="article-body-image-wrapper"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fxxv6fsoxtyacvi2xd94x.png" alt=" " width="800" height="425"></a></p>

<p>Copy the code into the .dockerignore file.<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight plaintext"><code>bin/
obj/
.vs/
.vscode/
.git/
.gitignore
Dockerfile
README.md
*.log
</code></pre>

</div>



<p><a href="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fdx7bwv0zvdnx4lgxg6tj.png" class="article-body-image-wrapper"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fdx7bwv0zvdnx4lgxg6tj.png" alt=" " width="800" height="425"></a></p>

<p><strong>2.3 Build and Test Container Locally</strong></p>

<p><code>docker build -t weather-app:local .</code></p>

<p><a href="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fcpnvzybqm0itqph09pkc.png" class="article-body-image-wrapper"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fcpnvzybqm0itqph09pkc.png" alt=" " width="800" height="425"></a></p>

<p><code>docker run -p 8080:8080 weather-app:local</code></p>

<p><a href="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Ffjkb3ef9qtbjglh46sdq.png" class="article-body-image-wrapper"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Ffjkb3ef9qtbjglh46sdq.png" alt=" " width="800" height="425"></a></p>

<p>Test again in browser or with curl.<br>
If it works locally, it will work in AKS.</p>

<p><a href="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2F2gzxbsjwc1asssfg0pnm.png" class="article-body-image-wrapper"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2F2gzxbsjwc1asssfg0pnm.png" alt=" " width="800" height="425"></a></p>

<p><strong>Step 3: Create Azure Infrastructure</strong></p>

<p><strong>3.1 Login to Azure</strong><br>
use the command <code>az login</code> or <code>az login --allow-no-subscriptions</code> <br>
once prompted to select a subscription or tenant ID, Click on the ENTER key on your keyboard.</p>

<p><a href="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2F6srkoqrzam65wwjo6je6.png" class="article-body-image-wrapper"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2F6srkoqrzam65wwjo6je6.png" alt=" " width="800" height="425"></a></p>

<p><strong>3.2 Create Resource Group</strong><br>
<code>az group create --name student-demo --location eastus</code></p>

<p><a href="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fb04hk6fnpf6713f7206l.png" class="article-body-image-wrapper"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fb04hk6fnpf6713f7206l.png" alt=" " width="800" height="425"></a></p>

<p><strong>3.3 Create Azure Container Registry (ACR)</strong></p>

<p><strong>3.3.1 Register your subscription to use Microsoft.ContainerRegistry</strong></p>

<p>Use the command to register the service <code>az provider register --namespace Microsoft.ContainerRegistry</code><br>
and use <code>az provider show --namespace Microsoft.ContainerRegistry --query registrationState</code> to check the registration status.</p>

<p><a href="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2F72ygujo3wc1eaqwrsuzd.png" class="article-body-image-wrapper"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2F72ygujo3wc1eaqwrsuzd.png" alt=" " width="800" height="425"></a></p>

<p><strong>3.3.2  Create Azure Container Registry (ACR)</strong><br>
Use the command to create Azure Container Registry and give it a Unique name.<br>
<code>az acr create --resource-group student-demo --name studentdemoacr --sku Basic</code></p>

<p><a href="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fwcnmwignecvyluwsc68q.png" class="article-body-image-wrapper"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fwcnmwignecvyluwsc68q.png" alt=" " width="800" height="425"></a></p>

<p><strong>3.4 Build &amp; Push Image Using ACR</strong></p>

<p><code>az acr build --registry studentdemoacr --image weather-app:latest .</code> or Instead of using <strong>az acr build</strong>, you can:</p>

<p><strong>Build locally with Docker and push manually</strong></p>

<p><code>docker build -t skillstudentdemoacr.azurecr.io/weather-app:latest .</code></p>

<p><a href="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2F3l8uxnfkltcnm10yr0x4.png" class="article-body-image-wrapper"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2F3l8uxnfkltcnm10yr0x4.png" alt=" " width="800" height="425"></a></p>

<p>*<em>Login into Azure Container registry *</em></p>

<p><code>az acr login --name skillstudentdemoacr</code></p>

<p><a href="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fel08ihoifgloi0ffyqcm.png" class="article-body-image-wrapper"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fel08ihoifgloi0ffyqcm.png" alt=" " width="800" height="425"></a></p>

<p><strong>Push image to Azure Container registry</strong></p>

<p><code>docker push skillstudentdemoacr.azurecr.io/weather-app:latest</code></p>

<p><a href="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Feqb4tdoc4f5emrfxpdd7.png" class="article-body-image-wrapper"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Feqb4tdoc4f5emrfxpdd7.png" alt=" " width="800" height="425"></a></p>

<p><strong>3.5 Create AKS Cluster with ACR Attached</strong></p>

<p>Register your subscription to use <strong>Microsoft.ContainerService</strong> first before creating the AKS cluster.</p>

<p><code>az provider register --namespace Microsoft.ContainerService</code><br>
use this to check the status <code>az provider show --namespace Microsoft.ContainerService --query registrationState</code></p>

<p><a href="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fwr719stswbzhlvkod554.png" class="article-body-image-wrapper"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fwr719stswbzhlvkod554.png" alt=" " width="800" height="425"></a></p>

<p><strong>Create aks cluster</strong></p>

<p><code>az aks create --resource-group student-demo --name skillstudent-aks --node-count 1 --attach-acr skillstudentdemoacr --enable-managed-identity --generate-ssh-keys --location eastus</code></p>

<p><a href="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2F0re6uimxe10kh2umn8xl.png" class="article-body-image-wrapper"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2F0re6uimxe10kh2umn8xl.png" alt=" " width="800" height="425"></a></p>

<p><strong>3.6 Connect kubectl to AKS</strong></p>

<p><code>az aks get-credentials --resource-group student-demo --name student-aks</code></p>

<p>Verify <code>kubectl get nodes</code></p>

<p><a href="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fn7qs0c931wf0tgf9g5or.png" class="article-body-image-wrapper"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fn7qs0c931wf0tgf9g5or.png" alt=" " width="800" height="425"></a></p>

<p><strong>Step 4: Deploy to Kubernetes</strong></p>

<p><strong>4.1 Create Kubernetes Manifests</strong></p>

<p>Create a directory  <code>mkdir k8s</code><br>
Enter the direwctory <code>cd k8s</code><br>
Create deployment.yaml file using <code>New-item or touch deployment.yaml</code></p>

<p><a href="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fisgl30qyfy2goqpx2glz.png" class="article-body-image-wrapper"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fisgl30qyfy2goqpx2glz.png" alt=" " width="800" height="425"></a></p>

<p>Copy this code into the deployment.yaml file<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight plaintext"><code>apiVersion: apps/v1
kind: Deployment
metadata:
  name: weather-app
spec:
  replicas: 2
  selector:
    matchLabels:
      app: weather-app
  template:
    metadata:
      labels:
        app: weather-app
    spec:
      containers:
      - name: weather-app
        image: studentdemoacr.azurecr.io/weather-app:latest
        ports:
        - containerPort: 8080
        livenessProbe:
          httpGet:
            path: /health
            port: 8080
        readinessProbe:
          httpGet:
            path: /health
            port: 8080
</code></pre>

</div>



<p><a href="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fg9lk7d8xxc2be2ty2b7w.png" class="article-body-image-wrapper"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fg9lk7d8xxc2be2ty2b7w.png" alt=" " width="800" height="425"></a></p>

<p>Create service.yaml file using <code>New-item or touch service.yaml</code></p>

<p><a href="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fanfigma1pyal98ym226u.png" class="article-body-image-wrapper"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fanfigma1pyal98ym226u.png" alt=" " width="800" height="425"></a></p>

<p>Copy this code into the service.yaml<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight plaintext"><code>apiVersion: v1
kind: Service
metadata:
  name: weather-service
spec:
  type: LoadBalancer
  selector:
    app: weather-app
  ports:
  - port: 80
    targetPort: 8080

</code></pre>

</div>



<p><a href="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2F3iuhg0783ttqo1efxfr9.png" class="article-body-image-wrapper"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2F3iuhg0783ttqo1efxfr9.png" alt=" " width="800" height="425"></a></p>

<p><strong>4.2 Apply Manifests</strong></p>

<p><code>kubectl apply -f deployment.yaml</code><br>
<code>kubectl apply -f service.yaml</code></p>

<p>Check status:<br>
<code>kubectl get pods</code><br>
<code>kubectl get services</code></p>

<p><a href="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fnwg33diegb35xav2dych.png" class="article-body-image-wrapper"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fnwg33diegb35xav2dych.png" alt=" " width="800" height="425"></a></p>

<p><strong>Step 5: CI/CD with GitHub Actions</strong></p>

<p><strong>5.1 Initialize Git</strong></p>

<p><code>git init</code><br>
<code>git add .</code><br>
<code>git commit -m "Initial Weather App"</code></p>

<p><a href="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Ft7b1tni1kp65qfhl22py.png" class="article-body-image-wrapper"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Ft7b1tni1kp65qfhl22py.png" alt=" " width="800" height="425"></a></p>

<p><strong>5.2 Create GitHub Repo</strong><br>
*<em>Login to Github *</em><br>
Use rename your branch to main using <code>git branch -m main</code><br>
<code>Use git remote add origin https://github.com/Subair-09/WeatherAPP.git</code> to add repository.<br>
<code>Use git push origin main</code> to push the codes to github</p>

<p><a href="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2F3454t2uclc494ri1rkgv.png" class="article-body-image-wrapper"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2F3454t2uclc494ri1rkgv.png" alt=" " width="800" height="425"></a></p>

<p><strong>5.3 Create Workflow File</strong></p>

<p><a href="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fu64y74w3w0le2nl7zrnt.png" class="article-body-image-wrapper"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fu64y74w3w0le2nl7zrnt.png" alt=" " width="800" height="425"></a></p>

<p>Copy and paste this code in the .github/workflows/deploy.yml file <br>
name: CI/CD to AKS<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight plaintext"><code>on:
  push:
    branches: [main]

jobs:
  deploy:
    runs-on: ubuntu-latest
    steps:
    - uses: actions/checkout@v4

    - uses: azure/login@v1
      with:
        creds: ${{ secrets.AZURE_CREDENTIALS }}

    - run: |
        az acr build \
          --registry ${{ secrets.ACR_NAME }} \
          --image weather-app:${{ github.sha }} .

    - run: |
        az aks get-credentials \
          --resource-group ${{ secrets.RESOURCE_GROUP }} \
          --name ${{ secrets.CLUSTER_NAME }}

    - run: |
        kubectl set image deployment/weather-app \
        weather-app=${{ secrets.ACR_NAME }}.azurecr.io/weather-app:${{ github.sha }}
</code></pre>

</div>



<p><a href="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fzbj2j9u2v82nzplgp82r.png" class="article-body-image-wrapper"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fzbj2j9u2v82nzplgp82r.png" alt=" " width="800" height="425"></a></p>

<p><strong>Step 6: Access the Application</strong></p>

<p><code>kubectl get service weather-service</code><br>
<a href="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fq3hnc8lgt7y9tlb0umns.png" class="article-body-image-wrapper"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fq3hnc8lgt7y9tlb0umns.png" alt=" " width="800" height="425"></a></p>

<p>Use the EXTERNAL-IP in your browser.</p>

<p><a href="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2F3ti59lcuf3t7o50f6ecu.png" class="article-body-image-wrapper"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2F3ti59lcuf3t7o50f6ecu.png" alt=" " width="800" height="425"></a></p>

<p><strong>Conclusion</strong></p>

<p>This project successfully demonstrates a complete, real-world cloud-native deployment workflow using modern DevOps practices. Starting from local development of a .NET 8 Web API, the application was containerized with Docker, securely stored in Azure Container Registry, and deployed to Azure Kubernetes Service with proper health checks, scalability, and service exposure.</p>

<p>By integrating GitHub Actions for CI/CD, the deployment process was fully automated, ensuring that every change pushed to the repository results in a reliable and repeatable update to the Kubernetes cluster. This mirrors how production systems are built, tested, and delivered in professional cloud environments.</p>

<p>Overall, the project reinforces key concepts such as containerization, Kubernetes orchestration, infrastructure provisioning, and continuous delivery on Azure. It provides a strong foundation for building scalable, resilient, and production-ready applications, and serves as a practical reference for engineers transitioning into cloud and DevOps roles.</p>

