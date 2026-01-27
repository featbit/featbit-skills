# FeatBit Skills - Installation Guide

This guide walks you through installing the FeatBit Skills plugin for Claude Code.

## Quick Installation (Recommended)

Once you've added the FeatBit marketplace, install the skills plugin:

```bash
# In Claude Code, run:
/plugin marketplace add featbit/featbit-skills
/plugin install featbit-skills@featbit-marketplace
```

## Detailed Installation Steps

### Step 1: Add the FeatBit Marketplace

Open Claude Code and add the FeatBit marketplace using one of these methods:

**Method 1: Via GitHub (Recommended)**
```bash
/plugin marketplace add featbit/featbit-skills
```

**Method 2: Via Git URL**
```bash
/plugin marketplace add https://github.com/featbit/featbit-skills.git
```

**Method 3: Local Testing (For Development)**
```bash
/plugin marketplace add c:\Code\featbit\featbit-skills
# or on Mac/Linux
/plugin marketplace add /path/to/featbit-skills
```

### Step 2: Install the Plugin

After adding the marketplace, install the FeatBit skills:

```bash
/plugin install featbit-skills@featbit-marketplace
```

### Step 3: Verify Installation

Check that all 12 skills are loaded:

```bash
/plugin list
```

You should see `featbit-skills` in the list with status "Enabled".

### Step 4: Test a Skill

Try asking Claude a FeatBit question:

```
How do I integrate FeatBit in my ASP.NET Core application?
```

Claude should automatically activate the `featbit-dotnet-sdk` skill and provide detailed guidance.

## Available Skills

After installation, you'll have access to these 12 skills:

### Platform (2 skills)
- **featbit-documentation** - 75+ documentation topics
- **featbit-deployment** - Docker, Kubernetes, Terraform

### Server SDKs (5 skills)
- **featbit-dotnet-sdk** - .NET Server SDK
- **featbit-node-server-sdk** - Node.js Server SDK
- **featbit-python-sdk** - Python SDK
- **featbit-java-sdk** - Java SDK
- **featbit-go-sdk** - Go SDK

### Client SDKs (3 skills)
- **featbit-javascript-client-sdk** - Browser JavaScript
- **featbit-react-client-sdk** - React Hooks
- **featbit-react-native-sdk** - React Native Mobile

### OpenFeature (2 skills)
- **featbit-openfeature-node-server** - Node.js Provider
- **featbit-openfeature-js-client** - JavaScript Provider

## Updating

To get the latest version:

```bash
/plugin marketplace update featbit-marketplace
/plugin update featbit-skills@featbit-marketplace
```

Or update all marketplaces and plugins:

```bash
/plugin marketplace update
/plugin update
```

## Uninstalling

To remove the plugin:

```bash
/plugin uninstall featbit-skills@featbit-marketplace
```

To remove the marketplace:

```bash
/plugin marketplace remove featbit-marketplace
```

## Troubleshooting

### Plugin Not Found

If you get "plugin not found" error:

1. Verify the marketplace was added:
   ```bash
   /plugin marketplace list
   ```

2. Refresh the marketplace:
   ```bash
   /plugin marketplace update featbit-marketplace
   ```

3. Try installing again:
   ```bash
   /plugin install featbit-skills@featbit-marketplace
   ```

### Skills Not Activating

If skills don't activate when asking FeatBit questions:

1. Check plugin is enabled:
   ```bash
   /plugin list
   ```

2. Reload plugins:
   ```bash
   /plugin reload
   ```

3. Check file patterns - skills activate based on file types (.cs, .py, .js, etc.)

### Validation Errors

To validate your local copy:

```bash
/plugin validate c:\Code\featbit\featbit-skills
```

## For Enterprise/Team Deployment

### Auto-install for Team Members

Add to your project's `.claude/settings.json`:

```json
{
  "extraKnownMarketplaces": {
    "featbit-marketplace": {
      "source": {
        "source": "github",
        "repo": "featbit/featbit-skills"
      }
    }
  },
  "enabledPlugins": {
    "featbit-skills@featbit-marketplace": true
  }
}
```

Team members will be prompted to install when they trust the project folder.

### Restrict to Approved Marketplace Only

In managed settings, restrict to FeatBit marketplace only:

```json
{
  "strictKnownMarketplaces": [
    {
      "source": "github",
      "repo": "featbit/featbit-skills"
    }
  ]
}
```

## Support

- **GitHub Issues**: https://github.com/featbit/featbit-skills/issues
- **Email**: contact@featbit.co
- **FeatBit Docs**: https://docs.featbit.co
- **Claude Code Docs**: https://code.claude.com/docs

## Version History

See [CHANGELOG.md](CHANGELOG.md) for release notes.

---

**Current Version**: 1.1.0  
**Last Updated**: January 28, 2026
