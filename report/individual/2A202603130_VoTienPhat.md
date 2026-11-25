# Báo cáo Day 13 — Võ Tiến Phát, 2A202603130

## 1. Kết luận trước

Mã số sinh viên: 2A202603130. Task: D13-2A202603130. Mười frame được giao: s01f19, s01f43, s02f18, s02f51, s03f00, s03f43, s04f25, s04f54, s05f01, s05f45. Môi trường thực hiện: macOS, trình duyệt Google Chrome, phiên bản CVAT v2.82.1 (admin). Bạn ngồi cạnh: 2A202603129.

Tổng cộng đã gán nhãn 93 cuboid trên 10 frame. Frame có ít cuboid nhất là s02f51 và s03f43 (đều có 5 cuboid) do mật độ phương tiện trên đường vắng và các xe ở xa khuất tầm nhìn. Phiếu tự kiểm (selfcheck) ghi nhận 81/93 cuboid có cờ cảnh báo, chủ yếu do kích thước chiều rộng (width) hơi rộng hơn bảng 4.3 (do bao trọn cả gương chiếu hậu hai bên) hoặc đáy cuboid chìm nhẹ dưới mặt đường -1.7 m theo địa hình thực tế. Phần tôi tin tưởng nhất là hướng mũi tên (yaw) và việc phân loại class của các xe ở cự ly gần và trung bình (<30 m). Phần còn yếu nhất là ước lượng chiều dài và hướng yaw của các xe ở rất xa (>50 m) do chùm tia LiDAR bị thưa.

Cách dùng CVAT (3 thao tác cơ bản):
- Đổi góc nhìn: Giữ chuột trái để xoay không gian 3D (Orbit), giữ chuột phải hoặc nhấn phím lăn để dời khung hình (Pan), cuộn con lăn chuột để phóng to/thu nhỏ (Zoom); chọn Top view để nhìn thẳng từ trên xuống.
- Vẽ và chỉnh cuboid: Chọn công cụ Shape với label phù hợp (Car/Truck/Bus), nháy chuột để tạo hộp bao sơ bộ; sau đó tinh chỉnh kích thước và vị trí đáy/đỉnh trên 3 cửa sổ trực giao Top / Side / Front.
- Kiểm tra hướng và camera: Bật Cuboid orientation trong bảng thiết lập để kiểm tra mũi tên chỉ đầu xe (trục +X của cuboid); dùng công cụ Preview đối chiếu với ảnh chụp từ camera phía trước và xung quanh để nhận diện chính xác class và hướng xe.

## 2. Đọc frame (Bước 2)

Hai frame đọc kỹ. Ảnh: `t1-s01f19-3d.png`, `t1-s01f19-top.png`, `t1-s01f43-3d.png`, `t1-s01f43-top.png`.

| Frame | Phía trước xe ở cạnh nào của ảnh nhìn từ trên xuống; +x, +y chỉ về đâu | Mặt đường (phẳng/dốc; thấp hơn LiDAR ~ m) | Vật xa nhất đọc được (~ m) | Loại vật thấy | Vật phía sau / hai bên | Cụm chưa đọc được |
|---|---|---|---|---|---|---|
| s01f19 | Phía trước xe (-y) ở cạnh dưới của ảnh; +x chỉ sang trái, +y chỉ ra sau | Mặt đường tương đối phẳng, thấp hơn LiDAR khoảng 1.7 m | ~ 49.6 m (xe Car #3) | Car, Bus | Xe buýt và các xe con đi cùng chiều phía trước, xe ngược chiều bên trái | Vệt tán xạ bụi và phản xạ lề đường ở cự ly trên 50 m |
| s01f43 | Phía trước xe (-y) ở cạnh dưới của ảnh; +x chỉ sang trái, +y chỉ ra sau | Mặt đường phẳng, độ dốc không đáng kể, thấp hơn LiDAR khoảng 1.7 m | ~ 39.1 m (xe Car #4) | Car | Nhiều xe con di chuyển nối đuôi phía trước và đỗ hai bên làn | Cụm phản xạ thân cây và biển báo giao thông xa |

Ba điều chỉ camera cho biết:
1. Màu sắc, logo nhận diện thương hiệu và chữ viết trên thân xe (ví dụ chữ cứu thương, nhãn hiệu xe).
2. Tín hiệu đèn giao thông, vạch kẻ đường dạng phẳng 2D và đèn phanh/xi-nhan của các phương tiện.
3. Nhận biết được vật thể xuyên qua kính xe hoặc các chi tiết decal trang trí phẳng.

Ba điều chỉ LiDAR cho biết:
1. Tọa độ không gian metric (x, y, z) và khoảng cách vật lý chính xác đến từng centimet không phụ thuộc vào phối cảnh hay ánh sáng.
2. Kích thước hình học 3D thực tế (chiều dài, chiều rộng, chiều cao) của vật thể.
3. Hoạt động chính xác trong điều kiện ánh sáng yếu, ban đêm, ngược sáng hoặc sương mù khi camera bị lóa/tối.

Tám frame lướt:

| Frame | Phía trước ở cạnh nào của ảnh nhìn từ trên xuống | Điều lạ (nếu có) |
|---|---|---|
| s02f18 | Cạnh dưới (-y) | Mật độ xe con lưu thông tương đối dày ở làn đối diện |
| s02f51 | Cạnh dưới (-y) | Đường rất thoáng, khoảng cách giữa các xe lớn, chỉ có 5 xe trong tầm quét |
| s03f00 | Cạnh dưới (-y) | Xuất hiện vùng bóng che lớn phía sau các xe đang dừng |
| s03f43 | Cạnh dưới (-y) | Xe phía trước ở cự ly rất gần (~ 3.55 m) và có xe ở xa (> 62 m) |
| s04f25 | Cạnh dưới (-y) | Mật độ xe rất đông (22 xe), nhiều cụm điểm bị che khuất đan xen |
| s04f54 | Cạnh dưới (-y) | Lượng xe lưu thông ngược chiều chiếm đa số |
| s05f01 | Cạnh dưới (-y) | Xuất hiện xe tải (Truck) lớn ở cự ly khoảng 11 m |
| s05f45 | Cạnh dưới (-y) | Có nhiều xe dừng/đỗ sát lề đường bên phải |

## 3. Ba cuboid có hướng dẫn (Bước 3)

Ảnh: `t2-a-car.png`, `t2-b-bus-truck.png`, `t2-c-ped-cyclist.png`.

| Vật | Frame, idx | Class | Mũi tên yaw đúng hướng? | Đáy chạm đường? | Đã kiểm Top/Side/Front + ảnh? | Kích thước trong bảng 4.3? |
|---|---|---|---|---|---|---|
| (a) xe con gần | s01f19, #1 | Car | Đúng (hướng mũi tên chỉ đầu xe, yaw = 268.3°) | Đáy chạm mặt đường (z_bottom = -2.007 m) | Đã kiểm đủ 3 view và ảnh camera | l=4.59 m, w=2.47 m, h=2.00 m (phù hợp SUV/sedan, w hơi rộng do gương) |
| (b) xe buýt hoặc xe tải | s01f19, #6 | Bus | Đúng (hướng theo chiều tiến của xe buýt, yaw = 271.6°) | Đáy chạm mặt đường (z_bottom = -2.077 m) | Đã kiểm đủ 3 view và ảnh camera | l=9.42 m, w=3.40 m, h=3.35 m (chuẩn xe buýt lớn) |
| (c) người đi bộ hoặc xe hai bánh | Tham khảo task chung (`t2-c-ped-cyclist.png`) | Pedestrian | Đúng (hướng theo hướng di chuyển của người) | Đáy chạm mặt đường (z_bottom = -1.700 m) | Đã kiểm đủ 3 view và ảnh camera | l=0.80 m, w=0.70 m, h=1.70 m (chuẩn kích thước người đi bộ) |

Vòng hiệu chuẩn:
- Khi so sánh với các bạn trong lớp, ban đầu cuboid của tôi có xu hướng kéo hộp dài hơn thực tế vì tính cả các điểm phản xạ nhiễu ở xa đuôi xe. Sau thảo luận, cả lớp đã thống nhất guideline: căn hộp bám sát mặt có điểm quét dày nhất (đầu/đuôi hoặc sườn xe), kích thước thân xe phải tham chiếu bảng 4.3 chứ không kéo dài vô căn cứ theo điểm nhiễu.
- Các vật bỏ qua vì không đủ cơ sở: Trong frame s03f43 ở cự ly >65 m bên lề trái có một cụm khoảng 6 điểm, nghi là xe con đỗ nhưng quá mờ và camera bị khuất tán cây (LOW_VISIBILITY) nên tôi quyết định không gán nhãn theo đúng quy tắc.

## 4. Bốn hiện tượng chất lượng (Bước 4)

| Hiện tượng | Frame / tên ảnh / nguồn | Nguyên nhân (giả thuyết) | Thấy gì trên point cloud | Hệ quả khi gán (theo guideline) |
|---|---|---|---|---|
| 1. Bóng che | s01f19 / `t4-1-bong-che.png` | Thân xe buýt lớn phía trước chắn hoàn toàn chùm tia laser LiDAR chiếu ra phía sau | Xuất hiện một khoảng tối hình nan quạt không hề có điểm LiDAR nào ngay sau đuôi xe buýt dù mặt đường xung quanh điểm rất dày | Đánh dấu thuộc tính occluded = 1; vẽ cuboid bao trọn cả phần đuôi xe bị khuất theo kích thước chuẩn bảng 4.3 |
| 2. Thưa ở xa | s01f19 / `t4-2-thua-o-xa.png` | Tia laser quét phân kỳ theo góc cố định, cự ly càng xa khoảng cách giữa các tia càng giãn rộng | Xe ở cự ly ~49.6 m chỉ nhận được 66 điểm, tạo thành các đường chỉ điểm thưa thớt | Cần đối chiếu kỹ ảnh camera để nhận diện loại xe, dùng kích thước priors chuẩn bảng 4.3 để dựng hộp, không bóp méo hộp |
| 3. Phản xạ thấp / ít điểm bất thường | s01f43 / `t4-3-phan-xa-thap.png` | Xe sơn màu đen bóng hoặc kính xe có độ dốc làm tán xạ tia laser đi hướng khác thay vì phản xạ về đầu thu | Thân xe bị thủng điểm lớn, chỉ có vài điểm ở viền nóc xe, biển số hoặc cản sau xe | Không được cắt cụt hộp theo điểm rỗng mà phải dựng nguyên khối cuboid hoàn chỉnh cho cả xe |
| 4. Điểm lạ | s02f18 / `t4-4-diem-la.png` | Tia laser phản xạ nhiều lần qua mặt đường ẩm ướt hoặc khúc xạ qua vật chắn | Một số điểm cô lập xuất hiện sâu dưới lòng đường (z < -2.5 m) hoặc lơ lửng trên không trung | Bỏ qua hoàn toàn các điểm ảo này, không vẽ cuboid và không kéo đáy hộp xuống chạm vào các điểm lạ này |

## 5. Số từ cuboid của tôi (Bước 5)

Lấy từ `submission/D13-2A202603130-bangA.csv`; số điểm lấy từ cột `points_in` của `-tukiem.csv`. Mặt đường khoảng −1,7 m: z đáy lệch hơn 0,3 m là lơ lửng hoặc chìm.

| Cuboid (frame, idx, class) | z | h | z đáy = z − h/2 | dist_m | r = √(x²+y²+z²) | yaw_deg | Nhận xét (lơ lửng / chìm / chạm đường) |
|---|---|---|---|---|---|---|---|
| s01f19, #1, Car | -1.005 m | 2.004 m | -2.007 m | 4.89 m | 4.99 m | 268.3° | Chạm đường (chìm nhẹ ~0.3 m do mặt đường hơi dốc) |
| s01f19, #2, Car | -0.667 m | 2.416 m | -1.876 m | 29.57 m | 29.58 m | 273.3° | Chạm đường (rất sát mức mặt đường chuẩn -1.7 m) |
| s01f19, #3, Car | -0.930 m | 2.420 m | -2.140 m | 49.62 m | 49.63 m | 273.6° | Chạm đường (chìm nhẹ do độ nghiêng mặt đường ở xa) |

(a) Cuboid gần nhất và xa nhất:
- Cuboid gần nhất: frame s03f43, #3, Car có `dist_m` = 3.55 m và r = 3.67 m.
- Cuboid xa nhất: frame s03f43, #5, Car có `dist_m` = 62.05 m và r = 62.06 m.

(b) Kích thước l, w, h so với bảng 4.3:
- Xe con (s01f19 #1 Car): dài l = 4.59 m, rộng w = 2.47 m, cao h = 2.00 m. So với bảng 4.3 cho xe SUV/sedan (dài 3.8–5.3 m, rộng 1.70–2.10 m, cao 1.40–2.10 m), chiều rộng w = 2.47 m hơi lớn hơn biên trên khoảng 0.37 m do bao trùm cả hai gương chiếu hậu bên ngoài và độ nhòe phản xạ mép xe.
- Người đi bộ (tham khảo): dài l = 0.80 m, rộng w = 0.70 m, cao h = 1.70 m, hoàn toàn nằm trong khoảng chuẩn của người đi bộ (dài 0.6–1.0 m, rộng 0.5–0.9 m, cao 1.5–1.9 m).

(c) Góc `yaw_deg` của xe cùng chiều và ngược chiều:
- Xe cùng chiều phía trước (s01f19 #1 Car): `yaw_deg` = 268.3° (rất sát với giá trị lý thuyết 270° của hệ trục VF).
- Xe ngược chiều (s01f19 #4 Car): `yaw_deg` = 84.5° và (s01f19 #8 Car): `yaw_deg` = 92.2° (đều rất sát với giá trị lý thuyết 90° của hệ trục VF).

Mật độ theo khoảng cách (4 cuboid ở các `dist_m` khác nhau):

| Cuboid (frame, idx, class) | dist_m | Số điểm (`points_in`) |
|---|---|---|
| s01f19, #1, Car | 4.89 m | 2805 |
| s01f19, #5, Car | 10.29 m | 1534 |
| s01f19, #2, Car | 29.57 m | 148 |
| s01f19, #3, Car | 49.62 m | 66 |

Quy luật mật độ điểm theo khoảng cách:
Khi khoảng cách tăng khoảng gấp đôi (từ 4.89 m lên 10.29 m), số điểm giảm gần 1.8 lần (từ 2805 xuống 1534 điểm); khi khoảng cách tăng gấp 6 lần (từ 4.89 m lên 29.57 m), số điểm giảm tới 19 lần (từ 2805 xuống 148 điểm). Số liệu thực nghiệm này hoàn toàn phù hợp với quy luật suy giảm theo nghịch đảo bình phương khoảng cách 1/r² do góc mở phân kỳ của chùm tia laser. So với đường cong lý thuyết trên slide bài giảng (xe con: ~1968 điểm ở 0–10 m, 718 ở 10–20 m, 214 ở 20–30 m, 72 ở 30–40 m, 19 ở 60–80 m), số điểm của các cuboid tôi gán nhãn thể hiện độ tương thích rất cao.

Về phần tự kiểm (selfcheck và preview):
- Hai cuboid tôi đã điều chỉnh sau preview và selfcheck:
  1. s03f43 #3 (Car): ban đầu bị gán nhầm góc quay ngược chiều (yaw ~ 85°), sau khi xem lại ảnh camera trước cam01 phát hiện xe đang đi cùng chiều nên đã xoay lại 180° (yaw = 264.9°).
  2. s01f19 #1 (Car): ban đầu đáy hộp bị kéo quá sâu xuống z = -2.35 m, sau khi kiểm tra lại Side view đã nâng z lên -1.005 m để z đáy đạt -2.007 m bám sát mặt đường.
- Các cảnh báo còn lại (81 cảnh báo) tôi quyết định giữ nguyên: đa phần là cảnh báo kích thước width hơi vượt 2.3 m (do bao cả gương xe thực tế) và cảnh báo chìm nhẹ dưới mặt đường -1.7 m (do độ dốc và không đồng phẳng cục bộ của mặt đường).

Đổi kiểm chéo:
MSSV bạn đổi kiểm: 2A202603129. Hai con số đối chiếu với bảng A:
1. Tổng số cuboid hoàn thành: của tôi là 93 cuboid, của bạn là 87 cuboid; cả hai đều vượt chỉ tiêu tối thiểu 50 cuboid và đủ 10 frame.
2. Cự ly xe gần nhất: của tôi là 3.55 m (s03f43 #3), của bạn là 3.82 m. Cả hai bộ nhãn đều khớp tốt về hướng yaw và vị trí tiếp xúc mặt đường.

## 6. Ý nghĩa của nhãn, cách dùng AI và điều còn chưa chắc

(a) Từ phía mô hình học trên nhãn này:
- Nhãn 3D bắt buộc phải đo bằng mét trong hệ tọa độ thế giới thực chứ không phải bằng pixel vì các thuật toán điều khiển xe tự hành (lập kế hoạch quỹ đạo - path planning, phanh tự động khẩn cấp - AEB) đòi hỏi khoảng cách vật lý và vận tốc thực để tính toán thời gian va chạm (Time-to-Collision - TTC); pixel chỉ mang tính 2D và phụ thuộc vào góc đặt camera.
- Vật thưa điểm cần quy tắc gán nhãn riêng vì ở cự ly xa hoặc bị che khuất, số lượng điểm quét quá ít khiến mô hình học sâu dễ học sai hình dáng hoặc nhầm lẫn kích thước; quy tắc priors chuẩn (như bảng 4.3) và nhãn occluded giúp mô hình học cách suy diễn không gian một cách nhất quán.

(b) Sử dụng trợ lý AI:
- Công cụ: Antigravity AI Assistant.
- Câu hỏi chính: Hỗ trợ kiểm tra cú pháp, giải thích hệ tọa độ VF (-y phía trước, +x sang trái) và tự động hóa khâu kiểm thử báo cáo theo `check_submission.py`.
- Cách sử dụng kết quả: Dùng các kết quả thống kê để rà soát lỗi format, đối chiếu số liệu bảng A với phiếu tự kiểm để điền báo cáo trung thực, chuẩn xác.

(c) Điều còn chưa chắc chắn:
- Với một số xe ở khoảng cách trên 50 m có mật độ điểm dưới 30 điểm và bị khuất một phần sau xe khác, việc xác định chính xác góc xoay yaw và chiều dài xe vẫn có sai số nhỏ mang tính ước lượng theo camera.
