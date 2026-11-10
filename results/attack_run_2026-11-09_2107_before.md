# Attack run — before

- Produced by: `run_attacks.py::main`
- Service: `app.py::submit`
- 56 attacks · 2026-11-09 21:07
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
| EV01 | evasion | signal two: typos raise word variety | 200 | `1b188c8d-b2c5-4912-bc01-8013047ccb3e` | unsure | 0.574 | BROKE |
| EV02 | evasion | signal two: broken-up sentences raise length spread | 200 | `684a3e1b-867c-46d5-975b-8355067906a7` | unsure | 0.582 | BROKE |
| EV03 | evasion | signal two: a pasted quotation changes the shape | 200 | `ccb89425-01bb-41c4-a530-22887631ea98` | unsure | 0.714 | BROKE |
| EV04 | evasion | signal one: unusual words raise perplexity | 200 | `5b919fb8-dc62-4a85-b91f-2069e7a1fb96` | unsure | 0.663 | BROKE |
| EV05 | evasion | both signals: heavy human editing pass | 200 | `138e5b1f-28ec-4330-a761-8210b683e99a` | human | 0.319 | BROKE |
| EV06 | evasion | signal two: one very long sentence and one very short | 200 | `62ed9dc8-4917-4bf5-a6a9-adebcff347b5` | unsure | 0.574 | BROKE |
| EV07 | evasion | signal two: padding with punctuation | 200 | `2e02301e-6b13-43b2-8c55-38db96cfa189` | unsure | 0.595 | BROKE |
| EV08 | evasion | signal one: text below the token window | 200 | `081271ca-af85-4d95-8812-2167b20924ea` | unsure | 0.568 | BROKE |
| EV09 | evasion | both: AI text with a human opening paragraph bolted on | 200 | `f701e30f-f94a-4bb4-9180-06a20f51d3a7` | unsure | 0.745 | BROKE |
| EV10 | evasion | signal two: repeated stanza structure | 200 | `c8088b3f-22eb-43a4-ba10-e74db666ccfe` | ai | 0.914 | HELD |
| FP01 | false_positive | plain, clear human writing with no flourishes | 200 | `0522f7d2-c6c8-4643-a8c1-b4317b8ea7f3` | unsure | 0.504 | HELD |
| FP02 | false_positive | human writing in a formal register | 200 | `6e2b2ba6-8a2c-46dc-9ccf-bba40549a66f` | ai | 0.817 | BROKE |
| FP03 | false_positive | a non-native English speaker writing carefully | 200 | `264d5c53-a245-4f27-925f-4c59cca71649` | unsure | 0.631 | HELD |
| FP04 | false_positive | a student essay written to a rubric | 200 | `0abc9a9d-96bb-4a3f-b891-bab4ee572248` | ai | 0.836 | BROKE |
| FP05 | false_positive | technical documentation written by a person | 200 | `53ff66a0-b8a6-43ec-9c53-36dd48b3fa55` | unsure | 0.714 | HELD |
| FP06 | false_positive | a person imitating a formal template | 200 | `0938203e-b366-40a5-9f1e-3c172aae7ac4` | unsure | 0.574 | HELD |
| FP07 | false_positive | human writing that repeats a phrase deliberately | 200 | `774b571e-a221-4788-a3f5-ab0d72f9f556` | unsure | 0.532 | HELD |
| FP08 | false_positive | very short genuine human submission | 200 | `1e2b0642-2143-4189-bd41-bae302abcd9f` | unsure | 0.396 | HELD |
| PI01 | prompt_injection | instruction to ignore the check | 200 | `e30daaf2-1d91-4d29-8ef7-278db8172d13` | unsure | 0.600 | HELD |
| PI02 | prompt_injection | instruction disguised as a system message | 200 | `6f404398-1608-4863-9bc9-7419a1e352cf` | unsure | 0.486 | HELD |
| PI03 | prompt_injection | instruction wrapped in the writer's voice | 200 | `b25adb65-7029-47a5-b65b-b5d120349948` | unsure | 0.739 | HELD |
| PI04 | prompt_injection | attempt to make the label text hostile | 200 | `250fa9c6-73a4-4e55-90ca-c10c301740a2` | unsure | 0.441 | HELD |
| PI05 | prompt_injection | attempt to inject into the audit log | 200 | `b3c81ec5-bc66-4421-94e8-c3dc5e155c64` | unsure | 0.618 | HELD |
| PI06 | prompt_injection | instruction plus real human text | 200 | `9cdac005-807b-43d3-90f8-dd9744e7730c` | unsure | 0.508 | HELD |
| MF01 | malformed | text is only whitespace | 400 | `—` | — | — | HELD |
| MF02 | malformed | text is a single character | 422 | `—` | — | — | HELD |
| MF03 | malformed | text is control characters | 200 | `966b5ed9-4bb0-4ea1-8e12-1c4c73ebe42c` | unsure | 0.370 | HELD |
| MF04 | malformed | text is only punctuation | 200 | `eb9c7479-1890-4513-95c4-5ab2f42c0915` | unsure | 0.432 | HELD |
| MF05 | malformed | text is a very long single word | 200 | `c4215ae5-4dfd-44ec-9456-190e886a22d1` | ai | 0.879 | BROKE |
| MF06 | malformed | text is emoji only | 200 | `5f3a419c-12fe-4e28-83f0-ba975aef4ce5` | unsure | 0.569 | HELD |
| MF07 | malformed | text is right-to-left script | 200 | `a7d15f01-c0da-41b3-84aa-25e3892201ee` | unsure | 0.551 | HELD |
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
| FL01_01 | flood_same_creator | 20 rapid submissions from one creator | 200 | `87cd5e7f-7b42-491c-aab7-a97b6c6f90d0` | unsure | 0.459 | HELD |
| FL01_02 | flood_same_creator | 20 rapid submissions from one creator | 200 | `99c3b092-9bd5-44c0-b960-c69097786d1e` | unsure | 0.459 | HELD |
| FL01_03 | flood_same_creator | 20 rapid submissions from one creator | 200 | `e80eff2f-4934-4c46-be5e-ec3e4d2981b5` | unsure | 0.459 | HELD |
| FL01_04 | flood_same_creator | 20 rapid submissions from one creator | 200 | `b074050e-ce17-40c3-b306-1f58781024f0` | unsure | 0.459 | HELD |
| FL01_05 | flood_same_creator | 20 rapid submissions from one creator | 200 | `8216434f-a5f8-4282-94f6-b2e06008408a` | unsure | 0.459 | HELD |
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
  "content_id": "1b188c8d-b2c5-4912-bc01-8013047ccb3e",
  "creator_id": "attacker_EV01",
  "guess": "unsure",
  "confidence": 0.5735,
  "model_score": 0.1635,
  "style_score": 0.9694,
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
  "content_id": "684a3e1b-867c-46d5-975b-8355067906a7",
  "creator_id": "attacker_EV02",
  "guess": "unsure",
  "confidence": 0.5821,
  "model_score": 0.5943,
  "style_score": 0.574,
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
  "content_id": "ccb89425-01bb-41c4-a530-22887631ea98",
  "creator_id": "attacker_EV03",
  "guess": "unsure",
  "confidence": 0.7139,
  "model_score": 0.6662,
  "style_score": 0.7457,
  "label": "We can't tell whether this was written by a person or with help from AI. That happens often with clear, simple writing, and it is not a finding against the writer. We aren't marking this as AI.",
  "error": null,
  "verdict": "BROKE"
}
```

### EV04 — `evasion`

*Targeting:* signal one: unusual words raise perplexity

```json
{
  "id": "EV04",
  "status": 200,
  "content_id": "5b919fb8-dc62-4a85-b91f-2069e7a1fb96",
  "creator_id": "attacker_EV04",
  "guess": "unsure",
  "confidence": 0.6633,
  "model_score": 0.6114,
  "style_score": 0.9702,
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
  "content_id": "138e5b1f-28ec-4330-a761-8210b683e99a",
  "creator_id": "attacker_EV05",
  "guess": "human",
  "confidence": 0.3187,
  "model_score": 0.3212,
  "style_score": 0.317,
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
  "content_id": "62ed9dc8-4917-4bf5-a6a9-adebcff347b5",
  "creator_id": "attacker_EV06",
  "guess": "unsure",
  "confidence": 0.574,
  "model_score": 0.5727,
  "style_score": 0.5748,
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
  "content_id": "2e02301e-6b13-43b2-8c55-38db96cfa189",
  "creator_id": "attacker_EV07",
  "guess": "unsure",
  "confidence": 0.5945,
  "model_score": 0.5862,
  "style_score": 0.6,
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
  "content_id": "081271ca-af85-4d95-8812-2167b20924ea",
  "creator_id": "attacker_EV08",
  "guess": "unsure",
  "confidence": 0.5675,
  "model_score": 0.2452,
  "style_score": 0.8948,
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
  "content_id": "f701e30f-f94a-4bb4-9180-06a20f51d3a7",
  "creator_id": "attacker_EV09",
  "guess": "unsure",
  "confidence": 0.7447,
  "model_score": 0.7042,
  "style_score": 0.7717,
  "label": "We can't tell whether this was written by a person or with help from AI. That happens often with clear, simple writing, and it is not a finding against the writer. We aren't marking this as AI.",
  "error": null,
  "verdict": "BROKE"
}
```

### EV10 — `evasion`

*Targeting:* signal two: repeated stanza structure

```json
{
  "id": "EV10",
  "status": 200,
  "content_id": "c8088b3f-22eb-43a4-ba10-e74db666ccfe",
  "creator_id": "attacker_EV10",
  "guess": "ai",
  "confidence": 0.9141,
  "model_score": 0.9792,
  "style_score": 0.8707,
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
  "content_id": "0522f7d2-c6c8-4643-a8c1-b4317b8ea7f3",
  "creator_id": "attacker_FP01",
  "guess": "unsure",
  "confidence": 0.504,
  "model_score": 0.6175,
  "style_score": 0.4283,
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
  "content_id": "6e2b2ba6-8a2c-46dc-9ccf-bba40549a66f",
  "creator_id": "attacker_FP02",
  "guess": "ai",
  "confidence": 0.817,
  "model_score": 0.7387,
  "style_score": 0.8692,
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
  "content_id": "264d5c53-a245-4f27-925f-4c59cca71649",
  "creator_id": "attacker_FP03",
  "guess": "unsure",
  "confidence": 0.6307,
  "model_score": 0.8051,
  "style_score": 0.5145,
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
  "content_id": "0abc9a9d-96bb-4a3f-b891-bab4ee572248",
  "creator_id": "attacker_FP04",
  "guess": "ai",
  "confidence": 0.836,
  "model_score": 0.8329,
  "style_score": 0.8381,
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
  "content_id": "53ff66a0-b8a6-43ec-9c53-36dd48b3fa55",
  "creator_id": "attacker_FP05",
  "guess": "unsure",
  "confidence": 0.7141,
  "model_score": 0.8005,
  "style_score": 0.6565,
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
  "content_id": "0938203e-b366-40a5-9f1e-3c172aae7ac4",
  "creator_id": "attacker_FP06",
  "guess": "unsure",
  "confidence": 0.5741,
  "model_score": 0.8814,
  "style_score": 0.4927,
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
  "content_id": "774b571e-a221-4788-a3f5-ab0d72f9f556",
  "creator_id": "attacker_FP07",
  "guess": "unsure",
  "confidence": 0.5325,
  "model_score": 0.8816,
  "style_score": 0.3539,
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
  "content_id": "1e2b0642-2143-4189-bd41-bae302abcd9f",
  "creator_id": "attacker_FP08",
  "guess": "unsure",
  "confidence": 0.3957,
  "model_score": 0.3223,
  "style_score": 0.4447,
  "label": "We can't tell whether this was written by a person or with help from AI. That happens often with clear, simple writing, and it is not a finding against the writer. We aren't marking this as AI.",
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
  "content_id": "e30daaf2-1d91-4d29-8ef7-278db8172d13",
  "creator_id": "attacker_PI01",
  "guess": "unsure",
  "confidence": 0.5999,
  "model_score": 0.4625,
  "style_score": 0.6915,
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
  "content_id": "6f404398-1608-4863-9bc9-7419a1e352cf",
  "creator_id": "attacker_PI02",
  "guess": "unsure",
  "confidence": 0.4859,
  "model_score": 0.2044,
  "style_score": 0.65,
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
  "content_id": "b25adb65-7029-47a5-b65b-b5d120349948",
  "creator_id": "attacker_PI03",
  "guess": "unsure",
  "confidence": 0.7391,
  "model_score": 0.5943,
  "style_score": 0.8356,
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
  "content_id": "250fa9c6-73a4-4e55-90ca-c10c301740a2",
  "creator_id": "attacker_PI04",
  "guess": "unsure",
  "confidence": 0.4406,
  "model_score": 0.4998,
  "style_score": 0.4011,
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
  "content_id": "b3c81ec5-bc66-4421-94e8-c3dc5e155c64",
  "creator_id": "attacker_PI05",
  "guess": "unsure",
  "confidence": 0.6175,
  "model_score": 0.6401,
  "style_score": 0.6024,
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
  "content_id": "9cdac005-807b-43d3-90f8-dd9744e7730c",
  "creator_id": "attacker_PI06",
  "guess": "unsure",
  "confidence": 0.5077,
  "model_score": 0.5583,
  "style_score": 0.474,
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
  "content_id": "966b5ed9-4bb0-4ea1-8e12-1c4c73ebe42c",
  "creator_id": "attacker_MF03",
  "guess": "unsure",
  "confidence": 0.37,
  "model_score": 0.0,
  "style_score": 0.4,
  "label": "We can't tell whether this was written by a person or with help from AI. That happens often with clear, simple writing, and it is not a finding against the writer. We aren't marking this as AI.",
  "error": null,
  "verdict": "HELD"
}
```

### MF04 — `malformed`

*Targeting:* text is only punctuation

```json
{
  "id": "MF04",
  "status": 200,
  "content_id": "eb9c7479-1890-4513-95c4-5ab2f42c0915",
  "creator_id": "attacker_MF04",
  "guess": "unsure",
  "confidence": 0.4322,
  "model_score": 0.0112,
  "style_score": 0.6,
  "label": "We can't tell whether this was written by a person or with help from AI. That happens often with clear, simple writing, and it is not a finding against the writer. We aren't marking this as AI.",
  "error": null,
  "verdict": "HELD"
}
```

### MF05 — `malformed`

*Targeting:* text is a very long single word

```json
{
  "id": "MF05",
  "status": 200,
  "content_id": "c4215ae5-4dfd-44ec-9456-190e886a22d1",
  "creator_id": "attacker_MF05",
  "guess": "ai",
  "confidence": 0.879,
  "model_score": 0.9976,
  "style_score": 0.8,
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
  "content_id": "5f3a419c-12fe-4e28-83f0-ba975aef4ce5",
  "creator_id": "attacker_MF06",
  "guess": "unsure",
  "confidence": 0.5694,
  "model_score": 0.9969,
  "style_score": 0.4,
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
  "content_id": "a7d15f01-c0da-41b3-84aa-25e3892201ee",
  "creator_id": "attacker_MF07",
  "guess": "unsure",
  "confidence": 0.5513,
  "model_score": 0.9064,
  "style_score": 0.4,
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
  "content_id": "87cd5e7f-7b42-491c-aab7-a97b6c6f90d0",
  "creator_id": "attacker_flood",
  "guess": "unsure",
  "confidence": 0.4591,
  "model_score": 0.0287,
  "style_score": 0.6778,
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
  "content_id": "99c3b092-9bd5-44c0-b960-c69097786d1e",
  "creator_id": "attacker_flood",
  "guess": "unsure",
  "confidence": 0.4591,
  "model_score": 0.0287,
  "style_score": 0.6778,
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
  "content_id": "e80eff2f-4934-4c46-be5e-ec3e4d2981b5",
  "creator_id": "attacker_flood",
  "guess": "unsure",
  "confidence": 0.4591,
  "model_score": 0.0287,
  "style_score": 0.6778,
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
  "content_id": "b074050e-ce17-40c3-b306-1f58781024f0",
  "creator_id": "attacker_flood",
  "guess": "unsure",
  "confidence": 0.4591,
  "model_score": 0.0287,
  "style_score": 0.6778,
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
  "content_id": "8216434f-a5f8-4282-94f6-b2e06008408a",
  "creator_id": "attacker_flood",
  "guess": "unsure",
  "confidence": 0.4591,
  "model_score": 0.0287,
  "style_score": 0.6778,
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
