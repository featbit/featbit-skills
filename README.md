# FeatBit Skills for Claude Code

[![License: MIT](https://img.shields.io/badge/License-MIT-yellow.svg)](https://opensource.org/licenses/MIT)
[![Version](https://img.shields.io/badge/version-1.1.0-blue.svg)](CHANGELOG.md)

Official Claude Code skills plugin for [FeatBit](https://featbit.co) - an open-source, powerful, and scalable feature flags and A/B testing platform.

## 🎯 What is This?

This plugin provides comprehensive knowledge and guidance for integrating FeatBit into your applications through **13 specialized skills** covering platform documentation, deployment strategies, and SDK implementations across multiple programming languages and frameworks.

With these skills, you can:

- 🚀 Get instant guidance on feature flag implementation strategies
- 🔧 Learn SDK integration for 7+ programming languages and 15+ frameworks
- ☁️ Understand deployment options: Docker, Kubernetes, Terraform
- 📚 Access comprehensive documentation (75+ topics) without leaving your editor
- 💡 Receive contextual help based on your code and questions
- ⚡ Integrate OpenFeature providers for vendor-agnostic feature flagging

## 📦 Installation

### Quick Install (Recommended)

Install directly from Claude Code marketplace:

```bash
# In Claude Code
/plugin marketplace add featbit/featbit-skills
/plugin install featbit-skills@featbit-marketplace
```

That's it! All 13 skills are now available in Claude Code.

### Verify Installation

```bash
/plugin list
```

You should see `featbit-skills` with status "Enabled".

### Detailed Installation Guide

For troubleshooting, enterprise setup, or manual installation, see the [complete installation guide](docs/INSTALLATION.md).
   ln -s "$(pwd)" ~/.claude-code/skills/featbit
   
   # Windows (PowerShell as Administrator)
   New-Item -ItemType SymbolicLink -Path "$env:APPDATA\Claude Code\skills\featbit" -Target (Get-Location)
   ```

3. Restart Claude Code or reload skills:
   ```bash
   # In Claude Code
   /skills reload
   ```

### Verifying Installation

After installation, verify the skills are loaded:

```bash
# In Claude Code
/skills list
```

You should see all 13 FeatBit skills listed.

## 🎓 Available Skills

This plugin includes **13 specialized skills** organized by category:

### Platform & Infrastructure

#### **1. FeatBit Documentation** (`featbit-documentation`)
Complete knowledge of FeatBit's official documentation covering 75+ topics:
- ✅ Feature flags management and targeting rules
- ✅ Experimentation and A/B testing workflows
- ✅ Relay proxy configuration and usage
- ✅ REST API and programmatic access
- ✅ IAM, security, and access control
- ✅ Installation and deployment guides
- ✅ Integrations: SSO, observability, webhooks
- ✅ Architecture and technical stack

**Activates when**: You ask about FeatBit features, concepts, best practices, or general "how-to" questions.

#### **2. FeatBit Docker Deployment** (`featbit-deployment-docker`)
Expert guidance for Docker Compose deployments:
- 🐳 Docker Compose configurations (Standalone, Standard, Professional)
- 🔧 Environment variables and container orchestration
- 🔒 Security best practices and secrets management
- 💾 Backup and recovery strategies
- 📊 Monitoring with Prometheus and Grafana
- ⚡ Performance tuning and optimization
- 🌐 Reverse proxy with nginx and SSL/TLS

**Activates when**: Working with `docker-compose.yml`, `Dockerfile`, or Docker-based deployment questions.

**Example**: "How do I deploy FeatBit with Docker Compose using MongoDB and Redis?"

#### **3. FeatBit Kubernetes Deployment** (`featbit-deployment-kubernetes`)
Expert guidance for Kubernetes/Helm deployments:
- ☸️ Helm Charts installation and configuration
- 🌐 Service exposure (Ingress, LoadBalancer, NodePort)
- 📈 Auto-scaling with HPA
- ☁️ Cloud provider integrations (AKS, EKS, GKE)
- 🔐 TLS/SSL with cert-manager and Let's Encrypt
- 🗄️ External database configurations
- 🔄 Upgrade and rollback strategies

**Activates when**: Working with `values.yaml`, Kubernetes manifests, or Helm deployment questions.

**Example**: "How do I deploy FeatBit to Azure AKS with automatic SSL certificates?"

---

### Server-Side SDKs

#### **4. FeatBit .NET SDK** (`featbit-dotnet-sdk`)
Expert guidance for .NET Server SDK integration:
- 🏗️ ASP.NET Core with dependency injection
- 🖥️ Console applications and worker services
- 🔄 All variation types (bool, string, int, double, JSON)
- 📈 Custom event tracking for A/B testing
- ⚙️ Configuration via appsettings.json
- 🔧 Best practices and troubleshooting

**Activates when**: Working with `.cs`, `.csproj`, `Program.cs`, `Startup.cs` files or .NET integration questions.

**Example**: "How do I integrate FeatBit in my ASP.NET Core API?"

#### **5. FeatBit Node.js Server SDK** (`featbit-node-server-sdk`)
Expert guidance for Node.js Server SDK:
- ⚡ Express, Koa, NestJS framework integration
- 👥 Multi-user server applications
- 🔌 WebSocket and polling data synchronization
- 📊 Event tracking for experiments
- 📘 Full TypeScript support

**Activates when**: Working with `server.js`, `app.js`, or Node.js backend files.

**Example**: "Set up FeatBit in my Express API"

#### **6. FeatBit Python SDK** (`featbit-python-sdk`)
Expert guidance for Python Server SDK:
- 🌶️ Flask and Django framework integration
- ⚡ FastAPI support
- 🔌 WebSocket streaming for real-time updates
- 📴 Offline mode for degraded scenarios
- 📊 Event tracking and analytics

**Activates when**: Working with `.py` or Python backend files.

**Example**: "Integrate FeatBit with my Flask application"

#### **7. FeatBit Java SDK** (`featbit-java-sdk`)
Expert guidance for Java Server SDK:
- Spring Boot integration
- Maven and Gradle setup
- Servlet applications
- Offline mode and bootstrapping
- 🍃 Spring Boot integration with auto-configuration
- 📦 Maven and Gradle project setup
- 🏗️ Multi-module project support
- 🔔 Flag change listeners and event callbacks
- 🛡️ Thread-safe operations

**Activates when**: Working with `.java`, `pom.xml`, `build.gradle` files.

**Example**: "How to use FeatBit in Spring Boot?"

#### **8. FeatBit Go SDK** (`featbit-go-sdk`)
Expert guidance for Go Server SDK:
- 🍸 Gin and Echo framework integration
- 🌐 Standard HTTP server support
- 🔌 WebSocket streaming for real-time updates
- 📴 Offline mode support
- 📊 Event tracking and analytics

**Activates when**: Working with `.go`, `go.mod` files.

**Example**: "Integrate FeatBit in my Gin API"

---

### Client-Side SDKs

#### **9. FeatBit JavaScript Client SDK** (`featbit-javascript-client-sdk`)
Expert guidance for browser JavaScript SDK:
- 🌐 Vanilla JavaScript integration
- 🖥️ Browser applications (SPA, MPA)
- 🔄 Real-time flag updates via WebSocket
- 👤 Single-user context management
- 📊 Event tracking and analytics

**Activates when**: Working with `.html`, browser `.js` files.

**Example**: "Add feature flags to my vanilla JS app"

#### **10. FeatBit React Client SDK** (`featbit-react-client-sdk`)
Expert guidance for React SDK:
- ⚛️ React hooks: `useFlags()`, `useFbClient()`
- 🎯 Provider pattern with `FbProvider`
- ⚡ Next.js SSR/SSG integration
- 📘 Full TypeScript support
- 🔄 Real-time flag updates
- 🎨 Conditional rendering patterns

**Activates when**: Working with `.jsx`, `.tsx`, React component files.

**Example**: "Use feature flags in my React components"

#### **11. FeatBit React Native SDK** (`featbit-react-native-sdk`)
Expert guidance for React Native mobile SDK:
- 📱 iOS and Android platform support
- 📲 Device information integration
- 🎯 Platform-specific feature targeting
- 🧭 Navigation library integration
- 📴 Offline mode support

**Activates when**: Working with React Native files and mobile app development.

**Example**: "Implement feature flags in my React Native iOS app"

---

### OpenFeature Providers

#### **12. FeatBit OpenFeature Node.js Provider** (`featbit-openfeature-node-server`)
Expert guidance for vendor-neutral feature flagging:
- 🌐 OpenFeature standard API compliance
- ⚡ Express, Koa, NestJS integration
- 🔌 Event handling and hooks support
- 🔄 Provider switching without code changes
- 🛡️ Type-safe feature flag evaluation

**Activates when**: Working with OpenFeature implementations in Node.js.

**Example**: "Use OpenFeature with FeatBit in my Express API"

#### **13. FeatBit OpenFeature JavaScript Provider** (`featbit-openfeature-js-client`)
Expert guidance for browser OpenFeature integration:
- 🌐 Vendor-neutral browser feature flags
- ⚛️ React, Vue, Angular integration
- 🔄 Standardized flag evaluation API
- 🔌 Provider switching at runtime
- 📘 Full TypeScript support

**Activates when**: Working with OpenFeature implementations in browsers.

**Example**: "Integrate OpenFeature FeatBit provider in my React app"

---

## 💡 Usage Examples

Simply ask Claude questions while working in your codebase. The relevant skill will automatically activate based on your file context and questions:

### Example 1: Getting Started

**You**: "How do I add feature flags to my ASP.NET Core API?"

**Claude** (activating `featbit-dotnet-sdk`):
- OpenFeature standard API
- Express, NestJS integration
- Event handling
- Hooks support

**Activates when working with**: OpenFeature implementations in Node.js.

### 12. **FeatBit OpenFeature JavaScript Provider** (`featbit-openfeature-js-client`)
Expert guidance for OpenFeature JavaScript provider:
- Browser applications
- Vendor-neutral API
- React and Vue integration
- Standardized evaluation
- Provider switching

**Activates when working with**: OpenFeature implementations in browsers.

## Usage Examples

Simply ask Claude questions while working in your codebase. The relevant skill will automatically activate:

### Example 1: Feature Flag Integration

**You**: "How do I integrate FeatBit in my ASP.NET Core API?"

**Claude** (activating `featbit-dotnet-sdk`):
```
I'll help you integrate FeatBit into your ASP.NET Core API using dependency injection.

1. Install the SDK:
   dotnet add package FeatBit.ServerSdk

2. Configure in Program.cs:
   builder.Services.AddFeatBit(options => {
       options.EnvSecret = "your-env-secret";
       options.StreamingUri = "ws://localhost:5100";
   });

3. Use in your controller:
   public class FeatureController : ControllerBase {
       private readonly IFbClient _fbClient;
       
       public FeatureController(IFbClient fbClient) {
           _fbClient = fbClient;
       }
       
       [HttpGet]
       public async Task<IActionResult> GetFeature() {
           var user = new FbUser.Builder("user-123")
               .Name("John Doe")
               .Build();
           
           var enabled = await _fbClient.BoolVariation("new-feature", user, false);
           return Ok(new { enabled });
       }
   }
```

### Example 2: React Hooks

**You**: "Show me how to use feature flags in my React component"

**Claude** (activating `featbit-react-client-sdk`):
```
I'll show you how to use FeatBit with React hooks:

1. Install the SDK:
   npm install featbit-react-client-sdk

2. Wrap your app with FbProvider:
   import { FbProvider } from 'featbit-react-client-sdk';
   
   const options = {
     secret: 'your-env-secret',
     user: {
       keyId: 'user-123',
       name: 'John Doe'
     }
   };
   
   <FbProvider options={options}>
     <App />
   </FbProvider>

3. Use flags in components:
   import { useFlags, useFbClient } from 'featbit-react-client-sdk';
   
   function MyComponent() {
     const { 'new-ui-feature': newUI } = useFlags();
     const fbClient = useFbClient();
     
     return newUI ? <NewUI /> : <OldUI />;
   }
```

### Example 3: Docker Deployment

**You**: "How do I deploy FeatBit with Docker Compose?"

**Claude** (activating `featbit-deployment-docker`):
```
I'll guide you through deploying FeatBit in production with Docker Compose using the Standard edition:

1. Create docker-compose.yml:
   version: '3.8'
   services:
     featbit-api:
       image: featbit/featbit-api:latest
       environment:
         - POSTGRES_CONNECTION_STRING=Host=db;Database=featbit;...
       depends_on:
         - db
         - redis
     
     featbit-ui:
       image: featbit/featbit-ui:latest
       ports:
         - "8081:80"
     
     db:
       image: postgres:14
       volumes:
         - postgres_data:/var/lib/postgresql/data
     
     redis:
       image: redis:7

2. Start the services:
   docker-compose up -d

3. Configure monitoring with Prometheus:
   [Detailed configuration...]

4. Set up backup:
   [Backup strategy...]
```

---

## 🎯 Key Features

### Comprehensive Coverage
- **75+ Documentation Topics**: Complete platform knowledge
- **7 Programming Languages**: .NET, JavaScript/TypeScript, Python, Java, Go
- **15+ Frameworks**: ASP.NET Core, Express, React, Next.js, Django, Flask, FastAPI, Spring Boot, Gin, Echo, React Native
- **2 Deployment Methods**: Docker Compose (single-server), Kubernetes/Helm (cloud-native)

### Smart Activation
Skills automatically activate based on:
- 📁 **File Context**: Working with `.cs` files triggers .NET skill
- 💬 **Question Content**: Asking about "React" triggers React skill
- 🔍 **Code Analysis**: Claude understands your project structure

### Real-World Guidance
- ✅ Complete code examples with best practices
- ✅ Framework-specific integration patterns
- ✅ Common troubleshooting scenarios
- ✅ Production deployment strategies
- ✅ Performance optimization tips

---

## 📚 Resources

### Official Documentation
- **FeatBit Website**: [https://featbit.co](https://featbit.co)
- **Official Docs**: [https://docs.featbit.co](https://docs.featbit.co)
- **GitHub Repository**: [https://github.com/featbit/featbit](https://github.com/featbit/featbit)

### Support & Community
- **GitHub Issues**: [Report bugs or request features](https://github.com/featbit/featbit/issues)
- **Email Support**: support@featbit.co
- **Community**: Join our discussions on GitHub

### SDK Repositories
- **.NET**: [featbit/featbit-dotnet-sdk](https://github.com/featbit/featbit-dotnet-sdk)
- **JavaScript**: [featbit/featbit-js-client-sdk](https://github.com/featbit/featbit-js-client-sdk)
- **React**: [featbit/featbit-react-client-sdk](https://github.com/featbit/featbit-react-client-sdk)
- **Node.js**: [featbit/featbit-node-server-sdk](https://github.com/featbit/featbit-node-server-sdk)
- **Python**: [featbit/featbit-python-sdk](https://github.com/featbit/featbit-python-sdk)
- **Java**: [featbit/featbit-java-sdk](https://github.com/featbit/featbit-java-sdk)
- **Go**: [featbit/featbit-go-sdk](https://github.com/featbit/featbit-go-sdk)

---

## 🤝 Contributing

We welcome contributions to improve these skills!

### How to Contribute
1. Fork the repository
2. Create a feature branch: `git checkout -b feature/improvement`
3. Make your changes
4. Test thoroughly
5. Submit a pull request

### Areas for Contribution
- 📖 Documentation improvements
- 🐛 Bug fixes in skill content
- ✨ New code examples
- 🌐 Additional language support
- 🔧 Better activation patterns

See [CONTRIBUTING.md](docs/CONTRIBUTING.md) for detailed guidelines.

---

## 📝 License

This project is licensed under the MIT License - see the [LICENSE](LICENSE) file for details.

---

## 🔄 Changelog

See [CHANGELOG.md](CHANGELOG.md) for version history and release notes.

---

## ❓ FAQ

### Q: Do I need a FeatBit account to use these skills?
**A**: The skills provide knowledge and guidance without requiring an account. However, to actually use FeatBit, you'll need to deploy your own instance or use FeatBit's cloud service.

### Q: Which skill should I use for my project?
**A**: Skills activate automatically based on your file context. Just ask questions naturally, and Claude will use the appropriate skill(s).

### Q: Can I use multiple skills together?
**A**: Yes! Claude can combine knowledge from multiple skills. For example, asking about "deploying a Python app with FeatBit" might use both `featbit-python-sdk` and `featbit-deployment`.

### Q: Are these skills kept up to date?
**A**: We regularly update skills to match the latest FeatBit releases. Check [CHANGELOG.md](CHANGELOG.md) for update history.

### Q: How do I report an issue with a skill?
**A**: Open an issue on [GitHub](https://github.com/featbit/featbit-skills/issues) with details about the problem.

---

<p align="center">
  <strong>Made with ❤️ by the FeatBit Team</strong>
  <br>
  <a href="https://featbit.co">Website</a> • 
  <a href="https://docs.featbit.co">Documentation</a> • 
  <a href="https://github.com/featbit/featbit">GitHub</a>
</p>

