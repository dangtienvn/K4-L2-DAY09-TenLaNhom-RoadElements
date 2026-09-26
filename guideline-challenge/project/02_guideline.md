# Annotation guideline — Phân nhóm biển báo giao thông (biển nhỏ, xa, bị che)

**Version:** v2

<!--
v0 = chưa có bản nháp. Đổi dòng Version ở trên thành v1 khi xong bản nháp đầu, v2 sau calibration, v3 sau blind
handoff; mỗi lần tăng version ghi một dòng vào 08_revision_log.md. `make freeze` đòi v2 trở lên.

File này là thứ nhóm peer nhận nguyên văn trong blind pack và là Guide dán vào CVAT. Peer KHÔNG nhận
edge_case_cards.md, gold_decisions.csv hay sample_pack.csv. Rule nào peer cần biết phải nằm ở đây.
No hidden rules: rule chỉ giải thích bằng miệng thì coi như không tồn tại.
Ví dụ trong guideline chỉ dùng ảnh split example hoặc calibration, không dùng ảnh blind.
-->

## 1. Objective + scope

Label bounding box cho từng biển báo giao thông nhìn thấy được trong ảnh chụp từ xe trên đường phố và cao tốc, để huấn luyện mô hình ADAS phát hiện và phân nhóm biển báo.

**Trong scope — bắt buộc label:**

- Biển báo giao thông hình tròn, hình tam giác, hình chữ nhật, hình bát giác, hình thoi **gắn trên cột hoặc cầu vượt**.
- Biển tạm thời (biển công trường, biển phân làn tạm) nếu có khung/cột đỡ.
- Biển phản chiếu ban đêm có thể nhận ra hình dạng (dù không đọc được nội dung).
- Biển bị che **một phần** (nhìn thấy ≥ 20% diện tích) — label theo visible extent.

**Ngoài scope — IGNORE, không vẽ shape:**

- Mặt sau biển (chỉ thấy tấm kim loại phẳng).
- Biển phụ nhỏ gắn dưới biển chính (ví dụ tấm chữ nhật ghi thời gian hiệu lực).
- Biển quảng cáo thương mại (billboard), biển tên đường phố gắn tường.
- Hình ảnh biển báo **in trên thùng xe** hoặc in trên mặt đường (vạch sơn chỉ dẫn).
- Biển long môn (overhead gantry) treo cao trên đường.
- Phản chiếu biển báo trên kính xe hoặc mặt đường ướt.
- Biển quá nhỏ: cạnh ngắn nhất của bbox **< 12 px** → IGNORE.

## 2. Annotation unit

- Đơn vị: **ảnh tĩnh** (không có track/temporal), mỗi **biển báo vật lý riêng biệt** là một instance — một shape `rectangle`.
- Nếu hai biển gắn trên cùng một cột nhưng là hai biển riêng → vẽ **hai shape riêng biệt**.
- Nếu biển chính có biển phụ gắn bên dưới → vẽ **một shape** bao toàn bộ khung (biển phụ nằm trong scope biển chính).
- Nếu một biển báo tổng hợp (gộp nhiều thông tin trong một khung duy nhất) → vẽ **một shape** bao toàn bộ khung.
- Một biển bị vật che chia thành hai mảnh nhìn thấy → vẫn là **một** instance: vẽ một khung bao cả hai mảnh.

## 3. Geometry rule

- **Shape type:** `rectangle` (bounding box axis-aligned).
- **Visible extent:** box ôm sát phần biển **nhìn thấy được** — không bao gồm cột đỡ, không bao gồm phần bị che khuất.
- **Cạnh trên/dưới/trái/phải:** kéo đến viền ngoài cùng của vật liệu biển báo (bao gồm viền kim loại nếu nhìn thấy).
- **Cột đỡ:** không bao gồm cột vào bbox.
- **Tolerance:** ≤ 2 px lệch mỗi cạnh so với gold là đạt.
- **Biển bị cắt mép ảnh:** kéo bbox đến cạnh ảnh, gán `visibility = truncated`.

## 4. Taxonomy

Một class `traffic_sign`, hai attribute select bắt buộc và một checkbox. Phân nhóm **theo hình dạng và màu viền**, không theo hình vẽ bên trong.

> ⚠️ **`category` và `visibility` mặc định `__undefined__`** — bắt buộc chọn trước khi chuyển ảnh. Export còn `__undefined__` = chưa hoàn thành, bài tính là thiếu.

### Attribute `category`

| Giá trị | Định nghĩa | Nhận biết | Ví dụ điển hình |
|---|---|---|---|
| `prohibitory` | Biển **cấm** — cấm đoán hành động | Hình tròn, viền đỏ, nền trắng | Cấm vượt, giới hạn tốc độ, cấm xe tải |
| `danger` | Biển **nguy hiểm / cảnh báo** — cảnh báo mối nguy phía trước | Tam giác đỉnh hướng lên, viền đỏ | Đường trơn, đường cong, công trường, trẻ em |
| `mandatory` | Biển **bắt buộc** — chỉ định hành động phải thực hiện | Hình tròn, nền xanh dương | Đi thẳng, rẽ phải, tốc độ tối thiểu |
| `priority` | Biển **ưu tiên** — xác định quyền ưu tiên tại giao lộ | Bát giác STOP; tam giác đỉnh hướng xuống; thoi vàng viền trắng | STOP, nhường đường (Give Way), đường ưu tiên |
| `other` | Biển xác định được là biển giao thông nhưng không khớp 4 nhóm trên | Là biển giao thông nhưng không rõ nhóm | Biển đặc thù địa phương lạ |
| `unknown` | Không nhận ra được nhóm do mờ, loá, góc độ | Chắc chắn là biển nhưng không thấy hình dạng hoặc màu viền | Biển quá mờ, ngược sáng hoàn toàn |

> **Quy tắc phân nhóm nhanh:** tròn + viền đỏ → `prohibitory`; tam giác + viền đỏ → `danger`; tròn + nền xanh → `mandatory`; bát giác/tam giác ngược/thoi vàng → `priority`. Khi phân vân giữa hai nhóm: ưu tiên theo **hình dạng** rồi tick `needs_review`.

### Attribute `visibility`

| Giá trị | Khi nào chọn |
|---|---|
| `clear` | Thấy trọn mặt biển, hình dạng và màu viền rõ |
| `occluded` | Bị che một phần (cây, xe, cột), còn ≥ 20% diện tích |
| `blurred` | Xa, mờ hoặc tối nhưng vẫn nhận ra là biển |
| `truncated` | Bị cắt mép ảnh |

### Checkbox `needs_review`

- Mặc định **không tick** (false).
- **Tick khi:** không chắc object có phải biển giao thông không; hoặc phân vân giữa hai nhóm category mà hình dạng không đủ rõ.
- Sau khi tick vẫn phải chọn `category` và `visibility` tốt nhất có thể — không để `__undefined__`.

### Tag ảnh `image_escalate`

- Gắn tag này vào **cả frame** (không phải một shape cụ thể) khi chất lượng ảnh quá kém (mưa lớn, sương mù dày, tối hoàn toàn) hoặc có nhiều vị trí không quyết định được trong cùng một ảnh.

## 5. Inclusion / exclusion

**Bắt buộc label:**

- Mọi biển báo giao thông nhìn thấy ≥ 20% và cạnh ngắn nhất bbox ≥ 12 px.
- Biển bị che một phần bởi cành cây hoặc xe khác, nhưng tổng diện tích nhìn thấy ≥ 20%.
- Biển ở xa (nhỏ) nhưng bbox ≥ 12 px — gán `visibility = clear` hoặc `blurred` tùy tình trạng.
- Biển bị cắt mép ảnh (truncated) — label phần còn thấy, gán `visibility = truncated`.

**Không label (IGNORE):**

- Mặt sau biển (chỉ thấy tấm kim loại phẳng, không thấy mặt biển).
- Biển phụ đứng riêng lẻ (không có biển chính phía trên).
- Biển quảng cáo, biển thương mại, biển tên phố gắn tường, biển cửa hàng.
- Vật giống biển nhưng rõ ràng không phải biển giao thông (đồng hồ, bảng hiệu tròn, logo).
- Phản chiếu, bóng in, hình ảo biển báo trên kính/nước.
- Bbox cạnh ngắn nhất < 12 px — quá nhỏ, model không học được.

## 6. Visibility / occlusion

| Tình huống | `visibility` | `needs_review` | Hành động |
|---|---|---|---|
| Biển rõ nét, thấy toàn bộ | `clear` | Không tick | Label bình thường |
| Biển bị che 20–80% bởi cây, xe, cột | `occluded` | Không tick (nếu vẫn xác định được nhóm) | Label visible extent |
| Biển bị cắt mép ảnh | `truncated` | Không tick | Label sát mép ảnh |
| Biển mờ, loá — nhận được hình dạng nhưng không đọc được nội dung bên trong | `blurred` | Không tick | Label bbox, `category` theo hình dạng + màu viền |
| Biển mờ/tối — không nhận được hình dạng hoặc màu viền | `blurred` | **Tick** | `category = unknown`, vẽ bbox ước tính |
| Biển nhỏ/xa — bbox ≥ 12 px, nhận dạng được | `clear` hoặc `blurred` | Không tick | Label bình thường |
| Biển nhỏ — cạnh ngắn nhất bbox < 12 px | — | — | **IGNORE** — không vẽ |
| Phân vân giữa hai nhóm category | Chọn phù hợp | **Tick** | Chọn nhóm theo hình dạng, tick `needs_review` |

**Quy tắc "20% visible":** ước lượng bằng mắt — nếu nhìn thấy ít nhất 1/5 mặt biển thì label.

## 7. Ambiguity / escalation

| Quyết định | Khi nào | Thể hiện trong CVAT |
|---|---|---|
| **LABEL** | Rõ là biển, phân được nhóm | Bbox + `category` + `visibility` hợp lệ |
| **IGNORE** | Thuộc danh sách ngoài scope mục 5 | Không vẽ shape |
| **UNKNOWN** | Thấy rõ là biển (≥ 12 px, ≥ 20%) nhưng không đủ thông tin nhận nhóm | Bbox + `category = unknown` |
| **ESCALATE (1 biển)** | Không chắc có phải biển không, hoặc phân vân nhóm không giải được | Bbox + tick `needs_review = true` |
| **ESCALATE (cả ảnh)** | Chất lượng ảnh quá kém; nhiều vị trí không quyết định được | Tag `image_escalate` trên frame |

**Decision tree nhanh:**

```
Thấy vật thể →
  Thuộc danh sách IGNORE (mục 5)?
    → CÓ → IGNORE (không vẽ)
    → KHÔNG →
        Cạnh ngắn nhất bbox ≥ 12 px?
          → KHÔNG → IGNORE
          → CÓ →
              Nhìn thấy ≥ 20% mặt biển?
                → KHÔNG → IGNORE
                → CÓ →
                    Chắc chắn là biển giao thông?
                      → KHÔNG CHẮC → Bbox + needs_review = true (ESCALATE)
                      → CÓ →
                          Nhận được nhóm (category)?
                            → CÓ → LABEL (category cụ thể)
                            → KHÔNG → LABEL (category = unknown)
```

**Lưu ý:** Phân vân nhóm không phải lý do để để trống bbox. Luôn vẽ bbox, chọn nhóm tốt nhất có thể theo hình dạng, rồi tick `needs_review`.

## 8. Temporal rule

Không áp dụng — task ảnh tĩnh. Mỗi ảnh annotate độc lập.

## 9. Examples

Chỉ dùng ảnh split `example` hoặc `calibration`.

| sample_id | Thấy gì | Expected output | Rule áp dụng |
|---|---|---|---|
| GTS05 | Biển tròn viền đỏ, rõ nét, toàn bộ biển nhìn thấy | 1 bbox · `category = prohibitory` · `visibility = clear` · `needs_review = false` | Mục 4, 5 |
| GTS09 | Biển rất nhỏ ở xa, cạnh ngắn ≈ 14 px, nhận ra tam giác viền đỏ | 1 bbox · `category = danger` · `visibility = blurred` · `needs_review = false` | Mục 6 — blurred, bbox ≥ 12 px |
| GTS22 | Ảnh đường thẳng không có biển giao thông | Không vẽ gì (IGNORE toàn ảnh) | Mục 5 — negative case |
| GTS12 | Biển chính (tròn cấm) có biển phụ chữ nhật gắn bên dưới | 1 bbox bao cả hai biển · `category = prohibitory` | Mục 2 — biển phụ nằm trong scope biển chính |
| GTS07 | Biển STOP bát giác đỏ rõ nét | 1 bbox · `category = priority` · `visibility = clear` | Mục 4 — STOP = priority |
| BDD07 | Biển bị cành cây che ~45% — còn thấy phần tròn đỏ rõ | 1 bbox visible extent · `category = prohibitory` · `visibility = occluded` | Mục 5 — nhìn thấy ≥ 20% |

## 10. Common mistakes

| Lỗi thường gặp | Cách tránh |
|---|---|
| Bao gồm cột đỡ vào bbox | Kéo cạnh dưới bbox đến chân biển, không kéo xuống cột |
| Vẽ bbox quá rộng bao cả vùng trống xung quanh | Ôm sát visible extent — chỉ phần vật liệu biển |
| Quên label biển nhỏ ở xa | Zoom ảnh 2–3× để kiểm tra góc — biển nhỏ vẫn label nếu cạnh ngắn ≥ 12 px |
| Label biển quảng cáo thương mại | Hỏi: có phải biển cấm / nguy hiểm / bắt buộc / ưu tiên không? Không → IGNORE |
| Gán `category = prohibitory` cho biển STOP | STOP là bát giác → `priority`, không phải `prohibitory` |
| Để `category = __undefined__` khi export | Phải chọn category trước khi bấm Next — `__undefined__` = bài thiếu |
| Dùng `image_escalate` khi chỉ một biển khó | Tag `image_escalate` chỉ khi cả ảnh có vấn đề — một biển khó thì tick `needs_review` trên shape đó |
| Bỏ sót biển bị che 40% | Kiểm tra ngưỡng 20%: nhìn thấy ≥ 20% diện tích → phải label |
| Gán `category = unknown` khi thấy tam giác viền đỏ | Hình tam giác viền đỏ = `danger`, dù không đọc được nội dung bên trong |
| Vẽ bbox amodal (bao cả phần bị che ước tính) | Chỉ bao visible extent — phần bị che không vào bbox |
