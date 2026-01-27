# Claude Code Skills Plugin Structure

This directory contains the FeatBit Claude Code skills plugin, ready for distribution via the Claude Code plugin marketplace.

## Directory Structure

```
featbit-plugin/
├── README.md                           # Main plugin documentation
├── package.json                        # Plugin metadata and configuration
└── skills/                             # All skills in this plugin
    ├── featbit-documentation/          # General FeatBit documentation skill
    │   └── SKILL.md                    # Documentation expert skill
    ├── featbit-dotnet-sdk/             # .NET SDK integration skill
    │   └── SKILL.md                    # .NET SDK expert skill
    ├── featbit-sdk-integration/        # Multi-language SDK skill
    │   └── SKILL.md                    # JavaScript, Python, Java, Go SDK skill
    └── featbit-deployment/             # Deployment configuration skill
        └── SKILL.md                    # Deployment expert skill
```

## Skills Overview

### 1. featbit-documentation
- **File**: `skills/featbit-documentation/SKILL.md`
- **Purpose**: Comprehensive FeatBit platform knowledge
- **Covers**: Feature flags, experimentation, API, IAM, integrations, architecture
- **Applies to**: All files (`**`)

### 2. featbit-dotnet-sdk
- **File**: `skills/featbit-dotnet-sdk/SKILL.md`
- **Purpose**: .NET Server SDK integration expertise
- **Covers**: ASP.NET Core, console apps, DI, configuration, best practices
- **Applies to**: `*.cs`, `*.csproj`, `Program.cs`, `Startup.cs`

### 3. featbit-sdk-integration
- **File**: `skills/featbit-sdk-integration/SKILL.md`
- **Purpose**: Multi-language SDK integration guidance
- **Covers**: JavaScript, React, React Native, Node.js, Python, Java, Go
- **Applies to**: `*.js`, `*.jsx`, `*.ts`, `*.tsx`, `*.py`, `*.java`, `*.go`

### 4. featbit-deployment
- **File**: `skills/featbit-deployment/SKILL.md`
- **Purpose**: Deployment and infrastructure expertise
- **Covers**: Docker Compose, Terraform, HA, security, monitoring
- **Applies to**: `docker-compose.yml`, `*.tf`

## SKILL.md Format

Each skill follows the Claude Code skills format:

```markdown
---
name: skill-name
description: Brief description
appliesTo:
  - "file-pattern"
---

# Skill Title

Content with expertise and guidance...
```

### YAML Frontmatter Fields

- **name**: Unique identifier for the skill
- **description**: Brief description of when to use this skill
- **appliesTo**: File patterns that trigger this skill (glob patterns)

## Publishing to Marketplace

To publish this plugin to the Claude Code plugin marketplace:

1. **Ensure repository structure**:
   - All skills are in `skills/` directory
   - Each skill has a `SKILL.md` file
   - `package.json` contains correct metadata

2. **Register marketplace** (if not already):
   ```bash
   /plugin marketplace add featbit/featbit-skills
   ```

3. **Users can install**:
   ```bash
   /plugin install featbit-skills@featbit
   ```

4. **Users can update**:
   ```bash
   /plugin update featbit-skills
   ```

## Local Development

To test skills locally:

1. Copy skills to Claude Code directory:
   ```bash
   # Windows
   xcopy /E /I skills %USERPROFILE%\.claude\skills\featbit
   
   # macOS/Linux
   cp -r skills ~/.claude/skills/featbit
   ```

2. Restart Claude Code or run reload command

3. Test by asking relevant questions

## Maintenance

### Adding New Skills

1. Create new directory under `skills/`
2. Create `SKILL.md` with proper frontmatter
3. Update `package.json` with new skill entry
4. Update main `README.md`
5. Test locally
6. Commit and push

### Updating Existing Skills

1. Edit the `SKILL.md` file
2. Test changes locally
3. Increment version in `package.json`
4. Update `CHANGELOG` in `README.md`
5. Commit and push

### Best Practices

- **Clear descriptions**: Make skill descriptions concise and actionable
- **Specific patterns**: Use precise `appliesTo` patterns to avoid conflicts
- **Comprehensive content**: Include examples, best practices, and troubleshooting
- **Official URLs**: Always reference official documentation URLs
- **Stay updated**: Sync with FeatBit documentation changes

## Source Content

The skills in this plugin are derived from:

1. **FeatBit MCP Server Documentation**:
   - Located in: `../FeatBit.McpServer/Resources/`
   - Contains: Docs tree JSON, SDK markdown files, deployment guides

2. **Official FeatBit Documentation**:
   - URL: https://docs.featbit.co
   - Structure: docs-tree.json with 75+ documentation files

3. **SDK Repositories**:
   - .NET: https://github.com/featbit/featbit-dotnet-sdk
   - JavaScript: https://github.com/featbit/featbit-js-client-sdk
   - Python: https://github.com/featbit/featbit-python-sdk
   - Java: https://github.com/featbit/featbit-java-sdk
   - Go: https://github.com/featbit/featbit-go-sdk

## License

MIT License - same as FeatBit platform

## Support

- GitHub Issues: https://github.com/featbit/featbit/issues
- Documentation: https://docs.featbit.co
- Website: https://featbit.co
