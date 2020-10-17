[TOC]
#做决策的对象
每一个device
#决策所需要的输入
其他所有device的位置以及device的ip，选择该device作为offloading target的device的数量，device的时钟频率
#算法步骤(DSOA)
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
#进度
完成了算法初始化中，距离最近的点的选取，采用kdtree。
##优化思路
考虑到用户的位置会经常发生改变。优化策略是缓存$\sqrt{n}$个位置修改，当缓存满了之后，才再次重构整棵树，重构之前，选取最近点还是依据原来的kdtree
**缓存打算使用redis**
```ditaa{cmd=true args=["-E"] hiden=true}
+--------+    +----------+
|{s}     |    |          |
| Redis  |--->|  server  |
|        |    |          |
+--------+    +----------+
```
##10.14
实现了DSOA的offloading决策，还需要进行单元测试
单元测试打算先随机生成数据，先保证程序能够正常跑起来
##10.15
单元测试完成，1000个节点可以保证在较短时间
绘制了节点数-执行时间的折现图
![](https://gitee.com/zacharytse/image/raw/master/img/20201015152935.png)
###后面的想法
用多个线程模拟多个服务器，同时多个线程模拟多个节点，数据还是随便生成
每个服务器都有自己的位置，这个位置是固定的
节点也有自己的位置，这个在之前定义节点的代码中已经进行了定义(经纬度)，但这个位置是要会随着时间改变的
代码可以参考之前的工作
```c++{.line-numbers}
pair<double, double> randomPoint(double startlon, double startlat) {
	srand((unsigned)time(NULL));
	double r1 = rand() / double(RAND_MAX);//RAND_MAX=0X7fff
	double r2 = rand() / double(RAND_MAX);
	const double Pi = 3.14159265358979323846264338328;
	//covert lat and lon to radian
	startlon = startlon * Pi / 180;
	startlat = startlat * Pi / 180;
	double maxdist = 100.0f;//mile
	double radiusEarth = 3960.056052;//mile
	maxdist /= radiusEarth;
	double dist = acos(r1 * (cos(maxdist) - 1) + 1);
	double brg = 2 * Pi * r2;
	double lat = asin(sin(startlat) * cos(dist) + cos(startlat) * sin(dist) * cos(brg));
	double lon = startlon + atan2(sin(brg) * sin(dist) * cos(startlat), cos(dist) - sin(startlat) * sin(lat));
	if (lon < -Pi) {
		lon = lon + 2 * Pi;
	}
	if (lon > Pi) {
		lon = lon - 2 * Pi;
	}
	return { lon * 180 / Pi,lat * 180 / Pi };
}
```
####想要模拟得到的数据
不同节点数量下的单个节点获得offloading决策结果的时间。
不同节点数量下的单个节点获得最终结果的响应时间
单个节点频繁发出请求，获得offloading决策结果的响应时间
单个节点频繁发出请求，获得最终结果的响应时间
所有节点都频繁发出请求，获得offloading决策结果的响应时间
所有节点频繁发出请求，获得最终结果的响应时间
##架构模型的考虑
![](https://gitee.com/zacharytse/image/raw/master/img/framework.png)

