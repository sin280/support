# Flutter バージョンアップ差分まとめ（v3.38.6 → v3.47.8）

## 概要・前提情報

- **対象範囲**: Flutter `v3.38.6`（2026年1月8日リリース / Dart `3.10.7`）〜 最新 Stable `v3.47.8`（2026年10月23日リリース / Dart `3.13.7`）
  - ※2026年10月29日時点の最新 Stable バージョンは **`v3.47.8`** です（次期メジャーの `v3.50.0` は Beta チャンネル段階）。
- **Flutter Stable のバージョン番号規則について**:
  - Flutter の四半期リリースサイクルでは、間のマイナー番号（`3.39` / `3.40`、`3.42` / `3.43`、`3.45` / `3.46`）は `master` / `beta` チャンネルでの開発・プレリリース番号として使用され、**Stable チャンネルでは `3.38.x` → `3.41.x` → `3.44.x` → `3.47.x` の順に移行**します。
- **参照した公式ドキュメント**:
  - 各バージョンの詳細 Changelog: <https://github.com/flutter/flutter/blob/master/CHANGELOG.md>
  - 破壊的変更・非推奨一覧（Breaking Changes）: <https://docs.flutter.dev/release/breaking-changes>
  - Stable リリースノート一覧: <https://docs.flutter.dev/release/release-notes>

---

## クイックリファレンス（v3.38.6 から最新へ上げる際の最重要チェック表）

### 1. 廃止（削除）・非推奨クラス/API と代替手段一覧

| 導入バージョン | 区分 | 旧API・クラス（廃止 / 非推奨） | 新API・クラス（代替手段・移行先） | 自動修正 (`dart fix`) |
| :--- | :--- | :--- | :--- | :--- |
| **v3.41.0** | 非推奨 | `FontWeight.index` | `FontWeight.value`（`1`〜`1000` の整数値を返す） | 一部対応 |
| **v3.41.0** | 非推奨 | `containsSemantics(...)` (`flutter_test`) | 部分一致は `isSemantics(...)`、完全一致は `matchesSemantics(...)` | 対応 |
| **v3.41.0** | 非推奨 | `ListView.separated` / `SliverList.separated` の `findChildIndexCallback` | `findItemIndexCallback`（※セパレータ分を考慮した `* 2` が不要になり純粋なアイテム添字を返す） | 手動確認推奨 |
| **v3.44.0** | 破壊的変更 | `IconData` の継承・実装 (`extends` / `implements IconData`) | `final` クラス化されたため、`final IconData` をフィールドに持つラッパークラスに変更 | 不可（要手動改修） |
| **v3.44.0** | パッケージ移動 | `material.dart` 経由での `CupertinoPageTransitionsBuilder` 参照 | `import 'package:flutter/cupertino.dart';` を明示的に追加 | 対応 |
| **v3.44.0** | 非推奨 | `ReorderableListView` / `ReorderableList` / `SliverReorderableList` の `onReorder` | `onReorderItem`（※手動の `if (oldIndex < newIndex) newIndex -= 1;` 補正を削除すること） | 手動確認推奨 |
| **v3.44.0** | 非推奨 | `TextInputConnection.setStyle(...)` | `TextInputConnection.updateStyle(TextInputStyle(...))` | 不可（要手動改修） |
| **v3.44.0** | 非推奨 | `ListView` / `ScrollView` / `Viewport` 等の `cacheExtent`・`cacheExtentStyle` | `scrollCacheExtent` プロパティ + `ScrollCacheExtent.pixels(...)` / `ScrollCacheExtent.viewport(...)` | 対応 |
| **v3.47.0** | **完全削除** | `describeEnum(enumEntry)` (`foundation.dart`) | 標準 Enum の `.name` プロパティ（例: `MyEnum.hoge.name`） | 対応 |
| **v3.47.0** | 型制約厳格化 | `EnumProperty<T>` に Enum 以外の型を指定 | `EnumProperty<T extends Enum?>` に制限。Enum 以外は `DiagnosticsProperty<T>` を使用 | 不可（要手動改修） |
| **v3.47.0** | 動作無効化 | `Semantics(header: true)`（iOS / Android で無効化・no-op） | `Semantics(headingLevel: 1)`（`1`〜`6` 等の正の整数を指定） | 不可（要手動改修） |
| **v3.47.2** | 非推奨 | 旧来のカスタム Analyzer プラグイン機構 (`analyzer_plugin`) | 新しい Dart Analysis Server プラグイン機構へ移行 | 不可 |

### 2. 設定ファイル・ネイティブ環境の書き換え必須項目一覧

| 導入バージョン | 対象OS | 変更内容 | 書き換えが必要なファイル・対応 |
| :--- | :--- | :--- | :--- |
| **v3.41.0** | iOS | **`UIScene` (`UISceneDelegate`) ライフサイクルがデフォルト化** | 未編集ならビルド時に自動移行。`AppDelegate` / `Info.plist` をカスタマイズしている場合は `UIApplicationSceneManifest` の追加と `FlutterImplicitEngineDelegate.didInitializeImplicitFlutterEngine` への移行が必須 |
| **v3.44.0** | iOS / macOS | **Swift Package Manager (SwiftPM) がデフォルト化** & **Xcode 15以上必須（16+推奨）** | 初回ビルド時に自動移行（未対応プラグインがある場合は CocoaPods に自動フォールバック）。問題発生時は `pubspec.yaml` で一時無効化可能 |
| **v3.44.0** | Android | **Built-in Kotlin 移行準備** & **Kotlin Gradle Plugin (KGP) `2.0.0` 以上必須** | AGP 9.0+ 利用時は `build.gradle(.kts)` から `kotlin-android` プラグイン指定を削除し `kotlin.compilerOptions` へ移行 |
| **v3.47.0** | iOS | **最小サポートOSが `iOS 13.0` → `iOS 15.0` に引き上げ** | `ios/Podfile`（`platform :ios, '15.0'`）、`project.pbxproj`（`IPHONEOS_DEPLOYMENT_TARGET = 15.0`）、`AppFrameworkInfo.plist` を更新 |
| **v3.47.0** | macOS | **最小サポートOSが `macOS 10.15` → `macOS 12.0` に引き上げ** | `macos/Podfile`（`platform :osx, '12.0'`）、`project.pbxproj` / `AppInfo.xcconfig`（`MACOSX_DEPLOYMENT_TARGET = 12.0`）を更新 |
| **v3.47.0** | Desktop全般 | **macOS / Windows / Linux で Impeller がデフォルト有効化** | 描画不具合がある場合のみ各プラットフォーム設定または `--no-enable-impeller` で一時無効化 |

---

## 各バージョンごとの差分詳細

---

## 1. Flutter 3.38 系パッチリリース（v3.38.6 → v3.38.10）

> ベースライン: `v3.38.6`（2026-01-08 リリース / Dart `3.10.7`）

### v3.38.6 → v3.38.7：
*リリース日: 2026-01-14 / 同梱 Dart SDK: `3.10.7`*

- **CLI 不具合修正 (`flutter/179857`)**:
  - 複数のターゲットデバイスが接続・利用可能な状態で `flutter run -d all` を実行した際にクラッシュする問題を修正。

### v3.38.7 → v3.38.8：
*リリース日: 2026-01-27 / 同梱 Dart SDK: `3.10.7`*

- **Web 実行時のクラッシュ修正 (`flutter/178151`)**:
  - `flutter run -d chrome` の実行中、起動シーケンスの途中でアプリが終了した場合に `DartDevelopmentServiceException` が発生してクラッシュする問題を修正。

### v3.38.8 → v3.38.9：
*リリース日: 2026-01-29 / 同梱 Dart SDK: `3.10.7` → `3.10.8`*

- **Dart SDK の更新 (`flutter/181568`)**:
  - 同梱される Dart SDK を **`3.10.8`** にアップデート（ランタイムおよびツールチェーンの安定性修正）。

### v3.38.9 → v3.38.10：
*リリース日: 2026-02-11 / 同梱 Dart SDK: `3.10.8` → `3.10.9`（※3.38系の最終パッチ）*

- **macOS FFI プラグインのビルド構造修正 (`flutter/181607`)**:
  - macOS で FFI プラグインを使用した際、生成される `.framework` バンドルのディレクトリ構造が不正になる問題を修正。
- **Dart SDK の更新 (`3.10.9` / `dart-lang/sdk#62504`)**:
  - Dot shorthands（ドット省略記法）を用いた際に、他ライブラリのプライベート宣言にアクセスできてしまっていた不具合を修正。

---

## 2. Flutter 3.41 系リリース（v3.38.10 → v3.41.9）

### v3.38.10 → v3.41.0：
*リリース日: 2026-02-11 / 同梱 Dart SDK: `3.10.9` → `3.11.0`（四半期メジャーアップデート）*

#### 【廃止・非推奨になったクラス・API（コード書き換え対象）】
- **`FontWeight.index` が非推奨になり、`FontWeight.value` を使うようになった**
  - バリアブルフォント向けに `1`〜`1000` の任意の太さ（`FontWeight`）を生成できるようになったため、従来の `w100`〜`w900` に対応する固定添字（`0`〜`8`）を返す `FontWeight.index` が非推奨化。代わりに `400` や `700` などの実数値を返す **`FontWeight.value`** を使用する。
- **`containsSemantics`（`flutter_test`）が非推奨になり、`isSemantics` を使うようになった**
  - テストコードで Semantics の部分検証を行うマッチャー `containsSemantics(...)` が非推奨化。指定した引数のみを検証する場合は **`isSemantics(...)`**、全プロパティを厳密検証する場合は **`matchesSemantics(...)`** に書き換える（`dart fix --apply` 対応）。
- **`ListView.separated` / `SliverList.separated` の `findChildIndexCallback` が非推奨になり、`findItemIndexCallback` を使うようになった**
  - `.separated` コンストラクタにおいて、従来の `findChildIndexCallback` はセパレータウィジェットを含めた内部レンダーインデックス（`itemIndex * 2`）を返す必要がありバグの温床になっていたため非推奨化。
  - 新設された **`findItemIndexCallback`** では、`* 2` の計算を行わず**純粋なアイテムのインデックス（`0` 〜 `itemCount - 1`）をそのまま返す**ように書き換える（※`ListView.builder` 等の `findChildIndexCallback` は引き続き非推奨ではない）。
  ```dart
  // Before
  ListView.separated(
    findChildIndexCallback: (Key key) {
      final index = items.indexWhere((e) => ValueKey(e.id) == key);
      return index == -1 ? null : index * 2;
    },
    // ...
  )

  // After
  ListView.separated(
    findItemIndexCallback: (Key key) {
      final index = items.indexWhere((e) => ValueKey(e.id) == key);
      return index == -1 ? null : index; // * 2 が不要
    },
    // ...
  )
  ```

#### 【破壊的変更・動作変更（動かなくなる・挙動が変わる可能性がある箇所）】
- **バリアブルフォントに対する `FontWeight` の動作変更**
  - `TextStyle(fontWeight: ...)` を指定した際、バリアブルフォント（可変フォント）の OpenType `wght` 軸も自動的に連動して制御されるようになった。従来 `fontWeight` が効いていなかった可変フォントの太さが変わる場合があるため、ゴールデンテストの差分や重複した `FontVariation('wght', ...)` 指定を確認すること。
- **Material 3 カラー生成アルゴリズムの更新（Material 3 tokens update）**
  - 内部の `material_color_utilities` が `0.11.1` → `0.13.0` に更新され、`ColorScheme.fromSeed` や `ColorScheme.fromImageProvider` で動的生成される以下の4色のトーンが変更された：
    - `onPrimaryContainer` / `onSecondaryContainer` / `onTertiaryContainer` / `onErrorContainer`
  - 特定の色コードに依存するテストやデザインがある場合は `copyWith` で明示指定する。
- **Linux における UI スレッドとプラットフォームスレッドの統合（Merged threads on Linux）**
  - iOS / Android / macOS / Windows に続き、Linux でも Dart UI スレッドとネイティブ GTK メインスレッドがデフォルトで統合された。通常の Dart コードは変更不要だが、別スレッド前提でロック待ちをする独自 Linux ネイティブプラグインがある場合はデッドロックに注意。

#### 【書き換えが必要な設定・ネイティブコード】
- **iOS で `UIScene`（`UISceneDelegate`）ライフサイクルがデフォルトで有効化**
  - Apple の次期 Xcode / iOS での `UIScene` 必須化に合わせ、デフォルトで有効化された。
  - `AppDelegate` や `Info.plist` を初期状態から変更していない場合は `flutter run` / `flutter build ios` 実行時に自動マイグレーションされる。
  - **ネイティブコードをカスタマイズしている場合は手動書き換えが必須**：
    1. `application(_:didFinishLaunchingWithOptions:)` 実行時点では `window?.rootViewController`（`FlutterViewController`）が未生成になるため、そこでの `GeneratedPluginRegistrant.register(with: self)` や `MethodChannel` 登録はクラッシュ・失敗する。
    2. `AppDelegate` に `FlutterImplicitEngineDelegate` プロトコルを適合させ、`didInitializeImplicitFlutterEngine(_ engineBridge: FlutterImplicitEngineBridge)` 内で `GeneratedPluginRegistrant.register(with: engineBridge.pluginRegistry)` およびチャンネル初期化を行うよう書き換える。
    3. `ios/Runner/Info.plist` に `UIApplicationSceneManifest`（`FlutterSceneDelegate` 指定）を追加する。
    4. `applicationDidBecomeActive` 等のライフサイクル処理を書いている場合は `SceneDelegate` 側へ移行する。
- **Android Gradle Plugin (AGP) 9.0 を先行導入する場合の設定（Add-to-App 等）**
  - 手動で AGP 9.0 以上へ上げている Add-to-App ホストプロジェクト等では、`gradle.properties` に `android.builtInKotlin=false` および `android.newDsl=false` の一時指定が必要（標準の Flutter CLI ビルドは自動付与）。

#### 【v3.41.0 の主な新機能・改善サマリー（Release Notes）】
- **UI / フレームワーク**: ループアニメーションを宣言的に書ける `RepeatingAnimationBuilder`、複数ルートを戻り値付きでポップする `Navigator.popUntilWithResult`、`CupertinoSheet` のドラッグハンドル（`showDragHandle`）、`ColorFilter` の彩度調整機能を追加。
- **プラットフォーム別アセットバンドル**: `pubspec.yaml` の `assets` セクションで `platforms: [android, ios, ...]` を指定し、特定OS専用の重いアセットを他OSのビルドから除外可能に。
- **エンジン / Impeller**: `BackdropFilter` の端の色滲みを防ぐ Bounded Blur、同期ピクセルデコード `ui.decodeImageFromPixelsSync`、128bit 浮動小数点テクスチャ（`PixelFormat.rgbaFloat32`）サポート。
- **Add-to-App / Web / Tooling**: iOS / Android のネイティブ画面内でコンテンツ高さに自動追従する FlutterView、DevTools の `dart2wasm` デフォルト化による高速化、Widget Previewer への Flutter Inspector 組み込み。

---

### v3.41.0 → v3.41.1：
*リリース日: 2026-02-13 / 同梱 Dart SDK: `3.11.0`*

- **テストカバレッジ取得の不具合修正 (`flutter/182314`)**:
  - `flutter_test` 内で固定されていた `test_api` バージョン制約により、`flutter test --coverage`（特に Pub Workspace 環境）が正常に動作しなかった問題を修正。
- **テスト出力の改行によるテスト失敗の修正 (`flutter/182335`)**:
  - `test` パッケージ `1.29` が出力に余分な空行を挿入したことで意図しないテスト失敗が発生していた問題を修正。

### v3.41.1 → v3.41.2：
*リリース日: 2026-02-19 / 同梱 Dart SDK: `3.11.0`*

- **Android PlatformView の描画競合修正 (`flutter/179673`)**:
  - Android でコンテンツ自動サイズ調整（content sizing）が無効な場合に、まれに PlatformView（WebView や Map 等）が正しく描画されない競合状態を修正。
- **Web ビルドの `--web-define` フラグ修正 (`flutter/182076`, `flutter/182243`)**:
  - `flutter build web` 実行時に `--web-define` が無視されていた問題、および変数が未指定の場合に例外が発生していた問題を修正。
- **Web `MultiSurfaceRenderer` のレイアウト崩れ修正 (`flutter/182292`)**:
  - Web のキャンバス要素に `position: absolute` が欠落してレイアウトが崩れる問題を修正。

### v3.41.2 → v3.41.3：
*リリース日: 2026-03-02 / 同梱 Dart SDK: `3.11.0` → `3.11.1`*

- **Windows アプリ待機時の CPU 使用率削減 (`flutter/182501`)**:
  - アイドル状態の Flutter Windows デスクトップアプリで CPU 使用率が高止まりしていた問題を大幅に改善。
- **iOS ステータスバータップ時のクラッシュ修正 (`flutter/182233`)**:
  - Primary ScrollView が一度もレイアウトされていない状態でステータスバーをタップ（最上部スクロール発動）した際にクラッシュする問題を修正。
- **Dart SDK の更新**: **`3.11.1`** へアップデート。

### v3.41.3 → v3.41.4：
*リリース日: 2026-03-04 / 同梱 Dart SDK: `3.11.1`*

- **Xcode 26 での iOS シミュレータビルド失敗の修正 (`flutter/182748`)**:
  - `arm`（`arm64` シミュレータ）アーキテクチャ非対応の CocoaPods 依存関係が含まれるプロジェクトを Xcode 26 でシミュレータビルドした際に失敗する問題を修正。
- **iOS プラグインのライフサイクル登録クラッシュ修正 (`flutter/182361`)**:
  - ライフサイクルイベントの処理途中に iOS プラグインがイベント購読登録を行った際にクラッシュする可能性があった問題を修正。
- **Flutter Web (`skwasm`) のクラッシュ修正 (`flutter/182367`)**:
  - `skwasm` レンダラーを使用している Web アプリでの実行時クラッシュを修正。
- **テスト関連パッケージの更新 (`flutter/183071`)**:
  - 内部の `test` パッケージおよび関連依存関係を更新。

### v3.41.4 → v3.41.5：
*リリース日: 2026-03-18 / 同梱 Dart SDK: `3.11.1` → `3.11.3`*

- **Impeller での円描画の滲み・アーティファクト修正 (`flutter/182708`)**:
  - 全プラットフォームの Impeller において、45度回転させた円などを描画した際に滲みや視覚的な乱れが生じる問題を修正。
- **Dart SDK の更新 (`3.11.3`)**:
  - `skwasm` 環境での DevTools クラッシュ修正（`flutter/devtools#9701`）を含む Dart `3.11.3` に更新。

### v3.41.5 → v3.41.6：
*リリース日: 2026-03-26 / 同梱 Dart SDK: `3.11.3` → `3.11.4`*

- **Android 16 セキュリティパッチ適用端末でのデッドロック・ANR 修正 (`flutter/183887`)**:
  - 2026年3月セキュリティアップデート適用済みの Android 16 端末において、画面消灯（`SCREEN_OFF`）イベント時にデッドロックが発生してフレーム更新が停止し、ANR（応答なし）になる重大な不具合を修正。
- **Impeller の細い円・小さい円のジャギー修正 (`flutter/182708`)**:
  - 細い枠線の円や半径の小さい円がギザギザに表示される問題を追加修正。
- **Skia フォントグリフ描画の修正 (`flutter/184025`)**:
  - グリフマスク形式に対応するアトラスが一貫して選択されるよう Skia 側の修正を取り込み。
- **Dart SDK の更新 (`3.11.4`)**:
  - Dot-shorthand の関数式呼び出しで Analyzer がクラッシュする不具合（`dart-lang/sdk#62595`）を修正。

### v3.41.6 → v3.41.7：
*リリース日: 2026-04-16 / 同梱 Dart SDK: `3.11.4` → `3.11.5`*

- **iOS / macOS ビルド時の Git `multi-pack-index` エラー修正 (`flutter/184376`)**:
  - Git の `multi-pack-index` エラーに起因して iOS / macOS アプリのビルドが失敗する問題を修正。
- **Xcode 26.4+ での iOS 実機デバッグ中のクラッシュ修正 (`flutter/184254`)**:
  - Xcode 26.4 以降を用いて iOS 実機でデバッグ実行した際にアプリが頻繁にクラッシュする問題を修正。
- **Windows FFI パッケージの要件緩和 (`flutter/184689`)**:
  - Windows 向け FFI パッケージ利用時に不要なプラグインクラス実装を要求しないよう修正。
- **Dart SDK の更新 (`3.11.5`)**:
  - Dart MCP サーバー連携の修正（`dart-lang/ai#439`）を含む更新。

### v3.41.7 → v3.41.8：
*リリース日: 2026-04-27 / 同梱 Dart SDK: `3.11.5`*

- **iOS 実機の Profile モード接続不良の修正 (`flutter/185150`)**:
  - iOS 実機で Profile モード実行時に Dart VM への接続に失敗することがある問題を修正。

### v3.41.8 → v3.41.9：
*リリース日: 2026-04-30 / 同梱 Dart SDK: `3.11.5`（※3.41系の最終パッチ）*

- **APNG（アニメーションPNG）の整数オーバーフロー修正 (`flutter/185621`)**:
  - 特定の APNG ファイルをデコード・描画する際に整数オーバーフローが発生してクラッシュする脆弱性・不具合を修正。

---

## 3. Flutter 3.44 系リリース（v3.41.9 → v3.44.9）

### v3.41.9 → v3.44.0：
*リリース日: 2026-05-18 / 同梱 Dart SDK: `3.11.5` → `3.12.0`（四半期メジャーアップデート）*

#### 【廃止・非推奨になったクラス・API（コード書き換え対象）】
- **`ReorderableListView` / `ReorderableList` / `SliverReorderableList` の `onReorder` が非推奨になり、`onReorderItem` を使うようになった**
  - 従来の `onReorder` は削除前のインデックス基準だったため、後方に移動する際 `if (oldIndex < newIndex) newIndex -= 1;` という手動補正が必要だった。
  - 新しい **`onReorderItem`** では**補正済みの挿入先インデックスが直接渡される**ため、コールバック名を書き換えると同時に **`newIndex -= 1` の手動減算を必ず削除する**（残すと1つズレるため注意）。
  ```dart
  // Before
  ReorderableListView(
    onReorder: (int oldIndex, int newIndex) {
      if (oldIndex < newIndex) newIndex -= 1;
      final item = items.removeAt(oldIndex);
      items.insert(newIndex, item);
    },
    children: children,
  )

  // After
  ReorderableListView(
    onReorderItem: (int oldIndex, int newIndex) {
      final item = items.removeAt(oldIndex);
      items.insert(newIndex, item);
    },
    children: children,
  )
  ```
- **`TextInputConnection.setStyle` が非推奨になり、`TextInputConnection.updateStyle(TextInputStyle)` を使うようになった**
  - カスタムテキスト入力実装（`TextInputClient`）で IME の変換候補位置やキャレット位置をより正確に同期させるため、従来の `setStyle(...)` が非推奨化。`letterSpacing`・`wordSpacing`・`lineHeight` も指定可能な **`TextInputStyle`** クラスを渡す **`updateStyle(...)`** に書き換える。
- **スクロール系ウィジェットの `cacheExtent` と `cacheExtentStyle` が非推奨になり、`scrollCacheExtent`（`ScrollCacheExtent`）を使うようになった**
  - `ScrollView` / `ListView` / `GridView` / `CustomScrollView` / `Viewport` / `ShrinkWrappingViewport` / `RenderViewport` 等で別々に指定していた2つのプロパティが統合された。
  - ピクセル指定は **`ScrollCacheExtent.pixels(500.0)`**、ビューポート倍率指定は **`ScrollCacheExtent.viewport(2.0)`** を `scrollCacheExtent:` に渡すよう書き換える。

#### 【破壊的変更・動作変更（動かなくなる・挙動が変わる可能性がある箇所）】
- **`IconData` クラスが `final` 化され、継承・実装がコンパイルエラーになった**
  - パッケージ横断のアイコンツリーシェイキング最適化のため、`IconData` に `final` 修飾子が付与された。
  - `class MyIcon extends IconData` や `enum AppIcons implements IconData` のような実装はコンパイルエラーになるため、`final IconData iconData;` をフィールドとして保持するラッパークラス／enum に書き換える。
- **`CupertinoPageTransitionsBuilder` の所属ライブラリが `material.dart` から `cupertino.dart` へ移動**
  - Material と Cupertino のパッケージ分離準備の一環として、`package:flutter/material.dart` から `CupertinoPageTransitionsBuilder` が除外された。
  - `ThemeData(pageTransitionsTheme: ...)` 等で `CupertinoPageTransitionsBuilder` を使っているファイルでは、明示的に **`import 'package:flutter/cupertino.dart';` を追加**する必要がある。
- **`RawMenuAnchor`（および `MenuAnchor` / `CupertinoMenuAnchor`）を閉じる順序の動作変更**
  - 階層メニュー（ネストされたサブメニュー）を閉じる際、従来は親メニューのコールバックが先に発火する場合があったが、v3.44.0 からは**すべての子孫メニューの `onCloseRequested` / `onClose` が末端から順に発火し終えてから親メニューが発火する（子→親の順）**ように変更された。フォーカス制御やテストのアサーション順序に依存している場合は修正する。
- **背景色付きウィジェットで直接ラップされた `ListTile` がデバッグ時にエラーを投げるように変更**
  - `Container(color: ...)` や `ColoredBox` の直下に `ListTile` を置くと、`ListTile` が直近の `Material` 祖先に描画するリップル効果や `tileColor` が塗り潰されて見えなくなるため、デバッグモードで `FlutterError` が報告されるようになった。
  - **対処法**: `Container` の `color` をやめて `ListTile(tileColor: ...)` を使うか、`ListTile` を `Material` ウィジェットでラップする。
- **Android 17（API 37）以降をターゲットにした際、大画面での画面向き固定・リサイズ制限が無視される動作変更**
  - `targetSdkVersion >= 37` のアプリを最小幅 `600dp` 以上の端末（タブレット、折りたたみ端末、デスクトップウィンドウモード）で実行した場合、`SystemChrome.setPreferredOrientations` や `AndroidManifest.xml` の `screenOrientation` による縦画面固定・リサイズ不可設定が OS 仕様により無視される。大画面でもレイアウトが崩れないレスポンシブ対応が必要。

#### 【書き換えが必要な設定・ビルド環境】
- **iOS / macOS のデフォルトパッケージマネージャーが CocoaPods から Swift Package Manager (SwiftPM) に変更**
  - `flutter run` や `flutter build` を実行すると、既存の Xcode プロジェクトが自動的に SwiftPM 対応へマイグレーションされる（SwiftPM 非対応のプラグインが含まれている場合は自動的に CocoaPods へフォールバックし警告を表示）。
  - 万が一ビルドトラブルが発生して従来の CocoaPods に戻したい場合は、`pubspec.yaml` に以下を設定して一時オプトアウト可能：
    ```yaml
    flutter:
      config:
        enable-swift-package-manager: false
    ```
- **Android プロジェクトの Built-in Kotlin 対応と最小 KGP バージョンの引き上げ**
  - Kotlin Gradle Plugin (KGP) の最小必須バージョンが **`2.0.0`** に引き上げられた。
  - AGP 9.0+ では Built-in Kotlin が標準となるため、`android/app/build.gradle(.kts)` で明示的に `id("kotlin-android")` / `id("org.jetbrains.kotlin.android")` を適用していると AGP 9+ 環境でビルドエラーになる（v3.44 の CLI は既存プロジェクト互換のため一時的に `gradle.properties` へ `android.builtInKotlin=false` と `android.newDsl=false` を補完するが、AGP 9 へ上げる際はプラグイン指定の削除と `kotlin { compilerOptions { ... } }` 記法への移行が必要）。
- **Xcode 要件の引き上げ**:
  - 最小サポートバージョンが **Xcode 15**（推奨 **Xcode 16 以上**）に変更。

#### 【v3.44.0 の主な新機能・改善サマリー（Release Notes）】
- **Android Hybrid Composition++ (HCPP)**: Android API 34+ & Impeller/Vulkan 環境向けに `SurfaceControl` を用いた新しい PlatformView 合成モードを導入し、`WebView` や `GoogleMaps` スクロール時のフレーム遅延やズレを解消。
- **Cupertino / Material 強化**: インタラクティブにドラッグで閉じられる `CupertinoSheetRoute`、`CupertinoMenuAnchor`、`ProgressIndicator` のパーセンテージ読み上げ対応など。
- **AI 開発連携**: AI エージェントから実行中アプリを自動リロードする Agentic Hot Reload、Dart & Flutter MCP サーバー、LLM 出力をネイティブ UI 化する GenUI SDK。
- **Dart `3.12.0` 同梱**: プライベート名前付き引数（`Foo({required this._bar})`）の正式導入、Git LFS 対応など。

---

### v3.44.0 → v3.44.1：
*リリース日: 2026-06-01 / 同梱 Dart SDK: `3.12.0` → `3.12.1`*

- **Analysis Server 異常終了時の CLI クラッシュ修正 (`flutter/186962`)**:
  - Dart Analysis Server のサブプロセスが予期せず終了した際、`flutter` ツールが未処理例外でクラッシュせず適切なエラーメッセージを表示するよう修正。
- **Windows での Chrome 接続失敗時のクラッシュ修正 (`flutter/186963`)**:
  - Windows 環境で Chrome インスタンスへの接続に失敗した際に `flutter` ツールがクラッシュする問題を修正。
- **Dart SDK の更新 (`3.12.1`)**:
  - `await` を跨いだ際に `Zone` オーバーライド（カスタム `print` 等）がリセットされてしまう重大な非同期バグ（`dart-lang/sdk#63408`）、および Windows ARM64 での `dart analyze` 終了時クラッシュ（`dart-lang/sdk#63343`）を修正。

### v3.44.1 → v3.44.2：
*リリース日: 2026-06-11 / 同梱 Dart SDK: `3.12.1` → `3.12.2`*

- **Android Edge-to-Edge 切り替え時のシステムバー非表示バグ修正 (`flutter/186723`)**:
  - Android アプリが `SystemUiMode` から Edge-to-Edge 表示へ遷移した際、システムバーが見えないままになる問題を修正。
- **SwiftPM リモート依存関係のダウンロード失敗修正 (`flutter/186054`)**:
  - SwiftPM 有効時に iOS / macOS アプリをビルドすると、リモートの Swift パッケージ取得に失敗することがある問題を修正。
- **非 Apple ターゲットビルド時の不要な SwiftPM 警告抑止 (`flutter/185775`)**:
  - Android / Windows / Web 等（iOS・macOS 以外）のビルド時に誤って出力されていた SwiftPM の警告を非表示化。
- **Widget Preview の DTD 切断時クラッシュ修正 (`flutter/187322`)**:
  - `widget-preview` 起動時に Dart Tooling Daemon (DTD) への接続失敗・切断が起きても未処理クラッシュせず正常終了するよう修正。
- **Dart SDK の更新 (`3.12.2`)**:
  - `analysis_options.yaml` に存在しないファイルパスが `include` されている場合に `dart format` がクラッシュする問題（`dart_style#1840`）を修正。

### v3.44.2 → v3.44.3：
*リリース日: 2026-06-22 / 同梱 Dart SDK: `3.12.2`*

- **フレーバー・プラットフォーム別アセットの警告ログ削減 (`flutter/186738`)**:
  - 別フレーバーや別OS向けアセットをスキップした際のログレベルを `warning` から `trace` に引き下げ、ビルドログの大量のノイズを解消。
- **Android GLES テクスチャ破棄時のクラッシュ修正 (`flutter/186899`)**:
  - 保持している GLES フェンスが使用される前にテクスチャを破棄するとクラッシュする問題を修正。
- **SwiftPM 並列ビルド時の競合クラッシュ修正 (`flutter/186953`)**:
  - macOS / iOS で SwiftPM を用いて並行ビルドを行った際、ディレクトリやシンボリックリンク生成の競合でクラッシュする問題を修正。
- **APNG 描画時のクラッシュ修正 (`flutter/183179`)**:
  - アニメーション PNG（APNG）の描画時にクラッシュするケースを修正。

### v3.44.3 → v3.44.4：
*リリース日: 2026-06-24 / 同梱 Dart SDK: `3.12.2`*

- **Linux アクセシビリティの境界チェック修正 (`flutter/188192`)**:
  - Flutter Linux の `FlAccessibleTextField` におけるインデックス境界チェックの不備を修正。

### v3.44.4 → v3.44.5：
*リリース日: 2026-07-06 / 同梱 Dart SDK: `3.12.2`*

- **Android App Bundle (`.aab`) に `libapp.so` が含まれない重大バグの修正 (`flutter/186810`, `flutter/187388`, `flutter/187553`)**:
  - フレーバー（Flavors）を使用している場合や、古いアプリテンプレートかつアルファベット順で `app` より前に来るプラグインを使用している場合に、ビルドした `.aab` に `libapp.so` が同梱されず起動不能になる問題を修正。
- **テキストの影（Shadow）が文字から分離して描画される不具合の修正 (`flutter/188035`)**:
  - テキストのドロップシャドウが本来の文字位置から離れた別の場所に描画されてしまうリグレッションを修正。
- **AGP 9 未満環境での誤った Built-in Kotlin 警告の修正 (`flutter/188718`)**:
  - AGP 9 未満 + KGP の構成でビルドした際に、不要な Built-in Kotlin 移行警告が出力されていた問題を修正。

### v3.44.5 → v3.44.6：
*リリース日: 2026-07-09 / 同梱 Dart SDK: `3.12.2`*

- **Linux Native Assets ビルド時のクラッシュ修正 (`flutter/187980`)**:
  - ネイティブビルドディレクトリ（`CMakeCache.txt`）が未作成の状態で Native Assets 有効のアセットバンドルをビルドするとクラッシュする問題を修正。
- **Android Instrumented Test 実行時のクラッシュ修正 (`flutter/188805`)**:
  - Android 実機・エミュレータ結合テスト実行時に発生していたクラッシュを修正。

### v3.44.6 → v3.44.7：
*リリース日: 2026-07-20 / 同梱 Dart SDK: `3.12.2`*

- **特定 GPU（Arm Mali 等）での外部テクスチャのリソースリーク・クラッシュ修正 (`flutter/188161`)**:
  - カメラや動画・WebView 等の外部テクスチャを使用した際、Arm Mali などの一部 GPU でリソースリークからクラッシュに至る問題を修正。

### v3.44.7 → v3.44.8：
*リリース日: 2026-07-23 / 同梱 Dart SDK: `3.12.2`*

- **新しい Xcode ツールチェーン（Xcode 27 等）での `lipo` 検証エラー修正 (`flutter/188346`)**:
  - macOS / iOS ビルド時に `lipo` の検証エラーでビルドが失敗する問題を修正。
- **一部メーカー製 Android 11 端末でのアクセシビリティクラッシュ修正 (`flutter/184286`)**:
  - ベンダー独自のカスタム Android 11 ROM 端末において、`AccessibilityBridge.java` のフィールド解決に失敗してクラッシュする問題を修正。

### v3.44.8 → v3.44.9：
*リリース日: 2026-08-06 / 同梱 Dart SDK: `3.12.2`（※3.44系の最終パッチ）*

- **iOS デバッグ中にアプリがクラッシュした際の CLI フリーズ修正 (`flutter/186366`)**:
  - iOS アプリのデバッグ実行中やテスト中にアプリ側がクラッシュした際、Flutter CLI やテストプロセスが終了せず無限にハングアップする問題を修正。

---

## 4. Flutter 3.47 系リリース（v3.44.9 → v3.47.8：現在最新）

### v3.44.9 → v3.47.0：
*リリース日: 2026-08-12 / 同梱 Dart SDK: `3.12.2` → `3.13.0`（四半期メジャーアップデート）*

#### 【廃止・削除されたクラス・API（コンパイルエラーになるため書き換え必須）】
- **`describeEnum()` 関数が完全に廃止（削除）され、Enum の `.name` を使うようになった**
  - `package:flutter/foundation.dart` のグローバル関数 `describeEnum(Object enumEntry)` が完全に削除された。標準の Enum 拡張プロパティ **`.name`** に書き換える（`dart fix --apply` で一括置換可能）。
  ```dart
  // Before（v3.47.0 でコンパイルエラー）
  final label = describeEnum(MyStatus.active);

  // After
  final label = MyStatus.active.name;
  ```
- **`EnumProperty<T>` の型パラメータが `EnumProperty<T extends Enum?>` に厳格化された**
  - `Enum` を継承していない任意のクラスを `EnumProperty` に渡すとコンパイルエラーになる。`Enum` 以外のオブジェクトをデバッグプロパティに出す場合は **`DiagnosticsProperty<T>`** を使用する。

#### 【破壊的変更・動作変更（動かなくなる・挙動が変わる可能性がある箇所）】
- **iOS / Android における `Semantics` の `header` と `headingLevel` の動作変更**
  - `Semantics(header: true)` や `SemanticsProperties(header: true)` の **`header` プロパティが iOS / Android で何もしない（no-op）ように変更**された（コンパイルエラーにはならないが、VoiceOver や TalkBack で「見出し」として認識されなくなるサイレントな挙動変更）。
  - 見出しとして認識させるには、必ず **`headingLevel: 1`**（`1`〜`6` 等の `0` より大きい整数）に書き換える。
  ```dart
  // Before（iOS/Android のスクリーンリーダーで見出し扱いされなくなる）
  Semantics(
    header: true,
    child: Text('セクション見出し'),
  )

  // After
  Semantics(
    headingLevel: 1,
    child: Text('セクション見出し'),
  )
  ```
- **OpenGL ES における Render-to-Texture の格納向きがトップダウンに統一**
  - Impeller の OpenGL ES バックエンドにおいて、オフスクリーンレンダーターゲットのテクスチャ格納向きが従来のボトムアップから Metal / Vulkan と同じ**トップダウン（上基準）**に統一された。
  - 通常の Flutter ウィジェットには影響しないが、カスタム GLSL シェーダー（`FragmentShader` / `flutter_gpu`）内で `#ifdef IMPELLER_TARGET_OPENGLES` を使って手動で Y 座標を上下反転させていた場合は描画が上下逆さまになるため、手動反転処理を削除する（旧バージョンと両対応する場合は `#ifndef IMPELLER_OPENGLES_UNFLIPPED_DEPRECATED` でガードする）。
- **デスクトップ全プラットフォーム（macOS / Windows / Linux）で Impeller がデフォルト有効化**
  - モバイルに続き、macOS・Windows・Linux でもレンダリングエンジンのデフォルトが Skia から **Impeller** に切り替わった。万が一デスクトップ固有の描画不具合が出た場合は、`--no-enable-impeller` や各ネイティブランナーの設定（`FLTEnableImpeller` 等）で一時的に無効化できる。

#### 【書き換えが必要な設定ファイル・プラットフォーム最小要件の引き上げ】
- **iOS の最小サポート対象バージョンが `iOS 13.0` → `iOS 15.0` に引き上げ**
  - iOS 13 / 14 のサポートが終了したため、以下の設定ファイルを **`15.0` 以上**に書き換える必要がある：
    1. `ios/Podfile`: `platform :ios, '15.0'`（および `post_install` 内で `IPHONEOS_DEPLOYMENT_TARGET` を固定している箇所も `'15.0'` 以上へ更新）
    2. `ios/Runner.xcodeproj/project.pbxproj`: Debug / Profile / Release すべての `IPHONEOS_DEPLOYMENT_TARGET = 15.0;`
    3. `ios/Flutter/AppFrameworkInfo.plist`: `MinimumOSVersion` を `15.0` に更新
- **macOS の最小サポート対象バージョンが `macOS 10.15` → `macOS 12.0` に引き上げ**
  - `macos/Podfile` の `platform :osx, '12.0'`、および Xcode プロジェクト／`AppInfo.xcconfig` の `MACOSX_DEPLOYMENT_TARGET = 12.0` への書き換えが必要（あわせて Intel Mac サポートの段階的縮小も進行）。
- **Android AGP 9.0+ / Built-in Kotlin 対応の徹底**
  - サードパーティプラグインも含め AGP 9+ / Built-in Kotlin 対応が標準化。古いプラグインが原因でビルドエラーになる場合はまず `flutter pub upgrade` でプラグイン側を最新化し、それでも未対応プラグインが残る場合のみ `android/gradle.properties` で `android.builtInKotlin=false` / `android.newDsl=false` を一時使用する。

#### 【v3.47.0 の主な新機能・アーキテクチャ変更サマリー（Release Notes）】
- **Material / Cupertino のスタンドアロンパッケージ化（`material_ui` 1.0 / `cupertino_ui` 1.0 公開）**
  - デザインシステムを Flutter SDK 本体から切り離して週次アップデート可能にする `material_ui` および `cupertino_ui` パッケージ（v1.0）が `pub.dev` に公開された。
  - **※v3.47 時点での扱い**: 既存の `import 'package:flutter/material.dart';` および `cupertino.dart` は**非推奨化されておらず、書き換えなしでもそのまま動作する**。先行して新パッケージへ移行する場合は `flutter pub add material_ui` のうえで `dart fix --apply --code=migrate_design_widgets`（※`v3.47.1` 以降推奨）を実行し、必要に応じて `MaterialUiCompatibilityBridge` を併用する。
- **Flutter Widget Previews の Stable（安定版）昇格**
  - `@Preview()` アノテーションを用いたウィジェットプレビュー機能が実験段階を卒業し正式版に昇格。ローカルビルドキャッシュによる高速起動やリアルタイム検索・フィルタリングに対応。
- **Dart `3.13.0` 同梱**:
  - クラス宣言とコンストラクタを簡潔に記述できる **Primary Constructors（プライマリコンストラクタ）** などの新言語機能に対応。

---

### v3.47.0 → v3.47.1：
*リリース日: 2026-08-19 / 同梱 Dart SDK: `3.13.0` → `3.13.1`*

- **Windows / Linux リリースビルドでの Flutter GPU 有効化修正 (`flutter/190871`)**:
  - プロジェクト単位のオプトイン設定がなく環境変数スイッチがリリースビルドで無視されていたため、Linux / Windows のリリースビルドで Flutter GPU を有効化できなかった問題を修正。
- **iOS / macOS での SwiftPM 並列ビルド競合の修正 (`flutter/188446`)**:
  - マルチターゲット構成のアプリを並列ビルドした際、SwiftPM 統合処理の競合により `FileSystemException` が発生する問題を修正。
- **Dart Workspace 内での `flutter pub get` 挙動修正 (`flutter/189550`)**:
  - Dart Workspace 環境で `flutter pub get` を実行した際、Flutter に直接依存していないパッケージに対して不要なプラットフォームツール再生成や `analysis_options` 移行をスキップするよう修正。
- **Dart SDK の更新 (`3.13.1`)**:
  - `dart fix --apply --code=migrate_design_widgets` が `import` だけでなく `export` 文の URI も正しく書き換えるよう修正（`dart/63968`）。
  - `dart2wasm` における `noSuchMethod` フォワーダや独自 `Type` 実装存在時のコンパイルエラー・クラッシュを修正（`dart/63904`, `dart/63843`）。

### v3.47.1 → v3.47.2：
*リリース日: 2026-08-27 / 同梱 Dart SDK: `3.13.1` → `3.13.2`*

- **【非推奨化】レガシー Analyzer プラグイン機構の非推奨化 (`dart/62164`)**:
  - 旧来の `analyzer_plugin` プロトコルを用いたカスタム静的解析プラグイン機構が非推奨となり、新しい Analysis Server プラグイン機構への移行が推奨されるようになった。
- **SwiftPM 有効時の iOS / macOS ビルドおよび Add-to-App ビルド失敗の修正 (`flutter/188265`, `flutter/190846`)**:
  - SwiftPM 有効時のビルドエラー、および既存のネイティブ iOS アプリに Flutter を組み込む（Add-to-App）構成で新しい Xcode（Xcode 27）を使用した際に Flutter の Swift パッケージビルドが失敗する問題を修正。
- **Web の DDS 起動時 WebSocket エラー処理の修正 (`flutter/191179`)**:
  - Web で DDS 起動中に WebSocket アップグレードが失敗した際、`DartDevelopmentServiceException` でクラッシュせず適切にハンドリングするよう修正。
- **Linux のタッチイベントメモリリークおよびスクリーンリーダー修正 (`flutter/190518`, `flutter/187925`)**:
  - Linux でタッチイベント処理によりメモリリークが発生する問題と、見出し要素がスクリーンリーダーに読み上げられない問題を修正。
- **Dart SDK の更新 (`3.13.2`)**:
  - Primary Constructor を持つ Enum に対する `dart format` のクラッシュ修正（`dart_style#1885`, `#1888`）、`dart2wasm` / `dart2js` のコンパイル不具合修正、Windows での `dart run` 実行ファイル拡張子修正など。

### v3.47.2 → v3.47.3：
*リリース日: 2026-09-09 / 同梱 Dart SDK: `3.13.2` → `3.13.3`*

- **`Actions.handler` が常に `null` を返す重大リグレッションの修正 (`flutter/191045`)**:
  - 全プラットフォームにおいて `Actions.handler` の呼び出しが常に `null` を返してショートカット・アクション委譲が機能しなくなっていたフレームワークの不具合を修正。
- **PowerVR B-Series GPU 搭載 Android 端末での Impeller 描画・性能問題の修正 (`flutter/181315`)**:
  - 該当 GPU を搭載する Android 端末で発生していた描画の乱れとパフォーマンス低下を修正。
- **Xcode 未インストール・不完全環境でのエラーメッセージ改善 (`flutter/191176`)**:
  - Xcode が未導入または不完全なマシンで iOS / macOS ビルドを実行した際、未処理の `ProcessException` でクラッシュする代わりに具体的な対処方法を表示するよう修正。
- **Android SDK Command-line Tools 23.0+ での `flutter doctor` 誤判定修正 (`flutter/191487`)**:
  - cmdline-tools 23.0 以降を使用していると `flutter doctor` が誤って `"Android license status unknown"` と報告する問題を修正。
- **Dart SDK の更新 (`3.13.3` / `dart/63953`)**:
  - Windows から Linux へのクロスコンパイル時にネイティブコードをリンクする Build Hooks を使うと壊れた実行ファイルが生成される問題を修正。

### v3.47.3 → v3.47.4：
*リリース日: 2026-09-11 / 同梱 Dart SDK: `3.13.3`*

- **iOS で Native Assets を同梱したアプリの App Store 提出失敗を修正 (`flutter/191964`)**:
  - デプロイメントターゲットが iOS 13 より上でコンパイルされたネイティブライブラリ（Native Assets）を含む iOS アプリが、App Store Connect への提出（バリデーション）でリジェクトされる問題を修正。
- **Windows Smart App Control ブロック時のエラーハンドリング改善 (`flutter/191899`)**:
  - Windows の「スマート アプリ コントロール」やセキュリティポリシーによりバイナリ実行がブロックされた際、クラッシュせず対処可能なエラーメッセージを表示するよう修正。
- **開発ツールの計測・テレメトリ修正 (`flutter/192120`, `flutter/181560`)**:
  - `LspPreviewDetector` のリロード計測漏れおよび SwiftPM 移行時の警告・エラー計測を修正。

### v3.47.4 → v3.47.5：
*リリース日: 2026-09-18 / 同梱 Dart SDK: `3.13.3` → `3.13.4`*

- **iOS 実機（iOS 27）デバッグ時の断続的なクラッシュ修正 (`flutter/190307`)**:
  - 物理 iOS デバイスでデバッグ実行中にアプリが断続的にクラッシュする問題を修正。
- **Widget Previewer のスクロール復元クラッシュ修正 (`flutter/191242`)**:
  - Widget Previewer でプレビューグループを再展開した際、スクロール位置復元の型エラーでクラッシュする問題を修正。
- **DDS 起動失敗時の `FormatException` クラッシュ修正 (`flutter/189507`)**:
  - Dart Development Service (DDS) の起動失敗時に `flutter_tools` が未処理の `FormatException` でクラッシュする問題を修正。
- **Dart SDK の更新 (`3.13.4` / `dart/64150`)**:
  - `dart2js` で `--merge-fragments-threshold` と `--minify` を併用した際のコンパイラクラッシュを修正。

### v3.47.5 → v3.47.6：
*リリース日: 2026-10-01 / 同梱 Dart SDK: `3.13.4` → `3.13.5`*

- **Windows 本番（Production）アプリがフリーズ・ハングアップする重大不具合の修正 (`flutter/192513`)**:
  - リリースビルドした Windows デスクトップアプリが実行中に応答不能（ハング）になる重大なバグを修正。
- **Dart SDK の更新 (`3.13.5` / `dart/64153`)**:
  - `dart2wasm` のツリーシェイキング（TFA）において、静的 JS interop 呼び出しを含む到達可能なメソッドまで誤って削除してしまっていた不具合を修正。

### v3.47.6 → v3.47.7：
*リリース日: 2026-10-08 / 同梱 Dart SDK: `3.13.5`*

- **iOS で `OverlayPortal` 表示時に VoiceOver / XCUITest のタップ位置がズレる不具合の修正 (`flutter/182604`)**:
  - iOS で `OverlayPortal`（`DropdownMenu` やツールチップ等の内部でも使用）を表示した際、アクセシビリティフレームがデバイスピクセル比の分だけ縮小して画面左上に固定され、VoiceOver のフォーカスや XCUITest の自動テストタップが実際のボタン位置に当たらなくなる問題を修正。
- **iOS の `ReorderableListView` での VoiceOver フォーカス飛び修正 (`flutter/100946`)**:
  - iOS で VoiceOver 使用中に `ReorderableListView` を操作すると、Semantics フォーカスが誤って画面左上へ移動してしまう問題を修正。

### v3.47.7 → v3.47.8：
*リリース日: 2026-10-23 / 同梱 Dart SDK: `3.13.5` → `3.13.7`（※2026年10月29日時点の最新 Stable）*

- **【重要セキュリティ修正】BoringSSL 更新による危殆化した中間 CA 証明書のブロック (`GHSA-frr6-h895-5pxp` / `dart/568000`)**:
  - Dart ランタイム（`3.13.7`）内の BoringSSL を更新し、危殆化した `eMudhra` 中間 CA 証明書をブロック。
  - **Android / Windows / Linux 向けにアプリを配布している場合は、`v3.47.8` へアップデートしてアプリを再ビルド・再配布することが公式に推奨されている。**
- **Android 14+ におけるルート証明書ストアの動作更新 (`dart/64696`)**:
  - Android 14 以降において、`dart:io` がシステム更新可能な **Conscrypt APEX ルート証明書ストア**を参照するように更新。
- **デスクトップ向け `flutter pub get` / ビルド時のシンボリックリンク例外修正 (`flutter/190235`)**:
  - 既存のプラグイン用シンボリックリンクや壊れたリンクが残っている状態で `flutter pub get` やデスクトップビルドを実行した際、`PathExistsException` でクラッシュしていた問題を修正。
- **`flutter run` からのツリーダンプ実行時のクラッシュ修正 (`flutter/192934`)**:
  - UI Isolate の初期化完了前または破棄後に `flutter run` からフォーカスツリーやレイヤーツリーをダンプしようとした際、Null check operator 例外でクラッシュする問題を修正。
- **Dart SDK の更新 (`3.13.6` → `3.13.7` 相当)**:
  - `dart2js` で1回の Dart 関数呼び出しがコンパイル後の JavaScript で複数回呼び出しに展開されてしまうコード生成バグ（`dart/64410`）の修正も包含。

---

## 補足：公式 Breaking Changes ページ閲覧時の注意点（次期 v3.50 予定項目との区別）

公式の [Breaking changes](https://docs.flutter.dev/release/breaking-changes) ページ最上部にある **「Not yet released to stable」** セクションには、まだ `v3.47.8`（Stable）には強制適用されていない**次期 `v3.50` 向けの変更（またはオプトイン段階の変更）**が並んでいます。今回の `v3.47.8` へのバージョンアップ時点での扱いは以下の通りです：

1. **`material_ui` / `cupertino_ui` スタンドアロンパッケージへの移行**:
   - `v3.47` では公開・オプトイン利用開始段階であり、`package:flutter/material.dart` / `cupertino.dart` は削除・非推奨化されていません（既存コードのまま動作します）。
2. **`DropdownButton` への `enabled` プロパティ追加と `onChanged` のオプショナル化**:
   - 次期 `v3.50` での導入項目です（`v3.47.8` 時点では `DropdownButton` は従来のまま `onChanged: null` で無効化します）。
3. **Android リリースモードでのエンジンフラグ制限 / Intent 検証の厳格化 / ソフトハイフン改行描画 / Web `viewport-fit=cover` SafeArea 対応 / RTL テキスト内の `WidgetSpan` 順序修正**:
   - いずれも次期 `v3.50`（現在 Beta）に含まれる変更です。

---

## 推奨アップグレード手順（Full Flutter アプリ向け：最速・最安全ルート）

### 基本方針：「SDK は一気に `v3.47.8` へ上げ、設定変更をフェーズ1〜4に分割する」

Full Flutter アプリを `v3.38.6` から最新へ上げる場合、途中のマイナーバージョン（`3.41` / `3.44`）を順番に踏むとパッケージ互換パズルや複数回のビルド検証でかえって手戻りが発生します。
一方で、すべての変更を一度に適用するとビルドエラーの原因切り分けが困難になり、修復不能な状態に陥るリスクがあります。

そのため、**「Flutter SDK 自体は一気に `v3.47.8` へ上げつつ、ビルドシステムの変更をフラグでロックしてフェーズごとに1つずつ解放・Gitコミットする」**手順を採用します。

> **【重要前提：社内 JFrog 利用（SwiftPM 未対応）環境について】**
> - `v3.44.0` 以降はデフォルトで Swift Package Manager (SwiftPM) が有効になり、何もしないままビルドを実行すると `ios/Runner.xcodeproj` が自動的に SwiftPM 用へ書き換えられ、外部（GitHub 等）への直接通信を試みて JFrog 環境下でビルド不能になります。
> - これを防ぐため、**必ず SDK 更新・ビルド実行【前】に `pubspec.yaml` で `enable-swift-package-manager: false` を明示設定し、今回のアップデートでは一貫して従来の CocoaPods（JFrog 経由）を維持**します（`v3.47.8` でも CocoaPods ビルドは公式に完全動作します）。

---

### フェーズ1：新ビルド機構をオフに固定し、最小限の必須対応だけで `v3.47.8` を起動させる

まずは「SwiftPM」と「Android Built-in Kotlin」を従来構成にロックした状態で、SDK 更新と必須の破壊的変更のみに対応し、アプリがビルド・起動するベースラインを作ります。

1. **作業ブランチの作成とビルド機構の事前ロック（※必ず `flutter upgrade` / `build` 前に実施）**
   - **SwiftPM の無効化（JFrog 環境のため恒久設定）**: `pubspec.yaml` の `flutter:` セクション配下に以下を追加する（CLI コマンドではなく `pubspec.yaml` に書くことで、チーム全員・CI 環境に強制適用される）。
     ```yaml
     flutter:
       config:
         enable-swift-package-manager: false
     ```
   - **Android Built-in Kotlin の一時無効化**: `android/gradle.properties` に以下を一時追加する。
     ```properties
     android.builtInKotlin=false
     android.newDsl=false
     ```
2. **Flutter SDK と依存パッケージの更新**
   ```bash
   flutter channel stable
   flutter upgrade        # v3.47.8 (Dart 3.13.7) へ更新
   flutter pub outdated
   flutter pub upgrade    # 社内 JFrog ミラー経由でパッケージ更新
   ```
3. **コンパイルエラーになる必須の Dart コード修正のみ実施**
   - `describeEnum(xxx)` を使用している箇所を `xxx.name` へ置換する。
   - `IconData` を継承・実装（`extends` / `implements IconData`）している独自クラスがあればラッパークラスへ変更する。
   - `CupertinoPageTransitionsBuilder` を使用しているファイルに `import 'package:flutter/cupertino.dart';` を追加する。
4. **iOS / macOS 最小 OS バージョンの引き上げと `UIScene` 対応**
   - **iOS 15.0 引き上げ**: `ios/Podfile` の `platform :ios, '15.0'`（および `post_install` 内の `IPHONEOS_DEPLOYMENT_TARGET`）、`ios/Runner.xcodeproj/project.pbxproj` の `IPHONEOS_DEPLOYMENT_TARGET = 15.0`、`ios/Flutter/AppFrameworkInfo.plist` の `MinimumOSVersion`（`15.0`）を更新する。
   - **macOS 12.0 引き上げ**（macOS ターゲットがある場合）: `macos/Podfile` と `MACOSX_DEPLOYMENT_TARGET` を `12.0` に更新する。
   - **iOS `UIScene` 対応**:
     - `AppDelegate`・`Info.plist` が未編集の標準状態であれば、ビルド時に自動マイグレーションされるため手動作業は不要。
     - カスタマイズしている場合は、`Info.plist` への `UIApplicationSceneManifest` 追加、および `AppDelegate` の `FlutterImplicitEngineDelegate`（`didInitializeImplicitFlutterEngine`）への移行を行う。
5. **クリーンビルド＆実機・シミュレータ起動確認**
   ```bash
   flutter clean
   flutter pub get
   cd ios && pod install && cd ..   # JFrog の CocoaPods リポジトリ経由で通ることを確認
   flutter run                      # iOS / Android 両方で起動確認
   ```
   - **チェックポイント**: 両 OS でアプリが起動することを確認したら、**ここで必ず1回目の Git コミットを作成する**（以降で問題が起きてもいつでもこの動く状態に戻せる）。

---

### フェーズ2：iOS / macOS 依存管理の確定（JFrog / CocoaPods 運用の検証と将来対応メモ）

通常の環境ではここで SwiftPM を有効化しますが、**社内 JFrog が SwiftPM 未対応の環境では、フェーズ1で設定した `enable-swift-package-manager: false` を外さずにそのまま維持（確定）**します。

1. **SwiftPM が誤って有効化されていないことの確認**
   - `ios/Runner.xcodeproj/project.pbxproj` の `git diff` を確認し、`FlutterGeneratedPluginSwiftPackage` などの SwiftPM 参照が混入していないことを確認する。
   - ※万が一、`pubspec.yaml` 設定前にビルドを実行して Xcode プロジェクトが書き換わり壊れてしまった場合は、`git checkout -- ios/` で一度戻してからフェーズ1の順序でやり直すだけでクリーンに復旧できる。
2. **中長期のインフラ検討メモ（2026年12月以降に向けた注意点）**
   - 今回の `v3.47.8` アップデートおよび当面のビルドは CocoaPods のままで 100% 問題なく稼働する。
   - ただし、**2026年12月2日に CocoaPods 公式中央リポジトリ（Trunk）が読み取り専用（Read-Only）に移行**するため、2027年以降にリリースされる新しいバージョンの iOS ネイティブ SDK（Firebase 等）や Flutter プラグインは順次 CocoaPods サポートを終了し SwiftPM 専用へ移行していく。
   - 次回以降の大型アップデートに備え、別タスクとして社内インフラ部門へ **JFrog Artifactory での SwiftPM（Swift Package Registry）リポジトリ対応** を相談・依頼しておくと安全。

---

### フェーズ3：Android の Built-in Kotlin / AGP 9 対応への移行

iOS とアプリ本体が安定している状態で、Android の Gradle / Kotlin 設定だけを単独でモダン化します。

1. **Android Gradle 設定の更新**
   - Kotlin Gradle Plugin (KGP) バージョンが `2.0.0` 以上になっていることを確認する。
   - AGP 9.0 以上を使用する場合は、`android/app/build.gradle(.kts)` からレガシーな `id("kotlin-android")` / `id("org.jetbrains.kotlin.android")` 指定を削除し、JVM ターゲット指定を `kotlin { compilerOptions { ... } }` 記法へ移行したうえで、フェーズ1で一時追加した `android/gradle.properties` の `android.builtInKotlin=false` / `android.newDsl=false` を削除（または `true` に変更）する。
2. **Android ビルド（Debug および Release AAB/APK）の検証**
   ```bash
   flutter build apk --debug
   flutter build appbundle --release
   ```
   - **ロールバック基準**: もし社内 JFrog 上の AGP / Gradle プラグインミラーバージョンや、未対応のサードパーティプラグインが原因でエラーになる場合は、無理に AGP 9 へ上げずフェーズ1の設定（AGP 8系 + `android.builtInKotlin=false`）に戻して進めても `v3.47.8` では問題なく動作する。
   - 確認できたら **ここで2回目の Git コミットを作成する**。

---

### フェーズ4：非推奨（Deprecated）API・動作変更の解消と最終テスト

最後に、ビルドシステムが完全に固まった安全な状態で Dart コードの品質改善と動作変更チェックを行います。

1. **`dart fix` による機械的置換の一括適用**
   ```bash
   dart fix --dry-run   # 変更差分の事前確認
   dart fix --apply     # 自動修正の適用（containsSemantics → isSemantics、cacheExtent → scrollCacheExtent 等）
   ```
2. **手動確認が必須なロジック・UI 変更箇所の洗い出しと修正**
   - **`ListView.separated` / `SliverList.separated`**: `findChildIndexCallback` を `findItemIndexCallback` に変更し、インデックス計算の **`* 2` を必ず削除**する。
   - **`ReorderableListView` / `ReorderableList` / `SliverReorderableList`**: `onReorder` を `onReorderItem` に変更し、**`if (oldIndex < newIndex) newIndex -= 1;` の手動補正を必ず削除**する。
   - **`Semantics(header: true)`**: コンパイルエラーにならないまま iOS / Android で見出し認識が無効化されているため、プロジェクト内を全文検索して **`Semantics(headingLevel: 1)`** 等へ書き換える。
   - **色付き `Container` / `ColoredBox` 直下の `ListTile`**: デバッグ実行時に画面を遷移して `FlutterError` が出ないか確認し、該当箇所があれば `ListTile(tileColor: ...)` または `Material` ラップへ修正する。
   - **Material 3 生成色 / バリアブルフォント**: シードカラー（`onPrimaryContainer` 等）や可変フォントの太さに意図しない見た目変化がないか主要画面を目視確認する。
3. **静的解析・テストの実行と完了コミット**
   ```bash
   flutter analyze
   flutter test
   ```
   - すべての解析とテストがパスしたことを確認し、**最終コミットを作成してアップデート完了**。
