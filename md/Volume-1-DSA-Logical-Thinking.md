# THE COMPLETE .NET DEVELOPER ENCYCLOPEDIA + INTERVIEW MASTERY GUIDE
## Volume 1: Logical Thinking & Data Structures and Algorithms

> **Target Audience:** .NET backend developer (~3 years experience) aiming for senior-level algorithmic fluency and acing technical interview rounds at tier-1 MNCs like Infosys, TCS, Wipro, Accenture, as well as product engineering firms.  
> **Core Objective:** Bridge the gap between writing enterprise business CRUD code and developing instinctive, low-level algorithmic logic. Transform from "memorizing LeetCode" to "architectural decomposition and problem solving."  
> **Production Domains Used:** Order Management Systems, Core Banking & Ledger Engines, Healthcare EHR Records, High-Throughput SaaS, Distributed Notification Hubs, Payment Gateways.  
> **Language & Runtime:** C# 12 / .NET 8 LTS (featuring `Span<T>`, `ReadOnlySpan<T>`, `PriorityQueue<TElement, TPriority>`, Memory-efficient Collections, and CLR Heap layout insights).

---

# TABLE OF CONTENTS

1. [The Engineer's Mindset: From "Struggling with DSA" to "Architectural Problem Solver"](#1-the-engineers-mindset-from-struggling-with-dsa-to-architectural-problem-solver)
   - 1.1 Why .NET Enterprise Developers Struggle with DSA
   - 1.2 The "LINQ Crutch" vs. Mechanical Sympathy
   - 1.3 The Infosys & MNC Interview Philosophy for 3 YOE
   - 1.4 The 7-Step Universal Problem-Solving Blueprint
2. [Mastering the Core Algorithmic Thinking Patterns](#2-mastering-the-core-algorithmic-thinking-patterns)
   - 2.1 The 10 Essential Algorithmic Archetypes
   - 2.2 Pattern Recognition Decision Matrix (The Clue-to-Pattern Rosetta Stone)
3. [Arrays: Memory Layout & Fundamental Algorithms](#3-arrays-memory-layout--fundamental-algorithms)
   - 3.1 CLR Internals: Array Layout on the Managed Heap & CPU Cache Locality
   - 3.2 Modern .NET 8 Mechanics: `Span<T>`, `ReadOnlySpan<T>`, and Zero-Allocation Slicing
   - 3.3 Problem 1: Two Sum (Unsorted & Sorted Variants)
   - 3.4 Problem 2: Kadane's Maximum Subarray Sum (Transaction Delta Optimization)
   - 3.5 Problem 3: Rotate Array (Cyclic Shifts & In-Place Reversals)
   - 3.6 Problem 4: Find Duplicate Number (Floyd's Cycle Detection on Array Pointers)
   - 3.7 Problem 5: Merge Intervals (Booking & Calendar Overlap Consolidation)
   - 3.8 Problem 6: Product of Array Except Self (Prefix-Suffix Accumulation Without Division)
4. [Strings: Immutability, Memory Optimization & Pattern Matching](#4-strings-immutability-memory-optimization--pattern-matching)
   - 4.1 CLR Internals: String Layout, Interning Pool, UTF-16, and GC Pressures
   - 4.2 Problem 7: Reverse String & Words in Sentence (Zero-Allocation Token Reversal)
   - 4.3 Problem 8: Valid Anagram (Frequency Buckets vs. Sorting)
   - 4.4 Problem 9: Longest Substring Without Repeating Characters (Sliding Window)
   - 4.5 Problem 10: String Compression (Run-Length Encoding with Write Pointers)
   - 4.6 Problem 11: Palindrome Check & Longest Palindromic Substring
5. [HashMap & Dictionary Internals: Deep CLR Dive & Applications](#5-hashmap--dictionary-internals-deep-clr-dive--applications)
   - 5.1 CLR Deep Dive: How `Dictionary<TKey, TValue>` Works Internally in .NET 8
   - 5.2 Problem 12: Group Anagrams (Categorization of Financial Audit Records)
   - 5.3 Problem 13: Top K Frequent Elements (Bucket Frequency vs. Min-Heap)
   - 5.4 Problem 14: First Unique Character in a Stream
6. [Binary Search: The Monotonic Mindset](#6-binary-search-the-monotonic-mindset)
   - 6.1 The Universal Binary Search Paradigm
   - 6.2 Problem 15: Classic Binary Search (Exact Order ID Match)
   - 6.3 Problem 16: Search in Rotated Sorted Array (Shifted Ledger Partitions)
   - 6.4 Problem 17: Find First and Last Position in Sorted Array (Lower & Upper Bounds)
7. [Sorting Algorithms: From First Principles to .NET IntroSort](#7-sorting-algorithms-from-first-principles-to-net-introsort)
   - 7.1 QuickSort & MergeSort Internals
   - 7.2 Under the Hood: .NET `Array.Sort` and IntroSort (QuickSort + HeapSort + InsertionSort)
8. [Stack & Queue: LIFO, FIFO, and Expression Parsing](#8-stack--queue-lifo-fifo-and-expression-parsing)
   - 8.1 Memory Models: Array-Backed vs. Node-Backed Queues
   - 8.2 Problem 18: Valid Parentheses (Syntax Validation for Config Tokens)
   - 8.3 Problem 19: Min Stack (O(1) Retrieval via Companion State)
   - 8.4 Problem 20: Evaluate Reverse Polish Notation (Financial Math Engine)
   - 8.5 Problem 21: Implement Queue Using Stacks (Amortized Cost Transfer)
   - 8.6 Problem 22: Moving Average from Data Stream (Circular Queue Buffer)
9. [Linked Lists: Pointer Manipulation & Cycle Detection](#9-linked-lists-pointer-manipulation--cycle-detection)
   - 9.1 Linked List Memory Architecture & Cache Miss Penalty
   - 9.2 Problem 23: Reverse Linked List (Iterative Pointer Rewiring)
   - 9.3 Problem 24: Detect Cycle in Linked List (Floyd's Tortoise and Hare)
   - 9.4 Problem 25: Find Middle Node of Linked List (Fast-Slow Pointers)
   - 9.5 Problem 26: Merge Two Sorted Lists (Splice Nodes with Sentinel Head)
10. [Trees: Hierarchical Structures, Traversal & Binary Search Trees](#10-trees-hierarchical-structures-traversal--binary-search-trees)
    - 10.1 Tree Representations & Stack Frame Traversal Mechanics
    - 10.2 Problem 27: Maximum Depth of Binary Tree (DFS vs. BFS)
    - 10.3 Problem 28: Binary Tree Level Order Traversal (Batch Queue Sizing)
    - 10.4 Problem 29: Validate Binary Search Tree (Range Constraint Propagation)
    - 10.5 Problem 30: Lowest Common Ancestor (Organizational Hierarchy Resolution)
11. [Graphs: Modeling Complex Enterprise Networks](#11-graphs-modeling-complex-enterprise-networks)
    - 11.1 Adjacency List vs. Adjacency Matrix Trade-offs
    - 11.2 Problem 31: Number of Islands (Grid DFS/BFS Connected Components)
    - 11.3 Problem 32: Course Schedule (Cycle Detection & Topological Sort via Kahn's Algorithm)
12. [Heap & PriorityQueue (.NET 6+)](#12-heap--priorityqueue-net-6)
    - 12.1 Binary Heap Mechanics on Contiguous Arrays
    - 12.2 Problem 33: Kth Largest Element in an Array (Min-Heap of Size K)
    - 12.3 Problem 34: Merge K Sorted Lists / Telemetry Streams (K-Way Min-Heap Merge)
13. [Recursion & Backtracking: State Space Exploration](#13-recursion--backtracking-state-space-exploration)
    - 13.1 Backtracking Template: Choose -> Explore -> Unchoose
    - 13.2 Problem 35: Subsets (Combinatorial Role-Based Access Matrix)
    - 13.3 Problem 36: Permutations (Distinct Sequencing Generation)
    - 13.4 Problem 37: Word Search (2D Board Backtracking with Visited Tracking)
14. [Dynamic Programming: Memoization & Tabulation](#14-dynamic-programming-memoization--tabulation)
    - 14.1 The DP Framework: Optimal Substructure + Overlapping Subproblems
    - 14.2 Problem 38: Climbing Stairs (Fibonacci State Transitions)
    - 14.3 Problem 39: Coin Change (Unbounded Knapsack / Minimum Denominations)
    - 14.4 Problem 40: Longest Common Subsequence (Audit Diff & Reconciliation Engine)
    - 14.5 Problem 41: 0/1 Knapsack Problem (SaaS Resource Allocation Under Budget)
15. [Greedy Algorithms & Intervals](#15-greedy-algorithms--intervals)
    - 15.1 The Greedy-Choice Property
    - 15.2 Problem 42: Best Time to Buy and Sell Stock (Running Valley & Peak)
    - 15.3 Problem 43: Jump Game (Boundary Horizon Expansion)
    - 15.4 Problem 44: Meeting Rooms I & II (Resource Contention Peaks)
16. [Bit Manipulation: Hardware-Level Logic](#16-bit-manipulation-hardware-level-logic)
    - 16.1 Bitwise Operators and Bitmask Flags in C#
    - 16.2 Problem 45: Single Number (XOR Cancellation Property)
    - 16.3 Problem 46: Number of 1 Bits (Brian Kernighan's Bit Clear)
    - 16.4 Problem 47: Power of Two (Bitmask Decrement Boundary)
17. [Infosys & Technical Interview Survival Playbook](#17-infosys--technical-interview-survival-playbook)
    - 17.1 How Infosys Evaluates a 3 YOE .NET Developer
    - 17.2 Real 45-Minute Interview Simulation Transcript
    - 17.3 The .NET 8 Big-O Time & Space Complexity Reference Matrix

---

# 1. THE ENGINEER'S MINDSET: FROM "STRUGGLING WITH DSA" TO "ARCHITECTURAL PROBLEM SOLVER"

## 1.1 Why .NET Enterprise Developers Struggle with DSA

A typical backend .NET developer with 3 years of experience spends 90% of their working day doing the following:
1. Creating ASP.NET Core Web API controllers or Minimal API endpoints.
2. Writing Entity Framework Core LINQ queries (`_context.Orders.Where(x => x.Status == OrderStatus.Pending).Include(x => x.Items).ToListAsync()`).
3. Mapping DTOs using AutoMapper or Mapster.
4. Injecting services through dependency injection containers (`IServiceCollection`).
5. Calling external REST APIs or reading messages off RabbitMQ / Azure Service Bus.

When dropped into a technical interview room and asked:
> *"Given an array of integers representing stock prices, find the maximum profit you can achieve with at most two transactions,"*

the developer freezes. Panic sets in. Why?

### The Three Disconnects
1. **The Declarative Abstraction Trap:** In daily enterprise work, you tell the framework *what* you want (LINQ: `.OrderBy().GroupBy().Select()`), not *how* to achieve it. DSA requires you to dictate the exact byte-level, index-by-index execution mechanism.
2. **The "Everything is an Object" Mental Model:** In C#, we get comfortable treating everything as high-level heap allocations. We pass around `List<T>`, `IEnumerable<T>`, and complex nested domain models. When an interviewer asks to manipulate an array in-place without auxiliary memory, the developer struggles because they are conditioned to create new collections with LINQ.
3. **The Absence of Systematic Tracing:** In enterprise code, when something goes wrong, you attach the Visual Studio debugger, inspect the locals window, and step through. In an interview, **you are the debugger**. You must be able to maintain variable state inside your head and walk through an execution table on a whiteboard or shared notepad.

```
Enterprise Mindset (Declarative):
Database / EF Core  ──>  LINQ Query  ──>  DTO Projection  ──>  JSON Response
"Framework, give me the data that matches this shape."

Algorithmic Mindset (Procedural & Structural):
Raw Memory / Array  ──>  Pointers / State  ──>  In-Place Mutations  ──>  O(1) Space Result
"CPU, look at index i, compare with index j, swap bytes, increment pointer."
```

---

## 1.2 The "LINQ Crutch" vs. Mechanical Sympathy

Consider a real interview scenario. The interviewer asks:
> *"Given an array of integers, remove duplicates in-place such that each unique element appears only once, returning the count of unique elements."*

The developer with a LINQ crutch immediately responds:
```csharp
// THE WRONG WAY TO ANSWER AN ALGORITHMIC QUESTION
public int[] RemoveDuplicates(int[] nums)
{
    return nums.Distinct().ToArray();
}
```

### Why the Interviewer Rejects This Answer
1. **Time Complexity:** `.Distinct()` internally allocates a `HashSet<int>`. It iterates over the array and adds items to the hash set. Time complexity: $O(N)$.
2. **Space Complexity:** It allocates a hash set on the Managed Heap ($O(N)$ memory) and then allocates a completely brand-new array via `.ToArray()` ($O(N)$ memory).
3. **Ignores Constraints:** The problem specified **in-place**. Allocating $2N$ heap memory violates the fundamental requirement.
4. **Hides Problem-Solving Capability:** The interviewer learns nothing about whether you understand memory traversal, pointers, or loop invariants.

### The Mechanical Sympathy Approach
Mechanical sympathy means understanding how the underlying CLR runtime and CPU hardware execute your code. An in-place two-pointer approach writes directly to memory addresses:

```csharp
// THE SENIOR ENGINEER'S ANSWER (Mechanical Sympathy)
public int RemoveDuplicates(Span<int> nums)
{
    if (nums.IsEmpty) return 0;

    int writeIndex = 1;
    for (int readIndex = 1; readIndex < nums.Length; readIndex++)
    {
        if (nums[readIndex] != nums[readIndex - 1])
        {
            nums[writeIndex] = nums[readIndex];
            writeIndex++;
        }
    }
    return writeIndex;
}
```
**Zero allocations ($O(1)$ space), single pass ($O(N)$ time), cache-friendly contiguous memory reads, CPU branch predictor optimal.**

---

## 1.3 The Infosys & MNC Interview Philosophy for 3 YOE

Interviews at massive tech consultancies (Infosys, TCS, Cognizant, Wipro) and tier-1 product delivery teams for candidates with ~3 years of experience are standardized. They follow a predictable pattern:

| Dimension | What Junior Developers Think Interviewers Want | What Interviewers Actually Grade You On |
|---|---|---|
| **Speed** | Blurt out working code in 60 seconds. | Structured thought process, clarifying edge cases, calm decomposition. |
| **Complexity** | Using fancy syntax or one-liner LINQ queries. | Writing optimal loops, analyzing Big-O time and space accurately. |
| **Edge Cases** | "I'll handle nulls if it crashes." | Proactively listing empty arrays, duplicates, negative numbers, overflow. |
| **Communication** | Silence while typing, then saying "Done." | Thinking out loud: explaining trade-offs before writing a single bracket. |
| **Code Cleanliness** | Single-letter variables (`a`, `b`, `t`). | Production-grade C#: descriptive naming, guard clauses, proper types. |

### The Infosys Assessment Pattern
1. **Screening Round:** Online test with 2–3 algorithmic problems (typically 1 String manipulation, 1 Array manipulation, 1 Math/DP or Hash problem).
2. **Technical Round 1 (60 Mins):**
   - 20 mins: C# / CLR internals & ASP.NET Core architecture.
   - 25 mins: Live coding on an interactive editor (e.g., HackerRank CodePair or Notepad). Focus: **Strings, Arrays, Dictionaries, Two Pointers**.
   - 15 mins: SQL Queries & EF Core performance.
3. **Technical Round 2 (Managerial / Architect):**
   - Scenario-based problem solving: "How would you deduplicate 10 million transactions using minimal RAM?"

---

## 1.4 The 7-Step Universal Problem-Solving Blueprint

Whenever an interviewer presents a DSA problem, **do not touch the keyboard for the first 3 minutes.** Follow this ironclad protocol:

```
Step 1: CLARIFY CONSTRAINTS & BOUNDARIES
        ├── What are the input sizes (N <= 10^3, 10^5, 10^9)?
        ├── Can elements be negative, null, zero, or duplicate?
        └── What is the expected return type on invalid input?
            ↓
Step 2: WORKED EXAMPLE (MANUAL DRY RUN)
        ├── Trace a small, concrete test case by hand.
        └── Formulate an edge case (e.g., single element, empty, all identical).
            ↓
Step 3: STATE THE BRUTE FORCE (ESTABLISH THE BASELINE)
        ├── Express the naive approach clearly (e.g., "Nested loops gives O(N^2)").
        └── State its time and space complexity without apology.
            ↓
Step 4: IDENTIFY THE BOTTLENECK & SELECT PATTERN
        ├── "We are repeatedly scanning backward: could a HashMap give O(1)?"
        ├── "The array is sorted: can we use Two Pointers or Binary Search?"
        └── "We need sliding boundaries: can a Sliding Window reduce O(N^2) to O(N)?"
            ↓
Step 5: PROPOSE THE OPTIMAL ALGORITHM & VALIDATE COMPLEXITY
        ├── State: "I can achieve O(N) time and O(1) auxiliary space using Two Pointers."
        └── Gain interviewer buy-in: "Would you like me to code this approach?"
            ↓
Step 6: WRITE CLEAN, PRODUCTION-GRADE C#
        ├── Guard clauses first (`ArgumentNullException`, empty check).
        ├── Meaningful variable names (`leftPointer`, `currentSum`, `frequencyMap`).
        └── Modern C# idioms (`Span<T>`, Pattern Matching where beneficial).
            ↓
Step 7: TRACE TEST CASES & STATE TIME/SPACE COMPLEXITY
        ├── Walk through your code line-by-line with the sample input.
        └── Conclude with formal Big-O: "Time is O(N), Space is O(1) auxiliary."
```

---

# 2. MASTERING THE CORE ALGORITHMIC THINKING PATTERNS

## 2.1 The 10 Essential Algorithmic Archetypes

Every algorithmic problem asked in an interview is an instantiation of one of ten fundamental patterns. Once you learn to recognize the pattern signature, you no longer memorize individual problems.

```
                                  ┌──────────────────────────────┐
                                  │ DSA Problem Solving Toolkit  │
                                  └──────────────┬───────────────┘
                                                 │
         ┌───────────────────┬───────────────────┼───────────────────┬───────────────────┐
         │                   │                   │                   │                   │
  [1. Two Pointers]  [2. Sliding Window]  [3. Fast & Slow]   [4. Binary Search]  [5. Hash Caching]
  (Sorted Arrays,     (Contiguous Sub-    (Cycle Detection,   (Monotonic Search   (O(1) Lookups,
   Converging/Read-    arrays, Strings,    Middle of List,     Space, Rotated      Frequency Counts,
   Write Bounds)       Max/Min Windows)    Memory Loops)       Bounds)             Complements)
         │                   │                   │                   │                   │
  [6. Monotonic Stack][7. BFS Traversal]  [8. DFS / Backtrack][9. Dynamic Prog]  [10. Greedy Choice]
  (Next Greater/      (Shortest Path,     (Exhaustive Search, (Overlapping Sub-   (Local Optimal
   Smaller, Histograms,Level Order,        Permutations,       problems, Min/Max   Step = Global
   Parentheses)        Unweighted Graphs)  Grid Traversals)    Cost Paths)         Optimal Solution)
```

### Pattern 1: Two Pointers
- **Core Mechanism:** Maintain two indices moving across one or two linear collections.
- **Variations:**
  - *Converging:* Left starts at 0, Right starts at $N-1$; move toward each other (used in sorted pair sums, palindrome verification).
  - *Parallel / Read-Write:* Slow write pointer, fast read pointer (used in array deduplication, zero compaction).
  - *Two Arrays:* Merge two sorted arrays into one.

### Pattern 2: Sliding Window
- **Core Mechanism:** Maintain a dynamic or fixed subarray range `[left, right]`. Expand `right` to include elements; shrink `left` when window invariants are violated.
- **Trigger Clues:** "Longest contiguous substring...", "Subarray with sum equal to $K$...", "Maximum sum of $K$ consecutive elements...".

### Pattern 3: Fast and Slow Pointers (Tortoise & Hare)
- **Core Mechanism:** Two pointers traverse at different speeds (usually slow advances 1 step, fast advances 2 steps).
- **Trigger Clues:** "Detect cycle in linked list", "Find middle node", "Find duplicate in array where values point to indices".

### Pattern 4: Binary Search
- **Core Mechanism:** Repeatedly divide a sorted or monotonic search space in half.
- **Trigger Clues:** "Find element in sorted array", "Search in logarithmic time $O(\log N)$", "Find the minimum value that satisfies condition $P(x)$".

### Pattern 5: HashMap / HashSet Caching
- **Core Mechanism:** Trade space for time. Store seen elements or cumulative states in an $O(1)$ lookup table.
- **Trigger Clues:** "Find pair with sum $K$", "Check if duplicate exists", "Subarray sum equals $K$ (prefix sums)", "Group items by signature".

### Pattern 6: Stack / Monotonic Stack
- **Core Mechanism:** Last-In, First-Out (LIFO). A monotonic stack maintains elements in strictly increasing or decreasing order.
- **Trigger Clues:** "Valid parentheses / brackets", "Next greater element", "Daily temperatures", "Largest rectangle in histogram".

### Pattern 7: Breadth-First Search (BFS)
- **Core Mechanism:** Level-order exploration using a FIFO `Queue<T>`.
- **Trigger Clues:** "Shortest path in unweighted graph/grid", "Level-by-level tree traversal", "Minimum steps to reach target".

### Pattern 8: Depth-First Search (DFS) & Backtracking
- **Core Mechanism:** Deep branch exploration using recursion or explicit `Stack<T>`. Backtracking undoes the state when a branch terminates.
- **Trigger Clues:** "Find all permutations/subsets", "Word search in 2D grid", "Tree path sums", "Connected components / Islands".

### Pattern 9: Dynamic Programming (DP)
- **Core Mechanism:** Breaking a problem into overlapping subproblems with optimal substructure. Store subproblem answers in an array or dictionary (memoization or tabulation).
- **Trigger Clues:** "Count total number of ways to...", "Find minimum / maximum cost to reach...", "Can you partition the set...".

### Pattern 10: Greedy Strategy
- **Core Mechanism:** Make the locally optimal choice at each step, proving that it leads to the globally optimal solution.
- **Trigger Clues:** "Interval scheduling", "Minimum number of platforms / meeting rooms", "Gas station circuit".

---

## 2.2 Pattern Recognition Decision Matrix

When you read a question prompt in an interview, refer to this mapping:

| If the problem states... | And the data structure is... | Then your primary intuition should be... |
|---|---|---|
| "Array is sorted" and "Find pair/triplet" | Array | **Two Pointers (Converging)** |
| "Contiguous subarray / substring" with "Max/Min/Length" | Array / String | **Sliding Window** |
| "Find cycle", "Find middle", "Loop in pointer references" | Linked List / Array | **Fast & Slow Pointers (Floyd's)** |
| "Search target in $O(\log N)$" or "Monotonic condition" | Sorted Collection | **Binary Search** |
| "Frequency of elements", "Pair sum unsorted", "Group by" | Unsorted Collection | **HashMap / HashSet** |
| "Matching brackets", "Nested tags", "Next Greater Element" | Sequence / Expression | **Stack / Monotonic Stack** |
| "Shortest path (unweighted)", "Minimum transformation steps" | Graph / Matrix / Tree | **BFS with Queue** |
| "Generate all combinations", "Exhaustive paths", "Sudoku" | Combinatorial / Board | **Recursion with Backtracking** |
| "Count total ways", "Longest increasing...", "Choices with weights" | Integer / Sequence | **Dynamic Programming** |
| "Schedule meetings", "Non-overlapping intervals", "Stocks" | Intervals / Values | **Sorting + Greedy Selection** |

---

# 3. ARRAYS: MEMORY LAYOUT & FUNDAMENTAL ALGORITHMS

## 3.1 CLR Internals: Array Layout on the Managed Heap & CPU Cache Locality

In C#, arrays are reference types (`System.Array`), but their payload is allocated as a single, contiguous block of raw memory on the Managed Heap.

### The Memory Layout of `int[] arr = new int[4] { 10, 20, 30, 40 }` on 64-bit .NET:

```
Managed Heap Memory Address:
[Object Header: 8 bytes]   -> SyncBlockIndex (lock synchronization, hash codes)
[MethodTable Pointer: 8B] -> Points to System.Int32[] Type Descriptor
[Length: 8 bytes (padded)] -> 32-bit integer length (4) + 4 bytes padding
[Element 0: 4 bytes]       -> 10 (0x0000000A)
[Element 1: 4 bytes]       -> 20 (0x00000014)
[Element 2: 4 bytes]       -> 30 (0x0000001E)
[Element 3: 4 bytes]       -> 40 (0x00000028)
Total Overhead: 24 bytes header + (4 elements * 4 bytes) = 40 bytes.
```

### Why Contiguous Memory Matters: CPU Cache Lines
Modern CPUs do not fetch single bytes from RAM; they fetch memory in **64-byte Cache Lines** into L1, L2, and L3 caches.
- When your code accesses `arr[0]`, the CPU loads the entire 64-byte block containing `arr[0]`, `arr[1]`, `arr[2]`, up to `arr[15]`.
- Linear array traversals (`for (int i = 0; i < arr.Length; i++)`) yield **$100\%$ cache hits**. The CPU's hardware prefetcher detects the sequential stride and loads the next cache line before your code even requests it.
- In contrast, traversing a `LinkedList<T>` causes cache misses on every node because nodes are scattered non-contiguously across the heap.

```
Sequential Array Traversal (Cache Line Hit):
[ L1 Cache: 64 Bytes ] ──> [arr[0] | arr[1] | arr[2] | arr[3] | arr[4] | ...]
Single RAM fetch satisfies 16 integer reads! Latency: ~1 nanosecond.

Linked Node Traversal (Cache Miss):
Node 0 (0x00FF10) ──> RAM Fetch (~100ns) ──> Dereference next (0x0A9B20) ──> RAM Fetch (~100ns)
```

---

## 3.2 Modern .NET 8 Mechanics: `Span<T>`, `ReadOnlySpan<T>`, and Zero-Allocation Slicing

Traditionally in C#, if you wanted a sub-segment of an array, you had to allocate a new array:
```csharp
// NAIVE: Allocates a new array on the GC Heap!
int[] subArray = arr.Skip(2).Take(5).ToArray();
```

In .NET 8, `Span<T>` and `ReadOnlySpan<T>` are **ref structs** that live exclusively on the stack. A `Span<T>` consists simply of:
1. An interior pointer (`ref T _reference`).
2. A 32-bit length (`int _length`).

```csharp
// MODERN ZERO-ALLOCATION SLICING:
Span<int> slice = arr.AsSpan().Slice(start: 2, length: 5);
// Memory allocated: 0 bytes! Points directly into the existing heap buffer.
```

> **Interview Tip:** Whenever you demonstrate an array or string algorithm, mentioning or using `Span<T>` immediately signals to the interviewer that you understand low-level .NET memory dynamics and high-performance engineering.

---

## 3.3 Problem 1: Two Sum (Unsorted & Sorted Variants)

### 1. How to Think
You need to find two numbers in an array that add up to a target sum.
- **Mental Model:** For any number $X$, its required partner is $Target - X$ (the "complement").
- **Pattern Recognition:**
  - If unsorted: Can we remember what we've already seen? Storing numbers in a `Dictionary<int, int>` gives $O(1)$ lookups.
  - If sorted: The smallest elements are at the left, largest at the right. A converging Two-Pointer approach eliminates the need for extra memory.

### 2. How Interviewer Thinks
- Does the candidate immediately jump to nested loops ($O(N^2)$)?
- Do they understand the trade-off between Time and Space (using a Hash Map to reduce time from $O(N^2)$ to $O(N)$ at the expense of $O(N)$ RAM)?
- Do they ask if the array is sorted?

### 3. Brute Force
Test all pairs with nested loops.
- **Time Complexity:** $O(N^2)$
- **Space Complexity:** $O(1)$

```csharp
public int[] TwoSumBruteForce(int[] nums, int target)
{
    for (int i = 0; i < nums.Length; i++)
    {
        for (int j = i + 1; j < nums.Length; j++)
        {
            if (nums[i] + nums[j] == target)
                return new[] { i, j };
        }
    }
    return Array.Empty<int>();
}
```

### 4. Optimization
Use a Hash Map (`Dictionary<int, int>`). As we iterate through each element `nums[i]`, we compute `complement = target - nums[i]`.
- If `complement` exists in our dictionary, we have found our pair: return `[dictionary[complement], i]`.
- If not, insert `nums[i]` with its index `i` into the dictionary and continue.

### 5. Dry Run
**Input:** `nums = [2, 7, 11, 15]`, `target = 9`

| Step | Index $i$ | Value `nums[i]` | Complement (`9 - nums[i]`) | Map Contains Complement? | Action / Map State |
|---|---|---|---|---|---|
| 1 | 0 | 2 | $9 - 2 = 7$ | No | Add `{2: 0}` to Map |
| 2 | 1 | 7 | $9 - 7 = 2$ | **Yes!** (Index 0) | Return `[0, 1]` |

### 6. C# Solution (.NET 8)

```csharp
using System;
using System.Collections.Generic;

public static class TwoSumSolver
{
    /// <summary>
    /// Finds two indices such that their values sum to the specified target.
    /// Scenario: Matching transaction debits and credits in an Order Settlement Engine.
    /// </summary>
    public static int[] TwoSum(int[] nums, int target)
    {
        ArgumentNullException.ThrowIfNull(nums);

        // Pre-size dictionary to avoid rehashing allocations
        var seenComplements = new Dictionary<int, int>(capacity: nums.Length);

        for (int i = 0; i < nums.Length; i++)
        {
            int current = nums[i];
            int complement = target - current;

            if (seenComplements.TryGetValue(complement, out int complementIndex))
            {
                return [complementIndex, i]; // C# 12 collection expression
            }

            // In case of duplicates, keeping the latest index is acceptable unless specified
            seenComplements[current] = i;
        }

        return [];
    }

    /// <summary>
    /// Sorted Variant: Solves Two Sum in O(1) auxiliary space using Converging Two Pointers.
    /// </summary>
    public static int[] TwoSumSorted(ReadOnlySpan<int> sortedNums, int target)
    {
        int left = 0;
        int right = sortedNums.Length - 1;

        while (left < right)
        {
            long currentSum = (long)sortedNums[left] + sortedNums[right]; // Guard against integer overflow

            if (currentSum == target)
            {
                return [left, right];
            }

            if (currentSum < target)
            {
                left++; // Need a larger sum, move left pointer rightward
            }
            else
            {
                right--; // Need a smaller sum, move right pointer leftward
            }
        }

        return [];
    }
}
```

### 7. Complexity Analysis
- **Unsorted (Hash Map):**
  - **Time Complexity:** $O(N)$ — Single pass through the array. Dictionary lookup and insertion are $O(1)$ average.
  - **Space Complexity:** $O(N)$ — Stores up to $N$ elements in the dictionary on the Managed Heap.
- **Sorted (Two Pointers):**
  - **Time Complexity:** $O(N)$ — Each step moves at least one pointer; at most $N$ iterations.
  - **Space Complexity:** $O(1)$ — Only two scalar pointers on the stack.

### 8. Variations
- **What if array contains duplicate values?** The Hash Map handles this naturally if we check for complement *before* overwriting the index.
- **What if we need all unique pairs that sum to target?** Sort the array first, then use two pointers while skipping duplicates with `while (left < right && nums[left] == nums[left+1]) left++;`.

### 9. Follow-up Questions
- **Interviewer:** *"What happens if two numbers sum to greater than `int.MaxValue`?"*
  - **Answer:** It will cause an arithmetic overflow, producing an incorrect negative number. We cast the summation to `long` before comparing.
- **Interviewer:** *"Can Dictionary lookup ever degrade to $O(N)$?"*
  - **Answer:** Yes, if malicious inputs produce severe hash collisions (hash collision attack). In .NET, string hash codes are randomized per app domain to mitigate this.

---

## 3.4 Problem 2: Kadane's Maximum Subarray Sum (Transaction Delta Optimization)

### 1. How to Think
You are given an array of integers (which can include negative numbers). You need to find the contiguous subarray that has the largest sum.
- **Production Analogy:** In a Payment Gateway audit log, each integer represents a net minute-by-minute profit/loss delta. We want to find the consecutive sequence of minutes with the highest net financial surge.
- **Mental Model:** As you move forward, you have two choices at element $i$:
  1. Add element $i$ to your current running streak.
  2. Abandon your previous streak and start a fresh streak at element $i$ (if the previous streak was dragging you down below zero).

### 2. How Interviewer Thinks
- Do they understand Greedy / Dynamic Programming intuition?
- Can they avoid the $O(N^3)$ or $O(N^2)$ prefix-sum brute force?
- Do they properly handle an array where **all numbers are negative**? (Common bug: initializing `maxSum = 0` instead of `nums[0]`).

### 3. Brute Force
Compute sum of all possible subarrays `nums[i..j]`.
- **Time Complexity:** $O(N^2)$ (or $O(N^3)$ without cumulative sum).
- **Space Complexity:** $O(1)$.

### 4. Optimization (Kadane's Algorithm)
Maintain two variables:
- `currentSum`: The best subarray sum ending at the current index.
- `maxSum`: The global best sum seen across the entire array.
At each step:
$$\text{currentSum} = \max(\text{nums}[i], \text{currentSum} + \text{nums}[i])$$
$$\text{maxSum} = \max(\text{maxSum}, \text{currentSum})$$

### 5. Dry Run
**Input:** `nums = [-2, 1, -3, 4, -1, 2, 1, -5, 4]`

| $i$ | `nums[i]` | `currentSum + nums[i]` | New `currentSum` = $\max(\text{nums}[i], \text{sum})$ | New `maxSum` | Explanation |
|---|---|---|---|---|---|
| Init | - | - | -2 | -2 | Start with index 0 |
| 1 | 1 | $-2 + 1 = -1$ | $\max(1, -1) = \mathbf{1}$ | $\max(-2, 1) = \mathbf{1}$ | Drop negative streak, start fresh |
| 2 | -3 | $1 + (-3) = -2$ | $\max(-3, -2) = \mathbf{-2}$ | 1 | Keep running streak |
| 3 | 4 | $-2 + 4 = 2$ | $\max(4, 2) = \mathbf{4}$ | $\max(1, 4) = \mathbf{4}$ | Drop negative streak, start at 4 |
| 4 | -1 | $4 + (-1) = 3$ | $\max(-1, 3) = \mathbf{3}$ | 4 | |
| 5 | 2 | $3 + 2 = 5$ | $\max(2, 5) = \mathbf{5}$ | $\max(4, 5) = \mathbf{5}$ | |
| 6 | 1 | $5 + 1 = 6$ | $\max(1, 6) = \mathbf{6}$ | $\max(5, 6) = \mathbf{6}$ | **Peak reached: [4, -1, 2, 1]** |
| 7 | -5 | $6 + (-5) = 1$ | $\max(-5, 1) = \mathbf{1}$ | 6 | |
| 8 | 4 | $1 + 4 = 5$ | $\max(4, 5) = \mathbf{5}$ | 6 | Final result: 6 |

### 6. C# Solution (.NET 8)

```csharp
using System;

public static class SubarrayOptimizer
{
    /// <summary>
    /// Computes the maximum contiguous subarray sum using Kadane's Algorithm.
    /// Scenario: Finding the peak operational revenue delta in a payment settlement window.
    /// </summary>
    public static int MaxSubArray(ReadOnlySpan<int> nums)
    {
        if (nums.IsEmpty)
            throw new ArgumentException("Array cannot be empty.", nameof(nums));

        int currentStreak = nums[0];
        int globalMax = nums[0];

        for (int i = 1; i < nums.Length; i++)
        {
            int val = nums[i];
            // Either continue the previous streak or start anew at current element
            currentStreak = (currentStreak > 0) ? currentStreak + val : val;

            if (currentStreak > globalMax)
            {
                globalMax = currentStreak;
            }
        }

        return globalMax;
    }

    /// <summary>
    /// Variation: Returns the actual start and end indices of the maximal subarray.
    /// </summary>
    public static (int MaxSum, int StartIndex, int EndIndex) MaxSubArrayWithIndices(ReadOnlySpan<int> nums)
    {
        if (nums.IsEmpty)
            throw new ArgumentException("Array cannot be empty.", nameof(nums));

        int globalMax = nums[0];
        int currentStreak = nums[0];
        int start = 0, end = 0, tempStart = 0;

        for (int i = 1; i < nums.Length; i++)
        {
            if (currentStreak > 0)
            {
                currentStreak += nums[i];
            }
            else
            {
                currentStreak = nums[i];
                tempStart = i; // Mark candidate start
            }

            if (currentStreak > globalMax)
            {
                globalMax = currentStreak;
                start = tempStart;
                end = i;
            }
        }

        return (globalMax, start, end);
    }
}
```

### 7. Complexity Analysis
- **Time Complexity:** $O(N)$ — Single linear scan through the array.
- **Space Complexity:** $O(1)$ — Only a few 32-bit registers allocated on the thread execution stack.

### 8. Variations
- **Circular Subarray Sum:** What if the array wraps around?
  - *Insight:* Max circular sum is either standard `MaxSubArray` or `TotalSum - MinSubArray(nums)`.
- **Maximum Product Subarray:** What if we need the product instead of sum?
  - *Insight:* Negative numbers multiplied by negative numbers become positive! We must track both `currentMax` and `currentMin`.

### 9. Follow-up Questions
- **Interviewer:** *"What if all numbers are negative, e.g., `[-5, -2, -8]`?"*
  - **Answer:** By initializing both `currentStreak` and `globalMax` to `nums[0]`, the algorithm correctly returns `-2`. If we had initialized `globalMax = 0`, it would return 0, which is incorrect because 0 is not in the array.

---

## 3.5 Problem 3: Rotate Array (Cyclic Shifts & In-Place Reversals)

### 1. How to Think
Rotate an array of $N$ elements to the right by $K$ steps.
- **Example:** `[1, 2, 3, 4, 5, 6, 7]`, $K = 3 \implies [5, 6, 7, 1, 2, 3, 4]$.
- **Pattern Recognition:**
  - Notice what happens to the segments: The last $K$ elements move to the front; the first $N-K$ elements shift to the back.
  - Can we achieve this without allocating an auxiliary array of size $N$?

### 2. How Interviewer Thinks
- Tests candidate's ability to manipulate memory in-place.
- Tests modular arithmetic: What if $K > N$? ($K = K \pmod N$).
- Distinguishes between someone who creates a temporary array ($O(N)$ space) vs. someone who knows the Three-Step In-Place Reversal algorithm ($O(1)$ space).

### 3. Brute Force
Shift the array by 1 step, $K$ times. Shifting 1 step takes $O(N)$ time.
- **Time Complexity:** $O(N \times K)$ — Unacceptable for large $N$ and $K$.
- **Space Complexity:** $O(1)$.

### 4. Optimization (The Three-Step Reversal Magic)
Mathematical property of reversal:
1. Reverse the entire array: `[7, 6, 5, 4, 3, 2, 1]`
2. Reverse the first $K$ elements: `[5, 6, 7, 4, 3, 2, 1]`
3. Reverse the remaining $N-K$ elements: `[5, 6, 7, 1, 2, 3, 4]`
Every element is now in its exact rotated destination!

### 5. Dry Run
**Input:** `nums = [1, 2, 3, 4, 5]`, $K = 2$. Length $N = 5$. $K = 2 \pmod 5 = 2$.
1. **Reverse all:** `[1, 2, 3, 4, 5]` $\rightarrow$ `[5, 4, 3, 2, 1]`
2. **Reverse first $K=2$ (`0..1`):** `[4, 5, 3, 2, 1]`
3. **Reverse rest (`2..4`):** `[4, 5, 1, 2, 3]`
Result is `[4, 5, 1, 2, 3]`. Correct!

### 6. C# Solution (.NET 8)

```csharp
using System;

public static class ArrayRotator
{
    /// <summary>
    /// Rotates array in-place to the right by k steps in O(N) time and O(1) space.
    /// Scenario: Shifting active server partitions in a round-robin load balancer.
    /// </summary>
    public static void Rotate(Span<int> nums, int k)
    {
        if (nums.Length <= 1) return;

        k %= nums.Length; // Normalize k when k >= Length
        if (k == 0) return;

        // Step 1: Reverse entire span
        Reverse(nums, 0, nums.Length - 1);

        // Step 2: Reverse first k elements
        Reverse(nums, 0, k - 1);

        // Step 3: Reverse remaining n - k elements
        Reverse(nums, k, nums.Length - 1);
    }

    private static void Reverse(Span<int> span, int start, int end)
    {
        while (start < end)
        {
            // C# tuple swap (compiled to efficient register swaps without temp variable)
            (span[start], span[end]) = (span[end], span[start]);
            start++;
            end--;
        }
    }
}
```

### 7. Complexity Analysis
- **Time Complexity:** $O(N)$ — Phase 1 reverses $N$ items; Phase 2 reverses $K$ items; Phase 3 reverses $N-K$ items. Total operations $= 2N \implies O(N)$.
- **Space Complexity:** $O(1)$ — Zero memory allocations. All modifications occur in-place.

### 8. Variations
- **Rotate Left by $K$ steps:**
  - *Equivalent:* Rotating left by $K$ is identical to rotating right by $N - K$.

---

## 3.6 Problem 4: Find Duplicate Number (Floyd's Cycle Detection on Array Pointers)

### 1. How to Think
Given an array of integers `nums` containing $N + 1$ integers where each integer is in the range $[1, N]$ inclusive. There is only one repeated number, but it could be repeated multiple times.
- **Strict Constraints:** You must not modify the array and you must use only $O(1)$ extra space.
- **Mental Model:** Treat the array as a Linked List!
  - Let index $i$ point to node `nums[i]`.
  - Because numbers are in range $[1, N]$, index 0 will always point to a valid index.
  - Because there is a duplicate number, two different indices will point to the same next index.
  - **A duplicate value creates a cycle in the directed graph!**

### 2. How Interviewer Thinks
- Tests whether the candidate can map an Array problem into a Graph / Linked List Cycle problem.
- Interviewers love this problem because naive solutions (sorting or hash sets) violate either the $O(1)$ space or immutable array constraints.

### 3. Brute Force
- Sorting the array: $O(N \log N)$ time, but modifies the array.
- `HashSet<int>`: $O(N)$ time, but consumes $O(N)$ space.

### 4. Optimization (Floyd's Tortoise and Hare on Array)
1. Initialize `slow = nums[0]`, `fast = nums[0]`.
2. Move `slow = nums[slow]` (1 step), `fast = nums[nums[fast]]` (2 steps).
3. When they collide inside the cycle:
   - Reset `slow = nums[0]`.
   - Keep `fast` at collision point.
   - Move both one step at a time (`slow = nums[slow]`, `fast = nums[fast]`).
4. The node where they meet again is the entrance to the cycle $\implies$ **the duplicate number!**

### 5. Dry Run
**Input:** `nums = [1, 3, 4, 2, 2]`. $N = 4$. Indices: 0, 1, 2, 3, 4.
Graph transitions:
- $0 \rightarrow \text{nums}[0] = 1$
- $1 \rightarrow \text{nums}[1] = 3$
- $3 \rightarrow \text{nums}[3] = 2$
- $2 \rightarrow \text{nums}[2] = 4$
- $4 \rightarrow \text{nums}[4] = 2$ (Points back to 2! Cycle is: $2 \rightarrow 4 \rightarrow 2$)

**Phase 1: Finding Collision Point**
- Start: `slow = nums[0] = 1`, `fast = nums[0] = 1`
- Iteration 1: `slow = nums[1] = 3`, `fast = nums[nums[1]] = nums[3] = 2`
- Iteration 2: `slow = nums[3] = 2`, `fast = nums[nums[2]] = nums[4] = 2`
- Collision! Both are at 2.

**Phase 2: Finding Cycle Entrance**
- Reset `slow = nums[0] = 1`. `fast` remains at 2.
- Step 1: `slow = nums[1] = 3`, `fast = nums[2] = 4`
- Step 2: `slow = nums[3] = 2`, `fast = nums[4] = 2`
- They meet at **2**. Duplicate is 2!

### 6. C# Solution (.NET 8)

```csharp
using System;

public static class DuplicateFinder
{
    /// <summary>
    /// Finds the duplicate number in an immutable array using Floyd's Tortoise and Hare.
    /// Scenario: Detecting duplicate transaction identifiers in a validated batch stream.
    /// </summary>
    public static int FindDuplicate(ReadOnlySpan<int> nums)
    {
        if (nums.Length <= 1) return -1;

        int slow = nums[0];
        int fast = nums[0];

        // Phase 1: Detect cycle
        do
        {
            slow = nums[slow];
            fast = nums[nums[fast]];
        } while (slow != fast);

        // Phase 2: Locate cycle start (duplicate value)
        slow = nums[0];
        while (slow != fast)
        {
            slow = nums[slow];
            fast = nums[fast];
        }

        return slow;
    }
}
```

### 7. Complexity Analysis
- **Time Complexity:** $O(N)$ — Phase 1 takes at most $2N$ steps; Phase 2 takes at most $N$ steps.
- **Space Complexity:** $O(1)$ — Only two scalar pointers on the stack. Array remains unmutated.

---

## 3.7 Problem 5: Merge Intervals (Booking & Calendar Overlap Consolidation)

### 1. How to Think
Given an array of intervals where `intervals[i] = [start_i, end_i]`, merge all overlapping intervals and return an array of the non-overlapping intervals that cover all intervals in the input.
- **Production Analogy:** In a SaaS Appointment Booking system or Healthcare Operating Room scheduler, multiple overlapping reservations must be compacted into continuous maintenance blocks.
- **Mental Model:** If the intervals are sorted by their start time:
  - Interval $B$ overlaps with Interval $A$ if and only if:
    $$\text{Start}_B \le \text{End}_A$$
  - When they overlap, their merged interval has:
    $$\text{End}_{\text{merged}} = \max(\text{End}_A, \text{End}_B)$$

### 2. How Interviewer Thinks
- Tests ability to sort multi-dimensional structures.
- Tests interval manipulation and tracking boundary states.
- Tests understanding of custom comparers in C# (`Array.Sort` with lambda or `IComparable`).

### 3. Optimization
1. Sort intervals by start time in ascending order.
2. Initialize an output list with the first interval.
3. Iterate through subsequent intervals:
   - If the current interval's `start` is $\le$ the last merged interval's `end`, update the last interval's `end` to $\max(\text{last.end}, \text{current.end})$.
   - Otherwise, append the current interval as a new non-overlapping interval.

### 4. Dry Run
**Input:** `[[1, 3], [2, 6], [8, 10], [15, 18]]`
- Already sorted by start times.
- List starts with: `[[1, 3]]`
- Inspect `[2, 6]`: $2 \le 3 \implies$ Overlap! Merge: end becomes $\max(3, 6) = 6$. List: `[[1, 6]]`.
- Inspect `[8, 10]`: $8 > 6 \implies$ No overlap. Add to list. List: `[[1, 6], [8, 10]]`.
- Inspect `[15, 18]`: $15 > 10 \implies$ No overlap. Add to list. List: `[[1, 6], [8, 10], [15, 18]]`.

### 5. C# Solution (.NET 8)

```csharp
using System;
using System.Collections.Generic;

public static class IntervalMerger
{
    public readonly record struct Interval(int Start, int End);

    /// <summary>
    /// Merges overlapping time intervals.
    /// Scenario: Healthcare operating room appointment conflict consolidation.
    /// </summary>
    public static List<Interval> Merge(Interval[] intervals)
    {
        if (intervals == null || intervals.Length == 0)
            return [];

        // Sort by Start time ascending
        Array.Sort(intervals, static (a, b) => a.Start.CompareTo(b.Start));

        var merged = new List<Interval>(capacity: intervals.Length);
        Interval current = intervals[0];

        for (int i = 1; i < intervals.Length; i++)
        {
            Interval next = intervals[i];

            if (next.Start <= current.End)
            {
                // Overlap detected: expand current interval's boundary
                current = current with { End = Math.Max(current.End, next.End) };
            }
            else
            {
                // Disjoint interval: commit current and advance
                merged.Add(current);
                current = next;
            }
        }

        // Commit the final interval
        merged.Add(current);
        return merged;
    }
}
```

### 6. Complexity Analysis
- **Time Complexity:** $O(N \log N)$ dominated by sorting the intervals. The merging pass is a single linear scan $O(N)$.
- **Space Complexity:** $O(N)$ to hold the merged output intervals (and $O(\log N)$ stack space for IntroSort).

---

## 3.8 Problem 6: Product of Array Except Self (Prefix-Suffix Accumulation Without Division)

### 1. How to Think
Given an integer array `nums`, return an array `answer` such that `answer[i]` is equal to the product of all elements of `nums` except `nums[i]`.
- **Constraint:** You **cannot use the division operator `/`**, and the algorithm must run in $O(N)$ time.
- **Mental Model:** For any index $i$:
  $$\text{Product Except Self} = (\text{Product of all elements to the left of } i) \times (\text{Product of all elements to the right of } i)$$

### 2. How Interviewer Thinks
- Tests if candidate can think in terms of cumulative prefix and suffix arrays.
- Tests if candidate can optimize space: from using two auxiliary arrays ($O(N)$ space) to using the output array itself for prefix accumulation and a running scalar for suffix ($O(1)$ auxiliary space).

### 3. Optimization
1. Allocate result array `output` of size $N$.
2. **First Pass (Prefix):** Let `output[i]` store the product of all elements to the left of $i$.
3. **Second Pass (Suffix):** Maintain a running scalar `suffixProduct = 1` moving from right to left. Multiply `output[i]` by `suffixProduct`, then update `suffixProduct *= nums[i]`.

### 4. Dry Run
**Input:** `nums = [1, 2, 3, 4]`

**Pass 1 (Prefix into `output`):**
- `output[0] = 1` (nothing to the left)
- `output[1] = output[0] * nums[0] = 1 * 1 = 1`
- `output[2] = output[1] * nums[1] = 1 * 2 = 2`
- `output[3] = output[2] * nums[2] = 2 * 3 = 6`
- `output` is now `[1, 1, 2, 6]`

**Pass 2 (Suffix accumulation):**
- `suffix = 1`
- $i = 3$: `output[3] = 6 * 1 = 6`. `suffix = 1 * 4 = 4`
- $i = 2$: `output[2] = 2 * 4 = 8`. `suffix = 4 * 3 = 12`
- $i = 1$: `output[1] = 1 * 12 = 12`. `suffix = 12 * 2 = 24`
- $i = 0$: `output[0] = 1 * 24 = 24`.
Final result: `[24, 12, 8, 6]`. Correct! ($2*3*4=24$, $1*3*4=12$, $1*2*4=8$, $1*2*3=6$).

### 5. C# Solution (.NET 8)

```csharp
using System;

public static class ArrayProductSolver
{
    /// <summary>
    /// Computes product of array except self in O(N) time and O(1) auxiliary space (excluding output).
    /// Scenario: Calculating multi-factor risk weighting scores across independent portfolio tranches.
    /// </summary>
    public static int[] ProductExceptSelf(int[] nums)
    {
        ArgumentNullException.ThrowIfNull(nums);
        int n = nums.Length;
        int[] result = new int[n];

        // Step 1: Compute prefix products directly in result array
        result[0] = 1;
        for (int i = 1; i < n; i++)
        {
            result[i] = result[i - 1] * nums[i - 1];
        }

        // Step 2: Accumulate suffix products in a single scalar variable
        int runningSuffix = 1;
        for (int i = n - 1; i >= 0; i--)
        {
            result[i] *= runningSuffix;
            runningSuffix *= nums[i];
        }

        return result;
    }
}
```

### 6. Complexity Analysis
- **Time Complexity:** $O(N)$ — Exactly two linear passes over the array.
- **Space Complexity:** $O(1)$ auxiliary space (the output array does not count toward auxiliary space complexity).

---

# 4. STRINGS: IMMUTABILITY, MEMORY OPTIMIZATION & PATTERN MATCHING

## 4.1 CLR Internals: String Layout, Interning Pool, UTF-16, and GC Pressures

In C#, a `System.String` is an immutable reference type. Let's look at its low-level memory representation on a 64-bit machine:

```
Managed Heap Memory Structure of string s = "Infosys":
[SyncBlockIndex: 8 bytes]
[MethodTable Pointer: 8 bytes] (Points to System.String Type Descriptor)
[String Length: 4 bytes]        (7 characters)
[Character Buffer: 7 * 2 bytes] (UTF-16 encoding: 'I', 'n', 'f', 'o', 's', 'y', 's' = 14 bytes)
[Null Terminator: 2 bytes]      (0x0000 for native C/C++ interop compatibility)
[Padding: 4 bytes]              (64-bit alignment)
Total Memory = 8 + 8 + 4 + 14 + 2 + 4 = 40 bytes!
```

### The String Immutability Trap in Loops
Because strings are immutable, modifying a string creates a completely new string object on the Managed Heap:
```csharp
// NAIVE CONCATENATION (DANGEROUS IN HIGH-THROUGHPUT CODE)
string result = "";
for (int i = 0; i < n; i++)
{
    result += i.ToString(); // Allocates a new string on EVERY iteration!
}
// Time Complexity: O(N^2) due to repeated memory copying.
// Generates massive Gen 0 garbage collection pressure.
```

### Mitigation Strategies in .NET 8
1. `StringBuilder`: Backed by an internal character buffer that dynamically expands, amortizing allocations.
2. `ReadOnlySpan<char>`: Enables zero-allocation parsing, sub-stringing, and character comparison without creating new heap strings.
3. String Interning: The CLR maintains an internal hash table called the **Intern Pool**. Literal strings with identical content share the exact same reference address in memory (`string.IsInterned(s)`).

---

## 4.2 Problem 7: Reverse String & Words in Sentence (Zero-Allocation Token Reversal)

### 1. How to Think
- **Task A:** Reverse a character array in-place.
- **Task B:** Reverse the order of words in a sentence string: `"the sky is blue"` $\implies$ `"blue is sky the"`.
- **Mental Model:**
  1. Reverse the entire character array: `"blue is sky the"` inverted becomes `"eulb si yks eht"`.
  2. Iterate through and reverse each individual word within its own bounds: `"blue is sky the"`.
  3. Clean up excess whitespace using a two-pointer read/write compaction.

### 2. How Interviewer Thinks
- Tests if the developer knows how to avoid `string.Split(' ')` (which allocates an array and a string per word).
- Tests low-level pointer management.

### 3. C# Solution (.NET 8)

```csharp
using System;

public static class StringReverser
{
    /// <summary>
    /// Reverses words in a sentence in-place without allocating auxiliary arrays.
    /// Scenario: Sanitizing and standardizing inbound text for an SMS notification gateway.
    /// </summary>
    public static string ReverseWords(string s)
    {
        if (string.IsNullOrWhiteSpace(s)) return string.Empty;

        // Convert to mutable char buffer for in-place transformation
        char[] chars = s.ToCharArray();
        int n = chars.Length;

        // Step 1: Clean spaces and shift characters forward (Read/Write Two Pointers)
        int write = 0;
        for (int read = 0; read < n; read++)
        {
            if (chars[read] != ' ')
            {
                if (write > 0) chars[write++] = ' '; // Add single space between words

                int wordStart = read;
                while (read < n && chars[read] != ' ')
                {
                    chars[write++] = chars[read++];
                }
            }
        }

        // Sliced span containing only the compacted characters
        Span<char> cleanedSpan = chars.AsSpan(0, write);

        // Step 2: Reverse entire compacted span
        cleanedSpan.Reverse();

        // Step 3: Reverse each individual word back
        int start = 0;
        for (int i = 0; i <= cleanedSpan.Length; i++)
        {
            if (i == cleanedSpan.Length || cleanedSpan[i] == ' ')
            {
                cleanedSpan[start..i].Reverse();
                start = i + 1;
            }
        }

        return new string(cleanedSpan);
    }
}
```

### 4. Complexity Analysis
- **Time Complexity:** $O(N)$ — Single pass for space compaction, single pass for full reversal, single pass for word reversals. Total operations $\approx 3N \implies O(N)$.
- **Space Complexity:** $O(N)$ only for the initial char buffer. Zero intermediate allocations.

---

## 4.3 Problem 8: Valid Anagram (Frequency Buckets vs. Sorting)

### 1. How to Think
Two strings $S$ and $T$ are anagrams if they contain the exact same characters with the exact same frequencies.
- **Mental Model:** A hash map or fixed-size array can tally character frequencies.
- If strings have different lengths, return `false` immediately.

### 2. How Interviewer Thinks
- Did the candidate sort both strings ($O(N \log N)$) or use a frequency array ($O(N)$)?
- Did they use a heavy `Dictionary<char, int>` or a lightweight `int[26]` stack-allocated buffer?

### 3. C# Solution (.NET 8)

```csharp
using System;

public static class AnagramValidator
{
    /// <summary>
    /// Determines if two strings are valid anagrams using stack-allocated frequency buckets.
    /// Scenario: Detecting duplicate transaction signatures under permutated attribute orders.
    /// </summary>
    public static bool IsAnagram(ReadOnlySpan<char> s, ReadOnlySpan<char> t)
    {
        if (s.Length != t.Length) return false;

        // Stack-allocated 26-element buffer for lowercase ASCII English letters (Zero Heap Allocation)
        Span<int> charCounts = stackalloc int[26];

        for (int i = 0; i < s.Length; i++)
        {
            charCounts[s[i] - 'a']++;
            charCounts[t[i] - 'a']--;
        }

        foreach (int count in charCounts)
        {
            if (count != 0) return false;
        }

        return true;
    }
}
```

### 4. Complexity Analysis
- **Time Complexity:** $O(N)$ — Single pass over both strings of length $N$.
- **Space Complexity:** $O(1)$ — `stackalloc int[26]` consumes 104 bytes on the thread stack; zero GC heap allocations.

---

## 4.4 Problem 9: Longest Substring Without Repeating Characters (Sliding Window)

### 1. How to Think
Find the length of the longest substring without repeating characters.
- **Example:** `"abcabcbb"` $\implies 3$ (`"abc"`).
- **Pattern Recognition:** Contiguous substring with a uniqueness condition $\implies$ **Sliding Window**.
- As the window `[left, right]` expands, if we encounter a character already inside the window, where should `left` jump?
  - Instead of incrementing `left` one step at a time, store the **last seen index** of each character. `left` can immediately jump past the previous duplicate's position!

### 2. How Interviewer Thinks
- Tests candidate's mastery of the Sliding Window invariant.
- Edge case: Does the candidate remember that `left` should **only move forward**? If a duplicate occurred *before* the current `left`, `left` must not move backward!

### 3. Dry Run
**Input:** `s = "abba"`

| `right` | Char `c` | Last Seen Index of `c` | Window Left | Max Length Update | Map Update |
|---|---|---|---|---|---|
| 0 | 'a' | None | 0 | $\max(0, 0 - 0 + 1) = 1$ | `'a' \rightarrow 0` |
| 1 | 'b' | None | 0 | $\max(1, 1 - 0 + 1) = 2$ | `'b' \rightarrow 1` |
| 2 | 'b' | 1 | Jump: $\max(0, 1 + 1) = 2$ | $\max(2, 2 - 2 + 1) = 2$ | `'b' \rightarrow 2` |
| 3 | 'a' | 0 | $\max(2, 0 + 1) = \mathbf{2}$ (Left does not move back!) | $\max(2, 3 - 2 + 1) = 2$ | `'a' \rightarrow 3` |
Result: 2.

### 4. C# Solution (.NET 8)

```csharp
using System;

public static class SubstringFinder
{
    /// <summary>
    /// Computes length of longest substring without repeating characters using Sliding Window.
    /// Scenario: Identifying unique sequence tokens in an authentication JWT claim string.
    /// </summary>
    public static int LengthOfLongestSubstring(ReadOnlySpan<char> s)
    {
        if (s.IsEmpty) return 0;

        // Use direct-mapped 128-element array for all standard ASCII characters
        Span<int> lastSeenIndex = stackalloc int[128];
        lastSeenIndex.Fill(-1);

        int maxLength = 0;
        int windowLeft = 0;

        for (int windowRight = 0; windowRight < s.Length; windowRight++)
        {
            char currentChar = s[windowRight];

            // If character seen and inside current window, jump left boundary past it
            if (currentChar < 128 && lastSeenIndex[currentChar] >= windowLeft)
            {
                windowLeft = lastSeenIndex[currentChar] + 1;
            }

            if (currentChar < 128)
            {
                lastSeenIndex[currentChar] = windowRight;
            }

            int currentWindowLength = windowRight - windowLeft + 1;
            if (currentWindowLength > maxLength)
            {
                maxLength = currentWindowLength;
            }
        }

        return maxLength;
    }
}
```

### 5. Complexity Analysis
- **Time Complexity:** $O(N)$ — `windowRight` scans from 0 to $N-1$ once.
- **Space Complexity:** $O(1)$ — 128 integers on stack frame.

---

## 4.5 Problem 10: String Compression (Run-Length Encoding with Write Pointers)

### 1. How to Think
Given an array of characters `chars`, compress it in-place using the following algorithm:
- For each group of consecutive repeating characters:
  - If group length is 1, append the character.
  - If group length is $> 1$, append the character followed by the group's length.
- **Example:** `['a', 'a', 'b', 'b', 'c', 'c', 'c']` $\implies$ `['a', '2', 'b', '2', 'c', '3']`, returns 6.
- **Pattern:** Read/Write Two Pointers.

### 2. C# Solution (.NET 8)

```csharp
using System;

public static class RunLengthCompressor
{
    /// <summary>
    /// Compresses character array in-place, returning new length.
    /// Scenario: Log message payload compression prior to transport over network wire.
    /// </summary>
    public static int Compress(Span<char> chars)
    {
        int writeIndex = 0;
        int readIndex = 0;

        while (readIndex < chars.Length)
        {
            char currentChar = chars[readIndex];
            int count = 0;

            // Count contiguous identical characters
            while (readIndex < chars.Length && chars[readIndex] == currentChar)
            {
                readIndex++;
                count++;
            }

            // Write character
            chars[writeIndex++] = currentChar;

            // Write count if greater than 1
            if (count > 1)
            {
                // Write each digit of count (handles count >= 10)
                Span<char> digitBuffer = stackalloc char[10];
                int digitsWritten = 0;
                int tempCount = count;

                while (tempCount > 0)
                {
                    digitBuffer[digitsWritten++] = (char)('0' + (tempCount % 10));
                    tempCount /= 10;
                }

                // Digits are extracted in reverse order; write back correctly
                for (int d = digitsWritten - 1; d >= 0; d--)
                {
                    chars[writeIndex++] = digitBuffer[d];
                }
            }
        }

        return writeIndex;
    }
}
```

### 3. Complexity Analysis
- **Time Complexity:** $O(N)$ — `readIndex` advances linearly through the array.
- **Space Complexity:** $O(1)$ — In-place mutation with zero heap allocations.

---

## 4.6 Problem 11: Palindrome Check & Longest Palindromic Substring

### 1. How to Think
A string is a palindrome if it reads the same forward and backward.
- For finding the **Longest Palindromic Substring**, every palindrome has a center:
  - Odd-length palindrome: Center is a single character (e.g., `"aba"`, center `'b'`).
  - Even-length palindrome: Center is between two characters (e.g., `"abba"`, center between `'b'` and `'b'`).
- Expanding outward from all $2N - 1$ centers takes $O(N^2)$ time and $O(1)$ space.

### 2. C# Solution (.NET 8)

```csharp
using System;

public static class PalindromeEngine
{
    /// <summary>
    /// Finds the longest palindromic substring using Center Expansion in O(1) auxiliary space.
    /// </summary>
    public static string LongestPalindrome(string s)
    {
        if (string.IsNullOrEmpty(s) || s.Length < 2) return s;

        int start = 0;
        int maxLen = 1;
        ReadOnlySpan<char> span = s.AsSpan();

        for (int i = 0; i < span.Length; i++)
        {
            // Expand odd length center
            ExpandAroundCenter(span, i, i, ref start, ref maxLen);
            // Expand even length center
            ExpandAroundCenter(span, i, i + 1, ref start, ref maxLen);
        }

        return s.Substring(start, maxLen);
    }

    private static void ExpandAroundCenter(ReadOnlySpan<char> span, int left, int right, ref int start, ref int maxLen)
    {
        while (left >= 0 && right < span.Length && span[left] == span[right])
        {
            int currentLength = right - left + 1;
            if (currentLength > maxLen)
            {
                maxLen = currentLength;
                start = left;
            }
            left--;
            right++;
        }
    }
}
```

---

# 5. HASHMAP & DICTIONARY INTERNALS: DEEP CLR DIVE & APPLICATIONS

## 5.1 CLR Deep Dive: How `Dictionary<TKey, TValue>` Works Internally in .NET 8

The `Dictionary<TKey, TValue>` in C# is arguably the most critical collection in enterprise .NET development. An interviewer will frequently ask:
> *"Explain step-by-step what happens internally in the CLR when you call `dict.Add(key, value)` or `dict[key]`."*

### Internal Data Structures in .NET 8 Source Code
`Dictionary<TKey, TValue>` does not store items in a linked list per bucket (unlike Java's `HashMap` which uses linked nodes or red-black trees). Instead, .NET uses **Array-Based Separate Chaining**.

It holds two primary private arrays:
```csharp
private int[] _buckets;       // 1-based index pointing into _entries
private Entry[] _entries;     // Dense array of Entry structs
```

Where `Entry` is defined as:
```csharp
private struct Entry
{
    public uint hashCode; // 31-bit hash code (top bit masked)
    public int next;      // 0-based index of next entry in chain (-1 if end)
    public TKey key;      // The actual key
    public TValue value;  // The actual value
}
```

### The Step-by-Step Execution Flow of `dict.Add(key, value)`:

```
Step 1: Compute Hash Code
        uint hashCode = (uint)comparer.GetHashCode(key);

Step 2: Determine Bucket Index
        int bucketIndex = (int)(hashCode % (uint)_buckets.Length);
        (In .NET 8, FastMod arithmetic is used instead of expensive '%' operator)

Step 3: Check for Collisions
        int i = _buckets[bucketIndex] - 1; // Convert 1-based bucket to 0-based entry index
        while (i >= 0)
        {
            if (_entries[i].hashCode == hashCode && comparer.Equals(_entries[i].key, key))
            {
                ThrowHelper.ThrowAddingDuplicateWithKeyArgumentException(key);
            }
            i = _entries[i].next; // Follow chain
        }

Step 4: Store in _entries Array
        int entryIndex = _count; // Next available slot in dense _entries array
        _entries[entryIndex].hashCode = hashCode;
        _entries[entryIndex].next = _buckets[bucketIndex] - 1; // Chain existing entry
        _entries[entryIndex].key = key;
        _entries[entryIndex].value = value;

Step 5: Update Bucket Table
        _buckets[bucketIndex] = entryIndex + 1; // Store 1-based index
        _count++;
```

```
Visual Architecture of .NET Dictionary:
_buckets array:      [ 2 ]      [ 0 ]      [ 1 ]       (Bucket 0 points to Entry 1, Bucket 2 points to Entry 0)
                       │                              │
                       ▼                              ▼
_entries array:   ┌───────────────┐              ┌───────────────┐
  Index 0:        │ Hash: 0x4F12  │              │ Hash: 0x8A10  │
                  │ Key: "OrderId"│              │ Key: "UserId" │
                  │ Val: 10091    │              │ Val: 552      │
                  │ Next: -1      │ (No chain)   │ Next: 2       │──┐ Points to Entry 2
                  └───────────────┘              └───────────────┘  │
  Index 1: (Unused / free)                                          │
                                                                    │
  Index 2 (Collision on Bucket 2):                                  ▼
                  ┌─────────────────────────────────────────────────┘
                  │ Hash: 0x8A10 (Identical bucket index!)
                  │ Key: "ClientId"
                  │ Val: 8810
                  │ Next: -1 (End of chain)
                  └───────────────────────────────
```

### Why Struct Keys Cause Performance Disasters if Implemented Incorrectly
If you use a custom `struct` as a dictionary key without implementing `IEquatable<T>` and overriding `GetHashCode()`:
1. `EqualityComparer<T>.Default` falls back to `ValueType.GetHashCode()` and `ValueType.Equals()`.
2. `ValueType.Equals()` uses **Reflection** under the hood to compare each field of the struct!
3. Passing the struct into `ValueType.Equals(object? obj)` causes **boxing** on every lookup, creating millions of temporary heap allocations.
4. **Result:** Your $O(1)$ dictionary lookup becomes 100x slower and floods the Gen 0 GC heap.

---

## 5.2 Problem 12: Group Anagrams (Categorization of Financial Audit Records)

### 1. How to Think
Given an array of strings, group the anagrams together.
- **Example:** `["eat", "tea", "tan", "ate", "nat", "bat"]` $\implies$ `[["bat"], ["nat", "tan"], ["ate", "eat", "tea"]]`.
- **Insight:** All anagrams share a canonical signature. Two choices for the key:
  1. Sort the characters of each string: `"eat"` $\rightarrow$ `"aet"`.
  2. Character count string: `"#1#0#0...#1"`.
  For short strings, sorting characters into a canonical string key is extremely fast.

### 2. C# Solution (.NET 8)

```csharp
using System;
using System.Collections.Generic;

public static class AnagramGrouper
{
    /// <summary>
    /// Groups words that are anagrams into common lists.
    /// Scenario: Categorizing mutated audit log signatures in a financial fraud detector.
    /// </summary>
    public static IList<IList<string>> GroupAnagrams(string[] strs)
    {
        if (strs == null || strs.Length == 0) return [];

        var groups = new Dictionary<string, IList<string>>(capacity: strs.Length);

        foreach (string word in strs)
        {
            // Canonical key by sorting characters
            char[] chars = word.ToCharArray();
            Array.Sort(chars);
            string key = new string(chars);

            if (!groups.TryGetValue(key, out var list))
            {
                list = new List<string>();
                groups[key] = list;
            }

            list.Add(word);
        }

        return new List<IList<string>>(groups.Values);
    }
}
```

### 3. Complexity Analysis
- **Time Complexity:** $O(N \cdot K \log K)$, where $N$ is number of words and $K$ is max word length.
- **Space Complexity:** $O(N \cdot K)$ to store keys and grouped strings in dictionary.

---

## 5.3 Problem 13: Top K Frequent Elements (Bucket Frequency vs. Min-Heap)

### 1. How to Think
Given an integer array `nums` and an integer `K`, return the $K$ most frequent elements.
- **Insight:** Instead of sorting all elements ($O(N \log N)$), count frequencies in a hash map ($O(N)$), then use **Bucket Sort**:
  - The maximum possible frequency of any element is $N$.
  - Create an array of lists: `List<int>[] buckets = new List<int>[N + 1]`.
  - Put each element into `buckets[frequency]`.
  - Traverse buckets backward from index $N$ down to 1 until $K$ elements are collected!

### 2. C# Solution (.NET 8)

```csharp
using System;
using System.Collections.Generic;

public static class FrequencyAnalyzer
{
    /// <summary>
    /// Identifies the Top K frequent items in O(N) linear time using Bucket Sort.
    /// Scenario: Extracting top error codes in an ASP.NET Core API telemetry ingestion pipeline.
    /// </summary>
    public static int[] TopKFrequent(int[] nums, int k)
    {
        ArgumentNullException.ThrowIfNull(nums);

        // Step 1: Frequency count
        var frequencyMap = new Dictionary<int, int>();
        foreach (int num in nums)
        {
            frequencyMap[num] = frequencyMap.GetValueOrDefault(num, 0) + 1;
        }

        // Step 2: Bucket array where index represents frequency
        var buckets = new List<int>[nums.Length + 1];

        foreach (var (num, freq) in frequencyMap)
        {
            buckets[freq] ??= new List<int>();
            buckets[freq].Add(num);
        }

        // Step 3: Gather top K from highest to lowest bucket
        int[] result = new int[k];
        int count = 0;

        for (int freq = buckets.Length - 1; freq >= 0 && count < k; freq--)
        {
            if (buckets[freq] != null)
            {
                foreach (int item in buckets[freq])
                {
                    result[count++] = item;
                    if (count == k) break;
                }
            }
        }

        return result;
    }
}
```

### 3. Complexity Analysis
- **Time Complexity:** $O(N)$ — Counting takes $O(N)$, bucket population takes $O(N)$, reverse scan takes at most $O(N)$.
- **Space Complexity:** $O(N)$ for frequency map and bucket arrays.

---

## 5.4 Problem 14: First Unique Character in a Stream

### 1. How to Think
Find the first non-repeating character in a string.
- Two-pass frequency tally:
  1. Pass 1: Tally counts of each character in a frequency table.
  2. Pass 2: Iterate through the original string; the first character with a count of 1 is the answer.

### 2. C# Solution (.NET 8)

```csharp
using System;

public static class UniqueCharSolver
{
    /// <summary>
    /// Returns the index of the first non-repeating character, or -1 if none exists.
    /// </summary>
    public static int FirstUniqChar(ReadOnlySpan<char> s)
    {
        Span<int> charFrequencies = stackalloc int[26];

        for (int i = 0; i < s.Length; i++)
        {
            charFrequencies[s[i] - 'a']++;
        }

        for (int i = 0; i < s.Length; i++)
        {
            if (charFrequencies[s[i] - 'a'] == 1)
            {
                return i;
            }
        }

        return -1;
    }
}
```

---


# 6. BINARY SEARCH: THE MONOTONIC MINDSET

## 6.1 The Universal Binary Search Paradigm

Many developers think Binary Search only applies to sorted arrays looking for an exact target number. **That is only 10% of its utility.**

Binary Search is fundamentally a tool for **Monotonic Decision Spaces**. If you have a predicate function $P(x)$ such that:
$$P(x) \text{ is } [\text{False}, \text{False}, \dots, \text{False}, \mathbf{True}, \mathbf{True}, \dots, \mathbf{True}]$$
Then Binary Search can find the **exact boundary** where False becomes True in $O(\log N)$ evaluations.

```
Monotonic Search Space:
Index:    0       1       2       3       4       5       6       7
Value:   12      24      35      48      60      75      89      99
P(x)>=50: F       F       F       F       T       T       T       T
                                          ▲
                          First True Boundary (Lower Bound)
```

### The Three Critical Invariants
1. **Midpoint Overflow Protection:**
   ```csharp
   // WRONG: Can overflow int.MaxValue (2,147,483,647) when low + high > Int32.MaxValue!
   int mid = (low + high) / 2;

   // CORRECT: Mathematically equivalent, immune to integer overflow:
   int mid = low + (high - low) / 2;
   ```
2. **Loop Condition (`low <= high` vs. `low < high`):**
   - Use `low <= high` when searching for a single target where the search space reduces to `low == high`. Inside, use `low = mid + 1` and `high = mid - 1`.
   - Use `low < high` when searching for an inflection point or boundary where `mid` can be the answer (`high = mid`).
3. **Termination Guarantee:** Every branch MUST shrink the search space by at least 1 element; otherwise, you risk an infinite loop.

---

## 6.2 Problem 15: Classic Binary Search (Exact Order ID Match)

### 1. How to Think
Find the index of a target value in a sorted array in $O(\log N)$ time.
- If target exists, return its index. If not, return `-1`.

### 2. How Interviewer Thinks
- Tests if you handle `low + (high - low) / 2`.
- Tests if your boundary updates eliminate infinite loops.

### 3. C# Solution (.NET 8)

```csharp
using System;

public static class BinarySearchEngine
{
    /// <summary>
    /// Searches for a target key in a sorted span in O(log N) time.
    /// Scenario: Locating a customer order record in a sorted partition block.
    /// </summary>
    public static int BinarySearch(ReadOnlySpan<int> sortedArr, int target)
    {
        int low = 0;
        int high = sortedArr.Length - 1;

        while (low <= high)
        {
            int mid = low + (high - low) / 2;
            int midVal = sortedArr[mid];

            if (midVal == target)
            {
                return mid;
            }

            if (midVal < target)
            {
                low = mid + 1; // Target is in the upper half
            }
            else
            {
                high = mid - 1; // Target is in the lower half
            }
        }

        return -1; // Target not found
    }
}
```

### 4. Complexity Analysis
- **Time Complexity:** $O(\log N)$ — Each iteration cuts the search space in half. For $N = 1,000,000$, $\log_2(1,000,000) \approx 20$ comparisons.
- **Space Complexity:** $O(1)$ — Only three local integer variables.

---

## 6.3 Problem 16: Search in Rotated Sorted Array (Shifted Ledger Partitions)

### 1. How to Think
An array sorted in ascending order is rotated at an unknown pivot index (e.g., `[4, 5, 6, 7, 0, 1, 2]`). Find the index of `target` in $O(\log N)$ time.
- **Key Insight:** If you pick the midpoint `mid`, **at least one half of the array (either `[low..mid]` or `[mid..high]`) is guaranteed to be strictly sorted!**
- Determine which half is sorted:
  - If `nums[low] <= nums[mid]`, the left half is sorted.
    - Check if target falls within `[nums[low], nums[mid])`. If yes, narrow search to left (`high = mid - 1`). If no, search right (`low = mid + 1`).
  - Otherwise, the right half is sorted:
    - Check if target falls within `(nums[mid], nums[high]]`. If yes, narrow search to right (`low = mid + 1`). If no, search left (`high = mid - 1`).

### 2. How Interviewer Thinks
- Can the candidate adapt Binary Search to non-uniformly sorted data?
- Does the candidate accurately construct compound conditions without off-by-one errors?

### 3. Dry Run
**Input:** `nums = [4, 5, 6, 7, 0, 1, 2]`, `target = 0`
- `low = 0 (4)`, `high = 6 (2)`
- $mid = 0 + (6 - 0) / 2 = 3 \implies \text{nums}[3] = 7$.
- Compare `nums[low] (4)` vs `nums[mid] (7)`: $4 \le 7 \implies$ **Left half is sorted!**
- Does target (0) lie in `[4, 7)`? No! Therefore, target must be in the right half.
- Set `low = mid + 1 = 4`.
- Next iteration: `low = 4 (0)`, `high = 6 (2)`.
- $mid = 4 + (6 - 4) / 2 = 5 \implies \text{nums}[5] = 1$.
- `nums[mid]` is 1. Target is 0.
- Compare `nums[low] (0)` vs `nums[mid] (1)`: Left half is sorted.
- Does target (0) lie in `[0, 1)`? **Yes!**
- Set `high = mid - 1 = 4`.
- Next iteration: `low = 4`, `high = 4`.
- $mid = 4 \implies \text{nums}[4] = 0 == \text{target}$. Found at index 4!

### 4. C# Solution (.NET 8)

```csharp
using System;

public static class RotatedArraySearcher
{
    /// <summary>
    /// Searches for a target in a rotated sorted array in O(log N) time.
    /// Scenario: Querying time-series financial transactions rotated across ring buffer shards.
    /// </summary>
    public static int Search(ReadOnlySpan<int> nums, int target)
    {
        int low = 0;
        int high = nums.Length - 1;

        while (low <= high)
        {
            int mid = low + (high - low) / 2;
            if (nums[mid] == target) return mid;

            // Check if left half is normally sorted
            if (nums[low] <= nums[mid])
            {
                // Is target within the sorted left boundaries?
                if (target >= nums[low] && target < nums[mid])
                {
                    high = mid - 1;
                }
                else
                {
                    low = mid + 1;
                }
            }
            else // Right half must be normally sorted
            {
                // Is target within the sorted right boundaries?
                if (target > nums[mid] && target <= nums[high])
                {
                    low = mid + 1;
                }
                else
                {
                    high = mid - 1;
                }
            }
        }

        return -1;
    }
}
```

### 5. Complexity Analysis
- **Time Complexity:** $O(\log N)$ — Every step discards half the search space.
- **Space Complexity:** $O(1)$ — Zero heap allocation.

---

## 6.4 Problem 17: Find First and Last Position in Sorted Array (Lower & Upper Bounds)

### 1. How to Think
Given a sorted array with possible duplicate values, find the starting and ending index of a given target value. If target is not found, return `[-1, -1]`.
- **Example:** `nums = [5, 7, 7, 8, 8, 10]`, `target = 8` $\implies [3, 4]$.
- **Pattern:** Two separate Binary Searches:
  1. `FindFirst`: When `nums[mid] == target`, don't stop! Continue searching to the left (`high = mid - 1`) to see if an earlier occurrence exists.
  2. `FindLast`: When `nums[mid] == target`, continue searching to the right (`low = mid + 1`) to see if a later occurrence exists.

### 2. C# Solution (.NET 8)

```csharp
using System;

public static class RangeFinder
{
    /// <summary>
    /// Finds starting and ending index of a target in a sorted span in O(log N).
    /// </summary>
    public static int[] SearchRange(ReadOnlySpan<int> nums, int target)
    {
        int first = FindBound(nums, target, isFirst: true);
        if (first == -1) return [-1, -1];

        int last = FindBound(nums, target, isFirst: false);
        return [first, last];
    }

    private static int FindBound(ReadOnlySpan<int> nums, int target, bool isFirst)
    {
        int low = 0;
        int high = nums.Length - 1;
        int result = -1;

        while (low <= high)
        {
            int mid = low + (high - low) / 2;

            if (nums[mid] == target)
            {
                result = mid; // Record potential answer
                if (isFirst)
                {
                    high = mid - 1; // Keep searching left
                }
                else
                {
                    low = mid + 1;  // Keep searching right
                }
            }
            else if (nums[mid] < target)
            {
                low = mid + 1;
            }
            else
            {
                high = mid - 1;
            }
        }

        return result;
    }
}
```

---

# 7. SORTING ALGORITHMS: FROM FIRST PRINCIPLES TO .NET INTROSORT

## 7.1 QuickSort & MergeSort Internals

| Algorithm | Strategy | Best Time | Average Time | Worst Time | Space | Stable? | Cache Friendly? |
|---|---|---|---|---|---|---|---|
| **QuickSort** | Divide & Conquer (Pivot Partition) | $O(N \log N)$ | $O(N \log N)$ | $O(N^2)$ (bad pivot) | $O(\log N)$ stack | No | **Excellent** (in-place) |
| **MergeSort** | Divide & Conquer (Sub-list Merging) | $O(N \log N)$ | $O(N \log N)$ | $O(N \log N)$ | $O(N)$ auxiliary | **Yes** | Poor (memory copying) |
| **IntroSort** | Hybrid (QuickSort + HeapSort + Insertion) | $O(N \log N)$ | $O(N \log N)$ | $O(N \log N)$ | $O(\log N)$ stack | No | **Maximum** |

### Why Naive QuickSort Can Degrade to $O(N^2)$
In naive QuickSort, if the pivot selected is always the smallest or largest element (e.g., already sorted array with first element as pivot), the partitioning creates unbalanced subarrays of size $1$ and $N - 1$. The recursion depth becomes $N$, yielding $O(N^2)$ runtime and threatening a `StackOverflowException`.

---

## 7.2 Under the Hood: .NET `Array.Sort` and IntroSort

If you examine the .NET 8 Runtime source code (`System.ArraySortHelper<T>`), calling `Array.Sort(arr)` does not use pure QuickSort. It uses **Introspective Sort (IntroSort)**:

```
                          ┌────────────────────────┐
                          │   Array.Sort() Call    │
                          └───────────┬────────────┘
                                      │
                         Is Partition Size <= 16 ?
                                ┌─────┴─────┐
                             Yes│           │No
                                ▼           ▼
                       [InsertionSort]   [QuickSort with Median-of-3 Pivot]
                                            │
                                  Did Recursion Depth
                                  Exceed 2 * Log2(N)?
                                       ┌────┴────┐
                                    Yes│         │No
                                       ▼         ▼
                                  [HeapSort]  Continue QuickSort
```

### The Three Phases of .NET IntroSort:
1. **QuickSort with Median-of-3 Pivot:** Selects the median of the first, middle, and last elements as the pivot. This eliminates the $O(N^2)$ worst-case on already-sorted or reverse-sorted inputs.
2. **HeapSort Fallback Guard:** If the recursion depth exceeds $2 \times \lfloor\log_2 N\rfloor$, IntroSort detects that QuickSort is degrading and immediately switches to **HeapSort**. HeapSort guarantees an upper bound of $O(N \log N)$ time and $O(1)$ space.
3. **InsertionSort for Small Partitions:** When a sub-partition shrinks to $\le 16$ elements, IntroSort switches to **InsertionSort**. On tiny arrays, InsertionSort outperforms QuickSort because it has zero recursion overhead and exploits CPU L1 cache locality with minimal branch mispredictions.

> **Interview Answer:** *"Why doesn't .NET use MergeSort for `Array.Sort`?"*  
> **Answer:** MergeSort requires $O(N)$ extra memory allocation to merge arrays. IntroSort is an in-place sort requiring $O(\log N)$ stack space with superior cache locality. For reference types where stability is required, LINQ's `.OrderBy()` uses an adaptive stable sort.

---

# 8. STACK & QUEUE: LIFO, FIFO, AND EXPRESSION PARSING

## 8.1 Memory Models: Array-Backed vs. Node-Backed Queues

In .NET:
- `Stack<T>` is backed internally by a contiguous `T[] _array`. When it grows, it doubles its capacity via `Array.Resize()`.
- `Queue<T>` is backed by a circular `T[] _array` with `_head` and `_tail` indices. It does **not** allocate individual node objects.
- `LinkedList<T>` is a doubly-linked node list (`LinkedListNode<T>`), allocating 24 bytes of object header overhead per node.

---

## 8.2 Problem 18: Valid Parentheses (Syntax Validation for Config Tokens)

### 1. How to Think
Given a string containing `'('`, `')'`, `'{'`, `'}'`, `'['`, `']'`, determine if the input string is valid.
- Open brackets must be closed by the same type of brackets in the correct order.
- **Mental Model:** A LIFO `Stack<char>`. When you see an open bracket, push the expected closing bracket. When you see a closing bracket, pop the stack and check for a match.

### 2. C# Solution (.NET 8)

```csharp
using System;
using System.Collections.Generic;

public static class SyntaxValidator
{
    /// <summary>
    /// Validates balanced delimiters using an array-backed stack.
    /// Scenario: Parsing nested enterprise JSON configuration blocks.
    /// </summary>
    public static bool IsValid(ReadOnlySpan<char> s)
    {
        // Odd length can never be balanced
        if ((s.Length & 1) != 0) return false;

        // Use stackalloc for small strings to avoid GC heap allocation
        Span<char> stack = stackalloc char[s.Length];
        int top = -1;

        foreach (char c in s)
        {
            switch (c)
            {
                case '(':
                    stack[++top] = ')';
                    break;
                case '{':
                    stack[++top] = '}';
                    break;
                case '[':
                    stack[++top] = ']';
                    break;
                case ')':
                case '}':
                case ']':
                    if (top == -1 || stack[top--] != c)
                        return false;
                    break;
            }
        }

        return top == -1; // Stack must be completely empty
    }
}
```

### 3. Complexity Analysis
- **Time Complexity:** $O(N)$ — Single pass over characters.
- **Space Complexity:** $O(1)$ heap allocation (runs completely on thread stack for strings up to reasonable size).

---

## 8.3 Problem 19: Min Stack (O(1) Retrieval via Companion State)

### 1. How to Think
Design a stack that supports `Push`, `Pop`, `Top`, and retrieving the minimum element `GetMin` in **$O(1)$ time**.
- **Insight:** Pair each pushed value with the running minimum at that point in time: `(Value, CurrentMin)`.

### 2. C# Solution (.NET 8)

```csharp
using System;
using System.Collections.Generic;

public sealed class MinStack
{
    private readonly Stack<(int Value, int MinAtLevel)> _stack = new();

    public void Push(int val)
    {
        int currentMin = _stack.Count == 0 ? val : Math.Min(val, _stack.Peek().MinAtLevel);
        _stack.Push((val, currentMin));
    }

    public void Pop()
    {
        _stack.Pop();
    }

    public int Top()
    {
        return _stack.Peek().Value;
    }

    public int GetMin()
    {
        return _stack.Peek().MinAtLevel;
    }
}
```

---

## 8.4 Problem 20: Evaluate Reverse Polish Notation (Financial Math Engine)

### 1. How to Think
Evaluate the value of an arithmetic expression in Reverse Polish Notation (Postfix). Valid operators: `+`, `-`, `*`, `/`.
- **Example:** `["2", "1", "+", "3", "*"]` $\implies ((2 + 1) * 3) = 9$.
- **Algorithm:**
  - If token is a number: push onto stack.
  - If token is an operator: pop two values $B$ and $A$ (order matters! first popped is right operand), apply operation $A \text{ op } B$, push result back.

### 2. C# Solution (.NET 8)

```csharp
using System;
using System.Collections.Generic;

public static class RpnEvaluator
{
    /// <summary>
    /// Evaluates postfix arithmetic notation.
    /// Scenario: Order pricing formula evaluation in an enterprise rules engine.
    /// </summary>
    public static int EvalRPN(string[] tokens)
    {
        var stack = new Stack<int>(capacity: tokens.Length);

        foreach (string token in tokens)
        {
            if (token is "+" or "-" or "*" or "/")
            {
                int b = stack.Pop();
                int a = stack.Pop();

                int result = token switch
                {
                    "+" => a + b,
                    "-" => a - b,
                    "*" => a * b,
                    "/" => a / b, // Integer division truncates toward zero
                    _ => throw new InvalidOperationException()
                };

                stack.Push(result);
            }
            else
            {
                stack.Push(int.Parse(token));
            }
        }

        return stack.Pop();
    }
}
```

---

## 8.5 Problem 21: Implement Queue Using Stacks (Amortized Cost Transfer)

### 1. How to Think
Implement a FIFO queue using two LIFO stacks: `_inStack` (for enqueue) and `_outStack` (for dequeue).
- `Enqueue(x)`: Simply push to `_inStack` $\implies O(1)$.
- `Dequeue()`:
  - If `_outStack` is not empty: pop from `_outStack` $\implies O(1)$.
  - If `_outStack` is empty: pop all elements from `_inStack` and push them into `_outStack`. This reverses their order to FIFO! Pop from `_outStack`.
- **Amortized Analysis:** Each element is pushed to `_inStack` once, popped from `_inStack` once, pushed to `_outStack` once, and popped from `_outStack` once. Total 4 operations per element $\implies$ **Amortized $O(1)$ per operation!**

### 2. C# Solution (.NET 8)

```csharp
using System.Collections.Generic;

public sealed class MyQueue<T>
{
    private readonly Stack<T> _inStack = new();
    private readonly Stack<T> _outStack = new();

    public void Enqueue(T x) => _inStack.Push(x);

    public T Dequeue()
    {
        MoveIfNeeded();
        return _outStack.Pop();
    }

    public T Peek()
    {
        MoveIfNeeded();
        return _outStack.Peek();
    }

    public bool IsEmpty => _inStack.Count == 0 && _outStack.Count == 0;

    private void MoveIfNeeded()
    {
        if (_outStack.Count == 0)
        {
            while (_inStack.Count > 0)
            {
                _outStack.Push(_inStack.Pop());
            }
        }
    }
}
```

---

## 8.6 Problem 22: Moving Average from Data Stream (Circular Queue Buffer)

### 1. How to Think
Compute the moving average of the last `size` numbers in a stream.
- Using a circular array of size $K$ avoids copying or shifting elements.

```csharp
public sealed class MovingAverage
{
    private readonly int[] _buffer;
    private int _head = 0;
    private int _count = 0;
    private double _sum = 0;

    public MovingAverage(int size)
    {
        _buffer = new int[size];
    }

    public double Next(int val)
    {
        if (_count < _buffer.Length)
        {
            _count++;
        }
        else
        {
            _sum -= _buffer[_head]; // Subtract outgoing element
        }

        _sum += val;
        _buffer[_head] = val;
        _head = (_head + 1) % _buffer.Length;

        return _sum / _count;
    }
}
```

---

# 9. LINKED LISTS: POINTER MANIPULATION & CYCLE DETECTION

## 9.1 Linked List Memory Architecture & Cache Miss Penalty

Each `ListNode` in C# is a heap reference:
```
Heap Node Layout (32 bytes per node on 64-bit):
[SyncBlockIndex: 8B] [MethodTable: 8B] [Val: 4B + 4B pad] [Next Pointer: 8B]
```
Traversing a linked list forces the CPU memory controller to perform non-sequential memory pointer dereferences, causing L1/L2 cache misses. However, linked lists excel at $O(1)$ insertion and deletion once a pointer is known.

---

## 9.2 Problem 23: Reverse Linked List (Iterative Pointer Rewiring)

### 1. How to Think
Reverse a singly linked list in-place.
- Given: `1 -> 2 -> 3 -> null`
- Result: `3 -> 2 -> 1 -> null`
- **Mental Model (Three Pointers):**
  - `prev` (starts at null)
  - `curr` (starts at head)
  - `next` (temporary holder for `curr.next`)

```
Iterative Step:
1. Save next:        next = curr.next
2. Reverse pointer:  curr.next = prev
3. Advance prev:     prev = curr
4. Advance curr:     curr = next
```

### 2. C# Solution (.NET 8)

```csharp
public sealed class ListNode
{
    public int Val { get; set; }
    public ListNode? Next { get; set; }
    public ListNode(int val = 0, ListNode? next = null)
    {
        Val = val;
        Next = next;
    }
}

public static class LinkedListReverser
{
    /// <summary>
    /// Reverses a singly linked list in-place in O(N) time and O(1) space.
    /// Scenario: Inverting sequence order of execution nodes in a workflow chain.
    /// </summary>
    public static ListNode? ReverseList(ListNode? head)
    {
        ListNode? prev = null;
        ListNode? curr = head;

        while (curr != null)
        {
            ListNode? next = curr.Next; // Stash next node
            curr.Next = prev;           // Invert pointer backward
            prev = curr;                // Advance prev
            curr = next;                // Advance curr
        }

        return prev; // New head of reversed list
    }
}
```

### 3. Complexity Analysis
- **Time Complexity:** $O(N)$ — Single traversal.
- **Space Complexity:** $O(1)$ — Pointer reassignment only.

---

## 9.3 Problem 24: Detect Cycle in Linked List (Floyd's Tortoise and Hare)

### 1. How to Think
Determine if a linked list has a cycle in it without modifying the nodes.
- **Floyd's Algorithm:** `slow` moves 1 step; `fast` moves 2 steps. If there is a cycle, `fast` enters the loop first and `slow` follows. With each step, the gap between them closes by 1 node until collision.

### 2. C# Solution (.NET 8)

```csharp
public static class CycleDetector
{
    public static bool HasCycle(ListNode? head)
    {
        if (head?.Next == null) return false;

        ListNode? slow = head;
        ListNode? fast = head;

        while (fast?.Next != null)
        {
            slow = slow!.Next;
            fast = fast.Next.Next;

            if (ReferenceEquals(slow, fast))
            {
                return true;
            }
        }

        return false;
    }
}
```

---

## 9.4 Problem 25: Find Middle Node of Linked List (Fast-Slow Pointers)

### 1. How to Think
Return the middle node of a linked list in a single pass.
- When `fast` reaches the end (moves at speed $2v$), `slow` (speed $v$) will be at exactly half the distance ($N/2$).

```csharp
public static ListNode? MiddleNode(ListNode? head)
{
    ListNode? slow = head;
    ListNode? fast = head;

    while (fast?.Next != null)
    {
        slow = slow!.Next;
        fast = fast.Next.Next;
    }

    return slow;
}
```

---

## 9.5 Problem 26: Merge Two Sorted Lists (Splice Nodes with Sentinel Head)

### 1. How to Think
Merge two sorted linked lists into one sorted list by splicing together the nodes.
- **Architectural Tip:** Always use a **Sentinel (Dummy) Node** as the anchor. It eliminates null-checking logic for the head of the list!

### 2. C# Solution (.NET 8)

```csharp
public static class ListMerger
{
    public static ListNode? MergeTwoLists(ListNode? list1, ListNode? list2)
    {
        ListNode dummy = new ListNode(-1);
        ListNode tail = dummy;

        while (list1 != null && list2 != null)
        {
            if (list1.Val <= list2.Val)
            {
                tail.Next = list1;
                list1 = list1.Next;
            }
            else
            {
                tail.Next = list2;
                list2 = list2.Next;
            }
            tail = tail.Next;
        }

        // Attach remaining chain
        tail.Next = list1 ?? list2;

        return dummy.Next;
    }
}
```

---

# 10. TREES: HIERARCHICAL STRUCTURES, TRAVERSAL & BINARY SEARCH TREES

## 10.1 Tree Representations & Stack Frame Traversal Mechanics

A Binary Tree is defined recursively:
```csharp
public sealed class TreeNode
{
    public int Val { get; set; }
    public TreeNode? Left { get; set; }
    public TreeNode? Right { get; set; }
    public TreeNode(int val = 0, TreeNode? left = null, TreeNode? right = null)
    {
        Val = val;
        Left = left;
        Right = right;
    }
}
```

### The Three Depth-First Traversal Orders:
- **Preorder ($Root \rightarrow Left \rightarrow Right$):** Used for copying/serializing a tree structure.
- **Inorder ($Left \rightarrow Root \rightarrow Right$):** In a Binary Search Tree (BST), visiting inorder yields all elements in **strictly sorted ascending order**.
- **Postorder ($Left \rightarrow Right \rightarrow Root$):** Used for bottom-up computation (e.g., directory size calculation, freeing resources).

---

## 10.2 Problem 27: Maximum Depth of Binary Tree (DFS vs. BFS)

### 1. C# Solution (.NET 8)

```csharp
using System;

public static class TreeDepthCalculator
{
    /// <summary>
    /// Computes maximum depth via DFS recursion.
    /// Time Complexity: O(N), Space Complexity: O(H) call stack where H is tree height.
    /// </summary>
    public static int MaxDepth(TreeNode? root)
    {
        if (root == null) return 0;

        int leftDepth = MaxDepth(root.Left);
        int rightDepth = MaxDepth(root.Right);

        return 1 + Math.Max(leftDepth, rightDepth);
    }
}
```

---

## 10.3 Problem 28: Binary Tree Level Order Traversal (Batch Queue Sizing)

### 1. How to Think
Return the level order traversal of its nodes' values (i.e., from left to right, level by level).
- **Pattern:** Breadth-First Search (BFS) using a FIFO `Queue<TreeNode>`.
- **Key Trick:** At the start of each level loop, capture `int levelSize = queue.Count`. Process exactly `levelSize` nodes to guarantee separating levels!

### 2. C# Solution (.NET 8)

```csharp
using System.Collections.Generic;

public static class LevelOrderTraverser
{
    /// <summary>
    /// Performs level-by-level BFS traversal.
    /// Scenario: Generating organizational chart tiers for an enterprise HR portal.
    /// </summary>
    public static IList<IList<int>> LevelOrder(TreeNode? root)
    {
        if (root == null) return [];

        var result = new List<IList<int>>();
        var queue = new Queue<TreeNode>();
        queue.Enqueue(root);

        while (queue.Count > 0)
        {
            int levelSize = queue.Count;
            var currentLevel = new List<int>(capacity: levelSize);

            for (int i = 0; i < levelSize; i++)
            {
                TreeNode node = queue.Dequeue();
                currentLevel.Add(node.Val);

                if (node.Left != null) queue.Enqueue(node.Left);
                if (node.Right != null) queue.Enqueue(node.Right);
            }

            result.Add(currentLevel);
        }

        return result;
    }
}
```

---

## 10.4 Problem 29: Validate Binary Search Tree (Range Constraint Propagation)

### 1. How to Think
Given the root of a binary tree, determine if it is a valid Binary Search Tree (BST).
- **Common Junior Mistake:** Checking only `node.Left.Val < node.Val` and `node.Right.Val > node.Val`. This fails if a node in the right subtree contains a value smaller than an ancestor!
- **Correct Mental Model:** Every node must satisfy an allowable range `(min, max)` propagated from its ancestors:
  - When going left: update upper bound $\implies (\text{min}, \text{node.Val})$.
  - When going right: update lower bound $\implies (\text{node.Val}, \text{max})$.

### 2. C# Solution (.NET 8)

```csharp
public static class BstValidator
{
    public static bool IsValidBST(TreeNode? root)
    {
        return Validate(root, long.MinValue, long.MaxValue);
    }

    private static bool Validate(TreeNode? node, long min, long max)
    {
        if (node == null) return true;

        if (node.Val <= min || node.Val >= max)
        {
            return false;
        }

        return Validate(node.Left, min, node.Val) &&
               Validate(node.Right, node.Val, max);
    }
}
```

---

## 10.5 Problem 30: Lowest Common Ancestor (Organizational Hierarchy Resolution)

### 1. How to Think
Find the lowest node in $T$ that has both $P$ and $Q$ as descendants.
- **DFS Postorder Logic:**
  - If current node is null, return null.
  - If current node is $P$ or $Q$, return current node.
  - Recurse left and right:
    - If both left and right return non-null, **current node is the LCA!**
    - If only one side returns non-null, pass that result up.

### 2. C# Solution (.NET 8)

```csharp
public static class LcaResolver
{
    /// <summary>
    /// Finds Lowest Common Ancestor in a binary tree.
    /// Scenario: Finding the nearest common manager in an enterprise approval chain.
    /// </summary>
    public static TreeNode? LowestCommonAncestor(TreeNode? root, TreeNode p, TreeNode q)
    {
        if (root == null || ReferenceEquals(root, p) || ReferenceEquals(root, q))
        {
            return root;
        }

        TreeNode? left = LowestCommonAncestor(root.Left, p, q);
        TreeNode? right = LowestCommonAncestor(root.Right, p, q);

        if (left != null && right != null)
        {
            return root; // p is in one subtree, q is in the other
        }

        return left ?? right;
    }
}
```

---


# 11. GRAPHS: MODELING COMPLEX ENTERPRISE NETWORKS

## 11.1 Adjacency List vs. Adjacency Matrix Trade-offs

In enterprise engineering, graphs model microservice dependency trees, banking transaction flow networks, role permission hierarchies, and build pipelines.

| Property | Adjacency Matrix (`bool[V, V]`) | Adjacency List (`Dictionary<int, List<int>>`) |
|---|---|---|
| **Space Complexity** | $O(V^2)$ — Heavy memory footprint | $O(V + E)$ — Optimal for sparse graphs |
| **Edge Lookup ($U \rightarrow V$)** | $O(1)$ direct array index | $O(\text{degree}(U))$ scan of neighbor list |
| **Iterate All Neighbors** | $O(V)$ scan through entire row | $O(\text{degree}(U))$ — exactly the adjacent nodes |
| **Enterprise Fit** | Dense graphs (e.g., all-to-all connectivity) | Sparse graphs (99% of real-world networks) |

---

## 11.2 Problem 31: Number of Islands (Grid DFS/BFS Connected Components)

### 1. How to Think
Given an $M \times N$ 2D binary grid `grid` which represents a map of `'1'`s (land) and `'0'`s (water), return the number of islands. An island is surrounded by water and is formed by connecting adjacent lands horizontally or vertically.
- **Mental Model:** Each island is a connected component in an undirected graph.
- When we find a `'1'`, increment island count and trigger a traversal (DFS or BFS) to "sink" the entire island (turn connected `'1'`s into `'0'`s) so we don't count it again!

### 2. How Interviewer Thinks
- Tests 2D matrix navigation and boundary checking (`row >= 0 && row < M && col >= 0 && col < N`).
- Tests recursion call stack awareness or BFS queue management.

### 3. C# Solution (.NET 8)

```csharp
using System;

public static class IslandCounter
{
    private static readonly (int dr, int dc)[] Directions = [(-1, 0), (1, 0), (0, -1), (0, 1)];

    /// <summary>
    /// Computes number of connected land components.
    /// Scenario: Identifying isolated server clusters in a distributed cloud topology.
    /// </summary>
    public static int NumIslands(char[][] grid)
    {
        if (grid == null || grid.Length == 0) return 0;

        int rows = grid.Length;
        int cols = grid[0].Length;
        int islandCount = 0;

        for (int r = 0; r < rows; r++)
        {
            for (int c = 0; c < cols; c++)
            {
                if (grid[r][c] == '1')
                {
                    islandCount++;
                    SinkIslandDfs(grid, r, c, rows, cols);
                }
            }
        }

        return islandCount;
    }

    private static void SinkIslandDfs(char[][] grid, int r, int c, int rows, int cols)
    {
        // Guard boundary and water condition
        if (r < 0 || r >= rows || c < 0 || c >= cols || grid[r][c] != '1')
        {
            return;
        }

        // Sink land node in-place to avoid auxiliary visited set
        grid[r][c] = '0';

        foreach (var (dr, dc) in Directions)
        {
            SinkIslandDfs(grid, r + dr, c + dc, rows, cols);
        }
    }
}
```

### 4. Complexity Analysis
- **Time Complexity:** $O(M \times N)$ — Every cell is visited at most twice.
- **Space Complexity:** $O(M \times N)$ worst-case call stack if the entire grid is land.

---

## 11.3 Problem 32: Course Schedule (Cycle Detection & Topological Sort via Kahn's Algorithm)

### 1. How to Think
There are `numCourses` courses labeled `0` to `numCourses - 1`. You are given an array `prerequisites` where `prerequisites[i] = [a, b]` means you must take course `b` before course `a`. Can you finish all courses?
- **Mental Model:** This is a **Directed Graph**.
  - Edge: $B \rightarrow A$ (B is prerequisite for A).
  - Can you complete all courses $\iff$ **Is the graph a Directed Acyclic Graph (DAG)?**
  - If there is a cycle (e.g., $A \rightarrow B \rightarrow C \rightarrow A$), you can never finish!
- **Algorithm (Kahn's BFS In-Degree Algorithm):**
  1. Compute `inDegree[node]`: count of incoming prerequisites for each node.
  2. Enqueue all nodes with `inDegree == 0` (courses with no prerequisites).
  3. While queue is not empty:
     - Dequeue course, increment `processedCount`.
     - For each dependent course, decrement its `inDegree`.
     - If a dependent course's `inDegree` reaches 0, enqueue it.
  4. If `processedCount == numCourses`, return `true`; else a cycle exists (`false`).

### 2. C# Solution (.NET 8)

```csharp
using System;
using System.Collections.Generic;

public static class WorkflowScheduler
{
    /// <summary>
    /// Validates if dependency graph is acyclic using Kahn's Topological Sort.
    /// Scenario: Validating circular dependencies in an ASP.NET Core service registration pipeline.
    /// </summary>
    public static bool CanFinish(int numCourses, int[][] prerequisites)
    {
        var graph = new List<int>[numCourses];
        for (int i = 0; i < numCourses; i++) graph[i] = new List<int>();

        int[] inDegree = new int[numCourses];

        // Build adjacency list and compute in-degrees
        foreach (var edge in prerequisites)
        {
            int course = edge[0];
            int prerequisite = edge[1];
            graph[prerequisite].Add(course);
            inDegree[course]++;
        }

        // Queue all nodes with 0 prerequisites
        var queue = new Queue<int>();
        for (int i = 0; i < numCourses; i++)
        {
            if (inDegree[i] == 0) queue.Enqueue(i);
        }

        int completedCourses = 0;

        while (queue.Count > 0)
        {
            int current = queue.Dequeue();
            completedCourses++;

            foreach (int neighbor in graph[current])
            {
                inDegree[neighbor]--;
                if (inDegree[neighbor] == 0)
                {
                    queue.Enqueue(neighbor);
                }
            }
        }

        return completedCourses == numCourses;
    }
}
```

### 3. Complexity Analysis
- **Time Complexity:** $O(V + E)$ where $V$ is courses and $E$ is prerequisites.
- **Space Complexity:** $O(V + E)$ for graph representation and in-degree table.

---

# 12. HEAP & PRIORITYQUEUE (.NET 6+)

## 12.1 Binary Heap Mechanics on Contiguous Arrays

Prior to .NET 6, C# developers had to build custom binary heaps or misuse `SortedSet<T>`. Starting with .NET 6, Microsoft added `System.Collections.Generic.PriorityQueue<TElement, TPriority>`.

### Memory Architecture of Array-Backed Binary Min-Heap:
In a 0-indexed array:
- Parent node index: $\lfloor(i - 1) / 2\rfloor$
- Left child index: $2i + 1$
- Right child index: $2i + 2$
- **Min-Heap Property:** Parent value is always $\le$ children values. Root (`arr[0]`) is always the absolute minimum.
- Insertion (`Enqueue`): Appended to end, "sifted up" in $O(\log N)$.
- Extraction (`Dequeue`): Root replaced with last element, "sifted down" in $O(\log N)$.

---

## 12.2 Problem 33: Kth Largest Element in an Array (Min-Heap of Size K)

### 1. How to Think
Find the $K$-th largest element in an unsorted array.
- **Insight:** Maintain a **Min-Heap of fixed size $K$**:
  - The min-heap stores the $K$ largest elements seen so far.
  - The root of this min-heap is the smallest among those $K$ largest $\implies$ **the $K$-th largest overall!**
  - For each element in the array:
    - Add to heap.
    - If heap count exceeds $K$, dequeue the minimum.
  - At the end, peek the root.

### 2. C# Solution (.NET 8)

```csharp
using System;
using System.Collections.Generic;

public static class KthLargestFinder
{
    /// <summary>
    /// Finds Kth largest element using .NET PriorityQueue in O(N log K) time and O(K) space.
    /// Scenario: Identifying top 99th percentile slowest database queries in an APM tool.
    /// </summary>
    public static int FindKthLargest(int[] nums, int k)
    {
        ArgumentNullException.ThrowIfNull(nums);

        // PriorityQueue in .NET is a Min-Heap by default
        var minHeap = new PriorityQueue<int, int>(capacity: k + 1);

        foreach (int num in nums)
        {
            minHeap.Enqueue(num, num); // Value is its own priority

            if (minHeap.Count > k)
            {
                minHeap.Dequeue(); // Drop the smallest element
            }
        }

        return minHeap.Peek();
    }
}
```

### 3. Complexity Analysis
- **Time Complexity:** $O(N \log K)$ — Much faster than sorting $O(N \log N)$ when $K \ll N$.
- **Space Complexity:** $O(K)$ — Heap never exceeds $K + 1$ elements.

---

## 12.3 Problem 34: Merge K Sorted Lists / Telemetry Streams (K-Way Min-Heap Merge)

### 1. How to Think
You are given an array of $K$ linked-lists, each sorted in ascending order. Merge all the linked-lists into one sorted linked-list.
- **Mental Model:** At any point, the next smallest element in the final merged list must be the minimum among the current heads of the $K$ lists.
- A Min-Heap holding at most $K$ nodes gives $O(\log K)$ retrieval of the smallest node.

### 2. C# Solution (.NET 8)

```csharp
using System.Collections.Generic;

public static class KWayMerger
{
    public static ListNode? MergeKLists(ListNode?[] lists)
    {
        if (lists == null || lists.Length == 0) return null;

        var minHeap = new PriorityQueue<ListNode, int>();

        // Enqueue head of each list
        foreach (var head in lists)
        {
            if (head != null)
            {
                minHeap.Enqueue(head, head.Val);
            }
        }

        ListNode dummy = new ListNode(-1);
        ListNode tail = dummy;

        while (minHeap.Count > 0)
        {
            ListNode smallestNode = minHeap.Dequeue();
            tail.Next = smallestNode;
            tail = tail.Next;

            if (smallestNode.Next != null)
            {
                minHeap.Enqueue(smallestNode.Next, smallestNode.Next.Val);
            }
        }

        return dummy.Next;
    }
}
```

### 3. Complexity Analysis
- **Time Complexity:** $O(N \log K)$ where $N$ is total number of nodes across all lists and $K$ is number of lists.
- **Space Complexity:** $O(K)$ for the priority queue.

---

# 13. RECURSION & BACKTRACKING: STATE SPACE EXPLORATION

## 13.1 Backtracking Template: Choose -> Explore -> Unchoose

Backtracking is exhaustive depth-first search on a decision tree. The standard template:

```csharp
void Backtrack(State state, List<Result> results)
{
    if (IsBaseGoalReached(state))
    {
        results.Add(state.Clone());
        return;
    }

    foreach (var choice in AvailableChoices(state))
    {
        Apply(choice, state);       // 1. CHOOSE
        Backtrack(state, results);  // 2. EXPLORE
        Revert(choice, state);      // 3. UNCHOOSE (Backtrack)
    }
}
```

---

## 13.2 Problem 35: Subsets (Combinatorial Role-Based Access Matrix)

### 1. How to Think
Given an integer array `nums` of unique elements, return all possible subsets (the power set).
- At each index $i$, we make a binary choice: either include `nums[i]` or exclude it. Total subsets $= 2^N$.

### 2. C# Solution (.NET 8)

```csharp
using System.Collections.Generic;

public static class PowerSetGenerator
{
    public static IList<IList<int>> Subsets(int[] nums)
    {
        var result = new List<IList<int>>();
        var currentCombination = new List<int>();

        GenerateSubsets(0, nums, currentCombination, result);
        return result;
    }

    private static void GenerateSubsets(int start, int[] nums, List<int> current, List<IList<int>> result)
    {
        // Snapshot current state
        result.Add(new List<int>(current));

        for (int i = start; i < nums.Length; i++)
        {
            current.Add(nums[i]);                     // 1. Choose
            GenerateSubsets(i + 1, nums, current, result); // 2. Explore
            current.RemoveAt(current.Count - 1);       // 3. Unchoose
        }
    }
}
```

---

## 13.3 Problem 36: Permutations (Distinct Sequencing Generation)

### 1. How to Think
Generate all $N!$ orderings of an array of distinct integers.
- Use a boolean array `used[i]` or swap in-place to explore all branchings.

### 2. C# Solution (.NET 8)

```csharp
using System.Collections.Generic;

public static class PermutationGenerator
{
    public static IList<IList<int>> Permute(int[] nums)
    {
        var result = new List<IList<int>>();
        var current = new List<int>(capacity: nums.Length);
        bool[] used = new bool[nums.Length];

        Dfs(nums, used, current, result);
        return result;
    }

    private static void Dfs(int[] nums, bool[] used, List<int> current, List<IList<int>> result)
    {
        if (current.Count == nums.Length)
        {
            result.Add(new List<int>(current));
            return;
        }

        for (int i = 0; i < nums.Length; i++)
        {
            if (used[i]) continue;

            used[i] = true;
            current.Add(nums[i]);

            Dfs(nums, used, current, result);

            current.RemoveAt(current.Count - 1);
            used[i] = false;
        }
    }
}
```

---

## 13.4 Problem 37: Word Search (2D Board Backtracking with Visited Tracking)

### 1. C# Solution (.NET 8)

```csharp
public static class BoardSearcher
{
    public static bool Exist(char[][] board, string word)
    {
        int rows = board.Length;
        int cols = board[0].Length;

        for (int r = 0; r < rows; r++)
        {
            for (int c = 0; c < cols; c++)
            {
                if (board[r][c] == word[0] && SearchDfs(board, word, 0, r, c))
                {
                    return true;
                }
            }
        }
        return false;
    }

    private static bool SearchDfs(char[][] board, string word, int charIdx, int r, int c)
    {
        if (charIdx == word.Length) return true;

        if (r < 0 || r >= board.Length || c < 0 || c >= board[0].Length || board[r][c] != word[charIdx])
            return false;

        char original = board[r][c];
        board[r][c] = '#'; // Mark visited

        bool found = SearchDfs(board, word, charIdx + 1, r + 1, c) ||
                     SearchDfs(board, word, charIdx + 1, r - 1, c) ||
                     SearchDfs(board, word, charIdx + 1, r, c + 1) ||
                     SearchDfs(board, word, charIdx + 1, r, c - 1);

        board[r][c] = original; // Backtrack / unchoose
        return found;
    }
}
```

---

# 14. DYNAMIC PROGRAMMING: MEMOIZATION & TABULATION

## 14.1 The DP Framework: Optimal Substructure + Overlapping Subproblems

Dynamic Programming is simply **recursion without doing duplicate work**.
- **Top-Down (Memoization):** Write recursion naturally, store results in cache (`int[] memo`).
- **Bottom-Up (Tabulation):** Build an array from base cases up to target index.
- **Space Optimization:** If state transition only depends on the previous 1 or 2 steps, discard the $O(N)$ array and keep scalar variables ($O(1)$ space).

---

## 14.2 Problem 38: Climbing Stairs (Fibonacci State Transitions)

### 1. How to Think
You are climbing a staircase. It takes $N$ steps to reach the top. Each time you can climb 1 or 2 steps. In how many distinct ways can you climb to the top?
- **Recurrence Relation:**
  $$\text{Ways}(N) = \text{Ways}(N - 1) + \text{Ways}(N - 2)$$
- Only the previous two values are required $\implies O(1)$ space.

### 2. C# Solution (.NET 8)

```csharp
public static class StairClimber
{
    public static int ClimbStairs(int n)
    {
        if (n <= 2) return n;

        int prev2 = 1; // n = 1
        int prev1 = 2; // n = 2

        for (int i = 3; i <= n; i++)
        {
            int current = prev1 + prev2;
            prev2 = prev1;
            prev1 = current;
        }

        return prev1;
    }
}
```

---

## 14.3 Problem 39: Coin Change (Unbounded Knapsack / Minimum Denominations)

### 1. How to Think
Given an integer array `coins` representing coins of different denominations and an integer `amount`, return the fewest number of coins needed to make up that amount. If impossible, return `-1`.
- **State Definition:** Let `dp[i]` be the minimum coins needed to make amount `i`.
- **Transition:**
  $$dp[i] = \min_{\text{coin} \in \text{coins}}(dp[i - \text{coin}] + 1)$$
- Base Case: `dp[0] = 0`. Initialize all other `dp[i] = amount + 1` (acting as infinity).

### 2. C# Solution (.NET 8)

```csharp
using System;

public static class CashierOptimizer
{
    /// <summary>
    /// Computes minimum coins needed to make up amount.
    /// Scenario: Optimal cash dispenser disbursement algorithm in an ATM terminal.
    /// </summary>
    public static int CoinChange(int[] coins, int amount)
    {
        if (amount < 0) return -1;
        if (amount == 0) return 0;

        int max = amount + 1;
        int[] dp = new int[amount + 1];
        Array.Fill(dp, max);
        dp[0] = 0;

        for (int i = 1; i <= amount; i++)
        {
            foreach (int coin in coins)
            {
                if (coin <= i)
                {
                    dp[i] = Math.Min(dp[i], dp[i - coin] + 1);
                }
            }
        }

        return dp[amount] > amount ? -1 : dp[amount];
    }
}
```

### 3. Complexity Analysis
- **Time Complexity:** $O(\text{amount} \times C)$ where $C$ is number of coin denominations.
- **Space Complexity:** $O(\text{amount})$ for DP array.

---

## 14.4 Problem 40: Longest Common Subsequence (Audit Diff & Reconciliation Engine)

### 1. How to Think
Given two strings `text1` and `text2`, return the length of their longest common subsequence.
- **State Definition:** `dp[i, j]` is length of LCS between `text1[0..i-1]` and `text2[0..j-1]`.
- **Transitions:**
  - If `text1[i-1] == text2[j-1]`: $dp[i, j] = 1 + dp[i-1, j-1]$
  - Else: $dp[i, j] = \max(dp[i-1, j], dp[i, j-1])$

### 2. C# Solution (.NET 8)

```csharp
using System;

public static class DiffEngine
{
    public static int LongestCommonSubsequence(string text1, string text2)
    {
        int m = text1.Length;
        int n = text2.Length;

        // Use two 1D rows instead of 2D matrix for O(N) space optimization
        int[] prev = new int[n + 1];
        int[] curr = new int[n + 1];

        for (int i = 1; i <= m; i++)
        {
            for (int j = 1; j <= n; j++)
            {
                if (text1[i - 1] == text2[j - 1])
                {
                    curr[j] = 1 + prev[j - 1];
                }
                else
                {
                    curr[j] = Math.Max(prev[j], curr[j - 1]);
                }
            }
            // Swap row buffers
            (prev, curr) = (curr, prev);
        }

        return prev[n];
    }
}
```

---

## 14.5 Problem 41: 0/1 Knapsack Problem (SaaS Resource Allocation Under Budget)

### 1. How to Think
Given weights `weights` and values `values` of $N$ items, find the maximum value that fits inside a knapsack of capacity $W$. Each item can either be taken (1) or left (0).
- **Tabulation State:** `dp[w]` stores max value achievable with capacity $w$.
- Traverse weights, and traverse capacities **backwards** from $W$ down to `weight` to avoid using the same item twice!

```csharp
using System;

public static class KnapsackSolver
{
    public static int SolveKnapsack(int[] weights, int[] values, int capacity)
    {
        int[] dp = new int[capacity + 1];

        for (int i = 0; i < weights.Length; i++)
        {
            int w = weights[i];
            int v = values[i];

            for (int cap = capacity; cap >= w; cap--)
            {
                dp[cap] = Math.Max(dp[cap], dp[cap - w] + v);
            }
        }

        return dp[capacity];
    }
}
```

---

# 15. GREEDY ALGORITHMS & INTERVALS

## 15.1 The Greedy-Choice Property

A problem exhibits the greedy choice property if a globally optimal solution can be reached by making a locally optimal choice at each step without ever needing to reconsider past choices.

---

## 15.2 Problem 42: Best Time to Buy and Sell Stock (Running Valley & Peak)

### 1. How to Think
Given an array `prices` where `prices[i]` is the price of a given stock on day $i$, maximize your profit by choosing a single day to buy and a different day in the future to sell.
- **Mental Model:** As you move forward through time, keep track of the **minimum price seen so far**. If you sold today, your profit would be `price - minPrice`. Track the maximum profit.

### 2. C# Solution (.NET 8)

```csharp
using System;

public static class StockTrader
{
    public static int MaxProfit(ReadOnlySpan<int> prices)
    {
        if (prices.Length <= 1) return 0;

        int minPrice = prices[0];
        int maxProfit = 0;

        for (int i = 1; i < prices.Length; i++)
        {
            int currentPrice = prices[i];
            if (currentPrice < minPrice)
            {
                minPrice = currentPrice;
            }
            else
            {
                int profitToday = currentPrice - minPrice;
                if (profitToday > maxProfit)
                {
                    maxProfit = profitToday;
                }
            }
        }

        return maxProfit;
    }
}
```

---

## 15.3 Problem 43: Jump Game (Boundary Horizon Expansion)

### 1. How to Think
Given an integer array `nums` where `nums[i]` is max jump length from position $i$, determine if you can reach the last index.
- **Greedy Horizon:** Maintain `maxReach`. At index $i$, if $i > \text{maxReach}$, you are stranded and can never progress further! Otherwise, expand $\text{maxReach} = \max(\text{maxReach}, i + \text{nums}[i])$.

```csharp
using System;

public static class JumpGame
{
    public static bool CanJump(ReadOnlySpan<int> nums)
    {
        int maxReach = 0;

        for (int i = 0; i < nums.Length; i++)
        {
            if (i > maxReach) return false;

            maxReach = Math.Max(maxReach, i + nums[i]);
            if (maxReach >= nums.Length - 1) return true;
        }

        return true;
    }
}
```

---

## 15.4 Problem 44: Meeting Rooms I & II (Resource Contention Peaks)

### 1. How to Think (Meeting Rooms II)
Given an array of meeting time intervals `intervals`, find the minimum number of conference rooms required.
- **Chronological Sweep-Line Insight:**
  - When a meeting starts, we need +1 room.
  - When a meeting ends, a room is freed: -1 room.
  - Separate start times and end times into two arrays and sort them both.
  - Use two pointers: if `start[i] < end[j]`, allocate a room (`rooms++`, `i++`). Otherwise, free a room (`rooms--`, `j++`).
  - Track peak rooms required!

### 2. C# Solution (.NET 8)

```csharp
using System;

public static class ConferenceScheduler
{
    public static int MinMeetingRooms(int[][] intervals)
    {
        if (intervals == null || intervals.Length == 0) return 0;

        int n = intervals.Length;
        int[] startTimes = new int[n];
        int[] endTimes = new int[n];

        for (int k = 0; k < n; k++)
        {
            startTimes[k] = intervals[k][0];
            endTimes[k] = intervals[k][1];
        }

        Array.Sort(startTimes);
        Array.Sort(endTimes);

        int roomsAllocated = 0;
        int peakRooms = 0;
        int startPtr = 0;
        int endPtr = 0;

        while (startPtr < n)
        {
            if (startTimes[startPtr] < endTimes[endPtr])
            {
                roomsAllocated++;
                peakRooms = Math.Max(peakRooms, roomsAllocated);
                startPtr++;
            }
            else
            {
                roomsAllocated--;
                endPtr++;
            }
        }

        return peakRooms;
    }
}
```

---

# 16. BIT MANIPULATION: HARDWARE-LEVEL LOGIC

## 16.1 Bitwise Operators and Bitmask Flags in C#

Bit manipulation operates directly on binary representation in CPU registers in a single clock cycle.

```
Operator Summary:
AND (&) : 1 & 1 = 1, else 0     (Masking / Clearing bits)
OR  (|) : 0 | 0 = 0, else 1     (Setting bits)
XOR (^) : Same = 0, Diff = 1    (Toggling bits, self-cancellation: A ^ A = 0, A ^ 0 = A)
NOT (~) : Flips all bits
<<      : Shift left (multiplication by 2^k)
>>      : Shift right (division by 2^k)
```

In .NET enterprise apps, bitwise flags represent user permission sets without database joins:
```csharp
[Flags]
public enum UserPermissions : uint
{
    None        = 0,
    CanRead     = 1 << 0, // 0001 (1)
    CanWrite    = 1 << 1, // 0010 (2)
    CanDelete   = 1 << 2, // 0100 (4)
    CanAudit    = 1 << 3  // 1000 (8)
}
```

---

## 16.2 Problem 45: Single Number (XOR Cancellation Property)

### 1. How to Think
Given a non-empty array of integers `nums`, every element appears twice except for one. Find that single one in $O(N)$ time and $O(1)$ space.
- **XOR Magic Properties:**
  1. $X \oplus X = 0$ (Any number XORed with itself cancels out).
  2. $X \oplus 0 = X$.
  3. XOR is associative and commutative.
- If we XOR all numbers together:
  $$(2 \oplus 2) \oplus (4 \oplus 4) \oplus 7 = 0 \oplus 0 \oplus 7 = 7!$$

### 2. C# Solution (.NET 8)

```csharp
using System;

public static class SingleNumberSolver
{
    public static int SingleNumber(ReadOnlySpan<int> nums)
    {
        int unique = 0;
        foreach (int num in nums)
        {
            unique ^= num;
        }
        return unique;
    }
}
```

---

## 16.3 Problem 46: Number of 1 Bits (Brian Kernighan's Bit Clear)

### 1. How to Think
Write a function that takes the binary representation of a positive integer and returns the number of set bits (Hamming weight).
- **Brian Kernighan's Insight:**
  Executing `n = n & (n - 1)` **always clears the lowest set bit (1) in binary**!
  The loop runs only as many times as there are set bits, not 32 times.

### 2. C# Solution (.NET 8)

```csharp
public static class BitCounter
{
    public static int HammingWeight(uint n)
    {
        int count = 0;
        while (n != 0)
        {
            n &= (n - 1); // Clears the least significant set bit
            count++;
        }
        return count;
    }

    // Modern .NET 8 Hardware Intrinsics / PopCount:
    public static int HammingWeightHardware(uint n) => System.Numerics.BitOperations.PopCount(n);
}
```

---

## 16.4 Problem 47: Power of Two (Bitmask Decrement Boundary)

### 1. How to Think
Given an integer `n`, return `true` if it is a power of two.
- A power of two in binary has **exactly one bit set** (e.g., $16 = 10000_2$, $15 = 01111_2$).
- Therefore, for any positive power of two:
  $$n \ \& \ (n - 1) == 0$$

```csharp
public static class PowerOfTwoChecker
{
    public static bool IsPowerOfTwo(int n)
    {
        return n > 0 && (n & (n - 1)) == 0;
    }
}
```

---

# 17. INFOSYS & TECHNICAL INTERVIEW SURVIVAL PLAYBOOK

## 17.1 How Infosys Evaluates a 3 YOE .NET Developer

Infosys technical interviewers for 3 YOE lateral hires evaluate candidates on a 4-quadrant rubric:

```
                          Infosys Evaluation Rubric
                                      ▲
                                      │
            [Algorithmic Rigor]       │       [C# / CLR Deep Truth]
            - Can write raw loops     │       - Knows Heap vs Stack
            - Avoids blind LINQ       │       - Explains Dictionary internals
            - Handles boundary cases  │       - Understands ref struct / Span
                                      │
         ─────────────────────────────┼─────────────────────────────►
                                      │
            [Clean Code & Idioms]     │       [Communication & Composure]
            - Meaningful naming       │       - Thinks out loud before coding
            - Guard clauses           │       - Asks clarifying constraints
            - Exception management    │       - Accepts interviewer hints
                                      │
```

### The "Deadly Sins" in an Infosys Coding Round:
1. **Writing LINQ when asked to solve in-place:** Saying `.Distinct().OrderBy()` when asked to implement duplicate removal will cost you the interview.
2. **Silent Coding:** Staring at the screen for 8 minutes without uttering a word. The interviewer cannot grade your thought process.
3. **Off-by-One Loop Bounds:** Writing `<` instead of `<=` or not checking `Length - 1`, leading to `IndexOutOfRangeException`.
4. **Failing to Trace Code:** Declaring "My code is complete" without walking through a single concrete test case.

---

## 17.2 Real 45-Minute Interview Simulation Transcript

**Interviewer:** Lead Architect, Infosys Financial Services Practice.  
**Candidate:** .NET Backend Developer (3 Years Experience).

---

**Interviewer:** *"Welcome, Rajesh. Let's start with a coding problem. Suppose we have an order processing pipeline. We receive an array of transaction integers, both positive and negative. I want you to find the contiguous subarray that produces the maximum total sum. How would you approach this?"*

**Candidate:** *"Thank you, sir. Before writing any code, I would like to clarify a few constraints. First, can the array be empty or null? Second, can all the numbers in the array be negative? Third, what is the expected maximum size of this array?"*

**Interviewer:** *"Good questions. The array will not be null or empty; it will have between 1 and $10^5$ elements. All numbers can indeed be negative. Values range between $-10^4$ and $+10^4$."*

**Candidate:** *"Understood. Let me walk through a small example. If the input is `[-2, 1, -3, 4, -1, 2, 1, -5, 4]`, the subarray `[4, -1, 2, 1]` gives a sum of 6.*  
*If I think about a brute-force approach, I could generate all possible subarrays using nested loops and compute their sums. That would take $O(N^2)$ time and $O(1)$ space. However, for $N = 10^5$, $N^2$ is $10^{10}$ operations, which will cause a Time Limit Exceeded (TLE).*  
*We can optimize this to $O(N)$ linear time using Kadane’s Algorithm."*

**Interviewer:** *"Explain the intuition behind Kadane's Algorithm."*

**Candidate:** *"Certainly. At every index $i$, we make a local decision: do we extend our previous contiguous subarray streak, or do we discard it and start fresh at the current element? If our running streak has dropped below zero, carrying it forward only penalizes any future elements. So if `currentStreak > 0`, we add `nums[i]`; otherwise, we reset `currentStreak = nums[i]`. We maintain a global variable `maxSum` to store the maximum streak seen.  
Crucially, to handle the case where all numbers are negative, like `[-5, -2, -8]`, I will initialize both `currentStreak` and `maxSum` to `nums[0]`, rather than 0. May I proceed to write the C# solution?"*

**Interviewer:** *"Please go ahead."*

*(Candidate writes the code on the screen)*
```csharp
public static int MaxSubArray(ReadOnlySpan<int> nums)
{
    int currentStreak = nums[0];
    int maxSubarraySum = nums[0];

    for (int i = 1; i < nums.Length; i++)
    {
        int val = nums[i];
        currentStreak = (currentStreak > 0) ? currentStreak + val : val;

        if (currentStreak > maxSubarraySum)
        {
            maxSubarraySum = currentStreak;
        }
    }

    return maxSubarraySum;
}
```

**Candidate:** *"Let me trace this code with `[-2, 1, -3]`.  
Init: `currentStreak = -2`, `maxSum = -2`.  
$i = 1$, `val = 1`: `currentStreak` was negative (-2), so `currentStreak = 1`. `maxSum = max(-2, 1) = 1`.  
$i = 2$, `val = -3`: `currentStreak` was 1 (>0), so `currentStreak = 1 + (-3) = -2`. `maxSum` remains 1.  
Loop terminates. Returns 1. The code executes in $O(N)$ time and $O(1)$ auxiliary stack space. Notice I used `ReadOnlySpan<int>` so the caller can pass an array slice without creating GC heap allocations."*

**Interviewer:** *"Excellent. Now a follow-up: what if I asked you to return not just the sum, but the actual start and end indices of that subarray?"*

**Candidate:** *"We can track three pointers: `start = 0`, `end = 0`, and `tempStart = 0`. Whenever we start a new streak (`currentStreak = nums[i]`), we set `tempStart = i`. Whenever `currentStreak > maxSubarraySum`, we update `maxSubarraySum = currentStreak`, set `start = tempStart`, and `end = i`. When the loop finishes, `[start, end]` will be the exact boundaries."*

**Interviewer:** *"Spot on, Rajesh. That's exactly what I wanted to hear."*

---

## 17.3 The .NET 8 Big-O Time & Space Complexity Reference Matrix

| Data Structure / Operation | Access | Search | Insertion | Deletion | Space Complexity | Underlying .NET 8 Internal Architecture |
|---|---|---|---|---|---|---|
| **Array (`T[]`)** | $O(1)$ | $O(N)$ | $O(N)$ (resize) | $O(N)$ (shift) | $O(N)$ | Contiguous block on GC Heap; CPU L1 cache line prefetching. |
| **`Span<T>` / `ReadOnlySpan<T>`** | $O(1)$ | $O(N)$ | N/A (fixed window) | N/A | $O(1)$ | Ref struct on Stack; interior pointer + 32-bit length. |
| **`List<T>`** | $O(1)$ | $O(N)$ | Amortized $O(1)$ | $O(N)$ | $O(N)$ | Backed by `T[] _items`; doubles capacity when full. |
| **`Dictionary<TKey, TValue>`** | N/A | $O(1)$ avg, $O(N)$ worst | $O(1)$ avg, $O(N)$ worst | $O(1)$ avg | $O(N)$ | Array-based separate chaining: `int[] _buckets`, `Entry[] _entries`. |
| **`HashSet<T>`** | N/A | $O(1)$ avg, $O(N)$ worst | $O(1)$ avg | $O(1)$ avg | $O(N)$ | Backed by hash buckets similar to `Dictionary`. |
| **`Stack<T>`** | N/A | $O(N)$ | $O(1)$ push | $O(1)$ pop | $O(N)$ | Backed by `T[] _array`; LIFO pointer index. |
| **`Queue<T>`** | N/A | $O(N)$ | $O(1)$ enqueue | $O(1)$ dequeue | $O(N)$ | Backed by circular `T[] _array` with head and tail index pointers. |
| **`LinkedList<T>`** | $O(N)$ | $O(N)$ | $O(1)$ (node known) | $O(1)$ (node known) | $O(N)$ | Doubly-linked nodes; high heap fragmentation; poor cache locality. |
| **`PriorityQueue<TElement, TPriority>`** | N/A | $O(N)$ | $O(\log N)$ | $O(\log N)$ | $O(N)$ | Array-backed 4-ary or binary Min-Heap; sift-up / sift-down. |
| **`SortedSet<T>` / `SortedDictionary<K, V>`** | N/A | $O(\log N)$ | $O(\log N)$ | $O(\log N)$ | $O(N)$ | Self-balancing Red-Black Tree. |
