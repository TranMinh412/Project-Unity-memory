# FEATURE: Tuần 1 - Thiết kế Luồng Vận Hành & Giải Thuật

**Status:** 🟢 COMPLETED

## 1. Feature Specification

Tính năng cốt lõi của Tuần 1 là thiết lập bộ khung cho cơ chế Gameplay chính của **Cheezy Savoround**.

- **Input:** Khởi tạo lưới Grid 3D động từ JSON (3x3 hoặc 4x4).
- **Tương tác:** Hệ thống Raycast để nhận diện thao tác kéo/thả từ Hold Slots vào lưới.
- **Snapping:** Khi người chơi thả tay, đĩa pizza (hiện tại là khối thô) tự động hút vào tâm ô lưới.
- **Thuật toán cốt lõi:** Sau khi đặt đĩa, quét 4 ô lân cận (trên, dưới, trái, phải). Nếu có miếng pizza cùng loại, hiển thị Console Log danh sách các ô trùng khớp (chưa cần làm animation bay miếng pizza ở tuần 1).

## 2. Technical Constraints

- Tuân thủ thiết kế Zero GC Alloc trong Update (không tạo rác).
- Bắt buộc phải có Sơ đồ luồng game (Mermaid) trong README.md.
- Commit Git tối thiểu 3 lần/tuần.

## 3. Execution Plan (Tasks)

- [x] **Task 1: Nghiên cứu kỹ thuật & Vẽ sơ đồ**
  - Đọc kỹ yêu cầu kỹ thuật (Bloom Sort 3D chuyển sang Pizza).
  - Hoàn thiện sơ đồ luồng hệ thống (Đã có Sơ đồ dạng markdown).
- [x] **Task 2: Khởi tạo dự án & Git**
  - Khởi tạo dự án Unity 3D (URP).
  - Setup cấu trúc thư mục (Scripts, Prefabs, Data, ...).
  - Cấu hình `.gitignore` và thực hiện First Commit.
- [x] **Task 3: Xây dựng Grid Logic**
  - Viết code đọc cấu hình lưới từ JSON (kích thước lưới).
  - Khởi tạo bàn chơi 3D bằng các khối Cube thô tượng trưng.
- [x] **Task 4: Hệ thống Raycast & Kéo Thả (Drag & Drop)**
  - Viết logic Raycast phát hiện va chạm với lưới khi drag.
  - Implement cơ chế Snapping (tự động hút đĩa vào tâm ô khi thả).
- [x] **Task 5: Thuật toán quét 4 hướng**
  - Viết hàm kiểm tra các đĩa liền kề (trên, dưới, trái, phải).
  - Log ra Console danh sách các chậu có miếng bánh cùng loại khi người chơi đặt đĩa xuống.
