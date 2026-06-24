https://claude.ai/public/artifacts/cfef89bb-a53b-4dc2-8b6a-3d2e2e40946f
# Disjoint Set Union (DSU)

## Intuition

Think of groups where each element points to a parent. Initially, every element points to itself — each is its own group.

**Find:** Follow the parent chain until you reach the root (element where `par[u] == u`). Two elements are in the same group iff they share the same root.

**Path Compression:** While finding the root, we redirect every node on the path to point directly to the root (`par[u] = find(par[u])`). Next time, `find` on any of these nodes is O(1). This is what makes DSU fast.

**Unite:** To merge two groups, make one root point to the other. We attach the smaller tree under the larger one (union by size) so the tree stays balanced.

Together, union by size + path compression → **amortized O(α(n))** per operation, effectively constant.

> **Union by rank vs size:** Rank tracks tree height instead of count. Both achieve the same complexity. Size is simpler and also lets you query component sizes directly, so it's generally preferred.

---

## Template

```cpp
struct DSU {
    int n;
    vector<int> par, sz;

    DSU(int n) : n(n), par(n), sz(n, 1) {
        for (int i = 0; i < n; i++) par[i] = i;
    }

    int find(int u) {
        return par[u] == u ? u : par[u] = find(par[u]);
    }

    bool unite(int u, int v) {
        u = find(u); v = find(v);
        if (u == v) return false;
        if (sz[u] < sz[v]) swap(u, v);
        par[v] = u;
        sz[u] += sz[v];
        return true;
    }

    bool connected(int u, int v) {
        return find(u) == find(v);
    }

    int size(int u) {
        return sz[find(u)];
    }
};
```

---

## Notes

- **0-indexed.** Convert 1-based input at read time.
- `unite` returns `false` if already connected — useful for Kruskal's or counting components.
- `size(u)` gives the component size containing `u`.
