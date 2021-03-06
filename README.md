# Disk-Failure-Prediction-Project-Report
Disk Failure Prediction Project Report
彭寒秋

这次项目的目标是用机器学习模型根据各项S.M.A.R.T.属性的参数来预测硬盘驱动器的故障。基于您提供的信息，我决定像Blackblaze一样，选择硬盘驱动器的SMART 5,187, 188, 197, 和 198属性作为特征矩阵X，来预测一个驱动器今天是否会故障y。当然，在查阅相关文献后，我发现增加特征X的维度（加入更多SMART属性如：9, 193, 194, 241 和 242）可以进一步提升模型的性能。不过我在本次项目中因为时间限制只用了5种属性作为X。我决定用2016年第一季度的原始硬盘测试数据来训练和开发我的模型，然后在2016年第二季度的硬盘数据上测试我的模型。

1.	The methods or flow of data preprocessing and feature engineering.

（1）这些硬盘驱动器是由不同的供应商生产的，比如Seagate，Hitachi，Samsung等。由于每个供应商都有自己的收集SMART属性的方法，因此并非所有硬盘驱动都包含相同的SMART属性集。于是我只选择ST4000DM000型号的驱动器进行训练和测试。但由于我把很多数据处理的过程都写成了函数，人们可以简单的改变函数的输入就能够训练和测试新的机器学习模型来服务其他型号的驱动器。我也把输入数据里空的单元格填为了0。
（2）在查看了Backblaze提供的原始硬盘测试数据后，我发现它很不平衡。每天，绝大部分的驱动都是正常运行的，只有极少数出了故障。如果直接将这样的数据拿去训练，得到的模型也许能够很好的预测正常运行的驱动，但却很难预测出出了故障的驱动，而知道哪些驱动出了故障才是我们想知道的。所以我想办法增加了出故障的驱动的样本个数（上采样）。我想，如果一个硬盘今天出了故障，它的参数会很奇怪对吧，但它不可能一下子坏掉吧，它的参数应该是在昨天，前天，甚至一周前就开始出现异样了。所以，一个驱动如果今天出了故障，那我就把它前n天的状态都又正常改成出故障，这样就增加了一大批出故障的驱动的样本。我通过实验发现如果n由3变为7，我的模型预测准确率会由95.7%提高到99%。同时我又随机采样出与故障样本数相同个正常驱动的样本，组成一个平衡的数据集。我将这个平衡数据集保存为了一个新的csv文件。
（3）我选择原始的SMART属性参数作为输入的特征X而不是它的归一化后的，因为不太清楚它是用什么方法归一化的。我自己后来又用RobustScaler()做了标准化，减小了outlier的影响。不过我通过比较发现做不做标准化对准确率的提升影响不大，如果标准化的方式不对的话，还会有负面影响。

2.	How to choose machine learning models and tune the parameters?

我尝试了几种常见的的机器学习模型，如Histogram-based Gradient Boosting Tree, KNN, 朴素贝叶斯等。最后我选择了GBDT，因为GBDT可以比较好的对付数据中的噪音，而且事实证明它在平衡数据集和原始不平衡数据集里都表现得优于其他我尝试的模型。这次数据中的噪音可能是由于SMART属性之间的相关性较差所致。由于GBDT目标是降低总体错误率，因此它倾向于提高多数类别（正常运行的驱动）的预测准确率上，却会稍微降低少数类别（故障的硬盘）的预测准确率。Scikit-learn里面GBDT的超参数的默认值大部分都能带来很好的效果，我将learning rate调低了一些，希望它能用更多的树来拟合数据，结果是准确率稍微提升了一些。

3.   How to evaluate the results?

我首先在平衡数据集里用10-fold cross validation评估我的模型。我计算了他们的
precision, recall, train accuracy, test accuracy 和 F1 score。以下是GBDT在平衡数据集（2016年第一季度）里的表现，可以发现它表现还不错，没有出现过拟合。
Precision: 0.9643695291986862 
Recall: 0.5380592323399906 
Test Accuracy: 0.7564273789649415 
Training Accuracy: 0.7659841203510237 
F1 score: 0.6901928247456602 

然后我就把模型拿到测试集（2016年第二季度）里去测试，这里我对特征X做了正则化，但用的是训练集里的参数，因为这样可以确保我的模型在评估时是是没有偏差的。以下是我的模型在原始数据集（2016年第二季度）里的表现。
Precision: 0.005235602094240838
Recall: 0.7280701754385965
Accuracy: 0.9898925673493016
F1 Score: 0.010396442662992424

Backblaze的默认检测方法（Baseline）是：如果某个驱动的SMART 属性5,187, 188, 197, 和 198中有任何一个参数大于了0，就认为它是出了故障。这种方法在原始数据集（2016年第二季度）里的表现是：
Precision: 0.0009105323474465848
Recall: 0.7631578947368421
Accuracy: 0.9389188290192892
F1 score: 0.0018188945511564095

可以发现我的模型比Baseline好了不少，Precision, Accuracy 和F1 Score都显著提高。Recall稍微下降，和我之前提到的GBDT为降低总体错误率所做的trade-off符合。而且Baseline方法会预测所有只要有一点出现异常参数的硬盘都会发生故障，因此机器学习方法很难在Recall上超过它。我还制作了混淆矩阵，来帮助解释模型的表现。

4.   What insights or lessons learned from this task?

（1）	数据预处理对提升模型性能很重要，特别是不平衡的数据。
（2）	Dataframe遍历太慢了，希望能找到更高效的把故障硬盘前n天的状态都改变的办法。
（3）	如果有计算能力更强大的电脑，更多的时间，我会用更长时间的硬盘数据（如一年）和加入更多SMART属性来训练我的模型。
（4）	还希望尝试用神经网络来解决这个问题，因为它能更好的考虑到特征之间的联系和其他非线性关系。
