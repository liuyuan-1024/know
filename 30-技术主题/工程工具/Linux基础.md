## 目录结构
Linux中万物皆文件, 甚至是硬件也会被映射为一个文件。

###  19 个顶级目录
1. **bin**：binary 二进制，存放常用系统级常用命令。
2. **sbin**：super user binary 超级用户二进制 ，存放超级管理员使用的系统管理命令。
3. **boot**：存放启动 Linux 内核时的核心文件，包括一些连接文件和镜像文件。
4. **dev**：device 设备，将所有硬件以文件形式映射在此目录中。
5. **<font style="background-color:#DF2A3F;">etc</font>**<font style="background-color:#DF2A3F;">：edit text config 可编辑文本配置，存放系统管理所需的配置文件。</font>
6. **<font style="background-color:#DF2A3F;">home</font>**<font style="background-color:#DF2A3F;">：存放普通用户的主目录</font>
7. **root**：存放超级管理员的主目录，不设置 root 用户时，第一个普通用户将作为小超级管理员，可以使用 sudo 来提升权限。
8. **lib**：存放 32 位共享库文件。
9. **lib64**：存放 64 位共享库文件。
10. **media**：将外部设备映射在此目录中。
11. **mnt**：mount 挂载点，临时挂载其他文件系统，可以将外部存储挂载到/mnt/上, 进入该目录就可以查看里面的内容，例如启动盘、 vmware tools的共享目录。
12. **opt**：optional 可选的，存放第三方软件或独立于系统发行版的软件。
13. **proc**：process information pseudp-filesystem 进程信息伪文件系统。运行中的每一个进程都会在此目录下创建一个/PID 的文件夹。
14. **run**：临时文件系统，用于存储系统启动以来的信息。它的内容在系统重启时会被删除或清除。且它的数据存储在内存中。
15. **srv**：存储由系统提供的服务所需的数据。这些数据通常是特定于站点的，并且由系统对外提供服务时所使用的。
16. **sys**：挂载为 sysfs 文件系统。
17. **tmp**：temp 临时，存放临时文件。
18. **<font style="background-color:#DF2A3F;">usr</font>**<font style="background-color:#DF2A3F;">：unix system resource，系统核心所在，包含了所有的共享文件</font>。
19. **var**：存放经常被修改的文件放在此目录下, 例如各种日志文件。

特殊的顶级目录

+ **selinux：**security-enhanced linux 安全子系统, Redhat/CentOS 特有的目录，能控制程序只能访问特定文件, 有三种工作模式可自行设置。

### 其他常用目录
+ /lost+found 这个目录一般情况下是空的, 当系统非法关机后, 这里就存放一些文件, 这个目录只能从终端中进入**"/" (根目录)** 使用 **ls** 显示所有文件才能看到.
+ /usr/local 另一个主机额外安装软件的目录, 一般是通过编译源码的方式安装的程序

## 系统级命令
系统级命令主要用于管理系统核心资源（如进程、用户、磁盘、网络、服务等），通常需要 root 权限执行，是维护系统正常运行的关键工具。

### 一、系统信息与状态
| 命令 | 说明 | 示例 |
| --- | --- | --- |
| `uname` | 查看内核版本、系统架构等信息 | `uname -a` 显示所有信息 |
| `hostname` | 查看/设置主机名 | `hostname; hostname newname` |
| `lsb_release` | 查看发行版信息 | `lsb_release -a` |
| `lscpu` | 查看 CPU 信息 | `lscpu` |
| `free` | 查看内存使用情况 | `free -h`（人类可读格式） |
| `df` | 查看磁盘分区情况 | `df -h`（人类可读格式） |
| `du` | 查看目录/文件占用的磁盘空间 | `du -sh /home`（查看 home 总大小） |
| `top` | 实时监控进程和系统资源 | `top` 按 q 退出；按 p 按照 CPU 排序 |
| `htop` | 增强版 top（需安装），支持鼠标操作 | `htop` |
| `vmstat` | 查看系统 | `vmstat 2`（每 2 秒刷新一次） |
| `iostat` | 查看磁盘 IO 性能统计 | `iostat -x 1`（每 1 秒显示详细 IO） |


### 二、进程管理
| 命令 | 说明 | 示例 |
| --- | --- | --- |
| `ps` | 查看当前系统进程快照 | `ps aux`（显示所有进程，包括其他用户） |
| `pstree` | 以树状结构显示进程关系 | `pstree -p`（显示进程 ID） |
| `pgrep` | 根据进程名查找进程 ID | `pgrep sshd`（查找 sshd 进程的 ID） |
| `kill` | 根据进程 ID 终止进程 | `kill -9 1234`（强制终止 ID 为 1234 的进程） |
| `pkill` | 根据进程名称终止进程 | `pkill -9 sshd`（强制终止所有 sshd 进程） |
| `nice` | 启动进程是设置优先级（-20 最高，19 最低，0 默认） | `nice -n 10 ./myprogram`（低优先级启动） |
| `renice` | 调整已运行进程的优先级 | `renice 5 1234`（将进程 1234 的优先级改为 5） |
| `nohup` | 是进程在终端关闭后继续运行 | `nohup ./myprogram &`（后台运行并输出到 nohup.out） |


### 三、用户与权限
| 命令 | 说明 | 示例 |
| --- | --- | --- |
| `useradd` | 创建新用户 | `useradd -m username`（-m 自动创建家目录） |
| `userdel` | 删除用户 | `userdel -r username`（-r 同时删除家目录） |
| `passwd` | 修改用户密码 | `passwd username`（管理员修改普通用户密码） |
| `groupadd` | 创建用户组 | `groupadd dev`（创建 dev 组） |
| `groupdel` | 删除用户组 | `groupdel dev` |
| `usermod` | 修改用户属性（加入组、修改家目录） | `usermod -aG sudo username`（添加到 sudo 组） |
| `chmod` | 修改文件权限（读 r=4，写 w=2，执行 x=1）（rwx=7，rw=6，rx=5） | `chmod 755 file.sh`（所有者 rwx，其他 rx） |
| `chown` | 修改文件的所有者和所属组 | `chown user:group file`（改为 user 所有，group 组） |
| `sudo` | 以 root 权限执行命令（需要配置 sudoers） | `sudo apt update` |


### 四、网络配置
| 命令 | 说明 | 示例 |
| --- | --- | --- |
| `ip` | 现代网络配置工具（替代 `ifconfig`<br/>），管理接口、路由等 | `ip addr`<br/>（查看所有接口 IP）；`ip link set eth0 up`<br/>（启动 eth0） |
| `ifconfig` | 旧版网络接口配置工具（部分系统需安装 net-tools） | `ifconfig eth0`<br/>（查看 eth0 信息） |
| `netstat` | 查看网络连接、端口监听等（替代工具：`ss`<br/>） | `netstat -tulpn`<br/>（显示监听的 TCP/UDP 端口及对应进程） |
| `ss` | 更高效的网络连接查看工具（替代 `netstat`<br/>） | `ss -tuln`<br/>（显示监听的 TCP/UDP 端口） |
| `pin` | 测试与目标主机的连通性 | `ping -c 4 baidu.com`<br/>（发送 4 个 ICMP 包） |
| `traceroute` | 跟踪数据包到目标主机的路由路径 | `traceroute baidu.com` |
| `firewall-cmd` | 防火墙管理（CentOS/RHEL 等，基于 firewalld） | `firewall-cmd --add-port=80/tcp --permanent`<br/>（永久开放 80 端口） |
| `ufw` | 简易防火墙管理（Ubuntu 等） | `ufw allow 22/tcp`<br/>（允许 SSH 端口连接） |


### 五、系统服务管理
| 命令 | 说明 | 示例 |
| --- | --- | --- |
| `systemctl` | 主流服务管理工具（systemd 系统） | `systemctl start sshd`<br/>`systemctl restart sshd`<br/>`systemctl stop sshd`<br/>`systemctl enable sshd`（开机自启动）<br/>`systemctl disable sshd`（关闭开机自启动）<br/>`systemctl status sshd`（查看 sshd 服务状态）<br/>`systemctl daemon-reload`（重载 systemd 配置文件） |


### 六、日志系统
| 命令 | 说明 | 示例 |
| --- | --- | --- |
| `dmesg` | 查看内核启动及硬件相关日志 | `dmesg | grep error`（查找错误信息） |
| `journalctl` | 查看系统日志 | `journalctl -u sshd`（查看 sshd 服务日志）<br/>`joutnalctl -f`（实时跟踪日志） |
| `tail` | 查看文件尾部内容（常用于日志实时监控） | `tail -f /var/log/syslog`（实时监控系统日志） |


### 七、系统控制
| 命令 | 说明 | 示例 |
| --- | --- | --- |
| `sync` |  将内存数据同步到磁盘 | `sync`<br/>下面的系统控制命令均会执行先一步执行 `sync` |
| `shutdown` | 关机或重启（支持定时） | `shutdown -h now`（立即关机）<br/>`shutdown -r +10`（十分钟后重启） |
| `reboot` | 重启系统 | `reboot`（立即重启） |
| `halt` | 停止系统（类似关机） | `halt` |
| `poweroff` | 关机并切断电源 | `poweroff` |


## 包管理器（apt）
### 镜像源
```bash
# 先备份系统默认的源列表，避免配置出错后无法恢复。
sudo cp /etc/apt/sources.list /etc/apt/sources.list.bak
# 用文本编辑器（如 nano）打开源列表文件
sudo nano /etc/apt/sources.list
# 删除原有内容，粘贴以下中科大镜像源内容（覆盖所有组件），粘贴后按Ctrl+O保存，再按Ctrl+X退出编辑器
deb https://mirrors.ustc.edu.cn/debian/ trixie main contrib non-free non-free-firmware
deb-src https://mirrors.ustc.edu.cn/debian/ trixie main contrib non-free non-free-firmware

deb https://mirrors.ustc.edu.cn/debian/ trixie-updates main contrib non-free non-free-firmware
deb-src https://mirrors.ustc.edu.cn/debian/ trixie-updates main contrib non-free non-free-firmware

deb https://mirrors.ustc.edu.cn/debian/ trixie-backports main contrib non-free non-free-firmware
deb-src https://mirrors.ustc.edu.cn/debian/ trixie-backports main contrib non-free non-free-firmware

deb https://mirrors.ustc.edu.cn/debian-security/ trixie-security main contrib non-free non-free-firmware
deb-src https://mirrors.ustc.edu.cn/debian-security/ trixie-security main contrib non-free non-free-firmware
# 配置完成后，执行以下命令更新缓存，让系统识别新镜像源
sudo apt update
# 若配置后出现软件更新错误，可执行下面的命令恢复原配置
sudo cp /etc/apt/sources.list.bak /etc/apt/sources.list
```

### 一、软件源更新与升级系统包
| 命令 | 说明 | 示例 |
| --- | --- | --- |
| `apt update` | 更新软件源列表（获取最新的包版本信息，但不升级包） | `apt update` |
| `apt upgrade` | 升级所有可更新的包（保留配置文件，不删除旧包） | `apt upgrade` |
| `apt full-upgrade` | 升级包并智能处理依赖/版本冲突（可能会删除旧包） | `apt full-upgrade` |


### 二、软件包安装
| 命令 | 说明 | 示例 |
| --- | --- | --- |
| `apt install` | **安装一个或多个软件包** | `apt install nginx` |
| `apt install -y` | **自动确认安装** | `apt install -y curl` |
| `apt install <包名>=<版本号>` | **安装指定版本的包**（需先确认仓库中有该版本） | `apt install docker-ce=24.0.6-1~debian.12` |


### 三、软件包查询与显示详细信息
| 命令 | 说明 | 示例 |
| --- | --- | --- |
| `apt search` | **搜索软件包**（支持关键词匹配） | `apt search python3`（搜索 Python3 相关包） |
| `apt show` | **查看包的详细信息** | `apt show nginx` |
| `apt list` | **列出包的状态** | `apt list --installed`（列出已安装的包）<br/>`apt list --upgradable`（列出可升级的包） |


### 四、软件包删除与清理
| 命令 | 说明 | 示例 |
| --- | --- | --- |
| `apt remove` | **删除软件包**（保留配置文件） | `sudo apt remove nginx` |
| `apt purge` | **彻底删除软件包及配置文件** | `sudo apt purge nginx` |
| `apt autoremove` | **删除无用的依赖包**（自动清理安装软件时的临时依赖） | `sudo apt autoremove` |
| `apt clean` | **清理本地缓存的包文件**（释放磁盘空间） | `sudo apt clean` |
| `apt autoclean` | **清理过时的缓存包文件**（仅删除已无更新的旧包缓存） | `sudo apt autoclean` |


### 五、其他实用命令
| 命令 | 说明 | 示例 |
| --- | --- | --- |
| `apt policy` | **查看包的版本策略**（显示仓库中可用的版本及优先级） | `apt policy docker-ce` |
| `apt --fix-broken install` | **修复破损的依赖关系**（解决包安装时的依赖错误） | `sudo apt --fix-broken install` |


## 7zip
```bash
7z [命令] [额外选项] <压缩包路径> [文件路径]
```

| 命令 | 说明 | 示例 |
| --- | --- | --- |
| `7z a source.7z source.txt` | a：压缩，将指定文件（可以多个文件）压缩为 一个.7z 文件 | `7z a source.7z source.txt`<br/>`7z a source.7z source/` |
| `7z x source.7z` | x：解压（保留原目录结构） | `7z x source.7z`<br/>`7z x source.zip` |
| `7z e source.7z` | e：解压（不保留原目录结构，所有文件解压到当前 7 目录） | `7z e source.7z`<br/>`7z e source.zip` |
| `7z l source.7z` | l：列出压缩包中所有文件 | `7z l source.7z` |


| 额外选项 | 说明 | 示例 |
| --- | --- | --- |
| `-t其他压缩包格式` | 压缩时指定压缩包格式 | `7z a -zip source.zip file.txt` |
| `-p密码` | 压缩时设置密码，对内容进行加密；<br/>解压时需要输入密码 | `7z x -p密码 source.7z` |
| `-o解压输出目录` | 解压时指定解压后的文件的存放位置 | `7z x source.7z -o./out` |


## Linux远程登录
远程登录---Xshell

文件管理---Xftp

## Vi和Vim
Linux系统会内置vi文本编辑器

vim具有程序编辑的能力, 可以看作是Vi的增强版本, 可以主动的以字体颜色辨别语法的正确性, 方便程序设计. 代码补全, 编译以及错误跳转等方便编程的功能特别丰富.

### Vi和Vim常用的三种模式
+ 正常模式: 以vim打开一个文件, 默认就是正常模式, 在这个模式下可以使用方向键, 删除字符, 删除整行, 复制, 粘贴
+ 插入模式: 按下 i, I, o, O, a, A, r, R 中任意一个按钮, 就会进入插入模式
+ 命令行模式: 输入 **冒号: ** 即可,  提供相关指令, 如:读取, 存盘, 替换, 离开vim, 显示行号等动作

### Vi和Vim各个模式的切换
<!-- 这是一张图片，ocr 内容为：在命令行下 #VIM XXX 一般模式/正常 或者/ ESC 或者A ESC 命令模式 编辑模式 :WQ(保存退出):Q(退出) 在命令行下 :Q!(强制退出,不保存) -->
![](https://cdn.nlark.com/yuque/0/2024/png/43050341/1710504289031-72cb0a25-4001-466a-a4cd-1cbaaa83f06e.png)

### Vi和Vim的快捷键
+ 一般模式下 
    1. **nyy**: 拷贝当前行及以下的共n行, 不写n, 默认为1, 输入 **p** 粘贴
    2. **ndd**: 删除当前行及以下的共n行
    3. **G**: 定位到文档的末尾
    4. **gg**: 定位到文档的开头
    5. **u**: 撤销
    6. **n shift + g**: 快速定位到第n行
+ 命令行模式下( : 进入命令行模式) 
    1. **/关键字**: 输入快捷键并回车, 系统会查找关系字, 输入 **n** 就是查找下一个
    2. **:set nu**: 显示文件行号
    3. **:set nonu**: 取消显示文件行号



