# The Unofficial Guide

Kaden — corpus: `campus_life`.

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

This is a question-answering system over `campus_life`, a corpus of 88 short
student posts about dining halls, dorms, courses, and the administrative
rules nobody explains properly. Ask it something concrete — wait times at a
specific dining hall, a deadline, a course's workload, laundry timing in a
specific building — and it retrieves the posts that actually cover it,
answers using only what's in them, and names the source file. Ask it
something the corpus doesn't cover and it says so instead of guessing.

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

**1.** I asked Claude to design a chunking strategy for campus_life. It
proposed splitting on paragraph breaks with short paragraphs merged into
their neighbor, but left the minimum-length threshold as a choice between
60/100/150 characters. I picked 100 because 150 would have merged real
single-sentence facts (like the CS 210 workload line) into unrelated
neighbors, and 60 barely changed anything from the raw paragraph structure.

**2.** I asked Claude to pressure-test my five criteria by testing each one
from the sentence alone, the way the milestone describes. It flagged that
criterion 4's "complete thought" phrase was a soft judgment call, only
checkable because it's paired with the objective "no sentence cut in half"
clause. It offered to tighten the wording, but I decided the objective clause
already did the real work, so I left criterion 4 as written.

---

# Unit 2

<!-- These sections get ADDED to what's already above. Don't delete or rewrite
     unit 1 — the point is that someone can see what you said before you knew
     how it went. -->

## Run Log — Before

Produced by `run_eval.py::main` (`results/run_2026-09-21_1316_before.md`),
retrieval from `store.py::search` over chunks from
`chunker.py::split_documents`, top-k 5, cutoff 0.6. No `scorer.py` exists yet,
so the Run columns are my own judgment against each question's `expects`
phrase in `questions.py` — the same standard criterion 1 is defined by.
Criteria 3 and 4 don't vary by generation run (the gate is a fixed comparison
against retrieval, and the chunk sample doesn't depend on a question at all),
so the same number is repeated across all three run columns for those two.

| Criterion | Target | Run 1 | Run 2 | Run 3 | Verdict |
|---|---|---|---|---|---|
| 1. Retrieved chunk contains the answer | 4 of 5 | 5/5 | 5/5 | 5/5 | MET |
| 2. Every answer names a source | 5 of 5 | 5/5 | 5/5 | 5/5 | MET |
| 3. Gate stops out-of-corpus questions | 4 of 5 | 5/5 | 5/5 | 5/5 | MET |
| 4. Chunks read as complete thoughts | 4 of 5 | 5/5 | 5/5 | 5/5 | MET |
| 5. Cited source is the correct document | 4 of 5 | 5/5 | 5/5 | 5/5 | MET |

**Criterion 1 & 5 evidence** — from `results/run_2026-09-21_1316_before.md`:

```
### When's the best time to do laundry at Fenwick Court? — run 1
- Best distance: 0.2964 (passed the gate)
- Sources retrieved: housing_aldridge_hall_laundry.txt, housing_fenwick_court_laundry.txt, housing_old_brewhouse_laundry.txt, housing_tamsin_court_laundry.txt, transit_walking.txt

The best time to do laundry at Fenwick Court is Tuesday or Wednesday morning.

Source: housing_fenwick_court_laundry.txt
```

```
### How fast do west lot parking permits sell out? — run 1
- Best distance: 0.2445 (passed the gate)
- Sources retrieved: admin_library_holds.txt, admin_parking_permits.txt, course_stat_150_workload.txt, dining_halden_hall.txt, dining_kestrel_commons.txt

Student permits for the west lots sell out in about three days (admin_parking_permits.txt).
```

Both name the one document that actually contains the `expects` phrase
(`Tuesday`, `three days`), even though 4 other, unrelated chunks were sitting
in the same context window (top-k 5 always fills all 5 slots, regardless of
whether the 2nd–5th results are actually relevant).

**Criterion 2 evidence** — all 15 generated answers (5 questions × 3 runs)
named at least one source file; zero were bare. Sample above shows both
inline-parenthetical and `Source: ...` citation styles, and both count.

**Criterion 3 evidence** — from the same run log:

```
| What is the capital of Mongolia? | 0.825 | refused |
| How do I change the oil in a diesel engine? | 0.923 | refused |
| Who won the 1994 World Cup? | 0.874 | refused |
| What is the recommended dosage of ibuprofen for a headache? | 0.803 | refused |
| How do I write a for loop in Rust? | 0.877 | refused |
```

**Criterion 4 evidence** — 5 chunks sampled from `chunker.py::split_documents`
(random seed 7), read for complete thoughts:

```
=== dining_verrill_street_grill.txt#0 ===
Verrill Street Grill

I'm a junior and I've done this twice now. Wait times: up to 30 minutes on Friday evenings, otherwise under 10. The thing worth going for is the burger, which is the only late-night hot food on campus. The thing to know is that one register, so the queue is a single line no matter how busy.

Hours are 11:00am to 1:00am daily during term. Costs declining balance, or cash after 11:00pm.

=== course_econ_101_workload.txt#1 ===
It's front-loaded — the first month is heavier than the rest, partly because you're learning the format.

=== housing_fenwick_court.txt#2 ===
Laundry costs $2.00 wash, $1.75 dry, app-based. On noise: thin walls between suites; the kitchenettes carry sound.

=== admin_study_abroad.txt#0 ===
On the study abroad

Applications open in October for the following academic year. The financial aid package travels with you, which is the single most misunderstood fact about the programme — most students assume it doesn't and rule themselves out.

=== course_biol_160.txt#0 ===
BIOL 160 Cell Biology

I lived here my sophomore year. Format is lecture three times a week with a weekly lab. Assessment: four unit tests and a cumulative final. Not curved.
```

No sentence is cut in half in any of the 5. Two of them
(`course_econ_101_workload.txt#1`, `housing_fenwick_court.txt#2`) are complete
thoughts but have lost their subject's name — that's not a criterion-4
failure (nothing is cut off), but it's the same pattern Milestone 3 flagged
and it's what the improvement below targets.

## Verdicts

| # | Criterion | Verdict | How I decided |
|---|---|---|---|
| 1 | Retrieved chunk contains the answer | MET | All 5 questions retrieved a chunk containing their `expects` phrase, in all 3 runs — 15/15, held to the same standard on every run since retrieval is deterministic. |
| 2 | Every answer names a source | MET | All 15 generated answers (5 questions × 3 runs) named at least one source filename. No bare answers. |
| 3 | Gate stops out-of-corpus questions | MET | All 5 `OUT_OF_SCOPE` questions were refused (best distances 0.803–0.923, all well past the 0.6 cutoff). |
| 4 | Chunks read as complete thoughts | MET | 5 randomly sampled chunks, none cut off mid-sentence at either end. |
| 5 | Cited source is the correct document | MET | For all 5 questions in all 3 runs, the filename named in the answer text was the document that actually contains the `expects` phrase. |

## Diagnoses

Nothing missed against the letter of any of the five criteria — every one hit
its target on all 3 runs, most at 5/5 rather than the 4/5 I targeted. Given
that, the honest thing to say is that 4 of my 5 targets (1, 3, 4, 5) were
probably set a little low for this corpus: campus_life topics are distinct
enough, and my test questions specific enough, that the pipeline didn't come
close to the numbers I picked. If I were rewriting them now, I'd tighten 1,
4, and 5 to 5 of 5 — criterion 3 I'd leave at 4 of 5, since a gate refusing an
answerable question is a real cost I'd rather guard against than optimize
away.

That doesn't mean the pipeline has no weak points, just that my 5 questions
didn't happen to hit the one I could already name. Milestone 3's chunking
writeup flagged that paragraph-split chunks past the first one (`#1`, `#2`,
...) lose their document's subject name — `course_econ_101_workload.txt#1`
reads "It's front-loaded..." with no mention of ECON 101 anywhere in the
chunk text. The criterion-4 sample above caught two more examples of the same
thing. This is a **chunking-stage** issue: `split_documents` in `chunker.py`
merges short paragraphs forward into their neighbor, but the title paragraph
only ever merges into chunk `#0` — every later chunk from a multi-chunk
document is title-blind.

None of my 5 test questions happened to depend on a title-blind chunk to
answer correctly (their answers all lived in a `#0` chunk, or the doc name
was unambiguous from the retrieved sibling context), which is why criterion 5
still hit 5/5. But it's a latent risk for criterion 5 specifically: a
title-blind chunk about, say, workload, retrieved for a question that's
ambiguous between two courses, gives the model less to go on when deciding
which file to cite. I'd rather fix it now, with a concrete mechanism already
identified, than wait for a broader question set to turn it into an actual
miss.

## The Improvement

**What I changed:** `chunker.py::split_documents` now prepends the
document's title paragraph to every chunk, not just the first one — a chunk
that used to read `"It's front-loaded — the first month is heavier than the
rest..."` now reads `"Workload for ECON 101 Introduction to Economics\n\nIt's
front-loaded..."`.

**Why I picked it:** It's a direct fix for the mechanism in the Diagnoses
section above — title-blind chunks past `#0` — rather than a general-purpose
change. It only touches how a chunk's text is assembled, not the boundaries
between chunks, so it shouldn't change *which* chunks get retrieved, only
what each one contains once it's retrieved.

### Run Log — After

Produced the same way as Before (`results/run_2026-09-21_1321_after.md`),
same 5 questions, same top-k and cutoff, corpus re-indexed with the fixed
`chunker.py::split_documents`.

| Criterion | Target | Run 1 | Run 2 | Run 3 | Verdict |
|---|---|---|---|---|---|
| 1. Retrieved chunk contains the answer | 4 of 5 | 5/5 | 5/5 | 5/5 | MET |
| 2. Every answer names a source | 5 of 5 | 5/5 | 5/5 | 5/5 | MET |
| 3. Gate stops out-of-corpus questions | 4 of 5 | 5/5 | 5/5 | 5/5 | MET |
| 4. Chunks read as complete thoughts | 4 of 5 | 5/5 | 5/5 | 5/5 | MET |
| 5. Cited source is the correct document | 4 of 5 | 5/5 | 5/5 | 5/5 | MET |

Best distances for all 5 test questions are byte-identical to Before (0.296,
0.205, 0.244, 0.264, 0.374) — their answers all live in a document's `#0`
chunk, which the fix never touches, so retrieval didn't move for this
question set. The out-of-scope distances shifted slightly (World Cup 0.874 →
0.886, ibuprofen 0.803 → 0.848) since some sibling `#1+` chunks changed text,
but the gap to the cutoff only widened.

**Did it help?**

Not measurably, against these five criteria and these five questions — every
number is identical before and after, because none of my test questions
depended on a title-blind chunk to answer correctly (I checked this
specifically going in; see Diagnoses). So no, this run log alone can't show
the fix did anything.

But I can show it did something, just not something these criteria were
built to catch. Directly comparing the same two chunks before and after:

```
Before: course_econ_101_workload.txt#1
"It's front-loaded — the first month is heavier than the rest, partly because you're learning the format."

After:  course_econ_101_workload.txt#1
"Workload for ECON 101 Introduction to Economics

It's front-loaded — the first month is heavier than the rest, partly because you're learning the format."
```

The "before" version, retrieved on its own, gives the model no way to know
which course it's about — HIST 118's and CS 340's workload chunks say the
literal same sentence. The "after" version does. That's a real fix for a real
mechanism, verified directly rather than inferred from a score that couldn't
move.

## What's Still Broken

Nothing is missed against the letter of any of my five criteria, before or
after. What's still true is that my 5 test questions never actually exercise
the fix — none of them ask a question ambiguous enough to need a `#1+`
chunk's title to disambiguate between two similar documents (e.g. "what's the
workload like" without naming a course, which would retrieve several
`_workload.txt#1` chunks that — before the fix — were textually identical
apart from their source filename). I didn't add that question to
`questions.py` because Milestone 2 asked me to fix those five before I'd seen
any results, and changing them now to manufacture a criterion-5 near-miss
would be gaming my own test, not diagnosing my system. If I get to it, that's
the next question I'd add.

## What I'd Do Differently

I'd tighten criteria 1, 4, and 5 to 5 of 5 instead of 4 of 5 — all three hit
5/5 on every run in both the before and after logs, which means the target I
set going in was looser than the pipeline turned out to need for this
corpus. I'd leave criterion 3 at 4 of 5; refusing a question the corpus could
actually answer is a real cost, and I'd rather keep margin there than
optimize a number that's already comfortably clear. I'd also add a sixth test
question that's deliberately ambiguous across similar documents (a
workload/hours/cost question that doesn't name the building or course), since
that's the shape of question my one real fix this unit was actually aimed
at, and none of my original five happen to be shaped that way.
