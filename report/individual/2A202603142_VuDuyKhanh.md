# Báo cáo Day 13 — Vũ Duy Khanh, 2A202603142

## 1. Kết luận trước

MSSV `2A202603142`, task `D13-2A202603142`, job 13. Chưa ghi nhận MSSV bạn ngồi cạnh. Môi trường thực hành gồm Windows, Microsoft Edge và CVAT Server 2.83.0. Mười frame theo thứ tự là `s01f09`, `s01f31`, `s02f24`, `s02f55`, `s03f10`, `s03f53`, `s04f18`, `s04f46`, `s05f26` và `s05f44`.

CVAT và bảng A khớp **108 cuboid**. Phiếu tự kiểm gắn cờ **10/108 hộp**. Frame `s01f31` ít nhất, chỉ có 4 nhãn Car, gồm một xe phía trước và ba xe phía sau. Chưa đủ bằng chứng để kết luận đã bỏ sót vật. Khoảng cách tới các xe gần có nhiều điểm là phần đáng tin hơn. Khó nhất là xác định đầu xe ở xa và phân biệt một xe lớn với nhiều vật nằm trong cùng hộp.

1. Trong khung Perspective, kéo chuột trái để xoay, kéo chuột phải để dời và cuộn để thu phóng. Xoay đến góc nhìn từ trên xuống để xem hai bên và phía sau xe.
2. Chọn **Cuboid → Shape**, chọn lớp và nháy đúp vào cụm điểm. Căn chiều dài, chiều rộng và yaw ở Top, rồi kiểm đáy và chiều cao ở Side và Front.
3. Bật **Appearance → Cuboid orientation** để xem vạch chỉ đầu hộp. Dùng `python scripts/lab.py preview` để đối chiếu hộp với các góc camera.

## 2. Đọc frame (Bước 2)

Ảnh của `s01f09` là `t1-s01f09-3d.jpg` và `t1-s01f09-top.jpg`. Ảnh của `s01f31` là `t1-s01f31-3d.jpg` và `t1-s01f31-top.jpg`. Khoảng cách trong bảng đo trên mặt phẳng ngang. Cao độ mặt đường là ước lượng từ các ô có điểm mặt đất trong vành 3–12 m quanh LiDAR.

| Frame | Phía trước xe ở cạnh nào của ảnh nhìn từ trên xuống; +x, +y chỉ về đâu | Mặt đường (phẳng/dốc; thấp hơn LiDAR ~ m) | Vật xa nhất đọc được (~ m) | Loại vật thấy | Vật phía sau / hai bên | Cụm chưa đọc được |
|---|---|---|---|---|---|---|
| `s01f09` | Trước gần mép trái, +x xuống, +y sang phải | Gần phẳng, thấp hơn LiDAR khoảng 1,74 m | Car #3 ở 59,43 m, ngoài phạm vi chấm 50 m. Trong 50 m, xa nhất là Car #8 ở 39,26 m | 10 Car, 1 Truck, 1 Bus | Truck #6 phía trái, Car #8 phía sau | Bus #7 ở 59,16 m có hộp rộng 5,322 m. Chưa xác định được biên hai bên. |
| `s01f31` | Trước gần mép trên, +x sang trái, +y xuống | Gần phẳng, thấp hơn LiDAR khoảng 1,73 m | Car #4 ở 44,57 m | 4 Car | #1 phía sau trái, #2 phía sau phải, #4 gần chính sau | Car #4 chỉ có 57 điểm trong hộp, khó đọc đầu và đuôi từ LiDAR. |

Camera cho thấy sắc xanh sẫm quanh tâm chiếu của Car #1 ở `s01f09`, sắc đỏ nâu ở vùng Truck #6 cùng frame và độ sáng khác nhau của vùng Car #4 giữa cam05 và cam09 ở `s01f31`. Đây là màu trên ảnh ở góc quan sát tương ứng. Với LiDAR, có thể đo trực tiếp khoảng cách ngang **4,99 m** đến Car #1, chiều cao hộp Truck #6 là **2,138 m** và cao độ mặt đường gần xe thu dữ liệu là khoảng **−1,73 m** ở `s01f31`.

Tám frame lướt. Hướng dưới đây ứng với góc Top suy từ phương vị mặc định của CVAT khi mở riêng từng frame. Xoay ngang khung Perspective sẽ làm hướng trên màn hình thay đổi.

| Frame | Phía trước ở cạnh nào của ảnh nhìn từ trên xuống | Điều lạ (nếu có) |
|---|---|---|
| `s02f24` | Phải, chếch xuống | Đáy Car #2 cao hơn mặt đường cục bộ 0,91 m. |
| `s02f55` | Trên | Car #5 phía sau cách 34,45 m, có 125 điểm. |
| `s03f10` | Phải | Car #1 ở 4,16 m, gần nhất toàn job. |
| `s03f53` | Trái | Car #4 và #5 vượt cả ba ngưỡng kích thước của bộ tự kiểm. |
| `s04f18` | Trên | Có 19 hộp. Phiếu tự kiểm không gắn cờ hộp nào. |
| `s04f46` | Trên phải | Car #8 phía trước cách 47,90 m, chỉ có 60 điểm. |
| `s05f26` | Trên phải | Car #4 cách 62,46 m, ngoài phạm vi chấm 50 m. |
| `s05f44` | Trên trái | Có một cụm điểm dưới mặt đường, minh họa ở mục 4. |

## 3. Ba cuboid có hướng dẫn (Bước 3)

Ba ảnh mẫu lần lượt là `t2-a-car.jpg`, `t2-b-bus-truck.jpg` và `t2-c-ped-cyclist.jpg`. `idx` là số thứ tự **trong frame** ở bảng A. Độ lệch trục dưới đây so yaw với trục dài ước từ các điểm phía trên đáy hộp 0,22 m. Phép so này còn mơ hồ 180° khi phân biệt đầu và đuôi.

| Vật | Frame, idx | Class | Mũi tên yaw đúng hướng? | Đáy chạm đường? | Đã kiểm Top/Side/Front + ảnh? | Kích thước trong bảng 4.3? |
|---|---|---|---|---|---|---|
| (a) xe con gần | `s01f09`, 1 | Car | 269,4°, lệch trục cụm điểm khoảng 9° | Đáy thấp hơn đường cục bộ 0,06 m | Có ảnh các hình chiếu. Tâm hộp chiếu vào cam01 và cam06 | So với sedan, chiều rộng thiếu 0,020 m. Dài và cao trong khoảng. |
| (b) xe buýt hoặc xe tải | `s01f09`, 6 | Truck | 90,0°, lệch trục khoảng 4° | Đáy thấp hơn đường cục bộ 0,22 m | Có ảnh các hình chiếu. Tâm hộp chiếu vào cam05 và cam06 | 5,876 × 2,418 × 2,138 m. So với xe tải hộp nhẹ, chiều cao thấp hơn cận dưới 0,262 m. Chưa xác nhận loại thân xe. |
| (c) người đi bộ hoặc xe hai bánh | `s03f53`, 7 | Cyclist | 268,6°, lệch trục khoảng 1,6° | Đáy cao hơn đường cục bộ 0,01 m | Có ảnh các hình chiếu. Tâm hộp chiếu vào cam01 và cam07 | So với xe đạp có người, chiều cao thiếu 0,026 m. Dài và rộng trong khoảng. |

Chưa có ghi nhận về độ lệch giữa các bạn trong vòng hiệu chuẩn hoặc quyết định chung của lớp. Với vùng khuất sau Car #3 ở `s03f53`, chưa đủ điểm để nhận diện thêm vật. Các vật đã bỏ qua trong lúc vẽ không xuất hiện trong bảng A nên chưa xác định được danh sách đầy đủ.

## 4. Bốn hiện tượng chất lượng (Bước 4)

Cả bốn ảnh lấy từ job 13 của task cá nhân `D13-2A202603142`.

| Hiện tượng | Frame / tên ảnh / nguồn | Nguyên nhân (giả thuyết) | Thấy gì trên point cloud | Hệ quả khi gán (theo guideline) |
|---|---|---|---|---|
| 1. Bóng che | `s03f53` · `t4-1-bong-che.jpg` · cá nhân | Car #3 có thể chắn tia LiDAR. | Xe ở 6,82 m. Sau xe có một dải trống, trong khi hai dải bên vẫn có điểm. | Nếu nhận diện được vật bị che một phần, bật `occluded` và bao phần suy ra được. Dải trống tự nó chưa đủ để vẽ thêm hộp. |
| 2. Thưa ở xa | `s01f31` · `t4-2-thua-o-xa.jpg` · cá nhân | Ở xa, hai tia liên tiếp có thể rơi cách nhau xa hơn trên thân xe. | Car #4 ở 44,57 m có 57 điểm, so với 1.381 điểm của Car #2 ở 8,78 m. | Vẽ khi thấy khoảng 15–20% thể tích hoặc đủ căn cứ từ hình dạng và camera. Nếu quá ít bằng chứng, ghi `LOW_VISIBILITY`. |
| 3. Phản xạ thấp / ít điểm bất thường | `s05f44` · `t4-3-phan-xa-thap.jpg` · cá nhân | Vật liệu bề mặt hoặc góc tới có thể khiến tín hiệu phản hồi yếu. | Car #1 ở 29,61 m có 170 điểm. Car #3 ở 27,09 m có 318 điểm. Ở phần cao hơn đáy 0,22 m, cường độ trung bình lần lượt khoảng 0,12 và 1,27 theo thang dữ liệu gốc. | Đối chiếu biên xe trên camera trước khi chỉnh hộp. Chênh lệch tín hiệu chưa đủ để kết luận màu sơn hay vật liệu. |
| 4. Điểm lạ | `s05f44` · `t4-4-diem-la.jpg` · cá nhân | Cụm này có thể do nhiễu đo hoặc tín hiệu phản xạ gián tiếp. | Bảy điểm tập trung quanh z ≈ −11,9 m, thấp hơn mặt đường lân cận khoảng 10 m. | Bỏ qua cụm điểm rời dưới đường khi chưa có dấu hiệu của một vật thật. |

## 5. Số từ cuboid của tôi (Bước 5)

Tọa độ, kích thước và yaw lấy từ `D13-2A202603142-bangA.csv`. Số điểm và độ lệch so với mặt đường quanh hộp lấy từ `D13-2A202603142-tukiem.csv`, ghép theo `frame, idx`. Trong bảng sau, tất cả độ dài tính bằng mét, yaw tính bằng độ. Đáy trong vòng ±0,30 m quanh mốc −1,70 m được xếp là gần mặt đường theo quy ước của bài.

| Cuboid (frame, idx, class) | z | h | z đáy = z − h/2 | dist_m | r = √(x²+y²+z²) | yaw_deg | Nhận xét (lơ lửng / chìm / chạm đường) |
|---|---|---|---|---|---|---|---|
| `s01f09` #1 Car | −1,085 | 1,541 | −1,856 | 4,99 | 5,108 | 269,4 | Gần mặt đường. Độ lệch cục bộ −0,06 m. |
| `s01f09` #6 Truck | −0,906 | 2,138 | −1,975 | 14,70 | 14,730 | 90,0 | Gần mặt đường. Độ lệch cục bộ −0,22 m. |
| `s03f53` #7 Cyclist | −0,952 | 1,574 | −1,739 | 11,41 | 11,450 | 268,6 | Gần mặt đường. Độ lệch cục bộ +0,01 m. |

Trong 108 hộp, gần nhất là **Car `s03f10` #1**, cách ngang **4,16 m**, khoảng cách 3D **4,277 m**. Xa nhất là **Car `s05f44` #6**, cách ngang **72,93 m**, khoảng cách 3D **72,939 m**, ngoài phạm vi chấm 50 m. Khoảng cách 3D có tính cả chênh cao giữa tâm hộp và LiDAR.

Car `s01f09` #1 đo được **4,405 × 1,680 × 1,541 m** theo thứ tự dài × rộng × cao. Chiều rộng thấp hơn cận sedan 1,70 m. Cyclist `s03f53` #7 đo được **1,660 × 0,518 × 1,574 m**. Chiều cao thấp hơn cận 1,60 m của xe đạp có người. Các chênh lệch nhỏ có thể do biên điểm hoặc tư thế người lái. Về hướng, Car #1 ở phía trước có yaw 269,4°, lệch 0,6° so với hướng cùng chiều 270°. Truck #6 cùng frame có yaw 90,0°, đúng hướng ngược chiều theo nhãn. Một frame tĩnh chưa xác nhận được hướng chuyển động thực.

Mật độ theo khoảng cách. Bốn mẫu đều là Car trong `s01f09`. `points_in` là số điểm bên trong mỗi hộp.

| Cuboid (frame, idx, class) | dist_m | Số điểm (`points_in`) |
|---|---|---|
| `s01f09` #1 Car | 4,99 m | 2.722 |
| `s01f09` #5 Car | 18,00 m | 759 |
| `s01f09` #2 Car | 23,07 m | 248 |
| `s01f09` #8 Car | 39,26 m | 59 |

Từ 4,99 m tới 39,26 m, khoảng cách tăng **7,87 lần**, số điểm giảm từ **2.722 xuống 59**, còn **2,17%**. Bốn mẫu giảm cùng chiều với các mốc trong bài giảng (khoảng 1.968, 718, 214 và 72 điểm cho các dải 0–10, 10–20, 20–30 và 30–40 m). Mỗi dải ở đây chỉ có một xe minh họa.

Tự kiểm có tám hộp bị cờ kích thước và hai hộp bị cờ đáy. Các cờ kích thước gồm `s01f09` #7, #12, `s03f53` #4, #5, `s05f26` #3, #4 và `s05f44` #5, #7. Đáng chú ý nhất là hai Car ở `s03f53` dài 8,792 m và 12,479 m. Ở `s05f26` #4, chiều rộng gốc 2,303 m chỉ vượt ngưỡng 2,300 m đúng 0,003 m. Hai cờ đáy là `s02f24` #2 (+0,91 m) và `s05f44` #19 (+0,46 m) so với đường cục bộ. Bản xuất hiện tại chưa có cặp số trước và sau để xác nhận hai lần sửa cuboid. Các cờ trên vẫn cần phán đoán theo biên vật, lớp và mặt đường tại chỗ.

**Chưa thực hiện đổi kiểm với bạn cùng lớp.** Chưa có MSSV người kiểm chéo hoặc hai số đo được đối chiếu theo cặp.

## 6. Ý nghĩa của nhãn, cách dùng AI và điều còn chưa chắc

Nhãn 3D tính bằng mét giúp mô hình học vị trí và kích thước thật của chiếc xe, dù cùng chiếc xe đó chiếm ít pixel hơn khi ở xa. Với vật chỉ có vài điểm LiDAR, nhiều hộp có thể cùng bao được cụm điểm ấy, nên cần quy tắc riêng để mô hình tránh học theo mép hộp hoặc hướng xe do người gán nhãn đoán.

- **Công cụ** — ChatGPT.
- **Câu hỏi chính** — Theo hướng dẫn gán nhãn, khi một hộp bị cảnh báo về kích thước hoặc vị trí đáy, cần xem lại điểm nào trên CVAT để quyết định giữ hay sửa?
- **Kết quả đã dùng** — Dùng Codex để đối chiếu cảnh báo với CVAT. 

Hai hộp lớn mang nhãn Car ở `s03f53` vẫn là các ca chưa chắc nhất. Kích thước lớn có thể do chọn sai lớp hoặc bao dư sang vật bên cạnh. Cần nhìn được ranh giới giữa các vật và đầu xe mới phân biệt được hai khả năng. Chưa có ghi nhận riêng về lỗi phát hiện sau hạn.
