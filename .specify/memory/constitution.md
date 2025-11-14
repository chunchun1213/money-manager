<!--
Sync Impact Report
==================
Version: 1.0.0 → 2.0.0
Change Type: MAJOR (Language requirement added - all specs, plans, and user-facing docs MUST be in Traditional Chinese)
Modified Principles:
  - All core principles remain the same (content unchanged)
  - NEW: Language & Documentation Requirements section added as first principle
Added Sections:
  - Language & Documentation Requirements (mandating Traditional Chinese for specs/plans/docs)
Removed Sections: None
Templates Status:
  ✅ .specify/memory/constitution.md - updated with language requirements
  ⚠ .specify/templates/plan-template.md - needs alignment with new language requirement
  ⚠ .specify/templates/spec-template.md - needs alignment with new language requirement
  ⚠ .specify/templates/tasks-template.md - needs alignment with new language requirement
  ⚠ .github/prompts/*.md - need to verify language requirement enforcement
Follow-up TODOs:
  - Update all template files to reflect Traditional Chinese requirement
  - Ensure all command prompt files enforce zh-TW for generated specs/plans
  - Update README.md to reflect Traditional Chinese documentation policy
Version Bump Rationale:
  This is a MAJOR version bump because it introduces a backward-incompatible governance change:
  All existing and future specifications, plans, and user-facing documentation MUST be
  written in Traditional Chinese (zh-TW). This fundamentally changes the documentation
  governance of the project.
-->

# Money Manager Constitution

## Language & Documentation Requirements

**All specifications, plans, and user-facing documentation MUST be written in Traditional Chinese (zh-TW).**

This includes but is not limited to:
- Feature specification documents (spec.md)
- Implementation plans (plan.md)
- Task lists (tasks.md)
- User guides and documentation
- User-facing portions of API documentation
- README and project introductions

Code comments may be in Traditional Chinese or English, but Traditional Chinese is recommended for team readability.

## Core Principles

### I. Code Quality (NON-NEGOTIABLE)

**All code MUST:**
- Follow consistent coding standards and style guides
- Include comprehensive inline documentation for complex logic
- Maintain single responsibility principle (SRP) for functions and modules
- Be linted and formatted before commit (automated via pre-commit hooks)
- Pass static analysis with zero critical issues
- Have meaningful variable and function names that express intent
- Avoid magic numbers; use named constants instead

**Rationale**: High code quality reduces bugs, improves maintainability, and accelerates onboarding. Poor code quality compounds technical debt exponentially.

### II. Testing Standards (NON-NEGOTIABLE)

**Test-Driven Development (TDD) is mandatory:**
- Write tests FIRST → Get user/team approval → Tests fail (Red) → Implement (Green) → Refactor
- Minimum 80% code coverage for business logic, 100% for critical paths
- Every bug fix MUST include a regression test before the fix
- Tests MUST be: Fast (<100ms unit tests), Isolated, Repeatable, Self-validating, Timely

**Test pyramid enforcement:**
- 70% Unit tests: Pure functions, business logic, utilities
- 20% Integration tests: API contracts, database operations, service boundaries
- 10% E2E tests: Critical user journeys only

**Rationale**: TDD prevents regressions, documents behavior, and enables fearless refactoring. Testing after implementation leads to poor coverage and brittle tests.

### III. User Experience Consistency

**All user-facing features MUST:**
- Follow established UI/UX patterns consistently across the application
- Provide immediate feedback for user actions (loading states, success/error messages)
- Support keyboard navigation and accessibility standards (WCAG 2.1 AA minimum)
- Handle errors gracefully with actionable error messages (never show stack traces to users)
- Maintain consistent terminology, icons, and visual hierarchy
- Work reliably across supported browsers/platforms without degradation

**User input validation:**
- Validate on client-side for immediate feedback
- Validate on server-side for security (never trust client input)
- Provide clear, specific error messages ("Amount must be positive" not "Invalid input")

**Rationale**: Consistency reduces cognitive load, builds user trust, and minimizes support burden. Inconsistent UX creates confusion and erodes confidence.

### IV. Performance Requirements

**Application MUST meet these benchmarks:**
- Initial page load: <2 seconds (3G network)
- Time to Interactive (TTI): <3.5 seconds
- API response time (p95): <500ms for reads, <1000ms for writes
- Database queries: <100ms for simple queries, <500ms for complex aggregations
- Bundle size: <250KB initial JS (gzipped), lazy-load non-critical assets

**Performance practices:**
- Implement pagination for lists >50 items
- Use debouncing/throttling for frequent user interactions
- Cache frequently accessed, infrequently changed data
- Optimize images (WebP format, lazy loading, responsive sizing)
- Monitor and alert on performance regressions via CI/CD

**Rationale**: Performance is a feature. Slow applications frustrate users, increase bounce rates, and waste computational resources.

### V. Maintainability & Simplicity

**Development principles:**
- Start simple: Implement only what's needed now (YAGNI - You Aren't Gonna Need It)
- Complexity MUST be justified in writing (architecture decision records)
- Prefer composition over inheritance, pure functions over stateful objects
- Dependencies MUST be justified, minimal, and actively maintained
- Configuration over code: Externalize environment-specific settings
- Delete unused code immediately; don't comment it out

**Documentation requirements:**
- README with setup instructions testable by new developers
- Architecture Decision Records (ADRs) for significant design choices
- API documentation auto-generated from code annotations
- Inline comments for "why" not "what" (code explains what it does)

**Rationale**: Simple systems are easier to understand, debug, and evolve. Premature complexity creates maintenance burden and slows development.

## Quality Enforcement

**Automated quality gates (blocking):**
- Linting and formatting checks (ESLint, Prettier, or equivalents)
- Type checking (TypeScript strict mode, or equivalent for other languages)
- Security scanning (dependency vulnerabilities, OWASP top 10)
- Test suite passes with required coverage thresholds
- Performance budget checks (bundle size, lighthouse scores)

**Manual review requirements:**
- Code reviews MUST check: logic correctness, test adequacy, UX consistency, performance impact
- At least one approval from maintainer with domain expertise
- Security-sensitive changes require security-focused review

**Continuous monitoring:**
- Error tracking with alerting thresholds
- Performance monitoring (APM) for production
- Quarterly dependency audits and updates

## Development Workflow

**Feature development process:**
1. Specification: Document requirements, acceptance criteria, UX flows
2. Design review: Validate approach aligns with constitution principles
3. Test implementation: Write failing tests covering happy path and edge cases
4. Implementation: Write minimal code to make tests pass
5. Refactoring: Clean up while maintaining test passing status
6. Review: Code review verifying quality, testing, UX, and performance standards
7. Staging deployment: Validate in production-like environment
8. Production deployment: Gradual rollout with monitoring

**Branch strategy:**
- `main` branch always deployable
- Feature branches from `main`, merged via pull requests
- Hotfixes follow expedited process but MUST include tests

**Commit hygiene:**
- Atomic commits with clear, imperative messages ("Fix login validation" not "Fixed stuff")
- Commits MUST be buildable and testable individually

## Governance

**This constitution supersedes all other development practices and guidelines.**

**Amendment process:**
- Amendments require documented rationale and team consensus
- MAJOR version bump: Principle removal or backward-incompatible governance changes
- MINOR version bump: New principle additions or material expansions
- PATCH version bump: Clarifications, typo fixes, non-semantic refinements

**Compliance:**
- All pull requests MUST demonstrate compliance with applicable principles
- Constitution violations require either: (a) fix the code, or (b) propose amendment
- Quarterly constitution review to ensure relevance and effectiveness

**Exceptions:**
- Emergency hotfixes may bypass non-critical gates (documentation, performance testing)
- Exceptions MUST be documented with rationale and followed up with compliance work

**Version**: 2.0.0 | **Ratified**: 2025-11-14 | **Last Amended**: 2025-11-14
