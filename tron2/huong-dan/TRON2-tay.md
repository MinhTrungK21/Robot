# Hướng dẫn TRON 2 — Cấu hình Tay (Dual-Arm / Mobile Dual-Arm)

> Tổng hợp từ tài liệu LimX Dynamics: *TRON 2 User Manual* (bản 2026-11-24, firmware V2.7), *TRON 2 SDK Development Guide* (V1.3) và *Candy Dispensing – Dual-Arm Manipulation Quickstart (Pi05)*.
> Khi có khác biệt, **tài liệu gốc của LimX là chuẩn**.

---

## Mục lục

1. [Tổng quan](#1-tổng-quan)
2. [An toàn — đọc trước khi dùng](#2-an-toàn--đọc-trước-khi-dùng)
3. [Lắp đặt và kiểm tra dây](#3-lắp-đặt-và-kiểm-tra-dây)
4. [Pin và sạc](#4-pin-và-sạc)
5. [Bật / tắt robot](#5-bật--tắt-robot)
6. [Các chế độ hoạt động](#6-các-chế-độ-hoạt-động)
7. [Điều khiển bằng tay cầm (Remote Controller)](#7-điều-khiển-bằng-tay-cầm-remote-controller)
8. [Điều khiển qua trang web 8080](#8-điều-khiển-qua-trang-web-8080)
9. [Điều khiển từ xa bằng VR (Teleoperation)](#9-điều-khiển-từ-xa-bằng-vr-teleoperation)
10. [Kết nối mạng](#10-kết-nối-mạng)
11. [Lập trình — SDK cấp cao (WebSocket)](#11-lập-trình--sdk-cấp-cao-websocket)
12. [Lập trình — SDK cấp thấp](#12-lập-trình--sdk-cấp-thấp)
13. [Demo AI: Phát kẹo bằng Pi0.5](#13-demo-ai-phát-kẹo-bằng-pi05)
14. [Nâng cấp, log, bảo dưỡng](#14-nâng-cấp-log-bảo-dưỡng)
15. [Tra cứu nhanh](#15-tra-cứu-nhanh)

---

## 1. Tổng quan

TRON 2 cấu hình tay gồm thân robot với **2 tay 7 bậc tự do** và **đầu 2 bậc tự do**. Có 2 phiên bản:

| Phiên bản | Mô tả |
|---|---|
| **Dual-Arm** (đế cố định) | Robot gắn trên giá đỡ cố định, cấp nguồn ngoài hoặc pin |
| **Mobile Dual-Arm** | Robot gắn trên **đế di động (Mobile Base)** có **giá nâng (Lift Stand)** |

### Thông số chính

| Hạng mục | Giá trị |
|---|---|
| Bậc tự do | Mỗi tay 7 DOF, đầu 2 DOF |
| Kích thước | Rộng vai 447 mm, chiều dài tay 730 mm (không tính kẹp) |
| Tải đầu tay | Tối đa 5 kg/tay; khi duỗi thẳng 3 kg/tay |
| Tốc độ đầu tay tối đa | 5 m/s |
| Gia tốc đầu tay tối đa | 36 m/s² |
| Độ lặp lại vị trí | ±0,5 mm |
| Độ trễ teleop | ~100 ms |
| Đầu tay (end-effector) | Kẹp 2 ngón LimX (lực 20 N, mở 85 mm) hoặc bàn tay khéo léo BrainCo Revo 2 |
| Camera | RGBD ở đầu, cổ tay (2 bên), eo; có IMU |
| Pin | 46,8 V – 9 Ah, lithium ternary, thay được |
| Máy tính dev (bản EDU) | Intel Core Ultra, 2 TB, Ubuntu 22.04, ROS 1 Noetic + ROS 2 Humble |
| Model / SN | Dạng `DACH_TRON2A_xxx` |

### Danh sách khớp (số thứ tự dùng trong SDK)

| # | Khớp | Giới hạn (rad) | # | Khớp | Giới hạn (rad) |
|---|---|---|---|---|---|
| 0 | proximal_pitch_L | −3,141 → 2,600 | 7 | proximal_pitch_R | −3,141 → 2,600 |
| 1 | proximal_roll_L | −0,261 → 2,967 | 8 | proximal_roll_R | −2,967 → 0,261 |
| 2 | proximal_yaw_L | −3,665 → 1,483 | 9 | proximal_yaw_R | −1,483 → 3,665 |
| 3 | elbow_L | −2,617 → 0,523 | 10 | elbow_R | −2,617 → 0,523 |
| 4 | wrist_yaw_L | −1,745 → 1,396 | 11 | wrist_yaw_R | −1,396 → 1,745 |
| 5 | wrist_pitch_L | −0,750 → 0,750 | 12 | wrist_pitch_R | −0,750 → 0,750 |
| 6 | wrist_roll_L | −0,750 → 0,750 | 13 | wrist_roll_R | −0,750 → 0,750 |
| 14 | head_pitch | −0,78 → 1,04 | 15 | head_yaw | −1,57 → 1,57 |

---

## 2. An toàn — đọc trước khi dùng

- Chỉ người đã được hướng dẫn mới được vận hành. Không để trẻ em lại gần.
- Giữ khoảng cách **tối thiểu 1 m** với robot khi đang chạy; luôn nhìn thấy robot.
- Trước khi chạy demo / phát lại quỹ đạo: **dọn sạch vật cản trong bán kính 1 m** — tay robot có thể va vào.
- Không chạm vào khớp ngay sau khi bật nguồn hoặc khi vận chuyển (nguy cơ kẹp tay).
- Không để robot bật mà không có người trông.
- Có cảnh báo pin yếu → tắt robot và sạc ngay.
- Môi trường: −5 °C → 40 °C, khô ráo, không bụi, tránh nguồn nhiễu điện từ / Wi-Fi trùng kênh. Robot **không chống nước, chống bụi**.

### Dừng khẩn cấp (E-Stop)

| Cách | Thao tác | Kết quả |
|---|---|---|
| Nút E-Stop trên thân | Nhấn nút | Ngắt điện mọi motor. Sau khi nhả nút phải **bật lại nguồn robot** |
| Tay cầm | Nhấn **đồng thời cả 2 cần joystick** | Ngắt điện mọi motor, **toàn bộ tay sẽ rơi xuống**. Nhấn **joystick phải** 1 lần để thoát, robot về trạng thái damping |
| Tay cầm VR | Nhấn đồng thời 2 joystick trên 2 tay cầm VR | Như trên; Mobile Base cũng dừng ngay |
| Web 8080 | Nút **Emergency Stop** góc trên phải | Ngắt điện mọi motor |
| API | `request_emgy_stop` | Ngắt điện mọi motor |

> ⚠️ **Tắt máy đúng cách:** luôn đưa robot về trạng thái nghỉ (L1+X hoặc nút **Home** trên web) trước khi nhấn nút nguồn. Nhấn nút nguồn trực tiếp → các khớp rơi tự do → có thể hỏng robot.

---

## 3. Lắp đặt và kiểm tra dây

### Video hướng dẫn chính thức

| Nội dung | Link |
|---|---|
| Mở thùng | https://oss.limxdynamics.com/uploads/material//TRON 2/video/1_unboxing_EN_output.mp4 |
| Lắp tay | https://oss.limxdynamics.com/uploads/material//TRON 2/video/dual_arm_installation2_EN.mp4 |
| Ghép đôi tay cầm | https://oss.limxdynamics.com/uploads/material//TRON 2/video/6_remote_controller_pairing_EN_output.mp4 |
| Điều khiển tay bằng tay cầm | https://oss.limxdynamics.com/uploads/material//TRON 2/video/8_remote_controller_arm_EN_output.mp4 |
| Teleop 2 tay | https://oss.limxdynamics.com/uploads/material//TRON 2/video/10_dual_arm_teleoperation_EN_output.mp4 |
| Cài đặt mạng | https://oss.limxdynamics.com/uploads/material//TRON 2/video/11_network_setup_EN_output.mp4 |
| Hiệu chuẩn điểm 0 | https://oss.limxdynamics.com/uploads/material//TRON 2/video/12_zero_calibration_EN_output.mp4 |
| Đóng gói | https://oss.limxdynamics.com/uploads/material//TRON 2/video/7_packing_EN_output.mp4 |

### Kiểm tra bắt buộc trước khi cấp điện

> 🔴 **NGUY HIỂM:** Tuyệt đối **không cắm dây 24 V vào cổng 48 V**.

1. Robot, adapter nguồn, Mobile Base đều đang **tắt**.
2. Đối chiếu nhãn dây với nhãn cổng theo sơ đồ đấu dây (đặc biệt là dây nguồn).
3. Đúng chiều đầu cắm, **không ép**.
4. Mọi đầu cắm cắm hết và khóa chặt, không lỏng, không lộ chân kim loại.
5. Dây không bị hỏng, biến dạng, cháy xém, có mùi lạ.
6. Không chắc chắn → **không bật**, liên hệ hỗ trợ LimX.

**Riêng Mobile Dual-Arm:** bó dây truyền thông (USB 3.0 + RS485) luồn qua xích dẫn cáp của ray, cắm vào cổng truyền thông Mobile Base bằng đầu nối aviation. Cắm dây sạc vào cổng sạc của Mobile Base.

> **Phải lắp đầu robot và cắm dây nguồn + dây truyền thông khớp đầu trước khi bật**, nếu không robot không khởi động được.

### Đổi tay ↔ chân

Tay và chân dùng chung 3 khớp đầu tiên tại giao diện **vai/hông**; robot tự nhận cấu hình.
- **Tắt nguồn** trước khi tháo/lắp.
- Dùng lục giác chữ L 3,0 mm, cắm đầu nối hết rồi mới siết từng vít M4.
- **Thay vít mới sau 3 lần tháo/lắp** (vít M4×8 lục giác chìm). Vít lỏng → mất kết nối.

---

## 4. Pin và sạc

### Pin robot

- **Xem mức pin:** nhấn nhanh nút nguồn 1 lần.
- **Sạc:**
  1. Nối đế sạc pin với adapter (đầu aviation, đúng khóa).
  2. Cắm pin vào đế sạc, ấn cho chặt; cắm adapter vào ổ 220 V.
  3. Đèn pin nhấp nháy xanh lá + đèn adapter xanh dương = đang sạc. Cả hai xanh lá đứng = đầy.
- Thời gian: 20→80% ≈ 30 phút; 20→100% ≈ 54 phút.
- Đèn pin đỏ nhấp nháy = lỗi sạc.
- Không dùng lâu: sạc về **50–60%** để cất, **3 tháng sạc lại 1 lần**. Không cắm sạc liên tục sau khi đầy.

### Tay cầm

- Xem pin ở góc trên phải màn hình.
- Sạc qua cáp Type-C, nguồn 5 V/1 A. Đèn xanh lá đứng = đang sạc.

### Mobile Base (chỉ bản Mobile Dual-Arm)

- Điện áp bình thường 24–29,4 V. LED đế **nhấp nháy đỏ = pin yếu**, sạc ngay.
- Sạc bằng bộ sạc 10 A đi kèm, cắm vào cổng sạc phía sau đế.

---

## 5. Bật / tắt robot

### Bật

1. Lắp đặt xong theo mục 3.
2. Cắm pin đã sạc đầy vào khoang, ấn đến khi khóa.
   - **Bản đế cố định:** nối adapter vào cổng nguồn ngoài + ổ 220 V trước khi bật.
   - **Bản Mobile Base:** bật công tắc nguồn phía sau đế.
3. Nhấn nhanh nút nguồn robot **0,5 s**, rồi **giữ 3 s** → đèn mức pin sáng.
4. Giữ nút nguồn tay cầm **2 s** → màn hình sáng.
5. Đèn trạng thái nhấp nháy **trắng** khi khởi động; sau ~20 s chuyển sang **xanh dương / xanh lơ / xanh lá đứng** = khởi động xong.
6. Màn hình tay cầm có vạch sóng màu xanh = đã kết nối.

> Không dùng tay cầm? Bỏ qua bước 4, 6 và điều khiển qua web `http://10.192.1.2:8080` (mục 8).

### Tắt

1. Nhấn **L1 + X** (hoặc **Home** trên web) → 2 tay về tư thế buông thõng, robot vào trạng thái nghỉ.
2. Bản đế cố định: rút nguồn ngoài. Bản Mobile Base: tắt công tắc phía sau đế.
3. Nhấn nhanh 0,5 s rồi giữ 3 s nút nguồn robot → đèn tắt.
4. Giữ nút nguồn tay cầm 3 s.

### Ý nghĩa đèn trạng thái

| Màu đèn | Ý nghĩa |
|---|---|
| Trắng nhấp nháy chậm | Đang khởi động / tự kiểm tra |
| Đỏ đứng | Có lỗi |
| Đỏ nhấp nháy | Pin < 20% |
| Đỏ nhấp nháy nhanh | Pin < 5% |
| Vàng nhấp nháy | Đang nhấn E-Stop trên thân |
| Xanh tím (blue-violet) | Show Mode (Motion) |
| Xanh dương đứng | VR Teleoperation Mode |
| Xanh lơ (cyan) đứng | High-Level Developer Mode |
| Xanh lá đứng | Low-Level Developer Mode |

**Đèn Mobile Base:** đỏ đứng = lỗi; đỏ nhấp nháy = pin < 15%; xanh lá nhấp nháy chậm = đang sạc; vàng đứng = đỗ (4 bánh khóa hình chữ X); xanh dương = chờ/đi thẳng.

---

## 6. Các chế độ hoạt động

| Chế độ (tên trên màn hình) | Mô tả |
|---|---|
| **Motion** (Show Mode) | Demo động tác có sẵn; **Hand Guide** — cầm tay kéo để ghi và phát lại quỹ đạo |
| **Teleop** (VR Teleoperation) | Điều khiển bằng kính VR PICO 4 Ultra; dùng để teleop và **thu dữ liệu** |
| **High-Level Dev** (chỉ EDU) | Gọi API cấp cao: MoveJ, MoveP, ServoJ, ServoP… dùng thuật toán điều khiển có sẵn |
| **Low-Level Dev** (chỉ EDU) | Gọi API cấp thấp, điều khiển trực tiếp từng khớp |

- Thứ tự xoay vòng: **Motion → Teleop → High-Level Dev → Low-Level Dev**.
- Chỉ đổi được chế độ khi đang ở trạng thái **Idle**.
- Robot nhớ chế độ cuối cùng trước khi tắt.

---

## 7. Điều khiển bằng tay cầm (Remote Controller)

### Màn hình

- **Trên trái:** vạch sóng (kết nối với robot).
- **Trên phải:** pin **tay cầm** (không phải pin robot).
- **Giữa:** phản hồi từ robot — `bat` = pin robot, `pwr` = trạng thái nguồn, `errcode` = mã lỗi.
- **Dưới trái:** chế độ (`Motion`, `Teleop`, `High-Level Dev`, `Low-Level Dev`).
- **Dưới phải:** trạng thái (`Idle`, `Active`, `Show`, `Hand Guide`).

### Cách đổi chế độ

1. Đỡ robot an toàn, nhấn **L1 + □** → trạng thái `Idle`.
2. Nhấn **R1 + →** (tiến) hoặc **R1 + ←** (lùi) để chuyển chế độ, mỗi lần nhấn 1 chế độ.
3. Kiểm tra tên chế độ ở dưới trái.

> Màn hình đã báo `Idle` mà không đổi được → nhấn **L1 + □** thêm 1 lần rồi thử lại.

### Bảng lệnh

| Lệnh | Nút | Điều kiện | Ghi chú |
|---|---|---|---|
| Đổi chế độ | **R1 + →** / **R1 + ←** | Idle | |
| Về tư thế nghỉ (Home) | **L1 + X** | Chế độ Teleop / High-Level Dev | 2 tay vung ra rồi buông thõng; nếu bật bảo vệ an toàn sẽ có động tác tránh mặt bàn. Xong tự vào Idle (motor damping) |
| Vào Idle trực tiếp | **L1 + □** | Mọi lúc | Không có động tác Home; motor damping, tay **từ từ hạ xuống**. Dùng khi Home lỗi |
| Vào Show (demo) | **L1 + ⚠️** | Idle của chế độ Motion | Dọn sạch 1 m quanh robot trước |
| Vào Hand Guide | **L1 + ⚠️** | Idle của chế độ Motion | Sau đó: **R1 + X** bắt đầu / dừng ghi; **R1 + ⚠️** phát lại quỹ đạo vừa ghi |
| E-Stop | Nhấn đồng thời 2 joystick | Mọi lúc | |
| Thoát E-Stop | Nhấn joystick phải | Đang E-Stop | Robot về trạng thái damping |
| Hiệu chuẩn điểm 0 | **L1 + R1** | Idle | Chỉ khi vừa nâng cấp main controller hoặc va đập mạnh làm lệch điểm 0 |

> ⚠️ **Cần xác nhận:** Ký hiệu nút hình học (□ △ ○ …) trong PDF User Manual là hình ảnh nên không trích ra được. **L1 + □ = Idle** đã được xác nhận từ tài liệu khác của LimX. Nút cho Show / Hand Guide / Replay hãy xem bảng 4.2.2 – 4.4 trong `1795487510662_TRON_2_User_Manual.pdf` rồi điền vào.

### Hiệu chuẩn điểm 0 (Zero Calibration)

1. Nhấn **L1 + X** (hoặc **Home** trên web) → 2 tay buông thõng, Idle.
2. Bản Mobile: **nâng giá lên vị trí cao nhất** trước.
3. Nhấn **L1 + R1** (hoặc **Zero Calibration** trên web). Xong thì 2 tay buông thõng.
4. Đèn chuyển đỏ = thất bại → làm lại.
5. Chờ ~5 s cho robot cập nhật điểm 0 mới rồi mới thao tác tiếp.

---

## 8. Điều khiển qua trang web 8080

Dùng thay tay cầm; kết hợp web + tay cầm VR thì **không cần tay cầm**.

1. Kết nối máy tính với robot (mục 10), mở `http://10.192.1.2:8080`.
2. Menu trái → **Robot Control**. Phía trên hiện trạng thái kết nối, trạng thái và chế độ hiện tại.

| Nút | Chức năng |
|---|---|
| **VR Teleoperation / Developer / Low-level Developer / Demo & Teaching** | Chuyển thẳng sang chế độ đó (chỉ khi đang Idle) |
| **Home** | 2 tay về tư thế buông thõng → Idle |
| **Idle Mode** | Vào Idle không qua Home (motor damping, tay hạ từ từ) |
| **Zero Calibration** | Hiệu chuẩn điểm 0 |
| **Emergency Stop** (góc trên phải) | Ngắt điện mọi motor |

**Đổi chế độ:** nếu chưa Idle → bấm **Home** hoặc **Idle Mode** → chờ trạng thái `ST_IDLE` → bấm chế độ cần dùng → kiểm tra đèn robot đã đổi màu.

**Demo & Teaching** (sau khi chuyển sang Demo & Teaching Mode):
- **Motion Demo:** robot chạy động tác mẫu.
- **Drag Teaching:**
  1. **Start recording** → cầm tay robot kéo theo động tác → bấm lại để dừng. Quỹ đạo tự lưu vào danh sách bên trái.
  2. **Rename:** chọn quỹ đạo, nhập tên mới, bấm Rename.
  3. **Replay:** chọn quỹ đạo, bấm Replay (chạy 1 lần).
  4. **Xóa:** bấm biểu tượng thùng rác.
- Muốn dừng chuyển động: **Home**. Khẩn cấp: **Emergency Stop**.

---

## 9. Điều khiển từ xa bằng VR (Teleoperation)

Thiết bị: kính **PICO 4 Ultra** + 2 tay cầm VR.

### 9.1 Chuẩn bị

1. **Hiệu chuẩn điểm 0** nếu cần (mục 7).
2. **Chuyển sang chế độ Teleop:**
   - Tay cầm: **L1 + □** → `Idle` → **R1 + →/←** đến khi màn hình hiện `Teleop` + `Active`.
   - Hoặc web 8080: **Idle Mode** → chờ `ST_IDLE` → **VR Teleoperation**.
3. **Thiết lập kính VR lần đầu** (không bóc miếng dán che cảm biến trên kính, nếu không kính hay tự ngủ):
   1. Giữ nút nguồn bên phải kính ~3 s, đeo kính.
   2. Settings → kết nối Wi-Fi `TRON2A_XXX_5G` (hoặc `_2.4G`), mật khẩu `12345678`.
   3. **Library → Unknown Sources → Limx_teleop**.
   4. Chọn **Tron2**, IP `10.192.1.2`, bấm **Connect**.
   5. Chọn **PICO Controllers**, bật **Head Follow** (camera đầu robot quay theo kính), bật hình camera cổ tay trái/phải. Nếu hỏi body following → chọn **No**.
4. Teleop trong mạng nội bộ: nên để **kính VR dùng Wi-Fi 5 GHz của robot**. Nếu robot cần ra Internet thì cho robot nối mạng ngoài qua **2,4 GHz** để tránh nhiễu (gây trễ hình, giật).

### 9.2 Nút tay cầm VR

> **Nút Grip là "công tắc cho phép"**: mọi lệnh VR (trừ nút thu dữ liệu) chỉ có tác dụng khi đang giữ ít nhất 1 nút Grip.

**Lệnh chung**

| Hành động | Nút | Ghi chú |
|---|---|---|
| Bắt đầu teleop | Giữ Grip trên một tay cầm bất kỳ | Đang ở High-Level Dev mà giữ Grip → tự chuyển sang Teleop. Show / Low-Level thì không |
| Kết thúc teleop | Nhả Grip cả 2 tay | Quay về chế độ trước đó |
| Bắt đầu thu dữ liệu | **Y** (tay trái) | Không cần giữ Grip |
| Dừng thu dữ liệu | **B** (tay phải) | |
| Hủy bản ghi đang thu | Nhấn **Y** lần nữa | Xóa gói dữ liệu vừa ghi |
| E-Stop | Nhấn đồng thời 2 joystick | |
| Thoát E-Stop | Nhấn joystick phải | |

**Tay robot**

| Hành động | Nút |
|---|---|
| Tay bám theo tay cầm | Giữ **Grip** trái/phải → tay tương ứng bám theo (điều khiển theo độ dời tương đối) |
| Khóa tay | Nhả Grip → tay đứng yên tại chỗ |
| Đóng/mở kẹp | Giữ Grip + bóp **Trigger** cùng bên; bóp càng sâu kẹp càng đóng, nhả trigger = mở |
| Giữ độ mở kẹp | Đang bóp trigger thì nhả Grip → kẹp giữ nguyên, tay khóa |
| Tay trái về tư thế teleop ban đầu | Giữ **Grip trái + X** (khuỷu gập 90°, cẳng tay nằm ngang) |
| Tay phải về tư thế teleop ban đầu | Giữ **Grip phải + A** |
| Cả 2 tay về Home (buông thõng) | Giữ **Grip phải + B** |

> Về tư thế: phải giữ nút đến khi tay tới nơi; nhả giữa chừng là tay dừng ngay.
> Robot 1 tay mặc định là tay phải, chỉ dùng tay cầm phải.

**Đầu robot** (cần bật Head Follow): giữ Grip → đầu robot cúi/ngẩng/quay theo kính. **Grip cả 2 + X + A** = đầu về thẳng.

**Mobile Base và Lift Stand** (chỉ bản Mobile)

| Hành động | Nút |
|---|---|
| Tiến / lùi / sang ngang / chéo | Giữ **Grip trái** + đẩy **joystick trái** (đẩy càng xa càng nhanh) |
| Xoay tại chỗ | Giữ **Grip phải** + đẩy **joystick phải** trái/phải |
| Nâng / hạ thân | Giữ **Grip phải** + đẩy **joystick phải** tới/lui |
| Đỗ (khóa 4 bánh hình X) | Giữ **joystick phải** 1 s |

### 9.3 Mẹo khi teleop

- Trước khi bắt đầu: đặt tư thế tay mình giống tư thế tay robot.
- Lần đầu: **cử động nhỏ, chậm**. Trễ ~100 ms nên cần đoán trước.
- **Nhả Grip trước khi đi lại**, nếu không 2 tay robot sẽ chạy theo người.
- Di chuyển xa: nhả Grip → đưa tay cầm về → giữ Grip lại và tiếp tục (tay không nhảy về vị trí tay cầm).
- Joystick trái và phải độc lập — có thể vừa tiến vừa xoay.

### 9.4 Teleop qua Internet

1. Cho robot nối Wi-Fi ngoài (mục 10.3). Robot và kính VR đều phải có Internet.
2. Đăng ký tài khoản tại `http://robonexus.limxdynamics.com/` → lấy **Access Key** ở trang thông tin cá nhân.
3. Mở `http://10.192.1.2:9191` → nhập địa chỉ platform + Access Key → lưu.
4. Ghi lại **SN** của robot ở trang 8080 → *Robot Information*.
5. Trên kính: mở app teleop → chọn kết nối mạng ngoài → cấu hình **tron2-online** → nhập Access Key + SN → **Connect**.

---

## 10. Kết nối mạng

| Thiết bị | IP | Ghi chú |
|---|---|---|
| Main controller robot | `10.192.1.2` | Web 8080, WebSocket 5000, log 8090, cấu hình teleop từ xa 9191 |
| Máy tính dev (Developer Expansion Module) | `10.192.1.4` | `ssh guest@10.192.1.4`, mật khẩu `123456` |

### 10.1 Có dây
Cắm cáp LAN vào cổng Gigabit phía sau robot. Đặt IP máy tính cùng dải, ví dụ `10.192.1.120` (subnet 255.255.255.0).

### 10.2 Không dây
Wi-Fi `TRON2A_XXX_2.4G` hoặc `TRON2A_XXX_5G`, mật khẩu `12345678`.

### 10.3 Cho robot vào Internet
Mở `http://10.192.1.2:8080` → biểu tượng Wi-Fi bên phải → nhập băng tần, SSID, mật khẩu, mật khẩu admin router.
(Hoặc dùng API `request_connect_wifi`.)

---

## 11. Lập trình — SDK cấp cao (WebSocket)

> Cần **bản EDU** và robot ở chế độ **High-Level Dev** (đèn xanh lơ).

### 11.1 Giao thức

- Địa chỉ: `ws://10.192.1.2:5000`
- Gói tin JSON:

```jsonc
{
  "accid": "DACH_TRON2A_001",      // SN robot (xem ở trang 8080)
  "title": "request_movej",        // request_xxx / response_xxx / notify_xxx
  "timestamp": 1672373633989,      // ms
  "guid": "746d937c...",           // ID duy nhất; response trả về cùng guid
  "data": { ... }
}
```

- `response_xxx` có `data.result`: `"success"` hoặc `"fail_xxx"`.
- Robot tự gửi `notify_robot_info` mỗi giây (SN, phiên bản phần mềm, tình trạng IMU/camera/motor, % pin, trạng thái).
- Gửi sai định dạng → nhận `notify_invalid_request`.
- Test nhanh: dùng Postman → tạo WebSocket request tới `ws://10.192.1.2:5000`.

### 11.2 Danh sách lệnh cho tay

| Lệnh | Chức năng | `data` |
|---|---|---|
| `request_movej` | Di chuyển 14 khớp tay tới góc đích trong `time` giây | `{"time": 5, "joint": [14 giá trị rad]}` |
| `request_moveh` | Di chuyển đầu | `{"time": 5, "joint": [pitch, yaw]}` |
| `request_movep` | Di chuyển đầu tay tới tư thế Descartes | `{"time": 5, "pos": [Lx,Ly,Lz, Lw,Lx,Ly,Lz, Rx,Ry,Rz, Rw,Rx,Ry,Rz]}` |
| `request_servoj` | Điều khiển khớp liên tục (khuyến nghị 300 Hz), không có response | `{"filter_ratio": 1.0, "q": [16 giá trị]}` |
| `request_servop` | Điều khiển tư thế đầu tay liên tục (300 Hz) | `[L_pos(3), L_quat wxyz(4), R_pos(3), R_quat wxyz(4)]` |
| `request_get_move_pose` | Đọc tư thế đầu 2 tay | `{}` → `left_position`, `left_quat`, `right_position`, `right_quat` |
| `request_get_joint_state` | Đọc trạng thái khớp | `{}` → `names`, `q`, `dq`, `tau` |
| `request_set_compliant_enable` / `request_get_compliant_enable` | Bật/tắt, đọc chế độ mềm (compliance) | `{"enable": true}` |
| `request_set_compliant_param` / `request_get_compliant_param` | Đặt/đọc tham số compliance | `{"title": "joint_impedance_kp" \| "cart_impedance_kp", "data": [...]}` (khớp 14 / Descartes 12 giá trị) |
| `request_set_payload_param` | Khai báo tải ở đầu tay | `{"arm": 0\|1, "data": [m, mcx, mcy, mcz]}` |
| `request_fit_payload_param` | Tự nhận dạng tải | `{"arm": 0\|1}` |
| `request_get_payload_param` | Đọc tải 2 tay | `{}` |
| `request_enter_drag_teach` | Vào chế độ dạy bằng tay | `{}` |
| `request_drag_teach_manage` | Quản lý quỹ đạo | `{"cmd": "status\|list\|start_record\|stop_record\|play\|delete\|rename\|gripper\|exit", "arg": "..."}` |
| `request_set_limx_2fclaw_cmd` | Điều khiển kẹp LimX | `{"left_opening":0-100, "left_speed":0-100, "left_force":0-100, "right_...": ...}` (0 = đóng hẳn, 100 = mở hẳn) |
| `request_get_limx_2fclaw_state` | Đọc trạng thái kẹp | `{}` |
| `request_set_brainco2_hand_cmd` / `request_get_brainco2_hand_state` | Bàn tay BrainCo Revo 2 bản thường | xem SDK mục 4.4 |
| `request_set_brainco2_touch_hand_cmd` / `request_get_brainco2_touch_hand_state` | BrainCo Revo 2 bản có cảm biến xúc giác | xem SDK mục 4.5 |
| `request_light_effect` | Đổi hiệu ứng đèn | xem SDK mục 3.5.4 |
| `request_emgy_stop` | Dừng khẩn cấp | `{}` |

**Chỉ bản Mobile Dual-Arm:**

| Lệnh | Chức năng | `data` |
|---|---|---|
| `request_lifter_state` | Đọc giá nâng | `{}` → `q`, `v` |
| `request_set_lifter_position` | Nâng tới vị trí (mm) trong `duration` ms (0 = nhanh nhất) | `{"position": 200.5, "duration": 2000}` |
| `request_set_lifter_velocity` | Nâng/hạ theo vận tốc (mm/s) | `{"velocity": 50, "duration": 2000}` |
| `request_chassis_state` | Đọc đế | `{}` → `[v_dài, v_góc, góc lái]` |
| `request_chassis_move` | Chạy đế | `{"x": 0.3, "y": 0.0, "yaw": 0.0}` (mỗi giá trị trong [−1, 1]) |

### 11.3 Giới hạn quan trọng

- **MoveJ** kiểm tra giới hạn từng khớp, vượt là báo lỗi và **không chạy**:
  - Trên: `[2.60054, 2.96706, 1.48353, 0.523599, 1.39626, 0.7505, 0.7505, 2.60054, 0.261799, 3.66519, 0.523599, 1.74533, 0.7505, 0.7505]`
  - Dưới: `[-3.14159, -0.261799, -3.66519, -2.61799, -1.74533, -0.7505, -0.7505, -3.14159, -2.96706, -1.48353, -2.61799, -1.39626, -0.7505, -0.7505]`
- **MoveH:** pitch ∈ [−0,78; 1,04], yaw ∈ [−1,57; 1,57].
- **MoveP** — vùng với tới (m), gốc tọa độ ở tâm mặt dưới đế:
  - Tay trái: x ∈ [0,250; 0,732], y ∈ [−0,213; 0,900], z ∈ [−0,673; 0,5]
  - Tay phải: x ∈ [0,250; 0,732], y ∈ [−0,900; 0,213], z ∈ [−0,673; 0,5]

> ⚠️ **Khi test MoveJ:** không gửi nguyên mảng giới hạn trên/dưới (2 tay sẽ va vào thân/bàn/nhau). Bắt đầu từ tư thế an toàn (đọc bằng `request_get_joint_state`), **mỗi lần chỉnh 1 khớp một chút**, `time` ≥ 5 s, không ai đứng trong vùng hoạt động, tay sẵn sàng ở nút E-Stop.

### 11.4 Code mẫu Python

```bash
pip3 install websocket-client
```

```python
import json, time, uuid, threading
import websocket

ROBOT_IP = "10.192.1.2"
ACCID = None  # tự lấy từ notify_robot_info

def send(ws, title, data=None):
    ws.send(json.dumps({
        "accid": ACCID,
        "title": title,
        "timestamp": int(time.time() * 1000),
        "guid": str(uuid.uuid4()),
        "data": data or {},
    }))

def on_message(ws, msg):
    global ACCID
    m = json.loads(msg)
    if m.get("title") == "notify_robot_info":
        ACCID = m["accid"]
        return
    print(m["title"], m.get("data"))

def on_open(ws):
    def run():
        while ACCID is None:          # chờ nhận SN robot
            time.sleep(0.1)
        send(ws, "request_get_joint_state")
        # Gập 2 khuỷu tay -1.6 rad trong 5 s (ví dụ trong SDK)
        send(ws, "request_movej", {"time": 5,
             "joint": [0,0,0,-1.6,0,0,0, 0,0,0,-1.6,0,0,0]})
        time.sleep(6)
        send(ws, "request_set_limx_2fclaw_cmd",
             {"right_opening": 100, "right_speed": 50, "right_force": 50})
    threading.Thread(target=run, daemon=True).start()

ws = websocket.WebSocketApp(f"ws://{ROBOT_IP}:5000",
                            on_open=on_open, on_message=on_message)
ws.run_forever()
```

Ví dụ đầy đủ C++ / Python: [TRON_2_SDK_Development_Guide.md](../TRON_2_SDK_Development_Guide.md) mục 3.7.

---

## 12. Lập trình — SDK cấp thấp

> Cần **bản EDU** và robot ở chế độ **Low-Level Dev** (đèn xanh lá). Bạn tự chịu trách nhiệm toàn bộ điều khiển khớp — chạy với tư thế an toàn và có người cầm E-Stop.

Cài thư viện:

```bash
git clone https://github.com/limxdynamics/limxsdk-lowlevel.git
# Linux x86_64
pip install limxsdk-lowlevel/python3/amd64/limxsdk-*-py3-none-any.whl
# Linux aarch64:  .../python3/aarch64/...   — Windows: .../python3/win/...
```

Các hàm chính (C++ `limxsdk::Tron2`, Python tương đương):

| Hàm | Chức năng |
|---|---|
| `getInstance()` | Lấy instance duy nhất của robot |
| `init(robot_ip)` | Khởi tạo kết nối (mặc định `10.192.1.2`) |
| `getMotorNumber()` | Số motor |
| `subscribeRobotState(cb)` | Nhận `q` (rad), `dq` (rad/s), `tau` (N·m) |
| `subscribeImuData(cb)` | Nhận `acc`, `gyro`, `quat` (w,x,y,z) |
| `publishRobotCmd(cmd)` | Gửi lệnh khớp (`RobotCmd`) |
| `subscribeSensorJoy(cb)` | Đọc tay cầm (`buttons[]`, `axes[0..3]`) |
| `subscribeDiagnosticValue(cb)` | Nhận thông tin chẩn đoán |
| `setRobotLightEffect(...)` | Đổi đèn |

Chi tiết và code mẫu: [TRON_2_SDK_Development_Guide.md](../TRON_2_SDK_Development_Guide.md) mục 2.

### Camera và mô hình

- Camera đầu/cổ tay: [Head and Wrist Camera Guide](https://cwjgfm21di.feishu.cn/docx/NZ68dQnIHoANwOxZXzQczQSQnIg).
- Camera eo (RealSense D435i) qua ROS 1: đặt IP máy tính `10.192.1.200`, rồi
  ```bash
  export ROS_MASTER_URI=http://10.192.1.2:11311
  export ROS_IP=10.192.1.200
  rostopic list
  rqt_image_view
  ```
- URDF: xem SDK mục 5.1.

---

## 13. Demo AI: Phát kẹo bằng Pi0.5

Robot nhận diện người **ngửa bàn tay** → gắp kẹo đưa cho người. Dùng mô hình VLA **Pi0.5** (Physical Intelligence) qua framework OpenPI. Có sẵn dataset (1953 episode) và **weights đã huấn luyện** — có thể bỏ qua thu dữ liệu và huấn luyện, chạy thẳng mục 13.4.

### 13.1 Yêu cầu máy

| | Huấn luyện | Chạy (inference) |
|---|---|---|
| GPU | NVIDIA, khuyến nghị A100/H100 | ≥ 12 GB VRAM, khuyến nghị RTX 3090 trở lên |
| RAM | ≥ 32 GB | ≥ 16 GB |
| Ổ cứng | ≥ 100 GB NVMe | ≥ 30 GB |
| OS / Python | Ubuntu 22.04+, Python 3.11+ (`uv`) | Ubuntu 20.04+, Python 3.11+ |

### 13.2 Bố trí

- Bàn 160 × 80 × 75 cm, mặt trắng. Mép robot cách mép bàn 15 cm (có giá cố định).
- Kẹo xếp dày ở giữa bàn phía robot, trên khay màu kem; 2 tay đều với tới.
- Lúc chạy phải **giữ nguyên** chiều cao bàn, khoảng cách, ánh sáng, góc đầu như lúc thu dữ liệu.

### 13.3 Thu dữ liệu (tùy chọn)

1. Robot ở chế độ **Teleop**. Ở chế độ này đầu robot tự về `pitch = 1.04, yaw = 0`.
2. Mở phần mềm thu dữ liệu tại `https://10.192.1.4/`. Định dạng `mcap`, Domain ID `0`, chọn 13 topic khuyến nghị (xem tài liệu gốc mục 2.5). 1 phút dữ liệu ≈ 0,8–1,5 GB.
3. Mỗi episode:
   1. Giữ **Grip trái + X** và **Grip phải + A** → 2 tay về tư thế sẵn sàng.
   2. **Y** (tay trái) → bắt đầu ghi.
   3. Teleop gắp kẹo đưa cho người — **chỉ cử động khi thấy bàn tay ngửa**.
   4. **L1 + X** trên tay cầm hoặc **Grip phải + B** → 2 tay về Home.
   5. **B** (tay phải) → dừng ghi.
4. Chuyển `mcap` sang LeRobot bằng script `limx_mcap2lerobot`.

### 13.4 Chạy mô hình

```bash
git clone https://github.com/limx-tron2/tron2_openpi.git
git clone https://github.com/limx-tron2/tron2_env.git
cd tron2_openpi
sudo apt install pkg-config software-properties-common -y
sudo add-apt-repository ppa:ubuntuhandbook1/ffmpeg7 -y
sudo apt install ffmpeg libavformat-dev libavcodec-dev libavdevice-dev libavutil-dev libswscale-dev libswresample-dev libavfilter-dev -y
uv sync && uv pip install -e . && source .venv/bin/activate
uv pip install -e "../tron2_env[bridge,openpi]"

# Tải weights
mkdir checkpoints
hf download limxdynamics/tron2-openpi-models --repo-type model \
  --include "pi05_tron2_candy/**" --local-dir ./checkpoints
```

Sửa `configs/deploy/candy_server.yaml`: `robot.ip` = IP robot (các giá trị khác đã đúng cho weights có sẵn: `config: pi05_tron2_candy`, `checkpoint_dir: checkpoints/pi05_tron2_candy/candy_h30_rtc_25000`, `default_prompt: Hand out the candy`).

Trên robot: **L1 + X** (2 tay buông thõng) → chuyển sang **High-Level Dev**. Sau đó:

```bash
# Terminal 1 — policy server
uv run scripts/serve_policy.py --profile configs/deploy/candy_server.yaml
# Terminal 2 — client
uv run examples/tron2/pi_client_rtc.py --profile configs/deploy/candy_client.yaml
```

Tay robot chuyển sang tư thế cẳng tay thẳng đứng → đưa bàn tay ngửa vào khung hình camera để thử.
Lấy lại quyền điều khiển trong lúc chạy: giữ Grip trên tay cầm VR. Lệnh về Home sẽ ngắt inference.

### 13.5 Huấn luyện (tùy chọn)

```bash
export HF_LEROBOT_HOME=/path/to/datasets
cp configs/train/tron2_tasks/example.yaml configs/train/tron2_tasks/my_task.yaml   # sửa repo_id
uv run scripts/compute_norm_stats.py --task-config configs/train/tron2_tasks/my_task.yaml
uv run scripts/train_tron2_task.py   --task-config configs/train/tron2_tasks/my_task.yaml
```

Chi tiết: [【Candy_Dispensing】TRON2_Dual-Arm_Manipulation_Quickstart_(Pi05).md](<../【Candy_Dispensing】TRON2_Dual-Arm_Manipulation_Quickstart_(Pi05).md>).

---

## 14. Nâng cấp, log, bảo dưỡng

### Firmware robot
> Không tắt nguồn trong khi nâng cấp.
1. `ping 10.192.1.2` để kiểm tra kết nối.
2. Tải firmware tại https://www.limxdynamics.com/en/downloads (chọn TRON2).
3. Trang 8080 → **Version Management** → chọn loại firmware → **Select file** → **Upgrade**.
4. **Firmware driver motor phải nâng 2 bước:** chọn tất cả motor → "Motor Driver Firmware" → Upgrade; rồi chọn tất cả motor lần nữa → "Motor Driver Firmware Supplementary Package" → Upgrade.
5. Một số firmware sẽ tự khởi động lại robot.
6. Sau khi nâng main controller → **hiệu chuẩn điểm 0**.

### App VR
Gỡ app cũ (giữ Grip trên icon → Uninstall) → mở trình duyệt trên kính → limxdynamics.com → Support > Downloads → TRON 2 → tải và cài.

### Firmware tay cầm (chỉ Windows)
Tải tool từ trang chủ → tắt tay cầm, cắm vào PC → giữ **joystick trái + Power** (vào Boot) → chọn COM → OPEN COM → OPEN FILE (firmware) → UPGRADING.
Xem phiên bản: nhấn đúp Power → mục đầu tiên → nút →.

### Phần mềm thu dữ liệu (máy dev 10.192.1.4)
Không tự cập nhật theo firmware. Nếu nút VR không ăn hoặc dữ liệu bị lỗi sau khi nâng firmware:
```bash
ssh guest@10.192.1.4        # mật khẩu 123456
cd limx-agent/ && git pull && bash upgrade.sh
```
Sau đó khởi động lại robot.

### Xuất log
Wi-Fi robot → `http://10.192.1.2:8090` → xuất log + bag tại thời điểm gặp lỗi (gửi cho LimX khi cần hỗ trợ).

### Bảo dưỡng
- Lau bằng khăn microfiber; cất trong thùng chuyên dụng của TRON 2.
- Vệ sinh bụi ở khớp, kiểm tra vít lỏng định kỳ.
- Cập nhật firmware chính thức.
- Pin: cất ở 50–60%, sạc lại 3 tháng/lần.

---

## 15. Tra cứu nhanh

| Việc | Cách |
|---|---|
| Wi-Fi robot | `TRON2A_XXX_5G` / `_2.4G`, mật khẩu `12345678` |
| Trang quản lý | `http://10.192.1.2:8080` |
| WebSocket API | `ws://10.192.1.2:5000` |
| Log | `http://10.192.1.2:8090` |
| Teleop qua Internet | `http://10.192.1.2:9191` |
| SSH máy dev | `ssh guest@10.192.1.4` / `123456` |
| Phần mềm thu dữ liệu | `https://10.192.1.4/` |
| Về Home | **L1 + X** · web **Home** · VR **Grip phải + B** |
| Idle ngay | **L1 + □** · web **Idle Mode** |
| Đổi chế độ | Idle → **R1 + →/←** |
| Hiệu chuẩn điểm 0 | Idle → **L1 + R1** |
| E-Stop | 2 joystick cùng lúc · nút trên thân · web · `request_emgy_stop` |
| Thoát E-Stop (tay cầm) | Nhấn joystick phải |
