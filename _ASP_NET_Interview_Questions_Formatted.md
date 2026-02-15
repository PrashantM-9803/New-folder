# Technical Interview Questions & Answers

## Table of Contents
1. [ASP.NET Core Interview Questions](#aspnet-core)
2. [ASP.NET Core Web API Interview Questions](#aspnet-web-api)
3. [Entity Framework Core Interview Questions](#entity-framework)
4. [.NET Security Interview Questions](#dotnet-security)
5. [Docker Interview Questions](#docker)
6. [Azure Interview Questions](#azure)

---

# ASP.NET Core Interview Questions {#aspnet-core}

## 1. What is ASP.NET Core and how is it different from ASP.NET MVC?

**Answer:**

ASP.NET MVC and ASP.NET Core are both frameworks for building web applications, but they differ significantly in architecture, platform support, and performance.

**ASP.NET MVC** is the older, Windows-only framework built on the .NET Framework. It follows the Model-View-Controller pattern, typically using Razor Views (.cshtml), Controllers, and Models. Applications are hosted on IIS and are limited to Windows servers. It's still used in many enterprise systems but is considered legacy for new development.

**ASP.NET Core** is a modern, open-source, cross-platform framework built on .NET Core / .NET 5+. It supports MVC, Razor Pages, Blazor, and Web API in a single unified framework. It runs on Windows, Linux, and macOS, can be hosted on Kestrel, IIS, or Nginx, and is optimized for high performance and cloud readiness.

### Key Differences:
- **Platform Support**: ASP.NET MVC (Windows only) vs ASP.NET Core (Cross-platform)
- **Architecture**: ASP.NET Core has built-in dependency injection and modular architecture
- **Performance**: ASP.NET Core is optimized for cloud-native applications and microservices
- **Hosting**: ASP.NET Core can run on Kestrel, IIS, or Nginx without requiring IIS

### Example: ASP.NET MVC Controller
```csharp
public class HomeController : Controller
{
   public ActionResult Index()
   {
       return View();
   }
}
```

### Example: ASP.NET Core Minimal Setup
```csharp
var builder = WebApplication.CreateBuilder(args);
builder.Services.AddControllersWithViews();
var app = builder.Build();
app.MapDefaultControllerRoute();
app.Run();
```

---

## 2. Explain the ASP.NET Core request processing pipeline.

**Answer:**

The ASP.NET Core request processing pipeline is a series of middleware components that handle incoming HTTP requests and responses. Each middleware component is responsible for specific tasks such as authentication, routing, logging, caching, encryption, and response generation.

### Pipeline Flow:
```
[HTTP Request]
    ↓
Kestrel Web Server
    ↓
Host / Middleware Pipeline (ordered!)
├─ Exception handler
├─ HTTPS redirection
├─ Static files
├─ Routing
├─ Authentication
├─ Authorization
├─ Endpoint execution (Minimal APIs / Controllers / Razor Pages)
    ↓
[HTTP Response]
```

### Key Middleware Components:

1. **Exception Handler**: Handles unhandled exceptions globally, logs them, and returns proper error responses.

2. **HSTS (HTTP Strict Transport Security)**: Enforces strict security policies, telling browsers to only interact with the application over HTTPS.

3. **HttpsRedirection**: Redirects HTTP requests to HTTPS to ensure all traffic is encrypted.

4. **Static Files**: Serves static files (HTML, CSS, JavaScript, images) directly without processing through the rest of the pipeline.

5. **Routing**: Maps incoming requests to appropriate controllers and actions based on URL patterns.

6. **CORS (Cross-Origin Resource Sharing)**: Handles cross-origin requests, allowing or denying them based on policy.

7. **Authentication**: Validates user credentials (JWT tokens, cookies, etc.).

8. **Authorization**: Determines whether authenticated users have permission to access specific resources.

9. **Endpoint**: The target of the request (controller action, Razor Page, or Minimal API endpoint).

### Example Program.cs:
```csharp
var builder = WebApplication.CreateBuilder(args);

builder.Services.AddControllers();
builder.Services.AddAuthorization();

var app = builder.Build();

if (app.Environment.IsDevelopment())
{
    app.UseDeveloperExceptionPage();
}
else
{
    app.UseExceptionHandler("/error");
    app.UseHsts();
}

app.UseHttpsRedirection();
app.UseStaticFiles();

// Custom middleware example
app.Use(async (context, next) =>
{
    const string headerName = "X-Correlation-Id";
    if (!context.Request.Headers.ContainsKey(headerName))
    {
        context.Response.Headers[headerName] = Guid.NewGuid().ToString();
    }
    await next();
});

app.UseRouting();
app.UseAuthorization();

app.MapGet("/hello", () => Results.Ok(new { message = "Hello!" }));
app.MapControllers();

app.Run();
```

---

## 3. What is middleware? Give examples.

**Answer:**

ASP.NET Core Middleware Components are the basic building blocks of the Request Processing Pipeline. They execute in the order they are added to the pipeline and each middleware component:
- Chooses whether to pass the HTTP Request to the next middleware
- Can perform tasks before and after the next component is invoked

### How Middleware Works:

Middleware components can:
1. **Handle** the incoming HTTP request by generating an HTTP response
2. **Process** the incoming HTTP request, modify it, and pass it to the next middleware
3. **Modify** the outgoing HTTP response before passing it back

### Built-in Middleware Components:
- **Static Files**: Serves static content from wwwroot folder
- **Routing**: Matches requests to endpoints
- **Authentication & Authorization**: Validates user credentials and permissions
- **CORS**: Handles cross-origin requests
- **Exception Handling**: Global error handling

### Example: Custom Middleware
```csharp
public class CustomMiddleware
{
   private readonly RequestDelegate _next;
   
   public CustomMiddleware(RequestDelegate next)
   {
       _next = next;
   }
   
   public async Task InvokeAsync(HttpContext context)
   {
       // Pre-processing logic
       Console.WriteLine($"Request: {context.Request.Path}");
       
       await _next(context); // Call the next middleware
       
       // Post-processing logic
       Console.WriteLine($"Response: {context.Response.StatusCode}");
   }
}

// Register in Program.cs
app.UseMiddleware<CustomMiddleware>();
```

---

## 4. How does dependency injection work in ASP.NET Core?

**Answer:**

ASP.NET Core Dependency Injection (DI) is a design pattern used to achieve loose coupling in software development. Instead of creating dependencies directly within a class, dependencies are injected into the class from outside, typically through constructor parameters.

### Key Components:

1. **Service Interface** - Declares the contract
2. **Service Implementation** - Implements the interface
3. **Service Registration** - Registers in the DI container
4. **Service Injection** - Injects into classes that need it

### Example Implementation:

**Model:**
```csharp
public class Student
{
    public int StudentId { get; set; }
    public string? Name { get; set; }
    public string? Branch { get; set; }
}
```

**Service Interface:**
```csharp
public interface IStudentRepository
{
    Student GetStudentById(int StudentId);
    List<Student> GetAllStudent();
}
```

**Service Implementation:**
```csharp
public class StudentRepository : IStudentRepository
{
    public Student GetStudentById(int StudentId)
    {
        // Implementation
    }

    public List<Student> GetAllStudent()
    {
        // Implementation
    }
}
```

**Program.cs Registration:**
```csharp
var builder = WebApplication.CreateBuilder(args);

// Register the service
builder.Services.AddScoped<IStudentRepository, StudentRepository>();

var app = builder.Build();
app.Run();
```

**Usage in Controller:**
```csharp
public class StudentController : ControllerBase
{
    private readonly IStudentRepository _repository;

    public StudentController(IStudentRepository repository)
    {
        _repository = repository;
    }

    public IActionResult GetStudent(int id)
    {
        var student = _repository.GetStudentById(id);
        return Ok(student);
    }
}
```

---

## 5. What are the different lifetimes for services in DI?

**Answer:**

There are three main service lifetimes: **Transient**, **Scoped**, and **Singleton**.

### Transient
- A new instance is created **every time** it is requested
- **Use Case**: Lightweight, stateless services
- **Registration**: `AddTransient<IService, Service>()`
- Even within the same HTTP request, multiple injections create different instances

### Scoped
- A single instance is created **once per scope** (typically one HTTP request)
- **Use Case**: Services maintaining state during a single request (e.g., DbContext)
- **Registration**: `AddScoped<IService, Service>()`
- Same instance shared within one HTTP request, new instance for each request

### Singleton
- A single instance is created **on first request** and reused throughout the application's lifetime
- **Use Case**: Shared, application-wide state like logging, configuration, caching
- **Registration**: `AddSingleton<IService, Service>()`
- Same instance across all users and requests; only disposed when app shuts down

### Comparison Table:

| Aspect | Transient | Scoped | Singleton |
|--------|-----------|--------|-----------|
| **Instance Created** | Every request | Per HTTP request | Once at startup/first use |
| **Instance Shared** | Never | Within single request | Entire application lifetime |
| **Use Case** | Stateless services | DbContext, per-request state | Caching, logging, config |
| **Memory Impact** | High (many instances) | Medium | Low (single instance) |
| **Thread-Safe** | Not required | Should be | Must be thread-safe |

### Example:
```csharp
builder.Services.AddTransient<ITransientService, TransientService>();
builder.Services.AddScoped<IScopedService, ScopedService>();
builder.Services.AddSingleton<ISingletonService, SingletonService>();
```

---

## 6. What is the purpose of the Startup class?

**Answer:**

The Startup class is the central entry point for configuring an application's behavior, services, and request pipeline. It provides two critical methods:

### ConfigureServices Method
- Registers all services needed by the application
- Takes `IServiceCollection` as a parameter
- Example: Adding MVC, Entity Framework, Authentication, etc.

### Configure Method
- Configures the HTTP request processing pipeline
- Takes `IApplicationBuilder` and `IWebHostEnvironment` as parameters
- Sets up middleware components in execution order

### Example:
```csharp
public class Startup
{
    public void ConfigureServices(IServiceCollection services)
    {
        services.AddMvc();
        services.AddScoped<IStudentRepository, StudentRepository>();
    }

    public void Configure(IApplicationBuilder app, IWebHostEnvironment env)
    {
        app.UseRouting();
        app.UseEndpoints(endpoints =>
        {
            endpoints.MapGet("/", async context => {
                await context.Response.WriteAsync("Hello From ASP.NET Core");
            });
        });
    }
}
```

### Modern Approach (Program.cs in .NET 6+)
The Startup class functionality is now integrated directly into Program.cs using the minimal hosting model, simplifying configuration.

---

## 7. How do you manage configuration and secrets in ASP.NET Core?

**Answer:**

Configuration and secrets management involves using multiple sources in a specific order of precedence.

### Configuration Sources (in order):
1. **appsettings.json** - Default configuration file
2. **appsettings.{EnvironmentName}.json** - Environment-specific settings
3. **Environment Variables** - Override file settings
4. **Command-line Arguments** - Override all sources
5. **Default Values** - In-memory defaults

### Example appsettings.json:
```json
{
  "Logging": {
    "LogLevel": {
      "Default": "Information"
    }
  },
  "ConnectionStrings": {
    "DefaultConnection": "Server=localhost;Database=MyDb"
  }
}
```

### Secrets Management:

#### Development: Secret Manager
```bash
dotnet user-secrets init
dotnet user-secrets set "ApiKey" "your-secret-key"
```

#### Production: Secure Solutions
- **Azure Key Vault**: Cloud-based secure secret storage
- **AWS Secrets Manager**: AWS equivalent
- **HashiCorp Vault**: On-premises solution
- **Environment Variables**: Less secure but usable (be careful!)

### Usage in Code:
```csharp
var builder = WebApplication.CreateBuilder(args);

// Configuration is automatically loaded
var apiKey = builder.Configuration["ApiKey"];
var connString = builder.Configuration.GetConnectionString("DefaultConnection");

var app = builder.Build();
```

---

## 8. What is Kestrel?

**Answer:**

Kestrel is Microsoft's cross-platform web server designed specifically for ASP.NET Core. It's the default web server for hosting ASP.NET Core applications on Windows, Linux, and macOS.

### Key Characteristics:
- **Lightweight and Fast**: Optimized for performance
- **Cross-Platform**: Runs on Windows, Linux, and macOS
- **Default**: Built-in to ASP.NET Core by default
- **Embedded**: Can be hosted in any .NET application

### Process Names:

**Self-Contained Deployment:**
- Application packaged with .NET runtime
- Process name: Your application's executable name (e.g., MyApp)

**Framework-Dependent Deployment:**
- Relies on pre-installed .NET runtime
- Process name: `dotnet.exe` (Windows) or `dotnet` (Linux/macOS)

### Why Kestrel?
- High throughput
- Supports HTTP/1.1 and HTTP/2
- No IIS dependency required
- Can be used with reverse proxies (Nginx, IIS, Apache)

---

## 9. How do you enable logging in ASP.NET Core?

**Answer:**

ASP.NET Core includes a built-in logging framework. Configuration involves setting log levels and configuring logging providers.

### Step 1: Configure Log Levels in appsettings.json
```json
{
  "Logging": {
    "LogLevel": {
      "Default": "Information",
      "Microsoft.AspNetCore": "Warning"
    }
  }
}
```

### Log Levels (in order of severity):
1. **Trace** - Most detailed information
2. **Debug** - Debugging information
3. **Information** - General information
4. **Warning** - Warning messages
5. **Error** - Error messages
6. **Critical** - Critical failures
7. **None** - No logging

### Step 2: Add Logging Services in Program.cs
```csharp
var builder = WebApplication.CreateBuilder(args);

builder.Host.ConfigureLogging(logging =>
{
    logging.ClearProviders();
    logging.AddConsole();
    logging.AddDebug();
});

var app = builder.Build();
```

### Step 3: Inject Logger in Your Classes
```csharp
public class HomeController : ControllerBase
{
    private readonly ILogger<HomeController> _logger;

    public HomeController(ILogger<HomeController> logger)
    {
        _logger = logger;
    }

    public IActionResult Index()
    {
        _logger.LogInformation("Index method called");
        _logger.LogWarning("This is a warning");
        _logger.LogError("This is an error");
        
        return View();
    }
}
```

### Default Logging Providers:
- **Console**: Logs to console output
- **Debug**: Logs to Visual Studio debug output
- **EventSource**: Windows Event Tracing
- **Windows Event Log**: Windows only

---

## 10. What is the difference between IApplicationBuilder and IServiceCollection?

**Answer:**

These two interfaces serve different purposes in the ASP.NET Core application lifecycle:

### IServiceCollection
- **Purpose**: Registers services with the dependency injection container
- **When Used**: During application startup (ConfigureServices or Program.cs)
- **Function**: Configures what services are available
- **Example**: `services.AddScoped<IRepository, Repository>()`

### IApplicationBuilder
- **Purpose**: Defines the HTTP request processing pipeline
- **When Used**: During pipeline configuration (Configure or Program.cs)
- **Function**: Chains middleware components for request handling
- **Example**: `app.UseHttpsRedirection()`

### Comparison:

| Aspect | IServiceCollection | IApplicationBuilder |
|--------|-------------------|-------------------|
| **Purpose** | Register services | Build request pipeline |
| **Method** | ConfigureServices | Configure |
| **Usage** | Dependency Injection setup | Middleware setup |
| **Example** | AddScoped, AddSingleton | UseAuthentication, UseRouting |
| **Timing** | Before app starts | Defines pipeline |
| **Related To** | Service Container | Middleware Pipeline |

### Example:
```csharp
// IServiceCollection - Register services
builder.Services.AddMvc();
builder.Services.AddScoped<IUserService, UserService>();

// IApplicationBuilder - Build pipeline
app.UseHttpsRedirection();
app.UseRouting();
app.UseAuthorization();
app.MapControllers();
```

---

# ASP.NET Core Web API Interview Questions {#aspnet-web-api}

## 1. How do you create a RESTful API in ASP.NET Core?

**Answer:**

Creating a RESTful API involves using controllers with attribute routing and standard HTTP verbs (GET, POST, PUT, DELETE).

### Step 1: Create a New Project
```bash
dotnet new webapi -n ApiProject
cd ApiProject
```

### Step 2: Define a Model
```csharp
namespace ApiProject.Models
{
    public class Product
    {
        public int Id { get; set; }
        public string Name { get; set; }
        public decimal Price { get; set; }
    }
}
```

### Step 3: Create a Controller
```csharp
[ApiController]
[Route("api/[controller]")]
public class ProductsController : ControllerBase
{
    private static List<Product> _products = new List<Product>
    {
        new Product { Id = 1, Name = "Laptop", Price = 999.99M },
        new Product { Id = 2, Name = "Mouse", Price = 19.99M }
    };
    
    // GET: api/products
    [HttpGet]
    public ActionResult<IEnumerable<Product>> GetProducts()
    {
        return Ok(_products);
    }
    
    // GET: api/products/1
    [HttpGet("{id}")]
    public ActionResult<Product> GetProduct(int id)
    {
        var product = _products.FirstOrDefault(p => p.Id == id);
        if (product == null)
            return NotFound();
        
        return Ok(product);
    }

    // POST: api/products
    [HttpPost]
    public ActionResult<Product> PostProduct(Product newProduct)
    {
        newProduct.Id = _products.Max(p => p.Id) + 1;
        _products.Add(newProduct);
        
        return CreatedAtAction(nameof(GetProduct), 
            new { id = newProduct.Id }, newProduct);
    }

    // PUT: api/products/1
    [HttpPut("{id}")]
    public IActionResult PutProduct(int id, Product product)
    {
        var existing = _products.FirstOrDefault(p => p.Id == id);
        if (existing == null)
            return NotFound();
        
        existing.Name = product.Name;
        existing.Price = product.Price;
        
        return NoContent();
    }

    // DELETE: api/products/1
    [HttpDelete("{id}")]
    public IActionResult DeleteProduct(int id)
    {
        var product = _products.FirstOrDefault(p => p.Id == id);
        if (product == null)
            return NotFound();
        
        _products.Remove(product);
        return NoContent();
    }
}
```

### Step 4: Run the API
```bash
dotnet run
```

### Key REST Concepts:
- **GET**: Retrieve resource(s)
- **POST**: Create new resource
- **PUT**: Update existing resource
- **DELETE**: Remove resource
- **Status Codes**: 200 (OK), 201 (Created), 204 (No Content), 404 (Not Found), 400 (Bad Request)

---

## 2. What is the [ApiController] attribute and what does it do?

**Answer:**

The `[ApiController]` attribute is a special attribute that marks a controller as an API controller and provides automatic default behaviors.

### Benefits:

1. **Automatic Model Validation**
   - Automatically returns HTTP 400 with validation errors if ModelState is invalid
   - No need for manual `if (!ModelState.IsValid)` checks

2. **Automatic Parameter Binding**
   - Complex types → Request Body (JSON)
   - Simple/primitive types → Route or Query String
   - Reduces need for explicit binding attributes

3. **Enforces Attribute Routing**
   - Requires the use of attribute routing ([Route], [HttpGet], etc.)
   - Conventional routing not supported

4. **Standard Error Response Format**
   - Returns errors in a consistent ProblemDetails format
   - Complies with RFC 7231

### Example:
```csharp
[ApiController]
[Route("api/[controller]")]
public class ProductsController : ControllerBase
{
    [HttpPost]
    public ActionResult<Product> CreateProduct([FromBody] Product product)
    {
        // If ModelState is invalid, API automatically returns 400
        // No need to check ModelState here
        
        return CreatedAtAction(nameof(GetProduct), new { id = product.Id }, product);
    }
}
```

---

## 3. How do you implement routing in Web API?

**Answer:**

ASP.NET Web API supports two types of routing: Convention-based and Attribute routing.

### Convention-Based Routing
Routes follow a standard template pattern. Less common in modern REST APIs.

```csharp
public static class WebApiConfig
{
    public static void Register(HttpConfiguration config)
    {
        config.Routes.MapHttpRoute(
            name: "DefaultApi",
            routeTemplate: "api/{controller}/{id}",
            defaults: new { id = RouteParameter.Optional }
        );
    }
}
```

### Attribute Routing (Recommended)
Provides explicit control over URIs using attributes.

```csharp
[ApiController]
[Route("api/[controller]")]
public class ProductsController : ControllerBase
{
    // GET api/products
    [HttpGet]
    public IActionResult GetAllProducts() { }

    // GET api/products/5
    [HttpGet("{id:int}")]
    public IActionResult GetProductById(int id) { }

    // POST api/products
    [HttpPost]
    public IActionResult CreateProduct([FromBody] Product product) { }

    // PUT api/products/5
    [HttpPut("{id:int}")]
    public IActionResult UpdateProduct(int id, [FromBody] Product product) { }

    // DELETE api/products/5
    [HttpDelete("{id:int}")]
    public IActionResult DeleteProduct(int id) { }
}
```

### Route Constraints:
- `:int` - Matches integers
- `:string` - Matches strings
- `:datetime` - Matches dates
- `:double` - Matches decimal numbers

### Route Parameters:
- `{controller}` - Replaced by controller name
- `{id}` - Route parameter from method
- `{action}` - Action method name (if needed)

---

## 4. What is model binding and model validation?

**Answer:**

Model binding and validation are sequential processes that prepare and validate data before it reaches your business logic.

### Model Binding
Automatically maps data from HTTP request to C# objects.

**Data Sources (in order):**
1. Route data (URL segments)
2. Query strings (URL parameters)
3. Request body (JSON/XML)
4. HTTP headers

### Model Validation
Ensures data conforms to business rules using attributes.

### Example Model:
```csharp
public class Employee
{
    [Required]
    public int Id { get; set; }
    
    [Required(ErrorMessage = "Name is required")]
    [StringLength(100)]
    public string Name { get; set; }
    
    [Range(18, 65)]
    public int Age { get; set; }
    
    [EmailAddress]
    public string Email { get; set; }
    
    [RegularExpression(@"^\d{10}$", ErrorMessage = "Phone must be 10 digits")]
    public string Phone { get; set; }
}
```

### How They Work Together:

```csharp
[HttpPost]
public ActionResult<Employee> CreateEmployee([FromBody] Employee employee)
{
    // Step 1: Model binding - Maps JSON to Employee object
    // Step 2: Model validation - Checks against attributes
    
    // Step 3: Check ModelState
    if (!ModelState.IsValid)
        return BadRequest(ModelState);
    
    // Step 4: Process valid data
    // Save to database
    
    return CreatedAtAction(nameof(GetEmployee), new { id = employee.Id }, employee);
}
```

### Validation Attributes:
- `[Required]` - Field is mandatory
- `[StringLength(max)]` - String length constraint
- `[Range(min, max)]` - Numeric range
- `[EmailAddress]` - Valid email format
- `[RegularExpression(pattern)]` - Regex matching
- `[Compare]` - Compare two fields
- `[Display(Name="...")]` - Custom display name

---

## 5. How do you return different HTTP status codes from an API action?

**Answer:**

ASP.NET Core provides helper methods to return specific HTTP status codes.

### Common Status Codes:

```csharp
[ApiController]
[Route("api/[controller]")]
public class EmployeesController : ControllerBase
{
    // 200 OK
    [HttpGet("{id}")]
    public ActionResult<Employee> GetEmployee(int id)
    {
        var employee = GetEmployeeById(id);
        if (employee == null)
            return NotFound(); // 404
        
        return Ok(employee); // 200
    }

    // 201 Created
    [HttpPost]
    public ActionResult<Employee> CreateEmployee([FromBody] Employee employee)
    {
        // Create employee
        var created = SaveEmployee(employee);
        
        return CreatedAtAction(nameof(GetEmployee), 
            new { id = created.Id }, created); // 201
    }

    // 204 No Content
    [HttpPut("{id}")]
    public IActionResult UpdateEmployee(int id, [FromBody] Employee employee)
    {
        var existing = GetEmployeeById(id);
        if (existing == null)
            return NotFound(); // 404
        
        UpdateEmployeeData(existing, employee);
        
        return NoContent(); // 204
    }

    // 400 Bad Request
    [HttpPost("validate")]
    public IActionResult ValidateEmployee([FromBody] Employee employee)
    {
        if (string.IsNullOrEmpty(employee.Name))
            return BadRequest("Name is required");
        
        return Ok();
    }

    // 403 Forbidden
    [HttpDelete("{id}")]
    [Authorize(Roles = "Admin")]
    public IActionResult DeleteEmployee(int id)
    {
        return Forbid(); // 403
    }

    // 500 Internal Server Error
    public IActionResult ProcessData()
    {
        try
        {
            // Some processing
        }
        catch (Exception ex)
        {
            return StatusCode(StatusCodes.Status500InternalServerError, 
                new { message = "An error occurred" });
        }
    }
}
```

### Common Response Methods:

| Method | Status Code | Use Case |
|--------|------------|----------|
| `Ok()` | 200 | Successful GET/POST |
| `Created()` | 201 | Resource created |
| `CreatedAtAction()` | 201 | Created with location header |
| `NoContent()` | 204 | Successful update/delete |
| `BadRequest()` | 400 | Invalid input |
| `Unauthorized()` | 401 | Not authenticated |
| `Forbid()` | 403 | Not authorized |
| `NotFound()` | 404 | Resource not found |
| `Conflict()` | 409 | Resource conflict |
| `StatusCode()` | Custom | Any status code |

---

## 6. How do you secure a Web API (e.g., JWT, OAuth)?

**Answer:**

Web APIs are secured using OAuth 2.0 for authorization and JSON Web Tokens (JWTs) for stateless authentication.

### JWT Authentication Implementation:

#### Step 1: Add Required NuGet Package
```bash
dotnet add package Microsoft.AspNetCore.Authentication.JwtBearer
```

#### Step 2: Configure appsettings.json
```json
{
  "Jwt": {
    "Key": "your-very-secret-key-min-32-characters",
    "Issuer": "your-app",
    "Audience": "your-app-users"
  }
}
```

#### Step 3: Configure JWT in Program.cs
```csharp
var builder = WebApplication.CreateBuilder(args);

var jwtSettings = builder.Configuration.GetSection("Jwt");
var key = Encoding.ASCII.GetBytes(jwtSettings["Key"]);

builder.Services.AddAuthentication(options =>
{
    options.DefaultAuthenticateScheme = JwtBearerDefaults.AuthenticationScheme;
    options.DefaultChallengeScheme = JwtBearerDefaults.AuthenticationScheme;
})
.AddJwtBearer(options =>
{
    options.TokenValidationParameters = new TokenValidationParameters
    {
        ValidateIssuerSigningKey = true,
        IssuerSigningKey = new SymmetricSecurityKey(key),
        ValidateIssuer = true,
        ValidIssuer = jwtSettings["Issuer"],
        ValidateAudience = true,
        ValidAudience = jwtSettings["Audience"],
        ValidateLifetime = true
    };
});

builder.Services.AddAuthorization();

var app = builder.Build();

app.UseAuthentication();
app.UseAuthorization();

app.Run();
```

#### Step 4: Create Token Generation Service
```csharp
public interface IAuthService
{
    string GenerateToken(User user);
}

public class AuthService : IAuthService
{
    private readonly IConfiguration _configuration;

    public AuthService(IConfiguration configuration)
    {
        _configuration = configuration;
    }

    public string GenerateToken(User user)
    {
        var jwtSettings = _configuration.GetSection("Jwt");
        var key = new SymmetricSecurityKey(
            Encoding.ASCII.GetBytes(jwtSettings["Key"]));
        
        var claims = new[]
        {
            new Claim(ClaimTypes.NameIdentifier, user.Id.ToString()),
            new Claim(ClaimTypes.Name, user.Username),
            new Claim(ClaimTypes.Email, user.Email),
            new Claim(ClaimTypes.Role, user.Role)
        };

        var token = new JwtSecurityToken(
            issuer: jwtSettings["Issuer"],
            audience: jwtSettings["Audience"],
            claims: claims,
            expires: DateTime.UtcNow.AddHours(24),
            signingCredentials: new SigningCredentials(
                key, SecurityAlgorithms.HmacSha256Signature)
        );

        return new JwtSecurityTokenHandler().WriteToken(token);
    }
}
```

#### Step 5: Create Login Endpoint
```csharp
[ApiController]
[Route("api/[controller]")]
public class AuthController : ControllerBase
{
    private readonly IAuthService _authService;

    public AuthController(IAuthService authService)
    {
        _authService = authService;
    }

    [HttpPost("login")]
    [AllowAnonymous]
    public IActionResult Login([FromBody] LoginRequest request)
    {
        // Validate credentials against database
        var user = AuthenticateUser(request.Username, request.Password);
        
        if (user == null)
            return Unauthorized();

        var token = _authService.GenerateToken(user);
        return Ok(new { token });
    }
}
```

#### Step 6: Secure Your Endpoints
```csharp
[HttpGet]
[Authorize]
public IActionResult GetProtectedData()
{
    return Ok("This is protected data");
}

[HttpDelete("{id}")]
[Authorize(Roles = "Admin")]
public IActionResult DeleteUser(int id)
{
    return Ok("User deleted");
}
```

### OAuth 2.0
- External authorization provider (Google, Microsoft, GitHub)
- User approves access to their data
- App receives access token for API calls
- More suitable for third-party integrations

---

## 7. What is CORS and how do you configure it?

**Answer:**

CORS (Cross-Origin Resource Sharing) is a browser security mechanism that allows controlled cross-domain requests from a web page to access resources on a different origin.

### The Problem CORS Solves:
By default, browsers block requests from one domain (e.g., `https://frontend.com`) to another domain (e.g., `https://api.example.com`) for security reasons.

### Configuring CORS in Program.cs:

```csharp
var builder = WebApplication.CreateBuilder(args);

const string MyAllowSpecificOrigins = "_myAllowSpecificOrigins";

// Step 1: Define CORS policy
builder.Services.AddCors(options =>
{
    options.AddPolicy(name: MyAllowSpecificOrigins,
        policy =>
        {
            policy.WithOrigins(
                "http://localhost:3000",
                "https://www.example.com")
                  .AllowAnyHeader()
                  .AllowAnyMethod()
                  .AllowCredentials();
        });
});

builder.Services.AddControllers();

var app = builder.Build();

app.UseHttpsRedirection();
app.UseStaticFiles();
app.UseRouting();

// Step 2: Enable CORS middleware
app.UseCors(MyAllowSpecificOrigins);

app.UseAuthorization();
app.MapControllers();
app.Run();
```

### CORS Policy Options:

```csharp
// Allow specific origins
policy.WithOrigins("http://localhost:3000", "https://example.com")

// Allow any origin (NOT SECURE!)
policy.AllowAnyOrigin()

// Allow specific headers
policy.WithHeaders("Authorization", "Content-Type")

// Allow any header
policy.AllowAnyHeader()

// Allow specific methods
policy.WithMethods("GET", "POST")

// Allow any method
policy.AllowAnyMethod()

// Allow credentials (cookies, auth headers)
policy.AllowCredentials()

// Specify max age for preflight cache
policy.SetPreflightMaxAge(TimeSpan.FromSeconds(600))
```

### Important Security Note:
**Never use** `AllowAnyOrigin()` with `AllowCredentials()` together—this is a serious security vulnerability!

---

## 8. How do you handle exceptions and errors globally?

**Answer:**

The modern recommended approach is using the `IExceptionHandler` interface (.NET 8+) or middleware for global exception handling.

### Method 1: IExceptionHandler (.NET 8+) - Recommended

**Step 1: Create Exception Handler**
```csharp
using Microsoft.AspNetCore.Diagnostics;
using Microsoft.AspNetCore.Mvc;

public sealed class GlobalExceptionHandler(ILogger<GlobalExceptionHandler> logger) 
    : IExceptionHandler
{
    public async ValueTask<bool> TryHandleAsync(
        HttpContext httpContext, 
        Exception exception, 
        CancellationToken cancellationToken)
    {
        logger.LogError(exception, 
            "An unhandled exception occurred: {Message}", 
            exception.Message);

        var problemDetails = new ProblemDetails
        {
            Status = StatusCodes.Status500InternalServerError,
            Title = "An error occurred while processing your request.",
            Detail = exception.Message,
            Instance = httpContext.Request.Path
        };

        await httpContext.Response.WriteAsJsonAsync(
            problemDetails, 
            cancellationToken);
        
        return true;
    }
}
```

**Step 2: Register in Program.cs**
```csharp
var builder = WebApplication.CreateBuilder(args);

builder.Services.AddControllers();
builder.Services.AddExceptionHandler<GlobalExceptionHandler>();
builder.Services.AddProblemDetails();

var app = builder.Build();

app.UseExceptionHandler();
app.UseHttpsRedirection();
app.MapControllers();
app.Run();
```

### Method 2: Exception Handling Middleware

**Create Middleware:**
```csharp
public class ExceptionHandlingMiddleware
{
    private readonly RequestDelegate _next;
    private readonly ILogger<ExceptionHandlingMiddleware> _logger;

    public ExceptionHandlingMiddleware(RequestDelegate next, 
        ILogger<ExceptionHandlingMiddleware> logger)
    {
        _next = next;
        _logger = logger;
    }

    public async Task InvokeAsync(HttpContext context)
    {
        try
        {
            await _next(context);
        }
        catch (Exception ex)
        {
            _logger.LogError(ex, "An unhandled exception occurred");
            await HandleExceptionAsync(context, ex);
        }
    }

    private static Task HandleExceptionAsync(HttpContext context, Exception exception)
    {
        context.Response.ContentType = "application/json";
        context.Response.StatusCode = StatusCodes.Status500InternalServerError;

        var response = new
        {
            message = "An error occurred while processing your request.",
            error = exception.Message,
            timestamp = DateTime.UtcNow
        };

        return context.Response.WriteAsJsonAsync(response);
    }
}
```

**Register in Program.cs:**
```csharp
app.UseMiddleware<ExceptionHandlingMiddleware>();
```

---

## 9. How do you version a Web API?

**Answer:**

API versioning allows you to evolve your API while maintaining backward compatibility.

### Versioning Strategies:

### 1. URI Path Versioning (Most Common)
```csharp
[ApiController]
[Route("api/v{version:apiVersion}/[controller]")]
public class ProductsController : ControllerBase
{
    [HttpGet]
    public IActionResult GetProducts()
    {
        return Ok(new[] { /* products */ });
    }
}
```

URL: `https://api.example.com/api/v1/products`

### 2. Query Parameter Versioning
```csharp
[ApiController]
[Route("api/[controller]")]
public class ProductsController : ControllerBase
{
    [HttpGet]
    public IActionResult GetProducts([FromQuery] int version = 1)
    {
        if (version == 1)
            return Ok(GetProductsV1());
        else if (version == 2)
            return Ok(GetProductsV2());
        
        return BadRequest("Unsupported version");
    }
}
```

URL: `https://api.example.com/api/products?version=2`

### 3. Custom Header Versioning
```csharp
[ApiController]
[Route("api/[controller]")]
public class ProductsController : ControllerBase
{
    [HttpGet]
    public IActionResult GetProducts([FromHeader(Name = "X-API-Version")] string version = "1")
    {
        // Handle different versions
        return Ok(new { version, data = /* ... */ });
    }
}
```

Header: `X-API-Version: 2`

### Using Microsoft.AspNetCore.Mvc.Versioning

**Install Package:**
```bash
dotnet add package Microsoft.AspNetCore.Mvc.Versioning
```

**Configure in Program.cs:**
```csharp
builder.Services.AddApiVersioning(options =>
{
    options.DefaultApiVersion = new ApiVersion(1, 0);
    options.AssumeDefaultVersionWhenUnspecified = true;
    options.ReportApiVersions = true;
});

builder.Services.AddApiVersioningApiExplorer(options =>
{
    options.GroupNameFormat = "'v'VVV";
});
```

**Usage:**
```csharp
[ApiController]
[Route("api/v{version:apiVersion}/[controller]")]
public class ProductsV1Controller : ControllerBase
{
    [ApiVersion("1.0")]
    [HttpGet]
    public IActionResult GetProducts()
    {
        // V1 implementation
    }
}

[ApiController]
[Route("api/v{version:apiVersion}/[controller]")]
public class ProductsV2Controller : ControllerBase
{
    [ApiVersion("2.0")]
    [HttpGet]
    public IActionResult GetProducts()
    {
        // V2 implementation
    }
}
```

---

## 10. What are filters in ASP.NET Core?

**Answer:**

Filters allow you to run custom logic before or after specific stages of the request processing pipeline, enabling cleaner separation of cross-cutting concerns.

### Types of Filters:

### 1. Authorization Filters
Run first and determine if the user can access the resource.

```csharp
[AttributeUsage(AttributeTargets.Class | AttributeTargets.Method)]
public class CustomAuthorizationFilter : Attribute, IAuthorizationFilter
{
    public void OnAuthorization(AuthorizationFilterContext context)
    {
        if (!context.HttpContext.User.Identity?.IsAuthenticated ?? false)
        {
            context.Result = new ForbidResult();
        }
    }
}

// Usage
[CustomAuthorizationFilter]
[HttpGet]
public IActionResult GetSecureData()
{
    return Ok("Secure data");
}
```

### 2. Resource Filters
Run after authorization but before model binding. Useful for caching.

```csharp
public class CachingFilter : Attribute, IResourceFilter
{
    public void OnResourceExecuting(ResourceExecutingContext context)
    {
        // Before execution
    }

    public void OnResourceExecuted(ResourceExecutedContext context)
    {
        // After execution
    }
}
```

### 3. Action Filters
Run before and after action methods. Useful for logging and validation.

```csharp
public class LoggingActionFilter : Attribute, IAsyncActionFilter
{
    private readonly ILogger<LoggingActionFilter> _logger;

    public LoggingActionFilter(ILogger<LoggingActionFilter> logger)
    {
        _logger = logger;
    }

    public async Task OnActionExecutionAsync(
        ActionExecutingContext context, 
        ActionExecutionDelegate next)
    {
        _logger.LogInformation("Executing action: {Action}", 
            context.ActionDescriptor.DisplayName);

        var result = await next();

        _logger.LogInformation("Completed action: {Action}", 
            context.ActionDescriptor.DisplayName);
    }
}
```

### 4. Exception Filters
Run when an unhandled exception occurs.

```csharp
public class ExceptionFilter : IAsyncExceptionFilter
{
    private readonly ILogger<ExceptionFilter> _logger;

    public ExceptionFilter(ILogger<ExceptionFilter> logger)
    {
        _logger = logger;
    }

    public Task OnExceptionAsync(ExceptionContext context)
    {
        _logger.LogError(context.Exception, "An error occurred");

        context.Result = new ObjectResult(new
        {
            message = "An error occurred",
            error = context.Exception.Message
        })
        {
            StatusCode = StatusCodes.Status500InternalServerError
        };

        context.ExceptionHandled = true;
        return Task.CompletedTask;
    }
}
```

### 5. Result Filters
Run before and after result execution.

```csharp
public class ResultFilter : Attribute, IAsyncResultFilter
{
    public async Task OnResultExecutionAsync(
        ResultExecutingContext context, 
        ResultExecutionDelegate next)
    {
        // Before result
        await next();
        // After result
    }
}
```

### Registering Filters Globally:

```csharp
builder.Services.AddControllers(options =>
{
    options.Filters.Add<ExceptionFilter>();
    options.Filters.Add<LoggingActionFilter>();
});
```

---

# Entity Framework Core Interview Questions {#entity-framework}

## 1. What is Entity Framework Core and how does it differ from classic Entity Framework?

**Answer:**

Entity Framework (EF) Core is a modern, lightweight, cross-platform Object-Relational Mapper (ORM) that enables .NET developers to work with databases using C# objects instead of writing SQL queries manually.

### Key Differences from EF6:

| Aspect | EF Core | EF6 |
|--------|---------|-----|
| **Platform** | Cross-platform (Windows, Linux, macOS) | Windows only |
| **Framework** | .NET Core / .NET 5+ | .NET Framework only |
| **Architecture** | Modern, modular, lightweight | Legacy, monolithic |
| **Performance** | Optimized, better query performance | Slower |
| **Development** | Actively developed, new features | Stable, no new features |
| **Usage** | Recommended for new projects | Maintenance only |

### Benefits of EF Core:
- Lightweight and modular
- Cross-platform support
- Better performance
- Modern async/await support
- LINQ integration
- Built-in dependency injection

---

## 2. Explain the difference between Code First and Database First approaches.

**Answer:**

### Code First Approach
You define your data model as C# classes, and EF Core generates the database schema.

**Workflow:**
1. Define model classes
2. Create DbContext
3. Use migrations to create/update database

**Advantages:**
- Full control over code
- Easy to version control
- Suitable for new projects
- Agile development

**Example:**
```csharp
public class Student
{
    public int Id { get; set; }
    public string Name { get; set; }
    public string Email { get; set; }
}

public class SchoolContext : DbContext
{
    public DbSet<Student> Students { get; set; }
}

// Create migration
dotnet ef migrations add InitialCreate

// Apply migration
dotnet ef database update
```

### Database First Approach
You start with an existing database, and EF Core generates model classes from it.

**Workflow:**
1. Database already exists
2. Use scaffolding to generate model classes
3. Use migrations for future changes

**Advantages:**
- Work with existing databases
- Good for legacy systems
- Database design is separate

**Example:**
```bash
# Scaffold from existing database
dotnet ef dbcontext scaffold "Server=localhost;Database=SchoolDb;User Id=sa;Password=..." Microsoft.EntityFrameworkCore.SqlServer

# This generates:
# - SchoolContext.cs (DbContext)
# - Student.cs, Course.cs (Model classes)
# - Grade.cs, etc.
```

### Comparison:

| Aspect | Code First | Database First |
|--------|-----------|-----------------|
| **Starting Point** | C# Classes | Database Schema |
| **Best For** | New projects | Legacy systems |
| **Control** | Full code control | Database focused |
| **Versioning** | Easy (migrations in code) | Requires DB tools |
| **Team Size** | Smaller teams | Large teams with DBA |

---

## 3. How do you configure relationships (one-to-many, many-to-many) in EF Core?

**Answer:**

Relationships can be configured using Data Annotations or Fluent API.

### One-to-Many Relationship

**Using Convention (Simplest):**
```csharp
public class Author
{
    public int AuthorId { get; set; }
    public string Name { get; set; }
    
    // Navigation property - collection
    public ICollection<Book> Books { get; set; }
}

public class Book
{
    public int BookId { get; set; }
    public string Title { get; set; }
    public int AuthorId { get; set; } // Foreign key
    
    // Navigation property - single reference
    public Author Author { get; set; }
}
```

**Using Fluent API:**
```csharp
protected override void OnModelCreating(ModelBuilder modelBuilder)
{
    modelBuilder.Entity<Book>()
        .HasOne<Author>(b => b.Author)
        .WithMany(a => a.Books)
        .HasForeignKey(b => b.AuthorId)
        .OnDelete(DeleteBehavior.Cascade);
}
```

### Many-to-Many Relationship

**Using Convention (EF Core 5.0+):**
```csharp
public class Student
{
    public int StudentId { get; set; }
    public string Name { get; set; }
    
    // Collection navigation property
    public ICollection<Course> Courses { get; set; }
}

public class Course
{
    public int CourseId { get; set; }
    public string Title { get; set; }
    
    // Collection navigation property
    public ICollection<Student> Students { get; set; }
}

// EF Core automatically creates join table: StudentCourse
```

**Using Fluent API (with explicit join table):**
```csharp
protected override void OnModelCreating(ModelBuilder modelBuilder)
{
    modelBuilder.Entity<Student>()
        .HasMany(s => s.Courses)
        .WithMany(c => c.Students)
        .UsingEntity(j => j.ToTable("StudentCourses"));
}
```

### Self-Referencing Relationship:
```csharp
public class Category
{
    public int CategoryId { get; set; }
    public string Name { get; set; }
    public int? ParentCategoryId { get; set; }
    
    public Category ParentCategory { get; set; }
    public ICollection<Category> SubCategories { get; set; }
}
```

---

## 4. What is migration in EF Core and how do you apply it?

**Answer:**

Migrations are used to incrementally update the database schema as your data model evolves, while preserving existing data.

### Creating a Migration:

```bash
# Create a new migration
dotnet ef migrations add AddUserTable

# Create a migration with specific name
dotnet ef migrations add [MigrationName]
```

### Generated Migration File:
```csharp
public partial class AddUserTable : Migration
{
    protected override void Up(MigrationBuilder migrationBuilder)
    {
        migrationBuilder.CreateTable(
            name: "Users",
            columns: table => new
            {
                UserId = table.Column<int>(type: "int", nullable: false)
                    .Annotation("SqlServer:Identity", "1, 1"),
                Name = table.Column<string>(type: "nvarchar(max)", nullable: true),
                Email = table.Column<string>(type: "nvarchar(max)", nullable: true)
            },
            constraints: table =>
            {
                table.PrimaryKey("PK_Users", x => x.UserId);
            });
    }

    protected override void Down(MigrationBuilder migrationBuilder)
    {
        migrationBuilder.DropTable(name: "Users");
    }
}
```

### Applying Migrations:

```bash
# Apply all pending migrations
dotnet ef database update

# Apply to specific migration
dotnet ef database update [MigrationName]

# Rollback to previous migration
dotnet ef database update [PreviousMigrationName]

# Remove last migration (if not applied)
dotnet ef migrations remove
```

### Using Package Manager Console (Visual Studio):

```powershell
# Create migration
Add-Migration [MigrationName]

# Apply migration
Update-Database

# Rollback
Update-Database -Migration [PreviousMigrationName]

# Remove migration
Remove-Migration
```

### Viewing Migrations:
```bash
# List all migrations
dotnet ef migrations list
```

---

## 5. How do you perform eager, lazy, and explicit loading in EF Core?

**Answer:**

These are different strategies to load related data.

### Eager Loading
Related data is loaded as part of the initial query using `Include()` and `ThenInclude()`.

```csharp
// Load blog with all posts
var blogs = context.Blogs
    .Include(b => b.Posts)
    .ToList();

// Load with nested includes
var blogs = context.Blogs
    .Include(b => b.Posts)
        .ThenInclude(p => p.Comments)
    .ToList();

// Multiple includes
var blogs = context.Blogs
    .Include(b => b.Posts)
    .Include(b => b.Author)
    .ToList();
```

**Advantages:**
- Single database query (with JOINs)
- No N+1 problem
- Best performance

**Disadvantages:**
- Loads all related data even if not needed
- Can result in large result sets

### Lazy Loading
Related data is automatically loaded when the navigation property is accessed.

**Prerequisites:**
- Navigation properties must be virtual
- Install: `Microsoft.EntityFrameworkCore.Proxies`

```csharp
builder.Services.AddDbContext<BlogContext>(options =>
    options.UseLazyLoadingProxies()
           .UseSqlServer("connection string"));

// Usage
var blog = context.Blogs.FirstOrDefault();

// Posts loaded here when accessed
var posts = blog.Posts; // Lazy loaded
```

**Advantages:**
- Loads data only when needed
- Clean code

**Disadvantages:**
- Multiple database queries
- Performance issues (N+1 problem)
- Hard to debug

### Explicit Loading
Related data is loaded explicitly at a later point using `Load()` or `LoadAsync()`.

```csharp
var blog = context.Blogs.FirstOrDefault();

// Explicitly load related data
context.Entry(blog).Collection(b => b.Posts).Load();

// For single reference
context.Entry(blog).Reference(b => b.Author).Load();

// Async version
await context.Entry(blog).Collection(b => b.Posts).LoadAsync();
```

**Advantages:**
- Full control over when data is loaded
- Flexible

**Disadvantages:**
- More code
- Still prone to N+1 problem if in loop

### Comparison:

| Method | Performance | Ease of Use | When to Use |
|--------|-------------|------------|-----------|
| **Eager** | Best | Medium | Default choice |
| **Lazy** | Worst | Easy | Avoid in production |
| **Explicit** | Medium | Complex | When you need control |

---

## 6. What is the purpose of the DbContext class?

**Answer:**

The `DbContext` class is the central component in EF Core that represents a session with the database.

### Key Responsibilities:

1. **Database Connection Management**
   - Manages connection to the database
   - Handles connection pooling
   - Manages transaction scope

2. **Query Execution**
   - Translates LINQ queries to SQL
   - Materializes results into entity objects
   - Manages query results

3. **Change Tracking**
   - Tracks added, modified, deleted entities
   - Maintains entity state
   - Detects changes automatically

4. **Saving Data**
   - Persists changes via `SaveChanges()` or `SaveChangesAsync()`
   - Generates INSERT, UPDATE, DELETE commands
   - Manages transactions

### Example DbContext:

```csharp
public class SchoolContext : DbContext
{
    public SchoolContext(DbContextOptions<SchoolContext> options) 
        : base(options)
    {
    }

    // DbSet represents tables
    public DbSet<Student> Students { get; set; }
    public DbSet<Course> Courses { get; set; }
    public DbSet<Grade> Grades { get; set; }

    protected override void OnModelCreating(ModelBuilder modelBuilder)
    {
        // Configure relationships, constraints, etc.
        modelBuilder.Entity<Student>()
            .HasMany(s => s.Courses)
            .WithMany(c => c.Students)
            .UsingEntity<Enrollment>();
    }
}
```

### Usage:

```csharp
public class StudentService
{
    private readonly SchoolContext _context;

    public StudentService(SchoolContext context)
    {
        _context = context;
    }

    public async Task<Student> GetStudentAsync(int id)
    {
        // Query
        var student = await _context.Students
            .Include(s => s.Courses)
            .FirstOrDefaultAsync(s => s.Id == id);
        
        return student;
    }

    public async Task AddStudentAsync(Student student)
    {
        // Add
        _context.Students.Add(student);
        
        // Save
        await _context.SaveChangesAsync();
    }

    public async Task UpdateStudentAsync(Student student)
    {
        // Update (tracking automatically detects changes)
        _context.Students.Update(student);
        await _context.SaveChangesAsync();
    }

    public async Task DeleteStudentAsync(int id)
    {
        var student = await _context.Students.FindAsync(id);
        if (student != null)
        {
            // Delete
            _context.Students.Remove(student);
            await _context.SaveChangesAsync();
        }
    }
}
```

---

## 7. How do you handle concurrency conflicts in EF Core?

**Answer:**

EF Core uses Optimistic Concurrency, assuming conflicts are rare and checking if data has changed only when `SaveChanges()` is called.

### Implementation:

**Step 1: Add Concurrency Token:**
```csharp
public class Product
{
    public int ProductId { get; set; }
    public string Name { get; set; }
    public decimal Price { get; set; }
    
    // Concurrency token (timestamp)
    [Timestamp]
    public byte[] RowVersion { get; set; }
}
```

**Or using Fluent API:**
```csharp
protected override void OnModelCreating(ModelBuilder modelBuilder)
{
    modelBuilder.Entity<Product>()
        .Property(p => p.RowVersion)
        .IsRowVersion();
}
```

**Step 2: Handle Concurrency Exception:**
```csharp
public async Task UpdateProductAsync(Product product)
{
    try
    {
        await _context.SaveChangesAsync();
    }
    catch (DbUpdateConcurrencyException ex)
    {
        // Get current database values
        var entry = ex.Entries.Single();
        var databaseValues = await entry.GetDatabaseValuesAsync();
        
        if (databaseValues == null)
        {
            // Entity was deleted by another user
            throw new InvalidOperationException("Entity was deleted");
        }
        
        // Option 1: Take database values
        entry.OriginalValues.SetValues(databaseValues);
        
        // Option 2: Keep your changes
        entry.Values.SetValues(databaseValues);
        // Then reapply your changes...
        
        // Retry
        await _context.SaveChangesAsync();
    }
}
```

### Concurrency Conflict Resolution Strategies:

**1. Last-One-Wins:**
```csharp
entry.OriginalValues.SetValues(databaseValues);
// Your changes overwrite
await _context.SaveChangesAsync();
```

**2. Database Wins:**
```csharp
entry.Reload(); // Discard changes
// Or use databaseValues
```

**3. Merge Strategy:**
```csharp
// Get current values from database
var dbValues = databaseValues.ToObject() as Product;

// Merge logic (custom)
product.Name = dbValues.Name;
// Keep your price change...
product.Price = product.Price;

// Retry
await _context.SaveChangesAsync();
```

---

## 8. How do you write LINQ queries with EF Core?

**Answer:**

EF Core allows you to write LINQ queries using C# syntax, which are translated to SQL.

### Common LINQ Operations:

**Basic Queries:**
```csharp
// Get all
var allBlogs = context.Blogs.ToList();

// Get first
var first = context.Blogs.FirstOrDefault();

// Get by id
var blog = context.Blogs.Find(1);

// Get with condition
var activeBlogs = context.Blogs
    .Where(b => b.IsActive)
    .ToList();

// Order by
var sorted = context.Blogs
    .OrderBy(b => b.Title)
    .ToList();

// Descending order
var sorted = context.Blogs
    .OrderByDescending(b => b.CreatedDate)
    .ToList();

// Select (projection)
var titles = context.Blogs
    .Select(b => b.Title)
    .ToList();

// Count
var count = context.Blogs.Count();

// Sum
var total = context.Posts.Sum(p => p.Views);

// Average
var avgRating = context.Posts.Average(p => p.Rating);

// Any (exists)
bool hasActive = context.Blogs.Any(b => b.IsActive);

// All
bool allPositive = context.Posts.All(p => p.Rating > 0);
```

**Advanced Queries:**
```csharp
// Join
var result = context.Blogs
    .Join(context.Posts,
        b => b.BlogId,
        p => p.BlogId,
        (b, p) => new { Blog = b, Post = p })
    .ToList();

// Group by
var grouped = context.Posts
    .GroupBy(p => p.BlogId)
    .Select(g => new 
    { 
        BlogId = g.Key, 
        Count = g.Count(),
        AvgRating = g.Average(p => p.Rating)
    })
    .ToList();

// Distinct
var uniqueAuthors = context.Posts
    .Select(p => p.Author)
    .Distinct()
    .ToList();

// Skip and Take (pagination)
var page2 = context.Blogs
    .OrderBy(b => b.Title)
    .Skip(10)
    .Take(10)
    .ToList();

// Where with multiple conditions
var results = context.Posts
    .Where(p => p.Title.Contains("Core") && p.Rating > 3)
    .ToList();
```

**Async Queries:**
```csharp
// Async versions
var blogs = await context.Blogs.ToListAsync();
var first = await context.Blogs.FirstOrDefaultAsync();
var count = await context.Blogs.CountAsync();
var exists = await context.Blogs.AnyAsync(b => b.IsActive);
```

### Best Practices:
- Use `.ToList()