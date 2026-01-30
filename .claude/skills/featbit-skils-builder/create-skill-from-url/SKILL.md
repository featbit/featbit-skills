---
name: featbit-skills-builder-create-from-url
description: Expert guidance on converting documentation URLs into well-structured FeatBit skills with proper formatting and standards
appliesTo:
  - "**"
---

# Creating FeatBit Skills from Documentation URLs

Expert guidance for converting documentation URLs into high-quality, well-structured FeatBit skills that follow project standards and best practices.

## Overview

This skill activates when users provide one or more documentation URLs with hints about creating a FeatBit skill. It provides step-by-step guidance on:
- Fetching and analyzing URL content
- Structuring skill documentation
- Following FeatBit Skills project standards
- Ensuring quality and completeness

## When This Skill Activates

- User provides documentation URLs (one or multiple)
- User explicitly requests creating a skill from URL(s)
- User mentions converting documentation to skill format
- User asks for help organizing external documentation as a skill

## Best Practices for URL-to-Skill Conversion

### 1. Content Acquisition Phase

**Fetch Documentation:**
- Use the `fetch_webpage` tool to retrieve content from provided URLs
- For multiple URLs, fetch them in parallel for efficiency
- Include the user's hints/topic in the query parameter for focused extraction
- Verify content is relevant and complete before proceeding

**Example Workflow:**
```
User provides: 
- URLs: https://docs.example.com/api, https://docs.example.com/sdk
- Hints: "Create skill for API integration"

Action:
1. Fetch all URLs with relevant queries
2. Analyze content for key topics
3. Identify overlaps and gaps
```

### 2. Content Analysis Phase

**Review and Synthesize:**
- Identify main topics and subtopics across all URLs
- Look for:
  - Installation/setup instructions
  - Configuration options
  - API references
  - Code examples
  - Best practices
  - Common pitfalls
  - Integration patterns

**Organize Information:**
- Group related concepts together
- Identify dependencies (what must be learned first)
- Note any contradictions between sources
- Extract code examples that demonstrate key concepts

### 3. Skill Structure Phase

**Follow FeatBit Skills Standard Format:**

```markdown
---
name: skill-name (kebab-case)
description: One-line description (concise, actionable)
appliesTo:
  - "**"  # Or specific file patterns
---

# {Skill Title}

One paragraph introduction explaining what this skill covers and why it's useful.

## Overview

Brief 2-3 sentence overview of the domain/topic.

## Core Knowledge Areas

### 1. {Primary Topic Area}

**{Subsection Title}:**
- Key point with details
- Key point with details
- Code examples where relevant

**{Another Subsection}:**
- More information
- Examples

### 2. {Secondary Topic Area}

...repeat pattern...

## Best Practices

1. **Practice Name**: Detailed explanation of the best practice
2. **Practice Name**: Another best practice
3. ...

## Common Patterns

### Pattern 1: {Pattern Name}

Description and when to use it.

```language
// Code example
```

### Pattern 2: {Pattern Name}

...

## Anti-Patterns and Pitfalls

**Avoid {Anti-Pattern}:**
- Why it's problematic
- What to do instead

## Integration Guidelines

(If applicable - how this integrates with other systems/tools)

## Documentation Reference

Official documentation: {Primary URL}

Additional resources:
- {Secondary URL}
- {Tertiary URL}

## Related Topics

- Related skill or topic 1
- Related skill or topic 2
```

### 4. Writing Style Guidelines

**Be Clear and Concise:**
- Use active voice
- Avoid jargon unless necessary (define when used)
- Write in present tense
- Be specific and actionable

**Structure Information Hierarchically:**
- Start with high-level concepts
- Drill down into specifics
- Use numbered lists for sequences
- Use bullet points for non-sequential items

**Include Code Examples:**
- Show real, working code
- Use proper syntax highlighting
- Include comments explaining key parts
- Provide both simple and advanced examples

**Use Emojis Sparingly:**
- Only in section headers if they add clarity
- Don't overuse - maintain professionalism
- Examples: 📚 for documentation, ⚠️ for warnings, ✅ for best practices

### 5. Quality Checklist

Before finalizing a skill from URLs, verify:

**Content Quality:**
- [ ] All key concepts from URLs are covered
- [ ] Information is accurate and up-to-date
- [ ] No contradictions or ambiguities
- [ ] Proper attribution to source documentation

**Structure:**
- [ ] Follows FeatBit Skills standard format
- [ ] Frontmatter is complete and correct
- [ ] Sections are logically organized
- [ ] Headings follow proper hierarchy (H1 → H2 → H3)

**Completeness:**
- [ ] Installation/setup instructions included (if applicable)
- [ ] Configuration options documented
- [ ] Code examples provided
- [ ] Best practices section is comprehensive
- [ ] Links to official documentation included

**Usability:**
- [ ] Clear introduction explains the skill's purpose
- [ ] Examples are runnable and practical
- [ ] Technical terms are defined
- [ ] Navigation is intuitive

**Standards Compliance:**
- [ ] Name is in kebab-case
- [ ] Description is one line, concise
- [ ] Markdown formatting is correct
- [ ] Code blocks have language specified

### 6. Special Considerations

**Multiple URLs:**
- Synthesize information from all sources
- Highlight any differences or conflicts
- Create unified, coherent narrative
- Reference specific sources when needed

**Incomplete Documentation:**
- Note gaps in knowledge
- Mark sections as "requires verification"
- Suggest additional research needed
- Provide placeholders for missing information

**Version-Specific Information:**
- Indicate which version(s) are covered
- Note breaking changes between versions
- Prefer latest stable version unless specified

**Framework-Specific Examples:**
- Organize by framework/language if covering multiple
- Provide parallel examples when possible
- Use clear subsection headers

## Example: Step-by-Step Conversion

**Given:** 
- URLs: https://docs.featbit.co/sdk/javascript, https://docs.featbit.co/getting-started
- Hint: "Create skill for JavaScript SDK basics"

**Process:**

1. **Fetch Content:**
   ```
   fetch_webpage([url1, url2], query="JavaScript SDK setup and basic usage")
   ```

2. **Analyze:**
   - Identify: Installation, initialization, feature flags, user context
   - Extract: Code examples for each topic
   - Note: Configuration options and best practices

3. **Structure:**
   - Introduction to FeatBit JavaScript SDK
   - Core Knowledge: Installation, Setup, Basic Usage, Advanced Features
   - Best Practices: Error handling, performance, testing
   - Common Patterns: Feature flag checks, user targeting

4. **Write:**
   - Follow standard format
   - Include extracted code examples
   - Add context and explanations
   - Link back to source documentation

5. **Review:**
   - Check against quality checklist
   - Verify all URLs are referenced
   - Ensure examples are complete
   - Validate frontmatter

## Tips for AI Agents

**When User Provides URLs:**

1. **Immediately fetch content** - don't ask for permission
2. **Fetch in parallel** - if multiple URLs, fetch simultaneously
3. **Be thorough** - read enough content to understand the full scope
4. **Synthesize, don't copy** - create cohesive narrative, not a paste job
5. **Add value** - provide structure, context, and best practices beyond the source
6. **Reference sources** - always link back to original documentation
7. **Follow standards** - use the FeatBit Skills format exactly
8. **Include examples** - code examples make skills actionable
9. **Think about the user** - what would help them learn this effectively?
10. **Iterate if needed** - if content is insufficient, ask for more URLs or clarification

**Common Mistakes to Avoid:**

- ❌ Copying documentation verbatim
- ❌ Not fetching URLs before starting
- ❌ Ignoring project standards (see AGENTS.md)
- ❌ Creating overly short or superficial skills
- ❌ Missing code examples
- ❌ Not linking to source documentation
- ❌ Inconsistent formatting
- ❌ Unclear or missing frontmatter
- ❌ Skipping the quality checklist

## Documentation Reference

FeatBit Skills Project Standards: See AGENTS.md in repository root

Related documentation:
- [Contributing Guidelines](../../docs/CONTRIBUTING.md)
- [Skill Structure Guide](../../docs/STRUCTURE.md)
- [Best Practices](../../docs/BEST_PRACTICES.md)

## Related Topics

- Skill creation best practices
- Documentation synthesis
- Technical writing standards
- FeatBit project conventions
