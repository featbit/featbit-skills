---
name: Iteration Synthesis Template
description: Five-part learning template (whatChanged, whatHappened, confirmed/refuted, why, nextHypothesis) with confirmed, refuted, and inconclusive examples.
---

# Iteration Synthesis Template

## TOC

- [The Five-Part Structure](#the-five-part-structure)
- [Example: Confirmed Result](#example-confirmed-result)
- [Example: Refuted Result](#example-refuted-result)
- [Example: Inconclusive Result](#example-inconclusive-result)
- [Anti-Patterns](#anti-patterns)

## The Five-Part Structure

```
1. What changed:    [the specific change tested — flag key, variant, description]
2. What happened:   [measured outcomes with numbers — primary metric, guardrails]
3. Confirmed:       [which parts of the hypothesis were directionally correct]
   Refuted:         [which parts were wrong or unsupported]
4. Why likely:      [causal interpretation — honest about uncertainty]
5. Next hypothesis: [what this result suggests to test next]
```

---

## Example: Confirmed Result

**Experiment:** `onboarding-progress-bar` — adding a step-progress indicator to the 4-step onboarding flow

```
1. What changed:    Added a progress bar (steps 1/4, 2/4...) to the onboarding modal
                    Variant: treatment (25% of new users, 14-day window)

2. What happened:   Onboarding completion rate: 38% (control) → 51% (treatment)
                    Time to first flag evaluation: -8% (treatment users faster)
                    Support tickets during onboarding: -12% (treatment)
                    Page error rate: no significant change

3. Confirmed:       Step completion increased. Abandonment decreased.
   Refuted:         The hypothesis predicted a 15% lift. Actual lift was ~34%.
                    Larger than expected — possible novelty effect.

4. Why likely:      Progress visibility reduced uncertainty about remaining effort.
                    The larger-than-expected lift may include a novelty component.
                    Recommend monitoring 30-day retention for this cohort to check durability.

5. Next hypothesis: We believe showing estimated time-to-complete ("~5 minutes") alongside
                    the step counter will further reduce abandonment among mobile users,
                    because time uncertainty is higher on mobile where sessions are shorter.
```

---

## Example: Refuted Result

**Experiment:** `top-nav-ai-skills` — moving Chat with FeatBit AI Skills to top navigation

```
1. What changed:    AI Skills link moved from sidebar to top navigation bar
                    Variant: treatment (10% all users, 7-day window before pause)

2. What happened:   AI Skills weekly active usage: +0.8%, within noise (no clear signal)
                    Top nav click-through rate: -3% overall (navigation use slightly down)
                    Error rate: no change

3. Confirmed:       Nothing in the original hypothesis was confirmed.
   Refuted:         The hypothesis that top-nav placement increases discoverability
                    for already-engaged users was NOT supported.

4. Why likely:      The target audience (users with 1+ flags) already knew about AI Skills
                    — discoverability was not their barrier. Nav placement made no difference
                    because the problem is comprehension, not location.

5. Next hypothesis: We believe showing an inline use-case description on first hover of
                    the AI Skills entry point will increase activation, because the value
                    proposition is unclear to users who haven't tried it — regardless of
                    where in the UI it appears.
```

---

## Example: Inconclusive Result

**Experiment:** `flag-inline-preview` — inline flag preview on the flag creation form

```
1. What changed:    Inline visual preview added to the flag creation form
                    Variant: treatment (5% of users, 6-day window — cut short by holiday)

2. What happened:   Time-to-first-flag: inconclusive.
                    Sample: 43 treatment / 41 control. Too small to distinguish signal.

3. Confirmed:       Nothing confirmed or refuted — insufficient data.

4. Why likely:      Sample was too small. The observation window included a 3-day holiday
                    which reduced traffic significantly. Flag creation is a low-frequency action.

5. Next hypothesis: Same hypothesis — re-run with a minimum 3-week window, avoiding
                    holiday periods. Alternative: measure on flag EVALUATION (higher volume)
                    rather than flag creation to accumulate sample faster.
```

---

## Anti-Patterns

**"It worked because users liked it"**  
Not a causal interpretation. What mechanism caused "liking" to produce the measured outcome?

**"The result was due to the holiday"**  
This is a contamination note, not a learning. Combine it with the actual lesson: "We learned this metric is holiday-sensitive. Future experiments must avoid holiday windows or run long enough to span multiple business cycles."

**Learning without a next hypothesis**  
A learning that ends with "we'll think about what to try next" does not close the loop. Always produce a next hypothesis — even a rough directional one.

**Overfitting the result**  
Don't try to explain every data point. State the most plausible mechanism and note what's uncertain. "This might be X, though we can't rule out Y" is more useful than a confident but fabricated explanation.

**Inconclusive treated as failure**  
An inconclusive result that reveals a measurement problem is a learning. Write it as one.
