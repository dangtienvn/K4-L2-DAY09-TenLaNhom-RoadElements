# Edge-case library

Tối thiểu **8 card**, khuyến nghị 10–12. Một edge case tốt là case mà hai annotator hợp lý có thể làm khác nhau nếu
guideline chưa rõ.

File này là kho nội bộ của nhóm, **không gửi cho peer**. Card dùng ảnh example/calibration thì chép rule + ví dụ sang `02_guideline.md`. Card về ảnh blind chỉ nằm ở đây, và decision của nó phải có trong `gold_decisions.csv` trước `make freeze`.

---

CASE ID: EC-01
Sample: BDD05 (calibration)
Scene: Cao tốc ban ngày, biển giới hạn tốc độ bị cành cây che ~40% phần trên
Observation: Nhìn thấy phần dưới tròn của biển cấm tốc độ 80 km/h rõ nét (~60% diện tích); cành cây che phần trên bên phải
Decision: LABEL
Expected: traffic_sign, category=prohibitory, visibility=occluded, confidence=certain; bbox bao visible extent (không bao phần bị che)
Rationale: Nhìn thấy ≥ 30% (thực tế ~60%); category prohibitory vì hình tròn + viền đỏ + số rõ ràng; đây là biển cấm tốc độ nên critical nếu bỏ sót
Common mistake: Annotator vẽ bbox bao toàn bộ ước tính diện tích biển kể cả phần bị che (amodal) — phải dùng visible extent
Diversity: occlusion / critical

---

CASE ID: EC-02
Sample: GTS15 (calibration)
Scene: Đường quốc lộ, biển tam giác cảnh báo ở xa — bbox ước tính ~12 px × 14 px
Observation: Biển nhỏ, rõ ràng nhìn thấy hình tam giác đỏ, nhưng không đọc được nội dung biển (quá nhỏ)
Decision: LABEL
Expected: traffic_sign, category=danger, visibility=clear, confidence=uncertain; bbox ôm sát biển
Rationale: bbox ≥ 10 px nên phải label; hình tam giác + viền đỏ đủ để xác định là danger dù không đọc được nội dung bên trong → confidence=uncertain
Common mistake: (1) Bỏ qua biển vì nghĩ quá nhỏ; (2) gán confidence=certain dù không đọc được nội dung
Diversity: small_far / ambiguity

---

CASE ID: EC-03
Sample: BDD07 (calibration)
Scene: Ảnh chạng vạng, ánh sáng yếu, có vật thể hình tròn sáng bên đường — nghi biển giao thông
Observation: Thấy hình tròn phản chiếu ánh đèn, màu không rõ, không đọc được nội dung
Decision: LABEL với UNKNOWN
Expected: traffic_sign, category=unknown, visibility=unreadable, confidence=uncertain
Rationale: Nhận dạng được hình dạng biển giao thông (hình tròn + phản chiếu đèn) nhưng không đọc được nội dung → UNKNOWN, không phải ESCALATE; ESCALATE chỉ khi không xác định được có phải biển giao thông không
Common mistake: Dùng ESCALATE thay vì UNKNOWN vì không đọc được nội dung — cần phân biệt: UNKNOWN = biết là biển, không biết category; ESCALATE = không biết có phải biển hay không
Diversity: low_visibility / ambiguity

---

CASE ID: EC-04
Sample: GTS04 (example)
Scene: Ảnh đường thẳng không có biển giao thông, chỉ có đường + cây + xe hơi
Observation: Không có biển giao thông nào trong ảnh
Decision: IGNORE (không vẽ gì)
Expected: Ảnh không có shape nào
Rationale: Không có biển giao thông → không vẽ gì; đây là negative case quan trọng để annotator biết khi nào bỏ trống là đúng
Common mistake: Annotator hoang mang vì ảnh trống, vẽ shape nhầm vào biển hiệu thương mại hoặc biển tên đường
Diversity: negative

---

CASE ID: EC-05
Sample: GTS25 (blind)
Scene: Ảnh giao lộ có 2 biển gắn trên cùng một cột: biển STOP (bát giác đỏ) và biển cấm rẽ trái (tròn đỏ)
Observation: Hai biển riêng biệt về mặt vật lý, gắn trên cùng cột, không chồng lên nhau
Decision: LABEL × 2 (hai instance riêng biệt)
Expected: Instance 1: traffic_sign, category=priority (STOP), visibility=clear. Instance 2: traffic_sign, category=prohibitory (cấm rẽ trái), visibility=clear
Rationale: Mục 2 guideline: "hai biển gắn trên cùng cột → vẽ hai shape riêng biệt"; STOP là priority, cấm rẽ trái là prohibitory
Common mistake: Vẽ một bbox duy nhất bao cả hai biển → count là 1 instance thay vì 2; hoặc bao luôn cột vào bbox
Diversity: conflict / critical

---

CASE ID: EC-06
Sample: BDD19 (calibration)
Scene: Ảnh đêm, có vật thể hình chữ nhật sáng bên đường — có thể là biển giao thông hoặc biển quảng cáo đèn LED
Observation: Thấy hình chữ nhật phát sáng, màu xanh/trắng, không phân biệt được biển giao thông hay quảng cáo
Decision: ESCALATE
Expected: Shape rectangle ước tính vị trí + tick needs_review=true + category=unknown + visibility=blurred
Rationale: Không đủ bằng chứng để xác định có phải biển giao thông không → needs_review=true
Common mistake: IGNORE vì nghĩ là biển quảng cáo — nhưng không chắc
Diversity: escalation / ambiguity

---

CASE ID: EC-07
Sample: GTS22 (blind)
Scene: Giao lộ có biển tam giác ngược nhường đường và bảng chỉ dẫn
Observation: Biển tam giác ngược nhường đường (Give Way)
Decision: LABEL
Expected: traffic_sign, category=priority, visibility=clear; bbox ôm sát viền biển tam giác
Rationale: Tam giác ngược = Give Way = priority critical
Common mistake: Gán category=danger vì nghĩ tam giác là danger — tam giác ngược là priority, tam giác xuôi mới là danger
Diversity: critical / priority

---

CASE ID: EC-08
Sample: BDD14 (example)
Scene: Đường quốc lộ, biển phản chiếu ánh đèn xe tải in trên thùng xe — hình dạng giống biển cấm vượt
Observation: Trên thùng xe tải có in hình biển cấm vượt nhưng là in trực tiếp lên xe, không phải biển cắm cột
Decision: IGNORE
Expected: Không vẽ shape nào cho hình biển báo in trên xe
Rationale: Mục 5 guideline: "Biển báo in trên thùng xe → ngoài scope → IGNORE"
Common mistake: Label biển in trên xe vì nhìn thấy hình biển cấm
Diversity: ambiguity / out_of_scope

---

CASE ID: EC-09
Sample: GTS20 (blind)
Scene: Biển chỉ hướng hình chữ nhật xanh
Observation: Biển chữ nhật lớn chỉ hướng đường
Decision: LABEL
Expected: traffic_sign, category=informational, visibility=clear
Rationale: Biển chữ nhật chỉ hướng = informational
Common mistake: Gán category=other vì không tìm thấy nhóm chỉ dẫn
Diversity: informational

---

CASE ID: EC-10
Sample: GTS23 (blind)
Scene: Biển cấm tốc độ rõ nét
Observation: Biển tròn viền đỏ giới hạn tốc độ
Decision: LABEL
Expected: traffic_sign, category=prohibitory, visibility=clear
Rationale: Hình tròn viền đỏ = prohibitory critical
Common mistake: Bỏ sót biển nhỏ ở xa
Diversity: critical / prohibitory


