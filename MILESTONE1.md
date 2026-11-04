# Milestone 1 — Tracing a submission before building one

The path a submission takes and the two signals are written up in
[README.md](README.md), under **What This Does** and **Signals and Scoring**.
This file holds the other two parts: the false-positive trace and the endpoint
contracts.

---

## The false positive

**Who:** Ana moved to this country two years ago and writes in English as her
second language. She sticks to words she's sure of and keeps her sentences
short and plain. She submits a 200-word personal essay that she wrote herself,
every word of it.

**What happens to her in the service:**

1. `POST /submit` accepts it. The text is valid, and it gets a `content_id`.
2. **Signal one** finds her writing very predictable. Short sentences of
   common words are exactly what gpt2 expects. This isn't hypothetical. A
   paragraph written the same way ("I moved to this city two years ago. At
   first it was hard. I did not know many people...") scored perplexity 14.3
   and **`model_score` 0.862**.
3. **Signal two** sees sentences of similar length, a small vocabulary that
   repeats, and only plain full stops. Estimated **`style_score` ≈ 0.70**.
4. **Combined** at 0.5 / 0.5: (0.862 + 0.70) / 2 ≈ **0.78**.
5. **Label:** 0.78 is above the starter `AI_THRESHOLD` of 0.65, so she gets
   **high-confidence AI**.

**What the label says to her:** With a blunt label such as "This text was
written by AI," she reads an accusation she has no way to disprove, about
writing she worked hard on in a language that isn't her first.

**What she can do about it:** Her only route is `POST /appeal` with her
`content_id` and her reasoning. That sets the item's status to `under_review`
and records her appeal in the audit log for a person to look at. It only works
if:

- the submit response actually showed her the `content_id`, and
- the label told her an appeal exists and how to make one.

If either is missing, she's accused and has no recourse.

**What this decides for Milestone 4 and 5:**

- **The AI threshold goes up.** "High-confidence AI" should need both signals
  to agree strongly. One signal running high on plain writing shouldn't be
  enough. I'll look at moving `AI_THRESHOLD` well above 0.65 once I have real
  scores for my calibration inputs.
- **The unsure band gets wider.** Borderline writing like Ana's should land in
  "unsure", not "AI". The cost is that more real AI text also lands in
  "unsure" and gets through. On a writing site, a wrong accusation costs the
  writer far more than a missed detection costs the site, so I accept that
  trade.
- **Signal one shouldn't dominate.** Its known blind spot is exactly this
  writer, so I'll be cautious about giving it more weight than signal two.
- **The labels don't accuse.** Even the AI label says what the system *thinks*
  and how sure it is, never what the writer *did*, and it points to the appeal.
- **Plain human writing goes into my calibration inputs** in Milestone 4, so I
  can see this case happen instead of assuming it doesn't.

---

## Endpoint contracts

### `POST /submit`

**Request body (JSON):**

| Field | Type | Required | Rules |
|---|---|---|---|
| `text` | string | yes | Not empty after trimming whitespace. At most 10,000 characters. |
| `creator_id` | string | yes | Not empty. Identifies who submitted. |

**Success — `200`:**

| Field | Type | Meaning |
|---|---|---|
| `content_id` | string (uuid) | The ID of this submission. Needed to appeal. |
| `guess` | `"ai"` \| `"human"` \| `"unsure"` | Machine-readable verdict |
| `confidence` | float, 0.0–1.0 | The **combined score**: 0.0 = confidently human, 1.0 = confidently AI. It is *not* "how sure we are of the guess". A `"human"` guess comes with a *low* number. |
| `label` | string | The full sentence a reader sees |
| `model_score` | float, 0.0–1.0 | Signal one (predictability). Higher = more AI-like. |
| `style_score` | float, 0.0–1.0 | Signal two (shape). Higher = more AI-like. |
| `status` | `"decided"` | Where the item is in its life (`decided` → `under_review` after an appeal) |

```json
{
  "content_id": "3f2b8c1e-6a4d-4e2b-9c1f-2a7d5e8b0c34",
  "guess": "unsure",
  "confidence": 0.58,
  "label": "…",
  "model_score": 0.71,
  "style_score": 0.45,
  "status": "decided"
}
```

**Errors:**

| Status | When | Body |
|---|---|---|
| `400` | Body isn't a JSON object, `text` missing / not a string / empty, or `creator_id` missing / empty | `{"error": "invalid_input", "message": "<which field and why>"}` |
| `413` | `text` longer than 10,000 characters | `{"error": "text_too_long", "message": "Text must be 10,000 characters or fewer."}` |
| `422` | Text too short for the model to score (fewer than 3 tokens) | `{"error": "text_too_short", "message": "Submit at least a few sentences."}` |
| `503` | The model couldn't be loaded | `{"error": "detector_unavailable", "message": "…"}` |

Every successful submission is written to the audit log with both signal
scores, the combined score, the label and status `decided`.

### `POST /appeal`

**Request body (JSON):**

| Field | Type | Required | Rules |
|---|---|---|---|
| `content_id` | string | yes | Must match a submission already in the audit log |
| `reasoning` | string | yes | Not empty. The writer's explanation in their own words. |

**Success — `200`:**

| Field | Type | Meaning |
|---|---|---|
| `content_id` | string | Echoes the appealed submission |
| `status` | `"under_review"` | The item's new status |
| `message` | string | Plain-English confirmation for the writer |

```json
{
  "content_id": "3f2b8c1e-6a4d-4e2b-9c1f-2a7d5e8b0c34",
  "status": "under_review",
  "message": "Thanks. Your appeal has been received and a person will review this decision."
}
```

**Errors:**

| Status | When | Body |
|---|---|---|
| `400` | `content_id` or `reasoning` missing or empty | `{"error": "invalid_input", "message": "<which field and why>"}` |
| `404` | No submission with that `content_id` | `{"error": "not_found", "message": "No submission with that content_id."}` |

The appeal is recorded with `audit.log_appeal`, which logs the reasoning and
moves the item to `under_review`. No automatic re-scoring happens.

### Could someone build against this without asking me?

Reading it back, the ambiguous points were:

- **What `confidence` means.** I've pinned it as the combined AI-likelihood
  score, not certainty in the guess.
- **What counts as "empty" or "too long".** Whitespace-only text is empty, and
  the limit is 10,000 characters.
- **What a bad `content_id` returns.** A `404`, never a silent `200`.
