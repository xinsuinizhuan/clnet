
# clNET: **OpenCL for Nets**
A Deep Learning Framework based on OpenCL, written by C++. Supports popular MLP, RNN(LSTM), CNN neural networks. 
基于OpenCL的深度学习计算框架，C++开发，支持多层感知器，长短时记忆模型，卷积神经网络。

Current Status: **pending release**.
-
Progress: Currently clnet can successfully run fully connected neural networks (MLP), CharRNN (LSTM) which uses dynamic computing graph to deal with loops, CNN (LeNet5 on MNIST dataset).  
Tested on Nvidia GTX1080, AMD R9 295X2, Intel HD Graphics 630 GPU.  
Support multiple devices training.

-
已完成进度：
可成功运行MLP全连接多层感知器，CharRNN（LSTM，基于动态计算图的循环实现），CNN（LeNet5，MNIST）的训练及推断。  
三种模型均在Nvidia GTX1080, AMD R9 295X2, Intel HD Graphics 630以及Intel CPU，AMD CPU/APU上测试通过。  
测试通过的编译环境：  
Windows 10，MSVS2015；  
Linux CentOS 7，g++ 4.8.5，Makefile；  
eclipse/CDT，MinGW64 6；  
eclipse/CDT，CrossGCC: CodeBench Lite 2014/05（gcc 4.8.3）；  
Kernel性能尚待进一步优化（矩阵乘法gemm及小卷积核FFT优化算法）。   
支持多显卡训练。分布式有待开发。

演示例子运行命令行：  
全连接MLP：  
```
.\Release\OpenCLNet.exe MLP /ds
```  
charRNN：
```  
.\Release\OpenCLNet.exe charRNN /ds :corpus_file D:\DataSets\charRNN\obama.txt :index_file D:\DataSets\charRNN\obama.index
```  
obama.txt可从[http://data.mxnet.io/mxnet/data/char_lstm.zip](http://data.mxnet.io/mxnet/data/char_lstm.zip)下载。  
MNIST CNN：  
```
.\Release\OpenCLNet.exe MNIST\_CNN /ds :mnist_folder D:\DataSets\MNIST\
```  
D:/DataSets/下需包含MNIST数据集文件train-images.idx3-ubyte，train-labels.idx1-ubyte，t10k-images.idx3-ubyte，t10k-labels.idx1-ubyte。可从[http://yann.lecun.com/exdb/mnist/](http://yann.lecun.com/exdb/mnist/)下载

如何调试：  
“/ds”生成执行树，“/ss”执行到第一个Tensor，停留，等待交互命令：  
```
.\Release\OpenCLNet.exe MLP /ss /ds /0  
```
<pre>
clnet::type::XavierNormalDistributionInitializer                XavierNormalDistributionInitializer
-       clnet::type::Weight             l0_weight[2,4096]
        clnet::type::Bias               l0_bias[4096]
        clnet::type::Weight             l1_weight[4096,1]
        clnet::type::Bias               l1_bias[1]
clnet::type::IterativeOptimizer         IterativeOptimizer[4]
        clnet::InstantTensor            data_generator
        clnet::type::Data               X[128,2]
        clnet::type::Weight             l0_weight[2,4096]
        clnet::type::Bias               l0_bias[4096]
        clnet::type::FullyConnectedLayer                FCLayer_0=sigmoid(l0_weight*X+l0_bias)
        clnet::type::Output             FCLayer_0[128,4096]
        clnet::type::Weight             l1_weight[4096,1]
        clnet::type::Bias               l1_bias[1]
        clnet::type::FullyConnectedLayer                FCLayer_1=softrelu(l1_weight*FCLayer_0+l1_bias)
        clnet::type::Output             FCLayer_1[128,1]
        clnet::type::Data               Y[128]
        clnet::back::Loss               linear_regression(FCLayer_1,Y)
        clnet::back::Gradient           gradient(FCLayer_1)[128,1]
        clnet::back::FullyConnectedLayer                back:FCLayer_1=softrelu(l1_weight*FCLayer_0+l1_bias)
        clnet::back::Gradient           gradient(FCLayer_0)[128,4096]
        clnet::back::FullyConnectedLayer                back:FCLayer_0=sigmoid(l0_weight*X+l0_bias)
        clnet::back::Gradient           gradient(l0_weight)[2,4096]
        clnet::back::Gradient           gradient(l0_bias)[4096]
        clnet::back::Gradient           gradient(l1_weight)[4096,1]
        clnet::back::Gradient           gradient(l1_bias)[1]
        clnet::type::StochasticGradientDescentUpdater           SGD
-               clnet::type::Weight             l0_weight[2,4096]
                clnet::type::Bias               l0_bias[4096]
                clnet::type::Weight             l1_weight[4096,1]
                clnet::type::Bias               l1_bias[1]
-       clnet::InstantTensor            MLPMonitor

[1,@2018-06-30 16:24:29] GeForce GTX 1050 Ti (kernels build: 119ms)
[debugger] interactive thread started on device 1.
[debugger] device 1 break on IterativeOptimizer: clnet::type::IterativeOptimizer 
</pre>
执行到SGD（别名为SGD的Tensor）：  
```
g SGD
```  
[debugger] device 1 continue to run.  
[debugger] device 1 break on SGD: clnet::type::StochasticGradientDescentUpdater  
观察输入样本X（别名为X的Tensor）：  
```
d X  
```
        this:                   0xfa1e60  
        type:                   clnet::type::Data  
        alias:                  X  
        volume:                 256  
        dimensions:             [128,2]  
        size:                   1024 bytes  
        pointer:                0xe7aac0  
        gradient:               NULL  
        inputs:  
                data_generator[]: clnet::InstantTensor  
        peers:  
                FCLayer_0=sigmoid(l0_weight*X+l0_bias)[]: clnet::type::FullyConnectedLayer  
```
X  
```
X[128,2]: clnet::type::Data  
0  
0:      1.00375,2.69076  
1:      1.57991,3.42622  
2:      2.75503,2.43962  
3:      2.05087,3.68789  
4:      3.46852,3.23981  
5:      1.52232,3.57683  
6:      3.1315,2.5406  
7:      1.91198,1.04495  
8:      1.27421,2.09336  
9:      1.44194,1.4977  
 ...  
观察梯度值：  
```
gradient(l0_weight)
```
gradient(l0_weight)[2,4096]: clnet::back::Gradient
0
0:      0.0932272,-0.00103467,0.616816,0.0487299,0.108153,0.453982,-0.168111,0.00612603,0.0466066,0.0776809,0.480914,0.00167271,-0.0579107,-0.171267,-0.00544866,0.0305377,0.396773,-0.0364095,-0.0105135,-0.244325,0.0070936,-0.0271294,0.0982886,0.000907668,0.0083473,0.000168261,0.038511,-0.00443278,-0.141771,-0.000452508,0.0574187,0.59741,-0.0461692,0.0273872,0.0211383,0.0937608,-0.0543251,-0.0177396,0.0404992,0.244961 ...
1:      0.043596,-0.105236,0.252182,0.0135588,0.0468406,0.208793,-0.0282288,0.0436221,0.0046685,0.0364535,0.231056,0.0131293,-0.0219158,-0.0984129,-0.000470661,0.010817,0.0848113,-0.00210151,-0.00500153,-0.113508,0.00290996,-0.00091675,-0.0437556,0.000426235,0.0348718,6.88916e-005,0.011789,-0.0166271,-0.046225,-0.000272511,0.0210079,0.22276,-0.0209225,0.0109369,0.00923857,0.0413359,0.0153701,0.0267138,0.0193877,0.177686 ...  
只看 部分数据：  
```
gradient(l0_weight)[:,0:8]
```
data[0:2/2,0:8/4096] for gradient(l0_weight)[2,4096]: clnet::back::Gradient
0:      0.0932272,-0.00103467,0.616816,0.0487299,0.108153,0.453982,-0.168111,0.00612603
1:      0.043596,-0.105236,0.252182,0.0135588,0.0468406,0.208793,-0.0282288,0.0436221  
```
l0_weight[:,0:8]
```
data[0:2/2,0:8/4096] for l0_weight[2,4096]: clnet::type::Weight  
0:      -0.280252,-1.62137,0.129004,0.495599,0.42723,0.0478061,-0.688217,1.87265  
1:      1.73239,0.18904,-0.326688,0.204418,-2.56337,-0.718758,-0.185233,-0.827314  
单步模式，执行完SGD，观察参数的变化：  
```
s
```
[debugger] step into mode activated.  
```
c
```
[debugger] device 1 continue to run.  
[debugger] device 1 break on MLPMonitor: clnet::InstantTensor  
```
l0_weight[:,0:8]
```
data[0:2/2,0:8/4096] for l0_weight[2,4096]: clnet::type::Weight  
0:      -0.280253,-1.62137,0.128998,0.495598,0.427229,0.0478016,-0.688215,1.87265  
1:      1.73239,0.189041,-0.326691,0.204418,-2.56337,-0.71876,-0.185233,-0.827315  
修改超参数：  
```
SGD.learning_rate
```
[debugger] SGD.learning_rate = 1e-005
```
SGD.learning_rate *= 0.5
```
[debugger] SGD.learning_rate = 1e-005  
[debugger] SGD.learning_rate *= 0.5  
[debugger] SGD.learning_rate = 5e-006  
执行profile，性能调优：  
```
pf
```
[debugger] profile mode activated.  
```
g
```
[debugger] breakpoint removed.  
```
c
```
[debugger] device 1 continue to run.  
[1,0,4ms] error rate: 0.331467  
[1,2000,39006/s] error rate: 0.00325364  
[1,4000,39072/s] error rate: 0.00251041  
```
p
```
[debugger] breakpoint added on SGD.  
[debugger] device 1 break on SGD: clnet::type::StochasticGradientDescentUpdater  
```
pf list
```
<pre>
back:FCLayer_1=softrelu(l1_weight*FCLayer_0+l1_bias): clnet::back::FullyConnectedLayer:              3s.271ms/20%  
FCLayer_1=softrelu(l1_weight*FCLayer_0+l1_bias): clnet::type::FullyConnectedLayer:              3s.87ms/19%  
back:FCLayer_0=sigmoid(l0_weight*X+l0_bias): clnet::back::FullyConnectedLayer:          923ms/5%  
SGD: clnet::type::StochasticGradientDescentUpdater:             872ms/5%  
FCLayer_0=sigmoid(l0_weight*X+l0_bias): clnet::type::FullyConnectedLayer:               854ms/5%  
X: clnet::type::Data:           805ms/4%  
linear_regression(FCLayer_1,Y): clnet::back::Loss:              641ms/3%  
Y: clnet::type::Data:           593ms/3%  
data_generator: clnet::InstantTensor:           507ms/3%  
MLPMonitor: clnet::InstantTensor:               455ms/2%  
gradient(FCLayer_0): clnet::back::Gradient:             440ms/2%  
l0_bias: clnet::type::Bias:             397ms/2%  
l0_weight: clnet::type::Weight:                 395ms/2%  
gradient(FCLayer_1): clnet::back::Gradient:             363ms/2%  
FCLayer_1: clnet::type::Output:                 355ms/2%  
l1_bias: clnet::type::Bias:             347ms/2%  
gradient(l0_weight): clnet::back::Gradient:             335ms/2%  
FCLayer_0: clnet::type::Output:                 334ms/2%  
gradient(l0_bias): clnet::back::Gradient:               324ms/2%  
l1_weight: clnet::type::Weight:                 289ms/1%  
gradient(l1_bias): clnet::back::Gradient:               287ms/1%  
gradient(l1_weight): clnet::back::Gradient:             278ms/1%  
</pre>
使用动态执行图，在执行“不等长”的数据如RNN-LSTM上，有性能优势：  
```
.\Release\OpenCLNet.exe charRNN /ds /0 :corpus_file D:\DataSets\charRNN\obama.txt :index_file D:\DataSets\charRNN\obama.index
```  
<pre>
clnet::type::XavierNormalDistributionInitializer                XavierNormalDistributionInitializer
-       clnet::type::Weight             embedding_matrix[84,256]
        clnet::type::Weight             lstm_weight_h0[256,1024]
        clnet::type::Weight             lstm_weight_x0[256,1024]
        clnet::type::Bias               lstm_bias0[1024]
        clnet::type::Weight             lstm_weight_h1[256,1024]
        clnet::type::Weight             lstm_weight_x1[256,1024]
        clnet::type::Bias               lstm_bias1[1024]
        clnet::type::Weight             lstm_weight_h2[256,1024]
        clnet::type::Weight             lstm_weight_x2[256,1024]
        clnet::type::Bias               lstm_bias2[1024]
        clnet::type::Weight             class_weight[256,84]
        clnet::type::Bias               class_bias[84]
clnet::type::IterativeOptimizer         IterativeOptimizer[4]
        SentenceIterator                [8289]
        clnet::type::Data               data[32,129]
        clnet::type::Weight             embedding_matrix[84,256]
        clnet::type::Embedding          Embedding(data)
        clnet::type::Output             embedding[32,129,256]
        clnet::type::LSTMInitializer            lstm_initializer
-               clnet::type::Output             lstm_cell_state0[32,256]
                clnet::type::Output             lstm_hidden0[32,256]
                clnet::type::Output             lstm_cell_state1[32,256]
                clnet::type::Output             lstm_hidden1[32,256]
                clnet::type::Output             lstm_cell_state2[32,256]
                clnet::type::Output             lstm_hidden2[32,256]
        clnet::type::LSTM               LSTM(embedding)
                clnet::type::Weight             lstm_weight_h2[256,1024]
                clnet::type::FullyConnectedLayer                lstm_cell2_FC_hidden=lstm_weight_h2*lstm_hidden2
                clnet::type::Weight             lstm_weight_h1[256,1024]
                clnet::type::FullyConnectedLayer                lstm_cell1_FC_hidden=lstm_weight_h1*lstm_hidden1
                clnet::type::Weight             lstm_weight_h0[256,1024]
                clnet::type::FullyConnectedLayer                lstm_cell0_FC_hidden=lstm_weight_h0*lstm_hidden0
                clnet::type::Output             lstm_input_timestep[32,256]
                clnet::type::Weight             lstm_weight_x0[256,1024]
                clnet::type::Bias               lstm_bias0[1024]
                clnet::type::FullyConnectedLayer                lstm_cell0_FC_input=lstm_weight_x0*lstm_input_timestep+lstm_bias0
                clnet::type::Output             lstm_cell0_FC_input[32,1024]
                clnet::type::BinaryOperator             lstm_cell0_FC_hidden+=lstm_cell0_FC_input
                clnet::type::Output             lstm_cell0_FC_hidden[32,1024]
                clnet::type::LSTMCell           lstm_cell0
                clnet::Tensor           lstm_dropout0_mask[32,256]
                clnet::type::DropOut            lstm_dropout0
                clnet::type::Output             lstm_hidden0[32,256]
                clnet::type::Weight             lstm_weight_x1[256,1024]
                clnet::type::Bias               lstm_bias1[1024]
                clnet::type::FullyConnectedLayer                lstm_cell1_FC_input=lstm_weight_x1*lstm_hidden0+lstm_bias1
                clnet::type::Output             lstm_cell1_FC_input[32,1024]
                clnet::type::BinaryOperator             lstm_cell1_FC_hidden+=lstm_cell1_FC_input
                clnet::type::Output             lstm_cell1_FC_hidden[32,1024]
                clnet::type::LSTMCell           lstm_cell1
                clnet::Tensor           lstm_dropout1_mask[32,256]
                clnet::type::DropOut            lstm_dropout1
                clnet::type::Output             lstm_hidden1[32,256]
                clnet::type::Weight             lstm_weight_x2[256,1024]
                clnet::type::Bias               lstm_bias2[1024]
                clnet::type::FullyConnectedLayer                lstm_cell2_FC_input=lstm_weight_x2*lstm_hidden1+lstm_bias2
                clnet::type::Output             lstm_cell2_FC_input[32,1024]
                clnet::type::BinaryOperator             lstm_cell2_FC_hidden+=lstm_cell2_FC_input
                clnet::type::Output             lstm_cell2_FC_hidden[32,1024]
                clnet::type::LSTMCell           lstm_cell2
                clnet::Tensor           lstm_dropout2_mask[32,256]
                clnet::type::DropOut            lstm_dropout2
                clnet::type::Output             lstm_hidden2[32,256]
-               clnet::Tensor           lstm_runtime_cell_no[3]
        clnet::type::Output             lstm[32,129,256]
        clnet::type::Weight             class_weight[256,84]
        clnet::type::Bias               class_bias[84]
        clnet::type::FullyConnectedLayer                FC=class_weight*lstm+class_bias
        clnet::type::Output             FC[4128,84]
        clnet::type::Data               label[32,129]
        clnet::back::Loss               negative_log_likelihood(softmax(FC),label)
        clnet::back::Gradient           gradient(FC)[4128,84]
        clnet::back::FullyConnectedLayer                back:FC=class_weight*lstm+class_bias
        clnet::back::Gradient           gradient(lstm)[32,129,256]
        clnet::type::LSTMInitializer            LSTM(embedding)_gradient_initializer
-               clnet::back::Gradient           gradient(lstm_cell_state0)[32,256]
                clnet::back::Gradient           gradient(lstm_hidden0)[32,256]
                clnet::back::Gradient           gradient(lstm_cell_state1)[32,256]
                clnet::back::Gradient           gradient(lstm_hidden1)[32,256]
                clnet::back::Gradient           gradient(lstm_cell_state2)[32,256]
                clnet::back::Gradient           gradient(lstm_hidden2)[32,256]
        clnet::back::LSTM               back:LSTM(embedding)
                clnet::back::DropOut            back:lstm_dropout2
                clnet::back::Gradient           gradient(lstm_hidden2)[32,256]
                clnet::back::LSTMCell           back:lstm_cell2
                clnet::back::Gradient           gradient(lstm_cell2_FC_hidden)[32,1024]
                clnet::back::BinaryOperator             back:lstm_cell2_FC_hidden+=lstm_cell2_FC_input
                clnet::back::Gradient           gradient(lstm_cell2_FC_input)[32,1024]
                clnet::back::FullyConnectedLayer                back:lstm_cell2_FC_input=lstm_weight_x2*lstm_hidden1+lstm_bias2
                clnet::back::DropOut            back:lstm_dropout1
                clnet::back::Gradient           gradient(lstm_hidden1)[32,256]
                clnet::back::LSTMCell           back:lstm_cell1
                clnet::back::Gradient           gradient(lstm_cell1_FC_hidden)[32,1024]
                clnet::back::BinaryOperator             back:lstm_cell1_FC_hidden+=lstm_cell1_FC_input
                clnet::back::Gradient           gradient(lstm_cell1_FC_input)[32,1024]
                clnet::back::FullyConnectedLayer                back:lstm_cell1_FC_input=lstm_weight_x1*lstm_hidden0+lstm_bias1
                clnet::back::DropOut            back:lstm_dropout0
                clnet::back::Gradient           gradient(lstm_hidden0)[32,256]
                clnet::back::LSTMCell           back:lstm_cell0
                clnet::back::Gradient           gradient(lstm_cell0_FC_hidden)[32,1024]
                clnet::back::BinaryOperator             back:lstm_cell0_FC_hidden+=lstm_cell0_FC_input
                clnet::back::Gradient           gradient(lstm_cell0_FC_input)[32,1024]
                clnet::back::FullyConnectedLayer                back:lstm_cell0_FC_input=lstm_weight_x0*lstm_input_timestep+lstm_bias0
                clnet::back::FullyConnectedLayer                back:lstm_cell2_FC_hidden=lstm_weight_h2*lstm_hidden2
                clnet::back::Gradient           gradient(lstm_weight_h2)[256,1024]
                clnet::back::FullyConnectedLayer                back:lstm_cell1_FC_hidden=lstm_weight_h1*lstm_hidden1
                clnet::back::Gradient           gradient(lstm_weight_h1)[256,1024]
                clnet::back::FullyConnectedLayer                back:lstm_cell0_FC_hidden=lstm_weight_h0*lstm_hidden0
                clnet::back::Gradient           gradient(lstm_weight_h0)[256,1024]
                clnet::back::Gradient           gradient(lstm_weight_x0)[256,1024]
                clnet::back::Gradient           gradient(lstm_bias0)[1024]
                clnet::back::Gradient           gradient(lstm_weight_x1)[256,1024]
                clnet::back::Gradient           gradient(lstm_bias1)[1024]
                clnet::back::Gradient           gradient(lstm_weight_x2)[256,1024]
                clnet::back::Gradient           gradient(lstm_bias2)[1024]
                clnet::back::Gradient           gradient(lstm_input_timestep)[32,256]
-               clnet::back::Gradient           gradient(embedding)[32,129,256]
                clnet::Tensor           lstm_runtime_cell_no[3]
        clnet::back::Gradient           gradient(embedding)[32,129,256]
        clnet::back::Embedding          back:Embedding(data)
        clnet::back::Gradient           gradient(embedding_matrix)[84,256]
        clnet::back::Gradient           gradient(class_weight)[256,84]
        clnet::back::Gradient           gradient(class_bias)[84]
        clnet::type::StochasticGradientDescentUpdater           SGD
-               clnet::type::Weight             embedding_matrix[84,256]
                clnet::type::Weight             lstm_weight_h0[256,1024]
                clnet::type::Weight             lstm_weight_x0[256,1024]
                clnet::type::Bias               lstm_bias0[1024]
                clnet::type::Weight             lstm_weight_h1[256,1024]
                clnet::type::Weight             lstm_weight_x1[256,1024]
                clnet::type::Bias               lstm_bias1[1024]
                clnet::type::Weight             lstm_weight_h2[256,1024]
                clnet::type::Weight             lstm_weight_x2[256,1024]
                clnet::type::Bias               lstm_bias2[1024]
                clnet::type::Weight             class_weight[256,84]
                clnet::type::Bias               class_bias[84]
-       clnet::InstantTensor            charRNN_monitor

[1,@2018-06-30 16:29:48] GeForce GTX 1050 Ti (kernels build: 297ms)
[debugger] interactive thread started on device 1.
[debugger] device 1 break on IterativeOptimizer: clnet::type::IterativeOptimizer
</pre>
