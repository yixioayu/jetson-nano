# jetson-nano视觉任务的模型，摄像头问题
## 使用jetson nano遇到的问题
**1.默认是python3.6.9和python2.7，可以设置成默认的3。6版本**
**2.在使用摄像头的时候，linux默认是用v4l2进行传输，会发现只能显示用户的屏幕界面，或者绿屏，可能因为没有办法兼容CSI的10bit色深的输出，而默认的是不支持GStreamer管道的
用cv2.getBuildInformation()发现GStreamer显示的是关闭的，那就需要重新编译opencv，官方支持的opencv可以是4.5版本的,这时候需要重新编译**
**3.onnx模型会不兼容，信的onnx的模型是IR 9和opset 19，需要降到8和15**

*降低IR版本如下*
import onnx

model_path = "your_model.onnx"
model = onnx.load(model_path)

model.ir_version = 8

onnx.save(model, "your_model_ir8.onnx")

print("Successfully downgraded ONNX IR version to 8")

*降低opset版本如下*
import onnx
from onnx import version_converter

model_path = r"your-onnx-model/path"
output_path = r"your/path"

model = onnx.load(model_path)

converted_model = version_converter.convert_version(model, 15)

onnx.save(converted_model, output_path)

print(f"成功降级到 Opset 15: {output_path}")

**4.重新编译opencv**
找到opencv的文件，使用apt，pip也可以，但是我害怕会有残留，因此选择强制删除sudo find / -name '*opencv*'
sudo rm -rf /usr/lib/aarch64-linux-gnu/*opencv*
sudo rm -rf /usr/local/lib/*opencv*

sudo rm -rf /usr/include/opencv4/
sudo rm -rf /usr/local/include/opencv4/

sudo find / -name 'cv2.so'  
sudo rm -rf /usr/lib/python2.7/dist-packages/cv2/
sudo rm -rf /usr/lib/python3/dist-packages/cv2/
sudo rm -rf /usr/local/lib/python3.*/dist-packages/cv2/

sudo rm -rf /usr/lib/pkgconfig/opencv*
sudo rm -rf /usr/local/lib/pkgconfig/opencv*

*安装编译的依赖*
sudo apt-get update && sudo apt-get upgrade -y
sudo apt-get install -y cmake g++ wget unzip
sudo apt-get install -y libgtk2.0-dev pkg-config libavcodec-dev libavformat-dev libswscale-dev
sudo apt-get install -y python3-dev python3-numpy python3-pip
sudo apt-get install -y libtbb2 libtbb-dev libdc1394-22-dev
sudo apt-get install -y libjpeg-dev libpng-dev libtiff-dev
sudo apt-get install -y libv4l-dev v4l-utils
sudo apt-get install -y libxvidcore-dev libx264-dev libgtk-3-dev
sudo apt-get install -y libtesseract-dev libatlas-base-dev gfortran

*之后就是下载源码，目前好像没有什么镜像可以用，github用代理上*
git clone https://github.com/opencv/opencv.git
git clone https://github.com/opencv/opencv_contrib.git
cd opencv
mkdir build && cd build
*配置Cmake*
cmake -D CMAKE_BUILD_TYPE=RELEASE \
      -D CMAKE_INSTALL_PREFIX=/usr/local \
      -D OPENCV_EXTRA_MODULES_PATH=~/opencv_contrib/modules \
      -D WITH_GSTREAMER=ON \
      -D WITH_CUDA=ON \
      -D ENABLE_NEON=ON \
      -D ENABLE_TBB=ON \
      -D BUILD_TESTS=OFF \
      -D BUILD_EXAMPLES=OFF \
      -D WITH_V4L=ON \
      -D WITH_QT=OFF \
      -D WITH_OPENGL=ON ..

*安装和编译*
make -j$(nproc)  nproc是用所有的cpu可以选择3，就是慢一点
sudo make install
sudo ldconfig

编译后的报错，检查相关的依赖是不是齐全，我遇到了编译失败之后，再编译一次，之后出现了相应的报错，解决：https://blog.csdn.net/weixin_52402390/article/details/122341561
https://blog.csdn.net/layra_liu/article/details/124032893
第一个报错是：opencvlib408_ml等相关408的库找不到，原因是环境变量配置的问题，解决方法是
sudo -i
vim /etc/ld.so.conf.d/opencv.conf
在这个文件下输入/usr/local/lib就可以解决了，但是每次重启好像都需要输入sudo -i获取管理员权限

第二个报错是：非法指令 (核心已转储)
这个的原因可能是numpy板子自带了一个版本，自己再下一个版本导致出错了，短暂解决办法就是直接输入下面这个指令
export OPENBLAS_CORETYPE=ARMV8
更多的原因就可以看第二个链接：https://blog.csdn.net/layra_liu/article/details/124032893

**5.一点小错误**
报错resized_frame = cv2.resize(frame,input_shape)   TpyeEooro: an integer is required (got type str)
明明他就是整数，但是却显示的是字符串，强制转化也不行，这直接手动修改就可以
