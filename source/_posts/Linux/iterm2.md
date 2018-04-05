title: iTerm2的使用

date: 2015-7-23

tags:
    - 软件
    - original

categories:
    - Linux
---
自从换了mac后连接远程linux服务器一直用电脑自带的终端，每次都要输密码，烦死了。看同事用iTerm挺方便的，就自己也弄了一个。

### 下载地址

http://www.iterm2.com/

### 记住远程服务器密码

```
set timeout 30
spawn ssh liuhaiming@122.92.222.122 -p51618
expect "*password*"
send “**********\n"
interact
```

<!-- more -->  

### 替换图标

原图标看着不好看,替换一个喜欢的图标
找好自己要替换的图标	
https://dribbble.com/shots/656627-Terminal-Macintosh-Icon	
找到自己要替换图标的app
右键显示简介，复制下载好的图标，并选中要替换的图标 command + v 就替换好了	

### 换个自己喜欢的主题

http://iterm2colorschemes.com/

### 相关命令

command + O: 打开配置方便选择	

<br>