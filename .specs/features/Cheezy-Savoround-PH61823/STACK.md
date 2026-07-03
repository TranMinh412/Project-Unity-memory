# STACK: Cheezy Savoround

**Last Updated:** 2026-05-23

> **AI CONTEXT:** This document is the authoritative source of truth for the project's scope, tech stack, and goals. DO NOT guess or hallucinate these details. Always refer to this document.

---

## 1. Objective & Scope

### Mission

Phát triển một tựa game Casual (Puzzle/Sort) dựa trên lối chơi của Bloom Sort 3D, nhưng với chủ đề là các đĩa bánh Pizza. Người chơi kéo thả đĩa bánh vào lưới 3D để gộp (merge) và tạo combo nổ dây chuyền.

### Value Proposition

- **For:** Người chơi hệ Casual thích giải đố nhẹ nhàng.
- **Problem:** Cần một trải nghiệm game kéo thả, có tính toán chiến thuật nhẹ nhưng thư giãn.
- **Solution:** Lối chơi kéo thả mượt mà (snapping), hiệu ứng kích thích (combo nổ, rung lắc, âm thanh pitch shift) và đồ họa 3D bắt mắt.

### In Scope

| Feature            | Status       | Description                                   |
| ------------------ | ------------ | --------------------------------------------- |
| Lưới Grid 3D       | 🟢 Completed | Sinh lưới động từ JSON (3x3, 4x4)             |
| Kéo thả & Snapping | 🟢 Completed | Dùng Raycast để bắt điểm, hiển thị Ghost Mesh |
| Merge & Cascade    | 🟢 Completed | Ghép 6 miếng bánh cùng loại để nổ, tạo combo  |
| Object Pooling     | 🟢 Completed | Tái sử dụng VFX, lát pizza, Floating Text     |
| Cửa hàng (Shop)    | 🟢 Completed | Mua skin đĩa bằng vàng (lưu JSON)             |
| FSM & Observer     | 🟢 Completed | Kiến trúc State Machine và Event-driven       |

### Out of Scope

- Các tính năng Monetization (Quảng cáo, In-app purchase) đã bị lược bỏ trong dự án thực tập này.

---

## 2. Tech Stack

### Current Stack

- **Engine:** Unity 3D (URP) - Version 6000.0.74f1
- **Ngôn ngữ:** C#
- **Kiến trúc:** Finite State Machine (FSM), Observer Pattern (Event System)
- **Data-driven:** JSON và ScriptableObjects
- **Version Control:** Git (Tối thiểu 3 commits/tuần)

### Key Technologies

| Layer      | Technology                    | Purpose                                         |
| ---------- | ----------------------------- | ----------------------------------------------- |
| Core Logic | C# & FSM                      | Xử lý luật chơi, trạng thái (Playing, GameOver) |
| Rendering  | GPU Instancing & Sprite Atlas | Tối ưu Batches (< 50 Batches)                   |
| Data       | C# JSON Utility               | Lưu trữ Map, Vàng, Skin, Nhiệm vụ               |
| UI         | Unity UI                      | Tách Canvas Tĩnh và Động riêng biệt             |

---

## 3. Stakeholders & Constraints

- **Thời hạn:** 4 Tuần (18/05/2026 - 14/06/2026)
- **Ràng buộc hiệu năng:** Steady 60 FPS trên mobile, GC Alloc tiệm cận 0 trong runtime loop. _(Chi tiết luật cụ thể → xem `CONVENTIONS.md §2`)_

## 4. Success Metrics

- Game chạy mượt, Batches < 50, GC Alloc tiệm cận 0.
- Mã nguồn Clean Code, không có memory leak.
- Đạt nghiệm thu từ Mentor của Taap Game.
