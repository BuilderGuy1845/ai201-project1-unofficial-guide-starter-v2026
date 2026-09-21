# The Unofficial Guide

<!-- Replace this line with your name and which corpus you picked. -->

> **This file is your submission.** Fill it in as you go — most sections get
> written during the milestone that produces them, not at the end.
>
> How the starter works, and every command you'll need, is in `RUNNING.md`.
> Leave that file alone.
>
> **Paste everything as text.** No screenshots, no video. A typed table gets
> full credit; a picture of the same table gets none.
>
> Delete these instruction blocks as you replace them. The `<!-- -->` comments
> are notes to you and don't show up when the page renders — you can leave them
> or remove them.

---

# Unit 1

## What This Does

<!-- Three or four sentences. Which corpus you picked, and the kinds of
     questions your system answers. Write it for someone who has never seen
     this repo.

     Milestone 5. -->

## Chunking Strategy

**Chunk size:** Paragraph-based, not a fixed character count — split on blank
lines, with a 100-character minimum before a paragraph gets merged into its
neighbor.
**Overlap:** 0 (unused by this strategy — see below).

The starter's 800-character window never split anything in campus_life: 88
documents went in, 88 chunks came out, because almost no post reaches 800
characters. But reading documents in Milestone 1 (`housing_old_brewhouse.txt`
is a good example) showed that a post isn't really one thought — it's a title
plus several short, single-topic paragraphs: background, the good, the bad,
laundry+noise. A question about heating only needs "the bad" paragraph, not
the other three glued to it. Treating the whole post as one chunk was
diluting exactly the kind of specific question this corpus is good at
answering.

So `split_documents` (in `chunker.py`) splits on blank lines instead, which
never cuts a sentence in half the way a fixed character window can. The one
risk that introduces is a fragment — a bare title line or a one-clause stub
standing alone as its own useless chunk, the same failure mode that produced
`advice_threads`' 2-character tail chunk under the fallback chunker. The
100-character minimum (repurposing `config.CHUNK_SIZE`) merges anything that
short into its neighbor, which is why the title never survives as a separate
chunk. `CHUNK_OVERLAP` is unused because paragraph breaks are already natural,
sentence-complete boundaries — there's no shared context to carry across them
the way there is with a character window.

Re-indexing with this strategy turned 88 chunks into 143 (average 194
characters, range 100–409) — evidence that documents really were holding more
than one topic, not just a chunker producing more output for its own sake.

## Sample Chunks

<!-- Five chunks, pasted as text. Label each one and name the file it came from
     AND the function that produced it — the grader checks your code against
     what you claim here.

     `python app.py chunks -n 5` prints all three for you. Copy them straight
     across.

     Milestone 3. -->

**Chunk 1** — source: `admin_grade_appeals.txt#0` — produced by: `chunker.py::split_documents`

```
On the grade appeals

A grade appeal starts with the instructor and has to be raised within fifteen days of the grade posting. Only after that does it go to the department. Skipping the instructor step gets the appeal returned, which wastes most of the fifteen days.
```

**Chunk 2** — source: `course_cs_210.txt#1` — produced by: `chunker.py::split_documents`

```
Expect 8 to 10 hours a week outside class.

The one piece of advice: do the labs even though they're only 10% — the exams reuse the lab problems.
```

**Chunk 3** — source: `housing_morrow_house.txt#1` — produced by: `chunker.py::split_documents`

```
The good: cheapest housing tier by about $900 a year, and the singles are real singles.

The bad: known damp problem on the ground floor; two rooms were taken offline in 2024.
```

**Chunk 4** — source: `dining_kestrel_commons.txt#1` — produced by: `chunker.py::split_documents`

```
Hours are 7:00am to 9:00pm weekdays, 9:00am to 8:00pm weekends. Costs one meal swipe, or $12.50 cash.
```

**Chunk 5** — source: `dining_verrill_street_grill_followup.txt#1` — produced by: `chunker.py::split_documents`

```
Also worth saying: one register, so the queue is a single line no matter how busy. Nobody tells you this at orientation.
```

## Sample Answer

**Question:** What happens if I drop a class after week two?

**Answer:**

```
If you drop a class after week two, it shows as a W on your transcript (admin_add_drop_deadline.txt).

Sources retrieved: admin_add_drop_deadline.txt, admin_grade_appeals.txt, admin_pass_fail_option.txt, admin_withdrawal_deadline.txt, course_stat_150.txt
```

Worth noting: top-k=5 means the model was handed four chunks that weren't
actually relevant (pass/fail, grade appeals, a course workload line) alongside
the one that was. It still answered from only `admin_add_drop_deadline.txt`
and didn't blend in the unrelated material — the grounding instruction held
even with noise in the context, not just when the context was clean.

**My relevance cutoff:** 0.6 (the starter default — I measured my own two
groups and it already sat almost exactly in the gap, so I kept it rather than
moving it for its own sake).

My five test questions topped out at 0.374 best distance; my five
`OUT_OF_SCOPE` questions never dropped below 0.803. That's a wide, clean gap
with nothing near the middle, so 0.6 isn't a fragile choice — it would still
separate the two groups correctly anywhere roughly between 0.45 and 0.7. I
also tried a harder case than either group: "Does dropping a class after week
two hurt my GPA?" (best distance 0.456, so it clears the gate) — the docs
don't actually say, and the model correctly answered "there is no mention of
whether dropping a class after week two affects your GPA" instead of guessing
from what it already knows about how GPA usually works. That's the second
grounding layer doing its job on a question the gate alone would have let
through.

| Question | In corpus? | Best distance |
|---|---|---|
| When's the best time to do laundry at Fenwick Court? | Yes | 0.296 |
| How fast do west lot parking permits sell out? | Yes | 0.244 |
| Do dining dollars roll over to the next school year? | Yes | 0.264 |
| What are wait times like at Halden Hall dining? | Yes | 0.205 |
| What happens if I drop a class after week two? | Yes | 0.374 |
| What is the capital of Mongolia? | No | 0.825 |
| How do I change the oil in a diesel engine? | No | 0.923 |
| Who won the 1994 World Cup? | No | 0.874 |
| What is the recommended dosage of ibuprofen for a headache? | No | 0.803 |
| How do I write a for loop in Rust? | No | 0.877 |

## How I Used AI

<!-- Two specific moments. For each: what you asked for, what came back, and
     what you changed about it.

     "I asked Claude to write the chunking function from my notes. It ignored
     the overlap, so I added that myself" is the level of detail we're after.
     "I used AI to help me code" is not.

     Milestone 5. -->

**1.**

**2.**

<!-- ── Stretch features ─────────────────────────────────────────────────────
     Doing one? Say so here BEFORE you start. A feature this README never
     claims earns nothing.
     ───────────────────────────────────────────────────────────────────────── -->

---

# Unit 2

<!-- These sections get ADDED to what's already above. Don't delete or rewrite
     unit 1 — the point is that someone can see what you said before you knew
     how it went. -->

## Run Log — Before

<!-- Your five criteria, three runs each. `python run_eval.py --label before`
     runs the questions, puts the OUT_OF_SCOPE ones through the gate, and
     writes it all into results/ for you. Targets come from criteria.md; the
     verdict column is your call.

     Criterion 3 is measured in one deterministic pass rather than three, so
     the same number goes in all three run columns. That's correct, not lazy.

     Milestone 1. -->

| Criterion | Target | Run 1 | Run 2 | Run 3 | Verdict |
|---|---|---|---|---|---|
| 1. Retrieved chunk contains the answer | 4 of 5 |  |  |  |  |
| 2. Every answer names a source | 5 of 5 |  |  |  |  |
| 3. Gate stops out-of-corpus questions | 4 of 5 |  |  |  |  |
| 4. | | | | | |
| 5. | | | | | |

<!-- Underneath, paste the REAL output for each criterion from one of your
     runs — the actual text your system produced, not a description of it.
     Name the file and function that produced it. -->

## Verdicts

<!-- MET or MISSED for each of the five, against the target you wrote last
     unit — not a new one. Plus a sentence on how you decided. That sentence
     matters most where it was close.

     If your target said 4 of 5 and your runs came out 4, 3, 4, that's a MISS.
     The target has to hold, not show up occasionally.

     Milestone 2. -->

| # | Criterion | Verdict | How I decided |
|---|---|---|---|
| 1 |  |  |  |
| 2 |  |  |  |
| 3 |  |  |  |
| 4 |  |  |  |
| 5 |  |  |  |

## Diagnoses

<!-- For each miss: which stage caused it, and how. The stage alone isn't
     enough — you need the mechanism.

     Not a diagnosis: "Question 3 didn't work."
     A diagnosis:     "Question 3 asks about laundry costs. The answer is in
                       one sentence that got split across two chunks, so
                       neither chunk on its own contains it."

     The five stages: loading → chunking → embedding → retrieval → generation.

     Look for a pattern. If three misses all ask about numbers, that's one
     problem, not three.

     Missed nothing? Say so, then say honestly whether your targets were set
     low, and which one you'd tighten and to what.

     Milestone 3. -->

## The Improvement

**What I changed:**

**Why I picked it:**

<!-- Connect it to a specific diagnosis above in one sentence. If you can't,
     you picked a fix because it sounded impressive. -->

### Run Log — After

<!-- Same format, same five criteria, three runs each.
     `python run_eval.py --label after` -->

| Criterion | Target | Run 1 | Run 2 | Run 3 | Verdict |
|---|---|---|---|---|---|
| 1. Retrieved chunk contains the answer | 4 of 5 |  |  |  |  |
| 2. Every answer names a source | 5 of 5 |  |  |  |  |
| 3. Gate stops out-of-corpus questions | 4 of 5 |  |  |  |  |
| 4. | | | | | |
| 5. | | | | | |

**Did it help?**

<!-- Say plainly whether it did, and how you know. If it made things worse,
     say that — a change that backfired, honestly reported, earns full credit
     and is more interesting than one that worked. What matters is that you can
     tell.

     Milestone 4. -->

## What's Still Broken

<!-- For each criterion still missed after your fix: what you'd do about it,
     and why you stopped where you did.

     "I ran out of time" is fine if it's true. Pretending nothing is left is
     not.

     Milestone 5. -->

## What I'd Do Differently

<!-- Knowing what you know now — which of your five criteria would you write
     differently, and why?

     Milestone 5. -->
