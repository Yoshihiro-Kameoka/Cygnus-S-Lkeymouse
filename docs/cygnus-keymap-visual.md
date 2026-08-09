# Cygnus キーマップ図（現行）

更新日: 2026-08-09  
ソース: [`config/Cygnus.keymap`](../config/Cygnus.keymap)  
生成: [keymap-drawer](https://github.com/caksoylar/keymap-drawer)（[`keymap-drawer/cygnus/`](./keymap-drawer/cygnus/)）

差分メモ: [cygnus-vs-shakupan.md](./cygnus-vs-shakupan.md)

---

## レイヤー

| # | 図 | 名前 | 役割 |
|---|---|---|---|
| 0 | [cygnus-win_base.svg](./keymap-drawer/cygnus/cygnus-win_base.svg) | `win_base` | Windows ベース |
| 1 | [cygnus-mac_base.svg](./keymap-drawer/cygnus/cygnus-mac_base.svg) | `mac_base` | Mac ベース |
| 2 | [cygnus-num.svg](./keymap-drawer/cygnus/cygnus-num.svg) | `num` | 数字・F キー（共用） |
| 3 | [cygnus-symbol.svg](./keymap-drawer/cygnus/cygnus-symbol.svg) | `symbol` | 記号（共用） |
| 4 | [cygnus-win_mouse.svg](./keymap-drawer/cygnus/cygnus-win_mouse.svg) | `win_mouse` | Windows マウス |
| 5 | [cygnus-mac_mouse.svg](./keymap-drawer/cygnus/cygnus-mac_mouse.svg) | `mac_mouse` | Mac マウス |
| 6 | [cygnus-win_scroll.svg](./keymap-drawer/cygnus/cygnus-win_scroll.svg) | `win_scroll` | ボールスクロール（Win） |
| 7 | [cygnus-mac_scroll.svg](./keymap-drawer/cygnus/cygnus-mac_scroll.svg) | `mac_scroll` | ボールスクロール（Mac） |
| 8 | [cygnus-ble.svg](./keymap-drawer/cygnus/cygnus-ble.svg) | `ble` | BT / bootloader |

全レイヤー: [cygnus.svg](./keymap-drawer/cygnus/cygnus.svg)

---

## 入口（覚えるところ）

| 操作 | 行き先 |
|---|---|
| Space 長押し | `win_mouse` / `mac_mouse`（ベースによる） |
| `-` 長押し / 内側 LANGUAGE 長押し | scroll |
| `LANGUAGE_2` 長押し | `num` |
| Enter 長押し | `symbol` |
| `CARET`（`^`）長押し | `ble` |
| BLE 上段 `BT_WIN` / `BT_MAC1` / `BT_MAC2` | BT0=Win, BT1/2=Mac |

---

## 再生成

```bash
export PATH="$HOME/.local/bin:$PATH"
keymap parse -z config/Cygnus.keymap -o docs/keymap-drawer/cygnus/cygnus.yaml
# layout は cygnus.json（default_layout）を参照するよう調整済みなら:
keymap draw docs/keymap-drawer/cygnus/cygnus.yaml -o docs/keymap-drawer/cygnus/cygnus.svg

for layer in win_base mac_base num symbol win_mouse mac_mouse \
             win_scroll mac_scroll ble; do
  keymap draw docs/keymap-drawer/cygnus/cygnus.yaml -s "$layer" \
    -o "docs/keymap-drawer/cygnus/cygnus-${layer}.svg"
done
```
