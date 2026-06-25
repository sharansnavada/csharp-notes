# Recursion and Backtracking in C#

> **Who is this for?**
> This guide assumes **zero prior knowledge** of recursion or backtracking, and only basic C# knowledge (variables, if statements, loops, methods, and lists). Every concept is built from the ground up with analogies, diagrams, full traces, and complete runnable code. By the end, you will be able to read, write, debug, and apply recursive and backtracking solutions to real problems.

---

## Table of Contents

1. [Why Learn This? — Real-World Applications](#1-why-learn-this--real-world-applications)
2. [What is Recursion?](#2-what-is-recursion)
3. [The Anatomy of a Recursive Method](#3-the-anatomy-of-a-recursive-method)
4. [How to Think Recursively — The Mental Model](#4-how-to-think-recursively--the-mental-model)
5. [How the Call Stack Works](#5-how-the-call-stack-works)
6. [How to Read Recursive Code Written by Others](#6-how-to-read-recursive-code-written-by-others)
7. [Classic Recursion Problems](#7-classic-recursion-problems)
8. [Types of Recursion](#8-types-of-recursion)
9. [Memoization — Making Recursion Fast](#9-memoization--making-recursion-fast)
10. [Common Mistakes and How to Avoid Them](#10-common-mistakes-and-how-to-avoid-them)
11. [What is Backtracking?](#11-what-is-backtracking)
12. [The Backtracking Template](#12-the-backtracking-template)
13. [Pruning — Making Backtracking Smarter](#13-pruning--making-backtracking-smarter)
14. [Classic Backtracking Problems](#14-classic-backtracking-problems)
15. [Recursion vs Iteration — When to Use Which](#15-recursion-vs-iteration--when-to-use-which)
16. [Problem Pattern Recognition — LeetCode & Interviews](#16-problem-pattern-recognition--leetcode--interviews)
17. [Tips for Solving Recursive Problems](#17-tips-for-solving-recursive-problems)
18. [Practice Problems](#18-practice-problems)
19. [Quick Reference Cheat Sheet](#19-quick-reference-cheat-sheet)

---

## 1. Why Learn This? — Real-World Applications

Before writing a single line of code, you need to know *why* this matters. Recursion and backtracking are not just interview topics — they appear in real production code every day.

### Where Recursion is Used

**File System Traversal**
When Windows Explorer or macOS Finder shows you a folder and all its subfolders, that is recursion. Every folder can contain more folders, which can contain more folders — a naturally recursive structure.

```
Documents/
├── Work/
│   ├── Projects/
│   │   └── Report.docx
│   └── Meetings/
└── Personal/
    └── Photos/
```

To list every file in `Documents`, you recurse into each subfolder.

**Game Development — Unity Scene Hierarchy**
In Unity, every `GameObject` can have child `GameObjects`, which can have their own children. The scene hierarchy *is* a recursive tree. When Unity calls `Update()`, renders objects, or searches for a component, it recursively walks this tree. When you write code like `transform.Find("Head/Eyes/LeftEye")`, Unity is doing recursive tree traversal internally.

**JSON and XML Parsing**
JSON objects can contain arrays, which can contain objects, which can contain arrays... Parsers for these formats use recursion because the data structure is recursive by definition.

**Sorting Algorithms**
Merge Sort and Quick Sort — two of the most commonly used sorting algorithms in computer science — are purely recursive. When C# sorts large collections internally, it uses a variant of these.

**Web Crawlers and Sitemap Generators**
A web crawler visits a page, finds all links on it, then visits each link and finds more links. That is recursive tree traversal across the internet.

**Where Backtracking is Used**

**Puzzle Solvers**
Sudoku apps, crossword solvers, and escape room puzzle systems all use backtracking to systematically try possibilities and abandon dead ends.

**Route Planning**
GPS and navigation systems use backtracking-based algorithms to explore routes and eliminate paths that exceed distance or time constraints.

**Compiler Design**
When a compiler parses your source code, it uses backtracking grammars to figure out the structure of expressions like `a + b * (c - d)`.

**AI and Game Bots**
Chess engines explore the game tree of possible moves using backtracking. The engine tries a move, evaluates the position, then backtracks and tries other moves to find the best one.

---

## 2. What is Recursion?

**Recursion** is when a method calls itself to solve a smaller version of the same problem.

### The Mirror Analogy

Imagine standing between two mirrors facing each other. You see a reflection, and inside that reflection is another reflection, getting smaller and smaller. Eventually the image disappears — that is the stopping condition (called the **base case**) preventing infinite repetition.

### The Chocolate Box Analogy

> "How many chocolates are in the box?"
> - Pick one chocolate out.
> - Ask: *"How many are left in the box now?"*
> - Keep removing one and asking — until the box is empty (0 chocolates).
> - Then count back up: 0 + 1 + 1 + 1 ... = total.

This is recursion: break the problem down by one step at a time, until you hit the simplest possible version.

### The Russian Doll Analogy

A matryoshka doll contains a smaller doll, which contains a smaller doll, which contains an even smaller doll — until you reach the tiniest doll that contains nothing. Opening the outer doll to find the smallest is exactly how recursive calls go deeper and deeper until the base case is reached.

---

## 3. The Anatomy of a Recursive Method

Every recursive method has exactly **two parts**. Missing either one breaks everything.

```
┌─────────────────────────────────────────────────────────┐
│                  RECURSIVE METHOD                       │
│                                                         │
│  1. BASE CASE                                           │
│     The stopping condition.                             │
│     Answers the simplest version directly.              │
│     No recursive call is made here.                     │
│                                                         │
│  2. RECURSIVE CASE                                      │
│     Calls itself with a smaller or simpler input.       │
│     Moves one step closer to the base case each time.   │
└─────────────────────────────────────────────────────────┘
```

### Simplest Example: Countdown

```csharp
static void Countdown(int n)
{
    // BASE CASE: stop when we reach 0
    if (n == 0)
    {
        Console.WriteLine("Go!");
        return;
    }

    // RECURSIVE CASE: print n, then solve for n - 1
    Console.WriteLine(n);
    Countdown(n - 1);  // calls itself with a SMALLER value
}

Countdown(5);
```

**Output:**
```
5
4
3
2
1
Go!
```

Notice:
- Each call receives a **smaller** `n`.
- Eventually `n` reaches `0` (base case) and the chain stops.

---

## 4. How to Think Recursively — The Mental Model

This is the hardest part for most beginners, and the most important. Most people try to trace the entire call chain in their head — that is the wrong approach and it makes your brain explode.

Instead, use this **three-question framework** every time you write a recursive method.

---

### The Three Questions

**Question 1: What is the base case?**
> What is the *simplest possible input* for which I can answer directly, without calling myself?

**Question 2: What is one recursive step?**
> How can I express the answer for input `n` in terms of the answer for a slightly smaller input like `n-1`?

**Question 3: Do I trust the recursion?**
> Assume that `MyMethod(n-1)` already returns the correct answer for `n-1`. What do I do with it to get the answer for `n`?

---

### Applying the Framework: Factorial

> n! = 1 × 2 × 3 × ... × n

**Q1: Base case?**
What is the simplest factorial? `0! = 1`. We can answer this directly.

**Q2: One recursive step?**
`Factorial(n) = n × Factorial(n-1)`. Express `n!` in terms of `(n-1)!`.

**Q3: Trust the recursion?**
If `Factorial(n-1)` already gives me `(n-1)!` correctly, I just multiply `n` by it.

```csharp
static int Factorial(int n)
{
    if (n == 0) return 1;               // Q1: base case
    return n * Factorial(n - 1);        // Q2 + Q3: trust the result
}
```

---

### Applying the Framework: Sum of Array

**Q1: Base case?**
If the array is empty (index has gone past the end), the sum is `0`.

**Q2: One recursive step?**
`Sum(arr, i) = arr[i] + Sum(arr, i + 1)`. Current element plus the sum of everything after it.

**Q3: Trust the recursion?**
If `Sum(arr, i + 1)` correctly gives the sum from `i+1` to the end, I just add `arr[i]` to it.

```csharp
static int Sum(int[] arr, int index)
{
    if (index == arr.Length) return 0;          // Q1: base case
    return arr[index] + Sum(arr, index + 1);    // Q2 + Q3
}
```

---

### The Golden Rule

> **Never try to trace the full call chain in your head while writing recursive code.**
> Define the base case, define ONE recursive step, and trust that the rest works.
> Verify by tracing only for `n = 0` and `n = 1`.

---

## 5. How the Call Stack Works

The call stack is what makes recursion actually work in memory. Understanding it explains both *why recursion works* and *why it can crash*.

### What is the Call Stack?

Think of the call stack as a stack of plates. Every time a method is called, a new plate (called a **stack frame**) is placed on top. The plate holds the local variables and the point to return to when the method finishes. When a method returns, its plate is removed.

### Visualising `Countdown(3)`

```
PLATES BEING ADDED (going deeper):

┌─────────────────┐
│ Countdown(0)    │  ← plate 4 added  →  prints "Go!", removed
├─────────────────┤
│ Countdown(1)    │  ← plate 3 added  →  removed after (0) returns
├─────────────────┤
│ Countdown(2)    │  ← plate 2 added  →  removed after (1) returns
├─────────────────┤
│ Countdown(3)    │  ← plate 1 added  →  removed last
└─────────────────┘
       STACK
```

### Tracing a Return Value: Factorial(4)

```
GOING DOWN (making recursive calls):
Factorial(4)
  └─ needs 4 * Factorial(3)
                └─ needs 3 * Factorial(2)
                              └─ needs 2 * Factorial(1)
                                            └─ needs 1 * Factorial(0)
                                                          └─ returns 1  ← BASE CASE

COMING BACK UP (results bubble back):
Factorial(0) returns 1
Factorial(1) returns 1 * 1 = 1
Factorial(2) returns 2 * 1 = 2
Factorial(3) returns 3 * 2 = 6
Factorial(4) returns 4 * 6 = 24  ← final answer
```

> **Key insight:** The calls go **down** (shrinking the problem). The results come **up** (building the final answer).

### Stack Overflow

If recursion never hits a base case, plates keep being added forever until the program runs out of memory:

```
System.StackOverflowException
```

This is why the base case is mandatory.

---

## 6. How to Read Recursive Code Written by Others

You will often need to understand recursive code you did not write. Use this 4-step process.

**Step 1: Find the base case first**
Look for the `if` statement that returns without making a recursive call. That is the anchor.

**Step 2: Identify the recursive call**
Find where the method calls itself. What argument is different? Is it smaller? Simpler?

**Step 3: Read one level only**
Do NOT try to mentally trace all levels. Just read: *"If I assume the recursive call works, what does this one level of code do?"*

**Step 4: Verify with the smallest input**
Test the base case manually, then test the smallest non-trivial input (usually `n = 1` or a single-element array).

### Practice Reading This

```csharp
static int Mystery(int n)
{
    if (n == 1) return 1;
    return n + Mystery(n - 1);
}
```

- **Step 1:** Base case is `n == 1`, returns `1`.
- **Step 2:** Recursive call is `Mystery(n - 1)` — `n` shrinks by `1` each time.
- **Step 3:** One level: returns `n + (result of n-1)`. It adds `n` to the result.
- **Step 4:** `Mystery(1) = 1`. `Mystery(2) = 2 + 1 = 3`. `Mystery(3) = 3 + 3 = 6`.

This is the sum of all numbers from `1` to `n`.

---

## 7. Classic Recursion Problems

### 7.1 Factorial

```csharp
static int Factorial(int n)
{
    if (n == 0) return 1;
    return n * Factorial(n - 1);
}

Console.WriteLine(Factorial(5));  // 120
```

---

### 7.2 Fibonacci Sequence

> Each number is the sum of the two before it: 0, 1, 1, 2, 3, 5, 8, 13 ...

```csharp
static int Fibonacci(int n)
{
    if (n == 0) return 0;  // base case 1
    if (n == 1) return 1;  // base case 2
    return Fibonacci(n - 1) + Fibonacci(n - 2);  // two recursive calls
}

for (int i = 0; i < 10; i++)
{
    Console.Write(Fibonacci(i) + " ");
}
// Output: 0 1 1 2 3 5 8 13 21 34
```

> ⚠️ This version is easy to understand but **very slow** for large `n` because it recomputes the same values many times. See Section 9 (Memoization) for the fix.

---

### 7.3 Sum of an Array

```csharp
static int Sum(int[] arr, int index)
{
    if (index == arr.Length) return 0;
    return arr[index] + Sum(arr, index + 1);
}

int[] numbers = { 1, 2, 3, 4, 5 };
Console.WriteLine(Sum(numbers, 0));  // 15
```

**Full trace:**
```
Sum([1,2,3,4,5], 0)
= 1 + Sum([1,2,3,4,5], 1)
      = 2 + Sum([1,2,3,4,5], 2)
            = 3 + Sum([1,2,3,4,5], 3)
                  = 4 + Sum([1,2,3,4,5], 4)
                        = 5 + Sum([1,2,3,4,5], 5)
                              = 0  ← base case
Back up: 5+0=5, 4+5=9, 3+9=12, 2+12=14, 1+14=15
```

---

### 7.4 Reverse a String

```csharp
static string Reverse(string s)
{
    if (s.Length <= 1) return s;
    return s[s.Length - 1] + Reverse(s.Substring(0, s.Length - 1));
}

Console.WriteLine(Reverse("hello"));  // "olleh"
```

**Trace for "hi":**
```
Reverse("hi")
= 'i' + Reverse("h")
         = "h"  ← base case
= "ih"
```

---

### 7.5 Power (x raised to n)

```csharp
static double Power(double x, int n)
{
    if (n == 0) return 1;
    return x * Power(x, n - 1);
}

Console.WriteLine(Power(2, 10));  // 1024
```

---

### 7.6 Check if a String is a Palindrome

> A palindrome reads the same forwards and backwards: "racecar", "madam"

```csharp
static bool IsPalindrome(string s, int left, int right)
{
    // BASE CASE: middle reached, all characters matched
    if (left >= right) return true;

    // If first and last characters differ, it is not a palindrome
    if (s[left] != s[right]) return false;

    // RECURSIVE CASE: check the inner substring
    return IsPalindrome(s, left + 1, right - 1);
}

Console.WriteLine(IsPalindrome("racecar", 0, 6));  // True
Console.WriteLine(IsPalindrome("hello", 0, 4));    // False
```

---

### 7.7 Binary Search (Recursive)

Binary search finds a target in a **sorted** array by repeatedly halving the search space.

```
Sorted array: [1, 3, 5, 7, 9, 11, 13]
Looking for 7:
  Step 1: middle = 7 → FOUND at index 3
```

```csharp
static int BinarySearch(int[] arr, int target, int left, int right)
{
    // BASE CASE: search space is empty
    if (left > right) return -1;

    int mid = (left + right) / 2;

    if (arr[mid] == target) return mid;

    // If target is smaller, look in the left half
    if (arr[mid] > target)
        return BinarySearch(arr, target, left, mid - 1);

    // If target is larger, look in the right half
    return BinarySearch(arr, target, mid + 1, right);
}

int[] sorted = { 1, 3, 5, 7, 9, 11, 13 };
Console.WriteLine(BinarySearch(sorted, 7, 0, sorted.Length - 1));  // 3
Console.WriteLine(BinarySearch(sorted, 6, 0, sorted.Length - 1));  // -1 (not found)
```

---

### 7.8 Count Digits in a Number

```csharp
static int CountDigits(int n)
{
    if (n < 10) return 1;          // single digit
    return 1 + CountDigits(n / 10);
}

Console.WriteLine(CountDigits(12345));  // 5
```

---

### 7.9 Flatten a Nested List

This shows recursion solving a real-world data problem — deeply nested structures.

```csharp
static void Flatten(List<object> list, List<int> result)
{
    for (int i = 0; i < list.Count; i++)
    {
        if (list[i] is List<object> nested)
        {
            // RECURSIVE CASE: go deeper into the nested list
            Flatten(nested, result);
        }
        else
        {
            // BASE CASE: it is a plain integer, collect it
            result.Add((int)list[i]);
        }
    }
}

// Example: [1, [2, [3, 4], 5], 6]
List<object> nested = new List<object>
{
    1,
    new List<object> { 2, new List<object> { 3, 4 }, 5 },
    6
};

List<int> flat = new List<int>();
Flatten(nested, flat);
Console.WriteLine(string.Join(", ", flat));  // 1, 2, 3, 4, 5, 6
```

---

### 7.10 Tower of Hanoi

> Move `n` disks from peg A to peg C using peg B as a helper.
> Rules: Only one disk moves at a time. A larger disk can never sit on top of a smaller one.

```
Initial:        After solving (n=3):
A   B   C       A   B   C
|   |   |       |   |   |
1   |   |       |   |   1
2   |   |       |   |   2
3   |   |       |   |   3
```

**Thinking it through:**
- To move `n` disks from A to C:
  1. Move the top `n-1` disks from A to B (using C as helper)
  2. Move the largest disk from A to C
  3. Move the `n-1` disks from B to C (using A as helper)

```csharp
static void HanoiTower(int n, string from, string to, string helper)
{
    // BASE CASE: one disk, just move it directly
    if (n == 1)
    {
        Console.WriteLine($"Move disk 1 from {from} to {to}");
        return;
    }

    // Step 1: move top n-1 disks out of the way
    HanoiTower(n - 1, from, helper, to);

    // Step 2: move the largest disk to its destination
    Console.WriteLine($"Move disk {n} from {from} to {to}");

    // Step 3: move the n-1 disks from helper to destination
    HanoiTower(n - 1, helper, to, from);
}

HanoiTower(3, "A", "C", "B");
```

**Output:**
```
Move disk 1 from A to C
Move disk 2 from A to B
Move disk 1 from C to B
Move disk 3 from A to C
Move disk 1 from B to A
Move disk 2 from B to C
Move disk 1 from A to C
```

---

## 8. Types of Recursion

### 8.1 Direct Recursion

A method calls itself directly. All examples so far use this.

```csharp
static int Factorial(int n)
{
    if (n == 0) return 1;
    return n * Factorial(n - 1);  // calls itself
}
```

### 8.2 Indirect Recursion

Two or more methods call each other in a cycle.

```csharp
static void IsEven(int n)
{
    if (n == 0) { Console.WriteLine("Even"); return; }
    IsOdd(n - 1);   // A calls B
}

static void IsOdd(int n)
{
    if (n == 0) { Console.WriteLine("Odd"); return; }
    IsEven(n - 1);  // B calls A
}

IsEven(4);  // Even
IsOdd(3);   // Odd
```

### 8.3 Tail Recursion

The recursive call is the **very last** operation in the method. Nothing happens after it returns.

```csharp
// NOT tail recursive — multiplication happens AFTER the call returns
static int Factorial(int n)
{
    if (n == 0) return 1;
    return n * Factorial(n - 1);  // multiply happens after return
}

// Tail recursive — result is accumulated, nothing left to do after return
static int FactorialTail(int n, int accumulator = 1)
{
    if (n == 0) return accumulator;
    return FactorialTail(n - 1, n * accumulator);  // call IS the last action
}
```

Tail recursion can be optimised by some compilers into a loop internally, saving stack space.

### 8.4 Tree Recursion

A method makes **more than one** recursive call, creating a tree-shaped call graph.

```
Fibonacci(4)
├── Fibonacci(3)
│   ├── Fibonacci(2)
│   │   ├── Fibonacci(1) = 1
│   │   └── Fibonacci(0) = 0
│   └── Fibonacci(1) = 1
└── Fibonacci(2)
    ├── Fibonacci(1) = 1
    └── Fibonacci(0) = 0
```

This is why naive Fibonacci is slow — it computes `Fibonacci(2)` twice and `Fibonacci(1)` three times.

---

## 9. Memoization — Making Recursion Fast

**Memoization** is caching the results of recursive calls so they are never computed more than once.

### The Problem

`Fibonacci(6)` calls `Fibonacci(5)` and `Fibonacci(4)`. But `Fibonacci(5)` also calls `Fibonacci(4)`. So `Fibonacci(4)` is computed twice. This duplication grows exponentially — `Fibonacci(40)` makes over a billion calls.

### The Fix: Cache Results in a Dictionary

```csharp
static Dictionary<int, int> cache = new Dictionary<int, int>();

static int FibMemo(int n)
{
    if (n == 0) return 0;
    if (n == 1) return 1;

    // If we already computed this, return the cached result
    if (cache.ContainsKey(n)) return cache[n];

    // Compute it, store it, return it
    int result = FibMemo(n - 1) + FibMemo(n - 2);
    cache[n] = result;
    return result;
}

Console.WriteLine(FibMemo(50));  // 12586269025 — computed instantly
```

### Before and After

```
WITHOUT memoization — Fibonacci(6) call count:
Fib(6) calls: Fib(5), Fib(4)
Fib(5) calls: Fib(4), Fib(3)
Fib(4) calls: Fib(3), Fib(2)   ← computed multiple times
...total: 25 calls for n=6, billions for n=40

WITH memoization — Fibonacci(6) call count:
Each Fib(n) computed exactly ONCE → 6 calls total for n=6
```

### General Memoization Pattern

```csharp
static Dictionary<int, ReturnType> memo = new Dictionary<int, ReturnType>();

static ReturnType Solve(int n)
{
    if (/* base case */) return baseValue;
    if (memo.ContainsKey(n)) return memo[n];

    ReturnType result = /* recursive computation */;
    memo[n] = result;
    return result;
}
```

---

## 10. Common Mistakes and How to Avoid Them

### Mistake 1: Missing Base Case → Stack Overflow

```csharp
// WRONG — no base case!
static int BadFactorial(int n)
{
    return n * BadFactorial(n - 1);  // never stops → StackOverflowException
}

// CORRECT
static int GoodFactorial(int n)
{
    if (n == 0) return 1;           // ← base case added
    return n * GoodFactorial(n - 1);
}
```

---

### Mistake 2: Base Case Never Reached (Wrong Direction)

```csharp
// WRONG — n grows instead of shrinking!
static void BadCount(int n)
{
    if (n == 0) return;
    BadCount(n + 1);    // ← grows away from base case → Stack Overflow
}

// CORRECT
static void GoodCount(int n)
{
    if (n == 0) return;
    GoodCount(n - 1);   // ← shrinks toward base case ✓
}
```

---

### Mistake 3: Forgetting to Return the Recursive Result

```csharp
// WRONG — return value is lost
static int BadSum(int[] arr, int index)
{
    if (index == arr.Length) return 0;
    arr[index] + BadSum(arr, index + 1);  // computed but never returned!
}

// CORRECT
static int GoodSum(int[] arr, int index)
{
    if (index == arr.Length) return 0;
    return arr[index] + GoodSum(arr, index + 1);  // ← return added ✓
}
```

---

### Mistake 4: Off-by-One in Base Case

```csharp
// WRONG — skips printing 1 because base case fires too early
static void Countdown(int n)
{
    if (n == 1) return;   // should be n == 0
    Console.WriteLine(n);
    Countdown(n - 1);
}

// CORRECT
static void Countdown(int n)
{
    if (n == 0) return;   // ✓
    Console.WriteLine(n);
    Countdown(n - 1);
}
```

---

### Mistake 5: Modifying Shared State Without Restoring It

This is the #1 backtracking mistake. If you modify a shared list or array during recursion, you must restore it afterward. Not doing so corrupts later recursive calls.

```csharp
// WRONG
current.Add(nums[i]);
Backtrack(...);
// forgot to remove — later calls see wrong data!

// CORRECT
current.Add(nums[i]);       // add before
Backtrack(...);
current.RemoveAt(...);      // remove after ✓
```

---

## 11. What is Backtracking?

**Backtracking** is a technique built on top of recursion for problems where you must explore multiple possible choices and eliminate the ones that do not work.

> **Try a choice → explore further → if it fails, undo the choice and try the next one.**

### The Maze Analogy

Imagine navigating a maze:
1. You walk forward until you hit a dead end.
2. You **backtrack** — physically walk back to the last junction.
3. You try a different path.
4. Repeat until you find the exit or exhaust all paths.

Backtracking does exactly this in code.

### Recursion vs Backtracking

| | Recursion | Backtracking |
|---|---|---|
| Purpose | Break a problem into smaller pieces | Explore all possibilities, discard bad ones |
| State change | Usually no shared state to undo | Always undoes choices after exploring |
| Results | Usually one answer | Often finds all valid answers |
| Example | `Factorial`, `Fibonacci` | `Permutations`, `Sudoku`, `N-Queens` |

Backtracking IS recursion — it just adds the "undo" step.

### The Decision Tree

```
                    Start
                   /     \
             Include 1   Exclude 1
             /    \           \
       Incl 2  Excl 2       Incl 2
         |       |              |
       [1,2]   [1]            [2]    ← these are valid outcomes
```

When we go down a path and it is invalid, we **backtrack** to the parent node and try the other branch.

---

## 12. The Backtracking Template

All backtracking problems use this same skeleton. Memorise this pattern.

```csharp
static void Backtrack(/* current state, available choices, results list */)
{
    // ───────────────────────────────────────────────
    // STEP 1 — BASE CASE
    // Is this state a complete and valid solution?
    // ───────────────────────────────────────────────
    if (IsSolution(currentState))
    {
        results.Add(new copy of currentState);
        return;
    }

    // ───────────────────────────────────────────────
    // STEP 2 — LOOP through all possible next choices
    // ───────────────────────────────────────────────
    for (int i = 0; i < choices.Count; i++)
    {
        // Skip invalid choices (this is called PRUNING — covered next)
        if (!IsValid(choices[i])) continue;

        // ── CHOOSE ──────────────────────────────
        MakeChoice(choices[i]);

        // ── EXPLORE ─────────────────────────────
        Backtrack(updated state);

        // ── UN-CHOOSE (backtrack) ────────────────
        UndoChoice(choices[i]);
    }
}
```

The critical three-word mantra: **Choose → Explore → Un-choose.**

---

## 13. Pruning — Making Backtracking Smarter

**Pruning** means cutting off branches of the decision tree that are guaranteed to fail — before going deeper into them.

Without pruning: explore every possible path, including ones that cannot possibly work.
With pruning: skip invalid paths immediately, saving a huge amount of time.

### Example: Combination Sum with Pruning

Find all combinations that add up to a target.

```csharp
static void CombinationSum(int[] candidates, int target, int start,
                            List<int> current, List<List<int>> result)
{
    if (target == 0)
    {
        result.Add(new List<int>(current));
        return;
    }

    for (int i = start; i < candidates.Length; i++)
    {
        // PRUNING: if this candidate is already too large, skip it
        // (assumes candidates are sorted)
        if (candidates[i] > target) break;

        current.Add(candidates[i]);
        CombinationSum(candidates, target - candidates[i], i, current, result);
        current.RemoveAt(current.Count - 1);
    }
}

int[] candidates = { 2, 3, 6, 7 };
Array.Sort(candidates);
List<List<int>> result = new List<List<int>>();
CombinationSum(candidates, 7, 0, new List<int>(), result);

for (int i = 0; i < result.Count; i++)
{
    Console.WriteLine("[" + string.Join(", ", result[i]) + "]");
}
// Output:
// [2, 2, 3]
// [7]
```

The `if (candidates[i] > target) break;` line is the pruning — it skips all larger candidates the moment one is found too big.

---

## 14. Classic Backtracking Problems

### 14.1 Include/Exclude Pattern (Foundation)

Before jumping to complex problems, understand this fundamental pattern. At each step, you have exactly two choices: **include** the current element or **exclude** it.

```csharp
// Print all possible decisions about elements in an array
static void IncludeExclude(int[] nums, int index, List<int> current)
{
    // BASE CASE: all elements decided
    if (index == nums.Length)
    {
        Console.WriteLine("[" + string.Join(", ", current) + "]");
        return;
    }

    // CHOICE 1: INCLUDE nums[index]
    current.Add(nums[index]);
    IncludeExclude(nums, index + 1, current);
    current.RemoveAt(current.Count - 1);  // undo

    // CHOICE 2: EXCLUDE nums[index]
    IncludeExclude(nums, index + 1, current);
}

IncludeExclude(new int[] { 1, 2, 3 }, 0, new List<int>());
```

**Output — every possible subset:**
```
[1, 2, 3]
[1, 2]
[1, 3]
[1]
[2, 3]
[2]
[3]
[]
```

This "include or exclude" pattern is the foundation of subsets, combination sums, and many more problems.

---

### 14.2 Subsets (Power Set)

> Given `[1, 2, 3]`, find all possible subsets: `[], [1], [2], [3], [1,2], [1,3], [2,3], [1,2,3]`

```csharp
static void FindSubsets(int[] nums, int start, List<int> current, List<List<int>> result)
{
    // Every state is a valid subset — save it
    result.Add(new List<int>(current));

    for (int i = start; i < nums.Length; i++)
    {
        current.Add(nums[i]);                          // CHOOSE
        FindSubsets(nums, i + 1, current, result);     // EXPLORE
        current.RemoveAt(current.Count - 1);           // UN-CHOOSE
    }
}

int[] nums = { 1, 2, 3 };
List<List<int>> result = new List<List<int>>();
FindSubsets(nums, 0, new List<int>(), result);

for (int i = 0; i < result.Count; i++)
{
    Console.WriteLine("[" + string.Join(", ", result[i]) + "]");
}
```

**Output:**
```
[]
[1]
[1, 2]
[1, 2, 3]
[1, 3]
[2]
[2, 3]
[3]
```

---

### 14.3 Permutations

> Given `[1, 2, 3]`, find all orderings: `[1,2,3], [1,3,2], [2,1,3], ...`

The idea: for each position, try every number that has not been placed yet.

```csharp
static void Permute(int[] nums, bool[] used, List<int> current, List<List<int>> result)
{
    // BASE CASE: all numbers placed
    if (current.Count == nums.Length)
    {
        result.Add(new List<int>(current));
        return;
    }

    for (int i = 0; i < nums.Length; i++)
    {
        if (used[i]) continue;      // skip already-placed numbers

        used[i] = true;             // CHOOSE: mark as used
        current.Add(nums[i]);

        Permute(nums, used, current, result);   // EXPLORE

        used[i] = false;            // UN-CHOOSE: unmark
        current.RemoveAt(current.Count - 1);
    }
}

int[] nums = { 1, 2, 3 };
bool[] used = new bool[nums.Length];
List<List<int>> result = new List<List<int>>();
Permute(nums, used, new List<int>(), result);

for (int i = 0; i < result.Count; i++)
{
    Console.WriteLine("[" + string.Join(", ", result[i]) + "]");
}
```

**Output:**
```
[1, 2, 3]
[1, 3, 2]
[2, 1, 3]
[2, 3, 1]
[3, 1, 2]
[3, 2, 1]
```

---

### 14.4 Combinations

> Given `n=4, k=2`, find all ways to pick 2 numbers from 1 to 4.

```csharp
static void Combine(int n, int k, int start, List<int> current, List<List<int>> result)
{
    // BASE CASE: picked k numbers
    if (current.Count == k)
    {
        result.Add(new List<int>(current));
        return;
    }

    for (int i = start; i <= n; i++)
    {
        current.Add(i);
        Combine(n, k, i + 1, current, result);
        current.RemoveAt(current.Count - 1);
    }
}

List<List<int>> result = new List<List<int>>();
Combine(4, 2, 1, new List<int>(), result);

for (int i = 0; i < result.Count; i++)
{
    Console.WriteLine("[" + string.Join(", ", result[i]) + "]");
}
// Output: [1,2] [1,3] [1,4] [2,3] [2,4] [3,4]
```

---

### 14.5 Generate Valid Parentheses

> For `n=3`, generate all valid combinations of 3 pairs of parentheses.

**The thinking:**
- You can add `(` if you have not yet opened `n` pairs.
- You can add `)` only if there are more open brackets than closed ones.

```csharp
static void GenerateParentheses(int n, int open, int close,
                                  string current, List<string> result)
{
    // BASE CASE: used all n pairs
    if (current.Length == 2 * n)
    {
        result.Add(current);
        return;
    }

    // CHOOSE to add '(' if we still have opens left
    if (open < n)
        GenerateParentheses(n, open + 1, close, current + "(", result);

    // CHOOSE to add ')' if there are unclosed opens
    if (close < open)
        GenerateParentheses(n, open, close + 1, current + ")", result);
}

List<string> result = new List<string>();
GenerateParentheses(3, 0, 0, "", result);

for (int i = 0; i < result.Count; i++)
{
    Console.WriteLine(result[i]);
}
```

**Output:**
```
((()))
(()())
(())()
()(())
()()()
```

---

### 14.6 Word Search in a Grid

> Given a 2D grid of characters and a word, find if the word exists by moving through adjacent cells.

**The thinking:**
- Start from each cell.
- If the character matches the first letter, try moving up/down/left/right for the next letter.
- Mark a cell as visited while exploring it, then unmark when backtracking.

```csharp
static bool WordSearch(char[,] board, string word, int row, int col, int index)
{
    // BASE CASE: all characters matched
    if (index == word.Length) return true;

    int rows = board.GetLength(0);
    int cols = board.GetLength(1);

    // Prune: out of bounds or wrong character
    if (row < 0 || row >= rows || col < 0 || col >= cols) return false;
    if (board[row, col] != word[index]) return false;

    // CHOOSE: mark as visited
    char saved = board[row, col];
    board[row, col] = '#';

    // EXPLORE: all 4 directions
    bool found = WordSearch(board, word, row + 1, col, index + 1)
              || WordSearch(board, word, row - 1, col, index + 1)
              || WordSearch(board, word, row, col + 1, index + 1)
              || WordSearch(board, word, row, col - 1, index + 1);

    // UN-CHOOSE: restore the cell
    board[row, col] = saved;

    return found;
}

char[,] grid = {
    { 'A', 'B', 'C', 'E' },
    { 'S', 'F', 'C', 'S' },
    { 'A', 'D', 'E', 'E' }
};

Console.WriteLine(WordSearch(grid, "ABCCED", 0, 0, 0));  // True
Console.WriteLine(WordSearch(grid, "SEE", 1, 3, 0));      // True
Console.WriteLine(WordSearch(grid, "ABCB", 0, 0, 0));     // False
```

---

### 14.7 N-Queens Problem

> Place N queens on an N×N chessboard so that no two queens attack each other (no shared row, column, or diagonal).

**The thinking:**
- Place queens one row at a time.
- For each row, try each column.
- Before placing, check if the position is safe (no conflict with earlier queens).
- If safe, place and go to the next row. If not, try the next column. If none work, backtrack.

```csharp
static void SolveNQueens(int row, int n,
                          bool[] columns, bool[] diag1, bool[] diag2,
                          List<int> placement, List<List<int>> results)
{
    // BASE CASE: placed a queen in every row — valid solution
    if (row == n)
    {
        results.Add(new List<int>(placement));
        return;
    }

    for (int col = 0; col < n; col++)
    {
        // PRUNING: check if this cell is under attack
        if (columns[col] || diag1[row - col + n - 1] || diag2[row + col])
            continue;

        // CHOOSE
        columns[col] = true;
        diag1[row - col + n - 1] = true;
        diag2[row + col] = true;
        placement.Add(col);

        // EXPLORE
        SolveNQueens(row + 1, n, columns, diag1, diag2, placement, results);

        // UN-CHOOSE
        columns[col] = false;
        diag1[row - col + n - 1] = false;
        diag2[row + col] = false;
        placement.RemoveAt(placement.Count - 1);
    }
}

int n = 4;
bool[] cols = new bool[n];
bool[] d1 = new bool[2 * n - 1];
bool[] d2 = new bool[2 * n - 1];
List<List<int>> results = new List<List<int>>();
SolveNQueens(0, n, cols, d1, d2, new List<int>(), results);

Console.WriteLine($"Found {results.Count} solutions for {n}-Queens:");
for (int i = 0; i < results.Count; i++)
{
    Console.WriteLine("Columns: [" + string.Join(", ", results[i]) + "]");
}
// Found 2 solutions for 4-Queens:
// Columns: [1, 3, 0, 2]
// Columns: [2, 0, 3, 1]
```

---

### 14.8 Sudoku Solver

> Fill a 9×9 grid so every row, column, and 3×3 box contains the digits 1–9 exactly once.

**The thinking:**
- Scan for the first empty cell.
- Try placing each digit 1–9.
- If valid, place it and recurse to solve the rest.
- If no digit works, backtrack (reset the cell to 0 and return false).
- If no empty cells remain, the puzzle is solved.

```csharp
static bool SolveSudoku(int[,] board)
{
    for (int row = 0; row < 9; row++)
    {
        for (int col = 0; col < 9; col++)
        {
            if (board[row, col] == 0)  // found an empty cell
            {
                for (int num = 1; num <= 9; num++)
                {
                    if (IsValidSudoku(board, row, col, num))
                    {
                        board[row, col] = num;          // CHOOSE

                        if (SolveSudoku(board))         // EXPLORE
                            return true;

                        board[row, col] = 0;            // UN-CHOOSE (backtrack)
                    }
                }
                return false;  // no digit worked — tell the caller to backtrack
            }
        }
    }
    return true;  // BASE CASE: no empty cells — puzzle solved!
}

static bool IsValidSudoku(int[,] board, int row, int col, int num)
{
    for (int c = 0; c < 9; c++)
        if (board[row, c] == num) return false;

    for (int r = 0; r < 9; r++)
        if (board[r, col] == num) return false;

    int startRow = (row / 3) * 3;
    int startCol = (col / 3) * 3;
    for (int r = startRow; r < startRow + 3; r++)
    {
        for (int c = startCol; c < startCol + 3; c++)
        {
            if (board[r, c] == num) return false;
        }
    }
    return true;
}
```

---

## 15. Recursion vs Iteration — When to Use Which

| Situation | Prefer Recursion | Prefer Iteration |
|---|---|---|
| Tree / graph traversal | ✅ Natural fit | ❌ Needs manual stack |
| Divide and conquer (merge sort, binary search) | ✅ Elegant | ✅ Also fine |
| Simple loops (sum, count, print) | ❌ Overkill | ✅ Cleaner |
| Backtracking / exhaustive search | ✅ Required | ❌ Very complex |
| Deep recursion (thousands of levels) | ⚠️ Stack overflow risk | ✅ Safer |
| Hierarchical data (JSON, XML, trees) | ✅ Natural fit | ❌ Complex |

---

## 16. Problem Pattern Recognition — LeetCode & Interviews

When you see a problem, map it to a pattern using these signals.

### Use Recursion When:

| Signal in the problem | Pattern to apply |
|---|---|
| "find in a sorted array" | Recursive Binary Search |
| "traverse all nodes / files / folders" | Tree / DFS Recursion |
| "sort the array" | Merge Sort / Quick Sort |
| "tree height, depth, path sum" | Tree Recursion |
| "parse nested structure (JSON, XML)" | Recursive Parsing |

### Use Backtracking When:

| Signal in the problem | Pattern to apply |
|---|---|
| "find all combinations / subsets" | Subset / Include-Exclude |
| "find all permutations / arrangements" | Permutation with `used[]` |
| "find one valid arrangement (puzzle, board)" | Constraint Satisfaction (N-Queens, Sudoku) |
| "find path through a grid" | Grid DFS with backtrack |
| "generate all valid strings of length N" | String building backtrack |
| "partition into subsets / groups" | Partition backtrack |

### Quick LeetCode Problem Mapping

| LeetCode Problem | Pattern |
|---|---|
| #78 Subsets | Backtracking — Include/Exclude |
| #46 Permutations | Backtracking — Permutation |
| #77 Combinations | Backtracking — Combination |
| #39 Combination Sum | Backtracking with Pruning |
| #22 Generate Parentheses | Backtracking with constraints |
| #79 Word Search | Backtracking on Grid |
| #51 N-Queens | Backtracking — Constraint |
| #509 Fibonacci Number | Recursion + Memoization |
| #700 Search in BST | Tree Recursion |
| #104 Maximum Depth of Binary Tree | Tree Recursion |

---

## 17. Tips for Solving Recursive Problems

### Tip 1: Always Write the Base Case First

Before anything else, ask: *"What is the simplest input I can answer directly?"*
Write that first. Then write the recursive case.

### Tip 2: Make Sure Each Call Moves Toward the Base Case

If you pass `n - 1`, the base case should handle `n == 0`. If you pass `index + 1`, the base case should handle `index == arr.Length`. The input must always shrink.

### Tip 3: Trust the Recursion

Do NOT trace all the levels in your head while writing code. Say: *"Assume `MyMethod(n-1)` is already correct. What do I do with it?"*

### Tip 4: Verify with n = 0 and n = 1 Only

If the base case is correct and one recursive step is correct, the whole thing works. Only trace the smallest cases.

### Tip 5: For Backtracking — Always Undo

The `current.RemoveAt(...)` or flag reset is not optional. Without it, your state becomes corrupted for sibling branches. Write the "undo" at the same time as the "do".

### Tip 6: Draw the Decision Tree for Backtracking

Before coding, sketch the tree on paper. Label each branch with the choice made. This makes the code structure obvious.

### Tip 7: Use a `results` List, Not Print Statements

Collect solutions in a list rather than printing. This makes solutions reusable and testable.

### Tip 8: When Stuck — Write a Simpler Version First

If you cannot solve for `n`, solve for `n = 2` or `n = 3` manually on paper first. The pattern almost always reveals itself.

---

## 18. Practice Problems

Try solving these yourself. The struggle is what builds the skill.

### Recursion Problems

| # | Problem | What to think about |
|---|---|---|
| 1 | Print numbers 1 to N using recursion | Print *after* the recursive call |
| 2 | Find the maximum element in an array | `max(arr[0], max of rest)` |
| 3 | Calculate GCD (Euclidean algorithm) | `GCD(a, b) = GCD(b, a % b)`, base: `b == 0` |
| 4 | Check if array is sorted recursively | Compare `arr[i]` and `arr[i+1]` |
| 5 | Count occurrences of a digit in a number | Use `n % 10` and `n / 10` |
| 6 | Merge Sort | Split, sort each half, merge |
| 7 | Fibonacci with memoization for n=100 | Use the `Dictionary<int, int>` cache pattern |

### Backtracking Problems

| # | Problem | Key insight |
|---|---|---|
| 1 | Generate all binary strings of length N | At each position, choose `0` or `1` |
| 2 | Letter combinations of a phone number | Map each digit to letters, build string char by char |
| 3 | Palindrome partitioning | At each index, try all substrings that are palindromes |
| 4 | Combination Sum II (each number used once) | Sort + skip duplicates |
| 5 | Rat in a Maze | Move right or down, mark visited, backtrack on walls |
| 6 | All paths from top-left to bottom-right in grid | Use direction arrays, undo each step |

---

## 19. Quick Reference Cheat Sheet

```
RECURSION SKELETON
──────────────────────────────────────────────────────
static ReturnType Method(input)
{
    if (BASE CASE) return directAnswer;    // simplest version
    return Method(SMALLER INPUT);          // trust the recursion
}

THREE QUESTIONS TO ASK BEFORE CODING:
1. What is the base case?
2. How does one recursive step reduce the problem?
3. What do I do with the recursive result?

──────────────────────────────────────────────────────
BACKTRACKING SKELETON
──────────────────────────────────────────────────────
static void Backtrack(state, choices, results)
{
    if (IS COMPLETE SOLUTION)
    {
        results.Add(copy of state);
        return;
    }

    for each choice in choices
    {
        if (!IsValid(choice)) continue;    // PRUNE invalid branches

        MakeChoice(choice);                // CHOOSE
        Backtrack(updated state);          // EXPLORE
        UndoChoice(choice);                // UN-CHOOSE ← never skip this
    }
}

MANTRA: Choose → Explore → Un-choose

──────────────────────────────────────────────────────
MEMOIZATION SKELETON
──────────────────────────────────────────────────────
Dictionary<int, ReturnType> memo = new();

static ReturnType Method(int n)
{
    if (BASE CASE) return directAnswer;
    if (memo.ContainsKey(n)) return memo[n];

    ReturnType result = Method(n-1) + Method(n-2); // etc.
    memo[n] = result;
    return result;
}

──────────────────────────────────────────────────────
DEBUGGING CHECKLIST
──────────────────────────────────────────────────────
□ Does the base case exist?
□ Does each recursive call move closer to the base case?
□ Is the recursive result being returned?
□ Does memoization exist for overlapping subproblems?
□ In backtracking — is every choice being undone after exploring?
□ Traced manually for n=0 and n=1?
□ Drew the decision tree before coding?
```

---

*Notes by Sharan | Repository: `sharansnavada/csharp-notes`*
