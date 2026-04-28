# Lab 3: Code Quality, Refactoring & Debugging with GitHub Copilot

## 🎯 Lab Objectives

By the end of this lab, you will:
- Use Copilot's `/fix` command to debug and resolve issues
- Use Copilot's **Generate Tests** feature to generate comprehensive test coverage
- Apply Copilot's refactoring suggestions to improve code quality
- Use code profiling insights to optimize performance
- Implement proper error handling and logging patterns
- Use `.copilotignore` to protect sensitive files
- Master Copilot Chat slash commands for productivity

**Time Required:** 50-60 minutes

---

## 📊 The Code Quality Challenge

### Mercury Survey Insights
- **89% dissatisfied** with code output quality
- **Common issues:** Code duplication, poor error handling, lack of documentation
- **Root cause:** Not using Copilot's quality improvement features

### Copilot's Quality Tools
- ✅ `/fix` - Automatically debug and fix errors
- ✅ **Generate Tests** (right-click > Copilot) - Generate comprehensive test coverage
- ✅ Refactoring suggestions - Improve code structure
- ✅ Code explanations - Understand complex logic
- ✅ Security scanning - Identify vulnerabilities

---

## 📋 Prerequisites

- Completed Lab 1 and Lab 2
- AccelerateDevGHCopilot project open in VS Code
- GitHub Copilot and Copilot Chat enabled
- Project builds successfully: `dotnet build`

---

## Part 1: Debugging with `/fix` Command (15 minutes)

### Exercise 1.1: Introducing Intentional Bugs

**Step 1:** Create a buggy version of LoanService

Open `src/Library.ApplicationCore/Services/LoanService.cs` and temporarily add these bugs:

```csharp
// Find the ExtendLoan method and introduce these bugs:

public async Task<LoanExtensionStatus> ExtendLoan(int loanId)
{
    // BUG 1: Not checking for null loan
    Loan loan = await _loanRepository.GetLoan(loanId);
    
    // BUG 2: Wrong date comparison (should be <, not >)
    if (loan.DueDate > DateTime.Now)
    {
        return LoanExtensionStatus.LoanNotFound;
    }
    
    // BUG 3: Not checking membership expiration
    // (Missing the loan.Patron!.MembershipEnd check entirely)
    
    // BUG 4: Wrong number of days (should be 14, not 7)
    loan.DueDate = loan.DueDate.AddDays(7);
    
    // BUG 5: Not saving the updated loan
    // await _loanRepository.UpdateLoan(loan);  // This line is commented out
    
    return LoanExtensionStatus.Success;
}
```

**Step 2:** Run the existing tests

```bash
dotnet test --filter "FullyQualifiedName~LoanServiceTests"
```

**Expected Result:** Tests should fail! 🔴

---

### Exercise 1.2: Using `/fix` to Debug

**Step 1:** Open Copilot Chat and select the buggy `ExtendLoan` method

**Step 2:** In Copilot Chat, type:
```
/fix
```

**Step 3:** Review Copilot's analysis

Copilot should identify:
- Null reference risk (loan could be null)
- Logic error in date comparison
- Missing membership expiration check
- Wrong extension period
- Missing database save operation

**Step 4:** Review the suggested fix

Copilot will provide corrected code. Example:

```csharp
public async Task<LoanExtensionStatus> ExtendLoan(int loanId)
{
    // Fix: Check for null
    Loan? loan = await _loanRepository.GetLoan(loanId);
    if (loan == null)
        return LoanExtensionStatus.LoanNotFound;
    
    // Fix: Check membership expiration via the Patron navigation property
    if (loan.Patron!.MembershipEnd < DateTime.Now)
        return LoanExtensionStatus.MembershipExpired;
    
    // Fix: Check if loan is already returned
    if (loan.ReturnDate != null)
        return LoanExtensionStatus.LoanReturned;
    
    // Fix: Correct date comparison — check if loan has expired (overdue)
    if (loan.DueDate < DateTime.Now)
        return LoanExtensionStatus.LoanExpired;
    
    // Fix: Correct extension period (14 days, not 7)
    loan.DueDate = loan.DueDate.AddDays(LoanService.ExtendByDays);
    
    // Fix: Save the changes
    try
    {
        await _loanRepository.UpdateLoan(loan);
        return LoanExtensionStatus.Success;
    }
    catch (Exception e)
    {
        return LoanExtensionStatus.Error;
    }
}
```

**Step 5:** Apply the fix and run tests again

```bash
dotnet test --filter "FullyQualifiedName~ExtendLoanTest"
```

**Expected Result:** Tests should now pass! ✅

---

### Exercise 1.3: Fix Compilation Errors

**Step 1:** Introduce a compilation error

```csharp
// In PatronService, change this line:
var patron = await _patronRepository.GetPatron(patronId);

// To this (wrong method name):
var patron = await _patronRepository.FetchPatron(patronId);
```

**Step 2:** Notice the red squiggly line

**Step 3:** Click on the error in the Problems panel

**Step 4:** Use `/fix` with the error message

```
/fix Method 'FetchPatron' does not exist in IPatronRepository
```

**Step 5:** Copilot will suggest the correction

```csharp
// Should be:
var patron = await _patronRepository.GetPatron(patronId);
```

---

### Exercise 1.4: Fix Runtime Errors

**Step 1:** Create a method with potential runtime errors

```csharp
public decimal CalculateLateFee(Loan loan)
{
    // Potential issues: null reference, negative days
    var daysOverdue = (DateTime.Now - loan.DueDate).Days;
    var baseFee = daysOverdue * 0.50m;
    var bookTitle = loan.BookItem.Book.Title.ToUpper();
    return baseFee;
}
```

**Step 2:** Select the method and use `/fix`

```
/fix Identify potential runtime errors and edge cases
```

**Step 3:** Review Copilot's suggestions

Copilot should identify:
- Null reference if `loan` is null
- Null reference if `loan.BookItem` is null
- Null reference if `loan.BookItem.Book` is null
- Negative days if loan not overdue
- Possible null `Title` causing NullReferenceException

**Step 4:** Apply the defensive programming fix

```csharp
public decimal CalculateLateFee(Loan loan)
{
    if (loan == null)
        throw new ArgumentNullException(nameof(loan));
    
    var daysOverdue = (DateTime.Now - loan.DueDate).Days;
    if (daysOverdue <= 0)
    {
        return 0m; // Not overdue
    }
    
    var baseFee = daysOverdue * 0.50m;
    
    // Safely access navigation properties
    var bookTitle = loan.BookItem?.Book?.Title ?? "Unknown";
    
    return baseFee;
}
```

---

## Part 2: Code Refactoring for Quality (20 minutes)

### Exercise 2.1: Identify Code Smells

**Step 1:** Find the `CommonActions.cs` file in `src/Library.Console/`

**Step 2:** Look for the `RenewPatronMembership` method - it's likely long and complex

**Step 3:** Select the entire method and ask Copilot:

```
Analyze this method for code smells and suggest refactorings:
- Long method
- Code duplication
- Complex conditionals
- Magic numbers
- Poor variable names
- Violation of Single Responsibility Principle
```

---

### Exercise 2.2: Extract Method Refactoring

**Scenario:** Long methods should be broken into smaller, focused methods.

**Step 1:** Find a long method (look for methods >30 lines)

**Step 2:** Ask Copilot to refactor:

```
Refactor this method by extracting logical sections into private helper methods.
Each helper method should have a single clear purpose.
Use descriptive method names that explain intent.
```

**Example transformation:**

**Before:**
```csharp
public async Task<LoanReturnStatus> ReturnLoan(int loanId)
{
    var loan = await _loanRepository.GetLoan(loanId);
    if (loan == null) return LoanReturnStatus.LoanNotFound;
    
    if (loan.ReturnDate != null) return LoanReturnStatus.AlreadyReturned;
    
    loan.ReturnDate = DateTime.Now;
    
    // Calculate overdue fine inline
    var daysOverdue = (DateTime.Now - loan.DueDate).Days;
    decimal fine = 0;
    if (daysOverdue > 0)
    {
        fine = daysOverdue * 0.50m;
        if (fine > 10.00m) fine = 10.00m;
    }
    
    await _loanRepository.UpdateLoan(loan);
    return LoanReturnStatus.Success;
}
```

**After:**
```csharp
public async Task<LoanReturnStatus> ReturnLoan(int loanId)
{
    var loan = await _loanRepository.GetLoan(loanId);
    if (loan == null) return LoanReturnStatus.LoanNotFound;
    
    if (loan.ReturnDate != null) return LoanReturnStatus.AlreadyReturned;
    
    MarkLoanAsReturned(loan);
    
    try
    {
        await _loanRepository.UpdateLoan(loan);
        return LoanReturnStatus.Success;
    }
    catch (Exception e)
    {
        return LoanReturnStatus.Error;
    }
}

private static void MarkLoanAsReturned(Loan loan)
{
    loan.ReturnDate = DateTime.Now;
}
```

**Step 3:** Run tests to ensure refactoring didn't break anything

```bash
dotnet test
```

---

### Exercise 2.3: Replace Magic Numbers with Constants

**Step 1:** Find magic numbers in the codebase

Ask Copilot:
```
Search the PatronService and LoanService for magic numbers 
(hardcoded numeric values) and suggest named constants.
```

**Step 2:** Review findings

Example magic numbers:
- `14` - Loan extension days
- `365` - Membership renewal days
- `5.00` - Maximum fine for borrowing
- `0.50` - Fine per day

**Step 3:** Create constants class

```csharp
File: src/Library.ApplicationCore/Constants/LibraryConstants.cs

namespace Library.ApplicationCore.Constants;

public static class LibraryConstants
{
    public static class Loans
    {
        public const int ExtensionDays = 14;
        public const int DefaultLoanDays = 21;
        public const decimal FinePerDay = 0.50m;
        public const decimal MaxFinePerBook = 10.00m;
    }
    
    public static class Memberships
    {
        public const int RenewalDays = 365;
        public const int ExpirationWarningDays = 30;
    }
    
    public static class Holds
    {
        public const int MaxHoldsPerPatron = 5;
        public const int HoldExpirationHours = 48;
    }
    
    public static class Borrowing
    {
        public const decimal MaxFineForBorrowing = 5.00m;
    }
}
```

**Step 4:** Replace magic numbers

Ask Copilot:
```
Replace all magic numbers in LoanService with constants from LibraryConstants.
```

**Before:**
```csharp
loan.DueDate = loan.DueDate.AddDays(14);
```

**After:**
```csharp
loan.DueDate = loan.DueDate.AddDays(LibraryConstants.Loans.ExtensionDays);
```

---

### Exercise 2.4: Improve Error Handling

**Step 1:** Find methods with poor error handling

Look for:
- Empty catch blocks
- Generic exception catching
- No logging
- Swallowed exceptions

**Step 2:** Ask Copilot to improve:

```
Review error handling in PatronService and suggest improvements:
- Add proper logging
- Use specific exception types
- Add try-catch around I/O operations
- Return meaningful error status
- Never swallow exceptions
```

**Example improvement:**

**Before:**
```csharp
public async Task<Patron> GetPatron(int id)
{
    try
    {
        return await _patronRepository.GetPatron(id);
    }
    catch
    {
        return null;
    }
}
```

**After:**
```csharp
public async Task<Patron?> GetPatron(int id)
{
    if (id <= 0)
    {
        throw new ArgumentException("Patron ID must be positive", nameof(id));
    }
    
    try
    {
        return await _patronRepository.GetPatron(id);
    }
    catch (Exception ex)
    {
        // Log or handle, but never swallow
        throw;
    }
}
```

---

## Part 3: Code Documentation & Understanding (10 minutes)

### Exercise 3.1: Generate XML Documentation

**Step 1:** Find methods without XML documentation

**Step 2:** Select a method and ask Copilot:

```
Generate comprehensive XML documentation for this method including:
- <summary> describing what it does
- <param> for each parameter
- <returns> describing return value
- <exception> for each exception thrown
- <remarks> for business rules or important notes
```

**Example:**

```csharp
/// <summary>
/// Extends the due date of a loan by 14 days if eligible.
/// </summary>
/// <param name="loanId">The unique identifier of the loan to extend.</param>
/// <returns>
/// A <see cref="LoanExtensionStatus"/> indicating the result:
/// - Success: Loan extended successfully
/// - LoanNotFound: Loan with specified ID does not exist
/// - LoanExpired: Loan is past due date and cannot be extended
/// - MembershipExpired: Patron's membership has expired
/// - LoanReturned: Book has already been returned
/// </returns>
/// <exception cref="ArgumentException">Thrown when loanId is less than or equal to 0.</exception>
/// <remarks>
/// Business rules:
/// - Loans can only be extended if not overdue
/// - Patron must have active membership
/// - Extension adds exactly 14 days to current due date
/// - Operation is logged for audit purposes
/// </remarks>
public async Task<LoanExtensionStatus> ExtendLoan(int loanId)
{
    // Implementation...
}
```

---

### Exercise 3.2: Understand Complex Code

**Step 1:** Find a complex LINQ query or algorithm in the codebase

**Step 2:** Select it and ask Copilot:

```
Explain this code step by step in simple terms.
What is it trying to achieve?
Are there any edge cases or potential issues?
```

**Step 3:** Ask for simplification:

```
Can this code be simplified or made more readable?
Show me alternative implementations.
```

---

### Exercise 3.3: Add Inline Comments for Complex Logic

**Step 1:** Find business logic without comments

**Step 2:** Ask Copilot:

```
Add inline comments explaining the WHY (not the WHAT) for this business logic.
Focus on explaining business rules and decision points.
```

**Example:**

```csharp
public async Task<MembershipRenewalStatus> RenewMembership(int patronId)
{
    var patron = await _patronRepository.GetPatron(patronId);
    if (patron == null) return MembershipRenewalStatus.PatronNotFound;
    
    // Business rule: Cannot renew if more than 1 month before expiration
    // Prevents "stockpiling" renewal years; encourages just-in-time renewal
    if (patron.MembershipEnd >= DateTime.Now.AddMonths(1))
        return MembershipRenewalStatus.TooEarlyToRenew;
    
    // Business rule: Cannot renew if patron has overdue (unreturned) loans
    // This prevents patrons from renewing to avoid fines or extend overdue books
    if (patron.Loans.Any(l => l.ReturnDate == null && l.DueDate < DateTime.Now))
        return MembershipRenewalStatus.LoanNotReturned;
    
    // Add 1 year from current expiration date (not from today)
    // This ensures patrons don't lose remaining days by renewing early
    patron.MembershipEnd = patron.MembershipEnd.AddYears(1);
    
    try
    {
        await _patronRepository.UpdatePatron(patron);
        return MembershipRenewalStatus.Success;
    }
    catch (Exception e)
    {
        return MembershipRenewalStatus.Error;
    }
}
```

---

## Part 4: Security & Privacy (10 minutes)

### Exercise 4.1: Use .copilotignore to Protect Sensitive Files

**Step 1:** Create `.copilotignore` file in project root

```
File: .copilotignore
```

**Step 2:** Add patterns for sensitive files:

```
# Secrets and configuration
appsettings.*.json
*.key
*.pfx
*.p12
secrets.json
.env
.env.*

# Personal data
**/PersonalData/**
**/PII/**
customer-data.json

# Security files
*.pem
*.crt
id_rsa
id_dsa

# Database files with real data
*.db
*.sqlite
*-production.json

# API keys and tokens
**/tokens/**
**/keys/**
```

**Step 3:** Test it

Try to ask Copilot about a file in the ignore list:
```
Read the contents of appsettings.Production.json
```

Copilot should respect the ignore file and not access it.

---

### Exercise 4.2: Security Code Review

**Step 1:** Ask Copilot to review security:

```
Review the entire Library.ApplicationCore project for security issues:
- SQL injection risks
- Authentication/authorization issues
- Input validation gaps
- Logging of sensitive data
- Error messages exposing internal details
- Use of hardcoded secrets
- Insecure cryptographic practices
```

**Step 2:** Address findings

For each issue found, ask:
```
/fix [specific security issue]
```

---

### Exercise 4.3: Validate User Inputs

**Step 1:** Find methods that accept user input

**Step 2:** Ask Copilot to add validation:

```
Add comprehensive input validation to this method:
- Null checks
- Range validation
- Format validation
- SQL injection prevention
- Cross-site scripting (XSS) prevention
- Length limits

Use ArgumentException and ArgumentNullException with descriptive messages.
```

**Example:**

```csharp
public async Task<LoanExtensionStatus> ExtendLoan(int loanId)
{
    // Input validation
    if (loanId <= 0)
    {
        throw new ArgumentException(
            "Loan ID must be a positive integer", 
            nameof(loanId));
    }
    
    // Continue with business logic...
}
```

---

## Part 5: Code Profiling & Performance (5 minutes)

### Exercise 5.1: Identify Performance Issues

**Step 1:** Look for performance anti-patterns

Ask Copilot:
```
Review the codebase for performance issues:
- N+1 query problems
- Unnecessary async/await
- Missing async in I/O operations
- Large object allocations in loops
- String concatenation in loops
- LINQ queries that could be optimized
```

**Step 2:** Review findings

Common issues in this codebase might include:
- Loading all loans then filtering in memory
- Multiple database calls in a loop
- Synchronous I/O operations

---

### Exercise 5.2: Optimize Database Queries

**Example N+1 Problem:**

**Before (N+1 queries):**
```csharp
public async Task<List<LoanDto>> GetActiveLoans()
{
    var loans = await _loanRepository.GetActiveLoans();
    
    var loanDtos = new List<LoanDto>();
    foreach (var loan in loans)
    {
        // N+1: Separate query for each loan!
        var patron = await _patronRepository.GetPatron(loan.PatronId);
        var book = await _bookRepository.GetBook(loan.BookItemId);
        
        loanDtos.Add(new LoanDto
        {
            LoanId = loan.Id,
            PatronName = patron.Name,
            BookTitle = book.Title,
            DueDate = loan.DueDate
        });
    }
    
    return loanDtos;
}
```

**Ask Copilot to optimize:**
```
Optimize this method to eliminate N+1 query problem.
Use eager loading or batch queries to reduce database roundtrips.
```

**After (single query):**
```csharp
public async Task<List<LoanDto>> GetActiveLoans()
{
    // Single query with all data
    var loans = await _loanRepository.GetActiveLoansWithDetails();
    
    var loanDtos = loans.Select(loan => new LoanDto
    {
        LoanId = loan.Id,
        PatronName = loan.Patron.Name,
        BookTitle = loan.BookItem.Book.Title,
        DueDate = loan.DueDate
    }).ToList();
    
    return loanDtos;
}
```

---

## Part 6: Slash Commands Mastery (5 minutes)

### Quick Reference: Essential Copilot Slash Commands

| Command | Purpose | Example |
|---------|---------|---------|
| `/fix` | Debug and fix code issues | `/fix NullReferenceException` |
| **Generate Tests** | Generate unit tests | Right-click method > Copilot > Generate Tests |
| `/explain` | Explain how code works | `/explain` (with code selected) |
| `/doc` | Generate documentation | `/doc` (with method selected) |
| `/help` | Show all available commands | `/help` |

---

### Exercise 6.1: Command Practice

**Step 1:** Select the `LoanService.ExtendLoan` method

**Step 2:** Try each command:

```
/explain
```
Review the explanation - does it capture the business logic?

Right-click the method > **Copilot** > **Generate Tests**

Review the generated tests - compare with your custom agent output.

```
/doc
```
Review the documentation - is it comprehensive?

---

## 🎓 Lab Summary & Key Takeaways

### What You Learned

1. **Debugging with Copilot**
   - `/fix` command for automatic bug detection
   - Identifying null reference risks
   - Fixing logic errors
   - Handling edge cases

2. **Code Refactoring**
   - Extract method refactoring
   - Replace magic numbers with constants
   - Improve error handling
   - Simplify complex logic

3. **Code Quality**
   - Generate XML documentation
   - Add meaningful comments
   - Follow naming conventions
   - Apply SOLID principles

4. **Security & Privacy**
   - Use `.copilotignore` for sensitive files
   - Security code reviews
   - Input validation
   - Prevent common vulnerabilities

5. **Performance Optimization**
   - Identify N+1 query problems
   - Optimize database access
   - Use async/await correctly
   - Profile code with Copilot insights

### Success Metrics

You successfully completed this lab if:
- ✅ Used `/fix` to debug and resolve issues
- ✅ Refactored code for better maintainability
- ✅ Added comprehensive documentation
- ✅ Created `.copilotignore` for sensitive files
- ✅ Identified and fixed performance issues
- ✅ Mastered Copilot commands for daily work

### Quality Improvements

**Before This Lab:**
- Code with bugs and magic numbers
- Poor error handling
- Missing documentation
- Security vulnerabilities
- Performance issues

**After This Lab:**
- Debugged and tested code
- Named constants instead of magic numbers
- Comprehensive error handling and logging
- Well-documented public APIs
- Security best practices applied
- Optimized database queries

---

## 🔧 Troubleshooting

### `/fix` Not Providing Good Suggestions
- Ensure error message is visible in Problems panel
- Select the problematic code before running `/fix`
- Provide more context in the prompt
- Try: `/fix [specific error message]`

### Refactoring Breaks Tests
- Run tests after each small refactoring
- Use version control - commit before refactoring
- Review Copilot's suggestions carefully
- Test edge cases after refactoring

### `.copilotignore` Not Working
- Ensure file is in project root
- Check file patterns are correct (glob syntax)
- Reload VS Code window
- Verify file is not already in Copilot's context

---

## 📚 Additional Resources

- [GitHub Copilot Slash Commands](https://docs.github.com/copilot/using-github-copilot/slash-commands)
- [Refactoring Catalog (Martin Fowler)](https://refactoring.com/catalog/)
- [OWASP Top 10 Security Risks](https://owasp.org/Top10/)
- [Clean Code Principles](https://www.oreilly.com/library/view/clean-code-a/9780136083238/)
- [Code Quality Best Practices](https://docs.microsoft.com/dotnet/csharp/fundamentals/coding-style/)

---

## 🚀 Next Steps

1. **Apply Quality Practices Daily**
   - Use `/fix` when you encounter errors
   - Refactor as you go (not as a separate phase)
   - Document public APIs immediately
   - Run security reviews regularly

2. **Integrate into CI/CD**
   - Add code coverage requirements
   - Run static analysis
   - Automated security scanning
   - Performance benchmarks

3. **Share with Team**
   - Commit `.copilotignore` to version control
   - Document refactoring patterns
   - Create team coding standards
   - Conduct code review sessions

4. **Continue Learning**
   - Explore advanced Copilot features
   - Learn new slash commands as they're released
   - Experiment with custom instructions for quality
   - Build quality-focused custom agents

---

## 🎯 Final Challenge: Complete Quality Review

**Task:** Conduct a comprehensive quality review of the entire Library.ApplicationCore project.

**Use Copilot to:**
1. Identify all code smells
2. Generate missing tests
3. Add missing documentation
4. Fix security vulnerabilities
5. Optimize performance bottlenecks
6. Refactor complex methods

**Goal:** Achieve:
- ✅ 80%+ test coverage
- ✅ All public APIs documented
- ✅ Zero high-severity security issues
- ✅ All methods under 20 lines
- ✅ No magic numbers
- ✅ Comprehensive error handling

**Time Challenge:** Can you do it in under 2 hours with Copilot? 
(Without Copilot, this would take 8-10 hours!)

---

**Congratulations!** You've mastered code quality, refactoring, and debugging with GitHub Copilot. You now have the skills to maintain high-quality codebases efficiently. 🎉

---

## 📋 Complete Labs Checklist

- ✅ **Lab 1:** Custom Instructions and Custom Agents
- ✅ **Lab 2:** Test-Driven Development with Copilot
- ✅ **Lab 3:** Code Quality, Refactoring & Debugging

**You are now a GitHub Copilot power user!** 🚀

Apply these skills to your daily work and experience:
- 3-5x faster development
- Higher code quality
- Fewer bugs in production
- Better team collaboration
- More time for creative problem-solving

**Join the 17% → Become a Mercury Copilot Expert!**
