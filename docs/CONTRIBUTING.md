# Contributing to FeatBit Claude Skills

Thank you for your interest in contributing to the FeatBit Claude Skills plugin! This document provides guidelines and instructions for contributing.

## 📋 Table of Contents

- [Code of Conduct](#code-of-conduct)
- [How Can I Contribute?](#how-can-i-contribute)
- [Development Setup](#development-setup)
- [Skill Development Guidelines](#skill-development-guidelines)
- [Pull Request Process](#pull-request-process)
- [Style Guide](#style-guide)

## 🤝 Code of Conduct

This project follows FeatBit's Code of Conduct. By participating, you are expected to uphold this code. Please report unacceptable behavior to support@featbit.co.

## 🎯 How Can I Contribute?

### Reporting Issues

Before creating an issue, please check if a similar issue already exists. When creating an issue:

- **Use a clear and descriptive title**
- **Describe the expected behavior**
- **Describe the actual behavior**
- **Provide steps to reproduce** (if applicable)
- **Include skill name** if the issue is skill-specific
- **Include Claude Code version** and OS information

### Suggesting Enhancements

Enhancement suggestions are tracked as GitHub issues. When creating an enhancement suggestion:

- **Use a clear and descriptive title**
- **Provide a detailed description** of the suggested enhancement
- **Explain why this enhancement would be useful**
- **List any alternative solutions** you've considered

### Contributing Code

We welcome contributions in several areas:

#### 1. Documentation Improvements
- Fix typos or clarify existing content
- Add missing information
- Improve code examples
- Update outdated information

#### 2. New Code Examples
- Add framework-specific examples
- Provide real-world use cases
- Include error handling patterns
- Add testing examples

#### 3. New Skills
- Support for additional programming languages
- Framework-specific skills
- Platform-specific skills (e.g., AWS, Azure, GCP)

#### 4. Bug Fixes
- Fix incorrect information in skills
- Correct activation patterns
- Fix formatting issues

## 🛠️ Development Setup

### Prerequisites

- Git
- Claude Code editor with skills support
- Text editor (VS Code recommended)
- Basic understanding of Markdown and YAML

### Setup Steps

1. **Fork the repository**
   ```bash
   # Click "Fork" on GitHub, then clone your fork
   git clone https://github.com/YOUR-USERNAME/featbit-skills.git
   cd featbit-skills/featbit-plugin
   ```

2. **Create a branch**
   ```bash
   git checkout -b feature/your-feature-name
   # or
   git checkout -b fix/your-bug-fix
   ```

3. **Link to Claude Code**
   ```bash
   # Linux/macOS
   ln -s "$(pwd)" ~/.claude-code/skills/featbit-dev
   
   # Windows (PowerShell as Administrator)
   New-Item -ItemType SymbolicLink -Path "$env:APPDATA\Claude Code\skills\featbit-dev" -Target (Get-Location)
   ```

4. **Reload skills in Claude Code**
   ```bash
   /skills reload
   ```

### Testing Your Changes

1. **Verify skill loads**
   ```bash
   # In Claude Code
   /skills list
   ```

2. **Test activation**
   - Create test files that match your skill's `appliesTo` pattern
   - Ask questions that should trigger your skill
   - Verify Claude uses the correct skill

3. **Test content**
   - Verify all code examples work
   - Check all links are valid
   - Ensure proper formatting

## 📝 Skill Development Guidelines

### Skill Structure

Each skill must follow this structure:

```
skills/
  skill-name/
    SKILL.md          # Main skill file with YAML frontmatter
```

### SKILL.md Format

```markdown
---
name: Skill Display Name
description: Brief description (1-2 sentences) of what the skill covers
appliesTo: "**/*.{ext1,ext2}"  # File pattern for activation
---

# Skill Name

## Overview

Brief overview of the skill's purpose and scope.

## When to Use This Skill

Describe when Claude should use this skill.

## Content Sections

### Section 1
Detailed content...

### Section 2
More content...

## Code Examples

Provide clear, working code examples.

## Common Issues

List common problems and solutions.

## Official Resources

Link to official documentation and resources.
```

### Content Guidelines

#### 1. Accuracy
- ✅ Verify all information against official FeatBit documentation
- ✅ Test all code examples before including them
- ✅ Keep content up-to-date with latest FeatBit versions
- ❌ Don't include speculative or unverified information

#### 2. Clarity
- ✅ Use clear, concise language
- ✅ Define technical terms when first used
- ✅ Organize content logically
- ✅ Use headers and lists for structure
- ❌ Avoid jargon without explanation

#### 3. Completeness
- ✅ Cover common use cases
- ✅ Include error handling
- ✅ Provide troubleshooting guidance
- ✅ Link to additional resources
- ❌ Don't assume prior knowledge without context

#### 4. Code Examples
- ✅ Include complete, runnable examples
- ✅ Add comments explaining key concepts
- ✅ Show both basic and advanced usage
- ✅ Include error handling
- ✅ Use consistent formatting
- ❌ Don't use pseudocode (unless clearly marked)

### File Patterns (appliesTo)

Choose appropriate file patterns for your skill:

```yaml
# Single extension
appliesTo: "**/*.cs"

# Multiple extensions
appliesTo: "**/*.{js,ts,jsx,tsx}"

# Specific files
appliesTo: "**/docker-compose.{yml,yaml}"

# All files (use sparingly)
appliesTo: "**"
```

### Skill Naming Conventions

- Use lowercase with hyphens: `featbit-python-sdk`
- Start with `featbit-` prefix
- Be specific: `featbit-react-client-sdk` not `featbit-react`
- Use clear descriptors: `-server-sdk` vs `-client-sdk`

## 🔄 Pull Request Process

### Before Submitting

1. **Test thoroughly**
   - Verify skill loads correctly
   - Test activation patterns
   - Validate all code examples
   - Check all links

2. **Update documentation**
   - Update CHANGELOG.md
   - Update README.md if adding new skills
   - Update package.json if adding new skills

3. **Follow style guide**
   - Use consistent formatting
   - Follow naming conventions
   - Keep line length reasonable (< 100 chars for prose)

### Submitting a Pull Request

1. **Commit your changes**
   ```bash
   git add .
   git commit -m "Add: Brief description of changes"
   ```

   Commit message prefixes:
   - `Add:` - New features or content
   - `Fix:` - Bug fixes
   - `Update:` - Updates to existing content
   - `Docs:` - Documentation-only changes
   - `Refactor:` - Code reorganization

2. **Push to your fork**
   ```bash
   git push origin feature/your-feature-name
   ```

3. **Create Pull Request**
   - Go to the original repository on GitHub
   - Click "New Pull Request"
   - Select your branch
   - Fill out the PR template

### PR Description Template

```markdown
## Description
Brief description of changes

## Type of Change
- [ ] New skill
- [ ] Bug fix
- [ ] Documentation update
- [ ] Enhancement to existing skill

## Testing Done
- [ ] Tested skill activation
- [ ] Verified code examples work
- [ ] Checked links are valid
- [ ] Updated relevant documentation

## Related Issues
Closes #issue-number (if applicable)

## Additional Notes
Any additional context or notes for reviewers
```

### Review Process

1. Maintainers will review your PR
2. Address any requested changes
3. Once approved, your PR will be merged
4. Your contribution will be included in the next release

## 📐 Style Guide

### Markdown Formatting

- Use ATX-style headers (`#` not underlines)
- Use fenced code blocks with language specification
- Use lists for items (unordered or ordered as appropriate)
- Use **bold** for emphasis, not _italic_
- Use `code` formatting for inline code, file names, and commands

### Code Examples

```javascript
// ✅ Good: Complete, clear, commented
import { FbClient } from 'featbit-node-server-sdk';

// Initialize client with configuration
const client = new FbClient({
  envSecret: 'your-env-secret',
  streamingUri: 'ws://localhost:5100'
});

// Wait for client to be ready
await client.waitForInitialization();

// Evaluate flag for user
const user = { keyId: 'user-123', name: 'John' };
const variation = client.boolVariation('feature-key', user, false);
```

```javascript
// ❌ Bad: Incomplete, unclear
const client = new FbClient({...});
const variation = client.boolVariation('key', user);
```

### Writing Style

- Use active voice: "Install the SDK" not "The SDK should be installed"
- Be direct: "Run `npm install`" not "You can run `npm install`"
- Use present tense: "The client connects" not "The client will connect"
- Avoid contractions in technical writing
- Use second person ("you") when addressing users

### Link Formatting

- Use descriptive link text: `[FeatBit documentation](url)` not `[click here](url)`
- Prefer relative links for internal resources: `[CHANGELOG](CHANGELOG.md)`
- Use full URLs for external resources

## 🎉 Recognition

Contributors will be recognized in:
- CHANGELOG.md for their contributions
- README.md contributors section (for significant contributions)

## 📧 Questions?

If you have questions about contributing:

- Open a GitHub issue with the `question` label
- Email: support@featbit.co
- Check existing documentation in the `docs/` folder

## 📄 License

By contributing, you agree that your contributions will be licensed under the MIT License.

---

Thank you for contributing to make FeatBit Claude Skills better! 🎉
