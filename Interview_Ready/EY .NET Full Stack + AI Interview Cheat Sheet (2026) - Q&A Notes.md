# EY .NET Full Stack + AI Interview Cheat Sheet (2026) - Q&A Notes

## Topic Index

1. [C# / .NET](#1-c--net-interview-questions)
2. [ASP.NET Core / Web API](#2-aspnet-core--web-api-questions)
3. [SQL Server](#3-sql-server-questions)
4. [React](#4-react-questions)
5. [Azure / Cloud](#5-azure--cloud-questions)
6. [Generative AI / AI](#6-generative-ai--ai-questions)
7. [System Design](#7-system-design-questions)
8. [Real Production Scenarios](#8-real-production-scenarios)
9. [Git & DevOps](#9-git--devops-questions)
10. [Senior Interview Formula](#10-senior-interview-formula)
11. [Mock Interview Answer Version](#10a-mock-interview-answer-version)
12. [Final Interview Tips](#11-final-interview-tips)

---

## 1) C# / .NET Interview Questions

### Sample Example: Interface vs Abstract Class

```csharp
public interface INotificationService
{
    void Send(string message);
}

public abstract class BaseNotificationService
{
    public void Log(string message)
    {
        Console.WriteLine($"Logging: {message}");
    }

    public abstract void Send(string message);
}
```

Use an interface when different implementations should share a common contract, such as email, SMS, and push notifications. Use an abstract class when you want a reusable base implementation with some shared logic.

### Q1: What is the difference between an abstract class and an interface?

Answer:
- An abstract class can have both abstract and concrete members.
- An interface defines a contract and typically contains only method signatures, properties, or events.
- Use abstract class when you want shared implementation and inheritance.
- Use interface when you want loose coupling and multiple inheritance-like behavior.

### Q2: What are SOLID principles?

Answer:
- S: Single Responsibility Principle
- O: Open/Closed Principle
- L: Liskov Substitution Principle
- I: Interface Segregation Principle
- D: Dependency Inversion Principle

Example:
- A class should have one reason to change.
- A class should be open for extension but closed for modification.
- High-level modules should not depend on low-level modules; both should depend on abstractions.

### Q3: What is Dependency Injection?

Answer:
- Dependency Injection is a design pattern where dependencies are provided from outside the class instead of being created internally.
- It improves testability, maintainability, and loose coupling.
- In ASP.NET Core, services are registered in Program.cs and injected through constructors.

Common lifetimes:
- Singleton: one instance for the whole app
- Scoped: one instance per request
- Transient: new instance every time it is requested

### Sample Example: Dependency Injection

```csharp
public interface IEmailService
{
    void Send(string to, string subject, string body);
}

public class SmtpEmailService : IEmailService
{
    public void Send(string to, string subject, string body)
    {
        Console.WriteLine($"Sending email to {to}");
    }
}

public class UserService
{
    private readonly IEmailService _emailService;

    public UserService(IEmailService emailService)
    {
        _emailService = emailService;
    }

    public void RegisterUser(string email)
    {
        _emailService.Send(email, "Welcome", "Welcome to our platform");
    }
}
```

```csharp
var builder = WebApplication.CreateBuilder(args);
builder.Services.AddSingleton<IEmailService, SmtpEmailService>();
var app = builder.Build();
```

This is a simple example of constructor injection where the service is resolved by the DI container.

### Q4: What is IAsyncEnumerable in C#?

Answer:
- It allows asynchronous streaming of data.
- It is useful when reading large result sets or processing data incrementally without loading everything into memory.
- It works well for APIs and data pipelines.

### Q5: What are delegates and events?

Answer:
- A delegate is a type that represents a method reference.
- An event is a special form of delegate designed for notifications.
- Events are useful for publisher-subscriber patterns such as UI events and service notifications.

### Q6: What are common design patterns?

Answer:
- Factory: creates objects without exposing the creation logic
- Singleton: ensures only one instance exists
- Repository: abstracts data access logic
- Observer: allows objects to subscribe to changes
- Strategy: selects behavior at runtime

### Q7: What is clean architecture?

Answer:
- Clean architecture separates concerns into layers such as:
  - Domain
  - Application
  - Infrastructure
  - Presentation
- It keeps business logic independent from frameworks and external systems.
- It improves testability and maintainability.

### Q8: What are the new features in modern C#?

Answer:
- Records
- Primary constructors
- Pattern matching improvements
- Enhanced switch expressions
- Required members
- Collection expressions
- Improved async and performance features

---

## 2) ASP.NET Core / Web API Questions

### Sample Example: ASP.NET Core Web API

```csharp
[ApiController]
[Route("api/[controller]")]
public class OrdersController : ControllerBase
{
    private readonly IOrderService _orderService;

    public OrdersController(IOrderService orderService)
    {
        _orderService = orderService;
    }

    [HttpGet("{id}")]
    public async Task<ActionResult<OrderDto>> Get(int id)
    {
        var order = await _orderService.GetByIdAsync(id);
        return Ok(order);
    }
}
```

This is a typical controller pattern where the service layer handles business logic, and the controller focuses on HTTP concerns.

### Q1: What is ASP.NET Core Web API?

Answer:
- It is a framework for building HTTP-based services.
- It is used for exposing endpoints to web and mobile clients.
- It is lightweight, cross-platform, and performance-oriented.

### Q2: What is JWT authentication?

Answer:
- JWT stands for JSON Web Token.
- It contains claims, usually used to identify a user.
- It is commonly used for stateless authentication in APIs.

### Sample Example: JWT Authentication in Minimal API

```csharp
using Microsoft.AspNetCore.Authentication.JwtBearer;
using Microsoft.IdentityModel.Tokens;
using System.Text;

var builder = WebApplication.CreateBuilder(args);

var jwtKey = builder.Configuration["Jwt:Key"] ?? "ThisIsASecretKey1234567890";
var issuer = builder.Configuration["Jwt:Issuer"] ?? "myapp";

builder.Services.AddAuthentication(JwtBearerDefaults.AuthenticationScheme)
    .AddJwtBearer(options =>
    {
        options.TokenValidationParameters = new TokenValidationParameters
        {
            ValidateIssuer = true,
            ValidateAudience = true,
            ValidateLifetime = true,
            ValidateIssuerSigningKey = true,
            ValidIssuer = issuer,
            ValidAudience = "myapp-users",
            IssuerSigningKey = new SymmetricSecurityKey(Encoding.UTF8.GetBytes(jwtKey))
        };
    });

builder.Services.AddAuthorization();

var app = builder.Build();

app.UseAuthentication();
app.UseAuthorization();

app.MapGet("/secure", () => "This is secured")
   .RequireAuthorization();

app.Run();
```

This is a minimal API setup where a JWT is validated and only authorized users can hit the secure endpoint.

### Q3: Authentication vs Authorization

Answer:
- Authentication: who are you?
- Authorization: what are you allowed to do?

Example:
- A user logs in -> authentication
- A user tries to access admin APIs -> authorization

### Q4: How do you implement global exception handling?

Answer:
- Use middleware or exception filters.
- Centralize error handling for consistent API responses.
- Log exceptions, return proper HTTP status codes, and avoid exposing internal raw details to clients.

### Q5: How do you handle API versioning?

Answer:
- Use URL-based versioning, header-based versioning, or query string versioning.
- It helps maintain backward compatibility as APIs evolve.

### Q6: How do you secure a REST API?

Answer:
- Use HTTPS
- Validate tokens and claims
- Use authorization policies
- Validate input and output models
- Enforce rate limiting
- Use CORS carefully
- Protect against injection and over-posting

### Q7: How can you improve API performance?

Answer:
- Use async/await
- Reduce unnecessary database calls
- Enable caching
- Use pagination
- Avoid large payloads
- Compress responses
- Use proper indexing and query optimization

### Q8: How do you handle file uploads?

Answer:
- Accept multipart/form-data
- Validate file size and type
- Store safely in blob storage or a secure target
- Use antivirus and content scanning if needed

### Q9: What is rate limiting?

Answer:
- It restricts how often a user or service can call an endpoint.
- It helps protect APIs from abuse, traffic spikes, and denial-of-service issues.

### Q10: How do you implement logging and monitoring?

Answer:
- Use structured logging with ILogger
- Log request IDs, correlation IDs, and important events
- Send metrics and logs to Azure Monitor, Application Insights, or similar tools
- Track latency, exception counts, and business failures

---

## 3) SQL Server Questions

### Sample Example: JOIN and Indexing

```sql
CREATE INDEX IX_Orders_CustomerId ON Orders(CustomerId);

SELECT c.Name, o.OrderId, o.TotalAmount
FROM Customers c
LEFT JOIN Orders o ON c.CustomerId = o.CustomerId
WHERE c.IsActive = 1;
```

This query uses an index to speed up lookups on `CustomerId` and returns all active customers, even if they have no orders yet.

### Q1: What is the difference between INNER JOIN and LEFT JOIN?

Answer:
- INNER JOIN returns only matching rows.
- LEFT JOIN returns all rows from the left table and matching records from the right table.
- If no match exists, NULL values appear on the right side.

### Q2: When should you use indexes?

Answer:
- Use indexes on columns used in WHERE, JOIN, ORDER BY, and GROUP BY clauses.
- Indexes speed up read operations but can slow down writes.
- Avoid over-indexing large tables.

### Q3: What is a stored procedure?

Answer:
- A stored procedure is a precompiled database routine.
- It can encapsulate business logic and reduce repeated SQL code.
- It improves performance, maintainability, and security in some cases.

### Q4: How do you optimize SQL queries?

Answer:
- Use proper indexes
- Avoid SELECT *
- Reduce unnecessary joins
- Analyze execution plans
- Use pagination for large datasets
- Rewrite inefficient queries

### Q5: What are locking and deadlocks?

Answer:
- Locking prevents concurrent conflicts in the database.
- Deadlock happens when two or more transactions block each other.
- To reduce deadlocks, keep transactions short, consistent in order, and avoid long locks.

### Q6: What is the difference between CTE and subquery?

Answer:
- A CTE is a named temporary result set and improves readability.
- A subquery is a nested query inside another query.
- CTEs are often easier to maintain for recursive queries and complex logic.

### Q7: What are ROW_NUMBER, RANK, and DENSE_RANK?

Answer:
- ROW_NUMBER: assigns a unique number to each row
- RANK: assigns same rank to ties, skipping numbers
- DENSE_RANK: assigns same rank to ties without skipping numbers

### Q8: What is database normalization?

Answer:
- Normalization organizes data to reduce redundancy and improve integrity.
- It usually means splitting data into related tables and reducing duplication.

### Sample Example: Second Highest Salary

```sql
SELECT MAX(Salary) AS SecondHighestSalary
FROM Employees
WHERE Salary < (SELECT MAX(Salary) FROM Employees);
```

If you want to handle duplicate salaries correctly, a common alternative is:

```sql
SELECT DISTINCT Salary
FROM Employees
ORDER BY Salary DESC
OFFSET 1 ROWS FETCH NEXT 1 ROW ONLY;
```

This is a classic SQL interview question and a good example of ranking and filtering logic.

---

## 4) React Questions

### Sample Example: useState + useEffect

```jsx
import { useEffect, useState } from 'react';

function UserList() {
  const [users, setUsers] = useState([]);
  const [loading, setLoading] = useState(true);

  useEffect(() => {
    fetch('/api/users')
      .then(res => res.json())
      .then(data => {
        setUsers(data);
        setLoading(false);
      });
  }, []);

  if (loading) return <p>Loading...</p>;

  return (
    <ul>
      {users.map(user => <li key={user.id}>{user.name}</li>)}
    </ul>
  );
}
```

This pattern loads data after the component renders and keeps the UI state in sync with the API response.

### Sample Example: Data Fetching with useEffect (Interview Style)

```jsx
import { useEffect, useState } from 'react';

function Products() {
  const [products, setProducts] = useState([]);
  const [error, setError] = useState('');

  useEffect(() => {
    const loadProducts = async () => {
      try {
        const res = await fetch('/api/products');
        if (!res.ok) throw new Error('Failed to load products');
        const data = await res.json();
        setProducts(data);
      } catch (err) {
        setError('Something went wrong');
      }
    };

    loadProducts();
  }, []);

  return (
    <div>
      {error && <p>{error}</p>}
      {products.map(p => <div key={p.id}>{p.name}</div>)}
    </div>
  );
}
```

This is a common React pattern for loading data from an API and handling loading/error states.

### Q1: What is the difference between useState and useEffect?

Answer:
- useState manages component state.
- useEffect runs after render and is useful for side effects like API calls, subscriptions, or DOM updates.

### Q2: What is the difference between useMemo and useCallback?

Answer:
- useMemo memoizes computed values.
- useCallback memoizes functions.
- They are useful for performance optimization, especially when props or state changes are expensive.

### Q3: What is the difference between props and state?

Answer:
- Props are external inputs passed from a parent component.
- State is internal component data that can change over time.

### Q4: What is Context API?

Answer:
- Context API allows passing data through the component tree without manually passing props at every level.
- It is useful for themes, auth, or global settings.

### Q5: How do you prevent unnecessary re-renders?

Answer:
- Use memoization
- Keep components small
- Avoid creating new objects or arrays in render unless needed
- Use React.memo when appropriate
- Keep state local to the component that needs it

### Q6: How do you do lazy loading?

Answer:
- Use React.lazy and Suspense for dynamic imports.
- Split large code bundles to reduce initial load time.
- Load heavy features only when needed.

### Q7: How do you handle API integration in React?

Answer:
- Use async functions and fetch/axios
- Handle loading, error, and success states
- Keep API logic separated from component UI logic when possible
- Use service layers or custom hooks

### Q8: What are best practices for component design?

Answer:
- Keep components small and focused
- Use reusable components
- Keep logic separate from presentation
- Prefer props over global state when possible
- Use TypeScript for better safety

---

## 5) Azure / Cloud Questions

### 5.1 Important Azure Services for Interviews

1. Azure App Service — used for hosting web apps, APIs, and backend services.
2. Azure Functions — used for serverless event-driven processing.
3. Azure SQL — best for relational database workloads.
4. Azure Storage (Blob, Queue, Table) — used for object, messaging, and structured storage.
5. Azure Service Bus — used for reliable messaging and async communication.
6. Azure Function / Azure AI — used for automation and intelligent processing.
7. Azure OpenAI — used to integrate LLM capabilities into business apps.
8. Azure Monitor — used for logging, metrics, alerts, and observability.
9. AKS / Containers — used for microservices and container orchestration.
10. Entra ID (Azure AD) — used for identity and authentication.

### Real-Time Example: Azure App Service and Azure SQL

```text
A company builds a customer portal with:
- Frontend web app hosted in Azure App Service
- Backend API hosted in Azure App Service
- SQL database in Azure SQL
- File uploads stored in Azure Blob Storage
- Orders processed asynchronously using Azure Service Bus and Azure Functions
```

This is a common production architecture: web app + API + database + messaging + background processing.

### Real-Time Example: Azure Storage

```text
Use Azure Blob Storage for:
- storing user profile images
- uploading PDFs and documents
- backups and static website content

Use Azure Queue Storage for:
- background jobs
- decoupling services
- messaging between producer and consumer services
```

### Sample Example: Azure App Service vs Azure Functions

```text
Scenario: A company wants to host a .NET web application and trigger background processing when an order is created.

Use Azure App Service for:
- public web application
- REST API
- admin portal

Use Azure Functions for:
- event-driven processing
- queue-based background jobs
- scheduled tasks
```

This is a common Azure design pattern: app hosting and background automation are separated based on workload type.

### Q1: What is Azure App Service?

Answer:
- Azure App Service is a managed platform for hosting web apps and APIs.
- It supports deployment, scaling, and integration.
- Real-world use case: hosting an internal employee portal or a public business API.

### Q2: When should you use Azure Functions?

Answer:
- Use Azure Functions for event-driven, serverless workloads.
- It is ideal for lightweight automation, file processing, scheduled jobs, and event handling.
- Real-world use case: sending an email after an order is placed or resizing an uploaded image.

### Q3: When should you use Azure SQL?

Answer:
- Use Azure SQL for relational database workloads requiring strong consistency, transactions, and enterprise-grade features.
- Real-world use case: storing customer orders, product information, and transactional records.

### Q4: What is Azure Blob Storage?

Answer:
- It is object storage for unstructured data such as files, images, documents, and backups.
- Real-world use case: storing uploaded invoices, documents, and media files.

### Q5: What is Azure Service Bus?

Answer:
- It is a messaging service used for reliable asynchronous communication between application components.
- Real-world use case: order processing where the API quickly accepts the order and a background service handles billing and notifications.

### Q6: What is Azure Monitor?

Answer:
- Azure Monitor provides insights into performance, logs, metrics, and health of services.
- It is essential for observability and diagnostics.
- Real-world use case: checking whether an API is failing or a database is experiencing latency spikes.

### Q7: What is Azure CDN?

Answer:
- Azure CDN accelerates content delivery by caching content closer to users.
- It helps reduce latency and improve performance for global users.
- Real-world use case: serving product images and static assets across multiple regions.

### Q8: What is Azure OpenAI and when is it used?

Answer:
- Azure OpenAI provides access to large language models in Azure.
- It is used for chatbots, summarization, document analysis, code generation, and natural language processing.
- Real-world use case: AI-powered support assistant that answers questions based on company documents.

### Q9: What is Entra ID?

Answer:
- Entra ID is Microsoft’s identity and access management service.
- It is used for authentication and authorization in Azure environments.
- Real-world use case: allowing employees to sign in to business apps using company accounts.

### Q10: What is the typical .NET app on Azure?

Answer:
- Client or mobile app
- App Service (API and web app)
- Azure SQL or Cosmos DB depending on needs
- Blob Storage for files
- Service Bus for messaging
- Monitor for observability
- Azure Functions for asynchronous jobs
- SOL for a relational database model in some cases

This is the common architecture the image is describing: a web/mobile client talks to an app service, which uses Azure SQL and other managed services.

---

## 6) Generative AI / AI Questions

### Sample Example: RAG Flow

```text
User Query: "What is our refund policy for prepaid plans?"

1. Search relevant documents in the knowledge base
2. Convert the query into vector embeddings
3. Find the most similar passages
4. Add retrieved context to the LLM prompt
5. Generate a grounded answer
6. Validate the answer before returning it to the user
```

This is the basic idea behind Retrieval-Augmented Generation (RAG): make the model answer from reliable documents instead of guessing.

### Q1: What is Generative AI?

Answer:
- Generative AI models can create text, code, images, or other content based on patterns learned from data.
- It is useful for chatbots, copilots, content generation, and automation tasks.

### Q2: How does Generative AI work?

Answer:
- It is trained on large datasets using neural network models.
- It predicts the next token or output based on learned patterns.
- It uses prompts and context to generate responses.

### Q3: What is RAG?

Answer:
- RAG stands for Retrieval-Augmented Generation.
- It combines a retriever with an LLM so the model can answer based on relevant documents instead of relying only on training data.
- It is useful for enterprise knowledge systems.

### Q4: What is a vector database?

Answer:
- A vector database stores embeddings that represent semantic meaning.
- It helps find similar documents or content using vector similarity search.

### Q5: How do you build an AI application?

Answer:
- Define the business problem
- Choose the model and architecture
- Add retrieval logic if needed
- Add validation and guardrails
- Monitor quality and cost
- Put the app behind a secure and scalable system design

### Q6: How do you evaluate AI outputs?

Answer:
- Measure relevance, factuality, latency, cost, and safety
- Test with real-world prompts and scenarios
- Evaluate hallucinations and edge cases
- Add human review for critical workflows

---

## 7) System Design Questions

### Sample Example: Scalable eCommerce Design

```text
Client -> API Gateway -> Order Service -> Database
                |                     |
                v                     v
            Auth Service          Redis Cache
                |
                v
            Message Queue -> Payment Service
```

This architecture separates concerns, uses caching for read-heavy operations, and uses a queue to decouple order processing from payment logic.

### Q1: How do you design a scalable enterprise application?

Answer:
- Split the system into clear service boundaries
- Use API gateways and service communication patterns
- Add caching, queues, and load balancing
- Choose the right database model based on workload
- Add monitoring and observability
- Plan for resilience and backup strategies

### Q2: What key decisions matter in system design?

Answer:
- Scalability
- Fault tolerance
- Reliability
- Service boundaries
- Database choice
- Microservice vs modular monolith
- Communication patterns
- Caching and eventual consistency

### Q3: What is the design principle “scale, built for resilience, and think about the next 10x”?

Answer:
- Systems should be designed not only for today’s load, but also for future growth.
- They should be resilient under failure, high load, and changing demand.
- The next 10x growth may come with different bottlenecks and scaling needs.

---

## 8) Real Production Scenarios

### Sample Example: Incident Response Flow

```text
Problem: API latency is rising during peak hours.

1. Check Application Insights / logs
2. Find slow endpoints and database calls
3. Review CPU, memory, and SQL execution plans
4. Add cache or optimize query
5. Scale horizontally if necessary
6. Monitor again to confirm improvement
```

This is the kind of structured incident response that senior engineers use in production.

### Q1: API latency suddenly increases. What do you do?

Answer:
- Check logs and tracing
- Measure database latency and dependency health
- Review traffic and request spikes
- Check CPU, memory, and connection pool usage
- Increase cache or optimize hot queries

### Q2: A database query takes too long. How do you diagnose it?

Answer:
- Inspect execution plan
- See if missing index or heavy joins are the cause
- Check blocking and locks
- Profile the query and optimize data access patterns

### Q3: One service is unstable in production. What do you do?

Answer:
- Check health metrics and logs
- Identify if the issue is dependency-related or resource-related
- Roll back or isolate the failing component
- Add retries, circuit breakers, and better observability

### Q4: An app works locally but fails in production. Why?

Answer:
- Environment differences
- Missing secrets or config values
- Network policies or firewall rules
- Database connectivity differences
- Deployment mismatch or resource limitations

---

## 9) Git & DevOps Questions

### Sample Example: CI/CD Pipeline

```yaml
steps:
  - script: dotnet restore
    displayName: Restore dependencies

  - script: dotnet build --configuration Release
    displayName: Build project

  - script: dotnet test --logger trx
    displayName: Run tests

  - task: PublishBuildArtifacts@1
    displayName: Publish artifacts
```

A basic pipeline checks the build, executes tests, and publishes the resulting artifact for deployment.

### Q1: What are common Git commands?

Answer:
- git status
- git add
- git commit
- git push
- git pull
- git fetch
- git checkout
- git merge
- git rebase
- git stash
- git log
- git revert

### Q2: What is the difference between merge and rebase?

Answer:
- Merge preserves history and creates a merge commit.
- Rebase rewrites commit history to produce a linear timeline.
- Rebase is useful for clean history but should be used carefully in shared branches.

### Q3: What is CI/CD?

Answer:
- CI: continuous integration — build and test automatically on code changes
- CD: continuous delivery or deployment — automatically deliver software to environments

### Q4: What are common pipeline stages?

Answer:
- Build
- Test
- Static analysis
- Package
- Deploy
- Validate

---

## 10) Senior Interview Formula

### Sample Example: Strong Interview Answer Structure

```text
Question: Why did you choose Azure Functions for this workflow?

Problem: We needed asynchronous processing without keeping the HTTP request open.
Investigation: We reviewed the workload and found it was event-driven and not user-facing.
Decision: Azure Functions was a better fit than a web API because it scales on demand and reduces idle cost.
Implementation: We created a queue-triggered function to process tasks asynchronously.
Impact: Response time improved, the app stayed responsive, and background jobs scaled reliably.
```

This pattern clearly shows engineering thought, not just memorized facts.

### Senior interview structure

Use this formula for strong engineering answers:

1. Problem
   - What is the issue or requirement?

2. Investigation
   - How do you analyze the problem?

3. Decision
   - Why is this the best approach?

4. Implementation
   - How do you execute the solution?

5. Impact
   - What was the result?

This structure shows clear thinking and engineering judgment.

### Q: How do you show strong engineering judgment?

Answer:
- Ask clarifying questions
- Balance trade-offs
- Think about maintainability, cost, and scalability
- Discuss alternatives
- Explain why one design is better than another
- Connect decisions to business impact

---

## 10A) Mock Interview Answer Version

### 1. What is the difference between an abstract class and an interface?

Mock Answer:

"I would use an abstract class when I want to provide shared implementation for a group of related classes. For example, if multiple services need the same logging behavior, I can put that logic in a base abstract class. I would use an interface when I want to define a contract and allow different implementations, such as email, SMS, and push notifications, to follow the same structure without inheriting behavior. In my experience, interfaces are useful for loose coupling and testability, while abstract classes are better when reusable logic is needed."

### 2. What is Dependency Injection and why is it important?

Mock Answer:

"Dependency Injection is a design pattern where dependencies are supplied from outside the class instead of being created inside it. This makes the code more testable and maintainable because we can replace implementations easily. In ASP.NET Core, we register services in the dependency container and inject them through constructors. For example, a controller depends on an `IOrderService`, and the container resolves the actual implementation. This reduces tight coupling and makes unit testing much easier."

### 3. How do you secure a REST API?

Mock Answer:

"To secure a REST API, I would first enforce HTTPS and validate all input. I would use authentication and authorization using JWT or Azure AD depending on the environment. I would also protect endpoints with role-based or policy-based authorization rules. In addition, I would validate request models, apply rate limiting, use CORS carefully, and ensure secrets are stored securely. Logging and monitoring are also important so we can detect suspicious patterns early."

### 4. How do you optimize a slow SQL query?

Mock Answer:

"I would start by identifying the slow query using execution plans and query statistics. Then I would check whether the query is missing indexes, performing unnecessary joins, or scanning too much data. I would also look for blocking or locking issues under concurrency. If needed, I would add the right index, reduce the result set, and avoid using `SELECT *`. In production, the goal is to reduce read cost and improve query execution without hurting write performance."

### 5. What is the difference between useState and useEffect in React?

Mock Answer:

"`useState` is used to manage state inside a component. For example, storing a list of users or a loading flag. `useEffect` is used for side effects, such as fetching data after the component renders or subscribing to events. A common pattern is using `useState` to hold the result of the API call and `useEffect` to trigger the fetch when the component mounts. This keeps the UI synchronized with the data."

### 6. What is Azure App Service and when would you use it?

Mock Answer:

"Azure App Service is a managed hosting platform for web applications and APIs. I would use it when I need to deploy a .NET web app or REST API with built-in scaling, monitoring, and deployment support. It is a good choice for public-facing applications where I want a managed environment without worrying about the underlying infrastructure. If the workload is event-driven or serverless, Azure Functions would be a better fit."

### 7. What is RAG in Generative AI?

Mock Answer:

"RAG stands for Retrieval-Augmented Generation. It combines a search or retrieval layer with an LLM so the model can answer questions using relevant documents from a knowledge base instead of relying only on its training data. For example, if a user asks about a company refund policy, the system retrieves the correct policy document, passes it to the model as context, and then generates a grounded answer. This reduces hallucination and improves accuracy in enterprise use cases."

### 8. How do you design a scalable application?

Mock Answer:

"I would start by separating the system into clear service boundaries and identifying the major business flows. I would use load balancing, caching, and message queues where needed to handle traffic spikes and decouple components. I would also choose the right database model based on the workload, such as SQL for relational consistency or NoSQL for flexible high-scale access patterns. Monitoring, retries, and failover strategies are also important so the system remains reliable under load."

### 9. What is CI/CD and why does it matter?

Mock Answer:

"CI/CD stands for Continuous Integration and Continuous Delivery or Deployment. In practice, CI ensures code is built and tested automatically whenever changes are made. CD ensures those changes can be released to environments in a consistent and repeatable way. This reduces manual work, catches issues early, and helps teams ship software faster with more confidence. In a .NET pipeline, I would typically restore dependencies, build the project, run tests, and then publish the artifact for deployment."

### 10. How do you answer a senior interview question in a strong way?

Mock Answer:

"I structure my answer around the problem, the investigation, the decision, the implementation, and the impact. I first explain the business or technical problem, then I discuss how I investigated the root cause or requirement. After that, I explain why I chose a particular design or technology. Then I describe the implementation and finally the outcome, such as reduced latency, lower cost, or improved reliability. This makes my answer clearer and demonstrates engineering judgment rather than memorized facts."

---

## 11) Final Interview Tips

### Q1: How do you prepare for the interview?

Answer:
- Review real projects with business impact
- Practice core technical questions
- Understand trade-offs and design decisions
- Practice coding and system design under time pressure
- Stay updated with .NET, Azure, and AI trends

### Q2: What is the most important thing to do during interviews?

Answer:
- Communicate clearly
- Show structured thinking
- Explain assumptions
- Talk through trade-offs
- Focus on problem-solving, not memorization

### Q3: What should you emphasize in your answers?

Answer:
- Clarity
- Ownership
- Impact
- Scalability
- Reliability
- Continuous learning

---

## 12) Final Takeaway

The best interview answers are not just technical—they are structured, practical, and tied to real-world outcomes.

As a .NET Full Stack + AI engineer, you should be ready to explain:
- C# and .NET fundamentals
- ASP.NET Core APIs
- SQL performance and design
- React architecture and state management
- Azure deployment and cloud services
- AI system design with RAG and LLM workflows
- system design trade-offs
- DevOps and Git best practices

The goal is not just to know the terms, but to explain how and why a solution works.

---

## Quick Summary

- Learn the fundamentals deeply
- Practice real-world examples
- Show engineering judgment
- Tie technical solutions to business impact
- Keep improving with .NET, Azure, and AI trends

This cheat sheet is best used as a compact review tool before interviews and coding rounds.
