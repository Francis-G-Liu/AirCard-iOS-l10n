# AirCard-iOS 多语言适配版

> **本仓库是 [Mak5er/AirCard-iOS](https://github.com/Mak5er/AirCard-iOS) 的多语言适配版本（fork），并非原版项目。**
>
> 我只做了本地化相关的改动：在原版英文基础上新增**简体中文、繁体中文、日语、韩语、俄语**五种语言。功能与上游保持一致，未改动任何业务逻辑。原版功能、设计与实现全部归功于上游作者，请以 [上游仓库](https://github.com/Mak5er/AirCard-iOS) 为准。

[简体中文](README.md) | [繁體中文](README.zh-Hant.md) | [English](README.en.md) | [日本語](README.ja.md) | [한국어](README.ko.md) | [Русский](README.ru.md)

<p align="center">
  <img src="ios-app/Assets.xcassets/AppIcon.appiconset/AppIcon.png" width="128" height="128" alt="AirCard-iOS Icon" style="border-radius: 28px; box-shadow: 0 8px 24px rgba(0,0,0,0.18);" />
</p>

<p align="center">
  直接在 iOS 27+ 上定制 Apple Wallet 卡面皮肤、锁屏密码键盘主题与 PosterBoard 壁纸。
</p>

<p align="center">
  <img src="https://img.shields.io/badge/Platform-iOS%2027+-blue?style=flat-square&logo=apple" alt="Platform" />
  <img src="https://img.shields.io/badge/Swift-5.0-orange?style=flat-square&logo=swift" alt="Swift" />
  <img src="https://img.shields.io/badge/Rust-FFI%20Core-red?style=flat-square&logo=rust" alt="Rust" />
  <img src="https://img.shields.io/badge/License-MIT-green?style=flat-square" alt="License" />
  <img src="https://img.shields.io/badge/Localizations-5%20languages-brightgreen?style=flat-square" alt="Localizations" />
  <img src="https://img.shields.io/badge/Fork%20of-Mak5er%2FAirCard--iOS-lightgrey?style=flat-square" alt="Fork of Mak5er/AirCard-iOS" />
</p>

## 关于本仓库

本仓库从上游 `Mak5er/AirCard-iOS` fork 而来，**只包含本地化改动**。除此以外的一切（功能、UI、Rust 核心、漏洞利用实现）都来自上游。

### 本仓库相对上游的改动

- **新增 5 种语言**：简体中文、繁体中文、日语、韩语、俄语。英文作为基准语言保留，未翻译的文案自动回退为英文原文。
- **本地化范围**：界面文案、错误与状态提示、系统权限说明（本地网络 / 照片），以及 `CFBundleLocalizations` 中注册的语言列表。
- **诊断日志保持英文**：`Activity Log`、`Flash Log`、syslog 等面板的文案一律不本地化，便于对照上游排查问题。
- **少量必要的代码调整**：把原先「靠匹配状态字符串判断 UI 颜色」的逻辑改为语言无关的标志位，否则本地化会破坏配对与扫描状态的显示；枚举新增 `displayName`，`rawValue` 保持为稳定的持久化标识。
- 顺带把 LocalDevVPN 按钮改为：未安装该 App 时跳转 App Store 下载页。

### 本地化实现方式

沿用 iOS 标准做法，不引入额外依赖：

- 语言资源放在 `ios-app/<语言>.lproj/` 下的 `Localizable.strings` 与 `InfoPlist.strings`。
- **键即英文原文**（natural-language keys），因此不需要维护 `en.lproj`：任何未翻译的键都会自动回退成英文原文。
- 占位符 `%@` / `%lld` / `%d` / `%.1fx` 在各语言文件中原样保留。
- 语言跟随系统设置切换，App 内不提供手动切换器。
- `AirCard-iOS.xcodeproj` 由 XcodeGen 依据 `project.yml` 生成；XcodeGen 会自动识别 `.lproj` 目录并建立 variant group，所以重新生成工程后本地化不会丢失。

### 如何只取上游改动

```bash
git remote add upstream https://github.com/Mak5er/AirCard-iOS.git
git fetch upstream
git merge upstream/main
```

## 概览

AirCard-iOS 无需越狱即可在设备上定制 Apple Wallet 卡面图案、锁屏密码拨号盘与锁屏壁纸。

App 通过 LocalDevVPN 提供的本地回环隧道（`10.7.0.1` 或 `127.0.0.1`）与系统内部服务通信。文件操作由 `AirliftFFI` 处理，这是一个与 AirTraffic 服务交互的 Rust 库。

> **兼容性**：AirCard-iOS 目前要求 **iOS 27.0 或更高版本（iOS 27+）**。

## 功能

### Apple Wallet 卡面皮肤
- 将自定义卡面图案写入 Passbook 缓存（`cardBackgroundCombined@3x.png`、`@2x.png`，以及面向 Suica 等交通卡的 `cardBackgroundCombined.pdf`）。
- 刷新正面与缩略图缓存，使新图案在 Wallet 打开时立即生效。
- 唤起 Apple Pay 时实时识别卡片标识符。
- 可对单张卡片应用图案，也可批量刷入所有已识别卡片。

### 密码拨号盘主题
- 实时拨号盘预览，支持触摸平移与缩放取景。
- 支持覆盖全部十个按键的整版海报布局，也支持单个圆形按键的独立抠图。
- 面向系统拨号盘缓存（`TelephonyUI-10`）。
- 数字副标题可选多种语言，包含乌克兰语与俄语西里尔字母布局。
- 主题以 `.passthm` 文件导入导出。

### PosterBoard 壁纸（.tendies）
- 直接从「文件」App 导入并解包 `.tendies` 壁纸归档。
- 自动识别 PosterBoard 壁纸容器与当前生效的描述符 UUID。
- 将壁纸配置与资源注入 PosterBoard 存储。
- 刷入后自动触发 NeoSpring respring，无需重启 iPhone 即可应用壁纸。

### 设备内配对
- 通过 Bonjour 在本地广播，使手机可与自身配对：设置 > 隐私与安全性 > 开发者模式 > 与 AirCard-iOS 配对。
- 自动读取并同步配对记录到 `aircard_pairing.plist`。
- 配对完成后不再需要电脑或任何外部连接。

## 环境要求

1. **iOS 27+**：当前的漏洞利用与路径针对 iOS 27.0 及以上。
2. **LocalDevVPN**：需运行在回环模式（`10.7.0.1` 或 `127.0.0.1`），使本地连接能访问设备内部服务。
3. **开发者模式配对**：在 设置 > 隐私与安全性 > 开发者模式 > 与 AirCard-iOS 中直接配对，或把已有的 pairing plist 放入 App 的文稿目录。

## 安装

使用你惯用的侧载方式安装 `AirCard-iOS.ipa`：

- SideStore 或 AltStore
- TrollStore
- LiveContainer
- Xcode 或 iOS App Signer

## 从源码构建

### 环境要求
- macOS 14.0 或更新版本，搭配 Xcode 16 或更新版本
- XcodeGen（`brew install xcodegen`）
- Rust 工具链（仅在需要重新编译 `rust-core` 时用到）

### 构建 IPA
```bash
git clone https://github.com/Francis-G-Liu/AirCard-iOS-l10n.git
cd AirCard-iOS-l10n
./build-ipa.sh
```

生成的安装包位于 `build/AirCard-iOS.ipa`。

### 重新构建 Rust 框架
若要编译 `rust-core` 中的改动：
```bash
./build-ios.sh
```

## 仓库结构

```
AirCard-iOS-l10n/
├── ios-app/                    # SwiftUI 应用
│   ├── AirCardApp.swift        # App 入口与生命周期
│   ├── AppViewModel.swift      # 状态管理与漏洞利用编排
│   ├── ContentView.swift       # 主界面视图
│   ├── TendiesView.swift       # PosterBoard 壁纸视图
│   ├── TendiesEngine.swift     # Tendies 解包与注入逻辑
│   ├── RespringHelper.swift    # NeoSpring WebKit respring 实现
│   ├── Models.swift            # 图片切片、主题布局、归档打包
│   ├── PairingController.swift # Bonjour 主机与配对同步
│   ├── NetworkStatus.swift     # VPN 回环检测
│   ├── Utilities.swift         # 后台保活与辅助函数
│   ├── GrappaHelper.[h,m]      # ATC 协议辅助
│   ├── Info.plist              # Bundle 配置
│   ├── zh-Hans.lproj/          # 简体中文本地化（本仓库新增）
│   ├── zh-Hant.lproj/          # 繁體中文本地化（本仓库新增）
│   ├── ja.lproj/               # 日本語ローカライズ（本仓库新增）
│   ├── ko.lproj/               # 한국어 현지화（本仓库新增）
│   ├── ru.lproj/               # Русская локализация（本仓库新增）
│   └── Assets.xcassets/        # 应用图标与图片资源
├── AirliftFFI.xcframework/     # 已编译的 arm64 Rust 静态库与头文件
├── rust-core/                  # Rust 核心源码
├── project.yml                 # XcodeGen 工程定义
├── build-ipa.sh                # IPA 构建脚本
├── build-ios.sh                # Rust 框架构建脚本
├── LICENSE                     # MIT 许可证
├── README.md                   # 项目说明（简体中文，默认）
├── README.zh-Hant.md           # 项目说明（繁體中文）
├── README.en.md                # 项目说明（English）
├── README.ja.md                # 项目说明（日本語）
├── README.ko.md                # 项目说明（한국어）
└── README.ru.md                # 项目说明（Русский）
```

## 致谢

### 上游项目

本仓库所有功能均来自上游，功劳归属如下（与上游 README 一致）：

- **[@mak5er](https://github.com/mak5er)**：主要开发者，UI、密码主题、Tendies 引擎、设备内配对。
- **[@merybist](https://github.com/merybist)**：最初的移植基础。
- **[AirLift](https://github.com/0xjohnnydev/airlift)**，作者 **[0xjohnny (@0xjohnnydev)](https://github.com/0xjohnnydev)**：`AirliftFFI` 所依赖的 AirTraffic 与 ATAirlock 沙箱逃逸研究。
- **[NeoSpring](https://github.com/rooootdev/neospring)**：Swift 实现由 **[@skadz108](https://github.com/skadz108)** 与 **[@rooootdev](https://github.com/rooootdev)** 完成，WebKit GPU 进程 respring 技术由 **[@neonmodder123](https://github.com/neonmodder123)** 提供。
- 项目概念源自 **AirCard** 项目。

### 本仓库

- **多语言适配**：由 [@Francis-G-Liu](https://github.com/Francis-G-Liu) 在上游基础上添加简体中文、繁体中文、日语、韩语、俄语支持。

## 支持上游作者

以下捐赠渠道**均属于上游项目作者 [@mak5er](https://github.com/mak5er)**，与本仓库维护者无关。本仓库不接收任何形式的捐赠；如果你希望支持这个项目的持续开发，请把支持给到上游。

- **PayPal**：[Donate via PayPal](https://www.paypal.com/donate/?hosted_button_id=98QRTC2HFRA4Y)
- **TON**：`UQBm9KPhtMw-XVVjirUoa09wzrlyWsbeZhKfefl1Uw-qNZ-r`
- **USDT (TRC20)**：`TDkDMCyjYxgvkWUnQiF5Erk2RyPQMT6G1n`
- **USDT / BNB (BEP20)**：`0x0954dc491c502849d04956ef74634aa5931a08e8`

## 许可证

MIT License，详见 [LICENSE](LICENSE)。原版权声明 `Copyright (c) 2026 Johnny Franks` 依据 MIT 条款原样保留。