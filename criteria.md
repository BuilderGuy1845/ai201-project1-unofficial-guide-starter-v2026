# Acceptance criteria — The Unofficial Guide

Five criteria that say what "working" means for this system, written in unit 1
**before** any results existed.

An acceptance criterion names a target: a number, a count, a rate, or something
a person could plainly observe. *"Retrieval works"* is an opinion. *"For at
least 4 of my 5 test questions, the top results include a chunk containing the
answer"* is a criterion.

Under each one, write a sentence or two on **why that target** and not a
stricter or looser one. A reason that says something about your corpus or your
pipeline earns credit; *"80% seemed reasonable"* does not.

> Missing your own targets next unit costs you nothing. Setting a target so
> easy you can't miss it does.

---

## 1. Retrieved chunks contain the answer

For at least 4 of my 5 test questions, the retrieved chunks include one that
contains the answer.

**Why this target:**

One of my questions (Fenwick laundry timing) draws on a single short doc, so
I expect it to be easy, but I set 4/5 instead of 5/5 in case a distance tie
pulls in a sibling laundry doc from a different building.

---

## 2. Every answer names a source

Every answer the system produces names at least one source document.

**Why this target:**

I expect all 5 since generate.py is built to always cite its source chunks —
the only way this fails is a bug, not a hard question.

---

## 3. The relevance gate stops out-of-corpus questions

When I ask a question my documents clearly don't cover, the relevance gate
stops it and the system returns "I don't have enough information about that" —
in at least 4 of 5 tries.

<!-- The five questions are the ones in `OUT_OF_SCOPE` at the bottom of
     `questions.py`, and `run_eval.py` puts them through the gate and writes
     what happened into your run log. Swap them for your own if you'd rather —
     just keep five of them, or the "4 of 5" above has nothing to be 4 of. -->

**Why this target:**

My five test questions topped out at a best distance of 0.374; the five
`OUT_OF_SCOPE` questions never dropped below 0.803 — a wide, clean gap with
nothing near the middle. I kept the default cutoff of 0.6 since it already
sits almost exactly in that gap. I still set 4/5 and not 5/5 because the gate
only guards against distance, not phrasing — the actual boundary case I found
was a question the gate correctly let through (best distance 0.456) where the
documents were silent on part of the answer, which is a different failure
mode from what this criterion measures.

---

## 4. Something about your chunks

At least 4 of 5 sampled chunks read as a complete thought, with no sentence
cut in half at either end.

**Why this target:**

I picked 4 of 5 and not 5 of 5 because the fallback chunker splits on
paragraph/length rules, not sentence boundaries, so a longer doc could still
get cut wrong even if most don't.

---

## 5. Your choice

For at least 4 of my 5 test questions, the source named in the answer is the
document that actually contains the fact — not just any retrieved chunk.

**Why this target:**

I picked 4 of 5 and not 5 of 5 because campus_life has sibling documents on
the same topic (e.g. dining_halden_hall.txt and
dining_halden_hall_followup.txt, or the three housing_*_laundry/_noise files
per building) — it's plausible for the model to cite a related-but-wrong
sibling instead of the doc that actually contains the fact.

---

<!-- ─────────────────────────────────────────────────────────────────────────
     UNIT 2 — read this before you change anything above.

     If a criterion turns out to be BROKEN rather than merely unmet, you can
     revise it, and that earns credit. But never delete or edit the original
     line. Add the revision underneath it, like this:

         ## 1. Retrieved chunks contain the answer

         For at least 4 of my 5 test questions, the retrieved chunks include
         one that contains the answer.

         **Why this target:** ...

         > **Revised in unit 2:** For at least 4 of 5 questions, the top three
         > results contain the answer.
         >
         > **Why revised:** I couldn't judge "the chunks include one that
         > contains the answer" the same way twice — I scored two questions
         > differently on Monday than on Wednesday. The new version is
         > something I can actually check.

     That's a revision because the criterion couldn't be MEASURED.

     Lowering a target because you missed it is not a revision, and it costs
     you the point:

         ✗ "I said 4 of 5 but got 2 of 5, so 2 of 5 is more realistic."

     A number you missed stays where it is, gets diagnosed, and gets a fix
     attempted. That's where the points are.

     The whole reason the originals stay visible is so someone can see what you
     said before you knew the answer.
     ───────────────────────────────────────────────────────────────────────── -->
