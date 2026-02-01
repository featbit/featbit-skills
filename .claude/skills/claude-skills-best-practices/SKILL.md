---
name: claude-skills-best-practices
description: Expert guidance for creating high-quality Claude skills. Use when building new skills, reviewing existing skills, or asking about skill structure, frontmatter, triggers, testing, or distribution. Covers YAML formatting, progressive disclosure, MCP integration, and troubleshooting.
license: MIT
metadata:
  author: FeatBit
  version: 1.0.0
  category: skill-development
---

# Claude Skills Best Practices

Expert guidance for creating production-ready Claude skills, aligned to the official Best Practices. Additional details from prior guides appear only as supplements.

## When to Use This Skill

Activate this skill when:
- Building a new skill from scratch
- Reviewing or improving existing skills
- Debugging skill triggering issues
- Integrating skills with MCP servers
- Planning skill architecture and structure
- Troubleshooting skill upload or execution problems

## Core Principles (Best Practices)

### Concise Is Key

Your Skill shares a limited context window with system prompts, conversation history, and other Skills. Keep SKILL.md short and focused.

**Assume Claude is already smart** — only add information Claude wouldn't know, or project-specific rules it must follow.

### Set the Right Degree of Freedom

Match instruction specificity to task risk:

- **High freedom** (heuristics and goals) for flexible tasks.
- **Medium freedom** (templates or pseudocode) when a preferred pattern exists.
- **Low freedom** (exact scripts/commands) for fragile or high-stakes operations.

### Test Across Models

Skills are model-sensitive. Verify your Skill works with all models you intend to use and adjust detail level as needed.

### Progressive Disclosure (Three-Level System)

**Level 1: YAML Frontmatter**
- Always loaded in system prompt
- Minimal activation signals only

**Level 2: SKILL.md Body**
- Loaded only when the Skill is relevant
- Provide primary instructions and navigation

**Level 3: Linked Files**
- Reference docs, examples, or scripts loaded only when needed
- Keep details out of SKILL.md unless they are required for most requests

### Design Philosophy (Supplement)

**Composability**: Skills work alongside others, not in isolation.

**Portability**: Works identically across Claude.ai, Claude Code, and API.

**Specificity**: Clear triggering conditions prevent over/under-activation.

## Required File Structure

```
your-skill-name/              # kebab-case ONLY
├── SKILL.md                  # Required - exact spelling, case-sensitive
├── scripts/                  # Optional - executable code
│   ├── validate.py
│   └── process.sh
├── references/               # Optional - documentation
│   ├── api-guide.md
│   └── examples/
└── assets/                   # Optional - templates, images
    └── template.md
```

## YAML Frontmatter: Critical Rules

### Minimal Required Format

```yaml
---
name: your-skill-name
description: What it does. Use when user asks to [specific phrases].
---
```

### Field Requirements

**name** (required):
- Max 64 characters
- Lowercase letters, numbers, hyphens only
- Use kebab-case: `notion-project-setup` ✅
- No spaces: `Notion Project Setup` ❌
- No underscores: `notion_project_setup` ❌
- No capitals: `NotionProjectSetup` ❌
- Must match folder name
- Cannot contain reserved words: `anthropic`, `claude`

**description** (required):
- MUST include BOTH what AND when
- Write in third person (not "I" or "you")
- Structure: `[What it does] + [When to use it] + [Key capabilities]`
- Under 1024 characters
- No XML tags (< or >)
- Include specific user phrases
- Mention relevant file types

✅ Good Examples:
```yaml
description: Analyzes Figma design files and generates developer handoff documentation. Use when user uploads .fig files, asks for "design specs", "component documentation", or "design-to-code handoff".

description: Manages Linear project workflows including sprint planning, task creation, and status tracking. Use when user mentions "sprint", "Linear tasks", "project planning", or asks to "create tickets".
```

❌ Bad Examples:
```yaml
description: Helps with projects.  # Too vague

description: Creates sophisticated multi-page documentation systems.  # Missing triggers

description: Implements the Project entity model with hierarchical relationships.  # Too technical
```

**license** (optional):
- Use for open-source skills: `MIT`, `Apache-2.0`

**compatibility** (optional):
- 1-500 characters
- Indicate environment requirements

**metadata** (optional):
- Custom key-value pairs
- Suggested: `author`, `version`, `mcp-server`

### Security Restrictions

**FORBIDDEN**:
- XML angle brackets (< >)
- Skills with "claude" or "anthropic" in name (reserved)

## Writing Effective Instructions

### Recommended Structure Template

Your SKILL.md should follow this structure:

**Frontmatter Section**:
```yaml
---
name: your-skill
description: [What it does + When to use it]
---
```

**Main Content**:
- Brief introduction paragraph
- Step-by-step instructions with examples
- Common usage examples
- Troubleshooting section

**Example Structure**:
- Step 1: Define the action
- Step 2: Show code/command examples
- Step 3: Describe expected results

See the "Real-World Examples" section below for complete, working examples.

### Best Practices for Instructions

✅ **Be Specific and Actionable**

Good:
```
Run `python scripts/validate.py --input {filename}` to check data format.
If validation fails, common issues include:
- Missing required fields (add them to the CSV)
- Invalid date formats (use YYYY-MM-DD)
```

Bad:
```
Validate the data before proceeding.
```

✅ **Include Error Handling**

```markdown
## Common Issues

### MCP Connection Failed

If you see "Connection refused":
1. Verify MCP server is running: Check Settings > Extensions
2. Confirm API key is valid
3. Try reconnecting: Settings > Extensions > [Service] > Reconnect
```

✅ **Reference Bundled Resources Clearly**

```
Before writing queries, consult `references/api-patterns.md` for:
- Rate limiting guidance
- Pagination patterns
- Error codes and handling
```

✅ **Use Progressive Disclosure**

Keep SKILL.md focused on core instructions. Move detailed documentation to `references/` and link to it.

## Progressive Disclosure Patterns

Treat SKILL.md as a table of contents and keep details in reference files.

### Reference Files (Explicit Index)

The following reference files are available and should be read when the task requires that level of detail:

- [references/progressive-disclosure.md](references/progressive-disclosure.md) — patterns, rules, and anti-patterns
- [references/real-world-examples.md](references/real-world-examples.md) — full example skills and workflows
- [references/workflows-and-feedback.md](references/workflows-and-feedback.md) — workflows, checklists, and validation loops
- [references/mcp-integration.md](references/mcp-integration.md) — MCP tool naming, dependency declaration, and error handling
- [references/troubleshooting.md](references/troubleshooting.md) — common failure modes and fixes

## Real-World Examples

See [references/real-world-examples.md](references/real-world-examples.md) for complete examples.

## Common Use Case Patterns (Concise)

1. **Sequential workflows**: explicit order, dependencies, and validation.
2. **Multi-MCP coordination**: phase separation and data handoffs.
3. **Iterative refinement**: quality criteria + validate → fix → repeat.

See [references/workflows-and-feedback.md](references/workflows-and-feedback.md) for full workflow patterns.

## Testing Your Skill

### Three-Phase Testing Approach

**1. Triggering Tests**

Goal: Ensure skill loads at the right times

✅ Should trigger:
- Obvious task requests
- Paraphrased requests
- Related file type mentions

❌ Should NOT trigger:
- Unrelated topics
- Generic requests

**2. Functional Tests**

Goal: Verify correct outputs

Check:
- Valid outputs generated
- API calls succeed
- Error handling works
- Edge cases covered

**3. Performance Comparison**

Goal: Prove improvement over baseline

Compare:
- Message count
- Token consumption
- API call failures
- User corrections needed

## Troubleshooting Guide

Details and common fixes are in [references/troubleshooting.md](references/troubleshooting.md).

## Skills + MCP Integration

If you use MCP tools, declare dependencies in metadata and provide deterministic workflows using fully qualified tool names. See [references/mcp-integration.md](references/mcp-integration.md) for workflow and error handling guidance.

## Length of Skill.md

Keep SKILL.md under **500 lines**. Split content into reference files as soon as you approach the limit.

## Distribution Checklist

Before publishing:

- ✅ Folder named in kebab-case
- ✅ SKILL.md exists with proper frontmatter
- ✅ Tested triggering on 10+ queries
- ✅ Functional tests pass
- ✅ README.md in repo root (not in skill folder)
- ✅ Installation instructions clear
- ✅ Screenshots/examples included
- ✅ License specified
- ✅ Version in metadata

## Quick Reference: Common Mistakes

| Issue | Wrong | Right |
|-------|-------|-------|
| File name | skill.md, SKILL.MD | SKILL.md |
| Folder name | My_Skill, MySkill | my-skill |
| Description | "Helps with tasks" | "Creates sprint plans. Use when..." |
| Frontmatter | No delimiters | Wrapped in `---` |
| Instructions | Vague guidance | Specific steps with examples |
| Structure | README.md in folder | Only SKILL.md |

## Resources

- **Skills Overview**: https://platform.claude.com/docs/en/agents-and-tools/agent-skills/overview
- **Best Practices**: https://platform.claude.com/docs/en/agents-and-tools/agent-skills/best-practices
- **MCP Docs**: https://modelcontextprotocol.io

---

**Remember**: Skills are living documents. Plan to iterate based on user feedback, triggering behavior, and execution results.
