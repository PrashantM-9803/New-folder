# ASP.NET Core Interview Questions

## Table of Contents

- [ASP.NET Core Fundamentals](#aspnet-core-fundamentals)
- [ASP.NET Core Web API](#aspnet-core-web-api)
- [Entity Framework Core](#entity-framework-core)
- [.NET Security](#net-security)
- [Docker](#docker)
- [Azure](#azure)

---

## ASP.NET Core Fundamentals

### 1. What is ASP.NET Core and how is it different from ASP.NET MVC?

**Answer:**

ASP.NET MVC and ASP.NET Core are both frameworks for building web applications, but they differ significantly in architecture, platform support, and performance.

**ASP.NET MVC** is the older, Windows-only framework built on the .NET Framework. It follows the Model-View-Controller pattern, typically using Razor Views (.cshtml), Controllers, and Models. Applications are hosted on IIS and are limited to Windows servers. It's still used in many enterprise systems but is considered legacy for new development due to its lack of cross-platform support and slower performance compared to Core.

**ASP.NET Core**, on the other hand, is a modern, open-source, cross-platform framework built on .NET Core / .NET 5+. It supports MVC, Razor Pages, Blazor, and Web API in a single unified framework. It runs on Windows, Linux, and macOS, can be hosted on Kestrel, IIS, or Nginx, and is optimized for high performance and cloud readiness. It also has built-in dependency injection and modular architecture, making it ideal for modern APIs, microservices, and cloud-native applications.

**Example: ASP.NET MVC Controller**

```csharp
public class HomeController : Controller
{
    public ActionResult Index()
    {
        return View();
    }
}
```

**Example: ASP.NET Core Minimal Setup**

```csharp
var builder = WebApplication.CreateBuilder(args);
builder.Services.AddControllersWithViews();
var app = builder.Build();
app.MapDefaultControllerRoute();
app.Run();
```

---

### 2. Explain the ASP.NET Core request processing pipeline.

**Answer:**

The ASP.NET Core request processing pipeline is a series of middleware components that handle incoming HTTP requests and responses in an ASP.NET Core Web application. Each middleware component is responsible for a specific task, such as authentication, routing, logging, caching, encryption and decryption, response generation, etc. The pipeline is configured in the Program class of an ASP.NET Core application.

**Pipeline Flow:**

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

**USER (OR) BROWSER → MIDDLEWARE → ROUTING → CONTROLLER → USER (RESPONSE)**

**Key Middleware Components:**

- **Request**: Incoming HTTP request enters the pipeline
- **ExceptionHandler**: Handles exceptions globally
- **HSTS**: Enforces HTTP Strict Transport Security
- **HttpsRedirection**: Redirects HTTP requests to HTTPS
- **Static Files**: Serves static files directly
- **Routing**: Maps requests to routes
- **CORS**: Handles cross-origin requests
- **Authentication**: Validates user credentials
- **Authorization**: Checks user permissions
- **Custom Middlewares**: Custom operations
- **Endpoint**: Target controller action or Razor Page
- **Response**: Processed response sent back to client

**Example:**

```csharp
// Program.cs (.NET 6/7/8 minimal hosting)
using Microsoft.AspNetCore.Builder;
using Microsoft.AspNetCore.Http;
using Microsoft.Extensions.DependencyInjection;
using Microsoft.Extensions.Hosting;

var builder = WebApplication.CreateBuilder(args);

// 1) Register services (available via DI)
builder.Services.AddControllers();
builder.Services.AddEndpointsApiExplorer();
builder.Services.AddAuthorization();

// 2) Build the app
var app = builder.Build();

// 3) Middleware pipeline (ORDER MATTERS)

// Global exception handler
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

// Map endpoints
app.MapGet("/hello", () => Results.Ok(new { message = "Hello from Minimal API!" }));
app.MapControllers();
app.MapGet("/error", () => Results.Problem("Something went wrong"))
   .ExcludeFromDescription();

app.Run();
```

**Controller Example:**

```csharp
// Controllers/GreetController.cs
using Microsoft.AspNetCore.Mvc;

[ApiController]
[Route("api/[controller]")]
public class GreetController : ControllerBase
{
    [HttpGet("{name}")]
    public IActionResult Get(string name)
    {
        return Ok(new { message = $"Hello, {name} from Controller!" });
    }
}
```

---

### 3. What is middleware? Give examples.

**Answer:**

ASP.NET Core Middleware Components are the basic building blocks of the Request Processing Pipeline in ASP.NET Core Applications. The Request Processing Pipeline determines how HTTP Requests and Responses will be processed in the ASP.NET Core Application.

**Key Points:**

- Middleware components are executed in the order they are added
- Each middleware can pass requests to the next component
- Can perform tasks before and after the next component is invoked
- Should have a single responsibility

**Example Configuration:**

```csharp
// Program.cs
var builder = WebApplication.CreateBuilder(args);
var app = builder.Build();

// Middleware pipeline
app.UseHttpsRedirection();  // Redirect HTTP to HTTPS
app.UseStaticFiles();       // Serve static files
app.UseRouting();           // Enable routing
app.UseAuthentication();    // Authenticate users
app.UseAuthorization();     // Authorize access

// Map endpoints
app.MapControllers();
app.Run();
```

**Custom Middleware Example:**

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

// Register custom middleware
app.UseMiddleware<CustomMiddleware>();
```

**Built-in Middleware Components:**

- **Static Files**: Serves static files from the wwwroot folder
- **Routing**: Matches requests to endpoints
- **Authentication and Authorization**: Validates user credentials and permissions
- **CORS**: Handles cross-origin requests
- **Exception Handling**: Provides global error handling

---

### 4. How does dependency injection work in ASP.NET Core?

**Answer:**

ASP.NET Core Dependency Injection (DI) is a powerful feature that helps manage object dependencies within our application. DI is a design pattern used to achieve loose coupling in software development. Instead of creating dependencies directly within a class, dependencies are injected into the class from outside, typically through constructor parameters.

**Example Model:**

```csharp
namespace FirstCoreMVCWebApplication.Models
{
    public class Student
    {
        public int StudentId { get; set; }
        public string? Name { get; set; }
        public string? Branch { get; set; }
        public string? Section { get; set; }
        public string? Gender { get; set; }
    }
}
```

**Service Interface:**

```csharp
using System.Collections.Generic;

namespace FirstCoreMVCWebApplication.Models
{
    public interface IStudentRepository
    {
        Student GetStudentById(int StudentId);
        List<Student> GetAllStudent();
    }
}
```

**Service Implementation:**

```csharp
using System.Collections.Generic;
using System.Linq;

namespace FirstCoreMVCWebApplication.Models
{
    public class StudentRepository : IStudentRepository
    {
        public List<Student> DataSource()
        {
            return new List<Student>()
            {
                new Student() { StudentId = 101, Name = "James", Branch = "CSE", Section = "A", Gender = "Male" },
                new Student() { StudentId = 102, Name = "Smith", Branch = "ETC", Section = "B", Gender = "Male" },
                new Student() { StudentId = 103, Name = "David", Branch = "CSE", Section = "A", Gender = "Male" },
                new Student() { StudentId = 104, Name = "Sara", Branch = "CSE", Section = "A", Gender = "Female" },
                new Student() { StudentId = 105, Name = "Pam", Branch = "ETC", Section = "B", Gender = "Female" }
            };
        }

        public Student GetStudentById(int StudentId)
        {
            return DataSource().FirstOrDefault(e => e.StudentId == StudentId) ?? new Student();
        }

        public List<Student> GetAllStudent()
        {
            return DataSource();
        }
    }
}
```

**Program.cs Configuration:**

```csharp
namespace FirstCoreMVCWebApplication
{
    public class Program
    {
        public static void Main(string[] args)
        {
            var builder = WebApplication.CreateBuilder(args);

            builder.Services.AddMvc();

            var app = builder.Build();

            app.UseRouting();

            app.MapControllerRoute(
                name: "default",
                pattern: "{controller=Home}/{action=Index}/{id?}"
            );

            app.Run();
        }
    }
}
```

---

### 5. What are the different lifetimes for services in DI? (Transient, Scoped, Singleton)

**Answer:**

In .NET Dependency Injection, three main service lifetimes determine how long a service instance lives and how it is shared within an application.

**1. Transient**

- **Definition**: A new instance is created every time it is requested
- **Use Case**: Lightweight, stateless services (e.g., calculators, data formatters)
- **Behavior**: Different instances even within the same HTTP request
- **Registration**: `AddTransient()`

**2. Scoped**

- **Definition**: Single instance per scope (typically one HTTP request)
- **Use Case**: Services maintaining state during a request (e.g., DbContext)
- **Behavior**: Same instance within a request, new instance for each request
- **Registration**: `AddScoped()`

**3. Singleton**

- **Definition**: Single instance for the application's lifetime
- **Use Case**: Application-wide state, caching, logging
- **Behavior**: Shared across all users and requests
- **Registration**: `AddSingleton()`

---

### 6. What is the purpose of the Startup class?

**Answer:**

The Startup class in ASP.NET Core is the central entry point for configuring an application's behavior, services, and request pipeline. It is responsible for setting up dependency injection (ConfigureServices) and defining how the application handles HTTP requests via middleware (Configure).

**Two Important Methods:**

**1. ConfigureServices Method:**

- Configures services for the application
- Takes `IServiceCollection` as input
- Used for dependency injection setup

**2. Configure Method:**

- Configures the HTTP request processing pipeline
- Takes `IApplicationBuilder` and `IWebHostEnvironment` as parameters
- Used for middleware configuration

**Example:**

```csharp
using Microsoft.AspNetCore.Builder;
using Microsoft.AspNetCore.Hosting;
using Microsoft.Extensions.DependencyInjection;

namespace ConsoleToWebAPI
{
    public class Startup
    {
        public void ConfigureServices(IServiceCollection services)
        {
            // Register services here
        }

        public void Configure(IApplicationBuilder app, IWebHostEnvironment env)
        {
            app.UseRouting();

            app.UseEndpoints(endpoints =>
            {
                endpoints.MapGet("/", async context => {
                    await context.Response.WriteAsync("Hello From ASP.NET Core Web API");
                });
            });
        }
    }
}
```

---

### 7. How do you manage configuration and secrets in ASP.NET Core?

**Answer:**

Managing configuration and secrets in ASP.NET Core involves using the built-in `Microsoft.Extensions.Configuration` system and the ASP.NET Core Secret Manager for development.

**Configuration Management:**

1. **appsettings.json** and **appsettings.{Environment}.json**: Store general application settings
2. **Environment Variables**: Override settings for different environments
3. **Command-line Arguments**: Temporary adjustments during startup
4. **Default Values**: Programmatic defaults using in-memory objects

**Secrets Management:**

**Development:**

- **Secret Manager Tool**: Local storage outside project directory
- Keeps secrets out of source control

**Production:**

- **Azure Key Vault**: Cloud-based secure storage
- **AWS Secrets Manager**: For AWS environments
- **HashiCorp Vault**: For on-premises or multi-cloud
- **Environment Variables**: Less secure than dedicated stores

---

### 8. What is Kestrel?

**Answer:**

Microsoft developed the Kestrel web server as the default Cross-Platform Web Server for hosting ASP.NET Core applications on Windows, Linux, and MacOS. Kestrel is designed to be lightweight, fast, and efficient, making it suitable for both development and production environments.

**Process Names:**

**Self-Contained Deployment:**

- Includes .NET runtime
- Process name: Application executable name (e.g., `FirstCoreWebApplication.exe`)
- Larger deployment size but more portable

**Framework-Dependent Deployment:**

- Relies on installed .NET runtime
- Process name: `dotnet.exe` (Windows) or `dotnet` (Linux/macOS)
- Smaller deployment size but requires runtime installed

---

### 9. How do you enable logging in ASP.NET Core?

**Answer:**

Logging means recording what your application is doing while it is running. ASP.NET Core provides a built-in logging framework out of the box.

**Why Logging?**

- Find errors quickly
- Understand application behavior
- Debug issues in development
- Monitor performance in production

**1. Configure Log Levels in appsettings.json:**

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

**Log Levels** (increasing severity):

- Trace
- Debug
- Information
- Warning
- Error
- Critical
- None

**2. Configure Logging in Program.cs:**

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

**3. Inject and Use Logger:**

```csharp
public class HomeController : Controller
{
    private readonly ILogger<HomeController> _logger;

    public HomeController(ILogger<HomeController> logger)
    {
        _logger = logger;
    }

    public IActionResult Index()
    {
        _logger.LogInformation("Log message in the Index() method");
        return View();
    }
}
```

---

### 10. What is the difference between IApplicationBuilder and IServiceCollection?

**Answer:**

| Aspect            | IServiceCollection                                | IApplicationBuilder                           |
| ----------------- | ------------------------------------------------- | --------------------------------------------- |
| **Purpose**       | Registers services and their lifetimes            | Defines HTTP request pipeline                 |
| **When Used**     | During startup (ConfigureServices)                | During configuration (Configure)              |
| **Role**          | Configures and adds services                      | Builds middleware pipeline                    |
| **How Used**      | `services.AddScoped()`, `services.AddSingleton()` | `app.UseRouting()`, `app.UseAuthentication()` |
| **Who Uses It**   | Developers during setup                           | Application at runtime                        |
| **Configuration** | Service lifetime and dependencies                 | Middleware order and settings                 |

---

## ASP.NET Core Web API

### 1. How do you create a RESTful API in ASP.NET Core?

**Answer:**

Creating a RESTful API in ASP.NET Core involves using controllers with attribute routing to define endpoints and standard HTTP verbs (GET, POST, PUT, DELETE) to perform actions on data.

**Prerequisites:**

- .NET SDK installed
- Code editor like Visual Studio or VS Code

**Step-by-Step Guide:**

**1. Create a New Project:**

```bash
dotnet new webapi -n ApiProject
cd ApiProject
```

**2. Define a Model:**

```csharp
// Models/Product.cs
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

**3. Add a Controller:**

```csharp
// Controllers/ProductsController.cs
using Microsoft.AspNetCore.Mvc;
using ApiProject.Models;
using System.Collections.Generic;
using System.Linq;

namespace ApiProject.Controllers
{
    [Route("api/[controller]")]
    [ApiController]
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
            {
                return NotFound();
            }

            return Ok(product);
        }

        // POST: api/products
        [HttpPost]
        public ActionResult<Product> PostProduct(Product newProduct)
        {
            newProduct.Id = _products.Count > 0 ? _products.Max(p => p.Id) + 1 : 1;
            _products.Add(newProduct);

            return CreatedAtAction(nameof(GetProduct), new { id = newProduct.Id }, newProduct);
        }
    }
}
```

**4. Run the API:**

```bash
dotnet run
```

**Key Concepts:**

- **Attribute Routing**: Maps URLs and HTTP methods to controller actions
- **HTTP Verbs**: GET, POST, PUT, DELETE for CRUD operations
- **ApiController Attribute**: Enables automatic model validation
- **Status Codes**: Ok() (200), NotFound() (404), CreatedAtAction() (201)

---

### 2. What is the [ApiController] attribute and what does it do?

**Answer:**

The `[ApiController]` attribute marks a controller as an API controller and provides several default functionalities:

**Benefits:**

1. **Automatic Model Validation**: Returns HTTP 400 with ProblemDetails when `ModelState.IsValid == false`
2. **Automatic Parameter Binding**:
   - Complex types → Request Body (JSON)
   - Simple/primitive types → Route or Query String
   - Special cases: `[FromHeader]`, `[FromForm]`, etc.
3. **Attribute Routing Required**: Enforces use of attribute routing
4. **Problem Details**: Standardized error responses

---

### 3. How do you implement routing in Web API?

**Answer:**

ASP.NET Web API supports two types of routing:

**1. Convention-Based Routing:**

```csharp
// In WebApiConfig.cs (ASP.NET Web API)
public static class WebApiConfig
{
    public static void Register(HttpConfiguration config)
    {
        config.MapHttpAttributeRoutes();

        config.Routes.MapHttpRoute(
            name: "DefaultApi",
            routeTemplate: "api/{controller}/{id}",
            defaults: new { id = RouteParameter.Optional }
        );
    }
}
```

**2. Attribute Routing (Recommended for REST APIs):**

```csharp
// In Program.cs (ASP.NET Core)
var builder = WebApplication.CreateBuilder(args);
builder.Services.AddControllers();
var app = builder.Build();
app.MapControllers();
app.Run();
```

**Example Controller:**

```csharp
[ApiController]
[Route("api/[controller]")]
public class ProductsController : ControllerBase
{
    // GET api/products
    [HttpGet]
    public IActionResult GetAllProducts() { ... }

    // GET api/products/5
    [HttpGet("{id:int}")]
    public IActionResult GetProductById(int id) { ... }

    // POST api/products
    [HttpPost]
    public IActionResult CreateProduct([FromBody] Product product) { ... }

    // DELETE api/products/5
    [HttpDelete("{id:int}")]
    public IActionResult DeleteProduct(int id) { ... }
}
```

**Key Points:**

- `[Route("api/[controller]")]`: Base route template
- `[HttpGet]`, `[HttpPost]`, etc.: HTTP verb attributes
- `{id:int}`: Route parameters with constraints

---

### 4. What is model binding and model validation?

**Answer:**

**Model Binding:**

- Automatic process of extracting data from HTTP request
- Maps data to C# objects
- Sources: Form fields, route data, query strings, request body, headers

**Model Validation:**

- Checks if bound data conforms to business rules
- Uses Data Annotation attributes
- Common attributes: `[Required]`, `[StringLength]`, `[Range]`, `[EmailAddress]`, `[RegularExpression]`

**How They Work Together:**

1. **Request Arrives**: HTTP request with data
2. **Model Binding**: Maps request data to C# model
3. **Model Validation**: Checks validation attributes
4. **Action Execution**: Controller checks `ModelState.IsValid`
5. **Response**:
   - If valid: Business logic proceeds
   - If invalid: Returns 400 Bad Request with errors

---

### 5. How do you return different HTTP status codes from an API action?

**Answer:**

In ASP.NET Core Web API, you can use methods like `Created`, `CreatedAtAction`, or `CreatedAtRoute` to return different HTTP status codes.

**Common Status Code Methods:**

- **Ok()**: 200 OK
- **Created()**: 201 Created
- **CreatedAtAction()**: 201 Created with Location header
- **NoContent()**: 204 No Content
- **BadRequest()**: 400 Bad Request
- **NotFound()**: 404 Not Found
- **Unauthorized()**: 401 Unauthorized
- **Forbid()**: 403 Forbidden

**Example:**

```csharp
namespace ReturnTypeAndStatusCodes.Models
{
    public class Employee
    {
        public int Id { get; set; }
        public string Name { get; set; }
        public string Gender { get; set; }
        public int Age { get; set; }
        public string Department { get; set; }
    }
}
```

---

### 6. How do you secure a Web API (e.g., JWT, OAuth)?

**Answer:**

Web APIs are secured primarily using OAuth 2.0 for authorization and JSON Web Tokens (JWTs) for authentication.

**Key Security Methods:**

**OAuth 2.0:**

- Open standard for authorization
- Client obtains access token from authorization server
- Token presented to API with each request
- Recommended for modern applications

**JSON Web Token (JWT):**

- Compact, self-contained, digitally signed token
- Used for authentication and secure information exchange
- Server issues JWT after login
- Client includes token in Authorization header: `Authorization: Bearer <token>`
- API validates token internally without querying auth server

---

### 7. What is CORS and how do you configure it?

**Answer:**

CORS (Cross-Origin Resource Sharing) is a browser security mechanism that allows a web page running on one origin to access resources from a different origin.

**Configuration in ASP.NET Core (.NET 6+):**

**1. Add CORS service and define policy:**

```csharp
var builder = WebApplication.CreateBuilder(args);

const string MyAllowSpecificOrigins = "_myAllowSpecificOrigins";

builder.Services.AddCors(options =>
{
    options.AddPolicy(name: MyAllowSpecificOrigins,
        policy =>
        {
            policy.WithOrigins("http://localhost:3000", "https://www.contoso.com")
                  .AllowAnyHeader()
                  .AllowAnyMethod();
        });
});

builder.Services.AddControllers();
```

**2. Enable CORS middleware:**

```csharp
var app = builder.Build();

app.UseHttpsRedirection();
app.UseStaticFiles();
app.UseRouting();

app.UseCors(MyAllowSpecificOrigins);

app.UseAuthorization();
app.MapControllers();
app.Run();
```

**Important:**

- `WithOrigins()`: Specifies allowed client domains (no trailing slash)
- `AllowAnyHeader()` and `AllowAnyMethod()`: Permit all headers and methods
- Avoid `AllowAnyOrigin()` with `AllowCredentials()` (security risk)
- Place `UseCors()` after `UseRouting()` and before `UseAuthorization()`

---

### 8. How do you handle exceptions and errors globally?

**Answer:**

In modern .NET applications (.NET 8+), use the `IExceptionHandler` interface for centralized error handling.

**1. Create Handler Class:**

```csharp
using Microsoft.AspNetCore.Diagnostics;
using Microsoft.AspNetCore.Mvc;
using System.Net;

public sealed class GlobalExceptionHandler(ILogger<GlobalExceptionHandler> logger) : IExceptionHandler
{
    public async ValueTask<bool> TryHandleAsync(
        HttpContext httpContext, Exception exception, CancellationToken cancellationToken)
    {
        logger.LogError(exception, "An unhandled exception occurred: {Message}", exception.Message);

        var problemDetails = new ProblemDetails
        {
            Status = (int)HttpStatusCode.InternalServerError,
            Title = "An error occurred while processing your request.",
            Detail = exception.Message,
            Instance = httpContext.Request.Path
        };

        await httpContext.Response.WriteAsJsonAsync(problemDetails, cancellationToken);
        return true;
    }
}
```

**2. Register Handler:**

```csharp
var builder = WebApplication.CreateBuilder(args);

builder.Services.AddControllers();
builder.Services.AddExceptionHandler<GlobalExceptionHandler>();
builder.Services.AddProblemDetails();

var app = builder.Build();

app.UseExceptionHandler();
app.MapControllers();
app.Run();
```

---

### 9. How do you version a Web API?

**Answer:**

API versioning manages changes and maintains backward compatibility.

**Common Versioning Strategies:**

1. **URI Path Versioning**: `https://api.example.com/v1/products`
2. **Query Parameter Versioning**: `https://api.example.com/products?version=1`
3. **Custom Header Versioning**: `X-API-Version: 1`
4. **Content Negotiation**: `Accept: application/vnd.example.v1+json`

**Implementation:**
Use the `Microsoft.AspNetCore.Mvc.Versioning` NuGet package with attributes like `[ApiVersion(1.0)]` and `[Route("api/v{version:apiVersion}/controller")]`.

---

### 10. What are filters in ASP.NET Core?

**Answer:**

Filters allow custom logic to run before or after specific stages of the action invocation pipeline.

**Main Filter Types:**

1. **Authorization Filters**: Run first, determine user authorization, can short-circuit
2. **Resource Filters**: Wrap most of the pipeline, run after authorization but before model binding
3. **Action Filters**: Run before and after action method, access action arguments and result
4. **Exception Filters**: Run only if unhandled exception occurs, for global error handling
5. **Result Filters**: Run before and after action result execution, can modify final response

---

## Entity Framework Core

### 1. What is Entity Framework Core and how does it differ from classic Entity Framework?

**Answer:**

Entity Framework (EF) Core is a modern, lightweight, cross-platform Object-Relational Mapper (ORM) from Microsoft that enables .NET developers to work with databases using C# objects instead of writing SQL queries manually.

**Key Differences from EF6:**

| Feature              | EF Core                                          | EF6                                |
| -------------------- | ------------------------------------------------ | ---------------------------------- |
| **Platform Support** | Cross-platform (Windows, Linux, macOS)           | Windows-only (.NET Framework)      |
| **Architecture**     | Modular, extensible, better performance          | Monolithic, mature                 |
| **Development**      | Actively developed, recommended for new projects | Stable, supported, no new features |

---

### 2. Explain the difference between Code First and Database First approaches.

**Answer:**

| Approach           | Description                                                                                         | When to Use                                                                         |
| ------------------ | --------------------------------------------------------------------------------------------------- | ----------------------------------------------------------------------------------- |
| **Code First**     | Define data model using C# classes, EF generates database schema and manages migrations             | Greenfield projects, domain-driven design, agile environments with evolving schemas |
| **Database First** | Start with existing database schema, EF tools reverse-engineer to generate C# classes and DbContext | Legacy systems, enterprise projects with DBAs, established database design          |

**Code First Workflow:**

1. Define model classes and relationships in code
2. Use migrations to create/update database schema
3. Database schema derived from code

**Database First Workflow:**

1. Pre-existing database in place
2. Use EF Core scaffold command to reverse-engineer
3. Code derived from database

**Summary:**

| Feature                | Code First                             | Database First                                   |
| ---------------------- | -------------------------------------- | ------------------------------------------------ |
| **Source of Truth**    | Application Code (Classes)             | Existing Database Schema                         |
| **Schema Management**  | Handled via migrations within codebase | Managed externally (e.g., by a DBA)              |
| **Generation Process** | Database generated from code           | Code generated from database                     |
| **Primary Use Case**   | New application development            | Integrating with existing databases              |
| **Developer Focus**    | Primarily on code and OOP principles   | Primarily on database design and existing schema |

---

### 3. How do you configure relationships (one-to-many, many-to-many) in EF Core?

**Answer:**

Relationships can be configured using Data Annotations or Fluent API.

**One-to-Many (Fluent API):**

```csharp
modelBuilder.Entity<Student>()
    .HasOne<Grade>(s => s.Grade)
    .WithMany(g => g.Students)
    .HasForeignKey(s => s.CurrentGradeId);
```

**Many-to-Many (EF Core 5.0+):**

```csharp
modelBuilder.Entity<Book>()
    .HasMany(b => b.Authors)
    .WithMany(a => a.Books)
    .UsingEntity(j => j.ToTable("BookAuthors"));
```

---

### 4. What is migration in EF Core and how do you apply it?

**Answer:**

Migrations provide a way to incrementally update the database schema as your data model evolves while preserving existing data.

**Using .NET CLI:**

**Add a new migration:**

```bash
dotnet ef migrations add [MigrationName]
```

**Apply migration to database:**

```bash
dotnet ef database update
```

**Using Package Manager Console:**

```powershell
Add-Migration [MigrationName]
Update-Database
```

---

### 5. How do you perform eager, lazy, and explicit loading in EF Core?

**Answer:**

**Eager Loading:**
Related data loaded with initial query using `Include` or `ThenInclude`:

```csharp
var blogs = context.Blogs.Include(b => b.Posts).ToList();
```

**Lazy Loading:**
Related data automatically loaded when navigation property accessed (requires `Microsoft.EntityFrameworkCore.Proxies` and virtual properties):

```csharp
var blog = context.Blogs.FirstOrDefault();
var posts = blog.Posts; // Posts loaded here
```

**Explicit Loading:**
Related data explicitly loaded later using `Entry(...).Reference(...).Load()` or `Entry(...).Collection(...).Load()`:

```csharp
var blog = context.Blogs.FirstOrDefault();
// ... later ...
context.Entry(blog).Collection(b => b.Posts).Load();
```

---

### 6. What is the purpose of the DbContext class?

**Answer:**

The `DbContext` class is the central component in EF Core that represents a session with the database.

**Key Responsibilities:**

1. **Database Connection Management**: Manages connection to database via configured provider
2. **Query Execution**: Translates LINQ queries into SQL and materializes results
3. **Change Tracking**: Automatically tracks entity changes (added, modified, deleted)
4. **Saving Data**: Persists changes using `SaveChanges()` or `SaveChangesAsync()`, generating INSERT, UPDATE, and DELETE commands

---

### 7. How do you handle concurrency conflicts in EF Core?

**Answer:**

EF Core handles concurrency conflicts using Optimistic Concurrency.

**Implementation:**

1. Add a concurrency token property to entity (e.g., `byte[]` with `[ConcurrencyCheck]` attribute or Fluent API's `IsConcurrencyToken()`)
2. When conflict occurs, `DbUpdateConcurrencyException` is thrown
3. Developer catches exception, retrieves current database values, merges changes, and retries

---

### 8. How do you write LINQ queries with EF Core?

**Answer:**

EF Core allows writing LINQ queries using C# syntax, which are translated into SQL.

**Example:**

```csharp
using (var db = new BloggingContext())
{
    var blogs = db.Blogs
        .Where(b => b.Rating > 3)
        .OrderBy(b => b.Url)
        .ToList(); // Executes query and returns results
}
```

Use extension methods like `Where`, `OrderBy`, `Select`, `Include` on `DbSet` properties.

---

## .NET Security

### 1. What is authentication vs. authorization in .NET?

**Answer:**

**Authentication**: Process of verifying user identity ("Who are you?")

- Handled by `IAuthenticationService` and middleware
- Common methods: username/password, MFA, certificates

**Authorization**: Process of determining what authenticated user can do ("What can you access?")

- Occurs after authentication
- Uses roles, claims, or policies
- Enforced using `[Authorize]` attribute

**Analogy**: At an airport, showing passport to enter building is authentication; showing boarding pass to access specific gate is authorization.

---

### 2. How do you implement JWT authentication in ASP.NET Core?

**Answer:**

Implementing JWT authentication involves configuring services and creating token generation logic.

**Steps:**

**1. Install Package:**

```bash
Install-Package Microsoft.AspNetCore.Authentication.JwtBearer
```

**2. Configure Settings in appsettings.json:**

```json
{
  "Jwt": {
    "Key": "YourSuperSecretKey",
    "Issuer": "YourIssuer",
    "Audience": "YourAudience"
  }
}
```

**3. Configure JWT Authentication in Program.cs:**
Configure authentication services with JWT Bearer scheme, including token validation parameters.

**4. Implement Token Generation:**
Create logic to generate JWT after successful authentication with claims, signing key, and expiration time.

**5. Secure Endpoints:**
Apply `[Authorize]` attribute to controllers or actions requiring valid JWT.

**6. Test:**
Use Postman to obtain JWT from login endpoint, then use token in `Authorization: Bearer <token>` header for protected endpoints.

---

### 3. What are the best practices for storing passwords?

**Answer:**

**Best Practices:**

1. **Never store plaintext passwords** or reversible formats
2. **Use strong hash functions**: Argon2id, bcrypt, or scrypt
3. **Use unique random salt** for each password before hashing
4. **Use work factor/iteration count** to make hashing computationally intensive
5. **Enforce multi-factor authentication (MFA)** for added security

---

### 4. How do you protect sensitive configuration data in .NET applications?

**Answer:**

**Development:**

- Use .NET Secret Manager tool to keep secrets out of project files

**Production:**

- Use Azure Key Vault or similar services
- Applications access secrets at runtime using managed identities

**Environment Variables:**

- External to source code but less robust than dedicated secrets management

---

### 5. What is role-based authorization and how is it implemented?

**Answer:**

Role-based authorization (RBAC) limits access based on user roles like Admin or Editor.

**Example:**

```csharp
[ApiController]
[Route("api/[controller]")]
public class AdminController : ControllerBase
{
    // Only Admin can access
    [HttpGet("sensitive-data")]
    [Authorize(Roles = "Admin")]
    public IActionResult GetSensitiveData()
    {
        return Ok("This is sensitive admin data.");
    }

    // Admins or Editors can access
    [HttpGet("shared-data")]
    [Authorize(Roles = "Admin,Editor")]
    public IActionResult GetSharedData()
    {
        return Ok("This data is for Admins and Editors.");
    }
}
```

Policy-based authorization can also be configured in Program.cs for complex rules based on roles or claims.

---

### 6. How do you prevent Cross-Site Scripting (XSS) and Cross-Site Request Forgery (CSRF)?

**Answer:**

**XSS Prevention:**

- ASP.NET Core automatically HTML-encodes output from Razor views
- Always validate and sanitize user input on server side

**CSRF Prevention:**

- Use anti-forgery tokens
- For MVC/Razor Pages: `@Html.AntiForgeryToken()` in views and `[ValidateAntiForgeryToken]` on actions
- For APIs: Anti-forgery middleware/attributes available
- Use same-site cookies
- Rely on JWT instead of cookie-based auth for third-party API clients

---

### 7. What is Data Protection API in .NET Core?

**Answer:**

The Data Protection API is a simple, secure, and extensible cryptographic system for protecting sensitive data like cookies, tokens, and API keys.

**Key Features:**

- **Simplified API**: `Protect()` and `Unprotect()` methods
- **Automatic Key Management**: Handles key generation, storage, and rotation
- **Built-in Integrations**: Used automatically for cookie authentication and Identity tokens
- **Isolation**: Uses "purpose strings" for distinct data protectors
- **Configurable Storage**: File system, Windows Registry, Azure Key Vault, Azure Blob, SQL databases
- **Security**: AES-256-CBC for confidentiality, HMACSHA256 for authenticity

**Example:**

```csharp
public class MySensitiveService
{
    private readonly IDataProtector _protector;

    public MySensitiveService(IDataProtectionProvider dataProtectionProvider)
    {
        _protector = dataProtectionProvider.CreateProtector("MySuperSecretPurpose");
    }

    public string EncryptData(string plainTextData)
    {
        return _protector.Protect(plainTextData);
    }

    public string DecryptData(string protectedData)
    {
        return _protector.Unprotect(protectedData);
    }
}
```

---

### 8. How do you enforce HTTPS in an ASP.NET Core application?

**Answer:**

**Steps:**

**1. Require HTTPS Redirection:**

```csharp
var builder = WebApplication.CreateBuilder(args);
builder.Services.AddControllersWithViews();
var app = builder.Build();

app.UseHttpsRedirection();

app.MapControllerRoute(
    name: "default",
    pattern: "{controller=Home}/{action=Index}/{id?}");

app.Run();
```

**2. Enforce HSTS (Production Only):**

```csharp
if (!app.Environment.IsDevelopment())
{
    app.UseHsts();
}

app.UseHttpsRedirection();
```

---

## Docker

### 1. What is Docker, and how is it different from a virtual machine?

**Answer:**

Docker is an open-source platform that uses OS-level virtualization (containerization) to package applications and dependencies into standardized, isolated units called containers.

**Key Differences:**

| Feature            | Docker Containers       | Virtual Machines                      |
| ------------------ | ----------------------- | ------------------------------------- |
| **Architecture**   | Share host OS kernel    | Run complete guest OS with own kernel |
| **Resource Usage** | Lightweight, efficient  | Heavy, more resource-intensive        |
| **Startup Time**   | Fast (seconds)          | Slower (minutes)                      |
| **Isolation**      | Process-level isolation | Hardware-level isolation              |

**Example:**

- **VM**: Running Windows VM on Linux host for Windows-only application
- **Docker**: Running multiple microservices (Python API, Node.js frontend, PostgreSQL) as separate containers on single Linux host, sharing same kernel

---

### 2. Explain the difference between Docker image and Docker container.

**Answer:**

**Docker Image:**

- Read-only template/blueprint
- Contains instructions, code, runtimes, libraries, dependencies
- Built from Dockerfile
- Immutable
- Like a recipe for a cake

**Docker Container:**

- Runnable instance of Docker image
- Thin writable layer added on top of image's read-only layers
- Runtime changes isolated to specific container
- Like the actual cake baked from recipe

**Example:**

- **Image**: `nginx` image (template for NGINX web server)
- **Container**: Running NGINX web server instance serving a website

---

### 3. What is Docker Hub?

**Answer:**

Docker Hub is a cloud-based registry service for finding, storing, and sharing Docker images publicly or privately.

**Features:**

- Default marketplace for Docker images
- Automated builds from source repositories (GitHub)
- Security scanning

**Example:**

```bash
# Pull official Python image
docker pull python

# Push custom image
docker push yourusername/yourimage
```

---

### 4. What is the role of the Dockerfile?

**Answer:**

A Dockerfile is a plain text file containing sequential command-line instructions Docker uses to automatically build a new Docker image. Each instruction creates a new read-only layer.

**Example:**

```dockerfile
# Use official Python runtime as parent image
FROM python:3.9-slim

# Set working directory
WORKDIR /app

# Copy contents into container
COPY . /app

# Install packages
RUN pip install --no-cache-dir -r requirements.txt

# Make port 80 available
EXPOSE 80

# Run app.py when container launches
CMD ["python", "app.py"]
```

---

### 5. Difference between docker run and docker start.

**Answer:**

**docker run:**

- Creates and starts a new container from specified image
- Combines `docker create` and `docker start`
- Used for initial deployment
- Allows setting configuration options (port mapping, environment variables)

**docker start:**

- Restarts an already stopped container
- Does not create new container
- Resumes existing container with previous state

**Example:**

```bash
# Create and start new container
docker run -d --name my_container nginx

# Stop container
docker stop my_container

# Restart same container
docker start my_container
```

---

### 6. How do you persist data in Docker containers?

**Answer:**

**1. Docker Volumes (Recommended):**

- Managed by Docker
- Stored in `/var/lib/docker/volumes/` on Linux
- Independent of container lifecycle
- Easy to back up and share

**Create and use volume:**

```bash
docker volume create my-volume
docker run -d --name my-container -v my-volume:/path/inside/container my-image
```

**2. Bind Mounts:**

- Link specific host directory to container path
- Changes visible in both host and container
- Less portable, potential security risk

**Example:**

```bash
docker run -d --name my-container -v /host/path:/container/path my-image
```

**3. tmpfs Mounts (Temporary):**

- Store data in host memory (RAM)
- Data lost when container stops
- Useful for sensitive data or performance
- Linux hosts only

**Example:**

```bash
docker run -d --name my-container --tmpfs /path/inside/container my-image
```

---

### 7. What are Docker volumes and bind mounts?

**Answer:**

**Docker Volumes:**

- Managed by Docker
- Stored in specific area of host filesystem
- Preferred method for most use cases (especially databases)
- Easy to back up and share

**Example:**

```bash
docker run -d --name db_container -v db_data:/var/lib/postgresql/data postgres
```

**Bind Mounts:**

- Link specific host file/directory to container
- Useful for development (live-coding)
- Exact mount point structure matters

**Example:**

```bash
docker run -d --name dev_app -v /path/on/host/app:/app my_app_image
```

---

### 8. Explain Docker networking modes.

**Answer:**

**Bridge (Default):**

- Creates private internal network on host
- Containers communicate with each other and outside via NAT
- **Use Case**: Single-host development, isolated container communication

**Host:**

- Removes network isolation
- Container shares host's network stack directly
- Better performance, risk of port conflicts
- **Use Case**: Maximum network performance

**None:**

- Disables all networking
- Only loopback interface
- **Use Case**: Security-sensitive containers, batch processing

**Overlay:**

- Enables communication between containers on different Docker hosts
- **Use Case**: Docker Swarm, Kubernetes, multi-host orchestration

**Example:**

```bash
docker run -d --name c1 --network bridge nginx
docker run -d --name c2 --network host nginx
docker run -d --name c3 --network none nginx
```

---

### 9. How do you optimize Docker image size?

**Answer:**

**Best Practices:**

1. **Use minimal base images**: Alpine or slim versions (e.g., `python:3.9-alpine`)
2. **Use multi-stage builds**: Build in one stage, run in smaller stage
3. **Minimize layers**: Combine multiple `RUN` commands using `&&`

**Example (Multi-stage build):**

```dockerfile
# Stage 1: Build
FROM node:18-alpine AS build
WORKDIR /app
COPY package*.json ./
RUN npm install
COPY . .
RUN npm run build

# Stage 2: Run
FROM nginx:alpine
COPY --from=build /app/build /usr/share/nginx/html
EXPOSE 80
CMD ["nginx", "-g", "daemon off;"]
```

---

### 10. What is the difference between COPY and ADD in Dockerfile?

**Answer:**

**COPY:**

- Simple, transparent file/directory copy from host to container
- Preferred command

**ADD:**

- Additional features:
  - Automatically extracts compressed archives (tar, gzip, bzip2)
  - Can fetch files from remote URLs

**Example:**

```dockerfile
# Simple file copy
COPY requirements.txt /app/requirements.txt

# Download and extract archive
ADD http://example.com/myapp.tar.gz /app/
```

---

### 11. How do you secure Docker containers in production?

**Answer:**

**Best Practices:**

1. **Principle of Least Privilege**: Run containers as non-root users
2. **Scan images for vulnerabilities**: Integrate security scanning in CI/CD, use trusted base images
3. **Minimize attack surface**: Use minimal base images, install only necessary packages
4. **Manage secrets securely**: Use Docker Swarm secrets or Kubernetes secrets, not environment variables
5. **Resource limits**: Use cgroups to limit CPU/memory usage

---

### 12. What is Docker Compose, and when would you use it?

**Answer:**

Docker Compose is a tool for defining and running multi-container Docker applications using a single YAML file.

**When to Use:**

- Development and testing environments
- Spin up entire application stack with single command
- Single-host production deployments

**Example (docker-compose.yml):**

```yaml
services:
  web:
    image: nginx:alpine
    ports:
      - "8080:80"
    volumes:
      - ./html:/usr/share/nginx/html
  db:
    image: postgres
    environment:
      POSTGRES_PASSWORD: mysecretpassword
    volumes:
      - db_data:/var/lib/postgresql/data

volumes:
  db_data:
```

---

### 13. Explain the concept of multi-stage builds in Docker.

**Answer:**

Multi-stage builds allow multiple `FROM` instructions in a single Dockerfile. Each `FROM` starts a new stage. You can selectively copy artifacts from one stage to the next, discarding previous stage's build dependencies.

**Benefits:**

- Smaller final image size
- Only contains minimum needed to run application
- Build tools left in build stage

**Example:**

```dockerfile
# Stage 1: Build
FROM node:18-alpine AS build
WORKDIR /app
COPY package*.json ./
RUN npm install
COPY . .
RUN npm run build

# Stage 2: Run
FROM nginx:alpine
COPY --from=build /app/build /usr/share/nginx/html
EXPOSE 80
CMD ["nginx", "-g", "daemon off;"]
```

---

### 14. What is Docker Swarm? How does it compare to Kubernetes?

**Answer:**

Docker Swarm is Docker's native orchestration tool for managing a cluster of Docker hosts.

**Comparison:**

| Feature         | Docker Swarm                           | Kubernetes                                                                   |
| --------------- | -------------------------------------- | ---------------------------------------------------------------------------- |
| **Complexity**  | Simpler to set up and use              | Steeper learning curve                                                       |
| **Integration** | Built into Docker CLI                  | Separate tool                                                                |
| **Features**    | Basic orchestration                    | Advanced capabilities (auto-scaling, service discovery, complex deployments) |
| **Use Case**    | Simpler use cases, smaller deployments | Large, complex, enterprise-grade production environments                     |

**Example (Docker Swarm):**

```bash
# Initialize Swarm manager
docker swarm init --advertise-addr <MANAGER-IP>

# Deploy service with 3 replicas
docker service create --name web-service --replicas 3 -p 8080:80 nginx
```

---

### 15. How do you monitor and log Docker containers?

**Answer:**

**Logging:**

- Docker captures stdout and stderr streams
- Default driver: `json-file`
- View logs: `docker logs my_container`
- Follow logs: `docker logs -f my_container`

**Monitoring:**

- Basic command: `docker stats` (real-time CPU, memory, network usage)
- Production: Third-party tools (Prometheus, Grafana, ELK stack, cloud services)

**Example:**

```bash
# View logs
docker logs my_container

# View real-time stats
docker stats

# Follow logs in real-time
docker logs -f my_container
```

---

## Azure

### 1. What is Microsoft Azure, and what are its core services?

**Answer:**

Microsoft Azure is a comprehensive, open, and flexible cloud computing platform offering over 600 services for building, managing, and deploying applications across a global network of managed data centers.

**Core Services:**

1. **Virtual Machines (Compute)**: Azure VMs, Azure Kubernetes Service (AKS)
2. **Virtual Network (Networking)**: VNet, VPN Gateway, Azure Firewall
3. **Blob Storage (Storage)**: Blob Storage for unstructured data, disk storage
4. **Azure SQL (Database)**: Azure SQL Database, Azure Cosmos DB
5. **Active Directory (Identity & Security)**: Microsoft Entra ID, Key Vault
6. **AI + Machine Learning**: Azure AI Services for speech, vision, cognitive intelligence
7. **DevOps**: Azure DevOps services

---

### 2. Difference between IaaS, PaaS, and SaaS in Azure.

**Answer:**

| Feature           | IaaS                                     | PaaS                                                   | SaaS                                  |
| ----------------- | ---------------------------------------- | ------------------------------------------------------ | ------------------------------------- |
| **Control Level** | High                                     | Medium                                                 | Low                                   |
| **Managed by**    | Customer (OS, Apps)                      | Provider (Runtime, OS)                                 | Provider (All)                        |
| **Target User**   | IT Admins, Network Architects            | Developers, DevOps                                     | End-users                             |
| **Flexibility**   | Highest                                  | Moderate                                               | Lowest                                |
| **Examples**      | Azure VMs, Virtual Network, Disk Storage | Azure App Service, Azure SQL Database, Azure Functions | Microsoft 365, Dynamics 365, Power BI |

---

### 3. What is an Azure Resource Group?

**Answer:**

An Azure resource group is a logical container that holds related resources for an Azure solution (VMs, databases, storage accounts).

**Key Characteristics:**

- **Centralized Management**: Group by project or lifecycle
- **Metadata Storage**: Must be assigned a location for metadata
- **Flexibility**: Resources can reside in different regions
- **Organization**: Essential for dev, test, or production environments

**Key Rules:**

- A resource can only belong to one resource group
- Deleting resource group deletes all resources within it
- Cannot create resource without assigning to resource group

**Example**: "Production-RG" group containing web app, SQL database, and storage account.

---

### 4. What is Azure App Service?

**Answer:**

Azure App Service is a platform that lets you run web applications, mobile backends, and RESTful APIs without managing infrastructure. Think of it as a powerful web hosting service.

**Features:**

- Supports .NET, Java, Node.js, Python, PHP
- Runs on Windows and Linux
- Can deploy custom containers
- Takes care of infrastructure management

---

### 5. Difference between Azure SQL Database and SQL Server on Azure VM.

**Answer:**

| Feature               | Azure SQL Database                 | SQL Server on Azure VM                        |
| --------------------- | ---------------------------------- | --------------------------------------------- |
| **Service Model**     | Platform-as-a-Service (PaaS)       | Infrastructure-as-a-Service (IaaS)            |
| **Management**        | Fully managed by Microsoft         | Full customer control                         |
| **OS Access**         | No access                          | Full administrative control                   |
| **Compatibility**     | High but some features unsupported | Nearly 100% compatibility                     |
| **High Availability** | Built-in (up to 99.995% SLA)       | Requires manual configuration                 |
| **Migration**         | Requires migration tools           | Simple lift-and-shift                         |
| **Use Cases**         | Modern cloud-native apps, SaaS     | Legacy apps, third-party software integration |

---

### 6. What is Azure Virtual Network (VNet)?

**Answer:**

An Azure VNet is the fundamental building block for your private network in Azure, allowing secure connection of Azure resources to each other, the internet, and on-premises networks.

**Key Functions:**

1. **Isolation**: Logically isolated network for Azure resources
2. **Communication**: Secure communication between VMs and services
3. **Hybrid Connectivity**: Connect to on-premises data centers via VPN or ExpressRoute
4. **Subnetting**: Divide VNet into smaller segments
5. **Security**: Integrates with NSGs and Azure Firewall
6. **Customization**: Bring your own IP addresses and DNS settings

---

### 7. How does Azure handle scalability?

**Answer:**

Azure handles scalability through vertical and horizontal scaling, both automated and manual.

**Key Approaches:**

1. **Horizontal Scaling (Scale Out/In)**: Add/remove instances (VMs, containers)
2. **Vertical Scaling (Scale Up/Down)**: Increase/decrease resource capacity
3. **Azure Autoscale**: Automatically manages resources based on metrics or schedules
4. **Predictive Autoscale**: Uses ML to analyze trends and proactively scale
5. **Database Scaling**: Azure SQL Database serverless tiers scale automatically
6. **Regional Distribution**: Deploy across multiple regions and availability zones

---

### 8. Explain Azure Storage types.

**Answer:**

Azure offers five core storage services:

**1. Azure Blob Storage:**

- Unstructured data (images, documents, videos, backups)
- Hot, cool, and archive access tiers

**2. Azure Files:**

- Fully managed SMB/NFS file shares
- Concurrent cloud and on-premises access

**3. Azure Disks:**

- High-performance block storage for VMs
- Managed disks

**4. Azure Table Storage:**

- NoSQL key-attribute store
- Fast, cost-effective structured data storage

**5. Azure Queue Storage:**

- Message queuing for asynchronous communication

---

### 9. What is Azure Functions? How is it different from WebJobs?

**Answer:**

Azure Functions is a serverless compute service for running event-triggered code without managing infrastructure.

**Differences:**

| Feature           | Azure Functions                  | Azure WebJobs                             |
| ----------------- | -------------------------------- | ----------------------------------------- |
| **Hosting Model** | Separate Azure App Service       | Feature of existing App Service           |
| **Scaling**       | Automatic, dynamic scaling       | Manual, tied to App Service plan          |
| **Pricing**       | Pay-per-use, serverless          | No additional cost, uses App Service plan |
| **Management**    | Created directly from portal     | Part of Web App project                   |
| **Triggers**      | Wide variety, extensive bindings | Requires WebJobs SDK for rich triggering  |

---

### 10. Explain the difference between Azure Load Balancer and Application Gateway.

**Answer:**

**Azure Load Balancer (Layer 4):**

- Operates at Transport Layer
- Balances based on IP address and port
- High-performance, low-latency
- Suitable for non-HTTP workloads (databases, gaming, internal traffic)

**Azure Application Gateway (Layer 7):**

- Operates at Application Layer
- Routes based on HTTP/HTTPS attributes (URL paths, host headers, cookies)
- SSL/TLS termination
- Built-in Web Application Firewall (WAF)
- URL-based routing

---

### 11. How do you implement high availability in Azure?

**Answer:**

**Key Mechanisms:**

**Availability Sets:**

- Logical grouping for VMs within single data center
- Spread across different physical hardware (fault domains)
- Separate maintenance cycles (update domains)
- SLA: 99.95%

**Availability Zones:**

- Physically separate data centers within region
- Independent power, cooling, networking
- Protects against data center failures
- SLA: 99.99%

**Zone-redundant Services:**

- Automatically replicate across zones (ZRS, Azure SQL Database)

**Load Balancing:**

- Distribute traffic across healthy instances

---

### 12. What is Azure Kubernetes Service (AKS), and why use it?

**Answer:**

AKS is a fully managed container orchestration service that simplifies Kubernetes deployment and operations in Azure.

**Why Use AKS:**

- **Simplified Management**: Azure manages Kubernetes master nodes
- **Scalability**: Automatic scaling of pods and nodes
- **Portability**: Uses open-source Kubernetes standard
- **Integration**: Seamless with Azure DevOps, Monitor, Active Directory
- **Efficiency**: Optimizes resource utilization and cost

---

### 13. Explain Azure Active Directory (Azure AD) and its use cases.

**Answer:**

Azure Active Directory (now Microsoft Entra ID) is a cloud-based identity and access management (IAM) service.

**Use Cases:**

1. **Single Sign-On (SSO)**: Single identity for multiple applications
2. **Multi-Factor Authentication (MFA)**: Enhanced security with multiple verification
3. **Conditional Access**: Granular access controls based on user context
4. **Application Management**: Central place to manage access permissions
5. **Identity Governance**: Manage identity lifecycle and access rights

---

### 14. How do you implement CI/CD with Azure DevOps?

**Answer:**

**Steps:**

1. **Source Control**: Store code in Azure Repos or GitHub
2. **Continuous Integration (CI)**: Configure Azure Pipeline to automatically build, test, validate
3. **Continuous Delivery (CD)**: Automatically deploy to Azure environments
4. **Testing and Monitoring**: Incorporate automated testing, use Azure Monitor and Application Insights

---

### 15. How does Azure handle disaster recovery and backup?

**Answer:**

**Azure Backup:**

- Cost-effective, one-click solution
- Backs up cloud, on-premises, edge data
- Long-term retention, centralized management
- Application-consistent backups

**Azure Site Recovery (ASR):**

- DR-as-a-service solution
- Orchestrates replication to secondary region
- Near-zero data loss (low RPO)
- Minimal downtime (low RTO)
- Automated failover and failback

**Geo-redundant Storage (GRS/GZRS):**

- Automatically replicates data to secondary region
- Protects against region-wide outages

**Example**: Automating nightly SQL database backups to Azure Blob storage, or replicating critical VMs from East US to West US for disaster recovery.

---

## End of Document
