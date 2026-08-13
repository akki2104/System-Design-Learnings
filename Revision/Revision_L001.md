# Revision — L001: OOP Fundamentals

**Format:** Active recall — answer before reading the answer.
**Completed:** 2026-08-13

---

## Q1. What's the precise difference between encapsulation and abstraction?

<details>
<summary>Answer</summary>

Encapsulation hides *state* — it controls who's allowed to touch a class's data, enforcing invariants through methods (an implementation concern). Abstraction hides *complexity* — it exposes a simple interface without revealing how the work is actually done (a design concern). A class can have one without the other; they answer different questions.

</details>

---

## Q2. Why is inheritance overused, and what should you check before reaching for it?

<details>
<summary>Answer</summary>

Inheritance models an is-a relationship, but many relationships are actually has-a (composition) or behaves-like. Using inheritance for a has-a relationship (e.g. a Car "is-a" Engine, when really a Car "has-a" Engine) creates a fragile hierarchy that breaks when requirements shift. Check: is this truly is-a, or would composition be safer/more flexible? Composition is the safer default (L007).

</details>

---

## Q3. Distinguish runtime polymorphism from compile-time polymorphism.

<details>
<summary>Answer</summary>

Runtime (dynamic) polymorphism = method overriding; the actual implementation is chosen based on the object's real type at runtime. Compile-time (static) polymorphism = method overloading; the compiler picks the method based on argument types at compile time. Different mechanisms resolved at different times.

</details>

---

## Q4. How does polymorphism directly enable the Open/Closed Principle?

<details>
<summary>Answer</summary>

Polymorphism lets calling code invoke one interface method and get different behavior per actual runtime type, eliminating type-checking if/else chains. Adding new behavior means writing a new class implementing the interface — zero existing code is touched or re-tested. That's exactly what Open/Closed requires: open for extension, closed for modification.

</details>

---

## Q5. Give an example (not from the lesson) that shows encapsulation and abstraction solving two different problems in the same system.

<details>
<summary>Answer</summary>

Example answer (parking lot system): a `ParkingSpot` class owning its own occupy/free logic so no other class can directly flip its state — that's encapsulation (protecting who can touch the data). Separately, a `Notification` interface lets a ticket be sent via email, SMS, or push without the caller knowing which channel is used — that's abstraction (hiding implementation behind a simple interface). Same system, two different pillars, two different problems.

</details>

---

## 30-Second Elevator Pitch

> OOP's four pillars: encapsulation hides state and enforces invariants ("who can touch my data"), abstraction hides complexity behind a simple interface ("what do you need to know to use me") — often conflated but answering different questions. Inheritance models is-a and is overused where has-a (composition) would be safer. Polymorphism lets the same call produce different runtime behavior, eliminating type-checking if/else chains — this is the direct mechanism that makes the Open/Closed Principle possible, and it's the foundation the rest of the LLD track (SOLID, patterns, DI) builds on.

---

## Weak Areas to Watch

- None this session — clean pass on all checkpoint questions, including a strong self-constructed example distinguishing encapsulation from abstraction.
