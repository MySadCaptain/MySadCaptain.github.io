---
layout:     post           
comments: true
title:      一键批量将PPT/Word文档转为PDF              # 标题 
subtitle:   
date:       2018-04-16             # 时间
author:     Matt                   
header-img: img/post-bg-desk.jpg    
catalog: true                      
tags:                               #标签
    - App
---

相信不少人都有或曾经有过需要将多个PPT/Word文件转为PDF的需求，可能是一堆PPT课件为了方便批注，也可能是一些Word文档为了方便阅读。每次只能打开一个文档，选择“另存为”，选“PDF”，点“保存”，关掉，再打开下一个文档，文档数目一多，整个过程就会变得很令人沮丧。

最近我研究了一下这个磨人的问题，制作了一个动作可以在不到2秒的时间将多个PPT/Word文件转为PDF。

[演示视频](https://cl.ly/0O1t1J0F3A05)

视频中我剪掉了转换过程中等待的时间，可以看到，每转换完成一个文件都会有通知，全部转换完成之后也会有通知。

## 准备工作

想要进行批量转换，肯定要依赖于Terminal命令，然而Microsoft Office系列并不支持通过Terminal命令进行文件格式转换。经过一番搜索，找到了一个免费的开源软件LibreOffice。通过其官网下载dmg的方式安装最新版即可。

通过查看其manual可知执行格式转换的命令如下
`soffice --convert-to pdf filename`
filename为待转换的文件。


如果想要批量转换，只需要
1. 将待转换文件放到一个文件夹
2. cd 到待转换的文件所在文件夹
3. 执行`soffice --convert-to pdf *.ppt` 或者`soffice --convert-to pdf *.doc`即可

![](https://i.imgur.com/SHFABZv.png)

\*是通配符，代替零个、单个或多个字符，\*.ppt会匹配所有格式为ppt的文件，如果需要转换的文件中既有ppt又有word文件，可以通过`soffice --convert-to pdf *`来实现，\*会匹配当前目录下所有文件。

到此为止已经实现了批量转换文件到PDF的工作，但是每次都要打开Terminal，cd到对应文件夹，复制粘贴命令，也有些麻烦，于是我通过制作LaunchBar动作的方式进一步简化。

## 制作动作

LaunchBar的instant send功能使得LaunchBar可以直接对选中的文件运行脚本，不需要打开Terminal。

在LaunchBar中，按下快捷键 ⌥Option-⌘Command-E，新建一个动作，贴上如下脚本，这里脚本语言我选择的是Python。你也可以在[这里](https://cl.ly/1p2l063Y1S3y)下载动作，直接添加到Launchbar。

```	
#!/usr/bin/env python
	#
	# LaunchBar Action Script
	#
	import sys
	import subprocess as sp
	import os
	import json
	import shutil
	
	my_env = os.environ.copy()
	my_env["PATH"] = "/usr/local/bin:" + my_env["PATH"]
	# Note: The first argument is the script's path
	
	for arg in sys.argv[1:]:
	        fileFolder = os.path.dirname(arg)
	        new_file= os.path.basename(arg)
	        my_command = ["soffice", "--convert-to", "pdf", arg, "--outdir", fileFolder]
	        sp.check_output(my_command, env=my_env)
	        my_command = ["osascript", "notification.scpt", new_file, "Conversion Finished"]
	        sp.check_output(my_command, env=my_env)
	
	my_command = ["osascript", "done.scpt", "All Finished!"]
	sp.check_output(my_command, env=my_env)

```
`for arg in sys.argv[1:]:`之前的代码负责导入库和声明环境变量，之后是针对选中的每一个文件进行如下操作。
值得注意的两条命令是

```
my_command = ["soffice", "--convert-to", "pdf", arg, "--outdir", fileFolder] 
```
和

```
my_command = ["osascript", "notification.scpt", new_file, "Conversion Finished"]
```
前者是格式转换的命令，注意这里加了 `"--outdir", fileFolder `来指明输出的目录为所选文件所在目录
后者负责为每一个完成的文件发送通知

这个动作到这里就制作完成了，操作起来也很简单
1. 选中文件
2. 长按“CMD+space”通过intant send功能发送到LaunchBar
3. 输入“convert to pdf”，一般输入前几个字母就可以匹配了

如视频中所展示的，整个过程不到2秒，可以将格式各异的Office文件统一转换为PDF

## 制作Automator动作

为了方便没有LaunchBar的人使用，在这里制作了同一动作的Automator版本，相比LaunchBar的动作，目前无法针对每一个文件发送完成通知，只能全部完成之后发送一个通知。

你可以在[这里](https://cl.ly/07131p1Y241e)下载这个Automator workflow，具体过程如下：
![](https://i.imgur.com/frzs21f.png)
在Automatic中新建一个workflow，这里选择Service，以便于选中文件之后右键在服务中找到这个workflow
 ![](https://i.imgur.com/0EdJ7nu.png)
接着按照图片里标记的顺序来创建这个动作即可
1. 把“Service receives selected”这里改为“documents”，因为我们处理的对象是office文档
2. 应用环境选择Finder 
3. 添加一个动作“Get Selected Finder Items”，因为我们处理的对象是Finder里的文件
4. 添加运行脚本动作“Run Shell Script”并将语言选择为Python
5. 把下面的一段代码粘贴上去，注意这里的代码和上一节的完全一样，只是去掉了通知部分代码，因为我不清楚怎么把通知脚本内嵌到这个workflow里，只能采用别的方式弹出通知
6. 添加弹出通知动作“Display Notification”，标题和副标题可以自己选择
7. 保存workflow，大功告成，我这里给这个workflow取的名字是“convertoPDF”

	
```
    #!/usr/bin/env python
	#
	import sys
	import subprocess as sp
	import os
	import json
	import shutil
	
	my_env = os.environ.copy()
	my_env["PATH"] = "/usr/local/bin:" + my_env["PATH"]
	# Note: The first argument is the script's path
	
	for arg in sys.argv[1:]:
	        fileFolder = os.path.dirname(arg)
	        new_file= os.path.basename(arg)
	        my_command = ["soffice", "--convert-to", "pdf", arg, "--outdir", fileFolder]
	        sp.check_output(my_command, env=my_env)
```

使用起来也很简单，选中需要转换的Office文档，右键，再服务里选择“convertoPDF”即可
![](https://i.imgur.com/9BODeap.png)

## 最后
理论上任何针对文件的Terminal命令都可以通过制作LaunchBar动作的方式将其操作简化。

Libreoffice支持的文件格式转换还有很多，除了PDF之外还有epub/html等等，Libreoffice的其他功能也很强大，有兴趣的可以自行研究。

**本文借鉴了Minja在Power+中的一篇文章《通吃常用格式，用 LaunchBar 快速无损压缩图片 工作日志》*





