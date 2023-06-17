---
title: "draw.ioをvscodeで編集するために、Zenn記事をgithub管理しVSCodeで書く"
emoji: "🦔"
type: "tech"
topics:
  - "github"
  - "vscode"
  - "drawio"
  - "zeneditor"
  - "drawiointegrator"
published: true
published_at: "2023-06-12 10:57"
---

# 概要
https://zenn.dev/tofucode/articles/76c75c09a9a13f でグラフ作成ツールにdraw.ioを利用したところ記事への画像埋め込みと管理周りが面倒で、その解決策としてdraw.ioをVSCodeから利用する方法を試す。その準備としてZenn記事をGitHub管理にし、Zenn記事全体をVSCodeで書くよう変更する。

# 方法
GitHub管理にする方法は公式で以下にまとめられている。
https://zenn.dev/zenn/articles/connect-to-github

上記に書いていないが、既にZennに記事がある場合はそれをエクスポートしGitHubにあげる必要がある。
https://zenn.dev/zenn/articles/setup-zenn-github-with-export

ちなみに記事と本には対応しているが、スクラップには対応していないとのこと。記事しか作っていなくて良かった。

# Zenn記事をgithub管理
以下に沿って行う。
https://zenn.dev/zenn/articles/setup-zenn-github-with-export
## 手順1: GitHubリポジトリ連携を行う
できた。
![](https://storage.googleapis.com/zenn-user-upload/8290b1f879bf-20230611.jpg)

## 手順2： 投稿データをエクスポートする
数分間、エクスポートの準備に時間がかかる。その後、24時間の間ダウンロード可能になる模様。
![](https://storage.googleapis.com/zenn-user-upload/261ea87e0812-20230611.png)
できた
![](https://storage.googleapis.com/zenn-user-upload/67815afcf69a-20230611.png)

## 手順3： 連携リポジトリのルートにarticles、booksディレクトリを配置
やった
![](https://storage.googleapis.com/zenn-user-upload/deb98a8abc42-20230611.png)

## 手順4: 変更をコミットしGitHubへプッシュする
PRを作成しマージした
![](https://storage.googleapis.com/zenn-user-upload/5dbb890fe4a9-20230611.jpg)

## できた確認
デプロイされた。デプロイ状況は Zenn のGitHub連携から確認できる。
![](https://storage.googleapis.com/zenn-user-upload/851cd7f29854-20230611.jpg)

既存の記事全部更新されとる！！！見えなくなった記事や画像がないか確認しないと！！！

と思ったけど大丈夫そう。見れなくなった画像とかない。何も変わっていない。

・・・いや、１つだけ変わった！
閲覧者が記事の編集を提案できるようになってる！！！（↓のボタンが全記事の末尾に追加された）
![](https://storage.googleapis.com/zenn-user-upload/4629fc9eeb48-20230611.jpg)

こ、こいつ余計なことを・・・！！穴だらけの私の記事に指摘が飛んできてしまうではないか！！
まぁ誰も反応しないか。

# Zenn記事をVSCodeで書く
VSCodeで記事を開いて編集しするだけ。mainにマージされたらZennにデプロイされる。
![](https://storage.googleapis.com/zenn-user-upload/bfb21daf7cb9-20230612.jpg)
編集中のプレビュー表示にVSCodeの拡張ZennEditorを使う。
https://zenn.dev/ctrlkeykoyubi/articles/e7d91c5286a409#5.2.2.-zenn-editor
![](https://storage.googleapis.com/zenn-user-upload/4a5de97fa114-20230611.png)
Zenn CLIを入れろと怒られたので入れる。
https://zenn.dev/zenn/articles/install-zenn-cli
https://zenn.dev/zenn/articles/zenn-cli-guide
```
npm init --yes
npm install zenn-cli
npx zenn init
npm install zenn-cli@latest
```
できた


# draw.ioをVSCodeで編集する
以下を参考にする。
https://qiita.com/riku-shiru/items/5ab7c5aecdfea323ec4e
https://zenn.dev/satonopan/articles/4177ed8b88e067
https://github.com/hediet/vscode-drawio

プラグインをインストール
![](https://storage.googleapis.com/zenn-user-upload/a27aeb296d1c-20230611.png)

画像を保存するためのディレクトリを作成
![](https://storage.googleapis.com/zenn-user-upload/0f273717e47c-20230612.png)

グラフを作成（.drawio.png形式のファイルを作ると勝手にエディタが開いてくれる）
![](https://storage.googleapis.com/zenn-user-upload/b752381346f5-20230612.jpg)

記事へ埋め込む
```
![](/images/dbe92eeac60d6f/sample.drawio.png)
```
![](/images/dbe92eeac60d6f/sample.drawio.png)

編集
![](/images/dbe92eeac60d6f/ss2.jpg)

できた。これで以下を達成した。
1. 修正のたびに画像を差し替える必要がない
2. 図の画像と管理用ファイルが同じため閲覧用と編集用でURLが同じになり管理が楽
3. 図作成時にgithubを開きURLをコピーするなどの手間がない
4. 図ファイルの管理が直感的

# 画像どうするんだこれ？
図ではなく普通の画像をZenn記事に貼り付けたい時、github管理していない状態だと「ブラウザの編集画面にドラッグ&ドロップする」という1アクションで画像をZennにアップロードすると同時に記事に埋め込むことができた。
記事をgithub管理してると執筆時点でZennサーバと繋がっていないため、画像を記事編集とは別でZennにアップロードする（↓）か、

![](/images/dbe92eeac60d6f/ss1.png)

もしくはgithubで画像を管理する（↓）必要がある。

![](/images/dbe92eeac60d6f/ss3.png)

Zennにアップロードだと別ブラウザでZennを開かないといけないので自分はgit管理とした。

# 新しい記事の作成
拡張機能Zenn EditorがZenn用のワークスペース「ZENN CONTENTS」に追加した記事作成ボタンを押す。
![](/images/dbe92eeac60d6f/ss4.jpg)

参考
https://zenn.dev/ctrlkeykoyubi/articles/e7d91c5286a409#6.1.-%E8%A8%98%E4%BA%8B%E3%82%92%E6%9B%B8%E3%81%8F%E5%89%8D%E3%81%AE%E6%BA%96%E5%82%99


# 感想
図を資料作成と同時に編集したいためだけに執筆環境がガラッと変わってしまったが、目的となる直感的な作図の管理は達成できたし、記事作成の環境がVSCodeになるのはそこまで苦じゃないのでいい感じ。

教えてくれたmskさんありがとう！！