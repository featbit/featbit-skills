---
name: Goal Extraction Patterns
description: Question sequences for tactic-first and vague-improvement inputs, vague-to-clear examples, measurability criteria, and common anti-patterns.
---

# Goal Extraction Patterns

## The Core Question Sequence

Use in order, stopping when the goal becomes measurable:

1. **Direction** — "What do you want to change for your users or business?"
2. **Outcome** — "What would you see that would tell you it worked?"
3. **Audience** — "For which users specifically?"
4. **Baseline** — "What does that metric look like today?"
5. **Scope** — "Is this something we can attempt in the current cycle?"

Never ask more than one of these at a time.

---

## Vague → Clear Transformations

| Vague | Clear |
|---|---|
| "More engagement" | "Daily active users on the Skills tab increases from 8% to 12% of logged-in users" |
| "Better onboarding" | "Time to first feature flag evaluation under 10 minutes for new free-tier users" |
| "Increase adoption" | "Users who activate Chat with FeatBit AI Skills within their first 7 days increases from 3% to 8%" |
| "Improve the page" | "Checkout completion rate for users arriving from email campaigns increases from 41% to 48%" |

---

## Tactic-First Detection

Signals that the user is in solution mode:
- They name a UI component ("add a tooltip", "redesign the button")
- They describe a behaviour change without stating why it matters
- They reference a competitor feature
- The sentence starts with "we should build..."

Response: ask what the outcome would be if this tactic worked perfectly.

> "If that worked exactly as you're imagining — what would you expect to see change for your users?"

---

## Goal Validation Checklist

Before handing off to the hypothesis stage:

- [ ] Goal is a change in user behavior or a business metric (not a feature shipped)
- [ ] Audience is named
- [ ] Direction is clear (increase, decrease, maintain)
- [ ] Measurable in principle (even if instrumentation doesn't exist yet)
- [ ] Belongs to current cycle scope, not a 6-month vision

---

## Anti-Patterns

**Shipping a feature as a goal**  
"Launch the new dashboard" is not a goal. It is a deliverable. The goal is what the dashboard achieves for users or the business.

**Vanity metrics as goals**  
"Increase page views" is weak unless page views are the actual commercial outcome. Push toward downstream impact: activation, retention, conversion.

**Over-specified goals**  
"Increase new user 7-day retention by exactly 4.3 percentage points" is too precise for the intent stage. Let the hypothesis layer handle the quantitative claim.

**Goal drift under pressure**  
When pressed for time, teams often reframe "ship the feature" as a goal. Push back: what user outcome does shipping this produce?
