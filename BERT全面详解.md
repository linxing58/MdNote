# BERT全面详解
一、从RNN开始
--------

NLP里最常用、最传统的深度学习模型就是循环神经网络 RNN（Recurrent Neural Network）。这个模型的命名已经说明了数据处理方法，是按顺序按步骤读取的。与人类理解文字的道理差不多，看书都是一个字一个字，一句话一句话去理解的。

RNN 有多种结构,如下所示：

![](https://pic2.zhimg.com/v2-81f56bec090d6175a8568eed8e5bd221_b.jpg)

### 1\. one-to-one

![](https://pic2.zhimg.com/v2-63a2ae7b152c621fb466452516d62d5d_b.jpg)

最基本的单层网络，输入是x，经过变换Wx+b和激活函数f得到输出y。

### 2\. one-to-n

![](https://pic3.zhimg.com/v2-772e2578a09e5e527b9278df2def22a6_b.jpg)

还有一种结构是把输入信息X作为每个阶段的输入：

![](https://pic2.zhimg.com/v2-564392f4443777d7ebbc37650436c115_b.jpg)

这种 one-to-n 的结构可以处理的问题有：

*   从图像生成文字（image caption），此时输入的X就是图像的特征，而输出的y序列就是一段句子，就像看图说话等
*   从类别生成语音或音乐等

### 3\. n-to-n

最经典的RNN结构，输入、输出都是等长的序列数据。

![](https://pic3.zhimg.com/v2-609d077666a8e3c0c8975c625ef8176a_b.jpg)

**用公式表示如下：** 

Ot=g(V⋅St)St=f(U⋅Xt+W⋅St−1)\\begin{aligned} &O_{t}=g\\left(V \\cdot S_{t}\\right) \\\ &S_{t}=f\\left(U \\cdot X_{t}+W \\cdot S_{t-1}\\right) \\end{aligned}

要注意的是，在计算时，**每一步使用的参数U、W、b都是一样的，也就是说每个步骤的参数都是共享的，这是RNN的重要特点，一定要牢记。** 

### 4\. n-to-one

要处理的问题输入是一个序列，输出是一个单独的值而不是序列，应该怎样建模呢？实际上，我们只在最后一个h上进行输出变换就可以了：

![](https://pic2.zhimg.com/v2-4e4921067e6ffff5c49fecebef773329_b.jpg)

### 5\. **Encoder-Decoder**

RNN的一种结构是 **n-to-m**，即输入、输出为不等长的序列，也称为 **Encoder-Decoder**， 这种结构常见于机器翻译中，因为源语言和目标语言的句子往往并没有相同的长度。

![](https://pic3.zhimg.com/v2-d14aff06a82ab7c998c148f95c21d6aa_b.jpg)

如上图所示，这种**Encoder-Decoder**结构通常由两个RNN构成，一个RNN用来Encode，另一个RNN用来Decode。

首先：Encoder-Decoder结构先将输入数据编码成一个上下文语义向量c；语义向量c可以有多种表达方式，最简单的方法就是把Encoder的最后一个隐状态赋值给c，还可以对最后的隐状态做一个变换得到c，也可以对所有的隐状态做变换。

之后：就用另一个RNN网络对其进行解码，这部分RNN网络被称为Decoder。Decoder的RNN可以与Encoder的一样，也可以不一样。具体做法就是将c当做之前的初始状态h0输入到Decoder中，还有一种做法是将c当做每一步的输入。

**Encoder-Decoder的缺点**

*   最大的局限性：编码和解码之间的唯一联系是固定长度的语义向量c  
    
*   编码要把整个序列的信息压缩进一个固定长度的语义向量c  
    
*   语义向量c无法完全表达整个序列的信息
*   先输入的内容携带的信息，会被后输入的信息稀释掉，或者被覆盖掉
*   输入序列越长，这样的现象越严重，这样使得在Decoder解码时一开始就没有获得足够的输入序列信息，解码效果会打折扣

因此，为了弥补基础的 Encoder-Decoder 的局限性，提出了attention机制。

二、Attention 机制
--------------

为了解决这一由长序列到定长向量转化而造成的信息损失的瓶颈，Attention注意力机制被引入了。

Attention机制跟人类翻译文章时候的思路有些类似，即将注意力关注于我们翻译部分对应的上下文。同样的，Attention模型中，当我们翻译当前词语时，我们会寻找源语句中相对应的几个词语，并结合之前的已经翻译的部分作出相应的翻译，

如下图左所示，当我们翻译“**knowledge**”时，只需将注意力放在源句中“**知识**”的部分。这样，当我们decoder预测目标翻译的时候就可以看到encoder的所有信息，而不仅局限于原来模型中定长的隐藏向量，并且不会丧失长程的信息。

![](https://pic4.zhimg.com/v2-f95ce1ee244fa9d0f055f263d3158a93_b.jpg)

以上是直观理解，我们来详细的解释一下数学上对应哪些运算。下图是带有Attention机制的Decoder：

**每一个c会自动去选取与当前所要输出的y最合适的上下文信息。具体来说，我们用aij衡量Encoder中第j阶段的hj和解码时第i阶段的相关性，最终Decoder中第i阶段的输入的上下文信息 ci就来自于所有 hj 对 aij 的加权和。** 

![](https://pic4.zhimg.com/v2-90d426c298bb6e8e2c922f0430994f73_b.jpg)

图1

以机器翻译为例（将中文翻译成英文）：

输入的序列是“我爱中国”，因此，Encoder中的h1、h2、h3、h4就可以分别看做是“我”、“爱”、“中”、“国”所代表的信息。在翻译成英语时，第一个上下文c1应该和 “我” 这个字最相关，因此对应的 a11 就比较大，而相应的 a12、a13、a14 就比较小。c2应该和“爱”最相关，因此对应的 a22 就比较大。最后的c3和h3、h4最相关，因此 a33、a34 的值就比较大。

至此，关于Attention模型，只剩最后一个问题了，那就是：这些权重 **aij** 是怎么来的？

![](https://pic4.zhimg.com/v2-d701aa26fad8808b3dd438317e0cff93_b.jpg)

图2

1.  首先我们利用RNN结构得到encoder中的**hidden state**(h1,h2,…,hT)\\left(h_{1}, h_{2}, \\ldots, h_{T}\\right) （图1所示）
2.  由图2所示，设当前**decoder**的**hidden state** 是 ht−1′h^{\\prime}_{t-1} ，我们可以计算每一个**encoder**中的**hidden stat** ht−1h_{t-1}与当前**hidden state**的关联性: etj=a(ht−1′,hj)e_{t j}=a\\left(h^{\\prime}_{t-1}, h_{j}\\right) ，写成相应的向量形式即为 et→=(a(ht−1′,h1),…,a(ht−1′,hT))\\overrightarrow{e_{t}}=\\left(a\\left(h^{\\prime}_{t-1}, h_{1}\\right), \\ldots, a\\left(h^{\\prime}_{t-1}, h_{T}\\right)\\right) 。其中 aa 是一种相关性的算符，例如常见的有点乘形式 et→=ht−1→Th→\\overrightarrow{e_{t}}=\\overrightarrow{h_{t-1}}^{T} \\vec{h} , 加权点乘 et→=ht−1→TWh→\\overrightarrow{e_{t}}=\\overrightarrow{h_{t-1}}^{T} W \\vec{h} ，加和 et→=v→Ttanh⁡(W1h→+W2ht−1→)\\overrightarrow{e_{t}}=\\vec{v}^{T} \\tanh \\left(W_{1} \\vec{h}+W_{2} \\overrightarrow{h_{t-1}}\\right) 等等。  
    对于 et→\\overrightarrow{e_{t}} 进行softmax操作得到attention的分布， αt→=softmax⁡(et→)\\overrightarrow{\\alpha_{t}}=\\operatorname{softmax}\\left(\\overrightarrow{e_{t}}\\right) ，展开形式为 αtj=exp⁡(etj)∑k=1Texp⁡(etk)\\alpha_{t j}=\\frac{\\exp \\left(e_{t j}\\right)}{\\sum_{k=1}^{T} \\exp \\left(e_{t k}\\right)}
3.  利用 αt→\\overrightarrow{\\alpha_{t}} 我们可以进行加权求和得到相应的context vector ct→=∑j=1Tαtjhj\\overrightarrow{c_{t}}=\\sum_{j=1}^{T} \\alpha_{t j} h_{j}**（上图图1）**
4.  由此，我们可以计算**decoder**的下一个**hidden state**ht=f(ht−1,yt−1,ct)h_{t}=f\\left(h_{t-1}, y_{t-1}, c_{t}\\right) 以及该位置的输出 p(yt∣y1,…,yt−1,x→)=g(yi−1,si,ci)p\\left(y_{t} \\mid y_{1}, \\ldots, y_{t-1}, \\vec{x}\\right)=g\\left(y_{i-1}, s_{i}, c_{i}\\right)
    
    **以上就是带有Attention的Encoder-Decoder模型计算的全过程。** 
    
    **当然，一个自然的疑问是，Attention机制如此有效，那么我们可不可以去掉模型中的RNN部分，仅仅利用Attention呢?**
    

三、self-attention机制
------------------

在上一章中，我们研究了注意力机制——现代深度学习模型中无处不在的方法。在这篇文章中，我们将介绍**The Transformer——**Google Brain团队在2017年提出的一种新的神经网络架构，该架构只使用了attention机制（+MLPs），不需要RNN、CNN等复杂的神经网络架构，并行度高。

### **1\. 从黑盒开始**

首先将模型视为一个黑盒。在机器翻译应用程序中，它会用一种语言输入一个句子，然后用另一种语言输出它的翻译：

![](https://pic3.zhimg.com/v2-4ffeb4e3b8e785cdfa32ba504f443d12_b.jpg)

### **2\. 细致一点**

如果我们稍微分析一下Transformer的内部，我们会看到一个Decoders、一个Encoders以及它们之间的连接。

![](https://pic2.zhimg.com/v2-3e6ac22ee89a78d740663eccc8129251_b.jpg)

如下图，Encoders是一堆Encoder（纸上将其中的六个堆叠在一起——数字 6 没有什么神奇之处，我们可以尝试其他的数字）。Decoders是一堆和Encoders数量相同的Decoder。

![](https://pic3.zhimg.com/v2-4733414be81c0c5953a3dc9f52d58be6_b.jpg)

Encoder在结构上都是相同的（但它们不共享权重）。每一层又分为两个子层：

![](https://pic3.zhimg.com/v2-85f56ebecffcda4b45e31c61911ac28e_b.jpg)

**3\. 注意！self-Attention出现了！**

Encoder的输入首先流经Self-Attention——该层帮助Encoder在编码特定单词时查看输入句子中的其他单词。我们将在后面的文章中仔细研究Self-Attention。

Self-Attention的输出被输送到前馈神经网络。完全相同的前馈网络独立应用于每个位置。

Dncoder同样具有这两层，但它们之间是一个Attention层，它帮助Dncoder专注于输入句子的相关部分（类似于seq2seq 模型中的注意力）

![](https://pic4.zhimg.com/v2-da7ce9049ff5df578ed70d72821f1c2b_b.jpg)

### **4\. 让我们看看张量是如何在Transformer中流动的！**

首先使用词嵌入算法将每个输入词转换为向量。embedding仅发生在最底层的Encoder中。但在其他Encoder中，它的输入是直接位于下方的Encoder的输出。

![](https://pic1.zhimg.com/v2-195a41e54a6d9a69612c96decb766f60_b.jpg)

对于 Encoder 侧，首先，6个大的模块之间是串行的，一个模块计算的结果做为下一个模块的输入，互相之前有依赖关系。

从每个模块的角度来说，注意力层和前馈神经层这两个子模块单独来看都是可以并行的，不同单词之间是没有依赖关系的。

当然对于注意力层在做 attention 的时候会依赖别的时刻的输入，不过这个只需要在计算之前就可以提供。

然后注意力层和前馈神经层之间是串行，必须先完成注意力层计算再做前馈神经层。

**简单讲，就是6个 encoder 之间是串行，每个 encoder 中的两个子模块之间是串行，子模块自身是可以并行的。** 

### **5\. Self-Attention的细节探讨**

![](https://pic4.zhimg.com/v2-dd7b4a753b4ea4cbbb2223727d1263cb_b.jpg)

首先从每个Encoder的每个词输入向量的创建三个向量。具体来说，我们对每个词创建一个 Query 向量、一个 Key 向量和一个 Value 向量。这些向量是通过将嵌入向量乘以我们在训练过程中训练的三个矩阵来创建的。

注意，这些新向量的维度比嵌入向量要小。它们的维数是 64，而嵌入和编码器输入/输出向量的维数是 512。它们不必更小，这是一种架构选择，可以使多头注意力的计算（大部分）保持不变。

然后计算得分。假设我们正在计算此示例中第一个单词“Thinking”的self-attention。我们需要根据这个词对输入句子的每个词进行评分。当我们在某个位置对单词进行编码时，分数决定了将多少注意力放在输入句子的其他部分上。

分数是通过Query向量与我们正在评分的各个单词的Key向量的点积来计算的。因此，如果我们正在处理位置#1 中单词“Thinking”的自注意力，第一个分数将是q1和k1的点积。第二个分数是q1和k2的点积。

![](https://pic3.zhimg.com/v2-49eda9a091db55a73c2f81a71029e1d6_b.jpg)

接下来是除以8（64的平方根）（这可以使得梯度更加稳定），然后经过 softmax操作，Softmax 对分数进行归一化，因此它们都是正数，加起来为 1。

![](https://pic1.zhimg.com/v2-3edf970f0f3d5895b071c31e70e53e14_b.jpg)

然后，将该位置的SOFTMAX得分乘以每个位置的value向量，将他们加起来，得到该位置的自注意力层的输出。

![](https://pic4.zhimg.com/v2-68bc8d52d5babe70de1033b44229debf_b.jpg)

### **6\. Self-Attention的矩阵计算**

在实际实现中，为了更快的处理，这个计算是以矩阵形式完成的。  
第一步是计算Query、Key和Value矩阵。我们通过将我们的嵌入词向量打包成一个矩阵X，该矩阵中的每一行对应于输入句子中的一个词。并将其乘以我们训练的权重矩阵（WQ、WK、WV）来实现。

![](https://pic3.zhimg.com/v2-7ea227cf2708e5ecea54cbe3d7cf7056_b.jpg)

**然后，**我们可以将剩余步骤浓缩到一个公式中来计算自注意力层的输出。

![](https://pic1.zhimg.com/v2-b1382753be347779f9c482217e2944d8_b.jpg)

**_该论文还通过添加一种称为“多头”注意力的机制进一步细化了自注意力层~_**

**四、多头注意力机制**
-------------

该论文通过添加一种称为“多头注意力“的机制进一步完善了自注意力层，通过两种方式提高了注意力层的性能：

1.  它扩展了模型关注不同位置的能力。比如，在上面的示例中，z1 包含一些其他编码信息，但它可能由实际单词本身主导。如果我们翻译“The animal didn' t cross the street because it was too tired”这样的句子会很有用，我们会想知道“它”指的是哪个词。
2.  它为注意力层提供了多个“表示子空间”。正如我们接下来将看到的，通过多头注意力，我们有8组Query/ Key/ Value 权重矩阵（Transformer 使用8个注意力头，因此每个Encoder/ Decoder最终有8个集合）。这些集合中的每一个都是随机初始化的。然后，在训练之后，每个集合用于将嵌入词向量（或来自较Encoder /Decoder的向量）投影到不同的表示子空间中。  
    

具体步骤如下图所示：

![](https://pic4.zhimg.com/v2-574724e70a44f6256dadd71a1732bc5b_b.jpg)

1）首先，将8组不同的权重矩阵与嵌入词向量（或来自较Encoder /Decoder的向量）相乘，得到8组K、Q、V矩阵。

2）之后，我们连接这些矩阵，然后将它们乘以一个额外的权重矩阵 WO，得到最后的Z矩阵。

例子：

![](https://pic3.zhimg.com/v2-cfb58029d2d4164f462e22063635fa6a_b.jpg)

可以看到当我们对“it”这个词进行编码时，一个注意力头最关注“animal”，而另一个注意力头最关注“tired”——从某种意义上说，模型对“it”这个词的表示包含了“animal”和“tired”两种表示。

五、位置编码
------

**1.使用位置编码表示序列的顺序**

直到目前为止所描述的，Transformer模型中缺少的一件事就是一种解释输入序列中单词顺序的方法。

在以前的模型中，NLP的每个Sequence都是一个token一个token的输入到模型当中。比如有一句话是“我喜欢吃洋葱”，那么输入模型的顺序就是“我”，“喜”，“欢“，”吃“，”洋“，”葱”，一个字一个字的。

![](https://pic4.zhimg.com/v2-c3b8c50a5044afff49c143c19e3407c7_b.jpg)

Transformer模型引用了Self-attention来解决这个问题。Self-attention的输入方式如下：

![](https://pic4.zhimg.com/v2-cb25f3f06881f463732d2b5bbb6e4147_b.jpg)

我们可以看到，对于Self-attention结果而言，它可以一次性的将所有的字都当做输入。但是NLP的输入是有特点的，其特点是输入的文本要按照一定的顺序才可以。因为，文本的顺序是带有一部分语义关系的。比如下面两句话，不同的语序就有不同的语义。

句子1：我喜欢吃洋葱  
句子2：洋葱喜欢吃我

所以，对于Transformer结构而言，为了更好的发挥并行输入的特点**，首先要解决的问题就是要让输入的内容具有一定的位置信息**。在原论文中，为了引入位置信息，加入了**Position机制**。

**2.位置编码原则**

**方法一：** 

这个方法的分配方式是，将0-1这个范围的，将第一个token分配0，最后一个token分配去1，其余的token按照文章的长度平均分配。具体形式如下：

我喜欢吃洋葱 【0 0.16 0.32.....1】  
我真的不喜欢吃洋葱【0 0.125 0.25.....1】

问题：我们可以看到，如果句子长度不同，那么位置编码是不一样，所以无法表示句子之间有什么相似性。

**方法二：1-n正整数范围分配**

这个方法比较直观，就是按照输入的顺序，一次分配给token所在的索引位置。具体形式如下：

> 我喜欢吃洋葱 【1，2，3，4，5，6】  
> 我真的不喜欢吃洋葱【1，2，3，4，5，6，7】

问题：往往句子越长，后面的值越大，数字越大说明这个位置占的权重也越大，这样的方式无法凸显每个位置的真实的权重。

过去的方法总有这样或者那样的不好，所以Transformer对于位置信息的编码做了改进。

一种好的位置编码方案需要满足以下几条要求：

*   它能为每个时间步输出一个独一无二的编码；
*   不同长度的句子之间，任何两个时间步之间的距离应该保持一致；
*   模型应该能毫不费力地泛化到更长的句子。它的值应该是有界的；
*   它必须是确定性的。

**3\. Transformer的相对位置**

相对位置即关注一个token与另一个token距离差几个token。比如：位置1和位置2的距离比位置3和位置10的距离更近，位置1和位置2与位置3和位置4都只相差1。

还是按照上面"我喜欢吃洋葱"中的“我”为例，看看相对位置关系是什么样子的：

![](https://pic4.zhimg.com/v2-cd2ec0b4f0c4ee716243b5458bafa9d3_b.jpg)

  
我们可以看到，使用相对位置的方法，我们可以清晰的知道单词之间的距离远近的关系。

Transformer的作者们提出了一个简单但非常创新的位置编码方法，能够满足上述所有的要求。首先，这种编码不是单一的一个数值，而是包含句子中特定位置信息的 d 维向量（非常像词向量），向量如下：

\\overrightarrow{p_{t}}=\\left\[\\begin{array}{c} \\sin \\left(\\omega_{1} \\cdot t\\right) \\\ \\cos \\left(\\omega_{1} \\cdot t\\right) \\\ \\sin \\left(\\omega_{2} \\cdot t\\right) \\\ \\cos \\left(\\omega_{2} \\cdot t\\right) \\\ \\vdots \\\ \\sin \\left(\\omega_{d / 2} \\cdot t\\right) \\\ \\cos \\left(\\omega_{d / 2} \\cdot t\\right) \\end{array}\\right\]  
将这个向量与模型的嵌入词向量 \\psi\\left(w_{t}\\right) 直接相加喂给模型： \\psi^{\\prime}\\left(w_{t}\\right)=\\psi\\left(w_{t}\\right)+\\overrightarrow{p_{t}}

为了保证这种相加操作正确，让位置向量（PE）的维度等于词向量（WE）的维度。

**关于每个元素的说明：** 

1) 其中，频率 w_{k} 定义如下：

\\omega_{k}=\\frac{1}{10000^{2 k / d}}

2) 这里的t就是每个token的位置，比如说是位置1，位置2，以及位置n

**为什么可以表示相对距离？**

这样的位置信息表示方法可以表示不同距离token的相对关系。这里我们通过数学来证明。

假设：某一个token的位置是 POS ，如果另一个一个token表示为 POS+k，那就表明这个位置距上一个token为 k。如果这时我们需要看看一个位置POS和 POS+k这两个字符的关系。按照位置编码的的公式，我们可以计算的位置编码，其结果如下：  
\\begin{aligned} &P E_{(p o s+k, 2 i)}=\\sin \\left(w_{i} \\cdot(p o s+k)\\right)=\\sin \\left(w_{i} p o s\\right) \\cos \\left(w_{i} k\\right)+\\cos \\left(w_{i} p o s\\right) \\sin \\left(w_{i} k\\right) \\\ &P E_{(p o s+k, 2 i+1)}=\\cos \\left(w_{i} \\cdot(p o s+k)\\right)=\\cos \\left(w_{i} p o s\\right) \\cos \\left(w_{i} k\\right)-\\sin \\left(w_{i} p o s\\right) \\sin \\left(w_{i} k\\right) \\end{aligned}

我们可以看看上面公式中，有一部分是似曾相识的：

\\begin{gathered} P E_{(p o s, 2 i)}=\\sin \\left(\\frac{p o s}{10000 \\frac{2 i}{d_{\\text {moded }}}}\\right) \\\ P E_{(p o s, 2 i+1)}=\\cos \\left(\\frac{p o s}{10000^{\\frac{2 i}{d_{\\text {model }}}}}\\right) \\end{gathered}

根据上面的公式我们可以看出，似曾相识的部分带入 P E_{\\text {pos+k }} 的公式中，代入之后的结果如下：

\\begin{gathered} P E_{(p o s+k, 2 i)}=\\cos \\left(w_{i} k\\right) P E_{(p o s, 2 i)}+\\sin \\left(w_{i} k\\right) P E_{(p o s, 2 i+1)} \\\ P E_{(p o s+k, 2 i+1)}=\\cos \\left(w_{i} k\\right) P E_{(p o s, 2 i+1)}-\\sin \\left(w_{i} k\\right) P E_{(p o s, 2 i)} \\end{gathered}

我们可以知道，距离K是一个常数，所以上面公式 \\sin () 和 \\cos () 的计算值也是常数，可以表示为：

u=\\cos \\left(w_{i} \\cdot k\\right), v=\\sin \\left(w_{i} \\cdot k\\right)

这样，就可以将 P E_{\\text {pos+k }} 写成一个矩阵的乘法。

\\left\[\\begin{array}{c} P E_{(p o s+k, 2 i)} \\\ P E_{(p o s+k, 2 i+1)} \\end{array}\\right\]=\\left\[\\begin{array}{cc} u & v \\\ -v & u \\end{array}\\right\] \\times\\left\[\\begin{array}{c} P E_{(p o s, 2 i)} \\\ P E_{(p o s, 2 i+1)} \\end{array}\\right\]

可以从上面的矩阵乘法角度看到，位置POS的编码与位置POS+k的编码是线性关系。

那么问题来了，上面的操作也只可以看到线性关系，怎么可以更直白地知道每个token的距离关系？

为了解答上面的问题，我们将 PE_{pos } 和 PE( pos +k), 相乘 (两个向量相乘)，可以得到如下结果：

\\begin{aligned} P E_{p o s} \\cdot P E_{p o s+k} &=\\sum_{i=0}^{\\frac{d}{2}-1} \\sin \\left(w_{i} p o s\\right) \\cdot \\sin \\left(w_{i}(p o s+k)\\right)+\\cos \\left(w_{i} p o s\\right) \\cdot \\cos \\left(w_{i}(p o s+k)\\right) \\\ &=\\sum_{i=0}^{\\frac{d}{2}-1} \\cos \\left(w_{i}(p o s-(p o s+k))\\right.\\\ &=\\sum_{i=0}^{\\frac{d}{2}-1} \\cos \\left(w_{i} k\\right) \\end{aligned}

我们发现相乘后的结果为一个余弦的加和。这里影响值的因素就是k。如果两个token的距离越大，也就是K越大，根据余弦函数的性质可以知道，两个位置向量的相乘结果越小。这样的关系可以得到，如果两个token距离越远则乘积的结果越小。

这样的方式虽说可以表示出相对的距离关系，但是也是有局限的。其中一个比较大的问题是：只能的到相对关系，无法得到方向关系。所谓的方向关系就是，对于两个token谁在谁的前面，或者谁在谁的后面是无法判断的。数学表示如下：

P E_{\\text {postk }} P E_{\\text {pos }}=P E_{\\text {pos-k }} P E_{\\text {pos }}

六、残差模块
------

上一篇我们已经说完了Transformer中encoder的核心，这一篇我们主要讲一下什么是残差模块？它为了解决什么问题？以及mulit -head self-attention输出后为什么接了一个残差模块？

![](https://pic3.zhimg.com/v2-aa13d93ac7b0359f5f8ae7a30bb3560a_b.jpg)

### **1\. 梯度弥散以及梯度爆炸**

现代神经网络一般是通过基于梯度的BP算法来优化，对前馈神经网络而言，一般需要前向传播输入信号，然后以目标的负梯度方向对参数进行调整，参数的更新为 w \\leftarrow w+\\Delta w

给定学习率 \\alpha ，得出 \\Delta w=-\\alpha \\frac{\\partial L o s s}{w} 。如果更新第二层隐层的权值信息，根据链式求导法则，更新梯度信息为:

\\Delta w_{1}=\\frac{\\partial L oss}{\\partial w_{2}}=\\frac{\\partial L o ss}{\\partial f_{4}} \\cdot \\frac{\\partial f_{4}}{\\partial f_{3}} \\cdot \\frac{\\partial f_{3}}{\\partial f_{2}} \\cdot \\frac{\\partial f_{2}}{\\partial w_{2}}

上式中, \\frac{\\partial f_{2}}{\\partial w_{2}}=f_{1} ，即第二隐藏层的输入。 \\frac{\\partial f_{4}}{\\partial f_{3}} 就是对激活函数进行求导，如果此部分大于1，那么层数增多的时候，最终的求出的梯度更新将以指数形式增加，即发生梯度爆炸，如果此部分小于1，那么随着层数增多，求出的梯度更新信息将会以指数形式衰减，即发生了梯度消失。

对于该问题的解决方法是正则化初始化和中间的正则化层（Batch Normalization），这样的话可以训练几十层的网络。（原文：This problem,however, has been largely addressed by normalized initialization and intermediate normalization layers, which enable networks with tens of layers to start converging for stochastic gradient descent (SGD) with backpropagation）

### 2\. **网络退化**

虽然通过上述方法能够训练了，但是又会出现另一个问题，就是退化问题，网络层数增加，但是在训练集上的准确率却饱和甚至下降了。**这个不能解释为overfitting**，因为overfit应该表现为在训练集上表现更好才对。

**退化问题说明了深度网络不能很简单地被很好地优化。** 

### **3\. 残差网络**

残差操作这一思想起源于论文《Deep Residual Learning for Image Recognition》，目前的引用量已达3万多。这篇文章发现，如果存在某个K层的网络f是当前最优的网络，那么可以构造一个更深的网络，其最后几层仅是该网络f第K层输出的恒等映射(Identity Mapping)，就可以取得与f一致的结果；也许K还不是所谓“最佳层数”，那么更深的网络就可以取得更好的结果。总而言之，与浅层网络相比，更深的网络的表现不应该更差**。因此，一个合理的猜测就是，对神经网络来说，恒等映射的某些层会导致模型不容易拟合。** 

### **4\. 残差网络是如何解决退化问题的？**

残差网络通过加入 ，变得更加容易被优化。包含一个 shortcut connection 的几层网络被称为一个残差块（residual block）。一个残差块 (shortcut connections/skip connections) 分为直接映射部分 \\left(x_{l}\\right) 和残差部分 F\\left(x_{l}, W_{l}\\right) ，可以表示为: x_{l+1}=x_{l}+F\\left(x_{l}, W_{l}\\right)

![](https://pic1.zhimg.com/v2-359c271137045c021bebf95a16545fd0_b.jpg)

当没有 shortcut connection（即图右侧从 x到⨁的箭头）时，残差块就是一个普通的 2 层网络。残差块中的网络可以是全连接层，也可以是卷积层。设第二层网络在激活函数之前的输出为 H(x)。如果在该 2 层网络中，最优的输出就是输入 x，那么对于没有 shortcut connection 的网络，就需要将其优化成 H(x)=x；对于有 shortcut connection 的网络，即残差块，如果最优输出是 x，则只需要将 **F(x)=H(x)−x** 优化为 0 即可。**后者的优化会比前者简单。这也是残差这一叫法的由来。** 

一个通俗的解释：

**F是求和前网络映射，H是从输入到求和后的网络映射。比如把5映射到5.1，那么引入残差前是F’(5)=5.1，引入残差后是H(5)=5.1, H(5)=F(5)+5, F(5)=0.1。这里的F’和F都表示网络参数映射，引入残差后的映射对输出的变化更敏感。比如s输出从5.1变到5.2，映射F’的输出增加了1/51=2%，而对于残差结构输出从5.1到5.2，映射F是从0.1到0.2，增加了100%。明显后者输出变化对权重的调整作用更大，所以效果更好。残差的思想都是去掉相同的主体部分，从而突出微小的变化。** 

**七、**归一化
---------

经过了残差模块后，Transformer还对残差模块输出进行了Normalization。

![](https://pic3.zhimg.com/v2-339bec59624fb231be38049a978d0eca_b.jpg)

### **1\. 什么是Normalization（归一化）？**

数据标准化（Normalization），也称为归一化，归一化就是将你需要处理的数据在通过某种算法经过处理后，限制将其限定在你需要的一定的范围内。

### **2\. 为什么要进行归一化？**

神经网络学习过程的本质就是为了学习数据分布，如果我们没有做归一化处理，那么每一批次训练数据的分布不一样，从大的方向上看，神经网络则需要在这多个分布中找到平衡点，从小的方向上看，由于每层网络输入数据分布在不断变化，这也会导致每层网络在找平衡点，显然，神经网络就很难收敛了。

主要包括以下几种方法：BatchNorm（2015年）、LayerNorm（2016年）、InstanceNorm（2016年）、GroupNorm（2018年）。

![](https://pic4.zhimg.com/v2-3aeecbe5d067c522af072e1d7657d88b_b.jpg)

### **3\. Batch Normalization-BN**

针对一个Batch，在同一维度的特征进行feature scaling。

缺点：

batch size较小的时候，效果差，因为其原理为用一个batch size的均值方差模拟整个数据分布的均值方差，如果batch size较小，其数据分布与整个数据分布差别较大。

下面是一个batch size=2案例，按BN方式，在第一”维度“进行归一化的话就是将”你“和”机“的特征进行归一化，但这明显不是一个维度的信息。显然BN在此处使用是很不合理的。NLP中同一batch样本的信息关联不大（差异很大，但要学习的就是这种特征），更多应该概率句子内部（单个样本内部）维度的归一化。

![](https://pic3.zhimg.com/v2-aa4fba8490b6dee59c3f974c3a968d7e_b.jpg)

### **4\. Layer Normalization-LN（Transformer中所使用的）**

单独对一个样本的所有单词作缩放，与batch normalization的方向垂直，对RNN作用明显。

**为什么图像处理用batch normalization效果好，而自然语言处理用 layer normalization好？**

CV使用BN是认为不同卷积核feature map（channel维）之间的差异性很重要，LN会损失channel的差异性，对于batch内的不同样本，同一卷积核提取特征的目的性是一致的，所以使用BN仅是为了进一步保证同一个卷积核在不同样本上提取特征的稳定性。

而NLP使用LN是认为batch内不同样本同一位置token之间的差异性更重要，而embedding维，网络对于不同token提取的特征目的性是一致的，使用LN是为了进一步保证在不同token上提取的稳定性。

八、Decoder层
----------

Decoder内部的小模块与Encoder的看上去差不多，但实际上运行方式差别很大，这里我们先来看一下Decoder内部多头注意力机制中的一个特别的机制——Mask（掩膜）机制。

![](https://pic1.zhimg.com/v2-5c4f689901db40208e80bdb968e7718c_b.jpg)

### **1\. Transformer中的Mask机制**

Mask机制经常被用于NLP任务中，按照作用总体来说可以分成两类：

1.  用于处理非定长序列的padding mask
2.  用于防止标签泄露的sequence mask

**Transformer中同时用到了这两种Mask机制。** 

### **2\. padding mask**

在NLP任务中，文本通常是不定长的，所以在输入一个样本长短不一的batch到网络前，要对batch中的样本进行truncating截断/padding补齐操作，以便能形成一个张量的形式输入网络，如下图所示。对于一个batch中过长的样本，进行截断操作，而对于一个长度不足的样本，往往采用特殊字符"<PAD>"进行padding（也可以是其他特殊字符，但是pad的字符要统一）。Mask矩阵中可以用1表示有效字，0代表无效字（也可以用True/False）。

![](https://pic1.zhimg.com/v2-b97a738f2b74959eab8cf447fc17fb64_b.jpg)

**padding mask生成和使用**

1.  padding（补齐）操作在batch输入网络前完成，同步生成padding mask矩阵；
2.  padding mask矩阵常常用在最终结果输出、损失函数计算等等，一切受样本实际长度影响的计算中。

### **3\. sequence mask （Transformer训练阶段、Decoder模块）**

Sequence Mask只在Decoder端进行,目的是为了使得decoder不能看见未来的信息.也就是对于一个序列中的第i个token,解码的时候只能够依靠i时刻之前(包括i)的的输出,而不能依赖于i时刻之后的输出.因此我们要采取一个遮盖的方法(Mask)使得其在计算self-attention的时候只用i个时刻之前的token进行计算,因为Decoder是用来做预测的,而在训练预测能力的时候,我们不能够"提前看答案",因此要将未来的信息给遮盖住. 该处理被称为 Mask

举个例子，Decoder 的 ground truth 为 "<start> I am fine"，我们将这个句子输入到 Decoder 中，经过 WordEmbedding 和 Positional Encoding 之后，将得到的矩阵做三次线性变换（WQ,WK,WV）。然后进行 self-attention 操作，  
首先通过 \\frac{Q K^{T}}{\\sqrt{d_{k}}} 得到 Scaled Scores，接下来非常关键，我们要对 Scaled Scores 进行 Mask，举个例子，当我们输入 "I" 时，模型目前仅知道包括 "I" 在内之前所有字的信息，即 "<start>" 和 "I" 的信息，不应该让其知道 "I" 之后词的信息。道理很简单，我们做预测的时候是按照顺序一个字一个字的预测，怎么能这个字都没预测完，就已经知道后面字的信息了呢？Mask 非常简单，首先生成一个下三角全 0，上三角全为负无穷的矩阵，然后将其与 Scaled Scores 相加即可：

![](https://pic3.zhimg.com/v2-3df4cc74988aacd17d5393665b5fc2be_b.jpg)

之后再做 softmax，就能将 - inf 变为 0，得到的这个矩阵即为每个字之间的权重。

![](https://pic1.zhimg.com/v2-b9609c365c6497632debb775b8463b20_b.jpg)

**再次强调,sequence mask 操作只发生在decoder阶段,而且对于self-attenion而言,这是一个必要的操作,不像之前的PADDING_MASK可以选择跳过。** 

**九、最终的Linear和 Softmax 层**
--------------------------

![](https://pic1.zhimg.com/v2-99dd80f63b6866d1a2064507833a070c_b.jpg)

Decoder的最后一个部分是过一个linear layer将decoder的输出扩展到与vocabulary size一样的维度上。经过softmax 后，选择概率最高的一个word作为预测结果。假设我们有一个已经训练好的网络，**在做预测时，步骤如下：** 

1.  给 decoder 输入 encoder 对整个句子 embedding 的结果 和一个特殊的开始符号 </s>。decoder 将产生预测，在我们的例子中应该是 ”I”。
2.  给 decoder 输入 encoder 的 embedding 结果和 “</s>为”，在这一步 decoder 应该产生预测 “什”。
3.  给 decoder 输入 encoder 的 embedding 结果和 “</s>为什”，在这一步 decoder 应该产生预测 “么”。
4.  给 decoder 输入 encoder 的 embedding 结果和 “</s>为什么”，在这一步 decoder 应该产生预测 “要”
5.  给 decoder 输入 encoder 的 embedding 结果和 “</s>为什么要”，在这一步 decoder 应该产生预测 “工”。
6.  给 decoder 输入 encoder 的 embedding 结果和 “</s>为什么要工”，在这一步 decoder 应该产生预测 “作”。
7.  给 decoder 输入 encoder 的 embedding 结果和 “</s>为什么要工作”, decoder应该生成句子结尾的标记，decoder 应该输出 ”</eos>”。
8.  然后 decoder 生成了 </eos>，翻译完成。

十、BERT
------

BERT (Bidirectional Encoder Representations from Transformers) 是Google AI Language 的研究人员最近发表的[一篇论文。](https://link.zhihu.com/?target=https%3A//arxiv.org/pdf/1810.04805.pdf)它在问答 (SQuAD v1.1)、自然语言推理 (MNLI) 等各种 NLP 任务中展都超越了目前最先进的结果，在机器学习社区引起了轰动。

BERT 的关键技术创新是将流行的注意力模型 Transformer 的双向训练应用于语言建模。论文的结果表明，双向训练的语言模型比单向语言模型可以更深入地感知语言上下文环境。在论文中，研究人员详细介绍了一种名为 Masked LM (MLM) 的新技术，该技术能够使得模型进行双向训练。

BERT 主要基于的是 Transformer，这是一种学习文本中单词（或子词）之间上下文关系的注意力机制（具体请看前几章）。Transformer 包括两个独立的机制——一个读取文本输入的编码器和一个为任务生成预测的解码器。由于 BERT 的目标是生成语言模型，**因此只需要编码器机制**。谷歌的[一篇论文](https://link.zhihu.com/?target=https%3A//arxiv.org/pdf/1706.03762.pdf)中描述了 Transformer 的详细工作原理。

与按顺序（从左到右或从右到左）读取文本输入的定向模型相反，Transformer 编码器一次读取整个单词序列。因此称它为双向的，**尽管说它是非定向的会更准确**。这一特性允许模型根据其上下文环境（单词的左侧和右侧）来学习单词的上下文。

许多模型预测序列中的下一个单词（例如 “The child came home from ___”），**这种定向方法本质上限制了上下文学习**。为了克服这一挑战，BERT 使用了两种训练策略：

### 1\. 掩码语言模型 (MLM)

在将单词序列输入 BERT 之前，每个序列中 15% 的单词被替换为 \[MASK\] 标记。然后，该模型尝试根据序列中其他非掩码单词提供的上下文来预测掩码单词的原始值。具体来讲，输出词的预测需要：

1.  在编码器输出之上添加一个分类层。
2.  将输出向量乘以嵌入矩阵，将它们转换为词汇维度。
3.  用 softmax 计算词汇表中每个单词的概率。  
    

![](https://pic4.zhimg.com/v2-9b0d056bcd745675d638c3a0e537b857_b.jpg)

BERT 损失函数只考虑了对掩码值的预测，而忽略了对非掩码词的预测。因此，该模型的收敛速度比定向模型慢，这一特征被其增强的上下文感知所抵消（参见要点 3）。

**_注意：在实践中，BERT 的实现稍微复杂一些，并没有取代所有 15% 的掩码词。_**

### 2\. 下一句预测（NSP）

在 BERT 训练过程中，模型接收成对的句子作为输入，并学习预测该对中的第二个句子是否是原始文档中的后续句子。在训练期间，50% 的输入是一对句子，其中第二个句子是原始文档中的后续句子，而在另外 50% 的输入中，从语料库中随机选择一个句子作为第二个句子，并假设该随机句子与第一句不相连。

为了帮助模型在训练中区分两个句子，输入在进入模型之前按如下方式处理：

1.  在第一句的开头插入一个 \[CLS\] 标记，在每个句子的结尾插入一个 \[SEP\] 标记。
2.  将指示句子 A 或句子 B 的sentence Embedding添加到每个标记中。**sentence Embedding在概念上类似于当词汇表长度为 2 的token embedding。** 
3.  positional embedding 被添加到每个标记以指示其在序列中的位置。Transformer 论文中介绍了positional embedding 的概念和实现。

为了预测第二个句子是否确实与第一个句子相关，执行以下步骤：

1.  整个输入序列通过 Transformer 模型。
2.  使用简单的分类层（权重和偏差的学习矩阵）将 \[CLS\] 标记的输出转换为 2×1 形状的向量。
3.  用 softmax 计算 IsNextSequence 的概率。

**在训练 BERT 模型时，Masked LM 和 Next Sentence Prediction 一起训练，目标是最小化这两种策略的组合损失函数。** 

### 3\. 如何使用 BERT（微调）

将 BERT 用于特定任务相对简单：

BERT 可用于多种语言任务，仅需在核心模型中添加一小层：

1.  情感分析等分类任务与 Next Sentence 分类类似，方法是在 \[CLS\] 令牌的 Transformer 输出之上添加一个分类层。
2.  在问答任务（例如 SQuAD v1.1）中，当需要接收与文本序列的问题，并且需要在序列中标记答案。使用 BERT，可以通过学习两个标记答案开始和结束的额外向量来训练问答模型。
3.  在命名实体识别 (NER) 中，接收文本序列并需要标记文本中出现的各种类型的实体（人、组织、日期等）。使用 BERT，可以通过将每个标记的输出向量输入到预测 NER 标签的分类层来训练 NER 模型。

在微调训练中，大多数超参数与 BERT 训练中保持一致，论文对需要调优的超参数给出了具体指导（第 3.5 节）。BERT 团队已经使用这种技术在各种具有挑战性的自然语言任务上取得了最先进的结果，详见论文的第 4 节。

### 4\. 要点

1.  _模型大小很重要，。_BERT\_large 拥有 3.45 亿个参数，是同类模型中最大的。它在小规模任务上明显优于 BERT\_base，BERT_base 使用相同的架构，“只有”1.1 亿个参数。
2.  _足够的训练数据，更多的训练步骤 == 更高的准确度。_例如，在 MNLI 任务上，与具有相同批量大小的 500K 步训练相比，在 1M 步（128,000 字批量大小）上训练时，BERT_base 准确度提高了 1.0%。
3.  _BERT 的双向方法 (MLM) 的收敛速度比从左到右的方法要慢_（因为每批中仅预测 15% 的单词），但经过少量预训练步骤后，双向训练的性能仍然优于从左到右的训练。