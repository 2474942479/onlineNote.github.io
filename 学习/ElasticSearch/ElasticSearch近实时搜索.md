# Elasticsearch近实时搜索的实现

## 1.近实时搜索

### 1.1 实时与近实时

实时搜索（Real-time Search）很好理解，对于一个数据库系统，执行插入以后立刻就能搜索到刚刚插入到数据。而近实时（Near Real-time），所谓“近”也就是说比实时要慢一点点。

### 1.2 近实时的挑战

对于一个单机系统来说，这也并不容易实现，因为还要保证数据的持久化，还要利用缓存等技术加快数据的访问（注：这里不讨论内存计算系统）。对于ElasticSearch这样一个分布式系统，保证持久化的同时，还要初始化好用于全文检索的内部数据结构，做到近实时的难度可想而知。而这就是ElasticSearch大获成功的地方，也正是本文所要学习的主题：ElasticSearch是如何解决这些实现近实时搜索的难题的。

## 2.ElasticSearch的实现

### 2.1 不可变的数据结构

有经验的程序员一定知道，在做并发编程时，控制可变数据的并发访问是个难题。古往今来，各种粗细粒度的锁，信号量，Actor模型等概念层出不穷。而另一流派函数式编程更为彻底，尤其是纯函数式比如Haskell，用不可变数据来彻底解决这个问题。

在ElasticSearch这样主要服务全文检索的系统中，Inverted Index是核心数据结构。这里简单说一句，Inverted Index本质上一组document中term的各种统计信息，比如最重要的词频，以及其他许多统计信息，比如文档长度，词序等等。要做到近实时搜索，就要保证新数据能快速构建，已有数据能被高速访问。解决问题的关键就在于Inverted Index的不可变性，这也是ElasticSearch底层依赖的高性能Lucene的根本奥秘。

### 2.2 从不可变到可变

所以当用户向ElasticSearch中的数据库插入一组document后，底层Lucene构建出一个不可变的Inverted Index。可我们知道，一个数据库不可能是静态的，当用户再次插入新数据时，Lucene该怎样处理呢？答案就是增量保存和逻辑标记。

所谓增量保存就是为新数据构建一个新的不可变的Inverted Index，当执行搜索时，要合并每个Inverted Index中的统计信息得到最终结果。保存新数据的问题解决了，而逻辑标记就是解决更新和删除的。Lucene为每个Inverted Index都额外维护一个del数据结构，当执行删除时，只需在del中标记，这样最终结果就会排出掉删除掉document。同理，更新时也是给老数据做标记，新document会保存在新的Inverted Index中，最终结果会使用最新版本数据的统计信息。在Lucene中，每个Inverted Index叫做Segment，而管理这些Segment的叫做Index。

> ElasticSearch中一个数据库被称为Index，每个Index可以在创建时指定要划分为几份，每一份叫做Shard。Shard会被ElasticSearch分配到不同结点，运行中还会根据压力做Rebalance。这个Shard其实就是Lucene中的Index。由于不同层级上名字的重复，初学时很容易混淆。

这种思想其实并非独创，在其他一些高级数据结构中也能找到它的影子。如果没记错的话，一个经典的例子就是LSM树：https://en.m.wikipedia.org/wiki/Log-structured_merge-tree。

### 2.3 分布式数据存储

对于分布式的数据存储，ElasticSearch采取了经典的做法，对数据进行分片和路由，这里每个分片Shard就是一个Lucene数据库Index。对于有副本replica的Shard，ElasticSearch操作完primary后，再去同步到replica。

### 2.4 挑战磁盘I/O

现在我们已经可以高效地维护全文检索的数据结构，也遵循经典做法解决了分布式数据存储。可就像前面提到的，还有个挑战就是磁盘读写的巨大开销。Lucene的做法是，每个Segment在文件系统Cache中构建起来就可以被访问，同步到磁盘的fsync之后才会执行。Lucene的Index内部的Commit Point会记住哪些Segment还未同步。ElasticSearch默认每隔1秒会用Buffer中的document新建一个Segment，这个操作叫做refresh。正因为这1秒钟的间隔，ElasticSearch支持的是近实时而非实时。

一个很自然的问题就是每秒钟都会新建一个Segment，那Lucene Index中的Segment个数岂不是很容易就爆炸了。每个Segment都是一个物理文件，操作系统中打开文件的句柄个数是有限的，而且即便不考虑上限，过多Segment也会拖慢搜索，因为前面讲过一次搜索的最终结果是要合并所有Segment中的统计信息的。

ElasticSearch的做法是维护一个后台线程去做Merge，Merge的过程中不仅将多个小Segment合并成大的，同时还会排除掉删除或修改的文件的老版本，最终修改Commit Point排除掉老的Segment，这样那些“垃圾”document就彻底被删除了。得益于Segment的不可变性，后台进程Merge时并不会影响数据插入和搜索的性能。

### 2.5 保证数据不丢失

一个可以预料到的问题就是，如果当前结点上的ElasticSearch进程意外中止，那Buffer中等待处理的document和未同步到磁盘的Segment中的数据都会丢失。为了避免这一点，ElasticSearch引入了传统数据库中所谓的Write-Ahead Log（WAL）日志，ElasticSearch为其起名为translog。每次插入Buffer时，都会同时写入translog。下面的图示清晰地展示ElasticSearch是如何与Lucene配合的。

当创建新Segment时，Buffer清空，但translog会一直保留到Segment同步到磁盘才会清空。**所以当ElasticSearch重启时，先根据Commit Point将所有之前已经commit到磁盘的Segment恢复到Cache，然后再重放（replay）translog中的所有操作。默认每30分钟或者translog很大时，ElasticSearch做一次full commit，即flush操作。

继续刨根问底，translog保证了Buffer和Segment的安全，谁来保证它的安全呢？默认情况下，translog每5秒钟会同步到磁盘，也就是说我们至多会丢失5秒到数据。因为translog只是原始的请求document，所以这里的写磁盘开销是远小于Segment的一次commit的。

## 3.题外话：如何深入学习ElasticSearch

以本文为例，谈一谈如何学习ElasticSearch。在有了一些分布式系统和开发经验后，像本文2.3和2.5节是完全可以跳过的。前者是分布式系统的通用做法，而后者则早已存在于传统数据库中。要掌握ElasticSearch，基本用法和系统命令是一方面，而设计中的精华往往在前文2.1和2.2中。光理解了设计还不行，就像前面说过的，思想可能流传已久，但做出来东西的质量则可能千差万别。“天下大事，必做于细”，实现中的精髓只能在源代码中体会。

其实这种方法在另一篇文章里也提到过，就是学一门编程语言时也是要抓住它的精髓，而不是每门语言都花很多时间去学基本语法，而没有精力去掌握精华，最终迷失了。在此再次强调一下，自己也引以为戒。
