---
title: Robocom世界机器人开发者大赛参赛总结
date: 2020-11-24 13:08:20
tags: 个人经历
---

**一些个人的总结**

<!--more-->

## 一、 机器人分析

### 1.机器人介绍

<img src="Robocom世界机器人开发者大赛参赛总结\spark机器人.jpg" alt="spark机器人" style="zoom:67%;" />

本次夺宝奇兵竞技赛采用的是spark机器人，搭配uArm Swift机械臂，机器人内部是一个嵌入式的Linux系统，更准确的说是一个Ubuntu系统(Linux发行版)

> - Linux 运行稳定、对网络的良好支持性、低成本，且可以根据需要进行软件裁剪，内核最小可以达到几百 KB 等特点，使其近些年来在嵌入式领域的应用得到非常大的提高。

### 2.与机器人建立连接

#### a)	建立与机器人的通讯

**方法1：**机器人开启个人热点，电脑连接热点从而建立通讯。

**方法2：**机器人与电脑同时连接同一局域网(WiFi)从而建立通讯。

#### b)	对机器人系统的操控

​	**方法1：**与其他的Linux服务器类似，用SSH客户端连接机器人。

> - **SSH 客户端**是一种使用 `Secure Shell（SSH）` 协议连接到远程计算机的软件程序 
> - Windows 下 SSH 客户端：(Linux与mac自带SSH客户端(即终端)，不需要第三方软件)
> - `Putty` http://www.chiark.greenend.org.uk/~sgtatham/putty/latest.html
> - `XShell` http://xshellcn.com

具体连接：``ssh [-p port] user@remote``

​	user是spark，即用户名；remote即ip地址。

缺点：只能通过终端查看机器人的Linux系统信息。

​	**\*方法2：**采用远程连接软件，如：**VNC Viewer**，Tight VNC

可以直观的看到机器人内部的Ubuntu，所以第一种办法可以忽略了

## 二、  初始spark代码分析

### 1. 知识点

#### a)	ROS基础入门知识点

​	1.从turtlesim(小海龟仿真结点)理解ros的基本通信机制：topic(异步通信机制)	service(同步通信机制)

​	2.异步通信机制(类似UDP)：发布者(Publisher)发消息，订阅者(Subscriber)接受消息(msg)。

​	3.同步通信机制(类似TCP)：服务端(Service)发布服务需求，客户端(Client)接受需求做出反应并给出回馈((respond))。

​	4.Ros master辅助不同结点的连接。(注意结点名不能相同，否则需要重映射)

​	5.Ros launch(xml语法)(类似于脚本.sh)，可用于启动其他所有结点并建立连接，实现多结点的配置与启动。

​	6.Ros TF可以管理所有坐标系(Tranfor变换)，记录十秒钟之内机器人所有坐标系的位置关系。实现机制：广播与监听。	(坐标平移，四元数，弧度，角度)

#### b)	ROS功能包Package内容

​	1.src—源代码，build和devel—可执行文件，install—对外呈现的接口。

​	2.CMakeLists.txt——功能包内的编译规则(CMake语法)

```cmake
如：add_executable(velocity_publisher src/velocity_publisher.cpp)

 # 把后者编译成前者这样的可执行文件

target_link_libraries(velocity_publisher ${catkin_LIBRARIES}) 

# 把可执行文件与ros的库连接起来
```

​	3.Package.xml——功能包的基本信息(XML语法)：

​           如：description(作用)，maintainer(维护者)，build(依赖信息)

​	4.msg后缀——自定义话题，srv后缀——自定义服务。

​	5.Qt工具箱提供很多可视化工具

​	如：rqt_graph(计算图可视化工具)：清晰的查看各节点的关系。

​			rqt_console(日志输出工具)：查看日志。

​	6.Rviz(robot的一个可视化平台，三维的可视化工具)

​	7.Gazebo(三维物理仿真平台)

#### c)    OpenCv基础

​	1.捕获图像：VideoCapture(0)，0是默认读取电脑摄像头的图像。

​	2.图像颜色识别：cvtColor(frame,cv2.BGR2HSV)把RGB转换为HSV。

​	这里直接看初始的grasp解析吧，其实就是OpenCv的各种接口：

```python
    #获取物块的信息并返回其hsv区间(颜色矩阵,最低与最高)
    def hsv_value(self):
        try:
            filename = os.environ['HOME'] + "/color_block_HSV.txt"#获取主目录这个文件的信息
            with open(filename, "r") as f:#python的with语句,以读的形式打开文件并赋值给f
                for line in f:
                    split = line.split(':')[1].split(' ')
                    lower = split[0].split(',')
                    upper = split[1].split(',')
                    for i in range(3):
                        lower[i] = int(lower[i])
                        upper[i] = int(upper[i])

            lower = np.array(lower)
            upper = np.array(upper)#numpy.array将数据转换为矩阵
        except:
            raise IOError('could not find hsv_value file : {},please execute #13 command automatically '
                          'generate this file'.format(filename))

        return lower, upper

    # 使用CV检测物体       
    def image_cb(self):
        global xc, yc, xc_prev, yc_prev, found_count, detect_color_blue	
        capture = cv2.VideoCapture(0)#捕获视频(通过内置摄像头)
        LowerBlue, UpperBlue = self.hsv_value()#通过自己编写的hsv_value()函数返回最低的hsv与最高的hsv

        while True:
            # time.sleep(0.08)
            ret,frame = capture.read()#读取图像(抓住某一帧)赋值给frame变量
            cv_image2 = cv2.cvtColor(frame, cv2.COLOR_BGR2HSV)#将rgb图像转为灰度图像
            mask = cv2.inRange(cv_image2, LowerBlue, UpperBlue)#对灰度图进行阈值计算，将低于LowerBlue，高于UpperBlue的都变为0
            mask = cv2.erode(mask, None, iterations=5)#收缩图像，增加画面中的白色区域，简单来说就是让白色能连城一整块，而不是黑白相间的方块，iterations是腐蚀次数
            #mask = cv2.dilate(mask, None, iterations=2)#腐蚀边界，让黑白分明，显然我们不需要，我们只需要把整个物块都白出来即可
            mask = cv2.GaussianBlur(mask, (9,9), 0)#(高斯模糊,图像大小,高斯内核大小)
            # detect contour
            cv2.imshow("win2", mask)#win2就是我们看到的黑白图
            cv2.waitKey(1)#等1ms的键盘输入，无输入的话就返回-1,有输入就返回输入的ascii码值，参数如果为0表示等无限长的时间
            if detect_color_blue :#全局变量,只有这个变量的值为真才会查找物块轮廓,而这个值在抓取函数grasp_cp中收到抓取命令时才会赋值为True
                _, contours, hier = cv2.findContours(mask, cv2.RETR_TREE, cv2.CHAIN_APPROX_SIMPLE)#这里的返回的contours是一个list,每个元素代表一个轮廓,用用numpy中的ndarray表示
                if len(contours) > 0:#如果检测到了轮廓
                    #cv2.drawContours(frame,contours,-1,(0,0,255),4)#在原图上画出轮廓
                    size = []#面积
                    size_max = 0
                    distance_list = []#距离列表
                    max_distance = 450#最远距离
                    for i, c in enumerate(contours):#enumerate是同时获取索引(i)和值(c)的函数(contours是一个数组，每个数组是一个方块的信息)
                        rect = cv2.minAreaRect(c)#返回一个2D结构，包含矩形中心点的坐标（x，y），矩形的宽和高（w，h），以及旋转角度
                        box = cv2.boxPoints(rect)#返回一个旋转矩阵的四个顶点,与cv2.minAreaRect()函数是配套用的
                        box = np.int0(box)#numpy.int64
                        x_mid, y_mid = rect[0]#获取中心点坐标

                        w = math.sqrt((box[0][0] - box[1][0])**2 + (box[0][1] - box[1][1])**2)#宽度,(y1-y0)平方+(x1-x0)平方,左-右
                        h = math.sqrt((box[0][0] - box[3][0])**2 + (box[0][1] - box[3][1])**2)#高度,上-下

                        size.append(w * h)#面积赋值
                        # 所有点到spark的距离
                        distance_list.append(math.sqrt((320 - x_mid) ** 2 + (300 - y_mid) ** 2))
					  #(320,300)是机械臂的坐标系初始位置位置,这里是自定义的,也就是说当摄像头位置变换导致机械臂初始位置发生改变时,应替换为那个新的初始坐标

                        if size[i] > size_max and distance_list[i] < max_distance:#面积大于0，距离小于450,维护size_max为最大的面积
                            size_max = size[i]
                            min_distance = distance_list[i]
                            index = i
                            xc = x_mid
                            yc = y_mid
                            cv2.circle(frame, (np.int32(xc), np.int32(yc)), 2, (255, 0, 0), 2, 8, 0)#画圆形函数img,center,radius,color,thickness
                    if found_count >= 15 and min_distance < 350:#至少找到十五次且抓的时候距离要小于350，这里用于自动赛的自动抓取
                        self.is_found_object = True#表示找到了,这里是判断是否要转圈的关键
                        cmd_vel = Twist()
                        self.cmd_vel_pub.publish(cmd_vel)#这里发布Twist是给全自动代码的
                    else:
                        # if box is not moving,更新抓取点,若这个点变动过大会引起found_count清零
                        if abs(xc - xc_prev) <= 50 and abs(yc - yc_prev) <= 50 and yc > 150 and yc < 370 and xc > 100 and xc < 540:
                            found_count = found_count + 1
                        else:
                            found_count = 0
                
                else:
                    found_count = 0
            xc_prev = xc
            yc_prev = yc
            cv2.imshow("win1", frame)#这里显示的是原图
            cv2.waitKey(1)
            #key = cv2.waitKey(10)
```



#### d)    Python基础

​			sys.stdin.fileno() 获取标准输入文件的编号。

​			sys.stdin.read(1) 读取键盘字符(这里是每次读取一个字符)

​	2.终端输入属性控制（tty库）：

​			tty.setraw()作用是取消输入终端的回显和缓冲。(输入的不会出现在屏幕上；输出也不用等回车键，入一个出一个)

​	3.终端属性获取：termios.tcgetattr()  获取(输入的)终端属性		termios.tcsetattr     设置终端属性

​	4.finally:   无论try是否抛出异常，finally的代码都会执行，常用于清理工作。

​	5.全局变量引用：函数想要使用全局变量时需在开头global该变量

​	6.self类似于C++的this指针

​	7.控制单次循环时间的方法：

​			①   rospy.sleep(1)进入死循环1s

​			②   rate=rospy.Rate(10)，设置频率为10HZ，即0.1s，再rate.sleep()

#### e)    Linux基础

​	Unix一开始存在的意义：为了让同一台主机给更多的用户使用。

​	延伸出C语言的来源： 美国贝尔实验室的 **Ken Thompson**用B语言写出了第一个Unix，但**BCPL** 语言的移植性太差，所以**Dennis M.Ritchie**为此设计了C语言。

<img src="Robocom世界机器人开发者大赛参赛总结\计算机组成.png" style="zoom:72%;" />

**Linux内核**：即终端命令与系统调用的封装。

**Linux发行版**：如Ubuntu，更多的是图形化的界面以及各种应用程序的提供。

**Windows**和**Linux**的差别：

**设计初衷差异：**

Windows更多的偏向单用户操作系统，即个人电脑。(当然现在也可以提供多用户操作了)

Linux是多用户操作系统，易开发，因此更偏向于服务器与嵌入式的应用。

**文件系统差异：**

Windows采用驱动器盘符，每个驱动器都有自己的根目录结构，这样形成了多个树并列的情形。

Linux没有盘符这个概念，只有一个根目录 `/`，所有文件都在它下面。

<img src="Robocom世界机器人开发者大赛参赛总结\Linux的树形示意图.png" style="zoom:72%;" />

> **Linux主要目录速查表**
>
> - /：根目录，
>
>   一般根目录下只存放目录
>
>   ，在 linux 下有且只有一个根目录，所有的东西都是从这里开始
>
>   - 当在终端里输入 `/home`，其实是在告诉电脑，先从 `/`（根目录）开始，再进入到 `home` 目录
>
> - /bin、/usr/bin：可执行二进制文件的目录，如常用的命令 ls、tar、mv、cat 等
>
> - /boot：放置 linux 系统启动时用到的一些文件，如 linux 的内核文件：`/boot/vmlinuz`，系统引导管理器：`/boot/grub`
>
> - /dev：存放linux系统下的设备文件，访问该目录下某个文件，相当于访问某个设备，常用的是挂载光驱`mount /dev/cdrom /mnt`
>
> - /etc：系统配置文件存放的目录，不建议在此目录下存放可执行文件，重要的配置文件有
>
>   - /etc/inittab
>   - /etc/fstab
>   - /etc/init.d
>   - /etc/X11
>   - /etc/sysconfig
>   - /etc/xinetd.d
>
> - /home：系统默认的用户家目录，新增用户账号时，用户的家目录都存放在此目录下
>
>   - `~` 表示当前用户的家目录
>   - `~edu` 表示用户 `edu` 的家目录
>
> - /lib、/usr/lib、/usr/local/lib：系统使用的函数库的目录，程序在执行过程中，需要调用一些额外的参数时需要函数库的协助
>
> - /lost+fount：系统异常产生错误时，会将一些遗失的片段放置于此目录下
>
> - /mnt: /media：光盘默认挂载点，通常光盘挂载于 /mnt/cdrom 下，也不一定，可以选择任意位置进行挂载
>
> - /opt：给主机额外安装软件所摆放的目录
>
> - /proc：此目录的数据都在内存中，如系统核心，外部设备，网络状态，由于数据都存放于内存中，所以不占用磁盘空间，比较重要的文件有：/proc/cpuinfo、/proc/interrupts、/proc/dma、/proc/ioports、/proc/net/* 等
>
> - /root：系统管理员root的家目录
>
> - /sbin、/usr/sbin、/usr/local/sbin：放置系统管理员使用的可执行命令，如 fdisk、shutdown、mount 等。与 /bin 不同的是，这几个目录是给系统管理员 root 使用的命令，一般用户只能"查看"而不能设置和使用
>
> - /tmp：一般用户或正在执行的程序临时存放文件的目录，任何人都可以访问，重要数据不可放置在此目录下
>
> - /srv：服务启动之后需要访问的数据目录，如 www 服务需要访问的网页数据存放在 /srv/www 内
>
> - /usr：应用程序存放目录
>
>   - /usr/bin：存放应用程序
>   - /usr/share：存放共享数据
>   - /usr/lib：存放不能直接运行的，却是许多程序运行所必需的一些函数库文件
>   - /usr/local：存放软件升级包
>   - /usr/share/doc：系统说明文件存放目录
>   - /usr/share/man：程序说明文件存放目录
>
> - /var：放置系统执行过程中经常变化的文件
>
>   - /var/log：随时更改的日志文件
>   - /var/spool/mail：邮件存放的目录
>   - /var/run：程序或服务启动后，其 PID 存放在该目录下

**SSH基础：**

<img src="Robocom世界机器人开发者大赛参赛总结\SSH示意图.png" style="zoom:72%;" />

- **IP 地址**：通过 **IP 地址** 找到网络上的 **计算机**

- **端口号**：通过 **端口号** 可以找到 **计算机上运行的应用程序**

  | 序号 | 服务       | 端口号 |
  | ---- | ---------- | ------ |
  | 01   | SSH 服务器 | 22     |
  | 02   | Web 服务器 | 80     |
  | 03   | HTTPS      | 443    |
  | 04   | FTP 服务器 | 21     |

SSH客户端的简单使用：``ssh [-p port] user@remote``

免密码登陆：把客户端的公钥给服务器。

<img src="Robocom世界机器人开发者大赛参赛总结\SSH 免密码示意图.png" style="zoom:72%;" />

> 非对称加密算法
>
> - 使用 **公钥** 加密的数据，需要使用 **私钥** 解密
> - 使用 **私钥** 加密的数据，需要使用 **公钥** 解密

- apt 是 `Advanced Packaging Tool`，是 Linux 下的一款安装包管理工具
- 可以在终端中方便的 **安装**／**卸载**／**更新软件包**

### 2.代码分析

#### a)   teleop.py

##### 1.通信人员：

发布者①：发布cmd_vel话题，话题的数据类型为geometry_msgs.msg(几何移动话题消息)的Twist(数据结构)

​		**作用：用于控制机器人底盘移动**

发布者②：发布/grasp话题，话题的数据类型为std_msgs.msg(标准库消息)的String(字符串)

​		**作用：用于发布指令给grasp.py的订阅者作出相应的操作(如抓取与放下)**

订阅者①：订阅/grasp_status话题，话题的数据类型同为std_msgs.msg的String

​		**作用：用于接受grasp.py返回的消息，维护机器人的所处的状态(抓取状态还是放下状态)**

##### 2.运行流程：

​     采用python标准库读取键盘按键做出相应处理(终端输入不回显)

#### b)    grasp.py

#####  1.通信人员：

订阅者①：订阅/grasp话题，话题的数据类型为std_msgs.msg的String。

​        **作用：接收teleop.py与move.py发布的抓取命令(‘1’)与放下命令(‘0’)并做出响应。**

发布者①：发布position_write_topic话题，话题的数据类型为position。

​        **作用：发布机械臂的位姿，告诉机械臂应该移动到哪里(x,y,z三个坐标)。**

发布者②：发布pump_topic话题，话题的数据类型为status。

​        **作用：发布机械臂吸盘的指令，控制气泵的吸取与松开。**

发布者③：发布grasp_status话题，话题的数据类型为String。

​        **作用：发布机械臂的状态，让teleop.py维护机器人所处的状态（1抓到,0放下,-1没抓到）。**

发布者④：发布/cmd_vel话题，话题的数据类型为geometry_msgs.msg的Twist。

​        **作用：控制机器人转动找物块。**

##### 2.运行流程

**多线程编码（python的threading模块）**

###### **线程一：维护ROS运作**

​	订阅者收到抓取信息后进入回调函数，处理如下：

​	**若接受到抓取指令(‘1’)则执行抓取**

​	抓取的前提是另一线程赋予了抓取点xc,yc并将is_found_object布尔变量赋值为真；

​	若前提不满足，则转圈，转完还不满足，发布者③发布’-1’表示没抓到同时退出函数；若前提满足，则执行抓取，同时发布者③发布’1’表示抓到了。

​	**若接受到放下命令(‘0’)则执行放下**，同时发布者③发布’0’表示放下了。 

​	**抓取的实现：**引进’thefile.txt’文件数据矫正抓取点的误差，发布机械臂位姿(发布者①)到xc,yc点上，发布吸盘启动命令(‘1’)(发布者②)并提起。

**放下的实现：**发布机械臂位姿(发布者①)到地面(200,0,-40)，发布吸盘释放命令(‘0’)(发布者②)。

###### **线程二：维护OpenCv运作**

OpenCv的捕获视频是通过循环捕捉图像形成的

图像的处理流程：

捕获——>转为灰度图——>阈值计算转为二值图像,在设定的HSV（在比赛中是识别蓝色物块）内加强为白，其余变暗为黑——>收缩（让识别到的东西尽量完整）——>腐蚀边界(让黑白分明)——>高斯模糊——>获取轮廓矩形中点xc,yc坐标，同时包含对机械臂到点位置的测量（这个测量是二维的），能否抓取的判断(距离与确保静止)

#### c)    move.py

##### 1.通信人员

订阅者①：订阅clicked_point话题，话题的数据结构为geometry_msgs.msg的PointStamped。

​    **作用：订阅RVIZ上的点击事件。**

订阅者②：发布/grasp_status话题，话题的数据结构为std_msgs.msg的String。

​    **作用：订阅grasp.py发布出来的抓取状态，维护机器人所处的状态(抓取状态还是放下状态)**

发布者①：发布cmd_vel话题，话题的数据结构为geometry_msgs.msg的Twist。

​    **作用：结束程序时发布停下来的指令(0,0,0)。**

发布者②：发布/grasp话题，话题的数据结构为sed_msgs.msg的String。

​    **作用：发布抓取命令与放下命令给grasp.py。**

服务器①：请求move_base服务，服务的数据类型为move_base_msgs.msg的MoveBaseAction。

​    **作用：与move_base服务器连接，实时维护机器人所处的位置。**

##### 2.运行流程

订阅者接受到点击事件后进入回调函数cp_callback，处理如下：

​    定义目标点变量，类型move_base.msgs的MoveBaseGoal()，是自定义的一种类

​    设置传进来的目标点的位置(Point)与机器人坐标位姿四元组(Quaternion)，位姿四元组是控制机器人旋转中很重要的参数。

​    用move函数移动到目标点，到达目标点之后就发布抓取命令(发布者②)，之后等待grasp返回的抓取状态信息(订阅者②) 

若接受到抓取成功信息(’1’)后，调用grasp_status_cp函数回到起始点并放下物体。

**move**函数实现：让move_base服务器发送目标点，服务器自动规划路线(采用ROS的gmaping规划路线算法)(全局规划与本地规划都由其他配置文件写出)，若超时未到达会取消目标。move_base服务器内部的许多函数都是其他配置文件配置好的，函数还蛮多的，由底层一层层封装上来的，这里我没有细究。

**grasp_status_cp**函数实现：与move类似，自动规划路线到起始点(0,0,0,0)，差别在于没有时间限制，因此也没有cancel_goal()

## 三、 整合功能需求

### 按实现的时间顺序列写需求

###### 1. 建立github项目(私仓)

代码常改动，备份麻烦而且机器人未开机时无法查看代码，git与github很好的解决了这个问题。

仓库地址：https://github.com/My-Polaris/spark-259e

**关键的git命令：**

git diff 查看版本库与工作区的不同

git checkout 将工作区回退到add(暂存区)的状态

git reset --hard HEAD 回退到版本库	(加个-f是强行回退，一般回退到比远程库还老的版本需要加-f)

###### 2. 使用SSH-FS让VS Code远程连接机器人

机器人本身linux系统无专门写代码的软件，python又注重缩进，所以有一个代码编辑软件很重要，而VS Code的SSH-FS可以远程编辑代码，Great。

###### 3. 调整机器人移动至流畅

机器人移动卡顿，需调整至移动流畅。(sleep()与stop_robot()函数)

###### 4. 添加丢物块与堆叠物块功能

减少放物块所需的时间。

```python
#丢物块
	注释掉机械臂挨到地底的动作直接松开气泵
    self.pub1 = rospy.Publisher('position_write_topic', position, queue_size=10)
    #self.pub1.publish(pos)
    self.pub2 = rospy.Publisher('pump_topic', status, queue_size=1)
    self.pub2.publish(0)#0是松开气泵，1是打开气泵,pub2
#堆叠物块
	调节position()对象的高度值
    pos.z = 50
```

###### 5. 机械臂抓取的速度

机械臂配置源代码路径：src/spark_driver/arm/uArm/swiftpro/src/swiftpro_write_node.cpp

```c++
配置文件是用C++写的，将机械臂移动速度的数值由4个0改为5个0
//Gcode = (std::string)"G0 X" + x + " Y" + y + " Z" + z + " F10000" + "\r\n";	
Gcode = (std::string)"G0 X" + x + " Y" + y + " Z" + z + " F100000" + "\r\n";
```

###### 6. 机械臂复位功能

机械臂被强制移位时抓取会有偏差，同时在对抗中机械臂是很容易发生碰撞的，因此需要复位功能。

用源代码里的position()移动到初始位置并不可行，因为机械臂被撞了之后已经失去了机械臂的位置信息，所以必须重启机械臂才能进行复位。

查阅机械臂[uArm Swift文档](https://github.com/uArm-Developer/RosForSwiftAndSwiftPro)找到控制机械臂状态的话题。

```python
# 机械臂复位
	声明SwiftProStatus这个类，发布重置机械臂状态的命令
    if msg.data=='3':
        # 机械臂恢复
        status = SwiftProStatus()
        status_pub = rospy.Publisher('/swiftpro_status_topic',SwiftProStatus,queue_size=1)
        rate = rospy.Rate(5)
        status.status = 0	#关闭对机械臂的控制
        status_pub.publish(status)
        rate.sleep()
        status.status = 1	#开启对机械臂的控制
        status_pub.publish(status)
        rate.sleep()
        pos = position()
        pos.x = 120
        pos.y = 0
        pos.z = 35
        #发布停止气泵的命令，让重置机器臂的同时关闭气泵
        self.pub2.publish(0)
        self.pub1.publish(pos)
```

###### 7. 扫物块功能

将机械臂置于左右侧约60°，从而增加底部面积，能推更多的物块。

```python
# 扫物块模式,x是远近(越远数值越大),y是左右(左正右负),z是高度(越高数值越大)
    这里的实现还蛮简单的，但是这个思路很好
        if msg.data=='4':
            #左扫
            pos = position()
            pos.x = 160
            pos.y = 208
            pos.z = -118
            self.pub1.publish(pos)
        if msg.data=='5':
            #右扫
            pos = position()
            pos.x = 160
            pos.y = -208
            pos.z = -118
            self.pub1.publish(pos)
```

这里其实还有个问题，由于position的x,y,z都是相对的，也就说说改变单一变量带来的效果不是固定的，并不是y越小机械臂就越靠右，也不是x越大机械臂就越远，因此要让机械臂移动到达理想的位置是很难通过该参数一次次试出来的。

**解决办法**：找到调出实时显示机械臂x,y,z参数值的窗口的方法，手动移动机械臂到目标位置然后记下这个点的x,y,z值。

```python
终端命令
.devel/setup.sh	#环境变量
roslaunch swiftpro pro_display.launch #机械臂位姿定位
rqt_console #日志输出工具
```



###### 8. 摄像头图像处理修改

**信息实时显示**：物块的坐标信息，识别物块的轮廓，计算出来的点与机械臂距离信息。

```python
 #数字信息显示
    str_x = "x_mid: "+str(int(x_mid)) #物块的点的x坐标
    str_y = "y_mid: "+str(int(y_mid)) #物块的点的y坐标
    str_A = "Area: "+str(int(w*h)) #物块的面积
    str_D = "dist: "+str(int(math.sqrt((340 - x_mid) ** 2 + (235 - y_mid) ** 2))) #物块到点(340,235)的距离
#物块轮廓画图                    
    frame = cv2.drawContours(frame,[contours[i]],0,(0,255,0),3)
    frame = cv2.putText(frame,str_x, (int(x_mid),int(y_mid)), cv2.FONT_HERSHEY_COMPLEX, 0.3, (0, 255, 255), 1)
    frame = cv2.putText(frame,str_y, (int(x_mid),int(y_mid)+10), cv2.FONT_HERSHEY_COMPLEX,0.3, (0, 255, 255), 1)
    frame = cv2.putText(frame,str_A, (int(x_mid),int(y_mid)+20), cv2.FONT_HERSHEY_COMPLEX, 0.3, (0, 255, 255), 1)
    frame = cv2.putText(frame,str_D, (int(x_mid),int(y_mid)+30), cv2.FONT_HERSHEY_COMPLEX, 0.3, (0, 255, 255), 1)
```

**图像识别的修改：**

1.注释掉图像扩张，将收缩函数的参数增大，让抓取点更加集中，准确。

```python
 mask = cv2.erode(mask, None, iterations=5)
 #mask = cv2.dilate(mask, None, iterations=2)
```

2.重新定义机械臂到抓取点的距离与最大距离

```python
 # 所有点到spark的距离
   	distance_list.append(math.sqrt((340 - x_mid) ** 2 + (235 - y_mid) ** 2))
```

这里的340与235分别对应机械臂红标在屏幕中的x,y坐标信息，这个时候前面的信息实时显示就派上用场啦
如果机械臂红标在摄像头中对应的值是355,245，那这个值也要更换为355,245。

3.判别抓取点能否抓取（红色点是即将抓取的点，蓝色点为识别到但不抓取或距离过远的点）。

```python
 #将目标点显示出来，红色的点，其余识别到但不抓取的点是蓝色的
	if found_count >= 15 and min_distance < 350:
		cv2.circle(frame, (np.int32(xc), np.int32(yc)), 4, (0,0,255), -1)
```

这里的350也是要可以变化的，得确保抓的点机械臂是能够得到的

这个图像识别的修改是很重要的，特别是对于自动赛的抓取来说，这极大降低了抓空的概率。

###### 9. 监听键盘

使用监控键盘功能(无堵塞)代替标准输入流(堵塞导致控制不灵活)，将读取键盘变得更流畅，能同时获取读取两个按键(边前进边转弯)

Keyboard.py

```python
from pynput import keyboard

class Keyboard:
    def __init__(self):
        self.pressedKey = {}
        self.Key = keyboard.Key

        def on_press(key):
            if type(key) is keyboard.KeyCode:
                self.pressedKey[key.char.lower()] = True
            else:
                self.pressedKey[key] = True

        def on_release(key):
            if type(key) is keyboard.KeyCode:
                self.pressedKey[key.char.lower()] = False
            else:
                self.pressedKey[key] = False

        listener = keyboard.Listener(
            on_press=on_press,
            on_release=on_release)
        #listener.join()
        listener.start()

    def isPress(self, key):
        return self.pressedKey.get(key, False)

    def isPressAndWaitForRelease(self, key):
        if not self.isPress(key):
            return False
        self.waitForRelease(key)
        return True

    def waitForPress(self, key):
        while not self.isPress(key):
            pass

    def waitForRelease(self, key):
        while self.isPress(key):
            pass

    def wait(self, key):
        self.waitForPress(key)
        self.waitForRelease(key
```

这里用了python的pynput库，主要监听键盘的按下与释放，这里比较麻烦的是得分开处理移动命令的监听与功能命令的监听，移动命令是只要按下就生效，而功能命令是按下并松开才生效。

###### 10. 微调功能

按住Shift键可使转动和移动都大幅降低。

```python
#速度变量
    # 慢速
    walk_vel_ = rospy.get_param('walk_vel', 0.1)
    # 快速
    run_vel_ = rospy.get_param('run_vel', 0.5)
    # 慢转
    yaw_rate_ = rospy.get_param('yaw_rate', 0.3)
    # 快转
    yaw_rate_run_ = rospy.get_param('yaw_rate_run', 3)
    # shift 慢速模式
    if keyboard.isPress(keyboard.Key.shift):
    	max_tv = walk_vel_
    	max_rv = yaw_rate_
```

###### 11. 叠三层

可用于刷强度分。

```python
#举到三层的位置
if msg.data=='6':
    pos = position()
    pos.x = 213
    pos.y = -4
    pos.z = 170
    self.pub1.publish(pos)
```

###### 12. 动态规划返航点

建立新的点击事件及其订阅者，实时选择返航点设置。

```python
 #收到点击事件后就把这个点赋值给self的返航点
    rospy.Subscriber('clicked_point_2', PointStamped, self.cp_callback_2)
    def cp_callback_2(self,msg):
    	self.EndPose = msg
```

###### 13. 自定义路径规划

建立新的点击订阅者，手敲自定义的moveTo，不用move_base服务器配置的动态规划路线(避障)，点哪去哪。

```python
def move_2(self,msg):
    #获取初始位置
    # self.initial_pose = PoseWithCovarianceStamped()    
        print "hahhaha"
        #print dest
        while True:
            dest = msg 
            pose = self.tf.get_pose("map", "base_link")
            print "hahah"
            print pose 
            yaw = self.tf.yawBetweenPoseAndPoint(pose, dest.point)

            delta_x = dest.point.x - pose.position.x
            delta_y = dest.point.y - pose.position.y
            distance = math.sqrt(delta_x ** 2 + delta_y ** 2)

            if distance < 0.05:
                self.move_1(0,0)
                break
            elif distance < 0.3:
                if abs(yaw) < self.tf.angel2Radian(90):
                    speed = -yaw * 1
                    if speed > 1:
                        speed = 1
                    self.move_1(0.1, speed)
                else:
                    speed = -self.tf.standardRadian(yaw - pi) * 10
                    if speed > 1:
                        speed = 1
                    self.move_1(-0.1, speed)
            else:
                if abs(yaw) < self.tf.angel2Radian(60):
                    speed = -yaw * 10
                    if speed > 2:
                        speed = 2
                    self.move_1(0.4, speed)
                else:
                    if yaw < 0:
                        self.move_1(0, 1.5)
                    else:
                        self.move_1(0, -1.5)
```

move_2在点击事件2中触发，具体实现是直接往定义的点移动，在接近点时减慢速度并微调角度以便摄像头识别到物块。

具体的TF移动代码封装在了TF.py里。

在src/3rd_app/move2grasp/rviz/spark_gmapping.rviz里创建Rviz页面的点击事件2。

```
    - Class: rviz/PublishPoint
      Single click: true
      Topic: /clicked_point_2
```

###### 14. 歪脖子抓取与自动堆叠物块

自动堆叠物块的实现思路：与抓取同理，只不过将抓取与开启气泵换为关闭气泵。

为什么要歪脖子抓取：为了不让机械臂挡住摄像头，所以在抓取时要让机械臂以歪脖子的形式举在侧方。

这里要注意的是要写多一个判别语句，当画面中找不到物块时将物块放在正前方，当找到物块时就放在这个物块的上方。

## 四、  最终代码实现及相关注释

​	见github项目：https://github.com/My-Polaris/spark-259e/tree/master/src

## 五、  比赛日记与参赛总结

### 1.深圳初赛

​	第一天的半自动赛让我了解到，还是少不了大心脏的(虽然不是我操控哈哈)。

​    同时，赛制也十分重要，搞清楚赛制才是拿下胜利的前提。

### 2.杭州决赛

###### **半自动对抗赛**：

​	差点玩拖，忽略了强度分，不过还好进了决赛。

​	最后决赛1V1的过程真的很惊险。

######  **自动对抗赛**：

​	比赛前一天的晚上我们搞定了MoveTo.py，但比赛期间遇到蛮多状况的，机器人热点突然连不上、**摄像头的螺丝太松了，机器人移动几下摄像头就偏位**，后者的问题影响太大了，第二场比赛结束后摄像头甚至垂直面向了地面，但直至比赛全部结束我们才发现问题出在螺丝上，蛮可惜的。

​    相比于自动赛优秀的队伍，启发就是我们不够重视建立静态地图，同时对于比赛的策略也不够灵活。

**总的来说这次参赛所学到的东西还蛮多的，一步步走来也多少有坎坷，竞赛所带来的提升是全方面的，得到的提升不仅仅是编码能力，还有解决问题的能力、检索信息的能力，团队协作的能力，临场应对的能力等等，许多东西都是课本上学不到的，一场比赛能获得什么其实不仅在于比赛本身，更在于自身如何去准备，如何去看待。**