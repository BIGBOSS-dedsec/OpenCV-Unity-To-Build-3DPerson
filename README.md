# Introduction
This article will introduce how to use **Python** with **OpenCV** image capture, with powerful **Mediapipe** library to achieve ** * human motion detection ** and recognition; The recognition results are synchronized to Unity** * in real time to realize the recognition of the character model's moving body structure in Unity
# Demo
[Demo](https://hackathon2022.juejin.cn/#/works/detail?unique=WJoYomLPg0JOYs8GazDVrw)：https://hackathon2022.juejin.cn/#/works/detail?unique=WJoYomLPg0JOYs8GazDVrw
![在这里插入图片描述](https://img-blog.csdnimg.cn/2ab27c1a2f59499597799e80a3863092.png)
![在这里插入图片描述](https://img-blog.csdnimg.cn/941276c880eb40c59dc4f32a6c326f0a.png)

[CSDN](https://blog.csdn.net/weixin_50679163?type=edu)： https://blog.csdn.net/weixin_50679163?type=edu

# Project environment
**Python			3.7
Mediapipe     0.8.9.1
Numpy		1.21.6
OpenCV-Python 		4.5.5.64
OpenCV-contrib-Python		4.5.5.64**
![在这里插入图片描述](https://img-blog.csdnimg.cn/e49c1c0fa21c4df08ee58e57ce57f8b4.png?x-oss-process=image/watermark,type_d3F5LXplbmhlaQ,shadow_50,text_Q1NETiBAQklHQk9TU3lpZmk=,size_20,color_FFFFFF,t_70,g_se,x_16)

# Body motion capture part
** Body data file **

This part is for us to save the data by reading the characters in the video and calculating the information of each feature point. This information is very important, and then import these action data in untiy
![**5**](https://img-blog.csdnimg.cn/9601413eb240484c9d4a1a4b97aa068c.png)
** Points about physical features **
![**加粗样式**](https://img-blog.csdnimg.cn/e7ab82d41bd84f0abc4bfda7aca356cf.png)
# Core code
** Camera capture part: **

```python
import cv2

cap = cv2.VideoCapture(0)       #OpenCV摄像头调用：0=内置摄像头（笔记本）   1=USB摄像头-1  2=USB摄像头-2

while True:
    success, img = cap.read()
    imgRGB = cv2.cvtColor(img, cv2.COLOR_BGR2RGB)       #cv2图像初始化
    cv2.imshow("HandsImage", img)       #CV2窗体
    cv2.waitKey(1)      #关闭窗体
```
**FPS**

```python
import time

#帧率时间计算
pTime = 0
cTime = 0

while True
cTime = time.time()
    fps = 1 / (cTime - pTime)
    pTime = cTime

    cv2.putText(img, str(int(fps)), (10, 70), cv2.FONT_HERSHEY_PLAIN, 3,
                (255, 0, 255), 3)       #FPS的字号，颜色等设置
```
**Body motion capture:**

```python
while True:
    if bboxInfo:
        lmString = ''
        for lm in lmList:
            lmString += f'{lm[1]},{img.shape[0] - lm[2]},{lm[3]},'
        posList.append(lmString)
```
## Final Code
### Motion.py
```python
import cv2
from cvzone.PoseModule import PoseDetector

# 读取cv来源，也可以调用摄像头使用
cap = cv2.VideoCapture('asoul.mp4')

detector = PoseDetector()
posList = []

while True:
    success, img = cap.read()
    img = detector.findPose(img)
    lmList, bboxInfo = detector.findPosition(img)

    if bboxInfo:
        lmString = ''
        for lm in lmList:
            lmString += f'{lm[1]},{img.shape[0] - lm[2]},{lm[3]},'
        posList.append(lmString)
        
    cv2.imshow("Image", img)
    key = cv2.waitKey(1)
    if key == ord('s'):
        with open("MotionFile.txt", 'w') as f:		# 将动作数据保存下来
            f.writelines(["%s\n" % item for item in posList])
```

![在这里插入图片描述](https://img-blog.csdnimg.cn/73d23740e37543a98a919105a6b30a90.png)
# Unity Part
## Model
In Unity, we need to build a model of the character, here we need a **33 Sphere** for the ** body feature points ** and **33 Cube** for the middle stand

![在这里插入图片描述](https://img-blog.csdnimg.cn/376f3bbc12434204b4f4f67e91589fc6.png)
Line numbers correspond to character model feature points
![在这里插入图片描述](https://img-blog.csdnimg.cn/a89bb24e9f1c4bd2a67ccfcd9900aee6.png)
## Line.cs
Here is the cs file corresponding to each Line to achieve the functions: ** Connect the feature points and the Line together **
```csharp
using System.Collections;
using System.Collections.Generic;
using UnityEngine;

public class LineCode : MonoBehaviour
{

    LineRenderer lineRenderer;

    public Transform origin;
    public Transform destination;

    void Start()
    {
        lineRenderer = GetComponent<LineRenderer>();
        lineRenderer.startWidth = 0.1f;
        lineRenderer.endWidth = 0.1f;
    }
// 连接两个点
    void Update()
    {
        lineRenderer.SetPosition(0, origin.position);
        lineRenderer.SetPosition(1, destination.position);
    }
}
```
## Action.cs
Here is ** read the character action data identified above and saved **, ** and loop each sub-data to each Sphere point ** so that the feature points move with the character action in the video
```csharp
using System.Collections;
using System.Collections.Generic;
using System.Linq;
using UnityEngine;
using System.Threading;

public class AnimationCode : MonoBehaviour
{

    public GameObject[] Body;
    List<string> lines;
    int counter = 0;
    
    void Start()
    {
    	// 读取MotionFile.txt的动作数据文件
        lines = System.IO.File.ReadLines("Assets/MotionFile.txt").ToList();
    }

    void Update()
    {
        string[] points = lines[counter].Split(',');
	
	// 循环遍历到每一个Sphere点
        for (int i =0; i<=32;i++)
        {
            float x = float.Parse(points[0 + (i * 3)]) / 100;
            float y = float.Parse(points[1 + (i * 3)]) / 100;
            float z = float.Parse(points[2 + (i * 3)]) / 300;
            Body[i].transform.localPosition = new Vector3(x, y, z);
        }

        counter += 1;
        if (counter == lines.Count) { counter = 0; }
        Thread.Sleep(30);
    }
}
```

![在这里插入图片描述](https://img-blog.csdnimg.cn/941276c880eb40c59dc4f32a6c326f0a.png)
**Good Luck，Have Fun and Happy Coding！！！**
