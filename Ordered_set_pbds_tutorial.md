# Ordered Set — The Only PBDS Tutorial You Need

Everything about ordered sets in GNU C++ PBDS — the standard way, every variation, every trick, every gotcha.

---

## 1 — What It Is

An ordered set is `std::set` with two extra O(log n) operations:

- `find_by_order(k)` — iterator to the k-th element (0-indexed).
- `order_of_key(x)` — count of elements strictly less than x.

That's it. Those two operations replace segment trees, BITs, and merge-sort trees for a huge class of problems.

---

## 2 — The Standard Setup

```cpp
#include <bits/stdc++.h>
#include <ext/pb_ds/assoc_container.hpp>
#include <ext/pb_ds/tree_policy.hpp>
using namespace std;
using namespace __gnu_pbds;

template <typename T>
using ordered_set = tree<
    T,                                    // key type
    null_type,                            // null_type = set (not map)
    less<T>,                              // comparator
    rb_tree_tag,                          // ALWAYS red-black tree
    tree_order_statistics_node_update     // enables the two magic methods
>;
```

This works on every GCC-based judge — Codeforces, AtCoder, CodeChef, SPOJ. It does NOT compile on MSVC or Clang.

`bits/stdc++.h` does NOT include the `ext/` headers. You always need those two includes explicitly.

---

## 3 — The Two Core Operations (And Everything You Derive From Them)

```cpp
ordered_set<int> s;
s.insert(1); s.insert(5); s.insert(6); s.insert(17); s.insert(88);
// s = {1, 5, 6, 17, 88}
```

### find_by_order(k) — get the k-th smallest element

```cpp
*s.find_by_order(0)   // 1    (0th smallest)
*s.find_by_order(2)   // 6    (2nd smallest)
*s.find_by_order(4)   // 88   (4th smallest)
s.find_by_order(5)    // s.end() — OUT OF BOUNDS, do NOT dereference
```

### order_of_key(x) — count of elements STRICTLY LESS than x

```cpp
s.order_of_key(1)     // 0   (nothing < 1)
s.order_of_key(6)     // 2   (1, 5)
s.order_of_key(7)     // 3   (1, 5, 6)
s.order_of_key(25)    // 4   (1, 5, 6, 17)
s.order_of_key(100)   // 5   (all of them)
```

### Derived operations — build everything from these two

```cpp
// Count elements <= x
s.order_of_key(x + 1)

// Count elements > x
s.size() - s.order_of_key(x + 1)

// Count elements >= x
s.size() - s.order_of_key(x)

// Count elements in [lo, hi]
s.order_of_key(hi + 1) - s.order_of_key(lo)

// Rank of x (1-indexed, like "x is the 3rd smallest")
s.order_of_key(x) + 1

// k-th smallest (1-indexed)
*s.find_by_order(k - 1)

// k-th largest (1-indexed)
*s.find_by_order(s.size() - k)

// Minimum
*s.find_by_order(0)

// Maximum
*s.find_by_order(s.size() - 1)

// Median
*s.find_by_order(s.size() / 2)           // upper median
*s.find_by_order((s.size() - 1) / 2)     // lower median

// Delete by index
s.erase(s.find_by_order(idx));
```

All standard `std::set` methods work as usual — `insert`, `erase`, `find`, `lower_bound`, `upper_bound`, `size`, `empty`, `clear`, `begin`, `end`, range-for.

---

## 4 — Variation: Ordered Multiset (Allowing Duplicates)

The standard ordered set rejects duplicates. You will need duplicates very often. There are two approaches; one is correct and one is a trap.

### 4.1 — The Pair Trick (USE THIS)

Store `pair<value, unique_id>`. The pair's natural `<` keeps values sorted, and the unique id distinguishes duplicates.

```cpp
ordered_set<pair<int, int>> ms;
int uid = 0;   // unique id counter — just increment forever

// ── Insert ──
ms.insert({5, uid++});    // {5, 0}
ms.insert({2, uid++});    // {2, 1}
ms.insert({5, uid++});    // {5, 2}   ← duplicate value, different id
ms.insert({3, uid++});    // {3, 3}
ms.insert({2, uid++});    // {2, 4}
// Internal order: {2,1}, {2,4}, {3,3}, {5,0}, {5,2}
```

**Querying — the key insight is what you pass to `order_of_key`:**

```cpp
// Count of elements with value < x
ms.order_of_key({x, 0})
// Why {x, 0}? Because {x, 0} is less than any {x, positive_id},
// so it counts everything before the first occurrence of x.

// Count of elements with value <= x
ms.order_of_key({x + 1, 0})
// Or equivalently: ms.order_of_key({x, INT_MAX})
// (INT_MAX works because no uid will ever be that large)

// Count of elements with value == x
ms.order_of_key({x + 1, 0}) - ms.order_of_key({x, 0})

// Count in range [lo, hi]
ms.order_of_key({hi + 1, 0}) - ms.order_of_key({lo, 0})

// k-th smallest VALUE (0-indexed)
ms.find_by_order(k)->first       // .first strips out the uid
```

**Erasing one occurrence of a value:**

```cpp
void erase_one(ordered_set<pair<int,int>>& ms, int val) {
    auto it = ms.lower_bound({val, 0});
    if (it != ms.end() && it->first == val)
        ms.erase(it);
}
```

**Erasing all occurrences of a value:**

```cpp
void erase_all(ordered_set<pair<int,int>>& ms, int val) {
    while (true) {
        auto it = ms.lower_bound({val, 0});
        if (it == ms.end() || it->first != val) break;
        ms.erase(it);
    }
}
```

### 4.2 — The `less_equal` Approach (DANGEROUS — know the risks)

```cpp
// DO NOT blindly copy this
using ordered_multiset = tree<int, null_type, less_equal<int>,
                              rb_tree_tag, tree_order_statistics_node_update>;
```

What breaks:

| Operation | What happens |
|-----------|-------------|
| `find(x)` | Always returns `end()`. Never finds anything. |
| `erase(x)` by value | Undefined behavior. May crash. |
| `lower_bound(x)` | Acts like `upper_bound` (first element > x). |
| `upper_bound(x)` | Acts like `lower_bound` (first element >= x). |
| `order_of_key(x)` | Returns count of elements strictly less than x — works. |
| `find_by_order(k)` | Works. |

You can work around `erase` like this:

```cpp
ordered_multiset ms;
ms.insert(5); ms.insert(5); ms.insert(3);

// Erase one occurrence of 5:
auto it = ms.find_by_order(ms.order_of_key(5));
if (it != ms.end()) ms.erase(it);
```

But honestly — the pair trick is cleaner, safer, and every operation just works. Use it.

---

## 5 — Variation: Descending Order (Largest First)

```cpp
ordered_set<int> s_asc;     // default: {1, 3, 5, 7}  smallest first
// For descending:
using desc_set = tree<int, null_type, greater<int>,
                      rb_tree_tag, tree_order_statistics_node_update>;
```

Now `find_by_order(0)` returns the **largest** element, and `order_of_key(x)` counts elements **strictly greater** than x.

```cpp
desc_set s;
s.insert(1); s.insert(5); s.insert(3); s.insert(7);
// Internal order: {7, 5, 3, 1}

*s.find_by_order(0)    // 7  (0th in descending = largest)
*s.find_by_order(3)    // 1  (3rd in descending = smallest)
s.order_of_key(5)      // 1  (one element "less than 5" under greater<>, i.e., 7)
```

Most of the time you don't need this — you can use `s.size() - s.order_of_key(x)` on a normal ascending set to get the same information. But it's there if it makes your logic cleaner.

---

## 6 — Variation: Ordered Map (Key-Value with Order Statistics)

Change `null_type` to a value type:

```cpp
template <typename K, typename V>
using ordered_map = tree<K, V, less<K>, rb_tree_tag,
                         tree_order_statistics_node_update>;
```

```cpp
ordered_map<int, string> m;
m.insert({10, "ten"});
m.insert({3, "three"});
m.insert({7, "seven"});
// Sorted by key: {3:"three", 7:"seven", 10:"ten"}

auto it = m.find_by_order(1);
cout << it->first << " " << it->second;  // 7 seven

m.order_of_key(7);   // 1 (one key < 7)
```

Order statistics operate on **keys only**. Values are just along for the ride.

---

## 7 — Variation: Custom Struct as Key

You need `operator<` or a custom comparator. Also, for `_GLIBCXX_DEBUG` to work, you need `operator<<`.

```cpp
struct Point {
    int x, y;
    bool operator<(const Point& o) const {
        if (x != o.x) return x < o.x;
        return y < o.y;
    }
};

// For debug mode compatibility, also define:
ostream& operator<<(ostream& o, const Point& p) {
    return o << "(" << p.x << "," << p.y << ")";
}

ordered_set<Point> pts;
pts.insert({1, 3});
pts.insert({2, 1});
pts.insert({1, 5});
// Sorted: {1,3}, {1,5}, {2,1}

pts.order_of_key({2, 0});  // 2  (two points < {2,0})
```

**Or with an external comparator:**

```cpp
struct CmpByY {
    bool operator()(const Point& a, const Point& b) const {
        if (a.y != b.y) return a.y < b.y;
        return a.x < b.x;
    }
};

using y_sorted_set = tree<Point, null_type, CmpByY,
                          rb_tree_tag, tree_order_statistics_node_update>;
```

The comparator MUST define a strict weak ordering (`a < b`, never `a <= b`). If it doesn't, you get undefined behavior — silent corruption, not a compiler error.

---

## 8 — Variation: Strings, Vectors, Pairs

The ordered set is templated — it works with any type that has `<` defined.

```cpp
// ── Strings ──
ordered_set<string> words;
words.insert("apple");
words.insert("banana");
words.insert("cherry");
words.order_of_key("banana");   // 1 ("apple" < "banana")
*words.find_by_order(2);        // "cherry"

// ── Pairs (very common) ──
ordered_set<pair<int,int>> ps;
ps.insert({3, 1});
ps.insert({1, 4});
ps.insert({3, 2});
// Sorted: {1,4}, {3,1}, {3,2}
ps.order_of_key({3, 0});   // 1  ({1,4} < {3,0})

// ── Vectors ── (yes, this works)
ordered_set<vector<int>> vs;
vs.insert({1, 2, 6});
vs.insert({1, 3, 3});
vs.insert({2, 6});
vs.order_of_key({1, 3});   // 1  ({1,2,6} < {1,3})
```

---

## 9 — Split & Join

These are O(log n) on `rb_tree_tag`. They let you split a set around a value and merge two non-overlapping sets.

### Split

```cpp
ordered_set<int> a, b;
a.insert(1); a.insert(3); a.insert(5); a.insert(7); a.insert(9);
// a = {1,3,5,7,9}, b = {} (MUST be empty)

a.split(5, b);
// a = {1,3,5}     (elements <= 5 stay)
// b = {7,9}       (elements > 5 moved to b)
```

### Join

```cpp
ordered_set<int> a, b;
a.insert(1); a.insert(3); a.insert(5);
b.insert(7); b.insert(9);

a.join(b);
// a = {1,3,5,7,9}, b = {} (emptied)
// REQUIREMENT: max(a) < min(b) — ranges must NOT overlap
// If they overlap → exception / UB
```

### Extracting a range [lo, hi] from a set

```cpp
ordered_set<int> s, right_part, middle;
// s = {1, 3, 5, 7, 9, 11}

s.split(hi, right_part);       // right_part gets everything > hi
s.split(lo - 1, middle);       // middle gets everything > lo-1, i.e., [lo, hi]
// s now has [min, lo-1]
// middle has [lo, hi]         ← this is what you wanted
// right_part has [hi+1, max]

// Put back:
s.join(middle);
s.join(right_part);
```

### Rules

- The second argument of `split` **must be empty**. Otherwise: UB.
- For `join`, all keys in the argument must be strictly greater (or strictly less, depending on comparator) than all keys in `this`. Otherwise: exception.
- Only `rb_tree_tag` gives O(log n) split/join. The other tags (`splay_tree_tag`, `ov_tree_tag`) take O(n). Always use `rb_tree_tag`.

---

## 10 — Practical Problem Patterns

### 10.1 — Count Inversions

Given array a[], count pairs (i, j) with i < j and a[i] > a[j].

```cpp
long long count_inversions(vector<int>& a) {
    ordered_set<pair<int,int>> s;   // pair trick for duplicate values
    long long inv = 0;
    for (int i = 0; i < (int)a.size(); i++) {
        // Elements already in set that are > a[i]
        inv += s.size() - s.order_of_key({a[i], i});
        s.insert({a[i], i});   // i as unique id
    }
    return inv;
}
```

Time: O(n log n). This replaces a merge-sort or BIT approach with 5 lines.

### 10.2 — Sliding Window Median

Given array a[] and window size k, find the median of each window.

```cpp
void sliding_median(vector<int>& a, int k) {
    ordered_set<pair<int,int>> s;
    
    for (int i = 0; i < (int)a.size(); i++) {
        s.insert({a[i], i});
        
        if (i >= k) {
            // Remove the element leaving the window
            s.erase(s.lower_bound({a[i - k], i - k}));
        }
        
        if (i >= k - 1) {
            // Window is full — get median
            // Lower median at index (k-1)/2
            cout << s.find_by_order((k - 1) / 2)->first << " ";
        }
    }
}
```

The `{a[i], i}` pair trick is essential here — window elements can have duplicate values, and we need to erase the exact one leaving the window (identified by its index).

### 10.3 — Dynamic k-th Smallest (Online)

```cpp
ordered_set<pair<int,int>> s;
int uid = 0;

void add(int x) {
    s.insert({x, uid++});
}

void remove(int x) {
    auto it = s.lower_bound({x, 0});
    if (it != s.end() && it->first == x)
        s.erase(it);
}

int kth(int k) {  // 1-indexed
    return s.find_by_order(k - 1)->first;
}

int rank_of(int x) {  // 1-indexed rank
    return s.order_of_key({x, 0}) + 1;
}

int count_range(int lo, int hi) {
    return s.order_of_key({hi + 1, 0}) - s.order_of_key({lo, 0});
}
```

### 10.4 — Number of Elements Greater Than x After Each Insertion

```cpp
ordered_set<pair<int,int>> s;
int uid = 0;

int n; cin >> n;
while (n--) {
    int x; cin >> x;
    int greater_count = s.size() - s.order_of_key({x + 1, 0});
    cout << greater_count << "\n";
    s.insert({x, uid++});
}
```

### 10.5 — Rank Queries with Updates

"Insert x", "Delete x", "What rank is x?", "What is the element at rank k?"

All four operations are O(log n) with an ordered multiset (pair trick).

```cpp
ordered_set<pair<int,int>> s;
int uid = 0;

// Insert
s.insert({x, uid++});

// Delete one occurrence
auto it = s.lower_bound({x, 0});
if (it != s.end() && it->first == x) s.erase(it);

// Rank of x (1-indexed)
int rank = s.order_of_key({x, 0}) + 1;

// Element at rank k (1-indexed)
int elem = s.find_by_order(k - 1)->first;
```

### 10.6 — Replace Fenwick Tree / Segment Tree

If your problem only needs:
- Insert/delete elements dynamically
- Query k-th smallest
- Query rank of an element
- Count elements in a range

Then an ordered set replaces a Fenwick tree + coordinate compression setup. It's less code, handles any value range (no coordinate compression needed), and has the same O(log n) complexity.

The trade-off: ordered sets use more memory (~5x more than a BIT). For n ≤ 10^6 this is fine. For n ≤ 10^7 it might be tight.

### 10.7 — Counting Distinct Elements in a Range (Offline with Sweep)

```cpp
// For each query [l, r], count distinct values in a[l..r]
// Offline: process queries sorted by r
ordered_set<int> s;
map<int, int> last_pos;  // value -> last position it was seen

// When processing position r:
//   If a[r] was seen before at position p, erase p from ordered_set
//   Insert r into ordered_set
//   For query [l, r]: answer = s.size() - s.order_of_key(l)
//   (count of positions in s that are >= l)
```

---

## 11 — Gotchas & Pitfalls

### `order_of_key` is STRICTLY LESS, not <=

This is the #1 source of bugs. `order_of_key(5)` does NOT count 5 itself. To get count ≤ 5, use `order_of_key(6)` or `order_of_key(5 + 1)`.

### `find_by_order` with out-of-bounds index

Returns `end()`. Dereferencing `end()` is undefined behavior. Always check:

```cpp
auto it = s.find_by_order(k);
if (it == s.end()) { /* k >= s.size() */ }
```

### No duplicates in standard ordered set

`s.insert(5); s.insert(5);` — the second insert does nothing. Size stays 1. Use the pair trick (Section 4).

### `split` destination must be empty

```cpp
ordered_set<int> a, b;
b.insert(100);      // b is NOT empty
a.split(5, b);      // UNDEFINED BEHAVIOR — may crash, may corrupt
```

### `join` requires non-overlapping ranges

```cpp
ordered_set<int> a, b;
a.insert(1); a.insert(5);
b.insert(3);                // 3 is between 1 and 5
a.join(b);                   // THROWS std::logic_error or UB
```

### Memory usage

An ordered set node stores the key + color + left/right/parent pointers + the subtree-size metadata. Roughly 40-80 bytes per element. For n = 10^6, that's ~40-80 MB. For n = 5×10^6, you might hit memory limits.

### GCC-only

Does not compile on Clang or MSVC. On Codeforces, AtCoder, etc. — you're fine. Locally on a Mac (Clang by default) — install GCC via Homebrew.

### `_GLIBCXX_DEBUG` mode

If you compile with `-D_GLIBCXX_DEBUG`, ordered set on custom structs may fail to compile unless you define `operator<<` for your struct. This is a quirk of the debug mode internals.

```cpp
// Add this if using custom structs with debug mode:
ostream& operator<<(ostream& o, const MyStruct& s) {
    return o << s.some_field;
}
```

### Integer overflow with `order_of_key`

If your key type is `int` and you call `order_of_key(x + 1)` where x = INT_MAX, you get overflow. Use `long long` as key type, or handle the edge case.

---

## 12 — Complexity Cheat Sheet

| Operation | Time |
|-----------|------|
| `insert` | O(log n) |
| `erase` (by value or iterator) | O(log n) |
| `find` | O(log n) |
| `lower_bound` / `upper_bound` | O(log n) |
| `find_by_order(k)` | O(log n) |
| `order_of_key(x)` | O(log n) |
| `split` | O(log n) |
| `join` | O(log n) |
| `size` | O(1) |
| Iteration (range-for) | O(n) |
| Memory per element | ~40-80 bytes |

---

## 13 — Copy-Paste Template

```cpp
#include <bits/stdc++.h>
#include <ext/pb_ds/assoc_container.hpp>
#include <ext/pb_ds/tree_policy.hpp>
using namespace std;
using namespace __gnu_pbds;

// ── Ordered Set (unique elements) ──
template <typename T>
using ordered_set = tree<T, null_type, less<T>, rb_tree_tag,
                         tree_order_statistics_node_update>;
// s.find_by_order(k)  → iterator to k-th smallest (0-indexed)
// s.order_of_key(x)   → count of elements strictly < x

// ── Ordered Multiset (via pair trick) ──
// Use: ordered_set<pair<int, int>>
// Insert:            s.insert({val, uid++})
// Count < x:         s.order_of_key({x, 0})
// Count <= x:        s.order_of_key({x + 1, 0})
// Count in [lo,hi]:  s.order_of_key({hi+1, 0}) - s.order_of_key({lo, 0})
// k-th smallest:     s.find_by_order(k)->first
// Erase one of val:  auto it = s.lower_bound({val,0}); if (it->first==val) s.erase(it);

// ── Ordered Map (key-value with order stats on keys) ──
// template <typename K, typename V>
// using ordered_map = tree<K, V, less<K>, rb_tree_tag,
//                          tree_order_statistics_node_update>;

int main() {
    ios::sync_with_stdio(false);
    cin.tie(nullptr);

    return 0;
}
```

---

## 14 — When NOT to Use Ordered Set

- **Need sum/min/max of a range?** Ordered set only gives count-based queries. Use a segment tree or BIT for aggregate queries.
- **Need persistent versions?** No persistent PBDS. Use a persistent segment tree.
- **Need to merge two overlapping sets efficiently?** `join` requires non-overlapping ranges. If ranges overlap you must insert one-by-one.
- **n > 5×10^6?** Memory might be too tight. Fall back to a BIT with coordinate compression.
- **Non-GCC judge?** Won't compile. Write your own balanced BST or use a different approach.

For everything else — dynamic rank queries, k-th element, counting in ranges, inversions, sliding medians — the ordered set is the fastest thing you can code.