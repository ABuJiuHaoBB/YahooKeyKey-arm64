# Yahoo! KeyKey（原生 Apple Silicon / arm64 版）

從 Yahoo! 開源的 KeyKey / OpenVanilla 原始碼（[bency/YahooKeyKey](https://github.com/bency/YahooKeyKey) 分支）重新編譯，可在 Apple Silicon 上原生執行（不再依賴 Rosetta），支援 macOS 15.0（Sequoia）以上。

## 安裝
1. 下載 `YahooKeyKey-arm64.pkg`（雙擊安裝，需管理員密碼；安裝到 `/Library/Input Methods`），或 `.dmg`（拖曳 Yahoo! KeyKey.app 到 `~/Library/Input Methods`，或雙擊 `Install Yahoo! KeyKey.command`）。
2. 登出再登入。
3. 系統設定 → 鍵盤 → 文字輸入 → 編輯，啟用 Yahoo! KeyKey。

> **首次開啟提示**：本版為 adhoc 簽署（未公證）。從 GitHub 下載後若 macOS 跳出「unidentified developer」，請在 Finder 對 App 按右鍵 →「打開」，或先執行 `xattr -cr "Yahoo! KeyKey.app"` 清除 quarantine。

## 檔案
- `YahooKeyKey-arm64.pkg` — macOS 標準安裝器（系統層級，需管理員密碼）
- `YahooKeyKey-arm64.dmg` — 磁碟映像（含 App + 安裝腳本）
- `YahooKeyKey-arm64.zip` — App 壓縮檔
- `YahooKeyKey.pkg` — 原始 Intel / Rosetta 版（供舊機或對照用）
- `LICENSE` — 主授權（BSD 3-clause，Yahoo! Inc.）
- `THIRD_PARTY_NOTICES.txt` — 第三方元件署名（OpenVanilla、OpenSSL、jieba-tw…）

## 已知限制
- 智慧注音（Smart Phonetic）目前以「重建語言模型」運作：
  - unigrams / bigrams 由開源、持續維護的 **小麥注音（McBopomofo）詞庫** 重建，
    （含片語頻率，頻率加權，可依上下文自動校正）。
  - 原始 Yahoo KeyKey.db 為 CEROD/SEE 加密（商用 codec 不在倉庫），
    因此不直接掛載；改用 McBopomofo 詞庫為替代來源（詞彙更新、頻率更準確）。
- 傳統注音、速成、倉頡、廣東拼音可用。
- 輔助 App（Preferences / PhraseEditor）仍為 x86_64，會走 Rosetta（不影響主輸入法）；若未安裝 Rosetta，可執行 `softwareupdate --install-rosetta`。

## 變更紀錄
見 [CHANGELOG.md](CHANGELOG.md)。

## 協作與推送（Pull Request）
- 本專案的變更一律透過 **feature branch + Pull Request** 送交 main。
- 可選輔助腳本 `~/scripts/push-pr.sh`（自動建立分支、提交、推送、開 PR）：
  ```
  ~/scripts/push-pr.sh fix-xxx "修正說明"
  ```
- 若未安裝 gh CLI，腳本會印出 GitHub compare URL，供你在網頁開 PR。

## 修正紀錄（2026-09-12 晚）
- **修正「啟動即崩潰、選單列看不到輸入法選項」**：找到兩處崩潰並修正。
  - `FetchSQLiteCERODKey` 回傳的 `:cerod:` 前置會讓標準 sqlite3 開啟失敗
    （本 build 的 `KeyKey.db` 其實是未加密的一般 SQLite），導致 `OVSQLiteConnection::Open`
    回傳 null、後續 `execute` 在 null 上 SIGSEGV。改為直接回傳檔名。
  - `mergeOneKeyData` / `mergeCannedMessagesData` 對空字串解析出 null 後未防護，
    造成 `dictionaryKeys` 崩潰；已加上 null 防護。
- **補回全型標點（全形符號）**：arm64 版缺少標點表格，導致 `shift+,` 只輸出半型 `<`。
  已加入 `DataTables/Punctuations/punctuation.cin`（表格名稱 `Punctuations-punctuation-cin`，
  對應模組查詢 `Punctuations-punctuation*`），使 `shift+,` → `，`、`shift+.` → `。` 等恢復全形。
- 已重新編譯 arm64 二進位、重新簽署（adhoc）、重建 pkg / dmg / zip。


## 修正紀錄（2026-09-13）
- **安裝器（PKG）調整**：
  - 在 `Distribution` 加上 `<title>Yahoo! KeyKey</title>`，讓安裝視窗顯示軟體名稱（先前只顯示空字串）。
  - 加入 `postinstall` 腳本：安裝完成後顯示提醒對話框，告知「請登出再登入」以啟用輸入法
    （如需強制登出，可將腳本中 `tell application "System Events" to log out` 那行取消註解）。
- 已重新簽署（adhoc）並重建 pkg。


## 修正紀錄（2026-09-14）
- **讓「好打注音（智慧注音 / SmartMandarin）」可以運作**：
  - 根因：`OVIMSmartMandarin::initialize()` 要求 KeyKey.db 內存在
    `bigrams` / `unigrams` 兩張表（Yahoo 的語言模型），但 arm64 版只有
    `associated_phrases`，導致初始化失敗、打字只出英文、選字框反白。
  - 做法：以「重建的最小語言模型」補上這兩張表（不需修改程式碼）：
    - `unigrams`：從已轉換的 `DataTables/Mandarin/bpmf.cin`（絕對順序編碼）
      重建候選字表，含 UNK(`*`) / BOS(`!`) / EOS(`$`) 特殊項目。
    - `bigrams`：由 `associated_phrases` 抽出相鄰字對，並用「字→注音編碼」
      反向對映產生上下文資料（機率高於 unigram，使能依上下文自動校正）。
  - 產生工具：`gen_lm.py`（unigrams）、`gen_lm_bigram.py`（bigrams）。
- **修正 Info.plist 最低系統版本不一致**：
  - 二進位實際 `LC_BUILD_VERSION` 的 minos 為 15.0（以 macOS 15.5 SDK 編譯），
    但 Info.plist 中 `LSMinimumSystemVersionByArchitecture` 的 arm64 寫成 10.6.0，
    會誤以為可在 macOS 10.6 執行。
  - 已把 arm64（及通用 `LSMinimumSystemVersion`）改為 15.0。
- **修正打包 zip 內容爆量**：macOS 的 `zip` 對含「!」的檔名（`Yahoo! KeyKey.app`）
  會誤當成 glob，造成壓縮內容爆量；改以 `ditto -c -k` 正確處理。
- 已重建 pkg / dmg / zip（含以上修正），並重新簽署（adhoc）。

## 修正紀錄（2026-09-15）
- **改用「小麥注音（McBopomofo）」詞庫重建語言模型，大幅提升選字精準度**：
  - 先前以 `bpmf.cin`（等機率）+ `associated_phrases`（片語）重建，
    選字只依表格順序、無頻率加權，導致「字不會自動校正」。
  - 改用 McBopomofo 的開源詞庫（`BPMFBase.txt` 單字映射、`BPMFMappings.txt`
    片語映射、`phrase.occ` 片語頻率），重建「頻率加權」的 unigrams / bigrams：
    - unigrams（26538 筆）：單字→絕對順序編碼，機率 = log(頻率+1)。
    - bigrams（137501 筆）：片語相鄰字對，機率 = log(頻率+1) + 2.0（讓上下文優先）。
  - 已驗證：輸入 `ㄊㄞˊ ㄨㄢ` 正確組出「台灣」，且 台(10.89)/灣(10.13) 頻率最高。
  - 產生工具：`gen_lm_mcbo.cpp`（含 Formosa::Mandarin 的注音→絕對順序轉換）。
- 已重建 pkg / dmg / zip，並重新簽署（adhoc）。

## 授權
本專案採 BSD 3-clause（見 LICENSE）。第三方元件之授權與署名見 THIRD_PARTY_NOTICES.txt；App 內亦附於 Contents/Resources/Licenses/。
