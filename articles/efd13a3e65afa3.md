---
title: "[API構築6] Golang appのdockerビルドと起動"
emoji: "🤖"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: 
  - "golang"
  - "docker"
  - "githubactions"
  - "Dockerfile"
  - "gin"
published: true
---
# 概要
[API作るものと構成](https://zenn.dev/tofucode/articles/d216a5d6c75193) の環境構築のために、[CI/CD周りの技術選定](https://zenn.dev/tofucode/articles/04f0b24bf7ceea) で選んだ技術を用いてGitHub ActionsでGolang ApplicationのdockerビルドとEC2デプロイする。
今回はGolang Appのdockerビルドまで。

コマンドやオプションなどは[よく使うdockerコマンドとdockerfile設定](https://zenn.dev/tofucode/articles/ff6c27d6b3cc6d)にまとめる。

# Golangアプリケーションのdockerビルド
Dockerの基本的な使い方はDockerのgolang公式イメージのリファレンスを参考にする
https://hub.docker.com/_/golang

ginの初歩的なサーバコードについては以下から持ってきた
https://qiita.com/Syoitu/items/8e7e3215fb7ac9dabc3a

ファイルレイアウト
```
.
├── README.md
├── api
│   └── main.go
└── Dockerfile
```

Dockerfile
```Dockerfile
FROM golang:latest

WORKDIR /usr/local/app

COPY go.mod go.sum ./
RUN go mod download && go mod verify

COPY . .
RUN go build -v -o /usr/local/bin/app ./...

CMD ["app"]
```

main.go（最低限のginサーバ）
```go
package main

import "github.com/gin-gonic/gin"

import "net/http"

func main() {
    engine:= gin.Default()
    engine.GET("/", func(c *gin.Context) {
        c.JSON(http.StatusOK, gin.H{
            "message": "hello world",
        })
    })
    engine.Run(":3000")
}
```

buildとrun
```
# go appの作成
go mod init aichatbot
go get -u github.com/gin-gonic/gin
go mod tidy

# docker imageのbuild
sudo docker build -t aichatbot-app ./

# docker起動
docker run -it --rm -p 80:3000 --name aichatbot-app-running aichatbot-app
```

できた。
![](/images/efd13a3e65afa3/ss3.png)

docker Desktop上でも確認できる。
![](/images/efd13a3e65afa3/ss2.jpg)

イメージサイズは1.2GBだった。まだGinしか入れてないのにこれは先行き不安
![](/images/efd13a3e65afa3/ss1.jpg)



参考
https://docs.aws.amazon.com/ja_jp/AmazonECR/latest/userguide/docker-push-ecr-image.html

# 参考
https://docs.github.com/ja/actions/automating-builds-and-tests/building-and-testing-go

https://qiita.com/smi/items/4a7149759bad15f966a0

https://www.utakata.work/entry//golang/tutorial/2-docker-go-gin