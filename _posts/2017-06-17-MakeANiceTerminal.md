---
layout: page
title: 定制一款漂亮的终端
date: 2017-06-17 13:12:13
tags:
- Terminal
- Oh-My-ZSH
- Ubuntu
- Linux
---

> Powerlevel9k 是一款为 ZSH 打造的漂亮终端主题。功能相当强大，第一次安装的时候我被他深深吸引，难怪 Github 上两千多颗星。接下来我们以安装的形式来探索一下这款强大的主题。


**先放张配置好的图**
![powelevel9k]({{ site.url }}/assets/MakeANiceTerminal/powerlevel9k.png "powerlevel9k")

## 安装

该主题可以被 **[Oh-My-Zsh](https://github.com/robbyrussell/oh-my-zsh)**, **[Prezto](https://github.com/sorin-ionescu/prezto)**, **[Antigen](https://github.com/zsh-users/antigen)**, and **[many others](https://github.com/bhilburn/powerlevel9k/wiki/Install-Instructions)** 使用。

### 使用 Oh-My-Zsh 安装

从 Github 上克隆项目到 Oh-My-Zsh 的主题目录下，一般都在 `~/.oh-my-zsh/theme/` 目录。
```
git clone https://github.com/bhilburn/powerlevel9k.git ~/.oh-my-zsh/custom/themes/powerlevel9k
```
修改 `~/.zshrc` 中的 `ZSH_THEME`：
```
ZSH_THEME="powerlevel9k/powerlevel9k"
```


然后重新打开终端就变成了 PowerLine 的形式了。但是由于没有安装相应的字体，导致符号显示不完全。
接下来就要安装 `Awesome-Powerline Fonts` 了。

### 安装 `awesome-terminal-fonts` 

这是一款可以在终端界面显示 awesome 图标的工具。

1. 获取该项目
```
git clone https://github.com/gabrielelana/awesome-terminal-fonts
```

2. 进入该项目，将 `build/` 目录里的所有文件拷贝到 `~/.fonts/` 目录（没有就创建一个）下
```
cp -R build/* ～/.fonts/
```

3. 执行以下命令让 freetype2 知道这些字体
```
fc-cache -fv ~/.fonts
```

4. 自定义 `config/10-symbols.conf` 配置文件里的字体，改成自己喜欢的。当然不改就是默认的。

5. 拷贝 `config/10-symbols.conf` 配置文件到 `~/.config/fontconfig/conf.d` 目录（没有就创建一个）下
```
cp config/10-symbols.conf ~/.config/fontconfig/conf.d
```

6. source 所有 `.fonts` 目录下的字体到你的 shell 启动脚本
```
source ~/.fonts/*.sh
```
至此安装完成，再重启一下终端，效果是不是跟上面一样了。

## 配置

powerlevel9k 提供了丰富的个性化配置功能。

### 两行提示符
powerlevel9k 默认只有一行提示符。

#### 如果你想另起一行，在 `~/.zshrc` 中加入以下定义即可
```
POWERLEVEL9K_PROMPT_ON_NEWLINE=true
```

如图

![prompt_newline]({{ site.url }}/assets/MakeANiceTerminal/prompt_newline.png "prompt_newline")

#### 如果你想让右边的提示符也显示到下一行，只需在以上基础上再加以下定义即可
```
POWERLEVEL9K_RPROMPT_ON_NEWLINE=true
```

如图

![rprompt_newline]({{ site.url }}/assets/MakeANiceTerminal/rprompt_newline.png "rprompt_newline")

#### 如果你想自定义多行连接的符号，定义以下方式即可

```
POWERLEVEL9K_MULTILINE_FIRST_PROMPT_PREFIX="↱"
POWERLEVEL9K_MULTILINE_SECOND_PROMPT_PREFIX="↳ "
```

如图

![custom_line_symbol]({{ site.url }}/assets/MakeANiceTerminal/custom_line_symbol.png "custom_line_symbol")

#### 命令执行完新加一行
```
POWERLEVEL9K_PROMPT_ADD_NEWLINE=true
```

如图

![prompt_end_newline]({{ site.url }}/assets/MakeANiceTerminal/prompt_end_newline.png "prompt_end_newline")


#### 禁用右边的提示符
```
POWERLEVEL9K_DISABLE_RPROMPT=true
```

#### 浅色主题
```
POWERLEVEL9K_COLOR_SCHEME='light'
```

#### 自定义左边提示符的元素
默认的元素只有提供了 context 、 root_indicator，可添加以下元素

元素 | 介绍 |
--- | --- 
os_icon | 系统标识 
battery | 电量 
context | 用户 
dir	| 路径 
dir_writable | 目录读写状态 
load | 加载 
rspec_stats | 统计 
status | 状态 
symfony2_tests | 测试 
user | 当前用户 
vcs	| 版本控制 
vi_mode	| vi 模式 

通过以下定义方式添加，例如：
```
POWERLEVEL9K_LEFT_PROMPT_ELEMENTS=(os_icon root_indicator dir vcs)
```

#### 更多
它的配置还有更多，点击进入 [wiki](https://github.com/bhilburn/powerlevel9k/wiki/Stylizing-Your-Prompt) 查看。

## 使用第三方热门主题
如果你觉得配置麻烦，那就是用热门主题，反正一大堆。

在 [powerlevel9k 的 wiki](https://github.com/bhilburn/powerlevel9k/wiki/Show-Off-Your-Config) 页面，点击即可进入。
