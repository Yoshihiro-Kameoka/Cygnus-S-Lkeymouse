# Cygnus 実装ロードマップ: OS別レイヤー × 接続先切替

作成日: 2026-08-08  
更新日: 2026-08-09（オートマウス不使用・レイヤー番号/名は仮置きと明記）  
関連:
- [firmware-comparison-cygnus-mona2.md](./firmware-comparison-cygnus-mona2.md)
- [shakupan-implementation-deep-dive.md](./shakupan-implementation-deep-dive.md)（参照実装の詳細）

## 1. 目指す体験

**接続先を切り替えるだけで、Windows / Mac / iOS の差分を意識せず使える Cygnus** を作る。

| BT スロット（案） | 想定デバイス | 有効になるベースレイヤー |
|---|---|---|
| BT0 | Windows PC | `win_*` 系 |
| BT1 | Mac | `mac_*` 系 |
| BT2 | iPhone / iPad | `ios_*` 系 |
| BT3〜4 | 予備 | 後で決める |

ポイント:

- OS を自動判定するのではない
- **「どの BT 番号に何を登録したか」** と **「その番号用レイヤー」** を紐づける
- 日常操作は接続先切替だけで、修飾キーやショートカットが切り替わる

参考実装: [shakushakupanda/zmk-config-moNa2-v2](https://github.com/shakushakupanda/zmk-config-moNa2-v2) の Win/Mac 二系統 + `BT0`/`BT1` マクロ

---

## 2. 現在までの確定方針（2026-08-08）

会話・調査を踏まえて、次を作業前提とする。

### 2.1 全体方針

| 方針 | 内容 |
|---|---|
| 参照する完成形 | **shakupan moNa2**（BT×OS レイヤー同期・運用設計） |
| 移行の土台 | **sayu moNa2-v2**（新世代トラボ / input processors） |
| 作業場所 | **このリポジトリ**でキーマップ正本化と設定の再実装を行う |
| ハードウェア | **交換不要**（PMW3610 配線は同系統。UF2 の作り替えで寄せる） |
| やってはいけない | shakupan / sayu の UF2 を Cygnus に**そのまま焼く**（キー配列ズレのリスク） |

### 2.2 キーマップの現状（重要）

- 実機の現行キーマップは **ZMK Studio 上だけで変更**されており、**この Git リポとはかけ離れている**
- Studio の内容はリポへ自動では戻らない
- 画面コピペやスクショだけでは、マクロ・hold-tap・レイヤー構造まで十分にわからない
- したがって次が必要:
  1. **いまのキーマップをこのリポの `config/Cygnus.keymap` に正本化（書き起こし）**
  2. その手癖を残したまま、**環境差を潰すレイヤー（Win/Mac/iOS）や優れた設定をここで再実装**

認識の確認:

> 現在のキーマップさえ明らかにすれば、環境差を潰すレイヤーはここで実装すればよい  
> → **その認識で正しい**

### 2.3 正本化のやり方（Studio のみの場合）

出所が Git に無い前提では、実質このルートのみ。

1. 実機を **ZMK Studio に接続したまま**表示を正本にする（記憶頼みより精度が高い）
2. レイヤーごとに割り当てを読み上げ / メモ / スクショ
3. チャットまたは直接編集で `config/Cygnus.keymap` へ落とし込む
4. 以降はこのリポを唯一の正本にし、Studio 単独いじりは避ける（ズレ防止）

一度正本化すれば、以後の修正・BT×OS 実装・レビューはすべて Git ベースで回せる。

### 2.4 役割分担（何を残し、何を作り直すか）

| 残す | 作り直す / 新設する |
|---|---|
| いま手に馴染んでいるキー配置・レイヤー役割（正本化後） | トラボ基盤（旧ドライバ → badjeff + pointing） |
| Cygnus ハードウェア前提のマトリクス | BT×Win/Mac/iOS レイヤー同期 |
| | CPI・スクロール倍率など体感チューニング |
| | （必要なら）エンコーダ・BLE 間隔などの詳細 |

つまり **「配列の記憶」は現行実機から取り込み、「環境差を消す仕組み」は shakupan 型をここで実装」** する。

### 2.5 キーマップ解釈上の注意（2026-08-09 確定）

いまリポにあるレイヤー構成を、最終仕様だと読まないこと。

| 項目 | 方針 |
|---|---|
| **オートマウス** | **使わない**（旧 `automouse-layer` 前提の説明・実装は採用しない） |
| **Layer 1 / `FUNCTION` という名前** | 作業・説明用の**仮置き**。AI が読みやすいようにマウス操作を Layer 1 寄りにまとめているだけ |
| **レイヤー番号・レイヤー名** | **これから調整する**。Win/Mac/iOS 分割や入口キー（例: Space→マウス）に合わせて再採番・改名する |
| **残すもの** | 手に馴染む**割り当て内容**（どの指で何をするか）。番号やシンボル名に固定されない |
| **参照する操作モデル** | shakupan 型（明示的にマウス層へ入り、その中でスクロールへホールド等）。Cygnus 旧来の「L→MOUSE / `-`→SCROLL / オートマウス」を最終形とはしない |

説明や比較で「Cygnus の FUNCTION」「Layer 4 MOUSE」などと言う場合は、**現行ファイルの仮の姿**を指しているだけ、と読む。

---

## 3. なぜこの方針か

1. 業務で複数 OS を行き来しても、キーボード側の思考コストを下げられる
2. shakupan moNa2 で実証済みの型を、Cygnus に拡張できる（iOS を 3 系統目として追加）
3. ハードウェア交換は不要。ファーム設計の問題として進められる
4. 現行の手癖を捨てずに、優れた運用設計だけを足せる

制約（忘れないこと）:

- 初回だけ「BT0=Win / BT1=Mac / BT2=iOS」の割り当てが必要
- プロファイルを消して付け直すと、割り当てを再確認する必要がある
- iOS は Mac に近いので、全レイヤーを三重管理しなくてよい（差分だけ分ける）
- Studio と Git を並行改造するとまた乖離する

---

## 4. 作業の大枠（順序が重要）

実装は次の順で進める。**先に現行キーマップをリポへ固定し、その後に基盤と OS 三系統を載せる。**

```text
Phase 0   現状把握・方針確定（本ドキュメント）           ← 済（方針確定）
   ↓
Phase 0.5 現行キーマップの正本化（Studio → Cygnus.keymap） ← 次の一手
   ↓
Phase 1   トラックボール基盤を shakupan/sayu 系（新世代）へ移行
   ↓
Phase 2   Win/Mac 二系統 + BT 連動（shakupan 型の最小移植）
   ↓
Phase 3   iOS を三系統目として追加
   ↓
Phase 4   速度・スクロール・エンコーダ等の体感チューニング
```

理由:

- リポ上の `Cygnus.keymap` は実機と一致していないため、このまま OS レイヤーを足すと手癖と衝突する
- いまの Cygnus は旧 PMW3610 ドライバ方式のため、スクロール/オートマウスのレイヤー番号がドライバに直結している
- OS 別レイヤーを増やす前に、**input processor 方式へ移した方がレイヤー設計が自由**になる

---

## 5. Phase 0.5 — 現行キーマップ正本化（次にやること）

目標: **実機（Studio 表示）と `config/Cygnus.keymap` を一致させる。**

### やること

| # | 作業 | 完了条件 |
|---|---|---|
| 1 | Studio を実機接続し、全レイヤーの割り当てを洗い出す | レイヤー一覧が文書またはチャットに残る |
| 2 | `config/Cygnus.keymap` を現行内容で書き起こす | ファイルが実機と対応する |
| 3 | （任意）レイヤー名・コメントを整理 | 後続の Win/Mac 分割が読みやすい |
| 4 | push → ビルド → 実機焼きで差分確認 | 「手癖どおり」が Git 経由でも再現 |

### やらないこと（この Phase）

- BT×OS 三系統の本実装
- トラボドライバ世代の切り替え（Phase 1）
- キー配列の大胆な作り直し（正本化が先）

### 完了条件

- [ ] このリポの keymap が、いま使っている実機配置の正本になっている
- [ ] 以降の変更は Studio 単独ではなく、このリポ経由で行う合意がある

---

## 6. Phase 1 — 基盤移行（必須前提）

目標: shakupan/sayu と同じ「新世代」スタックで Cygnus がビルド・動作する。

### やること

| # | 作業 | 主なファイル |
|---|---|---|
| 1 | `west.yml` の PMW3610 を `badjeff @ zmk-0.3` へ | `config/west.yml` |
| 2 | 必要なら `zmk-input-processor-keybind` を追加 | 同上 |
| 3 | `CONFIG_ZMK_MOUSE` 依存を `CONFIG_ZMK_POINTING` 側へ | `Cygnus_R.conf` / `Kconfig.defconfig` |
| 4 | overlay を sensor + listener + scroller 方式へ | `Cygnus_R.overlay` / `Cygnus.dtsi` |
| 5 | 旧 `automouse-layer` / `scroll-layers` / `CONFIG_PMW3610_CPI` を廃止または置換 | conf / overlay |
| 6 | CPI を当面 **600** 程度に（shakupan 準拠） | overlay |
| 7 | **正本化済み**キーマップがビルド通る最小差分で維持 | `Cygnus.keymap` |
| 8 | Actions で左右 + reset が通ることを確認 | CI |

### 完了条件

- [ ] push でファームが生成される
- [ ] 左右とも書き込み後、通常タイピングできる
- [ ] トラックボールでカーソルが動く（向きが逆なら invert で吸収）
- [ ] スクロール用レイヤー（仮）でボールスクロールできる

### やらないこと（この Phase）

- Win/Mac/iOS レイヤーの本格分割
- キー配列の大幅変更
- **オートマウスの再現はしない**（採用しない方針）

---

## 7. Phase 2 — Win/Mac + 接続先連動

目標: shakupan と同じく、**BT 切替と同時に OS 用ベースレイヤーが切り替わる。**

### レイヤー案（仮番号・実装時に再採番可）

| 番号 | 名前 | 役割 |
|---|---|---|
| 0 | `win_base` | Windows 既定 |
| 1 | `mac_base` | Mac 既定 |
| 2 | `win_num` / 共用 num | 数字・記号（差分が少なければ共用） |
| 3 | `mac_num` | 必要なら分離 |
| 4 | `win_mouse` | Windows マウス層 |
| 5 | `mac_mouse` | Mac マウス層 |
| 6 | `win_scroll` | ボールスクロール（Win） |
| 7 | `mac_scroll` | ボールスクロール（Mac） |
| 8 | `ble` | BT 選択・クリア・bootloader |

※ 番号は shakupan に寄せるか、Cygnus の使いやすさ優先で再設計してよい。  
重要なのは **「ベース 0/1 が Win/Mac」「スクロール層も OS 別 or 共用を明示」** すること。

### BT マクロ案

```text
BT_WIN  = 他 OS ベース OFF → BT_SEL 0 → win_base ON
BT_MAC  = 他 OS ベース OFF → BT_SEL 1 → mac_base ON
```

実装パターン（shakupan 準拠）:

1. `tog_off` で Win/Mac ベースを落とす
2. `bt BT_SEL n`
3. 短い wait
4. `tog_on` で対象ベースを上げる

### 運用ルール（ユーザー向けに README へも後で転記）

1. 初回: Windows を BT0、Mac を BT1 でペアリング
2. 以降: `BT_WIN` / `BT_MAC`（または ble レイヤーのキー）だけで切替
3. プロファイルを消したら、同じ番号に同じ OS を付け直す

### 完了条件

- [ ] Windows 接続中は Ctrl 系ショートカットが意図どおり
- [ ] Mac 接続中は Gui/Cmd 系が意図どおり
- [ ] 切替マクロ一発で接続先とレイヤーが両方変わる
- [ ] ボールスクロールが各 OS で破綻しない

---

## 8. Phase 3 — iOS を三系統目に追加

目標: **BT2 = iOS** でも同じ操作感で業務・日常入力ができる。

### 設計方針

iOS は Mac に近い。最初から全レイヤーを三重化しない。

| 項目 | 方針 |
|---|---|
| 修飾キー | 基本 Mac 準拠（Gui 中心） |
| 差分が出やすいところ | スクショ、ホーム相当、制御キー、メディア、言語切替などだけ `ios_*` で上書き |
| マウス | iPad 中心ならマウス層を薄く持ってもよい。iPhone のみなら簡略化可 |
| スクロール | まずは Mac スクロール層を共用し、違和感があれば分離 |

### 追加マクロ

```text
BT_IOS = 他 OS ベース OFF → BT_SEL 2 → ios_base ON
```

### レイヤー追加案

| 番号 | 名前 | 内容 |
|---|---|---|
| 9? | `ios_base` | Mac base をコピーし、差分だけ変更 |
| (必要なら) | `ios_mouse` / `ios_scroll` | 実機で不足が出てから追加 |

### 完了条件

- [ ] iOS とペアリングし、文字入力・修飾キーが実用レベル
- [ ] Win ↔ Mac ↔ iOS をマクロだけで往復できる
- [ ] 「今どの OS 想定か」をキー操作で確認できる（LED / 特定キー配置など、任意）

---

## 9. Phase 4 — 体感チューニング

基盤と OS 切替が安定してから行う。

| 項目 | 初期ターゲット（shakupan 参考） | メモ |
|---|---|---|
| CPI | 600 | Cygnus 現状 1000 より遅く感じる想定。好みで 600〜800 |
| ボールスクロール倍率 | 要実機調整 | shakupan はかなり精密（1/20） |
| BLE 間隔 | まず sayu 型（6〜12）または現行固定 7.5ms | 滑らかさ vs 電力 |
| エンコーダ | 現行維持 → 必要なら steps 見直し | |
| オートマウス | 既定 OFF 推奨 | 必要なら後付け |

---

## 10. 決定しておくと楽なこと（未決リスト）

実装前に決めると手戻りが減る。

- [ ] BT0/1/2 の割り当てをこの文書の案で確定するか
- [ ] 数字・記号レイヤーを OS 共用にするか、分離するか
- [ ] スクロールレイヤーを OS 別にするか、1 つにして scroller の `layers` に複数載せるか
- [x] オートマウス → **使わない**
- [ ] iOS は iPad 主か iPhone 主か（マウス層の厚みが変わる）
- [x] キー配列の起点 → **実機（Studio）の現行配置を正本化する**（shakupan 配列への全面寄せはしない）
- [x] レイヤー番号・名前 → **仮置き。後で再編する**（FUNCTION 等の名前に意味を固定しない）
- [ ] Phase 0.5 の書き起こし単位（一括 / レイヤーごと）
- [ ] マウス層への正式な入口キー（Space 長押し等）の確定

---

## 11. 触るファイル一覧（予定）

| ファイル | Phase |
|---|---|
| `config/Cygnus.keymap` | **0.5（正本化）**, 2, 3, 4 |
| `config/west.yml` | 1 |
| `config/boards/shields/Test/Cygnus_R.conf` | 1, 4 |
| `config/boards/shields/Test/Cygnus_L.conf` | 1, 4 |
| `config/boards/shields/Test/Cygnus_R.overlay` | 1, 4 |
| `config/boards/shields/Test/Cygnus.dtsi` | 1 |
| `config/boards/shields/Test/Kconfig.defconfig` | 1 |
| `docs/firmware-comparison-cygnus-mona2.md` | 参照のみ |
| `docs/shakupan-implementation-deep-dive.md` | 参照のみ |
| `README.md` | 運用手順の追記（Phase 2 以降） |

ハードウェア変更は想定しない（配線が現行 Cygnus のままである前提）。

---

## 12. 作業時のチェック手順（毎回）

1. 変更を push（または workflow_dispatch）
2. Actions 成功を確認
3. Artifacts から UF2 取得
4. 右（Central）を更新 → 動作確認
5. 必要なら左も更新
6. 問題があれば settings_reset → 再ペア → 再確認

BT 連動を試すとき:

1. いったん BT クリアが必要か判断（レイヤー設計を大きく変えた直後はリセットしやすい）
2. Win → Mac → iOS の順にペア
3. 切替マクロで往復し、修飾キーだけ重点確認

---

## 13. 今すぐやること / まだやらないこと

### 今やる

- 本ロードマップを方針の正本として使う
- **Phase 0.5: Studio 表示を見ながら現行キーマップをこのリポへ書き起こす**
- 正本化が終わってから Phase 1（基盤）→ Phase 2（Win/Mac）へ進む

### まだやらない

- shakupan / sayu の UF2 を Cygnus にそのまま焼く
- キー配列の全面刷新と基盤移行の同時実行
- ハードウェア購入・基板交換（通常不要）
- 正本化前に OS 三系統レイヤーだけ先に実装する（手癖と衝突する）
- Studio と Git の二重管理を続ける

---

## 14. 参考リンク

- 比較調査: [firmware-comparison-cygnus-mona2.md](./firmware-comparison-cygnus-mona2.md)
- shakupan 詳細: [shakupan-implementation-deep-dive.md](./shakupan-implementation-deep-dive.md)
- shakupan 設定: https://github.com/shakushakupanda/zmk-config-moNa2-v2
- sayu 公式ベース: https://github.com/sayu-hub/zmk-config-moNa2-v2
- shakupan 解説: https://zenn.dev/shakupan/articles/261ce435251607
- ZMK Input Processors: https://zmk.dev/docs/keymaps/input-processors

---

*この文書は「今後の作業用の設計メモ」です。実装が進んだらチェックボックスとレイヤー番号を実績に合わせて更新する。*
