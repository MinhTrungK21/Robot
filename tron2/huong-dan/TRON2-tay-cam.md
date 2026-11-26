# Hướng dẫn dùng tay cầm (Remote Controller) điều khiển TRON 2

> Nguồn: *TRON 2 User Manual* bản web (Main Controller V2.7) — https://www.limxdynamics.com/en/documents/852949782845591552 — mục 3, 4, 5.3; *Autonomous Charging Dock User Manual* (lệnh vào đế sạc).
> Áp dụng cho cả cấu hình **2 tay** và **chân (Sole / bánh xe)**.

---

## Mục lục

1. [Làm quen với tay cầm](#1-làm-quen-với-tay-cầm)
2. [Bật / tắt và sạc tay cầm](#2-bật--tắt-và-sạc-tay-cầm)
3. [Đọc màn hình](#3-đọc-màn-hình)
4. [Nguyên tắc: Chế độ và Trạng thái](#4-nguyên-tắc-chế-độ-và-trạng-thái)
5. [Lệnh chung (mọi cấu hình)](#5-lệnh-chung-mọi-cấu-hình)
6. [Điều khiển robot 2 tay](#6-điều-khiển-robot-2-tay)
7. [Điều khiển robot chân (Sole / bánh xe)](#7-điều-khiển-robot-chân-sole--bánh-xe)
8. [Dừng khẩn cấp](#8-dừng-khẩn-cấp)
9. [Hiệu chuẩn điểm 0](#9-hiệu-chuẩn-điểm-0)
10. [Nâng cấp firmware tay cầm](#10-nâng-cấp-firmware-tay-cầm)
11. [Xử lý sự cố](#11-xử-lý-sự-cố)
12. [Thẻ tra cứu nhanh](#12-thẻ-tra-cứu-nhanh)

---

## 1. Làm quen với tay cầm

```
              L2                                         R2
              L1                                         R1
        ┌──────────────────────────────────────────────────────┐
        │  [Share]        ┌────────────────┐       [Options]  │
        │                 │   MÀN HÌNH     │                  │
        │     ▲           │                │          △       │
        │   ◄   ►         └────────────────┘       □     ○    │
        │     ▼               [Power]                 X       │
        │   D-pad                                             │
        │         (●)                            (●)          │
        │    Joystick TRÁI                  Joystick PHẢI     │
        └──────────────────────────────────────────────────────┘
```

> Sơ đồ minh họa vị trí tương đối. Hình chính xác xem ảnh `remote_keys_en.png` trong User Manual bản web, hoặc video hướng dẫn tay cầm ở mục 6 và 7.

| Nút | Ký hiệu trong tài liệu | Dùng để |
|---|---|---|
| **L1**, **L2**, **R1** | Nút vai | Luôn **giữ** cùng một nút khác để tạo tổ hợp lệnh (VD `L1 + X`) |
| **□ △ ○ X** | 4 nút hình bên phải | Kết hợp với L1/L2/R1 |
| **D-pad** ▲▼◄► | Up / Down / Left / Right | Đổi chế độ (`R1 + ◄/►`), chỉnh chiều cao (`R1 + ▲/▼`) |
| **Joystick trái** | Left joystick | Đi tới/lui, đi ngang |
| **Joystick phải** | Right joystick | Xoay tại chỗ |
| **Nhấn** joystick xuống | Left/Right joystick button | Nhấn **cả hai** = dừng khẩn cấp; nhấn **joystick phải** = thoát dừng khẩn cấp |
| **Share** | Share Button | Giữ 2 s: đổi chế độ đất bằng ↔ cầu thang (robot chân) |
| **Power** | Power Button | Bật/tắt tay cầm |
| Cổng Type-C | Charging/Data Port | Sạc, nâng cấp firmware |

**Cách bấm tổ hợp:** **giữ** nút thứ nhất (VD `L1`), **rồi nhấn** nút thứ hai (VD `X`), sau đó nhả cả hai.

---

## 2. Bật / tắt và sạc tay cầm

| Việc | Thao tác |
|---|---|
| **Bật** | Giữ **Power > 2 s** → nghe tiếng bíp, màn hình sáng |
| **Tắt** | Giữ **Power ~2–3 s** → nghe tiếng bíp, màn hình tối |
| **Xem pin tay cầm** | Biểu tượng pin ở **góc trên phải** màn hình (đây là pin tay cầm, **không phải** pin robot) |
| **Sạc** | Cáp Type-C, nguồn **5 V / 1 A**. Đèn tay cầm **xanh lá đứng** = đang sạc |
| **Ghép đôi lần đầu** | Xem video: https://oss.limxdynamics.com/uploads/material//TRON 2/video/6_remote_controller_pairing_EN_output.mp4 |

**Thứ tự bật đúng:** bật **robot** trước → chờ đèn robot hết nhấp nháy trắng → bật **tay cầm** → thấy **vạch sóng xanh** ở góc trên trái = đã kết nối.

---

## 3. Đọc màn hình

```
┌───────────────────────────────────────┐
│ ▂▄▆  (sóng)                 🔋 (pin TC)│
│                                       │
│   bat: 85    pwr: ...   errcode: 0    │   ← thông tin từ robot
│   tốc độ, chiều cao thân ...          │
│                                       │
│ Drive                          Walk   │
│ └─ CHẾ ĐỘ                TRẠNG THÁI ─┘│
└───────────────────────────────────────┘
```

| Vùng | Ý nghĩa |
|---|---|
| Trên trái | **Vạch sóng xanh** = đã kết nối robot; càng nhiều vạch sóng càng mạnh. Không có = chưa kết nối |
| Trên phải | Pin **tay cầm** |
| Giữa | `bat` = pin **robot**, `pwr` = trạng thái nguồn robot, `errcode` = mã lỗi hiện tại, tốc độ, chiều cao thân |
| **Dưới trái** | **Chế độ** hiện tại |
| **Dưới phải** | **Trạng thái** trong chế độ đó |

> **Luôn nhìn 2 góc dưới trước khi bấm lệnh.** Hầu hết lệnh chỉ có tác dụng ở một chế độ/trạng thái nhất định.

---

## 4. Nguyên tắc: Chế độ và Trạng thái

Robot luôn ở **1 chế độ** (dưới trái) và **1 trạng thái** trong chế độ đó (dưới phải).

| Chế độ (màn hình) | Cấu hình | Các trạng thái |
|---|---|---|
| **Motion** | 2 tay | `Idle`, `Show` (demo), `Hand Guide` (cầm tay kéo để dạy) |
| **Teleop** | 2 tay | `Idle`, `Active` (đang điều khiển bằng kính VR) |
| **Drive** | Chân | `Idle`, `Prepare`, `Walk`, `Docking`, `Charging`, `Undocking` |
| **High-Level Dev** | Cả hai (bản EDU) | `Idle`, `Active` |
| **Low-Level Dev** | Cả hai (bản EDU) | `Idle`, `Active` |

### 3 quy tắc vàng

1. **Muốn đổi chế độ → phải về `Idle` trước** (`L1 + □`).
2. Đổi chế độ bằng **`R1 + ►`** (tiến) / **`R1 + ◄`** (lùi), mỗi lần bấm sang 1 chế độ.
3. Robot **nhớ chế độ cuối cùng** — bật lên lần sau sẽ ở đúng chế độ trước khi tắt.

### Quy trình đổi chế độ (từng bước)

1. **Đỡ robot an toàn** (robot sẽ thả lỏng khớp).
2. Bấm **`L1 + □`** → chờ góc dưới phải hiện **`Idle`**.
3. Bấm **`R1 + ►`** hoặc **`R1 + ◄`** → xem góc dưới trái đổi tên chế độ.
4. Bấm tiếp đến khi tới đúng chế độ cần dùng.
5. Kiểm tra màu **đèn trạng thái** trên thân robot:

| Đèn robot | Chế độ |
|---|---|
| Xanh tím (blue-violet) | Motion (Show) |
| Xanh dương đứng | Teleop (2 tay) / Drive (chân) |
| Xanh lơ (cyan) đứng | High-Level Dev |
| Xanh lá đứng | Low-Level Dev |

> **Mẹo:** Màn hình đã hiện `Idle` mà bấm `R1 + ►` không đổi được → bấm **`L1 + □` thêm 1 lần** rồi thử lại.

**Thứ tự xoay vòng:**
- 2 tay: **Motion → Teleop → High-Level Dev → Low-Level Dev** → (quay lại Motion)
- Chân: **Drive → High-Level Dev → Low-Level Dev** → (quay lại Drive)

---

## 5. Lệnh chung (mọi cấu hình)

| Lệnh | Nút | Khi nào dùng được | Kết quả |
|---|---|---|---|
| **Về Idle ngay** | **L1 + □** | Mọi lúc | Không làm động tác về; khớp chuyển **damping**, **từ từ hạ xuống**. Dùng để đổi chế độ, hoặc khi động tác về (L1+X) bị lỗi |
| **Động tác về (Homing)** | **L1 + X** | Drive / Teleop / High-Level Dev | 2 tay: buông thõng. Chân: ngồi xổm, gập chân. Xong tự vào `Idle` |
| **Đổi chế độ** | **R1 + ►** / **R1 + ◄** | `Idle` | |
| **Dừng khẩn cấp** | Nhấn **cả 2 joystick** cùng lúc | Mọi lúc | Ngắt điện mọi motor (mục 8) |
| **Thoát dừng khẩn cấp** | Nhấn **joystick phải** | Đang dừng khẩn cấp | Motor có điện lại, vào damping |
| **Hiệu chuẩn điểm 0** | **L1 + R1** | `Idle` | Mục 9 |

> **L1 + X khác L1 + □ thế nào?**
> - **L1 + X** = robot **tự đưa về tư thế an toàn rồi mới nghỉ** → dùng hằng ngày, trước khi tắt máy.
> - **L1 + □** = **nghỉ ngay tại chỗ**, khớp thả lỏng từ từ → dùng để đổi chế độ, hoặc khi L1+X không chạy. Phải **đỡ robot** trước.

---

## 6. Điều khiển robot 2 tay

Video: https://oss.limxdynamics.com/uploads/material//TRON 2/video/8_remote_controller_arm_EN_output.mp4

### 6.1 Bảng lệnh

| Lệnh | Nút | Điều kiện | Ghi chú |
|---|---|---|---|
| Đổi chế độ | **R1 + ►/◄** | Idle | Motion → Teleop → High-Level Dev → Low-Level Dev |
| Về tư thế buông thõng | **L1 + X** | Teleop / High-Level Dev | 2 tay vung ra rồi buông thõng (có tránh mặt bàn nếu bật bảo vệ an toàn). **Đầu không tự về giữa** |
| Về Idle ngay | **L1 + □** | Mọi lúc | Tay từ từ hạ xuống |
| **Vào Show (demo)** | **L1 + △** | **Motion → Idle** | ⚠️ Dọn trống 1 m quanh robot |
| **Vào Hand Guide** | **L1 + ○** | **Motion → Idle** | Chế độ cầm tay kéo để dạy |
| ↳ Bắt đầu / dừng ghi quỹ đạo | **R1 + X** | Hand Guide | Bấm lần 1 = bắt đầu, lần 2 = dừng |
| ↳ Phát lại quỹ đạo vừa ghi | **R1 + □** | Hand Guide | |
| Dừng khẩn cấp | 2 joystick | Mọi lúc | 2 tay **rơi xuống** |
| Hiệu chuẩn điểm 0 | **L1 + R1** | Idle | |

> Show và Hand Guide **chỉ vào được từ `Idle` của chế độ Motion** bằng `L1 + △` / `L1 + ○` — không vào bằng `R1 + ►/◄` (để tránh bấm nhầm).

### 6.2 Tình huống: Chạy demo có sẵn

1. Dọn trống **1 m** quanh robot.
2. `L1 + □` → `Idle` → `R1 + ►/◄` đến **`Motion`**.
3. **`L1 + △`** → màn hình `Show`, robot chạy động tác mẫu.
4. Dừng: **`L1 + □`** (hoặc `L1 + X` nếu dùng được).

### 6.3 Tình huống: Dạy robot bằng cách cầm tay kéo (Hand Guide)

1. `L1 + □` → `Idle` → `R1 + ►/◄` đến **`Motion`**.
2. **`L1 + ○`** → màn hình `Hand Guide`. Tay robot mềm, kéo bằng tay được.
3. **`R1 + X`** → bắt đầu ghi.
4. Cầm tay robot kéo theo động tác muốn dạy.
5. **`R1 + X`** → dừng ghi.
6. Thả tay ra, lùi khỏi vùng hoạt động, dọn trống 1 m.
7. **`R1 + □`** → robot lặp lại động tác vừa dạy.

> Muốn lưu, đặt tên, quản lý nhiều quỹ đạo → dùng trang web `http://10.192.1.2:8080` → Robot Control → **Demo & Teaching** → **Drag Teaching**.

### 6.4 Tình huống: Chuẩn bị điều khiển bằng kính VR

1. `L1 + X` → 2 tay buông thõng.
2. `L1 + □` → `Idle` → `R1 + ►/◄` đến khi màn hình hiện **`Teleop`** + **`Active`**.
3. Đeo kính, mở app **Limx_teleop** (xem [TRON2-tay.md](TRON2-tay.md) mục 9).

### 6.5 Tình huống: Chạy chương trình / mô hình AI (High-Level Dev)

1. `L1 + X` → 2 tay buông thõng.
2. `L1 + □` → `Idle` → `R1 + ►/◄` đến **`High-Level Dev`** (đèn xanh lơ).
3. Chạy chương trình (VD demo gắp kẹo — [TRON2-gap-keo.md](TRON2-gap-keo.md)).
4. Xong: Ctrl+C chương trình → **`L1 + X`**.

### 6.6 Tắt robot 2 tay

1. **`L1 + X`** → 2 tay buông thõng, Idle.
2. Bản đế cố định: rút nguồn ngoài. Bản Mobile: tắt công tắc sau đế.
3. Robot: nhấn nhanh nút nguồn 0,5 s rồi giữ 3 s.
4. Tay cầm: giữ Power đến khi màn hình tắt.

> ⚠️ **Không tắt nguồn khi tay đang giơ** — khớp rơi tự do, có thể hỏng robot.

---

## 7. Điều khiển robot chân (Sole / bánh xe)

Video (bánh xe): https://oss.limxdynamics.com/uploads/material//TRON 2/video/9_remote_controller_wheeled_foot_EN_output.mp4

### 7.1 Sơ đồ trạng thái

```
                L1 + ○                 L1 + △
   ┌──────┐  ──────────►  ┌─────────┐ ──────────►  ┌──────┐
   │ Idle │               │ Prepare │              │ Walk │ ◄── joystick để đi
   └──────┘  ◄──────────  └─────────┘              └──────┘
      ▲         L1 + □ (mọi lúc)                     │
      └───────────── L1 + X (ngồi xổm, gập chân) ◄───┘

   Bị ngã (đèn vàng đứng) ──── L2 + △ ────► đứng lại như trước khi ngã
   Chỉ bánh xe: Walk ── L2 + X ──► Docking ─► Charging ─► Undocking ─► Walk
```

### 7.2 Bảng lệnh

| Lệnh | Nút | Điều kiện | Ghi chú |
|---|---|---|---|
| Đổi chế độ | **R1 + ►/◄** | Idle | Drive → High-Level Dev → Low-Level Dev |
| **Vào Prepare** | **L1 + ○** | Idle | Chạm đất, ngẩng đầu |
| **Đứng dậy** | **L1 + △** | Prepare | Đứng dần vào `Walk` (mặc định chế độ đất bằng) |
| Động tác về (ngồi, gập chân) | **L1 + X** | Drive / High-Level Dev | Xong tự vào Idle |
| Về Idle ngay | **L1 + □** | Mọi lúc | **Chỉ dùng khẩn cấp** — robot thả lỏng, có thể đổ |
| **Tiến / lùi** | **Joystick trái ▲▼** | Walk | Đẩy càng xa càng nhanh |
| **Sang trái / phải** | **Joystick trái ◄►** | Walk | **Chỉ Sole** (bánh xe không đi ngang được) |
| **Xoay tại chỗ** | **Joystick phải ◄►** | Walk | |
| **Nâng / hạ thân** | Giữ **R1 + ▲ / ▼** | Walk | Lên/xuống đều, nhả là dừng. Tắt/bật lại → về chiều cao mặc định |
| **Đất bằng ↔ cầu thang** | Giữ **Share 2 s** | Walk | Đèn **tím động** = chế độ cầu thang. Khuyên dùng **bánh xe** |
| **Đứng dậy khi ngã** | **L2 + △** | Đã ngã (đèn vàng đứng) | Không tác dụng khi bình thường |
| **Vào đế sạc** | **L2 + X** | Walk, cách đế 200–650 mm | **Chỉ bánh xe** (theo Charging Dock Manual) |
| Rời đế sạc | *(chưa có trong tài liệu)* | Charging | Xem *Auto-Recharge Base User Manual*: https://www.limxdynamics.com/en/documents/882997113569546240 |
| Dừng khẩn cấp | 2 joystick | Mọi lúc | Robot **mất lực và ngã** |
| Hiệu chuẩn điểm 0 | **L1 + R1** | Idle | |

### 7.3 Joystick khi đang đi

```
         Joystick TRÁI                     Joystick PHẢI
              ▲ tiến
              │
  sang trái ◄─●─► sang phải          xoay trái ◄─●─► xoay phải
   (Sole)     │     (Sole)
              ▼ lùi
```

- Hai joystick dùng **cùng lúc** được: vừa tiến vừa xoay = đi vòng cung.
- Lần đầu: **đẩy nhẹ**, robot tăng tốc dần. Tốc độ tối đa Sole 2–3 m/s, bánh xe 3–5 m/s — rất nhanh.

### 7.4 Tình huống: Bật máy và cho robot đi

1. Đặt robot **đúng tư thế sẵn sàng**: Sole **gối hướng trước**, bánh xe **gối hướng sau**; robot tự đứng cân bằng; nút E-Stop trên thân đã nhả.
2. **Lần đầu / thử nghiệm: móc dây treo an toàn.**
3. Bật robot → chờ đèn hết nhấp nháy trắng → bật tay cầm → thấy vạch sóng.
4. Nếu chưa ở `Drive`: `L1 + □` → `Idle` → `R1 + ►/◄` đến **`Drive`** (đèn xanh dương).
5. **`L1 + ○`** → `Prepare` (chạm đất, ngẩng đầu).
6. **`L1 + △`** → robot đứng dậy → `Walk`.
7. Dùng joystick để đi.

### 7.5 Tình huống: Leo cầu thang (khuyên dùng bánh xe)

1. Robot đang `Walk`, đứng trước cầu thang.
2. Giữ **Share 2 s** → đèn tím động.
3. Đẩy **joystick trái ▲** tiến lên, đủ nhanh và dứt khoát.
4. Lên xong: giữ **Share 2 s** để về chế độ đất bằng.

> ⚠️ Sole leo cầu thang **rủi ro lật cao** (cần đúng chiều cao bậc, độ sâu bậc, ma sát). Luôn dùng dây bảo hộ khi thử.

### 7.6 Tình huống: Robot bị ngã

1. Đèn robot **vàng đứng** = đã phát hiện ngã.
2. Kiểm tra xung quanh an toàn, không có người sát robot.
3. Bấm **`L2 + △`** → robot tự đứng dậy, về trạng thái trước khi ngã.
4. Không đứng được → `L1 + □` (Idle), tắt máy, dựng robot về tư thế sẵn sàng bằng tay rồi bật lại.

### 7.7 Tình huống: Tự vào đế sạc (bánh xe)

1. Đế đã bật (đèn xanh lá). Robot đang `Walk`.
2. Lái robot tới **cách đế 200–650 mm**, phía trước đế.
3. **`L2 + X`** → màn hình `Docking` → robot tự vào đế.
4. Vào xong → `Charging`, đèn đế cam.
5. Nhấn nút mà chưa thấy phản ứng → chờ vài giây.

### 7.8 Tắt robot chân

1. **`L1 + X`** → robot ngồi xổm, gập chân, hạ xuống ổn định → `Idle`.
2. Robot: giữ nút nguồn 3 s → đèn tắt.
3. Tay cầm: giữ Power đến khi màn hình tắt.
4. Đèn tắt hết rồi mới rút pin.

> ⚠️ **Luôn L1 + X trước khi tắt** — tắt thẳng khi đang đứng, robot **rơi đột ngột** → hỏng.

---

## 8. Dừng khẩn cấp

| | Thao tác |
|---|---|
| **Kích hoạt** | Nhấn **đồng thời joystick trái + joystick phải** (ấn xuống) |
| **Kết quả** | Ngắt điện **mọi motor** ngay lập tức. Pin và đèn vẫn sáng. **Tay rơi xuống / robot chân ngã** |
| **Thoát** | Nhấn **joystick phải** 1 lần → motor có điện lại, vào damping; vài giây sau robot tự phục hồi trạng thái |
| Nút E-Stop **trên thân** | Nhấn = dừng khẩn cấp. Nhả nút xong phải **bật lại nguồn robot** |
| Không có tay cầm | Nút **Emergency Stop** trên web `http://10.192.1.2:8080` |

> **Tập trước** cách nhấn 2 joystick cùng lúc khi robot đang đứng yên, để phản xạ nhanh lúc cần.
> Trước khi bật robot: kiểm tra robot **không** ở trạng thái dừng khẩn cấp.

---

## 9. Hiệu chuẩn điểm 0

**Chỉ làm khi:** vừa **nâng cấp firmware main controller**, hoặc robot **va đập mạnh** làm lệch vị trí khớp.

| Bước | 2 tay | Chân |
|---|---|---|
| 1 | **L1 + X** → 2 tay buông thõng, Idle. Bản Mobile: nâng giá lên **cao nhất** | **L1 + X** → ngồi, gập chân, Idle |
| 2 | **L1 + R1** → robot tự hiệu chuẩn | **L1 + R1** |
| 3 | Xong: 2 tay buông thõng tự nhiên | |
| 4 | Đèn **đỏ** = thất bại → làm lại | |
| 5 | Chờ **~5 s** cho robot cập nhật rồi mới thao tác tiếp | |

Video: https://oss.limxdynamics.com/uploads/material//TRON 2/video/12_zero_calibration_EN_output.mp4

---

## 10. Nâng cấp firmware tay cầm

> Công cụ nâng cấp **chỉ chạy trên Windows**.

1. Tải công cụ nâng cấp tay cầm từ https://www.limxdynamics.com/en/downloads (chọn TRON 2), giải nén, mở phần mềm.
2. Tay cầm **đang tắt** → cắm cáp Type-C vào máy tính Windows.
3. **Giữ joystick trái + Power** → tay cầm vào chế độ Boot.
4. Ô **COM**: chọn cổng của tay cầm → **OPEN COM**. Màn hình tay cầm hiện `BootUSB`.
5. Tải firmware tay cầm mới nhất → **OPEN FILE** → chọn file.
6. **UPGRADING** → chờ thông báo thành công.
7. Giữ Power để tắt, bật lại.

**Xem phiên bản firmware:** nhấn **đúp Power** → vào Settings → **▲/▼** chọn mục đầu tiên → **►** → dòng *Firmware Version*.

---

## 11. Xử lý sự cố

| Hiện tượng | Cách xử lý |
|---|---|
| Không có vạch sóng | Robot đã khởi động xong chưa (đèn hết nhấp nháy trắng)? Tắt/bật tay cầm; ghép đôi lại (video mục 2) |
| Bấm `R1 + ►` không đổi chế độ | Chưa ở `Idle` → `L1 + □`; đã `Idle` mà vẫn không được → `L1 + □` thêm 1 lần |
| `L1 + △` không vào Show | Phải đang ở **Motion → Idle** |
| `L1 + X` không có tác dụng | Chỉ chạy ở Drive / Teleop / High-Level Dev; dùng `L1 + □` thay thế |
| Robot không phản ứng gì | Đang dừng khẩn cấp? Nhấn joystick phải. Nút E-Stop trên thân đang nhấn? Nhả và bật lại nguồn |
| `L2 + △` không có tác dụng | Chỉ dùng được khi robot **đã ngã** (đèn vàng đứng) |
| Có `errcode` khác 0 | Ghi lại mã lỗi; xuất log ở `http://10.192.1.2:8090` gửi LimX |
| Nút bấm phản hồi chậm / chập chờn | Xa robot hoặc nhiễu sóng → lại gần, tránh nguồn nhiễu; kiểm tra pin tay cầm |

---

## 12. Thẻ tra cứu nhanh

> In trang này ra, dán cạnh robot.

### Lệnh chung
| | |
|---|---|
| **L1 + □** | Về Idle ngay (đỡ robot trước) |
| **L1 + X** | Về tư thế an toàn → Idle |
| **R1 + ► / ◄** | Đổi chế độ (chỉ khi Idle) |
| **L1 + R1** | Hiệu chuẩn điểm 0 (Idle) |
| **2 joystick** | 🛑 DỪNG KHẨN CẤP |
| **Joystick phải** | Thoát dừng khẩn cấp |

### Robot 2 tay
| | |
|---|---|
| **L1 + △** | Demo (Motion → Idle) |
| **L1 + ○** | Hand Guide (Motion → Idle) |
| **R1 + X** | Bắt đầu / dừng ghi quỹ đạo |
| **R1 + □** | Phát lại quỹ đạo |

### Robot chân
| | |
|---|---|
| **L1 + ○** | Idle → Prepare |
| **L1 + △** | Prepare → Đứng dậy (Walk) |
| **Joystick trái** | ▲▼ tiến/lùi · ◄► đi ngang (Sole) |
| **Joystick phải** | ◄► xoay |
| **R1 + ▲ / ▼** (giữ) | Nâng / hạ thân |
| **Share** (giữ 2 s) | Đất bằng ↔ cầu thang |
| **L2 + △** | Đứng dậy khi ngã |
| **L2 + X** | Vào đế sạc (bánh xe) |

### Thứ tự chế độ
- **2 tay:** Motion → Teleop → High-Level Dev → Low-Level Dev
- **Chân:** Drive → High-Level Dev → Low-Level Dev
