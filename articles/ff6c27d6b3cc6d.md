---
title: "よく使うdockerコマンドとdockerfile設定"
emoji: "💬"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: 
  - "docker"
  - "dockerfile"
published: true
---

# 概要
dockerを勉強するため使い方をまとめていく

# dockerfile
docker build時にdocker イメージをビルドするために利用するファイル。中身はコマンドで、順次実行しイメージを作成する。

| 命令 | 意味 | 備考 |
| - | - | - |
| FROM | イメージビルドのための処理ステージを初期化し、ベースイメージを設定する。 |  |
| WORKDIR | 以降に続く RUN 、 CMD 、 ENTRYPOINT 、 COPY 、 ADD 命令の処理時に（コマンドを実行する場所として）使う 作業ディレクトリworking directory を指定する。なければ作る。 |  |
| COPY | 追加したいファイル、ディレクトリを <コピー元> で指定すると、これらをイメージのファイルシステム上のパス <コピー先> に追加する。<br><br>```COPY [--chown=<ユーザ>:<グループ>] <コピー元>... <コピー先>```<br>```COPY [--chown=<ユーザ>:<グループ>] ["<コピー元>",... "<コピー先>"]``` |  |
| RUN | 現在のイメージの最上位の最新レイヤーにおいて、あらゆるコマンドを実行し、処理結果を確定する。 |  |
| CMD | コンテナ実行時のデフォルト（初期設定）を指定する。複数の CMD 命令があれば、最後の CMD のみ有効。<br><br>```CMD ["実行ファイル","パラメータ1","パラメータ2"] （ exec形式。推奨）```<br>```CMD ["パラメータ1", "パラメータ2"] （ ENTRYPOINT のデフォルトパラメータ ）```<br>```CMD コマンド パラメータ1 パラメータ2 （シェル形式）```<br> |  |

:::details 参考
https://docs.docker.jp/engine/reference/builder.html?highlight=dockerfile
https://matsuand.github.io/docs.docker.jp.onthefly/engine/reference/builder/

ベストプラクティス
https://docs.docker.com/develop/develop-images/dockerfile_best-practices/
:::

Github Actionsで利用する場合のDockerfileの注意点
| 命令 | 注意点 | 備考 |
| - | - | - |
| FROM | 公式のDockerイメージを使う。GitHub Actionsは、DockerがサポートするデフォルトのLinuxの機能をサポートする。 |  |
| WORKDIR | 利用しない（github actionsの環境変数 GITHUB_WORKSPACEで設定する） |  |
| ENTRYPOINT | github actionsでentrypointを定義するとこちらが上書きされる。<br>WORKDIR を使用せず、絶対パスを使用する。<br>exec 形式の場合、アクションのメタデータ ファイルの args はコマンド シェルで実行されない。変数の置換が必要な場合は、shell 形式を使用する。<br>アクションのメタデータ ファイルで定義されている args を、ENTRYPOINT で exec 形式を使用している Docker コンテナーに指定するには、ENTRYPOINT 命令から呼び出す entrypoint.sh というシェル スクリプトを作成する。 |  |
| CMD | アクションのメタデータ ファイルで args を定義すると、args によって、Dockerfile で指定された CMD 命令がオーバーライドされる。 |  |

https://docs.github.com/ja/actions/creating-actions/dockerfile-support-for-github-actions

# build

Dockerfile からイメージを構築する。

```
docker build [オプション] パス | URL | -

例
docker build -t my-golang-app .
```

オプション
| オプション | 意味 | 備考 |
| - | - | - |
| -t | 名前と、オプションでタグを 名前:タグ の形式で指定（= --tag） |  |

:::details 参考
https://docs.docker.jp/engine/reference/commandline/build.html
https://matsuand.github.io/docs.docker.jp.onthefly/engine/reference/commandline/build/
:::

# run
プロセスを隔離（isolated；分離）したコンテナ内（isolated container）で実行する。

```
docker run [オプション] イメージ[:タグ|@ダイジェスト値] [コマンド] [引数...]

例
docker run -it --rm --name my-running-app my-golang-app
```

オプション
| オプション | 意味 | 備考 |
| - | - | - |
| --rm | コンテナの終了時に、自動的にコンテナをクリーンアップし、ファイルシステムを削除する |  |
| -it | -iと-tを同時に指定する書き方 | コマンド上でrunする場合によくつけるオプション |
| -i | アタッチされていなくても STDIN は開放し続ける。（= --interactive） |  |
| -t | 擬似 TTY（標準入出力先） を割り当てる。（= --tty） |  |
| --name | コンテナーに名前を割り当てる |  |
| -v | ボリュームのバインド・マウント（= --volume） |  |
| -w | コンテナー内部のワーキングディレクトリ。（--workdir） |  |

:::details 参考
https://docs.docker.jp/engine/reference/run.html
https://matsuand.github.io/docs.docker.jp.onthefly/engine/reference/commandline/run/
:::
