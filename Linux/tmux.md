# tmux

解决tmux没有反应的情况：

[(10条消息) linux无法正常启动tmux的解决方法_ZHAGNQ的博客-CSDN博客](https://blog.csdn.net/ZHAGNQ/article/details/89350661)



```
tmux ls
tmux at -t 0
tmux new -s mysession # 创建时建名
```

![截图](..\images\43561456534.png)

## 配置TMUX支持鼠标切换pane，鼠标调节pane大小等

```
tmux conf:** https://blog.csdn.net/fk1174/article/details/79220227

1,在~/.tmux.conf中加入：（2版本）
setw -g mouse-resize-pane on
setw -g mouse-select-pane on
setw -g mouse-select-window on
setw -g mode-mouse on

1,在~/.tmux.conf中加入：（3版本）
# Enable mouse mode (tmux 2.1 and above)
set -g mouse on
#Enable oh my zsh in tmux
set -g default-command /bin/bash
# Mouse based copy
bind-key -T copy-mode-vi MouseDragEnd1Pane send -X copy-pipe-and-cancel "reattach-to-user-namespace pbcopy"
bind-key -T copy-mode MouseDragEnd1Pane send -X copy-pipe-and-cancel "reattach-to-user-namespace pbcopy" 

2，然后在tmux里面按 Ctrl+b，:（也就是ctrl+b下摁冒号，进入命令输入模式）

3，输入source ~/.tmux.conf回车


```

## tmux 命令：

在 Tmux 中，按下 Tmux 前缀 ctrl+b，然后：

> %  垂直分割 
>
> "水平分割 
>
> o  交换窗格 
>
> x  关闭窗格
>
>  z 切换窗格最大化/最小化              

| Ctrl+b | 激活控制台；此时以下按键生效 |
| ------ | ---------------------------- |

### 系统操作

| ?    | 列出所有快捷键；按q返回                                      |
| :--- | :----------------------------------------------------------- |
| d    | 脱离当前会话；这样可以暂时返回Shell界面，输入tmux attach能够重新进入之前的会话 |
| D    | 选择要脱离的会话；在同时开启了多个会话时使用                 |
| z    | 放大、缩小当前会话                                           |
| r    | 强制重绘未脱离的会话                                         |
| s    | 选择并切换会话；在同时开启了多个会话时使用                   |
| :    | 进入命令行模式；此时可以输入支持的命令，例如kill-server可以关闭服务器 |
| [    | 进入复制模式；此时的操作与vi/emacs相同，按q/Esc退出          |
| ~    | 列出提示信息缓存；其中包含了之前tmux返回的各种提示信息       |

### 窗口操作

| c      | 创建新窗口                           |
| ------ | :----------------------------------- |
| &      | 关闭当前窗口                         |
| 数字键 | 切换至指定窗口                       |
| p      | 切换至上一窗口                       |
| n      | 切换至下一窗口                       |
| l      | 在前后两个窗口间互相切换             |
| w      | 通过窗口列表切换窗口                 |
| ,      | 重命名当前窗口；这样便于识别         |
| .      | 修改当前窗口编号；相当于窗口重新排序 |
| f      | 在所有窗口中查找指定文本             |

### 面板操作

| ”           | 将当前面板平分为上下两块                                     |
| ----------- | ------------------------------------------------------------ |
| %           | 将当前面板平分为左右两块                                     |
| x           | 关闭当前面板                                                 |
| !           | 将当前面板置于新窗口；即新建一个窗口，其中仅包含当前面板     |
| Ctrl+方向键 | 以1个单元格为单位移动边缘以调整当前面板大小                  |
| Alt+方向键  | 以5个单元格为单位移动边缘以调整当前面板大小                  |
| Space       | 在预置的面板布局中循环切换；依次包括even-horizontal、even-vertical、main-horizontal、main-vertical、tiled |
| q           | 显示面板编号                                                 |
| o           | 在当前窗口中选择下一面板                                     |
| 方向键      | 移动光标以选择面板                                           |
| {           | 向前置换当前面板                                             |
| }           | 向后置换当前面板                                             |
| Alt+o       | 逆时针旋转当前窗口的面板                                     |
| Ctrl+o      | 顺时针旋转当前窗口的面板                                     |