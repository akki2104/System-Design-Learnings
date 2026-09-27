# System Design Learnings

> A structured, AI-mentored curriculum taking me from "I know the buzzwords" to interview-ready
> system design — with every lesson, mistake, and revision tracked in the open.

**Learner:** [Akash Yadav](https://github.com/akki2104) · Software Engineer
**Started:** 2026-06-28 · **Last session:** 2026-09-26
**Progress:** 45 of 88 planned topics (~51%) · 1 case study · 1 of 6 LLD sessions

---

## What this repository is

This is not a collection of notes scraped from blog posts. It's a **self-contained learning system**
where an LLM mentor teaches from a fixed constitution, grades my answers honestly, logs every mistake
with its root cause, and schedules spaced-repetition revisions — all committed to git so the entire
learning trajectory is auditable.

Everything here was produced through live teaching sessions. The mistakes are real, the confidence
scores are self-assessed and often unflattering, and the pace warnings are accurate — including the
one that eventually forced a full restructure of the plan.

---

## Progress

### HLD Track

| Module | Topics | Status |
|--------|--------|--------|
| **0 — Orientation & Mental Models** | 001–005 | ✅ Complete |
| **1 — Networking & Communication** | 006–019 | ✅ Complete |
| **2 — Data Storage Foundations** | 020–031 | ✅ Complete |
| **3 — Caching** | 032–037 | ✅ Complete |
| **4 — Scaling & Distributing Data** | 038–044 | ✅ Complete |
| **5 — Distributed Systems Theory** | 045–057 | ⬜ Next |
| **6 — Messaging, Streaming & Async** | 058–066 | ⬜ Not started |
| **7 — Reliability, Resilience & Ops** | 067–079 | ⬜ Not started |
| **8 — Security & Identity** | 080–088 | ⬜ Not started |
| **9 — Specialized Building Blocks** | 089–101 | 🔄 098 done (pulled forward for TinyURL) |
| **10 — Architecture Styles & Delivery** | 102–110 | ⬜ Not started |
| **11 — HLD Capstone Method** | 111–114 | ⬜ Not started |
| **Track R — Technology Deep Dives** | R01 Redis | 📦 Consolidated, ready to learn |

### Parallel Tracks

| Track | Progress | Latest |
|-------|----------|--------|
| **Case Studies** | 1 / 14 | [CS001 — TinyURL / URL Shortener](CaseStudies/CS001_TinyURL.md) *(2026-09-26)* |
| **LLD** | 1 / 6 sessions | [L001 OOP](LLD/L001_OOP_Fundamentals.md) + [L002 SOLID](LLD/L002_SOLID_Principles.md) *(2026-08-13)* |
| **Mock Interviews** | 0 / 3 | — |

46 lesson files · 42 revision files · one commit per completed topic.

---

## How the curriculum works

Seven mechanisms make this different from reading a system design book:

**1. A constitution, not improvisation.**
[`SYSTEM_DESIGN_MASTER_GUIDE.md`](SYSTEM_DESIGN_MASTER_GUIDE.md) governs how teaching happens — the
24-step lesson structure, the 7-dimension scoring rubric, the mastery gate, company-specific flavour
notes. The mentor reads it at the start of every session. It is deliberately hard to deviate from.

**2. Case-study-driven, just-in-time theory.**
The current sequencing model (**Track v3**, Master Guide §0.2) inverts the usual order. Instead of
"learn all the theory, then do case studies," the mentor identifies only the concepts a *specific*
case study depends on, teaches those as full lessons, then runs the case study as a learner-driven
whiteboard session. Theory earns its place by being needed. This replaced the earlier
theory-first ordering after it produced 60% theory coverage and zero case-study reps.

**3. Explain fully, then test.**
Lessons run uninterrupted end-to-end, with all recall questions at the end. This was a correction I
requested after mid-lesson quizzing kept breaking my grasp of what was coming next.

**4. Every mistake is logged with a root cause.**
[`InterviewMistakes.md`](InterviewMistakes.md) records what I got wrong, *why* it was wrong, the
correct understanding, and a memory hook. Mistakes that recur get flagged as persistent weak areas and
force a change in teaching method — not just a repeat explanation. Three concepts have failed revision
three times each and are tracked accordingly.

**5. Spaced repetition with real failure.**
[`RevisionSchedule.md`](RevisionSchedule.md) queues each completed topic at +1, +3, +7, +15, +30, +60,
+90 days. Failed revisions reset to +1 day. Revision sessions are scored and the results recorded,
including the ones I bombed.

**6. Priority tiers, so nothing is studied by default.**
[`TopicPriority.md`](TopicPriority.md) rates every remaining topic 🔴 MUST / 🟡 SKIM / ⚫ SKIP with a
time estimate, complexity rating, and the schedule cost of overriding the recommendation. Before each
topic the mentor shows a briefing card and I decide. Deviations are logged with their cost. Tier
affects *session depth only* — every topic still gets a complete lesson file and revision file.

**7. A mastery gate that actually blocks.**
A topic moves from *Completed* to *Mastered* only on a 30-second elevator pitch from memory, 4/5
active-recall answers without notes, and drawing the key diagram from scratch.

---

## Repository structure

| Path | Contents |
|------|----------|
| [`SYSTEM_DESIGN_MASTER_GUIDE.md`](SYSTEM_DESIGN_MASTER_GUIDE.md) | The constitution — teaching methodology, 114-topic roadmap, scoring rubric, mastery rules, sequencing tracks |
| [`Progress.md`](Progress.md) | Live dashboard, per-topic status, confidence scores, weak areas, full session log |
| [`Schedule.md`](Schedule.md) | Compressed plan (v2) — now a scope/content reference rather than a sequencing rule |
| [`TopicPriority.md`](TopicPriority.md) | Tier / time / complexity / skip-cost for every remaining topic, plus the Deviation Log |
| [`Topics/`](Topics/) | One full lesson per HLD topic, plus Track-R technology deep dives — 46 files |
| [`Revision/`](Revision/) | Active-recall Q&A per topic, with collapsible answers — 42 files |
| [`CaseStudies/`](CaseStudies/) | Guided whiteboard sessions, learner-driven, with honest scoring |
| [`LLD/`](LLD/) | Low-level design track — OOP, SOLID, patterns, concurrency |
| [`InterviewMistakes.md`](InterviewMistakes.md) | Every mistake with root cause, fix, and mnemonic |
| [`CheatSheets.md`](CheatSheets.md) | One-screen condensed summary per topic |
| [`Glossary.md`](Glossary.md) | Every term introduced, linked to its source topic |
| [`TechChoices.md`](TechChoices.md) | "When to use what" decision playbook — problem → tech → why → why not the alternatives |
| [`Numbers.md`](Numbers.md) | Latency, capacity, and estimation reference card |
| `InterviewPractice/`, `Assessments/` | Reserved for mock interviews and milestone assessments |

---

## Sample lessons

Reasonable places to start if you're browsing:

- [**CS001 — TinyURL / URL Shortener**](CaseStudies/CS001_TinyURL.md) — the first full case study, run as a learner-driven whiteboard session with honest per-dimension scoring
- [**023 — B-Trees vs LSM-Trees**](Topics/023_B_Trees_vs_LSM_Trees.md) — write-vs-read optimization, compaction, and a misconception I had to have corrected mid-lesson
- [**027 — MVCC**](Topics/027_MVCC.md) — why readers never block writers, and the precise reason MVCC alone does *not* prevent lost updates
- [**042 — Consistent Hashing**](Topics/042_Consistent_Hashing.md) — virtual nodes, rebalancing, and why `hash % N` falls apart
- [**026 — Concurrency Control**](Topics/026_Concurrency_Control_Locks_2PL_Deadlocks.md) — 2PL, cascading rollback, and the lock-upgrade deadlock trap
- [**R01 — Redis, consolidated**](Topics/R01_Redis_Consolidated_Module.md) — one authoritative pass over Redis: data structures by use case, Redis *beyond* caching, replication vs Sentinel vs Cluster, and what actually breaks when it dies

Each lesson pairs with a [revision file](Revision/) containing active-recall questions, a 30-second
elevator pitch, and the specific weak areas to watch.

---

## Current status, honestly

**The plan has been restructured twice, both times because the arithmetic stopped working.**

The original linear calendar (June 28 → Aug 9) assumed no gaps and collapsed almost immediately.
The v2 compressed track (adopted 2026-07-25) cut 26 low-yield topics, compressed 26 more to skim
level, and pushed the target to Aug 18. On Aug 17, that target arrived with theory ~60% done but
**case studies at 0/14 and mocks at 0/3** — real study pace had run roughly 7× slower than the
2.25 hrs/day the plan assumed.

The diagnosis was that the bottleneck was never remaining theory; it was zero case-study reps. So
v3 (2026-08-17) inverted the sequencing entirely: pick a case study, teach only its genuine
prerequisites just-in-time, run the case study, repeat. **No fixed target date is currently set** —
progress is now tracked in hours-of-work-remaining (~44 hrs) rather than against a calendar, until a
real interview-driven deadline exists.

Current known gaps, all tracked in [`InterviewMistakes.md`](InterviewMistakes.md):

- Three concepts have failed spaced repetition three times each and are being drilled with changed
  teaching methods rather than re-explained
- Back-of-envelope estimation remains the most persistent weak area — it resurfaced during CS001
- CS001 surfaced a pattern worth watching: plausible-but-imprecise first-pass answers on mechanism
  questions, all self-corrected the same turn once named precisely. Whether that self-check becomes
  automatic is the thing to watch in Case Study #2

---

## Tech

Sessions are run with [Claude Code](https://claude.com/claude-code). The mentor reads the tracking
files at session start, teaches, grades, updates every affected file, and commits — one commit per
completed topic, so the git history doubles as a learning log.

```bash
git log --oneline --grep="^Topic"        # every topic completion, in order
git log --oneline --grep="^Case Study"   # case study sessions
```

---

*This is a personal learning repository. The curriculum is calibrated for a Software Engineer with
~1–2 years of experience targeting product companies. If you find it useful, feel free to fork the
structure — the master guide is written to be reusable by any learner or LLM mentor.*
