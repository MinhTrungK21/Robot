# Hướng dẫn TRON 2 — Cấu hình Chân (2 chân Sole / Chân bánh xe Wheeled-legs)

> Tổng hợp từ tài liệu LimX Dynamics: *TRON 2 User Manual* (bản 2026-11-24, firmware V2.7), *Biped & Wheeled-Legged Quick Start V1.0*, *Autonomous Charging Dock User Manual*, *TRON 2 SDK Development Guide* (V1.3) và *Open-Source Bipedal Locomotion Example*.
> Khi có khác biệt, **tài liệu gốc của LimX là chuẩn**.

---

## Mục lục

1. [Tổng quan](#1-tổng-quan)
2. [An toàn — đọc trước khi dùng](#2-an-toàn--đọc-trước-khi-dùng)
3. [Lắp đặt, đổi chân ↔ bánh](#3-lắp-đặt-đổi-chân--bánh)
4. [Pin và sạc](#4-pin-và-sạc)
5. [Bật / tắt robot](#5-bật--tắt-robot)
6. [Các chế độ và trạng thái](#6-các-chế-độ-và-trạng-thái)
7. [Điều khiển bằng tay cầm](#7-điều-khiển-bằng-tay-cầm)
8. [Điều khiển qua trang web 8080](#8-điều-khiển-qua-trang-web-8080)
9. [Đế sạc tự động (chỉ chân bánh xe)](#9-đế-sạc-tự-động-chỉ-chân-bánh-xe)
10. [Kết nối mạng](#10-kết-nối-mạng)
11. [Lập trình — SDK cấp cao (WebSocket)](#11-lập-trình--sdk-cấp-cao-websocket)
12. [Lập trình — SDK cấp thấp](#12-lập-trình--sdk-cấp-thấp)
13. [Demo AI: Huấn luyện dáng đi bằng RL](#13-demo-ai-huấn-luyện-dáng-đi-bằng-rl)
14. [Nâng cấp, log, bảo dưỡng](#14-nâng-cấp-log-bảo-dưỡng)
15. [Tra cứu nhanh](#15-tra-cứu-nhanh)

---

## 1. Tổng quan

TRON 2 cấu hình chân là robot đi bằng **2 chân**, mỗi chân **5 bậc tự do**, có 2 loại bàn chân thay qua lại được:

| Cấu hình | Model (`ROBOT_TYPE`) | Đặc điểm |
|---|---|---|
| **Sole** (bàn chân) | `SF_TRON2A` | Đi bộ, đi ngang được |
| **Wheeled-legs** (chân bánh xe) | `WF_TRON2A` | Chạy bánh nhanh hơn, leo dốc tốt hơn, có **chế độ cầu thang** và **đế sạc tự động** |

### Thông số chính

| Hạng mục | Sole | Wheeled-legs |
|---|---|---|
| Kích thước (rộng × cao đứng) | 453 × 987 mm | 478 × 1015 mm |
| Tốc độ tối đa | 2–3 m/s | 3–5 m/s |
| Dốc tối đa | 15° | 30° |
| Bậc thang tối đa | 20 cm | 20 cm |
| Tải | 30 kg đất bằng; 20 kg khi leo cầu thang | như bên trái |
| Chức năng có sẵn | Đi 4 hướng, xoay, đứng yên, ngồi xổm tại chỗ, chỉnh chiều cao thân, tự đứng dậy khi ngã, phát hiện bị nhấc khỏi mặt đất | như bên trái |

Chung: pin 46,8 V – 9 Ah (thay được), IMU, camera RGBD ở eo (RealSense D435i). Máy tính dev (bản EDU): Intel Core Ultra, 2 TB, Ubuntu 22.04, ROS 1 Noetic + ROS 2 Humble.

### Danh sách khớp

| # | Khớp | Giới hạn (rad) | # | Khớp | Giới hạn (rad) |
|---|---|---|---|---|---|
| 0 | proximal_pitch_L | −3,14 → 2,60 | 5 | proximal_pitch_R | −3,14 → 2,60 |
| 1 | proximal_roll_L | −0,26 → 3,19 | 6 | proximal_roll_R | −3,19 → 0,26 |
| 2 | proximal_yaw_L | −3,66 → 1,48 | 7 | proximal_yaw_R | −1,48 → 3,66 |
| 3 | knee_L | −2,61 → 0,26 | 8 | knee_R | −2,61 → 0,26 |
| 4 | ankle_pitch_L | −1,69 → 1,69 | 9 | ankle_pitch_R | −1,69 → 1,69 |
| — | wheel_L (chỉ bánh xe) | quay liên tục | — | wheel_R (chỉ bánh xe) | quay liên tục |

---

## 2. An toàn — đọc trước khi dùng

- Chân robot chuyển động **nhanh và rất mạnh**. Chỉ người đã được hướng dẫn mới được vận hành.
- Không ai đứng trong vùng chuyển động; luôn nhìn thấy robot; cách vật cản, người, nước **≥ 1 m**.
- **Dùng dây treo an toàn (tether / palăng)** khi điều khiển lần đầu và khi thử thuật toán mới.
- Mặt sàn bằng phẳng, ma sát tốt. Cẩn thận với sàn trơn.
- −5 °C → 40 °C; **không chống nước/bụi**; không dùng ngoài trời khi sương mù, mưa, tuyết, giông; tránh nhiễu điện từ.
- Pin yếu → dừng và sạc ngay.

### Dừng khẩn cấp (E-Stop)

> ⚠️ E-Stop ngắt điện mọi motor ngay lập tức → **robot mất lực và ngã**. Giữ người và đồ dễ vỡ tránh xa.

| Cách | Thao tác | Kết quả |
|---|---|---|
| Nút E-Stop trên thân | Nhấn nút | Ngắt điện motor. Sau khi nhả nút phải **bật lại nguồn robot** |
| Tay cầm | Nhấn **đồng thời 2 joystick** | Ngắt điện motor. Nhấn **joystick phải** 1 lần để thoát → damping, vài giây sau tự phục hồi |
| Web 8080 | **Emergency Stop** (góc trên phải) | Ngắt điện motor |
| API | `request_emgy_stop` | Ngắt điện motor |

---

## 3. Lắp đặt, đổi chân ↔ bánh

### Video hướng dẫn chính thức

| Nội dung | Link |
|---|---|
| Mở thùng | https://oss.limxdynamics.com/uploads/material//TRON 2/video/1_unboxing_EN_output.mp4 |
| Lắp chân bánh xe | https://oss.limxdynamics.com/uploads/material//TRON 2/video/3_wheeled_foot_installation_EN_output.mp4 |
| Đổi bánh xe ↔ bàn chân | https://oss.limxdynamics.com/uploads/material//TRON 2/video/4_wheeled_foot_sole_switching_EN_output.mp4 |
| Lắp thanh chống lật (anti-roll bar) | https://oss.limxdynamics.com/uploads/material//TRON 2/video/5_anti_roll_bar_installation_EN_output.mp4 |
| Ghép đôi tay cầm | https://oss.limxdynamics.com/uploads/material//TRON 2/video/6_remote_controller_pairing_EN_output.mp4 |
| Điều khiển chân bánh bằng tay cầm | https://oss.limxdynamics.com/uploads/material//TRON 2/video/9_remote_controller_wheeled_foot_EN_output.mp4 |
| Cài đặt mạng | https://oss.limxdynamics.com/uploads/material//TRON 2/video/11_network_setup_EN_output.mp4 |
| Hiệu chuẩn điểm 0 | https://oss.limxdynamics.com/uploads/material//TRON 2/video/12_zero_calibration_EN_output.mp4 |

### Kiểm tra dây trước khi bật

> 🔴 **Không bao giờ cắm dây 24 V vào cổng 48 V.**

Dây thân robot ở cấu hình chân **giống hệt** cấu hình tay — đối chiếu với sơ đồ đấu dây Dual-Arm trong User Manual. Kiểm tra: đúng nhãn, đúng chiều, cắm chặt và khóa, dây không hỏng. Không chắc → không bật.

### Đổi cấu hình

Robot tự nhận cấu hình, không cần cài đặt.
- **Đổi tay ↔ chân:** tháo/lắp tại giao diện **hông/vai** (3 khớp đầu dùng chung).
- **Đổi bàn chân ↔ bánh xe:** thay cẳng chân tại giao diện **đầu gối**. Có thiết kế chống lắp sai (không lắp ngược bàn chân hay lắp bánh quay vào trong).
- **Tắt nguồn trước khi tháo/lắp.** Dùng lục giác chữ L 3,0 mm; cắm đầu nối hết rồi mới siết từng vít M4.
- **Thay vít mới sau 3 lần tháo/lắp** (vít M4×8 lục giác chìm). Vít lỏng → robot đi bị lệch hoặc mất kết nối.

---

## 4. Pin và sạc

- **Xem mức pin:** nhấn nhanh nút nguồn 1 lần.
- **Sạc pin rời:**
  1. Nối đế sạc pin với adapter (đầu aviation, đúng khóa).
  2. Cắm pin vào đế sạc, ấn chặt; cắm adapter vào ổ 220 V.
  3. Đèn pin nhấp nháy xanh lá + adapter xanh dương = đang sạc. Cả hai xanh lá đứng = đầy. Đèn pin đỏ nhấp nháy = lỗi.
- Thời gian: 20→80% ≈ 30 phút; 20→100% ≈ 54 phút.
- Cất lâu: để 50–60%, sạc lại 3 tháng/lần. Không cắm sạc liên tục sau khi đầy.
- **Tay cầm:** sạc Type-C 5 V/1 A; đèn xanh lá đứng = đang sạc. Pin hiện ở góc trên phải màn hình.
- **Chân bánh xe** có thể sạc tự động bằng đế sạc — xem mục 9.

---

## 5. Bật / tắt robot

### Tư thế sẵn sàng trước khi bật

Đặt robot trên sàn bằng, rộng, có ma sát, **tự đứng cân bằng** ở tư thế sẵn sàng:
- **Sole:** **đầu gối hướng ra trước**.
- **Wheeled-legs:** **đầu gối hướng ra sau**.

> ⚠️ **Không bao giờ bật ở tư thế khác** (chân dạng ra, bánh xoay ra ngoài, robot nằm nghiêng/ngửa). Đầu có thể đập xuống đất, chân có thể xoắn ra ngoài hoặc robot bị lật.

### Bật

1. Robot ở tư thế sẵn sàng, **nút E-Stop trên thân đã nhả**.
2. Cắm pin đã sạc đầy vào khoang, ấn đến khi khóa.
3. Nhấn nhanh nút nguồn **0,5 s**, rồi **giữ 3 s** → đèn mức pin sáng.
4. Giữ nút nguồn tay cầm **2 s** → nghe bíp, màn hình sáng.
5. Đèn trạng thái nhấp nháy **trắng**; sau ~20 s chuyển **xanh dương / xanh lơ / xanh lá đứng** = khởi động xong.
6. Màn hình tay cầm có vạch sóng xanh = đã kết nối.

> Không có tay cầm? Điều khiển qua `http://10.192.1.2:8080` (mục 8).

### Tắt

1. Nhấn **L1 + X** → robot **ngồi xổm, gập chân**, hạ xuống ổn định và vào Idle.
2. Giữ nút nguồn robot **3 s** → đèn tắt.
3. Giữ nút nguồn tay cầm 2 s.
4. Đèn tắt hết rồi mới rút pin.

> ⚠️ **Luôn L1 + X (Homing) trước khi tắt nguồn.** Tắt thẳng → robot mất lực và **rơi đột ngột** → hỏng.

### Ý nghĩa đèn trạng thái

| Màu đèn | Ý nghĩa |
|---|---|
| Trắng nhấp nháy chậm | Đang khởi động / tự kiểm tra |
| Đỏ đứng | Có lỗi |
| Đỏ nhấp nháy | Pin < 20% |
| Đỏ nhấp nháy nhanh | Pin < 5% |
| Vàng nhấp nháy | Đang nhấn E-Stop trên thân |
| Xanh dương đứng | Remote-Control Mode (Drive) |
| Xanh lơ (cyan) đứng | High-Level Developer Mode |
| Xanh lá đứng | Low-Level Developer Mode |
| Tím động (dynamic purple) | Chế độ cầu thang (chỉ bánh xe) |
| Vàng đứng | Phát hiện robot **bị ngã** |
| Vàng "thở" (breathing) | Phát hiện robot **bị nhấc khỏi mặt đất** |

---

## 6. Các chế độ và trạng thái

### Chế độ

| Chế độ (tên trên màn hình) | Mô tả |
|---|---|
| **Drive** (Remote-Control) | Điều khiển bằng tay cầm: đi tới/lui/ngang, xoay, chỉnh chiều cao… |
| **High-Level Dev** (chỉ EDU) | Gọi API cấp cao, dùng thuật toán đi có sẵn của LimX |
| **Low-Level Dev** (chỉ EDU) | Gọi API cấp thấp, tự điều khiển từng khớp (dùng cho thuật toán RL của bạn) |

- Thứ tự xoay vòng: **Drive → High-Level Dev → Low-Level Dev**.
- Chỉ đổi được khi đang **Idle**. Robot nhớ chế độ cuối trước khi tắt.

### Trạng thái trong chế độ Drive

```
Idle ──(vào Prepare)──► Prepare ──(đứng dậy)──► Walk ──(L1+X)──► ngồi xổm, gập chân ──► Idle
                                                  │
                                    Ngã ──(L2+…)──► đứng lại như trước khi ngã
Chỉ bánh xe:  Walk ─► Docking ─► Charging ─► Undocking ─► Walk
```

| Trạng thái | Ý nghĩa |
|---|---|
| `Idle` | Nghỉ, motor damping |
| `Prepare` | Chạm đất, ngẩng đầu, tư thế chuẩn bị |
| `Walk` | Đang đứng hoặc đi |
| `Docking` | Đang tự đi vào đế sạc |
| `Charging` | Đang trên đế, đang sạc / đã đầy |
| `Undocking` | Đang rời đế |

---

## 7. Điều khiển bằng tay cầm

### Màn hình

- **Trên trái:** vạch sóng kết nối. **Trên phải:** pin **tay cầm**.
- **Giữa:** tốc độ, chiều cao thân, `bat` = pin robot, `pwr` = nguồn, `errcode` = mã lỗi.
- **Dưới trái:** chế độ (`Drive`, …). **Dưới phải:** trạng thái (`Idle`, `Prepare`, `Walk`, …).

### Chuyển sang chế độ Drive

1. Đỡ robot an toàn, nhấn **L1 + □** → dưới phải hiện `Idle`.
2. Nhấn **R1 + →** (tiến) / **R1 + ←** (lùi) để chuyển chế độ.
3. Xong khi dưới trái hiện **`Drive`** và đèn **xanh dương đứng**.

### Quy trình đi cơ bản

1. Idle → **vào Prepare** (**L1 + ○**) → robot chạm đất, ngẩng đầu.
2. **Đứng dậy** (**L1 + △**) → robot từ từ đứng vào trạng thái `Walk` (mặc định ở chế độ đất bằng).
3. Dùng joystick để đi.
4. Xong: **L1 + X** → robot ngồi xổm, gập chân → Idle.

### Bảng lệnh

| Lệnh | Nút | Điều kiện | Ghi chú |
|---|---|---|---|
| Đổi chế độ | **R1 + →** / **R1 + ←** | Idle | |
| Vào Prepare | **L1 + ○** | Idle | Chạm đất, ngẩng đầu |
| Đứng dậy | **L1 + △** | Prepare | Đứng dần vào `Walk` |
| Homing (ngồi, gập chân) | **L1 + X** | Drive / High-Level Dev | Xong tự vào Idle |
| Vào Idle ngay | **L1 + □** | Mọi lúc | Bỏ qua Homing, motor damping, hạ từ từ. **Chỉ dùng khẩn cấp** (VD Homing lỗi) |
| Tiến / lùi | **Joystick trái** lên/xuống | Đang đứng | |
| Sang trái / phải | **Joystick trái** trái/phải | Đang đứng | **Chỉ Sole** |
| Xoay tại chỗ | **Joystick phải** trái/phải | Đang đứng | |
| Chỉnh chiều cao | Giữ **R1 + ↑ / ↓** | Đang đứng | Lên/xuống đều, nhả là dừng. Khởi động lại về chiều cao mặc định |
| Đổi đất bằng ↔ cầu thang | Giữ **Share 2 s** | Đang đứng | Khuyên dùng **bánh xe** cho cầu thang. Đèn tím động = đang ở chế độ cầu thang |
| Đứng dậy khi ngã | **L2 + △** | Đã ngã | Về lại trạng thái đứng trước khi ngã; không tác dụng khi bình thường |
| E-Stop | Nhấn đồng thời 2 joystick | Mọi lúc | Robot mất lực, **ngã** |
| Thoát E-Stop | Nhấn joystick phải | Đang E-Stop | Về damping |
| Hiệu chuẩn điểm 0 | **L1 + R1** | Idle | Chỉ khi vừa nâng main controller hoặc va đập mạnh làm lệch điểm 0 |
| Vào đế sạc | **L2 + X** | Walk, cách đế 200–650 mm | Chỉ bánh xe (mục 9) |
| Rời đế sạc | **L2 + ?** | Charging | Chỉ bánh xe — nút chưa có trong tài liệu; xem *Auto-Recharge Base User Manual* |

> Ký hiệu nút theo bản web của User Manual (https://www.limxdynamics.com/en/documents/852949782845591552). Riêng nút **rời đế sạc** chưa có trong User Manual — xem *TRON 2 Auto-Recharge Base User Manual* (https://www.limxdynamics.com/en/documents/882997113569546240). Xem đầy đủ cách dùng tay cầm ở [TRON2-tay-cam.md](TRON2-tay-cam.md).

### Leo cầu thang

- **Bánh xe:** khuyên dùng. Bật chế độ cầu thang (giữ Share 2 s), tiến với tốc độ đủ lớn (qua API phải > 1,5 m/s, nếu không robot không leo được).
- **Sole:** đòi hỏi chiều cao bậc, độ sâu bậc và ma sát phù hợp; **nguy cơ lật cao** — tránh dùng nếu không cần, luôn có dây bảo hộ khi thử.

---

## 8. Điều khiển qua trang web 8080

1. Kết nối robot (mục 10), mở `http://10.192.1.2:8080` → menu **Robot Control**.
2. Các nút: chọn chế độ (chỉ khi Idle), **Idle Mode** (vào Idle không qua Homing), **Zero Calibration**, **Emergency Stop** (góc trên phải).
3. Đổi chế độ: nếu chưa Idle → bấm **Idle Mode** → chờ `ST_IDLE` → chọn chế độ → kiểm tra màu đèn robot.

> Trang Robot Control chủ yếu mô tả cho cấu hình tay (nút Home cho 2 tay). Với chân, đưa robot về tư thế ngồi gập bằng **L1 + X** trên tay cầm hoặc `request_sitdown` qua API trước khi dùng Idle Mode.

---

## 9. Đế sạc tự động (chỉ chân bánh xe)

Đế sạc tiếp xúc: robot tự tìm đế, căn chỉnh, hạ xuống đế để sạc rồi tự rời đế — không cần cắm dây.

### 9.1 Thông số

| Hạng mục | Giá trị |
|---|---|
| Robot hỗ trợ | Chỉ TRON 2 Wheeled-legs |
| Kích thước | 600 × 721 × 423 mm (có robot: cao 692 mm) |
| Vùng tự tìm đế | Cách đế **200–650 mm**, trong góc **180°** phía trước |
| Thời gian vào đế trung bình | ~30 s |
| Đầu vào | 100–240 V AC, tối đa 8 A |
| Đầu ra | 54,2 V, 10 A |
| Thời gian sạc | 20→80% ≈ 31 phút; 20→100% ≈ 66 phút |
| Bảo vệ | Chống hồ quang, ngắn mạch, ngược cực, quá áp, quá dòng; chỉ cấp điện khi công tắc hành trình xác nhận tiếp xúc |
| Bảo hành | 12 tháng |

### 9.2 Lắp đặt

- Khoảng trống **bán kính 1 m** (nửa vòng tròn phía trước, tâm là tiếp điểm sạc) — không để thùng, dây, thiết bị.
- Sàn phẳng, cứng, khô, không dầu; không có gì kẹt dưới tấm đế.
- Hoạt động tốt khi thiếu sáng; **tránh nắng trực tiếp / đèn mạnh** (làm hỏng định vị bằng hình ảnh).
- Nhiều đế: cách nhau **≥ 1,1 m** theo chiều ngang và **≥ 1,4 m** theo chiều dọc (tính từ tâm tiếp điểm).
- Robot phải lắp **module Bluetooth** ở lưng.
- Mỗi đế có Bluetooth ID dạng `TRON2A_DOCK_XXXX` (quét mã QR ở lưng đế hoặc trên nhãn).

### 9.3 Đèn đế sạc

| Đèn nguồn | Ý nghĩa |
|---|---|
| Xanh lá đứng | Đã bật, đang chờ; hoặc robot trên đế và pin đã đầy |
| Cam đứng | Đang sạc |
| Đỏ nhấp nháy | Đế có lỗi |

4 đèn chiếu sáng phía trước chỉ bật khi đang vào/rời đế.

### 9.4 Sử dụng

**Bằng tay cầm:**
1. Đế đã bật (xanh lá). Robot đang đứng (`Walk`), pin đủ, đặt cách đế 200–650 mm, phía trước đế.
2. **L2 + X** → màn hình `Docking` → robot tự tiến vào, căn chỉnh, hạ xuống.
3. Vào đế xong → `Charging`, chân chuyển damping, robot ngồi lên đế; đèn đế cam.
4. Đầy 100% thì dừng sạc; robot ở trên đế thì đế tự duy trì đầy.
5. Rời đế: **L2 + ?** (nút chưa có trong tài liệu) → `Undocking` → robot đứng dậy, rời đế và đứng chờ lệnh.

> Nhấn nút mà chưa thấy phản ứng → chờ vài giây. Có nhiều đế mà chưa ghép đôi đế mặc định → robot có thể vào nhầm đế.

**Bằng API** (dùng cho tuần tra 24/7, tự sạc khi pin yếu): xem mục 11.3.

### 9.5 Xử lý sự cố

| Hiện tượng | Nguyên nhân | Cách xử lý |
|---|---|---|
| Robot không thấy đế | Ngoài vùng 200–650 mm; bị che; đế chưa bật | Đặt lại robot, dọn vật cản, kiểm tra đèn xanh |
| Mất dấu đế | Nắng trực tiếp; lỗi cảm biến | Cải thiện ánh sáng; liên hệ hỗ trợ nếu còn lỗi |
| Vào đế thất bại / chỉnh mãi | Sàn không phẳng, đế lệch, mã QR định vị bị che | Kiểm tra sàn, cố định đế, lau tiếp điểm, hiệu chỉnh lại vị trí đế |
| Đèn không sáng | Chưa cấp điện; aptomat nhảy | Kiểm tra dây nguồn, aptomat, điện áp |
| Đèn đỏ nhấp nháy | Đế phát hiện lỗi | Ngắt điện, tìm nguyên nhân; liên hệ hỗ trợ |

**Bảo dưỡng:** ngắt điện trước khi lau; lau vỏ bằng khăn mềm khô, không dùng hóa chất; giữ tiếp điểm sạch, không gỉ.

---

## 10. Kết nối mạng

| Thiết bị | IP | Ghi chú |
|---|---|---|
| Main controller robot | `10.192.1.2` | Web 8080, WebSocket 5000, log 8090 |
| Máy tính dev | `10.192.1.4` | `ssh guest@10.192.1.4`, mật khẩu `123456` |

- **Có dây:** cáp LAN vào cổng Ethernet ngoài; đặt IP máy tính `10.192.1.200` (hoặc cùng dải `10.192.1.x`), `ping 10.192.1.2`.
- **Không dây:** Wi-Fi của robot (tên dạng `WF_TRON2A_xxx` / `TRON2A_xxx`), mật khẩu `12345678`.
- **Cho robot vào Internet:** trang 8080 → biểu tượng Wi-Fi → nhập băng tần, SSID, mật khẩu (hoặc API `request_connect_wifi`).
- **Đặt model cho chương trình:**
  ```bash
  echo 'export ROBOT_TYPE=WF_TRON2A' >> ~/.bashrc && source ~/.bashrc   # hoặc SF_TRON2A
  ```
  Model đọc từ SN tại trang 8080 → *Robot Information* (VD SN `WF_TRON2A_001` → model `WF_TRON2A`).

---

## 11. Lập trình — SDK cấp cao (WebSocket)

> Cần **bản EDU**, robot ở **High-Level Dev** (đèn xanh lơ). Luồng lệnh API **giống hệt** luồng tay cầm, có thể chuyển qua lại giữa tay cầm và API.

### 11.1 Giao thức

- Địa chỉ: `ws://10.192.1.2:5000`
- Gói tin JSON: `accid` (SN robot), `title` (`request_xxx` / `response_xxx` / `notify_xxx`), `timestamp` (ms), `guid` (ID duy nhất, response trả về cùng guid), `data`.
- `notify_robot_info` gửi mỗi giây: SN, phiên bản phần mềm, tình trạng IMU/camera/motor, % pin, `status`:
  `STAND`, `WALK`, `SIT`, `DAMPING`, `ROTATE`, `STAIR`, `ERROR_FALLOVER` (đã ngã), `RECOVER` (đang đứng dậy), `ERROR_RECOVER` (đứng dậy thất bại).
- Mã lỗi thường gặp: `fail_imu`, `fail_motor`, `fail_status` (trạng thái hiện tại không cho phép), `fail_invalid_cmd`.

### 11.2 Lệnh di chuyển

| Lệnh | Chức năng | `data` | Notify |
|---|---|---|---|
| `request_stand_mode` | Đứng dậy | `{}` | `notify_stand_mode` khi xong/lỗi |
| `request_walk_mode` | Vào trạng thái đi | `{}` | `notify_walk_mode` |
| `request_twist` | Ra lệnh vận tốc — **gửi liên tục ≥ 30 Hz** | xem dưới | `notify_twist` khi lỗi |
| `request_base_height` | Nâng/hạ thân **5 cm mỗi lần gọi** | `{"direction": 1}` (lên) / `-1` (xuống) | — |
| `request_sitdown` | Ngồi xuống | `{}` | `notify_sitdown` |
| `request_stair_mode` | Bật/tắt chế độ cầu thang (**chỉ bánh xe**) | `{"enable": true}` | — |
| `request_recover` | Tự đứng dậy sau khi ngã, về trạng thái đi | `{}` | `notify_recover` (`success` / `fail_recover`); response `fail_no_fallover` nếu chưa ngã |
| `request_enable_imu` | Bật đẩy dữ liệu IMU | `{"enable": true}` | `notify_imu`: `euler` (độ), `acc`, `gyro`, `quat` |
| `request_emgy_stop` | Dừng khẩn cấp | `{}` | — |
| `request_light_effect` | Đổi đèn | xem SDK 3.5.4 | — |

**`request_twist` — ý nghĩa khác nhau theo cấu hình:**

| | Sole | Wheeled-legs |
|---|---|---|
| `x` | tỉ lệ tốc độ tiến/lùi, [−1, 1] | tốc độ tiến/lùi **m/s**, [−3,0; 3,0] (cầu thang: > 1,5) |
| `y` | tỉ lệ tốc độ đi ngang, [−1, 1] | luôn `0.0` (không đi ngang được) |
| `z` | tỉ lệ tốc độ xoay, [−1, 1] | tốc độ xoay **rad/s**, [−1,5; 1,5] |

**Trình tự điển hình:** `request_stand_mode` → chờ `notify_stand_mode` → `request_walk_mode` → chờ `notify_walk_mode` → gửi `request_twist` liên tục → dừng gửi/gửi 0 → `request_sitdown`.

### 11.3 Lệnh đế sạc (chỉ `WF_TRON2A`)

| Lệnh | Chức năng | `data` |
|---|---|---|
| `request_scan_docks` | Quét các đế xung quanh | `{}` → `docks: [{"dock_id": 4660}, ...]`, `count` |
| `request_get_docking_status` | Đọc trạng thái sạc chi tiết (chỉ đọc) | `{}` → `phase`, `robot_state`, `dock_id`, `battery_percent`, `failure_*`… |
| `request_start_docking` | Bắt đầu vào đế | `{"dock_id": 4660}` hoặc `{}` (đế đầu tiên tìm thấy) |
| `request_cancel_docking` | Hủy vào/rời đế | `{}` |
| `request_undock` | Rời đế | `{}` |

- `phase` lần lượt: `idle/walking → preparing → docking → confirming → charging → undock_preparing → undocking → walking`; lỗi → `failed`.
- `dock_id = 0` là ID hợp lệ (không phải "không biết"). `minimum_undock_battery_percent` = 15 — pin dưới mức này không cho rời đế.
- Lỗi `fail_dock_busy` = robot đang quét/ghép đôi/vào/rời đế; `fail_unsupported_robot` = không phải model WF.

### 11.4 Code mẫu Python

```bash
pip3 install websocket-client
```

```python
import json, time, uuid, threading
import websocket

ROBOT_IP = "10.192.1.2"
ACCID = None          # tự lấy từ notify_robot_info
events = {}

def send(ws, title, data=None):
    ws.send(json.dumps({
        "accid": ACCID, "title": title,
        "timestamp": int(time.time() * 1000),
        "guid": str(uuid.uuid4()), "data": data or {},
    }))

def on_message(ws, msg):
    global ACCID
    m = json.loads(msg)
    t = m.get("title")
    if t == "notify_robot_info":
        ACCID = m["accid"]
        return
    print(t, m.get("data"))
    if t in ("notify_stand_mode", "notify_walk_mode", "notify_sitdown"):
        events.setdefault(t, threading.Event()).set()

def wait(t, timeout=20):
    events.setdefault(t, threading.Event()).wait(timeout)

def on_open(ws):
    def run():
        while ACCID is None:
            time.sleep(0.1)
        send(ws, "request_stand_mode");  wait("notify_stand_mode")
        send(ws, "request_walk_mode");   wait("notify_walk_mode")
        # Đi tới chậm 3 giây, gửi 50 Hz
        # Sole: x là tỉ lệ [-1,1]; Wheeled-legs: x là m/s
        for _ in range(150):
            send(ws, "request_twist", {"x": 0.2, "y": 0.0, "z": 0.0})
            time.sleep(0.02)
        send(ws, "request_twist", {"x": 0.0, "y": 0.0, "z": 0.0})
        time.sleep(1)
        send(ws, "request_sitdown");     wait("notify_sitdown")
    threading.Thread(target=run, daemon=True).start()

ws = websocket.WebSocketApp(f"ws://{ROBOT_IP}:5000",
                            on_open=on_open, on_message=on_message)
ws.run_forever()
```

> Lần đầu chạy: robot treo dây bảo hộ, sàn trống, một người cầm tay cầm sẵn sàng E-Stop.

Ví dụ đầy đủ C++ / Python: [TRON_2_SDK_Development_Guide.md](../TRON_2_SDK_Development_Guide.md) mục 3.9.

---

## 12. Lập trình — SDK cấp thấp

> Cần **bản EDU**, robot ở **Low-Level Dev** (đèn xanh lá). Thuật toán đi có sẵn bị tắt — chương trình của bạn chịu trách nhiệm giữ thăng bằng. **Luôn treo robot bằng palăng khi thử.**

```bash
git clone https://github.com/limxdynamics/limxsdk-lowlevel.git
pip install limxsdk-lowlevel/python3/amd64/limxsdk-*-py3-none-any.whl     # x86_64
# aarch64: .../python3/aarch64/...   Windows: .../python3/win/...
```

| Hàm | Chức năng |
|---|---|
| `getInstance()` / `init(robot_ip)` | Kết nối robot |
| `getMotorNumber()` | Số motor |
| `subscribeRobotState(cb)` | `q` (rad), `dq` (rad/s), `tau` (N·m) |
| `subscribeImuData(cb)` | `acc`, `gyro`, `quat` (w,x,y,z) |
| `publishRobotCmd(cmd)` | Gửi lệnh khớp |
| `subscribeSensorJoy(cb)` | Đọc tay cầm: `axes[0]` trái-ngang, `axes[1]` trái-dọc, `axes[2]` phải-ngang, `axes[3]` phải-dọc; `buttons[4]` = L1, `buttons[7]` = R1 |
| `subscribeDiagnosticValue(cb)` | Chẩn đoán |
| `setRobotLightEffect(...)` | Đèn |

**Vào Low-Level Dev:** sau khi bật → **R1 + →** đến `High-level Dev` → **L1 + □** (`Idle`) → **R1 + →** đến `Low-level Dev` → đèn xanh lá. Chế độ này được nhớ sau khi tắt/bật.

Camera eo (RealSense D435i) qua ROS 1:
```bash
export ROS_MASTER_URI=http://10.192.1.2:11311
export ROS_IP=10.192.1.200
rostopic list && rqt_image_view
```

---

## 13. Demo AI: Huấn luyện dáng đi bằng RL

Quy trình 4 bước: **Training → Play → Sim2Sim → Sim2Real**, dùng **Isaac Lab 2.3.2 + Isaac Sim 5.1.0** để huấn luyện, **MuJoCo** để kiểm tra.

### 13.1 Yêu cầu máy

GPU NVIDIA ≥ 8 GB VRAM (khuyến nghị RTX 3090 24 GB), CPU ≥ 6 nhân, RAM ≥ 16 GB, ổ trống ≥ 30 GB, **Ubuntu 22.04+**, Python ≥ 3.10, truy cập được GitHub.

### 13.2 Huấn luyện

1. Cài Isaac Sim 5.1.0 → Isaac Lab 2.3.2 → Conda env (theo tài liệu gốc mục 2).
2. Clone code project và cài package (mục 3).
3. Train (mục 4.1) → Play để xem policy (mục 4.2).
4. Xuất ra `policy.onnx` + `encoder.onnx` + `params.yaml`.

### 13.3 Thử trong mô phỏng (Sim2Sim)

```bash
conda activate tron2_deploy
echo 'export ROBOT_TYPE=WF_TRON2A' >> ~/.bashrc && source ~/.bashrc   # hoặc SF_TRON2A
# Chép model vào: ~/limx_ws/tron2-rl-deploy-python/controllers/model/$ROBOT_TYPE/
#   params.yaml, policy.onnx, encoder.onnx

cd ~/limx_ws
python tron2-mujoco-sim/simulator.py                 # Terminal 1: simulator
python tron2-rl-deploy-python/main.py                 # Terminal 2: controller
./tron2-mujoco-sim/robot-joystick/robot-joystick      # Terminal 3: tay cầm ảo
```
Robot ngã → bấm **Reset** ở thanh bên trái simulator.

### 13.4 Chạy trên robot thật (Sim2Real)

1. Máy tính nối dây với robot, IP `10.192.1.200`, `ping 10.192.1.4`.
2. Chuyển robot sang **Low-Level Dev** (mục 12).
3. **Treo robot bằng palăng**, chân/bánh chạm đất.
4. Chạy:
   ```bash
   conda activate tron2_deploy
   python3 tron2-rl-deploy-python/main.py 10.192.1.4
   ```
   Khởi động là robot tự vào `prepare` — mọi khớp về điểm 0, tư thế đứng thẳng.
5. **L1 + △** = bật đi; joystick trái = tiến/lùi, xoay; joystick phải = đi ngang. **L1 + □** = tắt đi.

### 13.5 Cài vào máy dev trên robot

```bash
cd ~/limx_ws
scp -r tron2-rl-deploy-python guest@10.192.1.4:/home/guest           # mật khẩu 123456
ssh guest@10.192.1.4 "pip install /home/guest/tron2-rl-deploy-python/limxsdk-lowlevel/python3/amd64/limxsdk-*-py3-none-any.whl"
ssh guest@10.192.1.4
python3 /home/guest/tron2-rl-deploy-python/main.py 10.192.1.2
```
Tự chạy khi khởi động: sửa `/home/guest/autolaunch/autolaunch.sh` (bỏ `#` ở dòng chạy `main.py`) rồi `reboot`.

Chi tiết: [TRON_2_Open-Source_Bipedal_Locomotion_Example.md](../TRON_2_Open-Source_Bipedal_Locomotion_Example.md).

---

## 14. Nâng cấp, log, bảo dưỡng

### Firmware robot
> Không tắt nguồn trong khi nâng cấp.
1. `ping 10.192.1.2`.
2. Tải firmware tại https://www.limxdynamics.com/en/downloads (chọn TRON2).
3. Trang 8080 → **Version Management** → chọn loại → **Select file** → **Upgrade**.
4. **Firmware driver motor nâng 2 bước:** chọn tất cả motor → "Motor Driver Firmware" → Upgrade; chọn lại tất cả → "Motor Driver Firmware Supplementary Package" → Upgrade.
5. Sau khi nâng main controller → **hiệu chuẩn điểm 0** (Idle → **L1 + R1**; đèn đỏ = thất bại, làm lại; chờ ~5 s sau khi xong).

### Firmware tay cầm (chỉ Windows)
Tải tool → tắt tay cầm, cắm vào PC → giữ **joystick trái + Power** (Boot) → chọn COM → OPEN COM → OPEN FILE → UPGRADING. Xem phiên bản: nhấn đúp Power → mục đầu tiên → nút →.

### Xuất log
Wi-Fi robot → `http://10.192.1.2:8090` → xuất log + bag tại thời điểm lỗi.

### Bảo dưỡng
- Lau bằng khăn microfiber, cất trong thùng chuyên dụng.
- Vệ sinh bụi ở khớp; kiểm tra vít lỏng (đặc biệt vít M4 ở hông và gối).
- Pin: cất ở 50–60%, sạc lại 3 tháng/lần.
- Tay cầm: giữ cổng sạch, không để joystick bị đè khi cất.

---

## 15. Tra cứu nhanh

| Việc | Cách |
|---|---|
| Wi-Fi robot | mật khẩu `12345678` |
| Trang quản lý | `http://10.192.1.2:8080` |
| WebSocket API | `ws://10.192.1.2:5000` |
| Log | `http://10.192.1.2:8090` |
| SSH máy dev | `ssh guest@10.192.1.4` / `123456` |
| Tư thế trước khi bật | Sole: gối ra **trước** · Bánh xe: gối ra **sau** |
| Idle ngay | **L1 + □** |
| Đổi chế độ | Idle → **R1 + →/←** (Drive → High-Level Dev → Low-Level Dev) |
| Ngồi gập trước khi tắt | **L1 + X** |
| Tiến/lùi · ngang · xoay | Joystick trái dọc · joystick trái ngang (Sole) · joystick phải ngang |
| Chiều cao | Giữ **R1 + ↑/↓** |
| Cầu thang | Giữ **Share 2 s** |
| Vào đế sạc | **L2 + X** (cách đế 200–650 mm) |
| Hiệu chuẩn điểm 0 | Idle → **L1 + R1** |
| E-Stop | 2 joystick cùng lúc · nút trên thân · web · `request_emgy_stop` |
| Thoát E-Stop | Nhấn joystick phải |
