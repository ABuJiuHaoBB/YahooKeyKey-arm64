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
- 智慧注音（Smart Phonetic）未註冊（依賴 Yahoo 未公開的加密普通話語言模型）。
- 傳統注音、速成、倉頡、廣東拼音可用。
- 輔助 App（Preferences / PhraseEditor）仍為 x86_64，會走 Rosetta，不影響主輸入法。

## 修正紀錄（2026-09-11）
- 修正 `libcrypto.3.dylib` 的 install name 為 `@rpath/libcrypto.3.dylib`
  （原為建置機絕對路徑 `/tmp/openssl-src/build/lib/libcrypto.3.dylib`，
  導致 dyld 啟動時找不到、輸入法啟動即崩潰、選字框不出現）。
  已重新簽署（adhoc）。
- 補齊 Info.plist 與 rosetta 版一致的鍵值（TICapsLockLanguageSwitchCapable、
  TISParticipatesInTouchBar、arm64 最小系統版本）。

## 授權
本專案採 BSD 3-clause（見 LICENSE）。第三方元件之授權與署名
見 THIRD_PARTY_NOTICES.txt；App 內亦附於 Contents/Resources/Licenses/。
