+++
date = '2026-06-07T21:49:13+08:00'
draft = false
title = 'linux字体如何解决，如何配置fontconfig？'
tags = ['fontconfig']
+++

## 前言

> fontconfig是linux上管理字体、匹配和渲染偏好的核心系统。
> fontconfig它背后的机制究竟是什么，为什么我按照网上配置后依然无法正常显示？
> 或许你目前被那个xml文件搞的头晕眼花,或许这篇文章能帮到你。

## 配置（fontconfig）
先贴配置,配置路径为~/.config/fontconfig/fonts.conf
```xml
<?xml version='1.0'?>
<!DOCTYPE fontconfig SYSTEM "urn:fontconfig:fonts.dtd">
<fontconfig>
  <match target="pattern">
    <test name="family">
      <string>monospace</string>
    </test>
    <edit name="family" mode="prepend" binding="strong">
      <string>Monaspace Radon NF</string>
      <string>AaManYuShouXieTijf</string>
      <string>LXGW WenKai Mono GB</string>
      <string>JetBrainsMono Nerd Font Mono</string>
      <string>Noto Color Emoji</string>
    </edit>
  </match>
  <match target="pattern">
    <test name="family">
      <string>serif</string>
    </test>
    <edit name="family" mode="prepend" binding="strong">
      <string>Crimson Text</string>
      <string>AaManYuShouXieTijf</string>
      <string>Monaspace Radon NF</string>
      <string>LXGW WenKai Mono GB</string>
      <string>Noto Color Emoji</string>
    </edit>
  </match>
  <match target="pattern">
    <test name="family">
      <string>sans-serif</string>
    </test>
    <edit name="family" mode="prepend" binding="strong">
      <string>LOVEISFREE</string>
      <string>AaManYuShouXieTijf</string>
      <string>Monaspace Radon NF</string>
      <string>LXGW WenKai Mono GB</string>
      <string>Noto Color Emoji</string>
    </edit>
  </match>

</fontconfig>
```

### 解释代码
- 首先配置主要分为三部分serif，sans-serif，monospace,当终端或者应用程序需要这些类型时，
fontconfig会给它提供一个font pattern,而这个font pattern就会包含字体表，若无法输出字体，从上至下依次回退。
- 代码的含义就是match匹配pattern,当字体识别到pattern中字体族为以上三者时，修改（edit）字体顺序为如下...
- mode="prepend"含义：字体选择从上到下匹配，排在前面的会被优先使用
- binding="strong"含义：让你的配置优先级最高，不会被后续系统更新或默认设置覆盖

### 为什么网上很多文章及视频中用alias配置不行？
你大概看到过这样的配置方案
```xml
<alias>
  <family>sans-serif</family>
  <prefer>
    <family>Noto Sans CJK SC</family>
    <family>WenQuanYi Micro Hei</family>
    <family>DejaVu Sans</family>
  </prefer>
</alias>
```
但是配置完后，有些字体并不会有效果。  
装了很多字体，之前配置的那个字体为什么怎么也改不掉。  
即便是把配置注释掉，电脑依然显示之前那个字体，为什么？  
- 我们得知道fontconfig反直觉的一点就是它不是一个你选择哪个字体就按照哪个来，它的目的在于让字体能够，
即使无法匹配也有备选项，因此内部会有一个*字体得分和强弱绑定*机制
- alias默认是binding="weak"，而fontconfig是按照文件数字顺序加载的，很简单，后面执行的match、
bingding="strong"能覆盖前面的alias

## 可能的一些帮助
### 一些命令
这里我贴上我从开始完全不懂到现在为止常用的一些命令
```sh

 fc-cache -fv
 fc-list : family | grep -i "查找字体"
 fc-list : family
 fc-match monospace
 fc-match sans:lang=zh
 fc-match "Monaspace Radon NF"
 fc-match -v "Monaspace Radon NF"
 FC_DEBUG=4 fc-match 'monospace'
 FC_DEBUG=4 应用程序
 FC_DEBUG=4 fc-match serif 2>&1 | less
```

### 字体安装位置
一些字体可以包管理器从aur安装，而如果是自己手动安装的字体，
要装到~/.local/share/fonts/（用户级）或者/usr/local/share/fonts/（系统级），
别装到/usr/share/fonts/那是包管理器的地盘

### 屏幕截图
![浏览器截图](/images/fontconfig.png)

