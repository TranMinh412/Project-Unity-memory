# 🤖 Danh mục AI Skills & Workflows (Brain-agent)
**Tags:** #ai-workflow #unity #brain-agent #architecture
**Mục đích:** Bảng tra cứu nhanh các "Kỹ năng" và "Quy trình" mà AI đã được học để áp dụng vào dự án.

---

## 📂 1. WORKFLOWS (Quy trình tự động)
*Đường dẫn: `.agents/workflows/`*
*Đóng vai trò như các "Lệnh Macros" hoặc "Slash Commands" ép AI làm việc theo quy trình chuẩn.*

- **`/post-execution-sync`** (Quy trình đồng bộ trí nhớ): Bắt buộc chạy sau khi hoàn thành một task khó. AI sẽ tự động soi lại code, cập nhật luật mới vào `ARCHITECTURE.md` hoặc `STATE.md`, và tick hoàn thành task.
- **`/global-workflow`**: Các lệnh khởi tạo, cài đặt môi trường chung.

---

## 🧠 2. CORE SKILLS (Kỹ năng cốt lõi)
*Đường dẫn: `.agents/skills/`*
*Bộ não quản lý dự án và đảm bảo chất lượng code.*

- **`brain`**: Hệ điều hành chính. Quản lý việc lên plan, chia nhỏ task và thao tác với file `.specs`.
- **`code-review` & `uw-code-review`**: "Cảnh sát" bắt lỗi Clean Code và Naming Convention trước khi commit lên Git.
- **`debugging` & `uw-unity-debugging`**: Bác sĩ trị bug theo chuẩn 4 bước: Thu thập -> Cách ly -> Giả thuyết -> Kê đơn (Sửa lỗi & gắn GameDebug.Log).
- **`uw-skill-creator`**: "Kỹ năng tạo Kỹ năng". Dùng để đóng gói kinh nghiệm vừa học được thành một file Skill mới vĩnh viễn cho AI.

---

## 🎮 3. UNITY GAMEPLAY SKILLS (Kỹ năng Lập trình Game)
*Các tuyệt chiêu chuyên biệt để lập trình Gameplay mượt mà, tối ưu.*

- **`unity-gameplay`**: Kiến thức nền tảng (Raycast NonAlloc, tối ưu vật lý, Zero GC).
- **`uw-state-machine`**: Viết Finite State Machine (FSM) bằng Interface. Tách biệt hoàn toàn các hành động (Run, Jump, Dash...) để code không bị dính chùm.
- **`uw-game-feel-integrator`**: Phù thủy "Juice" game. Tự động thêm Screen Shake, Hitstop (khựng thời gian), Squash & Stretch, Particles và âm thanh.

---

## 🏗️ 4. UNITY ARCHITECTURE SKILLS (Kỹ năng Kiến trúc vĩ mô)
*Dành cho việc mở rộng dự án và xây dựng hạ tầng.*

- **`uw-scriptable-object-arch`**: Kiến trúc Data-Driven. Dùng ScriptableObject làm Database, Event Channel (Truyền tin không dính code) và Runtime Sets. Chống Hardcode.
- **`uw-ui-toolkit-binder`**: Chuyên làm UI thế hệ mới (UI Toolkit Unity 6+) bằng mô hình MVVM (Data Binding).
- **`uw-unity-editor-tools`**: Chuyên "độ" lại giao diện Inspector của Unity (Custom Inspectors, Gizmos, TagSelector) giúp hạn chế lỗi gõ nhầm (Typo) cho Designer.
- **`uw-dependency-injection`**: (Dùng cho dự án siêu lớn) Bơm phụ thuộc (DI) tự động bằng Reflex.
- **`uw-network-setup`**: (Dùng cho game Online) Khởi tạo cấu trúc Server/Client, RPCs.

---

## 🛠️ 5. UNITY TOOLING SKILLS (Kỹ năng Công cụ & Hạ tầng)
- **`uw-unity-project-setup`**: Setup thư mục, Gitignore lúc mới đẻ dự án.
- **`uw-unity-feature-scaffold`**: Tự động sinh thư mục và file `.asmdef` khi cần tạo một hệ thống lớn mới (VD: Hệ thống Quest, Inventory).
- **`testing` & `uw-unity-test-runner`**: Tự động viết test case (PlayMode / EditMode) để kiểm tra game không bị hỏng sau khi sửa code.