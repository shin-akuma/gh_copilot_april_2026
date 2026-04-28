# Lab 1: Custom Instructions and Custom Agents

## 🎯 Lab Objectives

By the end of this lab, you will:
- Understand the difference between Custom Instructions and Custom Agents
- Create workspace-specific custom instructions for your team's C# standards
- Build a specialized Custom Agent for generating unit tests
- See measurable improvement in Copilot output quality
- Experience the 10x productivity gain from proper Copilot customization

**Time Required:** 45-60 minutes

---

## 📊 The Problem We're Solving

Common developer frustrations with GitHub Copilot:
- **Inconsistent output quality** — generated code doesn't follow team standards
- **Low adoption of customization features** — most developers don't use custom instructions or agents
- **Context switching** — leaving the IDE for ChatGPT/Claude because Copilot "doesn't get it"
- **"Short memory"** — Copilot forgets your standards between conversations

**The Solution:** Custom Instructions and Custom Agents eliminate these problems permanently.

---

## 📋 Prerequisites

- VS Code with GitHub Copilot extension installed
- **AccelerateDevGHCopilot folder opened directly as your VS Code workspace** (File > Open Folder > select `AccelerateDevGHCopilot`). This is required so that the `.github/` folder you create in this lab will be at the workspace root, where VS Code Copilot expects it.
- GitHub Copilot Chat enabled
- Basic understanding of C# and xUnit testing

### One-Time Setup: Install FluentAssertions

The project already includes xUnit and NSubstitute, but **FluentAssertions** must be added to match our testing standards. Run these commands in the VS Code terminal:

```bash
dotnet add tests/UnitTests/UnitTests.csproj package FluentAssertions
dotnet restore
```

Verify it was installed:

```bash
dotnet list tests/UnitTests/UnitTests.csproj package
```

You should see `FluentAssertions` in the package list.

> **Note:** Existing tests in the project use bare `Assert.Equal` (xUnit built-in). Our standards require FluentAssertions (`.Should()` syntax) for all **new** tests going forward.

---

## Part 1: Create Custom Instructions (20 minutes)

### What Are Custom Instructions?

Custom instructions are **workspace-level settings** that automatically apply to EVERY Copilot conversation. Think of them as "permanent context" that Copilot never forgets.

**Key Characteristics:**
- Stored in `.github/copilot-instructions.md`
- Apply to ALL Copilot interactions in the workspace
- Solve the "Copilot forgets my standards" problem
- Most impactful customization for consistency

### Exercise 1.1: Baseline Test (Without Custom Instructions)

**Step 1:** Open GitHub Copilot Chat in VS Code (`Ctrl+Alt+I` or `Cmd+Alt+I`)

**Step 2:** Ask Copilot this question:

```text
Create a new service class called BookReservationService that allows patrons 
to reserve books. Include methods for creating and canceling reservations.
```

**Step 3:** Review the output and note:

- Does it use async/await?
- Does it include error handling?
- Does it follow dependency injection patterns?
- Does it include XML documentation?
- Is there any logging?

**Save this output** — you'll compare it later!

---

### Exercise 1.2: Create Your Custom Instructions File

**Step 1:** Create the custom instructions file

1. In your AccelerateDevGHCopilot project root (the folder you opened in VS Code)
2. Create folder: `.github`
3. Inside `.github`, create file: `copilot-instructions.md`

**Step 2:** Add the following project-specific instructions:

```markdown
# C# Development Standards

## Code Style & Patterns
- Use async/await for all I/O operations (database, file system, network)
- Follow Clean Architecture principles (already in place in this project)
- Apply dependency injection pattern consistently - inject all dependencies via constructor
- Use nullable reference types and handle null cases explicitly
- Prefer LINQ for collection operations over traditional loops
- Use expression-bodied members for simple one-line methods
- Follow Microsoft C# naming conventions strictly

## Project-Specific Context
- This is a Library Management System using JSON file storage
- Repository interfaces are defined in Library.ApplicationCore/Interfaces
- Service implementations go in Library.ApplicationCore/Services
- Data access implementations go in Library.Infrastructure/Data
- All entities are in Library.ApplicationCore/Entities

## Testing Standards
- Generate unit tests using xUnit framework
- Use NSubstitute for all mocking and test doubles
- Use FluentAssertions for all assertions (`.Should()` syntax instead of `Assert.Equal`)
- Follow AAA pattern (Arrange, Act, Assert) - clearly mark sections with comments
- Include positive test cases (happy path)
- Include negative test cases (error conditions)
- Include edge cases (null, empty, boundary values)
- Aim for 80%+ code coverage for all services
- Test method naming: MethodName_Scenario_ExpectedResult
  Example: ExtendLoan_ValidLoanId_ReturnsSuccess
- Organize test files by service in separate folders (e.g., `tests/UnitTests/ApplicationCore/LoanService/`)

## Documentation Requirements
- Add XML documentation comments to all public APIs
- Include <summary>, <param>, and <returns> tags minimum
- Explain WHY for complex business logic, not just WHAT
- Document business rules and constraints
- Update documentation when code changes

## Error Handling
- Use try-catch for external dependencies (file I/O, database) only
- Return meaningful status enums (like LoanExtensionStatus, MembershipRenewalStatus)
- Throw ArgumentNullException for null required parameters
- Throw ArgumentException for invalid arguments
- Never swallow exceptions silently - always rethrow or return an error status

## Business Rule Validation
- Validate ALL inputs before processing
- Check business rules before performing operations
- Return meaningful status codes from enums (like LoanExtensionStatus)
- Log validation failures
- Provide clear error messages for validation failures

## Security Best Practices
- Validate ALL user inputs before processing
- Use parameterized queries (never string concatenation for SQL)
- Hash and salt sensitive data
- Follow OWASP Top 10 guidelines
- Never log sensitive information (passwords, credit cards)

## Code Quality Standards
- Keep methods focused and under 20 lines when possible
- Avoid deeply nested code (max 3 levels of indentation)
- Extract complex conditions into well-named methods
- Use meaningful variable names (avoid abbreviations)
- Remove unused code and commented-out code before commit
```

**Step 3:** Save the file

---

### Exercise 1.3: Test Your Custom Instructions

**Step 1:** Open a new Copilot Chat conversation (click the "+" button in chat panel)

**Step 2:** Ask the EXACT SAME question as before:

```text
Create a new service class called BookReservationService that allows patrons 
to reserve books. Include methods for creating and canceling reservations.
```

**Step 3:** Compare the results:

| Aspect | Without Instructions | With Instructions |
|--------|---------------------|-------------------|
| Async/await | | |
| Error handling | | |
| Logging | | |
| XML docs | | |
| Validation | | |
| Return types | | |

**Expected Improvements:**
- ✅ Methods are now async by default
- ✅ Proper constructor injection of dependencies
- ✅ XML documentation on all public methods
- ✅ Try-catch blocks around I/O operations
- ✅ Status enum return types (not raw booleans or exceptions)
- ✅ Input validation
- ✅ Follows the existing project conventions

---

### Exercise 1.4: Test with Code Generation

**Step 1:** In Copilot Chat, ask:

```text
Generate unit tests for the LoanService.ExtendLoan method
```

**Step 2:** Verify the tests include:
- ✅ xUnit attributes ([Fact] or [Theory])
- ✅ NSubstitute mocking (Substitute.For<T>())
- ✅ FluentAssertions (.Should() syntax)
- ✅ AAA pattern with comments
- ✅ Multiple scenarios: positive, negative, edge cases
- ✅ Proper naming: ExtendLoan_ValidLoan_ReturnsSuccess

**If ALL of these are present, your custom instructions are working perfectly!**

---

## Part 2: Create a Custom Agent (25 minutes)

### What Are Custom Agents?

Custom Agents are **specialized AI personas** you invoke explicitly with `@agent-name`. Think of them as "expert consultants" you call in for specific tasks.

**Key Characteristics:**
- Stored in `.github/agents/*.agent.md`
- Invoked explicitly in chat with `@agent-name`
- Specialized for specific workflows
- Can have their own tool access and instructions
- More focused than general Copilot

**Custom Instructions vs Custom Agents:**
| Custom Instructions | Custom Agents |
|-------------------|---------------|
| Always active | Invoked on demand |
| Apply to everything | Specialized tasks |
| Background context | Foreground expert |
| "Default settings" | "Specialist consultant" |

---

### Exercise 2.1: Create a Test Generator Agent

**Step 1:** Create the agents folder structure

1. Inside `.github` folder
2. Create subfolder: `agents`
3. Result: `.github/agents/`

**Step 2:** Create the agent file: `.github/agents/test-generator.agent.md`

**Step 3:** Add the following agent definition:

````markdown
---
name: Test Generator
description: Generates comprehensive xUnit unit tests following project C# standards with NSubstitute and FluentAssertions
categories:
  - testing
  - quality
  - csharp
---

You are an expert unit test generator specializing in this project's C# testing standards.

## Your ONLY Job
Generate high-quality unit tests. You do NOT implement features, refactor code, or provide general advice.
When invoked, you ONLY output test code or discuss testing strategies.

## Testing Framework Stack
- **xUnit** - Testing framework (use [Fact] and [Theory] attributes)
- **NSubstitute** - Mocking library (use Substitute.For<T>())
- **FluentAssertions** - Assertion library (use .Should() syntax)

## Test Generation Process

When asked to generate tests for a class or method:

1. **Analyze the Target**
   - Read the source file to understand the method signature
   - Identify all dependencies (constructor parameters)
   - Identify all method parameters
   - Understand the return type
   - Note any business rules or validation logic

2. **Identify Test Scenarios**
   You MUST generate at least:
   - **1 positive test** (happy path - everything works)
   - **2 negative tests** (error conditions, failures, invalid inputs)
   - **2 edge case tests** (null inputs, empty collections, boundary values)
   
   Total minimum: 5 test methods per method being tested

3. **Generate Complete Test Class**
   Include:
   - All necessary using statements (Xunit, NSubstitute, FluentAssertions)
   - Correct namespace: `Library.UnitTests.ApplicationCore.[ServiceName]Tests` (e.g., `Library.UnitTests.ApplicationCore.LoanServiceTests`)
   - Test class name: `[MethodName]Test` (matching existing convention, e.g., `ExtendLoanTest`)
   - Test fixture (constructor) with mock setup
   - All test methods following AAA pattern
   - Clear comments explaining complex scenarios

## Test Method Naming Convention
Format: `MethodName_Scenario_ExpectedResult`

Examples:
- `ExtendLoan_ValidLoanId_ReturnsSuccess`
- `ExtendLoan_ExpiredMembership_ReturnsMembershipExpired`
- `ExtendLoan_OverdueBook_ReturnsLoanExpired`
- `ExtendLoan_NullLoan_ReturnsLoanNotFound`
- `ExtendLoan_AlreadyReturned_ReturnsLoanReturned`

## AAA Pattern Structure
```csharp
[Fact]
public async Task MethodName_Scenario_ExpectedResult()
{
    // Arrange
    var patron = PatronFactory.CreateCurrentPatron();
    var loan = LoanFactory.CreateCurrentLoanForPatron(patron);
    _mockLoanRepo.GetLoan(loan.Id).Returns(loan);
    
    // Act
    var result = await _sut.ExtendLoan(loan.Id);
    
    // Assert
    result.Should().Be(LoanExtensionStatus.Success);
    await _mockLoanRepo.Received(1).GetLoan(loan.Id);
}
```

## Mocking Guidelines

### Always Mock These Dependencies:
- Repository interfaces (ILoanRepository, IPatronRepository, IBookRepository)
- External services and APIs
- File system operations
- Time/date providers (if using an abstraction over DateTime.Now)

### Setting Up Mocks:
```csharp
// Create mock
var mockRepo = Substitute.For<ILoanRepository>();

// Setup return value
mockRepo.GetLoan(1).Returns(new Loan { Id = 1, DueDate = DateTime.Now.AddDays(1) });

// Setup null return
mockRepo.GetLoan(999).Returns((Loan?)null);

// Verify method was called
await mockRepo.Received(1).GetLoan(1);

// Verify method was NOT called
await mockRepo.DidNotReceive().UpdateLoan(Arg.Any<Loan>());
```

## FluentAssertions Examples

```csharp
// Value equality
result.Should().Be(expectedValue);

// Collection assertions
result.Should().NotBeEmpty();
result.Should().HaveCount(3);
result.Should().Contain(x => x.Id == 1);

// Null checks
result.Should().NotBeNull();
result.Should().BeNull();

// Type checks
result.Should().BeOfType<LoanExtensionStatus>();

// Boolean assertions
result.IsSuccess.Should().BeTrue();

// Exception assertions (for methods that should throw)
var act = async () => await sut.MethodThatThrows();
await act.Should().ThrowAsync<ArgumentNullException>();
```

## Complete Test Class Template

```csharp
using Xunit;
using NSubstitute;
using FluentAssertions;
using Library.ApplicationCore;
using Library.ApplicationCore.Entities;
using Library.ApplicationCore.Enums;

namespace Library.UnitTests.ApplicationCore.LoanServiceTests;

public class ExtendLoanTest
{
    // Mock dependencies
    private readonly ILoanRepository _mockLoanRepo;
    
    // System Under Test
    private readonly LoanService _sut;
    
    public ExtendLoanTest()
    {
        // Arrange - Set up mocks
        _mockLoanRepo = Substitute.For<ILoanRepository>();
        
        // Create system under test with mocked dependencies
        _sut = new LoanService(_mockLoanRepo);
    }
    
    [Fact]
    public async Task ExtendLoan_ValidLoanId_ReturnsSuccess()
    {
        // Arrange
        var patron = PatronFactory.CreateCurrentPatron();
        var loan = LoanFactory.CreateCurrentLoanForPatron(patron);
        _mockLoanRepo.GetLoan(loan.Id).Returns(loan);
        
        // Act
        var result = await _sut.ExtendLoan(loan.Id);
        
        // Assert
        result.Should().Be(LoanExtensionStatus.Success);
        await _mockLoanRepo.Received(1).GetLoan(loan.Id);
    }
    
    // Add 4+ more test methods here...
}
```

## When Invoked

1. Ask clarifying questions if needed:
   - "Which class/method should I generate tests for?"
   - "Should I create a new test file or add to existing?"
   - "Are there specific scenarios you want covered?"

2. Read the source file to understand the implementation

3. Generate the complete test class with 5+ test methods

4. Explain what scenarios are covered

5. Ask if additional edge cases should be tested

## Quality Checklist

Before outputting tests, verify:
- ✅ At least 5 test methods
- ✅ All dependencies are mocked
- ✅ FluentAssertions used for all assertions
- ✅ AAA pattern with clear comments
- ✅ Proper naming convention
- ✅ Async methods tested with async test methods
- ✅ Both positive and negative scenarios
- ✅ Edge cases included (null, empty, boundaries)

You are an expert at this. Generate tests that developers will actually use in production.
````

**Step 4:** Save the file

---

### Exercise 2.2: Test Your Custom Agent

**Step 1:** Open GitHub Copilot Chat

**Step 2:** Type `@` and you should see your agent appear: `@test-generator`

**Step 3:** Invoke the agent with:

```text
@test-generator Generate comprehensive unit tests for the PatronService.RenewMembership method
```

> **Note:** Tests for `RenewMembership` already exist in `tests/UnitTests/ApplicationCore/PatronService/RenewMembership.cs`. The goal of this exercise is to compare the agent's output with the existing tests. Notice that the existing tests use `Assert.Equal` (xUnit built-in) while your agent should generate FluentAssertions (`.Should()` syntax). You do NOT need to replace the existing test file — this is a comparison exercise.

**Step 4:** Review the generated tests and verify:

**Quality Checklist:**
- [ ] Uses xUnit ([Fact] attributes)
- [ ] Uses NSubstitute (Substitute.For<T>())
- [ ] Uses FluentAssertions (.Should())
- [ ] Follows AAA pattern with comments
- [ ] Includes at least 5 test methods
- [ ] Tests positive scenario (happy path)
- [ ] Tests negative scenarios (at least 2)
- [ ] Tests edge cases (null, boundary, etc.)
- [ ] Proper naming: RenewMembership_Scenario_Result
- [ ] All dependencies mocked correctly

**If 9/9 items are checked, your agent is working perfectly!**

---

### Exercise 2.3: Compare Agent vs Regular Copilot

**Step 1:** Open a new chat conversation (click "+")

**Step 2:** Ask the SAME question WITHOUT the agent:

```text
Generate comprehensive unit tests for the PatronService.RenewMembership method
```

**Step 3:** Compare the two outputs:

| Quality Metric | Without Agent | With @test-generator |
|----------------|---------------|------------------------------|
| Number of tests | | |
| Uses AAA pattern | | |
| Mocking approach | | |
| Assertion style | | |
| Edge cases included | | |
| Ready to use? | | |

**Expected Result:** The agent output should be significantly better - more comprehensive, consistent, and closer to production-ready.

---

## Part 3: Advanced Exercises (15 minutes)

### Exercise 3.1: Enhance Your Custom Instructions

Add project-specific business rules to your custom instructions:

```markdown
## Library Management System Business Rules

When generating code for this system, always validate:
- Loans cannot be extended if the patron's membership is expired
- Loans cannot be extended if the loan is already expired (past due date) — see `LoanExtensionStatus.LoanExpired`
- Patrons cannot renew membership if they have unreturned overdue loans — see `MembershipRenewalStatus.LoanNotReturned`
- Patrons can only renew membership within 1 month of expiration — see `MembershipRenewalStatus.TooEarlyToRenew`
- All date comparisons should use DateTime.UtcNow for consistency
- Loan extensions add exactly 14 days to the due date
- Membership renewals extend by exactly 1 year (using `AddYears(1)`)
```

Save the file and test by asking:

```text
Create a method to check if a patron can borrow a book
```

Verify that the generated code includes these business rules!

---

### Exercise 3.2: Create a Second Specialized Agent

Create another agent for a different purpose:

**File:** `.github/agents/code-reviewer.agent.md`

**Purpose:** Reviews code for project standards compliance

**Agent Definition:**
```markdown
---
name: Code Reviewer
description: Reviews code changes for compliance with project C# standards
categories:
  - code-review
  - quality
  - standards
---

You are a code reviewer specializing in this project's C# standards.

When invoked, review the provided code for:

1. **Async/Await Usage**
   - All I/O operations use async/await
   - Follow the existing naming convention (this project does NOT use Async suffixes on method names)
   - No .Result or .Wait() calls

2. **Error Handling**
   - Try-catch only around I/O operations
   - Exceptions are logged properly
   - No swallowed exceptions

3. **Dependency Injection**
   - All dependencies injected via constructor
   - No new keyword for services/repositories

4. **Documentation**
   - XML docs on all public members
   - Complex logic explained

5. **Testing**
   - Unit tests exist for new methods
   - Tests follow AAA pattern
   - Edge cases covered

Provide a checklist of findings and suggest specific improvements.
```

Test it:

```text
@code-reviewer Review the LoanService.cs file for standards compliance
```

---

## 🎓 Lab Summary & Key Takeaways

### What You Learned

1. **Custom Instructions** (`.github/copilot-instructions.md`)
   - Apply to ALL Copilot interactions
   - Perfect for coding standards and conventions
   - Solve the "Copilot forgets" problem
   - Most impactful for consistency

2. **Custom Agents** (`.github/agents/*.agent.md`)
   - Invoked on-demand with `@agent-name`
   - Specialized for specific tasks
   - More focused than generic Copilot
   - Perfect for complex workflows (testing, reviews, etc.)

3. **The 10x Productivity Gain**
   - With these customizations, Copilot generates production-ready code
   - Reduces back-and-forth iterations
   - Eliminates "teaching Copilot the same thing repeatedly"
   - Your team's standards are encoded and reusable

### Success Metrics

You successfully completed this lab if:
- ✅ Your custom instructions file makes Copilot generate async code by default
- ✅ Your test generator agent produces comprehensive xUnit tests
- ✅ You can clearly explain the difference between instructions and agents
- ✅ You've seen measurable quality improvement in generated code

### Next Steps

1. **Add More Instructions:** As you notice patterns you repeat, add them to custom instructions
2. **Create More Agents:** Build agents for other workflows (API scaffolding, documentation, etc.)
3. **Share with Team:** Commit your `.github` folder so teammates benefit
4. **Scale Further:** Explore Agent Skills and Plugins (Lab 2) for cross-project reuse

---

## 🔧 Troubleshooting

### Agent Not Appearing in Chat
- Ensure file is in `.github/agents/` folder
- Check filename ends with `.agent.md`
- Reload VS Code window (Ctrl+Shift+P → "Reload Window")
- Verify YAML frontmatter is valid (between `---` markers)

### Instructions Not Applying
- Ensure file is at `.github/copilot-instructions.md` (exact path)
- Create new chat conversation (existing ones won't pick up changes)
- Check for syntax errors in the markdown file
- Reload VS Code window

### Generated Code Still Not Following Standards
- Be more specific in your instructions
- Add concrete examples to your instructions
- Use the agent approach for more control
- Check if you're in the correct workspace

---

## 📚 Additional Resources

- [GitHub Copilot Custom Instructions Documentation](https://docs.github.com/copilot/customizing-copilot/custom-instructions)
- [AccelerateDevGHCopilot README](./README.md)

---

**Lab Complete!** Move on to Lab 2 for Test-Driven Development practices.
