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

在此資料夾使用 Python 3.14 建立環境：

```powershell
py -3.14 -m venv .venv
.\.venv\Scripts\Activate.ps1
python -m pip install -r requirements.txt
python -m pip check
Copy-Item .env.example .env
```

在 `.env` 設定自己的金鑰與模型（不要將真正的 `.env` 提交至 Git）：

```dotenv
OPENAI_API_KEY=你的OpenAI_API金鑰
OPENAI_MODEL=gpt-5.4-mini
```

啟動 App：

```powershell
python -m streamlit run app.py
```

套件直接依賴版本：OpenAI 2.49.0、Streamlit 1.63.0、python-dotenv 1.2.3、Pydantic 2.14.1。若 Python 或套件版本不同，請在測試紀錄中填寫實際版本。金鑰亦可依 `.streamlit/secrets.example.toml` 設定在 Streamlit Community Cloud 的 App Secrets。

## 操作方式

1. 啟動後先保持「啟用真實 API」關閉，貼上虛構公告並送出；畫面只會做輸入與設計檢查，不會產生模型結果。
2. 確認已設定金鑰，再明確開啟 API；送出虛構公告以取得結構化結果。
3. 逐欄查看活動資訊、社員待辦與缺漏提醒，並與下方顯示的原文核對。
4. 資訊不足測試可使用刻意缺少地點或報名方式的虛構公告，確認 App 不會把推測內容當成公告事實。

虛構輸入範例（不是 API 測試紀錄）：

> 星河攝影社將於 11 月 20 日下午 2 點在活動中心 301 教室舉辦手機攝影分享會。請在 11 月 18 日前填寫社團群組置頂的報名表，並自備手機。

若使用此例，核對日期、時間、地點、截止時間與報名方式是否逐字有原文根據；「社員待辦」應只整理填表與自備手機等明確指示。範例沒有列出的費用或其他資訊，不應自行補上。這只是操作練習素材，不代表模型已產生或通過驗收。

## 測試與部署狀態

- 本機環境／空白與過長輸入：**尚未記錄實測，請完成後填入**。
- 資訊不足公告：**2026-11-10 使用者提供本機結果**，包含活動名稱「新生交流場」、時間「下週五晚上」，並將地點及報名截止標示為未提及；輸入原文、模型、請求數待確認，畫面另曾出現 `NotFoundError: removeChild` 前端錯誤。正常公告真實 API 測試仍未確認。
- Streamlit Community Cloud：**尚未部署**。部署必做，完成後請填入 App 網址、Repo／分支／入口、Cloud Python 版本、驗收日期、老師觀看權限，以及至少一筆雲端真實 API 成功結果。
- 詳細設計與驗收狀態請記錄在[專題設計與驗收紀錄](./第08週_課堂簡化版_專題設計與驗收紀錄.md)。

| Cloud 驗收項目 | 狀態 |
|---|---|
| Streamlit App 網址 | 未部署 |
| Repo／分支／Main file path | 待部署填寫；預計入口 `Midterm Mini-Project/app.py` |
| Cloud Python 版本／老師可否開啟 | 未測 |
| 雲端真實 API 成功日期與結果 | 未測；部署後填入實際虛構輸入、輸出及逐欄核對結果 |

部署入口預計為 `Midterm Mini-Project/app.py`，requirements 在入口同層。部署前請確認此資料夾已推送到 GitHub，並在 Streamlit Community Cloud 選擇正確 Repo、分支及 Main file path；平台可用的 Python 版本與安裝結果須實際確認。不要提交 `.env`、`.streamlit/secrets.toml`、虛擬環境或金鑰。

## 限制與後續

目前只接受貼上的文字，不讀取 PDF、Word 或其他檔案；日期不會轉換時區或補年份。公告措辭含糊時，模型仍可能誤解，因此應以原公告及社團正式資訊為準。後續可依測試結果改善欄位或加入公告文字檔匯入，但不應以延伸功能取代本專題的基本測試與 Cloud 部署。
