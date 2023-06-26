---
title: "[API構築5] コンテナレジストリ, コンテナオーケストレーションツール周りの技術選定"
emoji: "📘"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: 
  - "ecr"
  - "dockerhub"
  - "githubpackages"
  - "eks"
  - "kubernetes"
published: true
---
# 概要
[API作るものと構成](https://zenn.dev/tofucode/articles/d216a5d6c75193) の環境構築のために、GitHub ActionsでGolang ApplicationのdockerビルドとEC2デプロイするCI/CDの技術選定を行う。

# 技術選定,調査
## 前提：使うサーバはAWS(AWSの何かは未決定)
前提として今までの開発や今後の仕事から、使うサーバはAWSのものと決めている。
3大クラウド（Azure, AWS, GCP)の中でAWSを使う、と言う話。
ここはほぼ趣味なので比較はなし。
なので最終的にEC2, ECS, EKSなどにデプロイする。

## コンテナレジストリ選定（Docker Hub, ECR, Github Packages, Artifactry Registryの比較)
コンテナイメージを本番環境にデプロイする際に利用するコンテナレジストリを選定する。
以下の理由でECR（公開リポジトリ）を利用にする。

| docker image registory | どんなものか |
| - | - |
| Github Packages(旧Github container registry) | GitHubが提供するコンテナレジストリで、GitHubとのシームレスな統合が可能。GitHub ActionsやGitHubパッケージとの連携が容易であり、アクセス制御やバージョン管理もGitHubと一元化される。 |
| Docker Hub | Dockerが提供する公開コンテナレジストリ。オープンソースプロジェクトや個人プロジェクトに人気があり、多くの公開イメージが利用できる。プライベートリポジトリの利用も可能だが、有料プランが必要。 |
| Amazon Elastic Container Registry (ECR) | AMAZONが提供するAWSのクラウド環境で使用するためのコンテナレジストリ。AWSのアカウントとのシームレスな統合が可能であり、セキュリティとスケーラビリティの面で強力。 |
| Artifactry Registry | Googleが提供するコンテナレジストリ。 |

| docker image registory | 印象 |
| - | - |
| Github Packages(旧Github container registry) | 自分がgithub proプランなので追加課金なしで20回/月デプロイできるためこれで問題ない。github actionsとの親和性も高くて良さそう。<br><br>しかしEC2との親和性の点でECRに劣る。（こっちはプライベートでできるが、個人趣味でやるので公開リポジトリでいいのでプライベートでできる利点が活かせない） |
| Docker Hub | 1レジストリは無料なのでこれでもいいかと思った。ただgithub acitonsとの親和性の点でgithub packagesに、EC2との親和性の点でECRに軍配が上がる。扱うのはDocker imageだからDocker Hub使うのが正統、と言う考え方もあるが、EC2にpushする記事があまりにもECRが多いのでECRを使うべきかと思っている。あとimageが2個以上だと急に700円/月もかかるのも、他が従量課金なので高く感じてしまう。 |
| Amazon Elastic Container Registry (ECR) | 公開レジストリでいいならほぼ無料。公開リポジトリで知らない誰かがimageをpullしてサービスを立ち上げられてしまうリスクは、個人開発で気にするリスクではない。もしサービスが大きくなり、プライベートリポジトリの料金を賄えるようになったらプライベートに切り替える。<br><br>EC2との親和性が高い。EC2に配る場合に他と比較せずこれを利用している記事がたくさんあるが、一応EC2に配るのは他のリポジトリでもできる。が、AWSはECR->EC2をメイン動線としていてここが一番手厚い（逆に他は予想外の何かが発生しやすい）ことは明らか。 |
| Artifactry Registry | ECRの公開が安い、自分がgithub proプランだと気づいていないとき、どれも高いと思い選択肢に追加した。Googleは確かに安いが、安くするにはEC2をやめてGCPを使わないといけないため不採用。今回は知識習得の関係でAWSを使いたかった。 |

採用：Amazon Elastic Container Registry (ECR)
github packagesと迷ったが実質無料でできるならEC2と親和性の高いECRでやる方がいいと判断。値段が変わらないなら勉強のためにAWS使いたいし。 

:::details その他の検討, 参考
| docker image registory | 料金 |
| - | - |
| Github Packages(旧Github container registry) | 無料で以下が利用可能<br>ストレージ：500MB<br>データ転送/月：1GB<br>それ以上は課金が必要<br><br>github pro(4$/月）なら<br>ストレージ：2GB<br>データ転送/月：10GB<br><br>1imageが200MBで、デプロイ毎に転送が起こるとデータ転送1GB/月はすぐ到達するので実質有料。<br><br>proなら10GBは1image200MBなら50回/月, 1image500MBなら20回/月デプロイが可能。 |
| Docker Hub | Private リポジトリ1つなら無料。5つなら7$/月。それ以上の課金プランあり。 |
| Amazon Elastic Container Registry (ECR) | ストレージ：月額0.1 $ / GB<br>データ転送（アウトバウンド）：月額0.14$ / GB<br>12ヶ月間は無料。<br>公開リポジトリならEC2へのデプロイは無料。EC2以外でも5TB/月まで無料。 |
| Artifactry Registry | ストレージ：0.5GBまで無料<br>データ転送：月額0.042$ / GB（ 転送先がGCPで同リージョン内なら無料） |

| docker image registory | 特記事項 |
| - | - |
| Github Packages(旧Github container registry) | 割と新しい |
| Docker Hub | 老舗。ドキュメント豊富。 |
| Amazon Elastic Container Registry (ECR) | 公開リポジトリなら無料。公開リポジトリにすると知らない誰かがimageをpullしてサービスを立ち上げられてしまうリスクがある。 |
| Artifactry Registry | 無料にするにはサーバをGCPにする必要がある。次の仕事現場がAWSを利用しているためAWSを利用したい。他案の料金が許容範囲外の場合は苦肉の策としてこれにしてサーバもEC2からGCPに変更する。 |

ECRへのpush
https://zenn.dev/kou_pg_0131/articles/gh-actions-ecr-push-image#%E6%BA%96%E5%82%99

Docker Hub / Github Packages
https://docs.github.com/ja/actions/publishing-packages/publishing-docker-images

ECRとDocker Hubの料金比較
https://tracpath.com/works/devops/docker_hub_and_amazon_ec2_container_registry/

GitHub Packagesの料金
https://docs.github.com/ja/billing/managing-billing-for-github-packages/about-billing-for-github-packages

Github Packagesの費用館を試算
https://zenn.dev/junara/articles/c2ae71d0444781

docker imageはどの程度容量を使うのか？
https://blog.shinonome.io/lighter-docker-image/
詳しい方が軽量化しても200MBはかかる。

Artifactry Registryの料金
https://cloud.google.com/artifact-registry/pricing?hl=ja

EC2の料金
https://aws.amazon.com/jp/ecr/pricing/
:::

## コンテナオーケストレーションツール選定（Kubernetes, ECS, docker SWARM比較）
ECSがお手軽でいいかと思っていたが、以下2つの理由でKubernetes + EKSにすることにした。

 - Kubernetesよく聞くので今回の開発で触っておきたい（今まで触ったことがない）
 - 以下の記事を読んで世の中の流れ的にKubernetesなんだと思った

参考記事。とても面白かったので是非読んでほしい。[Zennのdockerタグ](https://zenn.dev/topics/docker?order=alltime)の中でぶっちぎりにLikeを貰ってるだけはある。
https://zenn.dev/ttnt_1013/articles/f36e251a0cd24e

決断の元となった記述↓
 - コンテナオーケストレーションツールは2015年にKubernetes、Docker Swarm、Apache Mesosの3つが有名になった
 - 2017~8年には3大クラウド（Azure, AWS, GCP)が全てKubernetesに対応しKubernetes一強となった。
   - AKS(Azure Container Service)
   - Amazon EKS（Amazon Elastic Container Service for Kubernetes）
   - GKS（Google Kubernetes Engine）

なるほどクラウド大きな会社はKubernetesを意識しているし、AmazonもKubernetesが主流になると思ってEKS作ったのね、と思った。
他、実用的な部分の判断材料は以下リンクを参考にした。
https://qiita.com/MetricFire/items/35a6d1a7297963661a64

まとめ

| コンテナオーケストレーションツール | 今後の覇権 | お手軽さ | 応用 | 開発元 |
| - | - | - | - | - |
| Kubernetes | ◎ | ○ | ◎ | オープン（Google） |
| ECS on Fargate |  | ◎ | ○ | Amazon |
| docker SWARM |  | ○ | ○ | docker |

:::details その他参考
https://zenn.dev/esaka/articles/2d117655af1f03cf2444
https://zenn.dev/gege/articles/80b55c345cc1cb
https://zenn.dev/akb428/articles/49e51d4db36896
:::

## golang imageの選定
dockerでgolangアプリケーションをビルドする際に利用するベースイメージを選定する。
今回は [Docker Hub](https://hub.docker.com/_/golang) の公式イメージのうち、通常のイメージを利用する。
理由は容量を抑えるためにalpineにしたかったが、golangのalpineイメージはまだかなり実験的な段階のようで、docker初心者の自分がこのタイミングで手を出すべきではないと判断した。今後dockerイメージを小さくしたい場合に検討する。
→ github actionsでdockerをbuildする場合[alpineは使えない可能性がある](https://docs.github.com/ja/actions/creating-actions/dockerfile-support-for-github-actions#supported-linux-capabilities)。

| golang imageの種類 | 説明 | 利用の注意点 |
| - | - | - |
| golang:<version> | これがデファクトイメージです。 自分のニーズが何かわからない場合は、おそらくこれを使用するとよいでしょう。 これは、使い捨てコンテナ (ソース コードをマウントし、コンテナを起動してアプリを起動する) としても、他のイメージを構築するためのベースとしても使用できるように設計されています。 | これらのタグの中には、本の虫やブルズアイなどの名前が含まれている場合があります。 これらは Debian のリリースのスイートのコード名であり、イメージがどのリリースに基づいているかを示します。 イメージに付属しているもの以外に追加のパッケージをイメージにインストールする必要がある場合は、Debian の新しいリリースの際に破損を最小限に抑えるために、これらのいずれかを明示的に指定する必要があるでしょう。 |
| golang:<version>-alpine | このイメージは、Alpine 公式イメージで入手できる、人気のある Alpine Linux プロジェクトに基づいています。 Alpine Linux は、ほとんどのディストリビューション ベース イメージ (約 5MB) よりもはるかに小さいため、一般的にイメージがよりスリムになります。 | この亜種は非常に実験的なものであり、Go プロジェクトでは正式にサポートされていません (詳細については golang/go#19938 を参照してください)。<br><br>注意すべき主な注意点は、glibc やその友達の代わりに musl libc を使用するため、予期しない動作が発生する可能性があることです。 発生する可能性のある問題の詳細や、Alpine ベースのイメージの使用の長所と短所の比較については、この Hacker News コメント スレッドを参照してください。<br><br>イメージ サイズを最小限に抑えるために、追加の関連ツール (git、gcc、bash など) は Alpine ベースのイメージには含まれていません。 このイメージをベースとして使用し、必要なものを独自の Dockerfile に追加します (慣れていない場合は、パッケージのインストール方法の例については、アルパイン イメージの説明を参照してください)。 詳しい説明については、docker-library/golang#250 (コメント) も参照してください。 |
| golang:<version>-windowsservercore | このイメージは Windows Server Core (microsoft/windowsservercore) に基づいています。 そのため、Windows 10 Professional/Enterprise (Anniversary Edition) や Windows Server 2016 など、そのイメージが動作する場所でのみ動作します。 | Windows 上で Docker を実行する方法については、Microsoft が提供する関連する「クイック スタート」ガイドを参照してください。 |

:::details 参考
https://zenn.dev/ken3pei/articles/1abbf7d974cf5d
https://hub.docker.com/_/golang
https://www.engilaboo.com/go-docker/
https://blog.inductor.me/entry/alpine-not-recommended
https://kleinblog.net/alpine-apk-cmd.html
:::

## docker slimとは
image容量を減らすための圧縮みたいなこと
https://github.com/slimtoolkit/slim

## Goアプリケーションのレイアウト
Go のディレクトリ構成スタンダート
https://github.com/golang-standards/project-layout/blob/master/README_ja.md

# Appendix
## ボツ案：ECS on Fargateか？ ECS on EC2か？
EKSを利用することにしたので以下の検討はボツ。


ECSをどう動かすかを決める必要がある。
https://aws.amazon.com/jp/ecs/

|  | ECS on Fargate | ECS on EC2 |
| - | - | - |
| どんなものか？ | ホストマシンを意識する必要がなく、vCPUと、メモリの量で課金されるため、お気軽にスタートできる。EC2のレイヤを省略できる、ただし高い。およそ40%くらい高いらしい。（参考記事を参照）| EC2にECSコンテナーエージェントをインストールして、ECSに登録する方式。EC2の制約を考慮する必要がある。 |
| 値段 | 高い | 安い |
| 設計での考慮事項 | 少ない | 多い |
| 運用コスト | 低い | 高い |
| セキュリティ考慮事項 | 少ない | 多い |

```
EC2と同等のCPUパフォーマンスをFargateで出そうとすると、コストは40%アップする
```

参考記事
https://tech.uzabase.com/entry/2022/12/01/175423

元々1000円くらいしかかからない想定なので多少高くなっても良い。
EC2の設定周りが省けるの大変助かる。
Fargateを採用。

## Zennで大きな表を見やすくする方法はないものか
Zennだと横幅が狭すぎて見づらくなったので分割した表。これをZenn上でストレスなく閲覧したい。

| docker image registory | どんなものか | 料金 | 特記事項 | 印象 | 採用 |
| - | - | - | - | - | - |
| Github Packages(旧Github container registry) | GitHubが提供するコンテナレジストリで、GitHubとのシームレスな統合が可能。GitHub ActionsやGitHubパッケージとの連携が容易であり、アクセス制御やバージョン管理もGitHubと一元化される。 | 無料で以下が利用可能<br>ストレージ：500MB<br>データ転送/月：1GB<br>それ以上は課金が必要<br><br>github pro(4$/月）なら<br>ストレージ：2GB<br>データ転送/月：10GB<br><br>1imageが200MBで、デプロイ毎に転送が起こるとデータ転送1GB/月はすぐ到達するので実質有料。<br><br>proなら10GBは1image200MBなら50回/月, 1image500MBなら20回/月デプロイが可能。 | 割と新しい<br>俺gitのプランproだった（昔申し込んで忘れてた）<br><br>前回のFE開発はデプロイが17回/月だったので許容範囲。 | 20回やれるならこれで問題ない。github actionsとの親和性も高くて良さそう。しかしEC2との親和性の点でECRに劣る。（こっちはプライベートでできるが、個人趣味でやるので公開リポジトリでいいのでプライベートでできる利点が活かせない） |  |
| Docker Hub | Dockerが提供する公開コンテナレジストリ。オープンソースプロジェクトや個人プロジェクトに人気があり、多くの公開イメージが利用できる。プライベートリポジトリの利用も可能だが、有料プランが必要。 | Private リポジトリ1つなら無料。5つなら7$/月。それ以上の課金プランあり。 | 老舗。ドキュメント豊富。 | 無料なのでこれでもいいかと思った。ただgithub acitonsとの親和性の点でgithub packagesに、EC2との親和性の点でECRに軍配が上がる。<br>扱うのはDocker imageだからDocker Hub使うのが正統、と言う考え方もあるが、EC2にpushする記事があまりにもECRが多いのでECRを使うべきかと思っている。<br>あとimageが2個以上だと急に700円/月もかかるのも、他が従量課金なので高く感じてしまう。 |  |
| Amazon Elastic Container Registry (ECR) | AMAZONが提供するAWSのクラウド環境で使用するためのコンテナレジストリ。AWSのアカウントとのシームレスな統合が可能であり、セキュリティとスケーラビリティの面で強力。 | ストレージ：月額0.1 $ / GB<br>データ転送（アウトバウンド）：月額0.14$ / GB<br>12ヶ月間は無料。<br>公開リポジトリならEC2へのデプロイは無料。EC2以外でも5TB/月まで無料。 | 公開リポジトリなら無料。公開リポジトリにすると知らない誰かがimageをpullしてサービスを立ち上げられてしまうリスクがある。 | EC2に配る場合に他と比較せずこれを利用している記事がたくさんあるが、一応EC2に配るのは他のリポジトリでもできる。が、AWSはECR->EC2をメイン動線としていてここが一番手厚い（逆に他は予想外の何かが発生しやすい）ことは明らか。<br><br>公開リポジトリで知らない誰かがimageをpullしてサービスを立ち上げられてしまうリスクは、個人開発で気にするリスクではない。もしサービスが大きくなり、プライベートリポジトリの料金を賄えるようになったらプライベートに切り替える。 | O<br><br>github packagesと迷ったが実質無料でできるならEC2と親和性の高いECRでやる方がいいと判断。値段が変わらないなら勉強のためにAWS使いたいし。 |
| Artifactry Registry |  | ストレージ：0.5GBまで無料<br>データ転送：月額0.042$ / GB（ 転送先がGCPで同リージョン内なら無料） | 無料にするにはサーバをGCPにする必要がある。次の仕事現場がAWSを利用しているためAWSを利用したい。<br>他案の料金が許容範囲外の場合は苦肉の策としてこれにしてサーバもEC2からGCPに変更する。 |  |  |