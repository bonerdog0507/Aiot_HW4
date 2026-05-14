# 🍓 Raspberry Pi 部署指南

## 需要傳到樹莓派的檔案

你只需要這些檔案（訓練已在 PC 上完成）：

```
RSP_demo/
├── hand_landmarker.task      ← MediaPipe 手部模型 (約 10MB)
└── demo/
    ├── carema_landmark.py    ← 即時辨識程式
    ├── best_landmark_model.pkl  ← 訓練好的 SVM 模型
    └── requirements.txt      ← 依賴清單
```

---

## Step 1: 傳檔案到樹莓派

### 方法 A: 用 SCP（推薦）

在 Windows 的 PowerShell 中執行：

```powershell
# 先用 scp 傳整個專案（請替換 <使用者名稱> 和 <主機名>）
scp -r C:\Users\vanos\Desktop\abc\RSP_demo-main <使用者名稱>@<主機名>.local:~/RSP_demo
```

例如，如果你的 RPi 使用者名稱是 `pi`，主機名是 `raspberrypi`：
```powershell
scp -r C:\Users\vanos\Desktop\abc\RSP_demo-main pi@raspberrypi.local:~/RSP_demo
```

### 方法 B: 用 USB 隨身碟

1. 把 `RSP_demo-main` 整個資料夾複製到 USB
2. 插到樹莓派
3. 複製到家目錄：
   ```bash
   cp -r /media/pi/USB隨身碟名稱/RSP_demo-main ~/RSP_demo
   ```

---

## Step 2: 在樹莓派上安裝依賴

SSH 進入樹莓派：
```bash
ssh <使用者名稱>@<主機名>.local
```

安裝系統套件（OpenCV 需要的）：
```bash
sudo apt update
sudo apt install -y python3-pip python3-opencv libatlas-base-dev
```

安裝 Python 套件：
```bash
cd ~/RSP_demo
pip3 install mediapipe opencv-python numpy scikit-learn joblib
```

> **注意：** 如果 `pip3 install mediapipe` 失敗，試試：
> ```bash
> pip3 install mediapipe --break-system-packages
> ```
> 或使用虛擬環境：
> ```bash
> python3 -m venv venv
> source venv/bin/activate
> pip install mediapipe opencv-python numpy scikit-learn joblib
> ```

---

## Step 3: 接上攝影機並執行

1. 接上 USB 攝影機 或 啟用 CSI 攝影機
2. 執行即時辨識：

```bash
cd ~/RSP_demo/demo
python3 carema_landmark.py
```

### 操作說明
| 按鍵 | 功能 |
|------|------|
| `s` | 截圖（儲存到 `demo/screenshots/`）|
| `q` | 退出 |

---

## 常見問題

### Q1: 攝影機打不開？
```bash
# 確認攝影機有被偵測到
ls /dev/video*

# 如果是 CSI 攝影機，需要先啟用
sudo raspi-config
# → Interface Options → Camera → Enable
```

### Q2: MediaPipe 安裝失敗？
Raspberry Pi OS 64-bit 才支援 MediaPipe。確認你的系統：
```bash
uname -m
# 應該顯示 aarch64
```

如果是 32-bit 系統，需要重刷 64-bit 的 Raspberry Pi OS。

### Q3: 顯示 "cannot open display"？
如果你用 SSH 連線，需要啟用 X11 forwarding：
```bash
ssh -X <使用者名稱>@<主機名>.local
```
或者直接在樹莓派上接螢幕操作。

### Q4: 畫面很卡？
在 `carema_landmark.py` 中可以降低解析度：
```python
cap = cv2.VideoCapture(0)
cap.set(cv2.CAP_PROP_FRAME_WIDTH, 320)
cap.set(cv2.CAP_PROP_FRAME_HEIGHT, 240)
```
