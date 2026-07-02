# CONVENTIONS: Cheezy Savoround

**Last Updated:** 2026-05-23

> **AI CONTEXT:** This document outlines the absolute rules for writing code in this project. Failure to follow these rules will result in rejected code.

---

## 1. Clean Code & Naming

- **Classes/Structs:** `PascalCase` (e.g., `GridManager`, `PizzaSlice`).
- **Methods/Functions:** `PascalCase` cho public (e.g., `UpdateGrid`), `camelCase` cho private (tùy chuẩn, nhưng ưu tiên `PascalCase` chung trong C# Unity).
- **Variables/Parameters:** `camelCase` (e.g., `pizzaType`, `gridSize`).
- **Private Fields:** Luôn có tiền tố `_` (e.g., `_scoreText`, `_isDragging`).
- **Comments:** Code logic phức tạp (đặc biệt thuật toán đệ quy, loang màu, bezier) bắt buộc phải có comment tiếng Việt rõ ràng.

## 2. Anti-Vibe Coding (Performance Rules)

- **Zero GC in Update:** KHÔNG `new List`, KHÔNG cộng chuỗi string, KHÔNG `GetComponent`, KHÔNG `GameObject.Find` trong `Update()`, `FixedUpdate()`, `LateUpdate()`. Cấm lạm dụng biến bool lồng nhau.
- **No Magic Numbers:** Mọi con số cấu hình phải được đặt thành hằng số (`const`), biến `[SerializeField]`, hoặc đọc từ JSON.
- **Event System:** UI không bao giờ được chứa logic gameplay. Bắt buộc dùng C# `Action` hoặc `UnityEvent`.

## 3. Unity Specifics

- Bắt buộc phải dùng `Physics.Raycast` trên LayerMask chuyên biệt để tối ưu va chạm.
- Chuyển động (bay, nhún nhảy) phải dùng thư viện Tween (như DOTween hoặc code nội suy Lerp/Slerp/Bezier chuẩn xác), không dùng Update thô kệch.
- Instantiation: Dùng `ObjectPool` cho mọi thứ xuất hiện và biến mất nhiều lần (VFX, text điểm, miếng bánh pizza).
