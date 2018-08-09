# DNN-Speech-Enhancement-Task
An Experimental Study on Speech Enhancement based on DNN.

# 基本要求：
1. 学习神经网络的主要知识，推导神经网络的参数迭代和优化方法，也就是BP算法;
2. 阅读两篇论文,了解语音增强任务以及谱到谱匹配的方法;
3. 学习TensorFlow基础知识,了解全连接DNN网络的搭建方法;使用TensorFlow训练谱到谱匹配的DNN增强网络;
4. 使用python脚本重构语音,了解语音增强结果评价的指标

通过4周的实习，我们小组总体上完成了以上的基本要求，并在部分方面进行了进一步的拓展学习。基于深层神经网络的语音增强方法，
也就是深度学习语音增强，利用深度神经网络结构强大的非线性映射能力，通过大量数据的训练，训练出一个非线性模型进行语音增强。
此类方法在工程界已经实现落地，华为发布的mate10手机，已成功地将该技术应用到了复杂声学环境中的语音通话中，开辟了深度学习
应用于语音增强的先河。

总结来看，整个语音增强实习项目主要分为两个步骤:训练和增强。
在训练阶段，从训练集的TFRecords文件中导出学长已经提取的特征(sfeats)和标签(labels,对应的干净语音)，所用的特征是幅度
谱，通过对时域信号进行离散傅里叶变换得到。然后通过队列传入DNN神经网络就可以进行回归训练，DNN的训练使用最小均方误差(MSE)
作为损失函数,基于反向传播算法(BP)的有监督调优。

在增强阶段，从测试集的TFRecords文件中导出学长已经提取的特征(sfeats) ，特征同样是幅度谱，然后输入到己经训练好的模型中，
得到对干净语音的幅度谱的估计(保存为mat数据格式），再用带噪语音的相位合成成可主观测听的语音波形文件，波形的合成方法是经典
的重叠相加算法。具体谈谈构建的DNN神经网络，我们使用最小均方误差(MSE)作为损失函数, 选择Adam算法的优化器(调用tf.train.
AdamOptimizer )，设置指数衰减学习率，基础学习率设置为0.005，然后随着迭代的继续,  逐步减小学习率，使得模型在训练后期更
加稳定。我们采用了5层的DNN网络，3层隐藏层的节点均设为2048，激活函数选择线性整流函数（Rectified Linear Unit, ReLU），
后期优化过程我们加入了Dropout, 随机删掉网络中1/10的隐藏神经元，防止过拟合。权值初始化的方法多次优化后选择了xavier初始化，
通过使用这种初始化方法，我们能够保证输入变量的变化尺度不变，从而避免变化尺度在最后一层网络中爆炸或者弥散。

网络搭建并完成模型训练后，我们接下来需要进行语音重构及PESQ测试。
语音重构主要是还原频域特征到时域上，需要使用原始带噪音频的相位信息。具体步骤如下：
1. 利用从训练好的模型得到对干净语音的幅度谱的估计(保存为mat数据格式)，首先需要获取一帧的幅度谱；
2. 然后获取原始带噪音频中对应帧的相位，进行分帧，预加重，加窗等操作，再进行快速傅里叶变换，获取相位后返回；
3. 对幅度谱只使用[1: 257]的值，幅度谱和相位点乘，进行快速傅里叶变换，取前400维得到一帧数据，加窗，采用重叠相加算法(OverlapAdd), 
实际上就是帧移相加。所有帧处理完之后，加一个低通滤波器，将采样值的范围恢复到原始音频的范围内。 

PESQ(Perceptual evaluation of speech quality，客观语音质量评估)是客观评价通信中语音质量的方法，输入原始音频和在信道
中传输的音频（内容相同，也称劣化音频），要求8K或16K编码格式，处理后得到两者相似度的评分，MOS得分(Mean Opinion Score,平均
意见得分)，0-5分，分值越高，说明语音质量越好。

# 不足分析：
不足一: loss值最终只能稳定在0.01左右，部分时候可以降至0.001，但无法保持稳定
不足二：未使用交叉验证，将验证集的数据直接用作了训练集
不足三：对多线程作业缺乏了解，导致网络数据输入部分代码优化难度较大

# 改进：实现交叉验证部分的代码，充分利用验证集数据优化模型。对模型超参数优化也只进行了部分尝试，应该还有更好的优化方法。可以尝试
其他神经网络结构，毕竟DNN能力确实有限，比如尝试使用GAN (生成对抗网络), 同时也可以应用传统的数字信号处理方法进行进一步优化。
