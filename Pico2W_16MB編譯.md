
當您成功地將 Pico 的快閃記憶體（Flash）升級到 16MB 後，
下一步就是透過軟體來駕馭這片新開拓的數位疆土。
官方提供的標準 MicroPython 韌體無法辨識超過預設大小的空間，
因此，我們必須親自動手編譯一份客製化的韌體。

本指南將以一個簡單、清晰的流程，詳細記錄如何為您的 Pico W 或 Pico 2 W 編譯一份能完全利用 16MB 空間的 MicroPython 韌體，並包含過程中常見的錯誤與解決方案。

步驟一：準備您的開發環境
一個穩定、配置正確的開發環境是成功的起點。我們將在 Linux (Ubuntu 24.04) 環境下進行操作。

1. 安裝必要工具
打開終端機，執行以下指令，一次性安裝所有編譯所需的軟體包：

# 首先，更新您的軟體包列表
sudo apt update

# 安裝 Git、CMake 及 ARM 交叉編譯工具鏈
sudo apt install git cmake gcc-arm-none-eabi libnewlib-arm-none-eabi build-essential

2. 下載 MicroPython 原始碼
使用 Git 從 GitHub 將最新的 MicroPython 專案程式碼複製到您的電腦。

git clone https://github.com/micropython/micropython.git
cd micropython

接下來的所有操作，都將在這個 micropython 資料夾內完成。

步驟二：修改韌體設定檔
這是整個流程最核心的環節。我們需要修改設定檔，告知編譯系統我們的新硬體規格是 16MB。Pico 的設定分為兩個層級：

MicroPython 層級：決定檔案系統的大小。

Pico SDK 層級：定義晶片實際溝通的物理 Flash 大小。

這兩個層級的設定必須匹配，否則編譯將會失敗。

針對 Pico W (RP2040) 的修改
對於 Pico W，我們需要手動修改兩個檔案。

檔案 1：MicroPython 層級設定
路徑： ports/rp2/boards/RPI_PICO_W/mpconfigboard.h

操作：將 MICROPY_HW_FLASH_STORAGE_BYTES 的值改為 (16 * 1024 * 1024)。

// 將 Flash 大小設定為 16MB
#define MICROPY_HW_FLASH_STORAGE_BYTES (16 * 1024 * 1024)

檔案 2：Pico SDK 底層設定
路徑： lib/pico-sdk/src/boards/include/boards/pico_w.h

操作：同樣地，將 PICO_FLASH_SIZE_BYTES 的值修改為 16MB。

// 將物理 Flash 大小定義為 16MB
#define PICO_FLASH_SIZE_BYTES (16 * 1024 * 1024)

針對 Pico 2 W (RP2350) 的修改 ✨
對於新一代的 Pico 2 W，其韌體設計更為智能，我們的修改流程也因此大幅簡化。

Pico 2 W 的 mpconfigboard.h 設定檔會自動讀取底層 SDK 的 Flash 大小，並減去系統保留空間。這意味著我們只需要修改一個檔案！

唯一需要的修改：Pico SDK 層級設定
路徑： lib/pico-sdk/src/boards/include/boards/pico2_w.h

操作：將 PICO_FLASH_SIZE_BYTES 的值從預設值（例如 4MB 或 8MB）修改為 16MB 即可。上層設定將會自動適應。

// 將物理 Flash 大小定義為 16MB
#define PICO_FLASH_SIZE_BYTES (16 * 1024 * 1024)

步驟三：編譯您的客製化韌體
修改完設定檔後，就可以開始編譯了。

進入 rp2 埠口目錄

cd ports/rp2

準備子模組 (Submodules)
這個步驟會下載板子所需的所有依賴庫，例如 Wi-Fi 驅動。請務必帶上 BOARD= 參數。

# 以 Pico W 為例
make BOARD=RPI_PICO_W submodules

# 若是 Pico 2 W，則使用
# make BOARD=RPI_PICO2_W submodules

開始編譯
執行最終的編譯指令。這個過程會持續數分鐘。

# 以 Pico W 為例
make BOARD=RPI_PICO_W

# 若是 Pico 2 W，則使用
# make BOARD=RPI_PICO2_W

常見編譯錯誤與解決方案
cmake: not found: 代表您未安裝 cmake 工具。請返回步驟一，執行 apt install 指令。

cyw43-driver not initialized: 代表您未下載板子專用的子模組。請執行帶有 BOARD= 參數的 make submodules 指令。

static assertion failed: 代表上層與底層的 Flash 大小設定不一致。請仔細檢查步驟二中的兩個檔案是否都已正確修改。

當您在終端機的結尾看到 [100%] Built target firmware 時，代表編譯成功！
![image](https://hackmd.io/_uploads/HyqbJc5Qle.png)

客製化韌體 firmware.uf2 已經在 build-RPI_PICO_W/ (或 build-RPI_PICO2_W/) 資料夾中產生。
![image](https://hackmd.io/_uploads/HypN155Xel.png)


步驟四：燒錄並驗證
進入 BOOTSEL 模式：將 Pico 拔除，按住 BOOTSEL 按鈕不放，再重新插入電腦。

使用 Thonny IDE 燒錄：打開 Thonny，選擇 執行 > 設定直譯器... > 安裝或更新韌體。點擊 瀏覽... 並選取您剛剛編譯好的 firmware.uf2 檔案進行安裝。

驗證空間大小：安裝完成後，在 Thonny 的 Shell 中執行以下程式碼：

```
import os
fs_stat = os.statvfs('/')
fs_size = fs_stat[0] * fs_stat[2]
fs_size_mb = fs_size / (1024 * 1024)
print(f"檔案系統總容量: {fs_size_mb:.2f} MB")
```

看到接近 15MB 的輸出結果，就代表您已成功解鎖了 Pico 的全部潛力！

https://github.com/gtgrthrst/OpenData/blob/main/build-RPI_PICO2_W_16MB.uf2
https://github.com/gtgrthrst/OpenData/blob/main/build-RPI_PICO_W_16MB.uf2

