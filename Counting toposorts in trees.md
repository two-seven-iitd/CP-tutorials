# Counting Valid Topological Sorts in a Rooted Forest / Tree

> **The Formula:**  For a rooted forest (or tree) with `n` nodes, the number of valid topological orderings is
>
> $$\frac{n!}{\displaystyle\prod_{u} s(u)}$$
>
> where $s(u)$ is the subtree size of node $u$.

---

## Table of Contents

- [1. What Are We Counting?](#1-what-are-we-counting)
- [2. Prerequisite: Interleaving Sequences](#2-prerequisite-interleaving-sequences)
  - [2.1 Two Sequences](#21-two-sequences)
  - [2.2 Three or More Sequences](#22-three-or-more-sequences)
- [3. The Formula](#3-the-formula)
- [4. Proof by Induction](#4-proof-by-induction)
  - [4.1 Base Case](#41-base-case)
  - [4.2 Inductive Step](#42-inductive-step)
  - [4.3 The Cancellation](#43-the-cancellation)
- [5. Worked Examples](#5-worked-examples)
  - [5.1 Three-Node Tree](#51-three-node-tree)
  - [5.2 Five-Node Tree](#52-five-node-tree)
  - [5.3 A Forest (Two Separate Trees)](#53-a-forest-two-separate-trees)
- [6. The Probability Intuition](#6-the-probability-intuition)
- [7. C++ Implementation](#7-c-implementation)
- [8. When This Formula Does NOT Apply](#8-when-this-formula-does-not-apply)

---

## 1. What Are We Counting?

Given a rooted tree (or forest), a **valid topological ordering** is a permutation of all nodes such that every parent appears before all of its children.

```
        r
       / \
      A   B
```

Valid orderings: `r A B` and `r B A`. That's **2**.

Invalid: `A r B` — `r` is the parent of `A` but appears after it.

> [!NOTE]
> This is the same as counting **linear extensions** of the partial order defined by the tree.

---

## 2. Prerequisite: Interleaving Sequences

This is the key building block. Make sure this is solid before reading the proof.

### 2.1 Two Sequences

You have two sequences whose internal order is fixed:

```
Sequence X:  x₁, x₂         (2 elements, must stay in this order)
Sequence Y:  y₁, y₂, y₃     (3 elements, must stay in this order)
```

**Question:** How many ways to merge them into one sequence of 5?

**Answer:** Choose which 2 of the 5 positions go to X. The rest go to Y. Both fill their positions in their fixed order.

$$\binom{5}{2} = \frac{5!}{2! \times 3!} = 10$$

A few of the 10 results:

| Positions for X | Merged sequence     |
|-----------------|---------------------|
| 1, 2            | x₁ x₂ y₁ y₂ y₃     |
| 1, 3            | x₁ y₁ x₂ y₂ y₃     |
| 1, 5            | x₁ y₁ y₂ y₃ x₂     |
| 4, 5            | y₁ y₂ y₃ x₁ x₂     |

Notice `x₂ x₁` never appears — X's internal order is always preserved.

**General formula for two sequences** of lengths $s_1$ and $s_2$:

$$\frac{(s_1 + s_2)!}{s_1! \times s_2!}$$

### 2.2 Three or More Sequences

For three sequences of lengths $s_1, s_2, s_3$, do it in two steps:

**Step 1:** Pick which $s_1$ of the $(s_1 + s_2 + s_3)$ positions go to sequence 1:

$$\frac{(s_1 + s_2 + s_3)!}{s_1! \times (s_2 + s_3)!}$$

**Step 2:** From the remaining $s_2 + s_3$ positions, pick which $s_2$ go to sequence 2 (sequence 3 gets the rest):

$$\frac{(s_2 + s_3)!}{s_2! \times s_3!}$$

**Multiply:** the $(s_2 + s_3)!$ cancels:

$$\frac{(s_1 + s_2 + s_3)!}{s_1! \times s_2! \times s_3!}$$

> [!TIP]
> This is called the **multinomial coefficient**. For $k$ sequences of lengths $s_1, \ldots, s_k$:
>
> $$\text{interleavings} = \frac{(s_1 + s_2 + \cdots + s_k)!}{s_1! \times s_2! \times \cdots \times s_k!}$$

---

## 3. The Formula

For a rooted tree (or forest) with $n$ nodes, where $s(u)$ is the subtree size of node $u$:

$$\boxed{\text{count} = \frac{n!}{\displaystyle\prod_{\text{all nodes } u} s(u)}}$$

**Quick examples:**

| Tree | Subtree sizes | Count |
|------|---------------|-------|
| Single node | $s = 1$ | $\frac{1!}{1} = 1$ |
| Chain: `A → B → C` | $3, 2, 1$ | $\frac{3!}{3 \times 2 \times 1} = 1$ |
| Star: `r → {A, B, C}` | $4, 1, 1, 1$ | $\frac{4!}{4 \times 1 \times 1 \times 1} = 6$ |
| No edges (3 nodes) | $1, 1, 1$ | $\frac{3!}{1 \times 1 \times 1} = 6$ |

The chain gives 1 (only one valid order). The star gives $3! = 6$ (root first, then any permutation of children). No edges gives $3! = 6$ (any permutation). All make intuitive sense.

---

## 4. Proof by Induction

### 4.1 Base Case

A single node. One way to order it.

$$\frac{1!}{1} = 1 \quad \checkmark$$

### 4.2 Inductive Step

Consider a tree rooted at $r$ with children subtrees $T_1, T_2, \ldots, T_k$ of sizes $s_1, s_2, \ldots, s_k$.

```
           r
         / | \
        T₁ T₂ ... Tₖ
       (s₁)(s₂)   (sₖ)
```

We build the ordering in two parts:

**Part A — Place the root:** $r$ must come first. No choice. (1 way)

**Part B — Interleave the subtrees:** We have $n - 1$ remaining positions. The $k$ subtrees must be interleaved while preserving each subtree's internal order. Number of interleavings:

$$\frac{(n - 1)!}{s_1! \times s_2! \times \cdots \times s_k!}$$

**Part C — Internal orderings:** Each subtree $T_i$ has its own valid orderings. By induction, $T_i$ is a tree rooted at some node $r_i$ whose children subtrees have sizes $a_1, a_2, \ldots$. So:

- $r_i$ goes first (1 way)
- Interleave $r_i$'s children subtrees in the remaining $s_i - 1$ positions

The interleaving count for $T_i$ is:

$$\frac{(s_i - 1)!}{\prod(\text{children subtree sizes of } r_i)}$$

> [!IMPORTANT]
> The numerator is $(s_i - 1)!$, **not** $s_i!$. The root of $T_i$ already took one position, leaving $s_i - 1$ positions to distribute among its children subtrees.

By the inductive hypothesis, each of $r_i$'s children subtrees further contributes its own internal orderings, and when everything is multiplied out, the total for $T_i$ is:

$$\frac{s_i!}{\displaystyle\prod_{\text{all nodes } u \in T_i} s(u)}$$

(This is just the formula applied to subtree $T_i$.)

**Multiply everything:**

$$1 \;\times\; \frac{(n-1)!}{s_1! \times \cdots \times s_k!} \;\times\; \frac{s_1!}{\prod_{u \in T_1} s(u)} \;\times\; \cdots \;\times\; \frac{s_k!}{\prod_{u \in T_k} s(u)}$$

### 4.3 The Cancellation

Write it as one big fraction:

$$\frac{(n-1)! \;\times\; s_1! \;\times\; s_2! \;\times\; \cdots \;\times\; s_k!}{s_1! \times s_2! \times \cdots \times s_k! \;\times\; \prod_{u \in T_1} s(u) \;\times\; \cdots \;\times\; \prod_{u \in T_k} s(u)}$$

Each $s_i!$ appears once on top (from $T_i$'s internal count) and once on bottom (from the interleaving). They cancel:

$$\frac{(n-1)!}{\prod_{u \in T_1} s(u) \;\times\; \cdots \;\times\; \prod_{u \in T_k} s(u)}$$

The denominator is the product of subtree sizes of every node **except the root** $r$. Since $s(r) = n$ and $(n-1)! = n!/n$:

$$\frac{n!/n}{\prod_{u \neq r} s(u)} = \frac{n!}{n \;\times\; \prod_{u \neq r} s(u)} = \frac{n!}{\prod_{\text{all } u} s(u)}$$

$\blacksquare$

> [!NOTE]
> **Where do the non-factorial terms in the denominator come from?**
>
> At each recursive level, the interleaving gives a factorial denominator, and the recursive formula from the level below gives a factorial numerator that's one less. For example, $s_i!$ on the bottom gets partially cancelled by $(s_i - 1)!$ from the top, since $s_i! = s_i \times (s_i - 1)!$. The leftover is the bare number $s_i$ — the subtree size. This happens at every node, leaving the product of bare subtree sizes.

---

## 5. Worked Examples

### 5.1 Three-Node Tree

```
      r
     / \
    A   B
```

| Node | Subtree size |
|------|-------------|
| r    | 3           |
| A    | 1           |
| B    | 1           |

$$\frac{3!}{3 \times 1 \times 1} = \frac{6}{3} = 2$$

Enumerate to verify: `r A B`, `r B A`. ✓

### 5.2 Five-Node Tree

```
        r
       / \
      A   B
     / \
    C   D
```

| Node | Subtree size |
|------|-------------|
| r    | 5           |
| A    | 3           |
| B    | 1           |
| C    | 1           |
| D    | 1           |

$$\frac{5!}{5 \times 3 \times 1 \times 1 \times 1} = \frac{120}{15} = 8$$

Let's trace through the logic to verify:

1. `r` goes first (forced)
2. Interleave left subtree `{A,C,D}` (size 3) with right subtree `{B}` (size 1): $\frac{3!}{3! \times 1!}$... wait, that's the interleaving of the **sequences**, not the subtrees directly. There are $\binom{4}{3} = 4$ interleavings.
3. Left subtree `{A,C,D}`: A goes first, then interleave `[C]` and `[D]` → $\binom{2}{1} = 2$ internal orderings.
4. Right subtree `{B}`: 1 ordering.

Total: $4 \times 2 \times 1 = 8$ ✓

All 8 orderings:

```
r A C D B
r A C B D
r A D C B
r A D B C
r A B C D
r A B D C
r B A C D
r B A D C
```

### 5.3 A Forest (Two Separate Trees)

```
  Tree 1:     Tree 2:
    r₁          r₂
   / \          |
  A   B         C
```

Treat as a single tree with a virtual root, or equivalently, interleave two independent trees.

| Node | Subtree size |
|------|-------------|
| r₁   | 3           |
| A    | 1           |
| B    | 1           |
| r₂   | 2           |
| C    | 1           |

$$\frac{5!}{3 \times 1 \times 1 \times 2 \times 1} = \frac{120}{6} = 20$$

This counts all ways to interleave the two trees' orderings while respecting parent-before-child within each tree.

---

## 6. The Probability Intuition

There's an elegant alternative way to see why the formula works, without induction.

Take a random permutation of all $n$ nodes. What's the probability that it happens to be a valid topological ordering?

For any node $u$ with subtree size $s(u)$, node $u$ must be the **first** among all nodes in its subtree. If the permutation is uniformly random, the probability of $u$ being first among its $s(u)$ subtree members is:

$$P_u = \frac{1}{s(u)}$$

**The key insight:** these events are independent across all nodes. This works because subtrees are **nested** (one contains the other or they're disjoint), never partially overlapping. So the constraints don't interfere.

Therefore:

$$P(\text{valid ordering}) = \prod_{u} \frac{1}{s(u)}$$

Since there are $n!$ total permutations:

$$\text{count} = n! \times \prod_{u} \frac{1}{s(u)} = \frac{n!}{\prod_{u} s(u)}$$

> [!TIP]
> This probability argument is the fastest way to derive the formula from scratch. No induction needed — just the observation that "node $u$ is first in its subtree" events are independent due to the nesting structure.

---

## 7. C++ Implementation

```cpp
#include <bits/stdc++.h>
using namespace std;

long long countTopoSortsTree(int n, vector<vector<int>>& children) {
    // Step 1: Find roots (nodes with no parent)
    vector<int> parent(n, -1);
    for (int u = 0; u < n; u++)
        for (int v : children[u])
            parent[v] = u;

    // Step 2: BFS to get processing order
    queue<int> q;
    vector<int> order;
    for (int i = 0; i < n; i++)
        if (parent[i] == -1) q.push(i);
    while (!q.empty()) {
        int u = q.front(); q.pop();
        order.push_back(u);
        for (int v : children[u]) q.push(v);
    }

    // Step 3: Compute subtree sizes (process leaves first)
    vector<int> subtreeSize(n, 1);
    for (int i = (int)order.size() - 1; i >= 0; i--) {
        int u = order[i];
        for (int v : children[u])
            subtreeSize[u] += subtreeSize[v];
    }

    // Step 4: Apply formula  n! / ∏ s(u)
    long long numerator = 1;
    for (int i = 1; i <= n; i++) numerator *= i;  // n!

    long long denominator = 1;
    for (int u = 0; u < n; u++) denominator *= subtreeSize[u];

    return numerator / denominator;
}

int main() {
    //        0
    //       / \
    //      1   2
    //     / \
    //    3   4
    int n = 5;
    vector<vector<int>> children(n);
    children[0] = {1, 2};
    children[1] = {3, 4};

    cout << "Count: " << countTopoSortsTree(n, children) << "\n";
    // Output: 8
}
```

**Complexity:**

| | |
|---|---|
| **Time** | $O(n)$ — one BFS + one pass for subtree sizes + one pass for the product |
| **Space** | $O(n)$ |

> [!CAUTION]
> For large $n$, the numerator $n!$ overflows `long long` around $n = 20$. For larger trees, use big-integer arithmetic or compute the result by dividing as you go (interleave multiplication and division to keep numbers small).

---

## 8. When This Formula Does NOT Apply

This formula is **only** for trees and forests (each node has at most one parent). It breaks for general DAGs:

```
    A
   / \
  B   C
   \ /
    D
```

This is a **diamond DAG**, not a tree. Node `D` has two parents. The subtree-size formula doesn't apply here because the "subtrees" overlap — `D` belongs to both `B`'s subtree and `C`'s subtree, violating the nesting property that makes the probability argument work.

For general DAGs, counting topological sorts is **#P-complete**. Use the bitmask DP approach ($O(2^n \cdot n)$) for small $n$, or approximate counting for larger instances.

| DAG Type | Method | Complexity |
|----------|--------|-----------|
| Tree / Forest | $n! / \prod s(u)$ | $O(n)$ |
| General DAG, $n \leq 20$ | Bitmask DP | $O(2^n \cdot n)$ |
| General DAG, large $n$ | Approximate (MCMC) | Research-level |

---

> *"Why does each node contribute exactly its subtree size to the denominator?"*
>
> Because each node demands to be first among its subtree members. The probability of that is $1/s(u)$. These demands are independent because subtrees nest. Multiply the probabilities, multiply by $n!$, done.
