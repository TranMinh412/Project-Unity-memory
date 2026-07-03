## **LỘ TRÌNH PHÁT TRIỂN & TIÊU CHÍ NGHIỆM THU CHI TIẾT** 

## **DỰ ÁN 1: CORE GAMEPLAY BLOOM SORT 3D (TUẦN 1 - TUẦN 4)** 

**Thời gian thực hiện:** 18/05/2026 - 14/06/2026 (4 tuần) 

**Yêu cầu cập nhật:** Sinh viên cập nhật mã nguồn lên Git tối thiểu **3 lần/tuần** (sử dụng đúng Commit Message Convention và phân chia nhánh tính năng). 

**Link game demo:** https://www.crazygames.com/game/bloom-sort 

## **Tài nguyên:** 

**- https://drive.google.com/drive/folders/1vWQKBfddgK5ZAKcDLEDidVlIn4_GoF1** 

**Yêu cầu tính năng:** Làm đủ các tính năng như trong GDD mô tả, bỏ phần monetization. Chủ đề game sẽ đổi thành pizza thay vì cánh hoa như game demo. 

**Level:** 30 level (có thể tham khảo level ở game demo). 

## **TUẦN 1 (18/05 - 24/05): Thiết kế Luồng Vận Hành & Giải Thuật** 

## **1. Kế hoạch công việc hàng ngày** 

- **Ngày 1 - Ngày 2 (Thiết kế hệ thống):** 

   - Nghiên cứu kỹ tài liệu yêu cầu kỹ thuật (Yêu Cầu Kỹ Thuật Dự Án 1 (Bloom Sort 3D). 

   - Vẽ **Sơ đồ hoạt động hệ thống (System Flow Diagram)** bằng công cụ tự chọn (Draw.io, Miro, Figma...). 

   - Xuất sơ đồ ra file ảnh, lưu vào thư mục /docs/ trên Git, nhúng vào file README.md và gửi link Repo cho Mentor. 

- **Ngày 3 (Khởi tạo dự án):** 

   - Tạo dự án mới trên Engine (Unity). 

   - Thiết lập cấu trúc thư mục chuẩn hóa: Scripts/, Prefabs/, Textures/, Models/, Resources/. 

   - Cấu hình tệp Git .gitignore chuẩn để tránh commit file rác sinh ra từ Library/Temp. 

- **Ngày 4 - Ngày 5 (Xây dựng Grid logic):** 

   - Viết mã nguồn khởi tạo Grid 3D ảo tự động từ file cấu hình JSON. 

   - Hiển thị thô bàn chơi bằng các khối Cube tượng trưng cho ô lưới trên màn hình 3D. 

- **Ngày 6 - Ngày 7 (Lập trình Raycast & Snapping):** 

   - Lập trình hệ thống **Raycast** phát hiện ô lưới khi rê chuột/touch. 

   - Viết cơ chế **Snapping** chậu hoa thô tự động hút vào tâm ô lưới. 

   - Code thuật toán kiểm tra lân cận 4 hướng: Trả về danh sách chậu liền kề có màu cánh hoa trùng khớp. 

## **2. Tiêu chí nghiệm thu (Cuối tuần 1)** 

- Trang chủ Git (README.md) hiển thị trực quan Sơ đồ hoạt động hệ thống dạng hình ảnh. 

- Thư mục Git sạch sẽ, không bị dính file rác của engine. 

- Chạy bản build debug thô: Kéo thả chậu hoa thô (khối hộp) đã tự động Snap chính xác vào tâm ô lưới. 

- Console log hiển thị chính xác danh sách các ô lân cận trùng màu khi đặt một chậu hoa xuống. 

## **TUẦN 2 (25/05 - 31/05): Tích Hợp Asset 3D, Tweening & Hoàn Thiện Core Loop** 

## **1. Kế hoạch công việc hàng ngày** 

- **Ngày 1 - Ngày 2 (Ráp Asset 3D & Chuyển động cánh hoa):** 

   - Import bộ Asset 3D (mô hình chậu hoa, cánh hoa) vào engine, thay thế hoàn toàn các khối hộp thô của Tuần 1. 

   - Lập trình logic di chuyển cánh hoa giữa các chậu theo đường cong Bezier 3D mượt mà. 

- **Ngày 3 (Xây dựng FSM luồng game):** 

   - Áp dụng **FSM** để quản lý trạng thái luồng game (Menu, Đang chơi, Đang kiểm tra combo, Đang chạy animation, Game Over). 

- **Ngày 4 - Ngày 5 (Xử lý Game Over & Object Pooling):** 

   - Code cơ chế kết thúc game ( **Game Over** ) khi tất cả các ô trên lưới và khay chứa đều bị lấp đầy mà không thể merge thêm. 

   - Thiết lập hệ thống **Object Pooling** cho các hiệu ứng Particle nổ hoa và Text điểm số bay. 

- **Ngày 6 - Ngày 7 (Đánh bóng Game Feel):** 

   - Lập trình hiệu ứng Squash/Stretch khi đặt chậu, hiệu ứng Shake rung lắc khi đặt sai ô và Pitch Shift của âm thanh khi nổ hoa liên tục. 

## **2. Tiêu chí nghiệm thu (Cuối tuần 2)** 

- Thao tác kéo thả chậu hoa mượt mà, có Ghost Mesh xem trước vị trí thả. 

- Cánh hoa di chuyển mượt theo đường cong Bezier, không bị giật lag hay biến mất đột ngột. 

- FSM được phân tách rõ ràng thành các class trạng thái riêng biệt đúng như thiết kế. 

- Hệ thống Object Pooling hoạt động ổn định, không phát sinh lỗi rò rỉ bộ nhớ (Memory Leak). 

## **TUẦN 3 (01/06 - 07/06): Tích Hợp Meta Features & Save/Load Dữ Liệu** 

## **1. Kế hoạch công việc hàng ngày** 

- **Ngày 1 - Ngày 2 (Xây dựng hệ thống Save/Load JSON):** 

   - Tạo hệ thống lưu trữ dữ liệu người dùng (Save/Load System) lưu các thông số: Số vàng, danh sách skin đã mua, tiến trình nhiệm vụ bằng tệp JSON. 

- **Ngày 3 (Lập trình UI Shop):** 

   - Thiết kế UI Shop đọc dữ liệu từ JSON cấu hình: Cho phép mua skin chậu hoa mới bằng vàng và tự động cập nhật thay đổi trực quan mô hình Mesh 3D của chậu hoa trong game. 

- **Ngày 4 - Ngày 5 (Lập trình Daily Rewards bảo mật):** 

   - Code tính năng Daily Rewards điểm danh nhận thưởng, kiểm tra thời gian thực tế qua Timestamp dạng UTC để tránh việc người chơi đổi giờ máy tính để hack vàng. 

- **Ngày 6 - Ngày 7 (Lập trình Achievement hướng sự kiện):** 

   - Lập trình hệ thống Achievement hoạt động hướng sự kiện độc lập (Event-driven) thông qua việc lắng nghe các Event từ Core game phát ra. 

## **2. Tiêu chí nghiệm thu (Cuối tuần 3)** 

- Tắt game mở lại: Dữ liệu vàng, skin đã mua và tiến độ nhiệm vụ phải được giữ nguyên. 

- Thay đổi thời gian hệ thống của máy tính/điện thoại: Daily Reward không bị hack nhận thưởng liên tục. 

- Review code: Đảm bảo module UI, Shop và Achievement hoàn toàn không gọi trực tiếp các lớp xử lý Gameplay (áp dụng đúng Event System). 

## **TUẦN 4 (08/06 - 14/06): Tối Ưu Hóa Hiệu Năng & Đóng Gói Bản Build Hoàn Chỉnh** 

## **1. Kế hoạch công việc hàng ngày** 

- **Ngày 1 - Ngày 2 (Tối ưu hóa tài nguyên & Render):** 

   - Đóng gói toàn bộ hình ảnh UI đơn lẻ vào Sprite Atlas để tối ưu hóa Draw Calls. 

   - Cấu hình Static/Dynamic Batching và GPU Instancing cho các asset mô hình 3D trong game. 

   - Tách Canvas UI tĩnh và động độc lập. 

- **Ngày 3 (Profiler & Fix Bugs):** 

   - Sử dụng công cụ Profiler của Engine để rà soát lượng rác thải bộ nhớ (GC Alloc) khi xảy ra combo nổ lớn. 

   - Sửa triệt để các lỗi Null Reference còn tồn đọng trong quá trình chơi thử. 

- **Ngày 4 - Ngày 5 (Viết tài liệu README & Đóng gói sản phẩm):** 

   - Hoàn thiện và cập nhật tệp README.md tại thư mục gốc của Git (giới thiệu luồng chạy chính của game và cách cấu hình dữ liệu JSON). 

   - Xuất bản build chơi được hoàn chỉnh (file APK + kèm video chơi game). 

- **Ngày 6 - Ngày 7 (Nghiệm thu tổng kết):** 

   - Mentor tiến hành chơi thử trên bản build thực tế, chấm điểm mã nguồn dựa trên bảng Checklist và đưa ra đánh giá tổng kết Dự án 1. 

## **2. Tiêu chí nghiệm thu của Mentor (Cuối tuần 4)** 

- Bản build game chơi mượt mà trong thời gian dài (không sụt FPS đột ngột, không nóng máy). 

- Chỉ số GC Alloc tiệm cận về trong quá trình chơi thông thường (không sinh rác trong Update). 

- Tổng số Batches (Draw Calls) hiển thị trên cửa sổ Profiler luôn dưới mức khống chế Batches. 

- Tệp README.md trình bày sạch sẽ, đầy đủ thông tin hướng dẫn vận hành game. 

