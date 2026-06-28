# C# APIs: From Zero to Real-World Usage

> **Who this is for:** Someone who knows basic C# (variables, loops, classes,
> methods) but has never touched APIs, HTTP, or async code before.
> Every concept is introduced before it is used. Every code line is explained.
> Nothing is assumed.

---

## Table of Contents

1.  [What is an API?](#1-what-is-an-api)
2.  [How the Web Works — HTTP Basics](#2-how-the-web-works--http-basics)
3.  [JSON — The Language APIs Speak](#3-json--the-language-apis-speak)
4.  [Project Setup — Running the Examples](#4-project-setup--running-the-examples)
5.  [Task — What It Is and Why It Exists](#5-task--what-it-is-and-why-it-exists)
6.  [Async/Await — Waiting Without Freezing](#6-asyncawait--waiting-without-freezing)
7.  [Making Your First GET Request with HttpClient](#7-making-your-first-get-request-with-httpclient)
8.  [Deserializing JSON into C# Objects](#8-deserializing-json-into-c-objects)
9.  [REST CRUD — GET, POST, PUT, DELETE](#9-rest-crud--get-post-put-delete)
10. [Error Handling and Status Codes](#10-error-handling-and-status-codes)
11. [Sending API Keys and Auth Tokens](#11-sending-api-keys-and-auth-tokens)
12. [Building Your Own API with ASP.NET Core](#12-building-your-own-api-with-aspnet-core)
13. [Real-World Example — GitHub Public API](#13-real-world-example--github-public-api)
14. [Common Pitfalls](#14-common-pitfalls)
15. [Practice Exercises](#15-practice-exercises)
16. [Quick Reference Cheat Sheet](#16-quick-reference-cheat-sheet)

---

## 1. What is an API?

### The restaurant analogy

Imagine you walk into a restaurant.

- You are the **client** — you want food.
- The kitchen is the **server** — it has the food and the logic to prepare it.
- You cannot walk into the kitchen directly. Instead there is a **waiter** who
  takes your order, passes it to the kitchen, and brings the food back.

**The waiter is the API.**

An API (Application Programming Interface) is a defined set of rules that lets
two programs talk to each other. You do not need to know how the other program
works internally. You just need to know what to ask for and what you will get
back.

---

### A real example

Say you open a weather app on your phone. Your app does not contain weather
data — it asks a weather service for it. Here is what happens behind the scenes:

```
Your App                   Weather API Server
  |                               |
  | "Give me weather for Delhi"   |
  | ---------------------------> |
  |                               |   (server looks up the data)
  | { "temp": 38, "city":         |
  |   "Delhi", "sunny": true }    |
  | <---------------------------- |
```

Your app sends a **request** to the API. The API sends back a **response**
with the data. Your app just displays it.

---

### What "REST API" means

Most modern APIs follow a style called **REST** (REpresentational State
Transfer). REST is not a programming language or a library — it is a set of
conventions for how to design an API over HTTP.

When people say "API" in web development, they almost always mean a REST API.

Key ideas:
- Communication happens over **HTTP** (the same protocol browsers use)
- You identify data using **URLs** (called endpoints)
- You tell the server what to do using **HTTP methods** (GET, POST, PUT, DELETE)
- Data is usually exchanged as **JSON** text

---

## 2. How the Web Works — HTTP Basics

Before writing any code, you need to understand HTTP, because that is the
foundation all API calls are built on.

### What is HTTP?

HTTP stands for HyperText Transfer Protocol. It is the agreed-upon format for
sending messages between a client (your app) and a server (the API).

Think of HTTP like postal mail:
- Every letter has an **address** (the URL — where to deliver it)
- Every letter has an **action** (what you want done)
- Every reply has a **status** (delivered, not found, access denied)
- Letters can contain a **body** (the actual content/data)

---

### HTTP Methods

HTTP methods describe the **intent** of your request — what you want to do.

| Method | What it means       | Real-world equivalent                     |
|--------|---------------------|-------------------------------------------|
| GET    | Read data           | "Show me my order history"                |
| POST   | Create new data     | "Place a new order"                       |
| PUT    | Replace data        | "Replace my entire delivery address"      |
| PATCH  | Partially update    | "Only change the door number"             |
| DELETE | Remove data         | "Cancel my order"                         |

---

### HTTP Status Codes

The server always replies with a 3-digit **status code** telling you what
happened. You need to know these because your code must handle them.

| Range   | Category     | Examples                                              |
|---------|--------------|-------------------------------------------------------|
| 200–299 | Success      | 200 OK, 201 Created, 204 No Content                   |
| 400–499 | Client error | 400 Bad Request, 401 Unauthorized, 404 Not Found      |
| 500–599 | Server error | 500 Internal Server Error, 503 Service Unavailable    |

The most important ones:

```
200 OK           → Success. Data is in the response body.
201 Created      → Something new was successfully created (after a POST).
204 No Content   → Success, but nothing to return (common after DELETE).
400 Bad Request  → You sent bad data (typo in JSON, missing required field).
401 Unauthorized → You must provide credentials (login, API key).
403 Forbidden    → You are logged in but not allowed to do this.
404 Not Found    → That URL or resource does not exist.
429 Too Many Req → You called the API too many times too fast (rate limited).
500 Server Error → The server itself broke. Not your fault.
```

---

### Anatomy of an HTTP Request

A full HTTP request looks like this. C# will build this for you automatically,
but knowing the parts helps you understand what is happening:

```
GET /users/42 HTTP/1.1          ← method + path + protocol version
Host: api.example.com           ← which server to send this to
Content-Type: application/json  ← the format of the body we are sending
Authorization: Bearer abc123    ← our credentials (API key, login token)
                                ← blank line signals end of headers
{ "key": "value" }              ← body (only sent with POST / PUT / PATCH)
```

---

## 3. JSON — The Language APIs Speak

Almost every REST API uses **JSON** (JavaScript Object Notation) to send and
receive data. Do not let the "JavaScript" in the name confuse you — JSON is
just a universal text format. Any language can read and write it, including C#.

### JSON syntax rules

```json
{
  "name": "Sharan",
  "age": 22,
  "isActive": true,
  "score": 9.5,
  "tags": ["developer", "unity"],
  "address": {
    "city": "Bengaluru",
    "country": "India"
  },
  "phone": null
}
```

Breaking it down:

| JSON syntax    | What it means                         |
|----------------|---------------------------------------|
| `{ }`          | Object — like a C# class instance     |
| `[ ]`          | Array — like a C# `List<T>`           |
| `"key": value` | Property — always double-quoted key   |
| `"hello"`      | String — always double quotes         |
| `42`           | Integer number                        |
| `3.14`         | Decimal number                        |
| `true`/`false` | Boolean                               |
| `null`         | No value                              |

---

### JSON vs C# type mapping

When you receive JSON from an API, you convert it into C# objects. Here is how
the types line up:

| JSON type       | C# type                   |
|-----------------|---------------------------|
| `"hello"`       | `string`                  |
| `42`            | `int` or `long`           |
| `3.14`          | `double` or `float`       |
| `true`/`false`  | `bool`                    |
| `null`          | `null` (nullable type)    |
| `{ }`           | A C# class                |
| `[ ]`           | `List<T>` or array `T[]`  |

---

## 4. Project Setup — Running the Examples

### For the API client examples (sections 7–11 and 13)

These examples are plain console apps. Create one like this:

```bash
dotnet new console -n ApiClient
cd ApiClient
dotnet run
```

Put your code inside `Program.cs`. All the libraries you need
(`System.Net.Http`, `System.Text.Json`) are included with .NET 6+ automatically
— no extra packages to install.

---

### For the "Build Your Own API" examples (section 12)

That section uses ASP.NET Core, which is a different project type from a
console app. A console app runs and exits. A web/API app runs and keeps
listening for requests indefinitely. Create one like this:

```bash
dotnet new web -n MyApi
cd MyApi
dotnet run
```

You will see output like:

```
Now listening on: http://localhost:5000
```

Keep this terminal running. Open a browser to `http://localhost:5000` to see
responses.

---

## 5. Task — What It Is and Why It Exists

Before you can work with APIs you need to understand `Task`, because every
single API call in C# returns one. If you skip this section, the next sections
will feel like magic words you are copying without understanding.

### The problem: waiting

Normally C# runs one line at a time, in order:

```csharp
Console.WriteLine("A");
// ... lots of work ...
Console.WriteLine("B"); // B runs only after all the work above is done
```

This is fine for calculations. But API calls involve **waiting for the
network** — which can take 100ms, 500ms, or several seconds. If C# just
stopped and waited during that time, your entire program would freeze.

In a console app that is annoying. In a UI app it is a disaster — the whole
window would be unresponsive.

---

### What Task solves

A `Task` is C#'s way of representing **work that is happening in the
background** and will finish at some point in the future.

Think of it like a restaurant receipt:

```
You order food → waiter hands you a receipt (a Task)
                → you can do other things while the kitchen cooks
                → when food is ready, you get notified
```

In code:

```csharp
// This line does NOT wait for the request to finish.
// It starts the request and immediately gives you back a Task —
// like a receipt saying "result coming soon".
Task<string> receiptForData = client.GetStringAsync(url);

// At some point later, you can collect the actual result:
string data = receiptForData.Result; // this DOES wait (but we use await instead, explained next)
```

---

### Task vs Task\<T\>

There are two flavors:

```csharp
Task          // represents work with NO return value  (like void, but async)
Task<string>  // represents work that will return a string when done
Task<int>     // represents work that will return an int when done
Task<MyClass> // represents work that will return a MyClass when done
```

Examples:

```csharp
Task<string> t1 = client.GetStringAsync(url);  // will return a string
Task<HttpResponseMessage> t2 = client.GetAsync(url); // will return HttpResponseMessage
```

---

### Why methods return Task

When you write a method that does network work, you mark it `async` and return
`Task` (or `Task<T>`) so the caller knows "this is background work, do not
block while waiting":

```csharp
// Returns Task because it does background work but gives nothing back
async Task SaveToServer(string data)
{
    await client.PostAsync(url, ...);
}

// Returns Task<string> because it does background work AND gives back a string
async Task<string> FetchName()
{
    return await client.GetStringAsync(url);
}
```

You will see `Task` and `Task<T>` everywhere in API code. Now you know what
they are — placeholders for results that are on their way.

---

## 6. Async/Await — Waiting Without Freezing

`async` and `await` are the keywords that make working with Tasks readable and
clean. They turn the "receipt" model into something that looks almost like
normal code.

### What await does

`await` says: **"pause here, let other code run, and resume when this Task
finishes"** — without freezing the program.

```csharp
// WITHOUT await — blocks the thread. The program freezes until the request
// finishes. Bad for UI apps; bad practice in general.
string data = client.GetStringAsync(url).Result; // do NOT do this

// WITH await — the thread is released while waiting.
// The program stays responsive and resumes here when data arrives.
string data = await client.GetStringAsync(url); // correct
```

### The three rules of async/await

**Rule 1 — A method that uses `await` must be marked `async`:**

```csharp
// CORRECT
async Task FetchData()
{
    string data = await client.GetStringAsync(url); // await is fine here
}

// WRONG — won't compile
Task FetchData()
{
    string data = await client.GetStringAsync(url); // ERROR: 'await' in non-async
}
```

**Rule 2 — An `async` method must return `Task`, `Task<T>`, or `void`
(only for event handlers):**

```csharp
async Task DoWork()           { /* no return value */ }
async Task<string> GetName()  { return await ...; }
async void OnButton()         { /* only for UI event handlers */ }
```

**Rule 3 — `async` is contagious.** If a method uses `await`, the method
calling it should also use `await` (and be `async`). This chains upward until
you reach `Main`:

```csharp
// Top of the chain — Main can be async in modern .NET
static async Task Main(string[] args)
{
    string name = await GetUserName();
    Console.WriteLine(name);
}

static async Task<string> GetUserName()
{
    // This is also async because it awaits something
    return await client.GetStringAsync("https://api.example.com/name");
}
```

---

### Top-level statements (shortcut)

In .NET 6+, you can write async code at the top of `Program.cs` without
typing `Main`:

```csharp
// You can write this directly at the top of Program.cs — no class, no Main needed
using System.Net.Http;

HttpClient client = new HttpClient();
string data = await client.GetStringAsync("https://jsonplaceholder.typicode.com/posts/1");
Console.WriteLine(data);
```

The compiler wraps it in `Main` for you automatically. This is the cleanest
way to write short examples.

---

## 7. Making Your First GET Request with HttpClient

`HttpClient` is the built-in C# class for making HTTP calls. It is in the
`System.Net.Http` namespace which is included by default.

### The one important rule about HttpClient

> **Never create `new HttpClient()` inside a loop or a method that is called
> many times.** Each instance holds network resources. If you create hundreds
> of them without disposing them, you run out of network connections (called
> socket exhaustion).

Correct pattern:

```csharp
// Create ONE client at the top level and reuse it everywhere
HttpClient client = new HttpClient();
```

---

### Your first GET request

Create a new console app (`dotnet new console`) and put this in `Program.cs`:

```csharp
using System;
using System.Net.Http;

// One client shared across all requests
HttpClient client = new HttpClient();

// The URL of the API endpoint we want to call
// jsonplaceholder.typicode.com is a free fake API, great for practice
string url = "https://jsonplaceholder.typicode.com/posts/1";

// Send the GET request. await means "wait for the result without freezing"
HttpResponseMessage response = await client.GetAsync(url);

// Read the response body as plain text
string body = await response.Content.ReadAsStringAsync();

// Print the raw JSON we got back
Console.WriteLine(body);
```

**What you will see:**

```json
{
  "userId": 1,
  "id": 1,
  "title": "sunt aut facere repellat provident",
  "body": "quia et suscipit..."
}
```

> **Note:** `jsonplaceholder.typicode.com` is a free, public fake API that
> accepts any request and returns realistic dummy data. Perfect for learning —
> no sign-up, no key needed.

---

### What each line does

```csharp
HttpResponseMessage response = await client.GetAsync(url);
```

- `client.GetAsync(url)` — sends a GET request; returns a `Task<HttpResponseMessage>`
- `await` — waits for the Task to finish and unwraps the result
- `HttpResponseMessage` — holds the status code, headers, and body of the reply

```csharp
string body = await response.Content.ReadAsStringAsync();
```

- `response.Content` — the body part of the response
- `ReadAsStringAsync()` — reads the body as a `string`; also a Task so we await

---

### Shortcut for GET + read body

If you only want the body text and do not care about the status code:

```csharp
// This does GetAsync + ReadAsStringAsync in one step
string body = await client.GetStringAsync(url);
```

---

## 8. Deserializing JSON into C# Objects

Getting raw JSON as a string is a start, but you almost always want to
convert it into a real C# object so you can access fields directly.

**Deserialization** = converting a JSON string into a C# object.
**Serialization** = converting a C# object into a JSON string.

C# has a built-in library for this: `System.Text.Json`.

---

### Step 1 — Create a class that mirrors the JSON

```json
{
  "userId": 1,
  "id": 5,
  "title": "hello world",
  "body": "some content here"
}
```

Write a C# class with matching property names:

```csharp
// Each property corresponds to a key in the JSON above
class Post
{
    public int    UserId { get; set; }
    public int    Id     { get; set; }
    public string Title  { get; set; }
    public string Body   { get; set; }
}
```

> **Where to put this class in your file?**
> In .NET 6+ console apps using top-level statements, class definitions must
> go **at the bottom of Program.cs**, after all the top-level code. The
> compiler requires this — you cannot define a class in the middle of
> top-level statements. All runnable code goes at the top, all class
> definitions go at the bottom.
>
> ```csharp
> // Program.cs layout
>
> // ── top-level code (runs when you start the app) ──
> HttpClient client = new HttpClient();
> string json = await client.GetStringAsync("...");
> Post post = JsonSerializer.Deserialize<Post>(json, options);
> Console.WriteLine(post.Title);
>
> // ── class definitions (always at the bottom) ──
> class Post
> {
>     public int    Id    { get; set; }
>     public string Title { get; set; }
> }
> ```

The property names do not have to match case-exactly — we will handle that
in the next step.

---

### Step 2 — Deserialize

```csharp
using System.Text.Json;

// Fetch the raw JSON string
string json = await client.GetStringAsync("https://jsonplaceholder.typicode.com/posts/5");

// Options: tells the deserializer to ignore casing differences
// "userId" in JSON will match "UserId" in our class, etc.
JsonSerializerOptions options = new JsonSerializerOptions
{
    PropertyNameCaseInsensitive = true
};

// Convert JSON text → Post object
Post post = JsonSerializer.Deserialize<Post>(json, options);

// Now you can use it like any C# object
Console.WriteLine(post.Title);
Console.WriteLine(post.UserId);
```

---

### Step 3 — Deserializing a JSON array

When the API returns a list instead of a single object:

```json
[
  { "id": 1, "title": "First" },
  { "id": 2, "title": "Second" }
]
```

```csharp
string json = await client.GetStringAsync("https://jsonplaceholder.typicode.com/posts");

// Deserialize into a List<Post> instead of a single Post
List<Post> posts = JsonSerializer.Deserialize<List<Post>>(json, options);

Console.WriteLine($"Total posts fetched: {posts.Count}");

for (int i = 0; i < posts.Count; i++)
{
    Console.WriteLine($"{posts[i].Id}: {posts[i].Title}");
}
```

---

### Handling property name mismatches

Sometimes the JSON uses `snake_case` (`first_name`) but your C# class uses
`PascalCase` (`FirstName`). The `PropertyNameCaseInsensitive = true` option
handles simple casing differences (`userId` ↔ `UserId`) but not
snake_case ↔ PascalCase (`first_name` ↔ `FirstName`).

For that, use the `[JsonPropertyName]` attribute. An attribute in C# is an
instruction placed above a property or class inside square brackets `[ ]`. It
gives extra information to the compiler or libraries:

```csharp
using System.Text.Json.Serialization;

class User
{
    // [JsonPropertyName("...")] tells the deserializer:
    // "when you see 'first_name' in JSON, put it into this property"
    [JsonPropertyName("first_name")]
    public string FirstName { get; set; }

    [JsonPropertyName("last_name")]
    public string LastName { get; set; }

    [JsonPropertyName("email_address")]
    public string Email { get; set; }
}
```

---

### Serialization — C# object → JSON string

You need this when sending data to an API (POST, PUT):

```csharp
Post newPost = new Post
{
    UserId = 1,
    Title  = "My new post",
    Body   = "Some content"
};

string json = JsonSerializer.Serialize(newPost);
// Result: {"UserId":1,"Id":0,"Title":"My new post","Body":"Some content"}

Console.WriteLine(json);
```

---

## 9. REST CRUD — GET, POST, PUT, DELETE

CRUD stands for Create, Read, Update, Delete. Every REST API is built around
these four operations. Here is how they map to HTTP methods and C# code.

We will use `https://jsonplaceholder.typicode.com` for all examples — a free
fake API that accepts every request and returns plausible responses.

---

### GET — Read data

**Read one item:**

```csharp
using System;
using System.Net.Http;
using System.Text.Json;

HttpClient client = new HttpClient();

JsonSerializerOptions options = new JsonSerializerOptions
{
    PropertyNameCaseInsensitive = true
};

// Fetch post with ID = 3
string json = await client.GetStringAsync("https://jsonplaceholder.typicode.com/posts/3");

Post post = JsonSerializer.Deserialize<Post>(json, options);

Console.WriteLine($"Title: {post.Title}");
Console.WriteLine($"Body:  {post.Body}");

class Post
{
    public int    UserId { get; set; }
    public int    Id     { get; set; }
    public string Title  { get; set; }
    public string Body   { get; set; }
}
```

**Read a list:**

```csharp
// Fetch all posts
string json = await client.GetStringAsync("https://jsonplaceholder.typicode.com/posts");
List<Post> posts = JsonSerializer.Deserialize<List<Post>>(json, options);

Console.WriteLine($"Total: {posts.Count}");

for (int i = 0; i < posts.Count; i++)
{
    Console.WriteLine($"{posts[i].Id}. {posts[i].Title}");
}
```

---

### POST — Create data

To send data to an API you need to:
1. Serialize your C# object into a JSON string
2. Wrap that string in a `StringContent` object (this packages the text along
   with metadata the HTTP protocol needs)
3. Send it with `PostAsync`

**What is `StringContent`?**

`StringContent` is a class that wraps a plain string and tells the HTTP system
two things: the text encoding (how to convert characters to bytes for the
network) and the content type (what format the data is in).

**What is `Encoding.UTF8`?**

Computers send bytes over networks, not characters. "Encoding" is the rule for
how characters (like 'A', '€', '₹') map to bytes. **UTF-8** is the universal
standard for the web — it handles every language and character correctly. You
should always use `Encoding.UTF8` for JSON. If you use the wrong encoding,
characters like quotes or non-ASCII letters can become corrupted.

```csharp
using System;
using System.Net.Http;
using System.Text;         // for Encoding.UTF8
using System.Text.Json;

HttpClient client = new HttpClient();

// The object you want to create on the server
Post newPost = new Post
{
    UserId = 1,
    Title  = "Learning APIs in C#",
    Body   = "This is working!"
};

// Step 1: Convert the object to a JSON string
string json = JsonSerializer.Serialize(newPost);

// Step 2: Wrap it in StringContent
// Arguments: (the JSON string, how to encode characters, the content type)
StringContent content = new StringContent(json, Encoding.UTF8, "application/json");

// Step 3: POST it
HttpResponseMessage response = await client.PostAsync(
    "https://jsonplaceholder.typicode.com/posts",
    content
);

// Read the response — the server returns the created object with its new ID
string responseJson = await response.Content.ReadAsStringAsync();

JsonSerializerOptions options = new JsonSerializerOptions
{
    PropertyNameCaseInsensitive = true
};

Post created = JsonSerializer.Deserialize<Post>(responseJson, options);

Console.WriteLine($"New post created with ID: {created.Id}");
Console.WriteLine($"Status code: {(int)response.StatusCode}"); // should be 201
```

---

### PUT — Replace data

PUT replaces the **entire** resource. You must send all fields, not just the
changed ones.

```csharp
// We are completely replacing post #1 with this data
Post updated = new Post
{
    Id     = 1,
    UserId = 1,
    Title  = "Completely New Title",
    Body   = "Completely new body text"
};

string json           = JsonSerializer.Serialize(updated);
StringContent content = new StringContent(json, Encoding.UTF8, "application/json");

// PUT sends to the URL of the specific resource (post with ID 1)
HttpResponseMessage response = await client.PutAsync(
    "https://jsonplaceholder.typicode.com/posts/1",
    content
);

Console.WriteLine($"Status: {(int)response.StatusCode}"); // 200 OK
```

---

### PATCH — Partially update data

PATCH sends **only the fields you want to change**. The server keeps everything
else the same.

To do this you need to send a partial object. C# has a feature called
**anonymous objects** — objects you create on the spot without defining a
class, using `new { }` syntax:

```csharp
// Anonymous object — created without a class definition
// Useful when you only need a temporary object for one purpose
var partialUpdate = new { title = "Only the title is changing" };

// There is no class here. The compiler creates a temporary type for you.
// You cannot reuse this "type" elsewhere in code — it only exists here.
```

Now use it in a PATCH request:

```csharp
// Only changing the title — body, userId, etc. are untouched
var partialUpdate = new { title = "Only the title is changing" };

string json           = JsonSerializer.Serialize(partialUpdate);
StringContent content = new StringContent(json, Encoding.UTF8, "application/json");

// HttpRequestMessage is a class that lets you build any HTTP request manually —
// you choose the method, the URL, and attach a body yourself.
// Use it when HttpClient's shortcuts (PostAsync, GetAsync, etc.) are not enough,
// such as when you need a PATCH request.
HttpRequestMessage request = new HttpRequestMessage(HttpMethod.Patch,
    "https://jsonplaceholder.typicode.com/posts/1");

// Attach the body to our request
request.Content = content;

// SendAsync works for any HTTP method — it sends whatever request you built
HttpResponseMessage response = await client.SendAsync(request);

Console.WriteLine($"Status: {(int)response.StatusCode}");
```

> **`HttpMethod.Patch`** — `HttpMethod` is a class with static properties for
> each HTTP method: `HttpMethod.Get`, `HttpMethod.Post`, `HttpMethod.Put`,
> `HttpMethod.Patch`, `HttpMethod.Delete`. You use it to specify the method
> when building a request manually.

---

### DELETE — Remove data

```csharp
// Delete post with ID = 5
HttpResponseMessage response = await client.DeleteAsync(
    "https://jsonplaceholder.typicode.com/posts/5"
);

// 200 OK or 204 No Content — both mean success
// 204 means "it worked, but there is nothing to return"
Console.WriteLine($"Status: {(int)response.StatusCode}");
```

---

## 10. Error Handling and Status Codes

A real-world API client must handle failures gracefully. Networks fail, servers
return errors, rate limits get hit. Never assume the request will succeed.

### Pattern 1 — Check IsSuccessStatusCode

`IsSuccessStatusCode` returns `true` for any status in the 200–299 range:

```csharp
HttpResponseMessage response = await client.GetAsync("https://api.example.com/data");

if (response.IsSuccessStatusCode)
{
    // 200–299: safe to read the body
    string json = await response.Content.ReadAsStringAsync();
    Console.WriteLine("Got data: " + json);
}
else
{
    // Something went wrong
    Console.WriteLine($"Error: {(int)response.StatusCode} {response.ReasonPhrase}");

    // APIs often include a helpful error message in the body
    string errorBody = await response.Content.ReadAsStringAsync();
    Console.WriteLine($"Details: {errorBody}");
}
```

---

### Pattern 2 — EnsureSuccessStatusCode + try/catch

`EnsureSuccessStatusCode()` throws an exception if the status is not 2xx.
Use this when you want all errors handled in one place:

```csharp
try
{
    HttpResponseMessage response = await client.GetAsync("https://api.example.com/data");

    // Throws HttpRequestException if status is 4xx or 5xx
    response.EnsureSuccessStatusCode();

    string json = await response.Content.ReadAsStringAsync();
    Console.WriteLine(json);
}
catch (HttpRequestException ex)
{
    // Catches both network errors and 4xx/5xx responses
    Console.WriteLine($"Request failed: {ex.Message}");
    Console.WriteLine($"Status code:    {ex.StatusCode}"); // null if network error
}
catch (TaskCanceledException)
{
    // The request took too long and was cancelled (timeout)
    Console.WriteLine("Request timed out.");
}
catch (Exception ex)
{
    // Anything else unexpected
    Console.WriteLine($"Unexpected error: {ex.Message}");
}
```

---

### Pattern 3 — React to specific status codes

Sometimes different codes need different responses from your program:

```csharp
HttpResponseMessage response = await client.GetAsync(url);

switch ((int)response.StatusCode)
{
    case 200:
        string json = await response.Content.ReadAsStringAsync();
        // process data
        break;

    case 401:
        Console.WriteLine("Invalid credentials. Check your API key.");
        break;

    case 403:
        Console.WriteLine("You do not have permission for this.");
        break;

    case 404:
        Console.WriteLine("That resource does not exist.");
        break;

    case 429:
        Console.WriteLine("Too many requests. Wait before trying again.");
        break;

    default:
        // 500 and above = the server broke, not your fault
        if ((int)response.StatusCode >= 500)
            Console.WriteLine("Server error. Try again later.");
        else
            Console.WriteLine($"Unexpected status: {(int)response.StatusCode}");
        break;

    default:
        Console.WriteLine($"Unexpected status: {(int)response.StatusCode}");
        break;
}
```

---

### Setting a timeout

By default `HttpClient` waits up to 100 seconds. You can change it:

```csharp
HttpClient client = new HttpClient();
client.Timeout = TimeSpan.FromSeconds(10); // give up after 10 seconds
```

If the timeout is exceeded, a `TaskCanceledException` is thrown.

---

## 11. Sending API Keys and Auth Tokens

Most real APIs require you to prove who you are before they respond. There are
three common ways to send credentials.

### Method 1 — API Key in a Request Header

Many APIs ask for a key in a custom header. You can set a default header that
goes with every single request from a client:

```csharp
HttpClient client = new HttpClient();

// Add a header that will be sent with every request made by this client
// "X-Api-Key" is the header name — each API specifies its own name
client.DefaultRequestHeaders.Add("X-Api-Key", "your-api-key-here");

// Every request from now on automatically includes that header
string json = await client.GetStringAsync("https://api.example.com/data");
```

To add a header to just one specific request:

```csharp
HttpRequestMessage request = new HttpRequestMessage(HttpMethod.Get, url);
request.Headers.Add("X-Api-Key", "your-api-key-here");

HttpResponseMessage response = await client.SendAsync(request);
```

---

### Method 2 — Bearer Token (Authorization header)

OAuth-based APIs (GitHub, Google, Stripe, etc.) give you an access token after
login. You send it in the `Authorization` header with the word `Bearer` in front:

```csharp
using System.Net.Http.Headers;

HttpClient client = new HttpClient();

// AuthenticationHeaderValue properly formats the Authorization header
// First argument: the scheme ("Bearer")
// Second argument: the actual token
client.DefaultRequestHeaders.Authorization =
    new AuthenticationHeaderValue("Bearer", "your-access-token-here");

string json = await client.GetStringAsync("https://api.github.com/user");
```

The header sent looks like:

```
Authorization: Bearer your-access-token-here
```

---

### Method 3 — API Key as a URL Query Parameter

Some APIs accept the key appended directly to the URL:

```csharp
string apiKey = "your-key-here";
string city   = "Bengaluru";

// The API key becomes part of the URL after a ?
string url = $"https://api.openweathermap.org/data/2.5/weather?q={city}&appid={apiKey}";

string json = await client.GetStringAsync(url);
```

---

### Keep API keys out of your code

Never write API keys directly in source code that gets committed to Git. Once
it is in your history it is essentially public. Use environment variables:

```csharp
// Read the key from an environment variable set on your machine
string apiKey = Environment.GetEnvironmentVariable("MY_API_KEY");

if (string.IsNullOrEmpty(apiKey))
{
    Console.WriteLine("API key not set. Use: export MY_API_KEY=your-key-here");
    return;
}

client.DefaultRequestHeaders.Add("X-Api-Key", apiKey);
```

Set the variable in terminal before running your app (Mac/Linux):

```bash
export MY_API_KEY="your-key-here"
dotnet run
```

On Windows (Command Prompt):

```cmd
set MY_API_KEY=your-key-here
dotnet run
```

---

## 12. Building Your Own API with ASP.NET Core

So far you have been a **client** — calling other people's APIs. Now you will
be the **server** — building your own API that others (or your own frontend)
can call.

**ASP.NET Core** is the .NET framework for building web apps and APIs.
**Minimal APIs** (introduced in .NET 6) let you write a full REST API with
very little code.

---

### The difference between a console app and a web app

```
Console app:  starts → runs some code → exits
Web app:      starts → listens for HTTP requests forever → handles each one
```

A web app is always waiting. When an HTTP request arrives at a URL, the right
bit of your code runs to handle it, and the app keeps running for the next one.

---

### Setup

```bash
dotnet new web -n MyApi   # creates an ASP.NET Core Minimal API project
cd MyApi
dotnet run                 # starts the server; it keeps running
```

You will see:

```
Now listening on: http://localhost:5000
```

Open `Program.cs` — it already has a few lines. Replace everything with your
own code.

---

### What `builder` and `app` are

Every ASP.NET Core app starts with two lines you will always write:

```csharp
var builder = WebApplication.CreateBuilder(args);
var app     = builder.Build();
```

- `WebApplication.CreateBuilder(args)` — creates a **builder** object that
  lets you configure your app (register services, set up databases, etc.).
  Think of it as filling in a blueprint.
- `builder.Build()` — takes the blueprint and builds the actual running app.
  From here you define your endpoints.
- `app.Run()` — starts the server and begins listening. Nothing before this
  line is responding to HTTP yet.

For a basic API you do not need to add anything between these two lines —
just build the app and define endpoints.

---

### Lambda expressions in endpoints

When you define an endpoint, you write:

```csharp
app.MapGet("/hello", () =>
{
    return "Hello!";
});
```

The second argument `() => { return "Hello!"; }` is a **lambda expression** —
an anonymous (nameless) function written inline. You have probably seen them
before in LINQ (like `.Where(x => x > 5)`). Here the same idea is used to
define what code should run when someone calls GET `/hello`.

- `()` — the parameters the function takes (empty here)
- `=>` — "goes to" / separates parameters from the body
- `{ return "Hello!"; }` — the function body

For a short body, you can skip the braces:

```csharp
app.MapGet("/hello", () => "Hello!"); // same thing, one line
```

When the endpoint needs a value from the URL (like an ID), it appears as a
parameter:

```csharp
// {id} in the URL is captured as the int parameter 'id'
app.MapGet("/posts/{id}", (int id) =>
{
    return $"You asked for post number {id}";
});
```

---

### Your first endpoint

```csharp
var builder = WebApplication.CreateBuilder(args);
var app     = builder.Build();

// Define a GET endpoint at /hello
// When someone sends GET http://localhost:5000/hello, this code runs
app.MapGet("/hello", () =>
{
    return "Hello from my first C# API!";
});

app.Run(); // start listening — nothing above this has handled any requests yet
```

Run with `dotnet run`, open a browser to `http://localhost:5000/hello`.
You see: `Hello from my first C# API!`

---

### Returning JSON

ASP.NET Core automatically converts C# objects to JSON when you return them:

```csharp
var builder = WebApplication.CreateBuilder(args);
var app     = builder.Build();

// In-memory list acting as our "database" for this example
List<Product> products = new List<Product>
{
    new Product { Id = 1, Name = "Keyboard", Price = 1500 },
    new Product { Id = 2, Name = "Mouse",    Price = 800  },
    new Product { Id = 3, Name = "Monitor",  Price = 12000 }
};

// GET /products — returns all products as a JSON array
app.MapGet("/products", () =>
{
    return products; // ASP.NET Core serializes this to JSON automatically
});

// GET /products/2 — returns one product by ID
// {id} in the route is captured as the int parameter named 'id'
app.MapGet("/products/{id}", (int id) =>
{
    Product found = null;

    for (int i = 0; i < products.Count; i++)
    {
        if (products[i].Id == id)
        {
            found = products[i];
            break;
        }
    }

    if (found == null)
    {
        return Results.NotFound($"Product {id} not found.");
    }

    return Results.Ok(found);
});

app.Run();

class Product
{
    public int    Id    { get; set; }
    public string Name  { get; set; }
    public double Price { get; set; }
}
```

Visit `http://localhost:5000/products` → JSON array of all 3 products.
Visit `http://localhost:5000/products/2` → just the mouse as JSON.
Visit `http://localhost:5000/products/99` → `{ "title": "Product 99 not found" }` with a 404 status.

---

### What is `Results`?

`Results` is a built-in ASP.NET Core class that creates HTTP responses with the
correct status code and body. Instead of returning raw objects (which always
gives 200), `Results` lets you control the exact status code:

```csharp
Results.Ok(data)             // 200 + data as JSON
Results.Created(url, data)   // 201 + data as JSON (after creating something)
Results.NoContent()          // 204 (success, nothing to return)
Results.NotFound("message")  // 404 + message
Results.BadRequest("message")// 400 + message
Results.Unauthorized()       // 401
```

---

### POST — Accept JSON and create a resource

ASP.NET Core automatically reads the request body and deserializes it into
your class:

```csharp
app.MapPost("/products", (Product newProduct) =>
{
    // 'newProduct' is already deserialized from the JSON body — ASP.NET Core
    // does this automatically when you put the type in the parameter list
    newProduct.Id = products.Count + 1;
    products.Add(newProduct);

    // 201 Created — convention is to also return the location of the new item
    return Results.Created($"/products/{newProduct.Id}", newProduct);
});
```

Test it from terminal:

```bash
curl -X POST http://localhost:5000/products \
  -H "Content-Type: application/json" \
  -d '{"name": "Headset", "price": 3000}'
```

---

### PUT — Replace a resource

```csharp
app.MapPut("/products/{id}", (int id, Product updated) =>
{
    for (int i = 0; i < products.Count; i++)
    {
        if (products[i].Id == id)
        {
            products[i].Name  = updated.Name;
            products[i].Price = updated.Price;
            return Results.Ok(products[i]);
        }
    }

    return Results.NotFound();
});
```

---

### DELETE — Remove a resource

```csharp
app.MapDelete("/products/{id}", (int id) =>
{
    for (int i = 0; i < products.Count; i++)
    {
        if (products[i].Id == id)
        {
            products.Remove(products[i]);
            return Results.NoContent(); // 204 — deleted, nothing to return
        }
    }

    return Results.NotFound();
});
```

---

## 13. Real-World Example — GitHub Public API

Let us combine everything: `HttpClient`, async/await, JSON deserialization,
headers, and error handling — using a real, live public API.

We will fetch a GitHub user's profile and their top repositories. GitHub's
public API does not need a key for basic read requests.

Create a new console app (`dotnet new console`) and use this `Program.cs`:

```csharp
using System;
using System.Collections.Generic;
using System.Net.Http;
using System.Net.Http.Headers;
using System.Text.Json;
using System.Text.Json.Serialization;

HttpClient client = new HttpClient();

// GitHub requires a User-Agent header — without it you get 403 Forbidden
// ProductHeaderValue formats it correctly: "AppName/Version"
client.DefaultRequestHeaders.UserAgent.ParseAdd("MyApp/1.0");

// Run our two functions
await PrintUserInfo("torvalds");
await PrintRepositories("torvalds");

// ─── Function 1: Print a user's profile ───────────────────────────────────

async Task PrintUserInfo(string username)
{
    string url = $"https://api.github.com/users/{username}";

    try
    {
        HttpResponseMessage response = await client.GetAsync(url);
        response.EnsureSuccessStatusCode();

        string json = await response.Content.ReadAsStringAsync();

        JsonSerializerOptions options = new JsonSerializerOptions
        {
            PropertyNameCaseInsensitive = true
        };

        GitHubUser user = JsonSerializer.Deserialize<GitHubUser>(json, options);

        Console.WriteLine("=== GitHub User ===");
        Console.WriteLine($"Username:  {user.Login}");
        Console.WriteLine($"Name:      {user.Name}");
        Console.WriteLine($"Location:  {user.Location}");
        Console.WriteLine($"Followers: {user.Followers}");
        Console.WriteLine($"Repos:     {user.PublicRepos}");
    }
    catch (HttpRequestException ex)
    {
        Console.WriteLine($"Could not fetch user: {ex.Message}");
    }
}

// ─── Function 2: Print top repos by star count ─────────────────────────────

async Task PrintRepositories(string username)
{
    // per_page=5: return only 5 results; sort=stars: sorted by star count
    string url = $"https://api.github.com/users/{username}/repos?per_page=5&sort=stars";

    try
    {
        HttpResponseMessage response = await client.GetAsync(url);
        response.EnsureSuccessStatusCode();

        string json = await response.Content.ReadAsStringAsync();

        JsonSerializerOptions options = new JsonSerializerOptions
        {
            PropertyNameCaseInsensitive = true
        };

        List<GitHubRepo> repos = JsonSerializer.Deserialize<List<GitHubRepo>>(json, options);

        Console.WriteLine("\n=== Top Repositories ===");

        for (int i = 0; i < repos.Count; i++)
        {
            Console.WriteLine($"{i + 1}. {repos[i].Name}");
            Console.WriteLine($"   Stars: {repos[i].StargazersCount}");
            Console.WriteLine($"   URL:   {repos[i].HtmlUrl}");
            Console.WriteLine();
        }
    }
    catch (HttpRequestException ex)
    {
        Console.WriteLine($"Could not fetch repos: {ex.Message}");
    }
}

// ─── Classes that mirror the GitHub API JSON ──────────────────────────────

class GitHubUser
{
    public string Login    { get; set; }  // "login" in JSON
    public string Name     { get; set; }  // "name" in JSON
    public string Location { get; set; }  // "location" in JSON
    public int    Followers { get; set; } // "followers" in JSON

    // "public_repos" in JSON doesn't auto-match "PublicRepos", so we map it
    [JsonPropertyName("public_repos")]
    public int PublicRepos { get; set; }
}

class GitHubRepo
{
    public string Name { get; set; }  // "name" in JSON

    [JsonPropertyName("html_url")]
    public string HtmlUrl { get; set; }

    [JsonPropertyName("stargazers_count")]
    public int StargazersCount { get; set; }
}
```

**Expected output:**

```
=== GitHub User ===
Username:  torvalds
Name:      Linus Torvalds
Location:  Portland, OR
Followers: 200000+
Repos:     7

=== Top Repositories ===
1. linux
   Stars: 150000+
   URL:   https://github.com/torvalds/linux
...
```

---

## 14. Common Pitfalls

### Pitfall 1 — Creating HttpClient inside a loop

```csharp
// BAD — new connections opened and never properly released
for (int i = 0; i < 100; i++)
{
    HttpClient client = new HttpClient(); // creates a new socket each time
    await client.GetStringAsync(url);
    // socket lingers even after the loop; you exhaust available ports
}

// GOOD — one instance, reused every time
HttpClient client = new HttpClient();
for (int i = 0; i < 100; i++)
{
    await client.GetStringAsync(url);
}
```

---

### Pitfall 2 — Forgetting await

```csharp
// BUG — GetStringAsync returns Task<string>, not string.
// Without await, 'json' holds a Task object, not the text content.
string json = client.GetStringAsync(url); // COMPILE ERROR — types don't match

// CORRECT
string json = await client.GetStringAsync(url);
```

---

### Pitfall 3 — Reading body after a failed request

```csharp
// BAD — body might be an error message, not valid JSON
// Deserializing it will throw an exception
string json = await client.GetStringAsync(url);
Post post = JsonSerializer.Deserialize<Post>(json); // crashes if response was an error

// GOOD — check first
HttpResponseMessage response = await client.GetAsync(url);
if (response.IsSuccessStatusCode)
{
    string json = await response.Content.ReadAsStringAsync();
    Post post   = JsonSerializer.Deserialize<Post>(json);
}
else
{
    Console.WriteLine($"Error: {response.StatusCode}");
}
```

---

### Pitfall 4 — JSON property name mismatch gives you null (silently)

```json
{ "first_name": "Sharan" }
```

```csharp
class Person
{
    public string FirstName { get; set; } // "FirstName" does NOT match "first_name"
}

Person p = JsonSerializer.Deserialize<Person>(json);
Console.WriteLine(p.FirstName); // prints nothing — null, no error thrown
```

Fix: use `[JsonPropertyName("first_name")]` on the property, or use
`PropertyNameCaseInsensitive = true` for simple casing differences.

---

### Pitfall 5 — Hardcoding API keys in source code

```csharp
// NEVER — this goes into Git and is visible to everyone
string key = "sk-live-abc123realkey";

// ALWAYS — read from environment variable
string key = Environment.GetEnvironmentVariable("MY_API_KEY");
```

---

### Pitfall 6 — async void instead of async Task

```csharp
// BAD — exceptions thrown inside cannot be caught by the caller
async void FetchData()
{
    await client.GetStringAsync(url); // if this throws, the exception vanishes
}

// GOOD — exceptions propagate normally
async Task FetchData()
{
    await client.GetStringAsync(url);
}
```

---

### Pitfall 7 — Calling app.Run() before defining endpoints

```csharp
// BAD — endpoints defined after Run() are never registered
var app = builder.Build();
app.Run();                          // server starts here — nothing below runs
app.MapGet("/hello", () => "Hi");  // never reached
```

```csharp
// GOOD — all endpoints before Run()
var app = builder.Build();
app.MapGet("/hello", () => "Hi");  // registered
app.Run();                          // now start listening
```

---

## 15. Practice Exercises

These exercises use free public APIs that need no account or API key.

---

### Exercise 1 — Random joke reader

**API:** `https://official-joke-api.appspot.com/random_joke`

**Sample response:**
```json
{
  "type": "general",
  "setup": "Why did the programmer quit his job?",
  "punchline": "Because he didn't get arrays."
}
```

**Tasks:**
1. Create a `Joke` class with `Type`, `Setup`, and `Punchline` properties
2. Fetch a random joke and print: `Q: [setup]  A: [punchline]`
3. Fetch 5 jokes in a loop and print them all

---

### Exercise 2 — Country info lookup

**API:** `https://restcountries.com/v3.1/name/{countryName}`

Example: `https://restcountries.com/v3.1/name/India`

**Tasks:**
1. Fetch data for "Japan"
2. Extract and print: official name, capital, population, and region
3. Handle the case where the country name is not found (404) gracefully

---

### Exercise 3 — Full CRUD console menu

**API:** `https://jsonplaceholder.typicode.com/todos`

Build a console app with an interactive menu:

```
1. List first 10 todos
2. Get todo by ID
3. Create a new todo
4. Update a todo
5. Delete a todo
6. Exit
```

Each option calls the appropriate HTTP method. Handle errors for each (e.g.
"Todo not found" if the user enters an invalid ID).

---

### Exercise 4 — Build your own Book Library API

Create an ASP.NET Core Minimal API for a book library. A book has:
`Id`, `Title`, `Author`, `Year`, `IsAvailable`.

Implement:
- `GET /books` — return all books
- `GET /books/{id}` — return one book or 404
- `POST /books` — add a new book, return 201
- `PUT /books/{id}` — update a book, return 200 or 404
- `DELETE /books/{id}` — remove a book, return 204 or 404

Test every endpoint using curl commands from your terminal.

---

### Exercise 5 — Proxy API

Create an ASP.NET Core API that calls an upstream API and reshapes the data:

- `GET /joke` — calls the joke API and returns:
  `{ "question": "...", "answer": "..." }` using your own field names
- `GET /country/{name}` — calls the countries API and returns only:
  `{ "name": "...", "capital": "...", "population": ... }`

This is what real backend APIs do: fetch from external services, filter the
data, and return only what the frontend needs.

---

## 16. Quick Reference Cheat Sheet

### HttpClient setup

```csharp
using System.Net.Http;
using System.Net.Http.Headers;

// Create once, reuse everywhere
HttpClient client = new HttpClient();

// Set timeout (default is 100 seconds)
client.Timeout = TimeSpan.FromSeconds(15);

// Add a default header sent with every request
client.DefaultRequestHeaders.Add("X-Api-Key", "your-key");

// Add a Bearer token
client.DefaultRequestHeaders.Authorization =
    new AuthenticationHeaderValue("Bearer", "your-token");

// Set User-Agent (required by some APIs like GitHub)
client.DefaultRequestHeaders.UserAgent.ParseAdd("MyApp/1.0");
```

---

### Making requests

```csharp
using System.Text;
using System.Text.Json;

// GET
string              text     = await client.GetStringAsync(url);
HttpResponseMessage response = await client.GetAsync(url);

// Build body for POST/PUT/PATCH
string        json    = JsonSerializer.Serialize(myObject);
StringContent content = new StringContent(json, Encoding.UTF8, "application/json");

// POST
HttpResponseMessage r = await client.PostAsync(url, content);

// PUT
HttpResponseMessage r = await client.PutAsync(url, content);

// PATCH (manual)
HttpRequestMessage req = new HttpRequestMessage(HttpMethod.Patch, url);
req.Content = content;
HttpResponseMessage r = await client.SendAsync(req);

// DELETE
HttpResponseMessage r = await client.DeleteAsync(url);

// Read response
bool   ok   = response.IsSuccessStatusCode;
int    code = (int)response.StatusCode;
string body = await response.Content.ReadAsStringAsync();
response.EnsureSuccessStatusCode(); // throws if not 2xx
```

---

### JSON serialize / deserialize

```csharp
using System.Text.Json;
using System.Text.Json.Serialization;

// Options (always use these)
JsonSerializerOptions options = new JsonSerializerOptions
{
    PropertyNameCaseInsensitive = true
};

// JSON → C# object
MyClass   obj  = JsonSerializer.Deserialize<MyClass>(json, options);
List<MyClass> list = JsonSerializer.Deserialize<List<MyClass>>(json, options);

// C# object → JSON
string json = JsonSerializer.Serialize(obj);

// Handle name mismatches with attribute
[JsonPropertyName("first_name")]
public string FirstName { get; set; }
```

---

### ASP.NET Core Minimal API skeleton

```csharp
var builder = WebApplication.CreateBuilder(args);
var app     = builder.Build();

// Endpoints — all before app.Run()
app.MapGet   ("/items",      ()              => items);
app.MapGet   ("/items/{id}", (int id)        => GetOne(id));
app.MapPost  ("/items",      (Item newItem)  => Create(newItem));
app.MapPut   ("/items/{id}", (int id, Item x)=> Update(id, x));
app.MapDelete("/items/{id}", (int id)        => Delete(id));

app.Run(); // start the server
```

---

### Results helper methods

```csharp
Results.Ok(data)              // 200 + JSON
Results.Created("/url", data) // 201 + JSON
Results.NoContent()           // 204, no body
Results.BadRequest("msg")     // 400 + message
Results.NotFound("msg")       // 404 + message
Results.Unauthorized()        // 401
```

---

### Error handling template

```csharp
try
{
    HttpResponseMessage response = await client.GetAsync(url);
    response.EnsureSuccessStatusCode();

    string json    = await response.Content.ReadAsStringAsync();
    MyClass result = JsonSerializer.Deserialize<MyClass>(json, options);
    // use result
}
catch (HttpRequestException ex)
{
    // Network failure or non-2xx status
    Console.WriteLine($"HTTP error: {ex.Message}  Status: {ex.StatusCode}");
}
catch (JsonException ex)
{
    // Response was not valid JSON
    Console.WriteLine($"Parse error: {ex.Message}");
}
catch (TaskCanceledException)
{
    // Request exceeded the timeout
    Console.WriteLine("Request timed out.");
}
```

---

### Status codes at a glance

```
2xx → You Win    → 200 OK, 201 Created, 204 No Content
4xx → Your Fault → 400 Bad Data, 401 No Auth, 403 Forbidden, 404 Not Found, 429 Too Fast
5xx → Their Fault→ 500 Server Error, 503 Unavailable
```
