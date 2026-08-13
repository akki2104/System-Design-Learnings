# LLD Session 1 (Part 1 of 2): OOP Fundamentals

**Track:** LLD — 6-Session Compressed List (Session 1: L001+L002 combined)
**Tier:** 🔴 MUST
**Completed:** 2026-08-13
**Confidence:** 5/5

---

## 1. Why This Matters for Interviews

Every LLD/machine-coding round ("design a parking lot," "design an elevator system," "make this thread-safe") is graded on whether the class design is *actually* object-oriented, not just "code that works." Interviewers specifically probe whether you reach for the right OOP tool — encapsulation to hide state, inheritance vs composition to model relationships, polymorphism to avoid a wall of `if/else` on type. This is the base layer everything else in the LLD track (patterns, SOLID, concurrency, machine-coding) sits on.

---

## 2. The Four Pillars

### Encapsulation
Bundling data with the methods that operate on it, and hiding internal state behind a controlled interface.
```java
class BankAccount {
    private double balance;          // hidden — no direct access
    public void withdraw(double amt) {
        if (amt > balance) throw new InsufficientFundsException();
        balance -= amt;               // only this class can mutate balance
    }
}
```
The point isn't just "make fields private" — the class *owns* the rules for how its own state can change. A caller can never put a `BankAccount` into an invalid state (negative balance) because the only door in is a method that enforces the invariant.

### Abstraction
Exposing *what* something does while hiding *how* it does it. Often confused with encapsulation, but they answer different questions: encapsulation protects state (an implementation concern), abstraction simplifies the interface (a design concern).
```java
interface PaymentProcessor {
    void processPayment(double amount);   // WHAT — no HOW visible
}
class StripeProcessor implements PaymentProcessor { ... }
class PaypalProcessor implements PaymentProcessor { ... }
```
Calling code depends on `PaymentProcessor`, never on `StripeProcessor` directly — this is what makes swapping payment providers later a non-breaking change.

### Inheritance
A class acquiring the fields/methods of a parent class, modeling an **is-a** relationship (`Car is-a Vehicle`).
```java
class Vehicle { void startEngine() { ... } }
class Car extends Vehicle { }   // Car IS-A Vehicle
```
**The trap:** inheritance is overused constantly. If the relationship is actually **has-a** (a `Car` *has an* `Engine`, doesn't *is-a* `Engine`), that's composition, not inheritance — using inheritance there creates a fragile hierarchy that breaks when requirements shift. This is why L007's "composition over inheritance" principle exists, and why Liskov Substitution (L002) exists as a formal check on whether an inheritance relationship is actually sound.

### Polymorphism
The same interface behaving differently depending on the actual runtime type. Two flavors:
- **Runtime (dynamic) polymorphism** — method overriding; the actual implementation is picked based on the object's real type at runtime.
- **Compile-time (static) polymorphism** — method overloading; the compiler picks the right method based on argument types at compile time.
```java
for (PaymentProcessor p : processors) {
    p.processPayment(100);   // runtime picks Stripe's or Paypal's actual code
}
```
This is what eliminates a wall of `if (type == "stripe") ... else if (type == "paypal") ...` — a large interview signal, since that `if/else` chain is exactly what the Open/Closed Principle (L002) forbids.

---

## 3. Worked Example — Parking Lot (Encapsulation vs Abstraction)

A `ParkingSpot` class owns its own fill/free logic — no other class can directly flip its occupied state; only `ParkingSpot.occupy()` / `ParkingSpot.free()` can (**encapsulation** — protecting who's allowed to touch its data). Separately, a `Notification` interface lets a ticket be sent via email, SMS, or push without the calling code knowing which channel is actually used (**abstraction** — hiding implementation behind a simple interface). Same system, two different pillars solving two different problems: one is about *ownership of state*, the other is about *hiding complexity behind an interface*.

---

## 4. Common Mistakes

| Mistake | Correction |
|---|---|
| Treating encapsulation and abstraction as the same thing | Encapsulation = hiding *state* (an implementation detail, "who can touch my data"); abstraction = hiding *complexity* behind a simple interface (a design detail, "what do you need to know to use me"). A class can have one without the other. |
| Defaulting to inheritance for any shared behavior | Ask "is this truly is-a, or is it has-a / behaves-like?" Composition is usually more flexible and is the safer default per L007. |
| Confusing overloading with overriding | Overloading = same method name, different parameters, resolved at compile time. Overriding = subclass redefines a parent's method, resolved at runtime. Different mechanisms, different polymorphism types. |

---

## 5. Revision Questions
See `Revision/Revision_L001.md`.

## 6. Summary
- **Encapsulation**: hide state, enforce invariants through controlled methods — "who can touch my data."
- **Abstraction**: hide complexity behind a simple interface — "what do you need to know to use me." Different from encapsulation even though often conflated.
- **Inheritance**: models is-a; overused constantly where has-a (composition) would be safer and more flexible.
- **Polymorphism**: same call, different runtime behavior — the mechanism that eliminates type-checking if/else chains and directly enables the Open/Closed Principle (L002).
- These four pillars are the foundation SOLID (L002) is built on top of — Open/Closed needs polymorphism, and Liskov Substitution is the formal test for whether an inheritance choice was actually sound.
