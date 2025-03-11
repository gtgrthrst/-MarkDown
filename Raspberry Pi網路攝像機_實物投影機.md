---
disqus: ahb0222
GA : G-VF9ZT413CG
---
# Raspberry Pi網路攝像機_實物投影機
> [color=#40f1ef][name=LHB阿好伯, 2021/07/16][:earth_africa:](https://www.facebook.com/LHB0222/)
###### tags: `Raspberry Pi`

[TOC]


![](https://hackmd.io/_uploads/S13Sj1XY5.png)

前陣子在網路上跟著集資一款[樹莓派1600萬像素的攝像頭模組](https://shopee.tw/product/4343295/21610481358/)

解析度比v2攝像頭高2倍，比HQ攝像頭高40%

1600萬像素索尼IMX519鏡頭

 V1/V2 尺寸，帶自動對焦功能

最近剛好有空來進行一些測試

畫質上來說真的非常好

但相對的需要使用到的效能就更大

Raspberry Pi 2W就無法使用錄製全尺寸的影片

作為實物投影機串流只能選擇640*480尺寸

後續依照下方說明安裝所需套件
# IMX519 自動對焦相機
https://www.arducam.com/docs/cameras-for-raspberry-pi/raspberry-pi-libcamera-guide/

使用picamera2的範例製作Web伺服器
將畫面傳輸至網路上可以使用電腦瀏覽器觀看

# picamera2
https://github.com/raspberrypi/picamera2

```python=
#!/usr/bin/python3

# Mostly copied from https://picamera.readthedocs.io/en/release-1.13/recipes2.html
# Run this script, then point a web browser at http:<this-ip-address>:8000
# Note: needs simplejpeg to be installed (pip3 install simplejpeg).

from picamera2 import Picamera2
from picamera2.encoders import JpegEncoder
from picamera2.outputs import FileOutput
import io
import logging
import socketserver
from threading import Condition, Thread
from http import server

PAGE = """\
<html>
<head>
<meta charset="utf-8">
<title>picamera2 MJPEG streaming demo</title>
</head>
<body>
<h1>阿好伯分享平台</h1>
<img src="stream.mjpg" width="640" height="480" />
</body>
</html>
"""


class StreamingOutput(io.BufferedIOBase):
    def __init__(self):
        self.frame = None
        self.condition = Condition()

    def write(self, buf):
        with self.condition:
            self.frame = buf
            self.condition.notify_all()


class StreamingHandler(server.BaseHTTPRequestHandler):
    def do_GET(self):
        if self.path == '/':
            self.send_response(301)
            self.send_header('Location', '/index.html')
            self.end_headers()
        elif self.path == '/index.html':
            content = PAGE.encode('utf-8')
            self.send_response(200)
            self.send_header('Content-Type', 'text/html')
            self.send_header('Content-Length', len(content))
            self.end_headers()
            self.wfile.write(content)
        elif self.path == '/stream.mjpg':
            self.send_response(200)
            self.send_header('Age', 0)
            self.send_header('Cache-Control', 'no-cache, private')
            self.send_header('Pragma', 'no-cache')
            self.send_header('Content-Type', 'multipart/x-mixed-replace; boundary=FRAME')
            self.end_headers()
            try:
                while True:
                    with output.condition:
                        output.condition.wait()
                        frame = output.frame
                    self.wfile.write(b'--FRAME\r\n')
                    self.send_header('Content-Type', 'image/jpeg')
                    self.send_header('Content-Length', len(frame))
                    self.end_headers()
                    self.wfile.write(frame)
                    self.wfile.write(b'\r\n')
            except Exception as e:
                logging.warning(
                    'Removed streaming client %s: %s',
                    self.client_address, str(e))
        else:
            self.send_error(404)
            self.end_headers()


class StreamingServer(socketserver.ThreadingMixIn, server.HTTPServer):
    allow_reuse_address = True
    daemon_threads = True


picam2 = Picamera2()
picam2.configure(picam2.video_configuration(main={"size": (640*2, 480*2)}))
output = StreamingOutput()
picam2.start_recording(JpegEncoder(), FileOutput(output))

try:
    address = ('', 8000)
    server = StreamingServer(address, StreamingHandler)
    server.serve_forever()
finally:
    picam2.stop_recording()
```
瀏覽器輸入`http://[Raspberry_Pi_IP]:8000/`

![](https://hackmd.io/_uploads/HyxityQFc.jpg)

![](https://hackmd.io/_uploads/SktnNgmK5.jpg)


# 縮時攝影機

```python=

from picamera2 import Picamera2, Preview
import time

picam2 = Picamera2()
picam2.start_preview(Preview.QTGL)
preview_config = picam2.preview_configuration()
capture_config = picam2.still_configuration()
picam2.configure(preview_config)
picam2.start()
    
def capture_frame(frame):
    picam2.switch_mode_and_capture_file(capture_config, '/home/pi/Pi/20220612/frame%03d.jpg' % frame)
# The parameters such as No of days the camera should be active, frames per hour and total no of frames

RUN_DAYS = 1
BLANK_s = 10
FRAMES = int(RUN_DAYS * 24 * 60 * 60 / BLANK_s )

#Pi camera as the source



for frame in range(538,FRAMES):
# Note the time before the capture
    start = time.time()
    capture_frame(frame)
    # Wait for the next capture. Note that we take into account the length of time it took to capture the image when calculating the delay
    time.sleep(
    int(BLANK_s )
    )


```

# 安裝說明
:::success 
### 版本說明
cat /etc/os-release
PRETTY_NAME="Debian GNU/Linux 12 (bookworm)"
NAME="Debian GNU/Linux"
VERSION_ID="12"
VERSION="12 (bookworm)"
VERSION_CODENAME=bookworm
ID=debian
HOME_URL="https://www.debian.org/"
SUPPORT_URL="https://www.debian.org/support"
BUG_REPORT_URL="https://bugs.debian.org/"
:::
## 下載並設置安裝腳本
```
wget -O install_pivariety_pkgs.sh https://github.com/ArduCAM/Arducam-Pivariety-V4L2-Driver/releases/download/install_script/install_pivariety_pkgs.sh

chmod +x install_pivariety_pkgs.sh


```
## 安裝 libcamera 相關套件
```
./install_pivariety_pkgs.sh -p libcamera_dev
./install_pivariety_pkgs.sh -p libcamera_apps
```
### 修改設定檔
`sudo nano /boot/firmware/config.txt`

:::info
`在 [all] 下方添加：`
dtoverlay=imx519,vcm=off

:::
重新啟動
```
sudo reboot
```

## 啟用相機介面
舊版本才需要
```
sudo raspi-config
```
![image](https://hackmd.io/_uploads/r1CPYEIPke.png)

# 定時拍攝
```
#!/bin/bash

# 建立儲存照片的資料夾（如果不存在）
mkdir -p timelapse_photos

while true; do
    # 取得當前時間作為檔案名稱
    timestamp=$(date +"%Y%m%d_%H%M%S")
    
    # 拍攝照片，使用最大解析度 4656x3496
    libcamera-still -o timelapse_photos/${timestamp}.jpg --width 4656 --height 3496
    
    # 等待 60 秒
    sleep 60
done
```

# 定時任務_tmux

```
sudo apt install tmux
# 建立新工作階段
tmux new -s timelapse
```
創建.sh檔
```
nano photo_timelapse.sh
```

```
#!/bin/bash

# 設定變數
PHOTO_DIR="timelapse_photos"
LOG_DIR="timelapse_logs"
INTERVAL=60  # 拍攝間隔（秒）
WIDTH=4656
HEIGHT=3496

# 建立所需資料夾
mkdir -p "$PHOTO_DIR"
mkdir -p "$LOG_DIR"

# 記錄開始時間
echo "開始執行時間: $(date)" > "$LOG_DIR/timelapse.log"

# 捕捉 Ctrl+C 信號
trap 'echo "程式結束時間: $(date)" >> "$LOG_DIR/timelapse.log"; exit' INT

# 主要拍攝迴圈
while true; do
    # 取得時間戳記
    timestamp=$(date +"%Y%m%d_%H%M%S")
    
    # 拍攝照片
    libcamera-still -o "$PHOTO_DIR/${timestamp}.jpg" --width $WIDTH --height $HEIGHT
    
    # 記錄到日誌
    echo "拍攝完成: ${timestamp}.jpg" >> "$LOG_DIR/timelapse.log"
    
    # 等待下一次拍攝
    sleep $INTERVAL
done
```

```bash=
#sudo nano /etc/systemd/system/timelapse.service

[Unit]
Description=定時拍攝服務

[Service]
Type=simple
User=ahb0222-2w
Group=ahb0222-2w
WorkingDirectory=/home/ahb0222-2w
Environment=HOME=/home/ahb0222-2w
Environment=DISPLAY=:0

# 攝像頭需要視訊群組權限
SupplementaryGroups=video gpio i2c spi

ExecStart=/bin/bash /home/ahb0222-2w/photo_timelapse.sh

Restart=always
RestartSec=60

StandardOutput=journal
StandardError=journal

[Install]
WantedBy=multi-user.target
```
查看timelapse 服務的狀態
`sudo systemctl status timelapse.service`


## 設定權限
```
chmod +x photo_timelapse.sh
```
## 執行腳本
```
# 建立新的 tmux 工作階段
tmux new -s photo

# 在 tmux 中執行腳本
./photo_timelapse.sh
```

自動建立照片和日誌資料夾
記錄每張照片的拍攝時間
可以透過 Ctrl+C 正常結束並記錄結束時間
使用變數方便修改設定

## 查看日誌
tail -f timelapse_logs/timelapse.log

## 檢查照片資料夾
ls -l timelapse_photos/

###  tmux 工作階段管理
```
tmux ls
#連接到現有的 photo 工作階段
tmux attach -t photo
#結束現有的 photo 工作階段
tmux kill-session -t photo
```

[自啟動服務_計算人數](/LTo60VePQ6GVMsKsj2yoFQ)

## Wifi 無頭連線管理
```
curl "https://www.raspberryconnect.com/images/scripts/AccessPopup.tar.gz" -o AccessPopup.tar.gz && tar -xvf ./AccessPopup.tar.gz && cd AccessPopup && sudo ./installconfig.sh
```

:::success
可用選項：

1. 安裝 AccessPopup 腳本
1. 修改存取點的 SSID 或密碼
1. 修改存取點的 IP 位址
1. 即時切換：WiFi 網路 <-> 存取點
1. 設定新的 WiFi 網路或修改現有網路密碼
1. 修改主機名稱
1. 解除安裝 AccessPopup
1. 立即執行 AccessPopup（自動判斷使用 WiFi 或存取點）
:::

按下2可以修改AP名稱及密碼
3可以查看IP(預設為192.168.50.5)
4可以進行測試

```bash=
# 執行安裝腳本
sudo ./installconfig.sh
```


## 無法執行sudo: ./installconfig.sh: command not found
```bash=
# 在家目錄和下載目錄中尋找
find ~/ -name "AccessPopup" -type d
# 進入 AccessPopup 目錄
cd /home/ahb0222-2w/AccessPopup
```

# 參考資料
https://blog.gtwang.org/iot/raspberry-pi-time-lapse-using-motion-and-webcam/2/

🌟有多幾個需要的可以到蝦皮訂購[樹莓派1600萬像素的攝像頭模組](https://shopee.tw/product/4343295/21610481358/)


全文分享至

https://www.facebook.com/LHB0222/

https://www.instagram.com/ahb0222/

有疑問想討論的都歡迎於下方留言

喜歡的幫我分享給所有的朋友 \o/

有所錯誤歡迎指教

# [:page_with_curl: 全部文章列表](https://hackmd.io/@LHB-0222/AllWritings)



![](https://i.imgur.com/nHEcVmm.jpg)


