# Peer Feedback — SieuDuAn gửi TenLaNhom

- **Blind pack:** `blind_pack.rar`
- **Export:** `job_22_annotations_2026_09_26_10_33_42_cvat for images 1.1.zip`
- **Định dạng:** CVAT for images 1.1
- **Phạm vi đã làm:** GTS20–GTS25, 6 ảnh, 14 bounding box

## 1. Rule nào rõ nhất / giúp quyết định nhanh nhất?

Quy tắc phân loại theo hình dạng và màu viền ở mục 4 rất dễ áp dụng: tròn viền đỏ là `prohibitory`, tam giác đỏ là `danger`, tròn xanh là `mandatory`, còn STOP, tam giác ngược và thoi vàng là `priority`. Ngưỡng cạnh ngắn bbox ≥ 12 px và quy tắc mỗi biển vật lý là một box riêng cũng giúp quyết định nhanh đối với các biển nhỏ, xa hoặc nhiều biển cùng cột.

## 2. Rule nào mơ hồ hoặc phải tự suy diễn?

Phần biển chữ nhật/biển chỉ hướng chưa thống nhất. Guideline yêu cầu label cả biển chữ nhật gắn trên cột và hướng dẫn dùng `other` cho biển không thuộc bốn nhóm chính, nhưng cấu hình CVAT lại có thêm `informational` mà guideline không định nghĩa. Vì vậy ở các biển chỉ hướng trong GTS20, GTS21 và biển vuông trong GTS24, annotator phải tự suy diễn giữa `informational`, `other` và IGNORE.

Quy tắc biển phụ cũng mâu thuẫn: mục 1 ghi biển phụ nhỏ dưới biển chính phải IGNORE, còn mục 2 lại yêu cầu vẽ một box bao cả biển chính và biển phụ. Cụm biển trong GTS22 vì thế khó xác định số instance và phạm vi box.

## 3. Sample nào khiến guideline "vỡ"?

GTS22 là trường hợp rõ nhất. Mục 9 ghi GTS22 là ảnh không có biển và expected output là không vẽ gì, nhưng ảnh GTS22 trong blind pack có biển tam giác ngược nhường đường và bảng hướng dẫn đỏ-trắng đủ lớn để label. Nội dung ví dụ vừa mâu thuẫn với ảnh vừa đưa đúng mã ảnh blind vào guideline. Khi làm, nhóm mình ưu tiên quan sát ảnh và các rule tổng quát thay vì dòng ví dụ này.

## 4. Attribute/default nào trong CVAT dễ gây thao tác sai?

Attribute `category` trong CVAT có giá trị `informational`, nhưng bảng taxonomy trong guideline không có giá trị này. Ngoài ra guideline gọi tag ảnh là `image_escalate`, còn cấu hình CVAT tạo tag tên `escalate`. Hai điểm lệch tên này khiến annotator không biết nên theo guideline hay theo lựa chọn thực tế trong giao diện. Mặc định `__undefined__` là hợp lý để phát hiện annotation chưa hoàn thành, nhưng cần bảo đảm tất cả box được đổi sang giá trị hợp lệ trước khi export.

## 5. Một thay đổi cụ thể giúp annotator mới ít hỏi hơn?

Nên đồng bộ hoàn toàn guideline với `03_cvat_labels.json`: thêm định nghĩa và ví dụ rõ cho `informational` (đặc biệt biển chữ nhật, biển chỉ hướng và biển vuông), phân biệt nó với `other`, đồng thời dùng thống nhất một tên tag (`escalate` hoặc `image_escalate`). Cũng cần bỏ GTS22 khỏi phần ví dụ và viết lại quy tắc biển phụ thành một quyết định duy nhất, không mâu thuẫn giữa mục 1 và mục 2.

---

## Phân loại nguyên nhân (Root Cause Classification)

| Feedback / Defect | Phân loại | Action |
|---|---|---|
| Lệch `informational` vs `other` (GTS20, GTS21, GTS24) | `guideline_gap` | Thêm định nghĩa rõ và ví dụ cho `informational` vào mục 4 và 9 của guideline v3 |
| Mâu thuẫn biển phụ (Mục 1 vs Mục 2) | `guideline_gap` | Viết lại mục 1 & 2: biển phụ nằm cùng khung biển chính thì vẽ 1 box bao cả hai; biển phụ đứng riêng không có biển chính thì IGNORE |
| GTS22 mã sample xuất hiện trong ví dụ mục 9 | `guideline_gap` | Sửa mã sample trong mục 9 thành GTS28 (calibration sample) |
| Lệch tag tên `escalate` vs `image_escalate` | `guideline_gap` | Đồng bộ tên tag thành `image_escalate` duy nhất ở mọi nơi |
| Peer gán nhầm GTS23 (biển cấm) thành mandatory/priority | `execution_error` | Giữ nguyên rule (hình tròn viền đỏ = prohibitory đã rất rõ ràng) |
