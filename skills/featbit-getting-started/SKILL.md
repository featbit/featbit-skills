---
name: featbit-getting-started
description: Step-by-step guide for getting started with FeatBit including creating feature flags, interactive demos, multi-variant flags, and SDK integration
appliesTo:
  - "**"
---

# FeatBit Getting Started Guide

Complete hands-on guide to getting started with FeatBit, covering environment initialization, creating boolean and multi-variant feature flags, interacting with the Dino Game demo, and integrating FeatBit SDKs into your applications.

## Overview

This skill provides practical, step-by-step guidance for new FeatBit users to quickly understand core concepts through the interactive "Getting Started" experience in the FeatBit portal. You'll learn to create feature flags, see them in action with the Dino Game demo, and integrate FeatBit into your applications using SDKs.

## Core Knowledge Areas

### 1. Environment Initialization

**First-Time Login Setup:**
- Log in to [FeatBit Cloud](https://app.featbit.co/) or your self-hosted instance
- Specify your organization name on initial login
- Define your project name
- Click "Get Started" button to proceed to the onboarding flow

**Prerequisites:**
- Access to FeatBit Cloud account, or
- Self-hosted FeatBit instance via [Docker Installation](https://docs.featbit.co/installation/full-installation)

### 2. Creating Boolean Feature Flags

**Basic Feature Flag Creation:**

The simplest feature flag type is boolean (true/false). The "game runner" flag demonstrates this:

**Steps:**
1. Navigate to Feature Flags page
2. Click "+ Add" button (top right)
3. Enter flag name: "game runner"
4. Keep default boolean type
5. Configure variations:
   - **OFF** → returns `false` (disables feature)
   - **ON** → returns `true` (enables feature)
6. Set initial state to OFF
7. Click "Save"

**Understanding Boolean Flags:**
- Boolean flags control binary feature states (on/off, enabled/disabled)
- SDK evaluates flag and returns boolean value to application
- Application code determines what happens based on the returned value
- Turning flag ON typically enables the feature; OFF disables it

### 3. Interactive Demo - Dino Game

**Accessing the Demo:**
1. Click "Try interacting with the demo" button on Getting Started page
2. Dino Game loads in demo area

**Initial State:**
- Game is hidden by default
- Message displays: "Switch 'game-runner' feature flag to ON to release Dino Game"

**Making the Game Appear:**
1. Go to Feature Flags page
2. Find "game runner" flag
3. Click on flag item or "Detail" button
4. Toggle flag to ON
5. Return to demo page
6. Dino Game now appears and is playable

**Key Concept:**
This demonstrates real-time feature control - changing flag state in the portal immediately affects the application behavior without code deployment or restart.

### 4. Creating Multi-Variant Feature Flags

**Beyond Boolean - Multiple Variations:**

Multi-variant flags allow more than two states, perfect for A/B/C testing, configuration values, or progressive rollouts.

**Example: Difficulty Mode Flag**

**Steps:**
1. Navigate to Feature Flags page
2. Click "+ Add" button
3. Name: "difficulty mode"
4. Choose **string** as variation type
5. Add three variants:
   - `easy`
   - `normal`
   - `hard`
6. Configure serving rules:
   - When flag is OFF → serves `easy`
   - When flag is ON → serves `normal`
   - Default: OFF
7. Click "Save"

**Demo Interaction:**
1. In demo page, click "Next Task" button
2. Dino Game shows difficulty adjustment controls
3. Change targeting rules in flag detail page
4. Observe game difficulty change in real-time

**Multi-Variant Use Cases:**
- Configuration values (API endpoints, feature limits)
- UI themes or styling options
- Algorithm variations for A/B testing
- Progressive feature rollouts (phases: alpha → beta → stable)

### 5. SDK Integration

**Portal-Guided Integration:**

FeatBit provides automatically generated starter code for quick SDK integration.

**Integration Steps:**

**Step 1 - Select Feature Flag:**
- Choose one of your created feature flags
- Click "Next" to proceed

**Step 2 - Choose SDK and Copy Code:**
- Select your programming language/framework
- Portal generates complete starter code including:
  - SDK installation command
  - SDK initialization with server URL and environment key
  - User identification with custom properties
  - Feature flag evaluation
  - Flag update listener (for real-time changes)
  - Basic usage examples

**Step 3 - Verify Connection:**
- Run the starter code in your IDE
- Portal displays testing results
- Confirms SDK successfully connected to FeatBit

**Supported SDKs:**
- JavaScript (Client & Server)
- React & React Native
- .NET (Server & Client)
- Python
- Java
- Go
- Node.js
- OpenFeature integrations

See [SDK Overview](https://docs.featbit.co/sdk/overview#supported-sdks) for complete list.

## Best Practices

1. **Start with Boolean Flags**: When learning FeatBit, begin with simple boolean flags before moving to multi-variant flags. This helps you understand the core concepts without complexity.

2. **Use the Interactive Demo**: The Dino Game demo is designed to provide immediate visual feedback. Use it to experiment with flag changes and understand the impact in real-time before implementing in your application.

3. **Follow the Portal Guidance**: The Getting Started wizard in the portal provides automatically generated, environment-specific code. This eliminates configuration errors and ensures correct setup.

4. **Test Flag Changes**: Before implementing flags in production applications, verify behavior in the demo environment or development environment to understand how flag evaluation works.

5. **Understand Flag States**: Remember that boolean flags have two states (ON/OFF) but multi-variant flags can serve different values in each state. Plan your flag variations based on your use case requirements.

6. **SDK Connection Verification**: Always verify SDK connection in the portal's Step 3 before proceeding to production integration. This confirms proper configuration and network connectivity.

7. **User Context Matters**: When evaluating flags, provide meaningful user context (user ID, attributes). This enables advanced targeting and segmentation later.

8. **Real-Time Updates**: FeatBit SDKs support real-time flag updates via WebSocket. Leverage this for instant feature control without application restarts.

## Common Integration Patterns

### Pattern 1: Simple Feature Toggle

**Use Case:** Show/hide a feature based on flag state

```javascript
// After SDK initialization
const isGameEnabled = await client.boolVariation('game-runner', false);

if (isGameEnabled) {
  // Render game component
  renderDinoGame();
} else {
  // Show placeholder or nothing
  showPlaceholder('Feature not available');
}
```

### Pattern 2: Configuration via Multi-Variant Flag

**Use Case:** Adjust application behavior based on flag variation

```javascript
// Get difficulty mode
const difficulty = await client.variation('difficulty-mode', 'easy');

// Configure game based on difficulty
switch(difficulty) {
  case 'easy':
    setGameSpeed(1.0);
    break;
  case 'normal':
    setGameSpeed(1.5);
    break;
  case 'hard':
    setGameSpeed(2.0);
    break;
}
```

### Pattern 3: Real-Time Flag Updates

**Use Case:** React to flag changes without page reload

```javascript
// Listen for flag changes
client.on('update', (changes) => {
  if (changes.includes('difficulty-mode')) {
    const newDifficulty = client.variation('difficulty-mode', 'easy');
    updateGameDifficulty(newDifficulty);
  }
});
```

## Next Steps After Getting Started

After completing the Getting Started guide, explore these topics:

**Immediate Next Steps:**
1. **Testing in Production** - Learn how to safely test features in production environments
2. **Targeted Progressive Delivery** - Roll out features to specific user segments
3. **User Targeting** - Create targeting rules based on user attributes
4. **Percentage Rollouts** - Gradually release features to percentage of users

**Advanced Topics:**
5. **Experimentation & A/B Testing** - Measure feature impact with metrics
6. **Entitlement Management** - Control feature access based on subscription tiers
7. **Remote Configuration** - Use feature flags for dynamic configuration
8. **Feature Workflow** - Implement scheduled flag changes and triggers

## Troubleshooting Common Issues

**Issue: Demo doesn't show flag changes**
- Verify flag is saved properly in portal
- Check browser console for WebSocket connection errors
- Refresh demo page if connection was interrupted

**Issue: SDK connection fails in Step 3**
- Verify environment key is correct (check for copy/paste errors)
- Ensure server URL is accessible from your network
- Check firewall settings if using self-hosted instance

**Issue: Flag evaluation returns default value**
- Confirm flag name matches exactly (case-sensitive)
- Verify SDK is initialized before evaluation
- Check that environment key corresponds to correct environment

## Documentation Reference

Official Getting Started Documentation:
- [Create a Feature Flag](https://docs.featbit.co/getting-started/create-two-feature-flags)
- [Try Interacting with the Demo](https://docs.featbit.co/getting-started/try-interacting-with-the-demo)
- [Create a Multi-Variant Feature Flag](https://docs.featbit.co/getting-started/create-a-multi-variant-feature-flag)
- [Connect an SDK](https://docs.featbit.co/getting-started/connect-an-sdk)

Additional Resources:
- [SDK Overview](https://docs.featbit.co/sdk/overview)
- [Feature Flags Management](https://docs.featbit.co/feature-flags/the-flag-list)
- [User Targeting Documentation](https://docs.featbit.co/feature-flags/targeting-users-with-flags)

## Related Topics

- FeatBit SDK Integration (language-specific skills)
- Feature Flag Management and Organization
- User Targeting and Segmentation
- A/B Testing and Experimentation
- FeatBit Deployment Options
