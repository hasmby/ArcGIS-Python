title: 空间统计（一）度量地理分布
categories:
  - 木工开物
date: 2015-02-26 17:50:36
tags: [Geoprocessing,Spatial Statistics]
---

ArcGIS Desktop 中有一个包含了一系列用于研究空间分布（spatial distribution）、空间模式（spatial pattern）、空间关系（spatial relationship）的统计的工具箱 —— Spatial Statistics Toolbox。这与普通的统计方法不同，空间统计将许多地理空间的概念融入到统计算法逻辑中，例如：邻域（proximity），面积（area），连通性（connectivity）等。

本篇总结一下有关 度量地理分布（ Measuring Geographic Distributions）工具集中提供的功能。这些工具用于研究要素的空间分布特征，下面一个一个来看：

![](http://img.blog.csdn.net/20150226151543719)

<br>

# Central Feature

中心要素（Central Feature）这个工具可用于寻找一组要素中处于最中心位置的要素，这一组要素可以是点、线、面。

可解决的问题例如：

某个区域的仓库中最中心位置的是哪个? 

![](http://img.blog.csdn.net/20150204110834566?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQva2lraXRhTW9vbg==/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast )

在几个区块中，哪一个具有最中心的位置？

![](http://img.blog.csdn.net/20150204111026216?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQva2lraXRhTW9vbg==/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast )

工具的算法是将每个要素质心与其他各要素质心之间的距离计算并求和，取最小值。那么问题来了……

**距离测量方法是什么？**

有欧式距离与曼哈顿距离可选，区别点这里。并且需牢记，投影坐标系下才有准确的测量值，如果数据是经纬度坐标系，需要投影变换。

**要素的质心如何确定？**

对于线和面要素，距离计算中会使用要素的质心。对于多点、折线或由多部件组成的面，将会使用所有要素部件的加权平均中心来计算质心。点要素的加权项是 1，线要素的加权项是长度，而面要素的加权项是面积。

**如何求多组要素的各自的中心要素？**

设置分组字段，可以对多个分组的要素分别计算中心要素。

<br>

# Mean Center

平均中心（Mean Center）用于计算输入的要素的质心的平均中心，因此这个工具会计算出一个新的点。

如果输入数据包含 Z 值，那么在平均中心也会计算 Z 值，也就是得到3D点。

![](http://img.blog.csdn.net/20150209145427121?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQva2lraXRhTW9vbg==/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/Center)


> 这个工具在现实中有什么应用呢？我从帮助中引用几个有意思的应用：

> ★  犯罪分析师在对白天事件点与夜间事件点进行对比评估时，可能希望查看盗窃行为的平均中心是否会有所变化。这有助于公安部门更好地分配资源。

> ★  野生生物学家可以计算某个公园若干年内的麋鹿观测值的平均中心，以了解夏季和冬季麋鹿会在何处聚集，从而为公园游客提供更好的信息。

> ★  GIS 分析师可以通过将 911 紧急电话的平均中心与紧急响应站的位置进行比较来评估服务水平。此外，该分析师还可以对由超过 65 岁的个人加权所得的平均中心进行评估，从而确定提供老人服务的理想位置。



举个栗子：

2012年西非的马里暴动的位置信息采集后，求取平均中心大致可以了解事件的分布情况。

所有事件：

![](http://img.blog.csdn.net/20150227112032811)


各类事件的平均中心：

![](http://img.blog.csdn.net/20150227113039745)


<br>

# Median Center

中位数中心（Median Center）工具使用迭代算法来查找可使所有要素间的欧氏距离达到最小的点。

平均中心和中位数中心均是中心趋势度量。但是，比较而言，中位数中心工具的算法受数据异常值的影响较小。

例如，对紧凑性群集点的平均中心进行计算的结果是该群集中心处的某个位置点。如果随后添加一个远离该群集的新点并重新进行计算平均中心，会注意到结果会向新的异常值靠近。而如果要使用中位数中心工具执行相同的测试，会发现新的异常值对结果位置的影响明显减小。

如下图中，向数据中增加两个相隔较远的点， Median Center 看起来要比 Mean Center 偏移的更小。

![](http://img.blog.csdn.net/20150210115001570)


PS： 尽管中位数中心工具只返回一个点，但实际上,距所有要素的距离最小的位置点可能有多个。

<br>

# Directional Distribution (Standard Deviational Ellipse)

方向分布（标准差椭圆）工具可以查看要素的分布是否是狭长的，是否具有方向性，从而使我们直观的感受数据的趋向。该方法是由平均中心作为起点对 x 坐标和 y 坐标的标准差进行计算，从而定义椭圆的轴，因此该椭圆被称为标准差椭圆。

![](http://img.blog.csdn.net/20150226154212460)

在工具生成的椭圆面中会包含：

- 平均中心的 X 和 Y 坐标、两个标准距离（长轴和短轴）及椭圆的方向，这五个值。
- 如果要素的基础空间模式是中心处集中而朝向外围的要素较少（一种空间正态分布），则一个标准差椭圆面会包含聚类中约 68％ 的要素，两个标准差椭圆面会包含聚类中约 95％ 的要素，三个标准差椭圆面则可包含聚类中约 99％ 的要素。


> 这个工具可能的应有哪些呢？从帮助文档中摘录过来：
> 
> - 在地图上标示一组犯罪行为的分布趋势可以确定该行为与特定要素（一系列酒吧或餐馆、某条特定街道等）的关系。 
> - 在地图上标示地下水井样本的特定污染可以指示毒素的扩散方式，这在部署减灾策略时非常有用。
> - 对各个种族或民族所在区域的椭圆的大小、形状和重叠部分进行比较可以提供与种族隔离或民族隔离相关的深入信息。
> - 绘制一段时间内疾病爆发情况的椭圆可建立疾病传播的模型。



我还使用上面马里的例子，了解各种事件的分布情况：

![](http://img.blog.csdn.net/20150227114022781)

且不说分布方向，我们可以看出，无论是哪种暴动事件，与之并发导致的无家可归的难民是分布最广的。


<br>


# Standard Distance

标准距离工具可通过绘制一个半径等于标准距离值的圆在地图上体现一组要素的紧密度。

![](http://img.blog.csdn.net/20150226163623440)

> 可能的应用：
> 
> - 利用两种或多种分布的值对分布进行比较。例如，犯罪分析家可以对袭击行为和汽车偷窃行为的紧密度进行比较。了解不同犯罪类型的分布情况可能有助于警察制定出应对犯罪行为的策略。如果特定区域内的犯罪行为分布很紧凑，那么在该区域中心附近配置一辆警车也许就足够了。但如果分布较分散，则可能需要几辆警车同时巡查该区域，才能更有效地对犯罪行为做出响应。
> - 可以对同一类型要素在不同时间段内的分布情况进行比较。例如，犯罪分析家可以对白天盗窃行为和夜间盗窃行为进行比较，以了解白天与夜间相比，盗窃行为是更加分散还是更加紧凑。
> - 可将要素分布与静态要素进行比较。例如，可以针对某个区域内各响应消防站在几个月内接到的紧急电话的分布情况进行度量和比较，以了解哪些消防站响应的区域较广。


# Linear Directional Mean

线性方向平均值工具可以计算一组线要素的趋势的平均角度。

许多具有一个起点和一个终点的线要素，例如飓风路径，我们就可以使用这个工具计算飓风的平均方向。再比如断层线，这种线要素我们一般认为它们有方位（orientation），但是没有方向（direction），我们也可以使用这个工具计算平均方位。

所以，如果要计算平均方向，请确保所有线的方向都是正确的。如果要计算平均方位，则会忽略线的方向。

请牢记，尽管大多数线在起点和终点之间具有多个折点，此工具将只使用起点和终点来确定方向。
 
![](http://img.blog.csdn.net/20150226174741467)

输出线要素的为位于要素平均中心，长度为输入要素的平均长度。包含了罗盘角（以正北为基准方向按顺时针旋转）、方向平均值（以正东为基准方向按逆时针旋转）、圆方差（指示方向/方位偏离方向平均值的程度）、平均中心 X 和 Y 坐标及平均长度。

![](http://img.blog.csdn.net/20150226174841124)



> 可能的应用：
> 
> - 比较两组或多组线。例如，研究河谷中麋鹿和驼鹿迁移状况的野生生物学家可计算这两个物种迁徙路径的方向趋势。
> - 比较不同时期的要素。例如，鸟类学家可逐月计算猎鹰迁徙的趋势。方向平均值可汇总多个个体的飞行路径并对每日的迁移进行平滑处理。这样便可很容易地了解鸟类在哪个月进得最快，以及迁徙在何时结束。
> - 评估森林中的伐木状况以了解风型和风向。
> - 分析可以指示冰川移动方式的冰擦痕。
> - 标识汽车失窃及被盗车辆追回的大体方向。


再举个栗子：

如下图中绿色的线段表示历年不同日期采集的风向数据，现在根据不同的年份计算每年的平均风向，结果如图，图形总是比表格数据直观的很：

![](http://img.blog.csdn.net/20150227162030456)