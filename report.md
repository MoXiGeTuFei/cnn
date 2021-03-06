# 毕设进展情况
## 2016/2/29
* 跑了caffe官网的一个例子 http://nbviewer.jupyter.org/github/BVLC/caffe/blob/master/examples/00-classification.ipynb
* 学习了ipython notebook基本用法（跑官网例子用）
* 尝试找kernal分解的code

## 2016/3/1
* 未找到基分解、迭代剪枝权重的代码实现
* 找到了kernal张量化的code: https://github.com/vadim-v-lebedev/cp-decomposition  官网：http://sites.skoltech.ru/compvision/projects/speeding-up-cnns/

## 2016/3/2
* 读了论文：SPEEDING-UP CONVOLUTIONAL NEURAL NETWORKS USING FINE-TUNED CP-DECOMPOSITION
* 读了该论文code（未全部完成），了解了code基本结构
* 配置该论文code所需环境

## 2016/3/3
* 了解了protobuf的使用方法，读了caffe代码中的caffe.proto,了解了用caffe写网络结构的基本流程
* 跑了官网的例子http://caffe.berkeleyvision.org/gathered/examples/mnist.html

## 2016/3/8
* 读完论文code，理解了code执行流程
* 尝试在服务器上跑通code，正在配置相关环境

## 2016/3/9
* 在实验室服务器上调通了code
* 粗略测量了一下mnist上加速后与加速前的数据，结果存放在 tensorizing_kernel/result.txt
* 学习了finetuning的例子http://caffe.berkeleyvision.org/gathered/examples/finetune_flickr_style.html

## 2016/3/10
* 跑了官网例子：CIFAR-10 tutorial
* 读完了http://caffe.berkeleyvision.org/tutorial/, 熟悉caffe，笔记写在 笔记/caffe tutorial笔记.txt
* 学习如何在caffe中添加自己的层

```
下周计划：
* 尝试找下一篇论文的代码，并调通
* 在mnist、和人脸数据集上分别测试kernel张量化方法，给出时间空间优化程度以及准确率drop
```
## 2016/3/14
* 准备详细测试cp-decomposition方法在mnist数据集上的表现
  1. 按照论文，要对加速后的model进行fine-tune （问题：调参方法未掌握，调后的网络误差更大）
  2. 想观察加速前后的网络的空间占用，尚未找到合适方法
  3. 已经测量R=4时CPU、GPU下分别加速网络的error、accuracy、time，放在tensorizing_kernel/result.txt中（随实验继续更新，为什么GPU下反而更慢？）

## 2016/3/15
* 为了修改代码，仔细读了一遍源代码,对应论文查看又发现了一些问题
* Questions
	ConvolutionParameter Pad,group？（pad是指对feature map进行padding；group还是不知道）
	每一层都需要激活函数? （caffe的convolution layer默认没有激活函数）
	bias？   （进行4层卷积之后再加上bias）
	matrix的2范数？

## 2016/3/17
* 尝试跑一下人脸数据集
* 调通该数据集并测出准确率与时间开销
* 尝试将CP分解应用在该数据集上

## 2016/3/18
* 尝试将CP分解应用在该数据集上，源代码好像有问题，尚未调通

## 2016/3/19
* 将代码成功用在了人脸数据集上
* 发现优化结果不稳定，有时快有时比原来方法还慢，查找原因

## 2016/3/20
* 测量了一些人脸数据集上的数据，数据存在tensorizing_kernel/CP-decomposition.xlsx

```
本周总结
   本周主要是对应论文详细读了一遍源码，然后将源码在新的数据集上调通。调通的过程中遇到一些问题，主要是原来的代码与mnist数据集存在一些耦合，加上我对caffe prototxt中各种参数还不是很理解，耽误了时间。调通代码之后，开始测量在人脸数据集上的表现，发现有的时候还没有原始网络快，于是又发现了一些问题，列举如下：
   1. 同一参数在不同时间测量两遍差异巨大，可能与服务器状态不一样有关
   2. tensorizing_kernel/CP-decomposition.xlsx的sheet2中，绘制了forward-time随着batch-size的变化图像。发现，time随batch并不是线性增长（意味着每幅图像的时间越来越小），有的时候还会有突变，这样导致forward-time的测量标准很模糊（因为平均时间=time/batchsize会受batchsize影响，还不知道什么时候就突变了）
   3. 经推算，只有当分解的秩R<S*T*d*d/(S+d+d+T)时，理论上卷积层才会有加速效果。开始时我分解的是conv1层，由于conv1的输入维度=1（S=1），所以只有R小于20的时候才会有效果，导致有的时候加速后的比加速前还慢。后来将分解的层改为conv2（S=96），R的范围就加大了很多。
   4. 
下周计划
* 继续测量CP分解方法在人脸数据集上的参数
* 查找其他方法
```
## 2016/3/22
* 继续测量数据，分解conv2在CPU上的数据全部跑完

## 2016/3/25
* 根据caffe time的输出信息与空间信息对照理论，想看看实验结果是否可信，突然发现之前对各个卷积层的维数计算有错，对照caffe输出信息做了更改
* 而且发现时间复杂度应该是tensor-size*output-dim,之前只考虑了tensor-size，计算有误，做了更改。改后caffe测出的时间复杂度与理论计算出的基本一致（除了conv1，可能是第一个layer没有数据预取之类的或者要与底层data有交互所以慢了？）

## 2016/3/26
* 分解conv2在GPU上的数据全部跑完
* GPU上结果很不好，可能是太快了？调大了batch size，还是不行
* 做了中期展示PPT
* 发现conv3和conv2占时间最多，但是conv3张量更大，可能有更大潜力？正在测分解conv3

## 2016/3/27
* 修改PPT
* conv3上的结果很好，加速比 比conv2大很多，准确率还比conv2小
* 计算parameters的降低，理论上就是tensor-size
* 把conv2，3一起分解了（二者占整个网络50%），发现速度很快，1.7×，但是准确率降了5个百分点，说明分解多层这个想法能用，但是需要进一步调参
* 最新数据已上传
  最新测试结果：tensorizing_kernel/CP-decomposition.xlsx
* 还是没有开始进行fine-tuning...时间不够了

```
本周总结：
    本周将CPU和GPU上的数据基本测完了，发现了之前计算的错误（很严重。。之前实际时间一直和理论对不上），修改了原来数据，现在基本能保证CPU上的结果是对的。
    发现conv3有非常大的加速空间，对其进行分解，并与conv2一起分解，得到了一些合理的结果
问题：
    没有找到GPU上结果烂的原因
    finetuning还没着手开始做
下周计划：
	下周答辩完之后我觉得要快点着手找下一个方法了？然后尝试看看GPU上到底是怎么回事，还有fine-tuning conv2和conv3一起分解的那个网络
```
## 2016/4/5
* 今天完成了将2、3卷积层（2个最大的卷积层，耗时占比50%多）一起分解并进行了finetuning。
  最终达到了1.82×的全网络加速，2.03%的accuracy drop。（未经ft的accuracy drop=6.4%）
  经过多次尝试，最终的调参流程为：
  先分解conv2（rank=32），然后固定新层调旧层，达到1%的accuracy drop。然后分解conv3，同样固定新层微调旧层，最终达到2%的accuracy drop。注意保持较小learning rate，并持续降低。

## 2016/4/6
* 将GPU的数据部分重测了一遍，还增添了分解第三卷积层的测试数据。将以前数据不准确的进行修补与重测。卷积核张量分解部分的实验已经全部完成。
* 读完了全连通层张量分解的论文。

## 2016/4/7
* 学习、调试全连通层张量分解的代码。配置相关环境。由于这个代码框架用的是theano+lasagne，所以要先学习一下theano的使用。

## 2016/4/10
* 学习、调试全连通层张量分解的代码。

```
本周总结：
   本周首先将卷积核张量分解部分的收尾工作完成：对全网络进行了多层分解，并完成finetune，补全了GPU部分的数据。其中以前的GPU数据在服务器GPU空闲时又重测了一遍，现在的数据应该能保证环境一致。
   然后第二部分是找到全连通层张量分解的代码，由于其用了theano+lasagne，正在学习新框架。这两篇论文虽然都是张量分解，但是分解的结果的表示方式不太一样（可能之前那篇文章用的CP分解是张量分解的一个特例？），思路好像也不太一样？（前一篇是说把大卷积层变成几个小卷积层，后一篇说的是说分解后的形式能加快矩阵乘向量这一步骤的运算所以能加速，但是为什么能加快这一步骤在另一篇论文，正准备看，不知二者本质是否一样？）这两点有待进一步考察。如果两者本质一样是否直接在中期前的工作上扩展全连通层就好了不用使用他们的代码了呢？

```
## 2016/4/11
* 读论文、比较两种分解方法的不同。
```
CP分解问题：
已知error，求最小R，NP难
已知error、R，求分解形式，可能无解
但是参数比TT形式少（r量级）
之前那篇论文作者以发现CP分解应用在大网络（Alexnet，R为100~200量级）时，尝试缩小error-rate但无果。怀疑就是上述问题。
```
## 2016/4/12
* 读了TT分解相关论文
TENSOR-TRAIN DECOMPOSITION
Tensor rank and the ill-posedness of the best low-rank approximation problem
* 学习theano

## 2016/4/17
* 尝试将CP分解应用在全连通层，修改代码。（之前的代码是只能分解卷积的，它规定输入的层类型只能是convolution layer）

```
本周总结：
   本周开始的时候想尝试应用新的方法，于是学习了一会theano、lasagne、numpy等工具的使用，后来发现两种方法有一定的联系，经过与老师的讨论，决定先将CP分解应用在全连通层上。现在正在实现这个。预计下周应该实现完毕并有部分实验结果。（本周又有一部分时间用来赶模式识别课的ddl了。。）
```







