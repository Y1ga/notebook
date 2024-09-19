# Clion

## 1 code editing

- 注释一个代码块：ctrl + shift + /

  ![image-20240905153647990](C:\Users\z1002\AppData\Roaming\Typora\typora-user-images\image-20240905153647990.png)

- 代码提示：alt + enter

  ![image-20240905153802992](C:\Users\z1002\AppData\Roaming\Typora\typora-user-images\image-20240905153802992.png)

- 选中word：ctrl + w

  - 选中多个word：多按几次 ctrl + w

  - 取消选中：ctrl + shift + w

  - 选中代码块：**在block头位置ctrl + w两次**（图中if）

    ![image-20240905153953157](C:\Users\z1002\AppData\Roaming\Typora\typora-user-images\image-20240905153953157.png)

- 环绕语句：ctrl + alt + t

  ![image-20240905154055424](C:\Users\z1002\AppData\Roaming\Typora\typora-user-images\image-20240905154055424.png)

  - 删除环绕语句：ctrl + shift + del

    ![image-20240905154302827](C:\Users\z1002\AppData\Roaming\Typora\typora-user-images\image-20240905154302827.png)

- ctrl + d：复制当前行

  - 组合技：选中上两条代码并复制：shift + ↑ 两次 + ctlr + d

  - 删除指定行：ctrl + shift + a -> delete line

    ![image-20240905154818903](C:\Users\z1002\AppData\Roaming\Typora\typora-user-images\image-20240905154818903.png)

- 移动整个代码块：ctrl + shift + ↑（指针需要在代码块首部）

  ![image-20240905155024429](C:\Users\z1002\AppData\Roaming\Typora\typora-user-images\image-20240905155024429.png)

  - 移动单条代码：alt + shift + ↑

- 折叠代码块：ctrl + `-`

  ![image-20240905155254825](C:\Users\z1002\AppData\Roaming\Typora\typora-user-images\image-20240905155254825.png)

  - 展开代码块：ctrl + `=`
  - 折叠所有/展开所有： ctrl + shift + `-` / ctrl + shift + `=`

- 选中基本数据类型：alt + j

  ![image-20240905155404346](C:\Users\z1002\AppData\Roaming\Typora\typora-user-images\image-20240905155404346.png)

  - 取消选中：alt + shift + j

  - 选中全部：ctrl + alt + shift + j；退出选出全部：esc

    ![image-20240905155456899](C:\Users\z1002\AppData\Roaming\Typora\typora-user-images\image-20240905155456899.png)

    

## 2 refactorings

- 改名：shift + f6 或 ctrl + shift + a找到`rename`

  - alt + shift + o 有高级选项

    ![image-20240905160221011](C:\Users\z1002\AppData\Roaming\Typora\typora-user-images\image-20240905160221011.png)

- 更改全局函数参数：ctrl + f6

  ![image-20240905160537245](C:\Users\z1002\AppData\Roaming\Typora\typora-user-images\image-20240905160537245.png)

  - 将常量重构函数参数：ctrl + alt + p

    ![image-20240905160939731](C:\Users\z1002\AppData\Roaming\Typora\typora-user-images\image-20240905160939731.png)

## code completion

- 后置补全：变量后 + “.make”

  ![image-20240905161734777](C:\Users\z1002\AppData\Roaming\Typora\typora-user-images\image-20240905161734777.png)

- 补全分号：ctrl + shift + enter

  ![image-20240905161854890](C:\Users\z1002\AppData\Roaming\Typora\typora-user-images\image-20240905161854890.png)

  - 自动跳入语句块：ctrl + shift + enter

    ![image-20240905161947530](C:\Users\z1002\AppData\Roaming\Typora\typora-user-images\image-20240905161947530.png)

    ![image-20240905161957549](C:\Users\z1002\AppData\Roaming\Typora\typora-user-images\image-20240905161957549.png)



## 3 navigation

- shift + shift：全局搜索

  ![image-20240905154510531](C:\Users\z1002\AppData\Roaming\Typora\typora-user-images\image-20240905154510531.png)

  - ctrl + shift + a：搜索actions

- 搜索类名：ctrl + n

- 在project里搜索：ctrl + shift + n

  - alt + w：精确匹配

    ![image-20240905152842022](C:\Users\z1002\AppData\Roaming\Typora\typora-user-images\image-20240905152842022.png)

  - 在project里替换：ctrl + shift + r

- 预览structure（支持搜索）：ctrl + f12

  ![image-20240905153052299](C:\Users\z1002\AppData\Roaming\Typora\typora-user-images\image-20240905153052299.png)

  - 在左下角预览structure：alt + 7

    ![image-20240905153127240](C:\Users\z1002\AppData\Roaming\Typora\typora-user-images\image-20240905153127240.png)

- 在当前文件搜索：ctrl + f

  ![image-20240905153338693](C:\Users\z1002\AppData\Roaming\Typora\typora-user-images\image-20240905153338693.png)

  - 滑动下一个：enter / f3
  - 滑动上一个：shift + f3



