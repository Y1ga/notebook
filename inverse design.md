# Inverse Design

## 背景

。。。。

## 发展

![image-20240903163052629](C:\Users\z1002\AppData\Roaming\Typora\typora-user-images\image-20240903163052629.png)

> Ji et al. Light: Science & Applications (2023)12:169

现有的逆向设计有4种

- FEM/FDTD：慢

- 拓扑优化：少，基于优化

- 深度学习：主流

- 物理信息神经网络（PINNs）：深度学习plus版

  **一般物理定律的先验知识在[神经网络](https://en.wikipedia.org/wiki/Neural_network)（NN）的训练中充当[正则化](https://en.wikipedia.org/wiki/Regularization_(mathematics))代理，限制可接受解的空间，提高函数逼近的正确性**。通过这种方式，将**先验信息嵌入到神经网络中可以增强可用数据的信息内容**，促进学习算法捕获正确的解决方案，并且即使在训练示例数量较少的情况下也能很好地概括。

  

### 2019

![image-20240903160803737](C:\Users\z1002\AppData\Roaming\Typora\typora-user-images\image-20240903160803737.png)

![image-20240903160944882](C:\Users\z1002\AppData\Roaming\Typora\typora-user-images\image-20240903160944882.png)

> An, Sensong, et al. "A deep learning approach for objective-driven all-dielectric metasurface design." Acs Photonics 6.12 (2019): 3196-3207.

最经典的`autoencoder`（也属于`end-to-end`的一种）

- `input`：光谱数据真实值 （31 * 1）
- `output`：光谱数据预测值 （31 * 1）
- `encoder`：逆向设计网络 
- `decoder`：正向预测网络

### 2020



![image-20240904144801363](C:\Users\z1002\AppData\Roaming\Typora\typora-user-images\image-20240904144801363.png)

> Vol. 28, No. 8 / 13 April 2020 / Optics Express 11618

1. 不需要大量数据集，一个偏微分方程对应一个数据集

2. 不需要它所预测的逆参数的任何数据，因此属于无监督学习。

3. **损失函数添加额外损失项来最小化偏微分方程及其边界条件的残差**，从而像解决正向问题一样解决高度非线性和分散的逆问题

   因此，PINNs 在解决通常用现有数学公式难以处理的不适定逆问题方面特别有效。

### 2021

![image-20240904145017391](C:\Users\z1002\AppData\Roaming\Typora\typora-user-images\image-20240904145017391.png)

> Nanophotonics 2021; 10(18): 4497–4509

问题：

**fab公差导致理论超表面与实际超表面指标差的较多**

解决：
将fab公差考虑入深度学习模型，具体地：加入了**fab公差模型去生成二维超表面参数**

![image-20240904145010912](C:\Users\z1002\AppData\Roaming\Typora\typora-user-images\image-20240904145010912.png)

### 2022

#### 1 将物理特性模型融入深度学习中

![image-20240904145140406](C:\Users\z1002\AppData\Roaming\Typora\typora-user-images\image-20240904145140406.png)

> Tang, Yingheng, et al. "Physics-informed recurrent neural network for time dynamics in optical resonances." *Nature computational science* 2.3 (2022): 169-178.


本文任务：预测光学谐振器的时域特性

背景：

1. 预测谐振特征的时域信号衰减缓慢，并且在数值模拟和实验测量中需要较长的数据采集时间才能进行准确分析；
2. 快速捕获的短时域信号包含不正确的相应频域表示，从而妨碍了常规谐波分析。因此，需要在数据采集时间和分析精度之间进行权衡。

解决：
深度学习时**不是直接使用大量从耗时的电磁计算或实验中获得的高保真数据来训练模型，而是采用两步多保真训练方法。**

1. **首先使用大量由低保真自由感应衰减（FID）模型生成的合成数据对模型进行预训练**，这减少了网络参数的搜索空间，有利于快速高效学习，缓解了局部最小值问题以实现高精度预测，并使模型的适用性更广泛。
2. **通过使用一小部分（至少比低保真数据小一个数量级）针对特定应用的高保真数据进行迁移学习**，将预训练的模型调整到广泛的共振特征



#### 2 end-to-end的超表面逆向设计，同2019

![image-20240904145502902](C:\Users\z1002\AppData\Roaming\Typora\typora-user-images\image-20240904145502902.png)

> End-to-end metasurface inverse design for single-shot multi-channel imaging
>
> Vol. 30, No. 16 / 1 Aug 2022 / Optics Express 28358

所提出的技术将单层超表面前端与高效的 Tikhonov  重建后端集成在一起，除了灰度传感器之外没有任何额外的光学器件。我们的方法通过自发解复用产生多通道成像：

**超光学前端将不同的通道分离成不同的空间域，其在传感器上的位置通过逆向设计算法最佳地发现。**

我们提出了与标准平版印刷兼容的大面积超表面设计，用于多光谱成像、深度光谱成像和 “一体化” spectro-polarimetric-depth 成像，具有强大的重建性能（10% 误差和 1%  检测器噪声）。与神经网络相比，我们的框架是物理可解释的，不需要大量的训练集。它可以用来重建具有全多波长光谱和偏振文本的任意三维场景

### 2023

#### 1. end-to-end的逆向设计，多考虑了频率、入射角两个参数

![image-20240904145749529](C:\Users\z1002\AppData\Roaming\Typora\typora-user-images\image-20240904145749529.png)

![image-20240904145728732](C:\Users\z1002\AppData\Roaming\Typora\typora-user-images\image-20240904145728732.png)

> Adv. Optical Mater. 2023, 11, 2202130
>
> Pushing the Limits of Metasurface Cloak Using Global Inverse Design

1. 一种全局逆向设计，以突破超表面斗篷的限制，使其具**有自由形状**，**与传统的基于物理知识的方法互补**。
2. 还构建了一个串联神经网络，在超表面斗篷及其电磁响应之间建立双向通道，缓解了不可避免的非唯一性问题，提高了输出精度（大于 93%）。

具体地：

1. 先锁定好一种结构的结构参数，长、宽、高。。。如图1.b
2. 在这个结构上排列圆形超表面，选择范围只有1维共计31个，但使用**single-hot编码总共有16种**，变量是半径(最大是10mm)
3. 网络输入的是31 * 16的一维超表面结构，但是还多了**频率、入射角两个通道**
4. 使用**利用串联神经网络来考虑这些难以捉摸的影响和多重散射**。

### 2024

#### 1. 算法的创新，使用多个子网络合成一个总网络

![image-20240904150348283](C:\Users\z1002\AppData\Roaming\Typora\typora-user-images\image-20240904150348283.png)

> Laser Photonics Rev. 2024, 2400063
>
> Dynamic Inverse Design of Broadband Metasurfaces with Synthetical Neural Networks

1. 逆向设计网络的加法缝合，并且具有很强的可拓展性

2. 光谱不做一维像素化

![image-20240904150414949](C:\Users\z1002\AppData\Roaming\Typora\typora-user-images\image-20240904150414949.png)

R8：表示 8 * 8 矩形
**$R_8^{10}$:表示10 * 10 - 8 * 8 的矩形环**

- a. 传统的，训练好8*8来一个新的10*10就要重新训练

- b. ASNN：训练10 * 10 只需组装8 * 8 和一个R810

- c. 整体架构
  ASNN - INN1
       \- INN2input：100 * 100 * 2（光谱的实部和虚部）
  ASNN output：100 * 100 * 4（两个反射光谱分别喂给INN1和INN2）
  INN1 input：100 * 100 * 2
  INN1 output：8 * 8

  INN2 input： 100 * 100 * 2
  INN2 output：10 * 10 （R810）



![image-20240904150554474](C:\Users\z1002\AppData\Roaming\Typora\typora-user-images\image-20240904150554474.png)

- a. ASNN输入-输出
- e. ASNN由INN1($R_8$) INN2($R_{8}^{10}$) INN3($R_{10}^{14}$)组成
- g. 传统的直接训练R14效果不好(90% vs 75%)
- 不如把它切割成3份$R_8$ 、$R_{8}^{10}$、$R_{10}^{14}$

#### 2. 算法的创新，将物理先验加入自动微分算法

![image-20240904151252821](C:\Users\z1002\AppData\Roaming\Typora\typora-user-images\image-20240904151252821.png)

> Eigendecomposition-free inverse design of  meta-optics devices
>
> Vol. 32, No. 8 / 8 Apr 2024 / Optics Express 13986

设计了一种在逆向设计中的新型自动微分算法，比pytorch自带的快30%

![image-20240904151315098](C:\Users\z1002\AppData\Roaming\Typora\typora-user-images\image-20240904151315098.png)