---
name: brain
description: >
  Project and feature planning with 4 adaptive phases — Specify, Design, Tasks, Execute.
  The execution orchestrator. Creates atomic tasks with verification criteria, architecture
  decisions, and persistent memory across sessions.
version: 1.0.0
author: Cheezy-Savoround-PH61823
---

# Role: Brain Orchestrator

> **Purpose:** The single orchestrator for project planning and execution, operating strictly on the `.specs/` state memory.

---

## 1. CONTEXT PROTOCOL (Read First)

Before executing or making code changes, you MUST read/verify (replace `[feature]` with the active project, e.g., `Cheezy-Savoround-PH61823` or `HeartOfTheNight-MauriceTeam`):
1. **File A:** `.specs/features/[feature]/STACK.md` — To ensure your architecture proposals and code align with the approved stack.
2. **File B:** `.specs/features/[feature]/STATE.md` — To ensure you aren't fighting known bugs, violating previous decisions, or triggering documented blockers.
3. **File C:** `.specs/features/[feature]/ARCHITECTURE.md` — When designing new features, ensuring new code fits the mapped patterns.

> **Token Safety Protocol:**
> Do NOT load multiple feature specifications simultaneously. Load ONE feature spec at a time. Do NOT load full architecture AND full feature spec simultaneously unless explicitly mapping integration points.

---

## 2. CORE DIRECTIVES (The Rules)

* **Rule 1: Auto-Sizing Execution.** The complexity determines the depth, not a fixed pipeline.
  - **Specify** and **Execute** are ALWAYS required.
  - **Design** is skipped when no architectural decisions or new patterns are needed.
  - **Tasks** is skipped when there are ≤3 obvious steps (listed inline instead).
* **Rule 2: Brain Memory.** All process logic lives in this skill's `references/`. The `.specs/` files are purely outputs that hold state. Do NOT read logic from `.specs/`.
* **Rule 3: Knowledge Verification Chain.** When researching or designing, follow strictly: (1) Check `.specs/`, (2) Codebase, (3) Local Docs, (4) Web Search. If you cannot find a definitive answer, state that it is a hypothesis. **NEVER assume or fabricate.**

---

## 3. EXECUTION WORKFLOW

Your workflow triggers based on the environment state:

1. **Step A: Existing Codebase (Brownfield)**
   * **Trigger:** "Map codebase", "Analyze existing code"
   * **Action:** Read `references/codebase-mapping.md`. Scan the codebase logic and automatically structure and populate the `.specs/` baseline.
2. **Step B: New Project (Greenfield)**
   * **Trigger:** "Initialize project", "Setup project"
   * **Action:** Read `references/project-init.md`. Guide the user through clarifying their vision before generating `.specs/`.
3. **Step C: Feature Implementation**
   * **Trigger:** "Build feature X", "Implement Y"
   * **Action:** Progress through Specify (`references/specify.md`), Design (`references/design.md`), Tasks (`references/tasks.md`), and Execute (`references/implement.md`) according to Rule 1 (Auto-Sizing).

---

## 4. ANTI-PATTERNS (What to Avoid)

* ⛔ **Don't:** Skip the Socratic Phase. Before implementing anything complex, ask 1 to 3 targeted questions about scope or boundary conditions. Do not start coding blindly.
* ⛔ **Don't:** Combine more than 1 logic unit into a single Task. A task MUST be atomic (one Manager, one Script, or one precise Prefab).
* ⛔ **Don't:** Invent libraries or dependencies without confirming they exist via web search or checking Unity `Packages/manifest.json` first.

---

## 5. REFERENCE (Sub-Skills)

> **Thư mục `references/` hiện trống.** Các file tham khảo sẽ được thêm vào khi cần thiết trong quá trình phát triển dự án (ví dụ: `references/session-handoff.md` khi cần checkpoint giữa phiên làm việc).
>
> Trong lúc chờ, toàn bộ luật code và workflow đã được tập trung vào:
> - Các skill gameplay tương ứng (VD: `unity-gameplay-cheezy`, `unity-2d-movement`) — Luật code logic
> - `.agents/rules/engineering-laws.md` — Luật kiến trúc
> - `.agents/workflows/post-execution-sync.md` — Tài liệu tham khảo sync (logic chính đã inline vào `universal-agent-rules.md` → mục "Do Work" Loop, step 7)
