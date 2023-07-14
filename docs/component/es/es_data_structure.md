# elasticsearch中的数据结构

# 前言
上篇文章中初步测试了一下elasticsearch（以下简称es）的性能。为了更好的理解es的性能，这篇文章总结一下es涉及到的一些数据结构，帮助我们更好的理解es的查询。

# 倒排索引
提到es，不能不提的就是倒排索引。学习es，我们应从搜索引擎的角度去想es，去思考倒排索引与关系型数据库的区别。

关系型数据库，以二维表的形式描述了实体。在二维表中，每行是一个实体，每列是一组实体的一项属性。关系型数据库中的索引，为的是针对实体的某些属性，通过查找算法，可以定位到这个实体记录。关系型数据库表的形式如下：

![](https://images.xiaozhuanlan.com/photo/2019/c035a8b980b92d4552f6edf56711a501.png)

如上图，编号是唯一的，我们可以对编号这列做索引，生成b树，再通过二分查找，快速的定位到我们想要的记录，而不用循环遍历每行，判断是不是我们要找的记录。

倒排索引是为了解决哪种场景呢？我们都用过搜索引擎，我们会根据关键字的组合，去检索我们想要的信息。在上面的例子中，比如我知道有个女术士，19岁，爱好是吃饭，我想搜索到这个女术士叫什么名字，我们来看看倒排索引是怎么做的。

![](https://images.xiaozhuanlan.com/photo/2019/2728b225d54bb02b84e9ae85db4d60ed.png)
倒排索引中记录的是属性与id的对应关系，我们知道19出现在1,3这两篇文档中，吃饭出现在1号文档中，两者取交集，便可定位到19岁爱好吃饭的记录是1号，再通过文档id，可以定位到整条记录。

这里有同学就会说了，关系型数据库更简单呀，只要查询（年龄=‘19’ and 爱好 =‘吃饭’）的记录就可以了。正如一开始说的，我们应该从搜索引擎的角度去想es的工作，搜索引擎中存储的大部分都是非结构化数据，很难像上面的例子把各个属性都分隔好。因此，es相比关系型数据库，可以对非结构化数据进行更好的检索。


# FST（Finite StateTransducers）
了解了倒排索引的基本结构后，我们来看看，倒排索引是用哪种数据结构表示的。主观上看，这种k,v格式，可以用hash 表来存储。但实际情况下，词和文档数量都过于庞大，无法全部存储在内存中。es中使用了FST这种结构来处理这个问题。
![](https://images.xiaozhuanlan.com/photo/2019/94ad5ef5ff32be8c65cbd01d55cd028c.png)

在说明FST的作用前，我们先说明一下，es中这个有词形成的字典是分为两部分的（前缀字典树 （term index）和后缀字典树（term dictionary）），分别对应lucence 后缀为tip和tim的文件。前缀树中存储词的前缀及对应前缀的词块的后缀字典树的地址，后缀字典树中存储着对应的后缀词块，及每个词对应的id列表地址。
而FST的作用是将输入的词映射成offset，这个offset指向后缀树的对应的词块。再根据词块去查找对应的id列表。

FST 因为复用了词前缀，因此在内存中的存储得到了压缩，使得对应的词典可以常驻内存。

具体FST的构建流程暂不展开，我们只要知道其优点是压缩了大小，使前缀字典可以常驻内存，加速了查询即可。

# Frame Of Reference
说完了倒排索引的词典部分，接下来就要说文档id列表（posting list）了。

文档id列表是词出现过的文档的id集合，文档数目多了，简单的把id用list串起来也是吃不消的。es中做了一步优化，增量编码压缩。

首先，文档id列表中的id是有序的。然后，我们只存储id的增量。再然后，我们根据增量需要占用的bit数分组，用字节来存储。

![](https://images.xiaozhuanlan.com/photo/2019/9ba9a431bcec50d0a16307e34e216626.png)
如上图，原来6个id要24个字节（一个int占4byte），现在只要7byte。


# Roaring bitmap

当我们的查询有多项，最后要把每个查询命中的文档id列表合并，做交，差，并等罗辑。我们一般会想到用bitmap这种数据结构。但如果文档id列表是稀疏的，就会造成极大的空间浪费。这里便使用roaring bitmap进行优化。

![](https://images.xiaozhuanlan.com/photo/2019/040ebb89133c0e368049e49545e6dc3e.png)

我们把id除65536，分别得到商和余数，将商相同的，放到一个块里，这样id的范围都是小于65536，不会无限增长。

# skip list

跳表也是一种空间换时间的数据结构，通过构建多层子链，加速数据的查找。除了使用bitmap合并结果，es也是用了跳表这种结构。es中filter类型的查询是会被缓存的。如果新的查询在缓存中，就会使用bitmap合并，否则构建跳表合并。
![](https://images.xiaozhuanlan.com/photo/2019/1afc147b1ccb0cb1eb2a32c12f7759eb.jpg)

从第二层开始遍历，如果数据大于6，则可以跨越原始数据中的3,4,5，否则的话，调到下一层索引遍历。

# kdtree
对于数值类型的范围查询，因为会涉及浮点数，如果构建倒排索引的话，可能会产生数目巨大的链合并操作。因此采用kdtree来优化

![](https://images.xiaozhuanlan.com/photo/2019/7a9f37c6a00561072619f2d2655ff19d.jpg)

一维的kdtree 类似二叉查找树，叶子节点中，存储了一个范围内的词和词对应的id。这里的id不要求像倒排索引中文档id列表是有序的。所以，数值类型的范围查询再和其他结果合并时，无法通过链表的形式在各个结果链上迭代。只能通过构建bitset来处理。


# 参考资料

[http://blog.mikemccandless.com/2013/06/build-your-own-finite-state-transducer.html]((https://zhuanlan.zhihu.com/p/35814539)
[https://www.cnblogs.com/dreamroute/p/8484457.html](https://www.cnblogs.com/dreamroute/p/8484457.html)
[https://yq.aliyun.com/articles/701175](https://yq.aliyun.com/articles/701175)
[https://zhuanlan.zhihu.com/p/35814539](https://zhuanlan.zhihu.com/p/35814539)


