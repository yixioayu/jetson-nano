#在使用deepstream-tensorRT加速的问题
首先就是模型转化的问题，这个找到trtexec相应的路劲，使用trtexec将onnx转化成engine模型

由于电脑上更好测试模型，并且转化也不用占用板子的存储（我的jetson nano磁盘占用了92%），所以onnx模型是在电脑上从pt模型转化的，直接在jetson nano上转化成engine模型的话，很容易遇到问题：维度不兼容导致转化失败，或者onnx模型是INT64,但是transform最高只支持INT32，还有动态维度问题

先在windows下转化成INT32，在转化的过程中，可以用netorn可视化模型之后，观察报错的地方的参数，比如Concat 层的输入维度不兼容，显示的是/model.11/Concat: dimensions not compatible for concatenation，也就是model.11出现了问题，看了一下果然是int64类型
还有一个报错 Error Code 2: Internal Error (Assertion enginePtr != nullptr failed. 那么这个可能就是onnx有int64，如果没有的话，那还可能是我在README.md中提到的，其默认支持的onnx的opset是12版本，而我的是15版本的（jetson nano支持的最高版本），目前最高的已经是18版本（官方最高版本），那这种我想的是重新转化了，但是我发现重新转化之后还是会有版本不兼容的问题，那就需要其他方法了，比如参考这篇文章：https://blog.csdn.net/qq_56548850/article/details/124256112?ops_request_misc=%257B%2522request%255Fid%2522%253A%252248c861846b38a13c0deee333bd9dcfb8%2522%252C%2522scm%2522%253A%252220140713.130102334..%2522%257D&request_id=48c861846b38a13c0deee333bd9dcfb8&biz_id=0&utm_medium=distribute.pc_search_result.none-task-blog-2~all~sobaiduend~default-1-124256112-null-null.142^v102^pc_search_result_base2&utm_term=jetson%20nano%20deepstream%E5%AE%89%E8%A3%85&spm=1018.2226.3001.4187 

前三步就是单纯的下载torch&&torchvision&&yolov5，之后用gen_wts.py 脚本转化 yolov5s.pt，中间可能需要加一个-w,不然会报错，下面是安装时遇到的问题 
今天连接不到网络，还有一个另外的报错，下载不好torch&&torchvision，明天再接着弄吧,心脏疼先去吃饭了，现在也就是yolo，deep，ten都弄好了，模型转化差torch，明天应该能把模型弄好。。。。

1.运用deepstream+tensorRT进行推理，首先是yolo，安装依赖项出现问题： gitpython>=3.1.30 可能是版本太高，因此下载3.1.20版本sudo pip3 install gitpython==3.1.20

2.安装deepstream时候，一般pip安装不了，手动去NVIDIA的官网下载https://github.com/marcoslucianops/DeepStream-Yolo
，一直往下翻，找到对应的版本，一般就是DeepStream 5.1 on Jetson platform，点击NVIDIA DeepStream SDK 5.1，之后登录就可以了,之后将下载好的文件传输给nano

我出现了相应的关于tensor lib7的报错，原因是版本下错了，我的JetPack 版本是 4.6.6（R32.7.6），这个版本的默认 TensorRT 版本是 8.2.1，所以下载deepstream 6.0 for jetson nano download.deb，以下是查看版本的代码：

3.检查版本的代码
输入 head -n 1 /etc/nv_tegra_release检查jetpack版本
输入dpkg -l | grep nvinfer
    dpkg -l | grep TensorRT
    dpkg -l | grep cuda检查cuda，tensorRT版本



