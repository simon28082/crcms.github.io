---
title: ubuntu18.04 安装
date: 2019-04-13 03:25:31
tags:
---

## 简述
尝试了`windows`,`macos`,`ubuntu`，最后还是觉得`ubuntu`好用。
可能是屌丝本质或者我没有领悟其精髓，`macos`在我看来并不好用，但界面确实还可以。
为了方便我折合了大部分于shell中，可以直接运行shell安装 [https://github.com/crcms/system-initialization](https://github.com/crcms/system-initialization)

来几张图看看效果

![ubuntu桌面1](/images/ubuntu-install/1.png)
![ubuntu桌面2](/images/ubuntu-install/2.png)

## 工具

### 基础功能包
```bash
# apt update
sudo apt-get update

sudo apt-get install -y vim unzip git
```

### 最主要的是什么，当然是`terminal`，推荐`deepin terminal`个人感觉非常好用
```bash
sudo apt-get install zssh
sudo apt-get install deepin-terminal
```

### 输入法
```bash
# 搜索拼音
wget http://cdn2.ime.sogou.com/dl/index/1524572264/sogoupinyin_2.2.0.0108_amd64.deb -O suogoupingyin.deb
sudo dpkg -i suogoupingyin.deb
sudo apt-get install -f
sudo dpkg -i suogoupingyin.deb
# 五笔
sudo apt-get install fcitx-table-wubi
```

### 常用功能包
```bash
git clone https://gitee.com/wszqkzqk/deepin-wine-for-ubuntu.git ~/deepin-wine-for-ubuntu
sudo bash ~/deepin-wine-for-ubuntu/install.sh
# qq
wget http://mirrors.aliyun.com/deepin/pool/non-free/d/deepin.com.qq.im/deepin.com.qq.im_8.9.19983deepin23_i386.deb -O qq.deb
sudo dpkg -i qq.deb
# 徾信
wget http://mirrors.aliyun.com/deepin/pool/non-free/d/deepin.com.wechat/deepin.com.wechat_2.6.2.31deepin0_i386.deb -O weichat.deb
sudo dpkg -i weichat.deb
# 百度网盘
wget http://mirrors.aliyun.com/deepin/pool/non-free/d/deepin.com.baidu.pan/deepin.com.baidu.pan_5.7.3deepin0_i386.deb -O pan.deb
sudo dpkg -i pan.deb
# 迅雷
wget http://mirrors.aliyun.com/deepin/pool/non-free/d/deepin.com.thunderspeed/deepin.com.thunderspeed_7.10.35.366deepin18_i386.deb -O xunlei.deb
sudo dpkg -i xunlei.deb
```
> 这里非常感谢开源作者 https://github.com/wszqkzqk/deepin-wine-ubuntu

### oh my zsh
```bash
sudo apt-get install -y curl zsh
zsh --version
chsh -s $(which zsh)
echo $SHELL
cd ~
sudo sh -c "$(curl -fsSL https://raw.githubusercontent.com/robbyrussell/oh-my-zsh/master/tools/install.sh)"
source ~/.zshrc
```

#### 增加代码提示插件

```bash
git clone https://github.com/zsh-users/zsh-autosuggestions ${ZSH_CUSTOM:-~/.oh-my-zsh/custom}/plugins/zsh-autosuggestions
```

#### 修改皮肤，增加Plugin

```bash
vim ~/.zshrc
# ZSH_THEME="af-magic" (个人比较喜欢这一款)
# plugins=(git zsh-autosuggestions)
```

### Socks代理

#### shadowsocks 安装
```bash
shadowsocks_path=/opt/shadowsocks
# 直接是本地已经包了
sudo cp -r ./tools/shadowsocks /opt
sudo chmod +x ${shadowsocks_path}/Shadowsocks-Qt5-3.0.1-x86_64.AppImage
sudo cp ${shadowsocks_path}/shadowsocks.desktop /usr/share/applications
sudo apt-get install -y python-pip
sudo pip install -U genpac
genpac --proxy="SOCKS5 127.0.0.1:1080" --gfwlist-proxy="SOCKS5 127.0.0.1:1080" -o autoproxy.pac --gfwlist-url="https://raw.githubusercontent.com/gfwlist/gfwlist/master/gfwlist.txt"
```
> shadowsocks的下载地址 https://github.com/shadowsocks/shadowsocks-qt5

#### 命令行开启代理
在`bashrc`或`zshrc`中加入两函数
```bash
# proxy
function proxy_off() {
     unset all_proxy
     echo -e "已关闭代理"
}

function proxy_on() {
     export no_proxy="localhost,127.0.0.1,localaddress,.localdomain.com,.test"
     export all_proxy=socks5://127.0.0.1:1080
     echo -e "已开启代理"
}
```
> 需要时在命令行中执行`proxy_off`或`proxy_on`即可

### 开发工具包
```bash
# gitkraken
wget https://release.axocdn.com/linux/gitkraken-amd64.deb -O gitkraken.deb
sudo dpkg -i gitkraken.deb

# PhpStorm
wget https://download.jetbrains.8686c.com/webide/PhpStorm-2018.3.5.tar.gz -O PhpStorm.tar.gz
sudo tar xfz PhpStorm.tar.gz -C /opt/
sudo chmod +x Php*/bin/phpstorm.sh

# Goland
wget https://download.jetbrains.8686c.com/go/goland-2018.3.5.tar.gz -O goland.tar.gz
sudo tar xfz goland.tar.gz -C /opt/
sudo chmod +x Go*/bin/goland.sh
```

## 美化，强制`macos`

![tweak](/images/ubuntu-install/3.png)
![tweak](/images/ubuntu-install/4.png)


```bash
sudo apt-get update
sudo apt-get install -y gnome-tweak-tool gnome-shell-extensions gnome-shell-extension-dashtodock gnome-shell-extension-top-icons-plus chrome-gnome-shell
```


### 主题安装
```bash
# 安装GTK3主题或 McHigh Sierra 
# https://github.com/vinceliuice/Sierra-gtk-theme/raw/releases/Sierra-light.tar.xz
# https://github.com/vinceliuice/Sierra-gtk-theme/raw/releases/Sierra-dark.tar.xz
wget https://dl.opendesktop.org/api/files/download/id/1523902544/s/ede5bc5c844b290e6e136ca68ae22cb2b72575769e1e9e6488acbfb3979c31b92fe01e8403fafe485a6c4c6f1b4a2fd1cd81806b455d6ffb613aac886ab9755d/t/1553693498/u//X-Arc-Collection-v1.4.9.zip -O X-Arc-Collection.zip
unzip -d /usr/share/themes X-Arc-Collection.zip 

# 下载Mac图标主题 la-capitaine-icon-theme
wget https://github.com/keeferrourke/la-capitaine-icon-theme/archive/v0.6.1.tar.gz -O la-capitaine-icon.tra.gz
tar zxvf la-capitaine-icon.tra.gz -C /usr/share/icons
```

### 应用主题

- `tweak->外观->应用主题或Shell`
- 重启`Gnome`


## 常见问题

#### 1、开启`TopIcons Plus`，增加系统`shell`托盘
- 下载`top-icons`
```bash
# 已安装跳过
sudo apt-get install -y gnome-shell-extension-top-icons-plus
```
- `tweak->扩展->开启Topicons plus`

#### 2、如何打开`tweak`

- 在应用程序中输入`tweak`

#### 3、如何重启`Gnome`
```
[Alt + F2] -> [输入 r] -> [点击 Enter]
```

#### 4、使用`[Dash to dock]`插件时同时出现两个dock的问题
在`tweak`设置里关闭`[Dash to dock]`，关闭后 `[Dash to dock]` 仍然正常工作，但是不会同时出现两个`dock`栏。

#### 5、`tweak`无法选择shell主题
需要在`tweak->扩展->开启User Themes`

#### 6、中文环境下将`Home`目录下的文件夹切换为英文名
```bash
# 设置英文语言环境 (恢复-> zh_CN)
export LANG=en_US
# 更新目录
xdg-user-dirs-gtk-update
```

#### 7、`18.04`开机启动特别慢的问题
```bash
# 列出程序开机占用时间排行
systemd-analyze blame
# 禁用plymouth
sudo systemctl mask plymouth-start.service
sudo systemctl mask plymouth-read-write.service
```

#### 网易云音乐`1.1.0`后不能打开的解决方法

```bash
# 使用root权限命令行后台启动并且屏蔽输出
sudo netease-cloud-music > /dev/null 2>&1 &
# 规避session-manager引起的bug
alias netease='unset SESSION_MANAGER && netease-cloud-music'
netease > /dev/null &
```

推荐使用这个第三方`client`
[https://github.com/trazyn/ieaseMusic](https://github.com/trazyn/ieaseMusic)
