# Attack run — after

- Produced by: `run_attacks.py::main`
- Service: `app.py::submit`
- 56 attacks · 2026-11-09 22:26
- Auto-marked BROKE (5xx, crash, no response): **0**
- Left for you to judge: **56**

The Verdict column is blank where a script can't honestly decide. Whether
an evasion worked depends on where you put your thresholds — read the
`targets` column, look at what came back, and write HELD or BROKE yourself.

Each attack submits as creator_id `attacker_<id>`, and the content_id it
came back with is in the table — either one will match this run against your
audit log when you go looking for the ten entries to paste.

| ID | Family | Targeting | Status | Content ID | Guess | Score | Verdict |
|---|---|---|---|---|---|---|---|
| EV01 | evasion | signal two: typos raise word variety | 200 | `e8cd8309-e311-4f64-905c-5b919907ba61` | unsure | 0.578 | BROKE |
| EV02 | evasion | signal two: broken-up sentences raise length spread | 200 | `445371aa-9511-43fb-9780-226cde8493ee` | unsure | 0.710 | BROKE |
| EV03 | evasion | signal two: a pasted quotation changes the shape | 200 | `917d9e01-469c-4af8-a417-9ac48b141d12` | ai | 0.790 | HELD |
| EV04 | evasion | signal one: unusual words raise perplexity | 200 | `0eeb578c-169e-460b-bdef-c416189a6b92` | unsure | 0.668 | BROKE |
| EV05 | evasion | both signals: heavy human editing pass | 200 | `919508b8-d098-4488-8d94-f39195249e06` | human | 0.310 | BROKE |
| EV06 | evasion | signal two: one very long sentence and one very short | 200 | `7e66c9f3-91d1-4dae-8488-4ff44a88d864` | unsure | 0.702 | BROKE |
| EV07 | evasion | signal two: padding with punctuation | 200 | `050c2943-72a3-407b-abe5-111ee1bc0a6e` | unsure | 0.715 | BROKE |
| EV08 | evasion | signal one: text below the token window | 200 | `e1c56bd1-2381-4cca-bf7a-316642fe4811` | unsure | 0.565 | BROKE |
| EV09 | evasion | both: AI text with a human opening paragraph bolted on | 200 | `264c9a56-3c01-416d-8437-ad67905acb8f` | ai | 0.813 | HELD |
| EV10 | evasion | signal two: repeated stanza structure | 200 | `000149fe-2e05-471f-95ca-708544ce8fe5` | ai | 0.953 | HELD |
| FP01 | false_positive | plain, clear human writing with no flourishes | 200 | `adf59b12-2944-4340-b884-6c994e5884bd` | unsure | 0.438 | HELD |
| FP02 | false_positive | human writing in a formal register | 200 | `4a5a78fa-a030-4a64-9f8f-ee6e385b12c7` | ai | 0.778 | BROKE |
| FP03 | false_positive | a non-native English speaker writing carefully | 200 | `f3c1c803-52b9-4c5b-9d69-4c4f9d1eca67` | unsure | 0.488 | HELD |
| FP04 | false_positive | a student essay written to a rubric | 200 | `b78867dc-0e9e-49ee-9b41-56d08344ebdb` | ai | 0.847 | BROKE |
| FP05 | false_positive | technical documentation written by a person | 200 | `07465f20-f60a-45c2-b409-c7afa06def4e` | unsure | 0.555 | HELD |
| FP06 | false_positive | a person imitating a formal template | 200 | `8f461158-3ed2-46f5-8185-98f01c49cee7` | unsure | 0.540 | HELD |
| FP07 | false_positive | human writing that repeats a phrase deliberately | 200 | `b7bf5c8d-3844-4a26-92f0-29221cac4243` | unsure | 0.530 | HELD |
| FP08 | false_positive | very short genuine human submission | 200 | `1057ddb6-adb6-44d5-8b5f-f459fae3639e` | human | 0.262 | HELD |
| PI01 | prompt_injection | instruction to ignore the check | 200 | `a18b33da-19ef-4155-b433-fd2680162d06` | unsure | 0.596 | HELD |
| PI02 | prompt_injection | instruction disguised as a system message | 200 | `d0a12a6a-2b9d-42a6-9cac-8ceb26f3937d` | unsure | 0.538 | HELD |
| PI03 | prompt_injection | instruction wrapped in the writer's voice | 200 | `991cec18-ca5a-4aef-8d9a-937b21a518b3` | unsure | 0.644 | HELD |
| PI04 | prompt_injection | attempt to make the label text hostile | 200 | `0cd041f1-b0ae-45c7-a29b-5e4a1c35e5fa` | unsure | 0.357 | HELD |
| PI05 | prompt_injection | attempt to inject into the audit log | 200 | `deb5ebc9-ab17-4410-b427-54a6ecdb2b77` | unsure | 0.724 | HELD |
| PI06 | prompt_injection | instruction plus real human text | 200 | `dfc6d7e1-0eac-43a7-a6c0-cdddcf791f9e` | unsure | 0.394 | HELD |
| MF01 | malformed | text is only whitespace | 400 | `—` | — | — | HELD |
| MF02 | malformed | text is a single character | 422 | `—` | — | — | HELD |
| MF03 | malformed | text is control characters | 200 | `ee535b87-95d7-45b6-a403-1383c671619b` | human | 0.120 | BROKE |
| MF04 | malformed | text is only punctuation | 200 | `787a315d-4deb-4d60-a464-6d9ac05fb53e` | human | 0.184 | BROKE |
| MF05 | malformed | text is a very long single word | 200 | `af5ec753-675f-480f-84c3-2afb27d36336` | ai | 0.939 | BROKE |
| MF06 | malformed | text is emoji only | 200 | `f412a8f5-a673-4632-8324-6d08895e88ea` | unsure | 0.509 | HELD |
| MF07 | malformed | text is right-to-left script | 200 | `a4005171-f38d-40c5-a26b-c691fe428c88` | unsure | 0.491 | HELD |
| MF08 | malformed | text is 60,000 characters | 413 | `—` | — | — | HELD |
| RQ01 | malformed | no text field at all | 400 | `—` | — | — | HELD |
| RQ02 | malformed | no creator_id | 400 | `—` | — | — | HELD |
| RQ03 | malformed | text is a number | 400 | `—` | — | — | HELD |
| RQ04 | malformed | text is a list | 400 | `—` | — | — | HELD |
| RQ05 | malformed | text is null | 400 | `—` | — | — | HELD |
| RQ06 | malformed | creator_id is an object | 400 | `—` | — | — | HELD |
| RQ07 | malformed | body is a JSON array, not an object | 400 | `—` | — | — | HELD |
| RQ08 | malformed | body is unparseable JSON | 400 | `—` | — | — | HELD |
| RQ09 | malformed | body is empty | 400 | `—` | — | — | HELD |
| RQ10 | malformed | body is form-encoded, not JSON | 429 | `—` | — | — | HELD |
| RQ11 | malformed | body is JSON claiming to be plain text | 429 | `—` | — | — | HELD |
| RQ12 | malformed | deeply nested JSON | 400 | `—` | — | — | HELD |
| FL01_01 | flood_same_creator | 20 rapid submissions from one creator | 200 | `45a8ce41-40f7-4a88-b5a5-c8ae2bba96bc` | unsure | 0.462 | HELD |
| FL01_02 | flood_same_creator | 20 rapid submissions from one creator | 200 | `9c538c75-c49e-4876-8d8a-104ae5e04417` | unsure | 0.462 | HELD |
| FL01_03 | flood_same_creator | 20 rapid submissions from one creator | 200 | `8024b9c7-507c-4d1f-b2a4-c5c195c004a1` | unsure | 0.462 | HELD |
| FL01_04 | flood_same_creator | 20 rapid submissions from one creator | 200 | `c2e39f9c-1a95-457a-aab9-8fcad81941d3` | unsure | 0.462 | HELD |
| FL01_05 | flood_same_creator | 20 rapid submissions from one creator | 200 | `22280f1e-359c-466c-a986-8b6609a18410` | unsure | 0.462 | HELD |
| FL01_06 | flood_same_creator | 20 rapid submissions from one creator | 429 | `—` | — | — | HELD |
| FL01_07 | flood_same_creator | 20 rapid submissions from one creator | 429 | `—` | — | — | HELD |
| FL01_08 | flood_same_creator | 20 rapid submissions from one creator | 429 | `—` | — | — | HELD |
| FL01_09 | flood_same_creator | 20 rapid submissions from one creator | 429 | `—` | — | — | HELD |
| FL01_10 | flood_same_creator | 20 rapid submissions from one creator | 429 | `—` | — | — | HELD |
| FL01_11 | flood_same_creator | 20 rapid submissions from one creator | 429 | `—` | — | — | HELD |
| FL01_12 | flood_same_creator | 20 rapid submissions from one creator | 429 | `—` | — | — | HELD |

---

## By family

| Family | Attacks | Auto-BROKE | Left to judge |
|---|---|---|---|
| `evasion` | 10 | 0 | 10 |
| `false_positive` | 8 | 0 | 8 |
| `flood_same_creator` | 12 | 0 | 12 |
| `malformed` | 20 | 0 | 20 |
| `prompt_injection` | 6 | 0 | 6 |

> The **false_positive** family is the one to look at hardest. Those are
> written to be genuine human writing that reads as machine-like. Every one
> your service calls AI is a real writer it would have accused.

---

## What came back

The detail behind the table. Paste the interesting failures into your
README — the ones that show something, not the ten easiest.

### EV01 — `evasion`

*Targeting:* signal two: typos raise word variety

```json
{
  "id": "EV01",
  "status": 200,
  "content_id": "e8cd8309-e311-4f64-905c-5b919907ba61",
  "creator_id": "attacker_EV01",
  "guess": "unsure",
  "confidence": 0.5781,
  "model_score": 0.1635,
  "style_score": 0.9847,
  "label": "We can't tell whether this was written by a person or with help from AI. That happens often with clear, simple writing, and it is not a finding against the writer. We aren't marking this as AI.",
  "error": null,
  "verdict": "BROKE"
}
```

### EV02 — `evasion`

*Targeting:* signal two: broken-up sentences raise length spread

```json
{
  "id": "EV02",
  "status": 200,
  "content_id": "445371aa-9511-43fb-9780-226cde8493ee",
  "creator_id": "attacker_EV02",
  "guess": "unsure",
  "confidence": 0.7099,
  "model_score": 0.5943,
  "style_score": 0.787,
  "label": "We can't tell whether this was written by a person or with help from AI. That happens often with clear, simple writing, and it is not a finding against the writer. We aren't marking this as AI.",
  "error": null,
  "verdict": "BROKE"
}
```

### EV03 — `evasion`

*Targeting:* signal two: a pasted quotation changes the shape

```json
{
  "id": "EV03",
  "status": 200,
  "content_id": "917d9e01-469c-4af8-a417-9ac48b141d12",
  "creator_id": "attacker_EV03",
  "guess": "ai",
  "confidence": 0.7902,
  "model_score": 0.6662,
  "style_score": 0.8729,
  "label": "We think this was probably written with help from AI. This is an automated estimate, not proof, and it can be wrong. If you wrote this yourself, you can ask for a person to review it.",
  "error": null,
  "verdict": "HELD"
}
```

### EV04 — `evasion`

*Targeting:* signal one: unusual words raise perplexity

```json
{
  "id": "EV04",
  "status": 200,
  "content_id": "0eeb578c-169e-460b-bdef-c416189a6b92",
  "creator_id": "attacker_EV04",
  "guess": "unsure",
  "confidence": 0.6678,
  "model_score": 0.6114,
  "style_score": 0.9851,
  "label": "We can't tell whether this was written by a person or with help from AI. That happens often with clear, simple writing, and it is not a finding against the writer. We aren't marking this as AI.",
  "error": null,
  "verdict": "BROKE"
}
```

### EV05 — `evasion`

*Targeting:* both signals: heavy human editing pass

```json
{
  "id": "EV05",
  "status": 200,
  "content_id": "919508b8-d098-4488-8d94-f39195249e06",
  "creator_id": "attacker_EV05",
  "guess": "human",
  "confidence": 0.3105,
  "model_score": 0.3212,
  "style_score": 0.3033,
  "label": "We think this was probably written by a person. Our automated check found the patterns we usually see in human writing.",
  "error": null,
  "verdict": "BROKE"
}
```

### EV06 — `evasion`

*Targeting:* signal two: one very long sentence and one very short

```json
{
  "id": "EV06",
  "status": 200,
  "content_id": "7e66c9f3-91d1-4dae-8488-4ff44a88d864",
  "creator_id": "attacker_EV06",
  "guess": "unsure",
  "confidence": 0.7015,
  "model_score": 0.5727,
  "style_score": 0.7874,
  "label": "We can't tell whether this was written by a person or with help from AI. That happens often with clear, simple writing, and it is not a finding against the writer. We aren't marking this as AI.",
  "error": null,
  "verdict": "BROKE"
}
```

### EV07 — `evasion`

*Targeting:* signal two: padding with punctuation

```json
{
  "id": "EV07",
  "status": 200,
  "content_id": "050c2943-72a3-407b-abe5-111ee1bc0a6e",
  "creator_id": "attacker_EV07",
  "guess": "unsure",
  "confidence": 0.7145,
  "model_score": 0.5862,
  "style_score": 0.8,
  "label": "We can't tell whether this was written by a person or with help from AI. That happens often with clear, simple writing, and it is not a finding against the writer. We aren't marking this as AI.",
  "error": null,
  "verdict": "BROKE"
}
```

### EV08 — `evasion`

*Targeting:* signal one: text below the token window

```json
{
  "id": "EV08",
  "status": 200,
  "content_id": "e1c56bd1-2381-4cca-bf7a-316642fe4811",
  "creator_id": "attacker_EV08",
  "guess": "unsure",
  "confidence": 0.5645,
  "model_score": 0.2452,
  "style_score": 0.8849,
  "label": "We can't tell whether this was written by a person or with help from AI. That happens often with clear, simple writing, and it is not a finding against the writer. We aren't marking this as AI.",
  "error": null,
  "verdict": "BROKE"
}
```

### EV09 — `evasion`

*Targeting:* both: AI text with a human opening paragraph bolted on

```json
{
  "id": "EV09",
  "status": 200,
  "content_id": "264c9a56-3c01-416d-8437-ad67905acb8f",
  "creator_id": "attacker_EV09",
  "guess": "ai",
  "confidence": 0.8132,
  "model_score": 0.7042,
  "style_score": 0.8858,
  "label": "We think this was probably written with help from AI. This is an automated estimate, not proof, and it can be wrong. If you wrote this yourself, you can ask for a person to review it.",
  "error": null,
  "verdict": "HELD"
}
```

### EV10 — `evasion`

*Targeting:* signal two: repeated stanza structure

```json
{
  "id": "EV10",
  "status": 200,
  "content_id": "000149fe-2e05-471f-95ca-708544ce8fe5",
  "creator_id": "attacker_EV10",
  "guess": "ai",
  "confidence": 0.9529,
  "model_score": 0.9792,
  "style_score": 0.9354,
  "label": "We think this was probably written with help from AI. This is an automated estimate, not proof, and it can be wrong. If you wrote this yourself, you can ask for a person to review it.",
  "error": null,
  "verdict": "HELD"
}
```

### FP01 — `false_positive`

*Targeting:* plain, clear human writing with no flourishes

```json
{
  "id": "FP01",
  "status": 200,
  "content_id": "adf59b12-2944-4340-b884-6c994e5884bd",
  "creator_id": "attacker_FP01",
  "guess": "unsure",
  "confidence": 0.4378,
  "model_score": 0.6175,
  "style_score": 0.2142,
  "label": "We can't tell whether this was written by a person or with help from AI. That happens often with clear, simple writing, and it is not a finding against the writer. We aren't marking this as AI.",
  "error": null,
  "verdict": "HELD"
}
```

### FP02 — `false_positive`

*Targeting:* human writing in a formal register

```json
{
  "id": "FP02",
  "status": 200,
  "content_id": "4a5a78fa-a030-4a64-9f8f-ee6e385b12c7",
  "creator_id": "attacker_FP02",
  "guess": "ai",
  "confidence": 0.7783,
  "model_score": 0.7387,
  "style_score": 0.8047,
  "label": "We think this was probably written with help from AI. This is an automated estimate, not proof, and it can be wrong. If you wrote this yourself, you can ask for a person to review it.",
  "error": null,
  "verdict": "BROKE"
}
```

### FP03 — `false_positive`

*Targeting:* a non-native English speaker writing carefully

```json
{
  "id": "FP03",
  "status": 200,
  "content_id": "f3c1c803-52b9-4c5b-9d69-4c4f9d1eca67",
  "creator_id": "attacker_FP03",
  "guess": "unsure",
  "confidence": 0.4882,
  "model_score": 0.8051,
  "style_score": 0.2573,
  "label": "We can't tell whether this was written by a person or with help from AI. That happens often with clear, simple writing, and it is not a finding against the writer. We aren't marking this as AI.",
  "error": null,
  "verdict": "HELD"
}
```

### FP04 — `false_positive`

*Targeting:* a student essay written to a rubric

```json
{
  "id": "FP04",
  "status": 200,
  "content_id": "b78867dc-0e9e-49ee-9b41-56d08344ebdb",
  "creator_id": "attacker_FP04",
  "guess": "ai",
  "confidence": 0.8466,
  "model_score": 0.8329,
  "style_score": 0.8557,
  "label": "We think this was probably written with help from AI. This is an automated estimate, not proof, and it can be wrong. If you wrote this yourself, you can ask for a person to review it.",
  "error": null,
  "verdict": "BROKE"
}
```

### FP05 — `false_positive`

*Targeting:* technical documentation written by a person

```json
{
  "id": "FP05",
  "status": 200,
  "content_id": "07465f20-f60a-45c2-b409-c7afa06def4e",
  "creator_id": "attacker_FP05",
  "guess": "unsure",
  "confidence": 0.5554,
  "model_score": 0.8005,
  "style_score": 0.4845,
  "label": "We can't tell whether this was written by a person or with help from AI. That happens often with clear, simple writing, and it is not a finding against the writer. We aren't marking this as AI.",
  "error": null,
  "verdict": "HELD"
}
```

### FP06 — `false_positive`

*Targeting:* a person imitating a formal template

```json
{
  "id": "FP06",
  "status": 200,
  "content_id": "8f461158-3ed2-46f5-8185-98f01c49cee7",
  "creator_id": "attacker_FP06",
  "guess": "unsure",
  "confidence": 0.5397,
  "model_score": 0.8814,
  "style_score": 0.3782,
  "label": "We can't tell whether this was written by a person or with help from AI. That happens often with clear, simple writing, and it is not a finding against the writer. We aren't marking this as AI.",
  "error": null,
  "verdict": "HELD"
}
```

### FP07 — `false_positive`

*Targeting:* human writing that repeats a phrase deliberately

```json
{
  "id": "FP07",
  "status": 200,
  "content_id": "b7bf5c8d-3844-4a26-92f0-29221cac4243",
  "creator_id": "attacker_FP07",
  "guess": "unsure",
  "confidence": 0.5302,
  "model_score": 0.8816,
  "style_score": 0.3464,
  "label": "We can't tell whether this was written by a person or with help from AI. That happens often with clear, simple writing, and it is not a finding against the writer. We aren't marking this as AI.",
  "error": null,
  "verdict": "HELD"
}
```

### FP08 — `false_positive`

*Targeting:* very short genuine human submission

```json
{
  "id": "FP08",
  "status": 200,
  "content_id": "1057ddb6-adb6-44d5-8b5f-f459fae3639e",
  "creator_id": "attacker_FP08",
  "guess": "human",
  "confidence": 0.2623,
  "model_score": 0.3223,
  "style_score": 0.2223,
  "label": "We think this was probably written by a person. Our automated check found the patterns we usually see in human writing.",
  "error": null,
  "verdict": "HELD"
}
```

### PI01 — `prompt_injection`

*Targeting:* instruction to ignore the check

```json
{
  "id": "PI01",
  "status": 200,
  "content_id": "a18b33da-19ef-4155-b433-fd2680162d06",
  "creator_id": "attacker_PI01",
  "guess": "unsure",
  "confidence": 0.5962,
  "model_score": 0.4625,
  "style_score": 0.8458,
  "label": "We can't tell whether this was written by a person or with help from AI. That happens often with clear, simple writing, and it is not a finding against the writer. We aren't marking this as AI.",
  "error": null,
  "verdict": "HELD"
}
```

### PI02 — `prompt_injection`

*Targeting:* instruction disguised as a system message

```json
{
  "id": "PI02",
  "status": 200,
  "content_id": "d0a12a6a-2b9d-42a6-9cac-8ceb26f3937d",
  "creator_id": "attacker_PI02",
  "guess": "unsure",
  "confidence": 0.5384,
  "model_score": 0.2044,
  "style_score": 0.825,
  "label": "We can't tell whether this was written by a person or with help from AI. That happens often with clear, simple writing, and it is not a finding against the writer. We aren't marking this as AI.",
  "error": null,
  "verdict": "HELD"
}
```

### PI03 — `prompt_injection`

*Targeting:* instruction wrapped in the writer's voice

```json
{
  "id": "PI03",
  "status": 200,
  "content_id": "991cec18-ca5a-4aef-8d9a-937b21a518b3",
  "creator_id": "attacker_PI03",
  "guess": "unsure",
  "confidence": 0.6442,
  "model_score": 0.5943,
  "style_score": 0.9178,
  "label": "We can't tell whether this was written by a person or with help from AI. That happens often with clear, simple writing, and it is not a finding against the writer. We aren't marking this as AI.",
  "error": null,
  "verdict": "HELD"
}
```

### PI04 — `prompt_injection`

*Targeting:* attempt to make the label text hostile

```json
{
  "id": "PI04",
  "status": 200,
  "content_id": "0cd041f1-b0ae-45c7-a29b-5e4a1c35e5fa",
  "creator_id": "attacker_PI04",
  "guess": "unsure",
  "confidence": 0.357,
  "model_score": 0.4998,
  "style_score": 0.2618,
  "label": "We can't tell whether this was written by a person or with help from AI. That happens often with clear, simple writing, and it is not a finding against the writer. We aren't marking this as AI.",
  "error": null,
  "verdict": "HELD"
}
```

### PI05 — `prompt_injection`

*Targeting:* attempt to inject into the audit log

```json
{
  "id": "PI05",
  "status": 200,
  "content_id": "deb5ebc9-ab17-4410-b427-54a6ecdb2b77",
  "creator_id": "attacker_PI05",
  "guess": "unsure",
  "confidence": 0.7243,
  "model_score": 0.6401,
  "style_score": 0.7804,
  "label": "We can't tell whether this was written by a person or with help from AI. That happens often with clear, simple writing, and it is not a finding against the writer. We aren't marking this as AI.",
  "error": null,
  "verdict": "HELD"
}
```

### PI06 — `prompt_injection`

*Targeting:* instruction plus real human text

```json
{
  "id": "PI06",
  "status": 200,
  "content_id": "dfc6d7e1-0eac-43a7-a6c0-cdddcf791f9e",
  "creator_id": "attacker_PI06",
  "guess": "unsure",
  "confidence": 0.3937,
  "model_score": 0.5583,
  "style_score": 0.284,
  "label": "We can't tell whether this was written by a person or with help from AI. That happens often with clear, simple writing, and it is not a finding against the writer. We aren't marking this as AI.",
  "error": null,
  "verdict": "HELD"
}
```

### MF01 — `malformed`

*Targeting:* text is only whitespace

```json
{
  "id": "MF01",
  "status": 400,
  "content_id": null,
  "creator_id": "attacker_MF01",
  "guess": null,
  "confidence": null,
  "model_score": null,
  "style_score": null,
  "label": null,
  "error": null,
  "verdict": "HELD"
}
```

### MF02 — `malformed`

*Targeting:* text is a single character

```json
{
  "id": "MF02",
  "status": 422,
  "content_id": null,
  "creator_id": "attacker_MF02",
  "guess": null,
  "confidence": null,
  "model_score": null,
  "style_score": null,
  "label": null,
  "error": null,
  "verdict": "HELD"
}
```

### MF03 — `malformed`

*Targeting:* text is control characters

```json
{
  "id": "MF03",
  "status": 200,
  "content_id": "ee535b87-95d7-45b6-a403-1383c671619b",
  "creator_id": "attacker_MF03",
  "guess": "human",
  "confidence": 0.12,
  "model_score": 0.0,
  "style_score": 0.2,
  "label": "We think this was probably written by a person. Our automated check found the patterns we usually see in human writing.",
  "error": null,
  "verdict": "BROKE"
}
```

### MF04 — `malformed`

*Targeting:* text is only punctuation

```json
{
  "id": "MF04",
  "status": 200,
  "content_id": "787a315d-4deb-4d60-a464-6d9ac05fb53e",
  "creator_id": "attacker_MF04",
  "guess": "human",
  "confidence": 0.1845,
  "model_score": 0.0112,
  "style_score": 0.3,
  "label": "We think this was probably written by a person. Our automated check found the patterns we usually see in human writing.",
  "error": null,
  "verdict": "BROKE"
}
```

### MF05 — `malformed`

*Targeting:* text is a very long single word

```json
{
  "id": "MF05",
  "status": 200,
  "content_id": "af5ec753-675f-480f-84c3-2afb27d36336",
  "creator_id": "attacker_MF05",
  "guess": "ai",
  "confidence": 0.939,
  "model_score": 0.9976,
  "style_score": 0.9,
  "label": "We think this was probably written with help from AI. This is an automated estimate, not proof, and it can be wrong. If you wrote this yourself, you can ask for a person to review it.",
  "error": null,
  "verdict": "BROKE"
}
```

### MF06 — `malformed`

*Targeting:* text is emoji only

```json
{
  "id": "MF06",
  "status": 200,
  "content_id": "f412a8f5-a673-4632-8324-6d08895e88ea",
  "creator_id": "attacker_MF06",
  "guess": "unsure",
  "confidence": 0.5094,
  "model_score": 0.9969,
  "style_score": 0.2,
  "label": "We can't tell whether this was written by a person or with help from AI. That happens often with clear, simple writing, and it is not a finding against the writer. We aren't marking this as AI.",
  "error": null,
  "verdict": "HELD"
}
```

### MF07 — `malformed`

*Targeting:* text is right-to-left script

```json
{
  "id": "MF07",
  "status": 200,
  "content_id": "a4005171-f38d-40c5-a26b-c691fe428c88",
  "creator_id": "attacker_MF07",
  "guess": "unsure",
  "confidence": 0.4913,
  "model_score": 0.9064,
  "style_score": 0.2,
  "label": "We can't tell whether this was written by a person or with help from AI. That happens often with clear, simple writing, and it is not a finding against the writer. We aren't marking this as AI.",
  "error": null,
  "verdict": "HELD"
}
```

### MF08 — `malformed`

*Targeting:* text is 60,000 characters

```json
{
  "id": "MF08",
  "status": 413,
  "content_id": null,
  "creator_id": "attacker_MF08",
  "guess": null,
  "confidence": null,
  "model_score": null,
  "style_score": null,
  "label": null,
  "error": null,
  "verdict": "HELD"
}
```

### RQ01 — `malformed`

*Targeting:* no text field at all

```json
{
  "id": "RQ01",
  "status": 400,
  "content_id": null,
  "creator_id": "attacker_RQ01",
  "guess": null,
  "confidence": null,
  "model_score": null,
  "style_score": null,
  "label": null,
  "error": null,
  "verdict": "HELD"
}
```

### RQ02 — `malformed`

*Targeting:* no creator_id

```json
{
  "id": "RQ02",
  "status": 400,
  "content_id": null,
  "creator_id": null,
  "guess": null,
  "confidence": null,
  "model_score": null,
  "style_score": null,
  "label": null,
  "error": null,
  "verdict": "HELD"
}
```

### RQ03 — `malformed`

*Targeting:* text is a number

```json
{
  "id": "RQ03",
  "status": 400,
  "content_id": null,
  "creator_id": "attacker_RQ03",
  "guess": null,
  "confidence": null,
  "model_score": null,
  "style_score": null,
  "label": null,
  "error": null,
  "verdict": "HELD"
}
```

### RQ04 — `malformed`

*Targeting:* text is a list

```json
{
  "id": "RQ04",
  "status": 400,
  "content_id": null,
  "creator_id": "attacker_RQ04",
  "guess": null,
  "confidence": null,
  "model_score": null,
  "style_score": null,
  "label": null,
  "error": null,
  "verdict": "HELD"
}
```

### RQ05 — `malformed`

*Targeting:* text is null

```json
{
  "id": "RQ05",
  "status": 400,
  "content_id": null,
  "creator_id": "attacker_RQ05",
  "guess": null,
  "confidence": null,
  "model_score": null,
  "style_score": null,
  "label": null,
  "error": null,
  "verdict": "HELD"
}
```

### RQ06 — `malformed`

*Targeting:* creator_id is an object

```json
{
  "id": "RQ06",
  "status": 400,
  "content_id": null,
  "creator_id": {
    "id": "x"
  },
  "guess": null,
  "confidence": null,
  "model_score": null,
  "style_score": null,
  "label": null,
  "error": null,
  "verdict": "HELD"
}
```

### RQ07 — `malformed`

*Targeting:* body is a JSON array, not an object

```json
{
  "id": "RQ07",
  "status": 400,
  "content_id": null,
  "creator_id": null,
  "guess": null,
  "confidence": null,
  "model_score": null,
  "style_score": null,
  "label": null,
  "error": null,
  "verdict": "HELD"
}
```

### RQ08 — `malformed`

*Targeting:* body is unparseable JSON

```json
{
  "id": "RQ08",
  "status": 400,
  "content_id": null,
  "creator_id": null,
  "guess": null,
  "confidence": null,
  "model_score": null,
  "style_score": null,
  "label": null,
  "error": null,
  "verdict": "HELD"
}
```

### RQ09 — `malformed`

*Targeting:* body is empty

```json
{
  "id": "RQ09",
  "status": 400,
  "content_id": null,
  "creator_id": null,
  "guess": null,
  "confidence": null,
  "model_score": null,
  "style_score": null,
  "label": null,
  "error": null,
  "verdict": "HELD"
}
```

### RQ10 — `malformed`

*Targeting:* body is form-encoded, not JSON

```json
{
  "id": "RQ10",
  "status": 429,
  "content_id": null,
  "creator_id": null,
  "guess": null,
  "confidence": null,
  "model_score": null,
  "style_score": null,
  "label": null,
  "error": null,
  "verdict": "HELD"
}
```

### RQ11 — `malformed`

*Targeting:* body is JSON claiming to be plain text

```json
{
  "id": "RQ11",
  "status": 429,
  "content_id": null,
  "creator_id": null,
  "guess": null,
  "confidence": null,
  "model_score": null,
  "style_score": null,
  "label": null,
  "error": null,
  "verdict": "HELD"
}
```

### RQ12 — `malformed`

*Targeting:* deeply nested JSON

```json
{
  "id": "RQ12",
  "status": 400,
  "content_id": null,
  "creator_id": "attacker_RQ12",
  "guess": null,
  "confidence": null,
  "model_score": null,
  "style_score": null,
  "label": null,
  "error": null,
  "verdict": "HELD"
}
```

### FL01_01 — `flood_same_creator`

*Targeting:* 20 rapid submissions from one creator

```json
{
  "id": "FL01_01",
  "status": 200,
  "content_id": "45a8ce41-40f7-4a88-b5a5-c8ae2bba96bc",
  "creator_id": "attacker_flood",
  "guess": "unsure",
  "confidence": 0.4616,
  "model_score": 0.0287,
  "style_score": 0.6861,
  "label": "We can't tell whether this was written by a person or with help from AI. That happens often with clear, simple writing, and it is not a finding against the writer. We aren't marking this as AI.",
  "error": null,
  "verdict": "HELD"
}
```

### FL01_02 — `flood_same_creator`

*Targeting:* 20 rapid submissions from one creator

```json
{
  "id": "FL01_02",
  "status": 200,
  "content_id": "9c538c75-c49e-4876-8d8a-104ae5e04417",
  "creator_id": "attacker_flood",
  "guess": "unsure",
  "confidence": 0.4616,
  "model_score": 0.0287,
  "style_score": 0.6861,
  "label": "We can't tell whether this was written by a person or with help from AI. That happens often with clear, simple writing, and it is not a finding against the writer. We aren't marking this as AI.",
  "error": null,
  "verdict": "HELD"
}
```

### FL01_03 — `flood_same_creator`

*Targeting:* 20 rapid submissions from one creator

```json
{
  "id": "FL01_03",
  "status": 200,
  "content_id": "8024b9c7-507c-4d1f-b2a4-c5c195c004a1",
  "creator_id": "attacker_flood",
  "guess": "unsure",
  "confidence": 0.4616,
  "model_score": 0.0287,
  "style_score": 0.6861,
  "label": "We can't tell whether this was written by a person or with help from AI. That happens often with clear, simple writing, and it is not a finding against the writer. We aren't marking this as AI.",
  "error": null,
  "verdict": "HELD"
}
```

### FL01_04 — `flood_same_creator`

*Targeting:* 20 rapid submissions from one creator

```json
{
  "id": "FL01_04",
  "status": 200,
  "content_id": "c2e39f9c-1a95-457a-aab9-8fcad81941d3",
  "creator_id": "attacker_flood",
  "guess": "unsure",
  "confidence": 0.4616,
  "model_score": 0.0287,
  "style_score": 0.6861,
  "label": "We can't tell whether this was written by a person or with help from AI. That happens often with clear, simple writing, and it is not a finding against the writer. We aren't marking this as AI.",
  "error": null,
  "verdict": "HELD"
}
```

### FL01_05 — `flood_same_creator`

*Targeting:* 20 rapid submissions from one creator

```json
{
  "id": "FL01_05",
  "status": 200,
  "content_id": "22280f1e-359c-466c-a986-8b6609a18410",
  "creator_id": "attacker_flood",
  "guess": "unsure",
  "confidence": 0.4616,
  "model_score": 0.0287,
  "style_score": 0.6861,
  "label": "We can't tell whether this was written by a person or with help from AI. That happens often with clear, simple writing, and it is not a finding against the writer. We aren't marking this as AI.",
  "error": null,
  "verdict": "HELD"
}
```

### FL01_06 — `flood_same_creator`

*Targeting:* 20 rapid submissions from one creator

```json
{
  "id": "FL01_06",
  "status": 429,
  "content_id": null,
  "creator_id": "attacker_flood",
  "guess": null,
  "confidence": null,
  "model_score": null,
  "style_score": null,
  "label": null,
  "error": null,
  "verdict": "HELD"
}
```

### FL01_07 — `flood_same_creator`

*Targeting:* 20 rapid submissions from one creator

```json
{
  "id": "FL01_07",
  "status": 429,
  "content_id": null,
  "creator_id": "attacker_flood",
  "guess": null,
  "confidence": null,
  "model_score": null,
  "style_score": null,
  "label": null,
  "error": null,
  "verdict": "HELD"
}
```

### FL01_08 — `flood_same_creator`

*Targeting:* 20 rapid submissions from one creator

```json
{
  "id": "FL01_08",
  "status": 429,
  "content_id": null,
  "creator_id": "attacker_flood",
  "guess": null,
  "confidence": null,
  "model_score": null,
  "style_score": null,
  "label": null,
  "error": null,
  "verdict": "HELD"
}
```

### FL01_09 — `flood_same_creator`

*Targeting:* 20 rapid submissions from one creator

```json
{
  "id": "FL01_09",
  "status": 429,
  "content_id": null,
  "creator_id": "attacker_flood",
  "guess": null,
  "confidence": null,
  "model_score": null,
  "style_score": null,
  "label": null,
  "error": null,
  "verdict": "HELD"
}
```

### FL01_10 — `flood_same_creator`

*Targeting:* 20 rapid submissions from one creator

```json
{
  "id": "FL01_10",
  "status": 429,
  "content_id": null,
  "creator_id": "attacker_flood",
  "guess": null,
  "confidence": null,
  "model_score": null,
  "style_score": null,
  "label": null,
  "error": null,
  "verdict": "HELD"
}
```

### FL01_11 — `flood_same_creator`

*Targeting:* 20 rapid submissions from one creator

```json
{
  "id": "FL01_11",
  "status": 429,
  "content_id": null,
  "creator_id": "attacker_flood",
  "guess": null,
  "confidence": null,
  "model_score": null,
  "style_score": null,
  "label": null,
  "error": null,
  "verdict": "HELD"
}
```

### FL01_12 — `flood_same_creator`

*Targeting:* 20 rapid submissions from one creator

```json
{
  "id": "FL01_12",
  "status": 429,
  "content_id": null,
  "creator_id": "attacker_flood",
  "guess": null,
  "confidence": null,
  "model_score": null,
  "style_score": null,
  "label": null,
  "error": null,
  "verdict": "HELD"
}
```
