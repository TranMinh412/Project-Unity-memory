# CONVENTIONS: Tối ưu & Naming

**Last Updated:** 2026-07-02

> **AI CONTEXT:** This document outlines the absolute rules for writing code in this project. Failure to follow these rules will result in rejected code.

---

## 1. Coding Conventions
- **Classes/Structs:** `PascalCase` (e.g., `GridManager`).
- **Methods/Functions:** `PascalCase` cho public, `camelCase` cho private.
- **Variables/Parameters:** `camelCase`.
- **Private Fields:** Luôn có tiền tố `_` (e.g., `_scoreText`).
- **Comments:** Code phải có chú thích giải thích rõ ràng các thuật toán phức tạp.

## 2. Tối ưu Hiệu năng (Performance Rules)
- **Zero GC Alloc in Update:** Cấm tuyệt đối việc sử dụng `GameObject.Find()`, `GetComponent()`, hoặc tạo mới vùng nhớ (`new List`, `new Dictionary`, cộng chuỗi string) trong các hàm chạy liên tục mỗi khung hình (Update).
- **Object Pooling:** Phải xây dựng hệ thống tái sử dụng (Pool) cho các tài nguyên sinh ra liên tục (như quái vật, tia đạn, text sát thương).
- **Draw Call / Batches Control:** 
  - Gom toàn bộ ảnh UI phẳng và Sprite 2D vào Sprite Atlas.
  - Tận dụng Static Batching cho môi trường tĩnh và GPU Instancing/Dynamic Batching nếu có nhiều object giống nhau.
- **UI Canvas Optimization:** Bắt buộc tách UI thành Canvas tĩnh (như khung viền, background) và Canvas động (như thanh máu, điểm số) để tránh Engine phải render lại (Rebuild) toàn bộ UI mỗi khi điểm số thay đổi.
