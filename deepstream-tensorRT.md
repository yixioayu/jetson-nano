#在使用deepstream-tensorRT加速的问题
1.首先就是模型转化的问题，这个找到trtexec相应的路劲，使用trtexec将onnx转化成engine模型

由于电脑上更好测试模型，并且转化也不用占用板子的存储（我的jetson nano磁盘占用了92%），所以onnx模型是在电脑上从pt模型转化的，直接在jetson nano上转化成engine模型的话，很容易遇到问题：维度不兼容导致转化失败，或者onnx模型是INT64,但是transform最高只支持INT32，还有动态维度问题

先在windows下转化成INT32，在转化的过程中，可以用netorn可视化模型之后，观察报错的地方的参数，比如Concat 层的输入维度不兼容，显示的是/model.11/Concat: dimensions not compatible for concatenation，也就是model.11出现了问题，看了一下果然是int64类型
还有一个报错 Error Code 2: Internal Error (Assertion enginePtr != nullptr failed. 那么这个可能就是onnx有int64，如果没有的话，那还可能是我在README.md中提到的，其默认支持的onnx的opset是12版本，而我的是15版本的（jetson nano支持的最高版本），目前最高的已经是18版本（官方最高版本）
        

2.运用deepstream+tensorRT进行推理，首先是yolo，安装依赖项出现问题： gitpython>=3.1.30 可能是版本太高，因此下载3.1.20版本sudo pip3 install gitpython==3.1.20

3.安装deepstream时候，一般pip安装不了，手动去NVIDIA的官网下载https://github.com/marcoslucianops/DeepStream-Yolo
，一直往下翻，找到对应的版本，一般就是DeepStream 5.1 on Jetson platform，点击NVIDIA DeepStream SDK 5.1，之后登录就可以了,之后将下载好的文件传输给nano

我出现了相应的关于tensor lib7的报错，原因是版本下错了，我的JetPack 版本是 4.6.6（R32.7.6），这个版本的默认 TensorRT 版本是 8.2.1，所以下载deepstream 6.0 for jetson nano download.deb，以下是查看版本的代码：

4.检查版本的代码
输入 head -n 1 /etc/nv_tegra_release检查jetpack版本
输入dpkg -l | grep nvinfer
    dpkg -l | grep TensorRT
    dpkg -l | grep cuda检查cuda，tensorRT版本



