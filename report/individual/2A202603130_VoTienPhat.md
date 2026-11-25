# Báo cáo Day 13 — Võ Tiến Phát, 2A202603130

## 1. Kết luận trước

Mã số sinh viên: 2A202603130. Task: D13-2A202603130. Mười frame được giao: s01f19, s01f43, s02f18, s02f51, s03f00, s03f43, s04f25, s04f54, s05f01, s05f45. Môi trường thực hiện: macOS, trình duyệt Google Chrome, CVAT 3D (user `admin`). Bạn ngồi cạnh (đổi kiểm): chưa ghi nhận MSSV đối chiếu trực tiếp.

Tổng cộng đã gán nhãn 93 cuboid trên 10 frame (`s01f19=8`, `s01f43=7`, `s02f18=8`, `s02f51=5`, `s03f00=8`, `s03f43=5`, `s04f25=22`, `s04f54=10`, `s05f01=9`, `s05f45=11`). Hai frame có ít cuboid nhất là `s02f51` và `s03f43` (đều 5 cuboid) do đường thoáng hoặc xe ở xa ít điểm. Phiếu tự kiểm (`selfcheck`) gắn cờ 81/93 cuboid, chủ yếu do `width` hơi rộng hơn ngưỡng tham chiếu 2,3 m của `Car` (vùng 2,33–3,17 m) hoặc `ground_gap_m` lệch âm quanh -0,2 m đến -0,86 m. Phần tin tưởng nhất là phân loại class (`Car`, `Truck`, `Bus`) và hướng mũi tên (`yaw_deg`) ở cự ly gần (< 30 m). Phần còn yếu nhất là ước lượng chiều dài, chiều rộng sát thân xe (một số nhóm xe copy hộp ở `s03f00`, `s04f25` hơi dài/rộng hơn xe thật) và các xe ở xa > 40–50 m bị thưa điểm.

Cách dùng CVAT:
- Đổi góc nhìn: Kéo chuột trái để xoay khung 3D, kéo chuột phải để dời khung hình, lăn chuột để phóng to/thu nhỏ; xoay thẳng từ trên xuống để xem toàn cảnh Top view.
- Vẽ và chỉnh cuboid: Chọn công cụ Shape (không dùng Track), chọn label rồi nháy đúp đặt hộp; chọn cuboid để chỉnh tâm và yaw ở ô Top, chỉnh độ cao và đáy chạm đường ở ô Side và Front.
- Bật hướng và xem camera: Bật `Appearance -> Cuboid orientation` để thấy vạch đỏ `+X` chỉ đầu xe; đối chiếu ảnh camera (và `python scripts/lab.py preview`) để kiểm tra hướng xe và class.

## 2. Đọc frame (Bước 2)

Hai frame đọc kỹ: `s01f19`, `s01f43`. Ảnh: `t1-s01f19-3d.png`, `t1-s01f19-top.png`, `t1-s01f43-3d.png`, `t1-s01f43-top.png`.

| Frame | Phía trước xe ở cạnh nào của ảnh nhìn từ trên xuống; +x, +y chỉ về đâu | Mặt đường (phẳng/dốc; thấp hơn LiDAR ~ m) | Vật xa nhất đọc được (~ m) | Loại vật thấy | Vật phía sau / hai bên | Cụm chưa đọc được |
|---|---|---|---|---|---|---|
| s01f19 | Trên ảnh `t1-s01f19-top.png`, phía trước xe (-y) ở **cạnh trên**; +x chỉ sang **trái**, +y chỉ xuống **cạnh dưới** (ra sau) | Mặt đường tương đối phẳng, thấp hơn LiDAR khoảng 1,7 m | ~ 49,6 m (`s01f19 #3`, `Car`) | Car, Bus | Một xe buýt phía trước bên trái (`#6`), một số xe con phía sau (`#2`, `#3`, `#4`, `#5`, `#7`) và bên phải/trái | Các vệt điểm lề đường, tán cây và dải phân cách xa > 50 m |
| s01f43 | Trên ảnh `t1-s01f43-top.png`, phía trước xe (-y) ở **cạnh trên**; +x chỉ sang **trái**, +y chỉ xuống **cạnh dưới** (ra sau) | Mặt đường phẳng, thấp hơn LiDAR khoảng 1,7 m | ~ 39,1 m (`s01f43 #4`, `Car`) | Car | Nhiều xe con đi cùng chiều phía sau (`#1`, `#3`, `#4`) và phía trước bên trái (`#2`, `#6`, `#7`) | Cụm tán cây và biển báo dọc hai bên vỉa hè ở xa |

Ba điều **chỉ camera** cho biết:
1. Màu sắc thân xe, chữ viết và ký hiệu nhận diện (ví dụ chữ trên xe cứu thương, biển chỉ dẫn).
2. Tín hiệu đèn giao thông, vạch kẻ đường phẳng trên mặt nhựa và đèn phanh/đèn xi-nhan của xe.
3. Chi tiết bề mặt phẳng không có độ dày hình học (biển báo mỏng, dải sơn phân làn).

Ba điều **chỉ LiDAR** cho biết:
1. Tọa độ 3D theo đơn vị mét (`x, y, z`) và khoảng cách thực tế chính xác theo các vòng quét quanh xe.
2. Hình khối 3D và kích thước vật lý (dài, rộng, cao) không bị biến dạng bởi phối cảnh 2D của ống kính.
3. Khả năng quan sát 360° đồng thời cả các vùng tối, ngược sáng hoặc góc khuất giữa hai camera cạnh nhau.

Tám frame lướt:

| Frame | Phía trước ở cạnh nào của ảnh nhìn từ trên xuống | Điều lạ (nếu có) |
|---|---|---|
| s02f18 | Cạnh dưới (theo góc nhìn chụp trong `t2-a-car.png`) / hướng `-y` | Đường nhiều xe con đi ngược chiều dọc theo dải phân cách giữa |
| s02f51 | Hướng `-y` (mặc định cạnh trên nếu chưa xoay ngược) | Đường khá thoáng, chỉ có 5 xe con trong tầm quan sát |
| s03f00 | Cạnh dưới (theo góc nhìn chụp trong `t4-1-bong-che.png`) / hướng `-y` | Xe dừng đèn đỏ sát nhau; có xe cứu thương ở làn bên trái trên `cam03` nhưng ít điểm ở đuôi xe |
| s03f43 | Cạnh trên (theo góc nhìn chụp trong `t4-4-diem-la.png`) / hướng `-y` | Có xe rất gần (`#3`, `dist_m = 3,55 m`) và xe rất xa (`#5`, `dist_m = 62,05 m`) |
| s04f25 | Cạnh dưới (theo góc nhìn chụp trong `t4-3-phan-xa-thap.png`) / hướng `-y` | Mật độ phương tiện rất đông (22 xe), gồm cả xe đi cắt ngang ở ngã tư (`yaw ≈ 0°`) |
| s04f54 | Cạnh dưới (theo góc nhìn chụp trong `t4-2-thua-o-xa.png`) / hướng `-y` | Nhiều xe ở hai chiều và một số xe ở cự ly 40–55 m rất thưa điểm |
| s05f01 | Cạnh trên (theo góc nhìn chụp trong `t2-c-ped-cyclist.png`) / hướng `-y` | Có xe tải `Truck` (`#3`) ở cự ly `11,06 m` ngay phía sau xe thu dữ liệu |
| s05f45 | Hướng `-y` (góc chụp xoay chéo trong `t2-b-bus-truck.png`) | Tại ngã tư có nhiều xe tải `Truck` và xe con đi cắt ngang (`yaw ≈ 11°–15°`) |

## 3. Ba cuboid có hướng dẫn (Bước 3)

Ảnh: `t2-a-car.png`, `t2-b-bus-truck.png`, `t2-c-ped-cyclist.png`.

| Vật | Frame, idx | Class | Mũi tên yaw đúng hướng? | Đáy chạm đường? | Đã kiểm Top/Side/Front + ảnh? | Kích thước trong bảng 4.3? |
|---|---|---|---|---|---|---|
| (a) xe con gần | `s01f19`, `#1` (lưu ý `t2-a-car.png` chụp khung `s02f18`) | Car | Đúng (`yaw = 268,3°`, chỉ theo chiều tiến của xe) | Chạm đường (`z_bottom = -2,007 m`, thấp hơn -1,7 m khoảng 0,3 m) | Đã đối chiếu camera (`cam01`/`cam03`) | `l=4,59 m`, `w=2,47 m`, `h=2,00 m` (`w` hơi rộng hơn 1,70–2,10 m trong bảng 4.3) |
| (b) xe buýt hoặc xe tải | `s05f45`, `#1` (khớp ảnh `t2-b-bus-truck.png`) | Truck | Đúng (`yaw = 14,5°`, xe đi cắt ngang ngã tư) | Chạm đường (`z_bottom = -1,686 m`, rất sát -1,7 m) | Đã kiểm đủ Top/Side/Front trên ảnh chụp | `l=7,66 m`, `w=3,03 m`, `h=4,04 m` (phù hợp xe tải nặng; `w` hơi rộng hơn 2,60 m) |
| (c) người đi bộ hoặc xe hai bánh | Không có trong 10 frame của `2A202603130` (`t2-c-ped-cyclist.png` chụp `s05f01`) | Không có | Không áp dụng (10 frame chỉ có Car, Truck, Bus) | Không áp dụng | Đã rà soát lại cả 10 frame | Không có dữ liệu trong bảng A |

Vòng hiệu chuẩn và vật bỏ qua:
- Vòng hiệu chuẩn: Dùng task chung `D13-HC485087` để làm quen hệ tọa độ VF (`-y` là phía trước, `+x` sang trái) và kiểm tra lỗi hay gặp khi xoay ngược vạch đỏ `+X` của cuboid so với đầu xe.
- Vật bỏ qua vì không đủ cơ sở: Trong `s03f00` (`cam03`, nhóm xe dừng đèn đỏ bên trái) có một xe cấp cứu thân cao nhưng chỉ thu được vài chục điểm ở phần đuôi xe nên khó ước lượng chính xác chiều dài và chiều cao từ đám mây điểm (`LOW_VISIBILITY`); ngoài ra không gán các cụm điểm mờ quá xa > 50–60 m.

## 4. Bốn hiện tượng chất lượng (Bước 4)

| Hiện tượng | Frame / tên ảnh / nguồn | Nguyên nhân (giả thuyết) | Thấy gì trên point cloud | Hệ quả khi gán (theo guideline) |
|---|---|---|---|---|
| 1. Bóng che | `s03f00` / `t4-1-bong-che.png` (10 frame của bạn) | Các xe con dừng đèn đỏ sát nhau và dải tường rào chắn chùm tia laser chiếu tiếp ra phía sau | Phía sau hàng xe dừng và sau tường quây xuất hiện vùng tối hình nan quạt không có điểm LiDAR | Theo guideline, phần xe bị che khuất một phần cần bật `occluded` và vẽ cuboid bao trọn phần kích thước suy luận được (trong bảng A hiện vẫn để `occluded = 0`) |
| 2. Thưa ở xa | `s04f54` / `t4-2-thua-o-xa.png` (10 frame của bạn) | Góc phân kỳ giữa các tia laser cố định nên càng ra xa mật độ tia trên mét vuông càng giảm mạnh | Xe ở khoảng `40–55 m` như `#8` (`40,58 m`, 89 điểm), `#3` (`50,00 m`, 54 điểm), `#9` (`54,71 m`, 46 điểm) chỉ còn vài đường quét thưa | Chỉ vẽ khi thấy khoảng 15–20% thể tích hoặc nhận ra trên camera và dùng kích thước bảng 4.3; không đủ cơ sở thì bỏ qua (`LOW_VISIBILITY`) |
| 3. Phản xạ thấp / ít điểm bất thường | `s04f25` / `t4-3-phan-xa-thap.png` (10 frame của bạn) | Xe màu tối, kính chắn gió nghiêng hoặc góc tới xiên làm tia hồng ngoại hấp thụ/phản xạ lệch khỏi đầu thu | Thân xe bị khuyết điểm ở giữa, chỉ lộ các điểm ở viền hông, cản hoặc bánh xe dù ở cự ly không quá xa | Kết hợp đối chiếu ảnh camera để xác nhận sự có mặt của xe, không thu gọn kích thước hộp theo phần điểm bị khuyết |
| 4. Điểm lạ | `s03f43` / `t4-4-diem-la.png` (10 frame của bạn; có thể đối chiếu thêm `s02f35` ở `D13-HC485087`) | Phản xạ đa đường hoặc tán xạ bề mặt khiến LiDAR tính sai thời gian bay của tia | Xuất hiện một số điểm lẻ nằm tách biệt dưới mặt đường hoặc lơ lửng không thuộc thân xe nào | Không gán cuboid cho các điểm lạ và không kéo đáy hộp của xe thật xuống chạm vào các điểm nhiễu dưới mặt đất |

## 5. Số từ cuboid của tôi (Bước 5)

Lấy từ `submission/D13-2A202603130-bangA.csv`; số điểm lấy từ cột `points_in` của `submission/D13-2A202603130-tukiem.csv`. Mặt đường khoảng −1,7 m: `z_bottom` lệch hơn 0,3 m so với −1,7 m là dấu hiệu lơ lửng hoặc chìm.

| Cuboid (frame, idx, class) | z | h | z đáy = z − h/2 | dist_m | r = √(x²+y²+z²) | yaw_deg | Nhận xét (lơ lửng / chìm / chạm đường) |
|---|---|---|---|---|---|---|---|
| `s01f19`, `#1`, Car | -1,005 m | 2,004 m | -2,007 m | 4,89 m | 4,99 m | 268,3° | Hơi chìm (~0,31 m thấp hơn mốc -1,7 m) |
| `s01f19`, `#2`, Car | -0,667 m | 2,416 m | -1,876 m | 29,57 m | 29,58 m | 273,3° | Chạm đường (lệch -0,18 m, trong ngưỡng 0,3 m) |
| `s01f19`, `#3`, Car | -0,930 m | 2,420 m | -2,140 m | 49,62 m | 49,63 m | 273,6° | Chìm nhẹ (~0,44 m thấp hơn mốc -1,7 m) |

(a) Cuboid **gần nhất** và **xa nhất**:
- Cuboid gần nhất: `s03f43`, `#3`, Car với `dist_m = 3,55 m` và `r = 3,67 m`.
- Cuboid xa nhất: `s03f43`, `#5`, Car với `dist_m = 62,05 m` và `r = 62,06 m`.

(b) Kích thước `l, w, h` so với bảng 4.3:
- Xe con (`s01f19`, `#1`, Car): `l = 4,586 m`, `w = 2,469 m`, `h = 2,004 m`. So với bảng 4.3 (`Car` SUV/minivan: dài `4,4–5,3 m`, rộng `1,80–2,10 m`, cao `1,65–2,10 m`), chiều dài và chiều cao nằm trong khoảng chuẩn, còn chiều rộng `2,469 m` vượt ngưỡng trên khoảng `0,37 m` do bao cả phần gương/điểm tán xạ hai bên hông xe.
- Người đi bộ hoặc xe hai bánh: trong 10 frame được giao của `2A202603130` không xuất hiện `Pedestrian` hay `Cyclist` trong bảng A (`0` cuboid), nên chưa có số đo thực tế từ bảng A để đối chiếu.

(c) Góc `yaw_deg` của xe cùng chiều và ngược chiều:
- Xe cùng chiều phía trước (`s01f19`, `#1`, Car): `yaw_deg = 268,3°`, rất sát giá trị lý thuyết `270°` (hướng `-Y`).
- Xe đi ngược chiều (`s01f19`, `#4`, Car): `yaw_deg = 84,5°` và (`s01f19`, `#8`, Car): `yaw_deg = 92,2°`, đều sát giá trị lý thuyết `90°` (hướng `+Y`). (Ngoài ra tại ngã tư như `s04f25` và `s05f45` có các xe đi cắt ngang với `yaw_deg ≈ 0°`, `11,4°–14,5°`, `116,3°`).

Mật độ theo khoảng cách (4 cuboid ở các `dist_m` khác nhau):

| Cuboid (frame, idx, class) | dist_m | Số điểm (`points_in`) |
|---|---|---|
| `s01f19`, `#1`, Car | 4,89 m | 2805 |
| `s01f19`, `#5`, Car | 10,29 m | 1534 |
| `s01f19`, `#2`, Car | 29,57 m | 148 |
| `s01f19`, `#3`, Car | 49,62 m | 66 |

Quy luật mật độ và tự kiểm:
Khi khoảng cách tăng từ `4,89 m` lên `10,29 m` (gần gấp đôi), số điểm giảm từ `2805` xuống `1534` điểm (còn ~55%); khi tăng lên `29,57 m` (~6 lần), số điểm giảm xuống còn `148` điểm (giảm ~19 lần), phù hợp xu hướng suy giảm nhanh theo khoảng cách của đường cong slide bài giảng (`~1968` ở `0–10 m`, `~718` ở `10–20 m`, `~214` ở `20–30 m`, `~72` ở `30–40 m`).

Những chỉnh sửa đã thực hiện và cảnh báo giữ nguyên:
- Chỉnh sửa trên CVAT trước khi chốt bảng A: chuyển toàn bộ cuboid lỡ vẽ dạng `Track` sang `CUBOID SHAPE`, xoay lại hướng trục `+X` bị gắn ngược ở các frame đầu, và bổ sung thêm cuboid ở `s03f43` (từ 3 lên 5 cuboid gồm `#4` ở `37,53 m` và `#5` ở `62,05 m`) để đạt tối thiểu 5 cuboid/frame.
- Cảnh báo `selfcheck` giữ nguyên (81/93 cuboid): phần lớn là cảnh báo `width` hơi vượt ngưỡng 2,3 m của `Car` (nhiều hộp lấy rộng `2,33–2,98 m` để bao trọn cả gương và vùng điểm quanh xe) và một số hộp có `ground_gap_m` lệch xuống `-0,26 m` đến `-0,86 m` do độ dốc mặt đường ở xa.

Đổi kiểm:
- Chưa thực hiện đổi kiểm chéo trực tiếp với bạn bên cạnh trong buổi lab.

## 6. Ý nghĩa của nhãn, cách dùng AI và điều còn chưa chắc

(a) Từ phía mô hình học trên nhãn này:
- Nhãn 3D bắt buộc phải đo bằng mét trong hệ tọa độ thực chứ không phải pixel vì mô hình nhận thức của xe tự hành cần biết vị trí, kích thước hình học và khoảng cách vật lý thực để phục vụ dự đoán quỹ đạo và phanh/tránh va chạm; pixel 2D chỉ là phép chiếu phụ thuộc tiêu cự và góc máy ảnh.
- Vật thưa điểm cần quy tắc gán nhãn riêng vì khi chỉ còn vài điểm LiDAR ở xa hoặc bị che khuất, dữ liệu hình học không đủ ràng buộc kích thước; nếu gán tùy tiện theo vài điểm rời sẽ tạo nhãn méo lệch khiến mô hình học sai kích thước vật thể.

(b) Cách dùng AI:
- Công cụ: Trợ lý AI tích hợp trong IDE.
- Câu hỏi chính: Hướng dẫn thiết lập môi trường `.venv`, lệnh `scripts/lab.py`, giải thích hệ trục tọa độ VF (`+x` trái, `-y` trước, `+z` lên), hỗ trợ script chuyển đổi cuboid từ `Track` sang `Shape` qua `cvat-sdk` khi bị vẽ nhầm chế độ, và chạy kiểm tra `scripts/check_submission.py`.
- Cách sử dụng kết quả: Dùng các lệnh kiểm tra để đảm bảo đúng cấu trúc file nộp, đồng bộ bảng A và phiếu tự kiểm, đối chiếu số liệu CSV vào báo cáo.

(c) Điều còn chưa chắc chắn:
- Một số cuboid được copy kích thước giữa các xe cùng hàng (`s03f00`, `s04f25`) có chiều rộng/dài hơi dư so với kích thước xe thực tế; một vài xe xa > 50 m (`s03f43 #5`, `s04f25 #20–#22`, `s04f54 #3, #9`) thưa điểm nên độ chính xác hướng `yaw` và vị trí tâm chỉ mang tính ước lượng.
