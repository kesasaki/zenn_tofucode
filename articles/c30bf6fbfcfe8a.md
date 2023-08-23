---
title: "2回目以降のECRへのDockerイメージ更新手順"
emoji: "🐈"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: 
  - "docker"
  - "dockerfile"
  - "ecr"
  - "aws"
  - "awscli"
published: true
---

# 概要
以下3記事で紹介した手順だが、2回目以降のECRへのプッシュは毎回同じ割に面倒だったため記事としてまとめた。

https://zenn.dev/tofucode/articles/efd13a3e65afa3
https://zenn.dev/tofucode/articles/108b5310bd7712
https://zenn.dev/tofucode/articles/b0f6282c323879

# 手順
ECRのSSO start URLを[IAM Identtiy Centerのダッシュボード](https://ap-southeast-2.console.aws.amazon.com/singlesignon/home?region=ap-southeast-2#!/instances/8259577a3b0bdf9a/dashboard)の「AWS アクセスポータルのURL」から取得しておく。

```bash
# dockerイメージのビルド
docker build -t aichatbot-app ./

# 利用するAWS CLI profile名をメモる
cat ~/.aws/config

# AWSのSSOログイン
aws configure sso
SSO session name (Recommended): aichatbot-sso
SSO start URL [None]: https://XXXXXXXXXXXXX.awsapps.com/start
SSO region [None]: ap-southeast-2
SSO registration scopes [None]: sso:account:access
### ブラウザが開くのでAWSへログインする
### ブラウザ上で「Allow」ボタンを押す
### 2つ以上AWSアカウントがある場合はコンソールで選択する
#CLI default client Region [None]: ap-southeast-2
CLI default client Region [None]: ap-northeast-1
CLI default output format [None]: json
CLI profile name [PowerUserAccess-588666138300]:

# ECRログイン
aws ecr-public get-login-password --region us-east-1 --profile PowerUserAccess-588666138300 | docker login --username AWS --password-stdin public.ecr.aws/j3a2u6t5

# Dockerイメージにタグを付ける
docker tag aichatbot-app public.ecr.aws/j3a2u6t5/aichatbot-repository:latest

# ECRにDockerイメージをプッシュ
docker push public.ecr.aws/j3a2u6t5/aichatbot-repository:latest
```

# その後はECSへデプロイ
ECSデプロイは以下の「サービス作成」以降を行う。
https://zenn.dev/tofucode/articles/33e34d7fcdbbdd

# 感想
早くgithub actions連携をしたい。
