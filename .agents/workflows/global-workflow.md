---
description: Global development workflow — Unity project environment setup and common tasks
---

# Global Workflow: Cheezy-Savoround (Unity)

## Local Development Setup

```bash
# 1. Mở dự án bằng Unity Editor
# Khởi động Unity Hub, chọn đúng phiên bản Unity và mở thư mục Cheezy-Savoround-PH61823.

# 2. Xử lý lỗi Cache (Nếu cần)
# Nếu dự án bị lỗi cấu trúc bất thường, đóng Unity, xóa thư mục Library/ và mở lại.
```

## Common Workflows (Quy trình làm việc chung)

| Quy trình | Hành động |
|---------|-------------|
| **Test Logic** | Chạy trực tiếp trong Play Mode của Unity Editor. |
| **Commit Code** | Cập nhật mã nguồn lên Git tối thiểu 3 lần/tuần (theo yêu cầu dự án). |
| **Chia nhánh Git** | Tạo nhánh tính năng (feature branch) riêng biệt trước khi bắt đầu code. |

## Before Committing (Kiểm tra trước khi Commit)

1. **Unity Console Sạch Sẽ:** Đảm bảo Console không có bất kỳ lỗi đỏ (Compiler Error) nào cản trở việc Build game. Xóa bỏ các `Debug.Log` rác.
2. **Play Mode Test:** Chạy thử game ở Play Mode để đảm bảo không văng lỗi Exception lúc Runtime (đặc biệt là lỗi NullReferenceException khi tham chiếu Prefab).
3. **Lưu dữ liệu:** Nhấn `Ctrl + S` để lưu Scene và `File -> Save Project` để lưu các thay đổi của ScriptableObject / Asset.
4. **Dọn rác Git:** Dùng `git status` để đảm bảo file `.gitignore` đã hoạt động tốt, tuyệt đối KHÔNG commit các thư mục `Library/`, `Temp/`, `Logs/`, `Obj/` lên Git.

## Notes

- Bạn phải sử dụng đúng **Commit Message Convention**.
- Cập nhật tiến độ vào `.specs/features/[feature]/tasks.md` khi bắt đầu hoặc hoàn thành một tính năng.
