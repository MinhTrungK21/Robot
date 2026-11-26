# Quy trình huấn luyện mô hình AI cho TRON 2 (2 tay) — từ A đến Z

> Tổng hợp từ: *Hướng dẫn bắt đầu nhanh thao tác hai cánh tay TRON2 (ACT)* (`train_act.pdf`), *Candy Dispensing Quickstart (Pi05)*, *TRON 2 User Manual* (V2.7) và repo `tron2_openpi`.
> Các chỗ ghi **💡 Kinh nghiệm** là khuyến nghị chung, **không phải** số liệu LimX công bố.

---

## Mục lục

- [0. Bức tranh tổng thể](#0-bức-tranh-tổng-thể)
- [1. Chọn mô hình: ACT hay Pi0.5?](#1-chọn-mô-hình-act-hay-pi05)
- [2. Thiết kế nhiệm vụ](#2-thiết-kế-nhiệm-vụ)
- [3. Chuẩn bị phần cứng](#3-chuẩn-bị-phần-cứng)
- [4. Thu dữ liệu](#4-thu-dữ-liệu)
- [5. Xuất và kiểm tra dữ liệu](#5-xuất-và-kiểm-tra-dữ-liệu)
- [6A. Huấn luyện ACT (LeRobot)](#6a-huấn-luyện-act-lerobot)
- [6B. Huấn luyện Pi0.5 (OpenPI)](#6b-huấn-luyện-pi05-openpi)
- [7. Triển khai lên robot](#7-triển-khai-lên-robot)
- [8. Đánh giá và cải thiện](#8-đánh-giá-và-cải-thiện)
- [9. Lỗi thường gặp](#9-lỗi-thường-gặp)
- [10. Checklist toàn bộ quy trình](#10-checklist-toàn-bộ-quy-trình)

---

## 0. Bức tranh tổng thể

```
 ① THIẾT KẾ        ② THU DỮ LIỆU             ③ XUẤT DỮ LIỆU        ④ HUẤN LUYỆN        ⑤ TRIỂN KHAI        ⑥ ĐÁNH GIÁ
 nhiệm vụ   ──►  Robot + kính VR      ──►  .mcap → LeRobot  ──►  ACT / Pi0.5   ──►  chạy trên robot ──►  đo tỉ lệ thành công
                 (teleop, ghi .mcap)                                                                        │
                                                                                                            │
     ◄──────────────────────── chưa đạt: thu thêm dữ liệu ở chỗ robot hay sai, huấn luyện lại ◄──────────────┘
```

**Ý tưởng cốt lõi — học bắt chước (imitation learning):**
Bạn dùng kính VR điều khiển tay robot làm mẫu một việc nhiều lần. Mỗi lần, hệ thống ghi lại **hình 3 camera** và **góc các khớp + độ mở kẹp**. Mô hình học quy tắc: *"thấy hình như thế này → tay phải cử động như thế kia"*. Sau khi học xong, robot tự làm lại việc đó mà không cần người điều khiển.

> **Dữ liệu quyết định chất lượng.** Mô hình chỉ giỏi bằng các lần làm mẫu của bạn. Làm mẫu cẩu thả, do dự, mỗi lần một kiểu thì robot cũng sẽ làm như vậy.

---

## 1. Chọn mô hình: ACT hay Pi0.5?

| | **ACT** | **Pi0.5** |
|---|---|---|
| Bản chất | Mô hình học bắt chước nhỏ, mỗi mô hình chỉ làm 1 việc | Mô hình lớn thị giác–ngôn ngữ–hành động (VLA), đã học trước rất nhiều dữ liệu, nhận câu lệnh bằng chữ |
| Thư viện | **LeRobot** (Hugging Face) | **OpenPI** (bản `tron2_openpi` của LimX) |
| GPU để huấn luyện | **≥ 8 GB**, khuyên RTX 3090 24 GB | Khuyên **A100 / H100** (thường phải thuê cloud) |
| GPU để chạy | ≥ 8 GB | ≥ 12 GB |
| Python | 3.12 (conda) | 3.11+ (uv) |
| Tài liệu triển khai lên robot | ⚠️ Còn thiếu (`act_deploy.zip` chưa có — xem mục 7A) | ✅ Đầy đủ |
| Phù hợp khi | Mới bắt đầu, có 1 máy GPU, việc đơn giản (gắp-thả 1 vật) | Việc phức tạp, cần tổng quát hơn, có GPU mạnh |

**👉 Khuyến nghị:** Bắt đầu bằng **ACT** với một việc đơn giản để học trọn quy trình. Dữ liệu thu cho ACT **dùng lại được nguyên** cho Pi0.5 (cùng định dạng LeRobot), nên sau này nâng cấp không phải thu lại.

---

## 2. Thiết kế nhiệm vụ

Làm kỹ bước này trước khi chạm vào robot.

### 2.1 Chọn việc

| Nên | Tránh (khi mới bắt đầu) |
|---|---|
| 1 vật, 1 đích: "gắp chuối đặt vào đĩa" | Nhiều bước nối nhau: "làm bánh mì kẹp" |
| Vật cứng, dễ kẹp, màu nổi trên nền trắng | Vật trong suốt, quá nhỏ, mềm, trơn |
| Vật nằm trong tầm với của tay | Vật ở mép bàn, sát thân robot |
| Kết thúc rõ ràng (vật đã nằm trong đĩa) | Kết thúc mơ hồ |

### 2.2 Viết "luật làm mẫu" và tuân thủ cho mọi lần ghi

Ví dụ của LimX với việc gắp chuối:
- Đĩa đặt **ngẫu nhiên** trong tầm nhìn camera đầu.
- Chuối đặt **ngẫu nhiên**, không nằm trên đĩa. Lần thì **dựng đứng**, lần thì **nằm ngang**, quay nhiều hướng.
- Chuối ở **bên trái đĩa → dùng tay trái**; ở **bên phải → dùng tay phải**.
- Làm xong → 2 tay về tư thế sẵn sàng.

Ví dụ của LimX với việc gắp kẹo: **chỉ cử động khi thấy bàn tay ngửa**; chưa có tay thì giữ yên.

> 💡 **Kinh nghiệm:** Luật càng rõ, mô hình học càng nhanh. Mọi người thu dữ liệu phải làm **cùng một cách** (cùng tay, cùng hướng tiếp cận, cùng tốc độ).

### 2.3 Cố định môi trường

| Yếu tố | Giá trị LimX dùng | Ghi chú |
|---|---|---|
| Bàn | 160 × 80 × 75 cm, mặt trắng trơn | Không đổi bàn giữa lúc thu dữ liệu và lúc chạy |
| Vị trí robot | Mép giá đỡ cách mép bàn **15 cm** | Đo bằng thước, đánh dấu sàn |
| Góc đầu | **pitch = 1,04, yaw = 0** (chỉnh trong **LimX Studio**) | Chỉnh **trước** khi vào chế độ teleop |
| Ánh sáng | Đèn trong nhà, ổn định | Tránh nắng chiếu, tránh bật/tắt đèn giữa chừng |
| Nền | Không có vật lạ trong tầm camera đầu | |

> ⚠️ Mọi thứ trong bảng này **phải giống hệt** khi chạy mô hình. Đổi chiều cao bàn 5 cm cũng có thể làm robot gắp trượt.

---

## 3. Chuẩn bị phần cứng

| Thiết bị | Dùng để |
|---|---|
| TRON 2 bản **2 tay, EDU**, kẹp LimX 2 ngón, đủ 3 camera (đầu + 2 cổ tay) | Làm mẫu và chạy mô hình |
| **Developer Expansion Module** (`10.192.1.4`) | Chạy phần mềm thu dữ liệu, lưu file `.mcap` |
| Kính VR **PICO 4 Ultra** + 2 tay cầm | Điều khiển tay robot khi làm mẫu |
| Tay cầm robot | Đổi chế độ, về Home, E-Stop |
| Máy GPU Ubuntu 22.04 | Huấn luyện và chạy mô hình (cấu hình ở mục 1) |
| Ổ cứng | 1 phút ghi ≈ **0,8–1,5 GB** → 100 lần ghi × 30 s ≈ 40–75 GB |

---

## 4. Thu dữ liệu

### 4.1 Kết nối và mở phần mềm thu dữ liệu

1. Bật robot (xem [TRON2-tay.md](TRON2-tay.md) mục 5).
2. Máy tính nối Wi-Fi robot (`TRON2A_XXX_5G` / `12345678`) hoặc cắm dây, IP `10.192.1.120`.
3. Mở `https://10.192.1.4/` → đăng nhập phần mềm thu dữ liệu (chấp nhận cảnh báo chứng chỉ).
4. **Kiểm tra camera:** ở trang **Overview** phải thấy cả 3 hình.
   - Hình **cổ tay trái và phải bị tráo** → đổi lại vị trí 2 camera.
   - Số seri camera đã cài sẵn từ nhà máy, không cần cấu hình.

### 4.2 Chỉnh góc đầu

Mở **LimX Studio** → *Robot settings (机器人设置)* → *Camera angle (摄像头角度设置)* → đặt **yaw = 0, pitch = 1.04** → **Confirm (确认)**. Xem lại hình camera đầu: phải thấy toàn bộ vùng làm việc trên bàn và 2 kẹp.

> Bản Mobile Dual-Arm: LimX Studio cũng có ô điều khiển giá nâng (升降台控制) — giữ **cùng một độ cao** cho mọi lần ghi.

### 4.3 Chuyển sang chế độ Teleop và kết nối kính VR

1. Tay cầm: **L1 + □** → `Idle` → **R1 + →/←** đến khi màn hình hiện **`Teleop`** + `Active`.
2. Kính VR: Wi-Fi robot 5 GHz → **Library → Unknown Sources → Limx_teleop** → **Tron2**, IP `10.192.1.2` → **Connect**.
3. Bật hình 2 camera cổ tay trong app. **Không bật Head Follow** nếu muốn giữ đầu cố định ở góc đã chỉnh.
4. Tập điều khiển vài phút **khi chưa ghi**: giữ **Grip** để tay bám theo, bóp **Trigger** để kẹp, nhả Grip để khóa tay.

### 4.4 Cài đặt ghi dữ liệu

| Tham số | Giá trị |
|---|---|
| File Name | Tên việc, VD `banana_plate` |
| Domain ID | `0` (nhiều robot chung mạng thì mỗi robot 1 ID) |
| Format | **`mcap`** |
| Topic | 13 topic dưới đây |

```
 1. /camera/left/color/image_resized/compressed    sensor_msgs/msg/CompressedImage   ← camera cổ tay trái
 2. /camera/right/color/image_resized/compressed   sensor_msgs/msg/CompressedImage   ← camera cổ tay phải
 3. /camera/top/color/image_raw/compressed         sensor_msgs/msg/CompressedImage   ← camera đầu
 4. /gripper_command                               sensor_msgs/msg/JointState        ← lệnh kẹp
 5. /gripper_state                                 sensor_msgs/msg/JointState        ← trạng thái kẹp
 6. /joint_command                                 sensor_msgs/msg/JointState        ← lệnh khớp (= "đáp án" cho mô hình)
 7. /joint_states                                  sensor_msgs/msg/JointState        ← góc khớp thực tế
 8. /left_arm/ee_pose                              geometry_msgs/msg/PoseStamped
 9. /left_arm/ee_pose_cmd                          geometry_msgs/msg/PoseStamped
10. /right_arm/ee_pose                             geometry_msgs/msg/PoseStamped
11. /right_arm/ee_pose_cmd                         geometry_msgs/msg/PoseStamped
12. /lifter/state
13. /chassis/twist
```

Kiểm tra trước khi ghi: **mạng ổn định**, **ổ đĩa còn trống**.

### 4.5 Ghi một lần làm mẫu (episode)

| # | Việc | Nút trên tay cầm VR |
|---|---|---|
| 1 | **Sắp đặt cảnh:** đặt vật và đích ở vị trí mới theo luật mục 2.2 | — |
| 2 | **2 tay về tư thế sẵn sàng** | Giữ đồng thời **Grip trái + X** và **Grip phải + A** (~0,5 s sau tay mới chạy; nhả là dừng) |
| 3 | **Bắt đầu ghi** | **Y** (tay trái) — hoặc nút Start ở *Overview → Data Recording* |
| 4 | **Làm mẫu**: điều khiển tay gắp vật đặt vào đích, dứt khoát, không do dự | Grip + di chuyển, Trigger để kẹp |
| 5 | **Về tư thế sẵn sàng** | Grip trái + X và Grip phải + A |
| 6 | **Dừng ghi** | **B** (tay phải) — hoặc nút Stop trên phần mềm |
| — | Làm hỏng giữa chừng → **hủy** | Nhấn **Y** lần nữa khi đang ghi (xóa bản ghi vừa rồi) |

> ⚠️ **Nút Y/B:** Bảng trên theo **User Manual V2.7** (Y = bắt đầu, B = dừng). Bản dịch `train_act.pdf` ghi ngược lại (B bắt đầu, Y dừng). **Thử 1 lần ngắn** rồi xem trong phần mềm để biết robot của bạn đang dùng cách nào.
>
> Nếu nút VR không ăn sau khi nâng cấp firmware → cập nhật phần mềm thu dữ liệu: `ssh guest@10.192.1.4` (mật khẩu `123456`) → `cd limx-agent/ && git pull && bash upgrade.sh` → khởi động lại robot.

### 4.6 Bao nhiêu dữ liệu là đủ?

| | Số lần ghi |
|---|---|
| LimX — gắp kẹo (Pi0.5) | **1953** |
| 💡 ACT, việc đơn giản, chạy thử quy trình | **50** |
| 💡 ACT, muốn chạy khá ổn định | **100–200** |
| 💡 Pi0.5 fine-tune | Vài trăm trở lên |

💡 **Kinh nghiệm thu dữ liệu tốt:**
- **Đa dạng vị trí:** rải đều vật và đích khắp vùng làm việc, đừng dồn một chỗ. Robot chỉ làm tốt ở những chỗ đã thấy trong dữ liệu.
- **Đều đặn:** mọi lần làm cùng một kiểu, cùng tốc độ vừa phải.
- **Mỗi lần dài vừa đủ** (VD 10–30 s); không để thừa nhiều giây đứng yên ở đầu và cuối.
- **Không giữ lại lần làm hỏng** (rơi vật, va bàn) — hủy ngay bằng Y.
- **Chia nhiều buổi** thu dữ liệu nhưng **giữ nguyên môi trường** (mục 2.3).
- Ghi chép: ngày, người thu, số lần ghi, ghi chú bất thường.

### 4.7 Xem lại dữ liệu

Trong phần mềm thu dữ liệu → **Data**: xem lại, tải xuống hoặc xóa từng bản ghi. Xóa ngay các bản ghi lỗi (hình đen, thiếu camera, quá ngắn).

---

## 5. Xuất và kiểm tra dữ liệu

### 5.1 Chọn cách xuất

| Loại việc | Ví dụ | Cách xuất |
|---|---|---|
| **Đơn giản**, 1 giai đoạn | Gắp chuối bỏ vào đĩa | **Phần mềm thu dữ liệu** → *Data → Export to LeRobot dataset* (có chú thích hàng loạt) |
| **Phức tạp**, nhiều giai đoạn | Làm bánh mì kẹp (lấy bánh → thêm rau → thêm thịt) | **Embodied AI Data Platform** của LimX (chia giai đoạn, chú thích chi tiết) |
| Tự xử lý trên máy mạnh | | Script **`limx_mcap2lerobot.zip`** do LimX cung cấp |

### 5.2 Kiểm tra cấu trúc thư mục

Chép dataset sang máy GPU, ví dụ `~/datasets/banana_plate`, rồi:

```bash
cd ~/datasets/banana_plate
tree -d
```

Phải ra:
```
.
├── data/chunk-000/
├── meta/episodes/chunk-000/
└── videos/
    ├── observation.images.cam_high/chunk-000/          ← camera đầu
    ├── observation.images.cam_left_wrist/chunk-000/    ← camera cổ tay trái
    └── observation.images.cam_right_wrist/chunk-000/   ← camera cổ tay phải
```

### 5.3 Kiểm tra nội dung

```bash
# Phiên bản định dạng, số episode, số khung hình, fps, danh sách đặc trưng
python3 -c "import json; d=json.load(open('meta/info.json')); print({k:d[k] for k in ['codebase_version','total_episodes','total_frames','fps']}); print(list(d['features'].keys()))"
```

✅ Cần thấy:
- `total_episodes` đúng bằng số lần ghi bạn giữ lại.
- Có `observation.images.cam_high`, `observation.images.cam_left_wrist`, `observation.images.cam_right_wrist`, `observation.state`, `action`.
- Ghi lại `codebase_version` (VD `v3.0` hoặc `v2.1`) — cần ở mục 9 nếu LeRobot báo lỗi phiên bản.

💡 Mở vài file video trong `videos/` bằng trình phát bất kỳ để chắc hình không đen, không tráo camera.

---

## 6A. Huấn luyện ACT (LeRobot)

### 6A.1 Cài môi trường (1 lần)

```bash
# Cài Miniconda nếu chưa có: https://docs.conda.io/en/latest/miniconda.html
nvidia-smi                                     # kiểm tra GPU + driver

git clone https://github.com/huggingface/lerobot.git
cd lerobot
conda create -n lerobot python=3.12 -y
conda activate lerobot
pip install -e .

# Gói phụ thuộc thêm cho ACT
pip install einops timm wandb

# (Tùy chọn) Đăng nhập Weights & Biases để xem biểu đồ huấn luyện trên web
wandb login
```

Kiểm tra PyTorch thấy GPU:
```bash
python -c "import torch; print(torch.__version__, torch.cuda.is_available(), torch.cuda.get_device_name(0))"
```
✅ Phải in ra `True` và tên GPU. Nếu `False` → cài lại PyTorch đúng bản CUDA theo hướng dẫn tại pytorch.org.

### 6A.2 Chạy huấn luyện

```bash
conda activate lerobot
cd ~/lerobot

python -m lerobot.scripts.lerobot_train \
  --policy.type=act \
  --dataset.repo_id="banana_plate" \
  --dataset.root="/home/<user>/datasets/banana_plate" \
  --dataset.video_backend=pyav \
  --steps=3000 \
  --batch_size=8 \
  --log_freq=100 \
  --save_freq=1000 \
  --policy.push_to_hub=false \
  --output_dir=outputs/act_banana_plate
```

| Tham số | Ý nghĩa | Ghi chú |
|---|---|---|
| `--policy.type=act` | Dùng mô hình ACT | |
| `--dataset.repo_id` | Tên dataset | Đặt tùy ý |
| `--dataset.root` | **Đường dẫn tuyệt đối** tới thư mục dataset | ⚠️ **Bắt buộc sửa** |
| `--dataset.video_backend=pyav` | Thư viện giải mã video | Giữ nguyên |
| `--steps` | Số bước huấn luyện | LimX để 3000 — 💡 chỉ đủ để chạy thử, xem dưới |
| `--batch_size` | Số mẫu mỗi bước | Hết VRAM → giảm xuống 4 |
| `--log_freq` | Bao nhiêu bước in log 1 lần | |
| `--save_freq` | Bao nhiêu bước lưu checkpoint 1 lần | |
| `--policy.push_to_hub=false` | Không tải mô hình lên Hugging Face | |
| `--output_dir` | Thư mục lưu kết quả | **Không trùng** thư mục cũ, nếu không sẽ báo lỗi |

> Bản `train_act.pdf` do AI dịch đã dịch luôn tên tham số ("loại.chính sách=hành động", "kích thước lô"…). Khi gõ lệnh **phải dùng tên tiếng Anh** như trên.

💡 **Chọn số bước (`--steps`):**
1. Lần đầu chạy **3000** bước để chắc quy trình không lỗi (vài phút đến vài chục phút).
2. Sau đó chạy thật với **20.000–100.000** bước (mặc định của LeRobot cho ACT là 100.000). Với 50–200 episode, thường thử 30.000–50.000 trước.
3. Đặt `--save_freq` khoảng 5000–10000 để có nhiều checkpoint mà so sánh.

### 6A.3 Theo dõi

- Log trong terminal in `loss` mỗi `log_freq` bước. ✅ `loss` **giảm dần** rồi đi ngang là bình thường.
- Nếu đã `wandb login`: thêm `--wandb.enable=true` để xem biểu đồ trên web.
- Theo dõi GPU ở terminal khác: `watch -n 1 nvidia-smi`.
- Chạy qua SSH lâu giờ → dùng `tmux` để tắt máy mình mà huấn luyện vẫn chạy:
  ```bash
  tmux new -s train      # chạy lệnh huấn luyện trong này
  # Ctrl+B rồi D để thoát ra; quay lại: tmux attach -t train
  ```

### 6A.4 Kết quả

```
outputs/act_banana_plate/
└── checkpoints/
    ├── 001000/pretrained_model/    ← checkpoint ở bước 1000
    ├── 002000/pretrained_model/
    ├── 003000/pretrained_model/
    └── last -> 003000              ← checkpoint mới nhất
```

Mỗi thư mục `pretrained_model/` chứa `config.json`, `model.safetensors` (trọng số) và `train_config.json`. Thư mục này là thứ đem đi triển khai.

**Huấn luyện tiếp khi bị ngắt:**
```bash
python -m lerobot.scripts.lerobot_train \
  --config_path=outputs/act_banana_plate/checkpoints/last/pretrained_model/train_config.json \
  --resume=true
```

---

## 6B. Huấn luyện Pi0.5 (OpenPI)

Dùng khi có GPU A100/H100 (hoặc thuê cloud) và muốn mô hình mạnh hơn. Dùng **cùng dataset** đã xuất ở mục 5.

### 6B.1 Cài môi trường

```bash
mkdir -p ~/limx && cd ~/limx
git clone https://github.com/limx-tron2/tron2_openpi.git
git clone https://github.com/limx-tron2/tron2_env.git
cd tron2_openpi
sudo apt install -y pkg-config software-properties-common
sudo add-apt-repository ppa:ubuntuhandbook1/ffmpeg7 -y
sudo apt install -y ffmpeg libavformat-dev libavcodec-dev libavdevice-dev libavutil-dev libswscale-dev libswresample-dev libavfilter-dev
uv sync && uv pip install -e .
source .venv/bin/activate
uv pip install -e "../tron2_env[bridge,openpi]"
```

### 6B.2 Cấu hình và huấn luyện

```bash
# Dataset phải nằm ở $HF_LEROBOT_HOME/<repo_id>/  (VD ~/datasets/banana_plate)
export HF_LEROBOT_HOME=~/datasets

# 1. Tạo file cấu hình riêng (không sửa src/openpi/training/config.py)
cp configs/train/tron2_tasks/example.yaml configs/train/tron2_tasks/banana.yaml
#    Sửa trong banana.yaml: repo_id: banana_plate, prompt (câu lệnh), checkpoint gốc, thư mục xuất...

# 2. Tính thông số chuẩn hóa (bắt buộc trước lần đầu)
uv run scripts/compute_norm_stats.py --task-config configs/train/tron2_tasks/banana.yaml

# 3. Huấn luyện
uv run scripts/train_tron2_task.py --task-config configs/train/tron2_tasks/banana.yaml
```

Hoặc dùng script gộp (tự tính chuẩn hóa rồi huấn luyện; thêm `--skip-norm` để bỏ bước tính):
```bash
scripts/cloud_train_entrypoint_portable.sh \
  --task-config configs/train/tron2_tasks/banana.yaml \
  --exp banana \
  --data-dir "$HF_LEROBOT_HOME" \
  --max-frames 100000
```

File YAML cấu hình được: `repo_id`, `prompt`, các cột dữ liệu, `action_horizon`, `state_dim`, checkpoint gốc, thư mục xuất, `prompt_from_task`, `rtc_training_simulated_delay`.

---

## 7. Triển khai lên robot

### 7A. Mô hình ACT

Theo `train_act.pdf` mục 6:

1. **Rút dây dữ liệu camera D405** (camera cổ tay) khỏi máy tính trên robot, **cắm thẳng vào máy GPU** chạy mô hình.
2. Chuyển robot sang **chế độ nhà phát triển** (Developer Mode — tài liệu không ghi rõ High-Level hay Low-Level; nhiều khả năng là **High-Level Dev**, đèn xanh lơ).
3. Đặt 2 tay **phía trên mặt bàn** theo tư thế ảnh minh họa của LimX.
4. Giữ chiều cao bàn, khoảng cách người–bàn, ánh sáng, góc đầu **giống lúc thu dữ liệu**.
5. Tải gói **`act_deploy.zip`**, sửa các giá trị trong **`act_deploy.py`** (đường dẫn checkpoint, IP robot, camera…), rồi chạy script.

> ⚠️ **Đang thiếu:** `act_deploy.zip` và ảnh hướng dẫn sửa `act_deploy.py` chưa có (trang Feishu yêu cầu đăng nhập, bản PDF in bị mất ảnh). Khi tải được file zip về thư mục `tron2/`, tài liệu phần này sẽ được bổ sung chi tiết.

### 7B. Mô hình Pi0.5

Làm giống [TRON2-gap-keo.md](TRON2-gap-keo.md) Phần 5, chỉ khác file cấu hình server:

```bash
cp configs/deploy/candy_server.yaml configs/deploy/banana_server.yaml
cp configs/deploy/candy_client.yaml configs/deploy/banana_client.yaml
```

Sửa `banana_server.yaml`:
```yaml
policy:
  config: <tên config huấn luyện>          # khớp với lúc train
  repo_id: banana_plate                    # khớp thư mục asset trong checkpoint
  checkpoint_dir: <đường dẫn checkpoint/bước>
  default_prompt: <câu lệnh lúc train>
```

Chạy:
```bash
# Robot: L1 + X (2 tay buông thõng) → chế độ High-Level Dev
uv run scripts/serve_policy.py --profile configs/deploy/banana_server.yaml            # Terminal 1
uv run examples/tron2/pi_client_rtc.py --profile configs/deploy/banana_client.yaml    # Terminal 2
```

### An toàn khi chạy mô hình lần đầu

- Dọn trống **1 m** quanh robot; một người cầm tay cầm, sẵn sàng **E-Stop (nhấn 2 joystick cùng lúc)**.
- Bắt đầu với cảnh **dễ nhất** (vật ở vị trí giống dữ liệu nhiều nhất).
- Mô hình mới có thể cử động bất ngờ — đứng ngoài tầm với của tay robot.
- Dừng: Ctrl+C chương trình → **L1 + X** đưa tay về.

---

## 8. Đánh giá và cải thiện

### 8.1 Đo tỉ lệ thành công

💡 Chạy **10–20 lần** với vị trí vật **khác nhau**, ghi kết quả từng lần:

| Lần | Vị trí vật | Vị trí đích | Kết quả | Kiểu lỗi |
|---|---|---|---|---|
| 1 | trái, nằm ngang | giữa | ✅ | |
| 2 | phải, dựng đứng | trái | ❌ | kẹp trượt |
| … | | | | |

**Tỉ lệ thành công = số lần ✅ / tổng số lần.**

### 8.2 So sánh checkpoint

Thử vài checkpoint (VD bước 20k, 40k, cuối cùng) trên **cùng bộ vị trí thử**. Checkpoint cuối **không phải lúc nào cũng tốt nhất**.

### 8.3 Cải thiện theo kiểu lỗi

| Robot hay… | Nguyên nhân thường gặp | Cách sửa |
|---|---|---|
| Đứng yên, không làm gì | Cảnh khác dữ liệu (ánh sáng, bàn, góc đầu) | Đưa môi trường về giống lúc thu |
| Sai ở **một vùng** trên bàn | Ít dữ liệu ở vùng đó | **Thu thêm** dữ liệu ở vùng đó |
| Kẹp trượt, hụt vài cm | Dữ liệu chưa đủ, làm mẫu không đều | Thu thêm, làm mẫu dứt khoát hơn; huấn luyện thêm bước |
| Do dự, tay rung, đi đi lại lại | Dữ liệu có nhiều đoạn do dự, mỗi lần một kiểu | Lọc bỏ episode xấu, thống nhất cách làm |
| Làm tốt lúc đầu rồi lạc | Ít ví dụ cho tình huống giữa chừng | Thu thêm các lần có tình huống đó |
| Làm hoàn hảo đúng vị trí cũ, sai vị trí mới | Học thuộc, dữ liệu thiếu đa dạng | Rải vị trí đều hơn, thêm dữ liệu |

**Vòng lặp:** thu thêm dữ liệu → xuất lại → huấn luyện lại (từ đầu, với toàn bộ dữ liệu cũ + mới) → đánh giá lại.

---

## 9. Lỗi thường gặp

| Lỗi | Cách xử lý |
|---|---|
| `torch.cuda.is_available()` = `False` | Cài driver NVIDIA; cài lại PyTorch đúng bản CUDA |
| `CUDA out of memory` | Giảm `--batch_size` (8 → 4); tắt chương trình khác đang dùng GPU |
| Lỗi không tìm thấy dataset | `--dataset.root` phải là **đường dẫn tuyệt đối**, trỏ tới thư mục chứa `data/`, `meta/`, `videos/` |
| Lỗi về `codebase_version` / phiên bản dataset | Dataset và LeRobot lệch phiên bản. Xem `meta/info.json` → `codebase_version`. LeRobot mới dùng định dạng `v3.0`; nếu dataset là `v2.1` thì dùng script chuyển đổi đi kèm LeRobot hoặc cài bản LeRobot cũ hơn hỗ trợ `v2.1` |
| Lỗi giải mã video | Giữ `--dataset.video_backend=pyav`; kiểm tra đã cài `ffmpeg` |
| Báo lỗi thư mục output đã tồn tại | Đổi `--output_dir` sang tên mới, hoặc dùng `--resume=true` để chạy tiếp |
| `loss` không giảm | Kiểm tra dữ liệu (hình đen, camera tráo, action toàn 0); kiểm tra số episode |
| Nút VR không ăn / dữ liệu ghi thiếu | Cập nhật `limx-agent` trên `10.192.1.4` (mục 4.5) rồi khởi động lại robot |
| Camera cổ tay trái/phải bị tráo | Đổi vị trí 2 camera, thu lại |

---

## 10. Checklist toàn bộ quy trình

**① Thiết kế**
- [ ] Chọn việc đơn giản, viết luật làm mẫu
- [ ] Cố định bàn 75 cm, robot cách mép bàn 15 cm, ánh sáng, nền trắng
- [ ] Chọn mô hình: ACT (1 máy GPU) hay Pi0.5 (GPU lớn)

**② Thu dữ liệu**
- [ ] Góc đầu pitch 1,04 / yaw 0 trong LimX Studio
- [ ] 3 camera hiện đủ, không tráo
- [ ] Chế độ Teleop, kính VR kết nối
- [ ] Format `mcap`, Domain ID 0, đủ 13 topic
- [ ] Thử 1 lần ngắn để chắc nút bắt đầu/dừng ghi
- [ ] Thu ≥ 50 episode, vị trí rải đều, hủy lần làm hỏng

**③ Xuất dữ liệu**
- [ ] Xuất sang LeRobot
- [ ] `tree -d` đúng cấu trúc, `meta/info.json` đúng số episode, đủ 3 camera

**④ Huấn luyện**
- [ ] `torch.cuda.is_available()` = True
- [ ] Chạy thử 3000 bước không lỗi
- [ ] Chạy thật 20k–100k bước, `loss` giảm

**⑤ Triển khai**
- [ ] Môi trường giống lúc thu dữ liệu
- [ ] Chế độ Developer, 2 tay đúng tư thế khởi đầu
- [ ] Người cầm tay cầm sẵn sàng E-Stop

**⑥ Đánh giá**
- [ ] Thử 10–20 lần, ghi tỉ lệ thành công
- [ ] So sánh vài checkpoint
- [ ] Thu thêm dữ liệu ở chỗ robot hay sai → huấn luyện lại

---

### Tài liệu liên quan

| Tài liệu | Nội dung |
|---|---|
| [TRON2-tay.md](TRON2-tay.md) | Vận hành robot 2 tay, kính VR, các chế độ |
| [TRON2-gap-keo.md](TRON2-gap-keo.md) | Chạy mô hình Pi0.5 có sẵn (gắp kẹo) |
| `../train_act.pdf` | Hướng dẫn ACT gốc (bản AI dịch) |
| [Candy Dispensing Quickstart](<../【Candy_Dispensing】TRON2_Dual-Arm_Manipulation_Quickstart_(Pi05).md>) | Hướng dẫn Pi0.5 gốc |
| https://github.com/huggingface/lerobot | Thư viện LeRobot (ACT) |
| https://github.com/limx-tron2/tron2_openpi | OpenPI cho TRON 2 (Pi0.5) |
