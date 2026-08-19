---
name: Hypothesis Template
description: Five-component hypothesis template, good/bad examples, falsifiability check, and what belongs here vs. in evidence analysis stage.
---

# Hypothesis Template

## Full Template

```
We believe [change X]
will [move metric Y in direction Z]
for [audience A],
because [causal reason R].
```

---

## Component Definitions

| Component | What it is | What it is NOT |
|---|---|---|
| **Change X** | The specific thing being built or modified | A vague improvement ("better UX") |
| **Metric Y** | A single measurable outcome | A list of possible indicators |
| **Direction Z** | Increase / decrease / maintain | "improve" (not directional) |
| **Audience A** | A segmentable user group | "users" or "everyone" |
| **Reason R** | The mechanism that causes the effect | "because it's better" |

---

## Examples

### Weak
> "We believe redesigning the onboarding will improve retention because new users struggle."

Problems:
- **Change:** "redesigning" is too vague — what design change exactly?
- **Direction:** "improve" is not a direction
- **Reason:** "users struggle" describes a symptom, not a mechanism

### Strong
> "We believe adding a progress bar to the 4-step onboarding flow will increase 7-day retention for users on their first session, because visible progress reduces abandonment by signaling that the end is near."

All five components present. The causal mechanism is explicit and testable.

---

### Weak
> "We think putting Chat with FeatBit AI Skills in the top nav will increase usage."

Problems:
- **Audience:** undefined
- **Reason:** missing entirely

### Strong
> "We believe moving Chat with FeatBit AI Skills from the sidebar to the top nav will increase weekly active usage among users who have created at least one flag, because top-nav placement increases feature discoverability for already-engaged users who don't explore the sidebar."

---

## Falsifiability Check

Before finalizing, ask: "Under what result would we conclude this hypothesis was wrong?"

If the answer involves specific, observable data (e.g., "if 7-day retention does not increase for this audience over a 2-week window"), the hypothesis is testable.

If the answer is vague (e.g., "if it doesn't feel right"), the hypothesis needs more work.

---

## What Belongs Here vs. in Evidence Analysis

| Hypothesis layer | Evidence analysis layer |
|---|---|
| "Retention will increase" | "Retention increased by 3.2 percentage points" |
| "For new users" | "Across 1,240 new user sessions" |
| "Because of reduced abandonment" | "Step 2 drop-off decreased from 41% to 29%" |

The hypothesis states the direction and mechanism. The evidence layer fills in the numbers.

---

## Hypothesis Validation Checklist

- [ ] Change X is specific enough for two engineers to build the same thing
- [ ] Metric Y is a single metric (not a list)
- [ ] Direction Z is stated explicitly (increase / decrease / maintain)
- [ ] Audience A can be segmented in analysis
- [ ] Reason R describes a mechanism, not a preference
- [ ] The hypothesis can be falsified by observable data
