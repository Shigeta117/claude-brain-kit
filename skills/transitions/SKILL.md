---
name: transitions
description: UIモーション（トランジション・アニメーション）の設計・実装・推敲用のトークン規律。UI の状態遷移（モーダル/ドロップダウン/タブ/トースト等）を実装するとき、モーションが「全部300ms ease-in-outでAIっぽい」とき、「モーション見直して/揃えて/polishして」と言われたときに使う。原典: transitions.dev（MIT © Jakub Antalík）
---

# TRANSITIONS — モーショントークンと推敲規律

原典: [Jakubantalik/transitions.dev](https://github.com/Jakubantalik/transitions.dev)（★2.9k・MIT）。27種の本番品質CSSトランジションと、その背後にある5軸のモーショントークン体系＋判断基準を、日本語のスキルとして再構成したもの。AI生成UIのモーションは「全部 300ms ease-in-out」に均一化しがちで、それ自体が"AIっぽさ"の主成分 ── このスキルはそれを開閉非対称・用途別トークンで潰す。

**コア教義: トークンは「数値が近いから」ではなく「用途が一致するから」選ぶ。** 300ms のモーダルクローズは 250ms に丸めるのではなく、「モーダルクローズ」という用途が `--duration-quick` (150ms) に定義されているから 150ms に矯正する。用途がどのトークンにも該当しない値は**変更しない**。

---

## 1. モーショントークン（5軸・値は原典ママ）

### Duration
| トークン | 値 | 用途 |
|---|---|---|
| `--duration-stagger` | 40ms | アイテム毎の stagger オフセット |
| `--duration-micro` | 80ms | tooltip 遅延・shake セグメント・大アイテムの stagger |
| `--duration-quick` | 150ms | **modal/dropdown の close**・text swap・tooltip 表示 |
| `--duration-fast` | 250ms | icon swap・**dropdown/modal の open**・タブ・ページスライド |
| `--duration-medium` | 350ms | panel close・toast close |
| `--duration-slow` | 400ms | panel open・skeleton reveal・input clear |
| `--duration-very-slow` | 500ms | 強調の一瞬・badge 出現・text reveal・success check |

### Easing
| トークン | 値 | 用途 |
|---|---|---|
| `--ease-smooth-out` | `cubic-bezier(0.22, 1, 0.36, 1)` | 開閉・スライド・リサイズ・位置変更の既定 |
| `--ease-in-out` | `ease-in-out` | icon/text swap・reveal 系 |
| `--ease-out` | `ease-out` | tooltip |
| `--ease-linear` | `linear` | shimmer・pulse・spinner |
| `--ease-bounce` | `cubic-bezier(0.34, 1.36, 0.64, 1)` | badge pop（entrance のみ） |
| `--ease-bounce-strong` | `cubic-bezier(0.34, 3.85, 0.64, 1)` | hover-out の弾み（avatar return） |

### Distance
| トークン | 値 | 用途 |
|---|---|---|
| `--distance-micro` | 4px | text swap |
| `--distance-small` | 6px | error shake（小） |
| `--distance-base` | 8px | badge・ページスライド・error shake（大） |
| `--distance-medium` | 12px | text reveal |
| `--distance-large` | 30px | check badge 出現 |

頻度が高い動きほど距離は小さく、儀式的な瞬間ほど大きく。**40px 超の移動は鈍く感じる**（フル panel/drawer は例外）。

### Scale（「開始点」の値。常に 1 へ収束）
| トークン | 値 | 用途 |
|---|---|---|
| `--scale-large` | 0.96 | modal 開閉 |
| `--scale-medium` | 0.97 | dropdown open |
| `--scale-small` | 0.98 | tooltip open |
| `--scale-tiny` | 0.99 | dropdown close |

大きい面ほど遠く（小さい値）から。**0.9 未満は「ズーム」と読まれ UI には不適**。

### Blur（「開始点」の値。0 へ収束）
| トークン | 値 | 用途 |
|---|---|---|
| `--blur-small` | 2px | panel/skeleton reveal・icon/text swap・number pop-in |
| `--blur-medium` | 3px | ページスライド・text reveal |
| `--blur-large` | 8px | success check |

swap/slide を柔らかくする用。fade や色変化には使わない。

---

## 2. 推敲規律（polish rules・ここが判断の本体）

- **開閉は非対称**: open は招待的に遅く、close は素早く退場。dropdown/modal: open 250ms → close 150ms。panel: open 400ms → close 350ms。
  - **対称の例外**（両方向同値・分割しない）: page side-by-side / tabs / accordion / icon swap（各250ms）・text swap（150ms）＝「単一の可逆モーション」
- **close にバウンス禁止**: overshoot 曲線は entrance のみ（badge pop・number pop-in）
- **close と hover-out に遅延禁止**: 消えるものは即座に消え始める
- **hover は in が速く直線的**（`--duration-fast` 以下＋`--ease-smooth-out`）、**out は柔らかく**（`--ease-bounce-strong` 可）。out の方が in より作り込まれる唯一の場面
- **stagger は 1 アイテム 40ms・合計 300ms 未満**。長いリストはアイテム数を打ち切るかオフセットを縮める
- **delay の正当な用途は2つだけ**: 誤トリガー除け（tooltip の 80ms）と意図的なシーケンス。遅く感じたら delay でなく duration を削る

---

## 3. 決定ルール（UIの状況 → どのパターンか）

原典 27 パターン。実装時は原典リポの `skills/transitions-dev/<参照ファイル>` を Fetch して CSS を**逐語**で取り込む（raw URL: `https://raw.githubusercontent.com/Jakubantalik/transitions.dev/main/skills/transitions-dev/06-modal.md` の形式。トークン定義は同ディレクトリの `_root.css`）。

| 状況 | パターン | 参照 |
|---|---|---|
| コンテナの幅/高さが変わる | Card resize | 01 |
| 数字の更新 | Number pop-in | 02 |
| トリガーに小さいドットが浮く | Notification badge | 03 |
| 同じ場所でテキストが入れ替わる | Text states swap | 04 |
| トリガーから面が拡がる（アンカー起点） | Menu dropdown | 05 |
| 中央に面が出る | Modal open/close | 06 |
| 画面端からスライドする面 | Panel reveal | 07 |
| リスト↔詳細・ステップ1↔2 | Page side-by-side | 08 |
| 同スロットの2アイコン | Icon swap | 09 |
| 確認/成功/完了の瞬間 | Success check | 10 |
| 横並びスタックの hover | Avatar group hover | 11 |
| フォーム検証エラー | Error state shake | 12 |
| テキストフィールドのクリア | Input clear dissolve | 13 |
| プレースホルダ→実コンテンツ | Skeleton reveal | 14 |
| 進行中/考え中テキスト | Shimmer text | 15 |
| 移動するハイライトのタブ | Tabs sliding | 16 |
| hover/focus のヒント | Tooltip | 17 |
| 見出し＋補足が段階的に入る | Texts reveal | 18 |
| ポインタ追従の3Dカード | Card hover tilt | 19 |
| 丸トリガーがメニューに変形 | Plus to menu morph | 20 |
| 折りたたみヘッダー | Accordion | 21 |
| 一時メッセージ（自動で消える） | Toast | 22 |
| 有効化を祝う | Like button | 23 |
| 矢印が方向に効くインラインリンク | Learn more hover | 24 |
| 選択で塗り＋マーク描画 | Checkbox check | 25 |
| 値の間をロールするカウンタ | Spinning counter | 26 |
| 二値スイッチ | Toggle | 27 |

---

## 4. 使い方（2段階・原典の review/polish に準拠）

1. **review（監査・編集なし）**: 対象プロジェクトの `transition` / `animation` / `@keyframes` / ハードコード ms 値（inline style・CSS-in-JS・Tailwind 任意値含む）を洗い、セレクタ/コンポーネント名から用途を推定してトークン表・§2 と照合。**変更すべき値だけ**ファイル別に列挙（例: `modal close: 300ms → var(--duration-quick)` ─ close は open より短いのが正）。用途がトークンに無い値は `no matching token usage` と記して触らない
2. **polish（適用）**: 変更案の概要を出して確認を取ってから適用。`_root.css`（トークン定義）を import 済みなら `var(--…)`、未導入ならリテラル値＋導入提案。**モーション値以外は編集しない**。既存の単位表記（`0.25s` vs `250ms`）は維持
3. 新規実装は §3 で選定 → 参照ファイルを Fetch → CSS 逐語ペースト（セレクタ・shorthand・will-change を変えない）→ `prefers-reduced-motion` ブロックを必ず保持

**よくある誤り（原典の注意点の要点）**: `is-closing` クラスの削除タイムアウト漏れ／リプレイに必要なリフロー強制（`void el.offsetWidth`）忘れ／`transition: all` への置換（プロパティ列挙は意図的）／success check の `stroke-dasharray` ハードコード（`path.getTotalLength()` を使う）／ダークモードで `mix-blend-mode: multiply` 放置。

---

## 5. ライセンス・営利利用

**MIT © Jakub Antalík**（2026-07-25 に一次確認。canonical リポは LICENSE が root に無く、
確認はミラー `transitions-dev` 側で取れる）。営利・クライアント納品可。

- CSS をコピペするときはクレジットコメントを残す。納品物では THIRD-PARTY-NOTICES に載せる
- 純CSS・依存ゼロなので **vendoring が既定**（npm CLI `transitions-pro` はあるが依存にする理由が薄い）

## 6. 併用

- **`mobile-web-check`**: reduced-motion・低スペック実機での確認はあちらのチェックリストで
- 原典にはこのほか **Refine パネル**（実装中モーションのリアルタイム調整ツール、リポ `refine/`）が
  あるが未検証。使うときは原典を Fetch する

## 出典

https://github.com/Jakubantalik/transitions.dev （★2.9k）
原典スキル2本（transitions-dev / transitions-polish）のトークンと規律を統合して日本語化したもの。
**このファイルは要約であって原典ではない。** 27パターンの CSS 本体は原典リポにあるので、
実装時は §3 のとおり原典を Fetch して逐語で取り込むこと。
