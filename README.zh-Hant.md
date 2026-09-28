# AirCard-iOS 多語言適配版

> **本儲存庫是 [Mak5er/AirCard-iOS](https://github.com/Mak5er/AirCard-iOS) 的多語言適配分支版本（fork），並非原版專案。**
>
> 我只做了本地化相關的改動：在原版英文基礎上新增**簡體中文、繁體中文、日語、韓語、俄語**五種語言。功能與上游保持一致，未改動任何業務邏輯。原版功能、設計與實作全部歸功於上游作者，請以 [上游儲存庫](https://github.com/Mak5er/AirCard-iOS) 為準。

[简体中文](README.md) | [繁體中文](README.zh-Hant.md) | [English](README.en.md) | [日本語](README.ja.md) | [한국어](README.ko.md) | [Русский](README.ru.md)

<p align="center">
  <img src="ios-app/Assets.xcassets/AppIcon.appiconset/AppIcon.png" width="128" height="128" alt="AirCard-iOS Icon" style="border-radius: 28px; box-shadow: 0 8px 24px rgba(0,0,0,0.18);" />
</p>

<p align="center">
  直接在 iOS 27+ 上客製 Apple 錢包卡面外觀、鎖定畫面密碼鍵盤主題與 PosterBoard 桌布。
</p>

<p align="center">
  <img src="https://img.shields.io/badge/Platform-iOS%2027+-blue?style=flat-square&logo=apple" alt="Platform" />
  <img src="https://img.shields.io/badge/Swift-5.0-orange?style=flat-square&logo=swift" alt="Swift" />
  <img src="https://img.shields.io/badge/Rust-FFI%20Core-red?style=flat-square&logo=rust" alt="Rust" />
  <img src="https://img.shields.io/badge/License-MIT-green?style=flat-square" alt="License" />
  <img src="https://img.shields.io/badge/Localizations-5%20languages-brightgreen?style=flat-square" alt="Localizations" />
  <img src="https://img.shields.io/badge/Fork%20of-Mak5er%2FAirCard--iOS-lightgrey?style=flat-square" alt="Fork of Mak5er/AirCard-iOS" />
</p>

## 關於本儲存庫

本儲存庫是上游 `Mak5er/AirCard-iOS` 的分支版本（fork），**只包含本地化改動**。除此以外的一切（功能、UI、Rust 核心、漏洞利用實作）都來自上游。

### 本儲存庫相對上游的改動

- **新增 5 種語言**：簡體中文、繁體中文、日語、韓語、俄語。英文作為基準語言保留，未翻譯的文案自動回退為英文原文。
- **本地化範圍**：介面文案、錯誤與狀態提示、系統權限說明（本地網路 / 照片），以及 `CFBundleLocalizations` 中註冊的語言清單。
- **診斷日誌保持英文**：`Activity Log`、`Flash Log`、syslog 等面板的文案一律不本地化，便於對照上游排查問題。
- **少量必要的程式碼調整**：把原先「靠比對狀態字串判斷 UI 顏色」的邏輯改為語言無關的旗標，否則本地化會破壞配對與掃描狀態的顯示；列舉新增 `displayName`，`rawValue` 保持為穩定的持久化識別碼。
- 順帶把 LocalDevVPN 按鈕改為：未安裝該 App 時跳轉 App Store 下載頁。

### 本地化實作方式

沿用 iOS 標準做法，不引入額外依賴：

- 語言資源放在 `ios-app/<語言>.lproj/` 下的 `Localizable.strings` 與 `InfoPlist.strings`。
- **鍵即英文原文**（natural-language keys），因此不需要維護 `en.lproj`：任何未翻譯的鍵都會自動回退成英文原文。
- 佔位符 `%@` / `%lld` / `%d` / `%.1fx` 在各語言檔案中原樣保留。
- 語言跟隨系統設定切換，App 內不提供手動切換器。
- `AirCard-iOS.xcodeproj` 由 XcodeGen 依據 `project.yml` 產生；XcodeGen 會自動辨識 `.lproj` 目錄並建立 variant group，所以重新產生專案後本地化不會遺失。

### 如何只取上游改動

```bash
git remote add upstream https://github.com/Mak5er/AirCard-iOS.git
git fetch upstream
git merge upstream/main
```

## 概覽

AirCard-iOS 無需越獄即可在裝置上客製 Apple 錢包卡面圖案、鎖定畫面密碼撥號盤與鎖定畫面桌布。

App 透過 LocalDevVPN 提供的本地回環通道（`10.7.0.1` 或 `127.0.0.1`）與系統內部服務通訊。檔案操作由 `AirliftFFI` 處理，這是一個與 AirTraffic 服務互動的 Rust 函式庫。

> **相容性**：AirCard-iOS 目前要求 **iOS 27.0 或更高版本（iOS 27+）**。

## 功能

### Apple 錢包卡面外觀
- 將自訂卡面圖案寫入 Passbook 快取（`cardBackgroundCombined@3x.png`、`@2x.png`，以及面向 Suica 等交通卡的 `cardBackgroundCombined.pdf`）。
- 重新整理正面與縮圖快取，使新圖案在錢包開啟時立即生效。
- 喚起 Apple Pay 時即時辨識卡片識別碼。
- 可對單張卡片套用圖案，也可批次刷入所有已辨識卡片。

### 密碼撥號盤主題
- 即時撥號盤預覽，支援觸控平移與縮放取景。
- 支援覆蓋全部十個按鍵的整版海報版面，也支援單一圓形按鍵的獨立去背。
- 面向系統撥號盤快取（`TelephonyUI-10`）。
- 數字副標題可選多種語言，包含烏克蘭語與俄語西里爾字母版面。
- 主題以 `.passthm` 檔案匯入匯出。

### PosterBoard 桌布（.tendies）
- 直接從「檔案」App 匯入並解開 `.tendies` 桌布封存檔。
- 自動辨識 PosterBoard 桌布容器與目前生效的描述元 UUID。
- 將桌布設定與資源注入 PosterBoard 儲存區。
- 刷入後自動觸發 NeoSpring 重新載入桌面，無需重新啟動 iPhone 即可套用桌布。

### 裝置內配對
- 透過 Bonjour 在本地廣播，使手機可與自身配對：設定 > 隱私權與安全性 > 開發人員模式 > 與 AirCard-iOS 配對。
- 自動讀取並同步配對記錄到 `aircard_pairing.plist`。
- 配對完成後不再需要電腦或任何外部連線。

## 環境需求

1. **iOS 27+**：目前的漏洞利用與路徑針對 iOS 27.0 及以上。
2. **LocalDevVPN**：需執行於回環模式（`10.7.0.1` 或 `127.0.0.1`），使本地連線能存取裝置內部服務。
3. **開發人員模式配對**：在 設定 > 隱私權與安全性 > 開發人員模式 > 與 AirCard-iOS 中直接配對，或把既有的配對 plist 放入 App 的文稿目錄。

## 安裝

使用你慣用的側載方式安裝 `AirCard-iOS.ipa`：

- SideStore 或 AltStore
- TrollStore
- LiveContainer
- Xcode 或 iOS App Signer

## 從原始碼建置

### 環境需求
- macOS 14.0 或更新版本，搭配 Xcode 16 或更新版本
- XcodeGen（`brew install xcodegen`）
- Rust 工具鏈（僅在需要重新編譯 `rust-core` 時用到）

### 建置 IPA
```bash
git clone https://github.com/Francis-G-Liu/AirCard-iOS-l10n.git
cd AirCard-iOS-l10n
./build-ipa.sh
```

產生的安裝檔位於 `build/AirCard-iOS.ipa`。

### 重新建置 Rust 框架
若要編譯 `rust-core` 中的改動：
```bash
./build-ios.sh
```

## 儲存庫結構

```
AirCard-iOS-l10n/
├── ios-app/                    # SwiftUI 應用程式
│   ├── AirCardApp.swift        # App 進入點與生命週期
│   ├── AppViewModel.swift      # 狀態管理與漏洞利用協調
│   ├── ContentView.swift       # 主畫面檢視
│   ├── TendiesView.swift       # PosterBoard 桌布檢視
│   ├── TendiesEngine.swift     # Tendies 解開與注入邏輯
│   ├── RespringHelper.swift    # NeoSpring WebKit 重新載入桌面實作
│   ├── Models.swift            # 圖片切片、主題版面、封存封裝
│   ├── PairingController.swift # Bonjour 主機與配對同步
│   ├── NetworkStatus.swift     # VPN 回環偵測
│   ├── Utilities.swift         # 背景保活與輔助函式
│   ├── GrappaHelper.[h,m]      # ATC 協定輔助
│   ├── Info.plist              # Bundle 設定
│   ├── zh-Hans.lproj/          # 簡體中文本地化（本儲存庫新增）
│   ├── zh-Hant.lproj/          # 繁體中文本地化（本儲存庫新增）
│   ├── ja.lproj/               # 日本語ローカライズ（本儲存庫新增）
│   ├── ko.lproj/               # 한국어 현지화（本儲存庫新增）
│   ├── ru.lproj/               # Русская локализация（本儲存庫新增）
│   └── Assets.xcassets/        # 應用程式圖示與圖片資源
├── AirliftFFI.xcframework/     # 已編譯的 arm64 Rust 靜態函式庫與標頭檔
├── rust-core/                  # Rust 核心原始碼
├── project.yml                 # XcodeGen 專案定義
├── build-ipa.sh                # IPA 建置指令碼
├── build-ios.sh                # Rust 框架建置指令碼
├── LICENSE                     # MIT 授權條款
├── README.md                   # 專案說明（簡體中文，預設）
├── README.zh-Hant.md           # 專案說明（繁體中文）
├── README.en.md                # 專案說明（English）
├── README.ja.md                # 專案說明（日本語）
├── README.ko.md                # 專案說明（한국어）
└── README.ru.md                # 專案說明（Русский）
```

## 致謝

### 上游專案

本儲存庫所有功能均來自上游，功勞歸屬如下（與上游 README 一致）：

- **[@mak5er](https://github.com/mak5er)**：主要開發者，UI、密碼主題、Tendies 引擎、裝置內配對。
- **[@merybist](https://github.com/merybist)**：最初的移植基礎。
- **[AirLift](https://github.com/0xjohnnydev/airlift)**，作者 **[0xjohnny (@0xjohnnydev)](https://github.com/0xjohnnydev)**：`AirliftFFI` 所依賴的 AirTraffic 與 ATAirlock 沙箱逃逸研究。
- **[NeoSpring](https://github.com/rooootdev/neospring)**：Swift 實作由 **[@skadz108](https://github.com/skadz108)** 與 **[@rooootdev](https://github.com/rooootdev)** 完成，WebKit GPU 行程重新載入桌面技術由 **[@neonmodder123](https://github.com/neonmodder123)** 提供。
- 專案概念源自 **AirCard** 專案。

### 本儲存庫

- **多語言適配**：由 [@Francis-G-Liu](https://github.com/Francis-G-Liu) 在上游基礎上新增簡體中文、繁體中文、日語、韓語、俄語支援。

## 支持上游作者

以下捐贈管道**均屬於上游專案作者 [@mak5er](https://github.com/mak5er)**，與本儲存庫維護者無關。本儲存庫不接收任何形式的捐贈；如果你希望支持這個專案的持續開發，請把支持給到上游。

- **PayPal**：[Donate via PayPal](https://www.paypal.com/donate/?hosted_button_id=98QRTC2HFRA4Y)
- **TON**：`UQBm9KPhtMw-XVVjirUoa09wzrlyWsbeZhKfefl1Uw-qNZ-r`
- **USDT (TRC20)**：`TDkDMCyjYxgvkWUnQiF5Erk2RyPQMT6G1n`
- **USDT / BNB (BEP20)**：`0x0954dc491c502849d04956ef74634aa5931a08e8`

## 授權條款

MIT License，詳見 [LICENSE](LICENSE)。原版權宣告 `Copyright (c) 2026 Johnny Franks` 依據 MIT 條款原樣保留。