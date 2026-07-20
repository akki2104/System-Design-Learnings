# Revision — Topic 022: Indexing Deep Dive

Active recall only. No re-reading. Answer each question from memory before expanding.

---

## 30-Second Elevator Explanation

> "Without an index, finding rows means scanning every page — O(n). An index is a separate sorted structure (usually a B-Tree) that gives O(log n) lookups, because sorting lets you eliminate half the search space at each step, keeping the tree's height tiny even at huge scale. A clustered index IS the table's physical row order (one hop); a secondary index points to the row's location elsewhere (two hops — the bookmark lookup). Composite indexes only help queries filtering on a leftmost prefix of their columns. A covering index contains every column a query needs, skipping the bookmark lookup entirely. Indexes trade write speed and storage for read speed — the read:write ratio should drive how aggressively you index."

---

## Active-Recall Q&A

<details>
<summary>Q1: Why does B-Tree lookup cost stay near-constant (O(log n)) even as table size grows massively?</summary>

Sorting lets you eliminate half the remaining search space at each comparison — the same principle as binary search. A B-Tree extends this across disk pages. Because of this, the tree's HEIGHT stays tiny even as data grows massively: for 10 million rows, height ≈ log₂(10M) ≈ 24. Only ~24 page reads are needed to find any row, regardless of table size.

</details>

<details>
<summary>Q2: Walk through a secondary-index query vs. a clustered-index query — why does one cost more?</summary>

**Secondary index query (2 hops):** (1) find the row's location/ID in the secondary index, (2) fetch the actual row from that location — a "bookmark lookup."

**Clustered index query (1 hop):** the data IS physically stored in index order — finding the index entry IS finding the row. No second fetch needed.

</details>

<details>
<summary>Q3: Composite index on (country, city) — which of "WHERE city='Mumbai'" vs "WHERE country='India' AND city='Mumbai'" can use it?</summary>

Only `WHERE country='India' AND city='Mumbai'` can use it — leftmost-prefix rule. `WHERE city='Mumbai'` alone cannot use the index, because `city` is not the leftmost column; you can't skip past `country` to reach it, same as a phone book sorted by last name then first name.

</details>

<details>
<summary>Q4: A table has 50,000 writes/sec and 100 reads/sec. What's your indexing recommendation?</summary>

A ~500:1 write:read ratio is extremely write-heavy. Every index added must be updated on every single write, so with this little read traffic, the write-cost tax on indexes almost certainly isn't worth it. Recommendation: minimal or zero secondary indexes — at most the mandatory clustered/primary key index, and only add a secondary index if a specific, important read query pattern genuinely demands it.

</details>

<details>
<summary>Q5: What is a covering index, and what specific cost does it eliminate?</summary>

A covering index contains ALL the columns a query needs (not just the column it's sorted by). This eliminates the bookmark lookup entirely — the database can answer the query directly from the index (an "index-only scan") without ever touching the actual table.

</details>

---

## Key Diagram

```
FULL SCAN vs INDEX
─────────────────────────────────────────
Full table scan: O(n)         — check every row
B-Tree index lookup: O(log n)  — height stays tiny (10M rows → ~24 reads)

CLUSTERED (1 hop) vs SECONDARY (2 hops — bookmark lookup)
─────────────────────────────────────────
Clustered  → index entry IS the row
Secondary  → index entry → row location → fetch row

LEFTMOST PREFIX RULE
─────────────────────────────────────────
INDEX (A, B) helps: WHERE A=x | WHERE A=x AND B=y
does NOT help:      WHERE B=y alone
```

---

## My Weak Areas (from lesson 2026-07-13)

- None significant — clean 4/4 pass, including the write-heavy indexing trap question
- Q1's initial answer was correct but light on mechanism (why binary search applies) — sharpened during feedback

---

## Past Mistakes

None logged for this topic — clean pass across all checkpoint questions.
