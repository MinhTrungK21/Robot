# Hướng dẫn chạy demo Gắp kẹo đưa vào tay — TRON 2 (Pi0.5)

> **Demo này làm gì:** Robot TRON 2 hai tay nhìn qua camera. Khi thấy một **bàn tay người ngửa lên** trong khung hình, robot gắp 1 viên kẹo trên bàn và **đặt vào lòng bàn tay** đó. Khi không thấy bàn tay ngửa, tay robot đứng yên.
>
> **Mô hình:** Pi0.5 (mô hình Vision-Language-Action của Physical Intelligence), do LimX fine-tune trên 1953 lần thao tác mẫu. Weights đã có sẵn nên **không cần tự huấn luyện**.
>
> Nguồn: *【Candy_Dispensing】TRON2_Dual-Arm_Manipulation_Quickstart_(Pi05)*, *TRON 2 User Manual*, và các file cấu hình trong repo `tron2_openpi` (nhánh `main`, kiểm tra ngày 25/11/2026).

---

## Mục lục

- [Phần 0 — Tổng quan hệ thống](#phần-0--tổng-quan-hệ-thống)
- [Phần 1 — Chuẩn bị phần cứng và vật dụng](#phần-1--chuẩn-bị-phần-cứng-và-vật-dụng)
- [Phần 2 — Cài đặt máy tính chạy mô hình (làm 1 lần)](#phần-2--cài-đặt-máy-tính-chạy-mô-hình-làm-1-lần)
- [Phần 3 — Bố trí bàn và robot](#phần-3--bố-trí-bàn-và-robot)
- [Phần 4 — Bật robot và kết nối mạng](#phần-4--bật-robot-và-kết-nối-mạng)
- [Phần 5 — Chạy demo](#phần-5--chạy-demo)
- [Phần 6 — Dừng demo và tắt máy](#phần-6--dừng-demo-và-tắt-máy)
- [Phần 7 — Xử lý sự cố](#phần-7--xử-lý-sự-cố)
- [Phần 8 — Tham số cấu hình](#phần-8--tham-số-cấu-hình)
- [Phụ lục A — Tự thu dữ liệu](#phụ-lục-a--tự-thu-dữ-liệu)
- [Phụ lục B — Tự huấn luyện](#phụ-lục-b--tự-huấn-luyện)
- [Phụ lục C — Checklist in ra giấy](#phụ-lục-c--checklist-in-ra-giấy)

---

## Phần 0 — Tổng quan hệ thống

```
┌──────────────────────────── Máy tính GPU (Ubuntu) ─────────────────────────────┐
│                                                                                 │
│  Terminal 1: Policy Server  (serve_policy.py)                                   │
│     - Nạp mô hình Pi0.5 lên GPU, lắng nghe cổng 8000                            │
│                    ▲  ảnh + trạng thái khớp      │  chuỗi 30 hành động           │
│                    │                             ▼                               │
│  Terminal 2: Client (pi_client_rtc.py)                                          │
│     - Lấy ảnh 3 camera + trạng thái khớp, gửi cho server                        │
│     - Nhận hành động, gửi lệnh ServoJ 300 Hz xuống robot                        │
└───────────────┬───────────────────────────────────────┬─────────────────────────┘
                │ ảnh camera (Bridge)                   │ lệnh khớp + trạng thái
                │ wss://10.192.1.4/bridge/ws            │ ws://10.192.1.2:5000
                ▼                                       ▼
   ┌──────────────────────────────┐       ┌────────────────────────────────┐
   │ Developer Expansion Module   │       │ Main controller TRON 2         │
   │ 10.192.1.4 (máy x86 trên lưng)│◄────►│ 10.192.1.2                      │
   │ Camera đầu + 2 cổ tay        │       │ 14 khớp tay + 2 kẹp + đầu       │
   └──────────────────────────────┘       └────────────────────────────────┘
```

**Tóm tắt luồng chạy:**
1. Robot ở **High-Level Dev**, 2 tay buông thõng.
2. Chạy **server** → nạp mô hình.
3. Chạy **client** → tay robot đưa ra tư thế khởi đầu (khuỷu gập, cẳng tay dựng đứng) → bắt đầu suy luận.
4. Người đưa **bàn tay ngửa** vào khung hình → robot gắp kẹo đặt vào tay.
5. **Ctrl+C** ở client để dừng → **L1 + X** đưa tay về.

---

## Phần 1 — Chuẩn bị phần cứng và vật dụng

### 1.1 Robot

| Hạng mục | Yêu cầu |
|---|---|
| Robot | TRON 2 cấu hình **2 tay** (Dual-Arm), bản **EDU** (cần chế độ High-Level Dev) |
| Đầu tay | **Kẹp 2 ngón LimX** ở cả 2 tay |
| Camera | Camera đầu + 2 camera cổ tay đã lắp và cắm dây |
| Developer Expansion Module | Đã lắp, cắm dây mạng vào robot (IP `10.192.1.4`) |
| Giá cố định | Có giá kẹp/giá đỡ robot cố định với bàn |
| Pin / nguồn | Pin đầy, hoặc cắm nguồn ngoài (bản đế cố định) |
| Tay cầm | Đã sạc, đã ghép đôi với robot |

### 1.2 Máy tính chạy mô hình (inference)

| Hạng mục | Tối thiểu | Khuyến nghị |
|---|---|---|
| GPU | NVIDIA **≥ 12 GB VRAM** | RTX 3090 (24 GB) trở lên |
| CPU | 6 nhân | i7-10700K / Ryzen 5 5600X trở lên |
| RAM | 16 GB | 32 GB |
| Ổ trống | 30 GB | SSD |
| OS | **Ubuntu 20.04+** (khuyên 22.04) | |
| Python | 3.11+, quản lý bằng **uv** | |
| Mạng | Truy cập được GitHub và Hugging Face (để tải code và weights) | |
| Cổng mạng | **Cổng Ethernet** để cắm dây vào robot (ổn định hơn Wi-Fi khi truyền ảnh) | |

> Máy Windows **không** dùng được trực tiếp. Dùng máy Ubuntu thật (khuyên dùng) hoặc cài Ubuntu song song.

### 1.3 Vật dụng

| Vật dụng | Yêu cầu |
|---|---|
| Bàn | **160 × 80 × 75 cm** (dài × rộng × cao), **mặt trắng trơn**. Chiều cao càng sát 75 cm càng tốt |
| Kẹo | Kẹo sữa **Big White Rabbit (Thỏ Trắng / 大白兔)**, hộp 500 g — loại dùng khi thu dữ liệu |
| Khay | Khay đựng trái cây **màu kem** (cream white), lót dưới kẹo |
| Thước dây | Để đo khoảng cách robot – mép bàn |

> **Vì sao phải giống hệt?** Mô hình chỉ học từ dữ liệu LimX thu trong đúng môi trường này. Bàn khác màu, kẹo khác loại, khay khác màu hoặc ánh sáng khác đều có thể làm robot gắp trượt hoặc không phản ứng.

---

## Phần 2 — Cài đặt máy tính chạy mô hình (làm 1 lần)

Mọi lệnh dưới đây chạy trên **máy Ubuntu có GPU**.

### 2.1 Kiểm tra driver GPU

```bash
nvidia-smi
```
Phải hiện tên GPU và dung lượng VRAM. Nếu báo lỗi → cài driver NVIDIA trước (`sudo ubuntu-drivers autoinstall`, rồi khởi động lại).

### 2.2 Cài công cụ cơ bản

```bash
sudo apt update
sudo apt install -y git curl pkg-config software-properties-common

# uv — trình quản lý Python
curl -LsSf https://astral.sh/uv/install.sh | sh
source ~/.bashrc          # hoặc mở terminal mới
uv --version
```

### 2.3 Cài ffmpeg 7 (cần để đọc/ghi video)

```bash
sudo add-apt-repository ppa:ubuntuhandbook1/ffmpeg7 -y
sudo apt install -y ffmpeg libavformat-dev libavcodec-dev libavdevice-dev \
  libavutil-dev libswscale-dev libswresample-dev libavfilter-dev
```

### 2.4 Tải code

Đặt 2 repo **cạnh nhau** trong cùng thư mục (ví dụ `~/limx`):

```bash
mkdir -p ~/limx && cd ~/limx
git clone https://github.com/limx-tron2/tron2_openpi.git
git clone https://github.com/limx-tron2/tron2_env.git
```

Kết quả:
```
~/limx/
├── tron2_openpi/   # OpenPI đã chỉnh cho TRON2: server, client, cấu hình
└── tron2_env/      # Gói môi trường TRON2 (kết nối robot, Bridge camera)
```

### 2.5 Tạo môi trường Python

```bash
cd ~/limx/tron2_openpi
uv sync
uv pip install -e .
source .venv/bin/activate
uv pip install -e "../tron2_env[bridge,openpi]"
```

**Kiểm tra cài đặt:**
```bash
PYTHONPATH="$(cd ../tron2_env/src && pwd)" uv run python -c "import tron2_env; print(tron2_env.__file__)"
```
✅ Đúng khi in ra đường dẫn kết thúc bằng `../tron2_env/src/tron2_env/__init__.py`.

### 2.6 Tải weights mô hình

```bash
cd ~/limx/tron2_openpi
uv pip install -U "huggingface_hub[cli]"     # cung cấp lệnh hf (bỏ qua nếu đã có)
mkdir -p checkpoints
hf download limxdynamics/tron2-openpi-models \
  --repo-type model \
  --include "pi05_tron2_candy/**" \
  --local-dir ./checkpoints
```

✅ Xong khi có thư mục:
```
tron2_openpi/checkpoints/pi05_tron2_candy/candy_h30_rtc_25000/
```

> File weights khá lớn — tải qua mạng ổn định và để đủ chỗ trống trên ổ.

### 2.7 Kiểm tra file cấu hình

Mở `configs/deploy/candy_server.yaml`, các giá trị phải là:

```yaml
policy:
  config: pi05_tron2_candy
  repo_id: input
  checkpoint_dir: checkpoints/pi05_tron2_candy/candy_h30_rtc_25000
  default_prompt: Hand out the candy
  action_horizon: 30
  state_dim: 16
server:
  host: 0.0.0.0
  port: 8000
```

Mở `configs/deploy/candy_client.yaml`, kiểm tra:

```yaml
client:
  policy_host: 127.0.0.1      # server và client cùng máy
  policy_port: 8000
  observation_source: bridge  # lấy ảnh qua Bridge trên máy 10.192.1.4
robot:
  ip: 10.192.1.2              # IP main controller robot
  port: 5000
bridge:
  host: wss://10.192.1.4
  ws_path: /bridge/ws
```

Với weights có sẵn và robot dùng IP mặc định thì **không cần sửa gì**.

---

## Phần 3 — Bố trí bàn và robot

### 3.1 Sơ đồ (nhìn từ trên xuống)

```
                   NGƯỜI NHẬN KẸO (đứng đối diện robot)
                         ✋ đưa tay ngửa vào đây
   ┌──────────────────────────────────────────────────────────┐
   │                                                          │
   │                  MẶT BÀN TRẮNG 160 × 80 cm               │  80 cm
   │                                                          │
   │                 ┌───────────────────┐                    │
   │                 │  KHAY KEM + KẸO   │ ← giữa bàn,         │
   │                 │  xếp dày          │   sát phía robot    │
   │                 └───────────────────┘                    │
   └──────────────────────────────────────────────────────────┘
                      ↕ 15 cm (mép robot → mép bàn)
                    ┌─────────┐
                    │ TRON 2  │  (gắn giá cố định, nhìn về phía bàn)
                    └─────────┘
```

### 3.2 Các bước

1. **Bàn:** đặt chắc chắn trên sàn phẳng, cao 75 cm, mặt trắng sạch. Không có đồ khác trên bàn.
2. **Robot:** gắn giá cố định, đặt **mép robot cách mép bàn 15 cm** (đo bằng thước), robot nhìn thẳng vào giữa bàn.
3. **Khay + kẹo:** đặt khay kem **ở giữa bàn, sát phía robot**. Đổ kẹo **xếp dày** trong khay.
4. **Kiểm tra tầm với:** sau khi bật robot (Phần 4), có thể dùng VR teleop đưa thử từng tay ra để chắc **cả 2 tay đều với tới kẹo**.
5. **Ánh sáng:** đèn trong nhà đều, không để nắng chiếu thẳng vào bàn hoặc camera. Không đổi ánh sáng giữa chừng.
6. **Khoảng trống:** dọn sạch vật cản trong **bán kính 1 m** quanh robot, đặc biệt **phía trước và 2 bên** (khi khởi động tay robot sẽ dang sang ngang rồi đưa ra trước).

> ⚠️ Chiều cao bàn, khoảng cách 15 cm, ánh sáng và góc đầu robot phải **giữ nguyên** trong suốt quá trình chạy.

---

## Phần 4 — Bật robot và kết nối mạng

### 4.1 Kiểm tra trước khi bật

- [ ] Đầu robot đã lắp, dây nguồn + dây truyền thông khớp đầu đã cắm (thiếu thì robot không khởi động).
- [ ] Dây nguồn đúng cổng (**không cắm 24 V vào cổng 48 V**).
- [ ] Nút E-Stop trên thân đã nhả.
- [ ] Developer Expansion Module đã cắm dây mạng vào robot.

### 4.2 Bật robot

1. Cắm pin (hoặc nguồn ngoài).
2. Nhấn nhanh nút nguồn **0,5 s**, rồi **giữ 3 s** → đèn pin sáng.
3. Giữ nút nguồn tay cầm **2 s**.
4. Chờ ~20 s: đèn trạng thái chuyển từ **trắng nhấp nháy** sang màu đứng; tay cầm có vạch sóng.

### 4.3 Nối máy tính với robot

**Cách 1 — Dây mạng (khuyên dùng):**
1. Cắm cáp LAN từ máy tính vào **cổng Gigabit phía sau robot**.
2. Đặt IP tĩnh cho máy tính: Settings → Network → Wired → IPv4 → Manual:
   - Address: `10.192.1.120`
   - Netmask: `255.255.255.0`
   - Gateway: để trống
3. Tắt/bật lại kết nối dây.

**Cách 2 — Wi-Fi:** kết nối `TRON2A_XXX_5G`, mật khẩu `12345678`.

### 4.4 Kiểm tra kết nối

```bash
ping -c 3 10.192.1.2     # main controller
ping -c 3 10.192.1.4     # máy Bridge / camera
```
✅ Cả 2 đều có phản hồi.

Mở trình duyệt:
- `http://10.192.1.2:8080` → trang quản lý robot, xem trạng thái.
- `https://10.192.1.4/` → phần mềm thu dữ liệu (chấp nhận cảnh báo chứng chỉ). Ở trang **Overview** xem được hình camera — kiểm tra camera đầu **thấy rõ khay kẹo và vùng người đưa tay**.

### 4.5 Hiệu chuẩn điểm 0 (chỉ khi cần)

Chỉ làm khi **vừa nâng cấp firmware main controller** hoặc robot **bị va đập mạnh** làm lệch tay:
1. **L1 + X** → 2 tay buông thõng, Idle.
2. **L1 + R1** → robot tự hiệu chuẩn. Đèn đỏ = thất bại, làm lại.
3. Chờ ~5 s.

---

## Phần 5 — Chạy demo

### Bước 1 — Đưa 2 tay về tư thế buông thõng

Nhấn **L1 + X** trên tay cầm (hoặc **Home** trên trang `http://10.192.1.2:8080` → Robot Control).

✅ Cả 2 tay **buông thẳng dọc thân**.

> ⚠️ **Bắt buộc.** Client khởi động sẽ đưa tay **dang sang ngang rồi ra trước**. Nếu tay không ở tư thế buông thõng, robot có thể **giơ thẳng 2 tay lên** và va vào vật phía trước.

### Bước 2 — Chuyển robot sang High-Level Dev

**Bằng tay cầm:**
1. **L1 + □** → góc dưới phải màn hình hiện `Idle`.
2. Nhấn **R1 + →** (hoặc **R1 + ←**) cho đến khi góc dưới trái hiện **`High-Level Dev`**.

**Hoặc bằng web 8080:** Robot Control → **Idle Mode** → chờ hiện `ST_IDLE` → bấm **Developer**.

✅ Đèn trạng thái robot **xanh lơ (cyan) đứng**.

> Thứ tự chế độ: Motion → Teleop → **High-Level Dev** → Low-Level Dev. Chỉ đổi được khi đang `Idle`.

### Bước 3 — Khởi động Policy Server (Terminal 1)

```bash
cd ~/limx/tron2_openpi
source .venv/bin/activate
uv run scripts/serve_policy.py --profile configs/deploy/candy_server.yaml
```

- Lần chạy đầu JAX cần biên dịch mô hình nên **có thể mất vài phút**. Các lần sau nhanh hơn nhờ bộ nhớ đệm ở `/tmp/openpi_jax_cache` (đổi chỗ lưu bằng biến `OPENPI_JAX_CACHE_DIR`).
- ✅ Chờ đến khi log báo server đang lắng nghe cổng **8000**, không còn chạy chữ tải mô hình.
- **Để nguyên terminal này**, không tắt.

### Bước 4 — Đứng vào vị trí an toàn

- Người điều khiển cầm **tay cầm**, sẵn sàng nhấn **2 joystick cùng lúc** (E-Stop).
- Không ai đứng trong vùng tay robot với tới, trừ người đưa tay nhận kẹo ở phía đối diện bàn.

### Bước 5 — Khởi động Client (Terminal 2)

Mở terminal mới:

```bash
cd ~/limx/tron2_openpi
source .venv/bin/activate
uv run examples/tron2/pi_client_rtc.py --profile configs/deploy/candy_client.yaml
```

**Điều sẽ xảy ra:**
1. Client kết nối robot (`10.192.1.2:5000`), Bridge (`wss://10.192.1.4`) và server (`127.0.0.1:8000`).
2. Đầu robot về góc `pitch ≈ 1.047, yaw ≈ 0` (cúi nhìn xuống bàn).
3. 2 tay **dang sang ngang rồi đưa ra trước**, dừng ở tư thế **khuỷu gập ~90°, cẳng tay dựng đứng** ở 2 bên.
4. Log hiện `Running indefinitely (duration=0). Ctrl+C to stop.` → bắt đầu suy luận liên tục.

### Bước 6 — Đưa tay nhận kẹo

1. Người nhận đứng **đối diện robot, bên kia bàn**.
2. **Ngửa bàn tay lên**, xòe phẳng, đưa vào giữa khung nhìn của camera đầu — phía trên bàn, trước khay kẹo.
3. Giữ tay **yên**, chờ robot:
   - một tay robot vươn tới khay → kẹp 1 viên kẹo → đưa tới → **thả kẹo vào lòng bàn tay**.
4. Rút tay về. Robot quay lại chờ.
5. Muốn nhận tiếp → lại đưa tay ngửa vào.

**Mẹo để chạy ổn định:**
- Ngửa tay rõ ràng, lòng bàn tay hướng lên trần nhà.
- Mỗi lần chỉ đưa **1 bàn tay**.
- Không mặc găng tay hoặc tay áo che lòng bàn tay.
- Không đưa tay **quá gần** khay kẹo hoặc vào giữa đường tay robot đang chạy.
- Khi không nhận kẹo, **rút hẳn tay ra khỏi khung hình** — robot sẽ đứng yên.
- Hết kẹo ở vùng giữa khay → dừng demo (Phần 6) rồi mới đổ thêm kẹo.

### Bước 7 (tùy chọn) — Đổi câu lệnh (prompt)

Prompt mặc định là `Hand out the candy`. Muốn thử câu khác (mô hình chỉ được huấn luyện với câu mặc định nên kết quả có thể kém hơn):

```bash
uv run examples/tron2/pi_client_rtc.py --profile configs/deploy/candy_client.yaml --prompt "Hand out the candy"
```

### Chạy server và client trên 2 máy khác nhau (tùy chọn)

Ví dụ server chạy trên máy GPU mạnh, client chạy trên máy nối dây với robot:
- Máy server: giữ `host: 0.0.0.0`, mở cổng **8000** trên tường lửa.
- Máy client: sửa `candy_client.yaml` → `policy_host: <IP máy server>`.
- Máy client vẫn phải ping được `10.192.1.2` và `10.192.1.4`.

---

## Phần 6 — Dừng demo và tắt máy

### Dừng bình thường

1. Rút tay người ra khỏi bàn.
2. Ở **Terminal 2 (client):** nhấn **Ctrl+C** → log `Interrupted by operator.` → `Cleanup completed`. Robot ngừng nhận lệnh mới.
3. Nhấn **L1 + X** trên tay cầm → 2 tay về tư thế buông thõng (Idle).
4. Ở **Terminal 1 (server):** nhấn **Ctrl+C** nếu không chạy tiếp.

> Chạy lại: robot đang ở High-Level Dev nên chỉ cần **L1 + X** (tay buông thõng), rồi làm lại **Bước 5**. Server có thể để chạy liên tục.

### Dừng khẩn cấp

| Tình huống | Làm gì |
|---|---|
| Tay robot sắp va vào người/vật | Nhấn **đồng thời 2 joystick** trên tay cầm → ngắt điện motor |
| Không có tay cầm | Nhấn **nút E-Stop trên thân robot**, hoặc **Emergency Stop** trên web 8080 |
| Cần lấy lại quyền điều khiển nhưng không cần E-Stop | Giữ **nút Grip** trên tay cầm VR → robot chuyển sang Teleop (nếu đang dùng VR) |

> ⚠️ E-Stop làm **2 tay rơi xuống** do mất lực — tránh để tay người ngay bên dưới.
> Thoát E-Stop tay cầm: nhấn **joystick phải** 1 lần. Thoát E-Stop trên thân: nhả nút rồi **bật lại nguồn robot**.

### Tắt máy cuối ngày

1. Ctrl+C client và server.
2. **L1 + X** → 2 tay buông thõng, Idle. **Không tắt nguồn khi tay còn đang giơ** (khớp sẽ rơi tự do).
3. Nhấn nhanh 0,5 s rồi giữ 3 s nút nguồn robot.
4. Giữ 3 s nút nguồn tay cầm.
5. Pin cất lâu ngày: để 50–60%.

---

## Phần 7 — Xử lý sự cố

| Hiện tượng | Nguyên nhân thường gặp | Cách xử lý |
|---|---|---|
| `uv: command not found` | Chưa nạp lại shell sau khi cài uv | `source ~/.bashrc` hoặc mở terminal mới |
| Lệnh kiểm tra `tron2_env` không in ra đường dẫn | Chưa cài `tron2_env` hoặc 2 repo không nằm cạnh nhau | Đặt `tron2_openpi` và `tron2_env` chung thư mục cha, chạy lại bước 2.5 |
| `hf: command not found` | Chưa có huggingface_hub CLI | `uv pip install -U "huggingface_hub[cli]"` |
| Server báo không tìm thấy checkpoint | Sai đường dẫn hoặc tải chưa xong | Kiểm tra có `checkpoints/pi05_tron2_candy/candy_h30_rtc_25000/`; chạy lại lệnh `hf download` |
| Server báo hết bộ nhớ GPU (OOM) | GPU < 12 GB hoặc có chương trình khác chiếm GPU | `nvidia-smi` xem tiến trình, tắt bớt; dùng GPU ≥ 12 GB |
| Server khởi động rất lâu | JAX đang biên dịch lần đầu | Chờ; lần sau sẽ nhanh hơn |
| Client không kết nối được server | Server chưa sẵn sàng, sai `policy_host`/cổng | Chờ server chạy xong; kiểm tra `policy_host`, `policy_port: 8000`, tường lửa |
| Client không kết nối robot | Sai IP, mạng chưa nối, robot chưa ở High-Level Dev | `ping 10.192.1.2`; kiểm tra đèn xanh lơ; kiểm tra `robot.ip` |
| Client báo lỗi Bridge / không có ảnh | Máy 10.192.1.4 chưa lên hoặc phần mềm Bridge cũ | `ping 10.192.1.4`; mở `https://10.192.1.4/` xem có hình; cập nhật phần mềm (bên dưới) |
| Robot không cử động sau khi chạy client | Robot chưa ở High-Level Dev; đang E-Stop | Kiểm tra màn hình tay cầm và màu đèn; thoát E-Stop |
| Tay giơ thẳng lên khi khởi động | Tay không ở tư thế buông thõng trước khi chạy | Ctrl+C, **L1 + X**, chạy lại client |
| Robot đứng yên khi đưa tay | Tay không ngửa rõ, ngoài khung hình, ánh sáng kém | Ngửa tay rõ, đưa vào giữa khung camera đầu; cải thiện ánh sáng |
| Gắp trượt / gắp hụt | Bàn, khay, kẹo hoặc khoảng cách khác dữ liệu gốc | Đo lại 75 cm và 15 cm; dùng đúng kẹo Thỏ Trắng + khay kem; xếp kẹo dày giữa khay |
| Tay robot giật, trễ | Mạng Wi-Fi yếu/nhiễu | Dùng **dây mạng**; nếu robot đang nối Wi-Fi 5 GHz bên ngoài thì chuyển sang 2,4 GHz |
| Log báo mất ảnh tạm thời | Mất kết nối Bridge ngắn | Client tự phục hồi trong 5 s (`obs_timeout_budget_s`); nếu lặp lại thì kiểm tra dây mạng |

**Cập nhật phần mềm trên máy 10.192.1.4** (khi vừa nâng firmware robot mà Bridge hoặc nút VR không hoạt động đúng):
```bash
ssh guest@10.192.1.4          # mật khẩu 123456
cd limx-agent/
git pull
bash upgrade.sh
```
Sau đó khởi động lại robot. Máy 10.192.1.4 cần có Internet (cho robot vào Wi-Fi ngoài ở trang 8080 trước).

**Xuất log robot để gửi LimX:** `http://10.192.1.2:8090` → xuất log + bag tại thời điểm lỗi.

---

## Phần 8 — Tham số cấu hình

### `configs/deploy/candy_server.yaml`

| Tham số | Giá trị | Ý nghĩa |
|---|---|---|
| `policy.config` | `pi05_tron2_candy` | Tên cấu hình huấn luyện (trong `src/openpi/training/config.py`) |
| `policy.repo_id` | `input` | Tên thư mục asset trong checkpoint (chứa thông số chuẩn hóa) |
| `policy.checkpoint_dir` | `checkpoints/pi05_tron2_candy/candy_h30_rtc_25000` | Đường dẫn weights |
| `policy.default_prompt` | `Hand out the candy` | Câu lệnh mặc định |
| `policy.record` | `false` | Ghi input/output mô hình để debug |
| `policy.action_horizon` | `30` | Mỗi lần suy luận sinh 30 bước hành động |
| `policy.state_dim` | `16` | Trạng thái = 14 khớp tay + 2 kẹp |
| `server.host` / `port` | `0.0.0.0` / `8000` | Địa chỉ server lắng nghe |

### `configs/deploy/candy_client.yaml`

| Tham số | Giá trị | Ý nghĩa |
|---|---|---|
| `client.policy_host` / `policy_port` | `127.0.0.1` / `8000` | Địa chỉ server |
| `client.observation_source` | `bridge` | Lấy ảnh qua Bridge (`legacy` = cắm thẳng 3 camera RealSense bằng USB vào máy inference) |
| `client.state_dim` | `16` | 16 = tay + kẹp; 18 = thêm 2 khớp đầu |
| `client.fps` | `30` | Tốc độ phát hành động của policy |
| `client.publish_rate` | `300` | Tần số gửi ServoJ xuống robot (Hz) |
| `client.rtc_enabled` | `true` | Dùng RTC (real-time chunking) cho chuyển động mượt |
| `client.duration` | `0` | Thời gian chạy (giây); 0 = chạy đến khi Ctrl+C |
| `client.execution_horizon` / `delay` | `10` / `6` | Tham số RTC |
| `client.obs_timeout_budget_s` | `5.0` | Cho phép mất ảnh tối đa 5 s rồi tự phục hồi |
| `client.save_record` | `false` | Bật để ghi CSV hành động/trạng thái vào `debug_images/` |
| `client.prompt` | `null` | Dùng prompt mặc định của server |
| `robot.ip` / `port` | `10.192.1.2` / `5000` | WebSocket điều khiển robot |
| `robot.init_joints` | 14 giá trị | Tư thế khởi đầu của tay (trái 7 + phải 7) |
| `robot.init_head` | `[1.0467, -0.014]` | Góc đầu khởi đầu [pitch, yaw] |
| `bridge.host` / `ws_path` | `wss://10.192.1.4` / `/bridge/ws` | Bridge lấy ảnh camera |
| `bridge.verify_tls` | `false` | Không kiểm tra chứng chỉ (Bridge dùng chứng chỉ tự ký) |
| `bridge.save_debug_images` | `false` | Bật để lưu ảnh camera vào `debug_images/` khi cần kiểm tra |

> ⚠️ Không sửa `init_joints` / `init_head` nếu chưa kiểm tra kỹ tư thế đó an toàn trên robot của mình.

---

## Phụ lục A — Tự thu dữ liệu

Chỉ cần khi muốn **thêm dữ liệu** cho môi trường của mình hoặc **làm bài toán gắp-thả khác**.

### A.1 Chuẩn bị

1. Bố trí giống Phần 3.
2. Robot sang chế độ **Teleop**: **L1 + □** → `Idle` → **R1 + →/←** đến khi hiện `Teleop` + `Active`. Đầu robot tự về `pitch = 1.04, yaw = 0`.
3. Thiết lập kính **PICO 4 Ultra**: Wi-Fi `TRON2A_XXX_5G` / `12345678` → Library → Unknown Sources → **Limx_teleop** → chọn **Tron2**, IP `10.192.1.2` → **Connect** → bật **Head Follow** và hình 2 camera cổ tay.
4. Mở `https://10.192.1.4/` → **Overview**, kiểm tra góc nhìn camera.
5. Kiểm tra ổ đĩa còn trống: **1 phút ghi ≈ 0,8–1,5 GB**.

### A.2 Cài đặt ghi dữ liệu (trong phần mềm thu dữ liệu)

| Tham số | Giá trị |
|---|---|
| File Name | Tùy ý |
| Domain ID | `0` (nhiều robot chung mạng thì mỗi robot một ID) |
| Format | `mcap` |
| Topic | 13 topic dưới đây |

```
/camera/left/color/image_resized/compressed    sensor_msgs/msg/CompressedImage
/camera/right/color/image_resized/compressed   sensor_msgs/msg/CompressedImage
/camera/top/color/image_raw/compressed         sensor_msgs/msg/CompressedImage
/gripper_command                               sensor_msgs/msg/JointState
/gripper_state                                 sensor_msgs/msg/JointState
/joint_command                                 sensor_msgs/msg/JointState
/joint_states                                  sensor_msgs/msg/JointState
/left_arm/ee_pose                              geometry_msgs/msg/PoseStamped
/left_arm/ee_pose_cmd                          geometry_msgs/msg/PoseStamped
/right_arm/ee_pose                             geometry_msgs/msg/PoseStamped
/right_arm/ee_pose_cmd                         geometry_msgs/msg/PoseStamped
/lifter/state
/chassis/twist
```

### A.3 Ghi một lần thao tác (episode)

| # | Việc | Nút VR |
|---|---|---|
| 1 | 2 tay về tư thế sẵn sàng | Giữ đồng thời **Grip trái + X** và **Grip phải + A** (~0,5 s sau tay bắt đầu chạy; nhả là dừng) |
| 2 | Bắt đầu ghi | **Y** (tay trái) — hoặc nút xanh ở Overview → Data Recording |
| 3 | Thao tác | Giữ **Grip** để tay bám theo, bóp **Trigger** để kẹp. Gắp kẹo, đặt vào tay người. **Chỉ cử động sau khi thấy bàn tay ngửa**; chưa có tay ngửa thì giữ yên |
| 4 | Về Home | **Grip phải + B** (hoặc **L1 + X** trên tay cầm) |
| 5 | Dừng ghi | **B** (tay phải). Ghi hỏng → nhấn **Y** lần nữa trong lúc đang ghi để hủy |
| 6 | Lặp lại 1–5 | |

### A.4 Chuyển đổi dữ liệu

Dùng script `limx_mcap2lerobot` (LimX cung cấp) chuyển `mcap` → LeRobot v2.1. Cấu trúc đúng:

```
lerobot_dataset/
├── data/chunk-000/
├── meta/episodes/chunk-000/
└── videos/
    ├── observation.images.cam_high/chunk-000/
    ├── observation.images.cam_left_wrist/chunk-000/
    └── observation.images.cam_right_wrist/chunk-000/
```

---

## Phụ lục B — Tự huấn luyện

**Yêu cầu:** GPU NVIDIA (khuyên **A100 / H100**), RAM ≥ 32 GB, ≥ 100 GB SSD NVMe, Ubuntu 22.04+, Python 3.11+. Cài môi trường giống Phần 2.

```bash
cd ~/limx/tron2_openpi
source .venv/bin/activate
export HF_LEROBOT_HOME=/path/to/datasets       # chứa thư mục my_dataset/{data,meta,videos}

# 1. Tạo file cấu hình riêng (không sửa trực tiếp config.py)
cp configs/train/tron2_tasks/example.yaml configs/train/tron2_tasks/my_task.yaml
#    sửa repo_id: my_dataset, prompt, đường dẫn checkpoint gốc...

# 2. Tính thông số chuẩn hóa (bắt buộc trước lần train đầu)
uv run scripts/compute_norm_stats.py --task-config configs/train/tron2_tasks/my_task.yaml

# 3. Huấn luyện
uv run scripts/train_tron2_task.py --task-config configs/train/tron2_tasks/my_task.yaml
```

Hoặc dùng script gộp (tự tính chuẩn hóa rồi train; thêm `--skip-norm` để bỏ bước tính):

```bash
scripts/cloud_train_entrypoint_portable.sh \
  --task-config configs/train/tron2_tasks/my_task.yaml \
  --exp my_task \
  --data-dir "$HF_LEROBOT_HOME" \
  --max-frames 100000
```

Train xong: sửa `candy_server.yaml` (hoặc file server mới) cho `config`, `repo_id`, `checkpoint_dir` khớp với kết quả train, rồi chạy như Phần 5.

Dataset gốc của LimX (1953 episode, LeRobot 2.1): https://huggingface.co/limx-tron2/tron2-openpi-models

---

## Phụ lục C — Checklist in ra giấy

**Chuẩn bị (1 lần)**
- [ ] Máy Ubuntu + GPU ≥ 12 GB, `nvidia-smi` chạy được
- [ ] Đã clone `tron2_openpi` + `tron2_env` cạnh nhau
- [ ] `uv sync` + cài `tron2_env[bridge,openpi]` thành công, lệnh kiểm tra in đúng đường dẫn
- [ ] Đã tải weights `pi05_tron2_candy/candy_h30_rtc_25000`

**Mỗi lần chạy**
- [ ] Bàn trắng 75 cm, robot cách mép bàn 15 cm, khay kem + kẹo Thỏ Trắng xếp dày ở giữa
- [ ] Ánh sáng ổn định, không nắng trực tiếp; trống 1 m quanh robot
- [ ] Robot bật, đèn đứng màu; tay cầm đã kết nối
- [ ] Máy tính cắm dây mạng, IP `10.192.1.120`, ping được `10.192.1.2` và `10.192.1.4`
- [ ] **L1 + X** → 2 tay buông thõng
- [ ] Chế độ **High-Level Dev** (đèn xanh lơ)
- [ ] Terminal 1: server chạy xong
- [ ] Người cầm tay cầm sẵn sàng E-Stop
- [ ] Terminal 2: client → tay về tư thế cẳng tay dựng đứng
- [ ] Đưa tay ngửa → nhận kẹo 🍬

**Kết thúc**
- [ ] Ctrl+C client → **L1 + X** → Ctrl+C server → tắt robot → tắt tay cầm

---

### Tài nguyên

| Tài nguyên | Link |
|---|---|
| Code train + inference | https://github.com/limx-tron2/tron2_openpi |
| Gói môi trường TRON2 | https://github.com/limx-tron2/tron2_env |
| Weights đã huấn luyện | https://huggingface.co/limxdynamics/tron2-openpi-models |
| Hướng dẫn gốc (EN) | [【Candy_Dispensing】TRON2_Dual-Arm_Manipulation_Quickstart_(Pi05).md](<../【Candy_Dispensing】TRON2_Dual-Arm_Manipulation_Quickstart_(Pi05).md>) |
| Hướng dẫn vận hành tay (VI) | [TRON2-tay.md](TRON2-tay.md) |
