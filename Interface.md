
# 电子眼接口定义说明文档 v 0.1

## 宏定义
### 最大检测框数目
```
int DKMAXBOXNUM =30; // 最多可检测对象数目
int DKMINFACEREGISTERIMGNUM = 5; // 人脸注册时最少需采集人脸数目
int DKMINOBJREGISTERIMGNUM = 5; // 物体注册时最少需采集物体数目
```
### 物体类别
```
int DKONONE = 0;
int DKOTEXT = 1;
int DKOFACE = 2;
int DKOGESTRUE = 3;
int DKOBANKNOTE = 4;
int DKOBANKNOTE_RMB1 = 41;
int DKOBANKNOTE_RMB5 = 42;
int DKOBANKNOTE_RMB10 = 43;
int DKOBANKNOTE_RMB20 = 44;
int DKOBANKNOTE_RMB50 = 45;
int DKOBANKNOTE_RMB100 = 46;
```
### 颜色类别
```
int DKOCOLOR_WHITE = 80;
int DKOCOLOR_GRAY = 81;
int DKOCOLOR_BLACK = 82;
int DKOCOLOR_RED = 83;
int DKOCOLOR_YELLOW = 84;
int DKOCOLOR_GREEN = 85;
int DKOCOLOR_BLUE = 86;
int DKOCOLOR_PURPLE = 87;
int DKOCOLOR_TAN = 88;
int DKOCOLOR_BROWN = 89;
int DKOCOLOR_SYAN = 90;
int DKOCOLOR_PINK = 91;
```
### 错误码
```
int DKERROR_INIT = 10001; // 初始化错误，待细分类
int DKERROR_DETECTION = 10010; // 检测过程出现错误
```

## 结构体定义
### 检测框
```
typedef struct
{
    // 顺时针排列
    int x1; // 偏左上角横坐标
    int y1; // 偏左上角纵坐标
    int x2; // 偏右上角横坐标
    int y2; // 偏右上角纵坐标
    int x3; // 偏右下角横坐标
    int y3; // 偏右下角纵坐标
    int x4; // 偏左下角横坐标
    int y4; // 偏左下角纵坐标
}DKSBox;
```
### 单检测结果
```
typedef struct
{
    int tag; // 标签，与宏定义对应
    float confidence; // 置信度
    DKSBox box;
}DKSSingleDetectionRes;
```
### 多检测结果
```
typedef struct
{
    int num; // 当前检测出的物体总数目
    DKSSingleDetectionRes boxes[DKMAXBOXNUM];
}DKSMultiDetectionRes;
```
### 多类检测参数
```
typedef struct
{
    int undefined;
}DKSMultiDetectionParam;
```
### 场景分类参数
```
typedef struct
{
    int undefined;
}DKSSceneClassificationParam;
```
### 人脸识别参数
```
typedef struct
{
    float threshold; // 相似度阈值，相似度超过该值认为不能认出该人脸
}DKSFaceRecognizationParam;
```
### 人脸注册参数
```
typedef struct
{
    int undefined;
}DKSFaceRegisterParam;
```
### 检测框文字识别参数
```
typedef struct
{
    int undefined;
}DKSBoxTextRecognizationParam;
```
### 文档识别参数
```
typedef struct
{
    int undefined;
}DKSTextRecognizationParam;
```
### 物体识别参数
```
typedef struct
{
    float threshold; // 相似度阈值，相似度超过该值认为不能认出该物体
}DKSObjectRecognizationParam;
```
### 物体注册参数
```
typedef struct
{
    int undefined;
}DKSObjectRegisterParam;
```
### 颜色识别参数
```
typedef struct
{
    int undefined;
}DKSColorRecognizationParam;
```
 
## 函数定义
### 多类检测
```
// 说明：手势、纸币、人脸、文字区域的检测，使用NNIE加速
// 初始化，用于加载NNIE模型
void DKMultiClassDetectionInit();
// 运行NNIE执行检测，得到检测结果
DKSMultiDetectionRes * DKMultiClassDetectionProcess(char * yuvfilename, int iWidth, int iHeight, DKSMultiDetectionParam param);
// 释放NNIE资源
void DKMultiClassDetectionEnd();
```
### 场景分类
```
// 说明：区分普通场景与书籍页面场景
// 初始化，用以加载ncnn场景分类模型
void DKSceneClassificationInit();
// 运行ncnn场景分类，得到分类结果
int DKSceneClassificationProcess(char * yuvfilename, int iWidth, int iHeight, DKSSceneClassificationParam param);
// 释放ncnn场景分类结构资源
void DKSceneClassificationEnd();
```
### 检测模式分类
```
// 说明：区分普通场景中的手势/纸币/人脸/文字区域，用以判断接下来做何种识别。
// 运行ncnn场景分类，得到分类结果。返回分类类别，例如DKOFACE。index为从DKSMultiDetectionRes选择出的boxes序号，下面的步骤即针对该框进行识别。
int DKSelectMode(DKSMultiDetectionRes boxes, int &index);
```
### 人脸识别
```
// 说明：从已注册的人脸中识别对应的人脸
// 初始化，连接sqlite，获取人脸库中各个人脸的特征
void DKFaceRecognizationInit();
// 运行knn人脸识别，得到识别结果，如果没有识别出或相似度大于某一阈值（在识别参数中定义），则输出null，否则输出识别出的人的index。
int DKFaceRecognizationProcess(char * yuvfilename, int iWidth, int iHeight, DKSMultiDetectionRes boxes, DKSFaceRecognizationParam param);
// 释放人脸识别资源
void DKFaceRecognizationEnd();
```
### 人脸注册
```
// 说明：录入人脸，在学习人脸阶段，获取至少DKMINFACEREGISTERIMGNUM张同一人脸图片
// 初始化，连接sqlite，准备写入人脸特征和语音，初始化学习图片次数为0
void DKFaceRegisterInit();
// 根据检测到的人脸结果计算特征
int DKFaceRegisterProcess(char * yuvfilename, int iWidth, int iHeight, DKSMultiDetectionRes boxes, DKSFaceRegisterParam param);
// 将计算好的特征存入sqlite中（flag为1），或取消学习人脸（flag为0）
void DKFaceRegisterEnd(int flag);
```
### 检测框文字识别
```
// 说明：根据识别出的文本框对里面的文字进行识别
// 初始化，用以加载ncnn lstm文字识别模型
void DKBoxTextRecognizationInit();
// 运行ncnn lstm文字识别，输出识别字符串
char * DKBoxTextRecognizationProcess(char * yuvfilename, int iWidth, int iHeight, DKSBox box, DKSBoxTextRecognizationParam param);
// 释放ncnn文字识别结构资源
void DKBoxTextRecognizationEnd();
```
### 文档识别
```
// 说明：对整篇文档调用ABBYY接口进行识别
// ABBYY初始化
void DKTextRecognizationInit();
// ABBYY整篇文档识别，输出识别字符串
char * DKTextRecognizationProcess(char * yuvfilename, int iWidth, int iHeight, DKSTextRecognizationParam param);
// ABBYY资源释放
void DKTextRecognizationEnd();
```
### 物体识别
```
// 说明：从已注册的物体中识别对应的物体
// 初始化，连接sqlite，获取物体库中各个物体的特征
void DKObjectRecognizationInit();
// 计算图像中心区域（1/4原图像大小）特征，并与sqlite中特征进行对比，如果相似度大于某一阈值（在识别参数中定义），则输出null，否则输出识别出的物体的index。
int DKObjectRecognizationProcess(char * yuvfilename, int iWidth, int iHeight, DKSObjectRecognizationParam param, DKSObjectRecognizationParam param);
// 释放物体识别资源
void DKObjectRecognizationEnd();
```
### 物体注册
```
// 说明：录入物体，在物体学习阶段，获取至少DKMINOBJREGISTERIMGNUM张同一物体图片
// 初始化，连接sqlite，准备写入物体特征和语音，初始化学习图片次数为0
void DKObjectRegisterInit();
// 根据图像中间位置（1/4原图像大小）子图像计算特征
char * DKObjectRegisterProcess(char * yuvfilename, int iWidth, int iHeight, DKSObjectRegisterParam param);
// 将计算好的特征存入sqlite中（flag为1），或取消学习人脸（flag为0）
void DKObjectRegisterEnd(int flag);
```
### 颜色识别
```
// 说明：识别某个区域的颜色
// 初始化
void DKColorRecognizationInit();
// 识别图像中间位置或手指指向位置区域，如果该区域纹理平滑则输出与12中基色最相近的颜色序号，否则输出0
int DKColorRecognizationProcess(char * yuvfilename, int iWidth, int iHeight, DKSColorRecognizationParam param);
// 释放资源
void DKColorRecognizationEnd();
```
