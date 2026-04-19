---
title: "ThinkSpeed 技術解説 — 0から100まで"
emoji: "⚡"
type: "tech"
topics: [nextjs, react, typescript, tailwindcss, tiptap]
published: false
---

このドキュメントは、ThinkSpeed というアプリケーションを構成する技術・設計・コードの全体像を、
プログラミング知識ゼロの状態から理解できるよう、順を追って解説します。

---

## 目次

1. [そもそも「Webアプリ」とは何か](#1-そもそもwebアプリとは何か)
2. [使用している言語とは](#2-使用している言語とは)
3. [フレームワーク・ライブラリの全体像](#3-フレームワークライブラリの全体像)
4. [プロジェクトのファイル構成](#4-プロジェクトのファイル構成)
5. [Next.js とは — アプリの土台](#5-nextjs-とは--アプリの土台)
6. [React とは — 画面を部品で作る仕組み](#6-react-とは--画面を部品で作る仕組み)
7. [TypeScript とは — 型付きJavaScript](#7-typescript-とは--型付きjavascript)
8. [Tailwind CSS とは — スタイルの仕組み](#8-tailwind-css-とは--スタイルの仕組み)
9. [Tiptap とは — エディタの心臓部](#9-tiptap-とは--エディタの心臓部)
10. [アプリの設計思想（アーキテクチャ）](#10-アプリの設計思想アーキテクチャ)
11. [データの流れ（データフロー）](#11-データの流れデータフロー)
12. [各ファイルの詳細解説](#12-各ファイルの詳細解説)
13. [localStorage — データ永続化の仕組み](#13-localstorage--データ永続化の仕組み)
14. [カスタム拡張機能の実装詳細](#14-カスタム拡張機能の実装詳細)
15. [CSSの設計 — なぜglobals.cssが重要か](#15-cssの設計--なぜglobalscssが重要か)
16. [デプロイメント構成（AWS Amplify）](#16-デプロイメント構成aws-amplify)
17. [セキュリティの考え方](#17-セキュリティの考え方)
18. [開発コマンドの意味](#18-開発コマンドの意味)

---

## 1. そもそも「Webアプリ」とは何か

ThinkSpeed は「Webアプリ」と呼ばれる種類のソフトウェアです。
スマホアプリやデスクトップアプリと違い、**ブラウザ（ChromeやSafari）の上で動く**アプリです。

```
あなたのPC・スマホ
  └── ブラウザ (Chrome, Safari, Firefox...)
        └── ThinkSpeed が表示・動作する
```

インストール不要で、URLを開くだけで使えます。

### サーバーとクライアントの概念

Webアプリには大きく2つの役割があります：

- **サーバー（Server）**: インターネット上のコンピュータ。HTMLやJavaScriptを配信する役割
- **クライアント（Client）**: あなたのブラウザ。受け取ったコードを実行して画面を描く

ThinkSpeed の場合、サーバーは AWS Amplify というサービスが担当し、
あなたのブラウザがコードを受け取って動かしています。

**重要**: ThinkSpeed はデータをサーバーに保存しません。
すべてのデータはあなたのブラウザの「localStorage（ローカルストレージ）」に保存されます。
そのためログイン機能が不要で、サーバーを最小構成にできます。

---

## 2. 使用している言語とは

### HTML（HyperText Markup Language）

Webページの「骨格」を作る言語です。見出し・段落・リストなどの構造を定義します。

```html
<!-- HTML の例 -->
<ul>
  <li>箇条書き1</li>
  <li>箇条書き2</li>
</ul>
```

ThinkSpeed では、Reactというフレームワークが自動的にHTMLを生成するため、
開発者が直接HTMLを書く場面は少ないです。

### CSS（Cascading Style Sheets）

Webページの「見た目（デザイン）」を決める言語です。色・フォント・余白・レイアウトなどを制御します。

```css
/* CSS の例 */
ul {
  list-style-type: disc;  /* 箇条書きの点 */
  padding-left: 1.5em;    /* 左の余白 */
}
```

### JavaScript（JS）

Webページに「動き」を付ける言語です。ボタンを押したとき・文字を入力したときなど、
ユーザーの操作に応じた処理を書きます。

```javascript
// JavaScript の例
document.getElementById('button').addEventListener('click', () => {
  alert('ボタンが押されました')
})
```

### TypeScript（TS）

JavaScript に「型（Type）」という概念を追加した言語です。
ThinkSpeed はすべてのコードをTypeScriptで書いています（`.ts` / `.tsx` ファイル）。

「型」とは、変数に入れられる値の種類を事前に宣言することです：

```typescript
// TypeScript の例
// id は文字列、name は文字列、と宣言している
type FileItem = {
  id: string
  name: string
}

// 間違った型の値を入れようとするとエラーになる
const file: FileItem = {
  id: 123,     // ❌ エラー：数字は入れられない（文字列が必要）
  name: 'メモ'  // ✅ OK
}
```

型があることで、コードを書くときにミスを早期発見できます。
AIがコードを生成する際にも、型情報があることで意図通りのコードが生成されやすくなります。

---

## 3. フレームワーク・ライブラリの全体像

「フレームワーク」とは、アプリを作るための土台となる仕組みです。
「ライブラリ」とは、特定の機能をまとめて提供するパッケージです。

```
ThinkSpeed で使っているもの（バージョン付き）
│
├── Next.js 16.2.4          ← アプリ全体の土台（フレームワーク）
│   └── React 19.2.4        ← 画面を作る仕組み（フレームワーク）
│       └── TypeScript 5.x  ← 言語（JavaScriptの拡張）
│
├── Tailwind CSS v4         ← デザインのユーティリティ（ライブラリ）
│
└── Tiptap v3               ← テキストエディタ（ライブラリ）
    ├── @tiptap/core
    ├── @tiptap/react
    ├── @tiptap/starter-kit
    ├── @tiptap/extension-placeholder
    ├── @tiptap/extension-youtube
    └── @tiptap/pm          ← ProseMirror（Tiptapの内部エンジン）
```

これらはすべて `package.json` というファイルで管理されており、
`npm install` コマンドを実行すると `node_modules/` フォルダにダウンロードされます。

---

## 4. プロジェクトのファイル構成

```
ThinkSpeed/               ← プロジェクトのルート（最上位）フォルダ
│
├── app/                  ← Next.js の「ページ」を置くフォルダ
│   ├── layout.tsx        ← 全ページ共通のHTMLの外枠（<html>, <body>タグなど）
│   ├── page.tsx          ← トップページ（ / でアクセスしたときに表示）
│   └── globals.css       ← アプリ全体に適用されるCSSスタイル
│
├── components/           ← 再利用可能なUIパーツを置くフォルダ
│   ├── Editor.tsx        ← テキストエディタ部品
│   └── Sidebar.tsx       ← 左サイドバー部品
│
├── hooks/                ← 状態管理ロジックを置くフォルダ
│   └── useStore.ts       ← フォルダ・ファイルデータの全管理
│
├── public/               ← 画像やアイコンなどの静的ファイル
│
├── next.config.ts        ← Next.js の設定ファイル
├── customHttp.yml        ← AWS Amplify 用のHTTPヘッダー設定
├── package.json          ← 使用ライブラリの一覧・バージョン管理
├── tsconfig.json         ← TypeScript の設定
├── postcss.config.mjs    ← CSS処理の設定（Tailwind が使う）
└── eslint.config.mjs     ← コード品質チェックの設定
```

### ファイルの拡張子について

| 拡張子 | 意味 |
|--------|------|
| `.ts`  | TypeScript ファイル（画面表示なし・ロジックのみ） |
| `.tsx` | TypeScript + JSX ファイル（画面表示あり） |
| `.css` | CSSスタイルファイル |
| `.yml` | YAML形式の設定ファイル |
| `.mjs` | モジュール形式のJavaScriptファイル |

「JSX」とは、JavaScript の中にHTMLのような記述を混ぜて書ける文法のことです：

```tsx
// JSX の例（.tsx ファイルの中で書ける）
function MyButton() {
  return (
    <button className="bg-blue-500 text-white">
      クリック
    </button>
  )
}
```

---

## 5. Next.js とは — アプリの土台

Next.js は React をベースにした「Webアプリフレームワーク」です。
React だけでもアプリは作れますが、Next.js を使うことで以下が追加されます：

### ファイルベースルーティング

`app/` フォルダの構造がそのままURLになります：

```
app/page.tsx        → https://あなたのサイト/
app/about/page.tsx  → https://あなたのサイト/about
```

ThinkSpeed は `app/page.tsx` の1ページのみです。

### App Router

Next.js には「App Router」と「Pages Router」の2種類があります。
ThinkSpeed は新しい方式の「App Router」を使っています。

App Router では、デフォルトでコンポーネントが「サーバーコンポーネント」として扱われます。
サーバーサイドで実行されるため、ブラウザのAPIである `localStorage` や `window` が使えません。

それを解決するために使うのが **`'use client'` ディレクティブ** です：

```typescript
'use client'  // ← この1行で「ブラウザで動かす」宣言

import { useState } from 'react'
// これ以降はブラウザ上で実行される
```

ThinkSpeed では `app/page.tsx`、`components/Editor.tsx`、`components/Sidebar.tsx`、
`hooks/useStore.ts` すべての先頭に `'use client'` があります。

### 静的ビルド（Static Generation）

`npm run build` を実行すると、Next.js はあらかじめHTMLを生成して `.next/` フォルダに保存します。
ユーザーがアクセスしたとき、このあらかじめ生成されたHTMLをそのまま返すため、非常に高速です。

ビルド出力の `○ (Static)` という表示がこれを示しています。

---

## 6. React とは — 画面を部品で作る仕組み

React は「コンポーネント（部品）」という考え方でUIを作るライブラリです。

### コンポーネントとは

コンポーネントとは、画面の一部を独立した部品として切り出したものです。

```
ThinkSpeed の画面構成

┌─────────────────────────────────────────┐
│  <Sidebar>          │  <Editor>          │
│  （サイドバー）       │  （エディタ）      │
│                     │                   │
│  ・フォルダ一覧       │  ・ファイル名      │
│  ・ファイル一覧       │  ・テキスト入力    │
│  ・エクスポート       │  ・コピーボタン    │
└─────────────────────────────────────────┘
     ↑                      ↑
  Sidebar.tsx           Editor.tsx
  というファイルに        というファイルに
  書かれている            書かれている
```

`app/page.tsx` がこれら2つのコンポーネントを組み合わせています：

```tsx
// app/page.tsx の中身（簡略版）
export default function Home() {
  return (
    <div className="flex h-screen">
      <Sidebar ... />  {/* サイドバー部品 */}
      <Editor  ... />  {/* エディタ部品  */}
    </div>
  )
}
```

### Props（プロパティ）— コンポーネントへのデータの渡し方

コンポーネント間でデータを渡す仕組みを「Props（プロパティ）」と言います。
HTMLのタグに属性を付けるのと似たような感覚です：

```tsx
// 親（page.tsx）から子（Editor）へデータを渡す
<Editor
  file={activeFile}          // 現在のファイルデータを渡す
  onChange={updateFileContent}  // 変更時の関数を渡す
/>
```

```tsx
// 子（Editor.tsx）は受け取ったデータを使う
function Editor({ file, onChange }: Props) {
  //               ↑受け取る      ↑受け取る
}
```

### State（状態）— 変化するデータの管理

画面に表示するデータで「変化するもの」は「State（状態）」として管理します。
`useState` というReactの機能を使います：

```typescript
const [copied, setCopied] = useState(false)
//     ↑現在の値   ↑値を変える関数   ↑初期値

// ボタンを押したときに状態を変える
function handleClick() {
  setCopied(true)   // これを呼ぶと画面が自動的に再描画される
}
```

Stateが変わると、Reactは差分だけ画面を更新します（Virtual DOMという仕組みです）。

### Hooks（フック）— Reactの便利機能

「Hooks」とは React が提供する便利な機能群です。名前が `use` で始まります：

| Hook名 | 役割 |
|--------|------|
| `useState` | 変化するデータを管理する |
| `useEffect` | 副作用（API呼出・DOM操作・タイマー等）を扱う |
| `useRef` | 再レンダリングをまたいで値を保持する・DOM要素を参照する |
| `useCallback` | 関数をメモ化（毎回再生成しないよう最適化）する |

ThinkSpeed で独自に定義したカスタムフックが `hooks/useStore.ts` の `useStore` です。

---

## 7. TypeScript とは — 型付きJavaScript

### なぜ型が必要か

JavaScript は型がなくても動きます。しかし大きなアプリになると：

```javascript
// JavaScript（型なし）— 実行してみないとエラーに気づかない
function updateFile(fileId, content) {
  // fileId に数字を渡してしまっても気づかない
  // content に undefined を渡してしまっても気づかない
}
```

```typescript
// TypeScript（型あり）— 書いた瞬間にエラーが出る
function updateFile(fileId: string, content: JSONContent): void {
  // fileId: string → 文字列しか渡せない
  // content: JSONContent → 定義した型の形しか渡せない
}
```

### ThinkSpeed の型定義

`hooks/useStore.ts` で定義している型：

```typescript
// ファイル1件の型
type FileItem = {
  id: string        // ユニークなID（例："abc-123-..."）
  name: string      // ファイル名（例："会議メモ"）
  content: JSONContent  // Tiptapのエディタ内容（JSON形式）
}

// フォルダ1件の型
type Folder = {
  id: string        // ユニークなID
  name: string      // フォルダ名
  files: FileItem[] // このフォルダに属するファイルの配列
}

// アプリ全体のデータ
type Store = {
  folders: Folder[]         // フォルダの配列
  activeFileId: string | null  // 今開いているファイルのID
}
```

---

## 8. Tailwind CSS とは — スタイルの仕組み

Tailwind CSS はユーティリティファーストのCSSフレームワークです。
CSSを別ファイルに書く代わりに、HTML（JSX）の中に短いクラス名を書いてスタイルを当てます。

```tsx
{/* 従来のCSSのやり方 */}
<button className="my-button">クリック</button>
// ↑別途 .my-button { background: blue; color: white; ... } を書く必要がある

{/* Tailwind のやり方 */}
<button className="bg-blue-500 text-white px-4 py-2 rounded">クリック</button>
// ↑クラス名だけでスタイルが完結する
```

よく使われるクラス名の読み方：

| クラス名 | CSSの意味 |
|---------|-----------|
| `flex` | `display: flex`（横並びレイアウト） |
| `h-screen` | `height: 100vh`（画面の高さ100%） |
| `bg-[#FAFAF8]` | `background-color: #FAFAF8`（任意の色指定） |
| `text-sm` | `font-size: 0.875rem`（小さい文字） |
| `px-4` | `padding-left/right: 1rem`（左右の余白） |
| `py-2` | `padding-top/bottom: 0.5rem`（上下の余白） |
| `rounded` | `border-radius: 0.25rem`（角丸） |
| `border` | `border: 1px solid`（枠線） |
| `overflow-hidden` | `overflow: hidden`（はみ出し非表示） |
| `truncate` | テキストをはみ出したら `...` で省略 |

### Tailwind v4 の注意点

ThinkSpeed が使っている Tailwind CSS は **v4** です。
v3 から大きく仕様が変わっており、最も影響を受けた点が `globals.css` の書き方です。

v4 では `@import "tailwindcss"` の一行で設定が完結しますが、
内部でCSSリセットが行われるため、`ul`（箇条書き）の `list-style: disc`（丸点）が消えてしまいます。
これを `globals.css` で明示的に上書きしています（詳細は後述）。

---

## 9. Tiptap とは — エディタの心臓部

Tiptap は「ヘッドレスエディタ」と呼ばれるライブラリです。
「ヘッドレス」とは「デザインを持たない」という意味で、エディタの機能だけを提供し、
見た目は完全に自分でカスタマイズできます。

### Tiptap の内部構造

```
Tiptap
  └── ProseMirror（コアエンジン）
        ├── Document（ドキュメント全体）
        │   ├── BulletList（箇条書きリスト）
        │   │   ├── ListItem（リストアイテム）
        │   │   │   └── Paragraph（段落）
        │   │   │         └── Text（テキスト）
        │   │   └── ListItem
        │   │         └── BulletList（ネストされたリスト）
        │   │               └── ...
        │   └── Paragraph
        │         └── Text
        └── Schema（どんな構造を許可するかのルール）
```

**ProseMirror（プロースミラー）** は Tiptap の内部エンジンです。
エディタの内容を「ノードのツリー構造」として管理します。
HTMLのDOMツリーに似た概念ですが、より厳密なスキーマ（構造ルール）があります。

### JSON形式での保存

Tiptap はエディタの内容を JSON形式で取り出せます（`editor.getJSON()`）。
ThinkSpeed では、このJSONを localStorage に保存しています。

例えば「- りんご\n  - 赤い」という箇条書きは次のようなJSONになります：

```json
{
  "type": "doc",
  "content": [
    {
      "type": "bulletList",
      "content": [
        {
          "type": "listItem",
          "content": [
            {
              "type": "paragraph",
              "content": [{ "type": "text", "text": "りんご" }]
            },
            {
              "type": "bulletList",
              "content": [
                {
                  "type": "listItem",
                  "content": [
                    {
                      "type": "paragraph",
                      "content": [{ "type": "text", "text": "赤い" }]
                    }
                  ]
                }
              ]
            }
          ]
        }
      ]
    }
  ]
}
```

### StarterKit — 機能の詰め合わせ

`@tiptap/starter-kit` は複数の拡張機能をまとめて提供するパッケージです：

| 機能名 | 役割 |
|--------|------|
| Document | ドキュメントのルートノード |
| Paragraph | 段落（普通の文章） |
| Text | テキストノード |
| BulletList | 箇条書きリスト |
| ListItem | リストの1項目 |
| OrderedList | 番号付きリスト |
| Bold | **太字** |
| Italic | *斜体* |
| Code | `インラインコード` |
| Strike | ~~取り消し線~~ |
| HardBreak | 強制改行 |
| History | Ctrl+Z / Ctrl+Y（undo/redo） |
| **Link** | ハイパーリンク（v3から内包） |

**重要**: Tiptap v3 では `StarterKit` の中に `Link` が含まれています。
別途 `import Link from '@tiptap/extension-link'` して追加すると「重複エラー」が起きます。
Link の設定は `StarterKit.configure({ link: { ... } })` で行います。

### 拡張機能（Extensions）

Tiptap の機能は「拡張機能（Extension）」という仕組みで追加・カスタマイズできます。
ThinkSpeed では以下を使っています：

```typescript
extensions: [
  StarterKit.configure({ link: { ... } }),  // 基本機能 + Link設定
  Youtube.extend({ priority: 200 }),         // YouTube埋め込み
  BulletListToggle,   // 独自: Ctrl+. で箇条書きトグル
  DeepIndent,         // 独自: 深いインデント対応
  LinkToggle,         // 独自: Ctrl+K でリンクトグル
  Placeholder,        // プレースホルダーテキスト表示
]
```

---

## 10. アプリの設計思想（アーキテクチャ）

### 単方向データフロー

ThinkSpeed は「単方向データフロー（Unidirectional Data Flow）」という設計を採用しています。
データは常に「親 → 子」の一方向に流れます。

```
useStore（データの源泉）
    │
    ├── store.folders ──→ Sidebar（表示する）
    │
    └── activeFile ────→ Editor（表示・編集する）
                              │
                              │ 変更があった
                              ↓
                        onChange コールバック
                              │
                              ↓
                        useStore の updateFileContent
                              │
                              ↓
                        store が更新される
                              │
                              ↓
                        localStorage に保存
                              │
                              ↓
                        React が再描画
```

この設計の利点は「データがどこから来てどこへ行くか」が常に明確なことです。

### 関心の分離（Separation of Concerns）

各ファイルが明確な役割を持っています：

| ファイル | 担当する関心事 |
|---------|--------------|
| `useStore.ts` | データ管理・永続化ロジック |
| `Sidebar.tsx` | フォルダ/ファイルのUI表示 |
| `Editor.tsx` | テキスト編集のUI |
| `page.tsx` | 上記を組み合わせるだけ |
| `globals.css` | 全体のスタイル |

---

## 11. データの流れ（データフロー）

アプリ起動から文字入力・保存までの流れを詳しく追います。

### 起動時

```
1. ユーザーがブラウザでURLを開く

2. Next.js が app/layout.tsx を描画
   → <html lang="ja"> と <body> が作られる

3. app/page.tsx が描画される
   → 'use client' があるのでブラウザで実行

4. useStore() が呼ばれる
   → useState({ folders: [], activeFileId: null }) で空の状態で初期化
   → Reactが画面を初回描画（空のサイドバーが表示）

5. useEffect(() => { setStore(loadFromStorage()) }) が実行
   → localStorage を読み込む
   → 保存データがあれば復元、なければ defaultStore() で初期データを生成
   → setStore() で状態が更新
   → Reactが再描画（フォルダ・ファイルが表示される）

6. Editor.tsx の useEditor() が初期化
   → activeFile のコンテンツをエディタに読み込む
   → フォーカスをエディタに移す
```

### 文字入力時

```
1. ユーザーがエディタに文字を入力

2. Tiptap の onUpdate コールバックが呼ばれる
   editor.getJSON() → 現在のJSONを取得

3. onChange(fileId, json) → page.tsx 経由で useStore の
   updateFileContent が呼ばれる

4. setStore(s => { ... }) でストアが更新

5. useEffect([store]) が反応
   localStorage.setItem(STORE_KEY, JSON.stringify(store))
   → localStorage に保存
```

### ファイル切り替え時

```
1. サイドバーでファイルをクリック

2. onSelectFile(fileId) → useStore の setActiveFile が呼ばれる
   → store.activeFileId が更新される

3. page.tsx で activeFile が変わる
   → Editor に新しい file が渡される

4. Editor.tsx の useEffect([editor, file]) が反応
   → activeFileIdRef.current !== file.id を検知
   → editor.commands.setContent(newContent) でエディタを書き換え
   → editor.commands.focus('start') でカーソルを先頭に移動
```

---

## 12. 各ファイルの詳細解説

### app/layout.tsx

```typescript
// アプリ全体のHTMLの外枠
export default function RootLayout({ children }) {
  return (
    <html lang="ja">  // 言語を日本語に設定
      <body>
        {children}    // ここに各ページのコンテンツが入る
      </body>
    </html>
  )
}
```

`Geist` と `Geist_Mono` は Google Fonts から読み込むフォントです。
Next.js の `next/font/google` を使うことで、パフォーマンスに最適化された方法でフォントを読み込めます。

`metadata` オブジェクトはブラウザのタブに表示されるタイトルと、
SEO（検索エンジン最適化）のための説明文を設定しています。

### app/page.tsx

```typescript
'use client'

export default function Home() {
  // useStore からデータとCRUD関数をすべて取得
  const { store, activeFile, setActiveFile, ... } = useStore()

  return (
    <div className="flex h-screen overflow-hidden bg-[#FAFAF8]">
      {/* 左サイドバー（幅240px固定） */}
      <Sidebar
        folders={store.folders}
        activeFileId={store.activeFileId}
        onSelectFile={setActiveFile}
        // ... その他のコールバック
      />
      {/* 右のメインエリア（残り全幅） */}
      <main className="flex-1 overflow-y-auto">
        <Editor file={activeFile} onChange={updateFileContent} />
      </main>
    </div>
  )
}
```

このファイルは非常にシンプルです。データの管理は `useStore` に、
UIの実装は `Sidebar` と `Editor` にそれぞれ委譲しているため、
`page.tsx` は「つなぎ役」に徹しています。

### hooks/useStore.ts

アプリの全データを管理するカスタムフックです。
「フック（Hook）」とは、React コンポーネントから呼び出せるロジックのまとまりです。

```typescript
export function useStore() {
  // ── 状態定義 ──────────────────────────────
  const [store, setStore] = useState<Store>({ folders: [], activeFileId: null })
  const hydrated = useRef(false)  // localStorage読み込み完了フラグ

  // ── 起動時：localStorage から読み込む ──────
  useEffect(() => {
    setStore(loadFromStorage())
    hydrated.current = true
  }, [])  // [] = マウント時（初回のみ）実行

  // ── 変更時：localStorage に保存 ────────────
  useEffect(() => {
    if (!hydrated.current) return  // 読み込み前は保存しない
    localStorage.setItem(STORE_KEY, JSON.stringify(store))
  }, [store])  // store が変わるたびに実行

  // ── 計算済み値 ────────────────────────────
  // 現在アクティブなファイルを全フォルダから探す
  const activeFile = store.folders
    .flatMap(f => f.files)  // [[file1, file2], [file3]] → [file1, file2, file3]
    .find(f => f.id === store.activeFileId)
    ?? null

  // ── CRUD 関数 ─────────────────────────────
  // （ファイル追加・削除・リネーム・フォルダ追加・削除・リネーム）

  // ── エクスポート / インポート ───────────────
  // ...

  return { store, activeFile, setActiveFile, ... }
}
```

#### なぜ `hydrated` が必要か

**ハイドレーション（Hydration）**という概念があります。

Next.js はサーバー側でHTMLを生成してからクライアント（ブラウザ）に送ります。
ブラウザはそのHTMLを受け取って、Reactのイベントハンドラー等を「取り付ける」処理をします。
これを「ハイドレーション」と呼びます。

問題は、`localStorage` はブラウザ専用のAPIであり、サーバーには存在しないことです。
もし初期状態を `useState(defaultStore)` のように設定すると：

```
サーバー: defaultStore() を実行 → id = "aaaa-..."（ランダムUUID）
クライアント: defaultStore() を実行 → id = "bbbb-..."（別のランダムUUID）
→ サーバーとクライアントのHTMLが食い違う → エラー！
```

これを防ぐため：
1. 初期値は常に空 `{ folders: [], activeFileId: null }` にする（サーバーとクライアントで同じ）
2. `useEffect` の中（ブラウザのみで実行）で `loadFromStorage()` を呼ぶ

```typescript
const [store, setStore] = useState<Store>({ folders: [], activeFileId: null })
// ↑ サーバーでもクライアントでも同じ空オブジェクト

useEffect(() => {
  setStore(loadFromStorage())  // ← ブラウザのみで実行
}, [])
```

#### イミュータブルな状態更新

```typescript
// ❌ NG（直接変更）— React は変化を検知できない
const updateFileContent = (fileId, content) => {
  store.folders[0].files[0].content = content  // 直接書き換えはNG
}

// ✅ OK（新しいオブジェクトを返す）
const updateFileContent = useCallback((fileId, content) => {
  setStore(s => ({
    ...s,                    // 元のstoreをコピー
    folders: s.folders.map(folder => ({
      ...folder,             // 元のフォルダをコピー
      files: folder.files.map(f =>
        f.id === fileId
          ? { ...f, content }  // 対象ファイルだけ新しいcontentで上書き
          : f                  // それ以外はそのまま
      ),
    })),
  }))
}, [])
```

`...` は「スプレッド演算子」と呼ばれ、オブジェクトの全プロパティをコピーします。
React は「参照が変わった」ことで状態の変更を検知するため、
元のオブジェクトを直接変更するのではなく、必ず新しいオブジェクトを返す必要があります。

### components/Editor.tsx

Tiptap エディタを包んだ React コンポーネントです。

```
Editor.tsx の構造
│
├── 定数 / ヘルパー
│   ├── EMPTY_BULLET     箇条書き開始用の空コンテンツ
│   ├── isEmptyDoc()     コンテンツが空か判定
│   ├── nodeToMarkdown() JSONをMarkdownテキストに変換
│   └── TiptapNode 型
│
├── カスタム拡張機能
│   ├── BulletListToggle  Ctrl+. で箇条書きトグル
│   ├── LinkToggle        Ctrl+K でリンクトグル
│   └── DeepIndent        Tab で深いインデント
│
└── Editor コンポーネント関数
    ├── useState（copied: コピー完了フラグ）
    ├── useRef（activeFileIdRef: 現在のファイルIDを追跡）
    ├── useEditor（Tiptapエディタの初期化）
    ├── useEffect（ファイル切り替え時の内容更新）
    ├── copyMarkdown（Markdownコピー処理）
    └── JSX（画面描画）
```

#### useEditor フック

Tiptap の `useEditor` は、エディタのインスタンスを作成して返す React フックです：

```typescript
const editor = useEditor({
  extensions: [ ... ],    // 使用する機能の一覧
  content: initialContent, // 初期表示するコンテンツ
  immediatelyRender: false, // サーバーサイドレンダリング対策
  editorProps: {
    attributes: {
      class: 'outliner-editor ...',  // エディタのDOM要素にクラスを付与
    },
  },
  onUpdate({ editor }) {
    // 文字が入力されるたびに呼ばれる
    onChange(fileId, editor.getJSON())
  },
})
```

#### activeFileIdRef の役割

```typescript
const activeFileIdRef = useRef<string | null>(null)
```

`useRef` で保持する値は、状態（useState）と違い「変化しても再描画を引き起こしません」。
ここでは「今エディタに表示しているファイルのID」を追跡するために使っています。

なぜ `useState` ではなく `useRef` を使うのか：
- ファイルIDが変わるたびに再描画が起きると、エディタが初期化されてしまう
- IDの追跡は「内部的なフラグ」であり、画面に表示するものではない

```typescript
onUpdate({ editor }) {
  const id = activeFileIdRef.current  // 今表示中のファイルIDを取得
  if (id) onChange(id, editor.getJSON())
}
```

ファイル切り替え直後にエディタの内容が変わると `onUpdate` が呼ばれることがあります。
その際、`activeFileIdRef` が正しいIDを示していれば、正しいファイルに保存されます。

### components/Sidebar.tsx

左サイドバーのUIコンポーネントです。

```
Sidebar の主要な状態
├── closedFolders: Set<string>  閉じているフォルダのID集合
├── editing: { id, type } | null  インライン編集中の項目
└── importFeedback: 'idle'|'ok'|'err'  インポート結果フィードバック
```

#### InlineEdit コンポーネント

フォルダ名・ファイル名のダブルクリック編集を実現する小さなコンポーネントです：

```typescript
function InlineEdit({ value, onCommit }) {
  const [val, setVal] = useState(value)  // 編集中のテキスト
  const ref = useRef<HTMLInputElement>(null)

  // マウント時にフォーカスして全選択
  useEffect(() => {
    ref.current?.focus()
    ref.current?.select()
  }, [])

  const commit = () => onCommit(val.trim() || value)  // 空なら元の名前を維持

  return (
    <input
      value={val}
      onChange={e => setVal(e.target.value)}
      onBlur={commit}                          // フォーカスが外れたら確定
      onKeyDown={e => {
        if (e.key === 'Enter') commit()        // Enterで確定
        if (e.key === 'Escape') onCommit(value) // Escapeでキャンセル
      }}
    />
  )
}
```

#### Set を使った折りたたみ状態管理

```typescript
const [closedFolders, setClosedFolders] = useState<Set<string>>(new Set())
```

`Set` は「重複のない値の集合」を表すJavaScriptの組み込みデータ構造です。
フォルダが「閉じている」かどうかを、そのIDをSetに入れているかどうかで表現しています。

```typescript
const toggleFolder = (id: string) => {
  setClosedFolders(prev => {
    const next = new Set(prev)     // コピー（直接変更NG）
    next.has(id)
      ? next.delete(id)            // 閉じていた → 開く
      : next.add(id)               // 開いていた → 閉じる
    return next
  })
}
```

---

## 13. localStorage — データ永続化の仕組み

`localStorage` はブラウザが提供するキーバリューストア（辞書）です。
ページを閉じても、PCを再起動しても、データが残り続けます（ブラウザのデータを削除するまで）。

```
localStorage のイメージ
┌────────────────────────┬────────────────────────────────┐
│ キー（Key）             │ 値（Value）                     │
├────────────────────────┼────────────────────────────────┤
│ outliner-store-v2      │ {"folders":[{"id":"...","name"  │
│                        │ :"はじめのフォルダ","files":[{  │
│                        │ ...}]}],"activeFileId":"..."}   │
└────────────────────────┴────────────────────────────────┘
```

### 保存と読み込み

```typescript
// 保存: オブジェクトをJSON文字列に変換してセット
localStorage.setItem('outliner-store-v2', JSON.stringify(store))

// 読み込み: JSON文字列をオブジェクトに戻す
const raw = localStorage.getItem('outliner-store-v2')
const store = JSON.parse(raw)
```

`JSON.stringify()` はオブジェクトを文字列に変換、`JSON.parse()` は文字列をオブジェクトに戻します。
localStorage は文字列しか保存できないため、この変換が必要です。

### マイグレーション処理

以前のバージョンのThinkSpeedでは `outliner-content` というキーで単一ファイルを保存していました。
新バージョンでフォルダ管理に移行した際、古いデータを引き継げるよう移行処理を書いています：

```typescript
function loadFromStorage(): Store {
  // まず新フォーマットを試みる
  const saved = localStorage.getItem('outliner-store-v2')
  if (saved) { ... return parsed }

  // なければ旧フォーマットを試みる（マイグレーション）
  const legacy = localStorage.getItem('outliner-content')
  if (legacy) {
    // 旧データを新フォーマットのフォルダ/ファイル構造に変換
    const folder = newFolder('はじめのフォルダ')
    folder.files[0].content = JSON.parse(legacy)
    return { folders: [folder], activeFileId: folder.files[0].id }
  }

  // どちらもなければデフォルト値
  return defaultStore()
}
```

---

## 14. カスタム拡張機能の実装詳細

### BulletListToggle（Ctrl+. 箇条書きトグル）

```typescript
const BulletListToggle = Extension.create({
  name: 'bulletListToggle',
  addKeyboardShortcuts() {
    return {
      'Mod-.': () => this.editor.commands.toggleBulletList(),
      // 'Mod-' は Mac では Cmd、Windows/Linux では Ctrl に対応
    }
  },
})
```

`Extension.create()` は Tiptap が提供する拡張機能の作成API です。
`addKeyboardShortcuts()` でキーボードショートカットを定義します。
`this.editor.commands.toggleBulletList()` は、既存のTiptapコマンドを呼び出しています。

### LinkToggle（Ctrl+K リンクトグル）

```typescript
const LinkToggle = Extension.create({
  name: 'linkToggle',
  addKeyboardShortcuts() {
    return {
      'Mod-k': () => {
        const { editor } = this

        // 既にリンクがある場合は解除
        if (editor.isActive('link')) {
          return editor.commands.unsetLink()
        }

        // 選択範囲のテキストを取得
        const { from, to } = editor.state.selection
        if (from === to) return false  // 何も選択されていなければ何もしない

        const selectedText = editor.state.doc.textBetween(from, to).trim()

        // 選択テキストがURLとして有効かチェック
        try {
          const url = new URL(selectedText)
          return editor.commands.setLink({ href: url.href, target: '_blank' })
        } catch {
          return false  // URL でなければ何もしない
        }
      },
    }
  },
})
```

`new URL(text)` はテキストが有効なURL形式か検証します。
無効なURLを渡すと例外（エラー）が投げられるため、`try...catch` で処理しています。

### DeepIndent（深いインデント対応 Tab）

これが最も複雑な実装です。ProseMirrorのスキーマ制約を理解する必要があります。

**問題**: ProseMirrorの `sinkListItem`（リストアイテムを1段深くする標準コマンド）は、
「前に兄弟アイテムが存在する」ことを前提としています。
先頭のアイテム（前に兄弟がいない）では失敗します。

**解決策**: 先頭アイテムの直前に「空の兄弟アイテム」を一時的に挿入してから `sinkListItem` を実行する

```typescript
const DeepIndent = Extension.create({
  name: 'deepIndent',
  priority: 150,  // 標準のTabキー処理（priority: 100）より先に実行
  addKeyboardShortcuts() {
    return {
      Tab: () => {
        const { editor } = this

        // ① まず標準コマンドを試みる（前に兄弟がいれば成功する）
        if (editor.commands.sinkListItem('listItem')) return true

        // ② 失敗した場合（先頭アイテム）の処理
        const { state } = editor
        const { $from } = state.selection
        const { schema } = state

        // 現在カーソルがある listItem を探す
        let liDepth = -1
        for (let d = $from.depth; d > 0; d--) {
          if ($from.node(d).type.name === 'listItem') {
            liDepth = d
            break
          }
        }
        if (liDepth < 0) return false  // listItem の中にいなければ何もしない

        // ③ 空の兄弟アイテムを直前に挿入
        // ※ listItem の中身は必ず paragraph で始まる（スキーマのルール）
        const emptyLi = schema.nodes.listItem.create(
          null,
          schema.nodes.paragraph.create()  // 空のparagraphを子に持つ
        )
        editor.view.dispatch(
          state.tr.insert($from.before(liDepth), emptyLi)
        )

        // ④ 前の兄弟ができたので sinkListItem が成功する
        editor.commands.sinkListItem('listItem')
        return true
      },
    }
  },
})
```

**スキーマ制約について**: ProseMirrorのスキーマでは `listItem` の `content` が
`paragraph block*` と定義されています。これは「最初の子は必ず `paragraph`」というルールです。
`bulletList` を `listItem` の直接の子にはできません。

### nodeToMarkdown — JSONをMarkdownに変換

```typescript
function nodeToMarkdown(node: TiptapNode, depth = 0): string {
  switch (node.type) {
    case 'doc':
      // すべての子ノードを変換して改行でつなぐ
      return (node.content ?? []).map(c => nodeToMarkdown(c, 0)).join('\n\n').trim()

    case 'bulletList':
      return (node.content ?? []).map(c => nodeToMarkdown(c, depth)).join('\n')

    case 'listItem': {
      const parts: string[] = []
      for (const child of node.content ?? []) {
        if (child.type === 'paragraph') {
          const text = (child.content ?? []).map(c => nodeToMarkdown(c)).join('')
          parts.push(`${'  '.repeat(depth)}- ${text}`)  // "  - テキスト"
        } else if (child.type === 'bulletList') {
          parts.push(nodeToMarkdown(child, depth + 1))  // 再帰的に深くする
        }
      }
      return parts.join('\n')
    }

    case 'text': {
      let t = node.text ?? ''
      // マーク（装飾）を適用
      for (const mark of node.marks ?? []) {
        if (mark.type === 'bold')   t = `**${t}**`
        if (mark.type === 'italic') t = `*${t}*`
        if (mark.type === 'link')   t = `[${t}](${mark.attrs?.href})`
      }
      return t
    }

    // YouTube は "[YouTube Video](URL)" に変換
    case 'youtube':
      return `[YouTube Video](${node.attrs?.src})`
  }
}
```

これは「再帰関数（再帰的な処理）」の典型例です。
ツリー構造（JSONの入れ子）を処理するのに再帰が適しています：
- `doc` の中に `bulletList` がある
- `bulletList` の中に `listItem` がある
- `listItem` の中に `paragraph` と子 `bulletList` がある
- `paragraph` の中に `text` がある

それぞれのノードを適切なMarkdown記法に変換し、最終的に1つの文字列になります。

---

## 15. CSSの設計 — なぜglobals.cssが重要か

### Tailwindのプリフライト（Preflight）

Tailwind CSS はデフォルトで「プリフライト」という全体リセットCSSを適用します。
これにより、`ul`（箇条書き）の `list-style: disc`（丸点）が消えてしまいます。

```css
/* Tailwindのプリフライトがやること（概念的に） */
ul, ol {
  list-style: none;  /* ← 箇条書きの点を消す */
  padding: 0;
  margin: 0;
}
```

### Tailwind v4 のショートハンド問題

さらに Tailwind v4 では `list-style` ショートハンドに固有のバグがあります：

```css
/* ❌ NG — Tailwind v4 がコンパイル時に 'disc' を落とす */
list-style: disc outside !important;
/* コンパイル後 → list-style: outside !important; */

/* ✅ OK — 個別プロパティを使う */
list-style-type: disc !important;
list-style-position: outside;
```

このため `globals.css` で明示的に上書きしています：

```css
.outliner-editor ul,
.ProseMirror ul {
  list-style-type: disc !important;
  list-style-position: outside;
  padding-left: 1.75em;
}
.outliner-editor ul ul   { list-style-type: circle !important; }  /* 2階層目 */
.outliner-editor ul ul ul { list-style-type: square !important; } /* 3階層目 */
```

### `.ProseMirror` セレクタが必要な理由

Tiptap のエディタDOM には自動的に `.ProseMirror` クラスが付与されます。
スタイルが確実に当たるよう、両方のセレクタを指定しています。

### YouTubeの16:9アスペクト比

```css
div[data-youtube-video] {
  aspect-ratio: 16 / 9;   /* 幅に対して高さを自動計算 */
  width: 100%;
  max-width: 640px;
  position: relative;      /* iframe の絶対配置の基準 */
}

div[data-youtube-video] iframe {
  position: absolute;
  inset: 0;               /* top:0, right:0, bottom:0, left:0 の省略形 */
  width: 100% !important;
  height: 100% !important;
}
```

`aspect-ratio` は親要素の幅に基づいて高さを自動計算するCSSプロパティです。
16:9 = 横16 に対して 縦9 の比率（一般的な動画の比率）。

---

## 16. デプロイメント構成（AWS Amplify）

### AWS Amplify とは

AWS（Amazon Web Services）が提供するフロントエンドホスティングサービスです。
GitHubと連携して、`main` ブランチにコードをプッシュすると自動でビルド＆デプロイされます。

```
開発者が GitHub にプッシュ
    ↓
AWS Amplify が自動検知
    ↓
npm run build を実行
    ↓
.next/ フォルダのファイルをCDN（コンテンツ配信ネットワーク）に配置
    ↓
世界中のユーザーが高速にアクセス可能
```

### next.config.ts — CSPヘッダーの設定

CSP（Content Security Policy）とは、ブラウザに「このページからは、これらのURLにしか接続してはいけない」というルールを設定するセキュリティ機構です。

```typescript
async headers() {
  return [{
    source: '/(.*)',   // すべてのURLに適用
    headers: [{
      key: 'Content-Security-Policy',
      value: [
        "default-src 'self'",           // デフォルト: 自サイトのみ
        "frame-src 'self' https://www.youtube.com https://www.youtube-nocookie.com",
        // ↑ iframe（YouTube埋め込み）を許可するドメイン
        "img-src 'self' data: https://i.ytimg.com",
        // ↑ 画像の読み込みを許可するドメイン
      ].join('; ')
    }]
  }]
}
```

YouTube 埋め込みは `<iframe>` という仕組みを使って表示されます。
CSPで `frame-src` に YouTube のドメインを指定しないと、
ブラウザがセキュリティ上の理由でブロックしてしまいます。

### customHttp.yml — Amplify用ヘッダー設定

Next.js の `next.config.ts` はサーバー経由での動的レスポンスに適用されますが、
AWS Amplify の静的ホスティングでは `customHttp.yml` というファイルで
HTTPヘッダーを設定します。両方に同じCSPを記述しています。

```yaml
customHeaders:
  - pattern: '**/*'    # すべてのファイルに適用
    headers:
      - key: Content-Security-Policy
        value: "..."   # next.config.ts と同じ内容
```

---

## 17. セキュリティの考え方

### なぜ .gitignore が重要か

`.gitignore` は「GitHubにアップロードしてはいけないファイル」を指定します。
OSSとして公開しているため、以下のものは公開すべきではありません：

| ファイル/フォルダ | なぜ危険か |
|----------------|----------|
| `.env*` | APIキーやシークレットが入ることがある |
| `node_modules/` | 巨大すぎる（数万ファイル）・再現可能なので不要 |
| `.next/` | ビルド成果物・再現可能なので不要 |
| `team-provider-info.json` | AWSの設定情報が含まれることがある |
| `aws-exports.js` | AWSのリソース設定が含まれることがある |

### ThinkSpeed のセキュリティ上の利点

- **APIキーなし**: 外部APIを使わないため、シークレット情報がコードに入らない
- **サーバーなし**: バックエンドサーバーがないため攻撃面が少ない
- **データはローカル**: ユーザーデータがサーバーに送られないため、データ漏洩リスクがない
- **nocookie: true**: YouTube埋め込みで `youtube-nocookie.com` を使用することで
  YouTubeによるユーザー追跡を最小限に抑えている

---

## 18. 開発コマンドの意味

### npm コマンド

```bash
# 依存ライブラリをすべてインストール（node_modules/ フォルダが作られる）
npm install

# 開発サーバーを起動（http://localhost:3000 でアクセス可能）
# コードを変更すると自動的にブラウザが更新される（ホットリロード）
npm run dev

# 本番用ビルドを実行（.next/ フォルダが作られる）
npm run build

# ビルドした成果物をローカルで確認するサーバーを起動
npm start

# ESLint（コード品質チェックツール）を実行
npm run lint
```

### ビルドとは何か

`npm run build` を実行すると以下が行われます：

```
1. TypeScript のコードを JavaScript に変換（型情報は除去）
2. Next.js が各ページのHTMLを事前生成
3. JavaScript / CSS ファイルを最適化（圧縮・バンドル）
4. .next/ フォルダに出力
```

ビルド出力で `○ (Static)` と表示されるのは、このページが完全に事前生成（静的）されることを意味します。
ユーザーがアクセスするたびにサーバーで処理が行われるのではなく、すでに生成されたHTMLをそのまま返すため非常に高速です。

---

## まとめ

ThinkSpeed の技術スタックをまとめると：

```
ユーザーの操作
  ↓
React（Tiptapエディタ）がイベントを検知
  ↓
カスタム拡張機能（BulletListToggle, DeepIndent, LinkToggle）が処理
  ↓
useStore の関数がデータを更新
  ↓
React が差分を計算して画面を再描画
  ↓
localStorage にJSONとして保存
```

このアプリは「シンプルさ」を最大の設計原則にしています：
- データベースなし（localStorage のみ）
- 認証なし（アカウント不要）
- サーバーロジックなし（純粋なフロントエンド）
- UIのシンプルさ（ミニマルなデザイン）

この設計により、コードの量が少なく、バグが起きにくく、理解しやすいアプリになっています。
