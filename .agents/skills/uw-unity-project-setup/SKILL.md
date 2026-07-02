---
name: uw-unity-project-setup
description: Full project initialization wizard from ProjectConfig, including folders, packages, Git, and MCP setup. Use when setting up a new Unity project from scratch, creating the folder structure, or bootstrapping infrastructure. Triggers on requests like "set up a new project", "create folder structure", "initialize the project", or "/uw-cmd-setup-project". Reads STACK.md for all decisions.
---

# Unity Project Setup

Orchestrate complete project initialization from `STACK.md`.

## Before You Start
1. Ensure `.specs/features/[feature]/STACK.md` is filled out.
2. Unity project must be created and open in Editor.
3. Check which MCP servers are available (Unity MCP for in-editor operations, GitHub for version control).

## Steps (in order)

### 1. Verify STACK.md
Confirm Unity version, packages, folder strategy, networking choice.

**Required fields** — warn the user if any are missing or empty:
- `project_name`
- `unity_version`
- `render_pipeline`
- `scripting_backend`
- `folder_strategy`

### 2. Create Folder Structure
Based on `folder_strategy`, create `Assets/_Project/` with standard subfolders:
- `Core/Scripts/` + `Core.asmdef`
- `Core/Tests/EditMode/` + `Core.Tests.EditMode.asmdef` (so the test runner skill has a test assembly from day one)
- `Features/` or `Scripts/` (depending on strategy)
- `Art/`, `Audio/`, `UI/`, `Prefabs/`, `Scenes/`, `Tests/`

### 3. Install Packages
From `STACK.md → packages`, install via Unity MCP `manage_packages` if available.

**Manual fallback (no MCP):** Write an editor script (`Editor/Setup/PackageInstaller.cs`) using `UnityEditor.PackageManager.Client.Add()` to install packages programmatically. The user runs it once from the Unity menu, then it can be deleted.

### 4. Generate Core Utilities
- **GameDebug.cs** via `uw-unity-debugging` skill
- Add `ENABLE_LOGS` define for Development builds

### 5. Git Setup
- Create `.gitignore` (Unity template)
- Create `develop` branch
- Initial commit
- Use GitHub MCP if available, otherwise git CLI

### 6. Configure Editor
Via Unity MCP if available:
- Render pipeline, color space (Linear), scripting backend (IL2CPP)

**Manual fallback (no MCP):** Write an editor script (`Editor/Setup/ProjectConfigurator.cs`) using `PlayerSettings` and `GraphicsSettings` APIs to set render pipeline, color space, scripting backend, and target platforms. Menu item under `Tools > Project Setup > Apply Settings`.

### 7. Documentation Setup
- Copy template files from `templates/` → `.specs/`
- Ensure STACK, STATE, ARCHITECTURE, ROADMAP are ready to be filled
- If `STACK.md → prompt_logging: true`, log to `.specs/features/[feature]/STATE.md`

### 8. Report MCP Status
```
✅ Unity MCP — Connected
✅ GitHub MCP — Connected
⬜ Linear MCP — Not configured
```

## Conditional Setup

After core setup, check STACK.md for features that need additional skill guidance:

| STACK.md Field | Condition | Action |
|---|---|---|
| `architecture_pattern` | `di-first` | Reference `uw-dependency-injection` skill for Reflex container setup |
| `networking` | not `none` | Reference `uw-network-setup` skill for networking boilerplate |
| `feel_tools` | any non-`none` value | Reference `uw-game-feel-integrator` skill for middleware integration |
| `ui_system` | `UIToolkit` or `Mixed` | Reference `uw-ui-toolkit-binder` skill for data binding setup |

Mention these as next steps — don't execute them during project setup.

## Asset Loading Strategy

| Approach | When to Use |
|----------|-------------|
| **Direct references** | Prefabs/SOs referenced in Inspector — always loads with scene |
| **Resources/** | Small-scale or early development. Loads synchronously. Everything in Resources/ ships in build. |
| **Addressables** | Production default. Async loading, memory management, optional remote hosting. Install `com.unity.addressables`. |
| **AssetBundles** | Advanced/custom CDN pipelines. Addressables wraps this — prefer Addressables unless you need low-level control. |

For persistent data: encrypt PlayerPrefs values at minimum. For production, prefer server-side storage with authentication.

## Rules
- **Never** create folders inside `Assets/` without `_Project/` prefix.
- Use `uw-unity-feature-scaffold` for feature modules.
- Use `Resources/` sparingly — prefer Addressables for production.

