# AATFC-Autonomous-Anti-theft-human-following-carrier-
graduation project
## 꼭 구현해야하는 3가지
1. Human following
2. 모터 잠금
3. 캐리어가 바닥과 떨어지는 것 감지 후 사이렌 울리기
```python
#include <ros/ros.h>
#include <sensor_msgs/Image.h>
#include <cv_bridge/cv_bridge.h>
#include <opencv2/opencv.hpp>
#include <image_transport/image_transport.h>

void imageCallback(const sensor_msgs::ImageConstPtr& msg)
{
try {
// 이미지 메시지를 OpenCV 이미지로 변환
cv::Mat image = cv_bridge::toCvShare(msg, "bgr8")->image;

// 이미지 출력
cv::imshow("Camera Image", image);
cv::waitKey(1);
} catch (cv_bridge::Exception& e) {
ROS_ERROR("cv_bridge 예외: %s", e.what());
}
}

int main(int argc, char** argv)
{
ros::init(argc, argv, "image_subscriber");
ros::NodeHandle nh;

// 이미지 구독을 위한 image_transport 사용
image_transport::ImageTransport it(nh);
image_transport::Subscriber sub = it.subscribe("/camera/image", 1, imageCallback);

ROS_INFO("이미지 구독 노드 실행 중...");
ros::spin();

return 0;
}
```

```python
#include <ros/ros.h>
#include <sensor_msgs/Image.h>
#include <cv_bridge/cv_bridge.h>
#include <opencv2/opencv.hpp>
#include <image_transport/image_transport.h>
#include <iostream>

void imageCallback(const sensor_msgs::ImageConstPtr& msg)
{
try {
// ROS 메시지를 OpenCV 이미지로 변환
cv::Mat image = cv_bridge::toCvShare(msg, "bgr8")->image;

// BGR 채널 분리
std::vector<cv::Mat> channels(3);
cv::split(image, channels); // [0] = Blue, [1] = Green, [2] = Red

// 빈 채널 (0으로 초기화)
cv::Mat empty_channel = cv::Mat::zeros(image.rows, image.cols, CV_8UC1);

// 각 채널 강조 컬러 이미지 만들기
cv::Mat red_image, green_image, blue_image;

cv::merge(std::vector<cv::Mat>{empty_channel, empty_channel, channels[2]}, red_image); // Red 강조
cv::merge(std::vector<cv::Mat>{empty_channel, channels[1], empty_channel}, green_image); // Green 강조
cv::merge(std::vector<cv::Mat>{channels[0], empty_channel, empty_channel}, blue_image); // Blue 강조

// Grayscale 변환
cv::Mat gray_image;
cv::cvtColor(image, gray_image, cv::COLOR_BGR2GRAY);

// 이미지 출력
cv::imshow("Red Channel (Emphasized)", red_image);
cv::imshow("Green Channel (Emphasized)", green_image);
cv::imshow("Blue Channel (Emphasized)", blue_image);
cv::imshow("Grayscale Image", gray_image);

cv::waitKey(1);

} catch (cv_bridge::Exception& e) {
ROS_ERROR("cv_bridge 예외: %s", e.what());
}
}

int main(int argc, char** argv)
{
ros::init(argc, argv, "image_channel_color_viewer");
ros::NodeHandle nh;

image_transport::ImageTransport it(nh);
image_transport::Subscriber sub = it.subscribe("/camera/image", 1, imageCallback);

ROS_INFO("R/G/B 강조 및 Grayscale 이미지 보기 노드 실행 중...");
ros::spin();

return 0;
}
```
