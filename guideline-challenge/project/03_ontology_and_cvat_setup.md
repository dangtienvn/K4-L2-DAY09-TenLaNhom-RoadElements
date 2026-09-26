# Ontology + CVAT setup

Bảng ontology là **source of truth** cho schema CVAT: `03_cvat_labels.json` phải khớp từng dòng ở đây.

## Ontology table

| Name | Geometry | Type | Allowed values | Default | Mutable? | Rationale |
|---|---|---|---|---|---|---|
| `traffic_sign` | rectangle | class | — | — | No | Object class duy nhất — detect trước, classify category sau |
| `category` | — | attribute (select) | `prohibitory` · `danger` · `mandatory` · `priority` · `other` · `unknown` | `__undefined__` | No | Buộc annotator chọn nhóm; mặc định `__undefined__` để lộ bài nếu bỏ trống |
| `visibility` | — | attribute (select) | `clear` · `occluded` · `blurred` · `truncated` | `__undefined__` | No | Buộc annotator quan sát; mặc định `__undefined__` để lộ bài nếu bỏ trống |
| `needs_review` | — | attribute (checkbox) | `false` · `true` | `false` | No | Đánh dấu biển khó để QA xem lại; mặc định tắt — chủ động tick khi không chắc |

**Frame tag (không phải shape attribute):**

| Tag | Ý nghĩa | Khi nào dùng |
|---|---|---|
| `image_escalate` | Cả ảnh cần QA review | Chất lượng ảnh quá kém, hoặc nhiều vị trí không quyết định được trong cùng một ảnh |

## Class hay attribute

- **`traffic_sign` là class** vì: là object type có geometry riêng cần detect, downstream model cần bbox để train.
- **`category` là attribute** vì: mọi biển đều cùng class `traffic_sign`, chỉ khác nhóm chức năng; tách ra 6 class riêng sẽ có geometry rule giống hệt nhau nhưng quản lý phức tạp hơn.
- **`visibility` là attribute** vì: đây là tính chất của instance trong ảnh cụ thể, không phải object type khác nhau — cùng biển cấm tốc độ có thể `clear` hoặc `occluded` tùy ảnh.
- **`needs_review` là checkbox (không phải select)** vì: đây là flag nhị phân đơn giản — có/không cần review; dùng checkbox giúp annotator tick nhanh hơn.
- **Tất cả select default là `__undefined__`**: bắt buộc annotator phải chủ động chọn. Nếu export còn `__undefined__` → QA flag ngay → tốt hơn là để trống và bắt được, thay vì gán default sai mà không biết.
- **`needs_review` default là `false`**: checkbox — mặc định không tick là hợp lý, chỉ tick khi có lý do cụ thể.
- **Dữ liệu là biển Đức (GTSDB)**: taxonomy 5 nhóm chính (prohibitory/danger/mandatory/priority/other) phân loại theo **hình dạng + màu viền**, không phụ thuộc ký tự ngôn ngữ.

## CVAT

- **Phiên bản CVAT** (`py lab9.py cvat`): CVAT 2.x (cài từ Day 2, `docker compose start` trong thư mục `cvat-day2`)
- **Tên task calibration:** `TenLaNhom-trafficsign-calib-v1`
- **Guide của task đã dán `02_guideline.md`?** Có — dán toàn bộ nội dung vào tab **Guide** của CVAT task
- **Nhóm dùng Track hay Shape, vì sao:** **Shape** — task ảnh tĩnh, không cần track liên frame; Shape đơn giản hơn, ít lỗi hơn

## Setup test

Một thành viên **chưa tham gia setup** mở task và phải tự trả lời được:
- **Label gì?** → `traffic_sign` (rectangle)
- **Dùng tool nào?** → Rectangle tool (phím tắt **R**)
- **Gán attribute nào?** → `category`, `visibility` (bắt buộc), `needs_review` (khi cần)
- **Khi không chắc nhóm?** → Chọn nhóm theo hình dạng rồi tick `needs_review`
- **Khi không biết có phải biển không?** → Tick `needs_review`; nếu cả ảnh có vấn đề → thêm tag `image_escalate`

**Kết quả test:** _(điền sau khi thực hiện — ghi ai test và chỗ họ vấp)_
