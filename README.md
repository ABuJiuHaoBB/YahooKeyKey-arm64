# Yahoo! KeyKey（Apple Silicon / arm64 原生版）

本專案將 Yahoo! 開源的「Yahoo! KeyKey」輸入法由 **Intel 版轉譯為 Apple Silicon（arm64）原生版**，
在 Apple Silicon 上原生執行，不再依賴 Rosetta。支援 macOS 15.0（Sequoia）以上。

> **穩定版聲明**：本版（**v0.1.2**，2026-09-24）為「好打注音」選字優化後、本機測試通過的穩定版。
> 歡迎大家安裝使用；若遇到任何問題，請回報到本專案的 **GitHub Issues**。

## 安裝

1. 下載 `YahooKeyKey-arm64.pkg`（雙擊安裝，需管理員密碼；安裝到 `/Library/Input Methods`），
   或 `.dmg`（拖曳 `Yahoo! KeyKey.app` 到 `~/Library/Input Methods`）。
2. **登出再登入**（輸入法需重載才會生效）。
3. 系統設定 → 鍵盤 → 文字輸入 → 編輯 → 啟用 Yahoo! KeyKey。

> **首次開啟提示**：本版為 adhoc 簽署（未公證）。若 macOS 跳出「unidentified developer」，
> 在 Finder 對 App 按右鍵 →「打開」，或先執行 `xattr -cr "Yahoo! KeyKey.app"`。

## 本版（v0.1.2，2026-09-24）

- 修正「好打注音（智慧注音）」連續打字卡頓：組字引擎改為「前向 beam 剪枝」，
  限制候選路徑探索量，避免同音字 × 多字組合造成指數爆炸（卡頓/彩球）。
- 提升「好打注音」選字準確度：
  - 放寬組字區間（考量更長的詞/句子，長詞優先）。
  - 調整自動組字（autocommit）門檻為滿 4 字，長詞優先、避免過早斷字。
- 偏好設定「軟體更新」的「您目前使用的版本」顯示為 `1.1.2535-arm64(0.1.2)`。

## 本版（v0.1.0，2026-09-22）

- 已轉譯為 **arm64 原生**：主程式、Preferences.app、PhraseEditor.app 均為 arm64（不再依賴 Rosetta）。
- 修正「偏好設定打不開」：補上 Preferences.app / PhraseEditor.app 的 MainMenu.nib（ibtool 改用完整 Xcode）。
- 移除 DownloadUpdate.app、InstallerHelp.app（已從選單移除、無程式碼啟動）。
- 新增「安裝後提醒」：pkg 安裝完成後跳出提醒，告知需重新啟動（或登出再登入）輸入法才會生效。
- 修正「傳統注音」選字框不出現、選單反白不能選（2026-09-18）。

## 已知限制

- 智慧注音（好打注音）的語言模型由開源「小麥注音（McBopomofo）」詞庫重建
  （原始 KeyKey.db 為 CEROD/SEE 加密，商用 codec 不在倉庫）。
- 傳統注音、倉頡、簡易/速成、Canton 可用。
- 本版為 adhoc 簽署（未公證），macOS 可能提示「unidentified developer」（見上）。

## 授權

BSD 3-clause（見 `LICENSE`）。第三方元件授權見 `THIRD_PARTY_NOTICES.txt`。
