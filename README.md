# -坐标获取实现步骤：


0.刷机
在拿到Jetson Xazier NX后的第一步是刷机，刷机有龙老师给的视频教程，但是需要注意的是，需要在自己的笔记本上安装虚拟机，版本一般为Ubuntu18.04，之后在虚拟机里安装SDKmaneger刷机模块，视频里都有。
需要注意的是，视频里刷机时没有装CUDA等库，我建议最好一次性装完，因为cuda和cudnn的作用是是处理神经网络，GPU版本的torch和torchvision安装前提需要cuda和cudnn。
还需要注意的是，Jetpack5.0以上一般都是Ubuntu20.04, Jetpack5.0以下为Ubuntu18.04，新系统资料少（但是现在其实也不少）。
1.安装Miniforge并配置虚拟环境
Jetson主板为arm结构，和普通的Ubuntu系统不一样，因此，大多数时候不能直接用pip install 安装某些库，从源码安装还是比较好的。
conda是一个开源的包、环境管理器，可以用于在同一个机器上安装不同版本的软件包及其依赖，并能够在不同的环境之间切换。搞深度学习的应该都十分熟悉anaconda，但是NVIDIA Jetson Xavier NX是arm架构的，anaconda及其精简版miniconda并不支持arm64架构。现在主流的CPU架构分为Intel的x86/x64架构和ARM的ARM/ARM64两种，平常用的电脑大部分都是x86/x64的（苹果除外），Xavier使用的是ARM64，所以很多在x86/x64上能用的的东西到了它这里就不能用了。这一点请谨记，如果你在Jetson上遇到什么奇奇怪怪的例如“No such file or directory”之类的问题，第一时间要考虑是不是版本不是ARM64的版本。
在ARM64上的anaconda替代品是miniforge，miniforge与miniconda的区别在于miniforge的下载通道是conda-forge，其他基本没什么不同。
Miniforge安装可以参考如下链接：NVIDIA Jetson Xavier NX入门（3）——安装miniforge和Pytorch_文山湖的猫的博客-CSDN博客
（1）先到miniforge的官方下载地址下载对应的sh文件：https://github.com/conda-forge/miniforge/releases
别下错了，例如我下载的是Miniforge-pypy3-4.10.3-3-Linux-aarch64.sh,代表适用于arrch64架构下的Linux系统。（ARM64对应32位和64位分为arrch32和arrch64）。
（2）进入到miniforge的sh文件所在目录，右键打开Terminal，输入以下命令进行安装：
sh Miniforge-pypy3-4.10.3-3-Linux-aarch64.sh （sh后面的看自己下载的文件名称）
（3）安装完毕后，添加环境变量，否则会出现bash:conda Command not found的错误。一般在home文件下有一个.bashrc文件，这是一个隐藏文件，需要按ctrl+h才能显示，打开之后在最后添加环境变量即可：
export PATH=/home/<username>/miniforge-pypy3/bin:$PATH
但是记得source .bashrc （使环境变量生效）
顺便提一下vim编辑器按a是进入编辑模式，编辑完毕后按ESC退出编辑模式，再输入:wq!是保存并退出。总感觉有人不知道。


	安装完之后，总会下载东西，因此需要换源，中科大，清华，阿里源都可以，前提是这个源必须是带有ports的源，否则arm架构无法使用。参考链接：[jetson]jetpack5.0.2的ubuntu20.04更换为清华源_jetson 清华源_FL1623863129的博客-CSDN博客
 


Miniforge安装之后，可以创建自己的虚拟环境，这里注意下，配环境或者安装其他库最好在自己创建的虚拟环境下进行，base环境如果乱了需要刷系统。
创建自己的虚拟环境：
 conda create -n name python=3.x 
name是虚拟环境的名字，python版本自己设定。到此，虚拟环境配置完成。

我已经创建了pytorch环境，激活：conda activate pytorch 
进入文件路径：python xxx.py（运行py文件）
2. 配置yolov5&pytorch环境

在安装之前，需要安装cmake，因为以后很多都是从源码安装，需要用到cmake，安装链接，这个链接很详细jetson nano 升级cmake_jetson nano升级cmake_AI浩的博客-CSDN博客

参考这个链接就够了Jetson nano部署Yolov5 ——从烧录到运行 1:1复刻全过程_jetson nano安装11复刻_IamYZD的博客-CSDN博客
需要说明的是，Jetpack torch torchvision版本需要对应，这个对应关系在csdn上很好查到，因此安装的时候torch和torchvision要注意版本对应。
torch的下载在PyTorch for Jetson - Jetson & Embedded Systems / Jetson Nano - NVIDIA Developer Forums
torchvision在pytorch/vision: Datasets, Transforms and Models specific to Computer Vision (github.com)
不过上面这俩科学上网下载更快一些。
安装torch和torchvision，参考上文jetson nano文章，torch是在pytorch环境下pip安装whl文件
Pip install “whl文件的路径”
Torchvision安装：进入torchvision文件路径，打开终端，输入：
python setup.py 

因为深度相机有自己的库，所以还需要单独安装pyrealsense2这个库，千万不能pip直接安装
Intel RealSense D435介绍、安装和使用：Intel RealSense D435介绍、安装和使用_英特尔d435_cherry_yu08的博客-CSDN博客
Realsense的GitHub地址：IntelRealSense/librealsense: Intel® RealSense™ SDK (github.com) 
安装步骤：librealsense/wrappers/python at master · IntelRealSense/librealsense · GitHub
同时结合这篇文章一起看：NVIDIA Jetson Xavier NX 安yolo v5 +D435i摄像头 pyrealsense2 亲测好用_思艺妄为的博客-CSDN博客

需要注意的是，这里需要从源码进行安装，因此比较慢，安装完之后，pyrealsense2和realsense的SDK就应该安装好了。

验证：进入虚拟环境，终端输入python，输入import pyrealsense2，没有报错即安装成功。 

3.安装ros
参考链接：详细介绍如何在ubuntu20.04中安装ROS系统，超快完成安装（最新版教程）_ubuntu安装ros_慕羽★的博客-CSDN博客
需要注意的是：
 
这一步会很慢，而且容易出错，很多时候是网络不好，换个人热点有时候就能解决问题。


这里附带一个ROS学习网站，讲的很详细，但是如果只是想会用的话不用详细去学。
Introduction · Autolabor-ROS机器人入门课程《ROS理论与实践》零基础教程

4.运行坐标转换程序

首先将压缩包git到本地文件，也可以去官网直接下载，建议后者，因为版本选择更直观和清晰，下载网址：Thinkin99/yolov5_d435i_detection: 使用realsense d435i相机，基于pytorch实现yolov5目标检测，返回检测目标相机坐标系下的位置信息。 (github.com)
 
需要注意的是，当更改权重文件的时候，需要使用yolov5 v5.0训练出来的权重，在rstest.py文件中修改config/xxx.yaml，需要自己在config里自己写一个yaml文件，更改权重路径、nc和种类即可。

运行：
Conda activate pytorch 进入pytorch环境
Cd rstest.py文件所在的路径
Python rstest.py （显示坐标的文件与直接下载完的不同，有改动，已备份）
