# ARCHITECTURE: Kiến trúc Code & Tư duy Hệ thống

**Last Updated:** 2026-07-02

> **AI CONTEXT:** This document is the authoritative technical reference. Read this FIRST for any technical question. Do not guess architectural patterns — verify here.

---

## 1. Data-Driven Design
- **Quy tắc tuyệt đối:** Không viết chết (hardcode) các thông số trong code.
- Toàn bộ thông tin cấu hình màn chơi, giá tiền vật phẩm shop, thuộc tính các loại phải được đọc từ file cấu hình ngoài (JSON hoặc ScriptableObject).

## 2. Finite State Machine (FSM)
- Quản lý luồng hoạt động của game (và cả nhân vật) chặt chẽ qua các Trạng thái (State) rõ ràng.
- **Nghiêm cấm** sử dụng các cờ hiệu boolean (`bool isJumping`, `bool isRunning`) lồng chéo nhau vô tội vạ tạo thành code "Spaghetti".

## 3. Observer Pattern (Event System)
- Sử dụng hệ thống Event (C# Action hoặc UnityEvent) để giảm thiểu sự phụ thuộc trực tiếp (Tight Coupling) giữa các Class.
- Mẫu kiến trúc này bắt buộc dùng chủ yếu để giao tiếp giữa Code Gameplay/Logic với Code UI. (VD: Nhân vật mất máu -> Phát Event -> UI nghe Event tự động trừ thanh máu).
