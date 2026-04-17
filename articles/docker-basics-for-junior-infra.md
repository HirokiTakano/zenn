---
title: "ジュニアエンジニアのためのDocker入門 ─ 概念から実践まで一気に理解する"
emoji: "🐳"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: [docker, infrastructure, beginner, linux, devops]
published: false
---

# はじめに

「Dockerって聞いたことあるけど、何が嬉しいの？」  
「コンテナって仮想マシンと何が違うの？」  
「触ってみたいけど、どこから始めれば良いかわからない……」

この記事はそんな **ジュニアインフラエンジニア** を対象に、Dockerの基礎概念から実際に手を動かすところまでを一通り解説します。  
記事を読み終える頃には、Docker を使って簡単な Web ページを公開できる力が身についているはずです。

:::message
この記事で扱うバージョン・環境
- Docker Engine 26.x（Docker Desktop でも同じ手順で動きます）
- OS: Linux / macOS / Windows（WSL2）
:::

---

# 1. Dockerとは何か？

**Docker** は **コンテナ型の仮想化技術** を提供するプラットフォームです。  
「アプリとその動作に必要なものをまとめてパッケージ化し、どこでも同じように動かせる」仕組みを実現します。

## 1-1. 従来の仮想マシン（VM）との違い

まずは図で比較してみましょう。

```
【仮想マシン（VM）の構成】

┌──────────────────────────────────────────┐
│              物理サーバー                  │
│  ┌────────────────────────────────────┐  │
│  │       ホスト OS (Linux 等)          │  │
│  │  ┌──────────────────────────────┐  │  │
│  │  │    ハイパーバイザー (VMware 等) │  │  │
│  │  │  ┌──────────┐ ┌──────────┐  │  │  │
│  │  │  │  ゲストOS │ │  ゲストOS│  │  │  │
│  │  │  │  (Linux) │ │ (Windows)│  │  │  │
│  │  │  │  ┌────┐  │ │  ┌────┐  │  │  │  │
│  │  │  │  │App │  │ │  │App │  │  │  │  │
│  │  │  │  └────┘  │ │  └────┘  │  │  │  │
│  │  │  └──────────┘ └──────────┘  │  │  │
│  │  └──────────────────────────────┘  │  │
│  └────────────────────────────────────┘  │
└──────────────────────────────────────────┘

→ ゲストOS ごとにCPU・メモリを占有するため重い
```

```
【Dockerコンテナの構成】

┌──────────────────────────────────────────┐
│              物理サーバー                  │
│  ┌────────────────────────────────────┐  │
│  │       ホスト OS (Linux 等)          │  │
│  │  ┌──────────────────────────────┐  │  │
│  │  │      Docker Engine           │  │  │
│  │  │  ┌──────────┐ ┌──────────┐  │  │  │
│  │  │  │コンテナ A │ │コンテナ B│  │  │  │
│  │  │  │  ┌────┐  │ │  ┌────┐  │  │  │  │
│  │  │  │  │App │  │ │  │App │  │  │  │  │
│  │  │  │  └────┘  │ │  └────┘  │  │  │  │
│  │  │  └──────────┘ └──────────┘  │  │  │
│  │  └──────────────────────────────┘  │  │
│  └────────────────────────────────────┘  │
└──────────────────────────────────────────┘

→ ホストOSのカーネルを共有するため軽量・高速
```

| 比較項目 | 仮想マシン（VM） | Dockerコンテナ |
| ---- | ---- | ---- |
| 起動時間 | 分単位 | 秒以内 |
| ディスク使用量 | GBオーダー | MBオーダー |
| OS の完全な分離 | ✅ あり | ❌ なし（カーネル共有） |
| ポータビリティ | △ | ✅ 高い |
| 用途 | 完全な環境分離が必要な場合 | アプリの配布・スケールアウト |

:::message
VMとコンテナは「どちらが優れている」ではなく **使い分け** です。  
セキュリティ要件が厳しい場合はVM、素早いデプロイが求められる場合はコンテナ、という考え方が一般的です。
:::

## 1-2. Dockerの3つの価値

1. **どこでも同じ動作（環境差異の解消）**  
   「自分のPCでは動くのに、本番では動かない」問題を解消します。

2. **素早い起動・停止（高速なライフサイクル管理）**  
   コンテナは秒単位で起動・停止できるため、CI/CDパイプラインとも相性抜群です。

3. **依存関係の隔離（クリーンな環境）**  
   各コンテナは独立した環境を持つため、複数のアプリのライブラリバージョンが衝突しません。

---

# 2. Dockerの主要な概念

```
【Dockerの全体像】

  ┌─────────────────────────────────────────────────┐
  │              Docker Hub（レジストリ）              │
  │  [nginx イメージ]  [mysql イメージ]  [自作イメージ] │
  └──────────────────┬──────────────────────────────┘
                     │ docker pull / push
                     ▼
  ┌─────────────────────────────────────────────────┐
  │              ローカルマシン                        │
  │                                                  │
  │  Dockerfile ──── docker build ──→ イメージ        │
  │                                       │           │
  │                              docker run           │
  │                                       │           │
  │                                       ▼           │
  │                                  コンテナ          │
  │                              (実行中のプロセス)    │
  │                                       │           │
  │                               ┌───────┴───────┐   │
  │                               │ ボリューム     │   │
  │                               │ (永続化領域)   │   │
  │                               └───────────────┘   │
  └─────────────────────────────────────────────────┘
```

## 2-1. イメージ（Image）

コンテナの **設計図（テンプレート）** です。  
アプリのコード、ランタイム、設定ファイルなどが階層（レイヤー）構造でまとめられた読み取り専用のファイルです。

```
イメージのレイヤー構造

┌─────────────────────────┐  ← アプリのコード（最上位レイヤー）
├─────────────────────────┤  ← 設定ファイル
├─────────────────────────┤  ← Node.js のインストール
├─────────────────────────┤  ← apt パッケージ
└─────────────────────────┘  ← ベースOS（ubuntu:22.04 等）
```

レイヤーをキャッシュで再利用するため、ビルドが速くなります。

## 2-2. コンテナ（Container）

イメージを **実際に起動したもの** です。  
VMでいう「起動中の仮想マシン」に相当します。  
コンテナを停止・削除してもイメージは残るので、何度でも同じコンテナを作り直せます。

## 2-3. レジストリ（Registry）

イメージを **保存・共有する場所** です。  
公式の [Docker Hub](https://hub.docker.com/) には nginx、mysql、python など膨大な公式イメージが公開されています。  
社内用には AWS ECR や GitHub Container Registry (GHCR) を使うことが多いです。

## 2-4. ボリューム（Volume）

コンテナ外にデータを **永続化する仕組み** です。  
コンテナを削除するとコンテナ内のデータは消えますが、ボリュームにマウントされたデータは残ります。  
データベース（MySQLなど）を扱う際に必須です。

## 2-5. ネットワーク（Network）

コンテナ間の **通信経路** を管理します。  
デフォルトでは各コンテナは独立したネットワーク空間を持ち、明示的に接続しないとコンテナ間通信はできません。

---

# 3. Dockerのインストール

## Linux（Ubuntu/Debian 系）

```bash
# 古い Docker があれば削除
sudo apt remove docker docker-engine docker.io containerd runc

# 必要パッケージをインストール
sudo apt update
sudo apt install -y ca-certificates curl gnupg

# Docker の公式 GPG キーを追加
sudo install -m 0755 -d /etc/apt/keyrings
curl -fsSL https://download.docker.com/linux/ubuntu/gpg | \
  sudo gpg --dearmor -o /etc/apt/keyrings/docker.gpg

# リポジトリを追加
echo \
  "deb [arch=$(dpkg --print-architecture) signed-by=/etc/apt/keyrings/docker.gpg] \
  https://download.docker.com/linux/ubuntu \
  $(. /etc/os-release && echo "$VERSION_CODENAME") stable" | \
  sudo tee /etc/apt/sources.list.d/docker.list > /dev/null

# Docker Engine をインストール
sudo apt update
sudo apt install -y docker-ce docker-ce-cli containerd.io docker-buildx-plugin docker-compose-plugin

# sudo なしで使えるよう現ユーザーを docker グループへ追加（要再ログイン）
sudo usermod -aG docker $USER
```

## macOS / Windows

[Docker Desktop](https://www.docker.com/products/docker-desktop/) をダウンロード・インストールするだけです。  
GUI も付いており、コンテナの起動状況を視覚的に確認できます。

## インストール確認

```bash
docker --version
# Docker version 26.x.x, build xxxxxxx

docker run hello-world
# Hello from Docker! と表示されれば成功
```

---

# 4. 基本的なDockerコマンド

## 4-1. イメージ操作

```bash
# Docker Hub からイメージを取得
docker pull nginx:latest

# ローカルにあるイメージ一覧を表示
docker images

# イメージを削除
docker rmi nginx:latest
```

## 4-2. コンテナ操作

```bash
# コンテナを起動（フォアグラウンド）
docker run nginx

# コンテナをバックグラウンドで起動（-d: detached mode）
docker run -d nginx

# ポートを公開して起動（ホスト:コンテナ）
docker run -d -p 8080:80 nginx
#          │   └─────┬──────┘
#          │         └ ホストの 8080 番ポートをコンテナの 80 番へ転送
#          └ バックグラウンド実行

# 起動中のコンテナ一覧
docker ps

# すべてのコンテナを表示（停止中を含む）
docker ps -a

# コンテナ内でコマンドを実行（-it: 対話的ターミナル）
docker exec -it <コンテナID> bash

# コンテナを停止
docker stop <コンテナID>

# コンテナを削除
docker rm <コンテナID>

# コンテナを停止して即削除
docker rm -f <コンテナID>

# 停止中の全コンテナを一括削除
docker container prune
```

## 4-3. ログの確認

```bash
# コンテナのログを表示
docker logs <コンテナID>

# リアルタイムでログを追跡（-f: follow）
docker logs -f <コンテナID>
```

## 4-4. よく使うオプションまとめ

| オプション | 説明 | 使用例 |
| ---- | ---- | ---- |
| `-d` | バックグラウンド実行 | `docker run -d nginx` |
| `-p` | ポートマッピング | `-p 8080:80` |
| `-v` | ボリュームマウント | `-v /host/path:/container/path` |
| `-e` | 環境変数の設定 | `-e MYSQL_ROOT_PASSWORD=pass` |
| `--name` | コンテナ名を指定 | `--name my-nginx` |
| `--rm` | 停止時に自動削除 | `docker run --rm nginx` |
| `-it` | 対話的ターミナル | `docker run -it ubuntu bash` |

---

# 5. 実践：簡単なWebページをDockerで公開する

実際に手を動かしてみましょう。  
**nginx コンテナ上でオリジナルの HTML ページを配信する** という課題に取り組みます。

## 5-1. ファイル構成

```
mywebsite/
├── Dockerfile
└── html/
    └── index.html
```

まずはディレクトリを作成しましょう。

```bash
mkdir -p mywebsite/html
cd mywebsite
```

## 5-2. index.html の作成

```bash
cat > html/index.html << 'EOF'
<!DOCTYPE html>
<html lang="ja">
<head>
  <meta charset="UTF-8">
  <meta name="viewport" content="width=device-width, initial-scale=1.0">
  <title>Docker で動く Web ページ</title>
  <style>
    * { box-sizing: border-box; margin: 0; padding: 0; }
    body {
      font-family: 'Helvetica Neue', Arial, sans-serif;
      background: linear-gradient(135deg, #1a1a2e 0%, #16213e 50%, #0f3460 100%);
      min-height: 100vh;
      display: flex;
      align-items: center;
      justify-content: center;
      color: white;
    }
    .card {
      background: rgba(255,255,255,0.08);
      border: 1px solid rgba(255,255,255,0.15);
      border-radius: 16px;
      padding: 48px 56px;
      text-align: center;
      max-width: 520px;
      backdrop-filter: blur(12px);
    }
    .whale { font-size: 80px; margin-bottom: 24px; }
    h1 { font-size: 2rem; margin-bottom: 12px; }
    p { color: rgba(255,255,255,0.7); line-height: 1.7; margin-bottom: 24px; }
    .badge {
      display: inline-block;
      background: #2496ed;
      padding: 6px 20px;
      border-radius: 99px;
      font-size: 0.85rem;
      font-weight: bold;
    }
  </style>
</head>
<body>
  <div class="card">
    <div class="whale">🐳</div>
    <h1>Hello, Docker!</h1>
    <p>このページは Docker コンテナ上の nginx から配信されています。<br>
       ようこそ、コンテナの世界へ！</p>
    <span class="badge">Running on Docker + nginx</span>
  </div>
</body>
</html>
EOF
```

## 5-3. Dockerfile の作成

**Dockerfile** は「イメージをどう作るか」を記述した設計書です。

```bash
cat > Dockerfile << 'EOF'
# ベースイメージに公式の nginx（軽量 alpine 版）を使用
FROM nginx:alpine

# ホスト側の html/ ディレクトリをコンテナの配信ルートへコピー
COPY html/ /usr/share/nginx/html/

# nginx はデフォルトで 80 番ポートを使用（ドキュメント用の宣言）
EXPOSE 80
EOF
```

### Dockerfile の命令解説

| 命令 | 役割 |
| ---- | ---- |
| `FROM` | ベースイメージを指定（必ず最初に書く） |
| `COPY` | ホストのファイル/ディレクトリをイメージに追加 |
| `RUN` | ビルド時にコマンドを実行（パッケージインストール等） |
| `CMD` | コンテナ起動時のデフォルトコマンドを指定 |
| `EXPOSE` | コンテナが使うポートをドキュメントとして宣言 |
| `ENV` | 環境変数を設定 |
| `WORKDIR` | 作業ディレクトリを変更 |

## 5-4. イメージのビルド

```bash
# -t でイメージに名前（タグ）を付ける
# 末尾の . は Dockerfile が存在するディレクトリ（カレントディレクトリ）
docker build -t mywebsite:v1 .
```

実行すると以下のような出力が得られます。

```
[+] Building 1.2s (7/7) FINISHED
 => [internal] load build definition from Dockerfile
 => [internal] load .dockerignore
 => [internal] load metadata for docker.io/library/nginx:alpine
 => [1/2] FROM docker.io/library/nginx:alpine
 => [2/2] COPY html/ /usr/share/nginx/html/
 => exporting to image
 => => naming to docker.io/library/mywebsite:v1
```

```bash
# ビルドされたイメージを確認
docker images | grep mywebsite
# mywebsite   v1   xxxxxxxxx   10 seconds ago   43.2MB
```

## 5-5. コンテナを起動して確認

```bash
docker run -d \
  --name mywebsite \
  -p 8080:80 \
  mywebsite:v1
```

```
ホスト（あなたのPC）           コンテナ
┌───────────────┐             ┌───────────────┐
│               │             │               │
│  ブラウザ      │ ─ 8080 ──→ │  nginx :80    │
│ localhost:8080 │            │  index.html   │
│               │             │               │
└───────────────┘             └───────────────┘
```

ブラウザで `http://localhost:8080` を開くと、先ほど作成した HTML ページが表示されます。

```bash
# curl で確認する場合
curl http://localhost:8080
```

## 5-6. コンテナの後片付け

```bash
# コンテナを停止
docker stop mywebsite

# コンテナを削除
docker rm mywebsite

# イメージを削除
docker rmi mywebsite:v1
```

---

# 6. Docker Compose 入門

複数のコンテナをまとめて管理するのが **Docker Compose** です。  
設定を `compose.yaml`（または `docker-compose.yml`）に記述します。

```
【Docker Compose を使う場合の例: Web + DB 構成】

  compose.yaml
     │
     ├── web サービス（nginx）    ─ポート 8080→80
     └── db サービス（mysql）    ─ボリューム mysql_data
               ↑
         コンテナ同士は同じネットワークで通信可能
```

以下は Web サーバー（nginx）と MySQL を同時に起動する例です。

```yaml
# compose.yaml
services:
  web:
    image: nginx:alpine
    ports:
      - "8080:80"
    volumes:
      - ./html:/usr/share/nginx/html:ro  # ro = read only
    depends_on:
      - db

  db:
    image: mysql:8.0
    environment:
      MYSQL_ROOT_PASSWORD: rootpass
      MYSQL_DATABASE: mydb
      MYSQL_USER: user
      MYSQL_PASSWORD: password
    volumes:
      - mysql_data:/var/lib/mysql  # 名前付きボリュームで永続化

volumes:
  mysql_data:  # Docker が管理する名前付きボリュームの定義
```

## Docker Compose の基本コマンド

```bash
# すべてのサービスをバックグラウンドで起動
docker compose up -d

# 起動中のサービス一覧を確認
docker compose ps

# ログを確認
docker compose logs -f

# すべてのサービスを停止・削除
docker compose down

# ボリュームも含めて完全に削除（データも消える！）
docker compose down -v
```

:::message alert
`docker compose down -v` はボリュームのデータも削除します。  
本番環境やデータが重要な場合は絶対に実行しないよう注意してください。
:::

---

# 7. よくあるトラブルと対処法

## ポートが使用中エラー

```
Error: Bind for 0.0.0.0:8080 failed: port is already allocated
```

**原因**: ホストの 8080 番ポートがすでに使われている  
**対処**: 別のポートを指定する（例: `-p 8081:80`）か、競合プロセスを停止する

```bash
# どのプロセスがポートを使っているか確認
sudo lsof -i :8080
# または
sudo ss -tlnp | grep 8080
```

## コンテナがすぐ終了してしまう

```bash
# 終了したコンテナのログを確認
docker logs <コンテナID>

# 終了ステータスを確認
docker inspect <コンテナID> | grep ExitCode
```

## イメージが見つからない

```
Unable to find image 'myapp:latest' locally
```

**原因**: イメージ名やタグのタイプミス、またはビルド前にrunを実行している  
**対処**: `docker images` でローカルのイメージ名を確認し、`docker build` を先に実行する

## コンテナ内でファイルを確認したい

```bash
# コンテナ内にシェルで入る（bash が無い場合は sh を試す）
docker exec -it <コンテナID> sh

# ファイルをホストへコピー
docker cp <コンテナID>:/path/in/container ./local-dir
```

---

# 8. セキュリティの基本

コンテナでも最低限のセキュリティ意識を持ちましょう。

| NG 例 | 推奨 |
| ---- | ---- |
| `FROM ubuntu:latest` で毎回最新を追いかける | バージョンを固定する（例: `ubuntu:22.04`） |
| root ユーザーでアプリを動かす | `USER` 命令で一般ユーザーへ切り替える |
| 秘密情報（パスワード等）を Dockerfile に直書き | 環境変数や Docker Secrets を使う |
| 不要なパッケージを大量にインストール | 必要最小限に絞る（alpine ベースを使う） |

```dockerfile
# セキュアな Dockerfile の例
FROM node:20-alpine

# 一般ユーザーを作成して切り替え
RUN addgroup -S appgroup && adduser -S appuser -G appgroup
USER appuser

WORKDIR /app
COPY --chown=appuser:appgroup . .

RUN npm ci --only=production

EXPOSE 3000
CMD ["node", "server.js"]
```

---

# 9. 次のステップ

この記事で学んだことを土台に、次は以下のテーマに挑戦してみましょう。

- **マルチステージビルド**: ビルド環境と本番環境を分けてイメージを軽量化する
- **Docker ネットワーク詳細**: bridge / host / overlay の使い分け
- **Kubernetes（k8s）**: コンテナのオーケストレーション（複数ホストへのデプロイ管理）
- **CI/CD 連携**: GitHub Actions + Docker で自動ビルド・自動デプロイ
- **コンテナ監視**: Prometheus + Grafana でコンテナのメトリクスを可視化

---

# まとめ

| 項目 | ポイント |
| ---- | ---- |
| Docker とは | アプリをコンテナとしてパッケージ化・実行するプラットフォーム |
| VM との違い | カーネルを共有するため軽量・高速だが完全な分離ではない |
| イメージ | コンテナの設計図（読み取り専用のレイヤー構造） |
| コンテナ | イメージを実際に起動したもの（実行中のプロセス） |
| Dockerfile | イメージをどう作るかの手順書 |
| Docker Compose | 複数コンテナをまとめて定義・管理する仕組み |

Dockerの世界へようこそ！  
まずは `docker run hello-world` から始めて、手を動かしながら概念を体感していきましょう。🐳

---

# 参考リンク

- [Docker 公式ドキュメント](https://docs.docker.com/)
- [Docker Hub](https://hub.docker.com/)
- [Play with Docker（ブラウザ上で無料で試せる）](https://labs.play-with-docker.com/)
