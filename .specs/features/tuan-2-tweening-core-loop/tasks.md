# FEATURE: Tuần 2 - Tích Hợp Asset 3D, Tweening & Hoàn Thiện Core Loop

**Status:** 🟡 IN PROGRESS
**Thời gian:** 25/05 - 31/05/2026

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

- [ ] **Task 3.2: Triển khai Game Over Detection**
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

- [ ] **Task 4.1: Hiệu ứng Squash/Stretch khi đặt đĩa**
  - Khi đĩa Snap vào lưới → apply scale animation (squash down → stretch up → settle).
  - Sử dụng `AnimationCurve` (SerializeField) để control easing.
  - **Verification:** Đặt đĩa xuống có hiệu ứng "dẻo" rõ ràng, không giật.
  - **File:** `Scripts/Gameplay/PizzaPlate.cs` hoặc `Scripts/Utils/TweenEffects.cs`

- [ ] **Task 4.2: Hiệu ứng Shake khi đặt sai ô + Ghost Mesh preview**
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
  - Không biết sao nhưng có lỗi logic quét các miếng pizza khi mà để các đĩa ở cạnh nhau mà có cách miếng pizza cùng loại nhưng trong đĩa cũng thêm vài miếng khác hoặc không có thì lại không tự di chuyển, vấn dề diễn ra khi đặt đĩa (tức đĩa đó ưu tiên 9)

---

## 4. Tiêu Chí Nghiệm Thu (Cuối Tuần 2)

- [ ] Thao tác kéo thả đĩa pizza mượt mà, có Ghost Mesh xem trước vị trí thả.
- [ ] Miếng pizza di chuyển mượt theo đường cong Bezier, không bị giật lag hay biến mất đột ngột.
- [ ] FSM được phân tách rõ ràng thành các class trạng thái riêng biệt đúng như thiết kế.
- [ ] Hệ thống Object Pooling hoạt động ổn định, không phát sinh lỗi rò rỉ bộ nhớ (Memory Leak).
- [ ] Không còn bất kỳ `GetComponent` runtime hoặc `new List/Array` trong gameplay loop (Zero GC).
- [ ] Cấu trúc thư mục đầy đủ `Utils/`, `UI/`, `States/` đúng `ARCHITECTURE.md`.
