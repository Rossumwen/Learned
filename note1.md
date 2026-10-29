# 知识点&踩坑记录
## 一、知识点
### 1\. 图像表示

图像以**二维数字矩阵**形式存储。

像素：图像最小单位。

分辨率：整张图总像素数量。

通道数：单像素的数据组数。

位深：单通道的比特位数。

灰度图：**单通道、只存亮度、体积小**。

彩色图：**多通道、存色彩信息、占用空间大**。

### 2\. 图像预处理（导航场景）

- 灰度化：彩色转灰度，降低计算量，用于提取道路边线

- 二值化：阈值分割纯黑白，识别路标、引导线

- ROI：截取感兴趣区域，只保留路面，过滤天空、远景干扰

- 缩放：统一图像尺寸，适配模型输入

### 3\. HSV颜色空间

H（色相）、S（饱和度）、V（明度）。

RGB三通道互相耦合，**受光照影响极大**。

HSV将**颜色与亮度分离**，H通道独立控制颜色，抗光照强，适合识别彩色路标、赛道线条。

### 4\. 滤波与Canny边缘检测（导航）

- 高斯模糊：图像平滑降噪，去除细碎噪点，让车道线边缘更干净

- 中值滤波：去除椒盐噪声，消除镜头灰尘、雨点白点黑点

用途：提取车道线、道路边界，实现导航循迹。

### 5\. vector 与 原生数组

vector：动态扩容、自动释放内存，适合数据长度不确定。

数组：长度固定、速度快，适合数据量固定场景。

### 6\. 浅拷贝、深拷贝

浅拷贝：只复制指针地址，多个对象共用同一块内存，容易重复释放崩溃。

深拷贝：开辟新内存、复制内容，对象完全独立。

**类内有动态指针成员必须手动深拷贝**。

### 7\. const

- 修饰变量：值不可修改

- `const T*`：指向内容只读

- `T* const`：指针地址只读

- 修饰成员函数：不能修改类成员变量

### 8\. static

- 静态成员变量：属于类，所有对象共享

- 静态成员函数：无this指针，只能访问静态成员

- 局部静态变量：只初始化一次，程序结束销毁

### 9\. Git 基础命令

- clone：拉取远程仓库到本地

- commit：本地保存修改

- push：上传代码到远程GitHub

### 10\. CMake

跨平台编译工具，管理多文件项目与第三方库。

- `cmake_minimum_required`：指定最低编译版本

- `add_executable`：生成可执行文件

- `target_link_libraries`：链接OpenCV等第三方库
### 11\. OpenCV

- Mat:存放图像的矩阵类型
- imread:读取图像
- imshow:显示
例：
```
string path = "photo/test.png";
Mat img = imread(path);
imshow("image",img);
```

- waitKey():等待
- VideoCapture:视频捕获
例：（获取摄像头视频捕获）
```
VideoCapture cap(0);
Mat img;
while (true){
   cap.read(img);
   imshow("image",img);
   waitKey(1);
}
```
- cvtColor(原图,灰度图,COLOR_BGR2GRAY):灰度转换
- GaussianBlur(原图,高斯图,Size(行,列),x,y):高斯模糊
- Canny(原图,canny图,阈值1,阈值2):边缘检测
- Mat kernel = getStructuringElement(MORPH_RECT,Size(长,宽));
  dilate(原图,膨胀图,kernel):膨胀

---

## 二、个人实操踩坑记录

### 坑1：外接移动硬盘（/media）编译OpenCV必崩

NTFS文件系统**不支持Linux内存映射**

现象：代码无报错、编译成功、运行**段错误（核心已转储）**

解决方法：**所有OpenCV项目必须放在家目录 \~/ 下**，不能放外接盘。

### 坑2：Wayland桌面 imshow 窗口直接崩溃

Ubuntu新版Wayland桌面与OpenCV窗口不兼容

现象：一运行imshow直接闪退、段错误

临时解决方案：

```Bash
export QT_QPA_PLATFORM=xcb
./程序名
```

终极方案：改用 **图片保存模式 imwrite**，不弹窗、百分百稳定。

### 坑3：图片读取失败、程序空白退出

错误原因：

1. 运行在build文件夹，图片放在上级目录

2. 相对路径失效

解决：

1. `cp ../test.jpg ./`

2. 或直接写绝对路径读取图片

### 坑4：CMake 编译成功但无 OpenCV 链接

报错特征：能编译但运行图像函数异常

原因：没写：

```CMake
find_package(OpenCV REQUIRED)
target_link_libraries(...)
```

解决：固定模板写全OpenCV依赖。

### 坑5：RGB与BGR搞混导致颜色完全错误

OpenCV **imread默认BGR**

错误写法：`COLOR_RGB2HSV`

正确写法：`COLOR_BGR2HSV`

### 坑6：VSCode OpenCV 头文件红波浪

Ubuntu OpenCV4 真实路径：

```Plain Text
/usr/include/opencv4
```

解决头文件写法：

```C++
#include <opencv4/opencv2/opencv.hpp>
```

---

## 四、实操最终稳定方案

1. 所有视觉项目**必须放在家目录**

2. 禁止依赖imshow弹窗，全部使用imwrite保存结果图

3. CMake固定模板链接OpenCV

4. 图片统一复制到build目录运行

5. 每次编译前：`rm -rf build && 重新cmake make`
