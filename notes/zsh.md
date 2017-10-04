# zsh

* zsh インストール
* oh-my-zsh インストール

## zsh インストール

### 前提
* Homebrew インストール済み

### 手順

```zsh
$ brew install zsh

# 現在のシェルの確認
$ echo $SHELL

# 利用可能なシェルを表示
$ cat /etc/shells

/bin/bash
/bin/csh
/bin/ksh
/bin/sh
/bin/tcsh
/bin/zsh

# zshに変更
$ chsh -s /bin/zsh

# ターミナル再起動
~

# bashの設定をそのまま反映させる場合
$ cat ~/.bash_profile >> ~/.zshrc
$ source ~/.zshrc

```



## oh my zsh インストール

```zsh
https://github.com/robbyrussell/oh-my-zsh
$ git clone git://github.com/robbyrussell/oh-my-zsh.git~/.oh-my-zsh
$ cp ~/.zshrc ~/.zshrc.origin
$ cp ~/.oh-my-zsh/templates/zshrc.zsh-template ~/.zshrc
$ chsh -s /bin/zsh

# ターミナル再起動

```

## bash に戻したい時

```zsh
echo $SHELL
cat /etc/shells
chsh -s /bin/bash
```