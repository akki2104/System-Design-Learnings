# LLD Session 1 (Part 2 of 2): SOLID Principles

**Track:** LLD — 6-Session Compressed List (Session 1: L001+L002 combined)
**Tier:** 🔴 MUST
**Completed:** 2026-08-13
**Confidence:** 5/5

---

## 1. Why This Matters for Interviews

SOLID is the single most commonly *named* concept in LLD interviews — "is this design SOLID?" is a direct question to expect. Each letter is a specific, checkable rule for keeping a class design flexible as requirements change, which is exactly what a 45-90 min machine-coding round tests (requirements *will* change mid-round). SOLID builds directly on L001's OOP pillars — several letters are only achievable *because* of encapsulation/abstraction/polymorphism.

---

## 2. S — Single Responsibility Principle

**A class should have exactly one reason to change.**
```java
// VIOLATION — two reasons to change: pricing logic AND persistence logic
class Order {
    double calculateTotal() { ... }
    void saveToDatabase() { ... }
}

// FIX — split by responsibility
class Order { double calculateTotal() { ... } }
class OrderRepository { void save(Order o) { ... } }
```
If pricing rules change, only `Order` changes. If the database is swapped, only `OrderRepository` changes. One class, one axis of change.

---

## 3. O — Open/Closed Principle

**Open for extension, closed for modification.** You should be able to add new behavior without editing existing, tested code.
```java
// VIOLATION — adding a new shape means editing this method
double calculateArea(Shape s) {
    if (s.type == "circle") return Math.PI * s.radius * s.radius;
    else if (s.type == "square") return s.side * s.side;
    // every new shape = another else-if, and this method must be re-tested
}

// FIX — polymorphism (L001!) replaces the if/else entirely
interface Shape { double area(); }
class Circle implements Shape { double area() { return Math.PI * r * r; } }
class Square implements Shape { double area() { return side * side; } }
// adding Triangle: write a new class, touch ZERO existing code
```
This is the direct, concrete payoff of L001's polymorphism — Open/Closed is what polymorphism is *for*.

---

## 4. L — Liskov Substitution Principle

**A subclass must be usable anywhere its parent is expected, without breaking behavior.**
```java
class Rectangle { void setWidth(int w) {...} void setHeight(int h) {...} }
class Square extends Rectangle {
    void setWidth(int w) { super.setWidth(w); super.setHeight(w); }  // forces both
    void setHeight(int h) { super.setWidth(h); super.setHeight(h); }  // forces both
}
// Code that does rect.setWidth(5); rect.setHeight(10); expecting a 5x10 rectangle
// silently gets a 10x10 square instead if `rect` is actually a Square — BROKEN substitution
```
`Square` is-a `Rectangle` mathematically, but modeling it that way violates Liskov because a `Square` can't honor a `Rectangle`'s independent width/height contract. **This is the formal test for whether an inheritance relationship from L001 is actually sound** — if substituting the subclass changes correctness, the hierarchy is wrong, no matter how "is-a" it sounds in English.

Another common form: a subclass overriding a parent method to throw an exception or no-op instead of implementing the expected behavior (e.g., a `RubberDuck extends Duck` that overrides `fly()` to throw, since rubber ducks can't fly) — the subclass can no longer stand in for the parent wherever `fly()` is called.

---

## 5. I — Interface Segregation Principle

**Don't force a class to implement methods it doesn't need.** Prefer several small, specific interfaces over one large, general one.
```java
// VIOLATION — a fat interface forces irrelevant methods on every implementer
interface Worker { void work(); void eat(); }
class Robot implements Worker {
    void work() { ... }
    void eat() { throw new UnsupportedOperationException(); }  // robots don't eat!
}

// FIX — segregate
interface Workable { void work(); }
interface Eatable { void eat(); }
class Robot implements Workable { void work() { ... } }
class Human implements Workable, Eatable { ... }
```

---

## 6. D — Dependency Inversion Principle

**High-level modules should depend on abstractions, not on concrete low-level implementations.**
```java
// VIOLATION — OrderService is hard-wired to a specific email implementation
class OrderService {
    private GmailSender sender = new GmailSender();  // concrete dependency
}

// FIX — depend on an abstraction, inject the concrete implementation
interface NotificationSender { void send(String msg); }
class OrderService {
    private NotificationSender sender;
    OrderService(NotificationSender sender) { this.sender = sender; }  // injected
}
```
This is the exact mechanism behind **Dependency Injection** (a full topic later, L014) — DI is simply the *technique* for satisfying Dependency Inversion in practice. Also directly reuses L001's abstraction: `OrderService` depends on the `NotificationSender` interface, never on `GmailSender` concretely — swapping to SMS or Slack later touches zero lines of `OrderService`.

---

## 7. Decision Box: Spotting SOLID Violations Live

```
Class doing too much (pricing + persistence + notifications)?     → violates S
New feature requires editing an existing, tested class's logic?   → violates O
Subclass overrides a method to throw/no-op/change the contract?   → violates L
Interface has methods some implementers must stub out?            → violates I
A class directly constructs (`new X()`) a concrete dependency?     → violates D
```
**Interview sentence:** "I'd inject the `PaymentProcessor` interface into `CheckoutService` rather than hard-coding `StripeProcessor` — that keeps checkout logic closed for modification (O) and lets me swap payment providers by writing a new class, not editing this one (D)."

---

## 8. Worked Example — Report Generator

A `ReportGenerator` class that both formats a report AND emails it to a distribution list has two reasons to change (formatting rules, or notification delivery) — a Single Responsibility violation. Fix: split into `ReportFormatter` (formatting only) and a separate sender abstraction (e.g. `NotificationSender`, not an email-specific class) — naming the sender generically rather than `EmailSender` also keeps the door open for other channels later without another SRP violation resurfacing.

---

## 9. Common Mistakes

| Mistake | Correction |
|---|---|
| Treating SOLID as five independent rules | They compound — O is achieved *through* polymorphism (L001) and correct inheritance (checked by L); D is implemented *via* DI (L014); I supports D by keeping injected interfaces minimal |
| Calling any inheritance "Liskov-safe" just because it's grammatically is-a | Liskov is a behavioral contract check, not a language check — "Square is-a Rectangle" reads fine in English but breaks substitution because it changes the parent's independent-width/height guarantee |
| Applying SOLID everywhere reflexively, even to a throwaway script | SOLID buys flexibility at the cost of more classes/indirection — worth it for code that will actually change; over-engineering a one-off script with 5 interfaces is its own smell (ties to L015 Clean Code) |

---

## 10. Revision Questions
See `Revision/Revision_L002.md`.

## 11. Summary
- **S**ingle Responsibility: one reason to change per class.
- **O**pen/Closed: extend without editing, achieved via polymorphism (L001).
- **L**iskov Substitution: a subclass must honor its parent's full contract — the formal test for whether an L001 inheritance choice was actually sound.
- **I**nterface Segregation: small, focused interfaces over one fat one.
- **D**ependency Inversion: depend on abstractions, not concretions; Dependency Injection (L014) is the technique that implements this.
- The five letters aren't independent — they lean on each other and on L001's OOP pillars (O needs polymorphism, D needs abstraction, L is the formal check on inheritance).
