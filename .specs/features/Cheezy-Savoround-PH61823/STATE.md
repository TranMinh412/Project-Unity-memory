# STATE: Cheezy Savoround

**Last Updated:** 2026-06-15

> **AI CONTEXT:** This document tracks the current state, active decisions, and known issues. Update it when major architectural decisions are made.

---

## 1. Recent Decisions (ADR)

- **[2026-06-17] Project Delivered & Finalized:**
  - Hoàn tất mọi khâu tối ưu hóa: Zero-GC đạt chuẩn (<100B Alloc/frame), CPU frame time < 10ms, Draw Calls đạt 41 (< 50 yêu cầu công ty).
  - Khóa vSyncCount = 0 để mở khóa 60 FPS thực tế trên Android.
  - Sơ đồ hệ thống, luồng chạy chính và Video Gameplay đã lên sóng GitHub và báo cáo Mentor. Đóng máy dự án thành công rực rỡ!

- **[2026-06-16] Booster System Architecture & Zero-GC Polish:**
  - **Data-Driven & Pattern Matching:** Triển khai thành công 4 loại Booster (Cutter, Sauce, Move, Trash) theo đúng sát thiết kế GDD.
    - `Cutter`: Đĩa mục tiêu thiếu -> Tìm ô kề cạnh trống -> Spawn đĩa mới bù đắp, ép nổ.
    - `Sauce`: Đĩa mục tiêu chờ -> Spawn trực tiếp miếng đa số vào đĩa để đạt 6/6 -> Kích nổ.
    - `Move`: Kéo nhấc 1 đĩa lên và đặt vào ô trống (tái sử dụng InputManager drag-and-drop).
    - `Trash`: Nhấn đĩa bất kì để xoá.
  - **Zero-GC & Hot-Path Optimizations:**
    - Thay thế `FindObjectsByType` bằng static lists (Registry pattern) cho `GoldDisplay`.
    - Biến `HitDistanceComparer` thành `class` để chặn Boxing Alloc khi `.Sort()` Array trong Update.
    - Caching `Queue<GridCell>` và `HashSet<GridCell>` cho thuật toán BFS trong `CalculateLocalPriorities`, giải quyết triệt để memory allocation / leak khi chuỗi Cascade liên tục nổ.
  - **UI Toggle & Raycast Control:** Xây dựng hệ thống Toggle an toàn cho `BoostButton`, block hoàn toàn Drag mặc định của `InputManager` khi đang ở trạng thái chọn mục tiêu (TrashTarget/MoveSource). Tự động trả về `Idle` khi người chơi huỷ.

- **[2026-06-15] Robust Time-Validation (Anti-Cheat) cho Daily Reward:**
  - **Server-Time Provider:** Khởi tạo `ServerTimeProvider` sử dụng `UnityWebRequest.Head("https://www.google.com")` để lấy Header `Date`. Kỹ thuật này giúp lấy chuẩn giờ quốc tế mà không tốn chi phí duy trì backend riêng.
  - **Offline Offset Validation:** Giải quyết triệt để bài toán "Game Offline vẫn chống được Hack giờ". `TimeValidationData` lưu lại cả `LastVerifiedServerTicks` và `LastVerifiedDeviceTicks` khi có mạng. Khi rớt mạng, hệ thống tự ước lượng giờ thật dựa vào chênh lệch (Offset) của giờ máy tính. Cảnh báo và phạt reset chuỗi 7 ngày nếu đếm được `SuspiciousJumpCount >= 3`.

- **[2026-06-14] Dynamic Board State & Progression Persistence:**
  - **Lưu trạng thái LevelProgressData:** Tích hợp `occupiedCells`, `traySlots`, và `currentScore` vào JSON để phục hồi hoàn toàn mạch chơi khi thoát game giữa chừng. 
  - **Tối ưu Khôi phục Tray (Zero-GC):** LevelManager truyền cờ `refillImmediately = false` cho `TrayManager` nếu phát hiện có file Save, giúp Tray không sinh 3 đĩa mới dư thừa rồi vứt đi, loại bỏ Memory Allocation vô ích lúc khởi động.
  - **Mobile Application Lifecycle Hooks:** Tích hợp `OnApplicationPause(true)` và `OnApplicationQuit` vào `GameStateManager` để "bắt" khoảnh khắc người chơi vuốt tắt game hoặc chuyển đa nhiệm trên điện thoại, đồng thời đổ dữ liệu từ RAM xuống Disk khẩn cấp, bảo toàn tiền và tiến độ ván cờ.

- **[2026-06-14] Gold Economy & Dynamic Skin Sync:**
  - **Kinh tế Vàng (Gold Economy):** Hệ thống tách biệt logic, `SaveLoadManager` hứng Event `OnPlateExploded` để cộng dồn vàng vào RAM mà KHÔNG I/O lưu file liên tục (chỉ lưu file khi chuyển Scene hoặc Pause app).
  - **Dynamic Skin Update:** Khởi tạo `GameEvents.OnSkinChanged`. Khi mua và trang bị từ Shop, mọi đĩa Pizza đang tồn tại trên Lưới và Khay tự động nghe sự kiện này và đổi Material ngay lập tức, không cần tải lại Scene.

- **[2026-06-14] Transit Station Logic (Priority 9) & Memory Leak Fix:**
  - **Epicenter Relay Mechanics (Trạm trung chuyển):** Đã hoàn thiện logic đĩa tâm chấn (Priority 9) trong `GridManager`. Đổi điều kiện `FindSelfExplodeType` từ cứng nhắc (>= 6) sang kiểm tra linh hoạt `availableFromNeighbors >= needed`, giúp lưới lớn (như 4x6) không bị nghẽn ở Phase 1. Nới lỏng kiểm tra Anti-Bounce ở Phase 2, cho phép đĩa epicenter tạm thời đầy 6/6 (dù dơ) để làm trạm trung chuyển bánh giữa các lân cận.
  - **Multi-Epicenter Cache Fix:** Nhằm giải quyết triệt để lỗi ghi đè dữ liệu khi có >2 tâm chấn nổ combo đồng thời, hệ thống `_pendingRelay` (cache relay 2-bước) được đổi sang cấu trúc `Dictionary<GridCell, (int, GridCell)>`. Mỗi epicenter sẽ giữ riêng thông tin trung chuyển của mình, bảo đảm không còn bị lỗi side-effect.
  - **Tie-breaker FIFO Queue:** Tích hợp bộ đếm `_enqueueOrder` trong `_cellsToProcess` để duy trì trật tự FIFO cho các cell có cùng Priority. Điều này giúp các đĩa ưu tiên hoàn thiện mạch combo trung chuyển của nó trước khi bị đĩa khác chen ngang.
  - **DOTween Memory Leak Fix:** Thêm lệnh `CancelAllTweens()` cho `BezierTween` và gọi `DOKill()` trên mọi thành phần Transform của `PizzaPlate` lẫn `PizzaSliceVisual` khi thực hiện `ClearGrid()` hoặc `ClearTray()`. Chặn đứng `MissingReferenceException` lúc tái tạo lưới qua các level khác nhau.

- **[2026-06-14] Combo Scoring Logic & UI Separation:**
  - **Kiến trúc Tách biệt Logic - View:** Refactor hoàn toàn hàm `EvaluateTurnCombo()` trong `GridManager`. Logic tính điểm (thực thi Event `TriggerComboAchieved` và `TriggerPlateExploded`) diễn ra NGAY LẬP TỨC ở cuối chuỗi chain, không bị kẹt hay delay. Ngược lại, phần View (hiển thị Text UI) được đẩy vào `DOVirtual.DelayedCall(_comboRevealDelay)` để canh timing chuẩn xác, chờ chữ `+100` của đĩa cuối cùng mờ đi mới ném chữ `Combo xN!` lên giữa màn hình. Điều này đảm bảo UI không thể làm nghẽn Data Game State.
  - **Tránh Lỗi Closure Capture (Lambda Race Condition):** Sửa lỗi hiển thị sai Combo (nhảy về x0) khi người chơi thả đĩa quá nhanh trong lúc delay text. Các biến trạng thái (`_explosionCountThisTurn` và `centerPos`) được sao chép ra local variable trước khi đưa vào Lambda closure để đảm bảo giữ đúng giá trị của lượt đánh đó.
  - **Tối ưu Hiển thị Feedback:** Đã gỡ bỏ dòng text `+Bonus`, chỉ giữ lại `Combo xN!` để giữ màn hình gọn gàng theo chuẩn Hyper-Casual, nhưng số điểm Bonus thực tế vẫn được cộng ngầm cực kì chính xác (`Base * Số Đĩa * 1.5 - Base đã cộng trước đó`).

- **[2026-06-14] Shop Architecture & Zero-GC Polish:**
  - **Data-Driven ShopConfig:** Hoàn tất việc chuyển dời toàn bộ cấu hình (Skins, Boosters, CoinPacks) vào một ScriptableObject duy nhất (`ShopConfig.cs`). Tinh giản `ShopCategory`, dọn dẹp Inspector. ShopManager linh hoạt đọc thông tin vật phẩm theo thời gian thực.
  - **Zero-GC UI Text:** Refactor 100% `ShopManager.cs`, đổi `.text =` sang `.SetText("{0}", value)`. Xóa bỏ hoàn toàn `.ToString()` và toán tử cộng chuỗi để chặn triệt để Memory Allocation trong lúc cập nhật UI.
  - **Dynamic Save Data Expansion:** Nâng cấp tính năng lưu Boosters. Tự động kiểm tra và mở rộng mảng `BoostersOwned` (`List.Add(0)`) đảm bảo tương thích mượt mà với các file Save cũ, chống lỗi IndexOutOfRange.

- **[2026-06-13] Shop UI & Skin System:**
  - **Shop Manager**: 
    - Đã triển khai thuật toán băng chuyền (Carousel) với cấu trúc Data-Driven `ShopCategory` giúp thay thế giao diện (Background, Nền vật phẩm, Bảng giá) tuỳ theo Tab (Boost, Coin, Skin) mà không sinh rác (Zero-GC).
    - Tích hợp thành công **3D Skin Preview**: Render trực tiếp mô hình `PizzaPlate` vào giữa giao diện Shop, tự động xoay và áp dụng Material (`Main_Texture`) từ `ShopConfig`.
      - *VFX Quyết định (Rotation)*: Cố tình sử dụng `Space.World` kết hợp độ nghiêng `_modelTiltAngle` (Thay vì `Space.Self`). Dù điều này tạo ra hiệu ứng hơi lắc nhẹ (UFO wobble) do Pivot lệch tâm, nhưng về mặt thị giác trong UI, kiểu xoay này tạo cảm giác không gian 3D sống động và sinh động hơn là xoay phẳng tuyệt đối. Đã phơi bày cấu hình (`_modelScale`, `_modelRotationSpeed`, `_modelTiltAngle`) ra Inspector (Zero-Hardcoding).
    - Hoàn thiện logic **Mua & Trang Bị Skin**: Tự động liên kết với dữ liệu lưu trữ `PlayerData` (`Gold`, `UnlockedSkins`, `CurrentSkinId`), cập nhật giá và trạng thái theo thời gian thực.
  - **Raycast Management**: 
    - Khóa vật lý (Raycast Target) cho các thành phần nền và UI tĩnh để tránh lỗi kẹt input trên Mobile. Mọi tương tác chạm chỉ kích hoạt khi Game FSM chuyển sang `PlayingState`. Khởi tạo `UIManager.cs` với hàm `StartGame()` để gắn vào nút Start. Đồng thời áp dụng luật cấm Raycast xuyên UI (`EventSystem.current.IsPointerOverGameObject()`) vào `InputManager` để triệt tiêu hoàn toàn lỗi đè hình (click lọt lưới).

- **[2026-06-13] Configurable Target FPS:** Đã thêm cấu hình `TargetFPS` vào `GameSettings` (lưu trong `playerdata.json`). `SaveLoadManager` sẽ tự động đọc và khóa FPS (`Application.targetFrameRate`) theo thông số này ngay khi khởi động, tuân thủ Data-Driven.

- **[2026-06-13] Hoàn thiện Kiến trúc đa Canvas UI (Zero-GC & Responsive):** Chốt phương án chia để trị cho giao diện: Sử dụng Canvas `Screen Space - Camera` (Plane Distance 100) để vẽ nền 2D gạch lót dưới đáy môi trường 3D. Đồng thời thiết lập `HUDCanvas` độc lập với chế độ `Screen Space - Overlay` để đè hệ thống nút bấm (TopBar, Boosters, GameOver) lên trên cùng, khắc phục lỗi đè hình. UI Shop (Lobby) chốt thiết kế Carousel 1 Khung dùng chung, quản lý chuyển Tab bằng thay đổi màu sắc (Color Tint) và TextMeshPro để tuân thủ tuyệt đối Zero-GC. Bảng GameOver bật `Raycast Target` làm "Bức tường khiên" chặn click lọt xuống các tương tác 3D.
- **[2026-06-12] Khởi tạo Giao diện (UI) và Màn hình chính:** Hoàn thành việc dựng Canvas cho màn hình Lobby (Starter, Help, Achievement, Shop). Thiết lập phân cấp Panel ẩn/hiện chuẩn xác để chuẩn bị cho UIManager. Khắc phục lỗi hiển thị Sprite UI bằng cách điều chỉnh `.meta` và áp dụng kỹ thuật "Trim" cắt phần trong suốt của Sprite bằng `Sprite Editor` (Multiple Mode) nhằm giảm thiểu GPU Overdraw cho nền tảng Mobile. Sử dụng đồng bộ `Vertical/Horizontal Layout Group` và `Prefab` cho các item danh sách (Thành tựu) để đảm bảo đồng nhất thiết kế.
- **[2026-06-12] Tối ưu hóa UI Text (Zero-GC):** Thống nhất quy tắc bắt buộc sử dụng `SetText("{0}/{1}", val1, val2)` của thư viện `TMPro` khi cập nhật chữ số biến động (như thanh tiến trình thành tựu) để tránh phát sinh chuỗi (string allocation), duy trì luật Zero-GC của dự án. Áp dụng `Image (Filled)` thay cho `Slider` để làm thanh tiến trình, loại bỏ hoàn toàn lỗi hiển thị viền đỏ rác.
- **[2026-06-12] Khởi tạo Hệ thống Save/Load JSON:** Đã tạo `PlayerData.cs` và Singleton `SaveLoadManager.cs`. Cấu trúc JSON hỗ trợ lưu Vàng (Gold), Skin đã mở khóa (UnlockedSkins), Skin hiện tại, thời gian nhận Daily Reward (UTC) và Tiến trình thành tựu (AchievementSaveData). Singleton `SaveLoadManager` sẽ dùng `DontDestroyOnLoad` để tự duy trì, không bị hủy giữa các scene.
- **[2026-06-12] Zero-GC Game Over Detection:** Triển khai hàm `CheckGameOver()` trong `GridManager`. Game Over chỉ xảy ra khi: (1) Lưới hoàn toàn đầy và (2) Không còn bất kỳ bước merge nào hợp lệ giữa các đĩa. Quá trình quét duyệt 4 hướng sử dụng `_gameOverTypeBuffer` được khởi tạo từ trước để tái sử dụng, đảm bảo Zero-GC allocation trong lúc quét, bảo vệ frame rate trên Mobile. Hàm này được trigger qua event `OnRefillComplete` của `TrayManager`.
- **[2026-06-12] Tách biệt Local BFS & Giữ nguyên hàng đợi (Queue):** Hàm `CalculatePriorities()` cũ reset toàn bộ lưới về Priority 0 trước khi chạy BFS, gây phá vỡ trật tự của các đĩa đang chờ trong `_cellsToProcess` khi có combo liên hoàn. Đã tạo thêm `CalculateLocalPriorities(GridCell epicenter)` chỉ thực hiện quét 1 lớp lân cận và **chỉ nâng** (không hạ) Priority lên 8. Điều này giữ nguyên các đĩa ưu tiên cũ trong hàng đợi, đảm bảo Combo Cascade (nổ dây chuyền) diễn ra chính xác.
- **[2026-06-12] Per-slot Refill (Sinh đĩa tức thì):** Thay đổi logic của `TrayManager`: Không chờ cả 3 slot trống mới sinh, mà sinh đĩa mới **ngay lập tức** cho slot nào vừa trống sau khi người chơi đặt đĩa thành công (như cơ chế của các game casual hiện đại). Khay sẽ báo event `OnRefillComplete` khi tất cả các slot trống đều đã nhận được đĩa mới.
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

- **Chưa có (Hoặc chờ test):** Các module Logic UI, Store, Save/Load và Board State đều đã hoàn tất và kết nối thành công. Hiện tại không có blocker nghiêm trọng.

## 3. Lessons Learned

- **Auto-Scale cho TrayManager (2026-05-29):** Đã bổ sung hàm `FitPlateToSlot()` trong `TrayManager` để tự động scale prefab đĩa pizza vừa vặn với `_slotSpacing` của khay (giải quyết blocker kích thước).
- **Trả nợ kỹ thuật Tuần 1 (2026-05-29):** Phát hiện và ĐÃ FIX thành công 2 Critical (GC trong GetComponent runtime + new alloc trong gameplay loop), 4 Warning (magic numbers, public field, RaycastAll đã được update sang NonAlloc), 3 Suggestion (thư mục thiếu, naming, scene đã chuẩn Architecture). Hệ thống đã đạt "Zero GC Alloc".
- **Zero-GC Raycast Pattern:** Việc kết hợp `Physics.RaycastNonAlloc` với mảng tĩnh đệm và hàm `Array.Sort` qua một `struct IComparer<RaycastHit>` là mẫu chuẩn tối ưu bắt buộc tái sử dụng cho các tính năng raycast trong tương lai.
- **Data-Driven approach:** JSON config cho level hoạt động tốt, sẽ mở rộng thêm ma trận đĩa bánh cho Tuần 2.
- **MaterialPropertyBlock cho Checkerboard (2026-05-29):** Dùng MaterialPropertyBlock thay đổi `_Color`/`_BaseColor` trên mỗi Renderer thay vì tạo Material instance. Pattern này đã chứng minh giữ được GPU Instancing và Zero GC. Áp dụng lại cho mọi hệ thống cần thay đổi màu runtime.
- **SetParent Scale Compensation (2026-05-30):** Khi sinh Prefab con ghép vào Parent đã bị thay đổi Scale (ví dụ đĩa bị ép Scale), lệnh `SetParent` mặc định của Unity tự động thay đổi `localScale` để bù trừ. Cần dùng `SetParent(parent, false)` và tái lập `localScale = Vector3.one` để khắc phục lỗi con phóng to/nhỏ bất thường.
- **Data-Driven Randomization (2026-05-30):** Tránh Random mù quáng. Dùng mảng `sliceCountProbabilities` lưu tỉ lệ phần trăm và thuật toán Roulette Wheel Selection để điều khiển tỉ lệ sinh miếng bánh trực tiếp từ file JSON Level Data.
