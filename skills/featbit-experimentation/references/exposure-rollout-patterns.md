---
name: Rollout Patterns
description: Vendor-agnostic progressive exposure strategy: conservative start, protected audiences, expansion criteria, rollback triggers, and anti-patterns.
---

# Rollout Patterns

Vendor-agnostic guidance for progressive exposure strategy. These patterns apply regardless of which flag management tool is used.

---

## Conservative Default Start

Begin at **5–10%** of eligible traffic unless specific circumstances justify more.

**Reasons to start lower (1–5%):**
- Change affects a critical user path (payment, auth, core feature)
- Guardrail metrics are expensive to recover if degraded
- The causal mechanism in the hypothesis is uncertain

**Reasons to start higher (20–25%):**
- Internal or beta users only in the first phase
- Change is cosmetic with no backend implications
- Strong prior evidence from previous experiments on the same surface

---

## Protected Audiences

Define before exposure begins. These users must NOT receive the candidate variant without deliberate decision:

**Never expose candidate without explicit approval:**
- High-value customers during contract renewal periods
- Users in active support tickets related to the area being changed
- Users in regulated jurisdictions if the change affects compliance-sensitive flows

**Typical protected-last audiences (see candidate last):**
- Power users who depend on stable behavior
- API integration users whose systems may break on behavioral change

**Typical first-look audiences (see candidate first):**
- Internal team members
- Opted-in beta users
- Users who explicitly requested the feature

---

## Expansion Criteria

Define in advance. Do not make expansion decisions based on feel.

Standard expansion criteria:
- Primary metric moving in expected direction (or neutral — not negative)
- All guardrail metrics within acceptable range
- Minimum observation window elapsed (typically 1–2 full business cycles)
- No anomalies in error rates or latency for the candidate variant

Standard expansion schedule:
```
Phase 1:   5%  → observe for [X days]
Phase 2:  25%  → observe for [Y days]
Phase 3:  50%  → observe for [Z days]
Phase 4: 100%  → full rollout or cleanup
```

Adapt the schedule to the criticality of the change and the pace of your user traffic. Low-traffic features may need longer windows at each phase.

---

## Rollback Triggers

Define these explicitly before Phase 1 begins — not after the problem appears.

- Guardrail metric degrades by more than [threshold] → **immediate rollback**
- Error rate on candidate variant exceeds baseline by [multiplier] → **immediate rollback**
- Primary metric moves in the wrong direction consistently after [minimum sample] → **pause and investigate**
- Unexpected user segment is receiving candidate variant → **pause and fix targeting**

Persist the thresholds with `featbit_experiment_update_experiment` under `constraints`.

---

## Expansion Anti-Patterns

**Expanding because "it looks fine"**  
Expansion requires defined evidence criteria, not intuition. If no criteria were defined, define them before expanding.

**Skipping phases under schedule pressure**  
Creates irreversibility risk. If the change causes harm at 50%, the window to limit exposure has already closed.

**100% rollout before the measurement window closes**  
Kills the ability to measure the impact of the change. You cannot run an A/B comparison without a control group.

**No rollback plan defined**  
Reversibility must be operational, not theoretical. The flag must be in a state where a single disable command takes effect immediately.
