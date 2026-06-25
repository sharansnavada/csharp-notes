# LINQ (Language Integrated Query)

## What is LINQ?

LINQ stands for **Language Integrated Query**. It gives you a built-in way to search, filter, sort, and transform lists of data — directly inside C# — without writing long manual loops.

You can think of it like asking questions about your data:
- "Give me only the items where price is below 500"
- "Sort these players by score, highest first"
- "Does anyone in this list have admin access?"

LINQ answers all of those in one clean line of code.

---

## Why Should You Care?

Here's the same task done two ways — finding all even numbers from a list:

**Without LINQ (the old way):**
```csharp
List<int> numbers = new List<int> { 1, 2, 3, 4, 5, 6, 7, 8, 9, 10 };
List<int> evenNumbers = new List<int>();

for (int i = 0; i < numbers.Count; i++)
{
    if (numbers[i] % 2 == 0)
    {
        evenNumbers.Add(numbers[i]);
    }
}
```

**With LINQ:**
```csharp
List<int> numbers = new List<int> { 1, 2, 3, 4, 5, 6, 7, 8, 9, 10 };
List<int> evenNumbers = numbers.Where(n => n % 2 == 0).ToList();
```

Same result. One line instead of six. This is what LINQ is for.

---

## Setup: Add This Line at the Top

Before using LINQ, add this import at the top of your `.cs` file:

```csharp
using System.Linq;
```

Without it, LINQ methods like `Where`, `Select`, `OrderBy` etc. won't be available.

---

## Understanding the Arrow Syntax (Lambda)

Almost every LINQ method uses an arrow `=>` (called a **lambda expression**). Before going further, you need to understand this clearly.

```csharp
n => n % 2 == 0
```

Read this as: **"for each item (called n), check if n % 2 == 0"**

- The left side (`n`) is just a temporary name you pick — it represents each item in the list, one at a time
- The `=>` means "goes into" or "for which"
- The right side is what you want to do or check with that item

You can name it anything — `n`, `item`, `x`, `product` — whatever makes sense:

```csharp
numbers.Where(n => n > 5)           // n is each number
names.Where(name => name.Length > 3) // name is each string
products.Where(p => p.Price < 500)   // p is each product object
```

All three are the same concept — just with different variable names on the left side.

---

## Two Ways to Write LINQ

LINQ has two syntaxes. Both produce the same result. You only need to learn one — most developers use **Method Syntax**.

**Method Syntax** (what this guide uses):
```csharp
var result = numbers.Where(n => n > 5).ToList();
```

**Query Syntax** (looks like SQL):
```csharp
var result = (from n in numbers
              where n > 5
              select n).ToList();
```

Stick with method syntax. It's cleaner, shorter, and what you'll see in real codebases.

---

## Core LINQ Methods

### `Where` — Filter items

Keeps only the items where your condition is true. Everything else is removed.

```csharp
List<int> scores = new List<int> { 45, 72, 88, 91, 33, 60 };

var passingScores = scores.Where(s => s >= 60).ToList();
// Result: [72, 88, 91, 60]
```

LINQ goes through each item in `scores`, checks if it's >= 60, and keeps it if true.

**Real-world uses:**
- Get only active users from a user list
- Get only in-stock products
- Get only unpaid invoices

---

### `Select` — Pick or transform items

`Select` lets you either **pick specific fields** from objects, or **transform** values into something new.

**Transforming values:**
```csharp
List<int> prices = new List<int> { 100, 200, 300 };

// Apply 10% discount to all prices
var discounted = prices.Select(p => p * 0.9).ToList();
// Result: [90.0, 180.0, 270.0]
```

**Picking a specific field from objects** (this is the most common real-world use):
```csharp
public class User
{
    public string Name { get; set; }
    public string Email { get; set; }
    public int Age { get; set; }
}

List<User> users = new List<User>
{
    new User { Name = "Alice", Email = "alice@email.com", Age = 25 },
    new User { Name = "Bob",   Email = "bob@email.com",   Age = 30 },
    new User { Name = "Carol", Email = "carol@email.com", Age = 22 },
};

// Get just the email addresses (not the full User objects)
var emails = users.Select(u => u.Email).ToList();
// Result: ["alice@email.com", "bob@email.com", "carol@email.com"]

// Get just the names
var names = users.Select(u => u.Name).ToList();
// Result: ["Alice", "Bob", "Carol"]
```

Think of `Select` as saying: "from each item, give me only this part."

---

### `OrderBy` / `OrderByDescending` — Sort items

```csharp
List<int> prices = new List<int> { 500, 150, 800, 200 };

var cheapFirst = prices.OrderBy(p => p).ToList();
// Result: [150, 200, 500, 800]

var expensiveFirst = prices.OrderByDescending(p => p).ToList();
// Result: [800, 500, 200, 150]
```

Sorting a list of objects by a specific field:
```csharp
// Sort users alphabetically by name
var sorted = users.OrderBy(u => u.Name).ToList();

// Sort users from youngest to oldest
var byAge = users.OrderBy(u => u.Age).ToList();
```

**Secondary sort with `ThenBy`:**

When two items tie on the first sort, `ThenBy` decides the order between them:

```csharp
// Sort by last name, and if last name is the same, sort by first name
var sorted = employees
    .OrderBy(e => e.LastName)
    .ThenBy(e => e.FirstName)
    .ToList();
```

---

### `First` / `FirstOrDefault` — Get a single item

`First` returns the first item that matches your condition.

```csharp
List<int> numbers = new List<int> { 3, 7, 2, 9, 1 };

int firstBig = numbers.First(n => n > 5);
// Result: 7
```

**Important:** If no item matches, `First` throws a crash (exception). Use `FirstOrDefault` to be safe:

```csharp
int result = numbers.FirstOrDefault(n => n > 100);
// Result: 0  (no crash — returns the default value for int)
```

What `FirstOrDefault` returns when nothing is found depends on the type:
- For `int` → returns `0`
- For `string` → returns `null`
- For a class (like `User`) → returns `null`

```csharp
// Searching for a user — safe approach
User found = users.FirstOrDefault(u => u.Name == "Bob");

if (found != null)
{
    Console.WriteLine(found.Email); // only runs if user was found
}
```

**Rule of thumb:** Always use `FirstOrDefault` when there's a chance the item might not exist.

---

### `Any` — Check if anything matches

Returns `true` if at least one item passes the condition. Returns `false` if nothing does.

```csharp
List<string> cart = new List<string> { "Laptop", "Mouse", "Keyboard" };

bool hasLaptop = cart.Any(item => item == "Laptop");  // true
bool hasPhone  = cart.Any(item => item == "Phone");   // false
```

You can also call `Any()` with no condition to just check if the list is not empty:

```csharp
bool hasItems = cart.Any(); // true (cart has 3 items)
```

**Real-world uses:**
- Does this user have admin role?
- Is there at least one item in the shopping cart?
- Does any order have a problem?

---

### `All` — Check if everything matches

Returns `true` only if **every single item** in the list passes the condition.

```csharp
List<int> ages = new List<int> { 18, 21, 25, 30 };

bool allAdults = ages.All(a => a >= 18); // true

List<int> mixed = new List<int> { 17, 21, 25 };
bool allAdults2 = mixed.All(a => a >= 18); // false — 17 fails
```

**Real-world uses:**
- Are all required form fields filled?
- Have all items in the order been shipped?
- Do all players meet the minimum level requirement?

---

### `Count` — Count how many match

Without a condition, returns the total number of items. With a condition, counts only the matches.

```csharp
List<string> fruits = new List<string> { "Apple", "Banana", "Avocado", "Mango" };

int total  = fruits.Count();                        // 4
int aCount = fruits.Count(f => f.StartsWith("A"));  // 2  (Apple, Avocado)
```

---

### `Sum` / `Min` / `Max` / `Average` — Aggregate numbers

These work on numbers and let you compute totals, ranges, and averages.

```csharp
List<int> salaries = new List<int> { 30000, 45000, 60000, 35000 };

int    total   = salaries.Sum();      // 170000
int    lowest  = salaries.Min();      // 30000
int    highest = salaries.Max();      // 60000
double average = salaries.Average();  // 42500.0
```

On a list of objects, pass a lambda to tell LINQ which field to calculate on:

```csharp
// Total revenue from all delivered orders
decimal totalRevenue = orders.Where(o => o.Status == "Delivered").Sum(o => o.Amount);

// Most expensive product price
decimal maxPrice = products.Max(p => p.Price);
```

---

### `Distinct` — Remove duplicates

```csharp
List<string> tags = new List<string> { "C#", "Unity", "C#", "Python", "Unity" };

var uniqueTags = tags.Distinct().ToList();
// Result: ["C#", "Unity", "Python"]
```

**Real-world use:** A user searched for the same city twice — remove duplicate results before displaying.

---

### `Take` / `Skip` — Get a portion of a list

`Take(n)` returns the first `n` items. `Skip(n)` jumps over the first `n` items.

Combined, they give you pagination (showing results page by page):

```csharp
List<int> items = new List<int> { 1, 2, 3, 4, 5, 6, 7, 8, 9, 10 };

var page1 = items.Take(3).ToList();         // [1, 2, 3]
var page2 = items.Skip(3).Take(3).ToList(); // [4, 5, 6]
var page3 = items.Skip(6).Take(3).ToList(); // [7, 8, 9]
```

**Real-world use:** "Show 10 products per page. User clicked page 3 → skip the first 20, take the next 10."

---

### `GroupBy` — Group items by a shared property

`GroupBy` splits your list into groups based on a field. Each group has a `Key` (what they were grouped by) and a collection of matching items.

Think of it like sorting a pile of papers into labelled folders.

```csharp
public class Employee
{
    public string Name { get; set; }
    public string Department { get; set; }
}

List<Employee> employees = new List<Employee>
{
    new Employee { Name = "Alice", Department = "Engineering" },
    new Employee { Name = "Bob",   Department = "Marketing"   },
    new Employee { Name = "Carol", Department = "Engineering" },
    new Employee { Name = "Dave",  Department = "Marketing"   },
    new Employee { Name = "Eve",   Department = "Engineering" },
};

var byDepartment = employees.GroupBy(e => e.Department).ToList();

for (int i = 0; i < byDepartment.Count; i++)
{
    var group = byDepartment[i];
    Console.WriteLine($"Department: {group.Key}");  // group.Key = "Engineering" or "Marketing"

    for (int j = 0; j < group.Count(); j++)
    {
        Console.WriteLine($"  - {group.ElementAt(j).Name}");
    }
}

// Output:
// Department: Engineering
//   - Alice
//   - Carol
//   - Eve
// Department: Marketing
//   - Bob
//   - Dave
```

`group.Key` is the value they all share (the department name). The group itself is the list of employees in that department.

**Real-world uses:**
- Group orders by status: Pending / Shipped / Delivered
- Group students by grade: A / B / C
- Group transactions by month for a monthly report

---

### `Contains` — Check if a specific value exists in a list

```csharp
List<int> allowedUserIds = new List<int> { 1, 2, 3, 5, 8 };

bool isAllowed = allowedUserIds.Contains(3);  // true
bool isBlocked = allowedUserIds.Contains(7);  // false
```

**Real-world use:** You have a list of blocked countries. Check if the user's country is in that list before showing content.

```csharp
List<string> blockedCountries = new List<string> { "CountryA", "CountryB" };
bool isBlocked = blockedCountries.Contains(user.Country);
```

---

## Chaining LINQ Methods

The real power of LINQ is that you can **chain** methods together — each method passes its result to the next one.

```csharp
List<int> numbers = new List<int> { 1, 2, 3, 4, 5, 6, 7, 8, 9, 10 };

var result = numbers
    .Where(n => n % 2 == 0)       // Step 1: keep evens → [2, 4, 6, 8, 10]
    .Select(n => n * 2)            // Step 2: double each → [4, 8, 12, 16, 20]
    .OrderByDescending(n => n)     // Step 3: sort desc  → [20, 16, 12, 8, 4]
    .Take(3)                       // Step 4: take first 3 → [20, 16, 12]
    .ToList();                     // Step 5: convert to List
```

Read it top to bottom — each line feeds into the next. The comments show what the data looks like at each step.

---

## LINQ on Objects — The Real-World Way

In real projects, you almost never run LINQ on a plain `List<int>`. You run it on lists of your own objects. Here's a complete example:

```csharp
public class Product
{
    public string Name { get; set; }
    public string Category { get; set; }
    public decimal Price { get; set; }
    public bool InStock { get; set; }
}

List<Product> products = new List<Product>
{
    new Product { Name = "Laptop",  Category = "Electronics", Price = 75000, InStock = true  },
    new Product { Name = "Phone",   Category = "Electronics", Price = 45000, InStock = false },
    new Product { Name = "Desk",    Category = "Furniture",   Price = 12000, InStock = true  },
    new Product { Name = "Chair",   Category = "Furniture",   Price = 8000,  InStock = true  },
    new Product { Name = "Headset", Category = "Electronics", Price = 3500,  InStock = true  },
};

// In-stock electronics under ₹50,000, sorted cheapest first
var affordable = products
    .Where(p => p.Category == "Electronics" && p.InStock && p.Price < 50000)
    .OrderBy(p => p.Price)
    .ToList();

for (int i = 0; i < affordable.Count; i++)
{
    Console.WriteLine($"{affordable[i].Name} - ₹{affordable[i].Price}");
}
// Output: Headset - ₹3500

// Just the product names
var names = products.Select(p => p.Name).ToList();
// ["Laptop", "Phone", "Desk", "Chair", "Headset"]

// Total value of all in-stock items
decimal totalValue = products.Where(p => p.InStock).Sum(p => p.Price);
// ₹98,500

// The single most expensive item
Product mostExpensive = products.OrderByDescending(p => p.Price).First();
Console.WriteLine(mostExpensive.Name); // Laptop

// Are all items in stock?
bool allAvailable = products.All(p => p.InStock); // false (Phone is not)

// Group by category and count
var categoryGroups = products.GroupBy(p => p.Category).ToList();

for (int i = 0; i < categoryGroups.Count; i++)
{
    var group = categoryGroups[i];
    Console.WriteLine($"{group.Key}: {group.Count()} products");
}
// Electronics: 3 products
// Furniture: 2 products
```

---

## Complete Runnable Example (Console App)

Paste this into a new Console App and run it to see everything in action:

```csharp
using System;
using System.Collections.Generic;
using System.Linq;

class Program
{
    public class Student
    {
        public string Name  { get; set; }
        public string Grade { get; set; }  // "A", "B", "C"
        public int    Score { get; set; }
    }

    static void Main()
    {
        List<Student> students = new List<Student>
        {
            new Student { Name = "Alice",   Grade = "A", Score = 95 },
            new Student { Name = "Bob",     Grade = "B", Score = 72 },
            new Student { Name = "Carol",   Grade = "A", Score = 88 },
            new Student { Name = "Dave",    Grade = "C", Score = 55 },
            new Student { Name = "Eve",     Grade = "B", Score = 78 },
            new Student { Name = "Frank",   Grade = "C", Score = 61 },
            new Student { Name = "Grace",   Grade = "A", Score = 92 },
        };

        // 1. Students scoring above 80
        var highScorers = students.Where(s => s.Score > 80).ToList();
        Console.WriteLine("--- Scoring above 80 ---");
        for (int i = 0; i < highScorers.Count; i++)
        {
            Console.WriteLine($"{highScorers[i].Name}: {highScorers[i].Score}");
        }

        // 2. Top 3 scorers
        var top3 = students.OrderByDescending(s => s.Score).Take(3).ToList();
        Console.WriteLine("\n--- Top 3 Scorers ---");
        for (int i = 0; i < top3.Count; i++)
        {
            Console.WriteLine($"{i + 1}. {top3[i].Name} - {top3[i].Score}");
        }

        // 3. Average score
        double avg = students.Average(s => s.Score);
        Console.WriteLine($"\n--- Class Average: {avg:F1} ---");

        // 4. Did anyone score 100?
        bool perfectScore = students.Any(s => s.Score == 100);
        Console.WriteLine($"\n--- Anyone scored 100? {perfectScore} ---");

        // 5. Students grouped by grade
        var byGrade = students.GroupBy(s => s.Grade).ToList();
        Console.WriteLine("\n--- Students by Grade ---");
        for (int i = 0; i < byGrade.Count; i++)
        {
            var group = byGrade[i];
            Console.WriteLine($"Grade {group.Key}:");
            for (int j = 0; j < group.Count(); j++)
            {
                Console.WriteLine($"  {group.ElementAt(j).Name}");
            }
        }
    }
}
```

**Expected Output:**
```
--- Scoring above 80 ---
Alice: 95
Carol: 88
Grace: 92

--- Top 3 Scorers ---
1. Alice - 95
2. Grace - 92
3. Carol - 88

--- Class Average: 77.3 ---

--- Anyone scored 100? False ---

--- Students by Grade ---
Grade A:
  Alice
  Carol
  Grace
Grade B:
  Bob
  Eve
Grade C:
  Dave
  Frank
```

---

## Real-World Scenarios

### Scenario 1: Cable TV Billing System

```csharp
public class Subscriber
{
    public string   Name      { get; set; }
    public string   Plan      { get; set; }  // "Basic", "Standard", "Premium"
    public bool     IsPaid    { get; set; }
    public DateTime DueDate   { get; set; }
    public string   Phone     { get; set; }
}

List<Subscriber> subscribers = GetAllSubscribers();

// 1. Who hasn't paid and is overdue?
var overdue = subscribers
    .Where(s => !s.IsPaid && s.DueDate < DateTime.Today)
    .OrderBy(s => s.DueDate)
    .ToList();

Console.WriteLine($"Overdue count: {overdue.Count}");

// 2. How many subscribers are on each plan?
var planGroups = subscribers.GroupBy(s => s.Plan).ToList();

for (int i = 0; i < planGroups.Count; i++)
{
    var group = planGroups[i];
    Console.WriteLine($"{group.Key}: {group.Count()} subscribers");
}

// 3. Get phone numbers of everyone who hasn't paid (for sending reminders)
var unpaidPhones = subscribers
    .Where(s => !s.IsPaid)
    .Select(s => s.Phone)
    .ToList();

// 4. Are all Premium subscribers paid up?
bool premiumAllPaid = subscribers
    .Where(s => s.Plan == "Premium")
    .All(s => s.IsPaid);

// 5. Total subscribers on each plan — quick count
int premiumCount  = subscribers.Count(s => s.Plan == "Premium");
int standardCount = subscribers.Count(s => s.Plan == "Standard");
int basicCount    = subscribers.Count(s => s.Plan == "Basic");
```

---

### Scenario 2: Game Leaderboard (Unity)

```csharp
public class Player
{
    public string Name     { get; set; }
    public int    Score    { get; set; }
    public int    Level    { get; set; }
    public bool   IsOnline { get; set; }
}

List<Player> players = GetAllPlayers();

// 1. Top 5 for the leaderboard
var leaderboard = players
    .OrderByDescending(p => p.Score)
    .Take(5)
    .ToList();

// 2. Average score of experienced players (level 10+)
double avgHighLevel = players
    .Where(p => p.Level >= 10)
    .Average(p => p.Score);

// 3. Currently online players
var onlinePlayers = players.Where(p => p.IsOnline).ToList();

// 4. Has anyone reached the max level?
bool maxLevelReached = players.Any(p => p.Level == 50);

// 5. Highest score ever recorded
int topScore = players.Max(p => p.Score);
```

---

### Scenario 3: E-Commerce Order Dashboard

```csharp
public class Order
{
    public int      OrderId      { get; set; }
    public string   CustomerName { get; set; }
    public string   Status       { get; set; }  // "Pending", "Shipped", "Delivered"
    public decimal  Amount       { get; set; }
    public DateTime OrderDate    { get; set; }
}

List<Order> orders = GetOrders();

// 1. Today's revenue from delivered orders
decimal todayRevenue = orders
    .Where(o => o.Status == "Delivered" && o.OrderDate.Date == DateTime.Today)
    .Sum(o => o.Amount);

// 2. Count of orders in each status (for dashboard tiles)
var statusGroups = orders.GroupBy(o => o.Status).ToList();

for (int i = 0; i < statusGroups.Count; i++)
{
    var group = statusGroups[i];
    Console.WriteLine($"{group.Key}: {group.Count()} orders");
}

// 3. Pending orders stuck for more than 2 days (alert needed)
var staleOrders = orders
    .Where(o => o.Status == "Pending" && o.OrderDate < DateTime.Today.AddDays(-2))
    .OrderBy(o => o.OrderDate)
    .ToList();

// 4. Top 3 highest-value orders today
var bigOrdersToday = orders
    .Where(o => o.OrderDate.Date == DateTime.Today)
    .OrderByDescending(o => o.Amount)
    .Take(3)
    .ToList();

// 5. Average order value
double avgOrderValue = orders.Average(o => (double)o.Amount);
```

---

## Deferred Execution — Why Your Query Isn't Running Yet

When you write a LINQ query, it doesn't run immediately. It runs only when you actually access the results (by calling `.ToList()`, looping over it, etc.). This is called **deferred execution**.

**Where this can bite you:**

```csharp
List<int> numbers = new List<int> { 1, 2, 3, 4, 5 };

var query = numbers.Where(n => n > 3);  // ← NOT executed here

numbers.Add(10);   // list changes AFTER the query was defined
numbers.Add(11);

var result = query.ToList();  // ← executes HERE, sees the updated list
// Result: [4, 5, 10, 11]  — includes 10 and 11 even though they were added after
```

If you had called `.ToList()` right away, you'd get `[4, 5]` and the added values would not appear in results.

**Safe rule:** Call `.ToList()` immediately after your query if you don't want this behaviour.

---

## `ToList()` vs `ToArray()` — What to Use

| Method | Returns | Use When |
|--------|---------|----------|
| `.ToList()` | `List<T>` | You might need to add/remove items later — most cases |
| `.ToArray()` | `T[]` | You just need a fixed snapshot, slightly faster |
| *(nothing)* | `IEnumerable<T>` | You're looping over results once and don't need a List |

When in doubt: **use `.ToList()`**. It's the most flexible.

---

## Common Mistakes to Avoid

**Mistake 1: Using `First()` when the item might not exist**

```csharp
// DANGEROUS — crashes if no user has Id 999
User user = users.First(u => u.Id == 999);

// SAFE — returns null instead of crashing
User user = users.FirstOrDefault(u => u.Id == 999);
if (user != null)
{
    Console.WriteLine(user.Name);
}
```

**Mistake 2: Forgetting `.ToList()` and confusing yourself**

```csharp
// This is a query definition, not a list — re-runs every time you use it
var result = numbers.Where(n => n > 5);

// This is an actual List — computed once, stays the same
var result = numbers.Where(n => n > 5).ToList();
```

**Mistake 3: Running LINQ on a null list**

```csharp
List<User> users = null;

// This crashes:
var result = users.Where(u => u.IsActive).ToList();

// Always check for null first:
if (users != null)
{
    var result = users.Where(u => u.IsActive).ToList();
}
```

**Mistake 4: Using LINQ for complex step-by-step logic**

LINQ is great for reading/querying data. If your logic involves multiple steps that change data, a regular `for` loop is clearer and easier to debug.

---

## Quick Reference Cheat Sheet

| Method | What it does | Example |
|--------|-------------|---------|
| `Where(x => condition)` | Filter — keep matching items | `.Where(p => p.Price < 500)` |
| `Select(x => value)` | Pick a field or transform each item | `.Select(u => u.Email)` |
| `OrderBy(x => key)` | Sort ascending (A→Z, 0→9) | `.OrderBy(p => p.Name)` |
| `OrderByDescending(x => key)` | Sort descending (Z→A, 9→0) | `.OrderByDescending(p => p.Score)` |
| `ThenBy(x => key)` | Secondary sort (use after OrderBy) | `.ThenBy(e => e.FirstName)` |
| `First(x => condition)` | First match — crashes if none found | `.First(u => u.IsAdmin)` |
| `FirstOrDefault(x => condition)` | First match — returns null if none | `.FirstOrDefault(u => u.Id == id)` |
| `Any(x => condition)` | Does at least one item match? | `.Any(o => o.Status == "Pending")` |
| `All(x => condition)` | Do all items match? | `.All(s => s.IsPaid)` |
| `Count(x => condition)` | Count matching items | `.Count(u => u.IsActive)` |
| `Sum(x => number)` | Add up a field | `.Sum(o => o.Amount)` |
| `Min(x => number)` | Smallest value | `.Min(p => p.Price)` |
| `Max(x => number)` | Largest value | `.Max(p => p.Price)` |
| `Average(x => number)` | Average value | `.Average(s => s.Score)` |
| `Distinct()` | Remove duplicates | `.Distinct()` |
| `Take(n)` | First n items | `.Take(10)` |
| `Skip(n)` | Skip first n items | `.Skip(20)` |
| `GroupBy(x => key)` | Split into groups by a field | `.GroupBy(e => e.Department)` |
| `Contains(value)` | Does the list have this value? | `.Contains(userId)` |
| `ToList()` | Convert result to `List<T>` | `.ToList()` |
| `ToArray()` | Convert result to `T[]` | `.ToArray()` |

---

## Practice Exercise

**Try this on your own to solidify everything:**

Create a `Student` class with `Name` (string), `Grade` (string: "A", "B", or "C"), and `Score` (int). Make a list of 10 students with mixed grades and scores.

Then write LINQ queries to:
1. Get all students who scored above 80
2. Find the top 3 scorers
3. Group students by grade and print each group
4. Calculate the class average score
5. Check if anyone scored a perfect 100
6. Get just the names of students who got grade "A"
7. Find the student with the lowest score (use `OrderBy` + `First`)
8. Count how many students passed (score >= 60)

A solution is available in the `practice/` folder of this repository.

---

You now have everything you need to use LINQ confidently in real C# projects. The key insight is this: whenever you find yourself writing a `for` loop to search, filter, or sort a list, there's almost certainly a LINQ method that does it in one line.
