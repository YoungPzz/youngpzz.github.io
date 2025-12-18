---
title: 【Android Tool】-- webrtc源码Android m94版本编译过程及踩坑
date: 2025-12-18 10:56:34 +0800
categories: [Android, Tool]
tags: [Android]     # TAG names should always be lowercase
description: webrtc源码编译过程及踩坑

---


  最近因项目需求，需要编译Android WebRTC m94版本的包。

  ## 环境准备
   这也是踩坑不断的地方。以下是经过的过程：

  1. **Linux环境**。因为编译过程中许多编译工具对Linux更友好。不想在虚拟机实现，所以先在实验室的服务器去弄，弄了半天发现不支持Ubuntu22.04版本.......
      
![image.png](https://p0-xtjj-private.juejin.cn/tos-cn-i-73owjymdk6/d73e6ec624d640d48e4846bf26cec1b3~tplv-73owjymdk6-jj-mark-v1:0:0:0:0:5o6Y6YeR5oqA5pyv56S-5Yy6IEAgS0REQXllc3A=:q75.awebp?policy=eyJ2bSI6MywidWlkIjoiMzEwNTQ3MzQyMzc0NDY2MyJ9&rk3s=e9ecf3d6&x-orig-authkey=f32326d3454f2ac7e96d3d06cdbb035152127018&x-orig-expires=1766114294&x-orig-sign=CwjUlZ8zPlSCkvS8zIBD9PKl9b0%3D)

  2. **网络环境**。因为是国外的源，所以开始想的是用本地主机（挂了梯子）作为**SSH 反向隧道**，它能在本机和云服务器间建立加密通道，让云服务器借助本机代理上网。一开始用这方法下载体积小的**depot_tools**还可以，一但开始拉webrtc源码，就开始崩溃不断。
  3. **服务器存储**。磁盘至少50GB，内存可以小一点，利用swap临时增加8GB交换分区缓解内存压力。
```
sudo fallocate -l 8G /swapfile
sudo chmod 600 /swapfile
sudo mkswap /swapfile
sudo swapon /swapfile
```

**最终环境** ：由于代理的方式实在不可靠，所以在阿里云上开了个在美国硅谷的服务器，几十块钱一个月，版本选择Ubuntu20.04、磁盘开了70GB。

ps：如果要用本地的xftp管理文件，设置代理会流畅很多

![image.png](https://p0-xtjj-private.juejin.cn/tos-cn-i-73owjymdk6/037a982689954b12b00f10ac7d1b3189~tplv-73owjymdk6-jj-mark-v1:0:0:0:0:5o6Y6YeR5oqA5pyv56S-5Yy6IEAgS0REQXllc3A=:q75.awebp?policy=eyJ2bSI6MywidWlkIjoiMzEwNTQ3MzQyMzc0NDY2MyJ9&rk3s=e9ecf3d6&x-orig-authkey=f32326d3454f2ac7e96d3d06cdbb035152127018&x-orig-expires=1766119670&x-orig-sign=xpxIiY407G2ejGzxfy3%2B00LgrWg%3D)

## 下载源码过程

### 1.下载**depot_tools**
官方没有使用git来管理源码，官方工具链：WebRTC的`depot_tools`（含`gclient`等工具）是依赖管理的“金标准”，手动下载工具极易出现兼容问题；

```
//clone
git clone https://chromium.googlesource.com/chromium/tools/depot_tools.git
//编辑环境变量
//编辑 ~/.bashrc 将 depot_tools 添加到路径中 
vim ~/.bashrc 
（加入该行）export PATH=$PATH:/(path to depot_tools)/depot_tools
source ~/.bashrc
//检查 depot_tools 是否安装配置成功，输出路径即成功
which gn
which gclient
```

### 2.拉取源码

```
//获取 Android WebRTC 代码
fetch --nohooks webrtc_android
//若其中发生中断则执行如下命令继续
gclient sync
//切换到 m94 分支并同步 
cd src git checkout -b m94 branch-heads/4606 
gclient sync --nohooks 
gclient runhooks 
下载安装相关依赖 
cd src ./build/install-build-deps.sh 
./build/install-build-deps-android.sh
```

### 3.编译
文件修改参考：https://juejin.cn/post/7221454955265556540#heading-3

执行编译命令 
x

```
./tools_webrtc/android/build_aar.py --extra-gn-args 'is_debug=false is_component_build=false is_clang=true rtc_include_tests=false rtc_use_h264=true rtc_enable_protobuf=false use_rtti=true use_custom_libcxx=false' --build-dir ./out/release-build/m94/ -------------
```

编译结果输出路径 out/release-build/m94/armeabi-v7a/obj/libwebrtc.a 

out/release-build/m94/armeabi-v7a/lib.java/sdk/android/libwebrtc.jar
![image.png](https://p0-xtjj-private.juejin.cn/tos-cn-i-73owjymdk6/d61d096cdae5455a9e9c30dfabd34b03~tplv-73owjymdk6-jj-mark-v1:0:0:0:0:5o6Y6YeR5oqA5pyv56S-5Yy6IEAgS0REQXllc3A=:q75.awebp?policy=eyJ2bSI6MywidWlkIjoiMzEwNTQ3MzQyMzc0NDY2MyJ9&rk3s=e9ecf3d6&x-orig-authkey=f32326d3454f2ac7e96d3d06cdbb035152127018&x-orig-expires=1766119734&x-orig-sign=oFfdBFPV%2BDALFSVuBehruli4Tgo%3D)
