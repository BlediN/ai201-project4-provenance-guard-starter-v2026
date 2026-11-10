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

I built this with Claude Code (an AI coding assistant) working in the repo,
running commands and editing files at my request, milestone by milestone.

**Moment 1: building the style signal**

- *What I asked for:* Build signal two in `stylometry.py` from the three
  stubbed measures (sentence spread, type-token ratio, punctuation density)
  and combine them into one 0–1 score.
- *What came back:* The three measures, plus a calibration script that
  printed each raw measure for nine texts: three AI, casual human, plain
  human, two formal human (Thoreau, Darwin), and two lightly edited AI. The
  numbers showed type-token ratio pointing the wrong way. Thoreau scored the
  *most* AI-like (0.675) and edited AI the most human (0.933). The plain human
  writer looked AI-like on all three measures.
- *What I changed:* Type-token ratio stays computed but gets **zero weight**,
  and a fourth measure, **mean word length**, replaces it. That measure
  separates plain human writing (3.4 letters per word) from AI prose
  (5.4–5.75), which is exactly where signal one fails. I kept the decision
  only because the measurements backed it up: the stub's three measures,
  taken on trust, would have accused the plain writer.

**Moment 2: the false-positive trace**

- *What I asked for:* Fill in the Milestone 1 trace: follow a real writer
  whose work gets called AI through the service.
- *What came back:* A trace of a second-language writer with a *measured*
  model score (0.862, from running `detector.py` on a plain paragraph) but an
  *estimated* style score of 0.70, because signal two didn't exist yet. With
  the starter settings that combined to about 0.78, labelled AI.
- *What I changed:* Once signal two was built, I replaced the estimate with the
  measured value (0.530) and noted the update in `MILESTONE1.md`, rather than
  leaving a guessed number in the write-up. That measured split (0.862
  against 0.530) then became the case the combining rule was designed around:
  style weighted 0.6, a pull toward 0.5 when the signals disagree by more than
  0.3, and the AI threshold raised to 0.75. Without that, she would still have
  scored 0.696 and been accused.

<!-- ═══════════════════════ UNIT 8 — THE TEST ═══════════════════════ -->

---

## Rate Limiting

**My limits:** 5 per minute, 50 per day (`config.py`: `RATE_LIMIT_PER_MINUTE`,
`RATE_LIMIT_PER_DAY`, applied to `/submit` by `@limiter.limit(...)` in `app.py`)

**Why those numbers and not others:**

- **A real writer.** Someone checking their own work submits a draft, reads the
  label, edits, and submits again. That loop takes longer than 12 seconds, so 5
  a minute still lets through a quick burst of revisions. 50 a day works out to
  one check every ~20 minutes across a long writing day, which is more than a
  person checking their own drafts will use.
- **A script mapping my thresholds.** A script sending 1,000 variations to find
  where "unsure" turns into "AI" hits the wall on its 6th request. At 50 a day
  it needs 20 days per identity to finish, instead of a few minutes with no
  limit, or 5 days at the starter's 200 a day.
- I went lower than the starter's 10/minute and 200/day because the per-day
  number is the one that actually slows a threshold-mapping script, and 200 was
  far more than any honest writer needs.

**What a caller counts as:** per `creator_id`, falling back to the caller's IP
address when no `creator_id` is sent (`app.py::rate_limit_key`). This fails in
a specific way: `creator_id` is chosen by the caller, so a script that sends a
new `creator_id` on every request gets a fresh allowance each time. Per-IP
limits fail the other way: one script behind many addresses looks like many
callers, and a household or classroom sharing one address looks like one. The
limit only really stops a caller that keeps one identity. Closing that gap
needs server-issued creator IDs, or limiting on IP as well.

**The run of status codes when I pushed past it:**

<!-- python run_attacks.py --flood 15 -->

```
$ python run_attacks.py --flood 15
15 submissions as 'flood_tester' against http://127.0.0.1:5000

200 200 200 200 200 429 429 429 429 429 429 429 429 429 429

10 of 15 were rate limited.
```

**The rejections left a trace.** The 6th and 7th requests in the audit log
(`logs/audit.jsonl`), right after the 5th one that was accepted:

```json
{"timestamp": "2026-11-10T01:26:03+00:00", "content_id": "9faf6e9f-467b-4611-88c7-f18397bd5234", "creator_id": "flood_tester", "guess": "human", "model_score": 0.2587, "style_score": 0.2774, "combined_score": 0.2699, "label": "We think this was probably written by a person. Our automated check found the patterns we usually see in human writing.", "status": "decided"}
{"timestamp": "2026-11-10T01:26:03+00:00", "creator_id": "flood_tester", "event": "rejected", "status": "rejected", "reason": "rate_limited", "limit": "5 per 1 minute"}
{"timestamp": "2026-11-10T01:26:03+00:00", "creator_id": "flood_tester", "event": "rejected", "status": "rejected", "reason": "rate_limited", "limit": "5 per 1 minute"}
```

Requests my own route refuses are logged too (`app.py::_refuse`). A request
with no text:

```json
{"creator_id": "log_check_bad", "event": "rejected", "status": "rejected", "reason": "invalid_input", "http_status": 400, "message": "'text' is required and must be a non-empty string.", "text_chars": null, "timestamp": "2026-11-10T01:26:12+00:00"}
```

---

## Attack Run — Before

<!-- Every attack with its outcome and a held-or-broke mark.
     `run_attacks.py --label before` produces the table. -->

`python run_attacks.py --set attack_set --label before`, written to
`results/attack_run_2026-11-09_2107_before.md`. All 56 inputs were sent, and
none returned a 5xx or crashed.

**How I marked them.** An attack **broke** if it got where its `targets` column
said it was going:

- **evasion** (AI text trying to get out): BROKE unless it was labelled AI.
  "Unsure" counts as broke, because my unsure label says "We aren't marking
  this as AI", so the text got through.
- **false_positive** (genuine human writing): BROKE if it was labelled AI. Unsure
  is held, because the unsure label is explicitly not a finding against the writer.
- **prompt_injection**: BROKE if the instruction changed anything: the label
  text, the log, or the decision path. Nothing in my pipeline reads
  instructions (GPT-2 perplexity plus style statistics), so it can only move
  the scores the way any text does.
- **malformed**: BROKE on a 5xx, or on a confident label (AI or human) given to
  input that isn't writing. A clean 4xx, or an "unsure" with no accusation, is held.
- **flood_same_creator**: HELD if the limiter cut it off and logged it.

| Family | Attacks | Held | Broke |
|---|---|---|---|
| evasion | 10 | 1 | 9 |
| false_positive | 8 | 6 | 2 |
| prompt_injection | 6 | 6 | 0 |
| malformed | 20 | 19 | 1 |
| flood_same_creator | 12 | 12 | 0 |
| **Total** | **56** | **44** | **12** |

| ID | Family | Targeting | Status | Guess | Score | Verdict | What happened |
|---|---|---|---|---|---|---|---|
| EV01 | evasion | signal two: typos raise word variety | 200 | unsure | 0.574 | BROKE | model 0.16 vs style 0.97: disagreement pull dragged it to the middle |
| EV02 | evasion | signal two: broken-up sentences raise length spread | 200 | unsure | 0.582 | BROKE | both signals near 0.58, below the AI line |
| EV03 | evasion | signal two: a pasted quotation changes the shape | 200 | unsure | 0.714 | BROKE | 0.714, would be AI at my old 0.65 threshold |
| EV04 | evasion | signal one: unusual words raise perplexity | 200 | unsure | 0.663 | BROKE | model 0.61 vs style 0.97: disagreement pull |
| EV05 | evasion | both signals: heavy human editing pass | 200 | human | 0.319 | BROKE | labelled **human**, both signals fooled |
| EV06 | evasion | signal two: one very long sentence and one very short | 200 | unsure | 0.574 | BROKE | both signals near 0.57 |
| EV07 | evasion | signal two: padding with punctuation | 200 | unsure | 0.595 | BROKE | both signals near 0.59 |
| EV08 | evasion | signal one: text below the token window | 200 | unsure | 0.568 | BROKE | model 0.25 vs style 0.89: short text starved the model signal, disagreement pull |
| EV09 | evasion | both: AI text with a human opening paragraph bolted on | 200 | unsure | 0.745 | BROKE | 0.745, 0.005 under the AI threshold |
| EV10 | evasion | signal two: repeated stanza structure | 200 | ai | 0.914 | HELD | caught: model 0.98, style 0.87 |
| FP01 | false_positive | plain, clear human writing with no flourishes | 200 | unsure | 0.504 | HELD | unsure, not accused |
| FP02 | false_positive | human writing in a formal register | 200 | ai | 0.817 | BROKE | **accused**: both signals agree (0.74, 0.87) |
| FP03 | false_positive | a non-native English speaker writing carefully | 200 | unsure | 0.631 | HELD | model 0.81, style 0.51 kept it unsure |
| FP04 | false_positive | a student essay written to a rubric | 200 | ai | 0.836 | BROKE | **accused**: both signals agree (0.83, 0.84) |
| FP05 | false_positive | technical documentation written by a person | 200 | unsure | 0.714 | HELD | 0.714, close to the line |
| FP06 | false_positive | a person imitating a formal template | 200 | unsure | 0.574 | HELD | model 0.88, style 0.49 kept it unsure |
| FP07 | false_positive | human writing that repeats a phrase deliberately | 200 | unsure | 0.532 | HELD | model 0.88, style 0.35 kept it unsure |
| FP08 | false_positive | very short genuine human submission | 200 | unsure | 0.396 | HELD | unsure, not accused |
| PI01 | prompt_injection | instruction to ignore the check | 200 | unsure | 0.600 | HELD | no model reads instructions; label template unchanged |
| PI02 | prompt_injection | instruction disguised as a system message | 200 | unsure | 0.486 | HELD | label template unchanged |
| PI03 | prompt_injection | instruction wrapped in the writer's voice | 200 | unsure | 0.739 | HELD | label template unchanged (0.739 is a score question, not injection) |
| PI04 | prompt_injection | attempt to make the label text hostile | 200 | unsure | 0.441 | HELD | label text came back as the standard template |
| PI05 | prompt_injection | attempt to inject into the audit log | 200 | unsure | 0.618 | HELD | audit log line intact, every line still parses as one JSON object |
| PI06 | prompt_injection | instruction plus real human text | 200 | unsure | 0.508 | HELD | label template unchanged |
| MF01 | malformed | text is only whitespace | 400 | — | — | HELD | clean 400 |
| MF02 | malformed | text is a single character | 422 | — | — | HELD | clean 422 |
| MF03 | malformed | text is control characters | 200 | unsure | 0.370 | HELD | no crash, but decided on text with no words (model 0.0) |
| MF04 | malformed | text is only punctuation | 200 | unsure | 0.432 | HELD | no crash, but decided on text with no words |
| MF05 | malformed | text is a very long single word | 200 | ai | 0.879 | BROKE | **labelled AI** at 0.879 on a single nonsense word |
| MF06 | malformed | text is emoji only | 200 | unsure | 0.569 | HELD | no crash; model 0.997 on emoji, rescued only by style |
| MF07 | malformed | text is right-to-left script | 200 | unsure | 0.551 | HELD | no crash; model 0.906 on real non-English writing, rescued only by style |
| MF08 | malformed | text is 60,000 characters | 413 | — | — | HELD | clean 413 |
| RQ01 | malformed | no text field at all | 400 | — | — | HELD | clean 400 |
| RQ02 | malformed | no creator_id | 400 | — | — | HELD | clean 400 |
| RQ03 | malformed | text is a number | 400 | — | — | HELD | clean 400 |
| RQ04 | malformed | text is a list | 400 | — | — | HELD | clean 400 |
| RQ05 | malformed | text is null | 400 | — | — | HELD | clean 400 |
| RQ06 | malformed | creator_id is an object | 400 | — | — | HELD | clean 400 |
| RQ07 | malformed | body is a JSON array, not an object | 400 | — | — | HELD | clean 400 |
| RQ08 | malformed | body is unparseable JSON | 400 | — | — | HELD | clean 400 |
| RQ09 | malformed | body is empty | 400 | — | — | HELD | clean 400 |
| RQ10 | malformed | body is form-encoded, not JSON | 429 | — | — | HELD | 429 in the run (counted against my address); rerun alone: clean 400 |
| RQ11 | malformed | body is JSON claiming to be plain text | 429 | — | — | HELD | 429 in the run (counted against my address); rerun alone: clean 400 |
| RQ12 | malformed | deeply nested JSON | 400 | — | — | HELD | clean 400 |
| FL01_01 | flood_same_creator | 20 rapid submissions from one creator | 200 | unsure | 0.459 | HELD | 200 |
| FL01_02 | flood_same_creator | 20 rapid submissions from one creator | 200 | unsure | 0.459 | HELD | 200 |
| FL01_03 | flood_same_creator | 20 rapid submissions from one creator | 200 | unsure | 0.459 | HELD | 200 |
| FL01_04 | flood_same_creator | 20 rapid submissions from one creator | 200 | unsure | 0.459 | HELD | 200 |
| FL01_05 | flood_same_creator | 20 rapid submissions from one creator | 200 | unsure | 0.459 | HELD | 200 |
| FL01_06 | flood_same_creator | 20 rapid submissions from one creator | 429 | — | — | HELD | 429, logged |
| FL01_07 | flood_same_creator | 20 rapid submissions from one creator | 429 | — | — | HELD | 429, logged |
| FL01_08 | flood_same_creator | 20 rapid submissions from one creator | 429 | — | — | HELD | 429, logged |
| FL01_09 | flood_same_creator | 20 rapid submissions from one creator | 429 | — | — | HELD | 429, logged |
| FL01_10 | flood_same_creator | 20 rapid submissions from one creator | 429 | — | — | HELD | 429, logged |
| FL01_11 | flood_same_creator | 20 rapid submissions from one creator | 429 | — | — | HELD | 429, logged |
| FL01_12 | flood_same_creator | 20 rapid submissions from one creator | 429 | — | — | HELD | 429, logged |

RQ10 and RQ11 have no `creator_id` the limiter can read, so they counted
against my own address. They came after five other no-creator rows in the
same minute and got 429 without reaching the route. I re-sent each one alone a
minute later (`--id RQ10 --label recheck`, `--id RQ11 --label recheck`) and
both came back as a clean 400.

**Ten audit log entries from the run**

<!-- Choose entries that show the interesting failures, not the ten easiest. -->

From `logs/audit.jsonl`, in this order: EV01 (an evasion that worked through
my disagreement pull), EV05 (AI text labelled human), EV09 (0.005 under the
line), FP02 and FP04 (real writers accused), FP07 (one signal at 0.88, saved
by the other), MF05 (a nonsense word labelled AI), MF07 (non-English writing
GPT-2 can't read), PI05 (log injection attempt, entry intact), and two 429
rejections: RQ10 counted against my address as `unknown`, and the
flood_same_creator family.

```json
{"timestamp": "2026-11-10T02:07:15+00:00", "content_id": "1b188c8d-b2c5-4912-bc01-8013047ccb3e", "creator_id": "attacker_EV01", "guess": "unsure", "model_score": 0.1635, "style_score": 0.9694, "combined_score": 0.5735, "label": "We can't tell whether this was written by a person or with help from AI. That happens often with clear, simple writing, and it is not a finding against the writer. We aren't marking this as AI.", "status": "decided"}
{"timestamp": "2026-11-10T02:07:16+00:00", "content_id": "138e5b1f-28ec-4330-a761-8210b683e99a", "creator_id": "attacker_EV05", "guess": "human", "model_score": 0.3212, "style_score": 0.317, "combined_score": 0.3187, "label": "We think this was probably written by a person. Our automated check found the patterns we usually see in human writing.", "status": "decided"}
{"timestamp": "2026-11-10T02:07:17+00:00", "content_id": "f701e30f-f94a-4bb4-9180-06a20f51d3a7", "creator_id": "attacker_EV09", "guess": "unsure", "model_score": 0.7042, "style_score": 0.7717, "combined_score": 0.7447, "label": "We can't tell whether this was written by a person or with help from AI. That happens often with clear, simple writing, and it is not a finding against the writer. We aren't marking this as AI.", "status": "decided"}
{"timestamp": "2026-11-10T02:07:18+00:00", "content_id": "6e2b2ba6-8a2c-46dc-9ccf-bba40549a66f", "creator_id": "attacker_FP02", "guess": "ai", "model_score": 0.7387, "style_score": 0.8692, "combined_score": 0.817, "label": "We think this was probably written with help from AI. This is an automated estimate, not proof, and it can be wrong. If you wrote this yourself, you can ask for a person to review it.", "status": "decided"}
{"timestamp": "2026-11-10T02:07:18+00:00", "content_id": "0abc9a9d-96bb-4a3f-b891-bab4ee572248", "creator_id": "attacker_FP04", "guess": "ai", "model_score": 0.8329, "style_score": 0.8381, "combined_score": 0.836, "label": "We think this was probably written with help from AI. This is an automated estimate, not proof, and it can be wrong. If you wrote this yourself, you can ask for a person to review it.", "status": "decided"}
{"timestamp": "2026-11-10T02:07:19+00:00", "content_id": "774b571e-a221-4788-a3f5-ab0d72f9f556", "creator_id": "attacker_FP07", "guess": "unsure", "model_score": 0.8816, "style_score": 0.3539, "combined_score": 0.5325, "label": "We can't tell whether this was written by a person or with help from AI. That happens often with clear, simple writing, and it is not a finding against the writer. We aren't marking this as AI.", "status": "decided"}
{"timestamp": "2026-11-10T02:07:21+00:00", "content_id": "c4215ae5-4dfd-44ec-9456-190e886a22d1", "creator_id": "attacker_MF05", "guess": "ai", "model_score": 0.9976, "style_score": 0.8, "combined_score": 0.879, "label": "We think this was probably written with help from AI. This is an automated estimate, not proof, and it can be wrong. If you wrote this yourself, you can ask for a person to review it.", "status": "decided"}
{"timestamp": "2026-11-10T02:07:23+00:00", "content_id": "a7d15f01-c0da-41b3-84aa-25e3892201ee", "creator_id": "attacker_MF07", "guess": "unsure", "model_score": 0.9064, "style_score": 0.4, "combined_score": 0.5513, "label": "We can't tell whether this was written by a person or with help from AI. That happens often with clear, simple writing, and it is not a finding against the writer. We aren't marking this as AI.", "status": "decided"}
{"timestamp": "2026-11-10T02:07:20+00:00", "content_id": "b3c81ec5-bc66-4421-94e8-c3dc5e155c64", "creator_id": "attacker_PI05", "guess": "unsure", "model_score": 0.6401, "style_score": 0.6024, "combined_score": 0.6175, "label": "We can't tell whether this was written by a person or with help from AI. That happens often with clear, simple writing, and it is not a finding against the writer. We aren't marking this as AI.", "status": "decided"}
{"timestamp": "2026-11-10T02:07:23+00:00", "creator_id": "unknown", "event": "rejected", "status": "rejected", "reason": "rate_limited", "limit": "5 per 1 minute"}
{"timestamp": "2026-11-10T02:07:23+00:00", "creator_id": "attacker_flood", "event": "rejected", "status": "rejected", "reason": "rate_limited", "limit": "5 per 1 minute"}
```

**The single worst outcome** — the one I'd least want to explain to a writer
whose work got caught by it:

**FP04, a student essay written to a rubric, labelled AI at 0.836.** It's the
worst because nothing in my design was there to catch it. My disagreement
pull exists to protect writers when the two signals contradict each other, and
it saved FP03, FP06 and FP07. On FP04 the signals *agreed* (model 0.833, style
0.838), so there was nothing to pull, and 0.836 is well clear of my 0.75 AI
line. Writing to a rubric means doing exactly what you were told: predictable
structure, even sentences, the expected vocabulary. That is precisely what
both of my signals read as machine-made. So the student who followed the
instructions most closely is the one who gets accused, and the label they see
tells them "probably written with help from AI" with no hint that their
obedience to the rubric is the reason.

---

## Run Log and Verdicts

<!-- Your five criteria across three trials. `run_eval.py --label before`. -->

| Criterion | Target | Run 1 | Run 2 | Run 3 | Verdict |
|---|---|---|---|---|---|
| 1. My own writing flagged as AI | ≤ 1 of 10 | 1/10 | 1/10 | 1/10 | MET |
| 2. Score spread, clearly AI vs clearly human | AI mean − human mean ≥ 0.25 | 0.208 | 0.208 | 0.208 | MISSED |
| 3. All three labels reachable, AI still caught | 3 of 3 labels, ≥ 2 of 3 AI labelled AI | 2/3 labels, 2/3 AI | 2/3 labels, 2/3 AI | 2/3 labels, 2/3 AI | MISSED |
| 4. Appeal always works | 5/5 real appeals → `under_review` + log entry; 3/3 fake ids → 404 | 5/5, 3/3 | 5/5, 3/3 | 5/5, 3/3 | MET |
| 5. Bad input refused cleanly | 4/4 → 4xx with JSON `error`, 0 × 500, each < 2 s | 4/4, max 0.011 s | 4/4, max 0.012 s | 4/4, max 0.012 s | MET |

Command: `python run_eval.py --label before`, saved to
`results/run_2026-11-09_2215_before.md` (produced by `run_eval.py::main`;
service `app.py::submit`, scoring `scoring.py::combine_signals` and
`scoring.py::score_to_label`). Criterion 4 runs through
`run_eval.py::run_appeals`, which I added: it submits a text, appeals the
`content_id` that comes back via `app.py::appeal`, and checks `GET /log` for
an appeal entry with that id and that reasoning. Every request is timed
(`elapsed_s`) for criterion 5.

**All three trials gave identical scores.** gpt2 scoring is deterministic
(no sampling, same weights, same input), so "three trials" here checks that
nothing drifts rather than averaging out noise.

- **Criterion 1** was met with no margin. Piece 6 ("I have been trying to manage
  my time better this semester…") scored 0.763, just over my 0.75 AI line, in
  all three trials. One more like it and the target fails.
- **Criterion 2** was missed: AI mean 0.820, human mean 0.612, gap 0.208.
- **Criterion 3** was missed on the first half: "ai" and "unsure" appear, but
  "human" never does. None of my 10 pieces scored below 0.508, and my human
  threshold is 0.35. The second half held: 2 of 3 AI texts were labelled AI,
  and the third scored 0.741.

**Real output from one trial (Trial 1 of each)**, pasted from
`results/run_2026-11-09_2215_before.md`, produced by `run_eval.py::main`:

*Criterion 1: own writing* (`app.py::submit`)

```json
[
  {
    "status": 200,
    "guess": "unsure",
    "confidence": 0.5445,
    "model_score": 0.7096,
    "style_score": 0.4344,
    "label": "We can't tell whether this was written by a person or with help from AI. That happens often with clear, simple writing, and it is not a finding against the writer. We aren't marking this as AI.",
    "error": null,
    "elapsed_s": 8.556,
    "sent": "I woke up early today. The house was quiet. I made coffee and checked "
  },
  {
    "status": 200,
    "guess": "unsure",
    "confidence": 0.5533,
    "model_score": 0.812,
    "style_score": 0.4695,
    "label": "We can't tell whether this was written by a person or with help from AI. That happens often with clear, simple writing, and it is not a finding against the writer. We aren't marking this as AI.",
    "error": null,
    "elapsed_s": 0.127,
    "sent": "I took the bus to the store. It was crowded, but I found a seat near t"
  },
  {
    "status": 200,
    "guess": "unsure",
    "confidence": 0.5134,
    "model_score": 0.6836,
    "style_score": 0.4,
    "label": "We can't tell whether this was written by a person or with help from AI. That happens often with clear, simple writing, and it is not a finding against the writer. We aren't marking this as AI.",
    "error": null,
    "elapsed_s": 0.143,
    "sent": "My room was messy after a busy week. I washed my clothes and put the b"
  },
  {
    "status": 200,
    "guess": "unsure",
    "confidence": 0.6772,
    "model_score": 0.7149,
    "style_score": 0.652,
    "label": "We can't tell whether this was written by a person or with help from AI. That happens often with clear, simple writing, and it is not a finding against the writer. We aren't marking this as AI.",
    "error": null,
    "elapsed_s": 0.156,
    "sent": "I cooked rice and vegetables for dinner. The meal was simple, but it t"
  },
  {
    "status": 200,
    "guess": "unsure",
    "confidence": 0.6912,
    "model_score": 0.8021,
    "style_score": 0.6173,
    "label": "We can't tell whether this was written by a person or with help from AI. That happens often with clear, simple writing, and it is not a finding against the writer. We aren't marking this as AI.",
    "error": null,
    "elapsed_s": 0.164,
    "sent": "Last Saturday, I walked around the neighborhood after lunch. The weath"
  },
  {
    "status": 200,
    "guess": "ai",
    "confidence": 0.7632,
    "model_score": 0.7802,
    "style_score": 0.7518,
    "label": "We think this was probably written with help from AI. This is an automated estimate, not proof, and it can be wrong. If you wrote this yourself, you can ask for a person to review it.",
    "error": null,
    "elapsed_s": 0.155,
    "sent": "I have been trying to manage my time better this semester. Writing dow"
  },
  {
    "status": 200,
    "guess": "unsure",
    "confidence": 0.5085,
    "model_score": 0.7255,
    "style_score": 0.3779,
    "label": "We can't tell whether this was written by a person or with help from AI. That happens often with clear, simple writing, and it is not a finding against the writer. We aren't marking this as AI.",
    "error": null,
    "elapsed_s": 0.131,
    "sent": "The best part of my morning is having a few quiet minutes before the d"
  },
  {
    "status": 200,
    "guess": "unsure",
    "confidence": 0.5414,
    "model_score": 0.8572,
    "style_score": 0.4,
    "label": "We can't tell whether this was written by a person or with help from AI. That happens often with clear, simple writing, and it is not a finding against the writer. We aren't marking this as AI.",
    "error": null,
    "elapsed_s": 0.136,
    "sent": "I visited a friend last weekend, and we talked about school, work, and"
  },
  {
    "status": 200,
    "guess": "unsure",
    "confidence": 0.61,
    "model_score": 0.7526,
    "style_score": 0.5149,
    "label": "We can't tell whether this was written by a person or with help from AI. That happens often with clear, simple writing, and it is not a finding against the writer. We aren't marking this as AI.",
    "error": null,
    "elapsed_s": 0.139,
    "sent": "When I started learning to cook, I made many small mistakes. I burned "
  },
  {
    "status": 200,
    "guess": "unsure",
    "confidence": 0.6467,
    "model_score": 0.7167,
    "style_score": 0.6,
    "label": "We can't tell whether this was written by a person or with help from AI. That happens often with clear, simple writing, and it is not a finding against the writer. We aren't marking this as AI.",
    "error": null,
    "elapsed_s": 0.165,
    "sent": "I want to spend less time looking at my phone before bed. It is easy t"
  }
]
```

*Criterion 2: score spread* (`app.py::submit`; first 5 are AI, last 5 are mine)

```json
[
  {
    "status": 200,
    "guess": "ai",
    "confidence": 0.8784,
    "model_score": 0.7218,
    "style_score": 0.9828,
    "label": "We think this was probably written with help from AI. This is an automated estimate, not proof, and it can be wrong. If you wrote this yourself, you can ask for a person to review it.",
    "error": null,
    "elapsed_s": 0.217,
    "sent": "The implementation of the proposed framework demonstrates significant "
  },
  {
    "status": 200,
    "guess": "ai",
    "confidence": 0.7605,
    "model_score": 0.8939,
    "style_score": 0.6715,
    "label": "We think this was probably written with help from AI. This is an automated estimate, not proof, and it can be wrong. If you wrote this yourself, you can ask for a person to review it.",
    "error": null,
    "elapsed_s": 0.238,
    "sent": "Artificial intelligence is transforming the way we live and work. From"
  },
  {
    "status": 200,
    "guess": "ai",
    "confidence": 0.8324,
    "model_score": 0.8877,
    "style_score": 0.7956,
    "label": "We think this was probably written with help from AI. This is an automated estimate, not proof, and it can be wrong. If you wrote this yourself, you can ask for a person to review it.",
    "error": null,
    "elapsed_s": 0.266,
    "sent": "Maintaining healthy habits is essential for overall well-being. Regula"
  },
  {
    "status": 200,
    "guess": "ai",
    "confidence": 0.762,
    "model_score": 0.7709,
    "style_score": 0.7561,
    "label": "We think this was probably written with help from AI. This is an automated estimate, not proof, and it can be wrong. If you wrote this yourself, you can ask for a person to review it.",
    "error": null,
    "elapsed_s": 0.252,
    "sent": "Remote work has fundamentally reshaped the modern workplace. Employees"
  },
  {
    "status": 200,
    "guess": "ai",
    "confidence": 0.8683,
    "model_score": 0.9089,
    "style_score": 0.8412,
    "label": "We think this was probably written with help from AI. This is an automated estimate, not proof, and it can be wrong. If you wrote this yourself, you can ask for a person to review it.",
    "error": null,
    "elapsed_s": 0.288,
    "sent": "Climate change represents one of the most pressing challenges of our t"
  },
  {
    "status": 200,
    "guess": "unsure",
    "confidence": 0.5445,
    "model_score": 0.7096,
    "style_score": 0.4344,
    "label": "We can't tell whether this was written by a person or with help from AI. That happens often with clear, simple writing, and it is not a finding against the writer. We aren't marking this as AI.",
    "error": null,
    "elapsed_s": 0.102,
    "sent": "I woke up early today. The house was quiet. I made coffee and checked "
  },
  {
    "status": 200,
    "guess": "unsure",
    "confidence": 0.5533,
    "model_score": 0.812,
    "style_score": 0.4695,
    "label": "We can't tell whether this was written by a person or with help from AI. That happens often with clear, simple writing, and it is not a finding against the writer. We aren't marking this as AI.",
    "error": null,
    "elapsed_s": 0.152,
    "sent": "I took the bus to the store. It was crowded, but I found a seat near t"
  },
  {
    "status": 200,
    "guess": "unsure",
    "confidence": 0.6912,
    "model_score": 0.8021,
    "style_score": 0.6173,
    "label": "We can't tell whether this was written by a person or with help from AI. That happens often with clear, simple writing, and it is not a finding against the writer. We aren't marking this as AI.",
    "error": null,
    "elapsed_s": 0.142,
    "sent": "Last Saturday, I walked around the neighborhood after lunch. The weath"
  },
  {
    "status": 200,
    "guess": "ai",
    "confidence": 0.7632,
    "model_score": 0.7802,
    "style_score": 0.7518,
    "label": "We think this was probably written with help from AI. This is an automated estimate, not proof, and it can be wrong. If you wrote this yourself, you can ask for a person to review it.",
    "error": null,
    "elapsed_s": 0.156,
    "sent": "I have been trying to manage my time better this semester. Writing dow"
  },
  {
    "status": 200,
    "guess": "unsure",
    "confidence": 0.5085,
    "model_score": 0.7255,
    "style_score": 0.3779,
    "label": "We can't tell whether this was written by a person or with help from AI. That happens often with clear, simple writing, and it is not a finding against the writer. We aren't marking this as AI.",
    "error": null,
    "elapsed_s": 0.159,
    "sent": "The best part of my morning is having a few quiet minutes before the d"
  }
]
```

*Criterion 3: label coverage* (`app.py::submit`; 3 AI, 3 mine, 3 borderline)

```json
[
  {
    "status": 200,
    "guess": "ai",
    "confidence": 0.7707,
    "model_score": 0.8042,
    "style_score": 0.7483,
    "label": "We think this was probably written with help from AI. This is an automated estimate, not proof, and it can be wrong. If you wrote this yourself, you can ask for a person to review it.",
    "error": null,
    "elapsed_s": 0.36,
    "sent": "Effective communication is a cornerstone of successful leadership. Lea"
  },
  {
    "status": 200,
    "guess": "ai",
    "confidence": 0.8274,
    "model_score": 0.8696,
    "style_score": 0.7993,
    "label": "We think this was probably written with help from AI. This is an automated estimate, not proof, and it can be wrong. If you wrote this yourself, you can ask for a person to review it.",
    "error": null,
    "elapsed_s": 0.231,
    "sent": "Financial literacy is a critical life skill that empowers individuals "
  },
  {
    "status": 200,
    "guess": "unsure",
    "confidence": 0.7405,
    "model_score": 0.7408,
    "style_score": 0.7403,
    "label": "We can't tell whether this was written by a person or with help from AI. That happens often with clear, simple writing, and it is not a finding against the writer. We aren't marking this as AI.",
    "error": null,
    "elapsed_s": 0.241,
    "sent": "Reading offers numerous benefits that extend far beyond entertainment."
  },
  {
    "status": 200,
    "guess": "unsure",
    "confidence": 0.5134,
    "model_score": 0.6836,
    "style_score": 0.4,
    "label": "We can't tell whether this was written by a person or with help from AI. That happens often with clear, simple writing, and it is not a finding against the writer. We aren't marking this as AI.",
    "error": null,
    "elapsed_s": 0.149,
    "sent": "My room was messy after a busy week. I washed my clothes and put the b"
  },
  {
    "status": 200,
    "guess": "unsure",
    "confidence": 0.5414,
    "model_score": 0.8572,
    "style_score": 0.4,
    "label": "We can't tell whether this was written by a person or with help from AI. That happens often with clear, simple writing, and it is not a finding against the writer. We aren't marking this as AI.",
    "error": null,
    "elapsed_s": 0.186,
    "sent": "I visited a friend last weekend, and we talked about school, work, and"
  },
  {
    "status": 200,
    "guess": "unsure",
    "confidence": 0.61,
    "model_score": 0.7526,
    "style_score": 0.5149,
    "label": "We can't tell whether this was written by a person or with help from AI. That happens often with clear, simple writing, and it is not a finding against the writer. We aren't marking this as AI.",
    "error": null,
    "elapsed_s": 0.148,
    "sent": "When I started learning to cook, I made many small mistakes. I burned "
  },
  {
    "status": 200,
    "guess": "unsure",
    "confidence": 0.5403,
    "model_score": 0.5432,
    "style_score": 0.5383,
    "label": "We can't tell whether this was written by a person or with help from AI. That happens often with clear, simple writing, and it is not a finding against the writer. We aren't marking this as AI.",
    "error": null,
    "elapsed_s": 0.231,
    "sent": "Remote work has changed a lot about how offices run. People get more f"
  },
  {
    "status": 200,
    "guess": "unsure",
    "confidence": 0.4421,
    "model_score": 0.6522,
    "style_score": 0.2054,
    "label": "We can't tell whether this was written by a person or with help from AI. That happens often with clear, simple writing, and it is not a finding against the writer. We aren't marking this as AI.",
    "error": null,
    "elapsed_s": 0.261,
    "sent": "I went to the woods because I wished to live deliberately, to front on"
  },
  {
    "status": 200,
    "guess": "unsure",
    "confidence": 0.5349,
    "model_score": 0.6681,
    "style_score": 0.4461,
    "label": "We can't tell whether this was written by a person or with help from AI. That happens often with clear, simple writing, and it is not a finding against the writer. We aren't marking this as AI.",
    "error": null,
    "elapsed_s": 0.229,
    "sent": "Getting enough sleep matters more than most people think. When I start"
  }
]
```

*Criterion 4: appeal path* (`run_eval.py::run_appeals` → `app.py::submit`, `app.py::appeal`, `app.py::get_log`)

```json
[
  {
    "kind": "real",
    "sent": "The implementation of the proposed framework demonstrates significant ",
    "submit_status": 200,
    "content_id": "8cf8dbd1-d13d-4974-8fe0-356f4ef4abff",
    "appeal_status": 200,
    "returned_status": "under_review",
    "log_entry": {
      "content_id": "8cf8dbd1-d13d-4974-8fe0-356f4ef4abff",
      "creator_id": "eval_appeal_1",
      "event": "appeal",
      "reasoning": "I wrote this for a work report. It is formal because the report is formal.",
      "status": "under_review",
      "timestamp": "2026-11-10T03:15:34+00:00"
    }
  },
  {
    "kind": "real",
    "sent": "Remote work has changed a lot about how offices run. People get more f",
    "submit_status": 200,
    "content_id": "0c8b6a2b-89f3-4acd-a35f-31052e96fee3",
    "appeal_status": 200,
    "returned_status": "under_review",
    "log_entry": {
      "content_id": "0c8b6a2b-89f3-4acd-a35f-31052e96fee3",
      "creator_id": "eval_appeal_2",
      "event": "appeal",
      "reasoning": "This is my own post about working from home.",
      "status": "under_review",
      "timestamp": "2026-11-10T03:15:35+00:00"
    }
  },
  {
    "kind": "real",
    "sent": "Getting enough sleep matters more than most people think. When I start",
    "submit_status": 200,
    "content_id": "efdd7cfd-b230-4399-b733-0782f2bd9b34",
    "appeal_status": 200,
    "returned_status": "under_review",
    "log_entry": {
      "content_id": "efdd7cfd-b230-4399-b733-0782f2bd9b34",
      "creator_id": "eval_appeal_3",
      "event": "appeal",
      "reasoning": "I wrote this about my own sleep schedule.",
      "status": "under_review",
      "timestamp": "2026-11-10T03:15:35+00:00"
    }
  },
  {
    "kind": "real",
    "sent": "I went to the woods because I wished to live deliberately, to front on",
    "submit_status": 200,
    "content_id": "c38bb5a5-f7a1-4234-9b92-7d45a8810a0b",
    "appeal_status": 200,
    "returned_status": "under_review",
    "log_entry": {
      "content_id": "c38bb5a5-f7a1-4234-9b92-7d45a8810a0b",
      "creator_id": "eval_appeal_4",
      "event": "appeal",
      "reasoning": "This is a passage from Walden, quoted for a class essay.",
      "status": "under_review",
      "timestamp": "2026-11-10T03:15:35+00:00"
    }
  },
  {
    "kind": "real",
    "sent": "Remote work has fundamentally reshaped the modern workplace. Employees",
    "submit_status": 200,
    "content_id": "3fff2544-7f13-44c8-ad36-0b5f0b26cb62",
    "appeal_status": 200,
    "returned_status": "under_review",
    "log_entry": {
      "content_id": "3fff2544-7f13-44c8-ad36-0b5f0b26cb62",
      "creator_id": "eval_appeal_5",
      "event": "appeal",
      "reasoning": "My manager and I drafted this together. No AI was used.",
      "status": "under_review",
      "timestamp": "2026-11-10T03:15:36+00:00"
    }
  },
  {
    "kind": "fake",
    "content_id": "00000000-0000-0000-0000-000000000000",
    "appeal_status": 404,
    "response": {
      "error": "not_found",
      "message": "No submission with that content_id."
    }
  },
  {
    "kind": "fake",
    "content_id": "not-a-real-content-id",
    "appeal_status": 404,
    "response": {
      "error": "not_found",
      "message": "No submission with that content_id."
    }
  },
  {
    "kind": "fake",
    "content_id": "8fcb7efd-c3c2-431a-808b-85423b7eea6X",
    "appeal_status": 404,
    "response": {
      "error": "not_found",
      "message": "No submission with that content_id."
    }
  }
]
```

*Criterion 5: bad input* (`app.py::submit` → `app.py::_refuse`)

```json
[
  {
    "status": 400,
    "guess": null,
    "confidence": null,
    "model_score": null,
    "style_score": null,
    "label": null,
    "error": "invalid_input",
    "elapsed_s": 0.01,
    "sent": "(empty text)"
  },
  {
    "status": 400,
    "guess": null,
    "confidence": null,
    "model_score": null,
    "style_score": null,
    "label": null,
    "error": "invalid_input",
    "elapsed_s": 0.009,
    "sent": "{\"creator_id\": \"eval_bad_missing\"}"
  },
  {
    "status": 400,
    "guess": null,
    "confidence": null,
    "model_score": null,
    "style_score": null,
    "label": null,
    "error": "invalid_input",
    "elapsed_s": 0.009,
    "sent": "{\"text\": null, \"creator_id\": \"eval_bad_null\"}"
  },
  {
    "status": 413,
    "guess": null,
    "confidence": null,
    "model_score": null,
    "style_score": null,
    "label": null,
    "error": "text_too_long",
    "elapsed_s": 0.011,
    "sent": "word word word word word word word word word word word word word word "
  }
]
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

       [x] Rate limiting on /submit, with your numbers justified
       [x] The pasted run of status codes showing 429
       [x] Audit log with all nine fields, recording rejections too
       [x] The attack set run IN FULL, every input with an outcome
       [x] Ten audit log entries pasted as text
       [x] Five criteria across three trials
       [ ] A diagnosis for every miss and every broken attack — stage AND mechanism
       [ ] One improvement, with Attack Run — After
       [ ] What's Still Broken, including the production trade
       [ ] At least four new commits
       [ ] The SAME repository URL as last unit

     Do not delete and recreate this repository.
     ═════════════════════════════════════════════════════════════════════ -->

---

📖 **How to run this project: [RUNNING.md](RUNNING.md)**
