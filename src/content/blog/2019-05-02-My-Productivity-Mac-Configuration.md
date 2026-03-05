---
title: 我的 Mac OS 生产力配置
pubDate: 2019-05-02 22:16:19
tags:
- Mac
---
写一个博客记录自己在 Mac 上做的一些生产力配置。
<!-- more -->
# Terminal 升级
Mac 原生的 Terminal 中规中矩，没有特别好看，也没有特别有效率。因此参考之前的经验决定：
1.  用 iTerm2 代替 Terminal。
2.  用 zsh 代替 bash。
3.  针对 zsh 做部分优化。

## iTerm2 安装与配置
因为 Homebrew 的 update 最近一直很慢，因此直接从 [iTerm2 网站](https://www.iterm2.com/index.html) 下载并安装。
修改 iTerm2 的 Color Schemes，在这个 [iTerm2-Color-Schemes](https://github.com/mbadolato/iTerm2-Color-Schemes) repro 中可以找到很多 schemes，下载后通过`iTerm2` > `Preferences` > `Profiles` > `Colors` > `Color Presets...`来 load 需要的 scheme。

## 安装 zsh 和 oh-my-zsh
Bash 作为 Mac 的默认 shell 已经足够好用，但是 zsh 在一些细节上更胜一筹。通过命令`cat /etc/shells`可以查看当前系统的 shell 类型。通过下面的命令来设置 zsh 作为默认 shell:
```
chsh -s /bin/zsh
```
Zsh 的一个问题是配置起来比较麻烦，然而 [oh-my-zsh](https://ohmyz.sh/) 的出现改变了这一现状。通过 oh-my-zsh 的 theme，可以轻松的实现 zsh 的定制化。
### 安装 oh-my-zsh:
```
$ sh -c "$(curl -fsSL https://raw.github.com/robbyrussell/oh-my-zsh/master/tools/install.sh)"
```
配置 oh-my-zsh 的 theme：`sudo vim ~/.zshrc`。在。zshrc 中找到并修改`ZSH_THEME="agnoster"`
### 安装 nerd font
Agnoster 主题的 prompt 依赖 [nerd font project](https://nerdfonts.com/) 中的一些特殊符号，如果没有，prompt 中会有乱码出现，因此需要按照对应的字体，安装方法：
```
brew tap caskroom/fonts
brew cask install font-sourcecodepro-nerd-font
```
最后到 iTerm2 中，通过`Preferences` > `Profiles` > `Text` > `Change Font`来启用安装的 nerd font`。
### 隐藏 hostname
当前的 hostname 又臭又长，不想在本机使用的时候看到 username@hostname 的形式。按照这个 [issue](https://github.com/agnoster/agnoster-zsh-theme/issues/39#issuecomment-307338817) 中的 comment 来修改 prompt 的形式：
```
# 编辑。zshrc
# 在文件的末尾粘贴如下内容：
prompt_context() {
  if [[ "$USER" != "$DEFAULT_USER" || -n "$SSH_CLIENT" ]]; then
    prompt_segment black default "%(!.%{%F{yellow}%}.)$USER"
  fi
}
```
最终的效果如下：
![600](/images/2019-05-02-My-Productivity-Mac-Configuration/effect-1.png)
## iTerm2 快捷键
```
新建 Tab：Cmd + t
关闭 Tab：Cmd + w
切换 Tab：Cmd + 数字 Cmd + 左右方向键
切换全屏：Cmd + enter

屏幕结果搜索：Cmd + F
剪切板历史记录：Cmd + Shift + H
cmd 历史记录：Cmd + Option + B
Tab 排列：Cmd + Option + E

清除当前行：ctrl + u 
到行首：ctrl + a 
到行尾：ctrl + e 
删除光标之前的单词：ctrl + w 
删除到文本末尾：ctrl + k 
Undo：ctrl + _ 
Paste the last thing to be cut：ctrl + y 
```
## Vim 语法高亮
开启 Vi 语法高亮：
```
#编辑。vimrc
vi ~/.vimrc

#添加
syntax on
```

# SSH 配置
1. 配置好登陆用的私钥。
2. 在`~/.ssh`下面创建 config 文件。
3. 复制私钥到`~/.ssh`的子目录下。
4. 编辑 config 文件：
```
Host <myshortname realname.example.com>
    HostName <realname.example.com>
    IdentityFile ~/.ssh/<realname_rsa> # private key for realname
    User <remoteusername>

# or

Host <hostalias>
    HostName <host ip address>
    IdentityFile ~/.ssh/<realname_rsa> # private key for realname
    User <remoteusername>

```
配置好之后可以直接`ssh <hostname>`登陆远程机器。

## VSCODE 配置
安装了 oh-my-zsh 之后，在 code 的 terminal 中无法显示 Nerd Font 中包含的 icon。根据这个 [post](https://gist.github.com/480/3b41f449686a089f34edb45d00672f28#file-gistfile1-md) 可以解决问题：
1. Install Nerd Fonts
2. Change settings in vscode: `"terminal.integrated.fontFamily": "Hack Nerd Font"`
