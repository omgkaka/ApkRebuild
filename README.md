#Apk反编译&回编译（一）
---
###本篇主要讲解未加固型（纯混淆+签名）apk反编译和回编译方法

1. 主要使用工具：ApkTool ， Smali2Java ， apk签名工具

2. Apk反编译之使用姿势   
这里使用demo.apk 作为反编译文件 
 
a.安装ApkTool， 将demo.apk放入ApkTool所在文件夹，打开Terminal终端，使用cd命令进入ApkTool所在路径。 示例： 
如ApkTool安装在D:\ApkTool 下，则将demo.apk放入该文件夹下，打开Terminal终端，依次输入 D:回车 --> cd ApkTool  回车 即可进入ApkTool所在路径 

b.此时Terminal定位在D:\ApkTool> , 输入 apktool d demo.apk  回车， 稍等几秒即会在D:\ApkTool下生成demo文件夹，该demo文件夹即是demo.apk反编译后的内容![](http://p1.bpimg.com/567571/f58e0fa5e27dc567.png) 

c.demo文件夹文件分析 
![](http://p1.bpimg.com/567571/a23b3e37482fc329.png) 
这里比较关键的文件（夹）： smali 和 AndroidManifest.xml   res主要是该apk文件所用到的资源（图片，文字等）文件 
smali ： 里面放置了许多smali文件，这里就是整个apk的核心逻辑代码，但是smali语言极难看懂，类似汇编语言（网上都这样说，哈哈），不过一般的方法（函数）的开始和结束以及调用能看懂一二，对于我们这种smali渣渣来说就足够用了。
AndroidManifest.xml ： 所谓的清单文件，主要配置了该apk版本，编译版本，包名，权限以及组件的声明等重要信息.下面的回编译注入自己的代码时需要对该文件进行写操作。
 
d.上面说到smali比较难看懂，Smali2Java这个工具即可将smali转为java，这就能看懂了，也就能分析别人的逻辑代码了，当然只能是部分，因为有代码混淆。 
  到此，Apk反编译也就结束了，反编译的作用主要是分析别人的逻辑代码，或者偷res的资源文件，再一个就是注入自己的逻辑代码回编译...

3. Apk回编译之使用姿势  

a. 简单回编译： 对于这种替换个App名称，版本号，换个图片什么的，图片可通过替换res下对应的图片（名字大小格式确保一样），App名称，版本号什么的可直接修改上面提到的AndroidManifest.xml清单文件，保存。 打开Terminal终端，使用cd命令进入ApkTool所在路径。 此时Terminal定位在D:\ApkTool>  输入 apktool b demo  回车， 稍等几秒即会在demo文件夹下生成dist和build文件夹（不用管build文件夹），dist里面就是回编译生成的apk文件![](http://p1.bqimg.com/567571/16d52ce06beb47c5.png)   
![](http://p1.bqimg.com/567571/f949e2f34cff515e.png)  
![](http://p1.bqimg.com/567571/4507d11d07db9db5.png)  
但是该apk不能安装使用，需要对其重新签名即可安装使用，签名步骤略去

b. 注入自己的逻辑代码回编译： 这里以嵌入有米广告积分墙SDK为例  
在正常的项目开发中嵌入此SDK需要配置这些信息--> 1.在AndroidManifest.xml 清单文件中配置有米积分墙SDK所需的权限，组件声明等信息； 2.在项目中引入有米SDK所需的jar包；  3.在代码中添加打开有米积分墙的逻辑代码  
现在换成回编译又怎么处理呢？  下面来分析思路： 1. 配置清单这一步骤和简单回编译一样的，直接在清单文件中添加权限，组件声明代码即可  2. 引入jar包， 回编译的时候怎么引入jar包呢， 查看反编译后的smali文件夹就会发现，项目依赖的所有jar包都被反编译到smali文件夹下面了，这样一来，我们只需自己撸个简单的集成了有米SDK的项目生成release版本的test.apk文件，反编译即可在smali文件夹下获得有米sdk的smali文件，放在了 smali/net/youmi/android这个文件夹下，如图： 
![](http://i1.piimg.com/567571/7c1e1ea9eb6987ee.png) 
复制net文件夹到demo/smali 文件夹下即可![](http://i1.piimg.com/567571/2def79ce0c0a1664.png) 
3.在代码中添加打开有米积分墙的逻辑代码， 回编译是把smali文件转为dex文件， 嵌入的逻辑代码又是java代码，这时候只需要上一步的test.apk生成的smali文件中的逻辑代码复制到demo/smali文件夹下对应的smali文件中。
下面是打开有米积分墙的smali代码，写成了一个方法，![](http://i1.piimg.com/567571/cf41e4d5d0970690.png)
调用的时候  invoke-virtual {p0}, Lcom/jeffer/ym/MainActivity;->showYM()V，其中的com/jeffer/ym/MainActivity换成你需要注入代码所在的Activity的全类名。 最后在Terminal定位到D:\ApkTool>  输入 apktool b demo  回车， 稍等几秒即会在demo文件夹下生成dist和build文件夹（不用管build文件夹），dist里面就是回编译生成的apk文件，重新签名后安装，打开APP即可在你注入代码处打开有米积分墙。

c. 注入逻辑代码和控件回编译： 下回分解 
 
 


 
---

							Date: 2016.12.2
