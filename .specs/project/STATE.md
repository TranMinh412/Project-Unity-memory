# STATE: Cheezy Savoround

**Last Updated:** 2026-06-02

> **AI CONTEXT:** This document tracks the current state, active decisions, and known issues. Update it when major architectural decisions are made.

---

## 1. Recent Decisions (ADR)

- **[2026-06-03] Anti-Bounce Loop triệt để (Chống hút ngược):** Sửa lỗi lặp vô hạn (Bounce Loop 4:2 vs 4:4) khi một đĩa vừa xả rác xong lại đi hút chính loại rác đó về. Áp dụng thêm Luật Thép: "Nếu một đĩa đang ở trạng thái 5/6 (chỉ còn 1 chỗ trống), nó CHỈ ĐƯỢC PHÉP HÚT loại bánh chiếm Đa Số trên đĩa". Điều này ép đĩa phải chờ đợi loại bánh phù hợp để nổ, thay vì hút bậy bạ loại thiểu số để rồi bị đầy 6/6 và lập tức xả rác ra lại.
- **[2026-06-02] FSM Deadlock & Đồng bộ hóa Nổ đĩa:** Sửa lỗi kẹt Game State không cho kéo đĩa sau khi đĩa Tâm Chấn (Priority 9) nổ. Quyết định thay đổi kiến trúc: Hủy bỏ đặc quyền "Nổ sau cùng" của đĩa Priority 9. Giờ đây MỌI đĩa (kể cả 9) khi đạt 6/6 tinh khiết sẽ nổ NGAY LẬP TỨC bên trong `ProcessNextMerge` và đều chia sẻ chung một lệnh Tween delay 0.4s. Code xử lý nổ thừa trong `CleanupPrivilegedPlates` đã bị xóa, giúp dọn dẹp hoàn toàn Dead code và lỗi nổ đồng loạt 1 frame.
- **[2026-06-02] DOTween & Object Pool UI (Zero Hardcoding):** Cài đặt và tích hợp DOTween cho hệ thống chữ Floating Text (báo điểm). Giải quyết triệt để bài toán UI trong môi trường Top-Down 3D: Chữ tự động xoay song song lăng kính Camera và bay mượt mà lên trên màn hình nhờ vector `Camera.main.transform.up` thay vì trục Y tuyệt đối. Loại bỏ hoàn toàn các dòng code can thiệp Scale và Màu sắc (Color) từ Script, trả lại 100% quyền tùy biến thiết kế cho Designer qua Inspector (TextMeshPro Auto Size/Gradient). Cấy lệnh `DOKill()` để đảm bảo an toàn tuyệt đối khi Object Pool tái sử dụng UI.

- **[2026-06-02] Game Feel Polish (VFX & Audio Pitch Shifting):**
  - Tích hợp VFX nổ đĩa sử dụng Asset "Cartoon FX Remaster" (Đã bóc tách toàn bộ script rác để giảm overhead). Để tuân thủ Data-Driven và Zero Hardcoding, `PooledVFX.cs` được nâng cấp thêm cơ chế `PlayAt` với biến `_autoScaleMultiplier` phơi bày ra Inspector, cho phép tự động Scale kích thước vụ nổ khớp với đĩa mà không can thiệp vào logic của `GridManager`.
  - Triển khai `AudioManager.cs` theo mẫu chuẩn Game Feel: Âm thanh nổ đĩa (`PlayExplosionSound`) được tích hợp thuật toán **Pitch Shifting** (Tăng dần cao độ nếu nổ dây chuyền trong vòng 1.5s). Thêm các SFX phản hồi thao tác UI (`PlayPlaceSound` và `PlayErrorSound` khi đặt sai slot). Tất cả được cấu hình chạy nén ADPCM / Force To Mono tối ưu tuyệt đối cho Mobile.

- **[2026-06-01] Giải quyết Infinite Pull Loop (Vòng lặp hút vô tận):** Phát hiện lỗi 2 đĩa liên tục hút 1 loại bánh qua lại. Sửa triệt để bằng 2 luật: (1) **"Bloom Sort - Kẻ mạnh hút kẻ yếu"**: Một đĩa chỉ được hút nếu số lượng bánh loại đó LỚN HƠN HOẶC BẰNG lân cận. (2) **Focus Identity (Định danh tập trung)**: Đĩa CHỈ HÚT loại bánh mà nó đang sở hữu Đa Số Nhất. Việc này triệt tiêu hoàn toàn khả năng đĩa tự hút lại miếng rác mà nó vừa xả ra, tạo tính định hướng rõ ràng cho từng đĩa.
- **[2026-06-01] Push-Only Purge & Anti-Bounce (Xả rác 1 chiều & Chống dội):** Loại bỏ hoàn toàn cơ chế `TrySwapMinoritySlice` vì Swap không làm đĩa bớt đi miếng bánh nào, gây lặp vô tận. Thay bằng `TryPushMinoritySlice`: Khi đĩa bị Đầy nhưng chứa rác (Bật cờ `IsPurging`), nó BỊ CẤM hút, chỉ được Push rác sang đĩa lân cận còn trống. Áp dụng thêm luật **Anti-Bounce**: CẤM Push rác vào đĩa lân cận đang có 5/6 miếng (vì sẽ làm đĩa kia đầy, kích hoạt Purging và dội ngược lại rác cũ), trừ khi miếng rác đó giúp đĩa kia nổ. Điều này đảm bảo Game Flow không bao giờ bị Deadlock.
- **[2026-06-01] Priority Map & Hút luân chuyển:** `_cellsToProcess` được chuyển từ Queue sang List và sort theo `Priority` (giảm dần từ 9 về 0, lan truyền BFS từ điểm thả đĩa). Luật hút bánh cũng được cập nhật: Đĩa ưu tiên cao hơn (gần tâm chấn hơn) sẽ có lực hút mạnh hơn, ép bánh chảy dồn về tâm.
- **[2026-06-01] Batch Refill — Sinh đĩa theo batch (3 đĩa/lượt):** Thay đổi flow sinh đĩa pizza từ "sinh từng cái" sang "sinh cả batch khi khay trống hết" (giống Candy Crush / Triple Tile). TrayManager theo dõi `_slotPlates[]`, lắng nghe `InputManager.OnPlatePlaced` để đánh dấu slot trống. Khi cả 3 slot trống + FSM về `PlayingState` (merge/bloom xong hẳn) → `RefillTray()` sinh 3 đĩa mới cùng lúc. GameStateManager bổ sung `OnStateChanged` event (Observer Pattern) để TrayManager lắng nghe.
- **[2026-05-31] Vẽ lại Sơ đồ Logic Game (Game Flow Diagram):** Cập nhật `ARCHITECTURE.md` với sơ đồ mới từ `docs/Ve_so_do_game_banh_pizza.png`. Thay đổi so với phiên bản cũ: (1) Bổ sung event **"Đặt đĩa thất bại"** — Snap sai cũng phát event để chạy hoạt ảnh phản hồi lỗi; (2) Đổi tên node "Vị trí thả" → **"Vị trí thả Snap"** chính xác hơn; (3) Node J từ hình chữ nhật thành **diamond** (điều kiện rõ ràng); (4) Bổ sung loop **O → F** (đĩa nổ → reset Grid) cho chain combo; (5) Tách **"Hoạt ảnh"** và **"VFX"** thành 2 output độc lập; (6) Mapping event → output được làm rõ chính xác per-event (Đặt đĩa thành công → UI sinh đĩa mới).
- **Phase 3 - Parallel Pull & Drop Animation (2026-05-31):** Nâng cấp thuật toán Merge để kéo bánh đồng thời từ 4 hướng (1 miếng/hướng/lượt) giống cơ chế Bloom Sort. Sửa lỗi teleport bằng cách gọi `SetParent(transform, true)` để giữ nguyên World Position trước khi bay Bezier. Thêm Coroutine `AnimateToCell` (Quadratic Bezier 3D) vào `PizzaPlate.cs` để tạo hiệu ứng vòng cung nhô cao 0.8 unit khi snap đĩa vào ô lưới. Mọi thứ vận hành mượt mà với FSM `CheckingComboState` và `AnimatingState`.

## 2. Blockers

- **Scripts/UI/ đang trống:** Chưa có script UI nào được tạo (HUD, Score). Tiến độ hiện tại cần chuyển sang Phase 2 (Tweening, Merge Logic) rồi mới ghép UI.

## 3. Lessons Learned

- **Auto-Scale cho TrayManager (2026-05-29):** Đã bổ sung hàm `FitPlateToSlot()` trong `TrayManager` để tự động scale prefab đĩa pizza vừa vặn với `_slotSpacing` của khay (giải quyết blocker kích thước).
- **Trả nợ kỹ thuật Tuần 1 (2026-05-29):** Phát hiện và ĐÃ FIX thành công 2 Critical (GC trong GetComponent runtime + new alloc trong gameplay loop), 4 Warning (magic numbers, public field, RaycastAll đã được update sang NonAlloc), 3 Suggestion (thư mục thiếu, naming, scene đã chuẩn Architecture). Hệ thống đã đạt "Zero GC Alloc".
- **Zero-GC Raycast Pattern:** Việc kết hợp `Physics.RaycastNonAlloc` với mảng tĩnh đệm và hàm `Array.Sort` qua một `struct IComparer<RaycastHit>` là mẫu chuẩn tối ưu bắt buộc tái sử dụng cho các tính năng raycast trong tương lai.
- **Data-Driven approach:** JSON config cho level hoạt động tốt, sẽ mở rộng thêm ma trận đĩa bánh cho Tuần 2.
- **MaterialPropertyBlock cho Checkerboard (2026-05-29):** Dùng MaterialPropertyBlock thay đổi `_Color`/`_BaseColor` trên mỗi Renderer thay vì tạo Material instance. Pattern này đã chứng minh giữ được GPU Instancing và Zero GC. Áp dụng lại cho mọi hệ thống cần thay đổi màu runtime.
- **SetParent Scale Compensation (2026-05-30):** Khi sinh Prefab con ghép vào Parent đã bị thay đổi Scale (ví dụ đĩa bị ép Scale), lệnh `SetParent` mặc định của Unity tự động thay đổi `localScale` để bù trừ. Cần dùng `SetParent(parent, false)` và tái lập `localScale = Vector3.one` để khắc phục lỗi con phóng to/nhỏ bất thường.
- **Data-Driven Randomization (2026-05-30):** Tránh Random mù quáng. Dùng mảng `sliceCountProbabilities` lưu tỉ lệ phần trăm và thuật toán Roulette Wheel Selection để điều khiển tỉ lệ sinh miếng bánh trực tiếp từ file JSON Level Data.
