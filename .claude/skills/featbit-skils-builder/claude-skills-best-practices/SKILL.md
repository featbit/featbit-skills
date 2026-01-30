---
name: claude-skills-best-practices
description: Comprehensive best practices and guidelines for building high-quality Claude Skills plugins, covering structure, content, versioning, and marketplace readiness
appliesTo:
  - "**"
---

# Claude Skills Development Best Practices

Comprehensive guide for developing, structuring, and maintaining high-quality Claude Skills plugins suitable for marketplace submission.

## Overview

This skill provides expert guidance on creating professional Claude Skills plugins that follow industry best practices and Claude's marketplace requirements. It covers everything from project structure to content quality, performance optimization, and versioning.

## When This Skill Activates

- When working on Claude Skills plugin development
- When structuring a new skills project
- When preparing skills for marketplace submission
- When refactoring or improving existing skills
- When questions arise about skill best practices

---

## 1. Planning and Design Strategy

### Before You Start: Define Your Goals

**Key Questions to Answer:**

1. **What is the purpose of your skills plugin?**
   - Target audience (developers, DevOps, data scientists, etc.)
   - Primary use cases
   - Value proposition

2. **Who are your users?**
   - Experience level (beginner, intermediate, expert)
   - Common workflows and pain points
   - Expected outcomes

3. **What scope should each skill cover?**
   - Single responsibility principle
   - Depth vs. breadth balance
   - Dependencies between skills

### User Journey Mapping

**Understand User Flow:**

```
User Discovery → Skill Activation → Content Consumption → Action/Implementation
     ↓                ↓                     ↓                      ↓
  Browse catalog   Auto-trigger      Read guidance         Apply knowledge
     or            based on file     Find examples        Execute code
  Ask question     pattern           Get best practices   Troubleshoot
```

**Design for Each Stage:**

1. **Discovery Stage**:
   - Clear skill names and descriptions
   - Categorized organization
   - Searchable keywords
   - Example use cases

2. **Activation Stage**:
   - Precise file patterns
   - Context-aware triggering
   - Multiple activation conditions
   - No false positives

3. **Consumption Stage**:
   - Scannable content structure
   - Progressive disclosure (basic → advanced)
   - Code examples at key points
   - Clear section headers

4. **Action Stage**:
   - Copy-paste ready examples
   - Step-by-step instructions
   - Troubleshooting guidance
   - Links to further resources

### Content Strategy

**Information Architecture:**

```
Plugin Level
└── Categories (e.g., SDKs, Deployment, APIs)
    └── Individual Skills (specific technology/framework)
        └── Sections (setup, usage, best practices, etc.)
            └── Content Blocks (concepts, examples, tips)
```

**Content Hierarchy Guidelines:**

1. **High-level Overview First**:
   - What the skill covers
   - Why it's useful
   - When to use it

2. **Progressive Detail**:
   - Start with common use cases
   - Build to advanced scenarios
   - Reference additional resources

3. **Practical Over Theoretical**:
   - Focus on "how-to" over "what is"
   - Include real-world examples
   - Show actual implementation code

### Skill Granularity Decision Framework

**When to Create Separate Skills:**

✅ **Create Separate Skills When:**
- Different programming languages (Python SDK vs .NET SDK)
- Different frameworks (React vs Vue)
- Different deployment methods (Docker vs Kubernetes)
- Different user personas (developer vs operations)
- Content exceeds 500-700 lines
- Activation patterns are distinct

❌ **Keep in One Skill When:**
- Closely related concepts (basic + advanced features of same SDK)
- Same file patterns and activation context
- Content is under 500 lines
- Topics are inseparable (setup + configuration)

**Example Decision Tree:**

```
Should "API Integration" be one skill or multiple?
├─ Question: Multiple languages?
│  ├─ Yes → Separate skills per language
│  └─ No → Continue to next question
├─ Question: Different frameworks?
│  ├─ Yes → Separate skills per framework
│  └─ No → Continue to next question
├─ Question: Content > 500 lines?
│  ├─ Yes → Consider splitting by topic
│  └─ No → Keep as one skill
└─ Decision: Single skill or split
```

### Naming Convention Strategy

**Skill Naming Best Practices:**

1. **Use Descriptive, Specific Names**:
   - ✅ `project-python-server-sdk`
   - ✅ `project-kubernetes-deployment`
   - ❌ `project-sdk` (too vague)
   - ❌ `project-deploy` (unclear scope)

2. **Follow Consistent Patterns**:
   ```
   {project}-{technology}-{aspect}
   
   Examples:
   - featbit-python-sdk
   - featbit-docker-deployment
   - featbit-react-client-sdk
   ```

3. **Use Kebab-Case**:
   - All lowercase
   - Hyphens between words
   - No spaces or underscores

### Documentation Planning

**Essential Documentation to Plan:**

1. **User-Facing**:
   - README.md: Plugin overview and installation
   - QUICKSTART.md: Getting started guide
   - FAQ.md: Common questions

2. **Developer-Facing**:
   - CONTRIBUTING.md: How to contribute skills
   - DEVELOPMENT.md: Local setup and testing
   - STRUCTURE.md: Project organization

3. **Maintenance**:
   - CHANGELOG.md: Version history
   - ROADMAP.md: Future plans
   - ISSUES.md: Known issues and workarounds

### Performance Considerations

**Plan for Scale:**

1. **Skill Loading Performance**:
   - Use `.skillsignore` from the start
   - Keep skills under 1000 lines each
   - Minimize nested directories

2. **Activation Performance**:
   - Use specific file patterns
   - Avoid regex when simple globs work
   - Test activation in large workspaces

3. **Memory Footprint**:
   - Remove redundant content
   - Link to external docs for deep dives
   - Compress large code examples

### Testing Strategy

**Plan Your Testing Approach:**

1. **Activation Testing**:
   - Create test files matching patterns
   - Verify skills activate correctly
   - Check for false positives

2. **Content Testing**:
   - Run all code examples
   - Verify all links work
   - Check for typos and grammar

3. **Integration Testing**:
   - Test with real projects
   - Validate against official documentation
   - Get user feedback

### Release Planning

**Version Strategy:**

```
Phase 1: MVP (v0.1.0)
├─ Core 3-5 essential skills
├─ Basic documentation
└─ Testing with small user group

Phase 2: Beta (v0.5.0)
├─ 10-15 comprehensive skills
├─ Full documentation
└─ Community testing

Phase 3: Production (v1.0.0)
├─ All planned skills
├─ Complete documentation
├─ Marketplace ready
└─ Support infrastructure

Phase 4: Growth (v1.x.0)
├─ New features based on feedback
├─ Performance improvements
└─ Additional skills
```

### Maintenance Planning

**Long-term Sustainability:**

1. **Regular Updates**:
   - Monthly: Check for outdated links
   - Quarterly: Review and update content
   - Yearly: Major content refresh

2. **Community Engagement**:
   - Monitor issues and feedback
   - Respond to questions promptly
   - Accept and review contributions

3. **Documentation Sync**:
   - Track upstream documentation changes
   - Update skills when APIs change
   - Maintain compatibility notes

---

## 2. Skill File Structure

### Clean Root Directory

**Principle**: Keep the root directory focused on essential files only.

```
your-skills-plugin/
├── package.json          # Plugin metadata
├── README.md            # Main documentation
├── CHANGELOG.md         # Version history
├── LICENSE              # License file (MIT recommended)
├── .skillsignore        # Exclude non-skill files
├── .editorconfig        # Code style consistency
├── .gitignore           # Version control exclusions
├── docs/                # All documentation
└── skills/              # All skill definitions
    ├── skill-name-1/
    │   └── SKILL.md
    └── skill-name-2/
        └── SKILL.md
```

### Key Principles

1. **Separation of Concerns**: 
   - Root: Only essential plugin files
   - `docs/`: All documentation files
   - `skills/`: All skill definitions

2. **One Skill Per Folder**:
   - Each skill in its own directory
   - Single `SKILL.md` per skill
   - Clear, descriptive folder names (kebab-case)

3. **Excluded from Loading**:
   - Use `.skillsignore` to exclude documentation, tests, and dev files
   - Improves loading performance

---

## 3. Skill File Structure

### Standard SKILL.md Format

```markdown
---
name: skill-identifier-kebab-case
description: Concise one-line description (actionable, specific)
appliesTo:
  - "**/*.{ext1,ext2}"  # Precise file patterns
---

# Skill Title

One paragraph introduction explaining what this skill covers and why it's useful.

## Overview

Brief 2-3 sentence overview of the domain/topic.

## Core Knowledge Areas

### 1. Primary Topic Area

**Subsection Title:**
- Key point with details
- Implementation guidance
- Code examples (if applicable)

**Another Subsection:**
- More information
- Specific techniques

### 2. Secondary Topic Area

**Subsection:**
- Detailed content
- Examples

## Best Practices

1. **Practice Name**: Detailed explanation with reasoning
2. **Practice Name**: Another best practice with examples
3. **Practice Name**: More guidance

## Common Patterns

Common usage patterns and approaches with examples.

## Troubleshooting

### Issue 1
**Problem**: Description of the issue
**Solution**: How to resolve it

### Issue 2
**Problem**: Another common issue
**Solution**: Resolution steps

## Documentation Reference

Official documentation: [Link text](https://official-docs-url)

## Related Topics

- Related skill or topic 1
- Related skill or topic 2
```

### YAML Frontmatter Rules

**Required Fields:**
- `name`: Unique identifier (kebab-case)
- `description`: Brief, actionable description (1-2 sentences)
- `appliesTo`: File pattern(s) for auto-activation

**File Pattern Examples:**
```yaml
# Language-specific
appliesTo: "**/*.{cs,csproj,sln}"    # C#/.NET
appliesTo: "**/*.py"                  # Python
appliesTo: "**/*.{js,jsx}"            # JavaScript/React

# Framework-specific
appliesTo: "**/*.{jsx,tsx}"           # React
appliesTo: "**/*.vue"                 # Vue

# Universal
appliesTo: "**"                       # All files
```

---

## 4. Skill Design Principles

### Specificity Over Breadth

**✅ Good Examples:**
- `project-dotnet-sdk`: Only covers .NET SDK
- `project-python-sdk`: Only covers Python SDK
- `project-deployment-docker`: Specific to Docker deployment

**❌ Bad Examples:**
- `project-sdk-integration`: Too broad, covers multiple SDKs
- `project-deployment`: Vague, unclear scope
- `project-everything`: Impossible to maintain

### Precise File Pattern Matching

**Purpose**: Ensure skills activate only when relevant.

**Best Practices:**
- Use specific file extensions
- Avoid overly broad patterns unless necessary
- Test activation on various file types
- Consider framework-specific patterns

**Examples:**
```yaml
# Specific - Good
appliesTo: "**/*.{cs,csproj}"

# Too broad - Bad
appliesTo: "**"

# Multiple patterns - Good for universal skills
appliesTo:
  - "**/*.md"
  - "**/*.txt"
```

### Clear Activation Scenarios

**Include explicit "When to Use" sections:**
- List specific scenarios
- Provide activation triggers
- Mention file types
- Describe user intents

**Example:**
```markdown
## When This Skill Activates

- Working with C# files (*.cs)
- Editing .csproj or .sln files
- User asks about .NET SDK integration
- User mentions FeatBit and .NET together
```

---

## 5. Essential Files and Configuration

### package.json

**Purpose**: Plugin metadata and skill registry

**Required Fields:**
```json
{
  "name": "@your-org/claude-skills",
  "version": "1.0.0",
  "description": "Clear, concise description of your plugin",
  "author": {
    "name": "Your Name or Organization",
    "email": "contact@example.com",
    "url": "https://your-website.com"
  },
  "repository": {
    "type": "git",
    "url": "https://github.com/your-org/your-plugin"
  },
  "bugs": {
    "url": "https://github.com/your-org/your-plugin/issues"
  },
  "homepage": "https://github.com/your-org/your-plugin#readme",
  "keywords": [
    "claude",
    "skills",
    "ai",
    "your-domain"
  ],
  "license": "MIT",
  "skills": [
    {
      "name": "Skill Display Name",
      "path": "skills/skill-folder",
      "description": "Brief skill description"
    }
  ]
}
```

**Best Practices:**
- Use semantic versioning
- Include all contact information
- Provide repository and bug tracking URLs
- Use relevant keywords for discoverability
- List all skills in the array

### .skillsignore

**Purpose**: Exclude non-skill files from loading for better performance

**Recommended Content:**
```
# Documentation (not skills)
docs/
*.md
CONTRIBUTING.md
QUICKSTART.md

# Development files
.git/
.gitignore
node_modules/
package-lock.json
yarn.lock

# IDE files
.vscode/
.idea/
*.swp
*.swo
.DS_Store

# Build artifacts
dist/
build/
coverage/
*.log
*.tsbuildinfo

# Tests
tests/
__tests__/
*.test.*
*.spec.*

# CI/CD
.github/
.gitlab-ci.yml
```

**Benefits:**
- Faster skill loading
- Reduced memory usage
- Cleaner skill selection

### CHANGELOG.md

**Purpose**: Version history and release notes

**Format**: Follow [Keep a Changelog](https://keepachangelog.com/)

```markdown
# Changelog

All notable changes to this project will be documented in this file.

The format is based on [Keep a Changelog](https://keepachangelog.com/en/1.0.0/),
and this project adheres to [Semantic Versioning](https://semver.org/spec/v2.0.0.html).

## [Unreleased]

### Added
- Features being worked on

## [1.0.0] - 2026-01-30

### Added
- Initial release
- Skill 1: Description
- Skill 2: Description

### Changed
- Improvement description

### Fixed
- Bug fix description

### Removed
- Deprecated feature removal
```

**Section Guidelines:**
- **Added**: New features, skills
- **Changed**: Changes to existing functionality
- **Deprecated**: Soon-to-be removed features
- **Removed**: Removed features
- **Fixed**: Bug fixes
- **Security**: Security improvements

### README.md

**Purpose**: Main plugin documentation

**Recommended Structure:**
```markdown
# Plugin Name

[![License](https://img.shields.io/badge/license-MIT-blue.svg)](LICENSE)
[![Version](https://img.shields.io/badge/version-1.0.0-green.svg)](CHANGELOG.md)

Brief value proposition (1-2 sentences)

## What is This?

Clear explanation of what the plugin provides.

## Installation

### From Marketplace
Steps for marketplace installation

### Manual Installation
Steps for manual/development installation

## Skills

Organized listing of all skills by category:

### Category 1
- **Skill Name**: Description

### Category 2
- **Skill Name**: Description

## Usage Examples

Real-world code examples showing skills in action.

## Features

Key features and benefits.

## Documentation

Links to documentation resources.

## Contributing

Link to CONTRIBUTING.md or inline guidelines.

## License

License information.

## Support

Contact and support information.
```

### .editorconfig

**Purpose**: Maintain consistent code style across editors

```ini
root = true

[*]
charset = utf-8
end_of_line = lf
insert_final_newline = true
trim_trailing_whitespace = true
indent_style = space
indent_size = 2

[*.md]
trim_trailing_whitespace = false
indent_size = 2

[*.{json,yml,yaml}]
indent_size = 2
```

---

## 6. Content Quality Standards

### Accuracy

**Guidelines:**
- Base content on official documentation
- Test all code examples
- Verify links and references
- Use current API versions and syntax
- Update regularly

### Clarity

**Best Practices:**
- Use clear, concise language
- Structure content logically
- Use headings effectively
- Employ lists for scannability
- Use emojis sparingly (professional context)

### Completeness

**Each skill should include:**
- Framework integrations (if applicable)
- Complete, runnable code examples
- Best practices section
- Troubleshooting guidance
- Official resource links
- Related topics/skills

### Code Examples

**Requirements:**
- Complete and runnable
- Include necessary imports/setup
- Use proper syntax highlighting
- Add comments for clarity
- Show both basic and advanced usage

**Example:**
````markdown
## Example: .NET SDK Integration

```csharp
using FeatBit.Sdk.Server;
using FeatBit.Sdk.Server.Options;

// Configure the SDK
var options = new FeatBitOptionsBuilder()
    .Streaming(new Uri("ws://localhost:5100"))
    .EventUrl(new Uri("http://localhost:5100"))
    .Secret("your-env-secret")
    .Build();

// Initialize the client
var client = new FeatBitClient(options);

// Wait for initialization
await client.InitializeAsync();

// Create user context
var user = FeatBitUser.Builder("user-id")
    .Name("User Name")
    .Custom("customProperty", "value")
    .Build();

// Evaluate feature flag
var flagValue = client.BoolVariation("feature-key", user, false);
Console.WriteLine($"Feature enabled: {flagValue}");
```
````

---

## 7. Performance Optimization

### Use .skillsignore Effectively

**Benefits:**
- Faster plugin loading
- Reduced memory footprint
- Better performance in large workspaces

**What to Exclude:**
- Documentation directories
- Test files
- Build artifacts
- IDE configurations
- Version control files
- Development tools

### Individual Skills vs Monolithic

**✅ Recommended: Individual Skills**
- Better activation precision
- Clearer content organization
- Easier maintenance
- Faster loading for specific files

**❌ Avoid: Monolithic Skills**
- Difficult to maintain
- Imprecise activation
- Performance issues
- Poor user experience

### Optimize File Patterns

**Efficient Patterns:**
```yaml
# Good - Specific
appliesTo: "**/*.{py,pyi}"

# Better - Very specific
appliesTo: "**/src/**/*.py"

# Best - Multiple specific patterns
appliesTo:
  - "**/src/**/*.py"
  - "**/app/**/*.py"
```

**Avoid:**
```yaml
# Too broad
appliesTo: "**"

# Regex when simple glob works
appliesTo: ".*\\.py$"
```

---

## 8. Versioning and Release Management

### Semantic Versioning

**Format**: `MAJOR.MINOR.PATCH`

**When to Increment:**

- **MAJOR (x.0.0)**: Breaking changes
  - Removing skills
  - Incompatible structure changes
  - Major API changes

- **MINOR (0.x.0)**: New features (backward compatible)
  - Adding new skills
  - New features to existing skills
  - Non-breaking enhancements

- **PATCH (0.0.x)**: Bug fixes
  - Bug fixes
  - Documentation updates
  - Typo corrections
  - Minor improvements

**Examples:**
- `1.0.0` → `1.1.0`: Added new skill
- `1.1.0` → `1.1.1`: Fixed documentation typo
- `1.9.0` → `2.0.0`: Removed deprecated skills

### Release Process

**Checklist:**
1. ✅ Update skill content
2. ✅ Test all changes
3. ✅ Update CHANGELOG.md
4. ✅ Bump version in package.json
5. ✅ Update version badges in README
6. ✅ Commit changes
7. ✅ Create git tag: `git tag -a v1.0.0 -m "Release message"`
8. ✅ Push commits: `git push origin main`
9. ✅ Push tag: `git push origin v1.0.0`
10. ✅ Create GitHub Release
11. ✅ Publish to marketplace (if applicable)

---

## 9. Marketplace Readiness Checklist

### Required Files
- [ ] `package.json` with complete metadata
- [ ] Professional `README.md` with examples
- [ ] `CHANGELOG.md` following standards
- [ ] `LICENSE` file (MIT recommended)
- [ ] `.skillsignore` for performance
- [ ] `.gitignore` for version control
- [ ] `.editorconfig` for consistency

### Content Quality
- [ ] All skills have YAML frontmatter
- [ ] Clear, concise descriptions
- [ ] Complete code examples
- [ ] Tested and verified examples
- [ ] All links working
- [ ] No broken references
- [ ] Professional language
- [ ] Consistent formatting

### Organization
- [ ] Clean root directory
- [ ] Documentation in `docs/`
- [ ] Skills in `skills/`
- [ ] One SKILL.md per skill folder
- [ ] Logical skill categorization
- [ ] Clear naming conventions

### Documentation
- [ ] Installation instructions
- [ ] Usage examples
- [ ] Skills catalog
- [ ] Contributing guidelines
- [ ] Support information
- [ ] FAQ (if applicable)

### Legal and Attribution
- [ ] License file present
- [ ] Copyright information
- [ ] Attribution for third-party content
- [ ] No proprietary content violations

---

## 10. Common Pitfalls and Anti-Patterns

### ❌ Avoid These Mistakes

**1. Overly Broad Skills**
- Problem: One skill tries to cover too many topics
- Solution: Split into focused, specific skills

**2. Imprecise File Patterns**
- Problem: Skills activate on wrong files
- Solution: Use specific file extensions and patterns

**3. Incomplete Code Examples**
- Problem: Code snippets that don't run
- Solution: Provide complete, tested examples

**4. Outdated Information**
- Problem: Content not synced with official docs
- Solution: Regular updates and verification

**5. Lack of Testing**
- Problem: Skills don't activate as expected
- Solution: Test in real scenarios before release

**6. Cluttered Root Directory**
- Problem: Documentation mixed with skills
- Solution: Use separate `docs/` and `skills/` folders

**7. Missing .skillsignore**
- Problem: Loading unnecessary files
- Solution: Properly configure `.skillsignore`

**8. Vague Descriptions**
- Problem: Users don't know when to use skill
- Solution: Clear, actionable descriptions and activation scenarios

---

## 11. Testing and Validation

### Pre-Release Testing

**1. Skill Activation Testing**
```
- Open files matching appliesTo patterns
- Verify skill appears in context
- Test with non-matching files
- Ensure no false activations
```

**2. Code Example Verification**
```
- Copy examples to test project
- Run and verify they work
- Check for missing imports/setup
- Test edge cases
```

**3. Link Validation**
```
- Click all external links
- Verify official documentation links
- Check repository links
- Ensure no 404 errors
```

**4. Documentation Review**
```
- Read through all content
- Check for typos and grammar
- Verify formatting consistency
- Test installation instructions
```

### Continuous Quality

**Regular Maintenance:**
- Review and update content quarterly
- Monitor official documentation for changes
- Update code examples with new APIs
- Fix reported issues promptly
- Respond to user feedback

---

## 12. Success Metrics

### Quality Indicators

**Technical Quality:**
- ✅ All skills load without errors
- ✅ Skills activate on appropriate files
- ✅ Code examples run successfully
- ✅ Links are valid and working
- ✅ Documentation is clear and complete

**User Experience:**
- Easy to install
- Clear usage instructions
- Helpful and accurate responses
- Comprehensive coverage
- Regular updates

**Performance:**
- Fast loading times
- Efficient activation
- Minimal memory usage
- No conflicts with other skills

---

## 13. Resources and References

### Official Resources

- **Claude Skills Documentation**: Official Claude guidelines
- **Marketplace Requirements**: Submission standards
- **YAML Frontmatter Format**: Specification
- **Glob Pattern Syntax**: File matching documentation

### Industry Standards

- **Keep a Changelog**: https://keepachangelog.com/
- **Semantic Versioning**: https://semver.org/
- **EditorConfig**: https://editorconfig.org/
- **MIT License**: https://opensource.org/licenses/MIT

### Best Practice Guides

- Plugin structure recommendations
- Content organization patterns
- Performance optimization techniques
- Testing methodologies

---

## 14. Quick Reference

### Essential Commands

```bash
# Test skills locally
claude-code --load-skills ./skills

# Validate package.json
npm pkg lint

# Check for broken links
npx markdown-link-check README.md

# Create release tag
git tag -a v1.0.0 -m "Release v1.0.0"
git push origin v1.0.0
```

### Skill Creation Checklist

1. ✅ Create skill folder: `skills/skill-name/`
2. ✅ Add SKILL.md with YAML frontmatter
3. ✅ Write clear description
4. ✅ Define precise file patterns
5. ✅ Add comprehensive content
6. ✅ Include code examples
7. ✅ Add best practices section
8. ✅ Include troubleshooting
9. ✅ Link to official docs
10. ✅ Update package.json
11. ✅ Update README.md
12. ✅ Test activation
13. ✅ Commit changes

---

## Summary

Building high-quality Claude Skills requires:

1. **Clean structure**: Organized, focused file layout
2. **Specific skills**: Each skill does one thing well
3. **Precise patterns**: Accurate file matching
4. **Complete content**: Examples, practices, troubleshooting
5. **Performance**: Optimized loading with .skillsignore
6. **Versioning**: Semantic versioning and clear changelogs
7. **Quality**: Tested examples and working links
8. **Documentation**: Clear installation and usage guides

Follow these best practices to create professional, maintainable, and user-friendly Claude Skills plugins ready for marketplace publication!

---

**Last Updated**: January 30, 2026
**Version**: 1.0.0
