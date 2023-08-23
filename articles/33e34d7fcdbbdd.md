---
title: "[API構築3] ECRのdockerイメージをECSにデプロイ"
emoji: "👌"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: 
  - "ecr"
  - "ecs"
  - "aws"
  - "docker"
  - "awsfargate"
published: true
---

# 概要
[API作るものと構成](https://zenn.dev/tofucode/articles/d216a5d6c75193) のために、[CI/CD周りの技術選定](https://zenn.dev/tofucode/articles/04f0b24bf7ceea) で選んだ技術を用いて環境構築していく。
前回は[dockerイメージをECRにプッシュした](https://zenn.dev/tofucode/articles/b0f6282c323879.md)ので、今回はそのdockerイメージをECSにデプロイする。

以下に沿って作業を行う。
https://docs.aws.amazon.com/ja_jp/AmazonECS/latest/userguide/getting-started-fargate.html

# ステップ 1: クラスターを作成する
ECSのGUI上でクラスターを作成する。画像ではcontainer insightsがoffになっているが、実際にはonにしている。 CPUやメモリの利用量の表示を行うため。
![](/images/33e34d7fcdbbdd/ss1.jpg)
![](/images/33e34d7fcdbbdd/ss2.jpg)

# ステップ 2: タスク定義を作成する
「新しいタスク定義の作成」からタスク定義を作成する。
JSONで設定を入力するかフォームかを選択できるが、フォームで設定していく（チュートリアルだとJSONでの設定だが設定の説明や選択肢値を確認するため）

## タスク定義とコンテナの設定
まずタスク定義の基本的な情報を設定する。
設定項目にあるポートマッピングとは、ホストのポートとコンテナのポートとを紐づけて、コンテナが外部通信できるようにするための機能。ECSが内部的に実行するdocker runで利用する。
ポートマッピングに関しては以下を参照する。
https://docs.aws.amazon.com/ja_jp/AmazonECS/latest/developerguide/task_definition_parameters.html#container_definition_portmappings

| 大項目 | 設定項目 | 説明 | 値 | 備考 |
| - | - | - | - | - |
| タスク定義 | タスク定義ファミリー | タスク定義の名前をつける | aichatbot-task-familly |  |
| コンテナの詳細 | 名前 | 今から作るコンテナの名前 | aichatbot-container |  |
|  | イメージURI | pullするコンテナレジストリのdockerイメージのURI  | public.ecr.aws/j3a2u6t5/aichatbot-repository | ECRにpushしたdockerイメージのURIを入れる |
| プライベートレジストリ | プライベートレジストリ認証 | プライベートレジストリを利用する場合の認証情報 | off | 今回は公開レジストリを利用するため |
| ポートマッピング | コンテナポート | コンテナ化しているアプリが受け付けているポート | 80 |  |
|  | プロトコル | ポートマッピングに使用されるプロトコル。有効な値は、tcpまたはudp。デフォルト: tcp。 | tcp |  |
|  | ポート名 | サービス紐付けで利用するポートの名前 | aichatbot-container-80-tcp |  |
|  | アプリケーションプロトコル | アプリケーションが使用するプロトコル | http |  |
| 環境変数 |  | ここで環境変数を設定できる |  | 今は設定なし |

## 環境、ストレージ、モニタリング、タグの設定

次に環境の設定を行う。

| 大項目 | 設定項目 | 説明 | 値 | 備考 |
| - | - | - | - | - |
| 環境 | アプリケーション環境 | FargateかEC2かを選択できる | AWS Fargate |  |
|  | オペレーティングシステム/アーキテクチャ |  | Linux/x86_64 |  |
|  | CPU |  | .25vCPU |  |
|  | メモリ |  | .5GB |  |

## タスク作成完了

タスク定義が作成できた

![](/images/33e34d7fcdbbdd/ss33.jpg)

# ステップ 3: サービスを作成する

タスク定義 > リビジョン > コンテナのデプロイボタンから「サービス作成」を選択する。
![](/images/33e34d7fcdbbdd/ss34.png)

以下の設定でサービスを作成する。

| 大項目 | 設定項目 | 説明 | 値 | 備考 |
| - | - | - | - | - |
| 環境 | 既存のクラスター | ステップ1で作成したクラスタを選択する | aichatbotcluster |  |
|  | コンピューティング設定 ((アドバンスト)) | コンピューティングオプション | キャパシティープロバイダー戦略 |  |
|  | プラットフォームのバージョン |  | latest |  |
| デプロイ設定 | アプリケーションタイプ |  | サービス |  |
|  | サービス名 |  | aichatbot-service |  |
|  | サービスタイプ |  | レプリカ |  |
|  | 必要なタスク |  | 1 |  |

作成に数分かかる。
サービスが作成できた。
![](/images/33e34d7fcdbbdd/ss4.jpg)

# ステップ 4: サービスを表示する
パブリックIDを確認する。
![](/images/33e34d7fcdbbdd/ss35.png)
これをブラウザで開くとHello Worldが表示されるはずだが表示されない。
なぜ・・・
![](/images/33e34d7fcdbbdd/ss36.png)

以下のデフォルトセキュリティグループだとダメ、というやつが原因な気がする。
https://teratail.com/questions/qefgdqqdm9kooi

以下を参考に新規にVPCを設定しこれを新しいセキュリティグループに設定する。
https://cloud5.jp/amazon-ecs-mk3/

## VPC作成
VPCダッシュボード > VPCを作

![](/images/33e34d7fcdbbdd/aa1.jpg)

以下のように設定
![](/images/33e34d7fcdbbdd/aa2.jpg)
作成ボタンを押すと以下のように作成される。
![](/images/33e34d7fcdbbdd/aa3.jpg)

## コンテナに HTTP アクセスを許可するために、セキュリティグループを変更
デフォルトだと外からのアクセスを全て弾くため弾かないルールを追加する。
1.左上のVPCでフィルタリング
2.作成したVPCをクリック
3.セキュリティグループをクリック
4.インバウンドのルールを編集をクリック
5.ルールを追加をクリック

| 項目 | 値 |
| - | - |
| タイプ | HTTP |
| ソース | Anywhere-IPv4 |

![](/images/33e34d7fcdbbdd/aa4.jpg)
できた。
![](/images/33e34d7fcdbbdd/aa5.jpg)

## 新VPCを設定した新しいECSクラスタを作成する

![](/images/33e34d7fcdbbdd/aa6.jpg)

できた！！！

![](/images/33e34d7fcdbbdd/aa7.jpg)

やったあ！！

![](/images/33e34d7fcdbbdd/bo.jpg)


# 感想
ECRはログイン周りがダルすぎるからDockerHubに移行しようかな
→ 移行した

https://zenn.dev/tofucode/articles/26f8f141b84d09

丸一ヶ月はまった(ブラウザからパブリックIPへのアクセス確認ができなかった)原因はデフォルトセキュリティグループがVPC外からのインバウンドトラフィックを通さないためでした。ダッセェww
VPCの設定などが必要だけど、自分の参考にしたページとかだとアクセスして終わり、という書き方だった。
AWSはドキュメントが玄人向き、というのはこーゆーことかな。

