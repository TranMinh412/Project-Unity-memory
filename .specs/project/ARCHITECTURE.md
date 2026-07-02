# ARCHITECTURE: <PROJECT_NAME>

**Last Updated:** [YYYY-MM-DD]
> **AI CONTEXT:** This document is the authoritative technical reference. Read this FIRST for any technical question. Do not guess architectural patterns — verify here.

---

## 1. High-Level Structure
[Mô tả kiến trúc tổng quan của dự án, ví dụ: MVC, ECS, Component-based...]

### Sơ đồ luồng (Game Flow Diagram)
[Thêm sơ đồ Mermaid flowchart tại đây]

---

## 2. Identified Patterns
[Liệt kê các design pattern chính đang sử dụng. Ví dụ: Singleton, Observer, Object Pooling...]

## 3. Data Flow & Performance
- **GC Alloc Rules:** [Luật quản lý bộ nhớ, tối ưu Garbage Collector]
- **Render Optimization:** [Luật tối ưu render, batching, UI]

## 4. Code Organization & Conventions
**Structure Approach:** [Ví dụ: Feature-based folders]
**File Naming:** [Ví dụ: PascalCase]
**Testing Strategy:** [Play mode test, Edit mode test...]

## 5. Structural Tree (Unity Assets)
```text
Assets/
├── Core/
├── UI/
└── Features/
```

## 6. Shared Utilities & Core APIs
[Mô tả các API hoặc hàm tiện ích chung quan trọng nhất để tái sử dụng]
