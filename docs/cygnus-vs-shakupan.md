# Cygnus と shakupan（moNa2）の差分

更新日: 2026-08-09  
対象:

| 略称 | リポジトリ |
|---|---|
| **Cygnus** | 本リポジトリ `Cygnus-S-Lkeymouse`（現行） |
| **shakupan** | [shakushakupanda/zmk-config-moNa2-v2](https://github.com/shakushakupanda/zmk-config-moNa2-v2) |

キーマップ図（現行 Cygnus）: [cygnus-keymap-visual.md](./cygnus-keymap-visual.md)

---

## 1. いまの関係

Cygnus は **shakupan と同じ新世代スタック**（badjeff PMW3610 + ZMK Pointing + input processors）に寄せたうえで、**キー配列と BT×OS の運用は Cygnus 独自**にしている。

- ハードウェア（XIAO BLE / SPI ピン）は moNa2 系と近い
- moNa2 の UF2 を Cygnus に焼くことはしない
- 配列の丸コピーもしない

---

## 2. スタック（似ているところ）

| 項目 | Cygnus（現行） | shakupan |
|---|---|---|
| ZMK | `v0.3` | `v0.3.0` |
| PMW3610 | `badjeff` `@ zmk-0.3` | 同左 |
| Pointing | `CONFIG_ZMK_POINTING` | 同左 |
| スクロール方式 | listener の `scroller` + `zip_*` | 同左 |
| オートマウス | **使わない** | 使わない |
| CPI | **600**（overlay） | 600 |
| keybind モジュール | `zettaface` あり | あり |

### トラックボールまわりの差

| 項目 | Cygnus | shakupan |
|---|---|---|
| スクロール有効レイヤー | **6 / 7**（`win_scroll` / `mac_scroll`） | 6 / 7（同趣旨） |
| スクロール倍率 | `zip_scroll_scaler 1 25`（shakupan 1/20 より少し遅い） | `1 20` |
| センサ invert | 当面オフ（実機で調整） | `invert-x` / `invert-y` 有効（COROPIT 前提） |
| listener の常時変換 | Y 反転 | Y 反転など |
| BLE 間隔 | **固定 7.5ms**（min=max=6） | 7.5〜15ms |
| USB HID | 特別指定なし | **1000Hz** 指定あり |

---

## 3. レイヤー構成の差

| # | Cygnus | shakupan |
|---|---|---|
| 0 | `win_base` | `win_layer` |
| 1 | `mac_base` | `mac_layer` |
| 2 | `num`（**OS 共用**） | `num_win` |
| 3 | `symbol`（**OS 共用**） | `num_mac` |
| 4 | `win_mouse`（ユーザー定義） | `mouse_win`（shakupan 配列） |
| 5 | `mac_mouse`（ユーザー定義・Gui 版） | `mouse_mac` |
| 6 | `win_scroll`（≈ win_mouse） | `scroll_win` |
| 7 | `mac_scroll`（≈ mac_mouse） | `scroll_mac` |
| 8 | `ble`（**1 層**） | `function_win` |
| 9 | — | `function_mac` |
| 10 | — | `ble_win` |
| 11 | — | `ble_mac` |

要点:

- Cygnus は **NUM / SYMBOL を OS で分けない**
- Cygnus に function 層は無い
- BLE は **Win/Mac 共通の 1 層**（入口は `CARET` 長押し）

---

## 4. BT × OS 連動の差

| 項目 | Cygnus | shakupan |
|---|---|---|
| 切替方式 | `tog_off` → `BT_SEL` → `tog_on` | 同型 |
| BT0 | **Windows** → `win_base` | Windows → win |
| BT1 | **Mac** → `mac_base` | Mac → mac |
| BT2 | **Mac（2 台目）** → `mac_base` | 素の `BT_SEL`（同期なし） |
| BT3/4 | 素の `BT_SEL` | 素の `BT_SEL` |
| マクロ名 | `BT_WIN` / `BT_MAC1` / `BT_MAC2` | `BT0` / `BT1` |

---

## 5. キーマップ方針の差

| 層 | Cygnus | shakupan |
|---|---|---|
| 文字ベース | ユーザー配列。修飾は **Win/Mac とも Ctrl / Win(GUI) / Alt** | Win/Mac で親指修飾が異なる |
| 数字・記号 | 写真ベースの独自 NUM / SYMBOL | num_win / num_mac |
| マウス | **ユーザー定義**（旧 L1）。shakupan mouse は不採用 | 独自の編集・スクショ配置 |
| スクロール | **各 mouse と同内容**（ボールだけホイール化） | mouse に近いが別配列あり |
| BLE 入口 | **`CARET` 長押し** | `/` 長押し（OS 別 ble 層） |
| 右下 `/` | `&mt RIGHT_SHIFT SLASH` | ble 層へ |

---

## 6. 意図的に寄せていないもの

1. shakupan のキー配列そのもの
2. COROPIT 向け invert-x/y
3. USB 1000Hz
4. function 層・ble の OS 二重化
5. iOS 三系統（Cygnus は当面 BT1/BT2 とも Mac）

---

## 7. 参照

- 現行キーマップ: [`config/Cygnus.keymap`](../config/Cygnus.keymap)
- 現行トラボ設定: [`config/boards/shields/Test/Cygnus_R.overlay`](../config/boards/shields/Test/Cygnus_R.overlay)
- shakupan 上流: https://github.com/shakushakupanda/zmk-config-moNa2-v2
