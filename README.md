# Provenance Guard

> ### 👋 Start here
>
> **New to this repo? Read [RUNNING.md](RUNNING.md) first.**
>
> Once `python test.py` passes, in one terminal:
>
> ```bash
> python app.py
> ```
>
> and in another:
>
> ```bash
> curl -X POST http://127.0.0.1:5000/ping \
>   -H "Content-Type: application/json" \
>   -d '{"message": "hello"}'
> ```
>
> That's a web service. Something sent it text and it sent something back.
> Everything this unit is filling in the middle.
>
> **The rest of this file is your submission.** Fill it in as you go.

---

<!-- ─────────────────────────────────────────────────────────────────────────
     Unit 7 asks for the first five sections. Unit 8 adds the five below.

     Everything is pasted as TEXT. A pasted curl response gets full credit;
     a picture of your terminal gets none.
     ───────────────────────────────────────────────────────────────────────── -->

<!-- ═══════════════════════ UNIT 7 — THE BUILD ═══════════════════════ -->

## What This Does

<!-- The path a submission takes, from arriving to the label a reader sees.
     Six or seven steps — this is the trace you wrote in Milestone 1. -->

1. **It arrives.** A writer sends `POST /submit` with `{"text": "...", "creator_id": "..."}`.
   Flask hands the JSON body to `app.py::submit`.
2. **The route checks it.** Missing, empty, or oversized `text` (or a missing
   `creator_id`) is turned away with a `400` before any model runs. Valid input
   gets a fresh `content_id` (a uuid), so this exact submission can be found
   again later, which is what an appeal needs.
3. **Signal one reads it** (`detector.model_signal`). A local gpt2 model
   measures how predictable the text is (perplexity) and squashes that onto a
   0–1 `model_score`, where higher means more AI-like.
4. **Signal two measures its shape** (`stylometry.style_signal`). It looks at
   how much sentence lengths vary, how much vocabulary repeats, and how
   punctuation is used. That gives a 0–1 `style_score` that points the same way.
5. **The scores are combined** (`scoring.combine_signals`). A weighted average
   of the two (weights in `config.py`) gives one combined score, returned as
   `confidence`: 0 means confidently human, 1 means confidently AI.
6. **The score becomes a label** (`scoring.score_to_label`). The thresholds in
   `config.py` sort it into high-confidence human, unsure, or high-confidence
   AI. The result is a short `guess` plus a plain-English `label` written for
   the person reading it.
7. **It's logged and returned.** `audit.log_decision` writes both signal scores,
   the combined score, the label, and status `decided` to `logs/audit.jsonl`,
   and the response goes back to the caller. If the writer disagrees,
   `POST /appeal` with that `content_id` moves the item to `under_review` and
   logs the writer's reasoning.



---

## Signals and Scoring

<!-- For each signal: what property it measures, why that might differ between
     human and AI writing, and WHAT IT CAN'T SEE.

     The blind spot is not optional. If you can't name it you don't understand
     the signal yet — and unit 8's attack set is built out of blind spots. -->

### Signal one — the model signal

**What it measures:** How predictable the text is to a language model.
`detector.py` runs gpt2 locally and computes perplexity, the model's average
surprise at each next word. Low perplexity (easy to predict) maps to a high
`model_score`.

**Why that might differ between human and AI writing:** An AI model writes by
repeatedly choosing a likely next word, so its output is exactly the kind of
text a model finds easy to predict. Human writing wanders more: unusual word
choices, specific details, abrupt turns. All of that surprises the model.

**What it can't see:** Who actually wrote the text. Predictable is not the same
as machine-written. Plain writing, with common words in short ordinary
sentences, scores as AI-like even when a person wrote every word: a
second-language writer sticking to words they're sure of, a younger writer,
anyone writing simply on purpose. I checked this. A plain paragraph I wrote
("I moved to this city two years ago. At first it was hard...") got perplexity
14.3 and a `model_score` of **0.862**. The reverse also fools it: AI text
dressed up with rare or ornate vocabulary surprises gpt2 and scores as
*human*. It also only reads the first 400 tokens.

### Signal two — the style signal

**What it measures:** The text's shape as text, without understanding a word:
how much sentence lengths vary (the spread relative to the mean), how much
vocabulary repeats (type-token ratio), and how much varied punctuation there is
(commas, semicolons, dashes, and so on).

**Why that might differ between human and AI writing:** Generated prose tends
toward even, mid-length sentences, a narrower band of vocabulary, and
tidy, uniform punctuation, because "most likely next word" produces the average
case. Human writing is lumpier: a three-word sentence next to a forty-word
one, a run of dashes, a repeated pet word.

**What it can't see:** What the text means. Anything that changes the shape
without changing the content moves this score: adding typos, splitting long
sentences into short ones, sprinkling in semicolons, pasting in a quotation.
Those are cheap, deliberate edits that can push AI text toward "human". It
also has almost nothing to measure on very short text: a single sentence has
no length spread at all. Type-token ratio drops as text gets longer, so length
alone can look like a signal. And a writer whose own style is naturally
even and repetitive gets pushed toward "AI" for the same reason signal one
accuses plain writers.

### The combining rule

<!-- Write the rule in words, then give the numbers. What happens when the two
     signals disagree? -->

**The rule:**

**The numbers:**

**Where it lives:** `scoring.py::combine_signals`

<!-- ⚠️ The grader checks your code against that line. If your rule lives
     somewhere else, say where. -->

**A case where my two signals split, and what my rule does with it:**



---

## Label Variants

<!-- The exact text of all three labels, as a reader would see them, and the
     score range each covers.

     Write them for someone who has never heard the word "threshold". -->

| Label | Score range | The exact text a reader sees |
|---|---|---|
| high-confidence human |  |  |
| unsure |  |  |
| high-confidence AI |  |  |

**Why I worded the "unsure" one this way:**
<!-- It's the hardest of the three. It has to admit uncertainty without
     sounding like an accusation, because the person reading it may well have
     written every word themselves. -->



---

## Sample Run

<!-- One submission and one appeal, pasted as text: the request, the response,
     and the log entries. -->

**A submission**

<!-- The response needs content_id, guess, confidence, label, AND both
     signal scores - model_score and style_score. Unit 8's tools read those
     two off the response, so leaving them out costs you next unit's
     diagnostics. -->

*Milestone 3 version: one signal. `guess`, `confidence` and `label` are
placeholders, and `style_score` is `null` until signal two exists. This gets
replaced with the full run in Milestone 5.*

Signal one on its own first, from the terminal, before wiring it in
(`python detector.py "<text>"`):

| Text | Sure it's… | Perplexity | `model_signal` |
|---|---|---|---|
| "The implementation of the proposed framework requires careful consideration…" | AI | 12.5 | 0.8864 |
| "Honestly the bus was late again, third time this week…" | Human | 67.9 | 0.3412 |
| "Remote work has changed a lot about how teams operate. Honestly, the biggest shift…" (AI, lightly edited) | In between | 35.8 | 0.5908 |

The three land in the order I expected, with the edited text in the middle.

The control route answers:

```bash
$ curl -X POST http://127.0.0.1:5000/ping \
  -H "Content-Type: application/json" \
  -d '{"message": "hello"}'
```

```json
{
  "ok": true,
  "service": "provenance-guard",
  "you_sent": "hello"
}
```

A submission:

```bash
$ curl -X POST http://127.0.0.1:5000/submit \
  -H "Content-Type: application/json" \
  -d '{"text": "Remote work has changed a lot about how teams operate. Honestly, the biggest shift for us was meetings: we went from three a day to maybe one, and nobody misses the rest. That said, it is important to keep communication clear, because small misunderstandings can grow quickly when people are not in the same room.", "creator_id": "bledar"}'
```

```json
{
  "confidence": 0.5908,
  "content_id": "29555484-df49-4444-82a9-79afb750bdcc",
  "guess": "pending",
  "label": "Not labelled yet: labels arrive in Milestone 5.",
  "model_score": 0.5908,
  "status": "decided",
  "style_score": null
}
```

Bad input is refused before the model runs, and nothing is logged:

```bash
$ curl -X POST http://127.0.0.1:5000/submit \
  -H "Content-Type: application/json" \
  -d '{"text": "", "creator_id": "bledar"}'
```

```json
{
  "error": "invalid_input",
  "message": "'text' is required and must be a non-empty string."
}
```

**An appeal**

*Not built yet: `/appeal` is Milestone 5.*

```bash
$ curl -X POST http://127.0.0.1:5000/appeal \
  -H "Content-Type: application/json" \
  -d '{"content_id": "...", "reasoning": "..."}'
```

```json

```

**What the log shows afterwards**

```bash
$ curl "http://127.0.0.1:5000/log?limit=3"
```

```json
{
  "count": 1,
  "entries": [
    {
      "combined_score": 0.5908,
      "content_id": "29555484-df49-4444-82a9-79afb750bdcc",
      "creator_id": "bledar",
      "guess": "pending",
      "label": "Not labelled yet: labels arrive in Milestone 5.",
      "model_score": 0.5908,
      "status": "decided",
      "style_score": null,
      "timestamp": "2026-11-04T04:13:07+00:00"
    }
  ]
}
```

The submission landed in `logs/audit.jsonl` with the same `content_id` the
response returned, and the rejected empty submission left no entry.

---

## How I Used AI

**Moment 1**

- *What I asked for:*
- *What came back:*
- *What I changed:*

**Moment 2**

- *What I asked for:*
- *What came back:*
- *What I changed:*

<!-- ═══════════════════════ UNIT 8 — THE TEST ═══════════════════════ -->

---

## Rate Limiting

**My limits:** ___ per minute, ___ per day

**Why those numbers and not others:**
<!-- Think about two people: a real writer submitting their own work a few
     times an hour, and a script sending a thousand variations to map your
     thresholds. Your numbers have to be liveable for the first and hostile to
     the second. -->

**What a caller counts as:** <!-- per address, or per creator_id? They fail
differently — one script can look like a thousand callers, and one household
can look like one. -->

**The run of status codes when I pushed past it:**

<!-- python run_attacks.py --flood 15 -->

```

```

---

## Attack Run — Before

<!-- Every attack with its outcome and a held-or-broke mark.
     `run_attacks.py --label before` produces the table. -->



**Ten audit log entries from the run**

<!-- Choose entries that show the interesting failures, not the ten easiest. -->

```json

```

**The single worst outcome** — the one I'd least want to explain to a writer
whose work got caught by it:



---

## Run Log and Verdicts

<!-- Your five criteria across three trials. `run_eval.py --label before`. -->

| Criterion | Target | Run 1 | Run 2 | Run 3 | Verdict |
|---|---|---|---|---|---|
| 1.  |  |  |  |  |  |
| 2.  |  |  |  |  |  |
| 3.  |  |  |  |  |  |
| 4.  |  |  |  |  |  |
| 5.  |  |  |  |  |  |

**Real output from one trial**, pasted as text, naming the file and function
that produced it:

```json

```

**Diagnoses**

<!-- For each miss AND each broken attack: the stage and the mechanism.

     Your service has five stages: the route, signal one, signal two, the
     combining rule, and the label mapping.

     Not a diagnosis:  "The evasion attacks got through."
     A diagnosis:      "Four evasion inputs got through. All four had typos
                        added to AI-generated text. My second signal measures
                        word variety, and typos raise word variety — so it read
                        them as more human. My combining rule weights that
                        signal at 0.6, so it dragged the whole score down."

     Look for a pattern. Five failures on one signal is one problem, not five. -->



---

## The Improvement

**What I changed:**

**Which diagnosis pointed at it:**

### Attack Run — After

<!-- Same format. `run_attacks.py --label after` -->



**Did it help, and how do I know:**

<!-- Widening the unsure band usually fixes one problem and creates another.
     Reporting that trade honestly is worth full credit. -->



---

## What's Still Broken

<!-- For each attack still getting through and each criterion still missed:
     what you'd do, and why you stopped. -->



**The trade I'd make if this ran for real:**

<!-- Two sentences. Fewer wrong accusations means more AI text getting through.
     Say which way you'd go and WHO PAYS FOR IT. -->



<!-- ═════════════════════════════════════════════════════════════════════

     SUBMISSION CHECKLIST — unit 7

       [ ] criteria.md has five numbered criteria, each naming a target
       [ ] Each has a reason underneath
       [ ] POST /submit returns content_id, guess, confidence, label,
           model_score and style_score
       [ ] Two signals that measure DIFFERENT things, each with a named blind spot
       [ ] Three label variants, all reachable, written out in full
       [ ] POST /appeal changes status and writes to the log
       [ ] Signals and Scoring names scoring.py::combine_signals
       [ ] Sample Run: a submission AND an appeal, as pasted text
       [ ] At least four new commits
       [ ] Repository URL submitted — WRITE IT DOWN

     SUBMISSION CHECKLIST — unit 8

       [ ] Rate limiting on /submit, with your numbers justified
       [ ] The pasted run of status codes showing 429
       [ ] Audit log with all nine fields, recording rejections too
       [ ] The attack set run IN FULL, every input with an outcome
       [ ] Ten audit log entries pasted as text
       [ ] Five criteria across three trials
       [ ] A diagnosis for every miss and every broken attack — stage AND mechanism
       [ ] One improvement, with Attack Run — After
       [ ] What's Still Broken, including the production trade
       [ ] At least four new commits
       [ ] The SAME repository URL as last unit

     Do not delete and recreate this repository.
     ═════════════════════════════════════════════════════════════════════ -->

---

📖 **How to run this project: [RUNNING.md](RUNNING.md)**
