# Bash 快捷键

1. 移动光标快捷键
       ctrl+f 向前移动一个字符
       ctrl+b 向后移动一个字符
       ctrl+a 移动到当前行首home
       ctrl+e 移动到当前行尾end
       ctrl+l 清屏，并在屏幕最上面开始一个新行
       alt+f 向前移动一个单词
       alt+b 向后移动一个单词

2. 编辑命令行快捷键
       ctrl+d 删除当前的字符
       ctrl+t 交换当前字符和前一个字符的位置
       alt+t 交换当前单词和前一个单词的位置
       alt+u 把当前单词变成大写
       alt+l 把当前单词变成小写
       alt+c 把当前单词变成首字母大写的单词
       ctrl+v 添加一个特殊字符，例如，要添加一个制表符，按ctrl+v+tab

3. 剪切、粘贴快捷键
       alt a：将光标移到当前单词头部 
       alt e：将光标移到当前单词尾部
       alt d：删除从光标到当前单词结尾的部分
       alt+y 回退到先前剪切的文本并粘贴它

  ctrl+k 剪切文本直到行的末尾
  ctrl+u 剪切文本直到行的起始，删除光标前面所有字符相当于VIM里d shift+^
  ctrl+w 剪切光标前的单词，删除最后输入的单词
  ctrl + y 粘贴刚才所删除的字符
  ctrl+c 删除整行，终端进程
  ctrl + a 切换到命令行开始　　这个操作跟Home实现的结果一样的，但Home在某些unix环境下无法使用
  ctrl + e 切换到命令行末尾
  ctrl + r 在历史命令中查找 　有时history比较多时，想找一个比较复杂的，直接在这里，shell会自动查找并调用;将自动在命令历史缓存中增量搜索后面入的字符。
  ctrl + R - Search the history backwards with multi occurrence   
  ctrl + d 退出shell，logout作用是 EOF 即文件末尾(End-of-file)。如果你的光标处在一个空白的命令行上，将会退出bash，比你用exit命令退出要快得多。
  ctrl + z 转入后台运行　　转入后台运行的进程在当前用户退出后就会终止，不如用nohup命令&
  ctrl + xx - Move between EOL and current cursor position   
  ctrl + x @ - Show possible hostname completions    
  ctrl + z - Suspend/ Stop the command  
  ctrl + h - 删除当前字符   
  !$ 显示系统最近的一条参数 先用cat /etc/sysconfig/network-scripts/ifconfig-eth0，可用于vim编辑。一般做法：先↑ 显示最后一条命令，Home移到最前，删除cat，输入vim。其实可用vim !$代替
  ctrl+Page Up 上一个标签
  ctrl+Page Down 下一个标签
  Alt+N 切换到第N个标签（N为数字）


  窗口：这个快捷键总表可以通过长按super键
  Alt + F1 类似Windows下的Win键，在GNOME中打开"应用程序"菜单(Applications)
  Alt+F2    弹出命令行窗口，类似Windows下的Win + R组合键，在GNOME中运行应用程序
  Alt+F4    关闭当前窗口。
  Alt + F5 取消最大化窗口 (恢复窗口原来的大小)
  Alt + F7 移动窗口 (注: 在窗口最大化的状态下无效)
  Alt + F5 取消最大化窗口 (恢复窗口原来的大小)
  Alt + F7 移动窗口 (注: 在窗口最大化的状态下无效)
  Alt + F8 改变窗口大小 (注: 在窗口最大化的状态下无效)
  Alt + F9 最小化窗口
  Alt + F10 最大化窗口
  Alt + Space 打开窗口的控制菜单 (点击窗口左上角图标出现的菜单）
  Alt+Tab    在窗口之间快速切换。按住Shift可反向排序。
  Alt+`    在同一个应用程序的不同窗口或Alt+Tab后选中的程序间切换;此快捷键使用美国键盘上的`，Tab上方。

  Shift+Alt+↑    激活“Expo”模式。显示当前工作区的所有窗口。
  Ctrl+Alt+方向键    在工作区之间切换。
  Ctrl+Alt+Shift+方向键    将当前窗口移至其他工作区。
  Ctrl+Alt+Delete    注销。
  Ctrl+Alt+L    锁定屏幕。
  ctrl + Alt + Shift + → / ← 移动当前窗口到不同工作台
  ctrl+Alt+Shift+Fn 终端N或模拟终端N(n和N为数字1－6)
  ctrl+Alt+Shift+F7 返回桌面
  ctrl+Alt+Shift+F8 未知（终端或模拟终端）
  Ctrl+Super+D    隐藏所有窗口并显示桌面。再次按下按钮可以恢复窗口。
  Super+S    激活工作区切换器。缩小所有工作区。
  Super+W    激活“Expo”模式。显示当前工作区的所有窗口。

  Shift+Print Screen    获取屏幕上某个区域的截图。光标变为十字。点击并拖动选择区域。

  


  常用
  win+n切换背景颜色风格
  win+tab若开3D效果了切换
  ctrl+alt+backspace=相当于强制注销
  ctrl+alt+del=调出关机菜单
  ctrl+alt+l=锁定桌面
  ctrl+alt+d=最小化gnome所有窗口
  ctrl+alt+f2=linux终端用户（alt + f7返回xwindows，alt+ <- 或-> 进行终端切换）
  ctrl+alt+ <- 或-> =切换桌面


  默认特殊快捷键
  展示所有窗口程序 = F10
  展示当前窗口最上层程序 = F11
  展示当前窗口所有程序 = F12
  旋转3D桌面 = ctrl + Alt + 左/右箭头（也可以把鼠标放在标题栏或桌面使用滚轮切换）
  旋转3D桌面（活动窗口跟随） = ctrl + Shift + Alt + 左/右箭头
  手动旋转3D桌面 = ctrl + Alt + 左键单击并拖拽桌面空白处
  窗口透明/不透明 = possible with the “transset” utility or Alt + 滚轮
  放大一次 = 超级键 + 右击
  手动放大 = 超级键 + 滚轮向上
  手动缩小 = 超级键 + 滚轮向下
  移动窗口 = Alt + 左键单击
  移动窗口时贴住边框 = 左键开始拖动后再 ctrl + Alt
  调整窗口大小 = Alt + 中击
  Bring up the window below the top window = Alt + middle-click
  动态效果减速 = Shift + F10
  水纹 = 按住 ctrl+超级键
  雨点 = Shift-F9
  桌面展开＝ ctrl + Alt + 下箭头，然后按住 ctrl + Alt 和左/右箭头选择桌面Bash Shell 快捷键

  


  Linux下快捷键：在控制台/虚拟终端下

     1. Ctrl-Alt-Delete -关闭计算机
     2. Alt-Fn (F1, F2, F3,…) - 切换到第n个控制台
     3. Alt-Left 或者 Alt-Right - 切换到上/下一个虚拟终端
     4. Scroll Lock - 锁定终端的输入/输出－当屏幕输出滚动过快的时候可以用这个键给屏幕定格，再按一次Scroll Lock解除锁定。也可以用另外一种方法实现这个功能，使用Ctrl-S 锁定屏幕，使用Ctrl-Q解除锁定。如果你的控制台突然出现了不明原因无响应也可以尝试一下后面的这个解锁快捷键，也许是因为你无意中触发了CTRL-S导致屏幕假死。
     5. Shift-Page Up 或者 Shift-Page Down - 上、下滚动控制台缓存。这个功能在 Scroll Lock 启动的时候也是管用的。 在使用 (Alt-Fn) 更换控制台后缓存内容就被删除了，所以滚动无效。

  Kernel shortcuts
  下面的快捷键必须在内核中启用以后才可以使用,而且必须启用魔术组合键(SysRQ)：
  启用SysRq：$sudo echo 1 > /proc/sys/kernel/sysrq
  禁用SysRq：$sudo echo 0 > /proc/sys/kernel/sysrq

     1. Alt-SysRQ-S - 同步所有已挂载的文件系统。所有缓存中的数据将被立刻写入磁盘。
     2. Alt-SysRQ-U - 以只读方式重新挂载所有已挂载文件系统。
     3. Alt-SysRQ-B - 快速重起。不要在没有同步和卸载文件系统下执行，否则会导致文件系统严重错误。
     4. Alt-SysRQ-S，然后 Alt-SysRQ-U，然后 Alt-SysRQ-B - 同步所有文件系统、以只读方式重新挂载所有文件系统、立刻重新启动。这是重新启动Linux的最快方式。
     5. Alt-SysRQ-H - 输出其他魔术组合键列表(SysRQ)功能。


  X-Window快捷键

     1. Ctrl-Alt-Plus 或者 Ctrl-Alt-Minus- 改变屏幕分辨率(提高/降低)。前提是在X-Window server配置文件中写入了多种可用分辨率。
     2. Ctrl-Alt-Backspace - 杀死X-server，返回登录界面。所有正在运行的应用程序将被终止。
     3. Ctrl-Alt-Escape - xkill - 点击一个应用程序强制关闭。
     4. Ctrl-Shift-Num Lock 把键盘上的小键盘(数字键盘)变成鼠标，启动后你可以用小键盘进行鼠标操作。数字键盘上的/和 * 分别代表鼠标左键和鼠标右键，数字键盘上的 5是双击。(这个地方翻译的不确定，大家可以参考一下原文。我用的笔记本没办法测试这个功能。)
     5. Ctrl-Alt-Fn (F1, F2, F3,…) - 切换到第 n 个文本控制台。一般Alt+F7切换回X-window。
     6. Alt-F2 - 开启一个运行命令的小窗口。如果输入的是可执行命令则直接运行；如果输入的是文件名系统将调用适当的应用程序打开它；输入网址将用默认的浏览器打开它。

  KDE 快捷键

     1. Ctrl-Alt-Shift-Page Down - 直接关机
     2. Ctrl-Alt-Shift-Page Up - 直接重启

  Windows过来的初学者常遇到的问题,在Vi里写完东西,习惯性Ctrl+S保存,然后就死在那里了,完全没有反映,只好重启,高级点的用Alt+F2/3/4切换到另外的控制台干别的事情

  其实应该用Ctrl+Q来接触锁定,Ctrl+S在Linux下是锁定屏幕显示的意思和ScreenLock键是一个效果,不信你试试按下ScrLk或者Fn+ScrLk
  --------------------- 

