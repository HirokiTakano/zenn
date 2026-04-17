---
title: "PowerShell初心者向け：実行ポリシーでスクリプトが弾かれたときの対処法"
emoji: "⚡"
type: "tech"
topics: [powershell, windows, scripting]
published: false
---

## はじめに

PowerShell を使い始めたばかりのとき、スクリプトを書いて実行しようとしたら「弾かれた」経験はありませんか？
この記事では、PowerShell の実行ポリシーについて初心者にもわかりやすく解説し、私が実際に遭遇した現象とその対処法を紹介します。

## 1. なぜ PowerShell で弾かれるのか

PowerShell には「実行ポリシー」という仕組みがあります。これは、PC 上で実行できるスクリプトの種類を制限するためのセキュリティ機能です。

実行ポリシーの主な種類は次の通りです。

- `Restricted`
  - スクリプト実行を禁止します。PowerShell のデフォルト設定です。
- `RemoteSigned`
  - インターネットから取得したスクリプトには署名が必要です。
- `AllSigned`
  - すべてのスクリプトに署名が必要です。
- `Bypass`
  - 実行ポリシーのチェックを無視します。

初心者が自作したスクリプトでも、実行ポリシーが厳しいと実行できません。

## 2. 私が遭遇した状況

私の場合は、PowerShell でスクリプトを組み、`.	est.ps1` のように実行しようとしたときに次のようなエラーが出ました。

```powershell
c:\work>PowerShell .\test.ps1
.\test.ps1 : このシステムではスクリプトの実行が無効になっているため、ファイル C:\work\test.ps1 を読み込むことができませ
ん。詳細については、「about_Execution_Policies」(https://go.microsoft.com/fwlink/?LinkID=135170) を参照してください。
発生場所 行:1 文字:1
+ .\test.ps1
+ ~~~~~~~~~~
    + CategoryInfo          : セキュリティ エラー: (: ) []、PSSecurityException
    + FullyQualifiedErrorId : UnauthorizedAccess
```

原因を調べたところ、実行ポリシーが現在の環境でスクリプト実行を許可していなかったためでした。

## 3. 現在の実行ポリシーを確認する

まずは次のコマンドで設定内容を確認します。

```powershell
Get-ExecutionPolicy -List
```

これにより、`MachinePolicy`、`UserPolicy`、`Process`、`CurrentUser`、`LocalMachine` など、各スコープごとの実行ポリシーが表示されます。

## 4. 今回の対処：セッションのみ許可する方法

私が選んだ対処法は、今の PowerShell セッションだけ実行ポリシーを緩める方法です。これは一時的で、PC 全体の設定を変更しません。

```powershell
powershell.exe -NoProfile -ExecutionPolicy Bypass -File .\test.ps1
```

または、現在のセッション内でのみ実行ポリシーを変更する方法として次のコマンドも使えます。

```powershell
Set-ExecutionPolicy -Scope Process -ExecutionPolicy Bypass
.	est.ps1
```

この方法なら、PowerShell を閉じると元の実行ポリシーに戻ります。

## 5. 注意点と安全性

`Bypass` は便利ですが、常に安全とは限りません。以下の点に注意してください。

- 信頼できるスクリプトだけ実行する
- 不要なときは `Bypass` を使わない
- PC 全体の設定を変更する場合は慎重に

PowerShell の実行ポリシーは、悪意あるスクリプトの誤実行を防ぐための機能です。初心者のうちは、仕組みを理解した上で一時的な対処を行うのが安心です。

## 6. まとめ

- PowerShell でスクリプトが弾かれる原因は、実行ポリシーが許可されていないためです。
- `Get-ExecutionPolicy -List` で現在の設定を確認できます。
- 今回の対処法は、`-ExecutionPolicy Bypass` を使ったセッション限定の実行です。
- 常に信頼できるスクリプトだけ実行し、必要以上に設定を緩めないようにしましょう。

PowerShell 初心者でも、実行ポリシーの仕組みを押さえておけば、スクリプト実行時のエラーに冷静に対応できます。
