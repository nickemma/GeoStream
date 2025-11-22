# Contributing to GeoStream

Thank you for your interest in contributing to GeoStream! This document provides guidelines and information for contributors.

---

## 📋 Table of Contents

- [Code of Conduct](#code-of-conduct)
- [Getting Started](#getting-started)
- [Development Setup](#development-setup)
- [How to Contribute](#how-to-contribute)
- [Contribution Guidelines](#contribution-guidelines)
- [Architecture Overview](#architecture-overview)
- [Testing Requirements](#testing-requirements)
- [Documentation Standards](#documentation-standards)
- [Review Process](#review-process)
- [Community](#community)

---

## 📜 Code of Conduct

### Our Commitment

We are committed to providing a welcoming and inclusive environment for all contributors, regardless of:
- Experience level (beginners welcome!)
- Background or identity
- Geographic location
- Technical expertise

### Expected Behavior

**Do:**
- Be respectful and constructive in discussions
- Welcome newcomers and help them get started
- Give credit where it's due
- Focus on what's best for the project and community
- Show empathy towards other contributors

**Don't:**
- Use offensive, discriminatory, or inappropriate language
- Engage in personal attacks or trolling
- Share others' private information without permission
- Spam or promote unrelated content

### Enforcement

Violations of the code of conduct may result in:
1. Warning from maintainers
2. Temporary ban from the project
3. Permanent ban for serious or repeated violations

Report issues to: nicholasemmanuel321@gmail.com

---

## 🚀 Getting Started

### Who Can Contribute?

**Everyone!** We welcome contributions from:
- **Students** learning distributed systems
- **Professionals** with production experience
- **Researchers** exploring consensus protocols
- **Technical writers** improving documentation
- **Designers** enhancing UX/UI
- **Anyone** passionate about distributed systems

### What Can You Contribute?

**Code Contributions:**
- Bug fixes
- New features
- Performance improvements
- Refactoring and code quality
- Test coverage improvements

**Non-Code Contributions:**
- Documentation improvements
- Architecture diagrams
- Tutorial creation
- Translation to other languages
- Issue triage and bug reports
- Community support

**Learning Contributions:**
- Share distributed systems resources
- Document your learning journey
- Write blog posts about the project
- Create video tutorials
- Present at meetups or conferences

---

## 🛠️ Development Setup

### Prerequisites

Before you begin, ensure you have:
- **Go 1.21+** installed ([Download](https://go.dev/dl/))
- **PostgreSQL 14+** ([Download](https://www.postgresql.org/download/))
- **Redis 7+** ([Download](https://redis.io/download))
- **Git** for version control
- **Docker** (optional, recommended) ([Download](https://www.docker.com/get-started))
- **Make** (optional, for convenience commands)

### Local Setup

**1. Fork the Repository**

Click the "Fork" button at the top right of the repository page.

**2. Clone Your Fork**
```bash
git clone https://github.com/YOUR_USERNAME/geostream.git
cd geostream
```

**3. Add Upstream Remote**
```bash
git remote add upstream https://github.com/nickemma/geostream.git
```

**4. Install Dependencies**

Using Docker (Recommended):
```bash
docker-compose up -d postgres redis
```

Or manually:
```bash
# Start PostgreSQL and Redis on your system
# Then install Go dependencies
go mod download
```

**5. Set Up Environment**
```bash
cp .env.example .env
# Edit .env with your local configuration
```

**6. Run Database Migrations**
```bash
make migrate-up
# Or manually: go run cmd/migrate/main.go up
```

**7. Start the Development Server**
```bash
make run
# Or manually: go run cmd/server/main.go
```

**8. Verify Setup**
```bash
curl http://localhost:8080/health
# Should return: {"status":"healthy"}
```

### Development Tools

**Recommended IDE Setup:**
- **VS Code** with Go extension
- **GoLand** by JetBrains
- **Vim/Neovim** with Go plugins

**Useful Tools:**
- **golangci-lint** for linting
- **gopls** for language server
- **delve** for debugging
- **Postman** or **Insomnia** for API testing

---

## 🤝 How to Contribute

### Step-by-Step Contribution Process

**1. Find or Create an Issue**

- Browse [existing issues](https://github.com/nickemma/geostream/issues)
- Look for issues labeled `good first issue` or `help wanted`
- If you have a new idea, create an issue first to discuss it

**2. Claim the Issue**

Comment on the issue saying you'd like to work on it. A maintainer will assign it to you.

**3. Create a Branch**
```bash
# Update your main branch
git checkout main
git pull upstream main

# Create a feature branch
git checkout -b feature/your-feature-name
# Or for bug fixes: git checkout -b fix/bug-description
```

**Branch Naming Conventions:**
- `feature/` - New features
- `fix/` - Bug fixes
- `docs/` - Documentation changes
- `refactor/` - Code refactoring
- `test/` - Test additions or fixes
- `perf/` - Performance improvements

**4. Make Your Changes**

- Write clean, readable code
- Follow the project's coding style (see below)
- Add tests for new functionality
- Update documentation as needed
- Keep commits focused and atomic

**5. Test Your Changes**
```bash
# Run all tests
make test

# Run specific tests
go test ./internal/core/geography/...

# Check test coverage
make test-coverage
```

**6. Commit Your Changes**
```bash
git add .
git commit -m "feat: add intelligent geo-routing algorithm"
```

**Commit Message Format:**
```
type(scope): brief description

Longer explanation if needed.

Fixes #issue_number
```

**Commit Types:**
- `feat:` - New feature
- `fix:` - Bug fix
- `docs:` - Documentation changes
- `test:` - Test additions or fixes
- `refactor:` - Code refactoring
- `perf:` - Performance improvements
- `chore:` - Maintenance tasks

**7. Push Your Changes**
```bash
git push origin feature/your-feature-name
```

**8. Create a Pull Request**

- Go to your fork on GitHub
- Click "Compare & pull request"
- Fill out the PR template completely
- Link related issues

**9. Respond to Review Feedback**

- Maintainers will review your PR
- Address feedback promptly
- Make requested changes
- Push updates to the same branch

**10. Celebrate! 🎉**

Once merged, your contribution is part of GeoStream!

---

## 📏 Contribution Guidelines

### Code Style

**Go Code Standards:**

- Follow [Effective Go](https://go.dev/doc/effective_go)
- Use `gofmt` for formatting (automatic in most IDEs)
- Keep functions small and focused (prefer <50 lines)
- Use meaningful variable names (no single letters except common idioms like `i`, `j`, `err`)
- Add comments for exported functions and complex logic
- Organize imports: standard library, external packages, internal packages

**Example:**
```go
// Good: Clear function name, documented, focused responsibility
// RouteRequest determines the optimal region for a user request
// based on latency, cost, and availability metrics.
func (s *GeographyService) RouteRequest(ctx context.Context, origin Location) (*RoutingDecision, error) {
    regions, err := s.repo.GetHealthyRegions(ctx)
    if err != nil {
        return nil, fmt.Errorf("failed to get regions: %w", err)
    }
    
    return s.selectOptimalRegion(origin, regions), nil
}

// Bad: Unclear name, no documentation, too long
func (s *GeographyService) Do(c context.Context, l Location) (*RoutingDecision, error) {
    // 100+ lines of mixed concerns...
}
```

**Error Handling:**

- Always handle errors explicitly
- Wrap errors with context using `fmt.Errorf` with `%w`
- Log errors at appropriate levels
- Return errors instead of panicking (except for programmer errors)

**Testing Standards:**

- Write tests for all new functionality
- Aim for 80%+ code coverage
- Use table-driven tests for multiple scenarios
- Mock external dependencies
- Write both unit and integration tests

### Pull Request Requirements

**Before Submitting:**

✅ Code compiles without errors  
✅ All tests pass  
✅ New tests added for new functionality  
✅ Documentation updated  
✅ No linting errors (`golangci-lint run`)  
✅ Commit messages follow convention  
✅ Branch is up to date with main  

**PR Description Should Include:**

- **What:** Brief description of changes
- **Why:** Motivation and context
- **How:** Implementation approach
- **Testing:** How you tested the changes
- **Screenshots:** For UI changes (if applicable)
- **Related Issues:** Link to related issues

**PR Template:**
```markdown
## Description
Brief description of what this PR does.

## Motivation
Why is this change necessary? What problem does it solve?

## Changes
- List of specific changes made
- Another change
- And another

## Testing
How did you test this? What edge cases did you consider?

## Screenshots (if applicable)
Add screenshots for visual changes.

## Checklist
- [ ] Tests added/updated
- [ ] Documentation updated
- [ ] No breaking changes
- [ ] All tests passing

Fixes #issue_number
```

### Documentation Standards

**Code Documentation:**

- Document all exported functions, types, and constants
- Explain the "why" not just the "what"
- Include examples for complex functionality
- Keep documentation in sync with code

**Example:**
```go
// RoutingStrategy defines how requests are distributed across regions.
// Different strategies optimize for different goals:
//   - LatencyOptimized: Minimizes round-trip time
//   - CostOptimized: Balances traffic to reduce cloud spend
//   - AvailabilityOptimized: Prioritizes healthy regions
//
// Example:
//   strategy := NewLatencyOptimizedStrategy()
//   decision := strategy.Route(userLocation, availableRegions)
type RoutingStrategy interface {
    Route(origin Location, regions []Region) *RoutingDecision
}
```

**README and Wiki Updates:**

- Update README.md for significant feature changes
- Add tutorials for new workflows
- Document configuration changes
- Update architecture diagrams when structure changes

---

## 🏛️ Architecture Overview

Understanding GeoStream's architecture helps you contribute effectively.

### High-Level Architecture
```
┌─────────────────────────────────────────────────────┐
│                 GeoStream Monolith                  │
│                                                     │
│  ┌─────────────────────────────────────────────┐   │
│  │           API Layer (HTTP)                  │   │
│  │  - Request routing                          │   │
│  │  - Authentication/Authorization             │   │
│  │  - Input validation                         │   │
│  └──────────────────┬──────────────────────────┘   │
│                     │                               │
│  ┌──────────────────▼──────────────────────────┐   │
│  │         Core Business Modules               │   │
│  │                                             │   │
│  │  ┌────────────────────────────────────┐    │   │
│  │  │  Geography Module                  │    │   │
│  │  │  - Location detection              │    │   │
│  │  │  - Routing algorithms              │    │   │
│  │  │  - Latency calculation             │    │   │
│  │  └────────────────────────────────────┘    │   │
│  │                                             │   │
│  │  ┌────────────────────────────────────┐    │   │
│  │  │  Health Module                     │    │   │
│  │  │  - Region monitoring               │    │   │
│  │  │  - Circuit breakers                │    │   │
│  │  │  - Failover logic                  │    │   │
│  │  └────────────────────────────────────┘    │   │
│  │                                             │   │
│  │  ┌────────────────────────────────────┐    │   │
│  │  │  Storage Module                    │    │   │
│  │  │  - Data persistence                │    │   │
│  │  │  - Cache management                │    │   │
│  │  │  - Query optimization              │    │   │
│  │  └────────────────────────────────────┘    │   │
│  └─────────────────────────────────────────────┘   │
│                     │                               │
│  ┌──────────────────▼──────────────────────────┐   │
│  │         Platform Layer                      │   │
│  │  - Database connections                     │   │
│  │  - Cache connections                        │   │
│  │  - Observability (metrics, logs, traces)   │   │
│  └─────────────────────────────────────────────┘   │
└─────────────────────────────────────────────────────┘
```

### Module Responsibilities

**Geography Module:**
- Determines optimal region for requests
- Implements routing strategies (latency, cost, availability)
- Calculates geographic distance and network latency
- Manages region metadata

**Health Module:**
- Monitors region health continuously
- Implements circuit breaker pattern
- Triggers automatic failover
- Tracks availability metrics

**Storage Module:**
- Abstracts database access
- Manages caching layer
- Handles data consistency
- Optimizes query performance

**Platform Layer:**
- Database connection pooling
- Redis client management
- Structured logging
- Prometheus metrics
- Distributed tracing (future)

### Key Design Patterns

**Dependency Injection:**
Modules receive dependencies through constructors, making them testable and flexible.

**Repository Pattern:**
Data access is abstracted behind interfaces, allowing easy testing and database changes.

**Circuit Breaker:**
Prevents cascading failures by failing fast when dependencies are unhealthy.

**Strategy Pattern:**
Different routing strategies can be swapped at runtime based on requirements.

### Module Interfaces

When contributing, respect module boundaries:

- Modules communicate through interfaces, not concrete types
- Each module has a clear `service.go` defining its public API
- Avoid circular dependencies between modules
- Keep modules loosely coupled

**Example Interface:**
```go
// geography/service.go
type Service interface {
    RouteRequest(ctx context.Context, origin Location) (*RoutingDecision, error)
    GetRegionLatency(regionID string) (time.Duration, error)
    UpdateRegionMetrics(regionID string, metrics Metrics) error
}
```

---

## 🧪 Testing Requirements

### Testing Philosophy

**We believe in:**
- Tests as documentation
- Fast feedback loops
- High confidence in changes
- Tests that catch real bugs

**We avoid:**
- Tests that test implementation details
- Flaky tests
- Slow test suites
- Tests for the sake of coverage

### Types of Tests

**1. Unit Tests (60% of test suite)**

Test individual functions and methods in isolation.

**Location:** Next to the code being tested (`_test.go` suffix)

**Requirements:**
- Fast (<100ms per test)
- Isolated (no external dependencies)
- Deterministic (same result every time)
- Use mocks for dependencies

**Example Test Structure:**
```go
func TestGeographyService_RouteRequest(t *testing.T) {
    tests := []struct {
        name          string
        origin        Location
        regions       []Region
        expectedRegion string
        expectedError error
    }{
        {
            name: "routes to nearest region",
            origin: Location{Lat: 40.7128, Lon: -74.0060}, // New York
            regions: []Region{
                {ID: "us-east", Lat: 37.7749, Lon: -122.4194}, // SF
                {ID: "us-west", Lat: 40.7128, Lon: -74.0060},  // NY
            },
            expectedRegion: "us-west",
            expectedError: nil,
        },
        // More test cases...
    }

    for _, tt := range tests {
        t.Run(tt.name, func(t *testing.T) {
            // Test implementation
        })
    }
}
```

**2. Integration Tests (30% of test suite)**

Test module interactions and database operations.

**Location:** `tests/integration/` directory

**Requirements:**
- Test realistic scenarios
- Use test database (not production)
- Clean up after each test
- Can be slower (<5s per test)

**What to Test:**
- API endpoints with database
- Module interactions
- Caching behavior
- Error handling across layers

**3. Load Tests (10% of test suite)**

Test performance under realistic load.

**Location:** `tests/load/` directory

**Requirements:**
- Measure latency (P50, P95, P99)
- Test throughput (requests/second)
- Monitor resource usage
- Document baseline performance

**When to Run:**
- Before major releases
- After performance optimizations
- When investigating performance issues

### Running Tests
```bash
# Run all tests
make test

# Run tests with coverage
make test-coverage

# Run only unit tests
go test ./internal/...

# Run specific module tests
go test ./internal/core/geography/...

# Run integration tests
make test-integration

# Run load tests
make test-load

# Run tests with verbose output
go test -v ./...

# Run tests in parallel (faster)
go test -parallel 4 ./...
```

### Test Coverage Requirements

**Minimum Coverage:**
- Overall: 70%
- New code: 80%
- Critical paths (routing, failover): 90%

**Coverage Reports:**
```bash
# Generate HTML coverage report
make test-coverage
# Opens browser with coverage visualization
```

**What Not to Test:**
- Third-party libraries
- Generated code
- Trivial getters/setters
- Main function

### Writing Good Tests

**Do:**
- ✅ Use descriptive test names
- ✅ Test one thing per test
- ✅ Use table-driven tests for multiple scenarios
- ✅ Test edge cases and error conditions
- ✅ Make tests independent of each other
- ✅ Use test helpers to reduce duplication

**Don't:**
- ❌ Test implementation details
- ❌ Write flaky tests (tests that sometimes fail)
- ❌ Use sleep/time.Sleep in tests
- ❌ Share state between tests
- ❌ Hard-code test data that should be generated

---

## 📚 Documentation Standards

### What Needs Documentation?

**Always Document:**
- Public APIs and interfaces
- Complex algorithms
- Configuration options
- Deployment procedures
- Breaking changes

**Consider Documenting:**
- Design decisions and trade-offs
- Performance characteristics
- Error scenarios and recovery
- Security considerations

### Documentation Types

**1. Code Comments**
```go
// Good: Explains why, not just what
// We use exponential backoff here because linear retry
// can overwhelm the database during recovery from outages.
// The jitter prevents thundering herd when multiple instances retry.
func (c *Client) retryWithBackoff(ctx context.Context, fn func() error) error {
    // Implementation...
}

// Bad: Restates the code
// Retry with backoff
func (c *Client) retryWithBackoff(ctx context.Context, fn func() error) error {
    // Implementation...
}
```

**2. README Updates**

Update README.md when:
- Adding new major features
- Changing architecture
- Modifying setup instructions
- Adding new dependencies

**3. Architecture Documentation**

Document in `docs/architecture.md`:
- System design decisions
- Module responsibilities
- Data flow diagrams
- Scaling strategies

**4. API Documentation**

Document in `docs/api.md`:
- Endpoint specifications
- Request/response formats
- Authentication requirements
- Rate limiting rules
- Error codes

**5. Runbooks**

Document in `docs/runbooks/`:
- Common operational tasks
- Troubleshooting guides
- Emergency procedures
- Performance tuning

### Documentation Tools

**Go Documentation:**
- Use `go doc` to generate documentation from comments
- Follow [Go documentation guidelines](https://go.dev/doc/comment)

**Diagrams:**
- Use Mermaid for architecture diagrams
- Include diagrams in markdown files
- Keep diagrams in version control

**External Documentation:**
- Use GitHub Wiki for detailed guides
- Link to external resources
- Keep documentation close to code when possible

---

## 👀 Review Process

### What Reviewers Look For

**Code Quality:**
- Correctness and logic
- Error handling
- Edge cases considered
- Code style compliance
- Performance implications

**Testing:**
- Adequate test coverage
- Tests actually test the right thing
- Tests are maintainable
- Integration tests for critical paths

**Documentation:**
- Public APIs documented
- Complex logic explained
- README updated if needed
- Breaking changes noted

**Architecture:**
- Follows project patterns
- Doesn't introduce tight coupling
- Respects module boundaries
- Considers future extensibility

### Review Timeline

**Expected Response Times:**
- Initial review: Within 3-5 days
- Follow-up reviews: Within 2 days
- Final approval: Within 1 day of addressing feedback

**If No Response:**
- Ping reviewers after 5 days
- Mention maintainers in comments
- Be patient – this is an open-source project

### Addressing Review Feedback

**Best Practices:**
- Respond to all comments (even with just "Done" or "Thanks")
- Ask questions if feedback is unclear
- Explain your reasoning if you disagree
- Be open to suggestions
- Push updates promptly

**When You Disagree:**
- Explain your perspective respectfully
- Provide technical justification
- Be willing to compromise
- Remember maintainers have project context you might not

### Review Checklist for Contributors

Before requesting review, verify:

- [ ] All tests pass locally
- [ ] No linting errors
- [ ] Documentation updated
- [ ] Commit messages follow convention
- [ ] PR description is complete
- [ ] Self-reviewed the changes
- [ ] No debug code or commented-out code
- [ ] No unrelated changes included

---

## 🌍 Community

### Communication Channels

**GitHub:**
- Issues: Bug reports and feature requests
- Discussions: General questions and ideas
- Pull Requests: Code contributions

**Future (as community grows):**
- Discord server for real-time chat
- Monthly contributor calls
- Newsletter for project updates

### Getting Help

**Where to Ask:**

1. **Search existing issues first** – Your question might be answered
2. **GitHub Discussions** – For general questions
3. **Issue comments** – For questions about specific issues
4. **Email maintainers** – For sensitive matters

**How to Ask:**
- Be specific about your problem
- Include relevant code/errors
- Describe what you've tried
- Be patient and respectful

### Recognition

**We appreciate contributors through:**
- Credit in release notes
- Contributor badge on GitHub
- Feature in project announcements
- Invitation to contributor team (for regular contributors)

### Becoming a Maintainer

**Path to Maintainership:**

1. **Consistent Contributions** – Regular, quality contributions over time
2. **Community Involvement** – Help others, participate in discussions
3. **Technical Expertise** – Deep understanding of the project
4. **Alignment with Vision** – Share project values and direction

**Maintainer Responsibilities:**
- Review pull requests
- Triage issues
- Guide new contributors
- Make architectural decisions
- Maintain project health

---

## 🎯 Contribution Ideas

### Good First Issues

**For Beginners:**
- Fix typos in documentation
- Add code comments
- Write additional tests
- Improve error messages
- Add logging to existing functions

**For Intermediate:**
- Implement missing features
- Optimize slow queries
- Add integration tests
- Improve API error handling
- Write tutorials

**For Advanced:**
- Design new modules
- Implement complex features (Raft consensus)
- Performance optimization
- Architecture improvements
- Distributed systems research

### Areas Needing Help

**High Priority:**
- Test coverage improvements
- Documentation gaps
- Performance optimization
- Error handling improvements

**Medium Priority:**
- Additional routing strategies
- Monitoring enhancements
- Configuration improvements
- Deployment automation

**Research/Future:**
- Raft consensus implementation
- Multi-region coordination
- Advanced caching strategies
- Cost optimization algorithms

---

## 📞 Contact

**Maintainers:**
- Nicholas Emmanuel – [@nickemma](https://github.com/nickemma) – nicholasemmanuel321@gmail.com

**Project Links:**
- [GitHub Repository](https://github.com/nickemma/geostream)
- [Issue Tracker](https://github.com/nickemma/geostream/issues)
- [Discussions](https://github.com/nickemma/geostream/discussions)

---

## 📄 License

By contributing to GeoStream, you agree that your contributions will be licensed under the MIT License.

---

**Thank you for contributing to GeoStream! Every contribution, no matter how small, helps make this project better.** 🚀

*"Whatever you do, work at it with all your heart, as working for the Lord." - Colossians 3:23*
