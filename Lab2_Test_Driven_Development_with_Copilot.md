# Lab 2: Test-Driven Development with GitHub Copilot

## 🎯 Lab Objectives

By the end of this lab, you will:
- Master Test-Driven Development (TDD) workflow with GitHub Copilot
- Generate comprehensive unit tests before implementing features
- Use Copilot to write implementation code that passes tests
- Achieve 80%+ code coverage for new features
- Understand Copilot's test generation capabilities (Generate Tests and custom agents)
- Apply TDD to a real feature in the Library Management System

**Time Required:** 60-75 minutes

---

## 📊 Why TDD with Copilot?

### The Traditional TDD Problem
- Writing tests first is time-consuming
- Test boilerplate is repetitive
- Developers skip TDD because it "slows them down"

### TDD with Copilot: The Game Changer
- ✅ Generate comprehensive test suites in seconds
- ✅ Copilot writes implementation to pass your tests
- ✅ TDD becomes FASTER than code-first approach
- ✅ Higher quality, fewer bugs
- ✅ Better design (testable code is better code)

**Result:** TDD with Copilot is 3-5x faster than traditional development

---

## 📋 Prerequisites

- Completed Lab 1 (Custom Instructions and Agents)
- **AccelerateDevGHCopilot folder opened directly as your VS Code workspace** (File > Open Folder > select `AccelerateDevGHCopilot`). This is required so VS Code Copilot picks up the `.github/` customizations you created in Lab 1.
- GitHub Copilot and Copilot Chat enabled
- Custom instructions and test generator agent set up from Lab 1
- FluentAssertions installed (see Lab 1 prerequisites)
- Basic understanding of xUnit testing

---

## Part 1: TDD Fundamentals with Copilot (20 minutes)

### The TDD Cycle (Red-Green-Refactor)

1. **RED:** Write a failing test
2. **GREEN:** Write minimal code to make it pass
3. **REFACTOR:** Improve the code while keeping tests green

With Copilot:
1. **RED:** Generate comprehensive tests with Copilot
2. **GREEN:** Let Copilot implement to pass tests
3. **REFACTOR:** Use Copilot to improve code quality

---

### Exercise 1.1: Simple TDD Example - Adding a Feature

**Scenario:** Add a new feature to track overdue fines for patrons.

**Step 1: Define the Requirements**

Create a new file: `docs/OverdueFines_Requirements.md`

> **Note:** The `docs/` folder doesn't exist yet — create it first.

Add this content:

````markdown
# Overdue Fines Feature Requirements

## Business Rules
1. Fines are calculated at $0.50 per day per overdue book
2. Maximum fine per book is $10.00 (20 days)
3. Patrons cannot borrow new books if they have fines over $5.00
4. Patrons can pay fines in any amount
5. Fines are calculated based on days overdue (DueDate < DateTime.UtcNow)

## Method Signatures Needed

```csharp
public class PatronService
{
    // Calculate total fines for all overdue books
    decimal CalculateTotalFines(int patronId);
    
    // Check if patron can borrow (fines under $5)
    bool CanPatronBorrow(int patronId);
    
    // Record a fine payment
    Task<FinePaymentStatus> PayFine(int patronId, decimal amount);
}
```

## Test Scenarios Required

### CalculateTotalFines
- Patron with no overdue books → $0.00
- Patron with 1 book, 3 days overdue → $1.50
- Patron with 1 book, 25 days overdue → $10.00 (capped)
- Patron with 3 books overdue → sum of all fines
- Invalid patron ID → throws ArgumentException

### CanPatronBorrow
- No fines → true
- Fines under $5.00 → true
- Fines exactly $5.00 → false
- Fines over $5.00 → false
- Patron not found → false

### PayFine
- Valid payment reducing fines → Success
- Payment amount greater than total fines → Success (zero balance)
- Zero or negative payment amount → InvalidAmount
- Patron not found → PatronNotFound
````

**Save the file.**

---

### Exercise 1.2: Generate Tests FIRST (The "Red" Phase)

**Step 1:** Create the test file

Follow the existing project convention of organizing tests by service folder.
Create the file: `tests/UnitTests/ApplicationCore/PatronService/FinesTests.cs`

**Step 2:** Open Copilot Chat and use your custom agent:

> **Important:** When referencing files in Copilot Chat prompts, use the `#file:` prefix (e.g., `#file:docs/OverdueFines_Requirements.md`). This tells Copilot to actually read the file contents. Without `#file:`, Copilot may not find or read the file you're referencing.

```text
@test-generator I need to implement a new overdue fines feature. 
Read the requirements from #file:docs/OverdueFines_Requirements.md and generate 
comprehensive unit tests for all three methods: CalculateTotalFines, 
CanPatronBorrow, and PayFine.

The tests should be in a new test class called FinesTests inside the
Library.UnitTests.ApplicationCore.PatronServiceTests namespace.
Include all scenarios listed in the requirements.
```

**Step 3:** Review the generated tests

Verify the test class includes:
- [ ] Test fixture with mocked dependencies (IPatronRepository — the sole dependency of PatronService)
- [ ] Tests for CalculateTotalFines (5 scenarios)
- [ ] Tests for CanPatronBorrow (5 scenarios)
- [ ] Tests for PayFine (4 scenarios)
- [ ] All tests follow AAA pattern
- [ ] Proper naming convention
- [ ] FluentAssertions used

**Step 4:** Copy the generated tests to the test file and save

**Step 5:** Run the tests - they should ALL FAIL (compile errors)

```bash
dotnet test --filter "FullyQualifiedName~FinesTests"
```

**Expected Result:** Tests don't compile because the methods don't exist yet. This is perfect — we're in the RED phase!

---

### Exercise 1.3: Implement to Pass Tests (The "Green" Phase)

**Step 1:** Open the source file: `src/Library.ApplicationCore/Services/PatronService.cs`

**Step 2:** Use Copilot to generate implementation

In Copilot Chat:

```text
I have written comprehensive tests for three new methods in PatronService:
1. CalculateTotalFines
2. CanPatronBorrow
3. PayFine

Review the tests in #file:tests/UnitTests/ApplicationCore/PatronService/FinesTests.cs
and implement these three methods in #file:src/Library.ApplicationCore/Services/PatronService.cs to make all tests pass.

Follow the business rules from #file:docs/OverdueFines_Requirements.md.
```

**Step 3:** Review the generated implementation

Verify it includes:
- [ ] All three methods with correct signatures
- [ ] Async methods where appropriate
- [ ] Proper error handling
- [ ] Business logic matching requirements
- [ ] XML documentation comments

> **Note:** The current `PatronService` only depends on `IPatronRepository`. If your fines implementation needs access to loans (e.g., to check overdue books), note that `Patron.Loans` is a navigation property — the patron entity already includes its loan collection. You may need to adjust the implementation to work with this existing pattern rather than injecting `ILoanRepository` separately.

**Step 4:** Add the generated methods to PatronService.cs

**Step 5:** Create the FinePaymentStatus enum if needed

Create the file `src/Library.ApplicationCore/Enums/FinePaymentStatus.cs`:

```csharp
namespace Library.ApplicationCore.Enums;

public enum FinePaymentStatus
{
    Success,
    InvalidAmount,
    PatronNotFound
}
```

> **Note:** You also need to add the new methods (`CalculateTotalFines`, `CanPatronBorrow`, `PayFine`) to the `IPatronService` interface in `src/Library.ApplicationCore/Interfaces/IPatronService.cs` so the project compiles correctly.

**Step 6:** Run the tests again

```bash
dotnet test --filter "FullyQualifiedName~FinesTests"
```

**Expected Result:** All tests should now PASS!

---

### Exercise 1.4: Refactor with Copilot (The "Refactor" Phase)

**Step 1:** Review the implementation for code quality

In Copilot Chat (if you created the code-reviewer agent in Lab 1):

```text
@code-reviewer Review the PatronService fines methods 
(CalculateTotalFines, CanPatronBorrow, PayFine) for code quality 
and adherence to project standards.
```

OR without the agent:

```text
Review the following methods in PatronService for code quality:
- CalculateTotalFines
- CanPatronBorrow  
- PayFine

Check for:
- Code duplication
- Complex conditional logic that could be simplified
- Magic numbers that should be constants
- Opportunities to extract private helper methods
- Adherence to SOLID principles
```

**Step 2:** Apply suggested refactorings

Example improvements Copilot might suggest:
```csharp
// Extract magic numbers to constants
private const decimal FINE_PER_DAY = 0.50m;
private const decimal MAX_FINE_PER_BOOK = 10.00m;
private const decimal BORROW_THRESHOLD = 5.00m;

// Extract fine calculation logic
private decimal CalculateFineForLoan(Loan loan)
{
    if (loan.DueDate >= DateTime.UtcNow)
        return 0m;
        
    var daysOverdue = (DateTime.UtcNow - loan.DueDate).Days;
    var fine = daysOverdue * FINE_PER_DAY;
    return Math.Min(fine, MAX_FINE_PER_BOOK);
}
```

**Step 3:** Run tests after each refactoring

```bash
dotnet test --filter "FullyQualifiedName~FinesTests"
```

**Tests should still pass!** If they don't, revert and try again.

---

## Part 2: Advanced TDD Patterns (25 minutes)

### Exercise 2.1: Data-Driven Tests with Theory

**Scenario:** You need to test fine calculations with many different scenarios.

**Step 1:** Ask Copilot to convert tests to Theory-based tests:

```text
Convert the CalculateTotalFines tests to use [Theory] and [InlineData] 
for these scenarios:

- 0 days overdue → $0.00
- 1 day overdue → $0.50
- 5 days overdue → $2.50
- 10 days overdue → $5.00
- 20 days overdue → $10.00 (max)
- 30 days overdue → $10.00 (still max)

Use a single test method with InlineData for all scenarios.
```

**Step 2:** Review the generated Theory test:

```csharp
[Theory]
[InlineData(0, 0.00)]
[InlineData(1, 0.50)]
[InlineData(5, 2.50)]
[InlineData(10, 5.00)]
[InlineData(20, 10.00)]
[InlineData(30, 10.00)]
public async Task CalculateTotalFines_VariousDaysOverdue_ReturnsCorrectFine(
    int daysOverdue, decimal expectedFine)
{
    // Arrange
    var patron = PatronFactory.CreateCurrentPatron();
    var loan = new Loan
    {
        Id = 1,
        DueDate = DateTime.UtcNow.AddDays(-daysOverdue),
        BookItemId = 1,
        PatronId = patron.Id,
        Patron = patron
    };
    patron.Loans.Add(loan);
    _mockPatronRepo.GetPatron(patron.Id).Returns(patron);

    // Act
    var result = await _sut.CalculateTotalFines(patron.Id);

    // Assert
    result.Should().Be(expectedFine);
}
```

**Step 3:** Add to your test class and run tests

This single test method now tests 6 scenarios!

---

### Exercise 2.2: Using Copilot to Generate Tests from Code

GitHub Copilot can generate tests directly from your source code.

**Step 1:** Open `src/Library.ApplicationCore/Services/PatronService.cs` and select the `RenewMembership` method

**Step 2:** Right-click the selected code and choose **Copilot > Generate Tests**, or open Copilot Chat and type:

```text
Generate unit tests for the selected code
```

> **Tip:** If `Generate Tests` doesn't appear in the context menu, you can simply ask Copilot Chat: "Generate tests for PatronService.RenewMembership" with the file open.

**Step 3:** Review the generated tests

Copilot analyzes the selected code and generates tests automatically.

**Step 4:** Compare with your custom agent output

Open a new chat and run:

```text
@test-generator Generate tests for PatronService.RenewMembership
```

**Comparison:**
| Aspect | Generate Tests (built-in) | @test-generator |
|--------|----------------|------------------------|
| Speed | Faster | Slightly slower |
| Customization | Generic | Project-specific |
| Completeness | Basic scenarios | Comprehensive |
| Standards compliance | Mixed | 100% your team's style |

**Takeaway:** Use the built-in Generate Tests for quick test generation, use your custom agent for production-quality tests that follow your team's standards.

---

### Exercise 2.3: Test Coverage Analysis

**Step 1:** Run tests with coverage

```bash
dotnet test --collect:"XPlat Code Coverage"
```

> **Note:** The project uses `coverlet.collector`, so you must use `--collect:"XPlat Code Coverage"` (not `/p:CollectCoverage=true`, which requires the `coverlet.msbuild` package).

**Step 2:** Generate coverage report

```bash
# Install report generator if you haven't
dotnet tool install -g dotnet-reportgenerator-globaltool

# Generate HTML report (the coverage file is under TestResults/)
reportgenerator -reports:"**/coverage.cobertura.xml" -targetdir:coveragereport -reporttypes:Html
```

**Step 3:** Open the report

```bash
start coveragereport/index.html    # Windows
open coveragereport/index.html     # Mac/Linux
```

**Step 4:** Identify uncovered code

Find methods with <80% coverage.

**Step 5:** Generate missing tests with Copilot

For each uncovered method:

```text
@test-generator Generate tests for [MethodName] to achieve 
100% code coverage. Focus on the uncovered branches shown in the 
coverage report.
```

**Step 6:** Add tests and re-run coverage

Repeat until you achieve 80%+ coverage target.

---

## Part 3: Real-World Feature Development (30 minutes)

### Exercise 3.1: Complete Feature - Book Hold/Reservation System

**Scenario:** Implement a complete new feature using TDD.

**Requirements:**
- Patrons can place holds on books that are currently checked out
- When a book is returned, patron with the oldest hold is notified
- Patrons can have maximum 5 active holds
- Holds expire after 48 hours if not picked up
- Patrons with overdue books cannot place holds

---

**Step 1: Write Requirements Document**

Create the file: `docs/BookHolds_Requirements.md` (the `docs/` folder should already exist from Exercise 1.1)

````markdown
# Book Hold/Reservation System Requirements

## Business Rules
1. Patrons can place holds on books that are currently checked out
2. Maximum 5 active holds per patron
3. Holds are queued FIFO (first in, first out)
4. When book is returned, notify patron with oldest hold
5. Holds expire 48 hours after patron is notified
6. Patrons with overdue loans cannot place holds
7. Hold status: Pending, Ready, Expired, Completed, Cancelled

## Entities

### BookHold
```csharp
public class BookHold
{
    public int Id { get; set; }
    public int PatronId { get; set; }
    public int BookItemId { get; set; }
    public DateTime PlacedDate { get; set; }
    public DateTime? NotifiedDate { get; set; }
    public DateTime? ExpirationDate { get; set; }
    public HoldStatus Status { get; set; }
}

public enum HoldStatus
{
    Pending,      // Waiting for book to be available
    Ready,        // Book available, patron notified
    Expired,      // 48 hours passed after notification
    Completed,    // Patron picked up the book
    Cancelled     // Hold was cancelled
}
```

## Service Methods

### BookHoldService
```csharp
public class BookHoldService : IBookHoldService
{
    Task<PlaceHoldResult> PlaceHold(int patronId, int bookItemId);
    Task<GetHoldsResult> GetPatronHolds(int patronId);
    Task<CancelHoldResult> CancelHold(int holdId);
    Task ProcessReturnedBook(int bookItemId);  // Notify next patron
    Task ExpireOldHolds();  // Background job - expire holds after 48h
}
```

## Test Scenarios

### PlaceHold Tests
1. Valid hold placement → Success
2. Patron has 5 holds already → MaxHoldsReached
3. Patron has overdue loans → PatronIneligible
4. Book item doesn't exist → BookItemNotFound
5. Patron already has hold on this book → DuplicateHold
6. Book is available (not checked out) → BookAvailable (should borrow instead)

### GetPatronHolds Tests
1. Patron with multiple holds → Returns all holds
2. Patron with no holds → Returns empty list
3. Invalid patron ID → Returns empty list

### CancelHold Tests
1. Valid cancellation of pending hold → Success
2. Valid cancellation of ready hold → Success
3. Cannot cancel completed hold → InvalidStatus
4. Cannot cancel expired hold → InvalidStatus
5. Hold doesn't exist → NotFound

### ProcessReturnedBook Tests
1. Book returned, has pending holds → Notifies first patron, sets Ready
2. Book returned, no holds → No action
3. Multiple holds → Only first is notified
4. Sets NotifiedDate and ExpirationDate (48h later)

### ExpireOldHolds Tests
1. Hold ready for 49 hours → Expires, notifies next patron
2. Hold ready for 24 hours → No change
3. Multiple expired holds → All expired, next patrons notified
````

---

**Step 2: Create Required Interfaces, Entities, and Enums**

Before generating tests, you need stub files so the test code can compile. None of these types exist yet — this is intentional. In real TDD you often start before the interfaces exist.

Create the following files with empty/stub content. You can ask Copilot to generate them from the requirements document:

```text
Based on docs/BookHolds_Requirements.md, create the following stub files 
with just the type signatures (no implementation logic):
- src/Library.ApplicationCore/Entities/BookHold.cs
- src/Library.ApplicationCore/Enums/HoldStatus.cs
- src/Library.ApplicationCore/Enums/PlaceHoldResult.cs
- src/Library.ApplicationCore/Enums/CancelHoldResult.cs
- src/Library.ApplicationCore/Enums/GetHoldsResult.cs
- src/Library.ApplicationCore/Interfaces/IBookHoldService.cs
- src/Library.ApplicationCore/Interfaces/IBookHoldRepository.cs
```

> **Tip:** Review the generated stubs before saving. Make sure the method signatures on `IBookHoldService` match the requirements document.

**Step 3: Generate Complete Test Suite**

```text
@test-generator Read the requirements from #file:docs/BookHolds_Requirements.md
and generate a complete test suite for BookHoldService.

Create test class: BookHoldServiceTests
Include all test scenarios listed in the requirements.
Mock all dependencies: IBookHoldRepository, ILoanRepository, IPatronRepository.
```

**Step 4: Create Empty Implementation**

Create the file: `src/Library.ApplicationCore/Services/BookHoldService.cs`

Start with:
```csharp
namespace Library.ApplicationCore.Services;

public class BookHoldService : IBookHoldService
{
    private readonly IBookHoldRepository _bookHoldRepo;
    private readonly ILoanRepository _loanRepo;
    private readonly IPatronRepository _patronRepo;
    
    public BookHoldService(
        IBookHoldRepository bookHoldRepo,
        ILoanRepository loanRepo,
        IPatronRepository patronRepo)
    {
        _bookHoldRepo = bookHoldRepo;
        _loanRepo = loanRepo;
        _patronRepo = patronRepo;
    }
    
    // TODO: Implement methods
}
```

**Step 5: Run Tests - All Should Fail**

```bash
dotnet test --filter "FullyQualifiedName~BookHoldServiceTests"
```

---

**Step 6: Implement Each Method with Copilot**

For each method, use Copilot Chat:

```text
Implement the PlaceHold method in BookHoldService to pass all tests 
in BookHoldServiceTests.PlaceHold_* tests.

Follow the business rules from #file:docs/BookHolds_Requirements.md.
Use async/await and proper error handling (matching the existing service patterns).
```

Repeat for each method:
- GetPatronHolds
- CancelHold
- ProcessReturnedBook
- ExpireOldHolds

**Step 7: Run Tests After Each Implementation**

```bash
dotnet test --filter "FullyQualifiedName~BookHoldServiceTests.PlaceHold"
dotnet test --filter "FullyQualifiedName~BookHoldServiceTests.GetPatronHolds"
# etc.
```

**Step 8: Verify All Tests Pass**

```bash
dotnet test --filter "FullyQualifiedName~BookHoldServiceTests"
```

**Expected Result:** 100% of tests passing! ✅

---

**Step 9: Check Code Coverage**

```bash
dotnet test --filter "FullyQualifiedName~BookHoldServiceTests" --collect:"XPlat Code Coverage"
```

**Target:** 80%+ coverage for BookHoldService

---

**Step 10: Refactor with Copilot**

If you created the `code-reviewer` agent in Lab 1 Exercise 3.2:

```text
@code-reviewer Review the BookHoldService implementation for:
1. Code duplication across methods
2. Complex conditional logic
3. Opportunities to extract private helper methods
4. SOLID principle adherence
5. Error handling completeness

Suggest specific refactorings.
```

Or without the agent, just ask Copilot Chat directly with the same review request.

Apply refactorings and ensure tests still pass!

---

## Part 4: TDD Best Practices with Copilot (10 minutes)

### Exercise 4.1: Test Maintenance

**Scenario:** Requirements change - fine rate increases to $1.00 per day.

**Step 1:** Update requirements document

```markdown
File: docs/OverdueFines_Requirements.md

Change: Fines are calculated at $1.00 per day per book (was $0.50)
```

**Step 2:** Ask Copilot to update tests

```text
The fine rate has changed from $0.50 to $1.00 per day. 
Update all tests in `FinesTests` (in the `PatronServiceTests` namespace) to reflect the new rate.
Update the expected values in InlineData attributes.
```

**Step 3:** Update the implementation

```text
Update the FINE_PER_DAY constant in PatronService from $0.50 to $1.00
```

**Step 4:** Run tests - all should still pass

This demonstrates how TDD protects you during requirement changes!

---

### Exercise 4.2: Testing Edge Cases

**Prompt for Copilot:**

```text
Generate additional edge case tests for BookHoldService.PlaceHold:
- Patron ID is negative
- Patron ID is zero
- BookItem ID is negative
- Place hold with DateTime.MaxValue
- Place hold with DateTime.MinValue
- Concurrent holds on same book
- Database connection failure scenarios
- Repository returns null unexpectedly
```

Add these edge cases to make your code more robust!

---

### Exercise 4.3: Regression Test Creation

**Scenario:** A bug was found in production - expired holds weren't being cleaned up.

**Step 1:** Create a regression test first

```text
@test-generator Create a regression test for the following bug:

Bug: Expired holds (Ready status for >48 hours) are not being expired 
when ExpireOldHolds() is called. The NotifiedDate is compared to UtcNow 
but the calculation is off by one hour due to timezone issues.

Create a test that reproduces this bug, named:
ExpireOldHolds_HoldExpired49Hours_ShouldExpire

The test should set up a hold with NotifiedDate = UtcNow.AddHours(-49)
and verify it gets expired.
```

**Step 2:** Run the test - it should FAIL (reproducing the bug)

**Step 3:** Fix the bug

```text
Fix the timezone issue in BookHoldService.ExpireOldHolds() to make the 
regression test pass. The comparison should use UtcNow consistently.
```

**Step 4:** Run test - now it should PASS

**Step 5:** Commit both test and fix

This test now prevents the bug from happening again!

---

## 🎓 Lab Summary & Key Takeaways

### What You Learned

1. **TDD Workflow with Copilot**
   - Generate comprehensive tests FIRST
   - Implement to pass tests
   - Refactor with confidence (tests protect you)
   - TDD with Copilot is FASTER than code-first

2. **Test Generation Techniques**
   - Custom agents for project-specific tests
   - Built-in Generate Tests for quick generation
   - Theory tests with InlineData for multiple scenarios
   - Edge case identification

3. **Test-Driven Feature Development**
   - Write requirements first
   - Generate test suite from requirements
   - Implement incrementally
   - Achieve high code coverage naturally

4. **Test Maintenance**
   - Update tests when requirements change
   - Create regression tests for bugs
   - Use Copilot to maintain test suites

### Success Metrics

You successfully completed this lab if:
- ✅ Generated test suites before implementing code
- ✅ All tests pass after implementation
- ✅ Achieved 80%+ code coverage for new features
- ✅ Can use both Generate Tests and custom agents effectively
- ✅ Understand the TDD cycle (Red-Green-Refactor)
- ✅ Completed the book holds feature end-to-end

### Time Savings

**Traditional TDD (Manual):**
- Write requirements: 30 min
- Write 20 test methods: 2-3 hours
- Implement feature: 1-2 hours
- Debug and fix tests: 1 hour
- **Total: 4.5-6.5 hours**

**TDD with Copilot:**
- Write requirements: 30 min
- Generate 20 test methods: 5 min
- Implement feature with Copilot: 30 min
- Debug and refine: 30 min
- **Total: 1.5 hours** 

**Result: 67-77% time savings!**

---

## 🔧 Troubleshooting

### Tests Not Compiling
- Ensure all entities and enums are created
- Check using statements in test files
- Verify interface definitions exist
- Add missing NuGet packages (xUnit, NSubstitute, FluentAssertions)

### Tests Failing Unexpectedly
- Check mock setup - are you returning the right data?
- Verify expected values match business rules
- Use debugger to step through test
- Ask Copilot: "Why is this test failing?" with test code

### Implementation Not Passing Tests
- Re-read the requirements
- Check business logic carefully
- Ask Copilot: "Review this implementation against the test expectations"
- Verify async/await is used correctly

### Coverage Too Low
- Identify uncovered branches in coverage report
- Generate tests specifically for uncovered code
- Look for edge cases and error paths
- Use Theory tests to cover multiple inputs

---

## 📚 Additional Resources

- [xUnit Documentation](https://xunit.net/)
- [NSubstitute Documentation](https://nsubstitute.github.io/)
- [FluentAssertions Documentation](https://fluentassertions.com/)
- [GitHub Copilot Testing Guide](https://docs.github.com/copilot/using-github-copilot/code-review-and-testing)
- [Project Testing Standards](./README.md)
- [TDD Best Practices (Martin Fowler)](https://martinfowler.com/bliki/TestDrivenDevelopment.html)

---

## 🚀 Next Steps

1. **Apply TDD to Your Daily Work**
   - Start every new feature with tests
   - Use your test generator agent
   - Achieve 80%+ coverage on all new code

2. **Expand Your Test Suite**
   - Add integration tests
   - Add end-to-end tests
   - Test boundary conditions

3. **Share with Team**
   - Commit your test patterns
   - Share your custom agent
   - Run coverage reports in CI/CD

4. **Continue to Lab 3** (if available)
   - Advanced Copilot features
   - Code quality and refactoring
   - Debugging with Copilot

---

**Congratulations!** You've mastered TDD with GitHub Copilot. You're now equipped to write high-quality, well-tested code faster than ever before. 🎉
