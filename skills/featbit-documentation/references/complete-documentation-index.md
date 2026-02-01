# Complete FeatBit Documentation Index

This reference contains all 75 documentation URLs from docs.featbit.co with summaries. Use this to find the most relevant pages for user queries.

Base URL: https://docs.featbit.co

---

## 1. Getting Started

**Overview**: Comprehensive onboarding guide covering initial setup, SDK integration, and core feature flag concepts through interactive tutorials.

- [Connect an SDK](https://docs.featbit.co/getting-started/connect-an-sdk) - Explains how to integrate FeatBit SDK into applications using auto-generated code with installation, initialization, user identification, flag evaluation, and update listeners.
- [Create Two Feature Flags](https://docs.featbit.co/getting-started/create-two-feature-flags) - Step-by-step guide for creating a boolean feature flag through the onboarding flow including organization/project initialization and SDK connection.
- [Create a Multi-Variant Feature Flag](https://docs.featbit.co/getting-started/create-a-multi-variant-feature-flag) - Tutorial for creating a multi-variant string feature flag with three variations demonstrating real-time flag changes.
- [Try Interacting with the Demo](https://docs.featbit.co/getting-started/try-interacting-with-the-demo) - Interactive tutorial showing how to toggle feature flags and observe application behavior changes.

### How-To Guides

- [Testing in Production](https://docs.featbit.co/getting-started/how-to-guides/testing-in-production) - Guide for safely testing in production using feature flags with targeted rollouts and individual user targeting.
- [Remote Config](https://docs.featbit.co/getting-started/how-to-guides/remote-config) - Describes remote configuration as a cloud-based key-value store for managing non-boolean settings like strings, numbers, or JSON.
- [Beta Testing & Qualitative Feedback](https://docs.featbit.co/getting-started/how-to-guides/beta-testing-qualitative-feedback) - Covers beta testing with real users in production environments to gather customer feedback before full release.
- [Entitlement Management](https://docs.featbit.co/getting-started/how-to-guides/entitlement) - Explains entitlement management for enabling/disabling software features using feature flags and segments for subscription-based access.
- [A/B Testing](https://docs.featbit.co/getting-started/how-to-guides/ab-testing) - Introduces A/B testing capabilities for front-end and server-side experiments with statistical significance analysis.

---

## 2. Feature Flags

**Overview**: Complete reference for feature flag management including flag creation, variations, user targeting, and organizational workflows.

- [Audit Log](https://docs.featbit.co/feature-flags/audit-log) - Explains the audit log feature for tracking all changes made to feature flags in a given environment with filtering capabilities.
- [Create Flag Variations](https://docs.featbit.co/feature-flags/create-flag-variations) - Details how to create and edit feature flag variations for boolean and multivariate flags including default values and A/B/n testing.
- [Feature Workflow](https://docs.featbit.co/feature-flags/feature-workflow) - Introduction to feature workflows for complex flag management including triggers and scheduled changes.
- [Targeting Users with Flags](https://docs.featbit.co/feature-flags/targeting-users-with-flags) - Overview of the Targeting tab for controlling which users receive specific flag variations.
- [The Flag Insights](https://docs.featbit.co/feature-flags/the-flag-insights) - Explains the flag evaluations graph for observing flag performance over time and end-user variation tables.
- [Users and User Segments](https://docs.featbit.co/feature-flags/users-and-user-segments) - Introduction to users and user segments as identifiable resources that encounter feature flags.

### Feature Workflow

- [Flag Triggers](https://docs.featbit.co/feature-flags/feature-workflow/flag-triggers) - Describes flag triggers that allow remote flag targeting changes via unique webhook URLs from third-party tools like APM systems.
- [Scheduled Flag Changes](https://docs.featbit.co/feature-flags/feature-workflow/scheduled-flag-changes) - Explains how to schedule feature flag changes for future dates enabling progressive delivery strategies.

### Targeting Users with Flags

- [Targeting Rules](https://docs.featbit.co/feature-flags/targeting-users-with-flags/targeting-rules) - Targeting rules serve different variations to user segments based on built-in and custom user attributes with conditions and operators.
- [Percentage Rollouts](https://docs.featbit.co/feature-flags/targeting-users-with-flags/percentage-rollouts) - Percentage rollouts enable gradual feature releases by serving new features to a specified percentage of users with consistent assignment.
- [Individual User Targeting](https://docs.featbit.co/feature-flags/targeting-users-with-flags/individual-targeting) - Allows precise control over which specific users receive particular flag variations.

### Users and User Segments

- [User Segments](https://docs.featbit.co/feature-flags/users-and-user-segments/user-segments) - Covers segments which are lists of users used to manage flag targeting behavior in bulk with include/exclude rules.
- [User Attributes](https://docs.featbit.co/feature-flags/users-and-user-segments/user-attributes) - Explains user attributes (built-in and custom) that define user objects used in flag targeting rules.
- [The Users List](https://docs.featbit.co/feature-flags/users-and-user-segments/the-users-list) - Describes the Users list which automatically populates when users encounter feature flags with filtering capabilities.
- [Shareable Segments](https://docs.featbit.co/feature-flags/users-and-user-segments/shareable-segments) - Introduces shareable segments that can be reused across different environments, projects, or organizations for consistent targeting.
- [Global Users](https://docs.featbit.co/feature-flags/users-and-user-segments/global-users) - Explains global users for workspace-wide user definitions.

### Organizing Flags

- [Projects](https://docs.featbit.co/feature-flags/organizing-flags/projects) - Describes projects as organizational units for managing different business objectives with unique environments and flags.
- [Environments](https://docs.featbit.co/feature-flags/organizing-flags/environments) - Explains environments as organizational units within projects for managing flags throughout the development lifecycle.
- [The Flag List](https://docs.featbit.co/feature-flags/organizing-flags/the-flag-list) - Details the feature flags list page showing all flags within an environment with search and filtering capabilities.
- [Copying Flags Between Environments](https://docs.featbit.co/feature-flags/organizing-flags/copying-flags-between-environments) - Enables copying feature flags and their settings from one environment to another with restrictions and validation.
- [Archiving and Deleting Flags](https://docs.featbit.co/feature-flags/organizing-flags/archiving-and-deleting-flags) - Guidance on archiving and deleting flags safely for maintaining clean flag inventories.

---

## 3. Experimentation

**Overview**: Documentation for FeatBit's experimentation and A/B testing capabilities, explaining how to measure feature flag impact through tracked metrics.

- [Understanding Experimentation](https://docs.featbit.co/experimentation/understanding-experimentation) - Comprehensive overview of FeatBit's Experimentation feature for measuring flag impact through A/B/n testing with credible intervals.
- [Creating Experiments](https://docs.featbit.co/experimentation/creating-experiments) - Guide for setting up experiments by creating metrics (click conversion, custom conversion, page view, numeric) and connecting them to flags.
- [Analyzing Experiments](https://docs.featbit.co/experimentation/analyzing-experiments) - Explains how to read experiment results using frequentist approach with confidence intervals and p-values for determining winners.

---

## 4. Relay Proxy

**Overview**: Complete documentation for FeatBit Agent - a fast and lightweight relay proxy running within your infrastructure supporting auto and manual modes.

- [Relay Proxy Overview](https://docs.featbit.co/relay-proxy/overview) - Describes the Relay Proxy management portal for FeatBit Agents including Auto agents (WebSocket real-time sync) and Manual agents (complete lifecycle control).
- [FeatBit Agent Repository](https://github.com/featbit/featbit-agent) - Official GitHub repository with complete source code, installation guides (Docker/systemd/native), API reference, health check setup, and troubleshooting guides.

---

## 5. API Documentation

**Overview**: Complete REST API reference for programmatic feature flag management covering authentication, request/response formats, and error handling.

- [API Overview](https://docs.featbit.co/api-docs/overview) - General introduction to FeatBit REST API covering authentication with personal/service tokens, JSON format, error handling, and CORS support.
- [Using FeatBit REST API](https://docs.featbit.co/api-docs/using-featbit-rest-api) - Comprehensive guide for getting started with FeatBit REST API including authentication, making requests, and understanding data hierarchy.
- [Flag Evaluation API](https://docs.featbit.co/api-docs/flag-evaluation-api) - Brief reference to Flag Evaluation APIs for evaluating feature flags for users.
- [API FAQ](https://docs.featbit.co/api-docs/faq) - REST API FAQs covering API endpoint access, documentation location, and detailed anatomy of feature flag JSON structure.
- [Retrieve Feature Flags with API](https://docs.featbit.co/sdk/retrieve-feature-flags-with-api) - Explains how to use the Evaluation Server APIs for client-side (single user, server-evaluated) and server-side (multiple users, locally evaluated) approaches.

---

## 6. Data Import and Export

**Overview**: Documentation for importing and exporting data including end users, feature flags, segments, and analytics data.

- [Data Export](https://docs.featbit.co/data-import-and-export/data-export) - Describes method for exporting feature flag usage data and experiment events from ClickHouse or MongoDB to third-party data platforms.
- [End Users](https://docs.featbit.co/data-import-and-export/end-users) - Explains how to download and upload end users for the current environment via the Data sync page in JSON format.
- [Feature Flags and Segments](https://docs.featbit.co/data-import-and-export/feature-flags-and-segments) - Brief reference pointing to Relay Proxy documentation for synchronizing flags and segments between FeatBit servers.

---

## 7. IAM (Identity and Access Management)

**Overview**: Complete Identity and Role Based Access Control documentation providing granular access control for FeatBit resources.

- [IAM Overview](https://docs.featbit.co/iam/overview) - Overview of FeatBit's IAM system inspired by AWS IAM providing precise access control for feature flags, projects, environments, and teams.
- [Teams](https://docs.featbit.co/iam/teams) - Explains team member management including adding members with initial permissions/groups, deleting members, and assigning permission policies.
- [Groups](https://docs.featbit.co/iam/groups) - Explains IAM groups as collections of members for easier permission management with details on group creation and policy attachment.
- [Policies](https://docs.featbit.co/iam/policies) - Details FeatBit's IAM policy system for access control including three control levels (All, General, Project, Environment) and custom policies.

---

## 8. Installation

**Overview**: Comprehensive deployment guide covering three deployment options: Standalone, Standard, and Professional.

- [Deployment Options](https://docs.featbit.co/installation/deployment-options) - Compares three FeatBit deployment options: Standalone (Postgres-only), Standard (Postgres/MongoDB + Redis), and Professional (adds ClickHouse + Kafka).
- [Docker Compose](https://docs.featbit.co/installation/docker-compose) - Quick start guide for installing FeatBit using Docker Compose with commands for each deployment option and default credentials.
- [Terraform AWS](https://docs.featbit.co/installation/terraform-aws) - Brief reference to the featbit-terraform-aws repository for deploying FeatBit to AWS using Terraform.
- [Use Your Own Infrastructure](https://docs.featbit.co/installation/use-your-own-infrastructure) - Guide for running FeatBit with custom infrastructure including environment variable configuration and database seeding.
- [Installation FAQ](https://docs.featbit.co/installation/faq) - Installation FAQs covering Docker port conflicts resolution and making FeatBit portal publicly accessible.

---

## 9. Integrations

**Overview**: Extensive integration documentation covering authentication, webhooks, SSO, observability, chat apps, and data analytics platforms.

- [API Access Tokens](https://docs.featbit.co/integrations/api-access-tokens) - Explains API access token authentication for FeatBit REST API including token scoping, permissions, and differences between personal and service tokens.
- [Webhooks](https://docs.featbit.co/integrations/webhooks) - Comprehensive guide for creating and using webhooks to subscribe to flag/segment changes with events, timeout/retry logic, and payload templates.

### Single Sign-On

- [SSO Overview](https://docs.featbit.co/integrations/single-sign-on) - Overview of SSO configuration for FeatBit 3.0.0+ via OpenID Connect requiring Enterprise subscription with required OIDC settings.
- [Okta SSO](https://docs.featbit.co/integrations/single-sign-on/okta) - Step-by-step guide for configuring Okta as an Identity Provider for FeatBit SSO with application creation and redirect URI configuration.
- [KeyCloak SSO](https://docs.featbit.co/integrations/single-sign-on/keycloak) - Complete tutorial for setting up KeyCloak as an Identity Provider including running Keycloak in Docker and creating a realm.
- [Auth0 SSO](https://docs.featbit.co/integrations/single-sign-on/auth0) - Guide for integrating Auth0 with FeatBit SSO.

### Observability

- [OpenTelemetry](https://docs.featbit.co/integrations/observability/opentelemetry) - Explains OpenTelemetry instrumentation in FeatBit's backend services for publishing observability metrics, traces, and logs with ready-to-run examples.
- [New Relic](https://docs.featbit.co/integrations/observability/new-relic) - Details two New Relic integration methods: sending feature flag change events as deployment events and using triggers for performance metric thresholds.
- [Grafana](https://docs.featbit.co/integrations/observability/grafana) - Guide for integrating FeatBit with Grafana to display flag/segment changes as annotations on graphs with service account creation.
- [DataDog](https://docs.featbit.co/integrations/observability/datadog) - DataDog integration with RUM integration, events tracking, triggers, and APM insights export.

### Chat Apps

- [Slack](https://docs.featbit.co/integrations/chat-apps/slack) - Integration guide for sending feature flag and segment change notifications to Slack using webhooks and Slack API.

### Data Analytics

- [GrowthBook](https://docs.featbit.co/integrations/data-analytic/growthbook) - Integration documentation for connecting FeatBit's ClickHouse database with GrowthBook for advanced A/B testing analysis.

---

## 10. Licenses

**Overview**: Documentation explaining FeatBit's Open Core licensing model with MIT license for core functionality and enterprise features.

- [Licenses Overview](https://docs.featbit.co/licenses/overview) - Explains FeatBit's Open Core licensing model with MIT license and lists enterprise features requiring license keys.
- [Setup License Key](https://docs.featbit.co/licenses/setup-license-key) - Complete guide for obtaining and configuring a FeatBit license key (version 3.0.0+) to enable enterprise features.
- [Pricing](https://featbit.co/pricing) - Official FeatBit pricing page detailing self-hosting plans: Open Source & Free (forever free), Enterprise Standard License ($3,999/year), and Enterprise Premium Features (custom pricing).

---

## 11. Tech Stack

**Overview**: In-depth technical documentation of FeatBit's architecture, infrastructure, and performance characteristics.

- [Tech Stack Overview](https://docs.featbit.co/tech-stack/overview) - Explains FeatBit's architecture focusing on scalability, performance, and privacy with push-based real-time updates using WebSocket.
- [Application Services](https://docs.featbit.co/tech-stack/application-services) - Describes FeatBit's four core services: UI Service (Angular), API Service (.NET), Evaluation Server, and Data Analytics Service (Python).
- [Infrastructure Components](https://docs.featbit.co/tech-stack/infrastructure-components) - Describes the three critical infrastructure components: Database (MongoDB/Postgres/ClickHouse), Message Queue (Postgres/Redis/Kafka), and Caching (Redis/None).
- [Data Flows](https://docs.featbit.co/tech-stack/data-flows) - Details FeatBit's internal data flows including data synchronization between SDKs and the Evaluation Server with real-time propagation.
- [Benchmark](https://docs.featbit.co/tech-stack/benchmark) - Presents performance benchmark results: 1100 new connections per second with P99 response time under 200ms on AWS EC2 t2.micro.

---

## 12. GitHub Repositories

- [Main Repository](https://github.com/featbit/featbit) - Main FeatBit platform source code and documentation.
- [Relay Proxy](https://github.com/featbit/featbit-agent) - FeatBit Agent (Relay Proxy) source code with installation and configuration guides.
- [Terraform AWS](https://github.com/featbit/featbit-terraform-aws) - Terraform scripts for deploying FeatBit to AWS infrastructure.
