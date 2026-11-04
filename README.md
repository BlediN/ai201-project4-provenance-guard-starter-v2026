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

**What it measures:** The text's shape as text, without understanding a word.
It uses three scored measures (`stylometry.py::style_signal`):

| Measure | What it is | AI end → 1.0 | Human end → 0.0 | Weight |
|---|---|---|---|---|
| Sentence spread | stdev ÷ mean of words per sentence | 0.20 | 0.80 | 0.4 |
| Mean word length | average letters per word | 5.6 | 4.0 | 0.4 |
| Punctuation | commas, semicolons, colons, dashes, brackets, quotes, ! and ? per word (full stops left out) | 0.02 | 0.12 | 0.2 |
| Type-token ratio | unique ÷ total words, over a 50-word moving window | — | — | **0** |

Each measure is placed on a straight line between its AI end and its human
end, clamped to 0–1, then weighted. Type-token ratio is computed and printed
but gets no weight. On my calibration set it rated Thoreau (human) the *most*
AI-like text (0.675) and lightly edited AI the most human (0.933), so it was
adding noise. Mean word length replaced it: it's the measure that separates
plain human writing (3.4 letters per word) from AI prose (5.4–5.75).

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

**The rule:** Take a weighted average of the two signals, giving the style
signal more weight. If the two signals disagree by a wide margin, pull the
result halfway toward 0.5, because two methods contradicting each other is
uncertainty, and uncertainty belongs in "unsure", not in a confident label.
Style gets more weight because signal one's known blind spot (plain writers)
is exactly the false positive this service must not make.

**The numbers** (all in `config.py`):

| Setting | Value |
|---|---|
| `WEIGHT_MODEL_SIGNAL` | 0.4 |
| `WEIGHT_STYLE_SIGNAL` | 0.6 |
| `DISAGREEMENT_GAP` (when the pull applies) | signals more than 0.3 apart |
| `DISAGREEMENT_PULL` | halfway (0.5) toward 0.5 |
| `HUMAN_THRESHOLD` | 0.35 |
| `AI_THRESHOLD` | 0.75, raised from the starter's 0.65 |

```
combined = 0.4 × model_score + 0.6 × style_score
if |model_score − style_score| > 0.3:
    combined = combined + (0.5 − combined) × 0.5
```

**Where it lives:** `scoring.py::combine_signals`

**The four calibration inputs, plus extras.** These are both signals printed
separately, then combined:

| Input | `model_score` | `style_score` | Combined | Lands in |
|---|---|---|---|---|
| Clearly AI ("The implementation of the proposed framework…") | 0.886 | 0.867 | 0.875 | AI |
| Clearly AI (essay on AI) | 0.830 | 0.845 | 0.839 | AI |
| Clearly AI (healthy habits) | 0.879 | 0.970 | 0.934 | AI |
| Clearly human, casual ("Honestly the bus was late again…") | 0.341 | 0.049 | 0.166 | human |
| **Plain human** ("I moved to this city two years ago…") | **0.862** | **0.530** | **0.582** | unsure |
| Formal human (Thoreau, *Walden*) | 0.650 | 0.404 | 0.502 | unsure |
| Formal human (Darwin, *Origin of Species*) | 0.705 | 0.626 | 0.658 | unsure |
| Lightly edited AI (remote work) | 0.591 | 0.469 | 0.518 | unsure |
| Lightly edited AI (healthy habits) | 0.617 | 0.607 | 0.611 | unsure |

Where they agree: clean AI text (both high) and casual human text (both low).
Where they disagree: the plain human writer, and Thoreau to a lesser degree.

What surprised me: formal human writing is *not* safely human here. Signal one
gave Thoreau 0.650 and Darwin 0.705, because old, measured prose is still
fairly predictable to gpt2. Thoreau stays out of the AI label because his
words are short (3.7 letters) and his punctuation is heavy, which signal two
reads as human. Darwin stays out only because of the raised threshold: 0.658
would have been labelled AI at the starter's 0.65.

**A case where my two signals split, and what my rule does with it:**
The plain human paragraph ("I moved to this city two years ago. At first it
was hard…"). Signal one says **0.862**, as AI-like as real AI text, because
short common words are easy for gpt2 to predict. Signal two says **0.530**:
its sentences are even (AI-like), but its words average 3.4 letters, the
shortest of any text I tested, which no AI sample came close to. The gap
is 0.332, over 0.3, so the disagreement rule applies. A plain weighted average
would give 0.664, and the pull brings it to **0.582**, comfortably inside
"unsure". Under the starter settings (0.5 / 0.5, AI at 0.65) this writer would
have scored 0.696 and been labelled **high-confidence AI**, which is the
false positive from Milestone 1.

**What it costs:** both lightly edited AI texts (0.518 and 0.611) land in
"unsure" rather than AI. A few human touches are enough to get AI text out of
the AI label. That's the trade I chose: missed AI text over accused writers.



---

## Label Variants

<!-- The exact text of all three labels, as a reader would see them, and the
     score range each covers.

     Write them for someone who has never heard the word "threshold". -->

| Label | Score range | The exact text a reader sees |
|---|---|---|
| high-confidence human | 0.00 – 0.35 | We think this was probably written by a person. Our automated check found the patterns we usually see in human writing. |
| unsure | above 0.35, below 0.75 | We can't tell whether this was written by a person or with help from AI. That happens often with clear, simple writing, and it is not a finding against the writer. We aren't marking this as AI. |
| high-confidence AI | 0.75 – 1.00 | We think this was probably written with help from AI. This is an automated estimate, not proof, and it can be wrong. If you wrote this yourself, you can ask for a person to review it. |

The ranges come from `scoring.label_ranges()` and leave no gap: a score of
exactly 0.35 is human, and exactly 0.75 is AI (`scoring.py::score_to_label`).
All three were reached through `POST /submit`: see Sample Run.

**Why I worded the "unsure" one this way:**
On my calibration set, "unsure" is where the plain writer, both formal
writers, and lightly edited AI all land. So the person most likely to read it
is someone who wrote every word themselves. That's why it:

- **says what we don't know, not what they did.** "We can't tell", not "this
  may be AI".
- **names the reason it happens to innocent writers:** clear, simple writing.
  That's signal one's blind spot from Milestone 1, said in plain words. A
  writer who gets this label learns it's about *style*, not about them.
- **states plainly that it isn't an accusation.** "Not a finding against the
  writer" and "We aren't marking this as AI", so a teacher or an editor
  reading it over the writer's shoulder can't treat it as a soft "AI".

The AI label is worded with the same caution: "probably", "an automated
estimate, not proof", and it points straight at the appeal. A writer who is
wrongly accused should learn from the label itself that a person can review it.



---

## Sample Run

<!-- One submission and one appeal, pasted as text: the request, the response,
     and the log entries. -->

**A submission**

<!-- The response needs content_id, guess, confidence, label, AND both
     signal scores - model_score and style_score. Unit 8's tools read those
     two off the response, so leaving them out costs you next unit's
     diagnostics. -->

*From Milestone 3: signal one on its own first*, from the terminal, before wiring it in
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

*The full service: one submission per label.*

High-confidence AI:

```bash
$ curl -X POST http://127.0.0.1:5000/submit \
  -H "Content-Type: application/json" \
  -d '{"text": "The implementation of the proposed framework requires careful consideration of several key factors. First, it is important to note that the system must be able to handle a wide variety of different inputs. Additionally, the overall performance of the solution depends on a number of important variables.", "creator_id": "bledar"}'
```

```json
{
  "confidence": 0.875,
  "content_id": "37c1fca0-512d-4c93-a06e-215d6eaf79e8",
  "guess": "ai",
  "label": "We think this was probably written with help from AI. This is an automated estimate, not proof, and it can be wrong. If you wrote this yourself, you can ask for a person to review it.",
  "model_score": 0.8864,
  "status": "decided",
  "style_score": 0.8674
}
```

High-confidence human:

```bash
$ curl -X POST http://127.0.0.1:5000/submit \
  -H "Content-Type: application/json" \
  -d '{"text": "Honestly the bus was late again, third time this week, so I ended up jogging the last bit in my work shoes - terrible idea. Got there sweaty, missed half the standup, and my manager just gave me that look. Whatever. Tomorrow I am biking, rain or not.", "creator_id": "bledar"}'
```

```json
{
  "confidence": 0.3245,
  "content_id": "021d5071-6a08-4141-bfb5-c88d27788a67",
  "guess": "human",
  "label": "We think this was probably written by a person. Our automated check found the patterns we usually see in human writing.",
  "model_score": 0.3328,
  "status": "decided",
  "style_score": 0.0266
}
```

Unsure (the plain writer from Milestone 1, whose signals split):

```bash
$ curl -X POST http://127.0.0.1:5000/submit \
  -H "Content-Type: application/json" \
  -d '{"text": "I moved to this city two years ago. At first it was hard. I did not know many people and my English was not good. Now I have a job and some friends. I still miss my family, but I call them every week. I think I made a good choice.", "creator_id": "bledar"}'
```

```json
{
  "confidence": 0.5815,
  "content_id": "0abf5703-6dcb-43a7-bcea-c4b8ef5494a7",
  "guess": "unsure",
  "label": "We can't tell whether this was written by a person or with help from AI. That happens often with clear, simple writing, and it is not a finding against the writer. We aren't marking this as AI.",
  "model_score": 0.862,
  "status": "decided",
  "style_score": 0.5303
}
```

The human example only just makes it: 0.3245 against a cutoff of 0.35. The
human label is reachable, but it's a narrow band, and that's worth watching
in unit 8.

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

Against the real `content_id` of the AI-labelled submission above:

```bash
$ curl -X POST http://127.0.0.1:5000/appeal \
  -H "Content-Type: application/json" \
  -d '{"content_id": "37c1fca0-512d-4c93-a06e-215d6eaf79e8", "reasoning": "I wrote this myself for a project proposal. It sounds formal because it is a formal document."}'
```

```json
{
  "content_id": "37c1fca0-512d-4c93-a06e-215d6eaf79e8",
  "message": "Thanks. Your appeal has been received and a person will review this decision. Nothing else is needed from you right now.",
  "status": "under_review"
}
```

A made-up `content_id` is refused, so a typo can't look like a success:

```bash
$ curl -X POST http://127.0.0.1:5000/appeal \
  -H "Content-Type: application/json" \
  -d '{"content_id": "not-a-real-id", "reasoning": "typo test"}'
```

```json
{
  "error": "not_found",
  "message": "No submission with that content_id."
}
```

A missing `reasoning` returns `400 invalid_input`.

**What the log shows afterwards**

```bash
$ curl "http://127.0.0.1:5000/log?limit=4"
```

```json
{
  "count": 4,
  "entries": [
    {
      "combined_score": 0.875,
      "content_id": "37c1fca0-512d-4c93-a06e-215d6eaf79e8",
      "creator_id": "bledar",
      "guess": "ai",
      "label": "We think this was probably written with help from AI. This is an automated estimate, not proof, and it can be wrong. If you wrote this yourself, you can ask for a person to review it.",
      "model_score": 0.8864,
      "status": "decided",
      "style_score": 0.8674,
      "timestamp": "2026-11-04T04:23:55+00:00"
    },
    {
      "combined_score": 0.3245,
      "content_id": "021d5071-6a08-4141-bfb5-c88d27788a67",
      "creator_id": "bledar",
      "guess": "human",
      "label": "We think this was probably written by a person. Our automated check found the patterns we usually see in human writing.",
      "model_score": 0.3328,
      "status": "decided",
      "style_score": 0.0266,
      "timestamp": "2026-11-04T04:23:55+00:00"
    },
    {
      "combined_score": 0.5815,
      "content_id": "0abf5703-6dcb-43a7-bcea-c4b8ef5494a7",
      "creator_id": "bledar",
      "guess": "unsure",
      "label": "We can't tell whether this was written by a person or with help from AI. That happens often with clear, simple writing, and it is not a finding against the writer. We aren't marking this as AI.",
      "model_score": 0.862,
      "status": "decided",
      "style_score": 0.5303,
      "timestamp": "2026-11-04T04:23:56+00:00"
    },
    {
      "content_id": "37c1fca0-512d-4c93-a06e-215d6eaf79e8",
      "creator_id": "bledar",
      "event": "appeal",
      "reasoning": "I wrote this myself for a project proposal. It sounds formal because it is a formal document.",
      "status": "under_review",
      "timestamp": "2026-11-04T04:23:56+00:00"
    }
  ]
}
```

The log is append-only. The original AI decision for `37c1fca0…` is still
there unchanged, and the appeal sits after it with the writer's reasoning and
the new status `under_review`. Anyone reading it later can see what was
decided, that it was challenged, and in what order. Rejected requests (empty
text, unknown `content_id`) left no entry.

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
