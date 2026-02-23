# デバッグガイド — Hatena Blog Theme "Illuminatis"

## ビルドワークフロー

### SCSS → CSS コンパイル

```bash
cd Hatena-Blog-Theme-Boilerplate
npm install       # 初回のみ
npm run build     # scss/ → build/boilerplate.css を生成
```

- **編集対象**: `scss/lib/` 配下の `.scss` ファイル
- **ビルド成果物**: `build/boilerplate.css`
- **注意**: `build/boilerplate.css` を直接編集しても次回ビルドで上書きされる。必ず SCSS ソースを編集すること

### はてなブログへの反映

1. `npm run build` で CSS を生成
2. `build/boilerplate.css` の内容をコピー
3. はてなブログ管理画面 → デザイン → カスタマイズ → デザインCSS に貼り付け
4. 保存して公開

## ローカルプレビュー

### Claude Preview（推奨）

`.claude/launch.json` で設定済み。`_server.js` が簡易HTTPサーバーとして動作する。

```
ポート: 3000
テストページ: _test_hatena_bg.html（はてなブログのDOM構造を模したHTML）
```

**制限事項**:
- Google Fonts の外部読み込みがタイムアウトすることがある
- テストHTMLはライトモード表示になることがある（`prefers-color-scheme` はOSの設定に依存）
- `file://` URLはブラウザ拡張機能で開けないため、HTTPサーバー経由で確認する

### ブラウザで直接確認

本番サイト `https://1llum1n4t1.org/` をブラウザで開き、DevTools のコンソールで CSS を一時注入してテストできる。

```javascript
// 例: 要素にスタイルを注入
document.querySelector('#container').style.zIndex = '0';

// 例: computedStyle の確認
getComputedStyle(document.querySelector('#container')).zIndex;
```

## よくある問題と解決パターン

### 1. 背景グロー（放射状グラデーション）が表示されない

**症状**: ダークモードで背景にシアン/ブルーのグロー効果が見えない

**原因**: CSS スタッキングコンテキストの問題

`#container::before` と `#container::after` に `z-index: -1` を設定しているが、親の `#container` がスタッキングコンテキストを作成していないと、pseudo-elements は `body` の背景の裏に描画されてしまう。

**解決策**: `#container` に `z-index: 0` を追加

```scss
// scss/lib/_components.scss
#container {
  position: relative;
  z-index: 0; // スタッキングコンテキストを作成
}
```

**確認方法**:

```javascript
// ブラウザコンソールで一時テスト
document.querySelector('#container').style.zIndex = '0';

// 確認: pseudo-elements の位置
getComputedStyle(document.querySelector('#container'), '::before').position;
// → "fixed" であること
```

### 2. スマホ表示で横幅が広がる（水平オーバーフロー）

**症状**: モバイルビューポートでページが横にスクロールする

**原因**: CSS Grid の `min-width: auto` デフォルト

Grid アイテム（`#wrapper`）のデフォルトは `min-width: auto` で、中身（特に幅広のテーブル）がグリッドカラムを超えて拡張してしまう。

**解決策**: `#wrapper` に `min-width: 0` を追加

```scss
// scss/lib/_core.scss
#wrapper {
  grid-column: 1 / -1;
  order: -1;
  min-width: 0; // グリッドアイテムがカラム幅を超えないようにする
}
```

**検出方法**:

```javascript
// ブラウザコンソールでオーバーフロー検出
const overflow = document.body.scrollWidth - window.innerWidth;
console.log('overflow:', overflow, 'px');
// → 0 であれば問題なし

// どの要素がオーバーフローしているか特定
document.querySelectorAll('*').forEach(el => {
  if (el.scrollWidth > window.innerWidth) {
    console.log(el.tagName, el.id || el.className, el.scrollWidth);
  }
});
```

**確認方法**:

```javascript
// 修正の一時テスト
document.querySelector('#wrapper').style.minWidth = '0';
// その後オーバーフロー検出スクリプトを再実行
```

### 3. ライト/ダークモードが切り替わらない

**確認ポイント**:
- `prefers-color-scheme` はOSのダークモード設定に依存
- DevTools → Rendering → Emulate CSS media feature で切替テスト可能
- CSS Custom Properties が `:root` に正しく定義されているか確認

```javascript
// 現在のカラースキームを確認
window.matchMedia('(prefers-color-scheme: dark)').matches;

// CSS変数の値を確認
getComputedStyle(document.documentElement).getPropertyValue('--bg-deep');
// ダーク: "#0a0e1a" / ライト: "#f8fafc"
```

## CSS デバッグテクニック

### スタッキングコンテキストの確認

```javascript
// 要素のスタッキングコンテキストを調べる
function checkStackingContext(selector) {
  const el = document.querySelector(selector);
  const cs = getComputedStyle(el);
  console.log({
    zIndex: cs.zIndex,
    position: cs.position,
    opacity: cs.opacity,
    transform: cs.transform,
    isolation: cs.isolation,
  });
}
checkStackingContext('#container');
```

スタッキングコンテキストが作成される条件:
- `position: relative/absolute/fixed` + `z-index` が `auto` 以外
- `opacity` < 1
- `transform` が `none` 以外
- `isolation: isolate`

### Grid レイアウトの確認

```javascript
// Grid コンテナの情報
function checkGrid(selector) {
  const el = document.querySelector(selector);
  const cs = getComputedStyle(el);
  console.log({
    display: cs.display,
    gridTemplateColumns: cs.gridTemplateColumns,
    gridTemplateRows: cs.gridTemplateRows,
    gap: cs.gap,
  });
  // 子要素の grid 配置
  Array.from(el.children).forEach(child => {
    const ccs = getComputedStyle(child);
    console.log(child.id || child.className, {
      gridColumn: ccs.gridColumn,
      gridRow: ccs.gridRow,
      order: ccs.order,
      minWidth: ccs.minWidth,
    });
  });
}
checkGrid('#content-inner');
```

### モバイル表示の検証

1. ブラウザの DevTools → デバイスツールバー（Ctrl+Shift+M）
2. 推奨テストサイズ: **375×812**（iPhone SE/13 mini 相当）
3. オーバーフロー検出スクリプトを実行
4. 各ブレイクポイントで表示を確認:
   - `480px` 以下: 超小型スマホ
   - `768px` 以上: 2カラム化
   - `992px` 以上: デスクトップ
   - `1200px` 以上: ワイドスクリーン

## SCSS ファイルの編集ガイド

### ファイルの役割

| ファイル | 編集するケース |
|----------|---------------|
| `_variable.scss` | ブレイクポイント・サイズ・radius の変更 |
| `_theme.scss` | 色の変更（ライト/ダーク両方を変更すること） |
| `_animations.scss` | アニメーションの追加・変更 |
| `_core.scss` | レイアウト・記事・サイドバー・フッターの変更 |
| `_components.scss` | UI部品（カテゴリタグ・グロー・ボタン等）の変更 |

### 編集時の注意事項

1. **ライト/ダーク両対応**: `_theme.scss` の変数を変更する場合、`:root` と `@media (prefers-color-scheme: dark)` の両方を更新
2. **ブレイクポイント**: `$mq-sm` (`768px`) が1カラム→2カラムの境界。レイアウト変更時は必ずモバイルとデスクトップの両方で確認
3. **はてなブログのDOM制約**: HTMLは固定。CSSのみで対応。新しいクラス名は追加できない
4. **display: contents**: `#box2`, `#box2-inner` は `display: contents` でフラット化している。これらの要素にスタイルを直接当てても効果がない

## 過去の修正履歴

| 日付 | 問題 | 原因 | 修正ファイル | 修正内容 |
|------|------|------|-------------|---------|
| 2026-02-23 | 背景グローが表示されない | スタッキングコンテキスト未作成 | `_components.scss` | `#container` に `z-index: 0` 追加 |
| 2026-02-23 | スマホで横幅オーバーフロー | Grid の `min-width: auto` | `_core.scss` | `#wrapper` に `min-width: 0` 追加 |
