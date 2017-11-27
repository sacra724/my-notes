# git

* SSH key をgithubに登録する
* gitコマンドをエイリアス登録する

## SSH key をgithubに登録する

### 手順

```zsh
# すでに鍵があるか確認
$ ls -al ~/.ssh

# この辺りがなければない
id_dsa.pub
id_ecdsa.pub
id_ed25519.pub
id_rsa.pub

# 鍵の生成
$ ssh-keygen -t rsa -b 4096 -C "your_email@example.com"

Generating public/private rsa key pair.
Enter file in which to save the key (/Users/ryusakuma/.ssh/id_rsa):
Enter passphrase (empty for no passphrase):
Enter same passphrase again:
Your identification has been saved in /Users/ryusakuma/.ssh/id_rsa.
Your public key has been saved in /Users/ryusakuma/.ssh/id_rsa.pub.
The key fingerprint is:
SHA256:TKMXY3jVKZKaj8lHMY8XCUxloGZm4BP70v4G7cpTpi4 your_email@example.com
The key's randomart image is:
略

# githubにSSH keyを登録する
# id_rsa.pubをコピー
# 不可視ファイルをファイルを見れる設定なら */Users/<user>/.ssh/id_rsa.pub* を開いてコピー
# 見れないなら下記コマンド
$ pbcopy < ~/.ssh/id_rsa.pub

# githubのsettingから任意の名前で鍵を登録。「my key」とかでいいんんじゃない。

```



## gitコマンドをエイリアス登録する

### 手順
```zsh
# マシン全体に反映
$ git config --system alias.st status


# ユーザ単位で反映
$ git config --global alias.st status

# リポジトリ単位で反映
$ git config alias.st status


$ git config --system alias.st status
$ git config --system alias.co checkout
$ git config --system alias.cm commit

```