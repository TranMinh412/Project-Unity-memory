---
description: Post-execution context sync — reference guide for updating .specs after complex tasks
---

# Post-Execution Sync (Reference)

> **Vai trò:** File này là **tài liệu tham khảo chi tiết** cho bước Sync trong "Do Work" Loop.
> Logic sync chính đã được inline vào `universal-agent-rules.md` → mục "Do Work" Loop, step 7.
> Chỉ cần đọc file này khi cần tra cứu chi tiết hơn.

---

## Khi nào cần Sync?

Sync khi task vừa hoàn thành thuộc loại **COMPLEX** hoặc **ORCHESTRATE** VÀ có bất kỳ:

- Tạo/xóa file script hoặc Prefab
- Thay đổi cấu trúc thư mục Assets
- Thay đổi API public (Event, Interface, method signature)
- Thêm plugin/package mới
- Thay đổi kiến trúc (Design Pattern mới)
- Quyết định kỹ thuật ảnh hưởng tương lai dự án

**Bỏ qua** cho: typo, đổi màu, sửa 1 dòng, style-only.

---

## Step 1: Cập nhật .specs (chọn đúng file)

| Ưu tiên | Điều kiện | File cần sửa |
|---------|-----------|--------------|
| **CRITICAL** | Task có trong `tasks.md` | Mark `[DONE]`, cập nhật ngày |
| **CRITICAL** | Thay đổi kiến trúc | `@ARCHITECTURE` |
| **HIGH** | Tech stack thay đổi | `@STACK` |
| **LOGGED** | Quyết định kỹ thuật / risk mới | Append vào `@STATE` |
| **LOGGED** | Đạt milestone sprint | `@ROADMAP` |

## Step 2: Cập nhật Skills (nếu có pattern mới)

Nếu phát hiện pattern/hàm cốt lõi mới cần ghi nhớ:
1. Append vào `SKILL.md` của kỹ năng liên quan, HOẶC
2. Append vào `CONVENTIONS.md` nếu là quy tắc chung

> ⚠️ Luật cấm tạo Skill mới → xem `engineering-laws.md §AI & Prompt Integrity`

---

## Nguyên tắc

- **Surgical:** Chỉ sửa đúng section bị ảnh hưởng, KHÔNG rewrite toàn bộ file.
- **Append-only cho STATE:** Luôn append, không ghi đè lịch sử.
- **Inline report:** KHÔNG tạo response riêng. Ghi `🔄 Sync: [files] ✅` ở cuối response hiện tại.
