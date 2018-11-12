---
title: Mac平台配置fish shell
date: 2018-01-12 23:34
tags:
- fish
- fish shell
- develop
- linux
categories: Linux
---

从bash切换到fish有一段时间了，换fish之后terminal启动和新开tab速度明显加快。整理一下fish和一些常用shell工具的配置(2018-11-12更新fisher版本)。

* 2018-11-12 fishermen作者删除了fisherman的项目组织，文档相应更新，并更新插件

### 安装

1.安装fish shell及常用组件
```bash
# 安装依赖
brew install git autojump fzf
# 安装fish
brew install fish
```
- git版本控制工具
- autojump快速进入目录的工具，非常好用
- fzf命令行下模糊筛查工具

2.切的shell到fish
```bash
chsh -s /usr/local/bin/fish
```

3.安装fisher
```bash
curl https://git.io/fisher --create-dirs -sLo ~/.config/fish/functions/fisher.fish
```
- fisher提供fish包管理功能
- 安装完成后需要重启一下shell

4.安装插件
```bash
fisher add edc/bass jorgebucaran/fish-git-util oh-my-fish/theme-default \
jethrokuan/fzf daenney/rbenv FabioAntunes/fish-nvm \
danhper/fish-fastdir jhillyerd/plugin-git
```
- edc/bass 可以fish中执行bash
- jorgebucaran/fish-git-util 基础方法，供主题中展示git信息，是oh-my-fish/theme-default的依赖
- oh-my-fish/theme-default 主题插件，提供友好的信息显示
- jethrokuan/fzf 但供fzf快捷键映射
- daenney/rbenv 提供rbenv命令支持（ruby开发者使用）
- FabioAntunes/fish-nvm 提供nvm支持（nodejs开发者使用）
- danhper/fish-fastdir 提供快捷的路径切换功能
- jhillyerd/plugin-git 提供git命令缩写

### 配置
fish配置文件在 ~/.config/fish/config.fish，如果没有则手动创建即可。

1.设置欢迎语
vim ~/.config/fish/config.fish
```bash
# config welcome text
set fish_greeting 'Talk is cheap. Show me the code.'
```

2.配置vi_mode
```bash
# key binding
# vi mode
fish_vi_key_bindings
for mode in insert default visual
    # vi模式下ctrl-e到行尾
    bind -M $mode \ce end-of-line
    # vi模式下ctrl-a到行首
    bind -M $mode \ca beginning-of-line
end
# cancel key bindings: fish_default_key_bindings
```
~~3.配置fzf~~
* 已废弃，jethrokuan/fzf 插件已提供默认配置

4.配置autojump
```bash
# autojump
[ -f /usr/local/share/autojump/autojump.fish ]; and source /usr/local/share/autojump/autojump.fish
```

5.配置nvm
vim ~/.config/fish/config.fish
```bash
set -xg nvm_prefix "/usr/local/opt/nvm"
set -xg NVM_DIR "$HOME/.nvm"
set -xg NVM_PATH "/usr/local/opt/nvm"
set -xg NVM_NODEJS_ORG_MIRROR "https://npm.taobao.org/dist"
```

6.配置rbenv
rbenv默认需要添加到path中
```bash
set -U fish_user_paths $HOME/.rbenv/bin $fish_user_paths
```

7.配置命令缩略
fish shell可以通过abbr命令添加命名缩略。示例：
```bash
abbr -a g git
```
- 上面的命令将添加g为git的写，在命令行中敲g的时候，可以替代git。
- abbr 区别于 alias*

### 完整配置
```bash
# config welcome text
set fish_greeting 'Talk is cheap. Show me the code.'

# nvm
set -xg NVM_DIR "$HOME/.nvm"
set -xg nvm_prefix /usr/local/opt/nvm
set -xg NVM_NODEJS_ORG_MIRROR "https://npm.taobao.org/dist"
set -xg NODE_PATH "$NVM_DIR/versions/node/v8.9.2/lib/node_modules"
set -xg HOMEBREW_BOTTLE_DOMAIN https://mirrors.ustc.edu.cn/homebrew-bottles
# rbenv
set -xg RBENV_ROOT $HOME/.rbenv
# fzf
set -U FZF_LEGACY_KEYBINDINGS 0

# user path
# add user path
# set -U fish_user_paths [path] $fish_user_paths
# set user path
# set -U fish_user_paths /usr/local/opt/mysql@5.5/bin /usr/local/opt/coreutils/libexec/gnubin /usr/local/opt/fzf/bin $HOME/.rbenv/bin $HOME/.nvm/versions/node/v8.9.3/bin $HOME/.composer/vendor/bin /usr/local/bin

# alias
alias g=git
alias tnpm='npm --registry=https://registry.npm.taobao.org --disturl=https://npm.taobao.org/dist'
alias fpview='fzf --preview "head -n 100 {}" | read -l result; and vim $result'
alias fcd='find . -type d -maxdepth 1 | fzf --height 50% --reverse --border | read -l result; and cd $result'
alias fvim='find . -type f -maxdepth 1 | fzf --reverse --preview "head -n 100 {}" | read -l result; and vim $result'
alias fls='ls | fzf --height 50% --reverse --border'
alias rm=trash

# abbr
# add abbr: abbr -a b brew
# abbr b brew
# abbr f fuck
# abbr g git
# abbr gst 'g st'
# abbr gfa 'g fa'
# abbr gss 'g st -s'
# abbr gco 'g co'
# abbr gbr 'g br'
# abbr glg 'g lg'
# abbr gla 'g lga'

#function fish_user_key_bindings
#  bind \cr 'peco_select_history (commandline -b)'
#end

# autojump
[ -f /usr/local/share/autojump/autojump.fish ]; and source /usr/local/share/autojump/autojump.fish

# key binding
# backup fish_user_key_bindings
if functions -q fish_user_key_bindings
  functions -c fish_user_key_bindings custom_fish_user_key_bindings_copy
end
function fish_user_key_bindings
  fish_vi_key_bindings
  # fzf_key_bindings
  for mode in insert default visual
    bind -M $mode \ce end-of-line
    bind -M $mode \ca beginning-of-line
  end
  if functions -q custom_fish_user_key_bindings_copy
    custom_fish_user_key_bindings_copy
  end
end
# cancel key bindings: fish_default_key_bindings

function add_fish_path
  set -U fish_user_paths $argv $fish_user_paths
  echo current: $fish_user_paths
end

function remove_fish_path
  set -U fish_user_paths (string match -v $argv $fish_user_paths)
end
```

### 链接
fish shell https://github.com/fish-shell/fish-shell
fisher https://github.com/jorgebucaran/fisher
autojump https://github.com/wting/autojump
fzf https://github.com/junegunn/fzf
