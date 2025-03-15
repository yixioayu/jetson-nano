#在使用deepstream-tensorRT加速的问题
1.首先就是模型转化的问题，这个找到trtexec相应的路劲，使用trtexec将onnx转化成engine模型

我的onnx模型是在电脑上从pt模型转化的，直接在jetson nano上转化成engine模型的话，很容易遇到问题：维度不兼容导致转化失败，或者onnx模型是INT64,但是transform最高只支持INT32，还有动态维度问题，因此我放弃了从主机上直接复制模型，选择在jetson nano上从头开始

2.运用deepstream+tensorRT进行推理，首先是yolo，安装依赖项出现问题： gitpython>=3.1.30 可能是版本太高，因此下载3.1.20版本sudo pip3 install gitpython==3.1.20

3.安装deepstream时候，一般pip安装不了，手动去NVIDIA的官网下载https://github.com/marcoslucianops/DeepStream-Yolo
，一直往下翻，找到对应的版本，一般就是DeepStream 5.1 on Jetson platform，点击NVIDIA DeepStream SDK 5.1，之后登录就可以了

