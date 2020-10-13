[TOC]
#做决策的对象
每一个device
#决策所需要的输入
其他所有device的位置以及device的ip，选择该device作为offloading target的device的数量，device的时钟频率
#算法步骤
1. 初始化
  1.1 每一个device选择离自己最近的device作为offloading的target
  1.2 设置全局的$\mathcal{D}$代表$MD_i$的决策结果，即对应的$X_i$以及target device ip,同时记录这种决策下，该device执行任务所需花费的时间,使用$\mathcal{T}$记录
2. 迭代
  2.1 设置迭代次数$\tau$
     - 每一次迭代，所有的device都需要重新决策,决策过程如下
  使用公式计算$x^{*}_i$以及$T_i$，$x_i$
$$
x^*_i=\frac{{\frac{s_i}{f^{local}_i}} - \frac{d_{i,j}}{v}-\alpha}{\frac{b_i}{r_i} + \frac{s_i}{f^{local}_i} + \frac{s_i}{f_{i,j}}}
$$
$$
T_i=(1-x_i)\frac{s_i}{f^{local}_i}
$$
    - 更新决策
从$\mathcal{T}$中找出$T$最小的划分方案，并更新决策
