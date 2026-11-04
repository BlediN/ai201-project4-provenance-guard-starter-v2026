# Acceptance criteria — Provenance Guard

Five criteria that say what "working" means for this service, written in unit 7
**before** you built it.

**All five are yours.** None are given.

An acceptance criterion names a number or something plainly observable. *"The
detector is accurate"* is an opinion — and perfect AI detection isn't possible
anyway, so it's an opinion about something nobody can deliver. *"Of 10 pieces I
wrote myself, at most 1 is labelled high-confidence AI"* is a criterion.

Under each, write a sentence or two on **why that target**. *"I allow at most 1
false positive in 10 because on a writing site a wrong accusation costs more
than a missed detection"* is a real answer.

> Missing your own targets next unit costs you nothing. Setting a target so
> easy you can't miss it does.

---

## Pick numbers you can defend

**Cover at least three of these five areas.** They're prompts, not a form.

| Area | A question it could answer |
|---|---|
| False positives | How often may a human's writing get called AI? |
| Score spread | How far apart should clearly-AI and clearly-human text score? |
| Label coverage | Is every one of your three labels actually reachable? |
| The appeal path | Does an appeal always change status and always get recorded? |
| Bad input | What happens when `text` is missing, empty, or enormous? |

Two things worth knowing before you pick numbers:

- **The expensive mistake is one-sided.** Missing some AI text costs the site a
  little. Accusing a real writer costs that person a lot. A criterion that
  treats both errors as equally bad is describing a system nobody would ship.
- **Unit 8 tests across three trials, and the target has to hold across all
  three.** Your first signal runs a model, so identical input can score
  differently twice. A target of "at most 1 in 10" against trials of 2, 1, 2 is
  a **miss**.

---

### What I measured before writing these

Before setting any number I ran signal one (`python detector.py "<text>"`) on
four inputs. Signal two isn't built yet.

| Input | Who wrote it | Perplexity | `model_signal` |
|---|---|---|---|
| Built-in sample ("The implementation of the proposed framework…") | AI-style | 12.5 | 0.886 |
| Short essay on AI ("…it is important to note…") | AI-style | 16.7 | 0.830 |
| Plain, second-language style ("I moved to this city two years ago…") | Human | 14.3 | **0.862** |
| Casual ("Honestly the bus was late again…") | Human | 67.9 | 0.341 |

Plain human writing scores *between* the two AI texts. Signal one alone can't
tell them apart, so criteria 1 and 2 are not going to pass by accident.

---
## 1. False positives on human writing

Of **10 human-written pieces** (at least 4 of them in plain, simple style:
short sentences and common words), **at most 1** is labelled high-confidence
AI, and that has to hold in **each of the three trials**.

**Why this target:**
On a writing site, a wrong accusation costs the writer far more than a missed
detection costs the site. That person may have no way to prove they wrote it.
1 in 10 is strict but not zero, because perfect detection isn't possible. I
require 4 plain pieces because that's where signal one fails (0.862 above). A
test set of only casual, quirky writing would make this target easy to hit and
meaningless.

---

## 2. Score spread between clearly-AI and clearly-human text

The average combined score of **5 clearly AI-generated texts** is **at least
0.25 higher** than the average combined score of **5 clearly human texts**,
in each of the three trials. The human set must include at least 2 plain-style
pieces.

**Why this target:**
If the two groups score close together, the thresholds just pick a point in
noise, and every label is close to a coin flip. A gap of 0.25 is enough to fit
an "unsure" band between them. Signal one alone gives about 0.52 between AI
text and casual human text, but about 0 between AI text and plain human text.
So hitting 0.25 on a mixed human set depends on signal two pulling its weight.

---

## 3. All three labels are reachable, and AI text still gets caught

On a set of **9 texts** (3 clearly AI, 3 clearly human, 3 that I judge
borderline), **each of the three labels appears at least once**, and **at
least 2 of the 3 clearly AI texts** are labelled high-confidence AI, in each
of the three trials.

**Why this target:**
A label that can never appear is dead code dressed up as a feature. The config
notes warn that a weighted average can trap scores in the middle. The second
half stops me "solving" criterion 1 by widening the unsure band until nothing
is ever called AI. That would pass criterion 1 and make the service useless.

---

## 4. The appeal path always works

For **5 out of 5** appeals against real `content_id`s, `POST /appeal` returns
`200` with `status: "under_review"`, **and** a matching appeal entry, with the
writer's reasoning, appears in `GET /log`. For **3 out of 3** appeals with a
made-up `content_id`, it returns `404` and never `200` or `500`.

**Why this target:**
The appeal is the only thing between a wrong accusation and nothing at all, so
it has to work every time. Anything less than 5/5 means some writers are
silently ignored. The 404 half matters because a typo that "succeeds" looks
fine to the writer while nothing was recorded.

---

## 5. Bad input is refused cleanly

For **4 malformed requests** (`text` empty, `text` missing, `text` set to
`null`, and `text` of about 250,000 characters), **4 out of 4** return a `4xx`
status with a JSON `error` field, **0** return `500`, and each answers in
**under 2 seconds**.

**Why this target:**
A crash on malformed input is one of the most common real failures, and the
unit 8 attack set will send exactly these. The 2-second limit proves bad input
is rejected *before* the model runs. Feeding the model 250,000 characters
would be slow and pointless, and it's an easy way to tie up the service.


---

<!-- ─────────────────────────────────────────────────────────────────────────
     UNIT 8 — read this before you change anything above.

     If a criterion turns out to be BROKEN rather than merely unmet, you can
     revise it, and that earns credit. But never delete or edit the original
     line. Add the revision underneath, like this:

         ## 2. Uncertain cases land in the unsure band

         The system handles uncertain cases well.

         **Why this target:** ...

         > **Revised in unit 8:** Text I judged borderline scores between
         > 0.4 and 0.6 in at least 4 of 5 cases.
         >
         > **Why revised:** "well" gave me nothing to check. I couldn't
         > reach a verdict from it at all.

     That's a revision because the criterion couldn't be MEASURED.

     Lowering a target because you missed it is not a revision, and it costs
     you the point:

         ✗ "At most 1 of 10 of my own pieces flagged as AI" → "at most 3
            of 10", because 1 of 10 turned out to be hard.

     A number you missed stays where it is, gets diagnosed, and gets a fix
     attempted. That's where the points are.
     ───────────────────────────────────────────────────────────────────────── -->
