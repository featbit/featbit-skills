# Changelog

All notable changes to the FeatBit Agent Skills collection will be documented in this file.

The format is based on [Keep a Changelog](https://keepachangelog.com/en/1.0.0/),
and this project adheres to [Semantic Versioning](https://semver.org/spec/v2.0.0.html).

## [2.1.2] - 2026-03-17

### Fixed
- Synchronized `package.json` with the actual `skills/` directory by adding `featbit-getting-started`
  and `featbit-rest-api`, and removing stale OpenFeature entries that no longer exist in the repo
- Removed the tracked `skills-lock.json` development artifact and updated `.gitignore` to ignore
  local `.agents/` tooling state and future lockfiles
- Corrected stale skill cross-references in `featbit-deployment-docker` and `featbit-rest-api`
- Cleaned up legacy repository metadata and outdated FeatBit collection wording in root docs

### Changed
- Reworked `README.md` so the skill catalog appears before installation, added direct links to each
  `SKILL.md`, and clarified agent compatibility
- Removed the outdated `Future Releases` planning section from this changelog

## [2.1.1] - 2026-03-16

### Fixed
- **`featbit-evaluation-insights-api` skill** — added missing constraint that at least one of
  `variations` or `metrics` must be non-empty in each insight object (prevents silent no-op payloads)
- **`featbit-evaluation-insights-api` description** — added `"Unity SDK"` and `"embedded SDK"` trigger
  phrases to match platforms mentioned in the skill body
- **README catalog row** — updated platform list to include Unity and embedded for consistency

## [2.1.0] - 2026-03-16

### Added
- **`featbit-evaluation-insights-api` skill** — expert guidance for calling FeatBit's Flag Evaluation
  REST API and Track Insights REST API directly over HTTP, primarily for building custom SDKs on
  platforms without an official FeatBit client (Kotlin, Swift, Android, iOS, Unity, etc.).
  Covers the full evaluate → cache → track insights → poll workflow, with reference files for the
  complete request/response schemas of both endpoints.

## [2.0.0] - 2026-03-09

### Breaking Changes
- **Renamed all SDK skill directories** from `featbit-{language}-sdk` to `featbit-sdks-{language}` for
  consistent file-browser grouping. Systems referencing skills by the old names must update:
  - `featbit-dotnet-sdk` → `featbit-sdks-dotnet`
  - `featbit-java-sdk` → `featbit-sdks-java`
  - `featbit-node-server-sdk` → `featbit-sdks-node`
  - `featbit-javascript-client-sdk` → `featbit-sdks-javascript`
  - `featbit-go-sdk` → `featbit-sdks-go`
  - `featbit-react-client-sdk` → `featbit-sdks-react`
  - `featbit-react-native-sdk` → `featbit-sdks-react-native`
  - `featbit-python-sdk` → `featbit-sdks-python`

### Added
- **`featbit-sdks` router skill** — language-detection routing table that directs SDK questions to the
  correct language-specific skill without requiring users to know the exact skill name

### Improved
- **Slim all SDK SKILL.md files** to ≤107 lines using the GitHub-pointer model:
  - Each skill now contains only FeatBit-specific vocabulary (exact class/method names), one minimal
    Quick Start from the official README, and anchor links for advanced topics
  - Content that AI can generate from general language knowledge has been removed
  - All advanced topic links point directly to the official GitHub README anchors

---

## [1.5.0] - 2026-03-03

### Added
- **New AWS Deployment Skill** (`featbit-deployment-aws`)
  - Deployment guidance for AWS ECS Fargate and EKS (Kubernetes)
  - References `featbit-terraform-aws` as the recommended ECS Fargate Terraform baseline
  - High Availability setup guide: ALB routing, multi-AZ ECS task placement
  - Clarifies fault tolerance (same-AZ redundancy) vs. true High Availability (cross-AZ)
  - Points to `featbit-deployment-kubernetes` skill for EKS deployments
- Updated plugin from 14 to 15 specialized skills

---

## [1.4.0] - 2026-02-03

### Improved
- **Enhanced All SDK Documentation** - Comprehensive documentation improvements across all SDK skills
  - **React Native SDK**: Enhanced integration guidance with detailed initialization patterns and best practices
  - **React Client SDK**: Added comprehensive reference files covering initialization, flag consumption, user management, and configuration
  - **Python SDK**: Detailed integration guidelines with bootstrapping, lifecycle, data synchronization, evaluation, and offline mode
  - **.NET Server SDK**: Enhanced with OpenFeature integration details and updated examples
  - **Java SDK**: Improved documentation with OpenFeature integration guidance
  - **JavaScript & Node.js SDKs**: Enhanced with OpenFeature integration patterns and clearer examples
  - **Go SDK**: Enhanced for better clarity and implementation details
  
- **Improved Deployment Documentation**
  - **Kubernetes Deployment**: Updated for better clarity and completeness
  
- **Enhanced Observability Documentation**
  - **OpenTelemetry Skill**: Refined documentation for improved clarity
  
### Changed
- All SDK skills now follow consistent documentation patterns with improved examples
- Better progressive disclosure structure across all skills
- More actionable code examples and integration guidance

## [1.3.0] - 2026-02-01

### Added
- **Enhanced Documentation Skill** (`featbit-documentation` v2.0.0)
  - New complete documentation index with 75+ documented URLs
  - Organized into 12 main topic areas for faster navigation
  - Each URL now includes detailed summaries for better context
  - Reference file: `references/complete-documentation-index.md`
  
- **Enhanced Docker Deployment Skill** (`featbit-deployment-docker` v2.0.0)
  - Comprehensive environment variables guide with 50+ configuration options
  - Detailed configuration guides for Standalone, Standard, and Professional deployments
  - Troubleshooting reference with common issues and solutions
  - New reference files structure for progressive disclosure

- **Enhanced shared skill authoring references**
  - New reference files on MCP integration patterns
  - Progressive disclosure implementation guide
  - Real-world examples and troubleshooting tips
  - Workflow patterns for common scenarios

### Improved
- Better content organization using progressive disclosure pattern
- Enhanced skill structure following official best practices
- More concise main skill files with detailed references

## [1.2.0] - 2026-01-30

### Added
- **New OpenTelemetry Observability Skill** (`featbit-opentelemetry`)
  - Comprehensive guidance for monitoring FeatBit services with OpenTelemetry
  - Configuration for Api, Evaluation-Server, and Data Analytic services
  - Ready-to-run examples with Seq (logs), Jaeger (traces), and Prometheus (metrics)
  - Integration guidance for observability backends (Datadog, New Relic, Grafana)
  - Environment variables and collector setup instructions
  - Official documentation link: https://docs.featbit.co/integrations/observability/opentelemetry

### Changed
- Updated plugin to include 14 specialized skills (was 13)
- Enhanced platform & infrastructure category with observability capabilities

## [1.1.2] - 2026-01-28

### Added
- **IAM Q&A**: Added guidance on setting custom policies as default for new users
  - Step-by-step instructions for configuring default IAM groups
  - Best practices for using groups to manage policies at scale
  - Added to `featbit-documentation` skill under IAM section

## [1.1.1] - 2026-01-28

### Changed
- Split `featbit-deployment` skill into two specialized skills:
  - `featbit-deployment-docker`: Dedicated Docker Compose deployment guide
  - `featbit-deployment-kubernetes`: Dedicated Kubernetes/Helm deployment guide
- Fixed default login credentials in Docker skill (corrected from `admin@featbit.com` to `test@featbit.com`)
- Added comprehensive official resources section with direct links to:
  - Documentation guides
  - Source code and configuration files (docker-compose.yml, init scripts)
  - Docker Hub images
  - Infrastructure as Code templates

### Improved
- Enhanced deployment documentation accuracy by referencing official GitHub repository files
- Better skill organization for improved LLM retrieval efficiency
- Added verification links for all deployment configurations

## [1.1.0] - 2026-01-28

### Added
- Claude Code marketplace support with `.claude-plugin/` directory
- `plugin.json` manifest for Claude Code plugin system
- `marketplace.json` catalog for easy distribution
- Comprehensive installation guide for Claude Code users
- Users can now install via `/plugin marketplace add featbit/featbit-skills`

### Changed
- Updated README with Claude Code marketplace installation instructions
- Enhanced documentation for better Claude Code integration
## [1.0.0] - 2026-01-27

### Added

#### Platform & Deployment
- **featbit-documentation**: Complete FeatBit platform documentation skill covering 75+ documentation topics
- **featbit-deployment**: Comprehensive deployment guidance for Docker Compose, Kubernetes, and Terraform

#### Server-Side SDKs
- **featbit-dotnet-sdk**: .NET Server SDK integration for ASP.NET Core and console applications
- **featbit-node-server-sdk**: Node.js Server SDK for Express, Koa, and NestJS frameworks
- **featbit-python-sdk**: Python Server SDK for Flask, Django, and FastAPI applications
- **featbit-java-sdk**: Java Server SDK for Spring Boot and servlet applications
- **featbit-go-sdk**: Go Server SDK for Gin and Echo frameworks

#### Client-Side SDKs
- **featbit-javascript-client-sdk**: JavaScript Client SDK for browser applications
- **featbit-react-client-sdk**: React SDK with hooks and components for React and Next.js
- **featbit-react-native-sdk**: React Native SDK for iOS and Android mobile applications

#### OpenFeature Providers
- **featbit-openfeature-node-server**: OpenFeature provider for Node.js server applications
- **featbit-openfeature-js-client**: OpenFeature provider for JavaScript client applications

### Features
- All 12 skills include comprehensive code examples
- Framework-specific integration guides
- Best practices and troubleshooting sections
- Real-time flag updates and event tracking
- Complete API reference for each SDK
- TypeScript support where applicable

### Documentation
- Complete README with installation instructions
- Individual skill documentation with 10,000+ lines of content
- Usage examples for all major frameworks
- Official resource links and GitHub repositories
