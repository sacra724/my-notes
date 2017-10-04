# yarn

* yarn インストール

## yarn インストール

### 手順

```zsh
$ brew install yarn --without-node
$ brew upgrade yarn

# パス反映
# export PATH="$PATH:/opt/yarn-[version]/bin"
$ source ~/.zshrc

$ yarn --version
```

### 使い方

```zsh
$ yarn init
$ yarn add [package]
$ yarn add [package] -D
= $ yarn add [package] --dev
$ yarn upgrade [package]
$ yarn remove [package]

```
