---
name: featbit-getting-started
description: Step-by-step guide for getting started with FeatBit. Use when user asks about "getting started", "first feature flag", "FeatBit tutorial", "Dino Game demo", "create boolean flag", "multi-variant flag", or "how to use FeatBit". Do not use for advanced SDK integration, deployment configuration, REST API, or observability questions.
license: MIT
metadata:
  author: FeatBit
  version: 1.0.0
  category: platform
---

# FeatBit Getting Started Guide

Complete hands-on guide to getting started with FeatBit, covering environment initialization, creating boolean and multi-variant feature flags, interacting with the Dino Game demo, and integrating FeatBit SDKs into your applications.

## Execution Procedure

```
EP: featbit-getting-started

1. INITIALIZE environment
   - user logs in → set org name, project name → click "Get Started"
   - prerequisite: FeatBit Cloud account OR self-hosted instance

2. CREATE boolean feature flag ("game runner")
   - navigate Feature Flags → "+ Add" → name, boolean type, OFF/ON variations
   - set initial state OFF → save

3. INTERACT with Dino Game demo
   - open demo page → game hidden (flag OFF)
   - toggle "game runner" flag ON → game appears in real time
   - key takeaway: flag change → instant app behavior change, no deploy

4. CREATE multi-variant feature flag ("difficulty mode")
   - "+ Add" → name, string type → add variants: easy, normal, hard
   - configure serving rules (OFF→easy, ON→normal) → save
   - demo: click "Next Task" → observe difficulty change in real time

5. INTEGRATE SDK
   - select feature flag → choose SDK/language
   - portal generates starter code (install, init, evaluate, listen)
   - run code → portal verifies connection (Step 3)

6. VERIFY via common patterns
   - simple toggle (boolVariation)
   - config via multi-variant (variation + switch)
   - real-time updates (event listener)
```

## 1. Initialize the Environment

**First-Time Login Setup:**
- Log in to [FeatBit Cloud](https://app.featbit.co/) or your self-hosted instance
- Specify your organization name on initial login
- Define your project name
- Click "Get Started" button to proceed to the onboarding flow

**Prerequisites:**
- Access to FeatBit Cloud account, or
- Self-hosted FeatBit instance via [Docker Installation](https://docs.featbit.co/installation/full-installation)

## 2. Create a Boolean Feature Flag

The simplest feature flag type is boolean (true/false). The "game runner" feature flag demonstrates this:

**Steps:**
1. Navigate to Feature Flags page
2. Click "+ Add" button (top right)
3. Enter feature flag name: "game runner"
4. Keep default boolean type
5. Configure variations:
   - **OFF** → returns `false` (disables feature)
   - **ON** → returns `true` (enables feature)
6. Set initial state to OFF
7. Click "Save"

**Understanding Boolean Feature Flags:**
- Boolean feature flags control binary feature states (on/off, enabled/disabled)
- SDK evaluates the feature flag and returns a boolean value to the application
- Application code determines what happens based on the returned value
- Turning a feature flag ON typically enables the feature; OFF disables it

## 3. Interact with the Dino Game Demo

**Access the Demo:**
1. Click "Try interacting with the demo" button on Getting Started page
2. Dino Game loads in demo area

**Initial State:**
- Game is hidden by default
- Message displays: "Switch 'game-runner' feature flag to ON to release Dino Game"

**Make the Game Appear:**
1. Go to Feature Flags page
2. Find "game runner" feature flag
3. Click on the feature flag item or "Detail" button
4. Toggle the feature flag to ON
5. Return to demo page
6. Dino Game now appears and is playable

**Key Concept:**
This demonstrates real-time feature control - changing feature flag state in the portal immediately affects application behavior without code deployment or restart.

## 4. Create a Multi-Variant Feature Flag

Multi-variant feature flags allow more than two states, perfect for A/B/C testing, configuration values, or progressive rollouts.

**Example: Difficulty Mode Feature Flag**

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
   - When feature flag is OFF → serves `easy`
   - When feature flag is ON → serves `normal`
   - Default: OFF
7. Click "Save"

**Demo Interaction:**
1. In demo page, click "Next Task" button
2. Dino Game shows difficulty adjustment controls
3. Change targeting rules in the feature flag detail page
4. Observe game difficulty change in real-time

**Multi-Variant Use Cases:**
- Configuration values (API endpoints, feature limits)
- UI themes or styling options
- Algorithm variations for A/B testing
- Progressive feature rollouts (phases: alpha → beta → stable)

## 5. Integrate an SDK

FeatBit provides automatically generated starter code for quick SDK integration.

**Step 1 - Select a Feature Flag:**
- Choose one of your created feature flags
- Click "Next" to proceed

**Step 2 - Choose SDK and Copy Code:**
- Select your programming language/framework
- Portal generates complete starter code including:
  - SDK installation command
  - SDK initialization with server URL and environment key
  - User identification with custom properties
  - Feature flag evaluation
  - Feature flag update listener (for real-time changes)
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

## Common Integration Patterns

### Pattern 1: Simple Feature Toggle

**Use Case:** Show/hide a feature based on feature flag state

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

### Pattern 2: Configuration via Multi-Variant Feature Flag

**Use Case:** Adjust application behavior based on feature flag variation

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

### Pattern 3: Real-Time Feature Flag Updates

**Use Case:** React to feature flag changes without page reload

```javascript
// Listen for flag changes
client.on('update', (changes) => {
  if (changes.includes('difficulty-mode')) {
    const newDifficulty = client.variation('difficulty-mode', 'easy');
    updateGameDifficulty(newDifficulty);
  }
});
```

## Troubleshoot Common Issues

**Issue: Demo doesn't show feature flag changes**
- Verify feature flag is saved properly in portal
- Check browser console for WebSocket connection errors
- Refresh demo page if connection was interrupted

**Issue: SDK connection fails in Step 3**
- Verify environment key is correct (check for copy/paste errors)
- Ensure server URL is accessible from your network
- Check firewall settings if using self-hosted instance

**Issue: Feature flag evaluation returns default value**
- Confirm feature flag name matches exactly (case-sensitive)
- Verify SDK is initialized before evaluation
- Check that environment key corresponds to correct environment
