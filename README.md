# Wipe&Snap

スペインのトマト祭り「La Tomatina」をモチーフにした、2人協力プレイの体験型撮影ゲームです。  
1Pはカメラマンとして自作カメラ型コントローラーを構え、群衆の中からお題の被写体を探して撮影します。  
2PはLeap Motion Controllerで手を動かし、トマトで汚れたレンズを拭き取って1Pの撮影を支援します。

## プロジェクト概要

- **タイトル**: Wipe&Snap（リポジトリ名 `Wipe-Snap`）
- **ジャンル**: 2人協力 / 体験型 / ハードウェア連動
- **想定設置環境**: モニター + スマートフォン画面（デュアルディスプレイ）+ LeapMotion
- **プレイ人数**: 2 人(1P=カメラマン、2P=タオル拭き取り)
- **想定プレイ時間**: 3分ほど
- **用途**: 学内展示 / 試遊会 

## デモ

- プレイ動画:https://youtu.be/YlQxtFcei3Y
- Zenn: URL:https://zenn.dev/ippon/articles/32ad6608d7eb30

## 自分の担当

私は主に、1P/2Pの操作体験に関わるクライアント実装を担当しました。

- スマホ表示用 `SWindow` の生成
- `SceneCapture2D` + `RenderTarget` によるズームファインダー
- Leap Motion Controllerの手速度を使った拭き取り判定
- ズームカメラの壁抜け対策
- `ProfileGPU` による描画負荷確認とトマト影設定の改善
- 展示環境での表示安定化

| 担当者 | 主な担当 |
|--------|----------|
| 自分 | ゲーム体験の企画、UE5/C++実装、スマホ表示用SWindow、SceneCapture2D + RenderTargetによるズームファインダー、Leap Motion Controllerによる拭き取り判定、UI/UX調整 |
| メンバーA | 3Dモデル、アニメーション調整、モデルアタッチ、ワールドへのアクター配置、街並みの作成、Blueprintでの敵キャラの動き調整 |
| メンバーB | イラスト、背景作成 |
| メンバーC | ゲームルールの検討、制作補助、調整作業、カメラ型筐体製作など |

筐体制作、3Dモデル、ステージ制作、サウンド、ゲームルール検討は主に他メンバーが担当しました。

チーム制作のため、他メンバーが制作した素材・演出・レベル要素を自分の成果として誤解されないよう、担当範囲を分けて記載しています。

## チーム制作での役割

4人チームの中で、自分は技術選定の取りまとめ、進捗の調整、試遊フィードバックの集約も担当することが多くありました。本作で実際に判断・調整した主な内容を以下に挙げます。

### 技術選定: BPからC++への移行で合意形成

ロジックをC++に寄せる提案を出したとき、エンジンを触る他メンバーから「C++が自分には使えないので、BPのほうがわかりやすい」という反対が出ました。全部C++に倒すと他メンバーが触れなくなるため、毎フレーム動く中核ロジック(Leap Motion入力、フィルタ処理、拭き取り判定)のみC++に寄せ、ステージ構成や演出パラメータはBPに残し、Detailsパネルで触れるパラメータも増やす折衷案で進めました。チーム全員が引き続き自分の担当範囲を触れる状態を保ちつつ、ロジックの保守性を上げる落とし所にできました。

### 進捗管理: 遅れているメンバーへの個別対応

進捗が遅れているメンバーがいたときは、まず個別に話して原因を確認し、その上で対応を決めました。一例として、大学の課題が重なって時間が取れていないメンバーには、タスク量を減らし、足りない部分はヘルプメンバーを探したり、配布アセットで代用したりしました。「やっていないこと」を責めるより、現状を踏まえて出せる範囲を一緒に決めるほうが、結果的に進捗は出ました。

別のメンバーが進捗で詰まっていたときは、共同作業日を設けて一緒に作業を進める形にしました。本人だけで詰まっていたところを、その場で相談しながら進められたので、時間あたりの効率が上がりました。

### 制作物の代替判断

イラストやモデルの制作作業が間に合わないメンバーがいたため、ヘルプを探して背景イラストをお願いしたり、配布モデルで代用したりしました。納期に対して品質をどこまで妥協するかの判断を、メンバーの状況とゲーム全体の完成度を見ながら都度決めていました。

### 試遊フィードバックの集約

試遊で出た課題は、Discordなどの情報共有ツールでメンバー全員から改善案を出してもらい、その中から実装する案を選ぶ形で進めました。一人で方針を決めるより、メンバーが普段触っている領域からの提案も拾えるため、設計判断の質が上がる場面が多くありました。

## 設計判断

クライアント側の実装は、1Pの撮影体験(`ATomatinaPlayerPawn`)、2Pの拭き取り体験(`ATomatinaTowelSystem`)、汚れの状態管理(`ATomatoDirtManager`)、別ウィンドウ表示の管理(`ATomatinaHUD`)を、それぞれ独立したクラスに分けています。1Pと2Pの操作体験をクラス単位で切り離したことで、試遊フィードバックを受けて片方のパラメータを調整しても、もう片方の挙動に影響しにくい構成になりました。

別ウィンドウ生成を `ATomatinaHUD` に置き、`ATomatinaPlayerPawn` から切り離したのは、シングルウィンドウ + Widgetスパン方式から `SWindow` 方式へ切り替えた経緯があるためです。表示形態の変更がゲームロジック側に漏れないように責務を分けました。`ATomatinaTowelSystem` が `ATomatoDirtManager` に直接書き込まず、`WipeDirtAt(NormPos, Radius, Amount)` 経由で操作する形にしたのも同じ理由で、汚れの表現方式を変更しても2P側の判定処理を変えずに済むようにしています。

Leap Motion入力の取得・座標変換・スムージング・拭き取り判定は、当初Blueprint側でC++関数を呼ぶ構成でしたが、チーム開発の文脈で問題が見えてきたため、ロジックの中核を `ATomatinaTowelSystem` 側のC++に集約し、BPはDetailsパネルで触る公開パラメータと参照設定のみを残す構成に変更しました。BPからC++関数を毎フレーム呼ぶ前提で組むと、イベント接続忘れやマップごとの差分で入力が動かなくなる問題が出やすく、また他メンバーがパラメータを少し変えたい場合でもBPグラフを開いて追う必要があったためです。一方で、ステージ固有の演出パラメータや参照設定はBPに残し、エディタ上での即時調整しやすさを保っています。

Leap Motion 入力の不安定さに対しては、UltraLeap Visualizer 上でも速い動きでの取りこぼしが再現したため、ハードウェアの限界に達していると判断しました。そこから対処を、物理層(デバイス配置・USB・Tracking Service バージョン)、入力処理層(Confidence ゲート・Grace 内の速度維持・スムージング)、ゲームデザイン層(拭き取り判定を点から線分に変えてフレーム間の隙間を吸収)の3層に分けて取り組んでいます。1層だけで完結させようとせず、ハードの限界をソフトとゲームデザインで段階的に受け止める分担にしました。

### 操作概要

| 役割 | 入力デバイス | 主な操作 |
|------|--------------|----------|
| 1P カメラマン | キーボード/コントローラ(Enhanced Input) | 視点移動・トマト発射・スマホ画面でズーム確認 |
| 2P 拭き取り | LeapMotion(Ultraleap Tracking) | 手のジェスチャでメイン画面の汚れを拭く |

## Unreal Engine のバージョン

- **UE 5.7**(`Tomato.uproject` の `EngineAssociation` で確認済み)
- ビルド構成: C++ + Blueprint
- パッケージング: Monolithic(`Tomato.exe` 約 297MB)

## 解決済みの主な改善

試遊や `ProfileGPU` を踏まえて、改善まで完了した項目です。

| 項目 | 内容 |
|------|------|
| マルチディスプレイ表示 | シングルウィンドウ + Widgetスパン方式から `SWindow` 方式に切り替え。展示前の画面位置調整を、長いときの約1時間から数分程度に短縮しました |
| ズームファインダー表示の安定化 | `RenderTarget` を直接Brushに渡す代わりに `UMaterialInstanceDynamic` のTexture Parameter経由で表示し、サブウィンドウ側の更新反映を安定させました |
| 描画負荷の改善 | `ProfileGPU` で `ShadowDepths` がボトルネックであることを特定し、トマト影を無効化することで Frame を約31.7msから約23.7msへ改善しました。続けて `SceneCapture2D` をズーム中のみ更新する形に変更し、ズーム外で約16.0ms(60fps到達)、ズーム中も約19.0msまで詰めました |
| Leap Motion入力の安定化 | EMA / One Euro Filter によるスムージング、`InputGraceTime` 内での入力保持、座標Clamp、画面端での拭き取り半径補正に加え、Confidence + TimeVisible によるゲート、Grace 内の速度維持、拭き取り継続のヒステリシス、近すぎ警告の hysteresis + debounce、拭き取り判定の点 → 線分化を実装しました(PIE と試遊で改善を確認) |
| BP→C++ への寄せ替え | チームで触りやすくするため、Leap Motion関連のロジックをC++側に集約し、BPはDetailsで触るパラメータと参照設定のみを残す構成に変更しました |
| ズームカメラの壁抜け対策 | Sphere Sweep + Safety Margin による位置クランプと、`SkyFallbackDistance` による空方向の仮想ヒット処理を実装しました |

### 60fps目標と現状

`SceneCapture2D` をズーム中のみ更新する変更で、ズーム外の Frame は約16.0msとなり、60fps相当の16.6msに到達しました。ズーム中は約19.0msで、依然16.6msには届いていないため、続けて `Postprocessing` の軽量化、スマホ用 `RenderTarget` 解像度の調整、トマト以外の大量生成オブジェクトの影設定見直しを検討します。

## 現在実装されている機能

### ゲームプレイ

- [x] 1P 
- [x] 2P LeapMotion 拭き取り(`UTomatinaTowelSystem`)
- [x] トマト発射(`ATomatinaProjectile`)と着弾位置への汚れ生成
- [x] 汚れ管理(`ATomatoDirtManager`)
  - 通常トマト(赤): 円形範囲の Opacity 漸減
  - 特殊トマト(黄、Sticky): 連続ダッシュ × 4 回で剥がれる
- [x] スタイリッシュランク(C / B / A / S / SSS)と Sync Rate 計測
- [x] ミッション制ゲーム進行(`ATomatinaGameMode` 内 Missions 配列)
- [x] 全汚れクリア演出 → 最終リザルトへの遷移

### ハードウェア / 描画

- [x] **デュアルウィンドウ表示**: `FSlateApplication::AddWindow` + 動的 `SWindow` 生成によりメイン画面とスマホ画面を別ウィンドウで描画
- [x] スマホ画面に `RT_Zoom` レンダーターゲットを表示(`ConfigureZoomImageContent`、DMI 経由)
- [x] DPI スケーリング補正(`UWidgetLayoutLibrary::GetViewportScale()`)
- [x] UE 5.7 SceneCapture 自動キャプチャ問題の回避(Tick 内で `CaptureScene()` 手動呼び出し)
- [x] ズームカメラの壁抜け対策(Sphere Sweep + Safety Margin + `CustomNearClippingPlane`)
- [x] 空・無限遠ヒット対策(`SkyFallbackDistance` で仮想ヒット点)
- [x] キャラクタ全体に汚れマスクを乗せる `SetOverlayMaterial`(UE 5.0+)

### UI / 演出

- [x] ポーズ可能な UI(`SetGlobalTimeDilation(0)` + `FApp::GetDeltaTime()`)
- [x] 拭き取り SE のループ・エッジ検出(`UpdateWipeSound`)
- [x] BeginPlay 前のプロパティ注入(`SpawnActorDeferred` + `FinishSpawning`)

## 既知の問題

優先度の高い課題を3つに絞っています。

| # | 課題 | 重要度 | 現状・原因 | 改善方針 |
|---|------|--------|------------|----------|
| 1 | スマホファインダーへの視線誘導が弱い | 高 | 試遊では8人中5人が、ズーム中もメインモニターを見続けました。スマホ画面を見なくても最低限プレイできるため、見る必然性がまだ弱い状態です。 | スマホ側にのみ表示される構図補助や高得点判定のヒントを追加し、見るメリットをゲームルール側から強める。ズーム開始時や撮影直前に視線誘導演出を入れる。 |
| 2 | レベル開始時のスポーン処理によるカクつき | 高 | トマト・観客・投げる人を開始時に一括生成しているため、処理が一時的に集中します。 | スポーン処理のフレーム分散、動きの少ないオブジェクトの事前配置、トマト・エフェクトのオブジェクトプール化を検討します。 |
| 3 | ズーム中の60fps未達 | 中 | ズーム外は `SceneCapture2D` 改善後に約16.0msとなり60fpsに到達しましたが、ズーム中は約19.0msで16.6msに届いていません。 | `Postprocessing` の軽量化、スマホ用 `RenderTarget` 解像度の調整、トマト以外の大量生成オブジェクトの影設定見直しを順に検討します。 |

## 開発中・改善予定の機能

現在、展示や試遊で得られた課題をもとに、以下の改善を進めています。

| # | 項目 | 目的 | 状況 |
|---|------|------|------|
| 1 | スマホ側のみの構図補助・高得点ヒント表示 | スマホ画面を見るメリットをゲームルール側から強める | 設計検討中 |
| 2 | ズーム開始時・撮影直前の視線誘導演出 | プレイヤーがスマホ画面へ視線を移すタイミングを作る | 設計検討中 |
| 3 | `SceneCapture2D` のズーム中のみ更新 | ズーム外でのGPU負荷を削減する | 実装済み(ズーム外 約16ms / ズーム中 約19ms) |
| 4 | `Postprocessing` 軽量化 / `RenderTarget` 解像度調整 | ズーム中の Frame をさらに16.6msへ近づける | 改善予定 |
| 5 | スポーン処理のフレーム分散 / オブジェクトプール化 | レベル開始時のカクつきを解消する | 改善予定 |
| 6 | スマホ用 `SWindow` の自動配置 | 展示環境ごとのウィンドウ位置調整をさらに減らす | 改善予定 |
| 7 | カメラ型コントローラー筐体の改善 | 段ボール製の現行筐体を、3Dプリンターで持ちやすい形状に置き換える | 検討中 |

これらの課題は、現時点で未完成な点を隠すためではなく、試遊・計測を通して見えた改善対象として整理しています。特にスマホファインダーへの視線誘導と、ズーム中のさらなる負荷削減は今後の優先改善項目です。

## ライセンス

このリポジトリ内のコードおよび自作素材のライセンスは未設定です。第三者アセットは各配布元・購入元のライセンスに従います。

### 使用アセット・外部素材

| 種別 | 配置場所 / ファイル | 出典・ライセンス確認 |
|------|---------------------|----------------------|
| Unreal Engine テンプレート / Starter 系素材 | `Content/FirstPerson/`, `Content/LevelPrototyping/`, `Content/Characters/Mannequins/`, `Content/Input/` | Epic Games / Unreal Engine 付属コンテンツ。Unreal Engine EULA / Epic Content License の範囲で使用。 |
| Fab / Unreal Marketplace / Quixel 系 3D アセット | `Content/Fab/Gorilla/`, `Content/Fab/Megascans/3D/Trash_Bag_Pack_ve2hddjga/`, `Content/MSPresets/`, `Content/Fantastic_Dungeon_Pack/`, `Content/LowPolyMarket/`, `Content/ModularBuildingSet/`, `Content/RPGHeroSquad/`, `Content/Scanned3DPeoplePack/`, `Content/UFO/` | Fab Standard License または取得時の Marketplace ライセンスに従う。Fab Standard License は、プロジェクトへ組み込んだ形での商用/私的利用を許可する一方、アセット単体の再配布・再販売は禁止。 |
| Fab / Megascans | `Content/mono/gomi/`, `Content/mono/Firehydrant/`, `Content/mono/gomibako/` | `Trash_Bag_Pack_ve2hddjga` 等 |
| Ultraleap Tracking Plugin | `Plugins/UltraleapTracking_ue5_4-5.0.1/` | Ultraleap Unreal Plugin。GitHub版は Apache License 2.0。実機利用には Ultraleap Tracking Software が必要。各ライセンス・利用規約に従って使用。 |
| Freesound 環境音 | `Content/Sound/755969__lastraindrop__evening-food-market-atmosphere-at-street-side.uasset` | Freesound: "Evening food market atmosphere at street side" by `lastraindrop`。Creative Commons 0 (CC0)。 |
| ニコニ・コモンズ | `Content/Sound/nc146963.uasset` | ニコニ・コモンズ素材。素材ページの利用条件に従って使用。 |
| Pngtree 画像素材 | `Content/Assets/—Pngtree—white_crumpled_towel_after_use_13244949.uasset`, `Content/Tomato_Asset/pngtree-blood-splatter-drop-png-image_13534558.uasset` | Pngtree License Terms に従う。 |
| イラストくん素材 | `Content/Assets/illustkun-03200-tomato.uasset`, `Content/Assets/illustkun-03200-tomato_Sprite.uasset` | イラストくん利用規約に従う。個人・法人利用、商用利用は規約範囲内で可。素材自体の再配布・販売は禁止。 |
| The Noun Project アイコン | `Content/Assets/noun-focus-point-4695835.uasset` | The Noun Project 素材。 |
| SunoAI BGM | `Content/Sound/` 配下のBGMアセット | SunoAIで生成したBGMを使用。生成後、ゲームの雰囲気に合うものを選定し、音量や使用場面を調整して使用。利用規約に従って使用。 |
| 生成AIによる汚れ画像素材 | レンズ汚れ・トマト汚れ表現に使用している画像アセット | 画像生成AIで作成した汚れ画像をもとに、ゲーム内のレンズ汚れ表現として使用。必要に応じて見た目や透明度を調整し、ゲーム画面上での視認性を確認。 |


### 参照した主なライセンス情報

- Fab Standard License: https://www.fab.com/eula
- Fab / Quixel Megascans: https://www.fab.com/sellers/Quixel%20Megascans/about
- Ultraleap Unreal Plugin: https://github.com/ultraleap/UnrealPlugin
- Freesound `755969`: https://freesound.org/people/lastraindrop/sounds/755969/
- Pngtree License Terms: https://pngtree.com/legal/terms-of-license
- イラストくん ご利用について: https://illustkun.com/about-use/
- The Noun Project license help: https://help.thenounproject.com/hc/en-us/articles/200509798-What-licenses-do-you-offer-for-icons

## 生成AIの利用について

本プロジェクトでは、実装方針の整理、コード案の作成、既存コードの整理、バグ原因の仮説出しに Claude Code および Codex を使用しました。

ただし、最終的な仕様判断、プロジェクトへの組み込み、実機での動作確認、パラメータ調整、採用する実装方針の決定は自分で行いました。

特に、スマホファインダー表示、`RenderTarget` 表示、Leap Motion入力、`ProfileGPU` による負荷確認では、AIの提案をそのまま使うのではなく、プロジェクトの要件に合わせて修正・検証しました。

また、BGM素材の一部にSunoAI、レンズ汚れ表現の画像素材作成に画像生成AIを使用しました。生成した素材はそのまま使うのではなく、ゲームの雰囲気や視認性に合うかを確認し、必要に応じて音量・見た目・透明度などを調整しました。

本リポジトリにはAIによるコミットが含まれていますが、提出にあたっては自分の担当範囲とAI利用範囲を明記しています。
