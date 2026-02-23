# プロジェクト概要 — 1llum1n4t1s.github.io

## サイト構成

このリポジトリは **GitHub Pages** でホスティングされる静的サイトで、以下の2つの役割を持つ。

| 役割 | 対象ファイル | 公開URL |
|------|-------------|---------|
| ポートフォリオサイト（index.html） | `index.html`, `about.html`, `favicon.png` | https://1llum1n4t1s.github.io/ |
| はてなブログ「ゆろぐ」のテーマCSS | `Hatena-Blog-Theme-Boilerplate/` 配下 | https://1llum1n4t1.org/ に適用 |

## フォルダ構成

```
1llum1n4t1s.github.io/
├── index.html                      # ポートフォリオ（フリーソフト紹介ページ）
├── about.html                      # アバウトページ
├── favicon.png                     # ファビコン
├── doc/                            # ★ 本ドキュメント
├── .claude/
│   └── launch.json                 # Claude Preview用のdev server設定
├── Hatena-Blog-Theme-Boilerplate/  # はてなブログテーマ
│   ├── scss/                       # ★ SCSSソース（編集はこちら）
│   │   ├── boilerplate.scss        # エントリポイント（@import順序定義）
│   │   └── lib/
│   │       ├── _variable.scss      # 変数（ブレイクポイント・サイズ・radius等）
│   │       ├── _theme.scss         # CSS Custom Properties（ライト/ダーク定義）
│   │       ├── _animations.scss    # @keyframes定義
│   │       ├── _core.scss          # レイアウト・記事・サイドバー・フッター
│   │       └── _components.scss    # カテゴリタグ・グロー・ボタン等UI部品
│   ├── build/
│   │   └── boilerplate.css         # ★ ビルド成果物（はてなブログに貼り付けるCSS）
│   ├── package.json                # npm scripts（build / start）
│   ├── vite.config.js              # Vite設定（SCSS→CSS、autoprefixer）
│   └── server.js                   # ローカルプレビュー用サーバー
├── _server.js                      # Claude検証用簡易HTTPサーバー（一時ファイル）
└── _test_hatena_bg.html            # Claude検証用テストHTML（一時ファイル）
```

## はてなブログテーマ「Illuminatis」

### 概要

- **テーマ名**: Illuminatis
- **ベースデザイン**: `index.html` のダークテック・モダンデザインを移植
- **対応モード**: ライト/ダーク自動切替（`prefers-color-scheme`）
- **レイアウト**: 1カラム構成（CSS Gridでサイドバーをフッター下に配置）
- **ライセンス**: MIT

### デザインの特徴

- ダークモード: `#0a0e1a` ベースの深いダーク背景
- ライトモード: `#f8fafc` ベースの明るい背景
- シアン/ブルー/パープルのグラデーションアクセント
- カードスタイルの記事一覧（ホバーで浮き上がり＋グラデーションライン）
- 背景にシアン＆ブルーのグロー（放射状グラデーション）が浮遊
- ノイズオーバーレイ（`body::before`でSVGフラクタルノイズ）
- フォント: DM Sans / Noto Sans JP / JetBrains Mono

### はてなブログのHTML構造

はてなブログはHTML構造が固定されており、CSSでのみカスタマイズする。主要なDOM構造：

```
body
├── #container
│   ├── #blog-title          // ブログタイトル・説明
│   ├── .breadcrumb           // パンくずリスト（記事ページのみ）
│   └── #content-inner        // ★ CSS Gridコンテナ
│       ├── #wrapper          // 記事エリア
│       │   └── #main-inner
│       │       ├── .archive-entries  // 一覧ページ: カード群
│       │       └── .entry           // 記事ページ: 本文
│       └── #box2             // サイドバー（display:contentsで展開）
│           └── #box2-inner
│               ├── .hatena-module-search-box  // 検索バー
│               ├── .hatena-module-profile     // プロフィール
│               └── .hatena-module (各種)      // カテゴリ・アーカイブ等
└── #footer
```

### レイアウト戦略

1. `#content-inner` を `display: grid` にする
2. `#box2`, `#box2-inner` を `display: contents` で展開
3. `order` プロパティで並び順を制御:
   - 検索バー (`order: -2`) → 記事 (`order: -1`) → サイドバーモジュール (`order: 0`)
4. モバイル: 1カラム / デスクトップ(768px+): 2カラムグリッド

### CSS Custom Properties（テーマ変数）

`:root` に定義。`@media (prefers-color-scheme: dark)` でダーク値に切替。

| 変数 | 用途 | ライト値 | ダーク値 |
|------|------|---------|---------|
| `--bg-deep` | ページ背景 | `#f8fafc` | `#0a0e1a` |
| `--bg-card` | カード背景 | `#ffffff` | `#111827` |
| `--accent-cyan` | 主要アクセント | `#0891b2` | `#22d3ee` |
| `--accent-blue` | 副アクセント | `#2563eb` | `#3b82f6` |
| `--text-primary` | 本文色 | `#0f172a` | `#f1f5f9` |
| `--glow-cyan` | グロー効果 | `rgba(8,145,178,0.08)` | `rgba(34,211,238,0.15)` |

（全変数は `scss/lib/_theme.scss` を参照）

### ブレイクポイント

| 変数 | 値 | 用途 |
|------|-----|------|
| `$mq-xs` | `max-width: 480px` | 超小型スマホ |
| `$mq-sm` | `min-width: 768px` | タブレット以上（2カラム化） |
| `$mq-md` | `min-width: 992px` | デスクトップ |
| `$mq-lg` | `min-width: 1200px` | ワイドスクリーン |

## ビルド方法

```bash
cd Hatena-Blog-Theme-Boilerplate
npm install       # 初回のみ
npm run build     # scss/ → build/boilerplate.css を生成
```

- Vite + autoprefixer でSCSS→CSSコンパイル
- `cssMinify: false` なので出力は未圧縮（可読性優先）

## はてなブログへの適用手順

1. `build/boilerplate.css` の内容をコピー
2. はてなブログ管理画面 → デザイン → カスタマイズ → デザインCSS に貼り付け
3. 保存して公開
