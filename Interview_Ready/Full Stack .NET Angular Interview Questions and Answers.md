# Full-Stack .NET + Angular Interview Questions and Answers

Use these as spoken-answer starting points, not memorized scripts. Replace every bracketed item with a true detail from your experience. In particular, do not claim ownership of a feature, production deployment, AI agent, RAG system, or model deployment unless you have actually done it.

## Project and Experience

### 1. Explain your current project, its architecture, roles, and deployment

**Sample answer:**

"I work on **[project/product name]**, a **[one-sentence description of the business problem]**. The frontend is built with Angular and communicates with ASP.NET Core Web APIs over HTTPS. The API controllers handle HTTP concerns and delegate work to application or business services. Those services apply validation and business rules, then use a data-access layer, commonly Entity Framework Core, to interact with **[database]**. The application is organized into **[actual layers/modules]** to keep UI, business logic, and persistence responsibilities separate.

"My role is **[your role]**. I work with **[team roles, such as frontend, backend, QA, product owner, and DevOps]**. We deploy through **[actual CI/CD or release process]** to **[actual environment/platform]**."

Be ready to explain one real request from the Angular screen through the API and database, and state clearly which parts you personally worked on.

### 2. Explain the complete flow: Angular Frontend → API → Business Layer → Database

"The user interacts with an Angular component. The component calls an Angular service, which uses `HttpClient` to send an HTTP request to an ASP.NET Core endpoint. The API pipeline applies middleware such as authentication, authorization, and exception handling, then routes the request to a controller or endpoint. The controller validates the HTTP-level input and calls an application/business service. That service applies business rules and calls a repository or data-access abstraction. The data layer queries or updates the database, often through Entity Framework Core. The result travels back through the service and API, is serialized as JSON with an appropriate HTTP status code, and the Angular service returns it to the component, which updates the view."

### 3. What exactly was your responsibility in the project?

"My responsibility was **[specific responsibility]**. I personally handled **[tasks you owned]**, including **[design, implementation, tests, code review, troubleshooting, or release tasks that are true]**. I collaborated with **[relevant teammates]** for **[shared areas]**. I was accountable for **[specific outcome]**, while **[other team/member]** owned **[important boundary, if useful]**."

Use "I" for your contributions and "we" for team outcomes. Avoid describing the entire system as your individual work.

### 4. Which modules or features did you develop?

"I developed or contributed to **[module/feature 1]**, **[module/feature 2]**, and **[module/feature 3]**. For **[feature]**, I worked on **[Angular UI/API/business logic/database/tests]**. My level of ownership was **[implemented end-to-end / implemented the backend / shared ownership / fixed and extended existing functionality]**."

Choose only modules you can explain in detail, including a decision or challenge for each.

### 5. Explain one important feature that you developed end-to-end

"One feature I worked on was **[feature]**, which lets **[user]** accomplish **[business goal]**. I clarified the requirements and validation rules, built **[Angular component/form/table]**, and connected it to **[API endpoint]** through an Angular service. In ASP.NET Core, I implemented **[controller/endpoint]** and **[business service]**, then persisted or retrieved the data using **[data-access approach and database]**. I handled **[authorization, validation, errors, loading/empty states]** and tested **[important test cases]**. One challenge was **[real challenge]**; I addressed it by **[actual solution]**. The result was **[measurable or observable outcome]**."

### 6. Why did you use a layered architecture?

"Layering separates responsibilities. The API layer handles HTTP requests and responses, the business/application layer owns use cases and business rules, and the data-access layer handles persistence. This reduces coupling, makes business logic easier to test, and lets us change one concern with less impact on others. Layers should represent meaningful boundaries; I avoid adding layers that only pass calls through without adding value."

### 7. Why do you use Dependency Injection in .NET?

"Dependency Injection supplies a class's dependencies from outside instead of having the class construct concrete dependencies itself. In .NET, the built-in container registers services and resolves them, commonly through constructor injection. This reduces coupling, makes implementations replaceable, and makes unit testing easier because dependencies can be substituted with fakes or mocks."

### 8. Are you aware of the Repository Pattern?

"Yes. A repository provides an abstraction over data access, exposing operations in terms of the application's needs rather than database details. It can make persistence easier to isolate and test. I use it when it creates a useful boundary, especially with complex queries or multiple data sources. With Entity Framework Core, `DbContext` already provides repository- and unit-of-work-like behavior, so a generic repository that merely duplicates every `DbSet` operation can add unnecessary abstraction."

### 9. How does authentication and authorization work in your application (Angular + .NET)?

"Authentication establishes who the user is; authorization determines what that user is allowed to do. In a common JWT setup, the user signs in through an identity provider or authentication endpoint and receives a token. Angular attaches the access token to protected API requests, often with an HTTP interceptor. ASP.NET Core validates the token's signature, issuer, audience, and expiry, then makes its claims available to the application. `[Authorize]` and policies or roles restrict endpoints. The API is the security boundary: hiding a button in Angular improves the UI but does not replace server-side authorization."

For browser applications, explain your actual token-storage approach and its tradeoffs. Do not assume every application uses JWT; cookie-based authentication and an external identity provider are also common.

### 10. How do you handle errors and logging in your API?

"I handle expected failures deliberately and unexpected failures centrally. Input and business-rule failures return meaningful 4xx responses, not-found cases return 404, and unexpected exceptions are caught by centralized exception-handling middleware and returned as a safe 5xx response. I use structured logging with a correlation or trace ID and include useful context, but avoid logging secrets, credentials, tokens, or unnecessary personal data. I also make sure logs are available in the application's monitoring system and that the client receives a useful message without internal stack traces."

### 11. How do you integrate external APIs into your application?

"I isolate the integration behind a typed client or service, configure it through dependency injection and `IHttpClientFactory`, and keep credentials in a secure configuration provider rather than source code. I define request and response DTOs, set timeouts, handle non-success status codes, and use cancellation tokens. For transient failures, I apply bounded retries with backoff only when retrying is safe; I avoid retrying non-idempotent operations blindly. I also consider rate limits, authentication, validation, logging, and how the application behaves when the external service is unavailable."

### 12. Have you deployed your .NET application? Explain the process

**Use the version that matches your experience.**

**If you deployed it yourself:** "I have deployed **[application]** to **[platform/environment]**. The process was to restore and build the solution, run tests, publish the .NET application, configure environment-specific settings and secrets, apply database migrations through the approved process, and deploy the artifacts using **[pipeline/tool]**. I then verified health checks and key workflows, reviewed logs, and had a rollback or redeployment plan."

**If you supported but did not own deployment:** "I contributed to deployment by **[specific contribution]**. The release was built and tested in CI, configured for the target environment, deployed through **[tool/team]**, and verified using **[checks]**. The deployment itself was owned by **[team/person]**."

**If you have not deployed one:** "I have not independently deployed a production .NET application yet. I understand the typical build, test, publish, configuration/secrets, database migration, deployment, health-check, and rollback steps, and my hands-on exposure so far is **[truthful scope]**."

## C# and .NET

### 13. Explain nullable types. How does `int?` differ from `int`?

"A regular `int` is a non-nullable value type and always contains an integer value; its default is `0`. `int?` is shorthand for `Nullable<int>` and can contain an integer or `null`, which is useful when a value is optional or missing, such as an unset database field. I check for null before using it or provide a fallback with `??`. Nullable reference types are a separate compiler feature that helps identify possible null references for reference types such as `string`."

### 14. Abstract class vs interface: when would you choose one?

"I choose an interface when I need a contract that different, potentially unrelated classes can implement, such as `IPaymentProcessor`. A class can implement multiple interfaces. I choose an abstract class when related derived classes need a shared base implementation or protected state, while still leaving some behavior abstract. In short, interfaces describe capabilities; abstract classes can share both a contract and implementation."

### 15. Partial class: why and when would you use it?

"A partial class lets the compiler combine parts of one class declared across multiple files in the same assembly. It is useful when generated code and hand-written code need to coexist, because the hand-written file can remain separate from regenerated output. It can also help split a genuinely large class, though I would usually consider whether the class has too many responsibilities before using partial files just to organize it."

### 16. Method overloading vs method overriding

"Overloading uses the same method name with different parameter lists, usually in the same type; the compiler selects the matching overload at compile time. Overriding replaces a `virtual` or `abstract` base-class implementation in a derived class using `override`; which implementation runs is determined by the runtime object's type. Changing only the return type does not create a valid overload."

### 17. Array vs `List<T>`, and `IEnumerable<T>` vs `IQueryable<T>`

"An array has a fixed length after creation and is useful when the size is known or fixed. `List<T>` is a resizable collection with convenient add, remove, and indexing operations. Both are in-memory collections.

"`IEnumerable<T>` represents a sequence that can be iterated; LINQ operations on it generally execute in the application process. `IQueryable<T>` represents a query that a provider can translate, for example into SQL for a database. Filtering an `IQueryable<T>` before materializing it can let the database do the work. Calling `ToList()` materializes the results, so subsequent operations run in memory."

### 18. `FirstOrDefault()` vs `SingleOrDefault()`: when would you use each?

"`FirstOrDefault()` returns the first matching item, or the default value if there is none; it is appropriate when I need one item and multiple matches are acceptable. `SingleOrDefault()` returns the item only if there is at most one match; it returns the default when there are none and throws if there are multiple matches. I use it when uniqueness is part of the requirement and want duplicate data to be exposed rather than silently ignored."

### 19. How does `GroupBy` work in LINQ?

"`GroupBy` partitions a sequence by a key selector and produces groups, each with a key and the items for that key. For example, I can group employees by department and calculate a count or average per department. With `IQueryable<T>`, a LINQ provider may translate supported grouping operations into SQL; with `IEnumerable<T>`, grouping happens in memory. I check where the query executes and avoid loading a large dataset before grouping it."

### 20. Explain Garbage Collection in .NET

"The .NET garbage collector automatically reclaims memory used by managed objects that are no longer reachable. It uses generations: short-lived objects are collected more frequently, while longer-lived objects can move to older generations. Developers normally do not manually free managed memory, but they must release unmanaged resources such as file handles or database connections. `IDisposable` and `using` provide deterministic cleanup; garbage collection is not a substitute for disposing resources."

### 21. Transient vs Scoped vs Singleton: how do you decide?

"Transient creates a new instance each time the service is requested. Scoped creates one instance per request scope, which is usually appropriate for request-oriented services and an EF Core `DbContext`. Singleton creates one instance for the application's lifetime and is suitable for genuinely shared, thread-safe services. I consider state, lifetime, and thread safety; a singleton must not directly capture a scoped dependency, because that creates a lifetime mismatch."

## SQL and Data

### 22. Difference between `WHERE` and `HAVING`

"`WHERE` filters individual rows before grouping and aggregation. `HAVING` filters groups after `GROUP BY` and can use aggregate results. For example, I use `WHERE` to select orders from this year, then `HAVING COUNT(*) > 5` to keep customers with more than five qualifying orders."

### 23. Why can't aggregate functions like `COUNT()` or `SUM()` be used in a `WHERE` clause?

"`WHERE` is evaluated before grouping and aggregation, so an aggregate such as `COUNT()` does not exist at that stage. `HAVING` filters after the groups and their aggregates have been calculated. If I need to filter rows based on an aggregate, I use `HAVING` or put the aggregate query in a subquery or common table expression."

### 24. Stored procedure vs function

"A stored procedure is invoked as a command and can perform operations, return result sets, and use output parameters; exact capabilities depend on the database platform. A function is generally designed to return a value or table and can often be used within a query, subject to platform rules. I choose based on the database, reuse needs, transaction behavior, and whether the logic belongs in the database or application layer."

### 25. Clustered vs non-clustered index

"A clustered index defines how the table's rows are organized at the leaf level of the index; a table can have only one clustered index. A non-clustered index is a separate structure containing indexed key values and a row locator, and a table can have multiple non-clustered indexes. Both can speed up reads, but they consume storage and add work to inserts, updates, and deletes."

### 26. How does indexing improve query performance?

"An index gives the database an efficient path to locate rows by indexed columns, often avoiding a full table scan. Composite indexes can help queries that filter or sort on their leading columns, and included columns may let an index cover a query. Indexes are not automatically beneficial: poor or excessive indexes use storage and slow writes. I use the execution plan and workload measurements to choose and validate indexes."

## Angular

### 27. How do you implement routing in Angular?

"I define routes that map URL paths to components, then render the active route through `router-outlet`. I navigate with `routerLink` or the Angular `Router`, and can use route parameters, query parameters, lazy-loaded feature areas, and guards where appropriate. Guards improve navigation behavior, but authorization must still be enforced by the API."

Example:

```typescript
export const routes: Routes = [
  { path: 'employees', component: EmployeeListComponent },
  { path: 'employees/:id', component: EmployeeDetailsComponent },
  { path: '', redirectTo: 'employees', pathMatch: 'full' }
];
```

### 28. If an API returns an array of employees, how would you display it in a table?

"I define an `Employee` TypeScript type, call the API through an Angular service using `HttpClient`, and subscribe to the observable through the component's supported reactive pattern. The template iterates over the employees and binds each property to a table cell. I also handle loading, empty, and error states, and use a stable employee ID as the row track key."

Example template:

```html
<table>
  <thead>
    <tr><th>Name</th><th>Email</th><th>Department</th></tr>
  </thead>
  <tbody>
    @for (employee of employees; track employee.id) {
      <tr>
        <td>{{ employee.name }}</td>
        <td>{{ employee.email }}</td>
        <td>{{ employee.department }}</td>
      </tr>
    }
  </tbody>
</table>
```

### 29. What are `@Input()` and `@Output()`?

"They are component communication mechanisms. `@Input()` passes data from a parent component to a child. `@Output()` exposes an event from a child to its parent, commonly through an `EventEmitter`. In current Angular versions, signal-based `input()` and `output()` APIs are also available; I use the convention already established in the project."

### 30. How much UI work did you handle in your project?

"I handled **[honest scope, for example: building complete Angular screens / implementing forms and validation / connecting existing screens to APIs / mainly backend work with occasional UI fixes]**. Specifically, I worked on **[actual components, routing, responsive behavior, accessibility, validation, or styling]**. I collaborated with **[designer/frontend teammate]** for **[shared work]**."

## AI Use and Emerging Areas

### 31. How much AI are you using to complete work and write code?

"I use AI tools as an assistant for **[truthful uses, such as exploring unfamiliar code, generating test ideas, drafting boilerplate, or explaining errors]**. I remain responsible for the solution: I verify suggestions against the requirements and codebase, review security and edge cases, run tests, and follow company rules about confidential data and approved tools. I do not treat generated code as automatically correct, and I can explain and maintain the code I submit."

Add a real example of when you accepted, changed, or rejected an AI suggestion. Be clear about the tools and frequency you actually use.

### 32. Any exposure to AI agents, RAG applications, and model deployment?

**If you have hands-on experience:** "I worked on **[agent/RAG/model deployment project]**. The system **[briefly describe its purpose and architecture]**. My contribution was **[your specific work]**. For RAG, the flow was **[document ingestion, chunking, embeddings, vector search, retrieval, prompt construction, model response, evaluation]**. For deployment, I handled **[model serving, endpoint/container/cloud platform, monitoring, scaling, or security]**."

**If your exposure is learning or prototype-only:** "I have learning or prototype exposure, but I have not yet delivered a production AI agent or deployed a model. I understand the basic RAG flow: ingest and chunk source documents, create embeddings, retrieve relevant passages for a query, provide that context to a language model, and evaluate the answer for relevance and grounding. An AI agent adds a control loop in which a model can select tools and take bounded actions. Production deployment also requires evaluation, access control, observability, cost and latency controls, and protection against prompt injection and data leakage. My hands-on work so far is **[truthful example, course, or prototype]**."

Do not conflate calling a hosted model API with deploying or operating a model.
