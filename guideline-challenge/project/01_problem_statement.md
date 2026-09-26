# Problem statement + downstream contract

## Bài toán

Gắn nhãn biển báo giao thông trên ảnh đường phố và cao tốc: khoanh khung từng biển và phân loại vào nhóm lớn (cấm, bắt buộc, nguy hiểm, ưu tiên, khác) khi biển nhỏ, ở xa, bị che một phần hoặc mờ.

## Downstream contract

1. **Downstream task / model / user là ai?** Đội gán nhãn biển báo giao thông. Mô hình phát hiện và phân loại biển báo từ đó quyết định có cần đọc chi tiết hay không.
2. **Output annotation nào thực sự cần?** Một bounding box (hình chữ nhật) ôm sát từng biển báo. Mỗi box có các thuộc tính:
   - `category` (select): `prohibitory` · `danger` · `mandatory` · `priority` · `other` · `unknown`
   - `visibility` (select): `clear` · `occluded` · `blurred` · `truncated`
   - `needs_review` (checkbox): đánh dấu biển khó phân định — người QA sẽ xem lại
   - Tag ảnh `image_escalate`: khi chất lượng ảnh quá kém hoặc nhiều vị trí không quyết định được
3. **Failure nào gây hậu quả lớn nhất?** Bỏ sót hoặc gắn sai biển nhóm `priority` (STOP, Give Way) và `prohibitory` (giới hạn tốc độ, cấm ngược chiều) — lỗi critical khiến xe tự hành vi phạm luật hoặc gây tai nạn.
4. **Khi ambiguity không resolve được, escalation path là gì?** Tick `needs_review = true` trên từng bbox không chắc. Cả ảnh có nhiều vị trí không quyết định được → gắn tag `image_escalate`. QA reviewer xử lý tất cả mục này.

## Scope

- **Trong scope (bắt buộc label):** mọi biển giao thông thẳng đứng dành cho xe, nhìn thấy **≥ 20%** mặt biển, cạnh ngắn nhất của bbox **≥ 12 px**.
- **Ngoài scope (ignore):** mặt sau biển; biển phụ nhỏ gắn dưới biển chính; biển quảng cáo/thương mại; biển long môn (overhead gantry); hình biển in trên thùng xe hoặc mặt đường; bbox < 12 px.
- **Geometry tolerance:** bbox ôm sát mép ngoài của phần biển nhìn thấy (gồm viền biển, không bao cột đỡ). Sai lệch cho phép **≤ 2 px** mỗi cạnh.

## Output chấm được

| Decision | Cách thể hiện trong CVAT |
|---|---|
| LABEL | Bbox với `category` và `visibility` đã chọn (khác `__undefined__`) |
| IGNORE | Không vẽ bbox — object nằm ngoài scope |
| UNKNOWN | Bbox + `category = unknown` — xác định là biển nhưng không nhận được nhóm |
| ESCALATE (1 biển) | Bbox + `needs_review = true` — không chắc có phải biển hoặc phân vân nhóm |
| ESCALATE (cả ảnh) | Tag `image_escalate` — chất lượng ảnh quá kém |
| Geometry | Bbox ôm khít viền biển, dung sai ≤ 2 px mỗi cạnh |

## Dữ liệu và giới hạn

- **Nguồn ảnh:** `data/gtsdb/` (28 ảnh, bao gồm ảnh không có biển làm negative sample) + một số ảnh `data/bdd100k/` có biển.
- **Phân nhóm theo hình dạng + màu viền:** biển Đức theo chuẩn châu Âu — không phụ thuộc ký tự ngôn ngữ địa phương.
- **Tổng ảnh dự kiến dùng:** 15–18 ảnh (5 example + 6 calibration + 4–5 blind).
