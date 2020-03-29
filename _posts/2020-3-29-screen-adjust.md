

# 笔记本扩展屏设置方法ubuntu 18.04 lts

xrandr常用命令（这里的VGA与LVDS分别换成第1步中的设备名，如VGA1、LVDS1）：
xrandr --output VGA --same-as LVDS --auto
打开外接显示器(--auto:最高分辨率)，与笔记本液晶屏幕显示同样内容（克隆）
xrandr --output VGA --same-as LVDS --mode 1280x1024
打开外接显示器(分辨率为1280x1024)，与笔记本液晶屏幕显示同样内容（克隆）
xrandr --output VGA --right-of LVDS --auto
打开外接显示器(--auto:最高分辨率)，设置为右侧扩展屏幕
xrandr --output VGA --off
关闭外接显示器
xrandr --output VGA --auto --output LVDS --off
打开外接显示器，同时关闭笔记本液晶屏幕（只用外接显示器工作）
xrandr --output VGA --off --output LVDS --auto
关闭外接显示器，同时打开笔记本液晶屏幕 （只用笔记本液晶屏）

--------------------------------------------------------------------------------------------------------

像许多笔记本用户一样，我经常将笔记本插入到不同的显示器上（桌面上有多台显示器，演示时有投影机等）。运行 `xrandr` 命令或点击界面非常繁琐，编写脚本也不是很好。

最近，我遇到了 [autorandr](https://github.com/phillipberndt/autorandr)，它使用 EDID（和其他设置）检测连接的显示器，保存 `xrandr` 配置并恢复它们。它也可以在加载特定配置时运行任意脚本。我已经打包了它，目前仍在 NEW 状态。如果你不能等待，[这是 deb](https://www.donarmstrong.com/autorandr_1.2-1_all.deb)，[这是 git 仓库](https://git.donarmstrong.com/deb_pkgs/autorandr.git)。

要使用它，只需安装软件包，并创建你的初始配置（我这里用的名字是 close）：

```bash
    autorandr --save close
```

然后，连接你的笔记本（或者插入你的外部显示器），使用 `xrandr`（或其他任何）更改配置，然后保存你的新配置（我这里用的名字是 3s4）：



```bash
autorandr --save 3s4
```

`autorandr` 有 `udev`、`systemd` 和 `pm-utils` 钩子，当新的显示器出现时 `autorandr --change` 应该会立即运行。如果需要，也可以手动运行 `autorandr --change` 或 `autorandr - load 3s4`。你也可以在加载配置后在 `~/.config/autorandr/$PROFILE/postswitch` 添加自己的脚本来运行。我的配置如下所示：

屏幕全开

```python
#!/usr/bin/env python
import os,sys
os.system('autorandr --l 3s4')
```

看电影，只开主屏幕，其他屏幕关闭

```python
#!/usr/bin/python
import os,sys
os.system('autorandr close')
```

