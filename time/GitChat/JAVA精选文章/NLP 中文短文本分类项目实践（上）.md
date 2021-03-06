目前，随着大数据、云计算对关系型数据处理技术趋向稳定成熟，各大互联网公司对关系数据的整合也已经落地成熟，笔者预测未来数据领域的挑战将主要集中在半结构化和非结构化数据的整合，NLP 技术对个人发展越来越重要，尤其在中文文本上挑战更大。



在本场 Chat 以及现在和未来工作中，笔者都将致力于中文文本的挖掘与开发，而且是通过实战来增加对中文 NLP 需求的应用理解。

由于是第一讲，笔者在本次 Chat 并没有提及较深入的 NLP 处理技术，通过 WordCloud 制作词云、用 LDA 主题模型获取文本关键词、以及用朴素贝叶斯算法和 SVM 分别对文本分类，目的是让大家对中文文本处理有一个直观了解，为后续实战提供基础保障。

下面是一些约定：

1. 本 Chat 示例代码都是基于 Python3 写的，带有必要的注释；
2. 中文自然语言处理（Chinese natural language processing），后面笔者全部简称 CNLP；
3. 笔者所用开发环境是 Windows 10 操作系统和 Jupyter notebook 开发工具。相信示例代码在 Linux、Mac OS 等系统上运行也没问题。

### 一、WordCloud 制作词云

最近中美贸易战炒的沸沸扬扬，笔者用网上摘取了一些文本（自己线下可以继续添加语料），下面来制作一个中美贸易战相关的词云。

#### 1. jieba 分词安装

> jieba 俗称中文分词利器，作用是来对文本语料进行分词。

- 全自动安装：`easy_install jieba` 或者 `pip install jieba / pip3 install jieba`
- 半自动安装：先下载 <https://pypi.python.org/pypi/jieba/> ，解压后运行 `python setup.py install`
- 手动安装：将 jieba 目录放置于当前目录或者 site-packages 目录。
- 安装完通过 `import jieba` 验证安装成功与否。

#### 2. WordCloud 安装

> WordCloud 作用是用来绘制词云。

- 下载：<https://www.lfd.uci.edu/~gohlke/pythonlibs/#wordcloud>
- 安装（window 环境安装）找的下载文件的路径：`pip install wordcloud-1.3.2-cp36-cp36m-win_amd64.whl`
- 安装完通过 `from wordcloud import WordCloud` 验证安装成功与否。

#### 3. 开始编码实现

整个过程分为几个步骤：

- 文件加载
- 分词
- 统计词频
- 去停用词
- 构建词云

下面详细看代码：

```
#引入所需要的包
import jieba
import pandas as pd 
import numpy as np
from scipy.misc import imread 
from wordcloud import WordCloud,ImageColorGenerator
import matplotlib.pyplot as plt
#定义文件路径
dir =  "D://ProgramData//PythonWorkSpace//study//"
#定义语料文件路径
file = "".join([dir,"z_m.csv"])
#定义停用词文件路径
stop_words = "".join([dir,"stopwords.txt"])
#定义wordcloud中字体文件的路径
simhei = "".join([dir,"simhei.ttf"])
#读取语料
df = pd.read_csv(file, encoding='utf-8')
df.head()
#如果存在nan，删除
df.dropna(inplace=True)
#将content一列转为list
content=df.content.values.tolist()
#用jieba进行分词操作
segment=[]
for line in content:
    try:
        segs=jieba.cut_for_search(line)
        segs = [v for v in segs if not str(v).isdigit()]#去数字
        segs = list(filter(lambda x:x.strip(), segs))   #去左右空格
        #segs = list(filter(lambda x:len(x)>1, segs)) #长度为1的字符
        for seg in segs:
            if len(seg)>1 and seg!='\r\n':
                segment.append(seg)
    except:
        print(line)
        continue
#分词后加入一个新的DataFrame
words_df=pd.DataFrame({'segment':segment})
#加载停用词
stopwords=pd.read_csv(stop_words,index_col=False,quoting=3,sep="\t",names=['stopword'], encoding='utf-8')               
#安装关键字groupby分组统计词频，并按照计数降序排序
words_stat=words_df.groupby(by=['segment'])['segment'].agg({"计数":np.size})
words_stat=words_stat.reset_index().sort_values(by=["计数"],ascending=False)
#分组之后去掉停用词
words_stat=words_stat[~words_stat.segment.isin(stopwords.stopword)]
#下面是重点，绘制wordcloud词云，这一提供2种方式
#第一种是默认的样式
wordcloud=WordCloud(font_path=simhei,background_color="white",max_font_size=80)
word_frequence = {x[0]:x[1] for x in words_stat.head(1000).values}
wordcloud=wordcloud.fit_words(word_frequence)
plt.imshow(wordcloud)
wordcloud.to_file(r'wordcloud_1.jpg')  #保存结果
#第二种是自定义图片
text = " ".join(words_stat['segment'].head(100).astype(str))
abel_mask = imread(r"china.jpg")  #这里设置了一张中国地图
wordcloud2 = WordCloud(background_color='white',  # 设置背景颜色 
                     mask = abel_mask,  # 设置背景图片
                     max_words = 3000,  # 设置最大现实的字数
                     font_path = simhei,  # 设置字体格式
                     width=2048,
                     height=1024,
                     scale=4.0,
                     max_font_size= 300,  # 字体最大值
                     random_state=42).generate(text)

# 根据图片生成词云颜色
image_colors = ImageColorGenerator(abel_mask)
wordcloud2.recolor(color_func=image_colors)
# 以下代码显示图片
plt.imshow(wordcloud2)
plt.axis("off")
plt.show()
wordcloud2.to_file(r'wordcloud_2.jpg') #保存结果

```

这里只给出默认生产的图，自定义的图保存在 wordcloud_2.jpg，回头自己看。

![enter image description here](http://images.gitbook.cn/f9ef44f0-54ed-11e8-a37f-ad1a7c798536)

### 二、LDA 提取关键字

接下来完成 LDA 提取关键字的实战，如果有人对 LDA 的理论感兴趣，推荐阅读马晨《LDA 漫游指南》这本书。

#### 1. Gensim 安装

> Gensim 除了具备基本的语料处理功能外，Gensim 还提供了 LSI、LDA、HDP、DTM、DIM 等主题模型、TF-IDF 计算以及当前流行的深度神经网络语言模型 word2vec、paragraph2vec 等算法，可谓是方便之至。

- Gensim 安装：`pip install gensim`
- 安装完通过 `import gensim` 验证安装成功与否。

#### 2. LDA 提取关键字编码实现

语料是一个关于汽车的短文本，传统的关键字提取有基于 TF-IDF 算法的关键词抽取；基于 TextRank 算法的关键词抽取等方式。下面通过 Gensim 库完成基于 LDA 的关键字提取。

整个过程步骤：

- 文件加载
- 分词
- 去停用词
- 构建词袋模型
- LDA 模型训练
- 结果可视化

下面详细看代码：

```
#引入库文件
import jieba.analyse as analyse
import jieba
import pandas as pd
from gensim import corpora, models, similarities
import gensim
import numpy as np
import matplotlib.pyplot as plt
%matplotlib inline
#设置文件路径
dir = "D://ProgramData//PythonWorkSpace//study//"
file_desc = "".join([dir,'car.csv'])
stop_words = "".join([dir,'stopwords.txt'])
#定义停用词
stopwords=pd.read_csv(stop_words,index_col=False,quoting=3,sep="\t",names=['stopword'], encoding='utf-8')
stopwords=stopwords['stopword'].values
#加载语料
df = pd.read_csv(file_desc, encoding='gbk')
#删除nan行
df.dropna(inplace=True)
lines=df.content.values.tolist()
#开始分词
sentences=[]
for line in lines:
    try:
        segs=jieba.lcut(line)
        segs = [v for v in segs if not str(v).isdigit()]#去数字
        segs = list(filter(lambda x:x.strip(), segs))   #去左右空格
        segs = list(filter(lambda x:x not in stopwords, segs)) #去掉停用词
        sentences.append(segs)
    except Exception:
        print(line)
        continue
#构建词袋模型
dictionary = corpora.Dictionary(sentences)
corpus = [dictionary.doc2bow(sentence) for sentence in sentences]
#lda模型，num_topics是主题的个数，这里定义了5个
lda = gensim.models.ldamodel.LdaModel(corpus=corpus, id2word=dictionary, num_topics=10)
#我们查一下第1号分类，其中最常出现的5个词是：
print(lda.print_topic(1, topn=5))
#我们打印所有5个主题，每个主题显示8个词
for topic in lda.print_topics(num_topics=10, num_words=8):
    print(topic[1])

```

![enter image description here](http://images.gitbook.cn/037227d0-54ef-11e8-a37f-ad1a7c798536)

```
#显示中文matplotlib
plt.rcParams['font.sans-serif'] = [u'SimHei']
plt.rcParams['axes.unicode_minus'] = False
# 在可视化部分，我们首先画出了九个主题的7个词的概率分布图
num_show_term = 8 # 每个主题下显示几个词
num_topics  = 10  
for i, k in enumerate(range(num_topics)):
    ax = plt.subplot(2, 5, i+1)
    item_dis_all = lda.get_topic_terms(topicid=k)
    item_dis = np.array(item_dis_all[:num_show_term])
    ax.plot(range(num_show_term), item_dis[:, 1], 'b*')
    item_word_id = item_dis[:, 0].astype(np.int)
    word = [dictionary.id2token[i] for i in item_word_id]
    ax.set_ylabel(u"概率")
    for j in range(num_show_term):
        ax.text(j, item_dis[j, 1], word[j], bbox=dict(facecolor='green',alpha=0.1))
plt.suptitle(u'9个主题及其7个主要词的概率', fontsize=18)
plt.show()

```

![enter image description here](http://images.gitbook.cn/182b54d0-54ef-11e8-a37f-ad1a7c798536)

### 三、朴素贝叶斯和 SVM 文本分类

最后一部分，我们通过带标签的数据：

数据是笔者曾经做过的一份司法数据，其中需求是对每一条输入数据，判断事情的主体是谁，比如报警人被老公打，报警人被老婆打，报警人被儿子打，报警人被女儿打等来进行文本有监督的分类操作。

本次主要选这 4 类标签，基本操作过程步骤：

- 文件加载
- 分词
- 去停用词
- 抽取词向量特征
- 分别进行朴素贝叶斯和 SVM 分类算法建模
- 评估打分

下面详细看代码：

```
#引入包
import random
import jieba
import pandas as pd
#指定文件目录
dir = "D://ProgramData//PythonWorkSpace//chat//chat1//NB_SVM//"
#指定语料
stop_words = "".join([dir,'stopwords.txt'])
laogong = "".join([dir,'beilaogongda.csv'])  #被老公打
laopo = "".join([dir,'beilaopoda.csv'])  #被老婆打
erzi = "".join([dir,'beierzida.csv'])   #被儿子打
nver = "".join([dir,'beinverda.csv'])    #被女儿打
#加载停用词
stopwords=pd.read_csv(stop_words,index_col=False,quoting=3,sep="\t",names=['stopword'], encoding='utf-8')
stopwords=stopwords['stopword'].values
#加载语料
laogong_df = pd.read_csv(laogong, encoding='utf-8', sep=',')
laopo_df = pd.read_csv(laopo, encoding='utf-8', sep=',')
erzi_df = pd.read_csv(erzi, encoding='utf-8', sep=',')
nver_df = pd.read_csv(nver, encoding='utf-8', sep=',')
#删除语料的nan行
laogong_df.dropna(inplace=True)
laopo_df.dropna(inplace=True)
erzi_df.dropna(inplace=True)
nver_df.dropna(inplace=True)
#转换
laogong = laogong_df.segment.values.tolist()
laopo = laopo_df.segment.values.tolist()
erzi = erzi_df.segment.values.tolist()
nver = nver_df.segment.values.tolist()
#定义分词和打标签函数preprocess_text
#参数content_lines即为上面转换的list
#参数sentences是定义的空list，用来储存打标签之后的数据
#参数category 是类型标签
def preprocess_text(content_lines, sentences, category):
    for line in content_lines:
        try:
            segs=jieba.lcut(line)
            segs = [v for v in segs if not str(v).isdigit()]#去数字
            segs = list(filter(lambda x:x.strip(), segs))   #去左右空格
            segs = list(filter(lambda x:len(x)>1, segs)) #长度为1的字符
            segs = list(filter(lambda x:x not in stopwords, segs)) #去掉停用词
            sentences.append((" ".join(segs), category))# 打标签
        except Exception:
            print(line)
            continue 
#调用函数、生成训练数据
sentences = []
preprocess_text(laogong, sentences, 'laogong')
preprocess_text(laopo, sentences, 'laopo')
preprocess_text(erzi, sentences, 'erzi')
preprocess_text(nver, sentences, 'nver')

#打散数据，生成更可靠的训练集
random.shuffle(sentences)

#控制台输出前10条数据，观察一下
for sentence in sentences[:10]:
    print(sentence[0], sentence[1])
#用sk-learn对数据切分，分成训练集和测试集
from sklearn.model_selection import train_test_split
x, y = zip(*sentences)
x_train, x_test, y_train, y_test = train_test_split(x, y, random_state=1234)

#抽取特征，我们对文本抽取词袋模型特征
from sklearn.feature_extraction.text import CountVectorizer
vec = CountVectorizer(
    analyzer='word', # tokenise by character ngrams
    max_features=4000,  # keep the most common 1000 ngrams
)
vec.fit(x_train)
#用朴素贝叶斯算法进行模型训练
from sklearn.naive_bayes import MultinomialNB
classifier = MultinomialNB()
classifier.fit(vec.transform(x_train), y_train)
#对结果进行评分
print(classifier.score(vec.transform(x_test), y_test))

```

这时输出结果为：0.99284009546539376。

我们看到，这个时候的结果评分已经很高了，当然我们的训练数据集中，噪声都已经提前处理完了，使得数据集在很少数据量下，模型得分就可以很高。

下面我们继续进行优化：

```
#可以把特征做得更棒一点，试试加入抽取2-gram和3-gram的统计特征，比如可以把词库的量放大一点。
from sklearn.feature_extraction.text import CountVectorizer
vec = CountVectorizer(
    analyzer='word', # tokenise by character ngrams
    ngram_range=(1,4),  # use ngrams of size 1 and 2
    max_features=20000,  # keep the most common 1000 ngrams
)
vec.fit(x_train)
#用朴素贝叶斯算法进行模型训练
from sklearn.naive_bayes import MultinomialNB
classifier = MultinomialNB()
classifier.fit(vec.transform(x_train), y_train)
#对结果进行评分
print(classifier.score(vec.transform(x_test), y_test))

```

这时输出结果为：0.97852028639618138。

我们看到，这个时候的结果稍微有点下降，这与我们语料本身有关系，但是如果随着数据量更多，语料文本长点，我认为或许第二种方式应该比第一种方式效果更好。

**可见使用 scikit-learn 包，算法模型开发非常简单，下面看看 SVM 的应用：**

```
from sklearn.svm import SVC
svm = SVC(kernel='linear')
svm.fit(vec.transform(x_train), y_train)
print(svm.score(vec.transform(x_test), y_test))

```

这时输出结果为：0.997613365155，比朴素贝叶斯效果更好。

本次入门级 chat 就将分享到这里，希望给您的工作学习带来帮助。后期请继续关注，笔者将给大家分享更多关于中文自然语言处理实战的知识。也包括更多的知识。在模型上，我们后期还要实战调参、模型优化；在算法上，从传统机器学习算法到深度学习算法不断扩大算法知识面；在应用上，除了文本分类后期还有大量中文文本关键字、词抽取、情感分析、聊天机器人、语义分析、知识图谱等内容。