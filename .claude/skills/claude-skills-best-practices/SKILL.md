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

Expert guidance for creating production-ready Claude skills following Anthropic's official standards and battle-tested patterns.

## When to Use This Skill

Activate this skill when:
- Building a new skill from scratch
- Reviewing or improving existing skills
- Debugging skill triggering issues
- Integrating skills with MCP servers
- Planning skill architecture and structure
- Troubleshooting skill upload or execution problems

## Core Principles

### Progressive Disclosure (Three-Level System)

**Level 1: YAML Frontmatter**
- Always loaded in Claude's system prompt
- Provides just enough info for Claude to know when to activate
- Minimize token usage while maintaining discoverability

**Level 2: SKILL.md Body**
- Loaded when Claude determines skill is relevant
- Contains full instructions and guidance
- Keep focused and actionable

**Level 3: Linked Files**
- Additional files in `references/`, `scripts/`, `assets/`
- Claude navigates to these as needed
- Keeps main content lean

### Design Philosophy

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
- Use kebab-case: `notion-project-setup` ✅
- No spaces: `Notion Project Setup` ❌
- No underscores: `notion_project_setup` ❌
- No capitals: `NotionProjectSetup` ❌
- Must match folder name

**description** (required):
- MUST include BOTH what AND when
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

## Real-World Examples

### Example 1: Simple Skill Without MCP

**Pattern**: Pure document generation skill

**Key Elements**:

```yaml
---
name: tech-doc-generator
description: Generates technical documentation with consistent formatting. Use when user asks to "create technical docs", "write API documentation", "generate developer guide".
---
```

**Structure**:
1. **Gather Requirements**: Ask about doc type, audience, sections
2. **Generate Structure**: Overview → Prerequisites → Getting Started → Details → Troubleshooting
3. **Apply Formatting**: Headings, code blocks, tables, diagrams
4. **Quality Checklist**: Terms defined ✅ | Examples tested ✅ | Links verified ✅

**Why It Works**: Clear triggers in description, step-by-step workflow, built-in validation, no dependencies.

---

### Example 2: Single MCP Integration

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
1. **Validate Prerequisites**: Check MCP connection, credentials, naming conventions
2. **Execute Operation**: 
   - Create flag: `featbit_create_flag(name, description, type, environment)`
   - Update targeting: Fetch config → Apply rules → Validate → Update
   - Rollout strategy: 5% canary → Monitor 24h → 25% → Progressive increase
3. **Verification**: Check dashboard, test evaluation, document changes

**Best Practices Embedded**:
- Naming: `feature-area-description` format
- Never expose all users immediately (use targeting)
- Archive flags after full rollout

**Error Handling**:
- MCP connection failed → Check Settings > Extensions
- Flag exists → Use version suffix or update existing

**Why It Works**: Declares MCP dependency in metadata, validates before operations, includes domain best practices, handles common errors.

---

### Example 3: Multi-MCP Orchestration

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

**Phase 1 - Pre-Release** (GitHub MCP):
- Verify main branch healthy
- Generate release notes from commits
- Create release PR

**Phase 2 - Review** (Slack MCP):
- Notify team: `slack_post_message(channel="#releases", message="🚀 Release ready")`
- Monitor for approvals

**Phase 3 - Deploy** (GitHub MCP):
- Merge PR and create tag
- Track pipeline progress

**Phase 4 - Validate** (Datadog MCP):
- Health checks: Error rate < baseline, Response time < 500ms
- Post success metrics to Slack

**Phase 5 - Rollback** (If needed):
- Revert deployment
- Create incident report
- Notify stakeholders

**Safety Checks**: All tests pass ✅ | 2+ approvals ✅ | No conflicts ✅ | Release notes complete ✅

**Why It Works**: Coordinates 3 MCP servers seamlessly, clear phase separation, extensive safety checks, automated rollback, team communication built-in.

---

## Common Use Case Patterns

### Pattern 1: Sequential Workflow Orchestration

**Use when**: Multi-step processes in specific order

**Example workflow**:
1. Create Account - Call MCP tool `create_customer` with parameters: name, email, company
2. Setup Payment - Call MCP tool `setup_payment_method`, wait for verification
3. Create Subscription - Call MCP tool `create_subscription` with plan_id and customer_id from Step 1

**Key techniques**:
- Explicit step ordering
- Dependencies between steps
- Validation at each stage
- Rollback instructions for failures

### Pattern 2: Multi-MCP Coordination

**Use when**: Workflows span multiple services

**Key techniques**:
- Clear phase separation
- Data passing between MCPs
- Validation before moving to next phase
- Centralized error handling

### Pattern 3: Iterative Refinement

**Use when**: Output quality improves with iteration

**Key techniques**:
- Explicit quality criteria
- Iterative improvement loops
- Validation scripts
- Clear stopping conditions

### Pattern 4: Context-Aware Tool Selection

**Use when**: Same outcome, different tools based on context

**Key techniques**:
- Clear decision criteria
- Fallback options
- Transparency about choices

### Pattern 5: Domain-Specific Intelligence

**Use when**: Adding specialized knowledge beyond tool access

**Key techniques**:
- Domain expertise embedded in logic
- Compliance/validation before action
- Comprehensive documentation
- Clear governance

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

### Skill Won't Upload

**Error**: "Could not find SKILL.md in uploaded folder"
- **Cause**: File not named exactly SKILL.md
- **Solution**: Rename to SKILL.md (case-sensitive)

**Error**: "Invalid frontmatter"
- **Cause**: YAML formatting issue
- **Solution**: Verify `---` delimiters, closed quotes, proper indentation

**Error**: "Invalid skill name"
- **Cause**: Name has spaces or capitals
- **Solution**: Use kebab-case: `my-cool-skill`

### Skill Doesn't Trigger

**Symptom**: Skill never loads automatically

**Fix**: Revise your description field

Quick checklist:
- ❌ Too generic? ("Helps with projects")
- ❌ Missing trigger phrases users would say?
- ❌ Relevant file types not mentioned?

**Debug approach**: Ask Claude "When would you use the [skill name] skill?" Adjust based on response.

### Skill Triggers Too Often

**Symptom**: Skill loads for unrelated queries

**Solutions**:

1. Add negative triggers:
```yaml
description: Advanced data analysis for CSV files. Use for statistical modeling, regression, clustering. Do NOT use for simple data exploration (use data-viz skill instead).
```

2. Be more specific:
```yaml
# Too broad
description: Processes documents

# More specific  
description: Processes PDF legal documents for contract review
```

### MCP Connection Issues

**Symptom**: Skill loads but MCP calls fail

**Checklist**:
1. Verify MCP server is connected (Settings > Extensions)
2. Check authentication (API keys, permissions, OAuth tokens)
3. Test MCP independently (without skill)
4. Verify tool names match (case-sensitive)

### Instructions Not Followed

**Symptom**: Skill loads but Claude ignores instructions

**Common causes**:

1. **Too verbose**: Use bullet points, move details to references
2. **Instructions buried**: Put critical items at top with `## Important`
3. **Ambiguous language**: Be explicit with validations
4. **Model "laziness"**: Add encouragement in user prompts

**Advanced**: For critical validations, use scripts instead of language instructions.

## Skills + MCP Integration

If you have an MCP server, skills complete the story:

- **MCP**: Provides the tools (what Claude CAN do)
- **Skills**: Provides the workflows (how Claude SHOULD do it)

### The Value Proposition

**Without skills**:
- Users don't know what to do next
- Each conversation starts from scratch
- Inconsistent results
- Support tickets asking "how do I do X"

**With skills**:
- Pre-built workflows activate automatically
- Consistent, reliable tool usage
- Best practices embedded
- Lower learning curve

### Positioning Your Skill

✅ Good:
```
"The ProjectHub skill enables teams to set up complete project workspaces in seconds — including pages, databases, and templates — instead of spending 30 minutes on manual setup."
```

❌ Bad:
```
"The ProjectHub skill is a folder containing YAML frontmatter and Markdown instructions that calls our MCP server tools."
```

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

## Success Criteria

Your skill is ready when:

- ✅ Triggers on 90%+ of relevant queries
- ✅ Doesn't trigger on unrelated queries
- ✅ Completes workflows without user corrections
- ✅ Consistent results across sessions
- ✅ New users succeed on first try
- ✅ Reduces token usage vs. baseline
- ✅ Documentation is clear and actionable

## Resources

- **Official Docs**: https://docs.anthropic.com/en/docs/build-with-claude/skills
- **Example Skills**: https://github.com/anthropics/skills
- **MCP Docs**: https://modelcontextprotocol.io
- **Support**: Claude Developers Discord

## Pro Tips

💡 **Start Small**: Iterate on a single challenging task until Claude succeeds, then extract the pattern into a skill.

💡 **Use skill-creator**: Available in Claude.ai and Claude Code to generate and review skills.

💡 **Test Early**: Upload and test after writing frontmatter and first section. Iterate quickly.

💡 **Progressive Disclosure**: Don't load everything upfront. Link to references when appropriate.

💡 **Version Control**: Track changes with semantic versioning in metadata.

💡 **Learn from Examples**: Clone anthropics/skills repo and study patterns that work.

---

**Remember**: Skills are living documents. Plan to iterate based on user feedback, triggering behavior, and execution results.
