# spec.md: Cheezy-Savoround-PH61823

**Description:** Game Puzzle giải đố thả đĩa pizza (Pizza merge game).

## Tech Stack Đặc Thù
- **Thể loại:** Puzzle / Casual 3D (Góc nhìn Top-down).
- **Core Loop:** Kéo thả đĩa pizza lưới lục giác (Hex grid) -> Ghép bánh -> Nổ điểm.
- **Thư viện đặc thù:** DOTween (cho animation bay bánh), Cartoon FX Remaster (cho hiệu ứng nổ).

## Điểm lưu ý kiến trúc
- Sử dụng thuật toán đệ quy (BFS/Flood Fill) để tìm bánh cùng loại.
- Thuật toán kéo bánh đồng thời (Bloom Sort).
