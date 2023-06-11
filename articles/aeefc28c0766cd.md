---
title: "[FE構築2] github ActionsでS3にデプロイ"
emoji: "🙌"
type: "tech"
topics:
  - "github"
  - "s3"
  - "githubactions"
  - "web"
  - "iam"
published: true
published_at: "2023-04-21 15:24"
---

![](https://storage.googleapis.com/zenn-user-upload/cee4f6e74df3-20230421.png)

前の記事
https://zenn.dev/tofucode/articles/166f10c31fd1c3

# 概要
githubのhtml, js, cssをgithub actionsでS3にデプロイする方法を説明します。

# AWS IAM
AWSでS3へのアクセス権を持ったユーザを作成する。
AWS IAMで以下を作成する。
 - ポリシー
 - ロール
 - ユーザー

ポリシー
```json
{
    "Version": "2012-10-17",
    "Statement": [
        {
            "Sid": "VisualEditor0",
            "Effect": "Allow",
            "Action": "s3:*",
            "Resource": "arn:aws:s3:::<S3バケット名>"
        }
    ]
}
```
# github actions登録
Settings＞Secrets＞ActionsでActions secretsを追加。
| secretsキー | 役割 |
| ---- | ---- |
| AWS_ACCESS_KEY_ID | アクセスキーID |
| AWS_SECRET_ACCESS_KEY | シークレットアクセスキー |

ワークフローを作成する .github/workflows/deploy.yml
```yml
on:
  push:
    branches:
      - main
jobs:
  deploy:
    runs-on: ubuntu-latest
    steps:
      - name: Checkout
        uses: actions/checkout@main  # リポジトリをチェックアウト

      - name: Install Dependencies
        run: npm install

      - name: Build
        run: npm run build

      - name: Deploy  # S3にデプロイ
        env:
          AWS_ACCESS_KEY_ID: ${{ secrets.AWS_ACCESS_KEY_ID }}
          AWS_SECRET_ACCESS_KEY: ${{ secrets.AWS_SECRET_ACCESS_KEY }}
        run: aws s3 cp --recursive --region ap-northeast-1 dist/ s3://<S3バケット名>
```
上記をPRにする。
# github actions実行
RPをマージするとdeployが走る。
![](https://storage.googleapis.com/zenn-user-upload/4bb0fd8c5551-20230421.webp)
デプロイが完了するとS3にpushした内容が反映されている。

# 参考
https://docs.github.com/ja/actions/quickstart
https://docs.github.com/ja/actions/learn-github-actions
https://docs.github.com/ja/actions/using-workflows

次の記事
https://zenn.dev/tofucode/articles/ef7deb00d0f91c