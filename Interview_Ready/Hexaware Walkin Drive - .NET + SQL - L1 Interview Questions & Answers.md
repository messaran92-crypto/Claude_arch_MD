# Hexaware Walkin Drive - .NET + SQL - L1 Interview Questions & Answers

## Overview

This file contains practical interview questions and answers for a .NET + SQL L1 round, especially suited for a Hexaware walk-in drive style interview. The focus is on core .NET fundamentals, SQL, and integration concepts.

---

## 1) What are reflections in C#?

### Answer
Reflection in C# is the ability of a program to inspect and work with metadata about types, methods, properties, constructors, and assemblies at runtime.

It is provided by the `System.Reflection` namespace.

### Why it is useful
- Dynamically loading assemblies
- Creating objects without compile-time type information
- Reading metadata of classes and methods
- Building plugins or frameworks

### Example

```csharp
using System;
using System.Reflection;

class Program
{
    static void Main()
    {
        Type t = typeof(string);
        Console.WriteLine(t.FullName);
        Console.WriteLine(t.GetMethods().Length);
    }
}
```

### Interview note
Reflection is powerful but slower than direct compiled access, so it should be used carefully.

---

## 2) What is a sealed class?

### Answer
A sealed class is a class that cannot be inherited.

### Syntax

```csharp
public sealed class Employee
{
    public int Id { get; set; }
}
```

### Why use it?
- Prevent further extension of a class
- Improve security and design control
- Avoid unintended inheritance bugs

### Example

```csharp
public class Manager : Employee
{
}
```

This will fail because `Employee` is sealed.

---

## 3) Difference between static class and singleton

### Answer
A static class is a class that cannot be instantiated and contains only static members.
A singleton is a class that allows only one instance at runtime, but it is still an instance-based class.

### Static class

```csharp
public static class MathHelper
{
    public static int Add(int a, int b)
    {
        return a + b;
    }
}
```

### Singleton

```csharp
public sealed class DatabaseConnection
{
    private static DatabaseConnection? instance;
    private static readonly object lockObj = new();

    private DatabaseConnection() { }

    public static DatabaseConnection Instance
    {
        get
        {
            lock (lockObj)
            {
                instance ??= new DatabaseConnection();
                return instance;
            }
        }
    }
}
```

### Key differences
- Static class cannot be instantiated; singleton can be instantiated once.
- Singleton can implement interfaces and support inheritance (not usually for sealed)
- Static class is simpler for utility methods.
- Singleton is better when you need one shared object with instance state.

---

## 4) How to implement Data API integration?

### Answer
Data API integration usually means connecting your application to a database or external data source through a structured layer.

Typical approach:
- Use repository/service layer
- Add connection string/configuration
- Use ADO.NET / Entity Framework / Dapper
- Implement CRUD logic
- Validate input and handle exceptions

### Example with ADO.NET

```csharp
using System.Data.SqlClient;

public class EmployeeRepository
{
    private readonly string _connectionString;

    public EmployeeRepository(string connectionString)
    {
        _connectionString = connectionString;
    }

    public void GetEmployee(int id)
    {
        using var connection = new SqlConnection(_connectionString);
        connection.Open();

        var query = "SELECT * FROM Employees WHERE Id = @Id";
        using var command = new SqlCommand(query, connection);
        command.Parameters.AddWithValue("@Id", id);

        using var reader = command.ExecuteReader();
        while (reader.Read())
        {
            Console.WriteLine(reader["Name"]);
        }
    }
}
```

### Interview note
A good architecture is:
- Controller -> Service -> Repository -> Database

This keeps code clean, testable, and maintainable.

---

## 5) Reverse the hello world program

### Answer
A reverse hello world program prints the text in reverse order.

### Example

```csharp
using System;

class Program
{
    static void Main()
    {
        string str = "hello world";
        char[] chars = str.ToCharArray();
        Array.Reverse(chars);
        Console.WriteLine(new string(chars));
    }
}
```

### Output

```text
dlrow olleh
```

### Alternate version using loop

```csharp
string s = "hello world";
for (int i = s.Length - 1; i >= 0; i--)
{
    Console.Write(s[i]);
}
```

---

## 6) What are triggers?

### Answer
A trigger is a special database object that automatically executes when a specific event occurs such as:
- INSERT
- UPDATE
- DELETE

### Example

```sql
CREATE TRIGGER trgAfterInsert
ON Employees
AFTER INSERT
AS
BEGIN
    INSERT INTO AuditLog (Message, CreatedAt)
    VALUES ('New employee inserted', GETDATE());
END;
```

### Use cases
- Auditing changes
- Enforcing business rules
- Maintaining logs

### Interview note
Triggers are useful, but they can also introduce complexity, so they should be used carefully.

---

## 7) What is ACID?

### Answer
ACID is a set of properties that guarantee database transaction reliability.

### Acronym
- A = Atomicity
- C = Consistency
- I = Isolation
- D = Durability

### Meaning
- Atomicity: all steps in a transaction succeed or none do
- Consistency: database moves from one valid state to another
- Isolation: transactions do not interfere with each other
- Durability: once committed, data persists even after failure

### Example

```sql
BEGIN TRANSACTION;

UPDATE Accounts SET Balance = Balance - 100 WHERE Id = 1;
UPDATE Accounts SET Balance = Balance + 100 WHERE Id = 2;

COMMIT;
```

If either update fails, the transaction is rolled back.

---

## 8) Select and Select Many

### Answer
These are LINQ methods used with collections.

### Select
Returns one element for every source element.

```csharp
var numbers = new[] { 1, 2, 3, 4 };
var squares = numbers.Select(n => n * n).ToList();
```

Output:
```text
1, 4, 9, 16
```

### SelectMany
Flattens a sequence of collections into one collection.

```csharp
var groups = new[]
{
    new[] { 1, 2 },
    new[] { 3, 4 },
    new[] { 5 }
};

var flattened = groups.SelectMany(x => x).ToList();
```

Output:
```text
1, 2, 3, 4, 5
```

### Interview note
Use `Select` when you want one output per input item, and `SelectMany` when you want to flatten nested data.

---

## 9) Difference between DELETE, TRUNCATE, and DROP

### Answer
These are SQL commands with different effects.

### DELETE
- Removes rows one by one
- Can use WHERE clause
- Logged row by row
- Slower than TRUNCATE

```sql
DELETE FROM Employees WHERE Id = 5;
```

### TRUNCATE
- Removes all rows from a table quickly
- Cannot use WHERE clause
- Resets identity values in many databases
- Faster than DELETE

```sql
TRUNCATE TABLE Employees;
```

### DROP
- Removes the whole table structure permanently
- Deletes schema and data
- Cannot be rolled back easily in some systems

```sql
DROP TABLE Employees;
```

### Summary
- DELETE = remove specific rows
- TRUNCATE = remove all rows
- DROP = delete the entire table

---

## 10) Cross join and inner join

### Inner Join
Returns only matching rows present in both tables.

```sql
SELECT e.Name, d.DepartmentName
FROM Employees e
INNER JOIN Departments d ON e.DepartmentId = d.Id;
```

### Cross Join
Returns every row from the first table combined with every row from the second table.

```sql
SELECT e.Name, d.DepartmentName
FROM Employees e
CROSS JOIN Departments d;
```

### Example
If Employees has 3 rows and Departments has 2 rows:
- Inner join may return only matching 2 rows
- Cross join returns 6 combinations

### Interview note
Use `INNER JOIN` for logical relationships and `CROSS JOIN` only when you intentionally want all combinations.

---

## 11) Async and Await

### Answer
`async` and `await` are used to write asynchronous code in C#.

### Example

```csharp
public async Task<string> GetDataAsync()
{
    await Task.Delay(1000);
    return "Data loaded";
}
```

### Why it is important
- Prevents blocking the main thread
- Improves responsiveness in UI and web apps
- Helps handle I/O-heavy operations

### Example with HttpClient

```csharp
using System.Net.Http;

public async Task<string> FetchDataAsync()
{
    using var client = new HttpClient();
    return await client.GetStringAsync("https://example.com");
}
```

### Interview note
`await` waits asynchronously without blocking the thread, which is essential in scalable applications.

---

## 12) WHERE and HAVING clause

### WHERE
Filters rows before grouping.

```sql
SELECT *
FROM Employees
WHERE Salary > 50000;
```

### HAVING
Filters groups after aggregation.

```sql
SELECT DepartmentId, COUNT(*) AS TotalEmployees
FROM Employees
GROUP BY DepartmentId
HAVING COUNT(*) > 2;
```

### Key difference
- `WHERE` works on rows
- `HAVING` works on grouped results

---

## 13) Customer middleware in .NET Core

### Answer
Middleware is a software component in the ASP.NET Core request pipeline.

### Example

```csharp
app.Use(async (context, next) =>
{
    Console.WriteLine("Request started");
    await next();
    Console.WriteLine("Request ended");
});
```

### Custom middleware use cases
- Logging
- Authentication
- Request validation
- Exception handling
- Performance tracking

### Interview note
Middleware runs in order. Each middleware can decide whether to continue or short-circuit the request.

---

## 14) Delegate and types

### Answer
A delegate is a type that represents references to methods.

### Example

```csharp
public delegate int Calculator(int x, int y);

class Program
{
    static int Add(int x, int y) => x + y;
    static int Multiply(int x, int y) => x * y;

    static void Main()
    {
        Calculator calc = Add;
        Console.WriteLine(calc(5, 3));

        calc = Multiply;
        Console.WriteLine(calc(5, 3));
    }
}
```

### Types of delegates
- Single-cast delegate: points to one method
- Multicast delegate: points to multiple methods using `+`

### Example of multicast

```csharp
Calculator calc = Add;
calc += Multiply;
```

This is useful for event handling and notification patterns.

---

## Quick Revision Summary

### .NET core concepts
- Reflection
- Sealed class
- Static class vs singleton
- Async/await
- Middleware
- Delegates

### SQL concepts
- DELETE vs TRUNCATE vs DROP
- INNER join vs CROSS join
- WHERE vs HAVING
- Triggers
- ACID
- Select vs SelectMany

### Interview tip
For L1 interviews, keep the answers short, clear, and with one practical example. Most questions are checking your core understanding, not advanced theory.

---

## Final Interview Practice

Try answering these in your own words:

1. What is reflection and where is it used?
2. Why would you mark a class as sealed?
3. When do you use static class vs singleton?
4. How do you integrate database logic in .NET?
5. What is the use of a trigger?
6. Explain ACID with a simple example.
7. What is the difference between `WHERE` and `HAVING`?
8. Why is `await` useful in ASP.NET Core and UI applications?
9. What is middleware and why is it important?
10. What is the difference between `delete`, `truncate`, and `drop`?
