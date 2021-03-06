# 12 Factor App [官网链接](https://12factor.net/)
这是在做微服务时经常被提及的一套原则，提出者具有很多Cloud上微服务的开发经验 - 主要在Heroku平台上的应用开发，所以非常有参考价值。我在读完这12条原则后，感觉还是非常认同的，特别是对于刚起步做微服务的团队来说非常有指导意义。  


[代码库 - Codebase](https://12factor.net/codebase)
---
保持App和代码库是1：1关系。这里“代码库”可以理解为Git里面的一个Repository。  
一份代码，多处部署。  并不是说多个部署跑的版本都是一样的，完全可以例如D上跑最新的代码；Q上跑在测试的次新的代码；P上跑比较老但是很稳定的代码

[关于依赖 - 显示声明出来](https://12factor.net/zh_cn/dependencies)
---
简单来说，我们的程序不要偷偷去依赖一些东西，哪怕它一定存在于绝大多数操作系统层面，要把这种依赖声明出来。我觉得这种说法也是非常合理的，不同操作系统之间甚至同样系统的不同版本之间的差异是我们没法提前预知的，一旦它们给我们的应用造成影响那我们面对的可能是一堆操作系统级别或是至少是很奇怪的log，解决起来比较劳神。如果我们能尽量详细周全地明示我们地依赖，那么发生这种问题时我们地应用会以更容易理解地形式报警：例如在编译或部署时告诉我们，我缺少需要地依赖ABC。

[关于配置 - 用环境变量存](https://12factor.net/zh_cn/config)
---
把我们的程序中使用的配置信息通过环境变量来存，而不是具体的文件。优点是：  
1. 灵活，在不同环境上需要不同的配置？简单啊，只要改环境变量就好了，代码一点儿不动   
2. 保密，很多配置都是不能公开的，你放在代码库里面那很多人就看到了  
想想很多Spring的application.properites里面的内容是可以移到环境变量的，这并不是说.properties文件应该没有，技术性的信息还是要配在里面例如DB Driver用什么，这些东西不私密，不会随环境变化而变化

[服务 - 保持它们可替换](https://12factor.net/zh_cn/backing-services)
---
我们力求把“服务”变得非常容易替换，例如数据库服务，邮箱服务等等，只要有必要我们就可以去替换它们。不仅仅是我们通过‘拿来主义’复用的第三方服务，在我们自己的服务时候也要保持这种原则。  
我感觉作者还对替换的过程提出要求，不用改代码，只要系统管理员操作一下就可以完成，那将是最理想的。  
其实这是我们做微服务的一条重要原则，独立性，可替换性，非常重要

[构建 - 发布 - 运行，严格区分](https://12factor.net/build-release-run)
---
我们要在项目中严格划分出这三个阶段，定义好它们的内容和动作。作者对三者做如下定义：  
1. 构建：就是程序打包的过程
2. 发布：就是把打包好的程序结合它们的配置，组织好存放在某些地方，等待上线的指令，一旦指令到来，立刻就可以启动运行的过程。每一个发布都要有唯一的版本号加以区别，从而线上程序能够在版本之间切换。
3. 运行：启动一个版本，从失败中回复一个版本的运行等等。作者认为这个过程应该尽量自动化，不然线上程序半夜死掉没有人能及时发现并补救，但我想肯定不能完全自动化，例如线上版本的选择可定是人来做。

[要用无状态进程](https://12factor.net/processes)
---
我们的程序肯定是在一个或者多个进程中运行的，我们在做程序的时候要假设所有请求都是无状态的，一个请求不应该依赖于进程里前面请求形成的缓存数据，状态数据等，除非是前面请求明确的持久化了的数据。  
针对这一条要求我只能从复杂度的角度理解，它会让我们的程序比较的简单，服务器负担上可以减轻因为毕竟不用关心session数据的保持了，健壮性会好一点儿？但这个要求对开发理念冲击还是挺大的，特别是只做过之前tomcat上jsp类网站开发的程序员

[端口绑定 - 只需要做完这个就可以享受服务](https://12factor.net/zh_cn/port-binding)
---
作者提出如下要求：  
1. 我们的应用应该需要环境提供服务器，自己就应该内嵌服务器；  
2. 我们的应用可以被管理员绑定到其所在环境的某个端口上，然后就可以开始提供服务了。作者认为几乎所有的服务端程序都可以通过端口绑定的方式对外提供服务。
我觉得第一点要求还是过于绝对了，在sprint boot中是有内嵌服务器的，但其版本我们不可控，我很担心在一些情况下这种内嵌服务器的方式不可以接手。但总的来说我理解作者想达到的目的：从外部来看，一个 12 factor app是一个非常干净，省心的应用，你要做的就是启动它，把它绑到一个端口上，that is！剩下的app自己都包了，高度内聚，连server都自己提供了可见和环境耦合性是多么低！

[并发 - 确保在进程级别并发](https://12factor.net/zh_cn/concurrency)
---
只懂大意，一些细节没特别理解。这一点讲并发，但根本目的在可扩展性。作者要求12 facotr app的结构应该参考Unix的守护进程模式：守护进程会把不同类型的操作分发给不同的进程去处理，于是整个系统被“水平”的分为多种不同类型的进程，并且进程间共享的东西是很少的 - 它不像线程，那么每种进程的数量都可以增加从而达到扩容的目的。由于以进程为单位，所以扩容时在另外的主机上创建新资源的可行性是有的。  
作者提到“Twelve-factor app processes should never daemonize or write PID files. Instead, rely on the operating system’s process manager (such as systemd, a distributed process manager on a cloud platform, or a tool like Foreman in development) to manage output streams, respond to crashed processes, and handle user-initiated restarts and shutdowns.”是想说我们不要自己去创建和管理进程，而应该依靠操作系统工具吗？

[易处理 disposable - 快速启动+优雅退出](https://12factor.net/zh_cn/disposability)
---
我们的进程应该做到：  
首先能够快速启动，最好敲下命令后马上起来，这样对于恢复服务有利。其实我感觉作者想突出的是系统本身的简单性，独立性和无破坏性，关联众多，启动复杂的服务给人一种严肃沉重的感觉，如果能打散变轻快，变得起不来就再起一边也不会造成各种不一致的话将是很cool的。  
其次能够很好应对退出，例如遇到突发底层失败，那么我们的进程能够有序操作而终止。对于做苦力的进程，那么他能把手头作业合理“交还”到某处以便后续继续处理；对于网路进程它能先关闭继续接收服务请求，然后处理完手头现有工作，最后退出。

[尽量保持开发环境和线上环境等价](https://12factor.net/zh_cn/dev-prod-parity)
---
这个观点和我们的很多实践是有冲突的。它强调开发和生产有可能在三方面存在很大不同：首先是部署频率，开发上可能改完马上就在自己的开发环境来测；而生产是只有在几天几周甚至几个月后才会有一次更新。其次是人员，开发人员管开发，运维人员管生产。再次是环境，开发一般都是小巧的，如数据库用用sqlite，而线上一般都是稳定可靠的自然也就大一点。  
12-Factor的标准是拒绝这三个方面的不一致。
1. 从部署频率上来讲，要奔着几分钟就可以从开发到生产的目标来做，那么我的理解是CI的那套流程就很必要了；
2. 从人员上来说，最好不要分什么开发和运维，都是你们这拨人负责，其实这个就是这些年比较说的多的DevOps模式；
3. 从环境上来说，要拒绝不一致，保持开发和生产一致，作者认为现在PC都很强了，打包安装工具也很厉害，复杂后端服务（如大数据库，MQ等）的安装不再费力，那么就要保持一致，但我想这个也不可能绝对达到，只能是尽力了，试想在我本地pc转个SAP HANA是个几乎不可能完成的任务。  
总的来说，作者的出发点是保证开发到生产的成功率，这个还是挺重要的，因为现在云服务都是共享型的，所有客户都会直接受失败的影响，开发团队在不影响客户感受的要求下能有多少时间去修复问题呢？那么自然要把前序工作做充分。

[日志](https://12factor.net/zh_cn/logs)
---
首先，要把日志看成“事件流”，要把我们自己程序的事件流和数据库等后台服务的日志合并起来一起看，形成一个全局的view。    
其次，说我们程序自己不要试图把日志自己管起来，你要输出地话直接向系统地stdout打印好了，如果需要输出日志到文件，那么由操作系统去控制和决定。  
如果有日志分析的需求，那么操作系统应该收集这些log文件交给分析软件。作者还认为从log中可以拿到很多有用信息，例如观察在一定时间范围内的请求趋势，根据出错趋势触发报警信息等。

[管理为目的的进程](https://12factor.net/zh_cn/admin-processes)
---
在我们的应用中，经常会提供一些供管理员通过命令行的方式来执行从而进行管理的程序（其实我感觉我们的很多rest api都可以被用于管理目的，或者说我们可以做一组rest api，供授权者去使用），那么这就形成了所谓”管理进程“，作者要求这种管理进程呢要和被管理的其它工作进程的代码同步，放在一个库里保证这种一致性；同时其运行环境也要和工作进程一致，不要搞特殊化。  
其实我理解作者目的主要是避免不同步的问题，为了管理而单独造一套程序，单独发布，那么自然会有接口匹配等问题，是比较麻烦的：一旦分开，那么二者必然通过商定好的接口交流，那工作代码在演化的时候就需要考虑保护这些接口稳定，从而要花额外的经历。如果放在一起，可能在代码”编译“的时候就能马上知道管理程序某个地方要改，直接改掉就好了。  
但我感觉作者说的有道理不过也说不上更好，分开和合并各有千秋吧。
  