# Revision log

Mỗi lần tăng `Version` trong `02_guideline.md`, thêm một hoặc nhiều dòng vào bảng: đổi gì và vì sao, kèm bằng chứng (sample_id, dòng calibration report, câu hỏi trong clarification log, feedback của peer).

| Version | Đổi gì | Vì sao | Bằng chứng |
|---|---|---|---|
| v1 | Bản nháp đầu: đặt 10 mục cơ bản, định nghĩa 1 class `traffic_sign` + 3 attributes (`category`, `visibility`, `confidence`), ngưỡng 30% visible, ngưỡng bbox ≥ 10 px, decision tree LABEL/IGNORE/UNKNOWN/ESCALATE | Downstream contract yêu cầu detect + classify category; ADAS cần phân biệt prohibitory (safety-critical) rõ ràng với các nhóm khác | Họp nhóm phút 35–80; tham khảo GTSDB taxonomy và BDD100K attribute schema |
| v2 | Thêm mô tả "ước lượng bằng mắt" cho ngưỡng 30% visible (mục 6); bổ sung bảng decision tree dạng code block (mục 7); làm rõ phân biệt UNKNOWN vs ESCALATE bằng ví dụ cụ thể; bổ sung common mistake "dùng ESCALATE cho biển không đọc được nội dung" | Setup test: thành viên Phạm Thị Oanh vấp ở ngưỡng 30% và nhầm UNKNOWN với ESCALATE; calibration nội bộ phát hiện 2 bất đồng lớn ở BDD07 (ai ESCALATE, ai UNKNOWN) và GTS15 (ai bỏ qua biển nhỏ) | Calibration session: Đặng Thanh Tiến vs Hoàng Dương Thảo Hà — `06_calibration_report.csv` dòng 1 và 2; Setup test ghi trong `03_ontology_and_cvat_setup.md` |

