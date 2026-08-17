# Revision — Topic 038: Vertical vs Horizontal Scaling

**Format:** Active recall — answer before reading the answer.
**Completed:** 2026-08-17

---

## Q1. Why is vertical scaling usually tried before horizontal, even though horizontal scales further?

<details>
<summary>Answer</summary>

Horizontal scaling introduces real distributed-systems complexity that vertical scaling never has — network calls between nodes, partial failure, and consistency between nodes — while vertical scaling stays simple because it's still one machine. That complexity is only worth paying once a real ceiling (cost, max instance size, or an availability requirement) actually forces the switch.

</details>

---

## Q2. Name one new class of problem horizontal scaling introduces that vertical scaling doesn't have at all.

<details>
<summary>Answer</summary>

Consistency between nodes — with multiple machines each potentially holding a copy of data, they can disagree on the current state, a problem that simply doesn't exist when there's only one machine.

</details>

---

## Q3. Is horizontally scaling a stateless web server tier the same kind of problem as horizontally scaling a database?

<details>
<summary>Answer</summary>

No. Scaling the stateless app tier is nearly free: put a load balancer in front and add identical instances — any instance can serve any request because none hold request-specific state. Scaling the database (stateful) tier is the genuinely hard problem, because the data has to actually live somewhere and can't just be cloned onto N machines — this is exactly why replication, partitioning/sharding, and consistent hashing (Topics 039-043) exist as their own multi-topic block.

</details>

---

## Q4. Why is vertical scaling's cost curve worse than linear?

<details>
<summary>Answer</summary>

Cloud providers price larger instances at a premium — doubling CPU/RAM within an instance family typically costs more than 2x, because bigger instances are a smaller, less commoditized slice of the market. Horizontal scaling with many smaller/commodity instances can often deliver the same aggregate capacity for less total cost.

</details>

---

## Q5. Give an example of a real system that scaled vertically much further than the "you must eventually go horizontal" reflex would predict.

<details>
<summary>Answer</summary>

Stack Overflow ran successfully on a strikingly small number of large, vertically-scaled SQL Server boxes for years — their read-heavy workload plus an effective caching layer meant one well-specified machine could carry enormous load. A useful counter-example against treating horizontal scaling as an inevitable default.

</details>

---

## 30-Second Elevator Pitch

> Vertical scaling (bigger machine) is simple and has no cross-node consistency problem, but hits a hard ceiling, is a single point of failure, and its cost curve is worse than linear — bigger instances cost disproportionately more. Horizontal scaling (more machines) scales further and adds redundancy, but introduces real distributed-systems complexity: network calls, partial failure, consistency between nodes. The critical nuance is layer: horizontal scaling is cheap and nearly automatic for a stateless app tier (load balancer + more instances), but genuinely hard for the stateful database tier — which is why replication and sharding exist as their own topics. Practical default: scale vertically until a real ceiling forces the switch, and remember real systems (Stack Overflow) have run vertically far longer than the "eventually you must go horizontal" reflex suggests.

---

## Weak Areas to Watch

- The stateless-app-tier vs stateful-DB-tier distinction and the cost-curve point were added to the documentation after the live checkpoint (which covered the core vertical/horizontal tradeoff cleanly, 2/2) — not yet independently tested. Worth a quick check next revision pass rather than assuming they're internalized.
