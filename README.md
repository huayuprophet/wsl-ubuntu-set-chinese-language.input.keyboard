# WSLg 中文显示设置指南

WSLg 是微软的 WSL2 中自带的显示图形界面的功能，可以和 Windows 完美融合。不过，由于 WSL 系统比较精简，可能会缺少一些图形包和输入法等功能。

请按需执行以下各步骤。

**前提条件**

1. 一台搭载windows 10/11的计算机设备
2. 计算机设备已安装wsl2和ubuntu/ubuntu-24/ubuntu22，未安装则参考[微软wsl安装官方教程](https://learn.microsoft.com/zh-cn/windows/wsl/install)
3. 在国内，可能需要切换apt源，否则卡的一批，教程可参考[中科大ubuntu-wiki](https://chinanet.mirrors.ustc.edu.cn/help/ubuntu.html)（推荐）或[清华ubuntu-wiki](https://mirror.tuna.tsinghua.edu.cn/help/ubuntu/)
4. 教程将只会指导您使用fcitx5-rime输入法，使用其他输入法请参考其他朋友们发布的博文或issue
5. 教程不支持ubuntu20/18/16/14等更古老的系统
6. 最新的debian系统，大体上也可以按照本教程操作

## 设置中文显示

### 1. 安装依赖包

```bash
sudo apt install language-pack-zh-hans fonts-noto-cjk fonts-noto-cjk-extra
```

> 对于debian系统，您将无需安装 `language-pack-zh-hans`。请执行以下命令
>
> ```bash
> sudo apt install fonts-noto-cjk fonts-noto-cjk-extra
> sudo apt install locales
> sudo locale-gen zh_CN.UTF-8
> ```

### 2. 配置 locales

修改 `/etc/locale.gen` 文件以启用 `en_US.UTF-8` 和 `zh_CN.UTF-8`：

```bash
sudo sed -i 's/^# en_US.UTF-8 UTF-8/en_US.UTF-8 UTF-8/g' /etc/locale.gen
sudo sed -i 's/^# zh_CN.UTF-8 UTF-8/zh_CN.UTF-8 UTF-8/g' /etc/locale.gen
```

生成 locale：

```bash
sudo locale-gen
```

### 3. 设置显示语言为中文

创建或修改 `/etc/default/locale` 文件：

```bash
sudo tee /etc/default/locale <<-'EOF'
LANG=zh_CN.UTF-8
LANGUAGE="zh_CN:zh:en_US:en"
EOF
```

## 配置中文输入法

### 1. 安装 im-config

```bash
sudo apt install zenity im-config
im-config
# 选择 fcitx5，其余选项选择 OK
```

### 2. 安装中文输入法

```bash
sudo apt install fcitx5 fcitx5-chinese-addons fcitx5-frontend-gtk4 fcitx5-frontend-gtk3 fcitx5-frontend-gtk2 fcitx5-frontend-qt5 fcitx5-config-qt fcitx5-frontend-qt6
```

> 对于debian用户，您将还需要补充执行以下命令
>
> ```bash
> sudo apt install dbus dbus-x11
> ```

### 3. 配置环境变量

需要修改 profile 文件以开机自启动 fcitx5 并设置环境变量。以下适用于 bash，如果使用 zsh，请相应修改路径为 `/etc/zsh/zprofile`。

请注意，debian系统需要做少许改动。

```bash
sudo tee -a /etc/profile <<-'EOF'
# eval `dbus-launch --sh-syntax` #debian系统请删除最左侧井号
/usr/bin/fcitx5 --disable=wayland -d --verbose '*'=0
export INPUT_METHOD=fcitx
export GTK_IM_MODULE=fcitx
export QT_IM_MODULE=fcitx
export XMODIFIERS=@im=fcitx
export SDL_IM_MODULE=fcitx
export GLFW_IM_MODULE=ibus
EOF
```

### 4. 重启 WSL

```powershell
# 重启 WSL
wsl --shutdown
wsl
```

### 5. 增加输入法

使用 fcitx5 配置工具：

```bash
fcitx5-configtool
```

在界面中去掉默认输入法，增加中文输入法。

## 配置常用脚本

由于 WSL 的图形界面不完整，无法通过任务栏右击输入法图标执行部署或同步任务，因此需要通过脚本实现。

### 同步脚本

同步位置配置在 `~/.local/share/fcitx5/rime/installation.yaml` 内的 `sync_dir: "/mnt/c/Users/iuxt/OneDrive/sync/rime_sync"`。

```bash
#!/bin/bash
pkill fcitx5
cd ~/.local/share/fcitx5/rime/
rime_dict_manager -s
/usr/bin/fcitx5 --disable=wayland -d --verbose '*'=0
```

### 部署脚本

```bash
#!/bin/bash
pkill fcitx5
rime_deployer --build ~/.local/share/fcitx5/rime/ /usr/share/rime-data ~/.local/share/fcitx5/rime/build
/usr/bin/fcitx5 --disable=wayland -d --verbose '*'=0
```

## 更多技巧

### 1. 中英切换快捷键

先启动 fcitx5 配置工具：

```bash
fcitx5-configtool
```

然后在全局选项里配置快捷键即可，博主通常喜欢使用 `左shift`键。

### 2. 设置输入法字样与大小

默认的配置，输入法字体偏小，在大屏幕设备中尤为明显。

1. 启动fcitx5配置工具
2. 附加组件 - 经典用户界面 - 配置 - 字体 - 选择字体
3. 然后配置字体、字号即可

### 3.共享windows系统字体

使用linux中的诸如wps中文应用时，会缺少宋体等字体。可使用以下命令实现共享宿主机的全部字体。

```bash
sudo ln -s /mnt/c/Windows/Fonts /usr/share/fonts/WindowsFonts
fc-cache -fv
```

### 4. 更多技巧

未完待续，请到issue提交

## 结语

**参考资料**

* [WSLg配置图形支持和配置rime输入法](https://zahui.fan/posts/81886814/)
* [WSL2 安装中文字体](https://blog.csdn.net/oZuoZuoZuoShi/article/details/118977701)
* [linux-安装中文输入法-Fcitx5](https://lamirs.vercel.app/posts/%E5%BC%80%E5%8F%91%E7%8E%AF%E5%A2%83%E9%85%8D%E7%BD%AE-%E8%AE%B0%E5%BD%95-%E6%8A%A5%E9%94%99-%E8%A7%A3%E5%86%B3%E6%96%B9%E6%A1%88/linux-%E5%AE%89%E8%A3%85%E4%B8%AD%E6%96%87%E8%BE%93%E5%85%A5%E6%B3%95-fcitx5.html)

如果教程帮助到你，请为我点一下上面的星星star，这很重要。

谢谢。
