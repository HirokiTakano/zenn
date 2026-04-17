---
title: "Gitリポジトリの初期化から作成・プッシュまでの流れ"
emoji: "🧭"
type: "tech"
topics: [git, github, zenn]
published: false
---

## はじめに

この記事では、ローカルで Git リポジトリを初期化し、GitHub にリモートリポジトリを作成して、最初のコミットをプッシュする一連の流れを解説します。

## 1. 作業ディレクトリの準備

まず、記事やプロジェクトのあるディレクトリに移動します。

```bash
cd /home/hirokitakano/work/zenn
```

このディレクトリに `git init` で Git リポジトリを初期化します。

```bash
git init
```

成功すると、`.git` フォルダが作成され、ローカルリポジトリとして管理できるようになります。

## 2. Git の初期設定

初めて Git を使う場合は、ユーザー名とメールアドレスを設定します。

```bash
git config --global user.name "Your Name"
git config --global user.email "you@example.com"
```

設定済みか確認するには:

```bash
git config --global --list
```

## 3. .gitignore の作成

不要なファイルをコミットしないように、`.gitignore` を作成しておきます。

```gitignore
/node_modules
/dist
.DS_Store
```

この例では、Node.js の依存フォルダやビルド成果物、macOS の不要ファイルを無視します。

## 4. ファイルをステージングする

現在の変更を確認して、コミットするファイルをステージングします。

```bash
git status
git add .
```

`git status` で差分を確認し、問題がなければ `git add .` で全ファイルを追加します。

## 5. 最初のコミット

初回のコミットを作成します。

```bash
git commit -m "Initial commit"
```

コミットメッセージは、後から見ても分かりやすい内容にすると良いです。

## 6. GitHub にリモートリポジトリを作成する

### 6-1. GitHub Web から作成する場合

1. GitHub にログインする
2. 右上の「New」ボタンから新しいリポジトリを作成する
3. リポジトリ名、公開 / 非公開、README の有無などを設定する
4. 作成後に表示されるリモート URL をコピーする

### 6-2. GitHub CLI を使う場合

`gh` コマンドがインストールされている場合、次のようにリポジトリを作成できます。

```bash
gh auth login

gh repo create hirokitakano/zenn --public --source=. --remote=origin
```

- `--public` は公開リポジトリを作成するオプションです。
- プライベートにしたい場合は `--private` を使います。

## 7. リモートを追加する

GitHub でリポジトリを作成したら、ローカルリポジトリにリモートを追加します。

```bash
git remote add origin https://github.com/ユーザー名/リポジトリ名.git
```

既にリモートがある場合は `git remote -v` で確認します。

## 8. ブランチ名を整える

最近は `main` をメインブランチ名にすることが一般的です。必要に応じて変更します。

```bash
git branch -M main
```

## 9. リモートにプッシュする

ローカルの `main` ブランチをリモートにプッシュします。

```bash
git push -u origin main
```

`-u` オプションにより、今後の `git push` / `git pull` で `origin main` がデフォルトになります。

## 10. 動作確認

リモートへのプッシュが成功したら、GitHub のリポジトリページを開き、コミットとファイルが反映されているか確認します。

## 11. 以降の開発フロー

通常の開発では、次のような流れになります。

1. ブランチを作成する
   ```bash
   git checkout -b feature/new-article
   ```

````
2. 編集する
3. 変更をステージングする
   ```bash
git add .
````

4. コミットする
   ```bash
   git commit -m "Add new article"
   ```

````
5. リモートにプッシュする
   ```bash
git push origin feature/new-article
````

6. Pull Request を作成してマージする

## まとめ

Git リポジトリの初期化からリモート作成、最初のプッシュまでの流れは、次の手順で行います。

- `git init`
- `git add .`
- `git commit -m "Initial commit"`
- GitHub でリポジトリを作成
- `git remote add origin <URL>`
- `git branch -M main`
- `git push -u origin main`

最初の一回を確実に行えば、その後は同じ手順を繰り返すだけで、GitHub へのデプロイや共同開発がスムーズになります。
