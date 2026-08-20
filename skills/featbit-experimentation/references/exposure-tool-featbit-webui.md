---
name: Tool Adapter: FeatBit Web UI
description: FeatBit web UI operations: when to use, targeting rules setup, multi-variant flag configuration, and audit trail review.
---

# Tool Adapter: FeatBit Web UI

**Vendor:** FeatBit  
**Tool type:** Web UI (browser, FeatBit management console)  
**Default stage:** exposure control (CF-03/04)

The FeatBit web UI is the complete management interface — it covers every flag operation (create, enable, disable, rollout, targeting, archive, delete) plus capabilities that only exist in the UI (multi-variant editors, targeting rule chains, sendToExperiment, audit trail, RBAC).

When the FeatBit experimentation MCP server exposes feature-flag tools, prefer MCP for agent-executed create/read/update/toggle operations after explicit user approval, and use the web UI for visual review, complex manual targeting, audit inspection, or RBAC decisions.

## TOC

- [When to Use the Web UI](#when-to-use-the-web-ui)
- [Operations Reference](#operations-reference)
- [Targeting Rules](#targeting-rules)
- [Multi-Variant Flags](#multi-variant-flags)
- [Audit Trail and Compliance](#audit-trail-and-compliance)

---

## When to Use the Web UI

| Situation | Use |
|---|---|
| Fine-grained targeting rules (segments, attribute conditions, multi-rule chains) | Web UI |
| Multi-variant flags (string, number, JSON variation types) | Web UI |
| Experiment configuration (sendToExperiment for A/B data collection) | Web UI |
| Audit trail review — who changed what and when | Web UI |
| RBAC management — who can operate flags in production | Web UI |
| Agent-executed flag create / read / targeting update | MCP; mutations require explicit user approval and `confirmedByUser: true` |
| Agent-executed flag enable / disable | MCP; requires explicit user approval and `confirmedByUser: true` |
| Change-request update when reviewer ids are known | MCP; requires explicit user approval and `confirmedByUser: true` |

---

## Operations Reference

| Operation | Web UI location |
|---|---|
| Create a feature flag | Feature Flags → New Flag |
| Add variants (control / treatment values) | Flag detail → Variations |
| Configure multi-variant type (string, number, JSON) | Flag detail → Variations → type selector |
| Set percentage rollout | Flag detail → Targeting → Percentage rollout |
| Add individual user targeting rules | Flag detail → Targeting → Individual rules |
| Add segment targeting | Flag detail → Targeting → Segment rules |
| Add custom attribute rules | Flag detail → Targeting → Attribute conditions |
| Configure sendToExperiment | Flag detail → Targeting → sendToExperiment |
| Enable the flag | Flag list toggle or Flag detail toggle |
| Disable (rollback) the flag | Same toggle — all users revert to the off value immediately |
| Archive a flag | Flag detail → Archive |
| View audit trail | Flag detail → Activity tab |
| Manage team permissions | Settings → Members → Roles |

---

## Targeting Rules

Targeting rules define which users see a specific variation before percentage rollout applies. Rules are evaluated top-to-bottom — the first matching rule wins.

**Common rule types:**
- **Individual user** — match by user key (e.g., internal testers, stakeholders before launch)
- **Segment** — match users in a predefined segment (e.g., beta group, paying customers)
- **Custom attribute** — match on a user property such as `plan`, `country`, or `region`

**Setup order:**
1. Set targeting rules first (while flag is still OFF)
2. Verify rules cover protected audiences (who must NOT see the candidate)
3. Set rollout percentage to initial value (5–10%)
4. After explicit user approval, enable the flag with `featbit_experiment_toggle_feature_flag` and `confirmedByUser: true` when MCP is available, or with the UI toggle when it is not

Protected audiences that must NOT see a new variant should be in an individual rule returning the default OFF variant. This rule must be placed above the rollout rule.

---

## Multi-Variant Flags

Boolean flags (on/off) cover most rollout scenarios. Use multi-variant flags when:
- Testing UI copy — multiple text variants, one per user group
- A/B/n testing with more than two groups
- Configuration-style flags that return a string or JSON value

In the web UI: **Flag detail → Variations** to add or edit variants. Each variant requires a unique key that matches the value checked in the code:

```js
// Boolean flag
const enabled = client.variation("my-flag", false);

// Multi-variant flag
const layout = client.variation("checkout-layout", "default");  // returns "default", "compact", or "expanded"
```

---

## Audit Trail and Compliance

Every change made through the web UI is recorded with:
- Timestamp
- Acting user (authenticated identity)
- Change description (field modified and old/new value)

To view: **Flag detail → Activity tab**

This audit log matters when:
- Investigating an incident (who enabled this flag in production, and when?)
- Compliance review (was this change authorized by the right role?)
- Change management processes that require traceable approvals

MCP-driven flag operations also record changes in the audit trail through the FeatBit API. The acting identity is the MCP token owner or configured service identity, so production setups should use traceable identities rather than shared personal tokens.
