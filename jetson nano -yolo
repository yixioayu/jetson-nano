#The code that only uses ONNX acceleration can only run at a speed of 2-3 frames per second. It will even freeze sometimes. However, at least it can open the CSI camera.
import cv2
import os
import numpy as np
import onnxruntime as ort

# 设置gstreamer管道参数
def gstreamer_pipeline(
    capture_width=640, #摄像头预捕获的图像宽度
    capture_height=640, #摄像头预捕获的图像高度
    display_width=640, #窗口显示的图像宽度
    display_height=640, #窗口显示的图像高度
    framerate=20,       #捕获帧率
    flip_method=0,      #是否旋转图像
):
    return (
        "nvarguscamerasrc ! "
        "video/x-raw(memory:NVMM), "
        "width=(int)%d, height=(int)%d, "
        "format=(string)NV12, framerate=(fraction)%d/1 ! "
        "nvvidconv flip-method=%d ! "
        "video/x-raw, width=(int)%d, height=(int)%d, format=(string)BGRx ! "
        "videoconvert ! "
        "video/x-raw, format=(string)BGR ! appsink"
        % (
            capture_width,
            capture_height,
            framerate,
            flip_method,
            display_width,
            display_height,
        )
    )

if __name__ == "__main__":

    # 类别映射
    class_names = [
        '电池', '药片/药板', '瓶子', '金属瓶子', '土豆块', '胡萝卜块', 
        '白萝卜块', '砖块', '鹅卵石', '纸杯'
    ]

    # 加载 ONNX 模型
    model_path = "best.onnx"  # 替换为你的 ONNX 模型路径
    session = ort.InferenceSession(model_path)

    # 获取输入和输出名称
    input_name = session.get_inputs()[0].name
    output_name = session.get_outputs()[0].name

    # 获取模型的输入尺寸
    onnx_input_shape = session.get_inputs()[0].shape  # (batch, channels, height, width)
    input_h, input_w = onnx_input_shape[2], onnx_input_shape[3]
    input_shape = (640, 640)
    capture_width = 640
    capture_height = 640
    display_width = 320
    display_height = 320

    framerate = 20			# 帧数
    flip_method = 0			# 方向

    # 创建管道
    print(gstreamer_pipeline(capture_width,capture_height,display_width,display_height,framerate,flip_method))

    # 打开摄像头
    cap = cv2.VideoCapture(gstreamer_pipeline(),cv2.CAP_GSTREAMER)
    #cap=cv2.VideoCapture(0)

    while True:
        ret, frame = cap.read()
        if not ret:
            print("无法读取帧")
            break

        # 预处理
        print("leixingshi",type(input_shape))
        resized_frame = cv2.resize(frame, input_shape)
        input_data = resized_frame.astype(np.float32) / 255.0
        input_data = np.transpose(input_data, (2, 0, 1))
        input_data = np.expand_dims(input_data, axis=0)

        # 运行推理
        outputs = session.run([output_name], {input_name: input_data})

        # 解析输出
        detections = outputs[0]  # 获取检测结果

        # 处理检测结果
        for detection in detections:
            if len(detection) < 6:  # 如果数据不完整，则跳过
                continue

            x1, y1, x2, y2, confidence, class_id = detection[:6]

            # 过滤低置信度的检测结果
            if (confidence > 0.5).any():
                # 映射类别ID到类别名称
                class_name = class_names[int(class_id)]

                # 获取图像尺寸，进行坐标缩放
                scale_x = frame.shape[1] / input_shape[0]
                scale_y = frame.shape[0] / input_shape[1]
                x1 = int(x1 * scale_x)
                y1 = int(y1 * scale_y)
                x2 = int(x2 * scale_x)
                y2 = int(y2 * scale_y)

                # 绘制边界框和标签
                cv2.rectangle(frame, (x1, y1), (x2, y2), (0, 255, 0), 2)
                cv2.putText(frame, f"{class_name}: {confidence:.2f}", (x1, y1 - 10),
                cv2.FONT_HERSHEY_SIMPLEX, 0.9, (0, 255, 0), 2)

        # 显示结果
        cv2.imshow("YOLO Garbage Detection", frame)
        # 按 "q" 键退出
        if cv2.waitKey(1) & 0xFF == ord('q'):
            break

    cap.release()
    cv2.destroyAllWindows()
