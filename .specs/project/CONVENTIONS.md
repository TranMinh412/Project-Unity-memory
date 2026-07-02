# CONVENTIONS: <PROJECT_NAME>

**Last Updated:** [YYYY-MM-DD]

> **AI CONTEXT:** This document outlines the absolute rules for writing code in this project. Failure to follow these rules will result in rejected code.

---

## 1. Clean Code & Naming
- **Classes/Structs:** `PascalCase` (e.g., `GridManager`).
- **Methods/Functions:** `PascalCase` cho public, `camelCase` cho private (tùy chuẩn, ưu tiên `PascalCase` trong C#).
- **Variables/Parameters:** `camelCase`.
- **Private Fields:** Luôn có tiền tố `_` (e.g., `_scoreText`).
- **Comments:** Code logic phức tạp bắt buộc phải có comment tiếng Việt hoặc Anh rõ ràng.

## 2. Anti-Vibe Coding (Performance Rules)
- **Zero GC in Update:** KHÔNG `new List`, KHÔNG cộng chuỗi string, KHÔNG `GetComponent`, KHÔNG `GameObject.Find` trong vòng lặp cập nhật.
- **No Magic Numbers:** Mọi con số cấu hình phải được đặt thành hằng số (`const`), biến `[SerializeField]`, hoặc đọc từ JSON/ScriptableObject.

## 3. Unity Specifics
- Instantiation: Dùng `ObjectPool` cho mọi thứ xuất hiện và biến mất nhiều lần (VFX, text điểm, đạn).
- Tránh thay đổi transform qua code trong Update nếu dùng Rigidbody (ưu tiên dùng lực vật lý).
