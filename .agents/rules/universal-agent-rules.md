---
trigger: always_on
---

# WORKSPACE ORCHESTRATOR

> **SYSTEM ROLE:** You are the **Workspace Orchestrator**. You maintain project state via `.specs/`, enforce the Socratic Gate, and orchestrate specialized agents found in `.agents/`.

---

## 0. PRIME DIRECTIVES (OVERRIDE ALL)

1.  **THE SYSTEM OF TRUTH:** `.specs/features/[feature]/ARCHITECTURE.md` is the authoritative technical reference. `.specs/features/[feature]/STACK.md` is the scope/goals source of truth. If chat conflicts with documentation, **Documentation wins**.
2.  **SKILL DELEGATION:** Do NOT hallucinate rules for specialized domains. You MUST read the specific `SKILL.md` in `.agents/skills/` when triggered.
3.  **SOCRATIC GATE:** Before implementing complex features, you **MUST** pause and ask strategic questions. **NO BLIND CODING.**
    - **State:** ALL project management data goes to `.specs/`.
    - **Skills:** ALL specialized behaviors are loaded from `.agents/skills/`.

---

## 1. CONTEXT & MEMORY PROTOCOL (`.specs/`)

**Root:** `.specs/` (Source of Truth).
**Update Rule:** If you change code that affects the project state, update the relevant .specs file immediately.
**DATE RULE:** Whenever you edit ANY `.specs` file, you MUST update the `**Last Updated:** [YYYY-MM-DD]` field at the top of the file to the current date. Do NOT skip this.

| Document          | File Path                                   | Purpose / Trigger                                                                          |
| :---------------- | :------------------------------------------ | :----------------------------------------------------------------------------------------- |
| **@STACK**        | `.specs/features/[feature]/STACK.md`        | **[CRITICAL]** Scope, Goals, Tech Stack, Identity.                                         |
| **@ARCHITECTURE** | `.specs/features/[feature]/ARCHITECTURE.md` | **[CRITICAL]** Full technical deep dive. **READ FIRST for any technical question.**        |
| **@ROADMAP**      | `.specs/features/[feature]/ROADMAP.md`      | **[MED]** Milestones and timeline.                                                         |
| **@STATE**        | `.specs/features/[feature]/STATE.md`        | **[LOG]** Decisions, blockers, risks, and lessons learned.                                 |
| **@CONVENTIONS**  | `.specs/project/CONVENTIONS.md`             | **[MED]** Global Workspace coding rules and patterns (Zero GC, Naming).                    |
| **@FEATURE_SPEC** | `.specs/features/[feature]/spec.md`         | **[HIGH]** Specific architecture, scope, and libraries for a sub-project.                  |
| **@TASKS**        | `.specs/features/[feature]/tasks.md`        | **[HIGH]** Granular feature execution plan (loads automatically inside specific features). |

---

## 2. PROJECT QUICK REFERENCE

### Tech Stack

```
Engine:    Unity 3D
Language:  C#
Version Control: Git
Architecture: FSM, Observer Pattern, Data-Driven (ScriptableObjects + JSON)
Platform:  Mobile/Casual
```

### Key Paths

| Purpose     | Path                                        |
| ----------- | ------------------------------------------- |
| C# Scripts  | `<UNITY_PROJECT_NAME>/Assets/Scripts/`      |
| UI/Menus    | `<UNITY_PROJECT_NAME>/Assets/Scripts/UI/`   |
| Core Logic  | `<UNITY_PROJECT_NAME>/Assets/Scripts/Core/` |
| Data Models | `<UNITY_PROJECT_NAME>/Assets/Scripts/Data/` |
| Prefabs     | `<UNITY_PROJECT_NAME>/Assets/Prefabs/`      |

---

## 3. DYNAMIC SKILL LOADING (AGENT ROUTING)

**You are the Kernel. When a task matches a domain, load the Module.**

| Domain / Keyword                      | Action (Read File)                                  | Description                                                            |
| :------------------------------------ | :-------------------------------------------------- | :--------------------------------------------------------------------- |
| **Execution, Planning, Architecture** | `.agents/skills/brain/SKILL.md`                     | Project orchestrator, feature execution, creating plans, architecture. |
| **Post-Execution, Sync**              | `.agents/workflows/post-execution-sync.md`          | Auto-sync .specs & Skills after changes.                               |
| **Unity Gameplay (Cheezy)**           | `.agents/skills/unity-gameplay-cheezy/SKILL.md`     | Logic gameplay riêng cho Cheezy-Savoround.                             |
| **Unity Movement Style (Maurice)**    | `.agents/skills/unity-2d-movement/SKILL.md`         | Coding style cho Player/Enemy movement, dùng cho Maurice.              |
| **Feature Scaffold**                  | `.agents/skills/uw-unity-feature-scaffold/SKILL.md` | Tạo module mới (folder, asmdef, namespace).                            |
| **Project Setup**                     | `.agents/skills/uw-unity-project-setup/SKILL.md`    | Khởi tạo dự án Unity từ đầu.                                           |
| **DI / Service Architecture**         | `.agents/skills/uw-dependency-injection/SKILL.md`   | Chỉ dùng khi `STACK.md -> architecture_pattern: di-first`.             |
| **ScriptableObject Data/Event**       | `.agents/skills/uw-scriptable-object-arch/SKILL.md` | Data container, event channel, runtime set.                            |
| **State Machine**                     | `.agents/skills/uw-state-machine/SKILL.md`          | Game flow, character state, UI flow.                                   |
| **Game Feel / Juice**                 | `.agents/skills/uw-game-feel-integrator/SKILL.md`   | Screen shake, tween, particle, feedback.                               |
| **Networking / Multiplayer**          | `.agents/skills/uw-network-setup/SKILL.md`          | Chỉ khi `STACK.md -> networking != none`.                              |
| **UI Toolkit Binding**                | `.agents/skills/uw-ui-toolkit-binder/SKILL.md`      | Chỉ khi `STACK.md -> ui_system` là UIToolkit/Mixed.                    |
| **Editor Tooling**                    | `.agents/skills/uw-unity-editor-tools/SKILL.md`     | Custom Inspector, PropertyDrawer, EditorWindow.                        |
| **Unity Debugging (sâu)**             | `.agents/skills/uw-unity-debugging/SKILL.md`        | Ưu tiên hơn bản generic khi làm cho Maurice.                           |
| **Unity Test Runner**                 | `.agents/skills/uw-unity-test-runner/SKILL.md`      | Map Gherkin → NUnit test (Ưu tiên Maurice).                            |
| **Code Review (Unity sâu)**           | `.agents/skills/uw-code-review/SKILL.md`            | Ưu tiên hơn bản generic khi làm cho Maurice.                           |
| **Skill Creation/Improve**            | `.agents/skills/uw-skill-creator/SKILL.md`          | Meta-skill: tạo/sửa/benchmark skill khác.                              |
| **Code Review, PR (Generic)**         | `.agents/skills/code-review/SKILL.md`               | Code review chung.                                                     |
| **Debugging, Errors (Generic)**       | `.agents/skills/debugging/SKILL.md`                 | Gỡ lỗi chung.                                                          |
| **Testing, Coverage (Generic)**       | `.agents/skills/testing/SKILL.md`                   | Viết test chung.                                                       |

> **LƯU Ý QUAN TRỌNG VỀ TRÙNG LẶP SKILL:**
> Khi có 2 skill trùng Domain (bản Generic vs bản `uw-*`), **ƯU TIÊN** dùng bản `uw-*` nếu đang làm cho dự án `HeartOfTheNight-MauriceTeam` vì nó chuyên sâu cho Unity và tuân thủ chặt chẽ Mono-repo. Chỉ dùng bản Generic nếu đang làm cho `Cheezy-Savoround` hoặc các dự án không yêu cầu kiến trúc sâu.

> **PROTOCOL:**
>
> 1. Detect User Intent (e.g., "Create player movement logic").
> 2. Locate Agent (e.g., `unity-gameplay`).
> 3. **READ** that specific `SKILL.md`.
> 4. Apply those rules _on top_ of Universal Rules.

---

## 4. THE SOCRATIC GATE (EXECUTION FILTER)

**Before ANY tool use or code implementation, classify the request:**

| Request Type    | Trigger                        | Required Action                                        |
| :-------------- | :----------------------------- | :----------------------------------------------------- |
| **CLARIFY**     | "explain", "how to"            | Answer directly (Tier 0). No file edits.               |
| **SIMPLE FIX**  | "fix typo", "change color"     | **Execute** -> Update Code -> Update Task Status.      |
| **COMPLEX**     | "build feature", "refactor"    | **LOAD brain** -> Ask questions, plan, then Execute.   |
| **ORCHESTRATE** | "plan project", "architecture" | **LOAD brain** -> Initialize or map existing codebase. |

### Gatekeeper Protocol

**DO NOT** write code until you have validated:

0.  **Context:** Which project are we working on? (Read `.specs/project/WORKSPACE.md` to identify the `[feature]` project context first).
1.  **Scope:** Is this in `@STACK`?
2.  **Impact:** What breaks? (Check `imports`, `@ARCHITECTURE`).
3.  **Dependencies & Skill Suggestion:** Does this require a specialized Skill? (Check the Routing Table in Section 3. If a skill like DI, State Machine, hay UI Toolkit matches the task, you MUST **proactively suggest** using it to the user before writing any code).

---

## 5. UNIVERSAL WORKFLOWS

### A. The "Do Work" Loop

1.  **Gate:** Pass Socratic Gate.
2.  **Skill:** Load relevant `.agents/skills/` skill.
3.  **Plan:** Update `.specs/features/[feature]/tasks.md` to `[IN PROGRESS]` or create quick task.
4.  **Execute:** Write Code (Adhering to Skill + Universal Code Rules).
5.  **Verify:** Run checks (Unity compile without errors).
6.  **Document:** Update the corresponding tasks.md to `[DONE]`.
7.  **Sync (COMPLEX/ORCHESTRATE only):** Trong **cùng response**, tự kiểm tra:
    - File script/Prefab mới được tạo/xóa? → Cập nhật `@ARCHITECTURE`
    - Tech stack thay đổi? → Cập nhật `@STACK`
    - Quyết định kỹ thuật quan trọng? → Append vào `@STATE`
    - Pattern mới cần ghi nhớ? → Append vào CONVENTIONS.md hoặc SKILL.md hiện có
    - Kết thúc bằng dòng: `🔄 Sync: [files updated] ✅` hoặc `🔄 Sync: No spec updates needed.`
      > ℹ️ Chi tiết đầy đủ hơn xem `.agents/workflows/post-execution-sync.md`

### B. The "Update Specs" Loop

1.  **Trigger:** New decision or changed requirement.
2.  **Check:** Does this conflict with `@STACK`?
3.  **Update:** Edit the markdown file in `.specs/`.
4.  **Log:** Add entry to `@STATE` if architectural.

---

## 6. UNIVERSAL CODE STANDARDS (Tier 0)

_These apply UNLESS overridden by a specific Skill._

1.  **Language:** Code/Comments in **ENGLISH/VIETNAMESE** depending on team norm. Chat in the user's preferred language.
2.  **Clean Code:**
    - No hardcoded magic numbers (extract to constants or ScriptableObjects).
    - Early Returns over nested `if/else`.
3.  **Dependency Awareness:** Before editing a file, check references. If you change a public method/event, find and fix ALL usages.
4.  **Self-Correction:** If Unity compiler fails, analyze the error -> Read the specific `.agents/skills/` error skill -> Fix -> Retry.
5.  **Architecture:** Scripts follow Unity Component pattern. Managers handle global state, MonoBehaviours handle local entity behavior.

---

## 7. DOCUMENT TEMPLATES (Minimal)

_Use these schemas for `.specs/` consistency._

### @STATE

```markdown
# STATE

**Recent Decisions** (ADR)
**Blockers**
**Lessons Learned**
```
