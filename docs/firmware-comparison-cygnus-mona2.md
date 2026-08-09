# Cygnus / moNa2（sayu） / moNa2（shakupan）ファームウェア比較

調査日: 2026-08-08  
対象リポジトリ:

| 略称 | リポジトリ | 位置づけ |
|---|---|---|
| **Cygnus** | 本リポジトリ（`Cygnus-S-Lkeymouse`） | Cygnus 用ファーム（旧世代トラックボール実装） |
| **sayu** | [sayu-hub/zmk-config-moNa2-v2](https://github.com/sayu-hub/zmk-config-moNa2-v2) | moNa2 公式の最新ベース |
| **shakupan** | [shakushakupanda/zmk-config-moNa2-v2](https://github.com/shakushakupanda/zmk-config-moNa2-v2) | sayu を fork した個人チューニング版 |

> **関係の整理**  
> moNa2 の製品・販売は shakupan、最新ファームの公式公開起点は sayu（白湯_sayu）です。  
> shakupan リポは sayu の fork で、上流から **28 commit 先行 / 9 commit 遅れ**（diverged）しています。  
> キーマップやスクロール倍率など「使っていて効く調整」は shakupan 側に多く入っています。

---

## 1. 一言で言うと何が違うか

| 観点 | Cygnus | sayu | shakupan |
|---|---|---|---|
| 世代 | **旧世代**（ドライバ内でマウス処理） | **新世代**（ZMK Pointing + input processors） | sayu と同じ新世代 + 個人最適化 |
| カーソル速度感 | **速い**（CPI 1000） | 標準寄り（CPI 600） | 標準寄り（CPI 600） |
| オートマウス | **あり**（ドライバ機能） | 既定では使わない（コメントアウト） | 既定では使わない |
| スクロール切替 | レイヤー 5（ドライバ） | レイヤー 3（input processor） | レイヤー 6/7（Win/Mac 用） |
| キーマップ | マウス寄り・比較的シンプル | sayu 常用の実用配列 | Win/Mac 二系統など大幅カスタム |
| BLE 更新間隔 | **固定 7.5ms**（滑らか寄り） | 7.5〜15ms | 7.5〜15ms |
| USB | 特別指定なし | 特別指定なし | **HID poll 1ms** |

Cygnus はハードウェア配線（SPI ピンなど）は moNa2 系と近い一方、**トラックボールのソフトウェア実装が別系統**です。設定項目をそのままコピーしても期待どおり動きません。

---

## 2. 依存関係（`west.yml`）

ファームの「土台」はここで決まります。

| 項目 | Cygnus | sayu | shakupan |
|---|---|---|---|
| ZMK | `v0.3`（タグ） | `v0.3-branch` | `v0.3.0`（タグ） |
| PMW3610 ドライバ | **`Dist16384/zmk-pmw3610-driver` `@ main`** | **`badjeff/zmk-pmw3610-driver` `@ zmk-0.3`** | 同左（badjeff） |
| RGB LED widget | `caksoylar`（`revision` が二重定義: `main` と `v0.3`） | `v0.3-branch` | `v0.3` |
| Input Processor Keybind | なし | `zettaface/zmk-input-processor-keybind` | 同左 |

### 動作への影響

- **Cygnus**  
  旧来の PMW3610 ドライバ（inorichi 系派生）に依存。  
  オートマウス・スクロールレイヤー・CPI・閾値などを **Kconfig / DT のドライバ専用プロパティ**で制御します。  
  API 名も `CONFIG_ZMK_MOUSE` 側です。

- **sayu / shakupan**  
  ZMK v0.3 系の正式ポインティングに合わせ、sensor は相対移動を出し、  
  変換・スクロール化は **ZMK input processors** が担当します。  
  API 名は `CONFIG_ZMK_POINTING` 側です。

---

## 3. トラックボール実装アーキテクチャ

### 3.1 Cygnus（旧世代）

主要ファイル:

- `config/boards/shields/Test/Cygnus_R.overlay`
- `config/boards/shields/Test/Cygnus_R.conf`

ポイント:

1. DT ノード `trackball` に直接  
   - `automouse-layer = <4>;`  
   - `scroll-layers = <5>;`  
   を指定（**ドライバがレイヤー切替を解釈**）
2. `input-listener` は存在するが、変換パイプラインはほぼ空
3. 速度・閾値・省電力はすべて `CONFIG_PMW3610_*` で制御

```dts
trackball: trackball@0 {
    compatible = "pixart,pmw3610";
    /* ... SPI / IRQ ... */
    automouse-layer = <4>;
    scroll-layers = <5>;
};
```

### 3.2 sayu / shakupan（新世代）

主要ファイル:

- `boards/shields/mona2/mona2.dtsi`（listener 定義）
- `boards/shields/mona2/mona2_r.overlay`（sensor + scroller）
- `config/mona2_r.conf`（周辺設定）

ポイント:

1. sensor ノードは **CPI・軸反転・相対入力コード** まで
2. 動きの加工は `zmk,input-listener` + `input-processors`
3. スクロールは listener 内の `scroller { layers = <...>; ... }` で実現
4. オートマウスは `zip_temp_layer` をコメントで用意（既定 OFF）

```dts
/* sayu: 既定 */
cpi = <600>;
// invert-x;
// invert-y;

/* shakupan: COROPIT 向けに有効化済み */
cpi = <600>;
invert-x;
invert-y;
```

---

## 4. カーソル速度・滑らかさ・向き

### 4.1 CPI（速度の本命）

| | Cygnus | sayu | shakupan |
|---|---|---|---|
| CPI | **1000**（`CONFIG_PMW3610_CPI`） | **600**（overlay `cpi`） | **600** |
| 除算 | `CPI_DIVIDOR=1` | （方式が異なり、同種設定なし） | 同左 |

**体感差:** Cygnus は moNa2 系より明らかに速いです。  
一般的なトラックボール用途では 600〜800 が無難で、Cygnus の 1000 は速め寄りの設定です。

### 4.2 軸の向き

| | Cygnus | sayu | shakupan |
|---|---|---|---|
| センサ向き | `CONFIG_PMW3610_ORIENTATION_180=y` | listener で Y 反転 | 同左 + **invert-x/y 有効** |
| X 反転 | `INVERT_X=n` | コメントアウト | **有効（COROPIT）** |
| スクロール X | `INVERT_SCROLL_X=y` | scroller 内で X 反転 | 同左 |

**体感差:**  
shakupan は COROPIT 前提で軸反転が ON のため、通常モジュールのまま書くと左右/上下が逆転し得ます。  
Cygnus は 180° 回転 + スクロール X 反転で、別系統の向き合わせです。

### 4.3 BLE レポート間隔（カーソルのカクつき）

| | Cygnus | sayu | shakupan |
|---|---|---|---|
| `BT_PERIPHERAL_PREF_MIN_INT` | 6（7.5ms） | 6 | 6 |
| `BT_PERIPHERAL_PREF_MAX_INT` | **6（固定）** | **12（〜15ms）** | **12** |

**体感差:**  
Cygnus はホストが受け入れれば約 133Hz 固定になりやすく、高速移動時の段差感が減りやすいです。  
一方で消費電力は上がり気味です。sayu/shakupan はホスト都合で 15ms 側に寄ることがあります。

### 4.4 USB（有線接続時）

| | Cygnus | sayu | shakupan |
|---|---|---|---|
| USB HID poll | 未指定（既定） | 未指定 | **`CONFIG_USB_HID_POLL_INTERVAL_MS=1`（1000Hz）** |

**体感差:** 有線時のみ shakupan がより高応答になり得ます。無線比較では効きません。

### 4.5 Cygnus 固有の追加チューニング

`Cygnus_R.conf` にある旧ドライバ固有オプション:

| 設定 | 値 | 意味（概要） |
|---|---|---|
| `CONFIG_PMW3610_SCROLL_TICK` | 64 | スクロール変換の刻み |
| `CONFIG_PMW3610_AUTOMOUSE_TIMEOUT_MS` | 700 | オートマウス解除までの時間 |
| `CONFIG_PMW3610_MOVEMENT_THRESHOLD` | 100 | 動き始めの閾値 |
| `CONFIG_PMW3610_SMART_ALGORITHM` | y | 難表面向けアルゴリズム |
| `CONFIG_PMW3610_POLLING_RATE_125_SW` | y | ソフト側 125Hz 相当制限 |
| `CONFIG_PMW3610_RUN_DOWNSHIFT_TIME_MS` 等 | 設定あり | 省電力遷移タイミング |

これらは **badjeff 系（sayu/shakupan）にはそのまま存在しません**。  
同等のことをしたい場合は input processors / ZMK 側設定へ置き換える必要があります。

---

## 5. スクロール・オートマウスの動作差

### 5.1 ポインタ → スクロール切替

| | Cygnus | sayu | shakupan |
|---|---|---|---|
| 方式 | ドライバ `scroll-layers` | input processor `scroller` | 同左 |
| 有効レイヤー | **5（SCROLL）** | **3** | **6 と 7**（Win/Mac 用） |
| スクロール倍率 | `SCROLL_TICK=64`（別単位） | `zip_scroll_scaler 1 5`（約 1/5〜1/4） | **`1 20`（かなり遅い）** |

**体感差:**

- Cygnus: SCROLL レイヤーに入るとボールがスクロールになる（ドライバ処理）
- sayu: レイヤー 3 でスクロール。速度は控えめだが実用帯
- shakupan: スクロールが **かなり遅い／精密**（1/20）。誤爆しにくいが長距離スクロールは手間

### 5.2 オートマウスレイヤー

| | Cygnus | sayu | shakupan |
|---|---|---|---|
| 既定 | **ON**（layer 4、700ms） | OFF（`zip_temp_layer` コメント） | OFF |
| 動き | ボールを動かすと MOUSE レイヤーが一時有効 | 明示的にレイヤーへ入る必要あり | 同左 |

**体感差:**  
Cygnus は「ボールを転がすだけでクリックキーが使える」挙動になりやすいです。  
moNa2 新世代（特に sayu の説明）は意図しない遷移を嫌ってオートマウスを使わない方針です。

---

## 6. エンコーダ（左手ロータリー）

| | Cygnus | sayu | shakupan |
|---|---|---|---|
| `steps` | **12** | **24** | **96** |
| `triggers-per-rotation` | 10 | 10 | **24** |
| 左エンコーダ処理 | keymap の `sensor-bindings` | 同左 | sensor-bindings + **`encoder_left_listener`**（scroll mapper + scaler 1/4） |
| 左 conf の EC11 | `TRIGGER_GLOBAL_THREAD` | 同左 | **`TRIGGER_OWN_THREAD` + priority/stack 調整** |

**体感差:**

- Cygnus: 分解能が粗め（1 回転あたりのイベントが少ない傾向）
- sayu: 中間
- shakupan: 高分解能寄り + listener 経由でスクロール変換。回転感・連続スクロールの質が別物になりやすい

---

## 7. キーマップ・レイヤー設計

キー配列そのものは個人差が大きいため、**設計思想の差**に絞ります。

### 7.1 Cygnus

- レイヤー例: Default / Function / Num / Arrow / **MOUSE(4)** / **SCROLL(5)** / BT(6)
- デフォルトに `&mkp RCLK` / `&mkp LCLK` があり、**マウスボタン常駐寄り**
- `ZMK_POINTING_DEFAULT_SCRL_VAL 120`（既定より大きめ）
- オートマウス前提の MOUSE/SCROLL 分離

### 7.2 sayu

- レイヤーは少なめ（実用 4 + 無線）
- オートマウス非使用
- レイヤー 2 がマウスボタン兼用、レイヤー 3 がカーソル/スクロール
- `ZMK_POINTING_DEFAULT_SCRL_VAL 100`
- note 記事で製作者設定として公開されている系統

### 7.3 shakupan

- **Win / Mac 二系統**（`win_layer` / `mac_layer` ほか対になるレイヤー群）
- スクロール層が 6/7、BLE 層も Win/Mac 分離
- BT 切替マクロ（`BT0`/`BT1`）、スクショ・変換マクロなど独自機能が多い
- `&msc` の加速を実質オフ（即応）
- スクロールエンコーダの `tap-ms` を 30 に調整

**体感差:**  
同じ「moNa2-v2」でも、sayu と shakupan はキー体験が別物です。  
Cygnus との差分を見るとき、**トラックボール基盤（新/旧）**と**キーマップ趣味**は分けて考える必要があります。

---

## 8. ビルド・CI

| | Cygnus | sayu | shakupan |
|---|---|---|---|
| トリガー | push / PR / manual | 同左 | 同左 |
| 再利用 WF | `build-user-config.yml@v0.3` | `@v0.3.0` | `@v0.3.0` |
| 右シールド | `Cygnus_R rgbled_adapter` + Studio snippet | `mona2_r rgbled_adapter` + Studio | 同左（mona2） |
| 左シールド | `Cygnus_L rgbled_adapter` | `mona2_l rgbled_adapter` | 同左 |
| settings_reset | あり | あり | あり |
| ボード | `seeeduino_xiao_ble` | 同左 | 同左 |

ビルドの仕組み自体は三者とも同型です。成果物は Actions Artifacts の `.uf2` です。

---

## 9. ハードウェア（ピン）面

右手トラックボール SPI 周りは三者とも実質同じ配置です。

| 信号 | ピン |
|---|---|
| SCLK | P0.05 |
| SDIO (MOSI/MISO) | P0.04 |
| CS | P0.09 |
| MOTION IRQ | P0.02 |

つまり Cygnus と moNa2 の差の大半は **基板というよりファーム実装世代とチューニング**です。  
（物理レイアウト座標やキー数は Cygnus.dtsi / mona2.dtsi で差がありますが、ポインティング経路とは独立です。）

---

## 10. ユーザー視点の動作差まとめ

### カーソル

1. **Cygnus が最速寄り**（CPI 1000 + BLE 7.5ms 固定）
2. sayu / shakupan は CPI 600 で落ち着いた速度
3. 向きは shakupan（COROPIT invert）だけ別前提

### ボールでスクロール

1. Cygnus: SCROLL レイヤー（5）で切替、ドライバ処理
2. sayu: レイヤー 3、やや控えめな倍率
3. shakupan: レイヤー 6/7、**かなり精密（遅い）スクロール**

### マウスボタンへのアクセス

1. Cygnus: オートマウスで「動かすと MOUSE レイヤー」+ デフォルトにクリックキー
2. sayu: 明示レイヤー遷移（意図しない遷移を避ける）
3. shakupan: Win/Mac それぞれのマウス層を明示利用

### エンコーダ

1. Cygnus: 低 steps（12）
2. sayu: 24
3. shakupan: 96 + 専用 listener（感度・変換が独自）

---

## 11. Cygnus を moNa2 最新系に寄せるときの注意

「設定をコピーする」だけでは足りません。最低限次が必要です。

1. **ドライバを `Dist16384` → `badjeff @ zmk-0.3` に変更**（`west.yml`）
2. `CONFIG_ZMK_MOUSE` 依存から **`CONFIG_ZMK_POINTING` / input processors** へ移行
3. `automouse-layer` / `scroll-layers` / `CONFIG_PMW3610_CPI` をやめ、  
   overlay の `cpi` と listener の `scroller` / `zip_*` に置き換え
4. CPI を寄せるならまず **600〜800**
5. BLE を sayu と同じくするか、Cygnus の滑らかさ（固定 7.5ms）を残すか決める
6. キーマップは別問題として残す（基盤移行と同時に全部変えなくてよい）

比較の基準にする moNa2 側:

- **公式の土台・説明との一致** → sayu
- **実際に作り手がさらに詰めた挙動** → shakupan

---

## 12. 参照リンク

- [sayu-hub/zmk-config-moNa2-v2](https://github.com/sayu-hub/zmk-config-moNa2-v2)
- [shakushakupanda/zmk-config-moNa2-v2](https://github.com/shakushakupanda/zmk-config-moNa2-v2)
- [moNa2最新ファームウェア公開（note / 白湯_sayu）](https://note.com/pooh_polo/n/nc88afb19898a)
- [ZMK Input Processors](https://zmk.dev/docs/keymaps/input-processors)

---

## 付録: 主要設定ファイル対応表

| 関心事 | Cygnus | sayu / shakupan |
|---|---|---|
| 依存バージョン | `config/west.yml` | `config/west.yml` |
| 右ボード設定 | `config/boards/shields/Test/Cygnus_R.conf` | `config/mona2_r.conf` |
| 左ボード設定 | `config/boards/shields/Test/Cygnus_L.conf` | `config/mona2_l.conf` |
| トラックボール配線 | `.../Cygnus_R.overlay` | `boards/shields/mona2/mona2_r.overlay` |
| listener / 物理定義 | `Cygnus.dtsi` + overlay | `mona2.dtsi` + overlay |
| キーマップ | `config/Cygnus.keymap` | `config/mona2.keymap` |
| ビルド行列 | `build.yaml` | `build.yaml` |
| CI | `.github/workflows/build.yml` | 同左 |

---

*本ドキュメントは公開リポジトリの設定ファイルに基づく静的比較です。実機の OS・ホスト BLE スタック・ボールモジュール個体差により体感は変動します。*
