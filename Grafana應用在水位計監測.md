# 安裝 Grafana
1. 更新套件列表並安裝必要的相依套件

首先，打開終端機並執行以下命令來更新您的套件列表，並安裝一些必要的工具：
```Bash
sudo apt update
sudo apt install -y apt-transport-https software-properties-common wget 
```

apt-transport-https：允許 apt 透過 HTTPS 傳輸套件。
software-properties-common：提供管理套件庫的工具。
wget：用於從網路上下載檔案。
gpg：用於處理 GPG 金鑰。

2. 匯入 Grafana GPG 金鑰

為了確保下載的套件是官方且未被竄改的，您需要匯入 Grafana 的 GPG 金鑰。


```Bash
sudo mkdir -p /etc/apt/keyrings/
wget -q -O - https://apt.grafana.com/gpg.key | gpg --dearmor | sudo tee /etc/apt/keyrings/grafana.gpg > /dev/null
```


建立 /etc/apt/keyrings/ 目錄 (如果它不存在的話)。
使用 wget 下載 Grafana 的 GPG 金鑰。
使用 gpg --dearmor 將金鑰從 ASCII armor 格式轉換為 apt 所需的二進位格式。
將轉換後的金鑰儲存到 /etc/apt/keyrings/grafana.gpg。

3. 新增 Grafana APT 套件庫

接下來，將 Grafana 的套件庫新增到您的系統中。這會告訴 apt 從哪裡下載 Grafana 套件。


```Bash=
echo "deb [signed-by=/etc/apt/keyrings/grafana.gpg] https://packages.grafana.com/oss/deb stable main" | sudo tee /etc/apt/sources.list.d/grafana.list
```
這會建立一個新的套件庫設定檔 /etc/apt/sources.list.d/grafana.list，內容指向 Grafana 的開源版本 (oss) 的穩定 (stable) 套件庫。

4. 安裝 Grafana

現在更新 apt 的套件列表，並安裝 Grafana：


```Bash
sudo apt update
sudo apt install grafana
```
這會下載並安裝最新穩定版的 Grafana OSS。

5. 啟動並設定 Grafana 伺服器開機啟動

安裝完成後，您需要啟動 Grafana 伺服器，並設定它在系統開機時自動啟動：


```Bash
sudo systemctl daemon-reload
sudo systemctl start grafana-server
sudo systemctl enable grafana-server.service
```
6. 驗證安裝並存取 Grafana

您可以檢查 Grafana 伺服器的狀態：


```Bash

sudo systemctl status grafana-server
```
如果一切正常，您應該會看到 active (running) 的狀態。

Grafana 預設會在 3000 連接埠上執行。打開您的網頁瀏覽器，然後前往：
http://<您的伺服器IP位址>:3000
例如，如果您的伺服器 IP 位址是 192.168.192.241，則輸入 http://192.168.192.241:3000。
如果是本機安裝，可以使用 http://localhost:3000。

預設的登入帳號和密碼是：

使用者名稱：admin
密碼：admin
首次登入後，系統會要求您更改預設密碼。

# 抓取資料
```bash=
#!/bin/bash

# --- 請根據您的情況修改以下設定 ---
REMOTE_USER="admin"
REMOTE_HOST="192.168.192.241"
REMOTE_PATH="/ext/modbus_db" # 遠端伺服器上要下載的資料路徑 (假設 /ext/modbus_db 是個目錄)
LOCAL_BASE_DIR="$HOME/SFTP/RUT906-1/" # 本地儲存下載資料的基礎目錄

# 日誌檔案路徑 (可選，但建議保留，用於記錄下載狀態)
LOG_FILE="$HOME/rut906_download_log.txt" 
# --- 設定結束 ---

# 確保本地目標目錄存在
mkdir -p "$LOCAL_BASE_DIR"

# 記錄開始時間到日誌檔案
echo "----------------------------------------" >> "$LOG_FILE"
echo "開始下載 @ $(date '+%Y-%m-%d %H:%M:%S')" >> "$LOG_FILE"

# 使用 scp 進行下載。
# 因為我們假設遠端的 REMOTE_PATH (/ext/modbus_db) 是一個目錄，所以使用 -r (遞迴) 選項。
# 如果 REMOTE_PATH 其實是一個單一檔案，請從下面的 scp 指令中移除 -r 選項。
# -o ConnectTimeout=30: 設定連線超時30秒
# -o ConnectionAttempts=3: 設定在失敗前嘗試連線3次
# 2>&1: 將標準錯誤輸出也重定向到日誌檔案
scp -r -o ConnectTimeout=30 -o ConnectionAttempts=3 "${REMOTE_USER}@${REMOTE_HOST}:${REMOTE_PATH}" "$LOCAL_BASE_DIR" >> "$LOG_FILE" 2>&1

# 檢查 scp 指令的結束狀態碼
SCP_EXIT_CODE=$? # 獲取上一個指令 (scp) 的結束狀態碼

if [ $SCP_EXIT_CODE -eq 0 ]; then
  echo "下載成功 @ $(date '+%Y-%m-%d %H:%M:%S')" >> "$LOG_FILE"
else
  echo "下載失敗，SCP 結束狀態碼: $SCP_EXIT_CODE @ $(date '+%Y-%m-%d %H:%M:%S')" >> "$LOG_FILE"
  echo "詳細錯誤請查看日誌檔案上方 scp 的輸出。" >> "$LOG_FILE"
fi
echo "下載操作完成。請檢查 '$LOCAL_BASE_DIR' 以及日誌檔案 '$LOG_FILE'" >> "$LOG_FILE"
echo "----------------------------------------" >> "$LOG_FILE"
echo "" >> "$LOG_FILE" # 添加空行以便分隔日誌條目

exit $SCP_EXIT_CODE # 腳本以 scp 的結束狀態碼退出
```
## 查詢溫度Temperature

```sql=
SELECT
  time,
  level,
  AVG(level) OVER (ORDER BY time ASC ROWS BETWEEN 29 PRECEDING AND CURRENT ROW) AS level_rolling_avg_30min
FROM (
  SELECT
    time,
    CAST(SUBSTR(response_data, 2, LENGTH(response_data) - 2) AS REAL) AS level
  FROM
    modbus_data
  WHERE
    server_name = 'Level_TROLL_500' AND request_name = 'Temperature'
  ORDER BY
    time
)
```

### 最新一筆溫度
```sql=
SELECT
  CAST(SUBSTR(response_data, 2, LENGTH(response_data) - 2) AS REAL) AS current_temperature
FROM
  modbus_data
WHERE
  server_name = 'Level_TROLL_500' AND request_name = 'Temperature'
ORDER BY
  time DESC
LIMIT 1
```

## 查詢水位

```sql=
SELECT
  time,
  level,
  AVG(level) OVER (ORDER BY time ASC ROWS BETWEEN 29 PRECEDING AND CURRENT ROW) AS level_rolling_avg_30min
FROM (
  SELECT
    time,
    CAST(SUBSTR(response_data, 2, LENGTH(response_data) - 2) AS REAL) AS level
  FROM
    modbus_data
  WHERE
    server_name = 'Level_TROLL_500' AND request_name = 'TROLL_Level'
  ORDER BY
    time
)
```

### 最新一筆水位
```sql=
SELECT
  CAST(SUBSTR(response_data, 2, LENGTH(response_data) - 2) AS REAL) AS current_temperature
FROM
  modbus_data
WHERE
  server_name = 'Level_TROLL_500' AND request_name = 'Temperature'
ORDER BY
  time DESC
LIMIT 1
```

## 地圖合併查詢

```sql=
SELECT
  server_name,
  level_rolling_avg_30min AS current_temperature_avg
FROM (
  SELECT
    server_name,
    time,
    CAST(SUBSTR(response_data, 2, LENGTH(response_data) - 2) AS REAL) AS level,
    AVG(CAST(SUBSTR(response_data, 2, LENGTH(response_data) - 2) AS REAL)) OVER (PARTITION BY server_name ORDER BY time ASC ROWS BETWEEN 29 PRECEDING AND CURRENT ROW) AS level_rolling_avg_30min
  FROM
    modbus_data
  WHERE
    request_name = 'Temperature'
  ORDER BY
    time DESC
)
GROUP BY
  server_name
LIMIT 1
```

```sql=
SELECT
  server_name,
  level_rolling_avg_30min AS current_level_avg
FROM (
  SELECT
    server_name,
    time,
    CAST(SUBSTR(response_data, 2, LENGTH(response_data) - 2) AS REAL) AS level,
    AVG(CAST(SUBSTR(response_data, 2, LENGTH(response_data) - 2) AS REAL)) OVER (PARTITION BY server_name ORDER BY time ASC ROWS BETWEEN 29 PRECEDING AND CURRENT ROW) AS level_rolling_avg_30min
  FROM
    modbus_data
  WHERE
    request_name = 'TROLL_Level'
  ORDER BY
    time DESC
)
GROUP BY
  server_name
LIMIT 1
```