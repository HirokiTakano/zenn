---
title: "TypeScriptで作る簡単FPSゲームのロードマップとサンプルコード"
emoji: "🎮"
type: "tech"
topics: [typescript, game, web, copilot]
published: false
---

## この記事で学ぶこと

TypeScriptとHTML5 Canvasを使って、簡単なFPS風ゲームを作るためのロードマップと実際のコード例を紹介します。Webで動く最小限の実装からスタートし、段階的に機能を追加していくイメージを掴みましょう。

## 1. ロードマップ

### 1-1. 目的とスコープを決める

まずは「どのレベルのFPSを作るか」を決めます。最初はフル3Dの本格派ではなく、以下のような狙いを持つと進めやすいです。

- 画面は3Dっぽい視点（擬似3D / レイキャスティング）
- 移動、向きの変更、壁の表示ができる
- 敵や弾の処理は後回し
- ブラウザ上で動作する

### 1-2. 必要な機能を分解する

簡単なFPSを作るための基本要素は次の通りです。

1. レンダリング
   - HTML Canvas で画面を描画する
2. プレイヤーの制御
   - 前後移動、左右回転
3. マップ構造
   - 壁や床を表現するデータ
4. レイキャスティング
   - プレイヤーの視界に沿って壁を描画する
5. ゲームループ
   - 更新と描画を繰り返す

### 1-3. 開発環境を整える

TypeScriptで開発する場合、以下の環境があれば十分です。

- Node.js
- TypeScript
- 簡単なビルドツール（`tsc` で OK）
- ブラウザ

`package.json` があれば、自動ビルドやローカルサーバの設定もできます。

### 1-4. 最小構成で動かす

まずは最小構成で画面にレンダリングが出る状態を作ります。

- Canvas を配置
- 画面をクリアして単色で塗る
- ゲームループを回す

### 1-5. プレイヤー移動と視点制御

次に、キー入力からプレイヤー位置と向きを更新します。

- `W` / `S` で前後移動
- `A` / `D` で左右旋回
- `Shift` で速度調整

### 1-6. レイキャスティングで壁を描画

2Dマップを使ってレイを飛ばし、壁までの距離を計算します。距離に応じて縦線を描画すれば、簡易的な3D表示が実現できます。

### 1-7. 拡張例

基本が動いたら、次のような機能を追加してみましょう。

- マップのテクスチャや色
- 敵キャラクターの追加
- 弾の発射と命中判定
- 3Dオブジェクトの追加
- モバイル対応のタッチ操作

## 2. 実際のコード例

ここからは、シンプルなレイキャスティングFPSのサンプルコードを紹介します。

### 2-1. HTML の土台

```html
<!DOCTYPE html>
<html lang="ja">
  <head>
    <meta charset="UTF-8" />
    <meta name="viewport" content="width=device-width, initial-scale=1.0" />
    <title>TypeScript FPS Demo</title>
    <style>
      body {
        margin: 0;
        overflow: hidden;
        background: #111;
      }
      canvas {
        display: block;
        width: 100vw;
        height: 100vh;
      }
    </style>
  </head>
  <body>
    <canvas id="game"></canvas>
    <script src="dist/main.js"></script>
  </body>
</html>
```

### 2-2. TypeScript の基本構造

以下は `src/main.ts` の例です。

```ts
const canvas = document.getElementById("game") as HTMLCanvasElement;
const ctx = canvas.getContext("2d");
if (!ctx) throw new Error("Canvas context not found");

canvas.width = window.innerWidth;
canvas.height = window.innerHeight;

const map = [
  [1, 1, 1, 1, 1, 1, 1, 1],
  [1, 0, 0, 0, 0, 0, 0, 1],
  [1, 0, 1, 0, 1, 0, 0, 1],
  [1, 0, 0, 0, 0, 0, 0, 1],
  [1, 0, 1, 0, 1, 0, 0, 1],
  [1, 0, 0, 0, 0, 0, 0, 1],
  [1, 1, 1, 1, 1, 1, 1, 1],
];

const tileSize = 64;
const fov = Math.PI / 3;
const rayCount = 120;

const player = {
  x: 160,
  y: 160,
  angle: 0,
  speed: 120,
};

const keys = new Set<string>();
window.addEventListener("keydown", (e) => keys.add(e.key));
window.addEventListener("keyup", (e) => keys.delete(e.key));

function update(dt: number) {
  const moveStep = player.speed * dt;
  if (keys.has("w")) {
    player.x += Math.cos(player.angle) * moveStep;
    player.y += Math.sin(player.angle) * moveStep;
  }
  if (keys.has("s")) {
    player.x -= Math.cos(player.angle) * moveStep;
    player.y -= Math.sin(player.angle) * moveStep;
  }
  if (keys.has("a")) player.angle -= 2 * dt;
  if (keys.has("d")) player.angle += 2 * dt;
}

function castRays() {
  const rays = [] as Array<{ distance: number; hit: number }>;
  const halfFov = fov / 2;
  const angleStep = fov / rayCount;

  for (let i = 0; i < rayCount; i++) {
    const rayAngle = player.angle - halfFov + i * angleStep;
    let distance = 0;
    let hit = 0;

    while (distance < 1000) {
      distance += 1;
      const testX = player.x + Math.cos(rayAngle) * distance;
      const testY = player.y + Math.sin(rayAngle) * distance;
      const mapX = Math.floor(testX / tileSize);
      const mapY = Math.floor(testY / tileSize);
      if (map[mapY]?.[mapX] === 1) {
        hit = 1;
        break;
      }
    }

    rays.push({ distance, hit });
  }

  return rays;
}

function drawScene() {
  if (!ctx) return;
  ctx.clearRect(0, 0, canvas.width, canvas.height);

  const rays = castRays();
  const stripWidth = canvas.width / rayCount;

  for (let i = 0; i < rayCount; i++) {
    const ray = rays[i];
    const correctedDistance =
      ray.distance * Math.cos((i - rayCount / 2) * (fov / rayCount));
    const wallHeight = (tileSize * 320) / correctedDistance;
    const x = i * stripWidth;
    const y = canvas.height / 2 - wallHeight / 2;

    ctx.fillStyle = `rgb(${Math.max(0, 200 - correctedDistance)}, 0, 0)`;
    ctx.fillRect(x, y, stripWidth + 1, wallHeight);
  }
}

let lastTime = 0;
function loop(time: number) {
  const dt = (time - lastTime) / 1000;
  lastTime = time;

  update(dt);
  drawScene();
  requestAnimationFrame(loop);
}

requestAnimationFrame(loop);
```

### 2-3. 説明

- `map` は 1 が壁、0 が空間を表しています。
- `player.angle` を変えることで視点を左右に回転できます。
- `castRays()` は複数のレイを飛ばし、壁までの距離を計測します。
- `drawScene()` でレイごとに壁の高さを描画し、擬似的な3Dを表現します。

## 3. 拡張アイデア

### 敵やターゲットの追加

- レイを使った当たり判定を追加し、敵にヒットしたらスコアを増やす
- 球を発射して敵に当たる仕組みを実装する

### マップやテクスチャの強化

- `map` の定義に複数の壁種別を追加し、壁の色やテクスチャを変える
- ミニマップを表示して現在位置を可視化する

### 3D表現の進化

- さらに本格的にするなら `three.js` を導入し、真の 3D シーンを作る
- Canvas の代わりに WebGL を使い、モデルやライティングを追加する

## 4. まとめ

TypeScript で簡単なFPSゲームを作るなら、まずはCanvas とレイキャスティングで「視点を持つゲーム」を実装するのが近道です。基本が動いたら、敵の追加、弾の発射、マップ拡張、WebGL への移行など、段階的に機能を増やしていきましょう。

この記事をベースに、自分だけのFPSプロトタイプを作ってみてください。
