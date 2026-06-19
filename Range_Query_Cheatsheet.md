## Query Structures — Required Properties

| Structure             | Required Properties                          |
|-----------------------|----------------------------------------------|
| Prefix Array          | Associativity, Inverse (Commutativity helps)  |
| BIT / Fenwick Tree    | Associativity, Commutativity, Inverse         |
| Segment Tree          | Associativity                                 |
| Sparse Table          | Associativity, Idempotency (Commutativity helps) |
| Disjoint Sparse Table | Associativity                                 |

---

## Property Definitions

**Associativity** — Grouping doesn't matter: `(a ⊕ b) ⊕ c = a ⊕ (b ⊕ c)`. You can split and combine subranges in any grouping. Required by almost everything.

**Commutativity** — Order doesn't matter: `a ⊕ b = b ⊕ a`. BIT needs this because it combines partial ranges in an arbitrary order. Matrix multiplication is associative but *not* commutative, so it works in segment trees but not BIT.

**Identity** — There exists an element `e` such that `a ⊕ e = a`. For sum it's 0, for product it's 1, for min it's ∞. Needed to initialize empty nodes/ranges.

**Inverse** — For every `a`, there exists `a⁻¹` such that `a ⊕ a⁻¹ = e` (identity). Sum has subtraction, XOR is its own inverse, but min/max/gcd have no inverse. This is what lets prefix arrays cancel out: `prefix[r] ⊕ inverse(prefix[l-1])`.

**Idempotency** — `a ⊕ a = a`. Overlapping ranges give the same answer, so sparse tables can use two overlapping precomputed blocks to answer queries in O(1). Min, max, gcd, OR, AND are idempotent. Sum and XOR are not.
