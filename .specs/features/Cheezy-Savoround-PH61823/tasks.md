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
# FEATURE: Tuần 2 - Tích Hợp Asset 3D, Tweening & Hoàn Thiện Core Loop

**Status:** 🟢 DONE
**Thời gian:** 25/05 - 12/06/2026

## 1. Feature Specification

Tuần 2 tập trung vào hai mảng song song:

- **Trả nợ kỹ thuật (Tech Debt):** Sửa toàn bộ các issue từ Code Review Tuần 1 (2 Critical, 4 Warning, 3 Suggestion) để codebase sạch trước khi xây tầng mới.
- **Tính năng mới:** Import asset 3D thực tế, viết logic chuyển động Bezier cho miếng pizza, triển khai FSM quản lý game states, Object Pooling, Game Over detection, và đánh bóng Game Feel.

## 2. Technical Constraints

- Tuân thủ thiết kế **Zero GC Alloc** trong Update (không tạo rác).
- FSM **bắt buộc** dùng State class riêng biệt, cấm dùng nested booleans.
- Object Pooling **bắt buộc** cho mọi vật thể sinh/hủy liên tục (pizza slices, VFX, text).
- Mọi magic number phải được trích xuất ra `[SerializeField]` hoặc `const`.
- Commit Git tối thiểu 3 lần/tuần.

## 3. Execution Plan (Tasks)

---

### PHASE 0: Trả Nợ Kỹ Thuật Tuần 1 (Code Review Fixes)

> ⚠️ **BẮT BUỘC hoàn thành Phase 0 trước khi sang Phase 1.** Các issue Critical sẽ gây GC spike nghiêm trọng khi triển khai combo chain ở Phase 1.

- [x] **Task 0.1: [🔴 Critical] Loại bỏ `GetComponent` runtime trong GridManager**
  - Đổi `Dictionary<Vector2Int, GameObject> _gridCells` → `Dictionary<Vector2Int, GridCell>`.
  - Cập nhật hàm `GenerateGrid()` để lưu trực tiếp `GridCell` component vào Dictionary thay vì `GameObject`.
  - Loại bỏ hoàn toàn `GetComponent<GridCell>()` khỏi hàm `GetCell()`.
  - **Verification:** `GetCell()` trả về `GridCell` trực tiếp từ Dictionary, không còn dòng `GetComponent` nào trong `GridManager.cs`.
  - **File:** `Scripts/Core/GridManager.cs`

- [x] **Task 0.2: [🔴 Critical] Cache mảng `directions` và `matchingCells` trong GridManager**
  - Chuyển `Vector2Int[] directions` thành `private static readonly Vector2Int[]` ở cấp class.
  - Chuyển `List<GridCell> matchingCells` thành `private readonly List<GridCell>` field ở cấp class, gọi `.Clear()` đầu hàm `CheckAdjacentCells()`.
  - **Verification:** Không còn `new Vector2Int[]` hoặc `new List<GridCell>()` bên trong body hàm `CheckAdjacentCells`.
  - **File:** `Scripts/Core/GridManager.cs`

- [x] **Task 0.3: [🟡 Warning] Trích xuất Magic Numbers ra SerializeField/const**
  - `PizzaPlate.cs`: `y = 1f` → `[SerializeField] private float _spawnHeight = 1f`, `+ 0.5f` → `[SerializeField] private float _pickUpOffset = 0.5f`.
  - `GridCell.cs`: `+ 0.2f` → `[SerializeField] private float _snapOffsetY = 0.2f`.
  - `InputManager.cs`: `1.0f` → `[SerializeField] private float _dragHeight = 1.0f`, debug values (`5f`, `2f`) → `private const float DEBUG_RAY_LENGTH = 5f`, `private const float DEBUG_RAY_DURATION = 2f`.
  - `LevelGenerator.cs`: `3` → `private const int DEFAULT_HOLD_SLOT_COUNT = 3`.
  - **Verification:** Grep `Scripts/` cho các literal number trực tiếp → kết quả trống (trừ `0`, `1` dạng index).
  - **Files:** `PizzaPlate.cs`, `GridCell.cs`, `InputManager.cs`, `LevelGenerator.cs`

- [x] **Task 0.4: [🟡 Warning] Encapsulate `PizzaType` field trong PizzaPlate**
  - Chuyển `public PizzaType Type = PizzaType.Pho_mai` → `[SerializeField] private PizzaType _type = PizzaType.Pho_mai` + public property `public PizzaType Type => _type`.
  - Cập nhật tất cả references bên ngoài nếu cần (kiểm tra `GridManager.CheckAdjacentCells`).
  - **Verification:** Không còn public field trực tiếp trên `PizzaPlate`. Property `Type` chỉ có getter.
  - **File:** `Scripts/Gameplay/PizzaPlate.cs`

- [x] **Task 0.5: [🟡 Warning] Tối ưu Raycast — Sort + NonAlloc**
  - Thay `Physics.RaycastAll()` bằng `Physics.RaycastNonAlloc()` + buffer cache (`private RaycastHit[] _hitBuffer = new RaycastHit[10]`).
  - Sort kết quả theo `distance` trước khi duyệt (`System.Array.Sort`).
  - Thêm `LayerMask` cho Grid layer để thu hẹp phạm vi quét.
  - **Verification:** Không còn `Physics.RaycastAll` trong codebase. Buffer `_hitBuffer` là field cấp class.
  - **File:** `Scripts/Core/InputManager.cs`

- [x] **Task 0.6: [🟢 Suggestion] Dọn dẹp cấu trúc dự án**
  - Tạo thư mục `Scripts/Utils/` và `Scripts/UI/` (đúng `ARCHITECTURE.md §5`).
  - Đổi tên Prefab: `CubTest1.prefab` → `GridCellDark.prefab`, `Dia_test.prefab` → `PizzaPlateTest.prefab` (PascalCase).
  - Đổi tên Scene: `SampleScene` → `GameplayScene`.
  - **Verification:** Cấu trúc thư mục khớp với `ARCHITECTURE.md §5`. Không còn Prefab dùng snake_case hoặc tiếng Việt.

---

### PHASE 1: Import Asset 3D & Thay Thế Placeholder (Ngày 1-2)

- [x] **Task 1.1: Import và tổ chức Asset 3D**
  - Tải bộ asset 3D (đĩa pizza, miếng pizza) từ tài nguyên Google Drive của Mentor.
  - Tổ chức vào `Assets/Models/`. Import vào cứ thế mà dùng
  - Tạo Materials cơ bản cho mỗi loại pizza (Prefab-ready).
  - **Verification:** Tất cả model nằm trong `Assets/Models/`, material trong `Assets/Materials/`.
  -

- [x] **Task 1.2: Tạo Prefab hệ thống đĩa pizza thực tế**
  - Tạo Prefab `PizzaPlate` mới với Mesh 3D thực tế thay thế Cube thô.
  - Tích hợp các slot cho miếng pizza (6 vị trí) trên đĩa (có thể dùng empty Transform children).
  - Gắn script `PizzaPlate.cs` đã có, đảm bảo logic cũ vẫn hoạt động sau khi đổi Mesh.
  - **Verification:** Kéo thả đĩa pizza mới vẫn Snap đúng vào lưới. Visual không còn Cube.

- [x] **Task 1.3: Lập trình Bezier Tweening cho miếng pizza**
  - Viết class `BezierTween` trong `Scripts/Utils/` để tính đường cong Bezier 3D (Quadratic hoặc Cubic).
  - Triển khai logic: Khi quét 4 hướng phát hiện miếng pizza cùng loại → miếng pizza bay từ đĩa A sang đĩa B theo đường cong Bezier.
  - Sử dụng Coroutine hoặc Update-based interpolation (tránh GC).
  - **Verification:** Miếng pizza bay mượt theo đường cong (có điểm cao nhất arc), không teleport. Không có GC alloc trong quá trình bay.
  - **File:** `Scripts/Utils/BezierTween.cs` (mới)

---

### PHASE 2: FSM Game States (Ngày 3)

- [x] **Task 2.1: Thiết kế và triển khai FSM**
  - Tạo interface `IGameState` với các method: `Enter()`, `Execute()`, `Exit()`.
  - Tạo `GameStateManager` (hoặc tích hợp vào `GameManager`) quản lý FSM.
  - Triển khai các State class riêng biệt:
    - `PlayingState` — Cho phép kéo thả, nhận input.
    - `AnimatingState` — Lock input, chờ Bezier tween hoàn thành.
    - `CheckingComboState` — Chạy thuật toán quét + combo chain (BFS/Flood Fill).
    - `GameOverState` — Hiển thị UI Game Over, lock toàn bộ interaction.
  - **Verification:** Mỗi State là một class riêng biệt implement `IGameState`. Không có nested booleans (`isMoving && !isMerging`). `InputManager` chỉ xử lý input khi state == `PlayingState`.
  - **Files:** `Scripts/Core/IGameState.cs`, `Scripts/Core/GameStateManager.cs`, `Scripts/Core/States/PlayingState.cs`, `AnimatingState.cs`, `CheckingComboState.cs`, `GameOverState.cs`

---

### PHASE 3: Logic Gameplay Cốt Lõi (Ngày 4)

- [x] **Task 3.1: Nâng cấp thuật toán quét → BFS/Flood Fill + Merge**
  - Mở rộng `CheckAdjacentCells` để hỗ trợ combo chain (đĩa đặt xuống hút các miếng bánh cùng loại từ các đĩa lân cận).
  - Triển khai logic merge: Chuyển miếng pizza từ các đĩa lân cận sang đĩa trung tâm cho đến khi đủ 6 miếng → nổ đĩa → giải phóng ô.
  - Khi đĩa nổ → kích hoạt re-check lân cận (Combo Cascade).
  - **Verification:** Đảm bảo khi hút bánh, số lượng và type được chuyển đúng, và gọi `BezierTween` mượt mà, sau đó FSM chuyển đúng trạng thái. Đĩa nổ khi đủ 6 miếng cùng loại, và kích hoạt hút tiếp ở các đĩa xung quanh.
  - **File:** `Scripts/Core/GridManager.cs` (nâng cấp)
  - thêm: tạo chuổi ưu tiên theo kiểu từ thấp đến cao là 0 -> 9 rồi thì phải cho nó chạy lần lượt từ 9 -> 0 chứ.
  - Sao có cảm giác đặt đĩa này xong đĩa khác lại di chuyển pizza vậy?
  - Bàn lưới/grid bị lấp đầy nhưng không thể để thêm được nữa thì game không chuyển sang trạng thái game over vì do phải cả lưới trên bàn và khay chứa phải đầy

- [x] **Task 3.2: Triển khai Game Over Detection**
  - Code logic kiểm tra điều kiện thua: Tất cả ô Grid **và** tất cả Hold Slots đều bị chiếm, **và** không còn bất kỳ merge hợp lệ nào.
  - Khi phát hiện Game Over → chuyển FSM sang `GameOverState`.
  - Phát event `OnGameOver` để UI/Audio phản ứng.
  - **Verification:** Lấp đầy lưới + khay → game chuyển sang GameOverState, input bị lock, log "Game Over".
  - **File:** `Scripts/Core/GridManager.cs` hoặc `GameStateManager.cs`

- [x] **Task 3.3: Thiết lập Object Pooling**
  - Tạo class `ObjectPool<T>` generic trong `Scripts/Utils/ObjectPool.cs`.
  - Pool các đối tượng: Miếng pizza bay (Bezier), VFX Particle nổ đĩa, Text điểm số bay.
  - Tích hợp vào flow: Khi cần spawn → lấy từ pool. Khi hoàn thành → trả lại pool.
  - **Verification:** Không còn `Instantiate()`/`Destroy()` trong gameplay loop. Profiler không hiển thị GC spike khi nổ combo liên tục.
  - **File:** `Scripts/Utils/ObjectPool.cs` (mới)

---

### PHASE 4: Game Feel & Polish (Ngày 6-7)

- [x] **Task 4.1: Hiệu ứng Squash/Stretch khi đặt đĩa**
  - Khi đĩa Snap vào lưới → apply scale animation (squash down → stretch up → settle).
  - Sử dụng `AnimationCurve` (SerializeField) để control easing.
  - **Verification:** Đặt đĩa xuống có hiệu ứng "dẻo" rõ ràng, không giật.
  - **File:** `Scripts/Gameplay/PizzaPlate.cs` hoặc `Scripts/Utils/TweenEffects.cs`

- [x] **Task 4.2: Hiệu ứng Shake khi đặt sai ô + Ghost Mesh preview**
  - Khi thả đĩa vào ô đã chiếm → rung lắc đĩa (camera hoặc đĩa shake) rồi trả về slot cũ.
  - Hiển thị Ghost Mesh (bản sao mờ/trong suốt) tại ô lưới gần nhất khi đang kéo đĩa.
  - **Verification:** Kéo đĩa qua lưới → thấy bóng mờ preview ở ô gần nhất. Thả vào ô có đĩa → rung lắc + bay về.
  - **File:** `Scripts/Core/InputManager.cs`, `Scripts/Gameplay/GhostMesh.cs` (mới)

- [x] **Task 4.3: Âm thanh & VFX phản hồi**
  - Tạo `AudioManager` trong `Scripts/Core/` quản lý SFX (đặt đĩa, nổ đĩa, combo cascade).
  - Tích hợp Pitch Shift: Mỗi lần nổ liên tục trong cùng combo → pitch tăng dần.
  - Thêm VFX Particle cơ bản cho nổ đĩa (lấy từ Object Pool).
  - Cấu hình VFX Data-Driven tự scale theo kích thước bàn cờ.
  - **Verification:** Đặt đĩa có SFX. Combo nổ liên tục → âm thanh tăng pitch rõ rệt. VFX xuất hiện và tự trả về pool.
  - **File:** `Scripts/Core/AudioManager.cs` (mới)

---
# FEATURE: Tuần 3 - Tích Hợp Meta Features & Save/Load Dữ Liệu

**Status:** 🟡 IN PROGRESS
**Thời gian:** 12/06 - 19/06/2026 (Thực tế)

## 1. Feature Specification

Tuần 3 tập trung vào việc xây dựng các hệ thống Meta Game xung quanh Core Loop đã hoàn thiện ở Tuần 2:

- **Save/Load System:** Lưu trữ dữ liệu người dùng (Vàng, Skin, Tiến độ) bằng JSON.
- **UI Shop:** Cửa hàng mua skin bằng vàng (đọc từ cấu hình JSON) và thay đổi ngoại hình đĩa/bánh.
- **Daily Rewards:** Điểm danh nhận thưởng hàng ngày, chống hack giờ bằng UTC timestamp.
- **Achievement System:** Hệ thống thành tựu hướng sự kiện (Event-Driven), độc lập hoàn toàn với logic Gameplay.

## 2. Technical Constraints

- **Save/Load:** Sử dụng `System.IO.File` và `JsonUtility` (hoặc NewtonSoft.Json nếu có) để ghi/đọc JSON tại `Application.persistentDataPath`.
- **Bảo mật Daily Reward:** Phải dùng thời gian UTC (`DateTime.UtcNow`) lưu xuống file, tuyệt đối không dùng giờ Local của máy tính người chơi.
- **Event-Driven Architecture (Achievement):** Các script xử lý Thành Tựu KHÔNG ĐƯỢC phép reference trực tiếp đến `GridManager` hay `PizzaPlate`. Bắt buộc phải lắng nghe qua `C# event` (Action).
- **Decoupling (Shop):** UI Shop không được can thiệp logic gameplay. Khi đổi skin, hệ thống lưu ID Skin vào SaveData, `PizzaPlate` tự động đọc ID đó khi khởi tạo.

## 3. Execution Plan (Tasks)

---

### PHASE 1: Hệ thống Save/Load JSON (Ngày 1 - 2)

- [x] **Task 1.1: Thiết kế Data Models**
  - Tạo class `PlayerData.cs`: chứa `int Gold`, `List<string> UnlockedSkins`, `string CurrentSkinId`, `long LastDailyRewardTime`, `Dictionary<string, int> AchievementProgress`.
  - Tạo class `GameSettings.cs` (nếu cần cho setting âm thanh/nhạc).
  - **File:** `Scripts/Data/PlayerData.cs`

- [x] **Task 1.2: Triển khai SaveLoadManager**
  - Tạo Singleton `SaveLoadManager` xử lý việc mã hóa (tùy chọn) và ghi file `playerdata.json` xuống đĩa.
  - Viết hàm `Save()`, `Load()`, `ResetData()`.
  - Gọi `Load()` ở Awake của GameManager hoặc Bootstrap scene.
  - **Hoàn thiện (14/06/2026):**
    - Bổ sung cơ chế lưu State Bàn cờ (Grid & Tray) vào `LevelProgressData`. Lưu cả điểm số tiến trình (LevelProgressUI).
    - Tích hợp vòng lặp Game Economy (Cộng vàng khi nổ đĩa thông qua event `OnPlateExploded`).
    - Tích hợp hook tự động save an toàn cho Mobile qua `OnApplicationPause` và `OnApplicationQuit` trên `GameStateManager`.
  - **Verification:** Chạy game, đặt đĩa, thay đổi vàng, vuốt tắt màn hình/tắt game mở lại -> Bàn cờ và số vàng được giữ nguyên.
  - **File:** `Scripts/Core/SaveLoadManager.cs`

---

### PHASE 2: Cấu hình và UI Shop (Ngày 3)

- [x] **Task 2.1: Tạo ScriptableObject hoặc JSON Cấu hình Shop**
  - Định nghĩa danh sách vật phẩm: ID, Tên, Giá vàng, Prefab Mesh (hoặc Material).
  - **File:** `Scripts/Data/ShopConfig.cs` (ScriptableObject)

- [x] **Task 2.2: Thiết kế Visual/Layout UI (Lobby, Shop, HUD Ingame, Game Over)**
  - Chia 2 tầng Canvas (Camera cho nền lót sàn, Overlay cho Nút bấm/HUD) khắc phục lỗi đè 3D.
  - Setup UI Shop dạng Carousel (1 Khung Item dùng chung, đổi mảng dữ liệu qua Tab). Tối ưu Tab không dùng ảnh riêng mà dùng Color (Gray/White).
  - Setup Game Over Overlay bật `Raycast Target` làm Dimmer chặn tương tác 3D.

- [x] **Task 2.3: Lập trình Logic UI Shop & UIManager**
  - Code `ShopManager.cs`: Đọc `ShopConfig`, chuyển dữ liệu Carousel, tự sinh các Dots (Pagination).
  - Logic mua: Check vàng -> Trừ vàng -> Lưu `UnlockedSkins`.
  - **Hoàn thiện (14/06/2026):** Tích hợp Data-Driven triệt để qua ScriptableObject, hỗ trợ lưu Boosters tự động cấp phát bộ nhớ, và refactor Zero-GC toàn bộ text.
  - **File:** `Scripts/UI/ShopManager.cs`

- [x] **Task 2.4: Tích hợp Skin vào Gameplay**
  - Khi `TrayManager` hoặc `PizzaPlate` sinh đĩa, đọc `CurrentSkinId` từ `PlayerData` để áp dụng đúng Mesh/Material mới.
  - **Hoàn thiện (14/06/2026):** Triển khai Dynamic Skin Sync - tự động thay đổi ngoại hình toàn bộ đĩa trên Grid và Tray theo thời gian thực khi người chơi mua/trang bị trong Shop (sử dụng `GameEvents.OnSkinChanged`).
  - **File:** `Scripts/Gameplay/PizzaPlate.cs` (cập nhật)

---

### PHASE 3: Daily Rewards (Ngày 4 - 5)

- [x] **Task 3.1: Logic tính toán thời gian UTC & Chống Gian Lận (Anti-Cheat)**
  - Tạo class `DailyRewardManager`.
  - Viết hàm check: So sánh `DateTime.UtcNow` với `LastDailyRewardTime` (được lưu dưới dạng binary hoặc tick).
  - Nếu khoảng cách qua ngày mới (hoặc > 24h) -> Kích hoạt nhận quà.
  - **Hoàn thiện (15/06/2026):** Triển khai kiến trúc Anti-Cheat mạnh mẽ bằng cách kết hợp Server-Time (Ping Header Google) và Offline Offset Validation (tính độ trễ qua `TimeValidationData`). Phát hiện và phạt reset chuỗi khi cố tình đổi giờ điện thoại.

- [x] **Task 3.2: Giao diện nhận thưởng**
  - Hiển thị UI nút "Nhận Quà", bấm vào cộng vàng -> Gọi `SaveLoadManager.Save()`.
  - **Hoàn thiện (15/06/2026):** Xây dựng Data-Driven qua `DailyRewardConfig`, hỗ trợ mixed rewards (Chest) và cập nhật UI Zero-GC bằng TextMeshPro. Đã sửa lỗi hiển thị ngày kế tiếp.
  - **Verification:** Đã test tắt mạng, đổi giờ Local đi lùi -> Console văng log cảnh báo và không cấp quyền nhận quà.

---

### PHASE 4: Achievement System (Ngày 6 - 7)

- [x] **Task 4.1: Khai báo hệ thống Event chung**
  - Tạo class tĩnh `GameEvents.cs` chứa các Action: `OnPizzaMerged(int count)`, `OnPlateExploded(int totalSlices)`, `OnLevelCompleted()`.
  - Sửa `GridManager` để phát event này mỗi khi nổ đĩa/merge.

- [x] **Task 4.2: AchievementManager lắng nghe Event**
  - `AchievementManager` đăng ký lắng nghe các event trên trong `OnEnable/OnDisable`.
  - Cập nhật tiến độ `AchievementProgress` tương ứng -> Khi đạt mốc -> Mở khóa -> Phát popup -> Lưu Save file.
  - **Verification:** Thấy thông báo Popup "Thành tựu" hiện lên khi đủ điều kiện mà không dính líu code vào `GridManager`.

---

### PHASE 5: Tối Ưu Hiệu Năng & Fix Bug Logic (Dựa theo Performance Optimization Plan)

- [x] **Task 5.1: Sửa lỗi Logic Gameplay (Kích hoạt Thành Tựu)**
  - **Hoàn thiện (16/06/2026):**
    - Gắn `GameEvents.TriggerLevelCompleted()` vào `LevelProgressUI.LevelUp()` — gọi TRƯỚC `LoadNextLevel()` để achievement xử lý xong trước khi level mới bắt đầu.
    - Bổ sung `GameEvents.TriggerBoosterUsed()` vào `BoostButton.OnClickButton()` khi người chơi thực sự sử dụng boost (trừ số lượng, phát event, cập nhật UI).
    - Sửa lỗi `UnlockCakes` đếm sai: đổi `UnlockedSkins.Count` → `Count - 1` (trừ skin mặc định `plate_01`).
    - Xóa dead code `OnPizzaMerged` / `TriggerPizzaMerged()` khỏi `GameEvents.cs` (không ai subscribe/invoke).

- [x] **Task 5.2: Khử FindObjectsByType & Boxing (Hot-Path GC)**
  - **Hoàn thiện (16/06/2026):**
    - `GoldDisplay`: Tạo `private static List<GoldDisplay> _activeDisplays` thay vì quét cả Scene mỗi khi có thay đổi vàng.
    - `LevelProgressUI`: Đổi sang pattern Singleton `Instance` và áp dụng vào `UIManager` / `SaveLoadManager` để loại bỏ `FindFirstObjectByType`.
    - `InputManager`: Đổi `private struct HitDistanceComparer` thành `private class` để triệt tiêu Boxing alloc khi gọi `Array.Sort` trong update loop.

- [x] **Task 5.3: Tối ưu Cấp Phát Memory (Medium GC)**
  - **Hoàn thiện (16/06/2026):**
    - `GridManager`: Đưa `new Queue<GridCell>()` và `new HashSet<GridCell>()` trong hàm `CalculatePriorities` và `CalculateLocalPriorities` ra thành biến toàn cục (Cache).
    - Dùng `.Clear()` trước mỗi lần BFS chạy thay vì khởi tạo lại liên tục, xoá bỏ memory leak ngầm. Bảm bảo vòng lặp Cascade sạch bóng Zero-GC.

---

### PHASE 6: Implement Gameplay Booster System (Ngày 8)

- [x] **Task 6.1: Tạo BoosterManager & API hỗ trợ**
  - **Hoàn thiện (16/06/2026):**
    - Tạo `BoosterManager.cs` (Singleton) quản lý state machine của Booster (`Idle`, `MoveSource`, `TrashTarget`).
    - Viết logic 4 loại Booster:
      - **Cutter:** Tạo đĩa bánh mới ở ô trống kề bên đĩa thiếu, spawn lượng miếng tương ứng type đa số để gộp đủ 6 miếng (chuẩn Game Design Document).
      - **Sauce:** Tìm đĩa có nhiều miếng đa số nhất, xóa thiểu số và đổ thêm sốt (tạo miếng mới) cho đủ 6/6.
      - **Move:** Trực tiếp tích hợp vào hệ thống kéo-thả (Drag & Drop) của InputManager. Cho phép nhấc 1 đĩa lên và đặt vào ô trống bất kì, scale Ghost tự động khớp với GridCell.
      - **Trash:** Cho phép tap vào 1 đĩa bất kì để xóa ngay lập tức khỏi lưới, đưa vào ObjectPool và kích hoạt cascade.
    - Cập nhật `GridManager.cs`: Thêm `GetAllCells()`, `TriggerCascade(centerCell)`.

- [x] **Task 6.2: Tích hợp Booster Input vào Game Loop & UI**
  - **Hoàn thiện (16/06/2026):**
    - Sửa `InputManager.cs`: Ưu tiên bắt Raycast cho `BoosterManager` khi ở State `TrashTarget`. Kích hoạt luồng kéo thả đặc biệt cho State `MoveSource`.
    - Sửa `BoostButton.cs`: Xây dựng hệ thống Toggle UI (tự động làm tối các nút đang Active, sáng lại khi Cancel).
    - Logic Hủy (Cancel): Nhấn lại nút đang chọn để hủy, hoặc chọn nút Booster khác để hủy cái cũ và kích hoạt cái mới. Trả lại State Idle an toàn.
  - **Lưu ý thao tác Unity Editor:** Cần kéo thả Prefab (hoặc tạo GameObject) gắn script `BoosterManager` vào Scene.
# FEATURE: Tuần 4 - Tối Ưu Hóa Render, Profiler & Đóng Gói (Nghiệm Thu)

**Status:** 🟢 COMPLETED
**Thời gian:** 16/06 - 22/06/2026 (Thực tế)

## 1. Feature Specification

Tuần 4 là chặng cuối cùng của dự án. Mục tiêu chính là đập tan các vấn đề về hiệu suất (đặc biệt là Render), dọn dẹp các bug còn tồn đọng, hoàn thiện tài liệu và Build ra sản phẩm cuối cùng chuẩn bị cho nghiệm thu.

Các mục tiêu quan trọng nhất:

- **Tối ưu Render (Draw Calls < 50):** Đây là tiêu chí trừ điểm rất nặng. Hiện tại Game đang báo 176 Batches.
- **Tách Canvas UI:** Tách riêng các HUD thay đổi liên tục khỏi Background để tránh Rebuild UI diện rộng.
- **Tắt Bóng (Shadows):** Tối ưu ánh sáng cho Mobile.
- **Profiler Hunting:** Quét lại toàn bộ chuỗi nổ liên hoàn để đảm bảo 100% Zero-GC.
- **Build & Đóng gói:** Xuất APK và Update README.

## 2. Technical Constraints

- **Mobile First:** Mọi cấu hình đồ hoạ phải hướng tới nền tảng Android. Tắt các hiệu ứng dư thừa như Realtime Shadow Cast trên các object nhỏ.
- **Zero-GC Compliance:** Mọi đoạn code tối ưu phải duy trì chuẩn Zero-GC đã thiết lập từ các tuần trước.

## 3. Execution Plan (Tasks)

---

### PHASE 1: Tối Ưu Draw Calls & Render (Ngày 1 - 2)

- [x] **Task 1.1: Bật GPU Instancing (Thao tác trên Editor)**
  - Vật phẩm áp dụng: Material của đĩa bánh (`PizzaPlate`) và các miếng bánh (`PizzaSlice`).
  - **Cách làm:** Mở thư mục chứa Material, click chọn các Material của Đĩa và Miếng bánh -> Tích vào ô `Enable GPU Instancing` trong Inspector. (Cần shader chuẩn URP/Lit hoặc Unlit có hỗ trợ Instancing).

- [x] **Task 1.2: Tối ưu Bóng râm (Shadows)**
  - Tắt đổ bóng cho các vật thể nhỏ: Chọn Prefab của `GridCell` và `PizzaSlice`, trong component `MeshRenderer`, đổi `Cast Shadows` thành `Off` và `Receive Shadows` thành `Off`. (Có thể giữ đổ bóng cho đĩa `PizzaPlate` hoặc mặt bàn nếu muốn hình ảnh có chiều sâu, nhưng tắt hết các thứ nhỏ nhặt).

- [x] **Task 1.3: Static Batching cho Môi Trường**
  - Các vật thể như Mặt bàn lót, Background, và các thành phần tĩnh không di chuyển -> Chọn trên Hierarchy -> Đánh dấu tích vào ô `Static` (góc trên cùng bên phải Inspector). Unity sẽ gộp chúng thành 1 Batch.

- [x] **Task 1.4: Tách Canvas UI Tĩnh và Động**
  - Kiểm tra Canvas của Scene: Tách riêng Canvas chứa Nút bấm tĩnh/Hình nền ra khỏi Canvas chứa Text thay đổi (Gold, Score, Thời gian đếm ngược, Combo). UI tĩnh sẽ không bị rebuild khi UI động thay đổi Text.

- [x] **Task 1.5: Gộp Sprite Atlas**
  - Đóng gói toàn bộ hình ảnh UI 2D vào 1 (hoặc 2) `Sprite Atlas` để Unity có thể gộp tất cả ảnh 2D thành 1 Batch render duy nhất.

---

### PHASE 2: Quét Profiler & Clean Bugs (Ngày 3)

- [x] **Task 2.1: Chạy Profiler và Deep Profiling**
  - Chơi game trong Editor với Unity Profiler (Ctrl + 7). Theo dõi tab `Memory -> GC Alloc`.
  - Đặt đĩa tạo chuỗi nổ lớn (Cascade) và check xem có dòng nào báo Alloc đỏ lên không.

- [x] **Task 2.2: Dọn dẹp Null Reference và Warning**
  - Chơi thử với các Booster, ép lỗi để check Console. Sửa mọi Warning xuất hiện.

---

### PHASE 3: Đóng Gói (Build) và Báo Cáo (Ngày 4 - 5)

- [x] **Task 3.1: Hoàn thiện File README.md**
  - Nhúng hình ảnh sơ đồ Game Flow (System Flow Diagram) vào README.
  - Cập nhật hướng dẫn cài đặt và các tính năng chính.

- [x] **Task 3.2: Xuất APK và Quay Video**
  - Setup `Player Settings` (Icons, Tên App, Package Name).
  - Xuất APK chơi thử trên máy thật.
  - Quay video trải nghiệm Gameplay thực tế (chứng minh các logic Nổ dây chuyền, Booster, Store, Auto-save hoạt động mượt mà).

---

### PHASE 4: Nghiệm Thu (Ngày 6 - 7)

- [x] **Task 4.1: Kiểm tra chéo với Tiêu Chí Của Công Ty**
  - Bám sát file `Yêu Cầu Kỹ Thuật Dự Án 1 (Bloom Sort 3D).md` và `Lộ Trình & Tiêu Chí Nghiệm Thu Dự Án 1.md`. Này là chỉ cần hoàn thiện readme.md trên github và quay video là xong.
