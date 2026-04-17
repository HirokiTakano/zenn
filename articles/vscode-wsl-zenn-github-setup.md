---
title: "VSCode + WSL + GitHub + Zennで記事投稿環境を整える"
emoji: "📝"
type: "tech"
topics: [vscode, wsl, github, zenn, windows]
published: false
---

# はじめに

最近、自分でも記事を書いてアウトプットするようになりました。  
最初は「どうせなら Zenn に書いてみよう」と思ったのですが、思いのほか **環境構築でつまずくポイントが多かった** ので、この記事にまとめておきます。

「VSCode は使ったことあるけど GitHub はまだ触ったことない」という方に向けて、できるだけ丁寧に書きました。  
ゴールは **記事を書いて `git push` するだけで Zenn に投稿できる環境を作ること** です。

:::message
この記事の対象環境
- OS: Windows 10 / 11
- WSL2（Ubuntu）
- VSCode 最新版
:::

---

# 全体の流れ

この記事では以下の順番で進めます。

1. VSCode のインストール
2. WSL のインストール
3. VSCode の拡張機能を入れる（日本語化・WSL 連携）
4. WSL 内に作業ディレクトリを作る
5. GitHub アカウントの準備と Git の初期設定
6. Zenn の準備（アカウント・GitHub 連携）
7. Zenn CLI のセットアップ
8. 記事を書いて GitHub に push する

---

# 1. VSCode をインストールする

まず公式サイトから VSCode をダウンロードします。

👉 https://code.visualstudio.com/

「Download for Windows」ボタンをクリックしてインストーラーを取得し、指示に従ってインストールしてください。  
特にカスタマイズは不要です。デフォルト設定のまま進めて大丈夫です。

---

# 2. WSL をインストールする

WSL（Windows Subsystem for Linux）は、Windows 上で Linux 環境を動かすための仕組みです。  
「Mac っぽい黒い画面で Linux コマンドを叩けるようにする」と思っておけば最初は十分です。

:::message alert
VSCode の拡張機能「WSL」は、すでにインストールされた WSL に **接続するためのもの** です。  
WSL 本体のインストールは、PowerShell から行います。
:::

PowerShell を **管理者権限** で開き、以下を実行します。

```powershell
wsl --install
```

インストールが完了したら、**PC を再起動** します。

再起動後、自動的に Ubuntu のセットアップが始まります。  
ユーザー名とパスワードを求められるので、任意のものを設定してください。

:::message
パスワード入力中は画面に何も表示されませんが、正しく入力されています。入力後に Enter を押してください。
:::

インストールの確認は PowerShell で以下のコマンドで行えます。

```powershell
wsl --list --verbose
```

`Ubuntu` が表示されていれば成功です。

---

# 3. VSCode に拡張機能を入れる

VSCode を開き、左側のアイコンメニューから **拡張機能（四角いアイコン）** をクリックします。  
または `Ctrl + Shift + X` で拡張機能パネルを開けます。

## 3-1. 日本語化（Japanese Language Pack）

検索欄に `Japanese Language Pack` と入力し、Microsoft 公式のものをインストールします。

インストール後、右下に「Restart」または「再起動して適用」のボタンが出るのでクリックします。  
再起動後、VSCode が日本語表示になります。

## 3-2. WSL 拡張機能

検索欄に `WSL` と入力し、Microsoft 公式の「WSL」拡張機能をインストールします。

この拡張機能を入れると、VSCode から WSL（Linux 環境）に直接アクセスできるようになります。

---

# 4. VSCode から WSL に接続する

拡張機能を入れたら、VSCode から WSL に接続してみましょう。

VSCode 左下の **緑色のアイコン（`><`）** をクリックし、「WSL への接続」を選択します。

接続が完了すると、左下が `WSL: Ubuntu` という表示に変わります。  
この状態になれば、VSCode のターミナルから Linux コマンドが使えます。

ターミナルは `Ctrl + @` （または上のメニューから「ターミナル」→「新しいターミナル」）で開きます。

---

# 5. 作業ディレクトリを作成する

WSL のターミナルで、ホームディレクトリ配下に作業用ディレクトリを作ります。

```bash
# ホームディレクトリに移動（デフォルトでここにいるはずですが念のため）
cd ~

# work ディレクトリを作成
mkdir work

# work 配下に zenn ディレクトリを作成
mkdir work/zenn

# 確認
ls ~/work
```

`zenn` と表示されれば OK です。

---

# 6. GitHub の準備をする

## 6-1. GitHub アカウントを作成する

まだアカウントがない方は、以下から無料で作成できます。

👉 https://github.com/

## 6-2. Git の初期設定

WSL のターミナルで Git にユーザー情報を登録します。  
GitHub に登録したメールアドレスとユーザー名を使いましょう。

```bash
git config --global user.name "あなたのGitHubユーザー名"
git config --global user.email "あなたのメールアドレス"
```

設定を確認するには：

```bash
git config --global --list
```

## 6-3. SSH キーを作成して GitHub に登録する

GitHub と安全に通信するために SSH キーを作成します。

```bash
# SSH キーを生成（Enter を 3 回押せば OK）
ssh-keygen -t ed25519 -C "あなたのメールアドレス"

# 公開鍵を表示する
cat ~/.ssh/id_ed25519.pub
```

表示された文字列（`ssh-ed25519 AAAA...` で始まる1行）をコピーします。

次に GitHub の **Settings → SSH and GPG keys → New SSH key** を開き、コピーした鍵を貼り付けて保存します。

接続テストをして確認します。

```bash
ssh -T git@github.com
```

`Hi ユーザー名! You've successfully authenticated...` と表示されれば成功です。

---

# 7. GitHub にリポジトリを作成して連携する

## 7-1. GitHub にリポジトリを作る

GitHub にログインし、右上の「+」→「New repository」からリポジトリを新規作成します。

- **Repository name**: `zenn`
- **Public** または **Private**（Zenn との連携は Public でも Private でも OK です）
- README、.gitignore、License は **追加しない**（空のリポジトリにする）

「Create repository」をクリックします。

## 7-2. ローカルと GitHub を連携する

VSCode のターミナルで zenn ディレクトリに移動して、Git の初期化とリモート登録を行います。

```bash
cd ~/work/zenn

# Git リポジトリを初期化
git init

# リモートリポジトリを登録（あなたのGitHubユーザー名に変えてください）
git remote add origin git@github.com:あなたのGitHubユーザー名/zenn.git

# ブランチ名を main に変更
git branch -M main
```

---

# 8. Zenn の準備をする

## 8-1. Zenn アカウントを作成する

👉 https://zenn.dev/

GitHub アカウントでそのままログインできます。

## 8-2. Zenn と GitHub リポジトリを連携する

Zenn のダッシュボードから **「GitHubからのデプロイ」** を設定します。

1. ダッシュボード → 右上のアカウントアイコン → **「GitHub連携」**
2. 先ほど作った `zenn` リポジトリを選択して連携

これで、リポジトリに push するだけで Zenn に記事が反映されるようになります。

---

# 9. Zenn CLI をセットアップする

Zenn CLI は、記事のファイルを管理・プレビューするためのツールです。

## 9-1. Node.js をインストールする

Zenn CLI は Node.js が必要です。WSL に入っていない場合はインストールします。

```bash
# Node.js のバージョン管理ツール（nvm）をインストール
curl -o- https://raw.githubusercontent.com/nvm-sh/nvm/v0.39.7/install.sh | bash

# ターミナルを再読み込み
source ~/.bashrc

# 安定版の Node.js をインストール
nvm install --lts

# バージョン確認
node -v
npm -v
```

## 9-2. Zenn CLI をインストールする

```bash
cd ~/work/zenn

# package.json を作成（すべてデフォルトで Enter を押せば OK）
npm init --yes

# Zenn CLI をインストール
npm install zenn-cli

# Zenn 用のディレクトリ構成を作成
npx zenn init
```

`init` が成功すると、以下のような構成になります。

```
~/work/zenn/
├── articles/     ← 記事ファイルを置く場所
├── books/        ← 本のファイルを置く場所
├── .gitignore
└── README.md
```

## 9-3. プレビューを確認する

```bash
npx zenn preview
```

ブラウザで `http://localhost:8000` を開くと、記事のプレビューが確認できます。  
`Ctrl + C` で停止できます。

---

# 10. 記事を書く

## 10-1. 記事ファイルを作成する

```bash
npx zenn new:article
```

`articles/` 配下に `ランダムなスラッグ.md` というファイルが作成されます。

ファイルを開くと、以下のような frontmatter（記事の設定）が書かれています。

```yaml
---
title: ""           ← 記事タイトル
emoji: "😊"         ← サムネイルの絵文字
type: "tech"        ← tech: 技術記事 / idea: アイデア
topics: []          ← タグ（例: [vscode, github]）
published: false    ← true にすると公開される
---
```

`---` 以下から本文を書き始めます。Markdown 記法が使えます。

## 10-2. 記事を書いてプレビューする

```bash
npx zenn preview
```

プレビューを起動しながら記事を書くと、リアルタイムで見た目を確認できます。

---

# 11. GitHub に push して Zenn に公開する

記事が書けたら、`published: false` に変更します。

```yaml
published: false
```

あとは以下のコマンドで GitHub に push するだけです。

```bash
cd ~/work/zenn

# 変更をステージングする
git add .

# コミットする
git commit -m "Add new article"

# GitHub に push する
git push origin main
```

push が成功したら、数秒〜数十秒後に Zenn の記事ページに反映されます。  
Zenn ダッシュボードの「記事の管理」から確認してみてください。

---

# まとめ

この記事で構築した環境をまとめます。

| ツール | 役割 |
|---|---|
| VSCode | テキストエディタ・統合開発環境 |
| WSL（Ubuntu） | Windows 上で動く Linux 環境 |
| Git / GitHub | ソースコード・記事の管理とホスティング |
| Zenn CLI | 記事の雛形作成・プレビュー |

環境が整ってしまえば、あとは `npx zenn new:article` で記事を書いて `git push` するだけです。  
「記事を書く」以外の手間をできる限り省いた構成なので、ぜひ活用してみてください。

最初の環境構築は少し大変ですが、一度整えてしまえば快適にアウトプットできます。  
この記事が参考になれば嬉しいです。

---

# 参考リンク

- [Zenn CLI の使い方 – Zenn 公式](https://zenn.dev/zenn/articles/zenn-cli-guide)
- [GitHub SSH 接続の設定 – GitHub Docs](https://docs.github.com/ja/authentication/connecting-to-github-with-ssh)
- [WSL のインストール – Microsoft Docs](https://learn.microsoft.com/ja-jp/windows/wsl/install)
