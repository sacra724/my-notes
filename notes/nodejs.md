# Node.js

* Node.js アンインストール
* nodebrew インストール
* Node.js + nmp インストール

## Node.js アンインストール

### node.jsとnvmの削除

* [Node.jsの管理をnvmからnodebrewに変更した時のメモ
](https://qiita.com/kota2_0/items/ec7dc77955ff0f45f06f)


## nodebrew(nvm) インストール

### nodebrewとは？
node.jsのversionを管理ができるツール

### 手順

```zsh
# nodebrew 落とす
$ curl -L git.io/nodebrew | perl - setup
...
========================================
Add path:

export PATH=$HOME/.nodebrew/current/bin:$PATH
========================================

# nodebrewのパスを追加
$ vi ~/.zshrc などに下記追記
export PATH=$HOME/.nodebrew/current/bin:$PATH

# パス反映
$ source ~/.zshrc

```



## Node.js + nmp インストール

```zsh
# 存在するnode.jsのversion確認
$ nodebrew ls-remote

# versionを指定してインストール
$ nodebrew install v8.0.0

# インストールしたversionを反映する
$ nodebrew use v8.0.0

# デフォルトのバージョン指定
$ nodebrew alias default v8.0.0
default -> v8.0.0

# 別ターミナル起動し確認
$ node -v
v8.0.0

```
