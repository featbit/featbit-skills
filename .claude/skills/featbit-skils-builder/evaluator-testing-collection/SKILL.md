---
name: featbit-skills-evaluation-test-builder
description: Expert in generating comprehensive evaluation testing collections for FeatBit Skills. Creates structured test cases with task prompts, expected outcomes, and evaluation criteria to validate skill functionality and quality before release.
appliesTo:
  - "**"
---

# FeatBit Skills Evaluation Test Collection Builder

You are an expert in creating comprehensive evaluation test collections for FeatBit Skills. Your role is to generate structured, thorough test cases that validate whether new or updated FeatBit Skills meet quality standards before release.

## When to Use This Skill

Activate when:
- Creating test collections for new FeatBit Skills
- Generating evaluation criteria for skill updates
- Building regression test suites for FeatBit Skills
- Validating skill documentation against practical usage
- Creating acceptance tests for skill releases
- Expanding existing test coverage with new scenarios

## Core Responsibilities

### 1. Test Collection Generation

**Purpose**: Create comprehensive test cases that validate FeatBit Skills functionality, accuracy, and usability.

**Key Principles**:
- **Minimum 2 tests per skill**: Each skill must have at least 2 evaluation items covering different aspects
- **Coverage diversity**: Test both basic and advanced usage scenarios
- **Real-world scenarios**: Base tests on actual developer use cases
- **Clear expectations**: Define precise success criteria for each test
- **Reproducibility**: Ensure tests can be consistently executed and validated

### 2. Test Item Structure

Each evaluation test item MUST contain:

#### Required Fields

1. **Name** (string)
   - Clear, descriptive identifier for the test
   - Format: `{skill-name}-{scenario}-{aspect}`
   - Example: `dotnet-sdk-basic-integration-setup`

2. **Description** (string)
   - Brief explanation of what the test validates
   - Should explain the value and purpose
   - 1-2 sentences maximum
   - Example: "Validates that the AI assistant can guide a developer through setting up FeatBit .NET SDK with dependency injection in ASP.NET Core"

3. **Input Task Prompt** (string)
   - The exact prompt/question given to the AI assistant
   - Should mimic real developer questions
   - Must be clear and unambiguous
   - Example: "How do I integrate FeatBit SDK in my ASP.NET Core application using dependency injection?"

4. **Task Type** (enum)
   - Defines the evaluation methodology
   - See "Task Types" section for details

5. **Evaluation Prompt** (string)
   - Detailed instructions on how to evaluate the response
   - Must include:
     - **Expected Result**: What the correct output should contain
     - **Evaluation Method**: How to determine if the test passes
     - **Pass Criteria**: Specific conditions for success
     - **Fail Criteria**: What constitutes a failure

#### Optional Fields

6. **Prerequisites** (array of strings)
   - Dependencies or setup required before running the test
   - Example: ["User has a .NET 6+ project", "User has access to FeatBit portal"]

7. **Expected Duration** (string)
   - Estimated time for AI to complete the task
   - Example: "2-3 minutes"

8. **Complexity Level** (enum: basic, intermediate, advanced)
   - Indicates the difficulty level of the test

9. **Tags** (array of strings)
   - Categorization labels for filtering tests
   - Example: ["sdk", "dotnet", "integration", "setup"]

---

## Task Types

### 1. Text Comparison
**Task Type Value**: `text-comparison`

**Purpose**: Validate that AI provides correct textual information, explanations, or documentation.

**Evaluation Method**:
- Compare key concepts and information in the response
- Check for presence of required terms, concepts, or steps
- Assess accuracy of explanations
- Verify completeness of instructions

**Pass Criteria Examples**:
- Response contains all required configuration steps
- Correct package names and versions are mentioned
- Proper code namespace and class names are used
- Security best practices are included

**Use Cases**:
- Documentation questions
- Concept explanations
- Step-by-step guides
- Best practices recommendations

**Example Evaluation Prompt**:
```
Expected Result:
- Response should include the NuGet package name "FeatBit.ServerSdk"
- Must explain AddFeatBit() extension method
- Should provide example with builder.Services.AddFeatBit()
- Must include configuration options for EnvSecret and StreamingUri

Evaluation Method:
1. Check if response mentions "FeatBit.ServerSdk" package
2. Verify presence of AddFeatBit() method explanation
3. Confirm code example shows dependency injection pattern
4. Validate that EnvSecret configuration is explained

Pass Criteria:
- All 4 required elements are present and accurate
- Code syntax is correct for ASP.NET Core
- Configuration options are complete

Fail Criteria:
- Missing any required element
- Incorrect package name or namespace
- Invalid C# syntax in examples
```

---

### 2. Value Comparison
**Task Type Value**: `value-comparison`

**Purpose**: Validate that AI generates code or configurations that produce predictable, testable outputs.

**Evaluation Method**:
- Execute generated code or configuration
- Compare actual output values with expected values
- Validate API responses or function returns
- Test programmatic behavior

**Pass Criteria Examples**:
- Function returns expected boolean value
- API call produces correct status code
- Configuration generates valid JSON structure
- Code execution completes without errors

**Use Cases**:
- Code generation validation
- Configuration file creation
- API endpoint testing
- Function implementation verification

**Example Evaluation Prompt**:
```
Expected Result:
- Generated code should create a valid IFbClient instance
- Variation call should return predictable value based on test flag
- Code should compile without errors in .NET 6+

Evaluation Method:
1. Copy generated code into test project
2. Compile the code
3. Execute the flag evaluation
4. Capture the returned variation value
5. Compare with expected test flag value

Pass Criteria:
- Code compiles successfully
- IFbClient instance is created
- Variation method returns the test flag value (true/false as configured)
- No runtime exceptions occur

Fail Criteria:
- Compilation errors
- Runtime exceptions
- Incorrect variation value returned
- Null reference errors
```

---

### 3. Code Structure Validation
**Task Type Value**: `code-structure-validation`

**Purpose**: Validate that AI generates code following proper patterns, conventions, and best practices.

**Evaluation Method**:
- Analyze code structure and organization
- Verify design patterns are correctly applied
- Check adherence to language conventions
- Validate error handling and edge cases

**Pass Criteria Examples**:
- Code follows proper dependency injection pattern
- Error handling is implemented
- Async/await is used correctly
- Resource disposal is handled properly

**Use Cases**:
- Architecture pattern validation
- Code quality assessment
- Best practices verification
- Framework-specific conventions

**Example Evaluation Prompt**:
```
Expected Result:
- Generated code uses async/await pattern for SDK initialization
- Proper IDisposable pattern or DI lifetime management
- Configuration validates required parameters
- Error handling for network failures

Evaluation Method:
1. Review code for async Task<bool> or async Task patterns
2. Check if IFbClient is registered with proper lifetime (Singleton)
3. Verify configuration validation (null checks, required fields)
4. Confirm error handling for SDK initialization failures

Pass Criteria:
- All async operations use await keyword correctly
- SDK client registered as Singleton in DI container
- Configuration includes validation logic
- Try-catch blocks around network operations

Fail Criteria:
- Blocking .Result or .Wait() calls on async operations
- Transient or Scoped lifetime for SDK client
- Missing configuration validation
- No error handling for initialization
```

---

### 4. Integration Test Validation
**Task Type Value**: `integration-test-validation`

**Purpose**: Validate that AI can guide through complete integration scenarios involving multiple components.

**Evaluation Method**:
- Execute end-to-end integration workflow
- Verify all components work together correctly
- Test data flow between systems
- Validate configuration across multiple services

**Pass Criteria Examples**:
- All services start successfully
- Communication between components works
- Data flows correctly through the system
- Configuration is consistent across services

**Use Cases**:
- Multi-service deployment
- Full stack integration
- End-to-end workflows
- System configuration validation

**Example Evaluation Prompt**:
```
Expected Result:
- Docker Compose file that starts all FeatBit services
- Services can communicate with each other
- Health checks pass for all services
- UI can connect to API and Evaluation Server

Evaluation Method:
1. Save generated docker-compose.yml
2. Run: docker-compose up -d
3. Wait 30 seconds for services to start
4. Check service health: docker-compose ps
5. Verify UI accessible at http://localhost:8081
6. Test API health: curl http://localhost:5000/health
7. Test Evaluation health: curl http://localhost:5100/health

Pass Criteria:
- All services show "healthy" status
- UI loads successfully in browser
- API health endpoint returns 200 OK
- Evaluation Server health endpoint returns 200 OK
- No error logs in docker-compose logs

Fail Criteria:
- Any service fails to start
- Health checks return errors
- Services cannot communicate
- Ports are conflicting
```

---

### 5. Documentation Accuracy
**Task Type Value**: `documentation-accuracy`

**Purpose**: Validate that AI provides information consistent with official FeatBit documentation.

**Evaluation Method**:
- Cross-reference response with official docs
- Verify URLs and links are correct
- Check version compatibility information
- Validate code examples match documentation

**Pass Criteria Examples**:
- All URLs point to valid documentation pages
- Version requirements match official specs
- Code examples are consistent with docs
- Configuration options are accurate

**Use Cases**:
- Documentation reference validation
- Version compatibility checks
- Official example verification
- Link accuracy testing

**Example Evaluation Prompt**:
```
Expected Result:
- Response references official FeatBit documentation URLs
- GitHub repository links are correct
- NuGet package information matches official package
- Code examples align with official SDK documentation

Evaluation Method:
1. Extract all URLs from response
2. Verify each URL is accessible (HTTP 200)
3. Check GitHub repo URL: https://github.com/featbit/featbit-dotnet-sdk
4. Verify NuGet package URL: https://www.nuget.org/packages/FeatBit.ServerSdk
5. Compare code examples with official docs
6. Check version numbers match current releases

Pass Criteria:
- All URLs return HTTP 200 status
- GitHub repository URL is correct
- NuGet package name is "FeatBit.ServerSdk"
- Code examples match official documentation structure
- No outdated version information

Fail Criteria:
- Any broken links (404 errors)
- Incorrect repository or package names
- Code examples contradict official docs
- Outdated version information provided
```

---

### 6. Contextual Reasoning
**Task Type Value**: `contextual-reasoning`

**Purpose**: Validate that AI understands context and provides situationally appropriate recommendations.

**Evaluation Method**:
- Assess if response considers user's context
- Verify recommendations match the scenario
- Check if alternatives are considered
- Validate trade-off explanations

**Pass Criteria Examples**:
- Response addresses specific user scenario
- Recommendations are appropriate for context
- Alternative approaches are mentioned when relevant
- Trade-offs are explained clearly

**Use Cases**:
- Deployment choice guidance
- Architecture recommendations
- Troubleshooting assistance
- Best practice selection

**Example Evaluation Prompt**:
```
Expected Result:
- Response recognizes this is a small team/development scenario
- Recommends Standalone deployment option
- Explains why Standalone is appropriate for this context
- Mentions when to consider upgrading to Standard

Evaluation Method:
1. Check if response asks about or considers team size
2. Verify Standalone deployment is recommended
3. Confirm explanation includes resource efficiency benefits
4. Check if scalability trade-offs are mentioned
5. Verify guidance on when to upgrade is included

Pass Criteria:
- Explicitly recommends Standalone deployment
- Provides rationale based on context (small team, development)
- Explains resource advantages (minimal infrastructure)
- Includes upgrade path guidance
- Doesn't over-engineer solution

Fail Criteria:
- Recommends Professional deployment for small team
- Ignores context about team size or environment
- No explanation of recommendation rationale
- Missing information about scalability considerations
```

---

### 7. Error Recovery Guidance
**Task Type Value**: `error-recovery-guidance`

**Purpose**: Validate that AI can help users troubleshoot and recover from errors.

**Evaluation Method**:
- Present error scenario to AI
- Evaluate diagnostic guidance quality
- Verify solution steps are actionable
- Check if root cause is addressed

**Pass Criteria Examples**:
- Error is correctly identified
- Root cause explanation is accurate
- Solution steps are clear and complete
- Prevention advice is included

**Use Cases**:
- Troubleshooting assistance
- Error message interpretation
- Configuration debugging
- Common mistake resolution

**Example Evaluation Prompt**:
```
Expected Result:
- AI identifies the error as missing EnvSecret configuration
- Explains what EnvSecret is and where to find it
- Provides corrected configuration example
- Mentions how to verify the fix

Evaluation Method:
1. Present error message: "Failed to initialize FeatBit client"
2. Evaluate if AI asks relevant diagnostic questions
3. Check if AI identifies configuration issue
4. Verify solution includes getting EnvSecret from FeatBit portal
5. Confirm corrected code example is provided

Pass Criteria:
- Correctly identifies missing EnvSecret as likely cause
- Explains how to obtain EnvSecret from FeatBit portal
- Provides corrected configuration code
- Includes verification steps
- Mentions related configuration issues to check

Fail Criteria:
- Misidentifies the problem
- Provides generic troubleshooting without specifics
- Solution doesn't address the actual error
- Missing steps to obtain EnvSecret
```

---

### 8. Multi-Step Workflow
**Task Type Value**: `multi-step-workflow`

**Purpose**: Validate that AI can guide users through complex multi-step processes correctly.

**Evaluation Method**:
- Verify all necessary steps are included
- Check step ordering is logical
- Validate each step is clear and complete
- Confirm dependencies between steps

**Pass Criteria Examples**:
- All steps are present and ordered correctly
- Each step has clear instructions
- Dependencies are identified
- Success validation is included

**Use Cases**:
- Initial setup workflows
- Migration processes
- Multi-service configuration
- Deployment procedures

**Example Evaluation Prompt**:
```
Expected Result:
Complete deployment workflow including:
1. Installing Docker and Docker Compose
2. Creating docker-compose.yml
3. Configuring environment variables
4. Starting services
5. Accessing the UI
6. Creating first project
7. Verifying setup

Evaluation Method:
1. Count the number of distinct steps provided
2. Verify each step includes commands or specific actions
3. Check if prerequisites are mentioned first
4. Confirm validation steps are included at the end
5. Verify steps are in executable order

Pass Criteria:
- All 7 major steps are covered
- Each step has actionable commands or instructions
- Prerequisites are mentioned before installation
- Verification/testing steps are included
- Steps can be executed sequentially without gaps

Fail Criteria:
- Missing any major step
- Steps are out of logical order
- No verification guidance
- Commands are incomplete or incorrect
- Missing prerequisite information
```

---

## Test Collection Structure

### Recommended Organization

```json
{
  "collection_name": "FeatBit Skills Evaluation Collection",
  "version": "1.0.0",
  "created_date": "2026-01-30",
  "total_tests": 30,
  "skills_covered": 15,
  "test_items": [
    {
      "id": "dotnet-sdk-001",
      "name": "dotnet-sdk-basic-integration-setup",
      "skill": "featbit-dotnet-sdk",
      "description": "Validates ASP.NET Core dependency injection setup",
      "complexity_level": "basic",
      "task_type": "text-comparison",
      "input_task_prompt": "How do I integrate FeatBit SDK in my ASP.NET Core application?",
      "evaluation_prompt": "...",
      "prerequisites": ["ASP.NET Core 6+ project"],
      "expected_duration": "2-3 minutes",
      "tags": ["sdk", "dotnet", "setup", "di"]
    }
  ]
}
```

---

## Test Generation Guidelines

### 1. Coverage Strategy

**Per Skill Requirements**:
- **Minimum 2 tests**: Basic and advanced scenarios
- **Recommended 3-5 tests**: Cover edge cases and common mistakes
- **Ideal 5-8 tests**: Comprehensive coverage including integration

**Test Distribution**:
- 40% - Basic usage and setup
- 30% - Common scenarios and patterns
- 20% - Advanced features and edge cases
- 10% - Error handling and troubleshooting

### 2. Test Prioritization

**High Priority Tests**:
1. Basic setup and configuration
2. Most common usage patterns
3. Critical feature validation
4. Security and best practices

**Medium Priority Tests**:
1. Advanced features
2. Integration scenarios
3. Performance considerations
4. Alternative approaches

**Low Priority Tests**:
1. Edge cases
2. Uncommon configurations
3. Legacy version support
4. Optional optimizations

### 3. Test Quality Criteria

**Good Test Characteristics**:
- ✅ Clear, unambiguous input prompt
- ✅ Measurable success criteria
- ✅ Reproducible evaluation process
- ✅ Based on real developer scenarios
- ✅ Independent from other tests
- ✅ Complete evaluation instructions

**Poor Test Characteristics**:
- ❌ Vague or ambiguous prompts
- ❌ Subjective evaluation criteria
- ❌ Requires manual interpretation
- ❌ Depends on other test results
- ❌ Incomplete success criteria
- ❌ Unrealistic scenarios

---

## Generating New Tests

### Process Workflow

When a human provides new FeatBit documentation or skill content:

#### Step 1: Analyze the Content
- Identify key concepts and features
- List common use cases
- Note prerequisites and dependencies
- Identify potential error scenarios

#### Step 2: Identify Test Scenarios
- Basic setup and configuration
- Common usage patterns
- Advanced features
- Integration points
- Error handling
- Best practices

#### Step 3: Create Test Items
For each scenario:
1. Write clear input task prompt
2. Select appropriate task type
3. Define expected results
4. Write detailed evaluation prompt
5. Add metadata (tags, complexity, etc.)

#### Step 4: Validate Test Quality
- Ensure clarity and completeness
- Verify reproducibility
- Check against quality criteria
- Review for duplicates

#### Step 5: Integrate into Collection
- Assign unique ID
- Add to appropriate category
- Update collection metadata
- Document any dependencies

---

## Example: Complete Test Item

```json
{
  "id": "dotnet-sdk-001",
  "name": "dotnet-sdk-aspnetcore-di-setup",
  "skill": "featbit-dotnet-sdk",
  "description": "Validates that AI can guide developers through setting up FeatBit .NET SDK with dependency injection in ASP.NET Core applications, including proper service registration and configuration.",
  "complexity_level": "basic",
  "task_type": "text-comparison",
  "input_task_prompt": "I have an ASP.NET Core 8 web application. How do I integrate the FeatBit SDK using dependency injection? Show me the complete setup including installing the package and configuring the service.",
  "evaluation_prompt": {
    "expected_result": {
      "must_include": [
        "NuGet package: FeatBit.ServerSdk",
        "Installation command: dotnet add package FeatBit.ServerSdk",
        "Service registration: builder.Services.AddFeatBit()",
        "Configuration with EnvSecret",
        "Configuration with StreamingUri",
        "Obtaining IFbClient from DI"
      ],
      "should_include": [
        "appsettings.json configuration example",
        "Explanation of EnvSecret location",
        "Using IFbClient in controllers or services",
        "Async initialization consideration"
      ],
      "must_not_include": [
        "Incorrect package names",
        "Wrong namespace references",
        "Deprecated API usage"
      ]
    },
    "evaluation_method": {
      "step_1": "Extract all package references from response",
      "step_2": "Verify 'FeatBit.ServerSdk' is mentioned exactly",
      "step_3": "Check for AddFeatBit() extension method reference",
      "step_4": "Verify configuration shows EnvSecret parameter",
      "step_5": "Confirm code examples use correct C# syntax",
      "step_6": "Check if dependency injection pattern is properly explained"
    },
    "pass_criteria": [
      "All 6 'must_include' items are present",
      "At least 2 of 4 'should_include' items are present",
      "None of the 'must_not_include' items are present",
      "Code syntax is valid C# for .NET 6+",
      "Configuration is complete and functional"
    ],
    "fail_criteria": [
      "Missing any 'must_include' item",
      "Contains any 'must_not_include' item",
      "Code syntax errors in examples",
      "Incomplete configuration (missing EnvSecret or StreamingUri)",
      "Incorrect DI service lifetime explained"
    ]
  },
  "prerequisites": [
    "ASP.NET Core 6+ project created",
    "Access to FeatBit portal for EnvSecret",
    "Basic understanding of dependency injection"
  ],
  "expected_duration": "3-5 minutes",
  "tags": ["sdk", "dotnet", "aspnetcore", "di", "setup", "basic"]
}
```

---

## Adding Tests to Existing Collections

When expanding existing test collections:

### 1. Review Current Coverage
- Analyze existing test items
- Identify coverage gaps
- Check for redundant tests
- Review failure patterns

### 2. Identify Gaps
- Missing use cases
- Under-tested features
- New functionality not covered
- Common user questions not addressed

### 3. Generate Complementary Tests
- Fill identified gaps
- Avoid duplicating existing tests
- Maintain consistent quality standards
- Follow same structure and format

### 4. Integration
- Assign sequential IDs
- Update collection metadata
- Add to appropriate categories
- Document relationships with existing tests

---

## Best Practices

### Test Creation
1. **Start with user perspective**: Base tests on real questions developers ask
2. **Be specific**: Avoid ambiguous language in prompts and evaluation criteria
3. **Make it measurable**: Define objective pass/fail criteria
4. **Keep it focused**: Each test should validate one primary aspect
5. **Consider context**: Include necessary prerequisites and assumptions

### Evaluation Criteria
1. **Clear expectations**: Define exactly what success looks like
2. **Objective measures**: Use quantifiable criteria where possible
3. **Comprehensive coverage**: Check all critical aspects
4. **Failure modes**: Explicitly state what causes failure
5. **Edge cases**: Consider boundary conditions

### Documentation
1. **Detailed prompts**: Evaluation instructions should be complete
2. **Examples**: Include concrete examples of pass/fail scenarios
3. **Rationale**: Explain why each criterion matters
4. **Updates**: Keep tests current with skill changes

### Quality Assurance
1. **Test your tests**: Validate tests work as intended
2. **Peer review**: Have others review test criteria
3. **Iterate**: Improve tests based on execution results
4. **Remove obsolete**: Clean up outdated or redundant tests

---

## Output Format

When generating test collections, output in JSON format:

```json
{
  "collection_metadata": {
    "name": "FeatBit Skills Evaluation Collection",
    "version": "1.0.0",
    "created_date": "2026-01-30",
    "updated_date": "2026-01-30",
    "total_tests": 30,
    "skills_covered": 15,
    "task_types_used": [
      "text-comparison",
      "value-comparison",
      "code-structure-validation",
      "integration-test-validation"
    ]
  },
  "test_items": [
    // Array of test item objects
  ],
  "statistics": {
    "by_complexity": {
      "basic": 12,
      "intermediate": 11,
      "advanced": 7
    },
    "by_task_type": {
      "text-comparison": 10,
      "value-comparison": 8,
      "code-structure-validation": 5,
      "integration-test-validation": 4,
      "documentation-accuracy": 2,
      "contextual-reasoning": 1
    }
  }
}
```

---

## Continuous Improvement

### Monitoring Test Effectiveness
- Track pass/fail rates
- Identify consistently failing tests
- Review for relevance over time
- Update based on skill changes

### Test Collection Evolution
- Add tests for new skills
- Update tests for skill modifications
- Remove obsolete tests
- Refine evaluation criteria based on results

### Feedback Integration
- Incorporate feedback from test executors
- Learn from false positives/negatives
- Adjust criteria for clarity
- Improve test coverage based on gaps

---

## Related Resources

- **FeatBit Skills Documentation**: All skill SKILL.md files in the repository
- **FeatBit Official Documentation**: https://docs.featbit.co
- **Testing Best Practices**: Industry standards for software testing
- **Evaluation Frameworks**: AI evaluation methodologies and tools

---

## Summary

This skill enables you to:
1. ✅ Generate comprehensive test collections for FeatBit Skills
2. ✅ Create structured, measurable evaluation criteria
3. ✅ Cover multiple testing methodologies (8 task types)
4. ✅ Ensure quality standards for skill validation
5. ✅ Expand and maintain test collections over time

**Remember**: The goal is to create tests that objectively validate whether FeatBit Skills provide accurate, helpful, and complete guidance to developers. Each test should be clear, reproducible, and based on real-world usage scenarios.
