# MOON: A Modern pOet hiding on the cOmmand liNe  |  命令行里的现代诗人

![Moon](http://qiniu.shihanmax.top/20210518093459_BV24Yr_%E6%88%AA%E5%B1%8F2021-05-18%2009.34.19.jpeg)

我叫MOON，是一位练习时长2天半的作诗练习生。



## 1. 数据

- 古体诗：34个时期35839位古代诗人共853367首<a href="https://github.com/Werneror/Poetry">古体诗</a>
- 现代诗：1340位现代诗人的19367首<a href="https://github.com/sheepzh/poetry">现代诗</a>

使用上述两类数据分别训练了两个模型，分别用于古体诗和现代诗的生成。



## 2. 模型

训练时，使用堆叠（3层）GRU（单向）进行语言建模，使用linear层将GRU输出投影到词表空间。

解码时，对比使用了Greedy和Sampling-based decoding两种方式。



## 3. 效果

[在线演示](http://shihanmax.top:5000)

### 3.1 古体诗

> hint： 我
>
> 我无奈心事
>
> 我亦何意来人
>
> 何况人情不远人
>
> 一片寒云飞不尽春色如梦
>
> 如梦不到家乡



### 3.2 现代诗

> hint：我
>
> 我的心境里是你
>
> 像我
>
> 我们在路灯下
>
> 一直向下走过
>
> 你们的小说
>
> 在这样深度
>
> 你看见了
>
> 我的心里
>
> 一次的梦是一个人的一生
>
> 那里是我们一个
>
> 在那时
>
> 当一切的都已在那儿
>
> 你的心是无边无情
>
> 在这世间有一次失败
>
> 我是我的一生
>
> 你看我们这样谈笑



## 4. 其他

- 现代诗生成结果，从段落感上看，没有大的问题，但文字是经不起揣摩的，归根到底，这种生成，是在条件语言模型的基础上，依照下一个token的出现概率进行采样的结果，语无伦次也算正常
- 古体诗的生成质量更差一些，推测原因为：五言、七言、还有其他形式的作品（如词等）在一起混合训练，导致在生成时，对格式的控制能力较差，较难生成规矩的五言、七言诗


## 5. 扩展

当然，这个简单的模型也可以用于任何形式文本的生成，如小说、长篇诗歌、话剧、演讲等；修改数据也很简单：在`utils.py`下，参考`load_ancient_poems()`和`load_modern_poems()`实现自己的原始数据加载函数，约定输出格式为：

```json
// List[Dict[..]]
[
    {
        "title": title,  // 题目，optional
        "author": author,  // 作者，optional
        "date": "",  // 创作日期，optional
        "era": era_,  // 时代/年代，optional
        "body": body,  // 正文
    },
    {
        ...
    },
]
```

值得注意的是，对于古体诗，可以使用句号或者逗号标识其换行；但对于现代诗，一般不使用标点符号进行换行，针对上述两种典型的情况，分别的处理方式如下：

### 古体诗

直接拼接，忽略换行：

> 一二三四五，六七八九十。
>
> 十九八七六，五四三二一。
>
> -> 
>
> 一二三四五，六七八九十。十九八七六，五四三二一。



### 现代诗

使用`；`标识换行，替换原文本中的`\n`：

> 假装
>
> 我
>
> 是
>
> 一首
>
> 现代诗
>
> ->
>
> 假装；我；是；一首；现代诗

可以预处理自有语料，将文本分词后保存到文本文件中，并使用`vocab.py`中的接口训练自己的词向量，如使用随机初始化的词向量，则需要根据自己的数据生成str2idx和idx2str两个映射表；这里提供从古体诗中预训练的字向量文件（文本）：链接: https://pan.baidu.com/s/1EKIF10iVKQ7Gd_SGEZEMVw  密码: 7bl9，下载后放置于目录`./resource/w2v/`下

为了训练方便，所有参数、模型、trainer的加载均位于`main.py`内，运行`main.py`即可开始训练；

`serve`使用flask对inference模块进行了简单封装，实现通过http进行推理，有关inference接口`Sampler`的使用，可以参考`serve/app.py`。



