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
- 智慧注音（Smart Phonetic）未註冊（依賴 Yahoo 未公開的加密普通話語言模型）。
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

## 授權
本專案採 BSD 3-clause（見 LICENSE）。第三方元件之授權與署名見 THIRD_PARTY_NOTICES.txt；App 內亦附於 Contents/Resources/Licenses/。
