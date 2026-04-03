---
name: code-reviewer
description: Use when reviewing code, conducting code reviews, analyzing pull requests, or when the user mentions "code review", "PR review", "code quality", or "static analysis".
---

# Code Reviewer

## Overview

Code Reviewer provides a systematic approach to conducting thorough code reviews. It helps identify code quality issues, potential bugs, security vulnerabilities, performance problems, and ensures adherence to best practices and coding standards before merging changes.

## When to Use

Use this skill when:

- Reviewing pull requests or merge requests
- Conducting code reviews
- The user mentions "code review", "PR review", or "review this code"
- Analyzing code quality
- Checking for bugs or security issues
- Validating adherence to coding standards
- Pre-merging code changes

## Review Process

### Phase 1: Understand the Change

Before diving into details, understand the scope and purpose:

1. **Read the PR description**
   - What is the purpose of this change?
   - What problem does it solve?
   - Are there any related issues or tickets?

2. **Identify the scope**
   - What files were changed?
   - How large is the change?
   - Is this a breaking change?

3. **Check the baseline**
   - Review existing code in affected areas
   - Understand the context and architecture
   - Identify any related code that might be affected

### Phase 2: Security Review

Check for security vulnerabilities first:

#### Input Validation

```javascript
// ❌ BAD: Unvalidated input
function queryDatabase(userInput) {
  return db.query(`SELECT * FROM users WHERE id = ${userInput}`);
}

// ✅ GOOD: Parameterized query
function queryDatabase(userId) {
  return db.query("SELECT * FROM users WHERE id = ?", [userId]);
}
```

#### Authentication & Authorization

- Verify user is authenticated before sensitive operations
- Check that authorization checks are properly enforced
- Look for privilege escalation possibilities

#### Data Exposure

- Ensure sensitive data is not logged or exposed
- Check for proper encryption of sensitive information
- Verify secure storage of credentials and secrets

#### Common Security Issues

| Issue          | What to Check                               | Fix                         |
| -------------- | ------------------------------------------- | --------------------------- |
| SQL Injection  | String concatenation in queries             | Use parameterized queries   |
| XSS            | Unescaped user input in HTML                | Sanitize and escape output  |
| CSRF           | Missing CSRF tokens on forms                | Add anti-CSRF tokens        |
| SSRF           | User-provided URLs fetched server-side      | Validate and sanitize URLs  |
| IDOR           | Direct object references without auth check | Verify authorization        |
| Path Traversal | File paths constructed from user input      | Validate and sanitize paths |

### Phase 3: Code Quality Review

#### Code Structure

- **Single Responsibility**: Does each function/class do one thing?
- **DRY (Don't Repeat Yourself)**: Is there code duplication?
- **Clean abstractions**: Are interfaces clean and focused?
- **Modularity**: Is code properly organized into modules?

```python
# ❌ BAD: God function doing multiple things
def process_user(user_data):
    validate(user_data)
    save_to_db(user_data)
    send_email(user_data)
    update_analytics(user_data)
    log_action(user_data)

# ✅ GOOD: Separated concerns
def process_user(user_data):
    validate(user_data)
    save_to_db(user_data)
    notify_user(user_data)
    track_action(user_data)
```

#### Error Handling

- Are errors properly handled?
- Are error messages user-friendly?
- Are errors logged appropriately?
- Are resources cleaned up in finally blocks?

```javascript
// ❌ BAD: Swallowed error
try {
  doSomething();
} catch (e) {
  // Silently ignored
}

// ✅ GOOD: Proper error handling
try {
  doSomething();
} catch (error) {
  logger.error("Failed to do something", { error });
  throw new AppError("Operation failed", error);
}
```

#### Type Safety

- Are types properly defined?
- Are interfaces/types consistent?
- Is type inference appropriate?
- Are generics used correctly?

```typescript
// ❌ BAD: Any type
function processData(data: any) {
  return data.value;
}

// ✅ GOOD: Proper typing
function processData(data: UserData): ProcessedResult {
  return data.value;
}
```

#### Naming Conventions

- Are names descriptive and meaningful?
- Do names follow language conventions?
- Are there any confusing or misleading names?

| Type       | Convention               | Example                           |
| ---------- | ------------------------ | --------------------------------- |
| Variables  | camelCase                | `userName`, `isActive`            |
| Constants  | UPPER_SNAKE_CASE         | `MAX_RETRY_COUNT`                 |
| Functions  | camelCase (verb phrase)  | `getUserById`, `calculateTotal`   |
| Classes    | PascalCase (noun)        | `UserService`, `PaymentProcessor` |
| Interfaces | PascalCase (noun phrase) | `UserRepository`, `IUserService`  |

### Phase 4: Performance Review

#### Algorithm Efficiency

- Is the algorithm appropriate for the data size?
- Are there unnecessary iterations?
- Can operations be cached?

```python
# ❌ BAD: O(n²) nested loop
for user in users:
    for order in orders:
        if order.user_id == user.id:
            process(user, order)

# ✅ GOOD: O(n) with lookup
orders_by_user = defaultdict(list)
for order in orders:
    orders_by_user[order.user_id].append(order)

for user in users:
    for order in orders_by_user[user.id]:
        process(user, order)
```

#### Resource Management

- Are database connections properly pooled?
- Are files properly closed?
- Is memory usage reasonable?
- Are there potential memory leaks?

#### Database Queries

- Are indexes used appropriately?
- Are queries optimized?
- Are N+1 query problems avoided?

```sql
-- ❌ BAD: N+1 queries
SELECT * FROM users;
-- Then for each user:
SELECT * FROM orders WHERE user_id = ?;

-- ✅ GOOD: Single joined query
SELECT u.*, o.* FROM users u
LEFT JOIN orders o ON u.id = o.user_id;
```

### Phase 5: Testing Review

#### Test Coverage

- Are there tests for new functionality?
- Are edge cases covered?
- Are error conditions tested?

```javascript
// ❌ BAD: No tests for edge cases
function divide(a, b) {
  return a / b;
}

// ✅ GOOD: Edge cases covered
function divide(a, b) {
  if (b === 0) {
    throw new Error("Division by zero");
  }
  return a / b;
}
```

#### Test Quality

- Are tests isolated and independent?
- Are assertions meaningful?
- Do tests follow Arrange-Act-Assert pattern?

```python
# ❌ BAD: Poor test structure
def test_user():
    user = create_user()
    assert user.name == "John"  # What did we create?
    user.name = "Jane"
    save(user)
    assert user.name == "Jane"  # Saved what?

# ✅ GOOD: Clear AAA pattern
def test_create_user():
    # Arrange
    user_data = {'name': 'John', 'email': 'john@example.com'}

    # Act
    user = create_user(user_data)

    # Assert
    assert user.name == 'John'
    assert user.email == 'john@example.com'
    assert user.id is not None
```

### Phase 6: Documentation Review

#### Code Documentation

- Are complex algorithms explained?
- Are public APIs documented?
- Are edge cases and limitations documented?

```python
# ❌ BAD: No documentation
def calculate(x, y):
    return (x + y) / 2

# ✅ GOOD: Comprehensive docstring
def calculate(x, y):
    """Calculate the arithmetic mean of two numbers.

    Args:
        x: First number (int or float)
        y: Second number (int or float)

    Returns:
        The arithmetic mean of x and y.

    Raises:
        TypeError: If x or y are not numeric types.

    Examples:
        >>> calculate(2, 4)
        3.0
        >>> calculate(-1, 1)
        0.0
    """
    if not isinstance(x, (int, float)) or not isinstance(y, (int, float)):
        raise TypeError('Both arguments must be numeric')
    return (x + y) / 2
```

#### CHANGELOG/Commit Messages

- Is there a proper changelog entry?
- Are commit messages clear and descriptive?
- Does the change require version bump?

## Review Checklist

### Security

- [ ] Input validation on all user inputs
- [ ] Parameterized queries for database access
- [ ] Authentication and authorization checks
- [ ] No sensitive data in logs or responses
- [ ] No hardcoded credentials or secrets
- [ ] Proper error messages (no stack traces)
- [ ] HTTPS only for sensitive operations

### Code Quality

- [ ] Code follows style guidelines
- [ ] No code duplication
- [ ] Functions are small and focused
- [ ] Proper error handling
- [ ] Resources properly cleaned up
- [ ] Type annotations present (if applicable)
- [ ] Naming conventions followed

### Performance

- [ ] No obvious O(n²) or worse algorithms
- [ ] Database queries are optimized
- [ ] Appropriate use of caching
- [ ] Lazy loading where appropriate
- [ ] No memory leaks
- [ ] Connection pooling used

### Testing

- [ ] Tests for new functionality
- [ ] Edge cases covered
- [ ] Error conditions tested
- [ ] Tests follow AAA pattern
- [ ] Tests are independent and isolated

### Documentation

- [ ] Public APIs documented
- [ ] Complex logic explained
- [ ] README updated if needed
- [ ] CHANGELOG updated
- [ ] Examples provided

## Giving Feedback

### Be Constructive

- Focus on the code, not the person
- Explain why something should be changed
- Suggest alternatives
- Acknowledge good work

### Format Feedback

```
## Issue: [Brief title]

**Problem:** [What's wrong]

**Impact:** [Why it matters]

**Suggestion:** [How to fix]

**Example:** [Code snippet if applicable]
```

### Prioritize Issues

- **Blocker**: Must fix before merge
- **Important**: Should fix before merge
- **Suggestion**: Consider fixing in future
- **Nit**: Minor style issue

### Common Review Comments

| Comment                                    | Meaning                   |
| ------------------------------------------ | ------------------------- |
| "Can we extract this into a helper?"       | Code duplication          |
| "Consider adding tests for this edge case" | Missing test coverage     |
| "This might cause issues if x is null"     | Potential bug             |
| "Have you considered using y instead?"     | Better approach available |
| "Nit: variable name could be clearer"      | Minor style issue         |
| "This is outside the scope of this PR"     | Keep changes focused      |

## Quick Reference

| Review Phase  | Focus Areas                                |
| ------------- | ------------------------------------------ |
| Security      | Input validation, SQL injection, XSS, auth |
| Quality       | Structure, naming, error handling, types   |
| Performance   | Algorithms, queries, caching, memory       |
| Testing       | Coverage, edge cases, test quality         |
| Documentation | API docs, comments, changelog              |

## The Bottom Line

A thorough code review catches issues before they reach production. Focus on security first, then code quality, performance, testing, and documentation. Always be constructive and help the author improve, not just criticize.
