---
layout:     post 
title:      "篮球投篮识别及计分系统"
subtitle:   "basketball2020"
date:       2020-02-02
author:     "Jadon"
URL: ""
image:      "/img/"
---

> “”

篮球投篮识别及计分系统

摘要：
　　篮球爱好者日常训练需要用到投篮命中率统计工具，但当前都只能自己手工记录，非常不方便。本项目一款可以运行在手机上的投篮识别软件，篮球爱好者随身携带手机、三脚架，放在篮球场馆侧方，自动识别投篮并计分。经过大量的实验和优化之后可以基本准确地自动识别投篮的进与不进。对于想要提高自己能力的篮球爱好者来说非常方便，更高效地使用数据来改善篮球爱好者们的篮球水平。

问题：
　　　我们国家高级别篮球比赛会有专业分析人士分析比赛，但对于大量的篮球爱好者，现在来说却没有一款适用于日常训练用的软件系统。其中关键一项是统计球员们命中率，没有易用的工具，就无法方便地提升篮球爱好者的能力且给予球员们一个反馈。
　　　现有的类似系统通常用到MATLAB等专业软件，而且还需要特定的硬件，很难普及。

思路：
　　　识别投篮的进与不进需要有识别的对象，如篮筐和篮球。这款软件主要识别的是篮筐中的变化。如果有球进入已经识别到的区域，那么程序便会显示“ball in”的标示并统计总分。这款程序需要用到一个叫做opencv的开源计算机视觉库，先将视频导入，然后使用opencv画出需要识别的区域，观察该区域的变化，识别篮球是否进筐。
　　　在设计中要考虑用手机作为平台进行图像识别，软件要简单，不能太耗资源。运行时手机要能独立工作。
。


一、系统设计

图1：系统设计的思维导图

图2：系统分解
篮球爱好者随身携带手机、三脚架，放在篮球场馆侧方，向着篮筐方向放置
1、三脚架用于避免视频抖动。
2、手机（摄像头+屏幕+视频识别软件）组成视频识别和计分功能

二、篮球进筐识别的程序设计
思路：1）标记篮筐的矩形区域rcHoop；2）备份视频第一帧的篮筐区域hoopBack；3）读取当前帧的篮筐区域hoopCurrent；4）比较备份和当前区域，得出hoopDiff，算出均值m；5）用m值来判断是否用篮球进入篮筐区域。
表1：识别篮球是否进入篮筐的程序
#include <iostream>
#include <opencv2/imgproc.hpp>
#include <opencv2/highgui.hpp>

using namespace std;
using namespace cv;

int main(int argc, char** argv)
{
	cv::VideoCapture cap1("C:\\201905.MP4");
	cv::Mat frame;
	cap1.read(frame);
	Rect rcHoop = cv::selectROI("read", frame, true);
	cv::Mat hoopBack = frame(rcHoop).clone();     //backup
	cv::Mat hoopDiff;
	while (cap1.read(frame))
	{
		cv::Mat hoopCurrent = frame(rcHoop).clone();  //current
		cv::absdiff(hoopBack, hoopCurrent, hoopDiff);   //difference
		int dilation_size = 2;
		cv::Mat element = getStructuringElement(MORPH_RECT, Size(2 * dilation_size + 1, 2 * dilation_size + 1), Point(dilation_size, dilation_size));
		cv::erode(hoopDiff, hoopDiff, element);  //腐蚀操作
		threshold(hoopDiff, hoopDiff, 5, 255, CV_THRESH_BINARY);

		cv::Mat gray;
		cvtColor(hoopDiff, gray, CV_RGB2GRAY);
		double m = mean(gray)[0];      //m value of difference

		cout << m << endl;
		if (m > 20)
		{
			putText(frame, "Ball in!!!", Point(40, 40),
				CV_FONT_HERSHEY_COMPLEX, 1, CV_RGB(0, 255, 0), 2);
		}

		matDiff.copyTo(frame(rcBallIn));
		cv::rectangle(frame, rcBallIn, CV_RGB(0, 255, 0), 2);
		cv::imshow("read", frame);
		cv::waitKey(30);
		cap1.read(frame);
	}
} // main

absdiff()、erode()等都是opencv开源库中的函数，用来备份和当前区域图像的差异。
实际效果如下：
?进球后可以显示“ball in！！！”字样。
?球在球筐上方和下方，都没有出现“ball in!!!”字样。

图3：篮球进筐的分解图

四、在手机上实现“投篮识别”

图7：手机应用的功能分布
1、按钮
　　　Start按钮可以打开手机摄像头，按照30帧/秒获取视频，显示到手机屏幕上。每按一次Start按钮还可以将篮筐识别区域保存到hoopBack中。
　　　Stop按钮按一下可以冻结屏幕图像，再按一下恢复视频显示
2、篮筐识别区域
　　　该区域的视频图像对应着hoopCurrent，每帧都会计算出m值和m2值。然后判断篮球是否进筐
3、计分显示
　　　篮球进筐被识别后，手机上方的计分就+1。
4、软件运行信息
　　　为了方便软件调试，在手机的左下方有：hoopBack、hoopCurrent、hoopDiff三个矩形图形。还有一条m值曲线，可以看到实时的软件运行状况。

五、总结
　　　这款系统在试验过后可以基本准确地识别篮筐以及进球与否。
　　　1）第一次优化，误判率从800%降低到60%~100%
　　　2）第二次优化，误判率从60%~100%降低到49%
　　　3）第三次优化，误判率从49%降低到17%
　　　在设计中要考虑用手机作为平台进行图像识别，软件简单，可独立运行。篮球爱好者利用随身携带的手机配合三脚架，高效地使用统计自己投篮数据，从而改善篮球爱好者们的篮球水平。
　　　下一步的优化方向主要有两个：1）对场地中的篮板、罚球区、三分线进行识别，可以区分两分和三分的进球；2）加上给程序的自主训练，实现机器学习。



参考文献：
Instant OpenCV for iOS (English Edition), Alexander Shishkov、 Kirill Kornyakov
iOS Application Development with OpenCV 3 (English Edition), Joseph Howse
https://yq.aliyun.com/articles/64975 阿里云人工智能识别篮球动作视频
https://blog.csdn.net/qq_38604769/article/details/79305879 opencv3.4与vs2017环境搭配