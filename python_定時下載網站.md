---
title: python自動下載器
tags: [Python]

---

1. 建立專案資料夾並進入
```
mkdir streamlit_downloader_uv
cd streamlit_downloader_uv
```
2. 建立虛擬環境：
```
uv venv
```
3. 啟動虛擬環境：
- Windows (CMD)
```
.venv\Scripts\activate
```
- macOS / Linux
```
source .venv/bin/activate
```
4. 安裝套件：
```
uv pip install nicegui apscheduler pytz requests
```
5. 建立 requirements.txt 檔案 (推薦的最佳實踐)：
為了方便他人或未來的自己重現環境，可以將當前環境中的套件清單凍結到一個 requirements.txt 檔案中。
```
uv pip freeze > requirements.txt
```
6. 執行封裝指令
- 執行 Bundle 模式 (預設模式)
    - 這個模式會將 Python 直譯器和所有相依套件全部打包進去，產生的檔案最大，但相容性最好，可以在沒有安裝 Python 的相同作業系統上直接執行。
```
pyfuze . --entry app.py --reqs requirements.txt --output-name downloader-bundle.com --win-gui
```
- 執行 Online 模式
    - 這個模式產生的檔案小得多，並且可以跨平台執行。但它在第一次執行時需要網路連線，以下載 Python 直譯器和所有相依套件。
```
pyfuze . --mode online --entry app.py --reqs requirements.txt --output-name downloader-online.com --win-gui
```

```python=
import os
import pytz
import requests
import asyncio
from datetime import datetime
from typing import List, Dict, Any

from apscheduler.schedulers.background import BackgroundScheduler
from apscheduler.triggers.interval import IntervalTrigger
from nicegui import app, ui, context

# --- 全域設定與狀態管理 ---

TAIPEI_TZ = pytz.timezone("Asia/Taipei")
scheduler = BackgroundScheduler(timezone=TAIPEI_TZ)
# tasks 列表現在只作為當前連接客戶端的 UI 元件快取，用於暫停/刪除操作
tasks: List[Dict[str, Any]] = [] 
main_loop: asyncio.AbstractEventLoop = None

# --- 核心功能函式 ---

def download_csv_file(task_id: str):
    """
    由排程器呼叫的下載函式。
    這個函式在背景執行緒中運行，因此需要從持久化儲存中讀取資料。
    """
    all_stored_tasks = app.storage.general.get('download_tasks', {})
    stored_task_info = all_stored_tasks.get(task_id)
    
    if not stored_task_info:
        print(f"[{datetime.now()}] 在儲存中找不到任務 ID: {task_id} 的資訊，下載中止。")
        if scheduler.get_job(task_id):
            scheduler.remove_job(task_id)
        return

    url = stored_task_info['url']
    save_path = stored_task_info['save_path']
    filename_prefix = stored_task_info['prefix']
    
    try:
        print(f"[{datetime.now()}] 開始執行下載任務: {url}")
        os.makedirs(save_path, exist_ok=True)
        headers = {'User-Agent': 'Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/58.0.3029.110 Safari/537.3'}
        response = requests.get(url, headers=headers, stream=True)
        response.raise_for_status()
        
        timestamp = datetime.now().strftime("%Y%m%d_%H%M%S")
        filename = f"{filename_prefix}_{timestamp}.csv"
        full_path = os.path.join(save_path, filename)

        with open(full_path, 'wb') as f:
            for chunk in response.iter_content(chunk_size=8192):
                f.write(chunk)
        
        print(f"[{datetime.now()}] 檔案成功儲存至: {full_path}")
        
        # 更新持久化儲存中的成功次數
        stored_task_info['success_count'] += 1
        app.storage.general['download_tasks'][task_id] = stored_task_info

        # 如果有 UI 連接著，則安全地更新 UI
        task_info = next((task for task in tasks if task['id'] == task_id), None)
        if main_loop and task_info:
            def update_ui_on_success():
                task_info['count_label'].set_text(f"成功下載次數: {stored_task_info['success_count']}")
                ui.notify(f"檔案成功儲存: {filename}", color='positive')
            main_loop.call_soon_threadsafe(update_ui_on_success)

    except Exception as e:
        error_message = f"任務 '{filename_prefix}' 執行失敗: {e}"
        print(f"[{datetime.now()}] {error_message}")
        if main_loop:
            main_loop.call_soon_threadsafe(
                lambda: ui.notify(error_message, color='negative', multi_line=True, close_button=True)
            )

# --- UI 更新與互動函式 ---

def create_task_ui(task_id: str, url: str, save_path: str, prefix: str, interval: int, success_count: int):
    """根據任務資料建立 UI 卡片，並將其加入全域 tasks 列表。"""
    
    with tasks_container:
        card = ui.card().classes('w-full')
        with card:
            with ui.row().classes('w-full items-center no-wrap'):
                with ui.column().classes('flex-grow'):
                    ui.label(f"任務: {prefix}").classes('text-h6')
                    ui.label(f"路徑: {save_path}").classes('text-caption text-gray-500')
                    ui.label(f"URL: {url}").classes('text-caption text-gray-500').style('word-break: break-all;')
                    ui.label(f"間隔: {interval} 分鐘").classes('text-caption')
                    count_label = ui.label(f"成功下載次數: {success_count}").classes('text-caption')

                with ui.column().classes('items-center min-w-[120px]'):
                    ui.label("下次執行倒數").classes('text-caption')
                    countdown_label = ui.label('--:--:--').classes('text-h5 text-positive')

                with ui.column().classes('gap-y-1'):
                    pause_button = ui.button('暫停', on_click=lambda _, t_id=task_id: toggle_pause_task(t_id))
                    ui.button('刪除', on_click=lambda _, t_id=task_id: delete_task(t_id), color='negative')
    
    # 檢查任務初始是否為暫停狀態
    job = scheduler.get_job(task_id)
    if job and not job.next_run_time:
        countdown_label.set_text('已暫停').classes(replace='text-h5 text-warning')
        pause_button.set_text('恢復')

    # 為這張卡片建立一個獨立的計時器
    def update_this_countdown():
        # 這個函式會捕捉到 countdown_label 和 task_id
        job = scheduler.get_job(task_id)
        # 只有在任務不是暫停狀態時才更新倒數
        if pause_button.text == '暫停':
            if job and job.next_run_time:
                now = datetime.now(TAIPEI_TZ)
                remaining = job.next_run_time - now
                if remaining.total_seconds() < 0:
                    countdown_label.set_text("執行中...")
                else:
                    days = remaining.days
                    hours, rem = divmod(remaining.seconds, 3600)
                    minutes, seconds = divmod(rem, 60)
                    if days > 0:
                        remaining_str = f"{days}天{hours:02d}:{minutes:02d}"
                    else:
                        remaining_str = f"{hours:02d}:{minutes:02d}:{seconds:02d}"
                    countdown_label.set_text(remaining_str)
            elif not job:
                countdown_label.set_text("已刪除")

    ui.timer(1.0, update_this_countdown)

    # 將 UI 元件加入到 tasks 列表，以便暫停/刪除時可以找到它們
    task_data = {
        'id': task_id,
        'card': card,
        'countdown_label': countdown_label,
        'pause_button': pause_button,
        'count_label': count_label,
    }
    tasks.append(task_data)


def add_task():
    """新增一個下載任務，並動態建立對應的 UI 卡片，同時儲存到 app.storage。"""
    path_val = save_path_input.value
    if not os.path.isdir(path_val):
        try:
            os.makedirs(path_val, exist_ok=True)
            ui.notify(f"路徑 '{path_val}' 不存在，已自動建立。", color='info')
        except Exception as e:
            ui.notify(f"路徑錯誤或無法建立: '{path_val}'. 錯誤: {e}", color='negative')
            return

    job_id = f"job_{datetime.now().timestamp()}"
    
    scheduler.add_job(
        download_csv_file,
        trigger=IntervalTrigger(minutes=interval_input.value),
        args=[job_id],
        id=job_id
    )

    stored_task_data = {
        'url': url_input.value, 'save_path': save_path_input.value,
        'prefix': prefix_input.value, 'interval': interval_input.value,
        'success_count': 0,
    }
    app.storage.general.setdefault('download_tasks', {})[job_id] = stored_task_data

    create_task_ui(task_id=job_id, **stored_task_data)
    
    ui.notify(f"任務 '{prefix_input.value}' 已成功新增！", color='positive')
    url_input.value = ''
    prefix_input.value = ''

def delete_task(task_id: str):
    """刪除一個任務，包含排程器、UI 卡片和儲存的資料。"""
    if scheduler.get_job(task_id):
        scheduler.remove_job(task_id)
    
    task_to_delete = next((task for task in tasks if task['id'] == task_id), None)
    if task_to_delete:
        task_to_delete['card'].clear()
        task_to_delete['card'].delete()
        tasks.remove(task_to_delete)
        
    if 'download_tasks' in app.storage.general and task_id in app.storage.general['download_tasks']:
        del app.storage.general['download_tasks'][task_id]
        
    ui.notify("任務已刪除", color='warning')

def toggle_pause_task(task_id: str):
    job = scheduler.get_job(task_id)
    task_info = next((task for task in tasks if task['id'] == task_id), None)
    if not job or not task_info:
        return

    if job.next_run_time:
        scheduler.pause_job(task_id)
        task_info['countdown_label'].set_text('已暫停').classes(replace='text-h5 text-warning')
        task_info['pause_button'].set_text('恢復')
        ui.notify("任務已暫停", color='info')
    else:
        scheduler.resume_job(task_id)
        task_info['countdown_label'].set_text('--:--:--').classes(replace='text-h5 text-positive')
        task_info['pause_button'].set_text('暫停')
        ui.notify("任務已恢復", color='info')


# --- UI 頁面定義 ---
@ui.page('/')
def main_page():
    global url_input, save_path_input, prefix_input, interval_input, tasks_container
    
    tasks.clear()

    with ui.column().classes('w-full items-center'):
        ui.label('📊 CSV 週期性下載器 (持久化版)').classes('text-h4 text-center my-4')

        with ui.card().classes('w-full max-w-2xl'):
            ui.label('新增下載任務').classes('text-h6')
            url_input = ui.input(
                'CSV 網址 (URL)',
                value='https://data.taipei/api/dataset/190796c8-b992-468e-895c-2fe31c0cf186/resource/85038b74-3344-4365-94f3-a36a43e414e9/download'
            ).props('outlined dense').classes('w-full')
            default_path = 'D:\\downloads' if os.name == 'nt' else os.path.expanduser('~/nicegui_downloads')
            save_path_input = ui.input('本機保存路徑', value=default_path).props('outlined dense').classes('w-full')
            with ui.row().classes('w-full'):
                prefix_input = ui.input('檔名抬頭', value='ubike').props('outlined dense')
                interval_input = ui.number('下載間隔(分)', value=5, min=1, step=1).props('outlined dense')
            ui.button('加入排程', on_click=add_task).props('color=primary')

        ui.label('目前的排程任務').classes('text-h5 text-center my-4')
        
        tasks_container = ui.column().classes('w-full max-w-3xl gap-y-4')

        stored_tasks = app.storage.general.get('download_tasks', {})
        for task_id, task_data in list(stored_tasks.items()):
            if scheduler.get_job(task_id):
                create_task_ui(task_id=task_id, **task_data)
            else:
                print(f"警告：在儲存中找到任務 {task_id}，但在排程器中不存在。")

# --- 應用程式啟動與關閉事件 ---
def on_startup():
    """應用啟動時執行的函式，恢復排程任務並啟動排程器。"""
    global main_loop
    main_loop = asyncio.get_running_loop()
    
    print("正在從儲存中恢復任務...")
    stored_tasks = app.storage.general.get('download_tasks', {})
    for task_id, task_data in stored_tasks.items():
        if not scheduler.get_job(task_id):
            print(f"  - 正在恢復任務: {task_id} (間隔: {task_data['interval']} 分鐘)")
            scheduler.add_job(
                download_csv_file,
                trigger=IntervalTrigger(minutes=task_data['interval']),
                args=[task_id],
                id=task_id,
                misfire_grace_time=60
            )

    if not scheduler.running:
        scheduler.start(paused=False)
        print("排程器已啟動。")

def on_shutdown():
    print("應用程式準備關閉，正在關閉排程器...")
    if scheduler.running:
        scheduler.shutdown()
        print("排程器已關閉。")

app.on_startup(on_startup)
app.on_shutdown(on_shutdown)

# --- 執行 APP ---
ui.run(title='CSV 下載器', dark=True, reload=False, port=8080, storage_secret='THIS_IS_A_VERY_SECRET_KEY')

```1. 建立專案資料夾並進入
```
mkdir streamlit_downloader_uv
cd streamlit_downloader_uv
```
2. 建立虛擬環境：
```
uv venv
```
3. 啟動虛擬環境：
- Windows (CMD)
```
.venv\Scripts\activate
```
- macOS / Linux
```
source .venv/bin/activate
```
4. 安裝套件：
```
uv pip install nicegui apscheduler pytz requests
```
5. 建立 requirements.txt 檔案 (推薦的最佳實踐)：
為了方便他人或未來的自己重現環境，可以將當前環境中的套件清單凍結到一個 requirements.txt 檔案中。
```
uv pip freeze > requirements.txt
```
6. 執行封裝指令
- 執行 Bundle 模式 (預設模式)
    - 這個模式會將 Python 直譯器和所有相依套件全部打包進去，產生的檔案最大，但相容性最好，可以在沒有安裝 Python 的相同作業系統上直接執行。
```
pyfuze . --entry app.py --reqs requirements.txt --output-name downloader-bundle.com --win-gui
```
- 執行 Online 模式
    - 這個模式產生的檔案小得多，並且可以跨平台執行。但它在第一次執行時需要網路連線，以下載 Python 直譯器和所有相依套件。
```
pyfuze . --mode online --entry app.py --reqs requirements.txt --output-name downloader-online.com --win-gui
```

```python=
import os
import pytz
import requests
import asyncio
from datetime import datetime
from typing import List, Dict, Any

from apscheduler.schedulers.background import BackgroundScheduler
from apscheduler.triggers.interval import IntervalTrigger
from nicegui import app, ui, context

# --- 全域設定與狀態管理 ---

TAIPEI_TZ = pytz.timezone("Asia/Taipei")
scheduler = BackgroundScheduler(timezone=TAIPEI_TZ)
# tasks 列表現在只作為當前連接客戶端的 UI 元件快取，用於暫停/刪除操作
tasks: List[Dict[str, Any]] = [] 
main_loop: asyncio.AbstractEventLoop = None

# --- 核心功能函式 ---

def download_csv_file(task_id: str):
    """
    由排程器呼叫的下載函式。
    這個函式在背景執行緒中運行，因此需要從持久化儲存中讀取資料。
    """
    all_stored_tasks = app.storage.general.get('download_tasks', {})
    stored_task_info = all_stored_tasks.get(task_id)
    
    if not stored_task_info:
        print(f"[{datetime.now()}] 在儲存中找不到任務 ID: {task_id} 的資訊，下載中止。")
        if scheduler.get_job(task_id):
            scheduler.remove_job(task_id)
        return

    url = stored_task_info['url']
    save_path = stored_task_info['save_path']
    filename_prefix = stored_task_info['prefix']
    
    try:
        print(f"[{datetime.now()}] 開始執行下載任務: {url}")
        os.makedirs(save_path, exist_ok=True)
        headers = {'User-Agent': 'Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/58.0.3029.110 Safari/537.3'}
        response = requests.get(url, headers=headers, stream=True)
        response.raise_for_status()
        
        timestamp = datetime.now().strftime("%Y%m%d_%H%M%S")
        filename = f"{filename_prefix}_{timestamp}.csv"
        full_path = os.path.join(save_path, filename)

        with open(full_path, 'wb') as f:
            for chunk in response.iter_content(chunk_size=8192):
                f.write(chunk)
        
        print(f"[{datetime.now()}] 檔案成功儲存至: {full_path}")
        
        # 更新持久化儲存中的成功次數
        stored_task_info['success_count'] += 1
        app.storage.general['download_tasks'][task_id] = stored_task_info

        # 如果有 UI 連接著，則安全地更新 UI
        task_info = next((task for task in tasks if task['id'] == task_id), None)
        if main_loop and task_info:
            def update_ui_on_success():
                task_info['count_label'].set_text(f"成功下載次數: {stored_task_info['success_count']}")
                ui.notify(f"檔案成功儲存: {filename}", color='positive')
            main_loop.call_soon_threadsafe(update_ui_on_success)

    except Exception as e:
        error_message = f"任務 '{filename_prefix}' 執行失敗: {e}"
        print(f"[{datetime.now()}] {error_message}")
        if main_loop:
            main_loop.call_soon_threadsafe(
                lambda: ui.notify(error_message, color='negative', multi_line=True, close_button=True)
            )

# --- UI 更新與互動函式 ---

def create_task_ui(task_id: str, url: str, save_path: str, prefix: str, interval: int, success_count: int):
    """根據任務資料建立 UI 卡片，並將其加入全域 tasks 列表。"""
    
    with tasks_container:
        card = ui.card().classes('w-full')
        with card:
            with ui.row().classes('w-full items-center no-wrap'):
                with ui.column().classes('flex-grow'):
                    ui.label(f"任務: {prefix}").classes('text-h6')
                    ui.label(f"路徑: {save_path}").classes('text-caption text-gray-500')
                    ui.label(f"URL: {url}").classes('text-caption text-gray-500').style('word-break: break-all;')
                    ui.label(f"間隔: {interval} 分鐘").classes('text-caption')
                    count_label = ui.label(f"成功下載次數: {success_count}").classes('text-caption')

                with ui.column().classes('items-center min-w-[120px]'):
                    ui.label("下次執行倒數").classes('text-caption')
                    countdown_label = ui.label('--:--:--').classes('text-h5 text-positive')

                with ui.column().classes('gap-y-1'):
                    pause_button = ui.button('暫停', on_click=lambda _, t_id=task_id: toggle_pause_task(t_id))
                    ui.button('刪除', on_click=lambda _, t_id=task_id: delete_task(t_id), color='negative')
    
    # 檢查任務初始是否為暫停狀態
    job = scheduler.get_job(task_id)
    if job and not job.next_run_time:
        countdown_label.set_text('已暫停').classes(replace='text-h5 text-warning')
        pause_button.set_text('恢復')

    # 為這張卡片建立一個獨立的計時器
    def update_this_countdown():
        # 這個函式會捕捉到 countdown_label 和 task_id
        job = scheduler.get_job(task_id)
        # 只有在任務不是暫停狀態時才更新倒數
        if pause_button.text == '暫停':
            if job and job.next_run_time:
                now = datetime.now(TAIPEI_TZ)
                remaining = job.next_run_time - now
                if remaining.total_seconds() < 0:
                    countdown_label.set_text("執行中...")
                else:
                    days = remaining.days
                    hours, rem = divmod(remaining.seconds, 3600)
                    minutes, seconds = divmod(rem, 60)
                    if days > 0:
                        remaining_str = f"{days}天{hours:02d}:{minutes:02d}"
                    else:
                        remaining_str = f"{hours:02d}:{minutes:02d}:{seconds:02d}"
                    countdown_label.set_text(remaining_str)
            elif not job:
                countdown_label.set_text("已刪除")

    ui.timer(1.0, update_this_countdown)

    # 將 UI 元件加入到 tasks 列表，以便暫停/刪除時可以找到它們
    task_data = {
        'id': task_id,
        'card': card,
        'countdown_label': countdown_label,
        'pause_button': pause_button,
        'count_label': count_label,
    }
    tasks.append(task_data)


def add_task():
    """新增一個下載任務，並動態建立對應的 UI 卡片，同時儲存到 app.storage。"""
    path_val = save_path_input.value
    if not os.path.isdir(path_val):
        try:
            os.makedirs(path_val, exist_ok=True)
            ui.notify(f"路徑 '{path_val}' 不存在，已自動建立。", color='info')
        except Exception as e:
            ui.notify(f"路徑錯誤或無法建立: '{path_val}'. 錯誤: {e}", color='negative')
            return

    job_id = f"job_{datetime.now().timestamp()}"
    
    scheduler.add_job(
        download_csv_file,
        trigger=IntervalTrigger(minutes=interval_input.value),
        args=[job_id],
        id=job_id
    )

    stored_task_data = {
        'url': url_input.value, 'save_path': save_path_input.value,
        'prefix': prefix_input.value, 'interval': interval_input.value,
        'success_count': 0,
    }
    app.storage.general.setdefault('download_tasks', {})[job_id] = stored_task_data

    create_task_ui(task_id=job_id, **stored_task_data)
    
    ui.notify(f"任務 '{prefix_input.value}' 已成功新增！", color='positive')
    url_input.value = ''
    prefix_input.value = ''

def delete_task(task_id: str):
    """刪除一個任務，包含排程器、UI 卡片和儲存的資料。"""
    if scheduler.get_job(task_id):
        scheduler.remove_job(task_id)
    
    task_to_delete = next((task for task in tasks if task['id'] == task_id), None)
    if task_to_delete:
        task_to_delete['card'].clear()
        task_to_delete['card'].delete()
        tasks.remove(task_to_delete)
        
    if 'download_tasks' in app.storage.general and task_id in app.storage.general['download_tasks']:
        del app.storage.general['download_tasks'][task_id]
        
    ui.notify("任務已刪除", color='warning')

def toggle_pause_task(task_id: str):
    job = scheduler.get_job(task_id)
    task_info = next((task for task in tasks if task['id'] == task_id), None)
    if not job or not task_info:
        return

    if job.next_run_time:
        scheduler.pause_job(task_id)
        task_info['countdown_label'].set_text('已暫停').classes(replace='text-h5 text-warning')
        task_info['pause_button'].set_text('恢復')
        ui.notify("任務已暫停", color='info')
    else:
        scheduler.resume_job(task_id)
        task_info['countdown_label'].set_text('--:--:--').classes(replace='text-h5 text-positive')
        task_info['pause_button'].set_text('暫停')
        ui.notify("任務已恢復", color='info')


# --- UI 頁面定義 ---
@ui.page('/')
def main_page():
    global url_input, save_path_input, prefix_input, interval_input, tasks_container
    
    tasks.clear()

    with ui.column().classes('w-full items-center'):
        ui.label('📊 CSV 週期性下載器 (持久化版)').classes('text-h4 text-center my-4')

        with ui.card().classes('w-full max-w-2xl'):
            ui.label('新增下載任務').classes('text-h6')
            url_input = ui.input(
                'CSV 網址 (URL)',
                value='https://data.taipei/api/dataset/190796c8-b992-468e-895c-2fe31c0cf186/resource/85038b74-3344-4365-94f3-a36a43e414e9/download'
            ).props('outlined dense').classes('w-full')
            default_path = 'D:\\downloads' if os.name == 'nt' else os.path.expanduser('~/nicegui_downloads')
            save_path_input = ui.input('本機保存路徑', value=default_path).props('outlined dense').classes('w-full')
            with ui.row().classes('w-full'):
                prefix_input = ui.input('檔名抬頭', value='ubike').props('outlined dense')
                interval_input = ui.number('下載間隔(分)', value=5, min=1, step=1).props('outlined dense')
            ui.button('加入排程', on_click=add_task).props('color=primary')

        ui.label('目前的排程任務').classes('text-h5 text-center my-4')
        
        tasks_container = ui.column().classes('w-full max-w-3xl gap-y-4')

        stored_tasks = app.storage.general.get('download_tasks', {})
        for task_id, task_data in list(stored_tasks.items()):
            if scheduler.get_job(task_id):
                create_task_ui(task_id=task_id, **task_data)
            else:
                print(f"警告：在儲存中找到任務 {task_id}，但在排程器中不存在。")

# --- 應用程式啟動與關閉事件 ---
def on_startup():
    """應用啟動時執行的函式，恢復排程任務並啟動排程器。"""
    global main_loop
    main_loop = asyncio.get_running_loop()
    
    print("正在從儲存中恢復任務...")
    stored_tasks = app.storage.general.get('download_tasks', {})
    for task_id, task_data in stored_tasks.items():
        if not scheduler.get_job(task_id):
            print(f"  - 正在恢復任務: {task_id} (間隔: {task_data['interval']} 分鐘)")
            scheduler.add_job(
                download_csv_file,
                trigger=IntervalTrigger(minutes=task_data['interval']),
                args=[task_id],
                id=task_id,
                misfire_grace_time=60
            )

    if not scheduler.running:
        scheduler.start(paused=False)
        print("排程器已啟動。")

def on_shutdown():
    print("應用程式準備關閉，正在關閉排程器...")
    if scheduler.running:
        scheduler.shutdown()
        print("排程器已關閉。")

app.on_startup(on_startup)
app.on_shutdown(on_shutdown)

# --- 執行 APP ---
ui.run(title='CSV 下載器', dark=True, reload=False, port=8080, storage_secret='THIS_IS_A_VERY_SECRET_KEY')

```