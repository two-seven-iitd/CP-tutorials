# Pull vs Push DP

A complete tutorial with side-by-side examples, decision guides, and pitfalls

---

## Resources

- **Errichto's DP Lectures** — Push/pull intro, Fibonacci, grid, knapsack — [YouTube](https://www.youtube.com/playlist?list=PLl0KD3g-oDOGJUdmhFk19LaPgrfmAGQfo)
- **AtCoder DP Contest** — 26 problems, the best structured DP practice set — [AtCoder](https://atcoder.jp/contests/dp)
- **USACO Guide — Intro to DP** — Covers forward/backward DP with editorial clarity — [usaco.guide](https://usaco.guide/gold/intro-dp)
- **CSES Problem Set** — ~20 classic DP problems for drilling — [cses.fi](https://cses.fi/problemset/)
- **CP-Algorithms** — Reference-style DP articles (mostly pull) — [cp-algorithms.com](https://cp-algorithms.com)

---

## Section 01 — What is the Difference?

Every iterative DP fills a table cell by cell. The question is: **who initiates the data transfer?**

**← Pull (backward-looking):**
You stand at cell `dp[i]` and ask, *"Which earlier cells contribute to me?"* You **read** from the past and **write** to yourself.

**Push (forward-looking) →:**
You stand at cell `dp[i]` and ask, *"Which future cells do I contribute to?"* You **read** from yourself and **write** to the future.

They always produce the same final table. The difference is purely about **which cell's perspective you write the loop from.**

> **Mental Model:** Imagine a chain of dominoes. **Pull:** You're domino `i`, you look behind and ask "who knocked into me?" **Push:** You're domino `i`, you fall forward and ask "who do I knock into?"

---

## Section 02 — Fibonacci — The Simplest Example

**Recurrence:** `f(n) = f(n-1) + f(n-2)`, with `f(0) = 0, f(1) = 1`.

### Pull

```cpp
dp[0] = 0;
dp[1] = 1;
for (int i = 2; i <= n; i++) {
    dp[i] = dp[i-1] + dp[i-2];   // I pull from my two predecessors
}
```

At each `i`, you look backwards and combine the two values that feed into you.

### Push

```cpp
dp[0] = 0;
dp[1] = 1;
for (int i = 0; i <= n; i++) {
    if (i + 1 <= n) dp[i+1] += dp[i];   // I push into my successor
    if (i + 2 <= n) dp[i+2] += dp[i];   // I push into the one after
}
```

| Aspect | Pull | Push |
|--------|------|------|
| Each cell written | Exactly once | Multiple times (two sources) |
| Boundary handling | Need `i >= 2` | Need `i+1 <= n` |
| Readability | More natural | Slightly awkward |
| Winner | **Pull ✓** | — |

---

## Section 03 — Staircase / Climbing Stairs

**Problem:** You can climb 1 or 2 stairs at a time. How many ways to reach stair `n`?

### Pull — "How could I have arrived?"

```cpp
dp[0] = 1;  // one way to stand at the bottom
for (int i = 1; i <= n; i++) {
    dp[i] = 0;
    if (i >= 1) dp[i] += dp[i-1];   // came from one step below
    if (i >= 2) dp[i] += dp[i-2];   // came from two steps below
}
// answer: dp[n]
```

### Push — "Where can I go?"

```cpp
dp[0] = 1;
for (int i = 0; i <= n; i++) {
    if (i + 1 <= n) dp[i+1] += dp[i];   // take 1 step
    if (i + 2 <= n) dp[i+2] += dp[i];   // take 2 steps
}
// answer: dp[n]
```

> **Key Insight:** In **push**, transitions map to **actions** ("take 1 step", "take 2 steps"). In **pull**, they map to **origins** ("came from 1 below", "came from 2 below"). One framing is often more natural depending on the problem.

---

## Section 04 — Minimum Path Sum in a Grid

**Problem:** Given an `m × n` grid of costs, find the minimum cost path from top-left to bottom-right, moving only right or down.

### Pull

```cpp
// dp[i][j] = min cost to reach cell (i, j)
dp[0][0] = grid[0][0];
for (int i = 0; i < m; i++) {
    for (int j = 0; j < n; j++) {
        if (i == 0 && j == 0) continue;
        dp[i][j] = grid[i][j];
        int best = INT_MAX;
        if (i > 0) best = min(best, dp[i-1][j]);   // came from above
        if (j > 0) best = min(best, dp[i][j-1]);   // came from left
        dp[i][j] += best;
    }
}
// answer: dp[m-1][n-1]
```

### Push

```cpp
// initialize all to infinity except start
for (int i = 0; i < m; i++)
    for (int j = 0; j < n; j++)
        dp[i][j] = INF;
dp[0][0] = grid[0][0];

for (int i = 0; i < m; i++) {
    for (int j = 0; j < n; j++) {
        if (i + 1 < m)
            dp[i+1][j] = min(dp[i+1][j], dp[i][j] + grid[i+1][j]);
        if (j + 1 < n)
            dp[i][j+1] = min(dp[i][j+1], dp[i][j] + grid[i][j+1]);
    }
}
// answer: dp[m-1][n-1]
```

> **Key Observation:** In push, you must **initialize everything to infinity** because cells get updated from multiple sources. In pull, you compute each cell exactly once — no pre-initialization needed beyond the base case.

---

## Section 05 — 0/1 Knapsack — Where it Gets Interesting

**Problem:** `n` items, each with weight `w[i]` and value `v[i]`. Knapsack capacity `W`. Maximize total value.

### Pull (the textbook version)

```cpp
// dp[i][j] = max value using first i items with capacity j
for (int i = 1; i <= n; i++) {
    for (int j = 0; j <= W; j++) {
        // option 1: don't take item i
        dp[i][j] = dp[i-1][j];

        // option 2: take item i (pull from smaller capacity)
        if (j >= w[i])
            dp[i][j] = max(dp[i][j], dp[i-1][j - w[i]] + v[i]);
    }
}
// answer: dp[n][W]
```

### Push

```cpp
// initialize dp[0][0] = 0, everything else = -1 (unreachable)
for (int i = 0; i < n; i++) {
    for (int j = 0; j <= W; j++) {
        if (dp[i][j] < 0) continue;  // skip unreachable states

        // option 1: skip item i+1
        dp[i+1][j] = max(dp[i+1][j], dp[i][j]);

        // option 2: take item i+1
        if (j + w[i+1] <= W)
            dp[i+1][j + w[i+1]] = max(dp[i+1][j + w[i+1]], dp[i][j] + v[i+1]);
    }
}
// answer: max over all j of dp[n][j]
```

> **Why push shines here:** Notice `if (dp[i][j] < 0) continue;` — push naturally **skips unreachable states**. When the state space is sparse, this avoids wasted work that pull would do computing every cell.

---

## Section 06 — Space Optimization — Flattening to 1D

### Pull (iterate right to left)

```cpp
// dp[j] = max value with capacity j
for (int i = 1; i <= n; i++) {
    for (int j = W; j >= w[i]; j--) {        // RIGHT TO LEFT
        dp[j] = max(dp[j], dp[j - w[i]] + v[i]);
    }
}
```

Why right to left? Because `dp[j]` pulls from `dp[j - w[i]]`. Going left to right would overwrite `dp[j - w[i]]` before you read it — using item `i` twice (that's unbounded knapsack, not 0/1).

### Push (any order, with a copy)

```cpp
for (int i = 0; i < n; i++) {
    vector<int> ndp = dp;  // copy = skip option
    for (int j = 0; j <= W; j++) {
        if (dp[j] < 0) continue;
        if (j + w[i+1] <= W)
            ndp[j + w[i+1]] = max(ndp[j + w[i+1]], dp[j] + v[i+1]);
    }
    dp = ndp;
}
```

| Space-optimized | Pull | Push |
|-----------------|------|------|
| Extra array needed? | No (in-place, reverse order) | Yes (`ndp` copy) |
| Iteration order matters? | Yes (must go right-to-left) | No (any order) |
| Skip unreachable? | Not natural | Natural (`continue` on -1) |

---

## Section 07 — Contest Example — Seating Arrangement

**Problem:** `n` people arrive in order (I, E, or A). `x` tables with `s` seats. Introverts need empty tables, extroverts need non-empty, ambiverts can go either. Maximize people seated.

**State:** `dp[i][o]` = max fillers after first `i` people, with `o` tables opened. Available seats = `o * (s - 1) - fillers`.

### Push version

```cpp
vector<int> dp(x + 1, -1);
dp[0] = 0;

for (int i = 0; i < n; i++) {
    vector<int> ndp = dp;  // skip = copy
    for (int o = 0; o <= x; o++) {
        if (dp[o] < 0) continue;

        int fillers = dp[o];
        int avail = o * (s - 1) - fillers;

        // push: open a table
        if ((st[i] == 'I' || st[i] == 'A') && o < x)
            ndp[o + 1] = max(ndp[o + 1], fillers);

        // push: fill a seat
        if ((st[i] == 'E' || st[i] == 'A') && avail > 0)
            ndp[o] = max(ndp[o], fillers + 1);
    }
    dp = ndp;
}

int ans = 0;
for (int o = 0; o <= x; o++)
    if (dp[o] >= 0) ans = max(ans, o + dp[o]);
cout << ans << "\n";
```

### Pull version

```cpp
vector<vector<int>> dp(n + 1, vector<int>(x + 1, -1));
dp[0][0] = 0;

for (int i = 1; i <= n; i++) {
    char c = st[i - 1];
    for (int o = 0; o <= x; o++) {
        // pull: skip person i
        dp[i][o] = dp[i-1][o];

        // pull: person i opened a table
        if ((c == 'I' || c == 'A') && o > 0 && dp[i-1][o-1] >= 0)
            dp[i][o] = max(dp[i][o], dp[i-1][o-1]);

        // pull: person i filled a seat
        if ((c == 'E' || c == 'A') && dp[i-1][o] >= 0) {
            int avail = o * (s - 1) - dp[i-1][o];
            if (avail > 0)
                dp[i][o] = max(dp[i][o], dp[i-1][o] + 1);
        }
    }
}

int ans = 0;
for (int o = 0; o <= x; o++)
    if (dp[n][o] >= 0) ans = max(ans, o + dp[n][o]);
cout << ans << "\n";
```

> **Why push is more natural here:** Transitions map to **actions a person takes**: "open" or "fill." Push asks "what can this person do?" — mirroring the problem statement. Pull inverts: "if I have `o` tables open, who opened the last one?" — doable but less intuitive.

---

## Section 08 — When Each Style Wins

### Use Pull when

1. Fixed, small number of predecessors (Grid, Fibonacci, LCS, LIS)
2. You want top-down / memoization (recursive DP is inherently pull)
3. Textbook clarity needed (most editorials use pull)
4. No unreachable states — compute-everything wastes no effort

### Use Push when

1. Many unreachable states — skip them with `continue`
2. Transitions are actions ("take step", "open table")
3. Variable fan-out (graph traversal, BFS-style DP)
4. Dijkstra / shortest-path style (relax edges outward)

---

## Section 09 — Common Pitfalls

**Pitfall 1 — Forgetting initialization in push:**
Push writes with `max()` / `min()`. If future cells default to 0, you compare against garbage.
**Fix:** Initialize to `-1` or `-INF`, then `if (dp[i][j] < 0) continue;`

**Pitfall 2 — Corrupting current row in push:**
Pushing from `dp[j]` to `dp[j+w]` in the same array overwrites cells you haven't read yet.
**Fix:** Use a copy: `vector<int> ndp = dp;` — read from `dp`, write to `ndp`.

**Pitfall 3 — Forgetting reachability in pull:**
Pulling from `dp[i-1][o-1]` without checking it's reachable (`>= 0`) propagates garbage values.
**Fix:** Always guard: `if (dp[i-1][o-1] >= 0)`

**Pitfall 4 — Wrong iteration order in 1D pull:**
0/1 knapsack with 1D array: left-to-right lets you use item `i` twice (unbounded knapsack).
**Fix:** Iterate capacity **right to left**: `for (int j = W; j >= w[i]; j--)`

---

## Section 10 — Practice Problems

| Problem | Key Concept | Try First |
|---------|-------------|-----------|
| AtCoder DP-A: Frog 1 | 1D DP, min cost | Pull |
| AtCoder DP-B: Frog 2 | 1D DP, variable jump | Pull |
| AtCoder DP-D: Knapsack 1 | 0/1 Knapsack | Both |
| AtCoder DP-E: Knapsack 2 | Large weights | Pull |
| AtCoder DP-G: Longest Path | DAG longest path | Push |
| AtCoder DP-L: Deque | Game theory, interval | Pull |
| AtCoder DP-O: Matching | Bitmask DP | Push |
| CSES: Dice Combinations | Counting paths | Both |
| CSES: Grid Paths | 2D grid, counting | Pull |
| CSES: Book Shop | 0/1 knapsack variant | Both |

---

## Section 11 — Summary Cheat Sheet

| Aspect | ← Pull | Push → |
|--------|--------|--------|
| Question | "Where did I come from?" | "Where can I go?" |
| Writes to | `dp[i]` (current cell) | `dp[future]` (many cells) |
| Reads from | `dp[past]` (predecessors) | `dp[i]` (current cell) |
| Cell writes | Exactly once | Potentially many times |
| Init | Base cases only | Everything to ±INF |
| Unreachable | Still computed | Skipped (`continue`) |
| Top-down? | Yes (memoization) | No (iterative only) |
| Space opt. | Reverse iteration | Copy array (`ndp = dp`) |
| Best when | Few predecessors, clarity | Sparse, action-based |

---

## Section 12 — Final Advice

1. **Learn pull first.** It's the textbook standard. Most editorials use it. Build a strong foundation here.
2. **Learn push second.** Convert a few pull solutions to push. Some problems become drastically simpler.
3. **Don't mix them.** Within one solution, stick to one style. Mixing causes subtle bugs.
4. **The "action" test.** If transitions are actions ("jump", "take", "open"), try push — it maps directly to the problem.
5. **The "origin" test.** If you can enumerate the 2–3 cells feeding into the current cell, try pull — clean single-assignment code.
6. **When stuck, try the other one.** If your push DP has confusing boundaries, rewrite as pull (or vice versa). Switching perspective is sometimes all you need.