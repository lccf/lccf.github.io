### install
```
# 安装依赖
brew install git autojump fzf
# 安装fish
brew install fish
chsh -s /usr/local/bin/fish
```

### 安装插件
```
fisher git_util omf/theme-default rbenv nvm
```

### 配置提示语
vim ~/.config/fish/config.fish
```
# config welcome text
set fish_greeting 'Talk is cheap. Show me the code.'
```

### 配置vi_mode
```
# key binding
function fish_user_key_bindings
  # vi mode
  fish_vi_key_bindings
  for mode in insert default visual
      # vi模式下ctrl-e到行尾
      bind -M $mode \ce end-of-line
      # vi模式下ctrl-a到行首
      bind -M $mode \ca beginning-of-line
  end
end
# cancel key bindings: fish_default_key_bindings
```

###  配置fzf
brew install fzf
/usr/local/opt/fzf/install
在 fish_user_key_bindings 中添加一行 fzf_key_bindings， 可以使用ctrl-r补全历史命令

### 配置autojump
```
# autojump
[ -f /usr/local/share/autojump/autojump.fish ]; and source /usr/local/share/autojump/autojump.fish
```

### 配置nvm
vim ~/.config/fish/config.fish
```
set -xg NVM_DIR "$HOME/.nvm"
set -xg NVM_PATH "/usr/local/opt/nvm"
set -xg NVM_NODEJS_ORG_MIRROR "https://npm.taobao.org/dist"
```
因为我的nvm是用brew安装的，默认nvm.sh不在~/.nvm目录下，需要编辑nvm脚本
打开 ~/.config/fish/functions/nvm.fish 修改 set -g nvm_prefix 这一行，修改为
```
  set -g nvm_prefix $NVM_PATH
```

### 配置rbenv
rbenv默认需要添加到path中
```
set -U fish_user_paths $HOME/.rbenv/bin $fish_user_paths
```

### 配置命令缩略
fish shell可以通过abbr命令添加命名缩写例：
abbr -a g git
上面的命令将添加 g 为 git 的写，在命令行中敲 g 的时候，可以替代 git。
* abbr 区别于 alias
```
abbr -a b brew
abbr -a g git
```