---
trigger: always_on
---

# Engineering Laws: Workspace

> This document defines the non-negotiable architectural and performance standards for the Mono-repo Unity projects. All generated code and refactors must comply with these laws.

---

## 🏗️ LAW: Game Architecture (Clean & Modular)
**Goal:** Prevent "God Classes" and tightly coupled code.

* **No God Objects:** Do not put everything in `GameManager`. Split logic into specialized managers (e.g., `GridManager`, `InputManager`, `ScoreManager`).
* **Observer Pattern (Event-Driven):** Systems must communicate via Events (C# `Action` or `UnityEvent`) instead of direct references. Avoid tight coupling.
* **Finite State Machine (FSM):** Use explicit State classes or enums for game states (`Playing`, `Animating`, `GameOver`). NEVER use nested booleans (`isMoving && !isMerging`).
* **Separation of Logic and View:** Visual effects and animations must react to logical state changes, not drive them.

---

## ⚡ LAW: Performance & Memory Hygiene
**Goal:** Maintain steady 60FPS on mobile and prevent memory leaks.

* **No Expensive Calls in Update:** NEVER use `GetComponent<T>()`, `GameObject.Find()`, or `FindObjectOfType<T>()` inside `Update()`, `FixedUpdate()`, or `LateUpdate()`. Cache references in `Awake()` or `Start()`.
* **Object Pooling:** Frequent spawning/destroying of objects (like projectiles, VFX explosions, particles) MUST use Object Pooling. Avoid `Instantiate`/`Destroy` during active gameplay.
* **Garbage Collection Avoidance:** Minimize memory allocations in update loops (e.g., avoid creating new strings, lists, or closures in `Update`).

---

## 📊 LAW: Data-Driven Design
**Goal:** Keep configuration out of code to allow fast iteration without recompiling.

* **Zero Hardcoding:** Numbers (e.g., max health = 100, points = 10, grid size) must NOT be hardcoded in C# scripts.
* **ScriptableObjects for Config:** Use `ScriptableObject` for game settings, item definitions, shop skins, and visual references.
* **JSON for Level Data:** Use JSON files exclusively for level layouts and matrix configurations, allowing dynamic loading and external level editors.

---

## 📁 LAW: Project Structure & Hierarchy Hygiene
**Goal:** Ensure the Unity project is navigable and clean.

* **Prefab-Centric Workflow:** Every visual entity in the scene must be a Prefab. Do not build complex objects directly in the scene hierarchy.
* **Clean Hierarchy:** Use empty GameObjects as folders to organize the scene (e.g., `--- MANAGERS ---`, `--- ENVIRONMENT ---`, `--- UI ---`).
* **Naming Conventions:** 
  - Scripts: `PascalCase`
  - Variables/Methods: `camelCase`
  - Private Fields: Prefix with `_` (e.g., `_scoreText`)

---

## 🤖 LAW: AI & Prompt Integrity
**Goal:** Keep AI context sharp and prevent hallucinations.

* **Domain Restriction:** AI must NOT suggest web frameworks (React, Vue, Node.js) or database ORMs for this project. This is a pure Unity C# game.
* **Rule Adherence:** The AI must always consult `.specs/project/CONVENTIONS.md` and the relevant feature skill (e.g., `.agents/skills/unity-gameplay-cheezy/SKILL.md`) before writing gameplay logic.
* **No Auto-Creating Skills:** CẤM tuyệt đối tự động tạo thư mục/file SKILL mới. Khi phát hiện pattern mới:
  1. Thuộc kỹ năng đã có (`brain`, `testing`, `debugging`…) → Append vào `SKILL.md` của kỹ năng đó.
  2. Là quy tắc cấu trúc chung → Append vào `.specs/project/CONVENTIONS.md`.
  3. Chỉ được **ĐỀ XUẤT** tạo Skill mới cho mảng công nghệ hoàn toàn độc lập và quy mô lớn → **Bắt buộc đợi User tạo thủ công**.
