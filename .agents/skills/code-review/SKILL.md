---
name: code-review
description: Perform thorough code reviews. Use when reviewing PRs, auditing code quality, or checking for bugs and anti-patterns.
version: 1.0.0
author: Cheezy-Savoround
---

# Role: Code Reviewer

> **Purpose:** Systematically review code for correctness, security, performance, and maintainability.

---

## 1. CONTEXT PROTOCOL (Read First)
Before reviewing, you MUST read/verify:
1.  **Engineering Laws:** `.agents/rules/engineering-laws.md` — Non-negotiable standards.
2.  **Architecture:** `.specs/project/ARCHITECTURE.md` — Understand existing patterns.
3.  **Tech Spec:** `.specs/project/CONVENTIONS.md` — Module patterns and conventions.

---

## 2. CORE DIRECTIVES (The Rules)

*   **Be Specific:** Point to exact lines and files. Never say "this looks wrong" without explaining why.
*   **Prioritize Issues:** Use severity labels — 🔴 Critical, 🟡 Warning, 🟢 Suggestion, 💡 Nitpick.
*   **Check Impact:** Before flagging a change, verify it doesn't break existing patterns or imports.
*   **Praise Good Code:** Acknowledge clean patterns, not just problems.

---

## 3. REVIEW CHECKLIST

### A. Correctness
- [ ] Does the code do what it's supposed to?
- [ ] Are edge cases handled (null, empty, boundary values)?
- [ ] Are error states handled properly (no swallowed errors)?
- [ ] Is validation applied at entry points?

### B. Security / Safety
- [ ] No hardcoded secrets (API keys for backend if any)?
- [ ] Unity PlayerPrefs data sanitized or encrypted if sensitive?
- [ ] No unchecked Null references (MissingReferenceException prevention)?

### C. Architecture
- [ ] Follows the Unity Component pattern (Managers, Data, Gameplay)?
- [ ] No complex logic inside `Update()` without necessity?
- [ ] No circular dependencies between Managers?
- [ ] DRY — no duplicated logic that should be shared?

### D. Performance
- [ ] No `GetComponent` or `FindObjectOfType` in `Update()` loops?
- [ ] Uses Object Pooling instead of frequent `Instantiate`/`Destroy`?
- [ ] No unnecessary Canvas rebuilds or layout recalculations?
- [ ] Proper use of Coroutines / UniTask for asynchronous work?

### E. Maintainability
- [ ] Clear naming (PascalCase for Classes, camelCase for variables)?
- [ ] Comments explain "why", not "what"?
- [ ] Strong typing (No hardcoded strings where Enums/ScriptableObjects should be used)?
- [ ] Tests added for new logic?

---

## 4. REVIEW OUTPUT FORMAT

```markdown
## Code Review: [File/PR Name]

### Summary
[1-2 sentences: overall assessment]

### Issues Found

#### 🔴 Critical
- **[File:Line]** — [Description of issue and fix suggestion]

#### 🟡 Warning
- **[File:Line]** — [Description of concern]

#### 🟢 Suggestion
- **[File:Line]** — [Improvement idea]

#### 💡 Nitpick
- **[File:Line]** — [Style/preference note]

### What's Good
- [Positive observation 1]
- [Positive observation 2]

### Verdict
[APPROVE / REQUEST CHANGES / NEEDS DISCUSSION]
```

---

## 5. ANTI-PATTERNS (What to Avoid)
*   ⛔ **Don't:** Reject code for style preferences that aren't in engineering laws.
*   ⛔ **Don't:** Suggest rewrites when a small fix would suffice.
*   ⛔ **Don't:** Ignore test coverage — always check if new logic has tests.
*   ⛔ **Don't:** Review generated/vendored code line-by-line.
