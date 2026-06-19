# Fenwick Tree (Binary Indexed Tree) — Complete Tutorial
[cp handbook tutorial](https://cses.fi/book/book.pdf#page=96)

Everything you need to solve problems with Fenwick Trees — from the bit tricks to the templates to the problem patterns.

---

## 1 — What It Is

A Fenwick Tree (BIT) is a data structure that supports two operations on an array in O(log n):

- **Point update** — add a value to a single index.
- **Prefix query** — get the sum (or any associative operation) from index 1 to i.

That's it. From these two, you derive range queries, frequency tables, inversion counts, and more.

It uses O(n) memory, has tiny constants, and fits in ~15 lines of code. When a segment tree works but feels like overkill, a Fenwick Tree is probably what you want.

---

## 2 — The Bit Trick That Makes It Work

Every positive integer can be written in binary. The **lowest set bit** (LSB) of a number determines what range of the array each Fenwick node is responsible for.

The LSB of `i` is extracted by `i & (-i)`. This works because `-i` in two's complement flips all bits and adds 1, so only the lowest set bit survives the AND.

```
i = 12    →  binary: 1100
-i = -12  →  binary: ...0100
i & (-i)  →          0100  =  4
```

Each node `tree[i]` stores the sum of a range of size `i & (-i)` ending at index `i`:

```
tree[1]  (0001) → stores a[1]           range size 1
tree[2]  (0010) → stores a[1..2]        range size 2
tree[3]  (0011) → stores a[3]           range size 1
tree[4]  (0100) → stores a[1..4]        range size 4
tree[5]  (0101) → stores a[5]           range size 1
tree[6]  (0110) → stores a[5..6]        range size 2
tree[7]  (0111) → stores a[7]           range size 1
tree[8]  (1000) → stores a[1..8]        range size 8
tree[9]  (1001) → stores a[9]           range size 1
tree[10] (1010) → stores a[9..10]       range size 2
tree[12] (1100) → stores a[9..12]       range size 4
tree[16] (10000)→ stores a[1..16]       range size 16
```

The pattern: `tree[i]` covers `i & (-i)` elements ending at position `i`.

---

## 3 — The Two Core Operations

### Update — add `delta` to position `i`

Walk **up** the tree by adding the LSB at each step. Every ancestor that covers position `i` gets updated.

```cpp
void update(int i, int delta) {
    for (; i <= n; i += i & (-i))
        tree[i] += delta;
}
```

Trace for `update(3, +1)` with n=8:

```
i = 3  (011) → update tree[3],  then i += 1  → i = 4
i = 4  (100) → update tree[4],  then i += 4  → i = 8
i = 8  (1000)→ update tree[8],  then i += 8  → i = 16 > n, stop
```

Three nodes updated. Always O(log n).

### Query — prefix sum from 1 to `i`

Walk **down** by stripping the LSB at each step. You hop through exactly the right non-overlapping ranges to cover `[1..i]`.

```cpp
int query(int i) {
    int s = 0;
    for (; i > 0; i -= i & (-i))
        s += tree[i];
    return s;
}
```

Trace for `query(7)`:

```
i = 7  (111) → add tree[7] (covers [7,7]),   then i -= 1  → i = 6
i = 6  (110) → add tree[6] (covers [5,6]),   then i -= 2  → i = 4
i = 4  (100) → add tree[4] (covers [1,4]),   then i -= 4  → i = 0, stop
```

Result = tree[7] + tree[6] + tree[4] = sum of [1..7]. Three nodes visited. Always O(log n).

### Why does stripping the LSB work?

Each step peels off a range of exactly `i & (-i)` elements. After subtracting, the new `i` is the position just before that range started. The ranges are guaranteed to be non-overlapping and to tile `[1..original_i]` perfectly. This falls out of the binary representation — each set bit corresponds to one range in the decomposition.

---

## 4 — The Standard Template

```cpp
struct BIT {
    vector<int> tree;
    int n;

    BIT(int n) : n(n), tree(n + 1, 0) {}

    void update(int i, int delta = 1) {
        for (; i <= n; i += i & (-i))
            tree[i] += delta;
    }

    int query(int i) {
        int s = 0;
        for (; i > 0; i -= i & (-i))
            s += tree[i];
        return s;
    }

    int query(int l, int r) {
        return query(r) - query(l - 1);
    }
};
```

Important: the BIT is **1-indexed**. Index 0 breaks it because `0 & (-0) = 0`, causing an infinite loop. Allocate `n + 1` elements and use indices `1..n`.

---

## 5 — Derived Operations

```cpp
// Range sum [l, r]
bit.query(r) - bit.query(l - 1)

// Point value at index i (if BIT stores prefix sums of deltas)
bit.query(i) - bit.query(i - 1)

// Set index i to value v (not add, but set)
// Need to know current value:
int cur = bit.query(i) - bit.query(i - 1);
bit.update(i, v - cur);
```

---

## 6 — Coordinate Compression

BIT is indexed by value, so if values are up to 10^9, you can't allocate that. Compress them to ranks in `[1..n]`:

```cpp
vector<int> sorted_a(a.begin(), a.end());
sort(sorted_a.begin(), sorted_a.end());
sorted_a.erase(unique(sorted_a.begin(), sorted_a.end()), sorted_a.end());

auto rank = [&](int v) {
    return (int)(lower_bound(sorted_a.begin(), sorted_a.end(), v)
                 - sorted_a.begin()) + 1;  // +1 for 1-based
};
```

Now `rank(v)` maps any value to `[1..m]` where `m` = number of distinct values. Build the BIT with size `m`.

---

## 7 — Problem Patterns

### 7.1 — Count Inversions

An inversion is a pair (i, j) with i < j and a[i] > a[j]. Process left to right; for each element, count how many already-inserted elements are greater.

```cpp
long long countInversions(vector<int>& a) {
    int n = a.size();

    // coordinate compress
    vector<int> s(a.begin(), a.end());
    sort(s.begin(), s.end());
    s.erase(unique(s.begin(), s.end()), s.end());
    auto rank = [&](int v) {
        return (int)(lower_bound(s.begin(), s.end(), v) - s.begin()) + 1;
    };

    BIT bit(s.size());
    long long inv = 0;
    for (int i = 0; i < n; i++) {
        int r = rank(a[i]);
        inv += i - bit.query(r);  // elements already in with rank > r
        bit.update(r);
    }
    return inv;
}
```

### 7.2 — Controlling Inversion Strictness

When counting pairs where `a_i < a_j` and `b_i > b_j` (or variants with ≤, ≥), two knobs control strictness:

**Knob 1 — tie-breaking in sort (controls strictness of `a`):**
- Strict `<` on `a`: sort ties in `a` by `b` descending
- Non-strict `<=` on `a`: sort ties in `a` by `b` ascending

**Knob 2 — BIT query (controls strictness of `b`):**
- Strict `>` on `b`: `i - query(rank)`
- Non-strict `>=` on `b`: `i - query(rank - 1)`

| Condition | Sort ties in `a` by | Query |
|---|---|---|
| `a_i < a_j` and `b_i > b_j` | `b` descending | `i - query(rank)` |
| `a_i <= a_j` and `b_i >= b_j` | `b` ascending | `i - query(rank-1)` |
| `a_i < a_j` and `b_i >= b_j` | `b` descending | `i - query(rank-1)` |
| `a_i <= a_j` and `b_i > b_j` | `b` ascending | `i - query(rank)` |

### 7.3 — Dynamic Frequency Table / Count in Range

Insert and delete elements dynamically, query how many elements fall in a range `[lo, hi]`.

```cpp
BIT bit(MAX_VAL);  // or use coordinate compression

// Insert value x
bit.update(rank(x), +1);

// Delete value x
bit.update(rank(x), -1);

// Count elements in [lo, hi]
bit.query(rank(hi)) - bit.query(rank(lo) - 1)

// Count elements <= x
bit.query(rank(x))

// Count elements > x
total_inserted - bit.query(rank(x))
```

### 7.4 — k-th Smallest Element (Binary Lifting on BIT)

Find the smallest index whose prefix sum >= k. This is O(log^2 n) with binary search, or O(log n) with bit lifting:

```cpp
// O(log n) — walk the BIT structure directly
int kth(BIT& bit, int k) {
    int pos = 0;
    for (int pw = 1 << __lg(bit.n); pw > 0; pw >>= 1) {
        if (pos + pw <= bit.n && bit.tree[pos + pw] < k) {
            pos += pw;
            k -= bit.tree[pos];
        }
    }
    return pos + 1;  // 1-indexed position
}
```

This walks from the highest bit down, greedily jumping right when the subtree sum is less than k. Same idea as binary search on a segment tree, but on the BIT's implicit structure.

### 7.5 — Prefix Sum Queries with Point Updates

The most basic use case: maintain an array where you can update individual elements and query prefix sums.

```cpp
// Initial array a[1..n]
BIT bit(n);
for (int i = 1; i <= n; i++)
    bit.update(i, a[i]);

// Add delta to position i
bit.update(i, delta);

// Sum of a[1..i]
bit.query(i);

// Sum of a[l..r]
bit.query(r) - bit.query(l - 1);
```

### 7.6 — Number of Elements Greater/Less Than x After Each Insertion

```cpp
BIT bit(m);  // m = number of distinct values
int uid = 0;

for (int i = 0; i < n; i++) {
    int r = rank(a[i]);

    int less_than  = bit.query(r - 1);
    int greater_than = i - bit.query(r);
    int less_equal = bit.query(r);
    int greater_equal = i - bit.query(r - 1);

    bit.update(r);
}
```

---

## 8 — Range Update, Point Query (Difference Array Trick)

If you need to add `delta` to every element in `[l, r]` and query individual elements:

```cpp
// Range update: add delta to [l, r]
bit.update(l, delta);
bit.update(r + 1, -delta);

// Point query: value at position i
bit.query(i);
```

This works because `query(i)` computes the prefix sum of the difference array, which reconstructs the actual value at `i`.

---

## 9 — Range Update, Range Query (Two BITs)

If you need both range updates and range queries, use two BITs:

```cpp
struct RangeBIT {
    BIT b1, b2;
    int n;

    RangeBIT(int n) : n(n), b1(n), b2(n) {}

    void range_update(int l, int r, int delta) {
        b1.update(l, delta);
        b1.update(r + 1, -delta);
        b2.update(l, delta * (l - 1));
        b2.update(r + 1, -delta * r);
    }

    long long prefix_query(int i) {
        return (long long)b1.query(i) * i - b2.query(i);
    }

    long long range_query(int l, int r) {
        return prefix_query(r) - prefix_query(l - 1);
    }
};
```

The math: if you add `delta` to `[l, r]`, the prefix sum at position `i` increases by `delta * (i - l + 1)` for `i` in `[l, r]`. This decomposes into `delta * i - delta * (l - 1)`, which is tracked by the two BITs.

---

## 10 — 2D Fenwick Tree

For 2D prefix sums with point updates:

```cpp
struct BIT2D {
    vector<vector<int>> tree;
    int n, m;

    BIT2D(int n, int m) : n(n), m(m), tree(n + 1, vector<int>(m + 1, 0)) {}

    void update(int x, int y, int delta) {
        for (int i = x; i <= n; i += i & (-i))
            for (int j = y; j <= m; j += j & (-j))
                tree[i][j] += delta;
    }

    int query(int x, int y) {
        int s = 0;
        for (int i = x; i > 0; i -= i & (-i))
            for (int j = y; j > 0; j -= j & (-j))
                s += tree[i][j];
        return s;
    }

    int query(int x1, int y1, int x2, int y2) {
        return query(x2, y2) - query(x1 - 1, y2)
             - query(x2, y1 - 1) + query(x1 - 1, y1 - 1);
    }
};
```

Each operation is O(log n · log m).

---

## 11 — Building a BIT in O(n) Instead of O(n log n)

Instead of calling `update` for each element, you can build the tree directly:

```cpp
BIT(vector<int>& a) : n(a.size()), tree(a.size() + 1, 0) {
    for (int i = 1; i <= n; i++) {
        tree[i] += a[i - 1];  // a is 0-indexed
        int parent = i + (i & (-i));
        if (parent <= n)
            tree[parent] += tree[i];
    }
}
```

Each element propagates its value to its immediate parent exactly once.

---

## 12 — Gotchas

### Index 0 is death
`0 & (-0) = 0`, so `update(0, ...)` loops forever. Always use 1-based indexing.

### Integer overflow
If values can be large and n is large, use `long long` for the tree:
```cpp
vector<long long> tree;
```

### Coordinate compression overflow
If key type is `int` and you call `rank(x + 1)` where `x = INT_MAX`, you overflow. Use `long long` keys or handle the edge case.

### BIT is not a segment tree
BIT only supports operations where you can compute range answers from prefix answers (sum, xor, count). It cannot do range min/max queries. For those, use a segment tree.

### Off-by-one in range queries
`query(l, r) = query(r) - query(l - 1)`. If you write `query(l)` instead of `query(l - 1)`, you exclude index `l`. This is the #1 bug.

---

## 13 — Complexity Cheat Sheet

| Operation | Time |
|---|---|
| Point update | O(log n) |
| Prefix query | O(log n) |
| Range query [l, r] | O(log n) |
| Build from array | O(n) |
| k-th smallest (binary lifting) | O(log n) |
| Range update + point query | O(log n) |
| Range update + range query (two BITs) | O(log n) |
| 2D point update | O(log n · log m) |
| 2D range query | O(log n · log m) |
| Memory | O(n) |

---

## 14 — When to Use What

| Need | Use |
|---|---|
| Prefix sums + point updates | BIT |
| Range sum + point updates | BIT |
| Range min/max | Segment tree |
| Range update + range query | Two BITs or segment tree with lazy |
| Dynamic rank / k-th element | BIT with coord compression, or ordered set |
| Inversion counting | BIT |
| 2D prefix sums + updates | 2D BIT |
| Persistent queries | Persistent segment tree |

BIT is simpler, faster (2-3x), and uses less memory than a segment tree. Use a segment tree only when BIT can't express the query (min, max, lazy propagation on complex operations).

---

## 15 — Copy-Paste Template

```cpp
struct BIT {
    vector<long long> tree;
    int n;

    BIT(int n) : n(n), tree(n + 1, 0) {}

    void update(int i, long long delta = 1) {
        for (; i <= n; i += i & (-i))
            tree[i] += delta;
    }

    long long query(int i) {
        long long s = 0;
        for (; i > 0; i -= i & (-i))
            s += tree[i];
        return s;
    }

    long long query(int l, int r) {
        return query(r) - query(l - 1);
    }
};

// Coordinate compression
vector<int> sorted_vals(a.begin(), a.end());
sort(sorted_vals.begin(), sorted_vals.end());
sorted_vals.erase(unique(sorted_vals.begin(), sorted_vals.end()), sorted_vals.end());
auto rank = [&](int v) {
    return (int)(lower_bound(sorted_vals.begin(), sorted_vals.end(), v)
                 - sorted_vals.begin()) + 1;
};

// Usage
BIT bit(sorted_vals.size());
bit.update(rank(x));              // insert x
bit.query(rank(x));               // count of elements <= x
bit.query(rank(r)) - bit.query(rank(l) - 1);  // count in [l, r]
```
