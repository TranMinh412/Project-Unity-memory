---
name: skill-unity-gameplay
description: Áp dụng khi lập trình logic Gameplay cho Unity (Cheezy-Savoround). Chứa luật về FSM, Event System, Data-Driven và Conventions.
version: 1.0.0
author: Cheezy-Savoround
---

# Role: Unity Gameplay Engineer

> **Purpose:** Xử lý toàn bộ logic gameplay của Unity theo chuẩn kiến trúc sạch (Clean Architecture), đảm bảo dễ bảo trì và mở rộng.

---

## 1. CONTEXT PROTOCOL (Read First)

Trước khi viết code, BẮT BUỘC phải đọc/kiểm tra:

1.  **File `Readme.md`**: Nắm rõ System Flow (Luồng chạy chính) của game.
2.  **File `.specs/project/ARCHITECTURE.md` (nếu có)**: Kiểm tra các quyết định kiến trúc đã chốt.

---

## 2. CORE DIRECTIVES (The Rules - Kỷ Luật Thép)

- **Coding Conventions:** Viết code sạch. 
  - Class: `PascalCase`.
  - Biến/Hàm: `camelCase`.
  - Biến private: Bắt đầu bằng dấu gạch dưới `_` (VD: `_score`, `_gridManager`).
  - Phải có chú thích (comment) giải thích rõ ràng cho các thuật toán phức tạp (VD: Thuật toán quét lân cận, đệ quy combo).
- **Data-Driven Design:** KHÔNG viết chết (hardcode) các thông số. Toàn bộ cấu hình lưới (grid), giá tiền vật phẩm, số miếng tối đa của đĩa... phải được đọc từ **JSON** hoặc **ScriptableObject**.
- **Finite State Machine (FSM):** Quản lý luồng game chặt chẽ qua các trạng thái rõ ràng (VD: `PlayingState`, `AnimatingState`, `CheckingComboState`, `GameOverState`). **Nghiêm cấm** sử dụng các cờ hiệu boolean (bool) lồng chéo nhau vô tội vạ.
- **Observer Pattern (Event System):** Bắt buộc sử dụng hệ thống Event (Action/UnityEvent) để giao tiếp giữa các hệ thống, giảm thiểu Tight Coupling. Không lạm dụng Singleton để gọi chéo hàm lẫn nhau.

---

## 3. EXECUTION WORKFLOW

Khi triển khai một tính năng Gameplay mới:
1.  **Xác định Data:** Tạo dữ liệu cấu hình (ScriptableObject hoặc JSON class) trước. Không gõ trực tiếp số vào code.
2.  **Định nghĩa Event:** Khai báo các sự kiện (Events) mà tính năng này sẽ phát ra hoặc lắng nghe.
3.  **Thiết kế State (FSM):** Tính năng này nằm ở State nào? Có cần thêm State mới vào Game State Machine không?
4.  **Viết Logic & Comment:** Triển khai code tuân thủ Convention và thêm comment giải thích thuật toán.

---

## 4. ANTI-PATTERNS (Những điều cấm kỵ)

- ⛔ **Don't:** Viết code dạng `if (isMoving && !isMerging && hasPizza)`. Thay vào đó, hãy dùng FSM: `if (currentState == GameState.Playing)`.
- ⛔ **Don't:** Gọi trực tiếp `UIManager.Instance.UpdateScore()`. Thay vào đó, hãy phát sự kiện `OnPizzaMerged?.Invoke(score)`, và cho `UIManager` lắng nghe.
- ⛔ **Don't:** Viết `int maxSlices = 6;` rải rác khắp nơi. Hãy để nó trong `GameConfig` ScriptableObject.

---

## 5. REFERENCE

### Thư mục quan trọng
| Thư mục | Mục đích |
|---------|---------|
| `Cheezy-Savoround-PH61823/Assets/Scripts/Core/` | Chứa Game Manager, FSM, Event System cốt lõi. |
| `Cheezy-Savoround-PH61823/Assets/Scripts/Data/` | Chứa các class định dạng dữ liệu, ScriptableObject. |
| `Cheezy-Savoround-PH61823/Assets/Scripts/Gameplay/` | Logic điều khiển đĩa, bàn cờ, thao tác kéo thả. |
