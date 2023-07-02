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

# ecr-public get-login-password
 Amazon ECR 公開レジストリにログインするコマンド。公式の[AWS CLIでのAmazon ECRの使用](https://docs.aws.amazon.com/ja_jp/AmazonECR/latest/userguide/getting-started-cli.html)から追っていくと辿り着けない。


```
aws ecr-public get-login-password --region <リージョン名> --profile <profile名>

例（XXXX部分は~/.aws/configを参照する）
aws ecr-public get-login-password --region us-east-1 --profile AdministratorAccess-XXXX
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