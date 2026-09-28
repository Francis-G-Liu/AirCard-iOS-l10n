# AirCard-iOS 多言語対応版

> **本リポジトリは [Mak5er/AirCard-iOS](https://github.com/Mak5er/AirCard-iOS) の多言語対応版（フォーク）であり、オリジナルではありません。**
>
> 変更したのはローカライズ関連のみです。オリジナルの英語をベースに、**簡体字中国語、繁体字中国語、日本語、韓国語、ロシア語**の 5 言語を追加しました。機能は上流と同一で、ビジネスロジックには一切手を加えていません。オリジナルの機能・設計・実装はすべて上流の作者によるものです。正式な情報源は [上流リポジトリ](https://github.com/Mak5er/AirCard-iOS) を参照してください。

[简体中文](README.md) | [繁體中文](README.zh-Hant.md) | [English](README.en.md) | [日本語](README.ja.md) | [한국어](README.ko.md) | [Русский](README.ru.md)

<p align="center">
  <img src="ios-app/Assets.xcassets/AppIcon.appiconset/AppIcon.png" width="128" height="128" alt="AirCard-iOS Icon" style="border-radius: 28px; box-shadow: 0 8px 24px rgba(0,0,0,0.18);" />
</p>

<p align="center">
  iOS 27+ 上で Apple Wallet のカードスキン、ロック画面のパスコードキーパッドのテーマ、PosterBoard 壁紙を直接カスタマイズできます。
</p>

<p align="center">
  <img src="https://img.shields.io/badge/Platform-iOS%2027+-blue?style=flat-square&logo=apple" alt="Platform" />
  <img src="https://img.shields.io/badge/Swift-5.0-orange?style=flat-square&logo=swift" alt="Swift" />
  <img src="https://img.shields.io/badge/Rust-FFI%20Core-red?style=flat-square&logo=rust" alt="Rust" />
  <img src="https://img.shields.io/badge/License-MIT-green?style=flat-square" alt="License" />
  <img src="https://img.shields.io/badge/Localizations-5%20languages-brightgreen?style=flat-square" alt="Localizations" />
  <img src="https://img.shields.io/badge/Fork%20of-Mak5er%2FAirCard--iOS-lightgrey?style=flat-square" alt="Fork of Mak5er/AirCard-iOS" />
</p>

## 本リポジトリについて

本リポジトリは上流の `Mak5er/AirCard-iOS` からフォークしたもので、**ローカライズの変更のみ**を含みます。それ以外のすべて（機能、UI、Rust コア、エクスプロイトの実装）は上流由来です。

### 本リポジトリの上流に対する変更

- **5 言語を追加**：簡体字中国語、繁体字中国語、日本語、韓国語、ロシア語。英語は基準言語として残し、未翻訳の文言は自動的に英語の原文へフォールバックします。
- **ローカライズの範囲**：UI の文言、エラーおよび状態の通知、システム権限の説明（ローカルネットワーク / 写真）、ならびに `CFBundleLocalizations` に登録する言語リスト。
- **診断ログは英語のまま**：`Activity Log`、`Flash Log`、syslog などのパネルの文言は一切ローカライズしません。上流と突き合わせて問題を調査しやすくするためです。
- **必要最小限のコード変更**：従来「状態文字列の照合で UI の色を判定していた」ロジックを言語非依存のフラグに変更しました。そのままではローカライズによってペアリングおよびスキャンの状態表示が壊れるためです。また列挙型に `displayName` を追加し、`rawValue` は安定した永続化識別子として維持しています。
- あわせて LocalDevVPN ボタンの挙動を、該当 App が未インストールの場合に App Store のダウンロードページへ遷移するよう変更しました。

### ローカライズの実装方法

iOS の標準的な手法に従い、追加の依存関係は導入していません。

- 言語リソースは `ios-app/<言語>.lproj/` 配下の `Localizable.strings` と `InfoPlist.strings` に配置します。
- **キーは英語の原文そのもの**（natural-language keys）であるため、`en.lproj` を保守する必要はありません。未翻訳のキーは自動的に英語の原文へフォールバックします。
- プレースホルダ `%@` / `%lld` / `%d` / `%.1fx` は各言語ファイルでもそのまま保持します。
- 言語はシステム設定に追従して切り替わり、App 内に手動切り替え機能はありません。
- `AirCard-iOS.xcodeproj` は XcodeGen が `project.yml` に基づいて生成します。XcodeGen は `.lproj` ディレクトリを自動認識して variant group を構成するため、プロジェクトを再生成してもローカライズは失われません。

### 上流の変更だけを取り込む方法

```bash
git remote add upstream https://github.com/Mak5er/AirCard-iOS.git
git fetch upstream
git merge upstream/main
```

## 概要

AirCard-iOS は、ジェイルブレイクなしでデバイス上の Apple Wallet のカード画像、ロック画面のパスコードキーパッド、ロック画面の壁紙をカスタマイズできます。

App は LocalDevVPN が提供するローカルループバックトンネル（`10.7.0.1` または `127.0.0.1`）を通じてシステム内部のサービスと通信します。ファイル操作は `AirliftFFI` が担当し、これは AirTraffic サービスとやり取りする Rust ライブラリです。

> **互換性**：AirCard-iOS は現在 **iOS 27.0 以降（iOS 27+）** を必要とします。

## 機能

### Apple Wallet カードスキン
- カスタムのカード画像を Passbook キャッシュ（`cardBackgroundCombined@3x.png`、`@2x.png`、および Suica などの交通系カード向けの `cardBackgroundCombined.pdf`）へ書き込みます。
- 表面とサムネイルのキャッシュを更新し、Wallet を開いた時点で新しい画像が即座に反映されるようにします。
- Apple Pay の起動時にカード識別子をリアルタイムで認識します。
- 1 枚のカードに適用することも、認識済みのすべてのカードへ一括で書き込むこともできます。

### パスコードキーパッドのテーマ
- リアルタイムのキーパッドプレビューに対応し、タッチによる移動とズームでのフレーミングが可能です。
- 10 個すべてのキーを覆う全面ポスター配置にも、個々の円形キーを切り抜く個別配置にも対応します。
- システムのキーパッドキャッシュ（`TelephonyUI-10`）を対象とします。
- 数字のサブタイトルは複数言語から選択でき、ウクライナ語とロシア語のキリル文字レイアウトも含みます。
- テーマは `.passthm` ファイルとして読み込み・書き出しができます。

### PosterBoard 壁紙（.tendies）
- 「ファイル」App から `.tendies` 壁紙アーカイブを直接読み込んで展開します。
- PosterBoard の壁紙コンテナと、現在有効な記述子 UUID を自動認識します。
- 壁紙の設定とリソースを PosterBoard のストレージへ注入します。
- 書き込み後に NeoSpring respring を自動的に実行し、iPhone を再起動せずに壁紙を適用できます。

### デバイス内ペアリング
- Bonjour でローカルにブロードキャストし、端末が自身とペアリングできるようにします：設定 > プライバシーとセキュリティ > デベロッパモード > AirCard-iOS とペアリング。
- ペアリング記録を自動的に読み取り、`aircard_pairing.plist` へ同期します。
- ペアリング完了後は、パソコンや外部接続は一切不要です。

## 動作要件

1. **iOS 27+**：現在のエクスプロイトとパスは iOS 27.0 以降を対象としています。
2. **LocalDevVPN**：ループバックモード（`10.7.0.1` または `127.0.0.1`）で動作している必要があります。これによりローカル接続からデバイス内部のサービスへアクセスできます。
3. **デベロッパモードでのペアリング**：設定 > プライバシーとセキュリティ > デベロッパモード > AirCard-iOS とペアリング で直接ペアリングするか、既存の pairing plist を App の書類ディレクトリへ配置してください。

## インストール

普段お使いのサイドロード方法で `AirCard-iOS.ipa` をインストールします：

- SideStore または AltStore
- TrollStore
- LiveContainer
- Xcode または iOS App Signer

## ソースからのビルド

### 動作要件
- macOS 14.0 以降、および Xcode 16 以降
- XcodeGen（`brew install xcodegen`）
- Rust ツールチェーン（`rust-core` を再コンパイルする場合のみ使用）

### IPA のビルド
```bash
git clone https://github.com/Francis-G-Liu/AirCard-iOS-l10n.git
cd AirCard-iOS-l10n
./build-ipa.sh
```

生成されるインストールパッケージは `build/AirCard-iOS.ipa` にあります。

### Rust フレームワークの再ビルド
`rust-core` の変更をコンパイルする場合：
```bash
./build-ios.sh
```

## リポジトリ構成

```
AirCard-iOS-l10n/
├── ios-app/                    # SwiftUI アプリ
│   ├── AirCardApp.swift        # App のエントリポイントとライフサイクル
│   ├── AppViewModel.swift      # 状態管理とエクスプロイトのオーケストレーション
│   ├── ContentView.swift       # メイン画面のビュー
│   ├── TendiesView.swift       # PosterBoard 壁紙のビュー
│   ├── TendiesEngine.swift     # Tendies の展開と注入のロジック
│   ├── RespringHelper.swift    # NeoSpring WebKit respring の実装
│   ├── Models.swift            # 画像の分割、テーマレイアウト、アーカイブのパッケージング
│   ├── PairingController.swift # Bonjour ホストとペアリングの同期
│   ├── NetworkStatus.swift     # VPN ループバックの検出
│   ├── Utilities.swift         # バックグラウンド維持と補助関数
│   ├── GrappaHelper.[h,m]      # ATC プロトコルの補助
│   ├── Info.plist              # Bundle の構成
│   ├── zh-Hans.lproj/          # 簡体字中国語ローカライズ（本リポジトリで追加）
│   ├── zh-Hant.lproj/          # 繁体字中国語ローカライズ（本リポジトリで追加）
│   ├── ja.lproj/               # 日本語ローカライズ（本リポジトリで追加）
│   ├── ko.lproj/               # 韓国語ローカライズ（本リポジトリで追加）
│   ├── ru.lproj/               # ロシア語ローカライズ（本リポジトリで追加）
│   └── Assets.xcassets/        # アプリアイコンと画像リソース
├── AirliftFFI.xcframework/     # コンパイル済みの arm64 Rust 静的ライブラリとヘッダー
├── rust-core/                  # Rust コアのソースコード
├── project.yml                 # XcodeGen のプロジェクト定義
├── build-ipa.sh                # IPA ビルドスクリプト
├── build-ios.sh                # Rust フレームワークのビルドスクリプト
├── LICENSE                     # MIT ライセンス
├── README.md                   # プロジェクト説明（簡体字中国語、デフォルト）
├── README.zh-Hant.md           # プロジェクト説明（繁体字中国語）
├── README.en.md                # プロジェクト説明（English）
├── README.ja.md                # プロジェクト説明（日本語）
├── README.ko.md                # プロジェクト説明（한국어）
└── README.ru.md                # プロジェクト説明（Русский）
```

## クレジット

### 上流プロジェクト

本リポジトリのすべての機能は上流由来であり、功績は以下のとおりです（上流 README と同一）：

- **[@mak5er](https://github.com/mak5er)**：主要開発者。UI、パスコードテーマ、Tendies エンジン、デバイス内ペアリング。
- **[@merybist](https://github.com/merybist)**：当初の移植の基盤。
- **[AirLift](https://github.com/0xjohnnydev/airlift)**、作者 **[0xjohnny (@0xjohnnydev)](https://github.com/0xjohnnydev)**：`AirliftFFI` が依存する AirTraffic と ATAirlock のサンドボックスエスケープの研究。
- **[NeoSpring](https://github.com/rooootdev/neospring)**：Swift 実装は **[@skadz108](https://github.com/skadz108)** と **[@rooootdev](https://github.com/rooootdev)** が担当し、WebKit GPU プロセスによる respring 技術は **[@neonmodder123](https://github.com/neonmodder123)** が提供。
- プロジェクトのコンセプトは **AirCard** プロジェクトに由来します。

### 本リポジトリ

- **多言語対応**：[@Francis-G-Liu](https://github.com/Francis-G-Liu) が上流をベースに簡体字中国語、繁体字中国語、日本語、韓国語、ロシア語のサポートを追加しました。

## プロジェクトの支援

### 上流作者の支援

AirCard-iOS のすべての機能は上流に由来します。このプロジェクトの継続的な開発を支援したい場合は、上流リポジトリ **[Mak5er/AirCard-iOS](https://github.com/Mak5er/AirCard-iOS)** へお願いします。支援方法は上流の README をご覧ください。本リポジトリは寄付を受け付けていません。

### 本リポジトリ（日本語化版）の支援

支援方法は現在のところ未設定です。

## ライセンス

MIT License、詳細は [LICENSE](LICENSE) を参照してください。原著作権表示 `Copyright (c) 2026 Johnny Franks` は MIT 条項に従い原文のまま保持します。
