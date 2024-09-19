# Python

## 命名规范

![image-20240612131202585](C:\Users\z1002\AppData\Roaming\Typora\typora-user-images\image-20240612131202585.png)

## built-in types

### Tuple

不可变的序列，可存放任意元素

- np和tensor的shape会返回tuple，使用[index]进行索引

## built-in functions

### decorator

有点像类的面向对象，这里是方法的面向对象，可以返回一个方法

#### staticmethod

静态方法，可以在定义方法时修饰，也可以在调用时修饰

```python
def f():
    ...
f2 = staticmethod(f)
# 这两种是等价的，其实就是静态方法嘛
@staticmethod
def f():
    ...
```

#### functools

更高优先级的函数

##### @wraps

封装函数，形成一种夺舍效果

```python
from functools import wraps
# 此处的f即代表被夺舍的函数对象
def hihi(f):
    @wraps(f)
    def wrapper(*args, **kwds):
        print('Calling decorated function')
        return f(*args, **kwds)
    return wrapper

@hihi
def example():
    """Docstring"""
    print('Called example function')

# 会先调用hihi，再调用example本体函数
example()

example.__name__

>>> Calling decorated function
>>> Called example function
>>> 'Docstring'
```

### @overload

重载运算符，用于表示同一种参数要求传入的可能数据类型，最后会运行非@overload的方法

当然你不按指定的数据类型来，也不会运行错误

```python
from typing import Union, List, Tuple, overload  
  
@overload  
def process_data(data: int) -> str:  
    ...  
  
@overload  
def process_data(data: str) -> List[str]:  
    ...  
  
@overload  
def process_data(data: float) -> Tuple[float, float]:  
    ...  
  
def process_data(data: Union[int, str, float]) -> Union[str, List[str], Tuple[float, float]]:  
    if isinstance(data, int):  
        return f"Received integer: {data}"  
    elif isinstance(data, str):  
        return list(data)  
    elif isinstance(data, float):  
        return (data * 0.5, data * 1.5)  
  
# Now, let's call the function with different types of data  
result1 = process_data(10)  
print(result1)  # Output: Received integer: 10  
result2 = process_data("Hello")  
print(result2)  # Output: ['H', 'e', 'l', 'l', 'o']  
result3 = process_data(3.14)  
print(result3)  # Output: (1.57, 4.71)  
result4 = process_data(np.zeros([20, 10]))  
print(result4) # Output: None  
```

# Torch

## Tensor

- torch.tensor

- torch.arange

- torch.norm

- torch.dot

- torch.reshape

- torch.no_grad：关闭自动计算梯度

- torch.matmul：矩阵乘法，可以跨维数，自动识别

  ![image-20240617183142629](C:\Users\z1002\AppData\Roaming\Typora\typora-user-images\image-20240617183142629.png)



创建tensor时要注意非维度不同的tensor计算会出大问题

```py
import torch
x = torch.zeros(6, 1)
x1 = torch.tensor([0, 1, 2, 3, 4, 5])
# 记得先reshape成2维再计算！
x1 = x1.reshape(-1, 1)
x2 = torch.arange(6)
loss1 = (x1 - x) ** 2 / 2
loss2 = (x2 - x) ** 2 / 2
print(x1.shape)
print(loss1)
print(loss2)
```

![image-20240617190636158](C:\Users\z1002\AppData\Roaming\Typora\typora-user-images\image-20240617190636158.png)

- tensor.detach
- tensor.grad
- tensor.backward()
- tensor.grad.zero_()

```python
import torch
# 顺序生成[0, 1, 2, 3]
a = torch.arange(4)
# 转换成2列，第一行（-1表示）自适应
a = a.reshape(-1, 2)
c = torch.ones([4,1])

b = torch.tensor([-3, 4], dtype=torch.float)
# 第二范数
b = torch.norm(b)
print(b)
```

```py
import torch
x = torch.arange(4.0, requires_grad=True)
y = 2 * torch.dot(x, x)
y.backward()
print(x.grad)
print(y)

# 计算关于x的另一个函数前，先清除梯度
x.grad.zero_()
y = x.sum()
print(y)
y.backward()
print(x.grad)

# 分离计算detach
x.grad.zero_()
y = x * x
u = y.detach()
# 此处z计算梯度时把u当作常量而不是x**x
z = u * x

z.sum().backward()
print(x.grad)
```



## Linear

### DIY

#### 生成数据集

```py
def synthetic_data(w, b, num_examples):
    X = torch.normal(0, 1, size=(num_examples, len(w)), dtype=torch.float)
    y = torch.matmul(X, w) + b
    # 加入0.01标准差的高斯噪声
    y += torch.normal(0, 0.01, size=y.shape)
    # y.reshape(-1, 1)很关键，一定要转成2维！不然计算损失函数遭大重
    return X, y.reshape(-1, 1)
```



#### 读取数据

```py
def load_array(data_arrays, batch_size)
	# data_arrays可以是[input_data, ouitput_data...]等list的形式，相当灵活
    dataset_train = torch.utils.data.TensorDataset(*data_arrays)
    datalaoder = torch.utils.data.DataLoader(dataset_train, batch_size, shuffle=True)
```

#### 损失函数

```py
# 人造MSE
def loss(y, y_pred):
	return (y - y_pred.reshape(y.shape)) ** 2 / 2
```

#### 优化器

```py
def sgd(params, batch_size, lr):
	with torch.no_grad():
        for param in params:
            # 此处loss没有÷batch_size因此要手动÷一次
            param -= lr * param.grad / batch_size
            # 记得清零
            param.grad.zero_()
```

#### 训练

```py
for epoch in range(num_epoch):
	for (input_batch, label_batch) in dataloader:
        l = loss(net(input_batch), label_batch)
  		# 没有求平均，因此要手动sum，因为只有标量才能反向传播
        # 求得w和b的导数
        l.sum().backward()
        # 更新的参数是权重和偏置
        sgd([w, b], batch_size, lr)
```

# TensorFlow

## 参数计算

### 一维全连接

$$
(C_{in} + 1)\times C_{out}
$$

举个例子，2*2的滤光片组编码图像解码成31个通道的，则滤光片矩阵$(4 + 1)\times31$，1表示偏置量（当然滤光片所有偏置量=0，这里只是作演示用），搞懂$C_{in}\times C_{out}$是**压缩感知的观测矩阵**这一点就足够了

### 二维卷积

- $C_{out}$：卷积核数量
- $k_h \times k_w$：卷积核大小
- $1$：卷积核偏置量

$$
(k_h \times k_w\times C_{in} \times + 1) \times C_{out}\\
$$

![image-20240625201440819](C:\Users\z1002\AppData\Roaming\Typora\typora-user-images\image-20240625201440819.png)

1. InputLayer：不参与计算，仅声明提示用

2. Dense：Input:  100; Output: 576; total = (100 + 1) * 576

3. Reshape：不参与计算

4. Conv2DTranspose：Input: 64; Output: 32; total = (3 * 3 * 64 + 1) * 32

5. Conv2DTranspose：Input: 32; Output: 16; total = (3 * 3 * 32 + 1) * 16

6. Conv2DTranspose：Input: 16; Output: 8; total = (3 * 3 * 16 + 1) * 8

7. Conv2DTranspose：Input: 8; Output: 4; total = (3 * 3 * 8 + 1) * 4

8. Conv2DTranspose：Input: 4; Output: 1; total = (3 * 3 * 4 + 1) * 1

   ![image-20240625204009859](C:\Users\z1002\AppData\Roaming\Typora\typora-user-images\image-20240625204009859.png)

## Keras

### 卷积

- padding：当为''VALID"时，卷积核始终在输入矩阵移动（左图）；当为‘SAME'时，卷积核中心位于矩阵角的顶点位置（右图）

  ![在这里插入图片描述](https://img-blog.csdnimg.cn/20190102212951137.gif)![在这里插入图片描述](https://img-blog.csdnimg.cn/2019010221302420.gif)

- stride：表示水平和**垂直**移动的间隔单位

- padding = 2，以最边界为始，上下左右+2

  ![在这里插入图片描述](https://img-blog.csdnimg.cn/20190101215139260.gif)

### 反卷积

- **反卷积的stride**指的是在相邻元素之间插入stride-1个padding，下图为实际stride=2

![在这里插入图片描述](https://img-blog.csdnimg.cn/20190101220640484.gif)

![image-20240625191706443](C:\Users\z1002\AppData\Roaming\Typora\typora-user-images\image-20240625191706443.png)

- Tensorflow的反卷积比较奇葩，先用stride隔空padding，再根据padding的模式进行2次padding，最后卷积完后还有**裁剪**...，stride=2直接理解为采样翻1倍就行了；stride=3就是翻2倍



#### Conv2D
