---
Title: Simple steps to Multi-Tenant Architecture Design
Description: 
Author: Naimul Karim
Date: 2025-09-09T22:02:08.000Z
Robots: noindex,nofollow
Template: index
---
<p><strong>Flow :</strong></p>

<ul>
<li>Request comes with InstituteId in header </li>
<li>Middleware validates and sets TenantInfo</li>
<li>TenantServiceResolver selects the service as per InstituteId in TenantInfo</li>
<li>The resolved service uses TenantDbContextFactory to select DB context</li>
<li>The rest works as per-tenant</li>
</ul>

<p><strong>Components:</strong></p>

<p>Component           Purpose<br>
InstituteIdMiddlewar :Contains InstituteId from header<br>
TenantInfo (Scoped) : Has the current request’s InstituteId<br>
TenantServiceResolver :Selects services based on tenant<br>
TenantDbContext : Per-tenant EF Core DbContext<br>
appsettings.json : Contains list of valid tenants</p>

<p><strong>1. TenantInfo Class (Contains Current Institute Id)</strong><br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight plaintext"><code>public class TenantInfo
{
    public string? InstituteId { get; set; }
}
</code></pre>

</div>



<p>Register it as Scoped in DI:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight plaintext"><code>builder.Services.AddScoped&lt;TenantInfo&gt;();
</code></pre>

</div>



<p><strong>2. Middleware: Extract InstituteId &amp; Validate</strong><br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight plaintext"><code>public class InstituteIdMiddleware
{
    private readonly RequestDelegate _next;

    public InstituteIdMiddleware(RequestDelegate next)
    {
        _next = next;
    }

    public async Task InvokeAsync(HttpContext context, TenantInfo tenantInfo, IConfiguration config)
    {
        if (!context.Request.Headers.TryGetValue("InstituteId", out var instituteId))
        {
            context.Response.StatusCode = 400;
            await context.Response.WriteAsync("Missing InstituteId header.");
            return;
        }

        var validTenants = config.GetSection("Tenants").Get&lt;List&lt;string&gt;&gt;();
        if (!validTenants.Contains(instituteId))
        {
            context.Response.StatusCode = 403;
            await context.Response.WriteAsync($"Unauthorized InstituteId: {instituteId}");
            return;
        }

        tenantInfo.InstituteId = instituteId;
        await _next(context);
    }
}

</code></pre>

</div>



<p>Register it:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight plaintext"><code>app.UseMiddleware&lt;InstituteIdMiddleware&gt;();
</code></pre>

</div>



<ol>
<li>Configuration: appsettings.json
</li>
</ol>

<div class="highlight js-code-highlight">
<pre class="highlight plaintext"><code>{
  "Tenants": [
    "Tenant1",
    "Tenant2"
  ],
  "ConnectionStrings": {
    "Tenant1": "Server=.;Database=Tenant1Db;Trusted_Connection=True;",
    "Tenant2": "Server=.;Database=Tenant2Db;Trusted_Connection=True;"
  }
}
</code></pre>

</div>



<p><strong>4. Multi-Tenant DbContext</strong><br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight plaintext"><code>public class ApplicationDbContext : DbContext
{
    public ApplicationDbContext(DbContextOptions&lt;ApplicationDbContext&gt; options): base(options) { }

    public DbSet&lt;Product&gt; Products { get; set; }
}
</code></pre>

</div>



<p>*<em>5. DbContext Factory *</em><br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight plaintext"><code>public interface ITenantDbContextFactory
{
    ApplicationDbContext CreateDbContext();
}

public class TenantDbContextFactory : ITenantDbContextFactory
{
    private readonly IConfiguration _config;
    private readonly TenantInfo _tenantInfo;

    public TenantDbContextFactory(IConfiguration config, TenantInfo tenantInfo)
    {
        _config = config;
        _tenantInfo = tenantInfo;
    }

    public ApplicationDbContext CreateDbContext()
    {
        var instituteId = _tenantInfo.InstituteId
            ?? throw new Exception("InstituteId not set in TenantInfo");

        var connectionString = _config.GetConnectionString(instituteId);
        if (string.IsNullOrEmpty(connectionString))
            throw new Exception($"No connection string for institute: {instituteId}");

        var optionsBuilder = new DbContextOptionsBuilder&lt;ApplicationDbContext&gt;();
        optionsBuilder.UseSqlServer(connectionString);

        return new ApplicationDbContext(optionsBuilder.Options);
    }
}
</code></pre>

</div>



<p><strong>Register it in DI:</strong><br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight plaintext"><code>builder.Services.AddScoped&lt;ITenantDbContextFactory,TenantDbContextFactory&gt;();
</code></pre>

</div>



<p><strong>6. Using DbContext in Service</strong><br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight plaintext"><code>public class Tenant1ProductService : IProductService
{
    private readonly ApplicationDbContext _db;

    public Tenant1ProductService(ITenantDbContextFactory dbContextFactory)
    {
        _db = dbContextFactory.CreateDbContext();
    }

    public string GetProductList()
    {
        var Products = _db.Products.Select(c =&gt; c.Name).ToList();
        return string.Join(", ", Products);
    }
}
</code></pre>

</div>



<p><strong>7. TenantServiceResolver</strong><br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight plaintext"><code>public class TenantServiceResolver : ITenantServiceResolver
{
    private readonly IServiceProvider _serviceProvider;
    private readonly TenantInfo _tenantInfo;

    public TenantServiceResolver(IServiceProvider serviceProvider, TenantInfo tenantInfo)
    {
        _serviceProvider = serviceProvider;
        _tenantInfo = tenantInfo;
    }

    public IProductService GetProductService()
    {
        return _tenantInfo.InstituteId switch
        {
            "Tenant1" =&gt; _serviceProvider.GetRequiredService&lt;Tenant1ProductService&gt;(),
            "Tenant2" =&gt; _serviceProvider.GetRequiredService&lt;Tenant2ProductService&gt;(),
            _ =&gt; throw new NotSupportedException("Invalid InstituteId")
        };
    }
}
</code></pre>

</div>



<p>*<em>8. DI Registration *</em><br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight plaintext"><code>builder.Services.AddHttpContextAccessor();
builder.Services.AddScoped&lt;TenantInfo&gt;();

builder.Services.AddScoped&lt;Tenant1ProductService&gt;();
builder.Services.AddScoped&lt;Tenant2ProductService&gt;();
builder.Services.AddScoped&lt;ITenantServiceResolver, TenantServiceResolver&gt;();

builder.Services.AddScoped&lt;ITenantDbContextFactory, TenantDbContextFactory&gt;();
</code></pre>

</div>



<p><strong>9. Controller</strong><br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight plaintext"><code>[ApiController]
[Route("api/[controller]")]
public class CourseController : ControllerBase
{
    private readonly ITenantServiceResolver _tenantServiceResolver;

    public CourseController(ITenantServiceResolver tenantServiceResolver)
    {
        _tenantServiceResolver = tenantServiceResolver;
    }

    [HttpGet]
    public IActionResult GetCourses()
    {
        var courseService = _tenantServiceResolver.GetCourseService();
        var courses = courseService.GetCourseList();
        return Ok(courses);
    }
}
</code></pre>

</div>



