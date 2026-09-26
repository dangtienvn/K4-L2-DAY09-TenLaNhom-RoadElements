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
Expected: Instance 1: traffic_sign, category=prohibitory (STOP), visibility=clear, confidence=certain. Instance 2: traffic_sign, category=prohibitory (cấm rẽ trái), visibility=clear, confidence=certain
Rationale: Mục 2 guideline: "hai biển gắn trên cùng cột → vẽ hai shape riêng biệt"; cả hai đều là prohibitory critical — bỏ sót một cái là lỗi critical
Common mistake: Vẽ một bbox duy nhất bao cả hai biển → count là 1 instance thay vì 2; hoặc bao luôn cột vào bbox
Diversity: conflict / critical

---

CASE ID: EC-06
Sample: BDD19 (blind)
Scene: Ảnh đêm, có vật thể hình chữ nhật sáng bên đường — có thể là biển giao thông hoặc biển quảng cáo đèn LED
Observation: Thấy hình chữ nhật phát sáng, màu xanh/trắng, không phân biệt được biển giao thông hay quảng cáo; không thấy hình dạng đặc trưng (tròn/tam giác) hay màu đặc trưng (viền đỏ)
Decision: ESCALATE
Expected: Shape rectangle ước tính vị trí + frame tag ESCALATE + category=unknown + confidence=uncertain
Rationale: Không đủ bằng chứng để xác định có phải biển giao thông không (không có hình dạng đặc trưng, không có màu đặc trưng) → ESCALATE; khác với EC-03 (biết là biển, chỉ không biết category)
Common mistake: (1) IGNORE vì nghĩ là biển quảng cáo — nhưng không chắc; (2) LABEL với category=unknown mà không có frame tag ESCALATE; (3) không vẽ shape khi ESCALATE
Diversity: escalation / ambiguity

---

CASE ID: EC-07
Sample: BDD22 (blind)
Scene: Ảnh buổi chiều, ánh mặt trời thẳng vào camera, biển giao thông ở khu vực loá sáng — thấy hình dạng tròn trắng nhưng không có màu sắc nào khác
Observation: Biển hoàn toàn bị overexpose (loá trắng) — thấy hình tròn nhưng không đọc được bất kỳ nội dung hay màu sắc nào
Decision: LABEL với UNKNOWN
Expected: traffic_sign, category=unknown, visibility=unreadable, confidence=uncertain; bbox ôm hình tròn nhìn thấy
Rationale: Hình tròn trên cột đường → xác định được là biển giao thông (đủ hình dạng) → UNKNOWN, không ESCALATE; visibility=unreadable vì không đọc được nội dung
Common mistake: Bỏ sót vì nghĩ biển loá không label được; hoặc dùng ESCALATE vì không đọc được (nhưng đã xác định được là biển giao thông)
Diversity: low_visibility / unreadable

---

CASE ID: EC-08
Sample: BDD14 (example)
Scene: Đường quốc lộ, biển phản chiếu ánh đèn xe tải in trên thùng xe — hình dạng giống biển cấm vượt
Observation: Trên thùng xe tải có in hình biển cấm vượt (theo quy định khi xe tải) nhưng là in trực tiếp lên xe, không phải biển cắm cột
Decision: IGNORE
Expected: Không vẽ shape nào cho hình biển báo in trên xe
Rationale: Mục 5 guideline: "Biển báo in trên thùng xe → ngoài scope → IGNORE"; biển giao thông trên cột và biển in trên xe khác nhau về downstream utility — model detect biển trên cột, không detect hình in trên xe
Common mistake: Label biển in trên xe vì nhìn thấy hình biển cấm — phải kiểm tra "gắn trên cột/khung" hay "in trên xe"
Diversity: ambiguity / out_of_scope

---

CASE ID: EC-09
Sample: BDD02 (calibration)
Scene: Ảnh cao tốc, biển giới hạn tốc độ 80 bị xe bên cạnh che ~65% (chỉ thấy góc phải dưới với phần số "0")
Observation: Thấy rất ít của biển — ước tính < 35% diện tích; có thể thấy phần số "0" trên nền trắng, không chắc có viền đỏ
Decision: LABEL (nếu ước tính ≥ 30%) hoặc IGNORE (nếu ước tính < 30%)
Expected: traffic_sign, category=prohibitory, visibility=occluded, confidence=uncertain; bbox bao phần visible
Rationale: Đây là case boundary của ngưỡng 30% — annotator phải ước lượng bằng mắt; guideline cho phép sai sót trong zone 25–35% (→ confidence=uncertain); quan trọng là không bỏ sót biển cấm tốc độ nếu thấy ≥ 30%
Common mistake: (1) Bỏ sót vì thấy quá ít biển; (2) Dùng confidence=certain dù đang ở vùng không chắc; (3) label amodal thay vì visible extent
Diversity: occlusion / critical / boundary_case

---

CASE ID: EC-10
Sample: GTS18 (blind)
Scene: Đường quốc lộ ngày nắng, biển cấm tốc độ 60 km/h rõ nét nhưng phần trên bị một cành cây che ~35%
Observation: Biển tròn với số 60 rõ ràng, viền đỏ nhìn thấy ~65% diện tích; cành cây che phần trên góc trái
Decision: LABEL
Expected: traffic_sign, category=prohibitory, visibility=occluded, confidence=certain; bbox bao visible extent (phần tròn nhìn thấy)
Rationale: Nhìn thấy ~65% >> ngưỡng 30%; number + viền đỏ = đủ bằng chứng prohibitory; đây là critical case vì biển cấm tốc độ — bỏ sót = lỗi critical trong downstream ADAS
Common mistake: Vẽ bbox amodal (bao cả phần bị che ước tính) thay vì visible extent
Diversity: occlusion / critical

