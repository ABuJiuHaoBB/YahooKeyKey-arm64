# Yahoo! KeyKey（Apple Silicon 原生版）

從 Yahoo! 開源的 KeyKey / OpenVanilla 原始碼重新編譯，可在 Apple Silicon（arm64）上原生執行，
不再依賴 Rosetta。支援 macOS 15.0（Sequoia）以上。

## 安裝

1. 下載 `YahooKeyKey-arm64.pkg`（雙擊安裝，需管理員密碼；安裝到 `/Library/Input Methods`），
   或 `.dmg`（拖曳 `Yahoo! KeyKey.app` 到 `~/Library/Input Methods`）。
2. **登出再登入**（輸入法需重載才會生效）。
3. 系統設定 → 鍵盤 → 文字輸入 → 編輯 → 啟用 Yahoo! KeyKey。

> **首次開啟提示**：本版為 adhoc 簽署（未公證）。若 macOS 跳出「unidentified developer」，
> 在 Finder 對 App 按右鍵 →「打開」，或先執行 `xattr -cr "Yahoo! KeyKey.app"`。

## 本版修正（2026-09-18）

- 修正「傳統注音」輸入後選字框不出現：補上絕對順序查詢（`OVIMTRADITIONALMANDARIN_USE_ABSOLUTE_ORDER_QUERY_STRING`）。
- 修正「好打注音 / 傳統注音」在選單中反白不能選：模組識別碼改回 `SmartMandarin` / `TraditionalMandarin`，
  與選單一致（`-DOVIMSMARTMANDARIN_IDENTIFIER="SmartMandarin"` 等）。

## 已知限制

- 智慧注音（好打注音）的語言模型由開源「小麥注音（McBopomofo）」詞庫重建
  （原始 KeyKey.db 為 CEROD/SEE 加密，商用 codec 不在倉庫）。
- 傳統注音、倉頡、簡易/速成、Canton 可用。
- 輔助 App（Preferences / PhraseEditor）仍為 x86_64，走 Rosetta；未安裝可執行
  `softwareupdate --install-rosetta`。

## 授權

BSD 3-clause（見 `LICENSE`）。第三方元件授權見 `THIRD_PARTY_NOTICES.txt`。
