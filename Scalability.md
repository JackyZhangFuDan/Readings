# 什么是软件可伸缩性 （Scalability）  
可伸缩性描述系统在这方面的能力：它可以自动扩展自身或其基础设施，来应对大业务量，也能及时回收从而节省开销。（注：这里似乎包含了弹性，弹性和可伸缩性的区别是前者自动完成应对系统的突发变化，属于即时短期行为；而后者按计划根据业务变化而变化，属于长期变化）例如你有一个网站，某天突然被搜索引擎放在最突出的位置，它的访问量会一下子增大数倍，那么它能应对得了吗？一个可伸缩的Web应用会保证自动扩展去处理大业务量而不是直接崩溃。崩溃（或者只是变慢）让你的用户不开心，从而为你“赢得”一个坏名声。一般来说系统可在如下几个方面去伸缩：
- 存储I/O
- 内存
- 网络I/O
- CPU
而当我们在云计算背景下谈论伸缩性时，我们常听到如下几种种伸缩方案：横向，纵向和混合。首先我们深入看看它们什么意思。  

## 纵向伸缩 （Vertical Scaling）  
纵向伸缩相对来说简单一些。当对一个系统进行纵向伸缩时，你会增强应用已经使用的资源，这可以是更多的内存，更快的存储介质，或者更强大的CPU。纵向伸缩较易实施主要是因为在主流云平台上增减硬件是非常简单的操作，因为云平台上服务器已经虚拟化了。这种方式下我们需要做的软件上的调整几乎可以忽略不计。

## 横向伸缩（Horizontal Scaling）  
横向伸缩就稍微复杂一些了。当对一个系统进行横向伸缩时，你会添加更多的Server（意味着新系统实例）等新的资源，去让更多的机器去分担负载。但这么一来你的系统就变得复杂了。现在你具有了多个Server，那么像升级，安全和监控上的任务就加倍了，并且你要保持应用间数据等的同步。  

## 混合伸缩（Diagonal Scaling）  
就是既可以纵向也可以横向。但这实施起来更加的昂贵

## 哪个方案更好？  
一般认为横向伸缩被视为长久解决方案，而纵向伸缩是短期的。因为理论上说你可以不断添加Server到系统中，但却不可能不停升级硬件，硬件性能有其理论最高值。  

# 可伸缩的好处  
可伸缩架构的好处是能在反应实践很短的情况下处理突发大流量和大负载。一个可伸缩系统可以帮你的应用或线上业务在请求高峰时顺利运作，从而不损失钱和名誉。使你的系统微服务化可以让监控，功能发布，排错和伸缩更简单。

# 关于可伸缩性的误解  
可伸缩性不是万能药。构建一个可伸缩的平台及其上的可伸缩应用需要大量的工作，包括规划和反复测试等等。如果你已经有了一个App，那么调整代码从而分割它也是件很乏味的工作。  

# 伸缩性和数据库
每一个应用程序都不一样，但关键时要识别出当负载大增时最先称为瓶颈的那些关键服务。而比较常见的就是数据库服务了。数据库用来存储应用的数据，既可以是传统关系式数据库如MySQL或NoSQL类数据库如MongoDB。在高负载的系统中，数据库常常是最先down掉的那个组件。  

## 数据拆分 （Sharding）  
通过拆分数据库来达到可伸缩性是指，将应用数据分开存储到多个数据库服务器。你会将业务数据分开来存储到不同的Shard中，而不是都放在一个数据库。这对性能有好处：
- 每一个请求都会由个别数据库去响应，这不会造成整个数据库性能的下降  
- 多个数据库分担存储任务，索引更小使得访问更快
- 每个库中内容更少，搜索速度也会加快

## 数据分区（Partitioning）  
数据库分区类似数据库拆分，但不完全一样。分区是把数据分为不同的部分。典型的分区方法包括：  
- 按区间划分 （字母顺序或数字排序）  
- 按行划分（水平分区）  
- 按列划分（垂直分区）  

## 程序代码的数据库优化  
你也可以在代码上对数据库进行优化，包括：  
- 使用索引  
- 分表  
- 对查询进行缓存  
- 逆正规化（De-normalization，就是用些冗余）  
- 预先执行大的查询 （并缓存结果以备用）  

# 谈谈系统效能（Performance）  
达到可伸缩应的一个驱动力是增加系统效能。虽然这只是系统效能的一个方面，可伸缩性和许多其它概念紧密关联在一起，例如弹性（Elasticity）和容错性（Fault tolerance）  

## 响应时间  
系统效能可以用多个不同指标进行衡量，响应时间是主要的一个。有意思的是伸缩性可能会对响应时间产生负面影响。你把一个所有组件（数据库，业务代码，缓存）都在一个Server的应用改成了不同组件分处不同Server的架构，那么很自然一个请求的处理会受内部网络的延迟等因素影响，让我们先看看这两种体系结构把。

### 单体架构（Monolith）  
一个单体架构的系统把所有组件放在一处。从应用程序角度看，它涵盖所有服务例如数据层，缓存层，文件层和业务逻辑层；从硬件和服务器角度看，它包含所有处理，例如数据库，Web服务器和文件系统。  

### 微服务架构  
一个微服务架构的应用把其生态内的处理划分为多个服务。你的应用的核心部分可能是一个图片处理服务，包含保存，删除，缓存和修改图片。这个服务可以同其它服务区分开来，让其具有独立的基础设施。谈微服务时你将总能听到“概念划分”。虽然为核心服务单独划分基础设施有助于伸缩性，但这也同时增加了复杂性。现在你需要管理多个Server并为之修改代码。

# 总结  
那些使用云的公司不会让使用方式一成不变。它们用云协助业务的扩张。为了能更加高效，可伸缩性是架构师需要理解的核心概念之一。