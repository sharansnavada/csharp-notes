# C# Delegates & Events: A Complete Deep-Dive Guide

> A hands-on guide for C# / Unity developers — starting from zero, going all the way to real-world patterns.

---

## Table of Contents

1. [The Problem Delegates Solve](#1-the-problem-delegates-solve)
2. [Delegate Basics](#2-delegate-basics)
3. [Multicast Delegates](#3-multicast-delegates)
4. [Built-in Generic Delegates — Action, Func, Predicate](#4-built-in-generic-delegates--action-func-predicate)
5. [Lambda Expressions & Anonymous Methods](#5-lambda-expressions--anonymous-methods)
6. [Events — The Publisher-Subscriber Pattern](#6-events--the-publisher-subscriber-pattern)
7. [EventHandler & Custom EventArgs](#7-eventhandler--custom-eventargs)
8. [Delegates & Events in Unity](#8-delegates--events-in-unity)
9. [Common Patterns & Pitfalls](#9-common-patterns--pitfalls)
10. [Quick Reference Cheat Sheet](#10-quick-reference-cheat-sheet)

---

## 1. The Problem Delegates Solve

Before understanding what a delegate *is*, understand the problem it *solves*.

### The scenario

You are writing a method that sorts a list of enemies by some criterion. Sometimes you want to sort by health, sometimes by distance to the player, sometimes by name. How do you write a single sort method that can do all three?

Without delegates, you'd write three separate sort methods — duplicating all the sorting logic. Or you'd use an ugly switch/if-else inside the method.

```csharp
// Without delegates — messy and not scalable
void SortEnemies(string criterion)
{
    if (criterion == "health")
        // sort by health...
    else if (criterion == "distance")
        // sort by distance...
    // What happens when you add a 4th criterion? Edit this file again.
}
```

### The real solution

What you actually want is to **pass the comparison logic itself** as an argument — pass a method, not just data.

```csharp
// What we want conceptually:
void SortEnemies(??? comparisonMethod)
{
    // use comparisonMethod to compare any two enemies
}

// Call it like this:
SortEnemies(CompareByHealth);
SortEnemies(CompareByDistance);
```

**Delegates make this possible.** A delegate is a type that can hold a reference to any method that matches a specific signature. Think of it as a variable that stores a method instead of a value.

### The mental model

| Regular variable | Delegate variable |
|---|---|
| Stores a value: `int x = 5;` | Stores a method: `MyDelegate d = DoSomething;` |
| You call it directly: `x + 3` | You call it: `d()` |
| Can be passed as argument | Can be passed as argument |
| Can be returned from a method | Can be returned from a method |

---

## 2. Delegate Basics

### Declaring a delegate type

A delegate declaration defines the **shape** (signature) of methods it can hold. It's a type, like `int` or `string`.

```csharp
// Syntax: delegate <returnType> <DelegateName>(<parameters>);

// A delegate for methods that take two ints and return an int
delegate int MathOperation(int a, int b);

// A delegate for methods that take a string and return nothing
delegate void Printer(string message);

// A delegate for methods that take no arguments and return a bool
delegate bool Checker();
```

### Instantiating a delegate

After declaring the type, you create an instance by pointing it at a method whose signature matches.

```csharp
class Calculator
{
    delegate int MathOperation(int a, int b);

    static int Add(int a, int b) => a + b;
    static int Multiply(int a, int b) => a * b;

    static void Main()
    {
        // Long form (explicit):
        MathOperation op = new MathOperation(Add);

        // Short form (implicit, preferred):
        MathOperation op2 = Add;

        // Calling the delegate (same as calling the method it points to):
        int result = op(3, 5);   // result = 8
        int result2 = op2(3, 5); // result2 = 8

        // Swap the method it points to:
        op = Multiply;
        int result3 = op(3, 5);  // result3 = 15
    }
}
```

### The method signature must match

The key rule: the method you assign to a delegate must have **the same return type and parameter types** as the delegate.

```csharp
delegate int MathOperation(int a, int b);

static int Add(int a, int b) => a + b;             // Works — matches exactly
static int Subtract(int x, int y) => x - y;        // Works — parameter NAMES don't matter
static float Divide(int a, int b) => (float)a / b; // ERROR — return type float ≠ int
static int Triple(int a) => a * 3;                 // ERROR — wrong number of parameters
```

### Null checking before invoking

A delegate variable that has never been assigned is `null`. Calling a null delegate throws a `NullReferenceException`.

```csharp
MathOperation op = null;
op(3, 5); // NullReferenceException — crash!

// Safe invocation using null-conditional operator (preferred modern style):
op?.Invoke(3, 5); // Does nothing if op is null — no crash

// Old style:
if (op != null)
    op(3, 5);
```

---

## 3. Multicast Delegates

A single delegate variable can hold **multiple methods** at the same time. When you invoke it, all the methods are called in the order they were added.

### Adding and removing methods

```csharp
delegate void Logger(string message);

static void PrintToConsole(string msg) => Console.WriteLine($"[Console] {msg}");
static void PrintToFile(string msg)    => File.AppendAllText("log.txt", msg + "\n");
static void PrintToServer(string msg)  => SendToServer(msg); // hypothetical

static void Main()
{
    Logger log = null;

    // Add methods with +=
    log += PrintToConsole;
    log += PrintToFile;
    log += PrintToServer;

    // Calling log("Hello") now calls all three, in order
    log?.Invoke("Hello");
    // Output:
    // [Console] Hello
    // log.txt gets "Hello" appended
    // Server receives "Hello"

    // Remove a specific method with -=
    log -= PrintToFile;

    log?.Invoke("Goodbye");
    // Now only Console and Server receive "Goodbye"
}
```

### Return values with multicast delegates

If a delegate has a return type (other than `void`), and you assign multiple methods to it, **only the return value of the last method is captured**. The others are discarded.

```csharp
delegate int MathOp(int a, int b);

static int Add(int a, int b)      { Console.WriteLine("Add called");      return a + b; }
static int Multiply(int a, int b) { Console.WriteLine("Multiply called");  return a * b; }

static void Main()
{
    MathOp op = Add;
    op += Multiply;

    int result = op(3, 5);
    // Output:
    // Add called        (returns 8, discarded)
    // Multiply called   (returns 15, captured)

    Console.WriteLine(result); // 15
}
```

> **Rule of thumb:** Multicast delegates with return values are unusual. If you need all return values, iterate the `GetInvocationList()` manually. For most use cases with multicast delegates, prefer `void` return types (like event handlers).

### Inspecting the invocation list

```csharp
Logger log = PrintToConsole;
log += PrintToFile;

// Get an array of all methods in the delegate:
Delegate[] list = log.GetInvocationList();
Console.WriteLine(list.Length); // 2
```

---

## 4. Built-in Generic Delegates — Action, Func, Predicate

Declaring your own delegate type every time is verbose. C# provides ready-made generic delegate types that cover almost every scenario.

### Action — for void-returning methods

`Action` holds a method that returns nothing (`void`).

```csharp
// Action with no parameters
Action greet = () => Console.WriteLine("Hello!");
greet(); // Hello!

// Action<T> — one parameter, no return
Action<string> printName = (name) => Console.WriteLine($"Name: {name}");
printName("Sharan"); // Name: Sharan

// Action<T1, T2> — two parameters, no return
Action<string, int> printScore = (name, score) =>
    Console.WriteLine($"{name}: {score} points");
printScore("Sharan", 95); // Sharan: 95 points

// Goes up to Action<T1, T2, ..., T16> — 16 parameters max
```

### Func — for methods that return a value

`Func` holds a method that returns a value. The **last** type parameter is always the **return type**.

```csharp
// Func<TResult> — no parameters, returns TResult
Func<int> getRandomNumber = () => new Random().Next(1, 100);
int n = getRandomNumber(); // e.g. 42

// Func<T, TResult> — one parameter, returns TResult
Func<int, int> square = (x) => x * x;
int squared = square(5); // 25

// Func<T1, T2, TResult> — two parameters, returns TResult
Func<int, int, int> add = (a, b) => a + b;
int sum = add(3, 5); // 8

// Func<string, int, bool> — two params (string, int), returns bool
Func<string, int, bool> hasMinLength = (text, minLen) => text.Length >= minLen;
bool result = hasMinLength("Hello", 3); // true

// Goes up to Func<T1, ..., T16, TResult> — up to 16 params + 1 return
```

### Predicate — for bool-returning methods with one parameter

`Predicate<T>` is a shorthand for `Func<T, bool>`. It's commonly used with list filtering.

```csharp
Predicate<int> isEven = (x) => x % 2 == 0;
bool result = isEven(4); // true

// Using with List<T>.FindAll:
List<int> numbers = new List<int> { 1, 2, 3, 4, 5, 6 };
List<int> evens = numbers.FindAll(isEven); // [2, 4, 6]

// Inline:
List<int> odds = numbers.FindAll(x => x % 2 != 0); // [1, 3, 5]
```

### When to use each

| Use | When |
|---|---|
| `Action` | Method does work but returns nothing |
| `Func` | Method computes and returns a value |
| `Predicate<T>` | Method tests a condition and returns bool |
| Custom `delegate` | You need a very specific signature with ref/out params, or want a named type for documentation clarity |

---

## 5. Lambda Expressions & Anonymous Methods

Delegates need methods to point to — but you don't always want to write a named method for a small piece of logic. That's where lambdas and anonymous methods come in.

### Anonymous methods (older syntax — C# 2.0)

```csharp
// Without lambda — you'd write a named method:
static void PrintMessage(string msg) { Console.WriteLine(msg); }

Action<string> printer = PrintMessage;

// With anonymous method — inline, no name needed:
Action<string> printer = delegate(string msg)
{
    Console.WriteLine(msg);
};

printer("Hello!"); // Hello!
```

### Lambda expressions (modern syntax — C# 3.0+, preferred)

Lambdas are the clean, modern replacement for anonymous methods.

```csharp
// Syntax: (parameters) => expression_or_block

// Expression lambda — single expression, no curly braces, implicit return
Func<int, int> square = x => x * x;   // When one param, parentheses optional
Func<int, int, int> add = (a, b) => a + b;

// Statement lambda — multiple statements, needs curly braces
Func<int, int> absValue = x =>
{
    if (x < 0)
        return -x;
    return x;
};

// Void lambda (Action)
Action<string> log = msg => Console.WriteLine($"[LOG] {msg}");
log("Server started"); // [LOG] Server started
```

### Closures — capturing outer variables

A lambda can "capture" variables from its surrounding scope. This is called a **closure**.

```csharp
int multiplier = 3; // outer variable

Func<int, int> tripler = x => x * multiplier; // captures 'multiplier'

Console.WriteLine(tripler(5));  // 15
Console.WriteLine(tripler(10)); // 30

// IMPORTANT: the lambda captures the VARIABLE, not its value
multiplier = 10;
Console.WriteLine(tripler(5)); // 50! (not 15)
// The lambda uses the current value of multiplier, not what it was at capture time
```

### Practical example — using lambdas to avoid method clutter

```csharp
List<Enemy> enemies = GetAllEnemies();

// Sort by health — no need for a named CompareByHealth method
enemies.Sort((a, b) => a.Health.CompareTo(b.Health));

// Filter enemies close to player
List<Enemy> nearbyEnemies = enemies.FindAll(e => e.DistanceToPlayer < 10f);

// Transform list (LINQ)
List<string> names = enemies.ConvertAll(e => e.Name);
```

---

## 6. Events — The Publisher-Subscriber Pattern

### Why delegates alone aren't enough for communication

Delegates are powerful, but using a raw delegate as a public field for inter-object communication has serious problems:

```csharp
public class PlayerHealth
{
    public Action OnDeath; // raw public delegate — dangerous!
}

// Problem 1: Anyone can reset the whole invocation list
player.OnDeath = null; // wipes out ALL subscribers — disaster!

// Problem 2: Anyone can invoke it from outside
player.OnDeath(); // enemy invokes the death event? Wrong!

// Problem 3: Anyone can read the invocation list (encapsulation broken)
Delegate[] subscribers = player.OnDeath.GetInvocationList();
```

### The event keyword fixes this

Adding `event` to a delegate field restricts it so that outside classes can only `+=` and `-=`. They **cannot invoke, reset, or inspect** it.

```csharp
public class PlayerHealth
{
    public event Action OnDeath; // now an EVENT — safe!

    private void Die()
    {
        OnDeath?.Invoke(); // Only THIS class can invoke the event
    }
}

PlayerHealth player = new PlayerHealth();

player.OnDeath += SomeMethod;   // OK — subscribing
player.OnDeath -= SomeMethod;   // OK — unsubscribing
player.OnDeath();               // ERROR — outside code cannot invoke events
player.OnDeath = null;          // ERROR — outside code cannot reset events
```

### The Publisher-Subscriber pattern

This is the core pattern events enable. It decouples the class that raises the event (the **publisher**) from the classes that react to it (the **subscribers**).

```csharp
// PUBLISHER — the class that owns and fires the event
public class PlayerHealth
{
    private int _health = 100;

    public event Action OnPlayerDied;
    public event Action<int> OnHealthChanged; // passes new health value

    public void TakeDamage(int amount)
    {
        _health -= amount;
        OnHealthChanged?.Invoke(_health); // notify subscribers

        if (_health <= 0)
            OnPlayerDied?.Invoke();       // notify subscribers
    }
}

// SUBSCRIBERS — classes that listen and react
public class GameUI
{
    private PlayerHealth _player;

    public GameUI(PlayerHealth player)
    {
        _player = player;
        _player.OnHealthChanged += UpdateHealthBar;
        _player.OnPlayerDied    += ShowGameOverScreen;
    }

    private void UpdateHealthBar(int newHealth)
    {
        Console.WriteLine($"Health bar: {newHealth}/100");
    }

    private void ShowGameOverScreen()
    {
        Console.WriteLine("Game Over!");
    }
}

public class SoundManager
{
    private PlayerHealth _player;

    public SoundManager(PlayerHealth player)
    {
        _player = player;
        _player.OnPlayerDied += PlayDeathSound;
    }

    private void PlayDeathSound()
    {
        Console.WriteLine("Playing death sound...");
    }
}
```

> **Key insight:** `PlayerHealth` has no idea that `GameUI` or `SoundManager` exist. It just fires events. The subscribers react independently. Adding a new system (e.g., `AchievementManager`) requires zero changes to `PlayerHealth` — just subscribe to its events.

---

## 7. EventHandler & Custom EventArgs

While you can use `Action` or custom delegates for events, C# has a standard convention that most libraries and Unity patterns follow.

### The standard convention

The C# convention for events is:

```
void EventHandlerMethod(object sender, EventArgs e)
```

- `sender` — the object that raised the event (the publisher), so the subscriber knows *who* fired it
- `e` — additional data about the event (what happened)

C# provides the `EventHandler` delegate type that matches this signature exactly.

### EventHandler (no custom data)

```csharp
public class Button
{
    // Use EventHandler for events that send no extra data
    public event EventHandler OnClicked;

    public void Click()
    {
        OnClicked?.Invoke(this, EventArgs.Empty);
        //                ^     ^
        //                |     Empty EventArgs — no extra data
        //                the sender (this button)
    }
}

// Subscribing:
Button btn = new Button();
btn.OnClicked += (sender, e) =>
{
    Button clickedBtn = (Button)sender;
    Console.WriteLine("Button was clicked!");
};
```

### EventHandler<TEventArgs> with custom data

When you need to pass data with the event, create a custom `EventArgs` class.

```csharp
// Step 1: Create a custom EventArgs class (always inherit from EventArgs)
public class HealthChangedEventArgs : EventArgs
{
    public int OldHealth { get; }
    public int NewHealth { get; }
    public int DamageTaken => OldHealth - NewHealth;

    public HealthChangedEventArgs(int oldHealth, int newHealth)
    {
        OldHealth = oldHealth;
        NewHealth = newHealth;
    }
}

// Step 2: Use EventHandler<T> in your publisher
public class PlayerHealth
{
    private int _health = 100;

    // Typed event with custom data
    public event EventHandler<HealthChangedEventArgs> OnHealthChanged;
    public event EventHandler OnPlayerDied;

    public void TakeDamage(int amount)
    {
        int oldHealth = _health;
        _health = Math.Max(0, _health - amount);

        // Fire event with data
        OnHealthChanged?.Invoke(this, new HealthChangedEventArgs(oldHealth, _health));

        if (_health == 0)
            OnPlayerDied?.Invoke(this, EventArgs.Empty);
    }
}

// Step 3: Subscribe and use the data
public class HUD
{
    public HUD(PlayerHealth player)
    {
        player.OnHealthChanged += HandleHealthChanged;
        player.OnPlayerDied    += HandlePlayerDied;
    }

    private void HandleHealthChanged(object sender, HealthChangedEventArgs e)
    {
        Console.WriteLine($"Health changed: {e.OldHealth} → {e.NewHealth}");
        Console.WriteLine($"Damage taken: {e.DamageTaken}");

        // We even know which player fired this:
        PlayerHealth who = (PlayerHealth)sender;
    }

    private void HandlePlayerDied(object sender, EventArgs e)
    {
        Console.WriteLine("Player has died — show game over screen");
    }
}
```

---

## 8. Delegates & Events in Unity

Unity has its own `UnityEvent` type, but C# events are widely used too. Understanding the difference and knowing when to use each is key.

### C# event vs UnityEvent

| | C# event | UnityEvent |
|---|---|---|
| **Inspector visible** | No | Yes — can wire up in Editor |
| **Performance** | Faster | Slower (reflection-based) |
| **Type safety** | Strict at compile time | Looser |
| **Usage** | Code-driven systems | Designer-driven systems |
| **Persistence** | No (runtime only) | Yes (serialized in scene) |

**Use C# events** for code-driven game systems (health, scoring, achievements, game state).  
**Use UnityEvent** when designers need to wire things up in the Inspector without code.

### Important: subscribe/unsubscribe in Unity

In Unity, if you subscribe to an event in `Awake` or `Start` and the subscriber GameObject is destroyed, the publisher still holds a reference to the dead object — this causes errors or memory leaks.

**Always pair subscriptions and unsubscriptions:**

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
        _playerHealth.OnHealthChanged -= UpdateHealthBar;
        _playerHealth.OnPlayerDied    -= ShowGameOver;
    }

    private void UpdateHealthBar(int newHealth)
    {
        // update UI
    }

    private void ShowGameOver()
    {
        // show game over panel
    }
}
```

> **Golden rule in Unity:** Every `+=` in `OnEnable` must have a matching `-=` in `OnDisable`.

### Real Unity example — Game Manager system

Here's a typical architecture using events across multiple game systems:

```csharp
// ────────────────────────────────────────────────────
// PlayerHealth.cs
// ────────────────────────────────────────────────────
public class PlayerHealth : MonoBehaviour
{
    [SerializeField] private int _maxHealth = 100;
    private int _currentHealth;

    public event Action<int, int> OnHealthChanged; // (currentHp, maxHp)
    public event Action OnPlayerDied;

    private void Awake()
    {
        _currentHealth = _maxHealth;
    }

    public void TakeDamage(int damage)
    {
        _currentHealth = Mathf.Max(0, _currentHealth - damage);
        OnHealthChanged?.Invoke(_currentHealth, _maxHealth);

        if (_currentHealth == 0)
        {
            OnPlayerDied?.Invoke();
            gameObject.SetActive(false);
        }
    }

    public void Heal(int amount)
    {
        _currentHealth = Mathf.Min(_maxHealth, _currentHealth + amount);
        OnHealthChanged?.Invoke(_currentHealth, _maxHealth);
    }
}

// ────────────────────────────────────────────────────
// ScoreManager.cs
// ────────────────────────────────────────────────────
public class ScoreManager : MonoBehaviour
{
    private int _score;

    public event Action<int> OnScoreChanged;

    public void AddScore(int points)
    {
        _score += points;
        OnScoreChanged?.Invoke(_score);
    }
}

// ────────────────────────────────────────────────────
// GameManager.cs — central hub, listens to both
// ────────────────────────────────────────────────────
public class GameManager : MonoBehaviour
{
    [SerializeField] private PlayerHealth _playerHealth;
    [SerializeField] private ScoreManager _scoreManager;

    private bool _isGameOver;

    private void OnEnable()
    {
        _playerHealth.OnPlayerDied    += HandlePlayerDeath;
        _scoreManager.OnScoreChanged  += HandleScoreChanged;
    }

    private void OnDisable()
    {
        _playerHealth.OnPlayerDied    -= HandlePlayerDeath;
        _scoreManager.OnScoreChanged  -= HandleScoreChanged;
    }

    private void HandlePlayerDeath()
    {
        if (_isGameOver) return;
        _isGameOver = true;
        Debug.Log("Player died — triggering game over sequence");
        // Load game over screen, save score, etc.
    }

    private void HandleScoreChanged(int newScore)
    {
        Debug.Log($"Score updated: {newScore}");
        // Check for high score, trigger achievements, etc.
    }
}

// ────────────────────────────────────────────────────
// HealthUI.cs — purely reactive, no polling needed
// ────────────────────────────────────────────────────
public class HealthUI : MonoBehaviour
{
    [SerializeField] private PlayerHealth _playerHealth;
    [SerializeField] private Slider _healthSlider;
    [SerializeField] private TMP_Text _healthText;

    private void OnEnable()
    {
        _playerHealth.OnHealthChanged += RefreshHealthDisplay;
    }

    private void OnDisable()
    {
        _playerHealth.OnHealthChanged -= RefreshHealthDisplay;
    }

    private void RefreshHealthDisplay(int current, int max)
    {
        _healthSlider.value = (float)current / max;
        _healthText.text = $"{current} / {max}";
    }
}
```

> Notice how `HealthUI` never polls `_playerHealth.CurrentHealth` in `Update()`. It just reacts when the event fires — zero wasted CPU cycles checking for changes every frame.

### Static events — global communication

For truly global events (no specific sender), static events are common in Unity:

```csharp
// GameEvents.cs — a single static class to hold all game events
public static class GameEvents
{
    public static event Action OnGameStarted;
    public static event Action OnGamePaused;
    public static event Action OnGameResumed;
    public static event Action<int> OnLevelCompleted; // (level number)
    public static event Action<string> OnAchievementUnlocked;

    // Invoke helpers (only this class can fire them from outside, or make these internal)
    public static void RaiseGameStarted()     => OnGameStarted?.Invoke();
    public static void RaiseLevelCompleted(int level) => OnLevelCompleted?.Invoke(level);
    public static void RaiseAchievement(string id)    => OnAchievementUnlocked?.Invoke(id);
}

// Any MonoBehaviour can subscribe:
public class AchievementPanel : MonoBehaviour
{
    private void OnEnable()
    {
        GameEvents.OnAchievementUnlocked += ShowAchievementPopup;
    }

    private void OnDisable()
    {
        GameEvents.OnAchievementUnlocked -= ShowAchievementPopup;
    }

    private void ShowAchievementPopup(string id)
    {
        Debug.Log($"Achievement unlocked: {id}");
    }
}

// Any MonoBehaviour can raise:
public class EnemyKillSystem : MonoBehaviour
{
    public void OnEnemyKilled(Enemy enemy)
    {
        if (IsFirstKill())
            GameEvents.RaiseAchievement("first_blood");
    }
}
```

---

## 9. Common Patterns & Pitfalls

### Pattern 1 — Null-conditional invocation (always use this)

```csharp
// Old style — verbose and not thread-safe
if (OnHealthChanged != null)
    OnHealthChanged(newHealth);

// Modern style — concise, preferred
OnHealthChanged?.Invoke(newHealth);
// ?.Invoke() checks for null AND calls in one atomic step — safer in multithreaded code
```

### Pattern 2 — Passing delegates as method arguments (callbacks)

```csharp
// A loading system that calls back when done
public class AssetLoader
{
    public void LoadAsync(string path, Action<GameObject> onComplete, Action<string> onError)
    {
        // ... loading logic ...

        bool success = TryLoad(path, out GameObject asset);
        if (success)
            onComplete?.Invoke(asset);
        else
            onError?.Invoke($"Failed to load: {path}");
    }
}

// Usage:
loader.LoadAsync(
    "Prefabs/Enemy",
    onComplete: (go) => Instantiate(go, spawnPoint.position, Quaternion.identity),
    onError:    (err) => Debug.LogError(err)
);
```

### Pattern 3 — Delegates as strategy (strategy pattern)

```csharp
public class EnemyAI
{
    // The attack strategy is a delegate — easily swappable
    public Func<Enemy, Player, float> DamageCalculator;

    public void Attack(Player target)
    {
        float damage = DamageCalculator?.Invoke(this as Enemy, target) ?? 10f;
        target.TakeDamage(damage);
    }
}

// Boss enemy uses a different formula — no subclassing needed
boss.DamageCalculator = (enemy, player) =>
    10f * (1f - player.ArmorRating) * enemy.RageMultiplier;
```

### Pitfall 1 — Forgetting to unsubscribe (memory leaks)

```csharp
// BAD — if UIPanel is destroyed, the publisher still holds a reference
public class UIPanel : MonoBehaviour
{
    void Start()
    {
        GameManager.Instance.OnLevelComplete += ShowLevelCompleteScreen;
        // Never unsubscribed — memory leak and potential NullReference
    }
}

// GOOD
public class UIPanel : MonoBehaviour
{
    private void OnEnable()  => GameManager.Instance.OnLevelComplete += ShowLevelCompleteScreen;
    private void OnDisable() => GameManager.Instance.OnLevelComplete -= ShowLevelCompleteScreen;

    private void ShowLevelCompleteScreen() { /* ... */ }
}
```

### Pitfall 2 — Subscribing multiple times

```csharp
// If Start() is called more than once (or the component is re-enabled incorrectly),
// you can accidentally subscribe the SAME method multiple times:
void Start()
{
    player.OnDamaged += HandleDamage; // if this runs twice, HandleDamage fires twice per event!
}

// Fix: use OnEnable/OnDisable pairing (they're symmetric and safe)
// Or: -= before += to ensure single subscription:
player.OnDamaged -= HandleDamage;
player.OnDamaged += HandleDamage;
```

### Pitfall 3 — Modifying the invocation list while iterating

```csharp
// Dangerous: subscribing/unsubscribing inside an event handler
void HandlePlayerDied()
{
    player.OnPlayerDied -= HandlePlayerDied; // Risky — modifying while invoking
    // Use OnDisable() pattern instead
}
```

### Pitfall 4 — Closure variable capture bug

```csharp
// BUG: classic closure trap in a loop
List<Action> actions = new List<Action>();
for (int i = 0; i < 5; i++)
{
    actions.Add(() => Console.WriteLine(i)); // captures 'i' by reference
}
actions.ForEach(a => a()); // prints "5 5 5 5 5" — all see i=5 after loop ends

// FIX: capture a copy of the loop variable
for (int i = 0; i < 5; i++)
{
    int copy = i;
    actions.Add(() => Console.WriteLine(copy)); // captures 'copy', which is a new variable each iteration
}
actions.ForEach(a => a()); // prints "0 1 2 3 4"
```

---

## 10. Quick Reference Cheat Sheet

### Declare a delegate type

```csharp
delegate void MyAction(string msg);        // void, one param
delegate int  MyFunc(int a, int b);        // returns int, two params
```

### Assign and invoke

```csharp
MyAction d = SomeMethod;      // assign named method
MyAction d = (msg) => { };   // assign lambda
d?.Invoke("hello");           // null-safe invoke
d("hello");                   // direct invoke (throws if null)
```

### Built-in delegates

```csharp
Action                  a = () => { };
Action<int>             a = (x) => { };
Action<int, string>     a = (x, s) => { };
Func<int>               f = () => 42;
Func<int, int>          f = (x) => x * 2;
Func<int, int, bool>    f = (a, b) => a > b;
Predicate<int>          p = (x) => x > 0;
```

### Multicast

```csharp
Action d = MethodA;
d += MethodB;     // add
d -= MethodA;     // remove
d?.Invoke();      // calls MethodB only
```

### Declare an event

```csharp
public event Action OnDied;
public event Action<int> OnHealthChanged;
public event EventHandler<MyEventArgs> OnCustomEvent;
```

### Raise an event (inside the publisher only)

```csharp
OnDied?.Invoke();
OnHealthChanged?.Invoke(newHealth);
OnCustomEvent?.Invoke(this, new MyEventArgs(data));
```

### Subscribe / Unsubscribe

```csharp
publisher.OnDied += HandleDeath;    // subscribe
publisher.OnDied -= HandleDeath;    // unsubscribe
```

### Custom EventArgs

```csharp
public class MyEventArgs : EventArgs
{
    public int Value { get; }
    public MyEventArgs(int value) => Value = value;
}

public event EventHandler<MyEventArgs> OnThingHappened;
OnThingHappened?.Invoke(this, new MyEventArgs(42));
```

### Unity OnEnable / OnDisable pairing

```csharp
private void OnEnable()  => _publisher.OnEvent += Handler;
private void OnDisable() => _publisher.OnEvent -= Handler;
```

---

*Guide authored for C# / Unity developers. All examples compile against C# 8+ (.NET Standard 2.1 / Unity 2020+).*
