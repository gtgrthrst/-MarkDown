# 口罩辨識訓練機_Google Teachable Machine 與Raspberry Pi構建TensorFlow Lite應用

[![hackmd-github-sync-badge](https://hackmd.io/4-FA8PJMQ7SpOSivA38CIg/badge)](https://hackmd.io/4-FA8PJMQ7SpOSivA38CIg)


> [color=#40f1ef][name=LHB阿好伯, 2020/10/22][:earth_africa:](https://www.facebook.com/LHB0222/)
###### tags: `Python` `社群專案` `樹莓派`

[TOC]

最近剛好收到一個展示的邀約

要展示口罩辨識嘗試使用樹莓派搭配Google的Teachable Machine網站進行處理

以下文章主要是一些測試時所記錄的步驟

可能不是很完整若是有人發現問題歡迎提出來討論

## [Google Teachable Machine訓練模型](https://teachablemachine.withgoogle.com/)

首先使用[Teachable Machine網頁](https://teachablemachine.withgoogle.com/)利用電腦攝像機捕捉有戴口罩與沒戴口罩的畫面

### 訓練模型

以及一個沒有捕捉到人臉的數據庫

名稱請用英文避免後續出現不相容

![](https://i.imgur.com/raQcD5G.png)


![](https://i.imgur.com/4UNqtb5.png)

### 察看訓練模型結果

![](https://i.imgur.com/AhwVAMc.png)

### 下載模型

下載的檔案主要是要給樹莓派的TensorFlow Lite機器學習模型
![](https://i.imgur.com/4LDJAt3.png)

## 範例
訓練出來的結果就會向下方網址所展示的
大家可以上去試試看
[展示網址](https://teachablemachine.withgoogle.com/models/I__T4_I8h/)
![](https://i.imgur.com/ywTJYIF.png)


## 儲存模型
大家也可以選擇將專案存在Google Drive中

下次要修改或是增減都很方便
![](https://i.imgur.com/1ekNjcp.png)

[專案網址](https://teachablemachine.withgoogle.com/train/image/1njPCz1t_rPe6rZu9tWF_gsZAvJzrwPLu?network=true)

# 樹梅派設定

可以參考下方網站進行設定
> [參考網址_Using Raspberry Pi and TensorFlow Lite for Object Detection](
https://gpiocc.github.io/learn/raspberrypi/2020/04/18/martin-ku-using-raspberry-pi-and-tensorflow-lite-for-object-detection.html)

## 安裝模型框架Tensorflow Lite

安裝Tensorflow Lite模型框架


>[參考網址_Python quickstart](https://www.tensorflow.org/lite/guide/python)

>https://www.tensorflow.org/lite/guide/build_rpi

依照[Python quickstart](https://www.tensorflow.org/lite/guide/python)網站中列表選擇合適的Tensorflow Lite版本


```python=
pip3 install https://dl.google.com/coral/python/tflite_runtime-2.1.0.post1-cp37-cp37m-linux_armv7l.whl 
```
![](https://i.imgur.com/qFLJMes.png)




# 樹莓派接腳輸出

為了能夠將得到的判斷結果不只是顯示於螢幕上
我修改程式碼利用樹莓派進行簡單的GPIO輸出
後續偷懶使用Arduino進行其他元件控制
目前將接上兩個Servo作為柵欄示意與接上一個ws2812作為指示燈
![](https://i.imgur.com/ib34T2j.png)


## 樹莓派控制程式碼
修改官方提供的 [label_image.py](https://github.com/tensorflow/tensorflow/tree/master/tensorflow/lite/examples/python/) 為 label_image2.py
添加GPIO輸出程序
```python=
# python3
#
# Copyright 2019 The TensorFlow Authors. All Rights Reserved.
#
# Licensed under the Apache License, Version 2.0 (the "License");
# you may not use this file except in compliance with the License.
# You may obtain a copy of the License at
#
#     https://www.apache.org/licenses/LICENSE-2.0
#
# Unless required by applicable law or agreed to in writing, software
# distributed under the License is distributed on an "AS IS" BASIS,
# WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied.
# See the License for the specific language governing permissions and
# limitations under the License.
"""Example using TF Lite to classify objects with the Raspberry Pi camera."""

from __future__ import absolute_import
from __future__ import division
from __future__ import print_function

import argparse
import io
import time
import numpy as np
import picamera
import RPi.GPIO as GPIO   #GPIO套件
import time
pinLED1 = 26  #定義GPIO接腳
pinLED2 = 19
pinLED3 = 13

GPIO.setmode(GPIO.BCM) #定義GPIO接腳編號方式
GPIO.setup(pinLED1, GPIO.OUT)  #定義GPIO接腳模式
GPIO.setup(pinLED2, GPIO.OUT)
GPIO.setup(pinLED3, GPIO.OUT)

from PIL import Image
from tflite_runtime.interpreter import Interpreter


def load_labels(path):
  with open(path, 'r') as f:
    return {i: line.strip() for i, line in enumerate(f.readlines())}


def set_input_tensor(interpreter, image):
  tensor_index = interpreter.get_input_details()[0]['index']
  input_tensor = interpreter.tensor(tensor_index)()[0]
  input_tensor[:, :] = image


def classify_image(interpreter, image, top_k=1):
  """Returns a sorted array of classification results."""
  set_input_tensor(interpreter, image)
  interpreter.invoke()
  output_details = interpreter.get_output_details()[0]
  output = np.squeeze(interpreter.get_tensor(output_details['index']))

  # If the model is quantized (uint8 data), then dequantize the results
  if output_details['dtype'] == np.uint8:
    scale, zero_point = output_details['quantization']
    output = scale * (output - zero_point)

  ordered = np.argpartition(-output, top_k)
  return [(i, output[i]) for i in ordered[:top_k]]


def main():
  parser = argparse.ArgumentParser(
      formatter_class=argparse.ArgumentDefaultsHelpFormatter)
  parser.add_argument(
      '--model', help='File path of .tflite file.', required=True)
  parser.add_argument(
      '--labels', help='File path of labels file.', required=True)
  args = parser.parse_args()

  labels = load_labels(args.labels)

  interpreter = Interpreter(args.model)
  interpreter.allocate_tensors()
  _, height, width, _ = interpreter.get_input_details()[0]['shape']

  with picamera.PiCamera(resolution=(520, 400), framerate=30) as camera:
    camera.start_preview(fullscreen=False,window=(100,200,300,400))
    camera.preview.alpha = 300
    
    try:
      stream = io.BytesIO()
      for _ in camera.capture_continuous(
          stream, format='jpeg', use_video_port=True):
        stream.seek(0)
        image = Image.open(stream).convert('RGB').resize((width, height),
                                                         Image.ANTIALIAS)
        start_time = time.time()
        results = classify_image(interpreter, image)
        elapsed_ms = (time.time() - start_time) * 1000
        label_id, prob = results[0]
        stream.seek(0)
        stream.truncate()
        camera.annotate_text = '%s %.2f\n%.1fms' % (labels[label_id], prob,
                                                    elapsed_ms)
    # 簡單的IF判斷程序控制GPIO輸出
        if labels[label_id] == "0 YES":  
            GPIO.output(pinLED1, 0)
            GPIO.output(pinLED2, 0)
            GPIO.output(pinLED3, 0)
            GPIO.output(pinLED1, 1) #Right
            time.sleep(1)
            
        elif labels[label_id] == "1 NO":
            GPIO.output(pinLED1, 0)
            GPIO.output(pinLED2, 0)
            GPIO.output(pinLED3, 0)
            GPIO.output(pinLED2, 1)
            time.sleep(1)
            
        elif labels[label_id] == "2 NA":
            GPIO.output(pinLED1, 0)
            GPIO.output(pinLED2, 0)
            GPIO.output(pinLED3, 0)
            GPIO.output(pinLED3, 1) #Left
            time.sleep(1)
            
        
    finally:
      camera.stop_preview()


if __name__ == '__main__':
  main()

```

## 樹莓派與Arduino腳位記錄
| Raspberry Pi | <-----> | Arduino |
| :--------: | :--------: | :--------: |
| 13    |   <----->   | 12     |
|19 | <----->  | 3|
|26| <-----> |4|
|ws2812| <----->  | 5 |
|servoR| <----->  | 9 |
|servoL| <----->  | 10 |

後續有時間規劃以NRF24L01作為樹莓派與Arduino的連接橋梁

|NRF24L01| <-----> |樹莓派|
| :--------: | :--------: | :--------: |
|GND| <-----> |GND|
|VCC| <-----> |3.3V|
|CE| <-----> |GPIO25 即22管腳|
|CSN| <-----> |CE0(GPIO8) 即 24管腳|
|SCK| <-----> |SCLK(GPIO11)即23管腳|
|MOSI| <-----> |MOSI(GPIO10)即19管腳|
|MISO| <-----> |MISO(GPIO9)即21管腳|
|IRQ| <-----> |GPIO18即12管腳|

|NRF24L01| <-----> |Arduino|
| :--------: | :--------: | :--------: |
|VCC| <-----> |3.3V|
|GND| <-----> |GND|
|CE| <-----> |D9|
|CSN| <-----> |D10|
|MOSI|<-----> |D11|
|SCK| <-----> |D13|
|IRQ| <-----> |不接|



## arduino控制
```cpp=
#include <Adafruit_NeoPixel.h>
#include <Servo.h>
#ifdef __AVR__
#include <avr/power.h>
#endif
#define PIN 5  //ws2812接腳
#define NUMPIXELS 3  //ws2812 燈泡數
Servo servoR, servoL;

Adafruit_NeoPixel pixels(NUMPIXELS, PIN, NEO_GRB + NEO_KHZ800);
#define DELAYVAL 500

const  int IR1 = 2;
const  int IR2 = 3;
const  int IR3 = 4;
int IR1r = 1; 
int IR2r = 1;
int IR3r = 1;



void setup() {
servoL.attach(9,500,2400); 
servoR.attach(10,500,2400);
#if defined(__AVR_ATtiny85__) && (F_CPU == 16000000)
  clock_prescale_set(clock_div_1);
#endif

  pixels.begin();
  //pixels.Brightness(255);

  Serial.begin(9600);          //設定鮑率為9600
  pinMode(IR1, INPUT);  //設定Pin2為IR-1輸入接腳
  pinMode(IR2, INPUT);  //設定Pin3為IR-2輸入接腳
  pinMode(IR3, INPUT);  //設定Pin4為IR-3輸入接腳


}



void loop() {
   IR1r= digitalRead(IR1);//2
   IR2r= digitalRead(IR2);//3
   IR3r= digitalRead(IR3);//4

 if (IR1r == 1) {   //YES  綠線
    pixels.clear();
    pixels.setPixelColor(0, pixels.Color(0, 255, 0));
    pixels.setPixelColor(1, pixels.Color(0, 255, 0));
    pixels.setPixelColor(2, pixels.Color(0, 255, 0));
    pixels.show();
    servoR.write(90);
    servoL.write(90);
   delay(200);
  } else {
    if (IR2r == 1) {   //NO 黃線
    pixels.clear();
    pixels.setPixelColor(0, pixels.Color(255, 0, 0));
    pixels.setPixelColor(1, pixels.Color(255, 0, 0));
    pixels.setPixelColor(2, pixels.Color(255, 0, 0));
    pixels.show();
    servoR.write(0);
    servoL.write(180);
    delay(200);
  } else {
    if (IR3r == 1) {   //NA  橘線
    pixels.clear();
    pixels.setPixelColor(0, pixels.Color(255, 255, 255));
    pixels.setPixelColor(1, pixels.Color(255, 255, 255));
    pixels.setPixelColor(2, pixels.Color(255, 255, 255));
    pixels.show();
    servoR.write(0);
    servoL.write(180);
    delay(200);
  } 
  } 
  }
  }
```

# 執行分類程序

```cpp=
python3 /home/pi/examples/lite/examples/image_classification/raspberry_pi/classify_picamera2.py \
  --model /home/pi/ai/model.tflite \
  --labels /home/pi/ai/labels.txt
```
![](https://i.imgur.com/MxVrn6y.png)
![](https://i.imgur.com/rdVqaLl.png)

# ==以下是嘗試過的一些程式碼不知道有沒有甚麼用可忽略==

# 連接Arduino進行控制


```cpp=
#include <Servo.h>

//TODO: fix this url hinting
WebUSB WebUSBSerial(1 /* https:// */, "webusb.github.io/arduino/demos/rgb");

#define Serial WebUSBSerial
Servo myservo;

const int redPin = 9;
const int greenPin = 10;
const int bluePin = 11;
int pos = 0;    // variable to store the servo position

int color[0];
int colorIndex;

void setup() {
  pinMode(5, OUTPUT);
  pinMode(6, OUTPUT);
  pinMode(7, OUTPUT);
  colorIndex = 0;
  Serial.begin(9600);
  Serial.write("Sketch begins.\r\n");
  Serial.flush();
}

void loop() {
 color[0] = Serial.read();
      Serial.print(color[0]);
      if (color[0] == 0) {
        delay(2000);
        digitalWrite(5, HIGH);
        delay(1000);
        digitalWrite(5, LOW);
        delay(1000);
      }
      else if (color[0] == 1) {
        delay(2000);
        digitalWrite(6, HIGH);
        delay(1000);
        digitalWrite(6, LOW);
        delay(1000);
      }
   else if (color[0] == 2) {
        delay(2000);
        digitalWrite(6, HIGH);
        delay(1000);
        digitalWrite(6, LOW);
        delay(1000);
      }
  }
```




# 安裝OpenCV套件

sudo raspi-config

選擇"7 Advanced Options" à "A1  Expand filesystem "，重開機。

# 更新及升級所有套件包
sudo apt-get update && sudo apt-get upgrade

# 安裝開發者套件CMake 需要用來編譯
sudo apt-get install build-essential cmake pkg-config
 
# 安裝有關OpenCV的相依套件
```cpp=
sudo apt-get install libjpeg-dev libpng-dev libtiff-dev
sudo apt-get install libavcodec-dev libavformat-dev libswscale-dev libv4l-dev
sudo apt-get install libxvidcore-dev libx264-dev
sudo apt-get install libgtk-3-dev
sudo apt-get install libcanberra-gtk*
sudo apt-get install libatlas-base-dev gfortran
sudo apt-get install python3-dev
```

# 下載OpenCV4.0版至RPi4 。
cd ~
wget -O opencv.zip https://github.com/opencv/opencv/archive/4.0.0.zip
wget -O opencv_contrib.zip https://github.com/opencv/opencv_contrib/archive/4.0.0.zip
 

# 並解壓縮檔案
unzip opencv.zip
unzip opencv_contrib.zip
 

# 建立opencv 和 opencv_contrib資料夾及將檔案放置資料夾內
mv opencv-4.0.0 opencv
mv opencv_contrib-4.0.0 opencv_contrib
 

# 先在opencv資料夾內建立名為build的資料夾
 

cd ~/opencv
mkdir build
cd build
 

# 使用CMake來設置OpenCV 4環境(從這步驟開始是最花時間)
cmake -D CMAKE_BUILD_TYPE=RELEASE \
-D CMAKE_INSTALL_PREFIX=/usr/local \
-D OPENCV_EXTRA_MODULES_PATH=~/opencv_contrib/modules \
-D ENABLE_NEON=ON \
-D ENABLE_VFPV3=ON \
-D BUILD_TESTS=OFF \
-D OPENCV_ENABLE_NONFREE=ON \
-D INSTALL_PYTHON_EXAMPLES=OFF \
-D BUILD_EXAMPLES=OFF 
 

# 調整RPi4的SWAP交換空間，來解決編譯OpenCV記憶體不足的問題。
sudo nano /etc/dphys-swapfile

請把 CONF_SWAPSIZE=100改成 2048

# 重新開啟SWAP服務
sudo /etc/init.d/dphys-swapfile stop
sudo /etc/init.d/dphys-swapfile start

# 開啟四核心編譯OpenCV
make -j4

# 安裝OpenCV
sudo make install
sudo ldconfig






https://gpiocc.github.io/learn/raspberrypi/2020/04/18/martin-ku-using-raspberry-pi-and-tensorflow-lite-for-object-detection.html
```cpp=
cd ~/ai
python3 -m venv tfl
source tfl/bin/activate
pip install --upgrade pip
cd examples/lite/examples/object_detection/raspberry_pi
pip install -r requirements.txt


python detect_picamera.py \
  --model ~/ai/model.tflite \
  --labels ~/ai/labels.txt

 python detect_picamera.py \
  --model ~/ai/all_models/mobilenet_ssd_v2_coco_quant_postprocess.tflite \
  --labels ~/ai/all_models/coco_labels.txt
 
 
# https://gpiocc.github.io/learn/raspberrypi/2020/06/20/martin-ku-custom-tensorflow-image-classification-with-teachable-machine.html
git clone https://github.com/tensorflow/examples.git
cd examples/lite/examples/image_classification/raspberry_pi
pip install -r requirements.txt

  python classify_picamera.py   --model ~/ai/model.tflite   --labels ~/ai/labels.txt
```


