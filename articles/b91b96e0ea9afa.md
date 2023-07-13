---
title: "よく使うAWS CLIコマンド"
emoji: "😊"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: 
  - "awscli"
  - "aws"
  - "ecr"
published: true
---
# 概要
AWSを勉強するため使い方をまとめていく

# AWS CLIの認証
docker pushの前に毎回行う作業。AWS CLIからECR公開レジストリにログインするためにaws configure sso, ecr-public get-login-passwordを行う。
# SSOログイン
シングルサインオンのために認証を行う。
```
aws configure sso
```
コマンド後にブラウザ認証を挟む。ブラウザ認証ではIAMにログインした後、「Allow」ボタンを押すこと。
ブラウザ認証前の入力値
| 設定 | 説明 | 値の例 |
| - | - | - |
| SSO session name | ssoセッション名 | aichatbot-sso1 |
| SSO start URL [None] | SSOのstart URL。毎回取得すること。[IAM Identtiy Centerのダッシュボード](https://ap-southeast-2.console.aws.amazon.com/singlesignon/home?region=ap-southeast-2#!/instances/8259577a3b0bdf9a/dashboard)の「AWS アクセスポータルのURL」の値を使う。 | https://XXXXXXXXXXXXX.awsapps.com/start |
| SSO region [None] | リージョン | ap-southeast-2 |
| SSO registration scopes [None] | レジストリスコープ | sso:account:access |

ブラウザ認証後の入力値
| 設定 | 説明 | 値の例 |
| - | - | - |
| AWS account | AWSアカウント。複数ある場合は選択する。1つしかない場合は入力項目が省略される。 | PowerUserAccess |
| CLI default client Region [None] | リージョン | ap-southeast-2 |
| CLI default output format [None] | 入力フォーマット | json |
| CLI profile name [PowerUserAccess-xxxxxxxxxxxxxxx] | プロファイル名（~/aws/config内のプロフィール名） | <入力なし（デフォルトを利用）> |

## ecr-public get-login-password
SSOのパスワード取得を行う。（公式の[AWS CLIでのAmazon ECRの使用](https://docs.aws.amazon.com/ja_jp/AmazonECR/latest/userguide/getting-started-cli.html)から追っていくと辿り着けない。）

```
aws ecr-public get-login-password --region <リージョン名> --profile <profile名>

例（XXXX部分は~/.aws/configを参照する）
aws ecr-public get-login-password --region us-east-1 --profile PowerUserAccess-588666138300 | docker login --username AWS --password-stdin public.ecr.aws/j3a2u6t5
```

| オプション | 意味 | 備考 |
| - | - | - |
| --region | リージョンを指定する |  |
| --profile | プロファイル（./.aws/config内の設定グループ名）を設定する |  |

詳細
https://awscli.amazonaws.com/v2/documentation/api/latest/reference/ecr-public/get-login-password.html

:::details 参考
https://awscli.amazonaws.com/v2/documentation/api/latest/reference/ecr-public/get-login-password.html
https://zenn.dev/tofucode/articles/b0f6282c323879
:::

# 参照
https://awscli.amazonaws.com/v2/documentation/api/latest/index.html
https://awscli.amazonaws.com/v2/documentation/api/latest/reference/ecr-public/index.html