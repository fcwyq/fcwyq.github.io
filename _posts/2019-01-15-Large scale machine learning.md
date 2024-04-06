---
layout: post
title:      "TensorFlow：异构分布式系统的大规模机器学习"   
date:       2019-01-15 
author:     "fcwyq"
header-img: "img/post-1.jpg"
header-mask: 0.3
catalog: true
tags:
    - tensorFlow
        
---

（初步白皮书，2015年11月9日）

Mart´ın Abadi, Ashish Agarwal, Paul Barham, Eugene Brevdo, Zhifeng Chen, Craig Citro, Greg S. Corrado, Andy Davis, Jeffrey Dean, Matthieu Devin, Sanjay Ghemawat, Ian Goodfellow, Andrew Harp, Geoffrey Irving, Michael Isard, Yangqing Jia, Rafal Jozefowicz, Lukasz Kaiser, Manjunath Kudlur, Josh Levenberg, Dan Man´e, Rajat Monga, Sherry Moore, Derek Murray, Chris Olah, Mike Schuster, Jonathon Shlens, Benoit Steiner, Ilya Sutskever, Kunal Talwar, Paul Tucker, Vincent Vanhoucke, Vijay Vasudevan, Fernanda Vi´egas, Oriol Vinyals, Pete Warden, Martin Wattenberg, Martin Wicke, Yuan Yu, and Xiaoqiang Zheng Google Research*

arXiv：1603.04467v2 [cs.DC] 2016年3月16日

摘要

TensorFlow [1]是用于表达机器学习算法的接口，以及用于执行这些算法的实现。使用TensorFlow表示的计算可以在各种各样的异构系统上进行很少或没有变化，从手机和平板电脑等移动设备到数百台机器的大规模分布式系统和数千种计算设备，如GPU卡。该系统非常灵活，可用于表达各种算法，包括深度神经网络模型的训练和推理算法，并已用于进行研究和将机器学习系统部署到十几个生产中。计算机科学和其他领域，包括语音识别，计算机视觉，机器人，信息检索，自然语言处理，地理信息提取和计算药物发现。本文描述了Ten-sorFlow接口以及我们在Google上构建的该接口的实现。TensorFlow API和参考实现在2015年11月的Apache 2.0许可下作为开源软件包发布，可从www.tensorflow.org获取。

介绍

Google Brain项目于2011年启动，旨在探索使用非常大规模的深度神经网络，用于研究和在Google产品中使用。作为该项目早期工作的一部分，我们构建了DistBelief，这是我们的第一代可扩展分布式培训和推理系统[14]，该系统为我们提供了很好的服务。我们和Google的其他人使用DistBelief进行了各种各样的研究，包括无监督学习[31]，语言表示[ 35,52]，图像分类和对象检测模型[ 16,48]，视频分类[27] ]，语音识别[56,21,20 ]，序列预测[47]，移动选择Go [34]，行人检测[2]，强化学习[38]和其他领域[17,5 ]。此外，通常与Google Brain团队密切合作，Google和其他Alphabet公司的50多个团队在各种产品中使用DistBelief部署了深度神经网络，包括谷歌搜索[11]，我们的广告产品，我们的语音识别系统[50，6，46]，谷歌照片[43]，谷歌地图和街景[19]，谷歌翻译 [18]，YouTube和其他许多人。

 * 通讯作者：Jeffrey Dean和Rajat Monga：{jeff，rajatmonga}@google.com

基于我们对DistBelief的经验以及对培训和使用神经网络所需系统性能和要求的更全面理解，我们构建了TensorFlow，这是我们用于实施和部署大规模机器学习的第二代系统楷模。TensorFlow采用类似数据流的模型描述的计算，并将它们映射到各种不同的硬件平台，从Android和iOS等移动设备平台上的运行推理到使用单机的中等规模的培训和推理系统包含一个或多个GPU卡，用于在数百台具有数千个GPU的专用机器上运行的大型培训系统。拥有一个可以跨越如此广泛的平台的单一系统显着简化了机器学习系统的实际使用，因为我们发现拥有用于大规模培训和小规模部署的独立系统会导致显着的维护负担和泄漏的抽象。TensorFlow计算表示为有状态数据流图（在第二章节中有更详细的描述），我们专注于使系统具有足够的灵活性，以便快速试验用于研究目的的新模型，以及足够高的性能和强大的生产培训和机器学习模型的部署。为了将神经网络训练扩展到更大的部署，TensorFlow允许客户端通过核心模型数据流图的复制和并行执行轻松表达各种并行性，许多不同的计算设备都协作更新一组共享参数或其他状态。。在COM的putation的描述适度变化允许实现与较低的努力尝试了多种不同的方法来并行[14，29， 42]。一些TensorFlow使用允许在参数更新的一致性方面具有一定的灵活性，并且我们可以在一些较大的部署中轻松表达和利用这些宽松的同步要求。与DistBelief相比，TensorFlow的编程模型更加灵活，其性能显着提高，并且支持在更广泛的异构硬件平台上进行培训和使用更广泛的模型。

我们DistBelief的数十名内部客户已经切换到TensorFlow。这些客户依靠TensorFlow进行研究和生产，其任务多种多样，如手机上的计算机视觉模型的运行推理，以及数千亿个参数的大规模深度神经网络培训，涉及数千亿的示例记录使用许多数百台机器的[11，47，48，18，53，41。虽然这些应用主要集中在机器学习和深度神经网络上，但我们期望TensorFlow的抽象将在各种其他领域中发挥作用，包括其他类型的机器学习算法，以及可能的其他类型的数值计算。我们已于2015年11月在Apache 2.0许可下开源了TensorFlow API和参考实现，可从www.tensorflow.org获取。

本文的其余部分更详细地描述了TensorFlow。第2 节描述了TensorFlow接口的编程模型和基本概念，第3 节描述了我们的单机和分布式实现。第4 节描述了对基本编程模型的几种扩展，第5 节描述了对基本实现的几种优化。第6 节描述了我们使用Ten-orFlow的一些经验，第7 节描述了我们在使用TensorFlow时发现有用的几个编程惯例，第9 节描述了我们围绕核心TensorFlow系统构建的几个辅助工具。第10 节和第11节分别讨论了未来和相关工作以及部分12提供了结论性的想法。


编程模型和基本概念

TensorFlow计算由有向图描述，该有向图由一组节点组成。该图表示数据流计算，其中包含允许某些类型的节点维护和更新持久状态的扩展，以及用于以类似于Naiad的方式在图中分支和循环控制结构的扩展[36] 类似。客户端通常使用一种支持的前端语言（C ++或Python）构建计算图。使用Python前端构造然后执行TensorFlow图的示例片段如图1所示，结果计算图如图2所示。

 在一个TensorFlow图中，每个节点具有零个或多个放IN-和零级或多个的输出，和表示的instan- tiation 操作。沿图中的正常边缘（从输出到输入）流动的值是张量，任意维度数组，其中在图形构造时指定或推断基础元素类型。特殊边，称为控制依赖，也可以存在于图中：没有数据沿着这样的边缘流动，但它们表明控制依赖的源节点必须在控制依赖的目标节点开始执行之前完成执行。由于我们的模型包含可变状态，因此客户端可以直接使用控制依赖关系来强制执行关系之前。

我们的实现有时也会控制依赖关系以强制执行其他独立操作之间的排序，例如控制峰值内存使用的方式。


 操作和内核


 一个操作都有一个名称和表示一个抽象的COM putation（例如，“矩阵乘法”或“添加”）。操作可以具有属性，并且必须在图形构造时提供或推断所有属性，以便实例化节点以执行操作。对属性的一种常见用法是使操作在不同的张量元素类型上具有多态性（例如，添加两个类型为float的类型，而不是添加两个类型为int32的张量）。一个内核是可以在特定类型的设备（例如，CPU或GPU）上运行的操作的特定实现。TensorFlow二进制文件通过注册机制定义可用的操作集和内核集，并且可以通过链接其他操作和/或内核定义/注册来扩展此集。表 1显示了核心TensorFlow库中内置的一些操作。

 会议

 客户机程序通过创建会话与TensorFlow系统交互。为了创建一个计算图，会话接口支持一个扩展方法，用附加的节点和边（创建会话时的初始图是空的）来扩充会话管理的当前图。会话接口支持的另一个主要操作正在运行，它使用一组需要计算的输出名称，以及一组可选的张量，这些张量将被输入到图形中，而不是节点的某些输出。使用要运行的参数，TensorFlow实现可以计算所有必须执行的节点的可传递闭包，以计算请求的输出，然后可以安排以尊重其依赖性的顺序执行适当的节点（如3.1中更详细的描述）。我们使用TensorFlow的大多数方法都只设置了一次与图形的会话，然后通过run调用执行完整的图形或几个不同的子图形数千次或数百万次。

 import tensorflow as tf

b = tf.Variable(tf.zeros([100])) # 100-d vector, init to zeroes

W = tf.Variable(tf.random_uniform([784,100],-1,1)) # 784x100 matrix w/rnd vals

x = tf.placeholder(name="x") # Placeholder for input

relu = tf.nn.relu(tf.matmul(W, x) + b) # Relu(Wx+b)

C = [...] # Cost computed as a function # of Relu

s = tf.Session()

for step in xrange(0, 10):

input = ...construct 100-D input array ... # Create 100-d vector for input

result = s.run(C, feed_dict={x: input}) # Fetch cost, feeding x=input

print step, result


 图1：示例TensorFlow代码片段

![2](/img/post1/2.png)

 图2：图1的对应计算图

| Category | Examples |
|----------------|-----------------------------------------------------------|
| 元素数学运算 | Add，Sub，Mul，Div，Exp，Log，Greater，Less，Equal，.. |
| 数组操作 | Concat，Slice，Split，Constant，Rank，Shape，Shuffle，... |
| 矩阵运算 | MatMul，MatrixInverse，MatrixDeterminant，... |
| 状态操作 | 变量，赋值，赋值，... |
| 神经网络构建块 | SoftMax，Sigmoid，ReLU，Convolution2D，MaxPool，... |
| 检查点操作 | 保存，恢复 |
| 队列和同步操作 | Enqueue，Dequeue，MutexAcquire，MutexRelease，...... |
| 控制流程操作 | 合并，切换，回车，离开，NextIteration |

 表1：示例TensorFlow操作类型

 变量

 在大多数计算中，图形被执行多次。大多数张量都不会超过图表的单次执行。但是，变量是一种特殊的操作，它返回一个持久的可变张量的句柄，该张量可以在图形的执行中幸存下来。这些持久性可变张量的句柄可以传递给一些特殊操作，例如Assign和AssignAdd（相当于+ =），它可以改变引用的张量。对于TensorFlow的机器学习应用程序，模型的参数通常存储在变量中的张量中，并作为模型的训练图运行的一部分进行更新。


 履行

TensorFlow系统中的主要组件是客户端，它使用Session接口与主服务器和一个或多个工作进程通信，每个工作进程负责仲裁对一个或多个计算设备（如CPU核心或GPU）的访问（并且）用于按照主设备的指示在这些设备上执行图形节点。我们既有本地也有分布式TensorFlow接口的实现。当客户端，主服务器和工作服务器在单个操作系统进程的上下文中运行时，可以使用本地实现（可能有多个设备，例如，机器有许多GPU卡）安装）。分布式实现与本地实现共享大部分代码，但是通过支持环境来扩展它，客户端，主服务器和工作者都可以在不同的机器上处于不同的进程中。在我们的分布式环境中，这些不同的任务是由集群调度系统管理的作业中的容器[51]。这两种不同的模式如图所示3.本节剩下的大部分内容讨论了两种实现共同的问题，而 3.3 节讨论了分布式实现特有的一些问题。

设备

设备是TensorFlow的计算核心。每个工作人员负责一个或多个设备，每个设备都有设备类型和名称。设备名称由识别设备类型的部件，工作人员中的设备索引以及工作人员的作业和任务的标识组成（或者设备是本地设备的情况下的本地主机）过程）。示例设备名称为“/ job：localhost / device：cpu：0”或“/ job：worker / task：17 / device：gpu：3”。我们为CPU和GPU提供了Device接口的实现，并且可以通过注册机制提供其他设备类型的新设备实现。每个设备对象负责管理设备内存的分配和释放，以及安排任何设备内存的执行。 TensorFlow实现中更高级别请求的内核。


张量

我们实现中的张量是一个类型化的多维数组。我们支持各种张量元素类型，包括大小从8位到64位的有符号和无符号整数，IEEE浮点数和双精度类型，复数类型和字符串类型（一个任意字节数组）。适当大小的支持存储由分配器管理，该分配器特定于张量所在的设备。张量后备存储缓冲区是引用计数，并在没有引用时被释放。


 单器件执行

 让我们首先考虑最简单的执行方案：使用单个设备的单个工作进程。图的节点以遵循节点之间的依赖性的顺序执行。特别是，我们跟踪每个节点的计数，该节点尚未执行的节点的依赖性数量。一旦此计数降至零，该节点就有资格执行，并被添加到就绪队列中。就绪队列以某种未指定的顺序处理，将节点的核心执行委托给设备对象。当节点执行完毕后，依赖于已完成节点的所有节点的计数都会减少。


 多设备执行

一旦系统有多个设备，有两个主要的复杂因素：决定将每个节点的计算放在图表中的设备，然后管理这些放置决策所暗示的设备边界所需的数据通信。本小节讨论了这两个问题。


 节点布局

 给定计算图，TensorFlow实现的主要职责之一是将计算映射到可用设备集。这里给出了该算法的简化版本。有关此算法支持的扩展，请参见第 4.3 节。

![3](/img/post1/3.png) ![4](/img/post1/4.png)

   图3：单机和分布式系统结构

布局算法的一个输入是一个成本模型，它包含每个图节点的输入和输出张量的大小（以字节为单位）估计值，每个图节点的输入和输出张量，以及每个节点在与其输入张量一起呈现时所需的计算时间的估计。该成本模型要么基于与不同操作类型相关联的启发式方法进行静态估计，要么基于早期执行图形的实际放置决策集来度量。

放置算法首先运行图形的模拟执行。下面描述了模拟，最后使用贪婪的启发式方法为图中的每个节点选择一个设备。此模拟生成的节点到设备放置也用作实际执行的放置。

放置算法从计算图的源开始，并在系统中的每个设备上模拟其活动。对于在该遍历中到达的每个节点，都要考虑可行的设备集（如果设备不提供实现特定操作的内核，则设备可能不可行）。对于具有多个可行设备的节点，放置算法使用贪婪的启发式方法检查节点放置在每个可能设备上对节点完成时间的影响。这种启发式方法考虑了成本模型中该类设备上操作的估计或测量执行时间，还包括将引入的任何通信的成本，以便将输入从其他设备传输到此节点到考虑的设备。选择节点操作最快完成的设备作为该操作的设备，然后放置过程继续进行，以便为图中的其他节点（包括现在准备好进行自己模拟执行的下游节点）做出放置决策。第4.3节描述了一些扩展，这些扩展允许用户提供提示和部分约束来指导放置算法。放置算法是系统中正在开发的一个领域。   

   跨设备通信

   一旦计算出节点放置，图形就被划分为一组子图，每个设备一个子图。从任何交叉装置边缘X向ý从移除和更换由边缘X到一个新的发送在节点X的子图，并从边缘的相应接收节点Ý在ý 的子图。有关此图转换的示例，请参见图 4。

![5](/img/post1/5.png)

图4：插入发送/接收节点之前和之后

在运行时，发送和接收节点的实现协调以跨设备传输数据。这允许我们隔离Send和Receive实现中的所有通信，这简化了运行时的其余部分。

当我们插入发送和接收节点时，我们可以将特定设备上特定张量的所有用户统一为使用单个接收节点，而不是特定设备上每个下游用户的一个接收节点。这确保了所需张量的数据仅在源设备目标设备对之间传输一次，而目标设备上的张量存储器仅分配一次，而不是多次（例如，参见节点b和c）在图4中）

通过以这种方式处理通信，我们还允许将不同设备上的图形的各个节点的调度分散到工作器中：发送和接收节点传递必要的不同工作者和设备之间的同步，并且主服务器只需要为每个具有该图的任何节点的工作者执行每个图执行的单个运行请求，而不是参与每个节点或每个跨设备通信的调度。这使得系统具有更高的可扩展性，并且与主服务器强制执行调度相比，允许更精细的节点执行。
   
  分布式执行

  图的分布式执行非常类似于多设备执行。设备放置后，子图每个设备创建。发送/接收节点对跨工人流程进行通信使用远程通信,诸如TCP或RDMA之类的机制，用于跨机器边界移动数据。


  容错

  可以在各种地方检测分布式执行中的故障。我们依赖的主要是（a）发送和接收节点对之间的通信错误，以及（b）从主进程到每个工作进程的定期健康检查。

  检测到故障时，将中止整个图形执行并从头开始重新启动。然而，回想一下，变量节点是指在图的执行中持久存在的张量。我们支持在重启时对此状态进行一致的检查和恢复。另外，每个Variable节点都连接到Save节点。这些Save节点定期执行，例如每N次迭代执行一次，或每N秒执行一次。执行时，变量的内容将写入持久性存储，例如，分布式文件系统。类似地，每个变量都连接到Restore节点，该节点仅在重新启动后的第一次迭代中启用。有关如何仅在某些图形执行时启用某些节点的详细信息，请参见第 4.2 节。


 扩展

 在本节中，我们将介绍第2节中介绍的基本编程模型的几个更高级的功能。


  梯度计算

  许多优化算法，包括常规机器学习训练算法，如随机梯度下降 [45]，计算成本函数相对于一组输入的梯度。因为这是这样的普遍需要，TensorFlow内置支持自动梯度计算。如果Ten-sorFlow图中的张量C可能通过一个复杂的运算子图依赖于某些张量X k，那么就会有一个内置函数返回张量dC/dXk。与其他张量一样，通过使用以下过程扩展TensorFlow图来计算梯度张量。

![6](/img/post1/6.png)

图5：为图2中的图形计算的梯度  

  当TensorFlow需要计算的张量的梯度Ç相对于一些张量我在其上Ç取决于，它首先找到在计算图从路径我到Ç。然后它从C回溯到我，对于后向路径上的每个操作，它将一个节点添加到TensorFlow图，使用链规则组成沿向后路径的部分梯度。新添加的节点计算前向路径中相应操作的“梯度函数”。可以通过任何操作来注册梯度函数。该功能不仅可以输入已经沿后向路径计算的部分梯度，还可以选择输入和输出正向操作。图5显示了根据图2的示例计算的成本的梯度。灰色箭头显示了未用于所示特定操作的梯度函数的潜在输入。图 1计算这些梯度所需的附加是：

  [db，dW，dx] = tf.gradients（C，[b，W，x]）

  一般来说，一个操作可能有多个输出，而C可能只依赖其中的一些输出。例如，如果操作o有两个输出y1和y2，而c只依赖于y2，那么第一个输入o的梯度函数设置为0，因为dc=dy1=0。

  自动梯度计算使优化变得复杂，尤其是内存使用。当执行“前向”计算子图（即，由用户显式构造的子图）时，合理的启发式通过观察构造图的顺序来决定下一个要执行的节点时断开关系。

  这通常意味着临时输出在构建后很快消失，因此它们的存储器可以快速重复使用。当启发式无效时，用户可以更改图构造的顺序，或者添加控件依赖性，如第 5 节所述。当梯度节点自动添加到图形中时，用户控制较少，启发式可能会中断。特别是，因为梯度颠倒了正向计算顺序，在图的早期执行中使用的张量在梯度计算的末尾经常需要再次使用。这样的张量可以保留很多稀缺的GPU内存并且不必要地限制大小计算。我们正在积极致力于内存管理的改进，以更好地处理此类案例。选项包括使用更复杂的启发式算法来确定图形执行的顺序，推荐张量而不是将它们保留在内存中，以及将长寿命张量从GPU内存转换为更丰富的主机CPU内存。

  部分执行

  客户端通常只想执行整个执行图的子图。为了支持这一点，一旦客户端在Session中设置了计算图形，我们的Run方法允许它们执行整个图形的任意子图形，并沿图形中的任何边缘注入任意数据，并检索流向的数据。图中的任何边缘。

  图中的每个节点都有一个名称，节点的每个输出都由源节点名称和节点的输出端口标识，编号为0（例如，“bar：0”表示节点的第一个输出“bar”节点，而“bar：1”表示第二个输出）。

  Run调用的两个参数帮助定义将要执行的计算图的确切子图。首先，Run调用接受输入， name 的可选映射：端口名称为“fed”张量值。其次，Run调用接受输出名称，输出名称[： port ] 规范列表，指示应执行哪些节点，如果端口部分存在于名称中，则应返回该节点的特定输出张量值如果Run调用成功完成，则返回客户端。

  图表根据输入和输出的值进行转换。每个节点：输入中指定的端口将替换为feed节点，该节点将从用于Run调用的Rendezvous对象中的特殊初始化条目中获取提供的输入张量。类似地，带端口的每个输出名称都连接到一个特殊的获取节点，该节点安排保存输出张量并在Run调用完成时将其返回给客户端。最后，一旦通过插入特殊的feed和fetch节点重写了图形，就可以通过从任何输出命名的每个节点开始并使用图形依赖关系在图形中向后工作来确定要执行的节点集合。 必须在重写图中执行的完整节点集，以便计算输出。图 6显示了左侧的原始图形，以及使用inputs == {b}和outputs == {f：0}调用Run时得到的转换图形。由于我们只需要计算节点f的输
出，我们不会执行节点d和e，因为它们对f的输出没有贡献。

![7](/img/post1/7.png)

图6：部分执行的图转换前后

  设备限制

  TensorFlow客户端可以通过为节点提供可以执行的设备的部分约束来控制设备上节点的放置。例如，“仅将此节点放在GPU类型的设备上”，或“此节点可以放在任何设备上  / job：worker / task：17 “ ，或 ”将此节点与名为variable13的节点共存“ 。在这些约束的范围内，放置算法负责选择节点的分配，以便快速执行计算并满足设备本身施加的各种约束，例如限制总量设备上所需的内存，以便执行其图形节点的子集。

  支持此类约束需要更改第 3.2.1 节中描述的放置算法。我们首先为每个节点计算可行的设备集，然后在共置约束图上使用union-find来计算必须放在一起的图组件。对于每个这样的组件，我们计算可行设备集的交叉。每个节点计算的可能设备集很容易适应放置算法的模拟器。

  控制流

  虽然没有任何显式控制流的数据流图非常具有表现力，但我们观察到许多情况，其中支持条件和循环可以导致更简洁和有效的机器学习算法表示。

  就像Arvind [3] 描述的数据流机器方法一样，我们在TensorFlow中引入了一小组原始控制流操作符，并推广了Ten-sorFlow来处理循环数据流图。该交换机和合并运营商允许我们跳过整个子图的基础上的布尔ten- SOR的值执行。在进入，离开，并NextIteration运营商允许我们表达迭代。可以使用这些控制流操作符轻松地将if-conditionals和while-loops等高级编程结构编译成数据流图。

  TensorFlow运行时实现了标签和框架的概念，概念上类似于MIT Tagged-Token机器 [4]。循环的每次迭代由标记唯一标识，并且其执行状态由帧表示。只要输入可用，输入就可以进入迭代; 因此，可以同时执行多次迭代。

  TensorFlow使用分布式协调机制来执行具有控制流的图。通常，循环可以包含分配给许多不同设备的节点。因此，管理循环的状态成为分布式终止检测的问题。TensorFlow的解决方案基于图形重写。在图分区中，我们会自动将控制节点添加到每个分区。这些节点实现了一个小型状态机，它协调每次迭代的开始和终止，并决定循环的终止。对于每次迭代，拥有循环终止谓词的设备会向每个参与设备发送微小的控制消息。

  如上所述，我们经常通过梯度下降来训练机器学习模型，并将梯度计算表示为数据流图的一部分。当模型包括控制流操作时，我们必须在相应的梯度计算中考虑它们。例如，具有if条件的模型的梯度计算将需要知道条件的哪个分支，然后将梯度逻辑应用于此分支。类似地，具有while循环的模型的梯度计算将需要知道进行了多少次迭代，并且还将依赖于在这些迭代期间计算的中间值。基本技术是重写图形，以便记住梯度计算所需的值。我们省略了这种编码的一些内容细节。

  输入操作

  虽然可以通过馈送节点将输入数据提供给计算，但是用于训练大规模机器学习模型的另一种常见机制是在图中具有特殊的输入操作节点，这些节点通常配置有一组文件名，每次执行时，都会产生一个张量，其中包含存储在该组文件中的数据中的一个或多个示例。这允许将数据直接从底层存储系统读取到将对数据执行后续处理的机器的存储器中。在客户端进程与工作进程分离的配置中，如果数据被馈送，则通常需要额外的网络跃点（从存储系统到客户端，然后从客户端到工作者，直接从使用输入节点时，工作人员的存储系统）。


  队列

  队列是我们添加到Ten- sorFlow的有用功能。它们允许图形的不同部分异步执行，可能在不同的candence上执行，并通过Enqueue和Dequeue操作传递数据。入队操作可以阻塞，直到队列中有空间可用，并且出列操作可以阻塞，直到队列中有所需的最小元素数。队列的一个用途是允许从磁盘文件中预取输入数据，而之前的一批数据仍然由机器学习模型的计算部分处理。它们还可以用于其他类型的分组，包括累积许多梯度，以便在更大的批次上计算一些更复杂的梯度组合，除了普通的FIFO队列之外，我们还实现了一个混洗队列，它在一个大的内存缓冲区中随机地混洗它的元素。例如，这种混洗功能对于想要随机化处理示例的顺序的机器学习算法很有用。


  集装箱

  一个集装箱是管理长期存在的可变状态中TensorFlow机制。变量的后备存储位于容器中。默认容器是在进程终止之前一直存在的容器，但我们也允许其他命名容器。一个容器可以通过完全清除其内容来重置。使用容器，即使在与不同会话相关联的完全不相交的计算图中，也可以共享状态。


 优化


 在本节中，我们将介绍TensorFlow实现中的一些优化，这些优化可以提高系统的性能或资源使用率。


  常见的Subexpression消除

  由于计算图的构造通常由客户端代码中的许多不同的抽象层完成，因此计算图很容易得到相同计算的冗余副本。为了解决这个问题，我们实现了一个类似于Click [12] 所描述的算法的公共子表达式传递，该算法在计算图上运行，并将具有相同输入和操作类型的多个操作副本规范化为这些节点中的单个节点，并重定向图形边缘适当地反映这种规范化。


  控制数据通信和内存使用

  仔细调度TensorFlow操作可以带来更好的系统性能，特别是在数据传输和内存使用方面。具体而言，调度可以减少在操作期间需要在内存中保留中间结果的时间窗口，从而减少峰值内存消耗。这种减少对于内存稀缺的GPU设备尤其重要。此外，管理跨设备的数据通信可以减少对网络资源的争用。

  虽然有很多机会可以安排优化，但我们在这里专注于我们发现特别有必要和有效的机会。它涉及用于读取远程值的接收节点的调度。如果不采取预防措施，这些节点可能比必要时更早开始，可能在执行开始时一次性完成。通过执行尽可能快的/尽可能晚的（ASAP / ALAP）计算，在运算研究中常见的那种，我们分析图的关键路径，以便估计何时开始接收节点。然后我们插入控制边缘，目的是将这些节点的起点延迟到需要它们之前。

  异步内核

  除了在Compute方法结束时完成执行的普通同步内核之外，我们的框架还支持非阻塞内核。这种非阻塞内核使用稍微不同的接口，因此Compute方法被传递一个应该在内核执行完成时调用的延续。这是针对具有许多活动线程的环境在内存使用或其他资源方面相对昂贵的环境的优化，并且允许我们在等待I / O或其他事件发生时避免在无限制的时间段内占用执行线程。异步内核的示例包括Receive内核以及Enqueue和Dequeue内核（如果队列空间不可用或者没有数据可供读取，则可能需要阻塞）。


  用于内核实现的优化库

  我们经常使用预先存在的高度优化的数字库来为某些操作实现内核。例如，有许多用于在不同设备上执行矩阵乘法的优化库，包括BLAS [15]和cuBLAS [39]，或用于深度神经网络（如cuda-convnet）的卷积核的GPU库[28] ]和cuDNN [9]。我们的许多内核实现都是围绕此类优化库的相对较薄的包装器。

  我们对系统中的许多内核实现进行了相当广泛的开源Eigen线性代数库 [25]的使用。作为TensorFlow开发的一部分，我们的团队（主要是Benoit Steiner）扩展了开源特征库，支持任意维度张量操作。


  有损压缩

  一些机器学习算法，包括通常用于训练神经网络的算法，可以容忍噪声和降低精度算术。以类似于DistBelief系统的方式[14]，我们经常在设备之间发送数据时使用有限压缩的高精度内部表示（有时在同一台机器内，特别是在机器边界内）。例如，我们经常插入特殊的转换节点，将32位浮点表示转换为16位浮点表示（不是建议的IEEE 16位浮点标准，而只是32位IEEE 794浮点格式） ，但尾数中的精度低16位），然后转换回通信通道另一侧的32位表示（通过填充丢失部分的零）尾数，因为在进行32→16→32位转换时，这比在数学上正确的概率舍入计算成本更低。


 身份和经历

 TensorFlow接口和参考实现已在Apache 2.0许可下开源，该系统可从www.tensorflow.org 下载。该系统包括详细的文档，一些教程，以及一些演示如何将系统用于各种不同机器学习任务的示例。这些例子包括用于对来自MNIST数据集的手写数字进行分类的模型（机器学习算法的“hello world”）[32]，对来自IFAR-10
数据集的图像进行分类[30]，使用重复进行语言建模。租用LSTM [22]网络，培训单词嵌入矢量 [35]等。

 该系统包括用于在Python和C ++中指定Tensor-Flow计算的前端，并且我们期望随着时间的推移添加其他前端以响应内部Google用户和更广泛的开源社区的需求。

 我们之前的DistBelief系统 [14]中有很多机器学习模型，我们已经迁移到TensorFlow。本节的其余部分讨论了我们所学到的一些经验教训，这些经验教训可用

于将机器学习模型从一个系统迁移到另一个系统，因此可能对其他系统有价值。

 

 特别是，我们专注于我们的教训，即将最先进的卷积神经网络移植到名为Inception的图像识别[23]。该图像识别系统将224×224像素图像分类为1000个标签中的一个（例如，“猎豹”，“垃圾车”等）。当表示为Tensor-Flow图时，这样的模型包括1360万个可学习的参数和36,000个操作。在单个图像上运行推断需要20亿次乘法加法运算。

 在TensorFlow中构建所有必要的数学运算之后，将所有36,000个运算组装和调试到正确的图形结构中，证明了它具有挑战性。验证正确性是一项困难的事业，因为系统具有内在的随机性，并且只能在预期中以某种方式表现 - 可能在经过数小时的计算之后。鉴于这些情况，我们发现以下策略对于将Inception模型移植到TensorFlow至关重要：


  构建工具以深入了解给定模型中的确切参数数量。这些工具揭示了复杂网络体系结构规范中的细微缺陷。特别是我们能够识别由于在一个维度上的数学运算中的自动广播而不正确地实例化的操作和变量。


  从小处着手，扩大规模。我们从之前的系统移植的第一个卷积神经网络是在CIFAR-10数据集上使用的小型网络[30]。调试这样的网络阐明了机器学习系统中个别操作（例如，最大池）的细微边缘情况，这些情况在更复杂的模型中几乎是无法修复的。


  在学习关闭时，始终确保机器学习系统之间的目标（损失函数）匹配。将学习率设置为零有助于我们识别在模型中随机初始化变量的方式中的意外行为。

在动态的训练网络中很难识别出这样的错误。


  在调试分布式实现之前，使单个机器实现匹配。该策略帮助我们描绘和调整机器学习系统之间培训性能的差异。特别是，我们发现了由于竞争条件和非原子操作错误地认为是原子的错误。


  防止数字错误。数值库在处理非有限浮点值方面不一致。卷积神经网络特别容易受到数值不稳定的影响，并且在实验和调试阶段往往会发生相当大的分歧。通过检查非有限浮点值来防止这种行为，可以实时检测错误，而不是事后识别发散行为。


  分析网络的各个部分，了解数值误差的大小。在两个机器学习系统上并行运行神经网络的子设备提供了一种精确的方法，以确保两个系统中的数值算法是相同的。鉴于这些算法以浮点精度运行，重要的是预测和理解预期的数值误差的大小，以判断给定的组件是否正确实现（例如，区分 “在1e-2内，很好” ！“ 和 ”在1e-2内：为什么这么不正确？！“ ）。

![8](/img/post1/8.png)

 图7：同步和异步数据并行训练


 在存在固有随机系统的情况下验证复杂的数学运算是非常具有挑战性的。上述策略在获得对系统的信心以及最终在TensorFlow中实现Inception模型方面证明是非常宝贵的。与我们现有的模型的DistBelief实施相比，这些努力的最终结果使训练时间提高了6倍，并且这种速度增益在训练一类新的大型图像识别模型中证明是不可或缺的。


 常见编程成语


 TensorFlow的基本数据流图模型可以以多种方式用于机器学习应用程序。我们关心的一个领域是加速在大型数据集上训练计算密集型神经网络模型。本节描述了我们和其他人为实现这一目标而开发的几种技术，并说明了如何使用TensorFlow来实现这些不同的方法。

 本小节中的方法假设使用随机梯度下降（SGD）训练模型，其具有相对适中大小的100至1000个示例的小批量。

 数据并行训练


 加速SGD的一种简单技术是在小批量元素中平行化小批量梯度的计算。例如，如果我们使用1000个元素的小批量大小，我们可以使用模型的10个副本来计算100个元素的渐变，然后组合渐变并同步对参数应用更新，按顺序就像我们运行批量大小为1000个元素的顺序SGD算法一样。在这种情况下，TensorFlow图只是具有大量模型计算的图形部分的许多副本，并且单个客户端线程驱动该大图的整个训练循环。这在图7的顶部示出。


 这种方法也可以是异步的，其中TensorFlow图具有大量模型计算的图形部分的许多复制品，并且这些复制品中的每一个也异步地将参数更新应用于模型参数。

在此配置中，每个图表副本都有一个客户端线程。这在图7 的底部部分中说明。这种异步方法也在[14]中描述。

![9](/img/post1/9.png)

图8：模型并行训练

![10](/img/post1/10.png)

图9：并发步骤

模型并行训练

 模型并行训练，其中模型计算的不同部分在同一批示例中同时在不同的计算设备上完成，也很容易在TensorFlow中表达。图8显示了用于序列到序列学习的循环深LSTM模型的示例（参见 [47]），并行化了三个不同的设备。

模型计算流水线的并发步骤

  另一种更好地利用深度神经网络训练的常用方法是通过在同一组设备中运行少量并发步骤来在同一设备中管理模型的计算。这在图 9中示出。它有点类似于异步数据并行性，除了并行性发生在同一设备中，而不是在不同设备上复制计算图。这允许“填补空白”，在单个步骤中，一批单个示例的计算可能无法在所有设备上始终充分利用完全并行性。

 性能

本白皮书的未来版本将对单台机器和分布式实现进行全面的性能评估。
 
 工具

 本节描述了我们开发的一些工具，这些工具位于核心TensorFlow图形执行引擎旁边。

  TensorBoard：图形结构和摘要统计的可视化

  为了帮助用户理解计算图的结构以及理解机器学习模型的整体行为，我们构建了Ten-sorBoard，这是TensorFlow的配套可视化工具，包含在开源版本中。
  
  计算图的可视化

  许多深度神经网络的计算图可能非常复杂。例如，用于训练类似于Google的Incep-的模型的计算图模型 [48]是一个深度卷积神经网络，在ImageNet 2014竞赛中具有最佳分类性能，在其TensorFlow计算图中有超过36,000个节点，而一些用于语言建模的深度循环LSTM模型有超过15,000个节点。

  由于这些图的大小和拓扑结构，天真的视觉化技术通常会产生混乱和过度的图表。为了帮助用户查看图形的基础组织，Tensor-Board中的算法将节点折叠为高级块，突出显示具有相同结构的组。该系统还将高度节点（通常用于簿记功能）分离到屏幕的单独区域。这样做可以减少视觉混乱，并将注意力集中在计算图的核心部分。

  整个可视化是交互式的：用户可以平移，缩放和扩展分组节点，以深入了解详细信息。图10中示出了深度卷积图像模型的图形的可视化的示例。

  摘要数据的可视化

  包括标量汇总（例如，用于检查模型的整体属性，例如在一组示例中平均的损失函数的值，或执行计算图所花费的时间），基于组织的总结（例如，权重值的分布） 在神经网络层中，或基于图像的摘要（例如，在卷积神经网络中学习的滤波器权重的可视化）。 通常设置计算图以便包括Summary节点以监视各种有趣的值，并且在执行训练图期间，除了执行的正常节点集之外，还执行一组摘要节点， 并且客户端驱动程序将摘要数据写入与模型训练相关联的日志文件中。然后将TensorBoard程序配置为观察此日志文件以获取新的摘要记录，并可以显示此摘要信息及其随时间的变化（能够选择“时间”的测量值为自开始以来的相对壁时间 执行TensorFlow程序，绝对时间或“步骤”，这是自TensorFlow程序执行开始以来发生的图形执行次数的数字度量。 TensorBoard中汇总值可视化的屏幕截图如图11所示。

![11](/img/post1/11.png)

  图10：卷积神经网络模型的TensorBoard图形可视化


![12](/img/post1/12.png)

  图11：模型汇总统计时间序列数据的TensorBoard图形显示

  绩效追踪

  我们还有一个名为EEG的内部工具（未包含在2015年11月的初始开源版本中），我们用它来收集和可视化有关TensorFlow图执行的确切顺序和性能特征的非常精细的信息。 此工具适用于我们的单机和分布式实现，对于理解TensorFlow程序的计算和通信模式中的瓶颈非常有用。

  系统中的每台机器上都会同时收集跟踪信息，包括Linux内核ftrace，我们自己的轻量级线程跟踪工具和CUDA性能分析工具接口（CUPTI）。通过这些日志，我们可以使用每个线程切换，CUDA内核启动和DMA操作的微秒级详细信息重建分布式训练步骤的执行。

  跟踪在可视化服务器中组合，该服务器旨在快速提取指定时间范围内的事件，并在适当的详细级别汇总用户界面解析。通过可视化中的箭头识别和突出显示由于通信，同步或DMA相关停顿引起的任何显着延迟。最初，UI提供整个跟踪的概述，仅突出显示最重要的性能工件。随着用户逐渐放大，呈现越来越精细的分辨率细节。

  图12显示了在多核CPU平台上训练的模型的示例EEG可视化。屏幕截图的前三分之一显示了根据数据流限制并行调度的TensorFlow操作。跟踪的底部显示了大多数操作如何分解为多个工作项，这些工作项在线程池中并发执行。右侧大小的斜箭头表示在线程池中排队延迟的位置。图图13显示了另一个脑电图可视化，其计算主要发生在GPU上。可以看到主机线程在TensorFlow GPU操作变为可运行时（浅蓝色线程池）排队，并且可以在跨处理器核心迁移的其他颜色中看到后台管理线程。再次，箭头显示线程在GPU上转移到CPU传输的位置，或者操作经历显着的排队延迟。

  最后，图 14显示了一个更详细的视图，它允许我们检查Tensorflow GPU运算符如何分配给多个GPU流。每当数据流图允许并行执行或数据传输时，我们都会尝试使用流和流依赖性原语向GPU设备公开排序约束。


 未来工作

 我们有未来工作的几个不同方向。 我们将继续使用TensorFlow为人工智能开发新的和有趣的机器学习模型，在这样做的过程中，我们可能会发现需要扩展基本TensorFlow系统的方法。 开源社区也可能为TensorFlow实现提出新的有趣方向。

 我们正在考虑的基本编程模型的一个扩展是函数机制，由此用户可以将TensorFlow计算的整个子图指定为可重用组件。在我们设计的实现中，这些功能甚至可以在TensorFlow的不同前端语言中成为可重用的组件，以便用户可以使用Python前端定义功能，但随后使用该功能作为C ++前端内部的基本构建块。我们希望这种跨语言的可重用性将引导一个充满活力的机器学习研究团队，他们不仅发布他们研究的整个例子，而且还发布他们工作中可以在其他环境中重用的小型可重用组件。

 我们还有许多具体方向来改善TensorFlow的性能。一个这样的方向是我们对即时编译器的初步工作，该编译器可以获取TensorFlow执行的子图，可能具有关于张量的典型大小和形状的一些运行时分析信息，并且可以生成一个op-本子图的惯例。该编译器将理解执行许多优化的语义，例如循环融合，局部性的阻塞和平铺，特定形状和大小的特化等。

 我们还想象，未来工作的一个重要领域将是改进放置和节点调度算法，这些算法用于决定不同节点可执行的位置，以及何时应该开始执行。我们目前在这些子系统中实施了许多启发式算法，我们希望让系统学习做出好的放置决策（可能使用深度神经网络，结合强化学习目标函数）。


 相关工作

 还有许多其他系统可以通过TensorFlow以各种方式进行比较。 Theano [7]，Torch [13]，Caffe [26]，Chainer [49]和Computational Network Toolkit [54]是一些主要用于训练神经网络的系统。这些系统中的每一个都将计算映射到单个 机器，与分布式TensorFlow实现不同。像Theano和Chainer一样，TensorFlow支持符号区分，因此更容易定义和使用基于梯度的优化算法。 与Caffe一样，TensorFlow有一个用C ++编写的核心，简化了在各种生产环境中训练模型的部署，包括内存和计算受限的环境，如移动设备。

![13](/img/post1/13.png)

 图12：多线程的CPU的操作（x轴为时间的EEG可视化μS）。

![14](/img/post1/14.png)

 图13：初始训练的EEG可视化，显示CPU和GPU活动。


 

 TensorFlow系统与其前身系统DistBelief [14]和后来具有类似设计的系统（如Project Adam [10]和参数服务器项目[33]）共享一些设计特征。 与DistBelief和Project Adam一样，TensorFlow允许计算在许多机器上分布在许多计算设备上，并允许用户使用相对高级别的描述来指定机器学习模型。然而，与DistBelief和Project Adam不同，TensorFlow中的通用数据流图模型更灵活，更适合表达更多种类的机器学习模型和优化算法。它还允许通过允许将有状态参数节点表达为变量以及仅作为图中的附加节点的变量更新操作来实现显着的简化; 相比之下，DistBelief，Project Adam和Parameter Server系统都有整个单独的参数服务器子系统致力于通信和更新参数值。

 ![15](/img/post1/15.png)

 图14：多流GPU执行的时间线。 

 用于表示图像处理流水线的Halide系统 [40]使用与TensorFlow数据流图相似的中间表示。然而，与Ten-sorFlow不同，Halide系统实际上对其操作的语义具有更高级别的知识，并利用这些知识生成高度优化的代码片段，这些代码组合了多个操作，兼顾并行性和局部性。Halide仅在一台机器上运行生成的计算，而不是在分布式设置中运行。在未来的工作中，我们希望通过类似的跨操作动态编译框架扩展TensorFlow。

 与TensorFlow一样，已经开发了几个其他分布式系统用于跨集群执行数据流图。Dryad [24]和Flume [8]展示了如何将复杂的工作流表示为数据流图。CIEL[37]和Naiad [36]引入了对数据依赖控制流的通用支持：CIEL表示迭代作为动态展开的DAG，而Naiad使用带有周期的静态图来支持低延迟迭代。Spark [55]针对使用“有利的分布式数据集”（RDD）重复访问相同数据的计算进行了优化，RDD是早期计算的软状态缓存输出。蒲公英[44]在包括GPU在内的异质设备集群中执行数据流图。TensorFlow使用混合数据流模型，借用每个系统中的元素。其数据流调度程序是选择下一个节点执行的组件，它使用与Dryad，Flume，CIEL和Spark相同的基本算法。它的分布式架构最接近Naiad，在这一点上，系统使用单个优化的数据流图来表示整个计算，并在每个设备上缓存关于该图的信息，以最小化协调开销。像spark和naiad一样，当集群中有足够的RAM来保存计算的工作集时，TensorFlow工作得最好。TensorFlow中的迭代使用混合方法：同一数据流图的多个副本可以同时执行，同时共享同一组变量。副本可以通过变量异步共享数据，或者使用图中的同步机制（如队列）同步操作。TensorFlow还支持图中的迭代，该图是CIEL和NAIAD的混合体：为了简单起见，每个节点仅在其所有输入就绪时触发（如CIEL）；但是为了提高效率，图表示为静态循环数据流（如NAIAD）。


 结论


我们已经描述了TensorFlow，一种灵活的基于数据流的编程模型，以及该编程模型的单机和分布式实现。该系统源于在各种Google产品和服务中进行研究和部署多个机器学习项目的实际经验。我们开源了TensorFlow的一个版本，并希望一个充满活力的共享社区围绕着TensorFlow的使用而发展。我们很高兴看到Google以外的其他人如何在自己的工作中使用TensorFlow。

致谢

TensorFlow的开发受益于Google的大型和广泛的机器学习社区，特别是Google Brain团队其他成员以及Google内数百名DistBelief和TensorFlow用户的建议和贡献
。毫无疑问，通过倾听他们的反馈，TensorFlow的可用性和功能得到了极大的扩展。

许多人为TensorFlow及其开源版本做出了贡献，包括John Gian-nandrea（用于创建支持性研究环境），Irina Kofman和Phing Turner（项目管理），Bill Gruber和David Westbrook（技术文书），Dave Andersen，Anelia Angelova，Yaroslav Bulatov，Jianmin Chen，Jerjou Cheng，George Dahl，An-drew Dai，Lucy Gao，mig Gerard，Stephan Gouws，Naveen Kumar，Geoffrey Hinton，Mrinal Kalarishnan，Anjuli Kannan，Yutaka Leon-Suematsu，Frank Li，Pepert，刘小兵，Nishant Patil，Pierre Sermanet，Noam Shazeer，Jascha Sohl-dickstein，Philip Tucker，Yonghui Wu，Ke Yang和Cliff Young（一般贡献），Doug Fritz ，Patrick Hurst，Dilip Krish-nan，Daniel Smilkov，James Wexler，Jimbo Wilson，Kanit Ham Wongsuphasawat，Cassandra Xia，和Big Picture团队（图形可视化），Chris Leary，Robert Springer和Stream Executor团队，Kayur Patel，Michael Piatek和coLab团队，以及为TensorFlow设计和代码库做出贡献的许多人。

References

[1] Mart´ın Abadi, Ashish Agarwal, Paul Barham, Eugene Brevdo, Zhifeng Chen, Craig Citro, Greg S. Corrado, Andy Davis, Jeffrey Dean, Matthieu Devin, Sanjay Ghemawat, Ian Goodfellow, Andrew Harp, Geoffrey Irving, Michael Isard, Yangqing Jia, Rafal Jozefowicz,
Lukasz Kaiser, Manjunath Kudlur, Josh Levenberg, Dan Man´e, Rajat Monga, Sherry Moore, Derek Murray, Chris Olah, Mike Schuster, Jonathon Shlens, Benoit Steiner, Ilya Sutskever, Kunal Talwar, Paul Tucker, Vincent Vanhoucke, Vijay Vasudevan, Fernanda Vi´egas, Oriol
Vinyals, Pete Warden, Martin Wattenberg, Martin Wicke, Yuan Yu, and Xiaoqiang Zheng. TensorFlow: Large-scale machine learning on heterogeneous systems, 2015. Software available from tensorflow.org.

[2] Anelia Angelova, Alex Krizhevsky, and Vincent Vanhoucke. Pedestrian detection with a large-field-of-view deep network. In Robotics and Automation (ICRA), 2015 IEEE International Conference on, pages 704–711. IEEE, 2015. CalTech PDF.

[3] Arvind and David E. Culler. Annual review of computer science vol. 1, 1986. chapter Dataflow Architectures, pages 225–253. 1986.
www.dtic.mil/cgi-bin/GetTRDoc?Location=U2&doc=GetTRDoc.pdf&AD=ADA166235.

[4] Arvind and Rishiyur S. Nikhil. Executing a program on the MIT tagged-token dataflow architecture. IEEE Trans. Comput., 39(3):300–318, 1990.dl.acm.org/citation.cfm?id=78583.

[5] Jimmy Ba, Volodymyr Mnih, and Koray Kavukcuoglu.Multiple object recognition with visual attention.arXiv preprint arXiv:1412.7755, 2014.arxiv.org/abs/1412.7755.

[6] Franc¸oise Beaufays. The neural networks behind Google Voice transcription, 2015.googleresearch.blogspot.com/2015/08/the-neuralnetworks-behind-google-voice.html.

[7] James Bergstra, Olivier Breuleux, Fr´ed´eric Bastien, Pascal
Lamblin, Razvan Pascanu, Guillaume Desjardins,Joseph Turian, David Warde-Farley, and Yoshua Bengio.Theano: A CPU and GPU math expression compiler. In Proceedings of the Python for scientific computing conference(SciPy), volume 4, page 3. Austin, TX, 2010.
UMontreal PDF.

[8] Craig Chambers, Ashish Raniwala, Frances Perry,Stephen Adams, Robert R Henry, Robert Bradshaw,and Nathan Weizenbaum. FlumeJava: easy, efficient data-parallel pipelines. In ACM Sigplan Notices,
volume 45, pages 363–375. ACM, 2010. research.google.com/pubs/archive/35650.pdf.

[9] Sharan Chetlur, Cliff Woolley, Philippe Vandermersch,Jonathan Cohen, John Tran, Bryan Catanzaro,and Evan Shelhamer. cuDNN: Efficient primitives for deep learning. arXiv preprint arXiv:1410.0759, 2014.arxiv.org/abs/1410.0759.

[10] Trishul Chilimbi, Yutaka Suzue, Johnson Apacible, and
Karthik Kalyanaraman. Project Adam: Building an efficient and scalable deep learning training system. In 11th USENIX Symposium on Operating Systems Design and Implementation (OSDI 14), pages 571–582, 2014.www.usenix.org/system/files/conference/osdi14/osdi14-
paper-chilimbi.pdf.

[11] Jack Clark. Google turning its lucrative web search over to AI machines, 2015.www.bloomberg.com/news/articles/2015-10-26/googleturning-its-lucrative-web-search-over-to-ai-machines.

[12] Cliff Click. Global code motion/global value numbering.In ACM SIGPLAN Notices, volume 30, pages 246–257. ACM, 1995. courses.cs.washington.edu/courses/cse501/06wi/reading/click-pldi95.pdf.

[13] Ronan Collobert, Samy Bengio, and Johnny Mari´ethoz. Torch: A modular machine learning software library. Technical report, IDIAP, 2002.infoscience.epfl.ch/record/82802/files/rr02-46.pdf.

[14] Jeffrey Dean, Gregory S. Corrado, Rajat Monga, Kai Chen, Matthieu Devin, Quoc V. Le, Mark Z. Mao,Marc’Aurelio Ranzato, Andrew Senior, Paul Tucker,17 Ke Yang, and Andrew Y. Ng. Large scale distributed deep networks. In NIPS, 2012. Google Research PDF.

[15] Jack J Dongarra, Jeremy Du Croz, Sven Hammarling,and Iain S Duff. A set of level 3 basic linear algebra subprograms. ACM Transactions on Mathematical Software (TOMS), 16(1):1–17, 1990.
www.maths.manchester.ac.uk/˜sven/pubs/Level3BLAS-1-TOMS16-90.pdf.

[16] Andrea Frome, Greg S Corrado, Jonathon Shlens,Samy Bengio, Jeff Dean, Tomas Mikolov, et al.DeVISE: A deep visual-semantic embedding
model. In Advances in Neural Information Processing Systems, pages 2121–2129, 2013. research.google.com/pubs/archive/41473.pdf.

[17] Javier Gonzalez-Dominguez, Ignacio Lopez-Moreno, Pedro J Moreno, and Joaquin Gonzalez-Rodriguez. Frameby-frame language identification in short utterances using deep neural networks. Neural Networks, 64:49–58, 2015.

[18] Otavio Good. How Google Translate squeezes deep learning onto a phone, 2015.googleresearch.blogspot.com/2015/07/how-googletranslate-
squeezes-deep.html.

[19] Ian J. Goodfellow, Yaroslav Bulatov, Julian Ibarz, Sacha Arnoud, and Vinay Shet. Multi-digit number recognition from Street View imagery using deep convolutional neural networks. In International Conference on Learning Representations, 2014. arxiv.org/pdf/1312.6082.

[20] Georg Heigold, Vincent Vanhoucke, Alan Senior, Patrick Nguyen, Marc’Aurelio Ranzato, Matthieu Devin, and Jeffrey Dean. Multilingual acoustic models using distributed deep neural networks. In Acoustics, Speech and Signal Processing (ICASSP), 2013 IEEE International Conference on, pages 8619–8623. IEEE, 2013. research.
google.com/pubs/archive/40807.pdf.

[21] Geoffrey E. Hinton, Li Deng, Dong Yu, George E.Dahl, Abdel-rahman Mohamed, Navdeep Jaitly, Andrew Senior, Vincent Vanhoucke, Patrick Nguyen,Tara N. Sainath, and Brian Kingsbury. Deep
neural networks for acoustic modeling in speech recognition: The shared views of four research groups. IEEE Signal Process. Mag., 29(6):82–97, 2012. www.cs.toronto.edu/˜gdahl/papers/deepSpeechReviewSPM2012.pdf.

[22] Sepp Hochreiter and J¨urgen Schmidhuber. Long shortterm memory. Neural computation, 9(8):1735–1780,1997. ftp.idsia.ch/pub/juergen/lstm.pdf.

[23] Sergey Ioffe and Christian Szegedy. Batch normalization:
Accelerating deep network training by reducing internal covariate shift. CoRR, abs/1502.03167, 2015.arxiv.org/abs/1502.03167.

[24] Michael Isard, Mihai Budiu, Yuan Yu, Andrew Birrell, and Dennis Fetterly. Dryad: distributed data-parallel programs from sequential building blocks. In ACM SIGOPS Operating Systems Review, volume 41, pages 59–72. ACM, 2007.www.michaelisard.com/pubs/eurosys07.pdf.

[25] Benoˆıt Jacob, Ga¨el Guennebaud, et al. Eigen library for
linear algebra. eigen.tuxfamily.org.

[26] Yangqing Jia, Evan Shelhamer, Jeff Donahue, Sergey Karayev, Jonathan Long, Ross Girshick, Sergio Guadarrama,and Trevor Darrell. Caffe: Convolutional architecture for fast feature embedding. In Proceedings of the ACM International Conference on Multimedia, pages
675–678. ACM, 2014. arxiv.org/pdf/1408.5093.

[27] Andrej Karpathy, George Toderici, Sachin Shetty,Tommy Leung, Rahul Sukthankar, and Li Fei-Fei. Large-scale video classification with convolutional neural networks. In Computer Vision
and Pattern Recognition (CVPR), 2014 IEEE Conference on, pages 1725–1732. IEEE, 2014. research.google.com/pubs/archive/42455.pdf.

[28] A Krizhevsky. Cuda-convnet, 2014.code.google.com/p/cuda-convnet/.

[29] Alex Krizhevsky. One weird trick for parallelizing convolutional neural networks. arXiv preprint arXiv:1404.5997, 2014. arxiv.org/abs/1404.5997.

[30] Alex Krizhevsky, Vinod Nair, and Geoffrey Hinton. The CIFAR-10 dataset. www.cs.toronto.edu/˜kriz/cifar.html.

[31] Quoc Le, Marc’Aurelio Ranzato, Rajat Monga, Matthieu Devin, Greg Corrado, Kai Chen, Jeff Dean, and Andrew Ng. Building high-level features using large scale unsupervised learning. In ICML’2012, 2012. Google Research PDF.

[32] Yann LeCun, Corinna Cortes, and Christopher JC Burges. The MNIST database of handwritten digits,1998. yann.lecun.com/exdb/mnist/.

[33] Mu Li, Dave Andersen, and Alex Smola. Parameter server. parameterserver.org.

[34] Chris J Maddison, Aja Huang, Ilya Sutskever, and David Silver. Move evaluation in Go using deep convolutional neural networks. arXiv preprint arXiv:1412.6564, 2014.arxiv.org/abs/1412.6564.

[35] Tomas Mikolov, Kai Chen, Greg Corrado, and Jeffrey Dean. Efficient estimation of word representations in vector space. In International Conference on Learning Representations: Workshops Track, 2013.arxiv.org/abs/1301.3781.

[36] Derek G Murray, Frank McSherry, Rebecca Isaacs,Michael Isard, Paul Barham, and Mart´ın Abadi. Naiad:a timely dataflow system. In Proceedings of the Twenty-Fourth ACM Symposium on Operating Systems Principles,pages 439–455. ACM, 2013. Microsoft Research PDF.

[37] Derek G. Murray, Malte Schwarzkopf, Christopher Smowton, Steven Smit, Anil Madhavapeddy, and Steven Hand. Ciel: a universal execution engine for distributed data-flow computing. In Proceedings of the Ninth USENIX Symposium on Networked Systems Design and
Implementation, 2011. Usenix PDF.

[38] Arun Nair, Praveen Srinivasan, Sam Blackwell, Cagdas Alcicek, Rory Fearon, Alessandro De Maria, Vedavyas Panneershelvam, Mustafa Suleyman, Charles Beattie, Stig Petersen, et al. Massively parallel methods for deep reinforcement learning. arXiv preprint arXiv:1507.04296, 2015. arxiv.org/abs/1507.04296.

[39] CUDA Nvidia. Cublas library. NVIDIA Corporation,Santa Clara, California, 15, 2008. developer.nvidia.com/cublas.

[40] Jonathan Ragan-Kelley, Connelly Barnes, Andrew Adams, Sylvain Paris, Fr´edo Durand, and Saman Amarasinghe.Halide: A language and compiler for optimizing parallelism, locality, and recomputation in image processing pipelines. ACM SIGPLAN Notices, 48(6):519–
530, 2013. people.csail.mit.edu/fredo/tmp/Halide-5min.pdf.

[41] Bharath Ramsundar, Steven Kearnes, Patrick Riley, Dale Webster, David Konerding, and Vijay Pande. Massively multitask networks for drug discovery. arXiv preprint arXiv:1502.02072, 2015. arxiv.org/abs/1502.02072.

[42] Benjamin Recht, Christopher Re, Stephen Wright, and Feng Niu. Hogwild: A lock-free approach to parallelizing stochastic gradient descent. In Advances in Neural Information Processing Systems, pages 693–701,2011. papers.nips.cc/paper/4390-hogwild-a-lock-freeapproach-
to-parallelizing-stochastic-gradient-descent.

[43] Chuck Rosenberg. Improving Photo Search:A step across the semantic gap, 2013.googleresearch.blogspot.com/2013/06/improvingphoto-search-step-across.html.

[44] Christopher J Rossbach, Yuan Yu, Jon Currey, Jean-Philippe Martin, and Dennis Fetterly. Dandelion: a compiler and runtime for heterogeneous systems. In Proceedings of the Twenty-Fourth ACM Symposium on Operating Systems Principles, pages 49–68. ACM,
2013.research-srv.microsoft.com/pubs/201110/sosp13-dandelion-final.pdf.

[45] David E Rumelhart, Geoffrey E Hinton, and Ronald J Williams. Learning representations by backpropagating errors. Cognitive modeling, 5:3, 1988.www.cs.toronto.edu/ hinton/absps/naturebp.pdf.

[46] Has¸im Sak, Andrew Senior, Kanishka Rao,Franc¸oise Beaufays, and Johan Schalkwyk. Google Voice Search: faster and more accurate, 2015.googleresearch.blogspot.com/2015/09/google-voicesearch-
faster-and-more.html.

[47] Ilya Sutskever, Oriol Vinyals, and Quoc V. Le. Sequence to sequence learning with neural networks. In NIPS,2014. papers.nips.cc/paper/5346-sequence-to-sequencelearning-with-neural.

[48] Christian Szegedy, Wei Liu, Yangqing Jia, Pierre Sermanet,Scott Reed, Dragomir Anguelov, Dumitru Erhan,Vincent Vanhoucke, and Andrew Rabinovich. Going deeper with convolutions. In CVPR’2015, 2015.
arxiv.org/abs/1409.4842.

[49] Seiya Tokui. Chainer: A powerful, flexible and intuitive
framework of neural networks. chainer.org.

[50] Vincent Vanhoucke. Speech recognition and deep learning,
2015.googleresearch.blogspot.com/2012/08/speechrecognition-and-deep-learning.html.

[51] Abhishek Verma, Luis Pedrosa, Madhukar Korupolu,David Oppenheimer, Eric Tune, and John Wilkes.Large-scale cluster management at Google with Borg.In Proceedings of the Tenth European Conference on Computer Systems, page 18. ACM, 2015. research.
google.com/pubs/archive/43438.pdf.

[52] O. Vinyals, L. Kaiser, T. Koo, S. Petrov, I. Sutskever, and G. Hinton. Grammar as a foreign language. Technical report, arXiv:1412.7449, 2014. arxiv.org/abs/1412.7449.

[53] Oriol Vinyals, Meire Fortunato, and Navdeep Jaitly. Pointer networks. In NIPS, 2015.arxiv.org/abs/1506.03134.

[54] Dong Yu, Adam Eversole, Mike Seltzer, Kaisheng Yao, Zhiheng Huang, Brian Guenter, Oleksii Kuchaiev,Yu Zhang, Frank Seide, Huaming Wang, et al. An introduction to computational networks and the computational network toolkit. Technical report, Tech.
Rep. MSR, Microsoft Research, 2014, 2014. research.microsoft.com/apps/pubs/?id=226641.

[55] Matei Zaharia, Mosharaf Chowdhury, Tathagata Das,Ankur Dave, Justin Ma, Murphy McCauley, Michael J Franklin, Scott Shenker, and Ion Stoica. Resilient distributed datasets: A fault-tolerant abstraction for in-memory cluster computing. In Proceedings of the
9th USENIX conference on Networked Systems Design and Implementation. USENIX Association, 2012.www.usenix.org/system/files/conference/nsdi12/nsdi12-final138.pdf.

[56] Matthew D. Zeiler, Marc’Aurelio Ranzato, Rajat Monga,Mark Mao, Ke Yang, Quoc Le, Patrick Nguyen,Andrew Senior, Vincent Vanhoucke, Jeff Dean, and Geoffrey E. Hinton. On rectified linear units
for speech processing. In ICASSP, 2013. research.google.com/pubs/archive/40811.pdf.

