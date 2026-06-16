# C# Delegates & Events: A Complete Deep-Dive Guide

> **Who this is for:** Someone who has never seen delegates or events before.
> Every concept is built from scratch, every code line is explained, and common confusions are addressed inline.

---

## Table of Contents

1. [The Problem Delegates Solve](#1-the-problem-delegates-solve)
2. [Delegate Basics — Declaring, Assigning, Invoking](#2-delegate-basics--declaring-assigning-invoking)
3. [Multicast Delegates — One Variable, Many Methods](#3-multicast-delegates--one-variable-many-methods)
4. [Built-in Generic Delegates — Action, Func, Predicate](#4-built-in-generic-delegates--action-func-predicate)
5. [Lambda Expressions & Anonymous Methods](#5-lambda-expressions--anonymous-methods)
6. [Events — The Publisher-Subscriber Pattern](#6-events--the-publisher-subscriber-pattern)
7. [EventHandler & Custom EventArgs](#7-eventhandler--custom-eventargs)
8. [Delegates & Events in Unity](#8-delegates--events-in-unity)
9. [Common Patterns & Pitfalls](#9-common-patterns--pitfalls)
10. [Practice Exercises](#10-practice-exercises)
11. [Quick Reference Cheat Sheet](#11-quick-reference-cheat-sheet)

---

## 1. The Problem Delegates Solve

### Start here: a TV remote analogy

Imagine you have a TV remote. The remote has a **Volume Up** button. When you press it, the TV increases volume.

Now imagine you want a universal remote that can control ANY TV — Samsung, Sony, LG. The button's *job* (increase volume) is the same. But the exact *method* it calls is different per TV model.

This is exactly the problem delegates solve in code: **you want to define what a method's shape looks like, and let the actual implementation be swapped in at runtime.**

---

### The code problem

Let's say you are building a notification system. You want to notify the user through different channels — email, SMS, popup. Without delegates, you'd write this:

```csharp
// WITHOUT delegates — you have to write separate methods
// and the calling code must know about every channel

void NotifyByEmail(string message)
{
    Console.WriteLine($"EMAIL: {message}");
}

void NotifyBySMS(string message)
{
    Console.WriteLine($"SMS: {message}");
}

void NotifyByPopup(string message)
{
    Console.WriteLine($"POPUP: {message}");
}

// The calling code must manually pick the method
void SendAlert(string channel, string message)
{
    if (channel == "email")
        NotifyByEmail(message);
    else if (channel == "sms")
        NotifyBySMS(message);
    else if (channel == "popup")
        NotifyByPopup(message);
    // Every new channel = edit this method again
}
```

**Problems with this approach:**
- Every time you add a new channel, you edit `SendAlert`
- `SendAlert` needs to know every possible method — it is tightly coupled to all of them
- You cannot choose the channel at runtime without a growing if/else chain

---

### What we actually want

What if you could **pass the method itself** as an argument? Like this:

```csharp
// IDEAL — pass the method as an argument
void SendAlert(??? notifyMethod, string message)
{
    notifyMethod(message); // just call whatever method was passed in
}

// Call it like this:
SendAlert(NotifyByEmail,  "Server is down!");
SendAlert(NotifyBySMS,    "Server is down!");
SendAlert(NotifyByPopup,  "Server is down!");
```

The `???` is where a **delegate** goes. Delegates let you treat methods as values — you can store them in variables, pass them as arguments, and call them later.

---

### The mental model

Think of a delegate as a **variable that holds a method** instead of a data value.

```
Regular variable:   int x = 5;          → stores a number
Delegate variable:  MyDelegate d = Foo; → stores a method
```

Just like you can pass `x` to a method, you can pass `d` to a method. And just like calling `x + 1` reads the number, calling `d()` runs the stored method.

---

## 2. Delegate Basics — Declaring, Assigning, Invoking

### Step 1 — Declare the delegate type

A delegate declaration is like a **contract** that describes the exact shape of methods it can hold. It specifies the return type and parameter types.

```csharp
// Syntax: delegate <returnType> <DelegateName>(<parameters>);

delegate void Notifier(string message);
//      ^^^^                            → this delegate holds methods that return NOTHING
//           ^^^^^^^^                   → the name of this new delegate TYPE
//                    ^^^^^^^^^^^^^^    → methods it holds must take exactly one string parameter
```

Think of this line as saying: *"I am defining a new type called `Notifier`. Any method assigned to a `Notifier` variable must return void and take a single string parameter."*

> **Important:** This line does NOT create a method. It defines a *type* — similar to how `class Dog { }` defines a type without creating an object.

```csharp
// More examples of delegate declarations:

delegate int  Calculator(int a, int b);
// → holds methods that take two ints and return an int

delegate bool Validator(string input);
// → holds methods that take a string and return a bool

delegate void VoidNoArgs();
// → holds methods that take nothing and return nothing
```

---

### Step 2 — Write methods that match the delegate

The methods you plan to assign must have the **exact same return type and parameter types** as the delegate. The parameter *names* do not matter — only the types.

```csharp
delegate void Notifier(string message); // our delegate type

// These methods MATCH the Notifier delegate:
static void NotifyByEmail(string message)  { Console.WriteLine($"EMAIL: {message}"); }
static void NotifyBySMS(string msg)        { Console.WriteLine($"SMS: {msg}"); }   // 'msg' vs 'message' is fine
static void NotifyByPopup(string text)     { Console.WriteLine($"POPUP: {text}"); }

// These methods do NOT match — they cannot be assigned to a Notifier:
static void Greet()                        { }  // ❌ no parameter
static string GetMessage(string m)         { return m; } // ❌ returns string, not void
static void Log(string a, string b)        { }  // ❌ two parameters instead of one
```

---

### Step 3 — Assign a method to the delegate variable

```csharp
delegate void Notifier(string message);

static void NotifyByEmail(string message) { Console.WriteLine($"EMAIL: {message}"); }
static void NotifyBySMS(string message)   { Console.WriteLine($"SMS: {message}"); }

static void Main()
{
    // Create a Notifier variable and assign a method to it
    Notifier send = NotifyByEmail;
    //       ^^^^   ^^^^^^^^^^^^^
    //       |      The method we're storing (no parentheses — we're NOT calling it here)
    //       The variable name

    // You can also write it the long way:
    Notifier send2 = new Notifier(NotifyByEmail); // same result, more verbose

    // Swap the method it points to (just like reassigning any variable):
    send = NotifyBySMS;
}
```

> **Common confusion:** Writing `NotifyByEmail` (no parentheses) stores the method.
> Writing `NotifyByEmail("hello")` (with parentheses) *calls* the method immediately.
> When assigning to a delegate, always write the method name **without** parentheses.

---

### Step 4 — Invoke (call) the delegate

Calling a delegate is identical to calling a regular method:

```csharp
delegate void Notifier(string message);

static void NotifyByEmail(string message) { Console.WriteLine($"EMAIL: {message}"); }
static void NotifyBySMS(string message)   { Console.WriteLine($"SMS: {message}"); }

static void Main()
{
    Notifier send = NotifyByEmail;

    send("Server is down!"); // same as calling NotifyByEmail("Server is down!")
    // Output: EMAIL: Server is down!

    send = NotifyBySMS;
    send("Server is down!"); // now calls NotifyBySMS
    // Output: SMS: Server is down!
}
```

---

### Null safety — always check before calling

A delegate variable that has never been assigned is `null`. Calling a `null` delegate crashes the program.

```csharp
Notifier send = null;

send("hello"); // 💥 NullReferenceException — CRASH

// Safe way using the null-conditional operator ?.
send?.Invoke("hello"); // does nothing if send is null — no crash ✅

// Or the verbose way:
if (send != null)
    send("hello");
```

**Rule:** Always use `?.Invoke()` when calling a delegate unless you are 100% certain it is not null.

---

### Passing delegates as method arguments (the payoff)

Now we can solve the original problem cleanly:

```csharp
delegate void Notifier(string message);

static void NotifyByEmail(string message)  { Console.WriteLine($"EMAIL: {message}"); }
static void NotifyBySMS(string message)    { Console.WriteLine($"SMS: {message}"); }
static void NotifyByPopup(string message)  { Console.WriteLine($"POPUP: {message}"); }

// SendAlert takes a Notifier delegate as its first argument
static void SendAlert(Notifier notifyMethod, string message)
{
    Console.WriteLine("Sending alert...");
    notifyMethod?.Invoke(message); // call whatever method was passed in
}

static void Main()
{
    SendAlert(NotifyByEmail,  "Server is down!"); // EMAIL: Server is down!
    SendAlert(NotifyBySMS,    "Server is down!"); // SMS: Server is down!
    SendAlert(NotifyByPopup,  "Server is down!"); // POPUP: Server is down!

    // Adding a new channel? Just write a new method that matches Notifier.
    // SendAlert itself never needs to change.
}
```

`SendAlert` does not know or care which method it receives. It just knows the method takes a `string` — and that is enough.

---

### ✅ What you now know

- A delegate is a type that holds a reference to a method
- You declare a delegate type to specify the method signature it accepts
- You assign methods by name (no parentheses) to delegate variables
- You invoke delegates exactly like regular method calls
- Always use `?.Invoke()` to guard against null

---

## 3. Multicast Delegates — One Variable, Many Methods

So far, a delegate variable has held one method. But a single delegate can hold **multiple methods** at the same time. When you invoke it, all stored methods are called in order.

### The broadcast analogy

Think of a multicast delegate like a **radio broadcast**. One station transmits a signal, and every radio tuned to that frequency receives it simultaneously. You don't call each radio individually — you broadcast once and every listener receives it.

---

### Adding methods with +=

```csharp
delegate void Logger(string message);

static void WriteToConsole(string msg) { Console.WriteLine($"[Console] {msg}"); }
static void WriteToFile(string msg)    { Console.WriteLine($"[File]    {msg}"); }  // simplified
static void WriteToServer(string msg)  { Console.WriteLine($"[Server]  {msg}"); }  // simplified

static void Main()
{
    Logger log = null; // start empty

    log += WriteToConsole; // add first method
    log += WriteToFile;    // add second method
    log += WriteToServer;  // add third method

    log?.Invoke("Application started");
    // Output (in order):
    // [Console] Application started
    // [File]    Application started
    // [Server]  Application started
}
```

Each `+=` adds a method to the delegate's internal list. Invoking the delegate calls every method in that list, in the order they were added.

---

### Removing methods with -=

```csharp
log -= WriteToFile; // remove one method from the list

log?.Invoke("User logged in");
// Output:
// [Console] User logged in
// [Server]  User logged in
// (WriteToFile is no longer in the list)
```

> **What happens if you -= a method that was never added?**
> Nothing — it is silently ignored. No error.

---

### Return values in multicast delegates

If the delegate returns a value and multiple methods are assigned, **only the return value of the last method is kept**. The others run but their return values are thrown away.

```csharp
delegate int MathOp(int a, int b);

static int Add(int a, int b)
{
    Console.WriteLine($"Add called → {a + b}");
    return a + b;
}
static int Multiply(int a, int b)
{
    Console.WriteLine($"Multiply called → {a * b}");
    return a * b;
}

static void Main()
{
    MathOp op = Add;
    op += Multiply;

    int result = op(3, 5);
    // Output:
    // Add called → 8        (ran, but return value 8 is discarded)
    // Multiply called → 15  (ran, return value 15 is kept)

    Console.WriteLine(result); // 15
}
```

> **Takeaway:** Multicast delegates work best with `void` return types. If you need all return values, use `GetInvocationList()` and call each entry manually.

---

### Inspecting who is subscribed

```csharp
Logger log = WriteToConsole;
log += WriteToFile;

Delegate[] methods = log.GetInvocationList();
Console.WriteLine(methods.Length); // 2

for (int i = 0; i < methods.Length; i++)
{
    Console.WriteLine(methods[i].Method.Name);
}
// Output:
// WriteToConsole
// WriteToFile
```

---

### ✅ What you now know

- A delegate variable can hold multiple methods using `+=`
- `-=` removes a specific method from the list
- All methods are called in order when you invoke the delegate
- For multicast delegates with return types, only the last return value is captured

---

## 4. Built-in Generic Delegates — Action, Func, Predicate

Declaring a custom delegate type every time (`delegate void Notifier(string msg)`) is repetitive. C# ships with ready-made generic delegate types that handle the most common signatures.

### Why they exist

Before these were introduced, every developer wrote their own `delegate void SomeAction()` or `delegate int SomeFunc(int x)`. These are identical in purpose — they just clutter code with redundant type declarations. The built-in types eliminate that noise.

---

### Action — for methods that return nothing (void)

`Action` is a delegate for methods that **return nothing**.

```csharp
// Action — no parameters, no return value
Action greet = () => Console.WriteLine("Hello!");
greet(); // Hello!

// Action<T> — ONE parameter, no return value
Action<string> print = (name) => Console.WriteLine($"Hi, {name}!");
print("Sharan"); // Hi, Sharan!

// Action<T1, T2> — TWO parameters, no return value
Action<string, int> printScore = (name, score) =>
    Console.WriteLine($"{name}: {score} pts");
printScore("Sharan", 95); // Sharan: 95 pts

// This is equivalent to declaring:
// delegate void MyAction(string name, int score);
// ...but you did not have to write that declaration
```

`Action` goes up to `Action<T1, T2, ..., T16>` (16 parameters).

---

### Func — for methods that return a value

`Func` is a delegate for methods that **return something**. The rule: **the last type parameter is always the return type**.

```csharp
// Func<TResult> — no parameters, returns TResult
Func<int> rollDice = () => new Random().Next(1, 7);
int roll = rollDice(); // e.g. 4

// Func<T, TResult> — ONE parameter, returns TResult
Func<int, int> square = (x) => x * x;
int s = square(5); // 25

// Func<T1, T2, TResult> — TWO parameters, returns TResult
//         ↑    ↑          ↑
//   param1  param2     return type
Func<int, int, int> add = (a, b) => a + b;
int sum = add(3, 5); // 8

// Func<string, int, bool>
//              param1 (string)
//                     param2 (int)
//                            return type (bool)
Func<string, int, bool> hasMinLen = (text, minLen) => text.Length >= minLen;
bool ok = hasMinLen("Hello", 3); // true
```

> **Memory trick:** With `Func`, count the type parameters. There are N of them.
> The first N-1 are parameters. The last one is the return type.
>
> `Func<string, int, bool>` → 3 type params → 2 inputs (string, int) + 1 return (bool)

---

### Predicate\<T\> — for testing a condition

`Predicate<T>` is shorthand for `Func<T, bool>`. It is specifically for methods that test something about one object and return `true` or `false`.

```csharp
Predicate<int> isEven = (x) => x % 2 == 0;
Console.WriteLine(isEven(4));  // True
Console.WriteLine(isEven(7));  // False

// Very common with List<T> filtering:
List<int> numbers = new List<int> { 1, 2, 3, 4, 5, 6, 7, 8 };

List<int> evens = numbers.FindAll(isEven);   // [2, 4, 6, 8]
List<int> odds  = numbers.FindAll(x => x % 2 != 0); // [1, 3, 5, 7]

// FindAll internally has: bool Predicate<T>(T item)
// and calls it for each item, keeping only the ones where it returns true
```

---

### Which one to use?

| Your method... | Use |
|---|---|
| Returns nothing | `Action` or `Action<T>` or `Action<T1,T2,...>` |
| Returns a value | `Func<TResult>` or `Func<T,TResult>` etc. |
| Tests one item (returns bool) | `Predicate<T>` |
| Has `ref` / `out` parameters | Declare your own `delegate` |
| Needs a descriptive type name for readability | Declare your own `delegate` |

---

### ✅ What you now know

- `Action` = delegate for void-returning methods (0–16 parameters)
- `Func` = delegate for value-returning methods (last type param = return type)
- `Predicate<T>` = delegate for condition-checking methods (returns bool)
- These eliminate the need to write custom delegate declarations for most cases

---

## 5. Lambda Expressions & Anonymous Methods

Every example so far used a named method (`NotifyByEmail`, `WriteToConsole`, etc.). But you often need small, one-off pieces of logic that do not deserve a full named method. **Lambda expressions** let you write that logic inline.

### The problem with named methods for small logic

```csharp
// To sort a list of strings by length, without lambdas:
static int CompareByLength(string a, string b)
{
    return a.Length.CompareTo(b.Length);
}

List<string> words = new List<string> { "banana", "fig", "apple", "kiwi" };
words.Sort(CompareByLength);
// [fig, kiwi, apple, banana]
```

The method `CompareByLength` is only 1 line of logic but requires a full declaration. Lambdas solve this.

---

### Anonymous methods (older syntax — C# 2.0)

Before lambdas, C# introduced anonymous methods using the `delegate` keyword:

```csharp
List<string> words = new List<string> { "banana", "fig", "apple", "kiwi" };

words.Sort(delegate(string a, string b)
{
    return a.Length.CompareTo(b.Length);
});
// The method is defined right here, inline — no name, no separate declaration
```

This is rarely written today. Lambda expressions replaced it.

---

### Lambda expressions — the modern way (C# 3.0+)

The `=>` symbol is read as "goes to" or "such that". Left side = parameters. Right side = body.

```csharp
// Syntax: (parameters) => expression
//     OR: (parameters) => { statements; }

// Expression lambda — single expression, result is the return value (no 'return' needed)
Func<int, int> square = (x) => x * x;
//                       ^^^    ^^^^^
//                       param  expression (automatically returned)

// When there's exactly ONE parameter, parentheses are optional:
Func<int, int> square2 = x => x * x; // same thing

// Statement lambda — multiple statements need curly braces and explicit return
Func<int, int> absolute = x =>
{
    if (x < 0)
        return -x;
    return x;
};

// Action lambda (void return — no return statement):
Action<string> log = msg => Console.WriteLine($"[LOG] {msg}");
log("Server started"); // [LOG] Server started

// Rewriting the sort example:
words.Sort((a, b) => a.Length.CompareTo(b.Length)); // same result, much cleaner
```

---

### Lambdas assigned to different delegate types

A lambda is not tied to a specific type — C# infers which delegate type it matches:

```csharp
// Same lambda body, different types:
Action         a = ()  => Console.WriteLine("Hi");
Func<string>   f = ()  => "Hello";
Action<int>    a2 = x  => Console.WriteLine(x);
Func<int, int> f2 = x  => x * 2;
Predicate<int> p  = x  => x > 0;
```

---

### Closures — lambdas can capture outer variables

A lambda can read and use variables from its surrounding scope. This is called **capturing** a variable, and the lambda is called a **closure**.

```csharp
int minScore = 50; // outer variable

Predicate<int> isPassing = score => score >= minScore;
// 'minScore' is captured — the lambda holds a reference to it

Console.WriteLine(isPassing(40)); // False
Console.WriteLine(isPassing(75)); // True

// Key point: the lambda captures the VARIABLE, not its value at capture time
minScore = 80;
Console.WriteLine(isPassing(75)); // False — minScore is now 80, so 75 < 80
```

> The lambda does not take a snapshot of `minScore = 50`. It holds a live reference to the `minScore` variable. When the variable changes, the lambda sees the new value.

---

### The closure loop trap — a common bug

```csharp
// BUG — classic beginner mistake:
List<Action> actions = new List<Action>();
for (int i = 0; i < 5; i++)
{
    actions.Add(() => Console.WriteLine(i));
    // All lambdas capture the SAME 'i' variable
}
// After the loop, i == 5
actions.ForEach(a => a());
// Output: 5 5 5 5 5  ← not what you expected!

// FIX — capture a copy of i in each iteration:
List<Action> fixed_actions = new List<Action>();
for (int i = 0; i < 5; i++)
{
    int copy = i;           // a NEW variable each iteration
    fixed_actions.Add(() => Console.WriteLine(copy));
    // each lambda captures its OWN copy
}
fixed_actions.ForEach(a => a());
// Output: 0 1 2 3 4  ← correct!
```

---

### ✅ What you now know

- Lambda expressions write small methods inline using `=>`
- `x => x * x` is an expression lambda (single expression)
- `x => { ...; return y; }` is a statement lambda (multiple lines)
- Lambdas can capture outer variables — this is a closure
- Closures capture the variable itself (live reference), not its value at creation time

---

## 6. Events — The Publisher-Subscriber Pattern

### Why raw delegates have a problem for communication

Say you want `PlayerHealth` to notify other systems when the player takes damage. You make `OnHealthChanged` a public delegate:

```csharp
public class PlayerHealth
{
    public Action<int> OnHealthChanged; // public delegate — BAD!
}
```

Now any code in your project can do these dangerous things:

```csharp
PlayerHealth player = new PlayerHealth();

// ❌ Problem 1: Any class can INVOKE the event
player.OnHealthChanged(999); // Enemy can trigger "player took damage"? Dangerous!

// ❌ Problem 2: Any class can WIPE OUT all subscribers at once
player.OnHealthChanged = null; // Deletes every listener in one line — disaster!

// ❌ Problem 3: Any class can READ the internal subscriber list
var listeners = player.OnHealthChanged.GetInvocationList(); // breaks encapsulation
```

A public delegate field has no protection. Any class can do anything to it.

---

### The event keyword — what it actually does

Adding the `event` keyword creates a **restricted delegate**. From outside the class, only `+=` and `-=` are allowed. Invoking and resetting are blocked.

```csharp
public class PlayerHealth
{
    // Adding 'event' wraps the delegate with protection
    public event Action<int> OnHealthChanged;
    //     ^^^^^
    //     Now outside code can ONLY += and -=

    private int _health = 100;

    public void TakeDamage(int amount)
    {
        _health -= amount;
        OnHealthChanged?.Invoke(_health); // Only THIS class can invoke it
    }
}

PlayerHealth player = new PlayerHealth();

player.OnHealthChanged += UpdateUI;     // ✅ Subscribing — allowed
player.OnHealthChanged -= UpdateUI;     // ✅ Unsubscribing — allowed
player.OnHealthChanged(50);             // ❌ Compile error — cannot invoke from outside
player.OnHealthChanged = null;          // ❌ Compile error — cannot reset from outside
```

> **Summary:** `event` = a delegate field with restricted public access. The class that declares it can do anything with it. Outside code can only `+=` and `-=`.

---

### The Publisher-Subscriber pattern

This is the core pattern that events enable. You decouple the class that fires events (the **Publisher**) from the classes that respond (the **Subscribers**).

```
Publisher:   "Something happened — I will broadcast it."
Subscriber:  "I am listening — I will react when it happens."
```

The publisher does not know who is listening. The subscribers do not know when it will fire. They are independent of each other.

```csharp
// ──────────────────────────────
// PUBLISHER
// ──────────────────────────────
public class PlayerHealth
{
    private int _health = 100;

    // Declares the events — these are the "broadcast channels"
    public event Action           OnPlayerDied;
    public event Action<int>      OnHealthChanged; // passes new health value

    public void TakeDamage(int amount)
    {
        _health -= amount;

        // Broadcast to anyone listening
        OnHealthChanged?.Invoke(_health);

        if (_health <= 0)
            OnPlayerDied?.Invoke();
    }
}

// ──────────────────────────────
// SUBSCRIBER 1 — the UI
// ──────────────────────────────
public class HealthUI
{
    public HealthUI(PlayerHealth player)
    {
        // "I want to know when health changes"
        player.OnHealthChanged += RefreshBar;

        // "I want to know when the player dies"
        player.OnPlayerDied += ShowGameOver;
    }

    private void RefreshBar(int newHealth)
    {
        Console.WriteLine($"[UI] Health bar → {newHealth}/100");
    }

    private void ShowGameOver()
    {
        Console.WriteLine("[UI] Game Over screen shown");
    }
}

// ──────────────────────────────
// SUBSCRIBER 2 — sound manager
// ──────────────────────────────
public class SoundManager
{
    public SoundManager(PlayerHealth player)
    {
        player.OnHealthChanged += PlayHurtSound;
        player.OnPlayerDied    += PlayDeathSound;
    }

    private void PlayHurtSound(int newHealth)
    {
        Console.WriteLine("[SFX] Playing hurt sound");
    }

    private void PlayDeathSound()
    {
        Console.WriteLine("[SFX] Playing death music");
    }
}

// ──────────────────────────────
// USAGE
// ──────────────────────────────
static void Main()
{
    PlayerHealth player = new PlayerHealth();

    // Subscribe — order does not matter
    HealthUI     ui    = new HealthUI(player);
    SoundManager sound = new SoundManager(player);

    // PlayerHealth does not know about HealthUI or SoundManager at all
    player.TakeDamage(30);
    // [UI]  Health bar → 70/100
    // [SFX] Playing hurt sound

    player.TakeDamage(70);
    // [UI]  Health bar → 0/100
    // [SFX] Playing hurt sound
    // [UI]  Game Over screen shown
    // [SFX] Playing death music
}
```

> **Key insight:** If you add a new system tomorrow — say, `AchievementManager` — you do not touch `PlayerHealth` at all. `AchievementManager` just subscribes to the events it cares about. This is the power of the pattern.

---

### ✅ What you now know

- Raw public delegate fields are dangerous — anyone can invoke or reset them
- The `event` keyword restricts outside access to `+=` and `-=` only
- The class that declares the event is the only one that can invoke it
- The Publisher-Subscriber pattern decouples systems from each other

---

## 7. EventHandler & Custom EventArgs

### The C# standard convention

So far, events have used `Action` and `Action<T>` delegates. That works, but C# has an established convention that most .NET libraries follow:

```
void HandlerMethod(object sender, EventArgs e)
```

Where:
- `sender` — the object that fired the event (so the subscriber knows *who*)
- `e` — any extra data about the event (what happened, what changed, etc.)

C# provides a built-in delegate type for this exact signature: `EventHandler`.

---

### EventHandler — when there is no extra data

```csharp
public class Button
{
    // EventHandler matches: void Handler(object sender, EventArgs e)
    public event EventHandler OnClicked;

    public void Click()
    {
        Console.WriteLine("Button was clicked");
        OnClicked?.Invoke(this, EventArgs.Empty);
        //                 ^^^^  ^^^^^^^^^^^^^^^
        //                 |     No extra data — use the built-in empty instance
        //                 Pass 'this' so subscribers know which button fired
    }
}

// Subscribing:
Button btn = new Button();
btn.OnClicked += (sender, e) =>
{
    Button clickedBtn = (Button)sender; // cast to get back the Button that was clicked
    Console.WriteLine("Handled the click!");
};

btn.Click();
// Output:
// Button was clicked
// Handled the click!
```

---

### EventHandler\<TEventArgs\> — when you need to pass data

When your event needs to carry data (what changed, by how much, who caused it), create a custom `EventArgs` class.

**Step 1 — Create the custom EventArgs class:**

```csharp
// Always inherit from EventArgs
public class HealthChangedEventArgs : EventArgs
{
    // These are the data fields the event carries
    public int OldHealth { get; }
    public int NewHealth { get; }
    public int DamageTaken => OldHealth - NewHealth; // computed property

    // Constructor sets the data when the event is created
    public HealthChangedEventArgs(int oldHealth, int newHealth)
    {
        OldHealth = oldHealth;
        NewHealth = newHealth;
    }
}
```

**Step 2 — Use EventHandler\<T\> in the publisher:**

```csharp
public class PlayerHealth
{
    private int _health = 100;

    // EventHandler<HealthChangedEventArgs> matches:
    // void Handler(object sender, HealthChangedEventArgs e)
    public event EventHandler<HealthChangedEventArgs> OnHealthChanged;
    public event EventHandler OnPlayerDied;

    public void TakeDamage(int amount)
    {
        int oldHealth = _health;
        _health = Math.Max(0, _health - amount);

        // Create the event data and fire
        var args = new HealthChangedEventArgs(oldHealth, _health);
        OnHealthChanged?.Invoke(this, args);

        if (_health == 0)
            OnPlayerDied?.Invoke(this, EventArgs.Empty);
    }
}
```

**Step 3 — Subscribe and use the data:**

```csharp
PlayerHealth player = new PlayerHealth();

player.OnHealthChanged += (sender, e) =>
{
    Console.WriteLine($"Health changed: {e.OldHealth} → {e.NewHealth}");
    Console.WriteLine($"Damage taken:   {e.DamageTaken}");

    // We also know who fired it:
    PlayerHealth who = (PlayerHealth)sender;
};

player.OnPlayerDied += (sender, e) =>
{
    Console.WriteLine("Player has died.");
};

player.TakeDamage(30);
// Output:
// Health changed: 100 → 70
// Damage taken:   30

player.TakeDamage(70);
// Output:
// Health changed: 70 → 0
// Damage taken:   70
// Player has died.
```

---

### Action vs EventHandler — which to use?

| Situation | Use |
|---|---|
| Internal/simple events, game systems | `Action` / `Action<T>` — less boilerplate |
| Public APIs, library code, .NET conventions | `EventHandler<TEventArgs>` — standard pattern |
| Need to know who fired the event | `EventHandler` — has `sender` parameter |
| Just need to pass data, don't need sender | `Action<MyData>` is simpler |

Both are valid. In Unity game code, `Action` is common. In .NET library/framework code, `EventHandler<T>` is the standard.

---

### ✅ What you now know

- `EventHandler` is a built-in delegate for `void Handler(object sender, EventArgs e)`
- `EventHandler<T>` carries custom data by passing a typed `EventArgs` subclass
- `sender` lets subscribers know which object fired the event
- Create a custom `EventArgs` class to package data along with an event

---

## 8. Delegates & Events in Unity

### C# event vs UnityEvent

Unity has its own event system called `UnityEvent`. It is important to understand when to use each.

| | C# `event` | `UnityEvent` |
|---|---|---|
| **Visible in Inspector** | No — code only | ✅ Yes — designers can wire it up |
| **Performance** | Fast (compiled) | Slower (reflection-based) |
| **Serialized** | No | ✅ Yes (saved in scene/prefab) |
| **Type safety** | Strict — checked at compile time | Looser |
| **Best for** | Code-driven logic | Designer-driven hookups |

**Rule of thumb:**
- Use `UnityEvent` when a game designer should be able to connect things in the Inspector (e.g., button click → play animation)
- Use C# `event` for code-driven game systems (health, scoring, state machines, achievements)

---

### The most important Unity rule — always unsubscribe

In Unity, GameObjects and MonoBehaviours can be destroyed. If a subscriber is destroyed but its method is still registered with an event, the publisher holds a reference to a dead object. This causes `MissingReferenceException` errors or memory leaks.

**The golden rule: every `+=` must have a matching `-=`.**

The correct pattern is to subscribe in `OnEnable` and unsubscribe in `OnDisable`:

```csharp
public class HealthUI : MonoBehaviour
{
    [SerializeField] private PlayerHealth _playerHealth;

    private void OnEnable()
    {
        // Subscribe when this object becomes active
        _playerHealth.OnHealthChanged += UpdateHealthBar;
        _playerHealth.OnPlayerDied    += ShowGameOver;
    }

    private void OnDisable()
    {
        // Unsubscribe when this object is disabled or destroyed
        // OnDisable is called automatically before OnDestroy
        _playerHealth.OnHealthChanged -= UpdateHealthBar;
        _playerHealth.OnPlayerDied    -= ShowGameOver;
    }

    private void UpdateHealthBar(int newHealth)
    {
        // update the health bar UI
    }

    private void ShowGameOver()
    {
        // show game over panel
    }
}
```

> Why `OnEnable`/`OnDisable` and not `Awake`/`OnDestroy`?
> Because `OnEnable`/`OnDisable` fire every time the object is activated/deactivated, which means subscriptions stay in sync with the object's active state. If the object is disabled (not destroyed), it stops receiving events — which is usually the correct behaviour.

---

### Real Unity architecture example

Here is a typical event-driven architecture for a small game:

```csharp
// ─────────────────────────────────────────────────
// PlayerHealth.cs — the publisher
// ─────────────────────────────────────────────────
public class PlayerHealth : MonoBehaviour
{
    [SerializeField] private int _maxHealth = 100;
    private int _currentHealth;

    // C# events
    public event Action<int, int> OnHealthChanged; // (currentHp, maxHp)
    public event Action           OnPlayerDied;

    private void Awake()
    {
        _currentHealth = _maxHealth;
    }

    public void TakeDamage(int damage)
    {
        _currentHealth = Mathf.Max(0, _currentHealth - damage);
        OnHealthChanged?.Invoke(_currentHealth, _maxHealth); // fire event

        if (_currentHealth == 0)
            OnPlayerDied?.Invoke(); // fire event
    }

    public void Heal(int amount)
    {
        _currentHealth = Mathf.Min(_maxHealth, _currentHealth + amount);
        OnHealthChanged?.Invoke(_currentHealth, _maxHealth); // fire event
    }
}

// ─────────────────────────────────────────────────
// HealthUI.cs — subscriber 1
// ─────────────────────────────────────────────────
public class HealthUI : MonoBehaviour
{
    [SerializeField] private PlayerHealth _playerHealth;
    [SerializeField] private Slider       _healthSlider;
    [SerializeField] private TMP_Text     _healthText;

    private void OnEnable()
    {
        _playerHealth.OnHealthChanged += RefreshDisplay;
    }

    private void OnDisable()
    {
        _playerHealth.OnHealthChanged -= RefreshDisplay;
    }

    private void RefreshDisplay(int current, int max)
    {
        _healthSlider.value = (float)current / max;
        _healthText.text    = $"{current} / {max}";
    }
    // Notice: no Update() polling — this only runs when health actually changes
}

// ─────────────────────────────────────────────────
// SoundManager.cs — subscriber 2
// ─────────────────────────────────────────────────
public class SoundManager : MonoBehaviour
{
    [SerializeField] private PlayerHealth _playerHealth;
    [SerializeField] private AudioClip    _hurtClip;
    [SerializeField] private AudioClip    _deathClip;

    private void OnEnable()
    {
        _playerHealth.OnHealthChanged += PlayHurtSound;
        _playerHealth.OnPlayerDied    += PlayDeathSound;
    }

    private void OnDisable()
    {
        _playerHealth.OnHealthChanged -= PlayHurtSound;
        _playerHealth.OnPlayerDied    -= PlayDeathSound;
    }

    private void PlayHurtSound(int current, int max)
    {
        AudioSource.PlayClipAtPoint(_hurtClip, transform.position);
    }

    private void PlayDeathSound()
    {
        AudioSource.PlayClipAtPoint(_deathClip, transform.position);
    }
}

// ─────────────────────────────────────────────────
// GameManager.cs — subscriber 3
// ─────────────────────────────────────────────────
public class GameManager : MonoBehaviour
{
    [SerializeField] private PlayerHealth _playerHealth;
    private bool _isGameOver = false;

    private void OnEnable()
    {
        _playerHealth.OnPlayerDied += HandlePlayerDeath;
    }

    private void OnDisable()
    {
        _playerHealth.OnPlayerDied -= HandlePlayerDeath;
    }

    private void HandlePlayerDeath()
    {
        if (_isGameOver) return;
        _isGameOver = true;
        Debug.Log("Game Over — saving score and loading menu");
        // SceneManager.LoadScene("GameOver");
    }
}
```

> Notice that `PlayerHealth` has **zero references** to `HealthUI`, `SoundManager`, or `GameManager`. It is completely independent. You can add or remove any of those systems without touching `PlayerHealth` at all.

---

### Static events — global game events

For events that belong to the game itself (not a specific object), use static events on a static class:

```csharp
// GameEvents.cs — a central hub for game-wide events
public static class GameEvents
{
    public static event Action          OnGameStarted;
    public static event Action          OnGamePaused;
    public static event Action          OnGameResumed;
    public static event Action<int>     OnLevelCompleted;      // (level number)
    public static event Action<string>  OnAchievementUnlocked; // (achievement id)

    // Public raise methods — any class can call these to fire events
    public static void RaiseGameStarted()              => OnGameStarted?.Invoke();
    public static void RaiseLevelCompleted(int level)  => OnLevelCompleted?.Invoke(level);
    public static void RaiseAchievement(string id)     => OnAchievementUnlocked?.Invoke(id);
}

// Any MonoBehaviour subscribes — no reference to GameManager needed
public class AchievementBanner : MonoBehaviour
{
    private void OnEnable()  => GameEvents.OnAchievementUnlocked += ShowBanner;
    private void OnDisable() => GameEvents.OnAchievementUnlocked -= ShowBanner;

    private void ShowBanner(string id) => Debug.Log($"Achievement: {id}");
}

// Any MonoBehaviour can raise the event
public class EnemyKillTracker : MonoBehaviour
{
    private int _killCount = 0;

    public void RegisterKill()
    {
        _killCount++;
        if (_killCount == 1)
            GameEvents.RaiseAchievement("first_blood");
        if (_killCount == 10)
            GameEvents.RaiseAchievement("serial_killer");
    }
}
```

---

### ✅ What you now know

- C# events are better for code-driven game systems; `UnityEvent` is for Inspector-wired systems
- Always pair `+=` in `OnEnable` with `-=` in `OnDisable` — never leave subscriptions dangling
- Publishers should have zero references to their subscribers
- Static `GameEvents` classes are a clean pattern for broadcasting game-wide events

---

## 9. Common Patterns & Pitfalls

### Pattern — Delegates as callbacks

Pass a delegate to a method that will call it back when work is done. Common in loading, timers, and async operations.

```csharp
public class AssetLoader
{
    // onComplete is called with the loaded object; onError is called with the error message
    public void LoadAsync(string path, Action<GameObject> onComplete, Action<string> onError)
    {
        bool success = TryLoad(path, out GameObject asset);

        if (success)
            onComplete?.Invoke(asset);
        else
            onError?.Invoke($"Failed to load: {path}");
    }

    private bool TryLoad(string path, out GameObject asset)
    {
        asset = null; // simplified
        return false;
    }
}

// Usage — pass lambdas as the callbacks:
loader.LoadAsync(
    path:       "Prefabs/Enemy",
    onComplete: (go) => Instantiate(go, Vector3.zero, Quaternion.identity),
    onError:    (err) => Debug.LogError(err)
);
```

---

### Pattern — Delegates as strategies (swap logic at runtime)

Delegates let you replace a method's internal algorithm at runtime without subclassing.

```csharp
public class Sorter
{
    // The sort strategy is a delegate — can be swapped out
    public Func<int, int, int> Compare = (a, b) => a.CompareTo(b); // default: ascending

    public void Sort(List<int> list)
    {
        // ... uses Compare to compare elements
    }
}

Sorter s = new Sorter();
s.Sort(numbers);         // sorts ascending (default)

s.Compare = (a, b) => b.CompareTo(a); // swap to descending
s.Sort(numbers);         // now sorts descending — no new class needed
```

---

### Pitfall 1 — Forgetting to unsubscribe (memory leak / crash)

```csharp
// BAD — subscriber is destroyed but never unsubscribed
public class UIPanel : MonoBehaviour
{
    void Start()
    {
        GameManager.Instance.OnLevelComplete += ShowLevelComplete;
        // If this UIPanel is destroyed, GameManager still holds a reference to it
        // Next time OnLevelComplete fires → MissingReferenceException
    }
}

// GOOD
public class UIPanel : MonoBehaviour
{
    private void OnEnable()  => GameManager.Instance.OnLevelComplete += ShowLevelComplete;
    private void OnDisable() => GameManager.Instance.OnLevelComplete -= ShowLevelComplete;
    private void ShowLevelComplete() { /* ... */ }
}
```

---

### Pitfall 2 — Subscribing the same method multiple times

```csharp
// If this MonoBehaviour is disabled and re-enabled, Start() may not re-run,
// but if you use Awake instead, or call the method manually, double subscription can happen:
void Awake()
{
    player.OnDamaged += HandleDamage;
    player.OnDamaged += HandleDamage; // subscribed TWICE!
}
// Now HandleDamage fires TWICE per event — silent bug

// Prevention: use OnEnable/OnDisable pairing (they are always symmetric)
// Or: unsubscribe before subscribing to ensure it's added exactly once:
player.OnDamaged -= HandleDamage; // remove if already there
player.OnDamaged += HandleDamage; // add fresh
```

---

### Pitfall 3 — Modifying the subscriber list inside a handler

```csharp
void HandlePlayerDied()
{
    player.OnPlayerDied -= HandlePlayerDied; // modifying while the event is being invoked
    // This can cause unpredictable behaviour
    // Fix: use OnEnable/OnDisable pattern instead of self-unsubscribing
}
```

---

### Pitfall 4 — Using a return type with multicast (losing values)

```csharp
delegate int Calculator(int x);

Calculator calc = x => x + 1;
calc += x => x * 2;

int result = calc(5);
// First: 5 + 1 = 6 (discarded)
// Second: 5 * 2 = 10 (kept)
// result = 10

// If you need ALL results:
for (int i = 0; i < calc.GetInvocationList().Length; i++)
{
    int r = ((Calculator)calc.GetInvocationList()[i])(5);
    Console.WriteLine(r); // 6, then 10
}
```

---

### Pitfall 5 — Lambda closure trap in loops (recap)

```csharp
// BUG:
for (int i = 0; i < 3; i++)
    actions.Add(() => Console.WriteLine(i)); // captures 'i' by reference
// After loop: all print 3

// FIX:
for (int i = 0; i < 3; i++)
{
    int captured = i;                               // new variable per iteration
    actions.Add(() => Console.WriteLine(captured)); // captures 'captured'
}
// Prints: 0, 1, 2
```

---

## 10. Practice Exercises

Work through these to solidify your understanding. Start from the top and move down.

---

**Exercise 1 — Declare and invoke a delegate**

Declare a delegate type called `StringTransformer` that takes a string and returns a string.
Write two methods — `ToUpperCase` and `AddExclamation` — that match the delegate.
Assign each to a `StringTransformer` variable and invoke them with `"hello"`.

Expected output:
```
HELLO
hello!
```

---

**Exercise 2 — Multicast logger**

Create a delegate `LogWriter` that takes a string and returns void.
Write three methods: `ConsoleLog`, `PrefixLog` (adds `[INFO] ` prefix), and `UpperLog` (converts to uppercase).
Build a multicast delegate that calls all three when invoked, then remove `UpperLog` and call it again.

---

**Exercise 3 — Replace delegates with Action and Func**

Take this code and rewrite it using `Action<int>` and `Func<int, int>` instead of the custom delegate types:

```csharp
delegate void Printer(int value);
delegate int  Doubler(int value);

Printer print  = x => Console.WriteLine(x);
Doubler double = x => x * 2;
```

---

**Exercise 4 — Create a simple event system**

Create a class `Timer` with:
- A private counter that increments each "tick"
- An `event Action<int> OnTick` that fires each tick with the current count
- An `event Action OnComplete` that fires when count reaches 5

Create a `Display` class that subscribes to both events and prints the tick count, and shows "Timer done!" on complete.

---

**Exercise 5 — Unity architecture (on paper or in Unity)**

Design a `ScoreManager` class with:
- `event Action<int> OnScoreChanged`
- `event Action<int> OnHighScoreBeaten` (passes the new high score)
- `void AddScore(int points)` method

Write two subscribers: `ScoreUI` (updates a text display) and `AchievementTracker` (unlocks "First 100 points" if score crosses 100).

---

## 11. Quick Reference Cheat Sheet

### Declare a delegate type

```csharp
delegate void   MyAction(string msg);    // void, one string param
delegate int    MyFunc(int a, int b);    // returns int, two int params
delegate bool   MyCheck(string input);   // returns bool, one string param
```

### Assign a method

```csharp
MyAction d = SomeNamedMethod;           // assign a named method
MyAction d = (msg) => Console.Write(msg); // assign a lambda
```

### Invoke a delegate

```csharp
d("hello");            // direct call — throws if d is null
d?.Invoke("hello");    // null-safe — does nothing if d is null (preferred)
```

### Multicast

```csharp
d += AnotherMethod;    // add to invocation list
d -= AnotherMethod;    // remove from invocation list
```

### Built-in delegates

```csharp
Action              a = () => { };                   // no params, no return
Action<int>         a = x => { };                    // one param, no return
Action<int, string> a = (x, s) => { };               // two params, no return
Func<int>           f = () => 42;                    // no params, returns int
Func<int, int>      f = x => x * 2;                 // one param, returns int
Func<int, int, int> f = (a, b) => a + b;            // two params, returns int
Predicate<int>      p = x => x > 0;                 // one param, returns bool
```

### Declare an event

```csharp
public event Action            OnSomethingHappened;
public event Action<int>       OnValueChanged;
public event EventHandler      OnClicked;
public event EventHandler<T>   OnDataChanged;
```

### Raise an event (inside the publisher only)

```csharp
OnSomethingHappened?.Invoke();
OnValueChanged?.Invoke(newValue);
OnClicked?.Invoke(this, EventArgs.Empty);
OnDataChanged?.Invoke(this, new MyEventArgs(data));
```

### Subscribe and unsubscribe

```csharp
publisher.OnEvent += MyHandler;    // subscribe
publisher.OnEvent -= MyHandler;    // unsubscribe
```

### Custom EventArgs

```csharp
public class MyEventArgs : EventArgs
{
    public int Value { get; }
    public MyEventArgs(int value) => Value = value;
}

public event EventHandler<MyEventArgs> OnThing;
OnThing?.Invoke(this, new MyEventArgs(42));
```

### Unity subscribe/unsubscribe pattern

```csharp
private void OnEnable()  => _publisher.OnEvent += Handler;
private void OnDisable() => _publisher.OnEvent -= Handler;
```

---

*Guide authored for C# / Unity developers — zero to confident with delegates and events.*
*All examples compile against C# 8+ (.NET Standard 2.1 / Unity 2020+).*
