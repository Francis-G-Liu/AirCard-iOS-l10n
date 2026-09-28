# AirCard-iOS 다국어 지원 버전

> **이 저장소는 [Mak5er/AirCard-iOS](https://github.com/Mak5er/AirCard-iOS)의 다국어 지원 버전(포크)이며, 원본 프로젝트가 아닙니다.**
>
> 저는 현지화 관련 변경만 수행했습니다. 원본 영어를 기반으로 **중국어 간체, 중국어 번체, 일본어, 한국어, 러시아어** 다섯 개 언어를 추가했습니다. 기능은 업스트림과 동일하며 비즈니스 로직은 전혀 수정하지 않았습니다. 원본의 기능, 디자인, 구현은 모두 업스트림 작성자의 공로이므로, [업스트림 저장소](https://github.com/Mak5er/AirCard-iOS)를 기준으로 삼아 주세요.

[简体中文](README.md) | [繁體中文](README.zh-Hant.md) | [English](README.en.md) | [日本語](README.ja.md) | [한국어](README.ko.md) | [Русский](README.ru.md)

<p align="center">
  <img src="ios-app/Assets.xcassets/AppIcon.appiconset/AppIcon.png" width="128" height="128" alt="AirCard-iOS Icon" style="border-radius: 28px; box-shadow: 0 8px 24px rgba(0,0,0,0.18);" />
</p>

<p align="center">
  iOS 27+에서 Apple Wallet 카드 배경, 잠금 화면 암호 키패드 테마, PosterBoard 배경화면을 직접 커스터마이즈합니다.
</p>

<p align="center">
  <img src="https://img.shields.io/badge/Platform-iOS%2027+-blue?style=flat-square&logo=apple" alt="Platform" />
  <img src="https://img.shields.io/badge/Swift-5.0-orange?style=flat-square&logo=swift" alt="Swift" />
  <img src="https://img.shields.io/badge/Rust-FFI%20Core-red?style=flat-square&logo=rust" alt="Rust" />
  <img src="https://img.shields.io/badge/License-MIT-green?style=flat-square" alt="License" />
  <img src="https://img.shields.io/badge/Localizations-5%20languages-brightgreen?style=flat-square" alt="Localizations" />
  <img src="https://img.shields.io/badge/Fork%20of-Mak5er%2FAirCard--iOS-lightgrey?style=flat-square" alt="Fork of Mak5er/AirCard-iOS" />
</p>

## 이 저장소에 대하여

이 저장소는 업스트림 `Mak5er/AirCard-iOS`에서 포크되었으며 **현지화 변경만 포함**합니다. 그 외의 모든 것(기능, UI, Rust 코어, 익스플로잇 구현)은 업스트림에서 비롯됩니다.

### 업스트림 대비 본 저장소의 변경 사항

- **5개 언어 추가**: 중국어 간체, 중국어 번체, 일본어, 한국어, 러시아어. 영어는 기준 언어로 유지되며, 번역되지 않은 문자열은 자동으로 영어 원문으로 폴백됩니다.
- **현지화 범위**: 인터페이스 문자열, 오류 및 상태 안내, 시스템 권한 설명(로컬 네트워크 / 사진), 그리고 `CFBundleLocalizations`에 등록된 언어 목록.
- **진단 로그는 영어 유지**: `Activity Log`, `Flash Log`, syslog 등 패널의 문자열은 일절 현지화하지 않아 업스트림과 대조해 문제를 추적하기 쉽게 했습니다.
- **소수의 필수 코드 조정**: 기존에 "상태 문자열 매칭으로 UI 색상을 판단"하던 로직을 언어 독립적인 플래그로 변경했습니다. 그렇지 않으면 현지화가 페어링 및 스캔 상태 표시를 깨뜨립니다. 열거형에 `displayName`을 추가했고 `rawValue`는 안정적인 영속 식별자로 유지했습니다.
- 겸사겸사 LocalDevVPN 버튼을 수정해, 해당 앱이 설치되어 있지 않으면 App Store 다운로드 페이지로 이동하도록 했습니다.

### 현지화 구현 방식

iOS 표준 방식을 따르며 추가 의존성을 도입하지 않습니다:

- 언어 리소스는 `ios-app/<언어>.lproj/` 아래의 `Localizable.strings`와 `InfoPlist.strings`에 위치합니다.
- **키가 곧 영어 원문**(natural-language keys)이므로 `en.lproj`를 유지할 필요가 없습니다. 번역되지 않은 키는 자동으로 영어 원문으로 폴백됩니다.
- 자리 표시자 `%@` / `%lld` / `%d` / `%.1fx`는 각 언어 파일에서 그대로 유지됩니다.
- 언어는 시스템 설정을 따라 전환되며, 앱 내에 수동 전환기는 제공하지 않습니다.
- `AirCard-iOS.xcodeproj`는 XcodeGen이 `project.yml`을 기반으로 생성합니다. XcodeGen은 `.lproj` 디렉터리를 자동으로 인식해 variant group을 만들기 때문에, 프로젝트를 다시 생성해도 현지화가 유실되지 않습니다.

### 업스트림 변경 사항만 가져오는 방법

```bash
git remote add upstream https://github.com/Mak5er/AirCard-iOS.git
git fetch upstream
git merge upstream/main
```

## 개요

AirCard-iOS는 탈옥 없이 기기에서 Apple Wallet 카드 이미지, 잠금 화면 암호 다이얼, 잠금 화면 배경화면을 커스터마이즈할 수 있습니다.

앱은 LocalDevVPN이 제공하는 로컬 루프백 터널(`10.7.0.1` 또는 `127.0.0.1`)을 통해 시스템 내부 서비스와 통신합니다. 파일 작업은 `AirliftFFI`가 처리하며, 이는 AirTraffic 서비스와 상호작용하는 Rust 라이브러리입니다.

> **호환성**: AirCard-iOS는 현재 **iOS 27.0 이상(iOS 27+)**을 요구합니다.

## 기능

### Apple Wallet 카드 배경
- 사용자 정의 카드 이미지를 Passbook 캐시에 기록합니다(`cardBackgroundCombined@3x.png`, `@2x.png`, 그리고 Suica 등 교통카드를 위한 `cardBackgroundCombined.pdf`).
- 앞면과 썸네일 캐시를 갱신해 Wallet을 열 때 새 이미지가 즉시 반영되도록 합니다.
- Apple Pay 호출 시 카드 식별자를 실시간으로 인식합니다.
- 개별 카드에 이미지를 적용하거나, 인식된 모든 카드에 일괄 적용할 수 있습니다.

### 암호 다이얼 테마
- 실시간 다이얼 미리보기, 터치 이동과 확대/축소 프레이밍을 지원합니다.
- 열 개 키 전체를 덮는 전면 포스터 레이아웃과 개별 원형 키의 독립 오려내기를 모두 지원합니다.
- 시스템 다이얼 캐시(`TelephonyUI-10`)를 대상으로 합니다.
- 숫자 부제목은 우크라이나어와 러시아어 키릴 문자 레이아웃을 포함한 여러 언어를 선택할 수 있습니다.
- 테마는 `.passthm` 파일로 가져오기/내보내기 합니다.

### PosterBoard 배경화면 (.tendies)
- "파일" 앱에서 `.tendies` 배경화면 아카이브를 직접 가져와 압축을 해제합니다.
- PosterBoard 배경화면 컨테이너와 현재 적용 중인 디스크립터 UUID를 자동으로 식별합니다.
- 배경화면 구성과 리소스를 PosterBoard 저장소에 주입합니다.
- 적용 후 자동으로 NeoSpring 리스프링을 트리거하여, iPhone을 재부팅할 필요 없이 배경화면을 적용합니다.

### 기기 내 페어링
- Bonjour를 통해 로컬로 브로드캐스트하여 휴대폰이 자기 자신과 페어링할 수 있게 합니다: 설정 > 개인정보 보호 및 보안 > 개발자 모드 > AirCard-iOS와 페어링.
- 페어링 기록을 자동으로 읽어 `aircard_pairing.plist`에 동기화합니다.
- 페어링이 완료되면 더 이상 컴퓨터나 외부 연결이 필요하지 않습니다.

## 요구 사항

1. **iOS 27+**: 현재 익스플로잇과 경로는 iOS 27.0 이상을 대상으로 합니다.
2. **LocalDevVPN**: 루프백 모드(`10.7.0.1` 또는 `127.0.0.1`)로 실행되어야 하며, 로컬 연결이 기기 내부 서비스에 접근할 수 있게 합니다.
3. **개발자 모드 페어링**: 설정 > 개인정보 보호 및 보안 > 개발자 모드 > AirCard-iOS에서 직접 페어링하거나, 기존 pairing plist를 앱의 도큐멘트 디렉터리에 넣습니다.

## 설치

익숙한 사이드로딩 방식으로 `AirCard-iOS.ipa`를 설치합니다:

- SideStore 또는 AltStore
- TrollStore
- LiveContainer
- Xcode 또는 iOS App Signer

## 소스에서 빌드

### 요구 사항
- macOS 14.0 이상, Xcode 16 이상
- XcodeGen(`brew install xcodegen`)
- Rust 툴체인(`rust-core`를 다시 컴파일해야 할 때만 사용)

### IPA 빌드
```bash
git clone https://github.com/Francis-G-Liu/AirCard-iOS-l10n.git
cd AirCard-iOS-l10n
./build-ipa.sh
```

생성된 설치 패키지는 `build/AirCard-iOS.ipa`에 위치합니다.

### Rust 프레임워크 다시 빌드
`rust-core`의 변경 사항을 컴파일하려면:
```bash
./build-ios.sh
```

## 저장소 구조

```
AirCard-iOS-l10n/
├── ios-app/                    # SwiftUI 앱
│   ├── AirCardApp.swift        # 앱 진입점 및 라이프사이클
│   ├── AppViewModel.swift      # 상태 관리 및 익스플로잇 오케스트레이션
│   ├── ContentView.swift       # 메인 화면 뷰
│   ├── TendiesView.swift       # PosterBoard 배경화면 뷰
│   ├── TendiesEngine.swift     # Tendies 압축 해제 및 주입 로직
│   ├── RespringHelper.swift    # NeoSpring WebKit 리스프링 구현
│   ├── Models.swift            # 이미지 슬라이싱, 테마 레이아웃, 아카이브 패키징
│   ├── PairingController.swift # Bonjour 호스트 및 페어링 동기화
│   ├── NetworkStatus.swift     # VPN 루프백 감지
│   ├── Utilities.swift         # 백그라운드 유지 및 보조 함수
│   ├── GrappaHelper.[h,m]      # ATC 프로토콜 보조
│   ├── Info.plist              # Bundle 구성
│   ├── zh-Hans.lproj/          # 중국어 간체 현지화 (본 저장소에서 추가)
│   ├── zh-Hant.lproj/          # 중국어 번체 현지화 (본 저장소에서 추가)
│   ├── ja.lproj/               # 일본어 현지화 (본 저장소에서 추가)
│   ├── ko.lproj/               # 한국어 현지화 (본 저장소에서 추가)
│   ├── ru.lproj/               # 러시아어 현지화 (본 저장소에서 추가)
│   └── Assets.xcassets/        # 앱 아이콘 및 이미지 리소스
├── AirliftFFI.xcframework/     # 컴파일된 arm64 Rust 정적 라이브러리 및 헤더
├── rust-core/                  # Rust 코어 소스
├── project.yml                 # XcodeGen 프로젝트 정의
├── build-ipa.sh                # IPA 빌드 스크립트
├── build-ios.sh                # Rust 프레임워크 빌드 스크립트
├── LICENSE                     # MIT 라이선스
├── README.md                   # 프로젝트 설명 (중국어 간체, 기본)
├── README.zh-Hant.md           # 프로젝트 설명 (중국어 번체)
├── README.en.md                # 프로젝트 설명 (영어)
├── README.ja.md                # 프로젝트 설명 (일본어)
├── README.ko.md                # 프로젝트 설명 (한국어)
└── README.ru.md                # 프로젝트 설명 (러시아어)
```

## 크레딧

### 업스트림 프로젝트

이 저장소의 모든 기능은 업스트림에서 비롯되었으며, 공로는 다음과 같습니다(업스트림 README와 동일):

- **[@mak5er](https://github.com/mak5er)**: 주요 개발자, UI, 암호 테마, Tendies 엔진, 기기 내 페어링.
- **[@merybist](https://github.com/merybist)**: 최초의 포팅 기반.
- **[AirLift](https://github.com/0xjohnnydev/airlift)**, 저자 **[0xjohnny (@0xjohnnydev)](https://github.com/0xjohnnydev)**: `AirliftFFI`가 의존하는 AirTraffic 및 ATAirlock 샌드박스 탈출 연구.
- **[NeoSpring](https://github.com/rooootdev/neospring)**: Swift 구현은 **[@skadz108](https://github.com/skadz108)**과 **[@rooootdev](https://github.com/rooootdev)**가 완성했으며, WebKit GPU 프로세스 리스프링 기술은 **[@neonmodder123](https://github.com/neonmodder123)**이 제공했습니다.
- 프로젝트 개념은 **AirCard** 프로젝트에서 비롯되었습니다.

### 본 저장소

- **다국어 지원**: [@Francis-G-Liu](https://github.com/Francis-G-Liu)가 업스트림을 기반으로 중국어 간체, 중국어 번체, 일본어, 한국어, 러시아어 지원을 추가했습니다.

## 업스트림 작성자 후원

아래 기부 채널은 **모두 업스트림 프로젝트 작성자 [@mak5er](https://github.com/mak5er)의 것이며**, 본 저장소 관리자와는 무관합니다. 본 저장소는 어떤 형태의 기부도 받지 않습니다. 이 프로젝트의 지속적인 개발을 지원하고 싶으시다면 후원을 업스트림에 전해 주세요.

- **PayPal**: [Donate via PayPal](https://www.paypal.com/donate/?hosted_button_id=98QRTC2HFRA4Y)
- **TON**: `UQBm9KPhtMw-XVVjirUoa09wzrlyWsbeZhKfefl1Uw-qNZ-r`
- **USDT (TRC20)**: `TDkDMCyjYxgvkWUnQiF5Erk2RyPQMT6G1n`
- **USDT / BNB (BEP20)**: `0x0954dc491c502849d04956ef74634aa5931a08e8`

## 라이선스

MIT License, 자세한 내용은 [LICENSE](LICENSE)를 참고하세요. 원 저작권 표시 `Copyright (c) 2026 Johnny Franks`는 MIT 조건에 따라 원문 그대로 유지됩니다.