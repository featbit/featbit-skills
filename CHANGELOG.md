# Changelog

All notable changes to the FeatBit Claude Code Skills plugin will be documented in this file.

The format is based on [Keep a Changelog](https://keepachangelog.com/en/1.0.0/),
and this project adheres to [Semantic Versioning](https://semver.org/spec/v2.0.0.html).

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

---

## Future Releases

### Planned for 1.1.0
- Additional language SDKs (Ruby, PHP, Rust)
- Mobile SDKs (Android, iOS native)
- CLI tool integration
- Advanced experimentation patterns

### Under Consideration
- VS Code extension specific features
- Code snippets and templates
- Interactive examples
- Video tutorials integration

---

**Note**: This is the initial release with comprehensive coverage of all major FeatBit SDKs and deployment options.
