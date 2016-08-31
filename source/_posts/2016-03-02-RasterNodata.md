
title: 为影像数据去除无效值

date: 2016-02-02 13:00:00

tags: [Raster,Geodata]

categories: 木工开物



---


在使用栅格数据时，黑边和白边问题比较困扰我们，╮(╯-╰)╭ ，丑丑地如下所示：

![](http://img.blog.csdn.net/20160201174054625)

那如何处理这些问题呢？方法不是唯一的，我把 ArcMap 中最常用的几种方式列举一下，帮你给数据“化妆”，或者更确切的说是“整容”：

<br>

<br>


# Option 1  栅格计算器


在去黑边之前最先需要了解的是黑边的像元值是什么？ 可以使用 Identify 工具![](http://img.blog.csdn.net/20160201175818492) 查查看。  例如这份数据是 0。

![](http://img.blog.csdn.net/20160201175738451)


> 这时，你也许会想到设置图层背景值的色彩可以吧？
> 
> ![](http://img.blog.csdn.net/20160201181110700)
> 
> 如果仅为了显示使用，是可以解决问题的，但传统意义上的去黑边，我们一般理解成栅格数据的处理，也就是从存储上修改特定值（本例中是 0）值为 Nodata。
> 


单波段数据处理起来常常相对容易些，最常用的工具就是 **栅格计算器/Raster Calculator** 了。Raster Calculator 可以通过输入的Python语法的表达式，对当前数据框内的栅格图层进运算。它是个非常实用的栅格数据处理工具，如果你想 Dive In ，点 [**这里**](http://desktop.arcgis.com/zh-cn/desktop/latest/tools/spatial-analyst-toolbox/how-raster-calculator-works.htm)了解 Raster Calculator 的工作原理。

例如这个需求中仅需要一个简单表达式，使用到SetNull函数，如下所示：

![](http://img.blog.csdn.net/20160202100521847)

>  其中，表达式是 **SetNull("RasterLayer"  ==  0 , "RasterLayer" )**

这样执行工具之后，所有的 0 值变成了 Nodata，彻底去了黑边。

<br>

<br>

# Option 2  影像分析窗口

多波段数据同样会受到黑边的困扰，而且现实情况往往没有单波段数据那么理想，仅通过一个表达式就可以搞定。例如，我们常用的影像底图数据，通常有三个波段，通过包含RGB三个波段的一组值来表示像元值，例如 (0,255,129)。我一般会根据需要处理的数据量的多少给出不同的处理方法建议：

同样第一步需要确认黑边值是什么？本例中是 (0,0,0)

![](http://img.blog.csdn.net/20160202103049729)

<br>

ArcMap的 Windows 菜单中有 Image Analysis 。在 Image Analysis 窗口中的 Processing 部分可以对当前数据框中的图层赋予函数或函数链，从而对栅格数据实时处理。

![](http://img.blog.csdn.net/20160202104230098)

在弹出的窗口中，在 fx行右键插入函数，例如这个需求中会使用到 Mask Function。

![](http://img.blog.csdn.net/20160202115040077)

设置 Mask Function，(0,0,0) 组合是无效值。为什么选择 All 而不是 Any？ 这个答案很显见，同时都为0的像元值才是无效值，否则不是，例如（1，255，0）是有效的。或者说各个波段的0值是and关系，而不是or。

![](http://img.blog.csdn.net/20160202115853565)


这样带有函数的新栅格图层会自动加入 ArcMap 的 TOC，看起来万事大吉，然而这里需要说明下，这个图层需要Export到硬盘上的某个位置，它目前还是个临时数据，当layer被移除掉，这个结果就不复存在。

![](http://img.blog.csdn.net/20160202151625645)

所以，最后重要的一步，导出数据。

![](http://img.blog.csdn.net/20160202152154459)

<br>

(￣ˇ￣) **这种方法，还适用于具有多种无效值的情况**。例如，除了(0,0,0) 还包含 (255,255,255) ：

![](http://img.blog.csdn.net/20160202160314003)

我们需要做的仅是继续增加栅格函数。在 Function Template Editor 中函数们顺序相接，像个环环相接的链条，所以称为 ”函数链/ Function Chain“，咦，好像跑题了。请继续看如何设置无效值：

![](http://img.blog.csdn.net/20160202161443589)

这样就实现了去掉两组无效值。

<br>

<br>

# Option 3  镶嵌数据集  

那么摆在你面前的数据不是一个，而是“很多”呢？凡事保证质量之后，重复工作多了之后就同时需要保证效率，那么这种方法适用于“很多”、“大量”……

ArcGIS 的镶嵌数据集是个理想与实用兼备的影像数据管理模型，我们用它来“处理”大量栅格数据也是个不错的选择。

在地理数据库中创建镶嵌数据集，并将数据添加到镶嵌数据集中，之后：

> 在”之后“之前的预备动作，此文中不赘述，但是你可以看[**这篇**](http://blog.csdn.net/kikitamoon/article/details/40039903)了解这些内容。

![](http://img.blog.csdn.net/20160202171117222)


![](http://img.blog.csdn.net/20160202171131331)

在镶嵌数据集中可以使用工具 Define Mosaic Dataset Nodata 工具，对数据的无效值进行定义。

![](http://img.blog.csdn.net/20160202171615911)

从而批量去除了无效值。

镶嵌数据集本身也支持栅格函数，类似影像数据窗口中函数模板的设置，同样可以设置函数链来实现一些复杂的要求。

![](http://img.blog.csdn.net/20160202172231445)

有关镶嵌数据集函数，也可以参考[这篇](http://blog.csdn.net/kikitamoon/article/details/41673535)。


<br>

常见的操作一般就这几种，总结下，单波段优先考虑栅格计算器；多波段可以使用影像分析窗口；如果数据量较大，建议使用镶嵌数据集。


ArcGIS Desktop 虽然不是专业做影像数据处理的平台，但是拥有的这些功能能很大程度上解决影像处理的常见问题。写这一篇是因为不少用户遇到类似问题，集中总结下比较有意义，希望给大家一些思路，下次遇到此类的问题时，自己动手试试吧。