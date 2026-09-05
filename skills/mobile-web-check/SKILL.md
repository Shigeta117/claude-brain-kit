---
name: mobile-web-check
description: モバイルWeb/PWA の技術チェックリスト。スマホ向け画面・PWA・モバイルファーストの実装やレビューのとき、iOSズーム・キーボード追従・safe-area・PWA更新検知・状態保持などを「後追い修正」でなく実装時に潰すために使う。
---

# MOBILE-WEB-CHECK — モバイル/PWA 技術チェックリスト

4つの実プロジェクトで、下記の項目が**すべて「後追い修正」になった**実測から逆算したリスト。
どれも実装時に3分で潰せるのに、リリース後に踏むと修正コストが跳ねる。

モバイル向け実装の前に該当項目を確認し、レビュー時はこの表に照合する。

## 入力・キーボード

- [ ] **input/textarea の font-size は 16px 以上か**
  iOS Safari の自動ズームの発火条件。`maximum-scale=1` での抑止は Android のピンチズーム
  （アクセシビリティ）まで殺すので最終手段。個別に `text-base` を当てて回ると必ず漏れるので、
  グローバルCSS 1箇所で潰す:
  ```css
  @supports (-webkit-touch-callout: none) {
    input, textarea, select { font-size: 16px !important; }
  }
  ```
  `!important` が必須。Tailwind の `.text-sm` はクラスセレクタなので、要素セレクタだけでは
  特異性で負けて「font-size 未指定の textarea は直るのに `text-sm` 付き input はズームが残る」
  という中途半端な状態になる
- [ ] **`input[type="date"] / [type="time"]` がコンテナからはみ出していないか**
  WebKit 内部の日付UI chrome が独自の最小幅制約を持ち、`width: 100%` より優先される。
  `-webkit-appearance: none; appearance: none; min-width: 0;` で解除（ネイティブピッカーは動く）
- [ ] キーボード出現時に入力欄が隠れないか（`visualViewport` / meta `interactive-widget=resizes-content`）
- [ ] 意図しない自動フォーカスがないか（iOS でキーボードが勝手に開く）

## レイアウト

- [ ] safe-area 対応（`env(safe-area-inset-*)`）。ダイナミックアイランド・ノッチと固定ヘッダーの重なり
- [ ] モーダル・シート表示中の背面スクロールロック（`overscroll-behavior: contain` + body 固定）
- [ ] 下部固定バーがホームインジケータと重ならないか
- [ ] タップターゲット 44px 以上

## PWA

- [ ] **Service Worker の更新検知と再読み込み導線**
  これが無いと「更新が永遠に当たらない」事故が起きる。ユーザーは古いアプリを使い続け、
  こちらは新しいコードで動いている前提でサポートすることになる
- [ ] `display-mode: standalone` 検知で UI 出し分け（PWA 時にブラウザ用ボタンを隠す等）
- [ ] **アプリ内ブラウザ（LINE・X 等）での動作確認**。認証リダイレクトとストレージ制約で落ちる

## 挙動

- [ ] 連続表示画面（動画・ライブ集計）に Wake Lock
- [ ] **データ更新・再取得でソート順・フィルタ・スクロール位置がリセットされないか**
  状態保持は既定。「更新したら一覧が先頭に戻る」は、実際に使うと毎回イラつく類の欠陥で、
  レビューでは見つからず実運用で必ず刺さる
- [ ] **無料枠の呼び出し量見積り**: ポーリング間隔 × ユーザー数 × 期間を、機能追加**前**に概算する。
  マネージドDBの読み取り課金は、実装時の1行（`setInterval` の秒数）で桁が変わる

## 表現

- [ ] UI 要素に絵文字を使っていないか（通知の文章内は可）
- [ ] 防御的な注意書き・説明過多の UI テキストがないか

## 検証

実機の iOS Safari で確認する。**シミュレータ・PC DevTools のエミュレーションでは
上記のズーム系・safe-area 系は再現しない。**
