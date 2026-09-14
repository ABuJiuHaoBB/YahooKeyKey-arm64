# Yahoo! KeyKey（原生 Apple Silicon / arm64 版）

從 Yahoo! 開源的 KeyKey / OpenVanilla 原始碼（bency/YahooKeyKey 分支）重新編譯，
可在 Apple Silicon（含 macOS 26 以後）原生執行，不再依賴 Rosetta。

## 安裝
1. 下載 `YahooKeyKey-arm64.pkg`（雙擊安裝，需管理員密碼；安裝到 /Library/Input Methods）
   或 `.dmg`（拖曳 Yahoo! KeyKey.app 到 ~/Library/Input Methods）。
2. 登出再登入。
3. 系統設定 → 鍵盤 → 文字輸入 → 編輯，啟用 Yahoo! KeyKey。

## 檔案
- YahooKeyKey-arm64.pkg  — macOS 標準安裝器
- YahooKeyKey-arm64.dmg  — 磁碟映像（含 App + 安裝腳本）
- YahooKeyKey-arm64.zip  — App 壓縮檔
- LICENSE                — 主授權（BSD 3-clause，Yahoo! Inc.）
- THIRD_PARTY_NOTICES.txt — 第三方元件署名（OpenVanilla、OpenSSL、jieba-tw…）

## 已知限制
- 智慧注音（Smart Phonetic）目前以「重建的最小語言模型」運作：
  - unigrams（候選字）由已轉換的 `bpmf.cin` 重建；
  - bigrams（上下文）由 `associated_phrases` 重建；
  - 因此「好打注音」可正常選字、並能依上下文自動校正（非 Yahoo 原始語言模型，
    準確度與完整度有限）。
- 傳統注音、速成、倉頡、廣東拼音可用。
- 輔助 App（Preferences / PhraseEditor）仍為 x86_64，會走 Rosetta，不影響主輸入法。

## 修正紀錄（2026-09-11）
- 修正 `libcrypto.3.dylib` 的 install name 為 `@rpath/libcrypto.3.dylib`
  （原為建置機絕對路徑 `/tmp/openssl-src/build/lib/libcrypto.3.dylib`，
  導致 dyld 啟動時找不到、輸入法啟動即崩潰、選字框不出現）。
  已重新簽署（adhoc）。
- 補齊 Info.plist 與 rosetta 版一致的鍵值（TICapsLockLanguageSwitchCapable、
  TISParticipatesInTouchBar、arm64 最小系統版本）。
- **修正傳統注音選字框不出現的問題**：此 build 以 `OVIMTRADITIONALMANDARIN_USE_ABSOLUTE_ORDER_QUERY_STRING`
  查詢（使用「絕對順序」2 字元編碼），但原本的 `bpmf.cin` 鍵是「按鍵序列」，
  導致查詢幾乎都對不上、候選字為空、只會 beep。已用
  `Frameworks/Formosa/Tools/ConvertBPMFCin` 把 `bpmf.cin` 的鍵轉成絕對順序編碼，
  使查詢能正確命中候選字並顯示選字框。

## 修正紀錄（2026-09-12）
- **修正「輸入聲調後整組跳字」的問題**：先前「絕對順序」2 字元編碼會
  把大小寫字母當作編碼的一部分；而 `.cin` 表格預設以「不區分大小寫」
  方式載入（鍵會被小寫化），導致不同音節碰撞。例如：
  `zp4`（ㄈㄣˋ）與 `wu06`（ㄊㄧㄢˊ）分別編碼成 `}i` 與 `}I`，
  小寫化後都變成 `}i`，於是「輸入聲調後整組跳成另一組字」。
  （另 `zp6`→`tj06`、`vul4`→`tjo6` 等 178 組大小寫碰撞）。

  本次改為「不含大小寫」的 3 字元編碼（使用不含空白、且 `tolower`
  恆等的 68 個可印 ASCII 字元，`68^3 > 6160`），並重新產生 `bpmf.cin`，
  使小寫化後不再碰撞；同時在 `OVIMTraditionalMandarin::initialize`
  中把 bpmf 表格改為區分大小寫載入（對 SQLite 版本無影響）。
  已重新編譯 arm64 二進位、重新簽署（adhoc）並重建 pkg / dmg / zip。
  另修正編譯期相容性：`PVPropertyListExpat.cpp`（`ofs`→`sst`）、
  `StaticPack.h`（多餘限定）、各 `PackageMain.cpp`（改為 static 避免
  重複符號）、`CVSendKey.m`（arm64 走 `kchrID = 0` 分支）、
  若干控制器補上 delegate 協定、`string([obj method])` 的
  「most vexing parse」改寫，並補上無操作的 CEROD codec 相容實作
  （`Distributions/Takao/Keyring/CERODCodec.c`）。


## 協作與推送（Pull Request）
- 本專案的變更一律透過 **feature branch + Pull Request** 送交 main。
- 可用輔助腳本 `~/scripts/push-pr.sh`（自動建立分支、提交、推送、開 PR）：
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

## 授權
本專案採 BSD 3-clause（見 LICENSE）。第三方元件之授權與署名
見 THIRD_PARTY_NOTICES.txt；App 內亦附於 Contents/Resources/Licenses/。
