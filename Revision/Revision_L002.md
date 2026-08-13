# Revision — L002: SOLID Principles

**Format:** Active recall — answer before reading the answer.
**Completed:** 2026-08-13

---

## Q1. State the Single Responsibility Principle and give an example violation.

<details>
<summary>Answer</summary>

A class should have exactly one reason to change. Example violation: an `Order` class that both calculates totals (pricing logic) AND saves itself to the database (persistence logic) — two independent reasons to change. Fix: split into `Order` (pricing) and `OrderRepository` (persistence).

</details>

---

## Q2. How does the Open/Closed Principle rely on polymorphism (L001)?

<details>
<summary>Answer</summary>

Open/Closed requires adding new behavior without editing existing, tested code. A type-checking if/else chain (`if (type == "circle")...`) requires editing that method for every new type. Replacing it with a `Shape` interface and per-type classes means adding a new shape is a new class — zero existing code touched. Polymorphism is the mechanism; Open/Closed is the payoff.

</details>

---

## Q3. Why does "Square extends Rectangle" violate Liskov Substitution even though a square IS mathematically a rectangle?

<details>
<summary>Answer</summary>

Liskov Substitution is a behavioral contract check, not a grammatical one. A `Rectangle` guarantees independent width/height. `Square` can't honor that — setting one dimension forces the other to match. Code that substitutes a `Square` where a `Rectangle` is expected gets silently wrong behavior (e.g., setWidth(5) then setHeight(10) produces a 10x10 square instead of a 5x10 rectangle). This is the formal test for whether an inheritance choice (L001) was actually sound.

</details>

---

## Q4. Give an example of an Interface Segregation violation and its fix.

<details>
<summary>Answer</summary>

A fat `Worker` interface with both `work()` and `eat()` forces a `Robot` class to implement `eat()` even though robots don't eat (throwing an exception or no-op). Fix: segregate into `Workable` (work only) and `Eatable` (eat only) — `Robot` implements only `Workable`, `Human` implements both.

</details>

---

## Q5. How does Dependency Inversion relate to Dependency Injection (a later topic, L014)?

<details>
<summary>Answer</summary>

Dependency Inversion is the principle: high-level modules should depend on abstractions, not concrete implementations (e.g., `OrderService` should depend on a `NotificationSender` interface, not a concrete `GmailSender`). Dependency Injection is the technique that implements this principle in practice — the concrete implementation is passed in (injected) from outside rather than constructed directly inside the dependent class.

</details>

---

## 30-Second Elevator Pitch

> SOLID: Single Responsibility (one reason to change), Open/Closed (extend without editing, via polymorphism), Liskov Substitution (a subclass must honor its parent's full contract — the formal test for a sound inheritance choice), Interface Segregation (small focused interfaces over one fat one), Dependency Inversion (depend on abstractions; DI is the technique that implements this). The five letters aren't independent — O needs polymorphism from L001, D needs abstraction from L001, and L is the formal check on whether an L001 inheritance relationship was actually sound.

---

## Weak Areas to Watch

- None this session — clean pass on all checkpoint questions (Liskov violation correctly identified, SRP split correctly proposed with a forward-looking generic sender name).
