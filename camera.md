# Camera

## 自动曝光

自动曝光（Auto Exposure，AE）是相机或其他成像设备中的一项重要功能，它能够根据拍摄场景的光线条件自动调整曝光参数，以获得合适亮度的图像。

### 1 基于灰度均值

![image-20240918195543570](C:\Users\z1002\AppData\Roaming\Typora\typora-user-images\image-20240918195543570.png)

### 2 基于灰度直方图

![image-20240918195626653](C:\Users\z1002\AppData\Roaming\Typora\typora-user-images\image-20240918195626653.png)

- 动态范围定义：

  （ 最大-最小 + 1）/ 256

![image-20240918195727557](C:\Users\z1002\AppData\Roaming\Typora\typora-user-images\image-20240918195727557.png)

由于相机是线性的，曝光时间与得到的灰度值也是线性的

![image-20240918200058841](C:\Users\z1002\AppData\Roaming\Typora\typora-user-images\image-20240918200058841.png)