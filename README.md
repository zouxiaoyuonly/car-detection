# cars-detection
基于对YOLOv2的迁移学习构建的神经网络模型

训练集来自于drive.ai

使用模型前后图片效果见sample_input.jpg sample_output.jpg

以下对项目实现原理做一个简要介绍
----------------------------------------------------------------------------------------------------------------------------------------
这是我在coursera的深度学习课程上完成的一个项目的源代码，属于一个自动驾驶项目的一部分，用于检测道路上的车辆及其他障碍物

数据的采集是通过汽车前置摄像头拍摄，输入数据是一系列维度为（608,608,3）的RGB格式图片

输出是一个四维向量（19,19,5,85）的向量，最后一维结构为（p,x,y,h,w,c1,c2……c80）：

    p表示图片上检测出目标物体的概率，取值为（0,1）

    x,y表示物体中心点坐标(这里需要注意的是这个坐标表示的是每个网格内的坐标而不是整个图片的）

    h,w表示用于标记物体的方格的高度和宽度

    c1-c80表示可以识别的80种物体类型的概率，取值在（0,1），可以检测的物体类型在coco_classes.txt中

这一个维度用于标记图片上识别出的一个物体，也就是画出一个格子

倒数第二个维度表示5个anchor box，也就是可以同时在一个单元格中检测五个物体

前两个维度表示输入图片被切割成19x19个网格，就像围棋棋盘一样，每个单元格分别检测物体





可以看出来，在一张图片里最多可以标记出19*19*5=1805个格子，对于一个608x608的图片，显然太多了，当中包括很多错误的标记，所以通过设置threshold值定义一个长度为80的Mask用于过滤，

对于每个anchor box：
    将Mask与p*[c1……c80]比较，仅保留概率大于threshold的格子（代码里设定的threshold值为0.6)
    



另一个关键问题在于，存在同一个物体被多次检测的情况，即多个格子标记的是同一个物体。

为了解决这个问题，可以定义一个过滤器，这里使用的是Non-max suppression（NMS）算法，通过定义Iou（两个格子的交集除以并集）函数，确定

Iou_threshold 作为阈值，具体步骤为

    1.选取从上一步得到的格子中概率最高的
    
    2.计算它与其他格子的Iou，去掉所有大于Iou_threshold的格子
    
    3.重复1，直到没有概率比当前格子更小的
   
    
通过调整以上两个threshold值,能够在精确度和漏检率中取得平衡
