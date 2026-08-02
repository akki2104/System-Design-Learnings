# System Design Learnings

> A structured, AI-mentored curriculum taking me from "I know the buzzwords" to interview-ready
> system design — with every lesson, mistake, and revision tracked in the open.

**Learner:** [Akash Yadav](https://github.com/akki2104) · Software Engineer
**Started:** 2026-06-28 · **Target:** 2026-08-18
**Progress:** 34 of 88 planned topics complete (~39%)

---

## What this repository is

This is not a collection of notes scraped from blog posts. It's a **self-contained learning system**
where an LLM mentor teaches from a fixed constitution, grades my answers honestly, logs every mistake
with its root cause, and schedules spaced-repetition revisions — all committed to git so the entire
learning trajectory is auditable.

Everything here was produced through live teaching sessions. The mistakes are real, the confidence
scores are self-assessed and often unflattering, and the pace warnings are accurate.

---

## Progress

| Module | Topics | Status |
|--------|--------|--------|
| **0 — Orientation & Mental Models** | 001–005 | ✅ Complete |
| **1 — Networking & Communication** | 006–019 | ✅ Complete |
| **2 — Data Storage Foundations** | 020–031 | ✅ Complete |
| **3 — Caching** | 032–037 | 🔄 In progress (3/6) |
| **4 — Scaling & Distributing Data** | 038–044 | ⬜ Not started |
| **5 — Distributed Systems Theory** | 045–057 | ⬜ Not started |
| **6 — Messaging, Streaming & Async** | 058–066 | ⬜ Not started |
| **7 — Reliability, Resilience & Ops** | 067–079 | ⬜ Not started |
| **8 — Security & Identity** | 080–088 | ⬜ Not started |
| **9 — Specialized Building Blocks** | 089–101 | ⬜ Not started |
| **10 — Architecture Styles & Delivery** | 102–110 | ⬜ Not started |
| **11 — HLD Capstone Method** | 111–114 | ⬜ Not started |

Plus a parallel LLD track (6 sessions), 14 case studies, and 3 mock interviews — all pending.

---

## How the curriculum works

Six mechanisms make this different from reading a system design book:

**1. A constitution, not improvisation.**
[`SYSTEM_DESIGN_MASTER_GUIDE.md`](SYSTEM_DESIGN_MASTER_GUIDE.md) governs how teaching happens — the
24-step lesson structure, the 7-dimension scoring rubric, the mastery gate, company-specific flavour
notes. The mentor reads it at the start of every session. It is deliberately hard to deviate from.

**2. Explain fully, then test.**
Lessons run uninterrupted end-to-end, with all recall questions at the end. This was a correction I
requested after mid-lesson quizzing kept breaking my grasp of what was coming next.

**3. Every mistake is logged with a root cause.**
[`InterviewMistakes.md`](InterviewMistakes.md) records what I got wrong, *why* it was wrong, the
correct understanding, and a memory hook. Mistakes that recur get flagged as persistent weak areas and
force a change in teaching method — not just a repeat explanation. Three topics have failed revision
three times each and are tracked accordingly.

**4. Spaced repetition with real failure.**
[`RevisionSchedule.md`](RevisionSchedule.md) queues each completed topic at +1, +3, +7, +15, +30, +60,
+90 days. Failed revisions reset to +1 day. Revision sessions are scored and the results recorded,
including the ones I bombed.

**5. Priority tiers, so nothing is studied by default.**
[`TopicPriority.md`](TopicPriority.md) rates every remaining topic 🔴 MUST / 🟡 SKIM / ⚫ SKIP with a
time estimate, complexity rating, and the schedule cost of overriding the recommendation. Before each
topic the mentor shows a briefing card and I decide. Deviations are logged with their day-cost.

**6. A mastery gate that actually blocks.**
A topic moves from *Completed* to *Mastered* only on a 30-second elevator pitch from memory, 4/5
active-recall answers without notes, and drawing the key diagram from scratch.

---

## Repository structure

| Path | Contents |
|------|----------|
| [`SYSTEM_DESIGN_MASTER_GUIDE.md`](SYSTEM_DESIGN_MASTER_GUIDE.md) | The constitution — teaching methodology, 114-topic roadmap, scoring rubric, mastery rules |
| [`Progress.md`](Progress.md) | Live dashboard, per-topic status, confidence scores, weak areas, full session log |
| [`Schedule.md`](Schedule.md) | Day-by-day compressed plan (v2), rewritten from actual position |
| [`TopicPriority.md`](TopicPriority.md) | Tier / time / complexity / skip-cost for every remaining topic |
| [`Topics/`](Topics/) | One full lesson per topic — 34 files so far |
| [`Revision/`](Revision/) | Active-recall Q&A per topic, with collapsible answers — 28 files |
| [`InterviewMistakes.md`](InterviewMistakes.md) | Every mistake with root cause, fix, and mnemonic |
| [`CheatSheets.md`](CheatSheets.md) | One-screen condensed summary per topic |
| [`Glossary.md`](Glossary.md) | Every term introduced, linked to its source topic |
| [`TechChoices.md`](TechChoices.md) | "When to use what" decision playbook — problem → tech → why → why not the alternatives |
| [`Numbers.md`](Numbers.md) | Latency, capacity, and estimation reference card |
| `LLD/`, `CaseStudies/`, `InterviewPractice/`, `Assessments/` | Reserved for upcoming tracks |

---

## Sample lessons

Reasonable places to start if you're browsing:

- [**023 — B-Trees vs LSM-Trees**](Topics/023_B_Trees_vs_LSM_Trees.md) — write-vs-read optimization, compaction, and a misconception I had to have corrected mid-lesson
- [**027 — MVCC**](Topics/027_MVCC.md) — why readers never block writers, and the precise reason MVCC alone does *not* prevent lost updates
- [**026 — Concurrency Control**](Topics/026_Concurrency_Control_Locks_2PL_Deadlocks.md) — 2PL, cascading rollback, and the lock-upgrade deadlock trap
- [**012 — REST API Design**](Topics/012_REST_API_Design.md) — cursor vs. offset pagination as a *correctness* fix, not a performance one

Each lesson pairs with a [revision file](Revision/) containing active-recall questions, a 30-second
elevator pitch, and the specific weak areas to watch.

---

## Current status, honestly

Running roughly 4–5 days behind the compressed schedule as of 2026-08-02. The plan was rewritten once
already (2026-07-25) when the original calendar proved unachievable — 87 remaining topics in 15 days
was arithmetic that didn't work. The revised approach cuts 26 low-yield topics entirely, compresses 26
more to skim-level, and protects the case-study block, which is the highest-value remaining work.

Three concepts have failed spaced-repetition three times each and are the current focus of targeted
drilling rather than re-explanation.

---

## Tech

Sessions are run with [Claude Code](https://claude.com/claude-code). The mentor reads the tracking
files at session start, teaches, grades, updates every affected file, and commits — one commit per
completed topic, so the git history doubles as a learning log.

```bash
git log --oneline --grep="^Topic"   # every topic completion, in order
```

---

*This is a personal learning repository. The curriculum is calibrated for a Software Engineer with
~1–2 years of experience targeting product companies. If you find it useful, feel free to fork the
structure — the master guide is written to be reusable by any learner or LLM mentor.*
