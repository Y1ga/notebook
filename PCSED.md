# PCSED

## data

[T, L, H, D]

- T: $Si_3N_4$厚度 [200, 400], stride = 25
- L: Si边长[100, 200], stride = 12.5
- H: Si高度[50, 200], stride = 18.75
- D: $Si_3N_4$边长[300, 400], stride = 12.5

![image-20240306135046734](C:\Users\z1002\AppData\Roaming\Typora\typora-user-images\image-20240306135046734.png)

## Net

- rnet(IDN): input[151, 1] ouput[4, 1]

- fnet(FMN): input[4, 1] output[151, 1]

- tnet(TN): input[151, 1] output[151, 1]

- hsnet(SED): input[151, 1] output[151, 1]

- hybnet(PCSED): input[151, 1] output[151, 1]

## SED-inv

1. SED: 得到滤光片response, $W$
2. respose传给rnet得到structure

### tnet

由于一种response可以对应多种structure，但一种structure只对应一种response，因此rnet为满射，影响收敛

因此训练rnet时的数据集应先做处理

1. 使用FDTD生成原始数据集6500个，input[4, 6500] output[151, 6500]

2. 将fnet串接到rnet成为串联神经网络tnet，input[151, 1] output[151, 1]
   $$
   output[6500, 151] = fnet(rnet(response)[6500, 4])
   $$
   

### 特点

1. SED是根据CAVE数据集寻找最佳的response
2. 然后根据response去从已知的数据库(rnet)里寻找对应的structure

## PCSED

1. PCSED: fnet(structure)得到response，response作为SED第一层卷积核，得到编码后的$y$

$$
y[4, 1] =  response[4, 151]×input[151, 1]
$$

​	此时得到的不再是response而是structure

​	这样少了一层IDN层大大提高滤光片随机性

### fnet

1. 使用FDTD生成原始数据集6500个，input[4, 6500] output[151, 6500]
2. 使用input和output训练fnet，把fnet当作FDTD用

### 特点

1. PCSED根据CAVE数据集寻找最佳的response，这个response由fnet表征即response = fnet(structure)，这样变成寻找最佳的structure，一步到位

## 区别

- SED-inv受限于正则化项smooth_loss的影响（要求光谱曲线越平滑越好，这样才可以被制作出来），导致得到的response是受限制的，编码能力被严重削弱

​	![image-20240305193841364](C:\Users\z1002\AppData\Roaming\Typora\typora-user-images\image-20240305193841364.png)

- PCSED跳过了IDN直接从已有的数据库(fnet)中寻找response，也就是这个response一定存在，这样就不用被受正则化项smooth_loss限制，并且保证一定可以被设计出来

### 对比: SED vs PCSED

![image-20240305194808798](C:\Users\z1002\AppData\Roaming\Typora\typora-user-images\image-20240305194808798.png)

### 正则化项意义

图(a)为不加smooth项，杂乱无章不可能被设计出来；图(b)为加了smooth项，符合正常滤光片response

![image-20240305194619914](C:\Users\z1002\AppData\Roaming\Typora\typora-user-images\image-20240305194619914.png)

# ALFRED

1. pso生成数据集
2. resnet作FDTD用

## ResNet

是一个autoencoder

- input[100, 100, 1], output[100, 100, 1]

![image-20240625153239694](C:\Users\z1002\AppData\Roaming\Typora\typora-user-images\image-20240625153239694.png)

### Encoder

是预训练好的FDTD网络

- Input [100, 100, 1]，超表面结构
- Output[100, 1]，超表面透射率

### Decoder

- Input [100, 1]，超表面透射率
- Output [100, 100, 1]，超表面结构

[100, 1] - 3 * 3 * 64 - reshape[3, 3, 64] - Conv2D(3, 3, 32) - Conv2D(3, 3, 16) - Conv2D(3, 3, 8) - Conv2D(3, 3, 4) - Conv2D(3, 3, 1)

![image-20240625175142782](C:\Users\z1002\AppData\Roaming\Typora\typora-user-images\image-20240625175142782.png)



## Predict











# My Project

# Metasurface逆向设计解读

1. 使用一个基本的周期性模板，使用FDTD生成数据  

   数据参考：input: structure(l1, l2, Px, Py) output: transmission

   ![image-20240311141033766](C:\Users\z1002\AppData\Roaming\Typora\typora-user-images\image-20240311141033766.png)

2. 使用生成的数据训练 fnet替代FDTD

3. 使用fnet和数据训练tnet(rnet)，完成逆向设计

### fnet

1. 使用FDTD生成原始数据集48001个，input[16, 48001] output[302, 48001]
2. 使用input和output训练fnet，把fnet当作FDTD用

### rnet
