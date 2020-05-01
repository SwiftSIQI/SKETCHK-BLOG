---
title: 'A Better Development Environment for macOS'
comments: true
date: 2019-07-24 19:24:30
updated:
tags:
  - macOS
categories:
  - DIY
---

前段时间公司给配了新电脑, 刚好把电脑重新折腾了一下, 这篇文章记录了一些自己的小心得, 方便日后回顾.

<!-- more -->

## 系统通用设置

* 参考少数派的配置文章：[10 个实用技巧，让 Finder 带你飞 - 少数派](https://sspai.com/post/27403)

  * Finder 窗口提供更多信息
  * 自定义 toolbar
  * 显示文件扩展名
  * 标题栏显示完整路径
  
    ```sh
    defaults write com.apple.finder _FXShowPosixPathInTitle -bool YES
    ```

  * 显示隐藏文件
  
    ```sh
    defaults write com.apple.finder AppleShowAllFiles -boolean true ; killall Finder
    ```

  * 显示用户资料库

    ```sh
    chflags nohidden ~/Library/
    ```

  * 打开`允许安装任意来源软件`的选项

    ```sh
    sudo spctl —master-disable
    ```

## Xcode 的安装

* App Store 下载并安装 Xcode

* 用 `xcode-select -p`  检查 Xcode Command Line Tool 是否安装：如果安装则返回安装目录，否则返回 2

* 激活 command line tool: `Preference -> Location -> command line tools`

## VSCode 安装

* 使用 setting sync 同步配置

  * github token 需要在 `Setting - Developer setting - Personal access tokens` 里配置
  * gist id 是对于 gist 中 url 里用户名后面的 path
    * 上传： `Shift + Opt + U (Sync: Update / Upload Settings)`
    * 下载： `Shift + Opt + D (Sync: Download Settings)`

  * SSH Key 生成
    * `ssh-keygen -t rsa -C “youremail@example.com”`
  
  * 在 github 上注册公钥

## Homebrew 安装

* 安装命令

  ```sh
  /usr/bin/ruby -e “$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/master/install)”
  ```

* 安装常用的 package
  * xcproj
  * coreutils
  * openssl
  * gpg2

> 上面安装的 brew 为下面 OMZ 插件和语言环境需要的 brew

## iTerm 2

* 设置无限回滚
  * 安装好之后在 `Preference -> Profiles -> Terminal -> Scrollback Buffer` 中勾选上 `Unlimted scrollback`

* 开启单词跳转(option + → or ←)，单词删除(option + backspace) 的功能
  * 在 `iTerm → Preferences → Profiles → Keys → Load Preset… → Natural Text Editing`

* 光标设置
  * `iTerm2 → Preferences → Profiles → Text → Blinking cursor :  ON`

* 背景色设置
  * `iTerm2 → Preferences → Profiles → window → transparency`

## ZSH 配置

* 设置 zsh `chsh -s /bin/zsh`
* 安装 oh my zsh

    ```sh
    sh -c "$(curl -fsSL https://raw.githubusercontent.com/robbyrussell/oh-my-zsh/master/tools/install.sh)"
    ```

* 主题配置
  * Powerlevel9K: [GitHub - bhilburn/powerlevel9k: The most awesome Powerline theme for ZSH around!](https://github.com/bhilburn/powerlevel9k)
  * 通过 oh-my-zsh 安装

    ```sh
    git clone https://github.com/bhilburn/powerlevel9k.git ~/.oh-my-zsh/custom/themes/powerlevel9k
    ```

  * 在 `~/.zshrc` 中修改主题

    ```sh
    ZSH_THEME="powerlevel9k/powerlevel9k"
    ```

### ZSH 的进阶修改

* 自定义提示符
  * Prompt Customization 章节有详细说明, 通过这些配置, 可以实现很多方便的功能, 例如展示每个工程中不同语言环境的版本号

    ![01](01.jpg)

* 字体文件

  * Nerd字体: [GitHub - ryanoasis/nerd-fonts: Iconic font aggregator, collection, and patcher. 40+ patched fonts, over 3,600 glyph/icons, includes popular collections such as Font Awesome & fonts such as Hack](https://github.com/ryanoasis/nerd-fonts#option-4-homebrew-fonts)

    * 通过 homebrew 安装

      ```sh
      brew tap homebrew/cask-fonts
      brew cask install font-hack-nerd-font
      ```

    * 在 `~/.zshrc` 中增加对该字体的支持
      * `POWERLEVEL9K_MODE=‘nerdfont-complete’`

    * 修改 terminal 中的字体配置
      * `iTerm → Preferences → Profiles → Text → Change Font`
      * 选择  Hack Nerd Font
      * 记得在 vscode 的 terminal 里同样配置下字体为 Hack Nerd Font, 否则 terminal 会出现乱码

* 安装 oh-my-zsh 插件
  * Ruby相关: rvm/gem/bundler
  * Python相关: pyenv/pip
  * Node.JS相关: nvm/npm/yarn
  * iOS相关: xcode/pod/swiftpm
  * 效率提升相关
    * git
    * osx: 快速操作 terminal，finder，iTunes，Spotify 等工具
    * colored-man-pages: man 命令内容高亮
    * git-open: 在命令行里自动打开 remote 仓库链接，需要在 npm 上安装 npm install —global git-open
    * autojump: 自动跳转到文件，需要在 homebrew 里安装autojump
    * zsh-autosuggestions: 根据历史记录提示可能用到的内容
    * zsh-syntax-highlighting: 命令高亮
    * history-substring-search: 自动查看之前输入的命令
  * 其余插件
    * bat:  [https://github.com/sharkdp/bat](https://github.com/sharkdp/bat)
    * vim 插件
      * Vundle: [GitHub - VundleVim/Vundle.vim: Vundle, the plug-in manager for Vim](https://github.com/VundleVim/Vundle.vim)

## 语言环境配置

### NodeJS && NVM  

* 安装过程中的注意事项
  * 安装完毕后，需要在 zshrc 里添加如下代码

    ```sh
    export NVM_DIR="$HOME/.nvm"
    [ -s "$NVM_DIR/nvm.sh" ] && \. "$NVM_DIR/nvm.sh"  # This loads nvm*
    [ -s "$NVM_DIR/bash_completion" ] && \. "$NVM_DIR/bash_completion"  # This loads nvm bash_completion
    ```

  * 官方给出的 NVM_DIR 定义是 `export NVM_DIR=“${XDG_CONFIG_HOME/:-$HOME/.}nvm”` ，需要注意 `XDG_CONFIG_HOME` 这个定义

### Ruby && RVM

* 安装过程中的注意事项
  * 执行 gpg 命令出现语言问题的情况，需要在 `.zshrc` 里声明当前环境下的语言 `export LANG=en_US.UTF-8`
  * 执行 gpg 命令出现 `keyserver receiver faild` 无法生成的问题时，尝试以下解决方案

    ```sh
    gpg --keyserver hkp://ipv4.pool.sks-keyservers.net --recv-keys 409B6B1796C275462A1703113804BB82D39DC0E3 7D2BAF1CF37B13E2069D6956105BD0E739499BDB
    ```

### Python && pyenv

* 安装过程中的注意事项
  * pyenv 作者给出了自动安装的脚本：[GitHub - pyenv/pyenv-installer: This tool is used to install `pyenv` and friends.](https://github.com/pyenv/pyenv-installer)
  * 需要注意的是，在 zshrc 里面添加下面的代码

    ```sh
    export PATH="$HOME/.pyenv/bin:$PATH"
    eval "$(pyenv init -)"
    eval "$(pyenv virtualenv-init -)"
    ```

## 配色方案

* [Dracula — A dark theme for iTerm and 50+ apps](https://draculatheme.com/iterm/)

## 编程字体

* Input 字体: [Input: Fonts for Code](https://input.fontbureau.com/)
