# 主动式



- $r(\lambda)$: 物体表面反射率

- $s(\lambda)$: 相机的spectral response

- $l(\lambda)$: 光源spectral response

- $k_i$: 相机曝光量系数

- $O(t_i)$: $t_i$时的相机输出
  $$
  O(t_i) = k_i \int{l(\lambda) r(\lambda)s(\lambda)d\lambda}
  $$

  # 1

  当光源选择中心波长$\lambda_0$的单色光时，$r(\lambda)$在$\lambda_0$处附近可以认为是常数，其他地方为0，因而提到积分号外

  记常数$C_i$：只与相机和光源单独的response有关
  $$
  C_i=\int{l(\lambda)s(\lambda)d\lambda}
  $$
  则
  $$
  O(t_i)=k_iC_ir(\lambda) \\
  r(\lambda)=\frac{O(t_i)}{\int{l(\lambda)s(\lambda)d\lambda}}=\frac{O(t_i)}{k_iC_i}
  $$
  若$Ci$中的$l(\lambda)、s(\lambda)$互为倒数即$C_i$恒为1，则
  $$
  r(\lambda)=\frac{O(t_i)}{k_i}
  $$
  且曝光时间都相同，则相机的输出即为物体表面反射率
  $$
  r(\lambda)=O(t_i)
  $$

  ## 2

  若选择
  $$
  r(\lambda)=\frac{O(t_i)}{\int{l(\lambda)s(\lambda)d\lambda}}
  $$