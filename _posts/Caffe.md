title: 深度学习入门记（一）
date: 2015-10-21 19:58:27
tags: [Caffe,DL]
categories: 学习
---
----------


<font color=#808040 face=华文新魏>作为一名不知名大学的水硕一枚，了解CV和DL差不多也有一年的时间。期间走过很多的弯路，也遇到过很多挫折，但是感谢网络上的大神们的无私，让我从中获益良多。深度学习近年来相当热，同行人有一句话是这样说的，无GPU，不DL。设想如果一个没有GPU等硬件条件的话你去搞DL那简直是无法想象！然而我却是这样的一个人。究其原因，也是没有办法的办法啊。实验室没有这样的条件，也没钱买设备。电脑都是无GPU的。有时候真的挺无奈的。可是换个方向一想，也就那样了。曾经在群里有个人这样说过：如果你的导师花了大钱给你配了一个不错的GPU，结果你却什么成果都没有，那不简直要被骂死的节奏么！哈哈，开玩笑的啦。</font><!--more-->


----------
下面就讲讲自己的DL之路吧。
DL是一个相对较新的东西，学校里基本就没有什么人在搞，导师也是在摸索中前进。一开始我是真的不知道该如何下手。只能够靠网络了。百度CNN，结果就出来了一个matlab深度学习工具箱。

> 哇，这个工具箱好高大上啊！还是一个本科毕业生的毕设。
> 运行的程序的demo当然就是Mnist数据集啦。
> 感觉不错，怎么只能跑这个，还是灰度图像。简直了！
> 我还花了大量力气用这个toolbox运行cifar10。现在想想我也是蛮拼滴！

```matlab
% function test_example_CNN
load mnist_uint8;
global useGpu;

train_x = double(reshape(train_x',28,28,60000))/255;
test_x = double(reshape(test_x',28,28,10000))/255;
train_y = double(train_y');
test_y = double(test_y');

%% ex1 Train a 6c-2s-12c-2s Convolutional neural network 
%will run 1 epoch in about 200 second and get around 11% error. 
%With 100 epochs you'll get around 1.2% error

rand('state',0)
clear cnn;
cnn.layers = {
    struct('type', 'i') %input layer
    struct('type', 'c', 'outputmaps', 6, 'kernelsize', 6, 'activation', 'ReLU') %convolution layer
    struct('type', 's', 'scale', 2, 'method', 'm') %sub sampling layer 'm':maxpooling; 'a':average pooling; 's':stochastic pooling
%     struct('type', 'c', 'outputmaps', 12, 'kernelsize', 5, 'activation', 'ReLU') %convolution layer
%     struct('type', 's', 'scale', 2, 'method', 'm') %subsampling layer
};
debug =true;
if debug
    opts.alpha = 1;
    opts.batchsize = 10;
    opts.numepochs = 1;
    cnn = cnnsetup(cnn, train_x(:,:,1:10), train_y(:,1:10));
%     cnn = cnntrain(cnn, train_x(:,:,1:10), train_y(:,1:10), opts);
    cnnnumgradcheck(cnn, train_x(:,:,1:10), train_y(:,1:10));
end;

opts.alpha = 1;
opts.batchsize = 50;
opts.numepochs = 1;

cnn = cnnsetup(cnn, train_x, train_y);
cnn = cnntrain(cnn, train_x, train_y, opts);

[er, bad] = cnntest(cnn, test_x, test_y);

%plot mean squared error
figure; plot(cnn.rL);
assert(er<0.12, 'Too big error');
```

----------
结果可想而知，准确率实在感人。
接着我又一直找啊找，找了一大堆matlab,python代码，可以都是没有什么太大的进展。
期间，我试过[MatConvnet][1],哇感觉不错，也有不少的例子，可是呢，我除了例子我不知道该怎么用自己的dataset,再加上没有GPU，matlab效率又太低，简直不敢想。怎么办呢，想到了[Caffe][2]，那问题又来了。别人都是GTX980ti,Titian X及以上的显卡，而我什么都没有。我也不是没有配过caffe，但是失败了。当我再一次想起Caffe的时候，用到了[happynear-Caffe_windows][3]这个。虽然没有GPU，但还是成功了。但这是不是意味着就可以开始了? NO，远远没有开始。我又遇到了同样的问题，没有GPU事小，我还是不知道怎么用自己的数据集训练，训练结果只有**accuracy**，该怎么用model对我来说又是一个巨大的问题。这些我想放在后面再写，这里，我先放一些log
Cifar10的log
```log
I0805 17:16:22.139529  3164 net.cpp:248] Memory required for data: 88974128
I0805 17:16:22.140528  3164 solver.cpp:42] Solver scaffolding done.
I0805 17:16:22.140528  3164 solver.cpp:250] Solving CaffeNet
I0805 17:16:22.140528  3164 solver.cpp:251] Learning Rate Policy: step
I0805 17:16:22.262542  3164 solver.cpp:294] Iteration 0, Testing net (#0)
I0805 17:16:37.326541  3164 solver.cpp:343]     Test net output #0: accuracy = 0.085
I0805 17:16:37.327533  3164 solver.cpp:343]     Test net output #1: loss = 3.76392 (* 1 = 3.76392 loss)
I0805 17:16:39.253532  3164 solver.cpp:214] Iteration 0, loss = 2.44587
I0805 17:16:39.253532  3164 solver.cpp:229]     Train net output #0: loss = 2.44587 (* 1 = 2.44587 loss)
I0805 17:16:39.254533  3164 solver.cpp:486] Iteration 0, lr = 0.001
I0805 17:17:22.728545  3164 solver.cpp:214] Iteration 20, loss = 5.74343
I0805 17:17:22.728545  3164 solver.cpp:229]     Train net output #0: loss = 5.74343 (* 1 = 5.74343 loss)
I0805 17:17:22.729545  3164 solver.cpp:486] Iteration 20, lr = 0.001
I0805 17:18:05.551559  3164 solver.cpp:214] Iteration 40, loss = 2.88074
I0805 17:18:05.552556  3164 solver.cpp:229]     Train net output #0: loss = 2.88074 (* 1 = 2.88074 loss)
I0805 17:18:05.552556  3164 solver.cpp:486] Iteration 40, lr = 0.001
I0805 17:18:48.381570  3164 solver.cpp:214] Iteration 60, loss = 3.55146
I0805 17:18:48.382570  3164 solver.cpp:229]     Train net output #0: loss = 3.55146 (* 1 = 3.55146 loss)
I0805 17:18:48.383569  3164 solver.cpp:486] Iteration 60, lr = 0.001
I0805 17:19:30.568583  3164 solver.cpp:214] Iteration 80, loss = 2.15266
I0805 17:19:30.569582  3164 solver.cpp:229]     Train net output #0: loss = 2.15266 (* 1 = 2.15266 loss)
I0805 17:19:30.569582  3164 solver.cpp:486] Iteration 80, lr = 0.001
I0805 17:20:13.094594  3164 solver.cpp:214] Iteration 100, loss = 3.1185
I0805 17:20:13.095592  3164 solver.cpp:229]     Train net output #0: loss = 3.1185 (* 1 = 3.1185 loss)
```
还是cifar10的log
```log
I0828 14:24:12.793092  5492 solver.cpp:294] Iteration 0, Testing net (#0)
I0828 14:26:59.004948  5492 solver.cpp:343]     Test net output #0: accuracy = 0.1
I0828 14:27:03.255168  5492 solver.cpp:214] Iteration 0, loss = 2.28647
I0828 14:27:03.255168  5492 solver.cpp:486] Iteration 0, lr = 0.001
I0828 14:28:30.072227  5492 solver.cpp:214] Iteration 20, loss = 2.34191
I0828 14:28:30.072227  5492 solver.cpp:486] Iteration 20, lr = 0.001
I0828 14:30:00.069166  5492 solver.cpp:214] Iteration 40, loss = 2.30755
I0828 14:30:00.069166  5492 solver.cpp:486] Iteration 40, lr = 0.001
I0828 14:31:27.276870  5492 solver.cpp:214] Iteration 60, loss = 2.39797
I0828 14:31:27.276870  5492 solver.cpp:486] Iteration 60, lr = 0.001
I0828 14:32:55.750267  5492 solver.cpp:214] Iteration 80, loss = 2.29224
I0828 14:32:55.750267  5492 solver.cpp:486] Iteration 80, lr = 0.001
I0828 14:34:18.926513  5492 solver.cpp:294] Iteration 100, Testing net (#0)
I0828 14:37:07.560367  5492 solver.cpp:343]     Test net output #0: accuracy = 0.111875
```
从这两份log中看出了什么吗? 哈哈，肯定又人看出来了，下节再细说！


----------
梦想四季于2015/10/21/19:56

  [1]: http://www.vlfeat.org/matconvnet/
  [2]: http://caffe.berkeleyvision.org/
  [3]: https://github.com/happynear/caffe-windows
