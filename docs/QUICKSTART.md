# Quick Start Guide - FeatBit Claude Code Skills

This guide helps you quickly install and start using the FeatBit skills in Claude Code.

## Prerequisites

- Claude Code installed
- Basic understanding of FeatBit (optional - the skills will teach you!)

## Installation Options

### Option 1: Via Plugin Marketplace (Recommended)

**When Available**: Once this plugin is registered in the Claude Code Plugin marketplace.

```bash
# Add the FeatBit marketplace (if not already added)
/plugin marketplace add featbit/featbit-skills

# Install the plugin
/plugin install featbit-skills@featbit

# Verify installation
/plugin list
```

### Option 2: Manual Installation (For Development/Testing)

**Step 1**: Navigate to the plugin directory
```bash
cd c:\Code\featbit\featbit-skills\featbit-plugin
```

**Step 2**: Copy skills to your Claude Code directory

**On Windows**:
```powershell
# Create directory if it doesn't exist
New-Item -ItemType Directory -Force -Path "$env:USERPROFILE\.claude\skills\featbit"

# Copy all skills
Copy-Item -Path "skills\*" -Destination "$env:USERPROFILE\.claude\skills\featbit" -Recurse -Force
```

**On macOS/Linux**:
```bash
# Create directory if it doesn't exist
mkdir -p ~/.claude/skills/featbit

# Copy all skills
cp -r skills/* ~/.claude/skills/featbit/
```

**Step 3**: Restart Claude Code or reload skills

### Option 3: Direct Git Clone (Alternative)

```bash
# Clone to Claude skills directory on Windows
git clone https://github.com/featbit/featbit-skills "$env:USERPROFILE\.claude\plugins\featbit-skills"

# Or on macOS/Linux
git clone https://github.com/featbit/featbit-skills ~/.claude/plugins/featbit-skills
```

## Verify Installation

1. Open Claude Code
2. Ask a FeatBit-related question:
   ```
   What is FeatBit?
   ```
3. Claude should respond with detailed FeatBit information

## Quick Test

Try these test questions to verify each skill:

### Test 1: General Documentation
```
Question: "What's the difference between percentage rollout and targeting rules in FeatBit?"
Expected: Detailed explanation of both features
Skill Activated: featbit-documentation
```

### Test 2: .NET SDK
```
Question: "How do I integrate FeatBit in my ASP.NET Core application?"
Expected: Step-by-step integration guide with code
Skill Activated: featbit-dotnet-sdk
```

### Test 3: JavaScript/React
```
Question: "Show me how to use FeatBit in a React app with hooks"
Expected: React integration code with hooks
Skill Activated: featbit-sdk-integration
```

### Test 4: Deployment
```
Question: "How do I deploy FeatBit using Docker Compose?"
Expected: Docker Compose configuration and instructions
Skill Activated: featbit-deployment
```

## What You Can Ask

Here are example questions for each skill:

### General Documentation (featbit-documentation)
- "What is FeatBit?"
- "How do feature flags work?"
- "What's the difference between segments and targeting rules?"
- "How do I set up SSO with Okta?"
- "What deployment options does FeatBit offer?"
- "How does the relay proxy work?"
- "What's the pricing for FeatBit?"

### .NET SDK (featbit-dotnet-sdk)
- "How do I install the FeatBit .NET SDK?"
- "Show me dependency injection setup in ASP.NET Core"
- "How do I evaluate a boolean flag in C#?"
- "How do I track custom events for A/B testing?"
- "How do I configure FeatBit from appsettings.json?"
- "Show me a console app example"

### Multi-Language SDKs (featbit-sdk-integration)
- "How do I use FeatBit in JavaScript?"
- "Show me React hooks integration"
- "How do I integrate FeatBit in a Python Flask app?"
- "Show me Node.js Express integration"
- "How do I use FeatBit in a Spring Boot app?"
- "Show me Go SDK usage with Gin framework"

### Deployment (featbit-deployment)
- "How do I deploy FeatBit with Docker Compose?"
- "What's the minimal deployment option?"
- "How do I set up high availability?"
- "Show me the environment variables for the API server"
- "How do I troubleshoot port conflicts?"
- "How do I backup FeatBit data?"

## Working with Code Files

Skills automatically activate when you're working in relevant files:

### .NET Projects
```
Open: Program.cs, Startup.cs, *.cs files
Automatic Activation: featbit-dotnet-sdk
```

### JavaScript/TypeScript Projects
```
Open: *.js, *.jsx, *.ts, *.tsx files
Automatic Activation: featbit-sdk-integration
```

### Python Projects
```
Open: *.py files
Automatic Activation: featbit-sdk-integration
```

### Deployment Files
```
Open: docker-compose.yml, *.tf files
Automatic Activation: featbit-deployment
```

## Updating Skills

### Via Marketplace
```bash
# Update all plugins
/plugin update

# Or update specific plugin
/plugin update featbit-skills
```

### Manual Update
```bash
# Navigate to source
cd c:\Code\featbit\featbit-skills

# Pull latest changes
git pull

# Copy updated skills (Windows)
Copy-Item -Path "featbit-plugin\skills\*" -Destination "$env:USERPROFILE\.claude\skills\featbit" -Recurse -Force

# Copy updated skills (macOS/Linux)
cp -r featbit-plugin/skills/* ~/.claude/skills/featbit/
```

## Troubleshooting

### Skills Not Activating

1. **Check Installation Path**:
   ```bash
   # Windows
   dir "$env:USERPROFILE\.claude\skills\featbit"
   
   # macOS/Linux
   ls ~/.claude/skills/featbit
   ```

2. **Verify SKILL.md Files Exist**:
   - featbit-documentation/SKILL.md
   - featbit-dotnet-sdk/SKILL.md
   - featbit-sdk-integration/SKILL.md
   - featbit-deployment/SKILL.md

3. **Restart Claude Code**: Close and reopen the application

4. **Check Claude Code Version**: Ensure you're running a compatible version

### Skills Not Responding Correctly

1. **Be Specific**: Ask clear, specific questions
2. **Provide Context**: Mention the language or framework you're using
3. **Include File Context**: Have relevant files open in the editor

### Getting Help

- **FeatBit Documentation**: https://docs.featbit.co
- **GitHub Issues**: https://github.com/featbit/featbit/issues
- **Plugin Issues**: https://github.com/featbit/featbit-skills/issues

## Uninstalling

### Via Marketplace
```bash
/plugin uninstall featbit-skills
```

### Manual Removal
```bash
# Windows
Remove-Item -Path "$env:USERPROFILE\.claude\skills\featbit" -Recurse -Force

# macOS/Linux
rm -rf ~/.claude/skills/featbit
```

## Next Steps

1. ✅ Install the skills
2. ✅ Test with a few questions
3. 📖 Read the main README.md for detailed documentation
4. 💻 Start integrating FeatBit in your projects
5. 🚀 Deploy FeatBit using the deployment skill guidance

## Learning Path

**Beginner**: Start with general documentation questions
```
1. "What is FeatBit?"
2. "How do I create a feature flag?"
3. "What's a percentage rollout?"
```

**Intermediate**: Explore SDK integration
```
1. "How do I integrate FeatBit in [your language]?"
2. "Show me how to evaluate flags"
3. "How do I track custom events?"
```

**Advanced**: Deployment and architecture
```
1. "How do I deploy FeatBit to production?"
2. "What's the high availability setup?"
3. "How do I integrate with my observability tools?"
```

## Feedback

Found a bug or have a suggestion? 
- Open an issue: https://github.com/featbit/featbit-skills/issues
- Contribute: Fork the repo and submit a PR

---

Happy feature flagging with FeatBit! 🎉
