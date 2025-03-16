#在使用deepstream-tensorRT加速的问题
1.首先就是模型转化的问题，这个找到trtexec相应的路劲，使用trtexec将onnx转化成engine模型

由于电脑上更好测试模型，并且转化也不用占用板子的存储（我的jetson nano磁盘占用了92%），所以onnx模型是在电脑上从pt模型转化的，直接在jetson nano上转化成engine模型的话，很容易遇到问题：维度不兼容导致转化失败，或者onnx模型是INT64,但是transform最高只支持INT32，还有动态维度问题，先在windows下转化成INT32，在转化的过程中，用netorn可视化我模型之后。参数都是FP32，但是还是有报错，原因是其不能反映全部的参数，因此需要重新用pt模型转化，

2.运用deepstream+tensorRT进行推理，首先是yolo，安装依赖项出现问题： gitpython>=3.1.30 可能是版本太高，因此下载3.1.20版本sudo pip3 install gitpython==3.1.20

3.安装deepstream时候，一般pip安装不了，手动去NVIDIA的官网下载https://github.com/marcoslucianops/DeepStream-Yolo
，一直往下翻，找到对应的版本，一般就是DeepStream 5.1 on Jetson platform，点击NVIDIA DeepStream SDK 5.1，之后登录就可以了,之后将下载好的文件传输给nano

我出现了相应的关于tensor lib7的报错，原因是版本下错了，我的JetPack 版本是 4.6.6（R32.7.6），这个版本的默认 TensorRT 版本是 8.2.1，所以下载deepstream 6.0 for jetson nano download.deb，以下是查看版本的代码：

4.检查版本的代码
输入 head -n 1 /etc/nv_tegra_release检查jetpack版本
输入dpkg -l | grep nvinfer
    dpkg -l | grep TensorRT
    dpkg -l | grep cuda检查cuda，tensorRT版本



