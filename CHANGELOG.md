# 變更紀錄（CHANGELOG）

本檔記錄每次重建的修正內容。日期為重建日期；後續建議改用語意化版本號（如 v1.0.0）。

## 2026-09-13
- **安裝器（PKG）調整**：
  - 在 `Distribution` 加上 `<title>Yahoo! KeyKey</title>`，讓安裝視窗顯示軟體名稱（先前只顯示空字串）。
  - 加入 `postinstall` 腳本：安裝完成後顯示提醒對話框，告知「請登出再登入」以啟用輸入法（如需強制登出，可將腳本中 `tell application "System Events" to log out` 那行取消註解）。
- 已重新簽署（adhoc）並重建 pkg。

## 2026-09-12 晚
- **修正「啟動即崩潰、選單列看不到輸入法選項」**：找到兩處崩潰並修正。
  - `FetchSQLiteCERODKey` 回傳的 `:cerod:` 前置會讓標準 sqlite3 開啟失敗（本 build 的 `KeyKey.db` 其實是未加密的一般 SQLite），導致 `OVSQLiteConnection::Open` 回傳 null、後續 `execute` 在 null 上 SIGSEGV。改為直接回傳檔名。
  - `mergeOneKeyData` / `mergeCannedMessagesData` 對空字串解析出 null 後未防護，造成 `dictionaryKeys` 崩潰；已加上 null 防護。
- **補回全型標點（全形符號）**：arm64 版缺少標點表格，導致 `shift+,` 只輸出半型 `<`。已加入 `DataTables/Punctuations/punctuation.cin`（表格名稱 `Punctuations-punctuation-cin`，對應模組查詢 `Punctuations-punctuation*`），使 `shift+,` → `，`、`shift+.` → `。` 等恢復全形。
- 已重新編譯 arm64 二進位、重新簽署（adhoc）、重建 pkg / dmg / zip。

## 2026-09-12
- **修正「輸入聲調後整組跳字」的問題**：先前「絕對順序」2 字元編碼會把大小寫字母當作編碼的一部分；而 `.cin` 表格預設以「不區分大小寫」方式載入（鍵會被小寫化），導致不同音節碰撞。例如：`zp4`（ㄈㄣˋ）與 `wu06`（ㄊㄧㄢˊ）分別編碼成 `}i` 與 `}I`，小寫化後都變成 `}i`，於是「輸入聲調後整組跳成另一組字」。（另 `zp6`→`tj06`、`vul4`→`tjo6` 等 178 組大小寫碰撞）。
  - 本次改為「不含大小寫」的 3 字元編碼（使用不含空白、且 `tolower` 恆等的 68 個可印 ASCII 字元，`68^3 > 6160`），並重新產生 `bpmf.cin`，使小寫化後不再碰撞；同時在 `OVIMTraditionalMandarin::initialize` 中把 bpmf 表格改為區分大小寫載入（對 SQLite 版本無影響）。
  - 已重新編譯 arm64 二進位、重新簽署（adhoc）並重建 pkg / dmg / zip。
  - 另修正編譯期相容性：`PVPropertyListExpat.cpp`（`ofs`→`sst`）、`StaticPack.h`（多餘限定）、各 `PackageMain.cpp`（改為 static 避免重複符號）、`CVSendKey.m`（arm64 走 `kchrID = 0` 分支）、若干控制器補上 delegate 協定、`string([obj method])` 的「most vexing parse」改寫，並補上無操作的 CEROD codec 相容實作（`Distributions/Takao/Keyring/CERODCodec.c`）。

## 2026-09-11
- 修正 `libcrypto.3.dylib` 的 install name 為 `@rpath/libcrypto.3.dylib`（原為建置機絕對路徑 `/tmp/openssl-src/build/lib/libcrypto.3.dylib`，導致 dyld 啟動時找不到、輸入法啟動即崩潰、選字框不出現）。已重新簽署（adhoc）。
- 補齊 Info.plist 與 rosetta 版一致的鍵值（TICapsLockLanguageSwitchCapable、TISParticipatesInTouchBar、arm64 最小系統版本）。
- **修正傳統注音選字框不出現的問題**：此 build 以 `OVIMTRADITIONALMANDARIN_USE_ABSOLUTE_ORDER_QUERY_STRING` 查詢（使用「絕對順序」2 字元編碼），但原本的 `bpmf.cin` 鍵是「按鍵序列」，導致查詢幾乎都對不上、候選字為空、只會 beep。已用 `Frameworks/Formosa/Tools/ConvertBPMFCin` 把 `bpmf.cin` 的鍵轉成絕對順序編碼，使查詢能正確命中候選字並顯示選字框。
