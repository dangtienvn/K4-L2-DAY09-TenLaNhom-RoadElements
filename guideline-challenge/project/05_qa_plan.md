# QA plan + quality gates

## Flow

Guideline → Calibration → Production → Self-QC → Review → Rework → Quality Gate.

- **Ai review, review bao nhiêu:** Đặng Thanh Tiến (QA owner) review **100%** ảnh blind + **30%** ảnh calibration (random sample). Mỗi batch production review ít nhất 20% (random) + 100% ảnh có tag `image_escalate` + 100% shape có `needs_review = true`.
- **Chọn sample theo rule nào:**
  1. **Random 20%** toàn batch production.
  2. **100% tag `image_escalate`** và **100% shape `needs_review = true`** — high-risk mandatory.
  3. **100% ảnh được annotator mới thực hiện** (lần đầu tham gia project).
  4. **Stratified by visibility:** ưu tiên sample `occluded` và `blurred` (tỉ lệ defect cao hơn theo calibration evidence).
- **Issue được ghi ở đâu, đóng thế nào:** Issue ghi vào `05_qa_issues.csv` (nếu có) hoặc ghi note trong `transfer_score.csv`. Issue đóng khi annotator rework + QA verify lại instance đó. Issue `critical` phải đóng trong 1 ngày làm việc.
- **Khi phát hiện guideline gap thì update và version ra sao:** Ghi vào `08_revision_log.md` với version tiếp theo (v2 → v3). Guideline gap = khi ≥ 2 annotator độc lập làm khác nhau với cùng một loại case và không có rule nào cover. Sau khi update guideline, rework tất cả instance thuộc loại case đó.

## Defect severity

| Severity | Định nghĩa cho project này | Ví dụ | Action mặc định |
|---|---|---|---|
| Critical | Bỏ sót hoặc gắn sai biển nhóm `priority` (STOP, Give Way) hoặc `prohibitory` (cấm tốc độ, cấm ngược chiều) → model không cảnh báo nguy hiểm | Bỏ sót biển STOP; label `priority` thành `prohibitory` | Reject toàn batch, rework 100%, re-review |
| Major | Bounding box lệch > 10% chiều rộng biển; nhầm lẫn `danger` ↔ `mandatory`; bỏ sót biển rõ nét cạnh ngắn ≥ 20 px | Box quá rộng bao cả cột đỡ; label biển STOP thành `prohibitory` | Rework instance cụ thể, re-review instance đó |
| Minor | Bounding box lệch 3–10%; nhầm `other` ↔ `priority`; quên set `visibility = occluded` khi bị che | Box lệch 5 px; gán `clear` thay vì `truncated` | Sửa tại chỗ nếu có thể; ghi note nếu không ảnh hưởng model |
| Question | Annotator không chắc nhưng đã tick `needs_review = true`; trường hợp cần thảo luận thêm | Case mà guideline chưa cover rõ | Ghi vào review log, thảo luận trong session calibration tiếp theo |

## Metrics

| Metric | Cách tính | Vì sao phù hợp với bài toán |
|---|---|---|
| **Critical defect rate** | (số instance có lỗi Critical) / (tổng instance được review) | Directly maps to safety risk — bỏ sót biển cấm là hậu quả nghiêm trọng nhất |
| **Major defect rate** | (số instance có lỗi Major) / (tổng instance được review) | IoU thấp do box sai lớn → model miss detection |
| **Inter-annotator agreement (IoU)** | Mean IoU của cặp annotator trên cùng ảnh calibration | Đo consistency geometry — thấp → guideline geometry rule chưa rõ |
| **Escalation rate** | (số frame tag ESCALATE) / (tổng frame) | Cao → guideline ambiguity rule chưa đủ; thấp sau revision → cải thiện |
| **Unknown category rate** | (số instance `category = unknown`) / (tổng instance) | Cao → taxonomy chưa đủ hoặc ảnh chất lượng quá kém |

**Metric high-risk tách riêng:**
- **Critical escape rate = 0%** (zero tolerance) — nếu bất kỳ biển `prohibitory` nào bị bỏ sót trong batch đã pass QA → reject toàn batch, root cause analysis bắt buộc.

## Quality gate

```text
PASS if:
  Critical defect rate = 0%
  AND Major defect rate ≤ 5%
  AND Mean IoU (calibration) ≥ 0.75
  AND Escalation rate ≤ 10%

REWORK if:
  Critical defect rate = 0%
  AND (Major defect rate > 5% AND ≤ 15%)
  OR (Mean IoU ≥ 0.65 AND < 0.75)
  → Rework tất cả instance Major, re-review 100% instance rework

REJECT / ESCALATE if:
  Critical defect rate > 0%
  OR Major defect rate > 15%
  OR Mean IoU < 0.65
  → Reject toàn batch, tổ chức calibration session bổ sung, review lại guideline
```

**Trade-off:** Threshold IoU ≥ 0.75 khá cao cho task biển nhỏ/xa — nhưng downstream model cần IoU ≥ 0.5 để count as true positive, nên annotation IoU 0.75 cho headroom. Nếu sau calibration thực tế mean IoU dao động 0.70–0.80 thì threshold 0.75 là hợp lý. Có thể hạ xuống 0.70 nếu phần lớn biển nhỏ < 30 px mà team không thể đạt 0.75 — cần ghi lại trade-off trong revision log.

