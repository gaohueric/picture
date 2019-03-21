

### 一、约定

- 本场 chat 不针对“差评君”说事，专注于技术讨论；
- 一如既往，编程语言用 Python3；
- 边动手实战边看。

### 二、Python 如何判断相等

我们先说说常用的洗稿方式：直接复制粘贴段落（低级）、复制粘贴并打乱句子顺序（中级）、复制粘贴用工具替换同义词、近义词或者人工通读全文，然后断章取义（高级）等。

假设有 2 篇文本，我们如何判断其相似性呢？首先想到的是逐字（字节也可以）去比较，但是这种方法显然效率太低，那还有更好的方法吗？有，既然逐字效率低，那我们就逐句吧，没有更好的办法，我们勉强假设逐句可以，那此时我们就可以使用 Python3 最常用的 2 种判断相等的方式：`is` 和 `==` 进行操作，然后通过比较结果 True 的多少来判断。

```
a = 61
b = 61
print(a is b)
结果：True

a = 61
b = 61
print(a == b)
结果：True

a = '61'
b = '61'
print(a is b)
结果：True

a = '61'
b = '61'
print(a == b)
结果：True

```

上面从数值型和字符型分别验证了在 Python3 下，`is` 和 `==` 2种方式判断相等是没问题的。

然而，此时你会说上面方式局限性很大，有没有更好的方式呢？当然有，下面我们先看看文本相似性有哪些度量方式，然后再进行相似性判断。

### 三、文本相似性的度量

#### 1. 基于距离的计算

- 欧氏距离（Euclidean Distance）

两个 n 维向量 a(x11,x12,⋯,x1n)a(x11,x12,⋯,x1n) 与 b(x21,x22,⋯,x2n)b(x21,x22,⋯,x2n) 间的欧氏距离：

![enter image description here](http://images.gitbook.cn/f09b06b0-67c5-11e8-8b00-f1fee5374bf5)，其中（k=1,2,3......n）。

- 曼哈顿距离（Manhattan Distance）

两个 n 维向量 a(x11,x12,⋯,x1n)a(x11,x12,⋯,x1n) 与 b(x21,x22,⋯,x2n)b(x21,x22,⋯,x2n) 间的曼哈顿距离：

![enter image description here](http://images.gitbook.cn/97ccb140-67c6-11e8-811b-0fa70a363339)，其中（k=1,2,3......n）。

- 切比雪夫距离（Chebyshev Distance）

两个 n 维向量 a(x11,x12,⋯,x1n)a(x11,x12,⋯,x1n) 与 b(x21,x22,⋯,x2n)b(x21,x22,⋯,x2n) 间的切比雪夫距离

![enter image description here](http://images.gitbook.cn/fb376fe0-67c6-11e8-812c-750794b244e2)

- 闵可夫斯基距离（Minkowski Distance）

两个 n 维变量 a(x11,x12,⋯,x1n)a(x11,x12,⋯,x1n) 与 b(x21,x22,⋯,x2n)b(x21,x22,⋯,x2n) 间的闵可夫斯基距离定义为：

![enter image description here](http://images.gitbook.cn/3a87eee0-67c7-11e8-8b00-f1fee5374bf5)

> - 其中 p 是一个变参数。
> - 当 p=1 时，就是曼哈顿距离。
> - 当 p=2 时，就是欧氏距离。
> - 当 p→∞ 时，就是切比雪夫距离。
> - 根据变参数的不同，闵氏距离可以表示一类的距离。

- 标准化欧氏距离 (Standardized Euclidean distance )

标准化欧氏距离是针对简单欧氏距离的缺点而作的一种改进方案。标准欧氏距离的思路：因为原始数据各维的分布是不一样的，所以先把各个分量都进行“标准化”。

标准化变量的数学期望为 0，方差为 1。因此样本集的标准化过程(standardization)用公式描述就是：

![enter image description here](http://images.gitbook.cn/346b0870-67c8-11e8-a826-513048f4c795)

其中：X 为原始值，m 为均值，s 为标准差。

两个 n 维向量 a(x11,x12,⋯,x1n)a(x11,x12,⋯,x1n) 与 b(x21,x22,⋯,x2n)b(x21,x22,⋯,x2n) 间的标准化欧氏距离的公式：

![enter image description here](http://images.gitbook.cn/e2ec7eb0-67c8-11e8-8b00-f1fee5374bf5)，其中（k=1,2,3......n）。

除了上面这些距离，当然还有其他一些表示距离的方法，比如马氏距离（Mahalanobis Distance）以及在信息编码中经常使用到的汉明距离（Hamming distance）等。

下面我们给出如何计算这些距离的方法： 引入需要的包：

```
import jieba
from sklearn.metrics.pairwise import pairwise_distances

```

下面我们用 2 句话去计算距离，先进行分词（这里没有去标点和停用词，实际项目需要考虑使用），然后转成向量，在进行距离计算：

```
seg1 = jieba.lcut("随着人工智能的发展，机器人变得越来越智能。")
seg2 = jieba.lcut("随着人工智能的发展，自动驾驶变得越来越智能。")

```

```
seg1分词后的结果：['随着', '人工智能', '的', '发展', '，', '机器人', '变得', '越来越', '智能', '。']
seg2分词后的结果：['随着', '人工智能', '的', '发展', '，', '自动', '驾驶', '变得', '越来越', '智能', '。']

```

然后我们把结果合并、去重：

```
list_result = list(set(seg1 + seg2))

```

接来下，我们得到每句话分词后的词向量，并合成一个大的向量：

```
vec1 = [1 if x in seg1 else 0 for x in list_result ]
vec2 = [1 if x in seg2 else 0 for x in list_result ]
vec = [vec1,vec2]

```

得到的词向量：

```
[[1, 0, 1, 1, 1, 1, 1, 1, 1, 1, 1, 0], [1, 1, 1, 1, 1, 1, 1, 1, 0, 1, 1, 1]]

```

最后我们计算距离，这里演示欧氏距离和曼哈顿距离计算：

```
print(pairwise_distances(vec,metric='euclidean'))

```

欧氏距离结果：

```
[[ 0.          1.73205081]
 [ 1.73205081  0.        ]]

```

```
print(pairwise_distances(vec,metric='manhattan'))

```

曼哈顿距离结果：

```
[[ 0.  3.]
 [ 3.  0.]]

```

其他距离计算，请参考 sklearn 包，修改 metric 的值即可。

> [参考链接](http://scikit-learn.org/stable/modules/generated/sklearn.metrics.pairwise_distances.html#sklearn.metrics.pairwise_distances)

#### 2. 余弦相似

几何中夹角余弦可用来衡量两个向量方向的差异，机器学习中借用这一概念来衡量样本向量之间的差异。

对于两个n维样本点 a(x11,x12,⋯,x1n)a(x11,x12,⋯,x1n) 与 b(x21,x22,⋯,x2n)b(x21,x22,⋯,x2n)，可以使用类似于余弦相似的概念来衡量它们间的相似程度，余弦相似的计算公式如下：

![enter image description here](http://images.gitbook.cn/fa587a30-67c9-11e8-812c-750794b244e2)

夹角余弦取值范围为 [-1,1]。余弦值越大表示两个向量的夹角越小，余弦值越小表示两向量的夹角越大。当两个向量的方向重合时取最大值 1，当两个向量的方向完全相反取最小值 -1。

类似的，对于余弦相似计算如下：

```
print(pairwise_distances(vec,metric='cosine'))

```

余弦相似计算结果：

```
[[ 0.          0.14188367]
 [ 0.14188367  0.        ]]

```

#### 3. 相关系数 ( Correlation coefficient )与相关距离(Correlation distance)

相关系数是衡量随机变量X与Y相关程度的一种方法，相关系数的取值范围是 [-1,1]。相关系数的绝对值越大，则表明X与Y相关度越高。当 X 与 Y 线性相关时，相关系数取值为 1（正线性相关）或 -1（负线性相关）。

![enter image description here](http://images.gitbook.cn/9501ad90-67ca-11e8-a826-513048f4c795)

则相关距离表示为：

![enter image description here](http://images.gitbook.cn/a270b430-67ca-11e8-811b-0fa70a363339)

#### 4. 局部敏感哈希（simhash）

simhash 是 locality sensitive hash（局部敏感哈希）的一种，最早由 Moses Charikar 在《similarity estimation techniques from rounding algorithms》一文中提出。

介绍下这个算法主要原理（[原文](http://www.lanceyan.com/tech/arch/simhash_hamming_distance_similarity.html)），分为这几步：

1. **分词**：把需要判断文本分词形成这个文章的特征单词。最后形成去掉噪音词的单词序列并为每个词加上权重，我们假设权重分为 5 个级别（1~5）。比如：“ 美国“51 区”雇员称内部有 9 架飞碟，曾看见灰色外星人 ” ==> 分词后为 “ 美国（4） 51区（5） 雇员（3） 称（1） 内部（2） 有（1） 9架（3） 飞碟（5） 曾（1） 看见（3） 灰色（4） 外星人（5）”，括号里是代表单词在整个句子里重要程度，数字越大越重要。
2. **hash**：通过 hash 算法把每个词变成 hash 值，比如 “美国” 通过 hash 算法计算为 100101，“51 区”通过 hash 算法计算为 101011。这样我们的字符串就变成了一串串数字，还记得文章开头说过的吗，要把文章变为数字计算才能提高相似度计算性能，现在是降维过程进行时。
3. **加权**：通过 2 步骤的 hash 生成结果，需要按照单词的权重形成加权数字串，比如 “美国” 的 hash 值为 “100101”，通过加权计算为“4 -4 -4 4 -4 4”；“51 区” 的 hash 值为“101011”，通过加权计算为 “ 5 -5 5 -5 5 5”。
4. **合并**：把上面各个单词算出来的序列值累加，变成只有一个序列串。比如 “美国” 的 “4 -4 -4 4 -4 4”，“51 区” 的 “ 5 -5 5 -5 5 5”， 把每一位进行累加， “4+5 -4+-5 -4+5 4+-5 -4+5 4+5” ==》 “9 -9 1 -1 1 9”。这里作为示例只算了两个单词的，真实计算需要把所有单词的序列串累加。
5. **降维**：把 4 步算出来的 “9 -9 1 -1 1 9” 变成 0 1 串，形成我们最终的 simhash 签名。 如果每一位大于 0 记为 1，小于 0 记为 0。最后算出结果为：“1 0 1 0 1 1”。

整个过程图为：



![enter image description here](http://images.gitbook.cn/ac76e5f0-67cd-11e8-811b-0fa70a363339)

了解了原理，我们来动手实践一下：

这里有一个 Python 的算法实现，源码：<https://github.com/leonsim/simhash>

首先通过命令：`pip install simhash(or: easy_install simhash)` 进行安装。

下面我们测试 3 组，对比一下，看看结果：

```
print(Simhash('随着人工智能的发展，机器人变得越来越智能。').distance(Simhash('随着人工智能的发展，机器人变得越来越智能。')))
print(Simhash('随着人工智能的发展，机器人变得越来越智能。').distance(Simhash('随着人工智能的发展，自动驾驶变得越来越智能。')))
print(Simhash('随着人工智能的发展，机器人变得越来越智能。').distance(Simhash('随着人工智能的发展，机器人变得可以感知人的情绪。')))

```

结果：

```
0
11
15

```

上面给出了文本相似度常用的度量和计算方法，然而还有很多方法没有提及到，比如，通过关键词判断，通过语义分析来计算相似度等。对于文本来说，分词、特征向量是整个过程的重点，后面也可以考虑通过 TFIDF、Word2Vec 构建词向量，提高准确性。

最后给出一个不被轻易洗稿的的建议：就是在自己原创文章中尽量加入自己个性化的元素和思想，对于图片和音频可以加自己的 logo 等，以真实特性标榜你的作品。