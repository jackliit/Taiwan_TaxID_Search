# 統一編號查詢與整合服務 (Unified Business Number Service)

這是一個專門用於查詢與整合台灣公司、機關、學校及非營利事業「統一編號」的服務。系統整合了政府開放資料 API 與自建資料庫 (Supabase)，提供高效的單筆與批次查詢功能。

## 專案架構

本專案主要由以下幾個部分組成：

1.  **API 服務 (`api/index.py`)**
    - 部署於 Vercel 的 Serverless Function。
    - 提供 HTTP GET/POST 介面供外部呼叫。
    - 內建簡易 Web 介面 (GUI)，可直接在瀏覽器進行查詢。
    - **雙重查詢機制**：
        1.  優先查詢 [經濟部商業司開放資料 API](https://data.gcis.nat.gov.tw/main/index.jsp)。
        2.  若查無資料，自動轉查自建資料庫 (Supabase `unified_numbers` 表)。

2.  **資料更新腳本 (`batch_update.py`)**
    - 定期從多個政府公開 CSV 來源下載最新資料 (如全國各級學校、行政院所屬機關、非營利事業等)。
    - 清洗、去重後，將資料更新至 Supabase 資料庫。

3.  **自動化流程 (`.github/workflows/refresh_data.yml`)**
    - 使用 GitHub Actions 設定排程 (Cron Job)。
    - 每週一定期執行 `batch_update.py`，確保資料庫保持最新。

4.  **本地檔案處理 (`DownloadMergeCSV.py`)**
    - 用於本地端下載並合併 CSV 資料，產出 `final_unified_ids_unique.xlsx` 與 `.csv` 檔案供人工檢視或離線使用。

---

## 安裝與環境設定

若要在本地端執行或開發，請依照以下步驟：

### 1. 複製專案與安裝套件

```bash
git clone <your-repo-url>
cd <your-repo-folder>
pip install -r requirements.txt
```

### 2. 設定環境變數

本專案依賴 Supabase，請確保設定以下環境變數 (可建立 `.env` 檔案或直接在系統設定)：

- `SUPABASE_URL`: 您的 Supabase 專案 URL
- `SUPABASE_KEY`: 您的 Supabase Service Role Key (或 Anon Key，視權限設定而定)

---

## 如何使用

### 1. Web 介面 (GUI)

直接使用瀏覽器開啟部署後的網址 (例如 `https://your-project.vercel.app/`)，即可看到圖形化介面。

- **單筆查詢**：透過網址參數查詢。
- **批次查詢**：在首頁文字框輸入多個統一編號，點擊「查詢全部」即可一次獲取名稱與資料來源。

### 2. API 呼叫方法

#### 單筆查詢 (GET)

- **透過統編查詢**：
  `GET /?id=03730043` 或 `GET /?統一編號=03730043`
- **透過名稱查詢**：
  `GET /?name=台灣大學` 或 `GET /?單位名稱=台灣大學`
- **忽略政府 API (僅查資料庫)**：
  增加參數 `&skip_govt=true`

**回傳範例 (JSON)**：
```json
{
  "data": [
    {
      "統一編號": "03730043",
      "單位名稱": "國立臺灣大學",
      "資料來源": "全國各級學校"
    }
  ],
  "error": null
}
```

#### 批次查詢 (POST)

- **Endpoint**: `/api`
- **Method**: `POST`
- **Headers**: `Content-Type: application/json`
- **Body**:
  ```json
  {
    "ids": ["03730043", "04199019"],
    "skip_govt": false
  }
  ```

### 3. 資料更新 (手動/自動)

本專案支援手動與自動更新，讓資料隨時保持最 Fresh 的狀態！✨

- **自動定時更新**：
  透過 GitHub Actions 已經設定好排程（Cron Job），每週一凌晨會自動執行 `batch_update.py` 腳本，自動化處理超方便！

- **手動更新資料庫**：
  確保環境變數已設定，執行：
  ```bash
  python batch_update.py
  ```
- **產出本地 Excel/CSV**：
  ```bash
  python DownloadMergeCSV.py
  ```
  執行後會產生 `final_unified_ids_unique.xlsx`。

---

## 資料來源

本系統整合以下公開資料：
1. 全國各級學校
2. 行政院所屬機關
3. 地方政府機關
4. 非營利事業
5. 經濟部商業司 (即時 API)
