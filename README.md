## 雷达图像预测未来降水-CIKM AnalytiCup Top1清华团队思路分享

<div  align="center"> <img src="https://github.com/Jessicamidi/CIKM-Cup-2017/blob/master/pic/CIKM%20Analyticup%202017.jpg" width="1250" height="230" alt="Item-based filtering" /></div>

    《基于雷达图像的短期降水预报》是由ACM顶级数据挖掘会议CIKM举办的数据科学竞赛。CIKM 2017以“智慧城市，智慧型国家”为主题，通过人工智能同各学科领域的交叉研究，通过技术手段有效管理城市。

    本次 ___CIKM AnalytiCup 2017___ 由深圳气象局与阿里巴巴联合承办，旨在提升基于雷达回波外推数据的短期降水预报的准确性。比赛共吸引了来自全球1395个团队，来自清华大学的Marmot团队(姚易辰，李中杰)在比赛中脱颖而出，在复赛中以绝对优势排名第一。团队解题方案的核心思路如下：

 比赛官网：[阿里天池大数据平台](https://tianchi.aliyun.com/competition/introduction.htm?spm=5176.100066.0.0.773ef42f8FXDoN&raceId=231596)

<div  align="center"> <img src="https://github.com/Jessicamidi/CIKM-Cup-2017/blob/master/pic/radar.png" width="750" height="277" alt="Item-based filtering" /></div>

## 赛题目标

- 赛题提供10,000组的雷达图像样本。每组样本包含60幅图像，为过去90分钟内(间隔6 min,共15帧)，分布在4个高度(0.5km, 1.5km, 2.5km, 3.5km)上的雷达反射率图像。

- 每张雷达图像大小为[101,101]，对应的空间覆盖范围为101×101km。每个网格点记录的是雷达反射率因子值Z。反射率因子，表征气象目标对雷达波后向散射能力的强弱，散射强度一定程度上反映了气象目标内部降水粒子的尺度和数密度，进而推测其与降水量之间的联系。
<div  align="center"> <img src="https://github.com/Jessicamidi/CIKM-Cup-2017/blob/master/pic/sample_example.jpg" width="750" height="200" alt="Item-based filtering" /></div>

- 目标：利用各个雷达站点在不同高度上的雷达历史图像序列，预测图像中心位于[50,50]坐标位置的目标站点未来1-2小时之间的地面总降水量，损失函数为降水量预测值与真实值的均方误差。

<div  align="center"> <img src="https://github.com/Jessicamidi/CIKM-Cup-2017/blob/master/pic/Input_Output.png" width="750" height="315" alt="Item-based filtering" /></div>

## 算法架构
 本次比赛的特点在于时空序列的预测，即给出了目标站点周围一定空间范围的历史信息，需要预测在站点坐标上未来的降水走势，因而搭建时空之间的关联特性为解决问题的重中之重。同时有别于一般的计算机视觉问题，此次比赛提供的气象图像，其沿着时空方向的演化规律会满足一定的守恒律及连续性限制，发现物理问题的特殊性并寻找对应的表征量也是解决问题的关键。

 解决方案的流程分为前处理，特征提取，模型训练三个部分。前处理步骤中，完成局部图像的拼接，并通过SIFT描述子寻找时间方向的对应关系，获得云团运动的轨迹。特征描述中，将问题的特征归纳为3部分，分别为时间空间方向的矢量描述，云团形状的统计描述，及由云团轨迹外推得到目标站点的雷达反射率的空间图像描述。模型训练主模型采用了卷积神经网络CNN，图像部分采用2层卷积池化，随后将向量拉平到一维，即在全连接层与其余非图像类特征合并，共同输入到2个隐藏层的神经网络中。

<div  align="center"> <img src="https://github.com/Jessicamidi/CIKM-Cup-2017/blob/master/pic/Structure.png" width="550" height="270" alt="Item-based filtering" /></div>


## 图像拼接

 赛题给出的局部雷达图像，样本与样本之间并不完全独立，图像样本之间存在一定的重叠，可以通过模板匹配的方式寻找样本之间的坐标关联特性。通过样本之间的局部图像拼接，能够将一系列小范围的局部雷达图像恢复到空间更大范围的雷达图像，进而获得关于云团更加整体的特性。通过局部图像的拼接，能够获得如下两方面效果：

1. 为目标站点的时空轨迹追踪提供更大的空间延伸量。目标站点附近更大的空间图像范围，能够对应更长的时间外推量。

2. 获得云团整体的结构，方便从更为宏观的视角提取特征描述云团形态。

<div  align="center"> <img src="https://github.com/Jessicamidi/CIKM-Cup-2017/blob/master/pic/pintu.png" width="750" height="394" alt="Item-based filtering" /></div>

<div  align="center"> <img src="https://github.com/Jessicamidi/CIKM-Cup-2017/blob/master/pic/pin2.gif" width="750" height="514" alt="Item-based filtering" /></div>
 
## 轨迹追踪
 根据流体力学中的泰勒冻结假设(Taylor Frozen Hypothesis)，流场中存在显著的时空关联特性，即可以认为雷达反射图中云团在短时间内趋向于在空间以当地平均对流速度平移，短时间内并不会发生外形或者反射强度的剧烈改变。即监测点x处在未来τ时刻后的雷达信号f，能够通过平均对流速度U，从当前时刻t位于坐标的x-Uτ的信号中体现：

<div  align="center"> <img src="https://github.com/Jessicamidi/CIKM-Cup-2017/blob/master/pic/eqn-1.png" width="250" height="21" alt="Item-based filtering" /></div>

 为了寻找每个空间坐标对应的对流速度U， 可以通过SIFT描述子在一定时间间隔内，在空间坐标上的匹配，寻找相同关键点在较短时间间隔δt内像素的平移量δx,即得到空间每个位置处的对流速度。

<div  align="center"> <img src="https://github.com/Jessicamidi/CIKM-Cup-2017/blob/master/pic/eqn-2.png" width="200" height="50" alt="Item-based filtering" /></div>

 下图给出了相邻两帧图像上，SIFT描述子及相应的空间匹配关系。其中圆圈大小对应了关键点的特征尺度，圆圈中的刻度方向表征其主方向。两帧图像的匹配连线基本平行，即全场以一个近似相同的速度作对流运动。

<div  align="center"> <img src="https://github.com/Jessicamidi/CIKM-Cup-2017/blob/master/pic/SIFT.png" width="850" height="850" alt="Item-based filtering" /></div>

## 特征提取
 特征包含时间外插反射率图像，时间空间的矢量，云团形状的统计描述三部分。

 __时间外插反射率图像__：由上述的图像拼接及轨迹追踪后，能够定位出全场的速度矢量见下图。以泰勒冻结假设和关键点匹配追踪到未来1.5个小时流场速度矢量后，能够外插未来每个坐标点的运动轨迹，即能够推测出未来位于目标站点上方的云团，在当前时刻雷达图像上的空间坐标。 图中白色圆圈坐标点的云团，会在1.5小时由图中对流矢量的作用下，运动到红色目标站点上方。因此截取空间轨迹上白点周围41×41大小，3个空间高度(1.5km,2.5km,3.5km)的局部图像作为卷积神经网络的图像输入。

<div  align="center"> <img src="https://github.com/Jessicamidi/CIKM-Cup-2017/blob/master/pic/sub-image.png" width="850" height="445" alt="Item-based filtering" /></div>

__时间和空间特征提取__: 在时间和空间方向（高度方向）提取图像像素的统计值（平均值、最大值、极值点个数、方差等等），作为时空特征的描述输入CNN的全连接层。

 __全局云团形状特征提取__: 某些特定的云层形态会对应典型降水事件。从拼接后的全局图像中提取云团形状的整体形态特征，包含雷达反射率的直方图和统计类信息、云团运动速度和方向、加速度、流线曲率、SIFT描述子的直方图、监测点位置、检测点反射率与最大值比值等。

<div  align="center"> <img src="https://github.com/Jessicamidi/CIKM-Cup-2017/blob/master/pic/cloud-pattern.png" width="850" height="526" alt="Item-based filtering" /></div>

## 训练模型
- 卷积层中图像的输入为时间外推得到目标站点附近41×41的空间范围，采用较大的空间图像输入，希望能够包含轨迹预测的误差以及测评目标在1小时内的总降水量。图像部分采用2层卷积池化，随后将向量拉平到一维，即在全连接层与其余非图像类特征合并，共同输入到2个隐藏层的神经网络中。

- 模型通过dropout防止过拟合，keep_prob取值为0.65，梯度下降采用的Adam优化算法。1200个迭代步后即达到稳定。


## 总结
 虽然此前参加过多次大数据竞赛，但初次涉足图像类比赛能够获奖也是非常之意外。本次解题方案并未使用ImageNet上较为流行的InceptionNet或者ResNet，即用深度的图像卷积网络来做训练。而是针对气象问题的特殊性，针对时间空间关联这一重要线索，采用传统的关键点提取SIFT方法与卷积神经网络CNN结合的形式预测目标站点的降水量。

<div  align="center"> <img src="https://github.com/Jessicamidi/CIKM-Cup-2017/blob/master/pic/Rank.png" width="750" height="357" alt="Item-based filtering" /></div>


 由于思路的特殊性，团队在未做调参的情况下已经能够大幅领先其他队伍。未来会对气象业务有更多探讨，用大数据力量推动气象预报的发展。感谢天池大数据平台组织比赛，感谢深圳气象局提供比赛数据，感谢CIKM2017组委会。

 最后欢迎大家对于现有解题方案提出宝贵意见。队伍成员的邮箱是:

 姚易辰：yaoyichen23@163.com

 李中杰: lizhongjie1989@163.com

