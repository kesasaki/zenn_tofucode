---
title: "転職後のM1MACセットアップ"
emoji: "🐥"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: 
  - "M1MAC"
  - "iterm2"
  - "tmux"
  - "VSCode"
  - "MySQLWorkbench"
published: true
---
# 概要
入社後貸与されたMacBookPro M1に開発できるよう色々入れていく。社外秘は書かないので情報に欠落があるかも。
どんな開発をするの？
 - ローカルでgoを動かす
 - DB（MySQL）もローカルにある
    - OracleSQLDeveloperライクにMySQLを操作したい

# 導入した・設定したもの
以下をインストールしたり設定したりした。

| 対象 | 済 | 説明 |
| - | - | - |
| iterm2 | ✅ | 色々設定する |
| tmux | ✅ | 複数セッションをコマンドで切り替え。これがないと始まらない |
| vim | ✅ | VSCodeがメインだが、ちょっと見る時便利。 |
| VSCode | ✅ | 前職ではIntelliJだったが私用でこれを使うので転職先でもVSCodeかな。 |
| Xcode | ✅ | gitコマンドからの依存のため入れる。依存のためmacOS13(Venture)にした。 |
| Sublime Text4 | ✅ | メモ帳がわりに入れる。超便利。 |
| MySQL Workbench | ✅ | OracleSQLDeveloperライクにMySQLを扱いたいので導入。 |
| moom | ✅ | 愛用のウィンドウ操作アプリ。Cmd_→←でウィンドウを画面半分にリサイズ。Windowsと同じ |
| memory diag | ✅ | 愛用のメモリ監視アプリ。1時間に1回最適化してしまう |
| karabiner | ✅ | Windowsキーボード用 |
| outlook | ✅ | Googleカレンダーで管理しているが、ブラウザと別でスケジュールを管理したいのでこちらに連携する。 |
| slackフォルダ分け | ✅ | 怒涛のSlack部屋紹介で頭ごちゃごちゃになったので。 |
| ブラウザお気に入り整理 | ✅ | 保存、引き出し速度を上げたい。 |

# iTerm2とターミナル系
以下の設定を行う。
```bash
# brew インストール
/bin/bash -c "$(curl -fsSL <https://raw.githubusercontent.com/Homebrew/install/HEAD/install.sh>)"
echo 'eval "$(/opt/homebrew/bin/brew shellenv)"' >> ~/.bash_profile
source ~/.bash_profile

# tmux インストール
brew install tmux

# その他
brew install tree
```

https://iterm2.com/
https://formulae.brew.sh/formula/tmux
https://brew.sh/index_ja
https://namileriblog.com/terminal/iterm2_settings/#:~:text=いつでもどこでもHotkey用の,が引っ込んでくれます。
インストール後、 [bash, git, tmux, vimのconfigファイル](https://github.com/kesasaki/config) を設定する。
できた
![](/images/11bac5bbc9bef3/a.png)

# VSCode
https://code.visualstudio.com/download

| プラグイン | 説明 |
| - | - |
| http://Draw.io integration | Draw.io用のプラグイン |
| Japanese Language Pack for Visual | VSCode日本語化プラグイン |
| Go用パッケージ | 開発環境構築で入れる。開発環境構築はZennには書かない。|

# MySQL Workbench
Oracleでは Oracle SQL Developer が便利だったが、ほぼ同等の機能のものがMySQL Workbench のようなので入れる。
https://www.mysql.com/jp/products/workbench/

# その他エディタ
## Sublime Text4
4になってる・・・！✨
https://www.sublimetext.com/download

## XCode
依存で入れることになるので今のうちに入れる
https://developer.apple.com/jp/xcode/
macOSもupdateしないといけないやつだった

# ユーティリティ
## moom
https://apps.apple.com/jp/app/moom/id419330170?mt=12
1200円かかるけど便利

![](/images/11bac5bbc9bef3/b.jpg =600x)
![](/images/11bac5bbc9bef3/c.jpg =600x)
![](/images/11bac5bbc9bef3/d.jpg =600x)

## karabinerキー割り当て
リモートでは自宅でWindows用キーボードを使うので以下でキー設定を変える。
![](/images/11bac5bbc9bef3/e.jpg)
https://www.tarura.com/karabiner-elements-how-to/

# 感想
ワクワクしてきた😄