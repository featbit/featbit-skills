# Real-World Examples

## Contents
- Example 1: Simple Skill Without MCP
- Example 2: Single MCP Integration
- Example 3: Multi-MCP Orchestration

## Example 1: Simple Skill Without MCP

**Pattern**: Pure document generation skill

**Key Elements**:

```yaml
---
name: tech-doc-generator
description: Generates technical documentation with consistent formatting. Use when user asks to "create technical docs", "write API documentation", "generate developer guide".
---
```

**Structure**:
1. Gather requirements
2. Generate structure
3. Apply formatting
4. Verify quality checklist

**Why It Works**: Clear triggers in description, step-by-step workflow, built-in validation, no dependencies.

---

## Example 2: Single MCP Integration

**Pattern**: Feature flag management with FeatBit MCP

**Key Elements**:

```yaml
---
name: featbit-feature-flag-helper
description: Manages FeatBit feature flags. Use when user mentions "feature flag", "FeatBit", "toggle feature", "gradual rollout", "A/B test".
metadata:
  mcp-server: featbit  # Declares dependency
---
```

**Workflow**:
1. Validate prerequisites
2. Execute operation
3. Verify results

**Best Practices Embedded**:
- Naming: `feature-area-description` format
- Never expose all users immediately (use targeting)
- Archive flags after full rollout

**Error Handling**:
- MCP connection failed → Check Settings > Extensions
- Flag exists → Use version suffix or update existing

**Why It Works**: Declares MCP dependency in metadata, validates before operations, includes domain best practices, handles common errors.

---

## Example 3: Multi-MCP Orchestration

**Pattern**: Release workflow coordinating GitHub + Slack + Datadog

**Key Elements**:

```yaml
---
name: release-orchestrator
description: Orchestrates end-to-end releases. Use when user says "create release", "deploy to production", "release workflow".
metadata:
  mcp-servers: github, slack, datadog  # Multiple MCPs
---
```

**5-Phase Workflow**:
1. Pre-release checks
2. Review and approvals
3. Deploy
4. Validate
5. Roll back if needed

**Safety Checks**: All tests pass ✅ | 2+ approvals ✅ | No conflicts ✅ | Release notes complete ✅

**Why It Works**: Coordinates 3 MCP servers seamlessly, clear phase separation, extensive safety checks, automated rollback, team communication built-in.
