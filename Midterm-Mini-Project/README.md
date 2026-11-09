# 社團公告整理小幫手

把社團活動公告整理成方便社員快速查閱的活動資訊、報名方式與待辦。目標使用者是想快速知道「何時、何地、怎麼報名、接下來要做什麼」的社員；輸出只依公告原文，不替公告補造資料。

## 功能與欄位

貼上最多 6000 字元的公告，按「整理公告」後，App 以 OpenAI Responses API 與 Pydantic Structured Outputs 回傳以下必填欄位：

| 欄位 | 用途 |
|---|---|
| 活動名稱 | 找出公告所指的活動 |
| 活動日期與時間 | 依原文顯示，不自行補年份或推算日期 |
| 活動地點 | 快速確認活動在哪裡舉行 |
| 報名截止時間 | 提醒社員是否仍可報名 |
| 報名方式 | 找出公告明確提供的報名步驟 |
| 社員待辦 | 列出原文明確要求社員完成的動作 |
| 公告尚缺資訊 | 提醒判斷是否參加或報名時仍需確認的內容 |

公告沒有提供的單一欄位會標示「原文未提及」；清單欄位若沒有項目則顯示空清單。模型輸出格式通過檢查仍不代表事實正確，請逐欄對照畫面保留的原文。

## 使用技術與設計理由

- 使用 **Pydantic Structured Outputs**：以 `ProjectResult` 定義欄位型別與用途，讓 SDK 依模型產生 strict JSON Schema，並驗證解析結果。
- 使用 Streamlit 表單：只有明確提交才處理；API 開關預設關閉，離線檢查不會產生 AI 結果。
- 一次有效提交最多呼叫 API 一次，`max_retries=0`。輸入空白、超過 6000 字元、缺少金鑰或輸出驗證失敗時，不保存為成功結果。
- 原文會送到 OpenAI API；`store=False` 不代表不傳輸或不計費。請只使用虛構或已獲授權的公告內容，不要貼個資或機密。

## 本機安裝與啟動

在 PowerShell 進入此資料夾。第一次設定時建立虛擬環境；若 `.venv` 已存在，跳過第一行。**不必執行 `Activate.ps1`**，以下直接使用虛擬環境內的 Python：

```powershell
python -m venv .venv
.\.venv\Scripts\python.exe -m pip install -r requirements.txt
.\.venv\Scripts\python.exe -m pip check
if (-not (Test-Path .env)) { Copy-Item .env.example .env }
```

在 `.env` 設定自己的金鑰與模型（不要將真正的 `.env` 提交至 Git）：

```dotenv
OPENAI_API_KEY=你的OpenAI_API金鑰
OPENAI_MODEL=gpt-5.4-mini
```

啟動 App：

```powershell
.\.venv\Scripts\python.exe -m streamlit run app.py
```

套件直接依賴版本：OpenAI 2.49.0、Streamlit 1.63.0、python-dotenv 1.2.3、Pydantic 2.14.1。若 Python 或套件版本不同，請在測試紀錄中填寫實際版本。金鑰亦可依 `.streamlit/secrets.example.toml` 設定在 Streamlit Community Cloud 的 App Secrets。

## Cloud 套件設定

Streamlit Community Cloud 的 Main file path 使用 `Midterm-Mini-Project/app.py`；套件請使用 Repo 根目錄的 `requirements.txt`（已包含本 App 所需的 Pydantic）。根目錄清單直接列出安裝套件，不要把其他 requirements 檔案路徑寫成套件名稱。專案資料夾內的 `requirements.txt` 保留供本機在此資料夾安裝使用。

## 操作方式

1. 啟動後先保持「啟用真實 API」關閉，貼上虛構公告並送出；畫面只會做輸入與設計檢查，不會產生模型結果。
2. 確認已設定金鑰，再明確開啟 API；送出虛構公告以取得結構化結果。
3. 逐欄查看活動資訊、社員待辦與缺漏提醒，並與下方顯示的原文核對。
4. 資訊不足測試可使用刻意缺少地點或報名方式的虛構公告，確認 App 不會把推測內容當成公告事實。

虛構輸入範例（不是 API 測試紀錄）：

> 星河攝影社將於 11 月 20 日下午 2 點在活動中心 301 教室舉辦手機攝影分享會。請在 11 月 18 日前填寫社團群組置頂的報名表，並自備手機。

若使用此例，核對日期、時間、地點、截止時間與報名方式是否逐字有原文根據；「社員待辦」應只整理填表與自備手機等明確指示。範例沒有列出的費用或其他資訊，不應自行補上。這只是操作練習素材，不代表模型已產生或通過驗收。

## 測試與部署狀態

### 已完成

- 本機離線檢查：App 可載入、API 預設關閉、有效文字不產生 AI 結果、空白輸入會被擋下。
- 6001 字元輸入：2026-11-10 本機測試顯示超長提示，且不默默截斷。當時 API 開關開啟；程式會先檢查長度再呼叫 API，因此預期未送出請求。
- 本機資訊不足案例：2026-11-10 的畫面顯示真實 API 格式檢查通過；使用者提供的結果含活動名稱「新生交流場」、時間「下週五晚上」，並將地點及報名截止標示為未提及。

### 尚待完成或確認

- 資訊不足案例的原文、模型名稱、確切請求數及完整逐欄核對；結果中的「回覆是否出席」須確認是否有原文根據。
- 部署前曾出現 `NotFoundError: removeChild`；重新部署後是否重現尚未確認，不能僅憑舊截圖判定根因。
- 正常公告真實 API 測試尚未確認。
- Streamlit Community Cloud：**使用者回報重新部署已成功**。請確認 Cloud App 可正常操作，並完成真實 API 雲端驗收。

詳細狀態與測試證據請看[專題設計與驗收紀錄](./第08週_課堂簡化版_專題設計與驗收紀錄.md)。

| Cloud 驗收項目 | 狀態 |
|---|---|
| Streamlit App 網址 | 待貼入實際 Cloud App 網址 |
| Repo／分支／Main file path | `115_1_GenAI-App-Development1`／`main`／`Midterm-Mini-Project/app.py` |
| Cloud Python 版本／老師可否開啟 | Python 3.14.10；重新部署成功、Cloud Secrets 已由使用者設定；老師觀看權限待確認 |
| 雲端真實 API 成功日期與結果 | 未測；部署後填入實際虛構輸入、輸出及逐欄核對結果 |

### Streamlit Cloud Secrets 設定

Cloud Secrets 已由使用者設定；金鑰值不記錄於文件、程式碼或 GitHub。若日後需檢查設定，於 Streamlit Community Cloud 開啟此 App 的 **Manage app／Settings → Secrets**，依 `.streamlit/secrets.example.toml` 確認 `OPENAI_API_KEY` 與 `OPENAI_MODEL` 存在即可；不要複製或分享金鑰：

```toml
OPENAI_API_KEY = "your-key"
OPENAI_MODEL = "gpt-5.4-mini"
```

按 Save 後等待 App 重啟。回到 App 先確認畫面可正常操作；只有準備好進行一次真實測試時才開啟 API。若已設定 Secrets，不要為了確認而反覆送出 API 請求。

部署入口為 `Midterm-Mini-Project/app.py`；Cloud 使用 Repo 根目錄的 `requirements.txt`。不要在 GitHub 提交 `.env`、`.streamlit/secrets.toml`、虛擬環境或金鑰。

## 限制與後續

目前只接受貼上的文字，不讀取 PDF、Word 或其他檔案；日期不會轉換時區或補年份。公告措辭含糊時，模型仍可能誤解，因此應以原公告及社團正式資訊為準。後續可依測試結果改善欄位或加入公告文字檔匯入，但不應以延伸功能取代本專題的基本測試與 Cloud 部署。
