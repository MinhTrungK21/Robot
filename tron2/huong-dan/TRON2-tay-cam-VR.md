# Hướng dẫn dùng 2 tay cầm VR (PICO 4 Ultra) điều khiển TRON 2

> Nguồn: *TRON 2 User Manual* (Main Controller V2.7) — https://www.limxdynamics.com/en/documents/852949782845591552 — mục 5 "VR Device and Dual-Arms Teleoperation" và 6.4.
>
> **Áp dụng cho:** robot cấu hình **2 tay** (Dual-Arm) và **2 tay trên đế di động** (Mobile Dual-Arm).
> Tay cầm VR **không** điều khiển robot chân — robot chân dùng tay cầm có màn hình ([TRON2-tay-cam.md](TRON2-tay-cam.md)).

---

## Mục lục

1. [Các nút trên tay cầm VR](#1-các-nút-trên-tay-cầm-vr)
2. [Nguyên tắc quan trọng nhất: nút Grip](#2-nguyên-tắc-quan-trọng-nhất-nút-grip)
3. [Chuẩn bị (làm mỗi lần)](#3-chuẩn-bị-làm-mỗi-lần)
4. [Bảng lệnh đầy đủ](#4-bảng-lệnh-đầy-đủ)
5. [Thao tác từng bước](#5-thao-tác-từng-bước)
6. [Điều khiển đế di động và giá nâng](#6-điều-khiển-đế-di-động-và-giá-nâng)
7. [Thu dữ liệu để huấn luyện AI](#7-thu-dữ-liệu-để-huấn-luyện-ai)
8. [Điều khiển từ xa qua Internet](#8-điều-khiển-từ-xa-qua-internet)
9. [Mẹo và lỗi thường gặp](#9-mẹo-và-lỗi-thường-gặp)
10. [Thẻ tra cứu nhanh](#10-thẻ-tra-cứu-nhanh)

---

## 1. Các nút trên tay cầm VR

```
        TAY CẦM TRÁI                              TAY CẦM PHẢI
   ┌──────────────────────┐                ┌──────────────────────┐
   │        (●)           │                │           (●)        │
   │     Joystick trái    │                │    Joystick phải     │
   │                      │                │                      │
   │  [≡]   (X)   (Y)     │                │     (B)   (A)  [○]   │
   │  Menu                │                │               PICO   │
   └──────────┬───────────┘                └───────────┬──────────┘
              │  ◄── Trigger (cò) ở MẶT TRƯỚC, bóp bằng NGÓN TRỎ ──►  │
              │  ◄── Grip ở MẶT BÊN cán, bóp bằng NGÓN GIỮA ──►       │
```

| Nút | Vị trí / cách bấm | Dùng để điều khiển robot |
|---|---|---|
| **Grip** | Mặt bên cán tay cầm, **bóp bằng ngón giữa** | **Công tắc cho phép** — giữ thì tay robot bám theo, nhả thì tay robot khóa |
| **Trigger** (cò) | Mặt trước, **bóp bằng ngón trỏ** | Đóng/mở kẹp |
| **Joystick** | Cần gạt trên mặt | Đế di động, giá nâng |
| **Nhấn Joystick** | Ấn cần gạt xuống | Dừng khẩn cấp, đỗ xe |
| **X**, **Y** | Tay trái | Về tư thế, thu dữ liệu |
| **A**, **B** | Tay phải | Về tư thế, thu dữ liệu |
| **≡** (Menu), **○** (PICO) | | Nút hệ thống của kính, **không** dùng điều khiển robot |

> Robot chỉ có 1 tay → mặc định là **tay phải**, chỉ dùng **tay cầm phải**.

---

## 2. Nguyên tắc quan trọng nhất: nút Grip

> **Grip = công tắc cho phép của tay cầm đó.**
> Mọi lệnh VR **chỉ có tác dụng khi đang giữ ít nhất 1 nút Grip** — trừ 3 nút thu dữ liệu (Y, B) và dừng khẩn cấp.

| | Kết quả |
|---|---|
| **Giữ Grip trái** | Tay **trái** robot bám theo tay cầm trái |
| **Giữ Grip phải** | Tay **phải** robot bám theo tay cầm phải |
| **Nhả Grip** | Tay robot bên đó **đứng yên, khóa tại chỗ** |
| **Nhả cả 2 Grip** | Kết thúc teleop, robot về chế độ trước đó |

### Điều khiển theo "độ dời tương đối"

Tay robot **không nhảy tới vị trí tay cầm**. Nó di chuyển **cùng hướng, cùng quãng đường** mà tay cầm di chuyển **kể từ lúc bạn bóp Grip**.

Ví dụ: bóp Grip → đưa tay cầm sang phải 10 cm → tay robot cũng sang phải 10 cm, tính từ chỗ nó đang đứng.

**Di chuyển xa (giống nhấc chuột lên đặt lại):**
1. Giữ Grip, đưa tay cầm đi một đoạn.
2. **Nhả Grip** (tay robot đứng yên).
3. Đưa tay cầm về chỗ thoải mái.
4. **Bóp Grip lại**, đi tiếp.

---

## 3. Chuẩn bị (làm mỗi lần)

### 3.1 Bật robot và chuyển sang chế độ Teleop

1. Bật robot (xem [TRON2-tay.md](TRON2-tay.md) mục 5).
2. Chuyển sang chế độ **VR Teleoperation**, chọn 1 trong 2 cách:
   - **Tay cầm có màn hình:** `L1 + □` → màn hình hiện `Idle` → `R1 + ►` (hoặc `◄`) đến khi hiện **`Teleop`** + **`Active`**.
   - **Web:** mở `http://10.192.1.2:8080` → **Robot Control** → **Idle Mode** → chờ `ST_IDLE` → bấm **VR Teleoperation**.
3. Đèn robot **xanh dương đứng**.

> Nếu robot đang ở **High-Level Dev**, chỉ cần **giữ Grip** là robot tự chuyển sang Teleop. Ở Show Mode hoặc Low-Level Dev thì giữ Grip **không** có tác dụng.
>
> Dùng web 8080 + tay cầm VR thì **không cần** tay cầm có màn hình.

### 3.2 Bật kính và kết nối (lần đầu)

> **Không bóc** miếng dán che cảm biến trên kính — nếu bóc, kính sẽ hay tự ngủ.

Dùng **tay cầm phải** để trỏ, bóp nhẹ **Trigger** để chọn.

| Bước | Việc |
|---|---|
| 1 | Giữ nút nguồn bên phải kính **~3 s**, đeo kính vào màn hình chính |
| 2 | Quick panel / **Settings** → Wi-Fi → **`TRON2A_XXX_5G`**, mật khẩu **`12345678`** |
| 3 | **Library → Unknown Sources → Limx_teleop** |
| 4 | Chọn **Tron2**, IP **`10.192.1.2`** → **Connect** |
| 5 | Chọn **PICO Controllers** |
| 6 | Bật **Head Follow** nếu muốn đầu robot quay theo kính |
| 7 | Bật hình **camera cổ tay trái / phải** |
| — | Nếu hỏi *body following* → chọn **No** |

> 💡 Teleop tại chỗ: kính dùng **Wi-Fi 5 GHz của robot**. Nếu robot cần nối Internet thì cho robot nối Wi-Fi ngoài loại **2,4 GHz**. Hai mạng 5 GHz cùng lúc gây nhiễu, làm hình trễ và tay robot giật.

### 3.3 Tư thế người điều khiển

1. **Treo kính ngang trước cổ** (không cần đeo lên mắt nếu nhìn trực tiếp robot), cầm 2 tay cầm.
2. Đứng **ngoài tầm với** của tay robot.
3. **Đặt tay mình giống tư thế tay robot** trước khi bóp Grip — điều khiển sẽ tự nhiên hơn.

---

## 4. Bảng lệnh đầy đủ

### 4.1 Lệnh chung

| Việc | Nút | Điều kiện | Ghi chú |
|---|---|---|---|
| **Bắt đầu teleop** | Giữ **Grip** trên 1 tay cầm bất kỳ | Teleop, hoặc High-Level Dev (đứng yên / đang chạy mô hình AI) | Ở High-Level Dev: giữ Grip → tự chuyển sang Teleop |
| **Kết thúc teleop** | **Nhả Grip cả 2 tay** | Teleop | Robot về chế độ đang dùng trước đó |
| **Bắt đầu thu dữ liệu** | **Y** (tay trái) | Teleop, hoặc đang chạy mô hình AI | Không cần giữ Grip |
| **Dừng thu dữ liệu** | **B** (tay phải) | Mọi chế độ | Không cần giữ Grip |
| **Hủy bản ghi đang thu** | Nhấn **Y lần nữa** | Đang thu dữ liệu | **Xóa** bản ghi vừa thu |
| 🛑 **Dừng khẩn cấp** | Nhấn **cả 2 joystick** xuống cùng lúc | Mọi chế độ | Ngắt điện mọi motor, **2 tay rơi xuống**. Đế di động cũng dừng ngay |
| **Thoát dừng khẩn cấp** | Nhấn **joystick phải** | Đang dừng khẩn cấp | Motor có điện lại, vào damping |

### 4.2 Hai tay robot

| Việc | Nút | Ghi chú |
|---|---|---|
| **Tay trái bám theo** | Giữ **Grip trái** | Theo độ dời tương đối |
| **Tay phải bám theo** | Giữ **Grip phải** | |
| **Khóa tay** | **Nhả Grip** bên đó | Dùng khi làm 1 tay, giữ tay kia đứng yên |
| **Đóng/mở kẹp** | Giữ **Grip** + bóp **Trigger** cùng bên | Bóp càng sâu kẹp càng đóng; nhả Trigger = mở kẹp |
| **Giữ nguyên độ mở kẹp** | Đang bóp Trigger thì **nhả Grip** | Kẹp giữ nguyên, tay bên đó khóa |
| **Tay trái về tư thế teleop ban đầu** | Giữ **Grip trái + X** | Khuỷu gập 90°, cẳng tay nằm ngang |
| **Tay phải về tư thế teleop ban đầu** | Giữ **Grip phải + A** | Khuỷu gập 90°, cẳng tay nằm ngang |
| **Cả 2 tay về Home (buông thõng)** | Giữ **Grip phải + B** | |

> **Về tư thế:** phải **giữ nút** đến khi tay tới nơi. Nhả giữa chừng → tay **dừng ngay** tại chỗ; giữ lại để đi tiếp. Trong lúc về, tay **không** bám theo tay cầm.
> Về tư thế sẽ **ngắt mô hình AI đang chạy** và chuyển robot sang Teleop.

### 4.3 Đầu robot (cần bật Head Follow trong app)

| Việc | Nút |
|---|---|
| Cúi / ngẩng theo kính | Giữ **Grip** (ít nhất 1 tay) + cúi/ngẩng đầu |
| Quay trái / phải theo kính | Giữ **Grip** (ít nhất 1 tay) + quay đầu |
| **Đầu về thẳng** | Giữ **Grip cả 2 tay + X + A** |

### 4.4 Đế di động và giá nâng (chỉ Mobile Dual-Arm)

| Việc | Nút |
|---|---|
| Tiến / lùi | Giữ **Grip trái** + đẩy **joystick trái** lên/xuống |
| Sang trái / phải (không đổi hướng) | Giữ **Grip trái** + đẩy **joystick trái** trái/phải |
| Đi chéo | Giữ **Grip trái** + đẩy **joystick trái** chéo |
| Xoay tại chỗ | Giữ **Grip phải** + đẩy **joystick phải** trái/phải |
| **Nâng / hạ thân** | Giữ **Grip phải** + đẩy **joystick phải** lên/xuống |
| **Đỗ xe** (khóa 4 bánh hình chữ X) | Giữ **joystick phải** (ấn xuống) **1 s** |

Đẩy joystick càng xa → chạy càng nhanh.

---

## 5. Thao tác từng bước

### 5.1 Lần đầu tập điều khiển

1. Hoàn thành mục 3. Dọn trống 1 m quanh robot.
2. Giữ **Grip trái + X** và **Grip phải + A** → 2 tay về tư thế ban đầu (khuỷu 90°). Chờ tới nơi rồi nhả.
3. Đặt 2 tay mình giống tay robot: khuỷu gập, cẳng tay ngang.
4. **Bóp Grip phải**, di chuyển tay cầm phải **thật chậm, quãng ngắn** → quan sát tay phải robot.
5. **Nhả Grip** → tay robot dừng.
6. Làm tương tự với tay trái.
7. Tập kẹp: giữ Grip, bóp Trigger từ từ → kẹp đóng dần; nhả Trigger → kẹp mở.

> ⚠️ Teleop trễ khoảng **100 ms**. Cử động **nhỏ, chậm**; không vung tay đột ngột — tay robot có thể vung mạnh và va vào vật xung quanh.

### 5.2 Gắp một vật

1. Giữ **Grip** bên tay gần vật → đưa tay robot tới **phía trên** vật.
2. Hạ xuống từ từ, căn kẹp 2 bên vật (nhìn hình camera cổ tay trong kính nếu cần).
3. Bóp **Trigger** → kẹp đóng, giữ vật.
4. Vẫn giữ Grip + Trigger, nhấc vật lên, đưa tới chỗ đặt.
5. **Nhả Trigger** → kẹp mở, thả vật.
6. Rút tay robot lên → **nhả Grip**.

### 5.3 Làm việc bằng 1 tay

- Chỉ giữ Grip bên tay cần dùng. Tay kia **nhả Grip** → tay robot bên đó đứng yên.
- Muốn giữ vật bằng tay kia trong lúc làm: bóp Trigger cho kẹp đóng → **nhả Grip trước, vẫn giữ Trigger** → kẹp giữ nguyên, tay khóa.

### 5.4 Kết thúc

1. Giữ **Grip phải + B** → 2 tay về Home, buông thõng.
2. Nhả hết Grip.
3. Trên tay cầm có màn hình: **L1 + X** (hoặc **Home** trên web 8080) trước khi tắt robot.

> ⚠️ **Nhả Grip trước khi đi lại.** Nếu đang giữ Grip mà bước đi, cả 2 tay robot sẽ chạy theo người.

---

## 6. Điều khiển đế di động và giá nâng

(Chỉ bản **Mobile Dual-Arm**.)

- **Tay trái** lo **di chuyển**: giữ Grip trái, gạt joystick trái theo hướng muốn đi.
- **Tay phải** lo **xoay và nâng**: giữ Grip phải, joystick phải trái/phải = xoay, lên/xuống = nâng/hạ thân.
- Dùng **cùng lúc** được: VD vừa tiến vừa xoay để rẽ.
- Đứng lâu một chỗ (làm việc tại bàn): **giữ joystick phải 1 s** → đỗ xe, 4 bánh khóa chéo hình X, đèn đế **vàng đứng**.
- Nâng/hạ bị giới hạn bởi giới hạn trên/dưới đã cài trên robot.

---

## 7. Thu dữ liệu để huấn luyện AI

Chi tiết: [TRON2-train-mo-hinh.md](TRON2-train-mo-hinh.md) mục 4.

| # | Việc | Nút |
|---|---|---|
| 1 | 2 tay về tư thế sẵn sàng | Giữ **Grip trái + X** và **Grip phải + A** |
| 2 | **Bắt đầu ghi** | **Y** |
| 3 | Làm mẫu | Grip + Trigger |
| 4 | Về tư thế | Grip trái + X và Grip phải + A (hoặc **Grip phải + B** về Home) |
| 5 | **Dừng ghi** | **B** |
| — | Làm hỏng → hủy | **Y** lần nữa khi đang ghi |

> ⚠️ Bản dịch `train_act.pdf` ghi **ngược** (B bắt đầu, Y dừng). Bảng trên theo User Manual V2.7. **Thử 1 lần ngắn** và xem trong phần mềm thu dữ liệu để chắc.
>
> Nếu sau khi nâng cấp firmware mà nút VR không ăn: cập nhật phần mềm thu dữ liệu (`ssh guest@10.192.1.4` → `cd limx-agent/ && git pull && bash upgrade.sh`) rồi khởi động lại robot.

---

## 8. Điều khiển từ xa qua Internet

Các lệnh VR **giống hệt** khi điều khiển tại chỗ.

1. Robot nối Wi-Fi ngoài: web `http://10.192.1.2:8080` → biểu tượng Wi-Fi → nhập thông tin.
2. Đăng ký tài khoản tại `http://robonexus.limxdynamics.com/` → lấy **Access Key** ở trang cá nhân.
3. Mở `http://10.192.1.2:9191` → nhập địa chỉ platform và Access Key → lưu.
4. Ghi lại **SN** robot ở trang 8080 → *Robot Information*.
5. Trên kính: mở app teleop → chọn kết nối mạng ngoài → cấu hình **`tron2-online`** → nhập **Access Key** + **SN** → **Connect**.

---

## 9. Mẹo và lỗi thường gặp

| Hiện tượng | Nguyên nhân / cách xử lý |
|---|---|
| Bấm gì robot cũng không phản ứng | Chưa giữ **Grip**; robot chưa ở chế độ **Teleop**; đang dừng khẩn cấp (nhấn joystick phải) |
| Giữ Grip mà không vào Teleop | Robot đang ở Show Mode hoặc Low-Level Dev → chuyển sang Teleop bằng tay cầm có màn hình hoặc web 8080 |
| Tay robot giật, trễ, hình lag | Nhiễu Wi-Fi → kính dùng 5 GHz của robot, robot nối mạng ngoài qua 2,4 GHz; lại gần robot |
| Tay robot không tới được chỗ muốn | Nhả Grip, đưa tay cầm về, bóp Grip lại rồi đi tiếp (mục 2) |
| Tay robot chạy theo khi mình bước đi | Quên nhả Grip |
| Kẹp không đóng | Phải giữ **Grip** cùng bên rồi mới bóp Trigger; thử nhả Trigger 1 lần rồi bóp lại |
| Nút Y/B/X/A không ăn sau khi nâng firmware | Cập nhật phần mềm thu dữ liệu (mục 7) |
| Kính hay tự tắt màn hình | Không bóc miếng dán che cảm biến trên kính |
| Không thấy app Limx_teleop | Library → **Unknown Sources**. Cài lại: trình duyệt trên kính → limxdynamics.com → Support > Downloads → TRON 2 |

---

## 10. Thẻ tra cứu nhanh

```
                 TAY CẦM TRÁI                           TAY CẦM PHẢI
   ─────────────────────────────────────    ─────────────────────────────────────
   Grip            tay TRÁI bám theo         Grip            tay PHẢI bám theo
   Grip + Trigger  kẹp trái                  Grip + Trigger  kẹp phải
   Grip + X        tay trái về tư thế 90°    Grip + A        tay phải về tư thế 90°
   Y               bắt đầu ghi / hủy         Grip + B        2 tay về Home
                                             B               dừng ghi
   Grip + Joystick tiến/lùi/ngang (đế)       Grip + Joystick xoay (◄►) / nâng hạ (▲▼)
                                             Giữ Joystick 1s đỗ xe (khóa bánh)

   2 tay cùng:  Grip trái + Grip phải + X + A  → đầu về thẳng
   🛑 DỪNG KHẨN CẤP: nhấn CẢ 2 JOYSTICK      Thoát: nhấn JOYSTICK PHẢI
```

**Nhớ 3 điều:**
1. **Không giữ Grip = robot không nghe lệnh.**
2. **Cử động chậm, quãng ngắn** — trễ ~100 ms.
3. **Nhả Grip trước khi đi lại.**
