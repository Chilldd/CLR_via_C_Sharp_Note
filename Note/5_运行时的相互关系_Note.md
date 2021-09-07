<h2>4.4 运行时的相互关系</h2>



> 本节将解释**类型**、**对象**、**线程栈**和**托管堆**在运行时的相互关系。此外，还将解释调用静态方法、实例方法和虚方法的区别。首先从一些计算机基础知识开始。虽然下面讨论的东西不是CLR特有的，但掌握了这些之后，就有了一个良好的理论基础。接着，就可以将我们的讨论转向CLR特有的内容。

<img src="https://github.com/Chilldd/CLR_via_C_Sharp_Note/blob/main/IMG/4.4/f97d6ac874f84434bc22e8d3f9285173.png?raw=true =" width="700px" />

> 然后，M1调用M2方法，将局部变量name作为实参传递。这造成name局部变量中的地址被压入栈(参见图4-4)。M2方法内部使用参数变量s标识栈位置(注意，有的CPU架构用寄存器传递实参以提升性能，但这个区别对于当前的讨论来说并不重要)。另外，调用方法时还会将“**返回地址**”压入栈。被调用的方法在结束之后应**返回至该位置**(同样参见图4-4)。

<img src="https://github.com/Chilldd/CLR_via_C_Sharp_Note/blob/main/IMG/4.4/982ee69fd17a4a0f870635ceddf8459d.png?raw=true =" width="700px" />

> M2方法开始执行时，它的“序幕”代码在线程栈中为局部变量length和tally 分配内存，如图4-5所示。然后，M2方法内部的代码开始执行。最终，M2抵达它的return语句，造成CPU的指令指针被设置成栈中的返回地址，M2的栈帧"展开(unwind)”，恢复成图4-3的样子。之后，M1继续执行M2调用之后的代码，M1的栈帧将准确反映M1需要的状态。

<img src="https://github.com/Chilldd/CLR_via_C_Sharp_Note/blob/main/IMG/4.4/23812cf6b59a4abfac46d669e2cdf91f.png?raw=true =" width="700px" />

> 最终，M1会返回到它的调用者。这同样通过将CPU的指令指针设置成返回地址来实现(这个返回地址在图中未显示，但它应该刚好在栈中的name实参上方)，M1的栈帧展开(unwind)，恢复成图4-2的样子。之后，调用M1的方法继续执行M1调用之后的代码，那个方法的栈帧将准确反映它需要的状态。

> 1. 栈帧(stack frame)代表当前线程的调用栈中的一个方法调用。执行线程的过程中，进行的每个方法调用都会在调用栈中创建并压入一个StackFrame。 一译注
> 2. unwind 一般翻译成“展开”，但这并不是一个很好的翻译。wind 和unwind源于生活。把线缠到线圈上称为wind;从线圈上松开称为unwind。同样地，调用方法时压入栈帧，称为wind; 方法执行完毕，弹出栈帧，称为unwind。把这几张图的线程栈看成一个线圈，就很容易理解了。一译注



<img src="https://github.com/Chilldd/CLR_via_C_Sharp_Note/blob/main/IMG/4.4/0c8a24b077be40d486080b2f6b59bac3.png?raw=true =" width="700px" />

> JIT编译器将M3的IL代码转换成本机CPU指令时，会注意到M3内部引用的所有类型，包括Employee, Int32, Manager 以及String("Joe")。这时CLR要**确认**定义了这些类型的所有程序集**都已加载**。然后，利用程序集的**元数据**，CLR提取与这些类型有关的信息, 创建一些**数据结构来表示类型本身**。图4-7展示了为Employee和Manager**类型对象**使用的数据结构。由于线程在调用M3前已执行了一些代码，所以不妨假定Int32和String类型对象已经创建好了(这是极有可能的，因为它们都是很常用的类型)，所以图中没有显示它们。

<img src="https://github.com/Chilldd/CLR_via_C_Sharp_Note/blob/main/IMG/4.4/2b5e656d4e004d7298ca16993622b92d.png?raw=true =" width="700px" />

> 当CLR确认方法需要的所有类型对象都已创建，M3的代码已经编译之后，就允许线程执行M3的本机代码。M3的“序幕”代码执行时必须在线程栈中为局部变量分配内存，如图4-8所示。顺便说一句，作为方法“序幕”代码的一部分，CLR自动将所有局部变量初始化为null 或0(零)。然而，如果代码试图访问尚未显式初始化的局部变量，C#会报告错误消息:使用了未赋值的局部变量。

> 然后，M3执行代码构造一个Manager对象。这造成在托管堆创建Manager类型的一个实例(也就是一个Manager对象)，如图4-9所示。可以看出，和所有对象一样，Manager 对象也有类型对象指针和同步块索引。该对象还包含必要的字节来容纳Manager类型定义的**所有实例数据字段**，以及容纳由Manager的任何基类(本例就是Employee和Object)定义的所有实例字段。**任何时候在堆上新建对象，CLR都自动初始化内部的“类型对象指针”成员来引用和对象对应的类型对象(本例就是Manager类型对象)**。此外，在调用类型的构造器(本质上是可能修改某些实例数据字段的方法)之前，CLR会先初始化同步块索引，并将对象的所有实例字段设为null或0(零)。new操作符返回Manager对象的内存地址，该地址保存到变量e中(e在线程栈上)。

<img src="https://github.com/Chilldd/CLR_via_C_Sharp_Note/blob/main/IMG/4.4/87367fb49d02496dbbe4cf6025ba105f.png?raw=true =" width="700px" />

> M3的下一行代码调用Employee的**静态方法Lookup**。调用静态方法时，CLR会定位与**定义静态方法的类型**对应的**类型对象**。然后，JIT编译器在**类型对象的方法表**中查找与被调用方法对应的记录项，对方法进行JIT编译(如果需要的话)，再调用JIT编译好的代码。本例假定Employee的Lookup方法要查询数据库来查找Joe。再假定数据库指出Joe是公司的一名经理，所以在内部，Lookup 方法在堆上构造一个 新的Manager对象，用Joe的信息初始化它，返回该对象的地址。该地址保存到局部变量e中。这个操作的结果如图4-10所示。
>
> 注意，e不再引用第一个Manager对象。事实上，由于没有变量引用该对象，所以它是未来垃圾回收的主要目标。垃圾回收机制将自动回收(释放)该对象占用的内存。
>
> M3的下一行代码调用Employee的**非虚实例方法**GetYearsEmployed。调用非虚实例方法时，JIT编译器会找到与“**发出调用的那个变量(**e)**的类型**(Employee)”**对应的类型对象**(Employee类型对象)。这时的变量e被定义成一个Employee。如果Employee类型没有定义正在调用的那个方法，JIT编译器会**回溯类层次结构**(一直回溯到Object)，并在沿途的每个类型中查找该方法。之所以能这样回溯，是因为每个类型对象都有一个字段引用了它的基类型，这个信息在图中没有显示。

<img src="https://github.com/Chilldd/CLR_via_C_Sharp_Note/blob/main/IMG/4.4/e23f960aea0d4610bfa59ae9f115bc72.png?raw=true =" width="700px" />

> 然后，JIT编译器在类型对象的方法表中查找引用了被调用方法的记录项，对方法进行JIT编译(如果需要的话),再调用JIT编译好的代码。本例假定Employee的GetYearsEmployed方法返回5，因为Joe已被公司雇用5年。这个整数保存到局部变量year中。这个操作的结果如图4-11所示。

> gy_note：
>
> 调用静态方法：
>
> 1. 找到定义静态方法的类
> 2. 找到这个类对应的类型对象
> 3. 在类型对象的方法表上寻找方法
>
> 调用实例方法：
>
> 1. 找到调用方法的变量
> 2. 找到变量类型
> 3. 找到变量类型对应的类型对象
> 4. 在类型对象的方法表上寻找方法，如果找不到，就去找基类的方法，一直到object

> M3的下一行代码调用Employee的**虚实例方法**GetProgressReport。调用虚实例方法时，JIT编译器要在方法中生成一些**额外的代码**; 方法每次调用**都会执行这些代码**。这些代码首先检查发出调用的变量，并跟随地址来到发出调用的对象。变量e当前引用的是代表“Joe”的Manager对象。然后，代码检查对象内部的“**类型对象指针**”成员，该成员指向对象的实际类型。然后，代码在类型对象的方法表中查找引用了被调用方法的记录项，对方法进行JIT编译(如果需要的话)，再调用JIT编译好的代码。由于目前e引用一个Manager对象，所以会调用Manager的GetProgressReport实现。这个操作的结果如图4-12所示。

<img src="https://github.com/Chilldd/CLR_via_C_Sharp_Note/blob/main/IMG/4.4/4db638708bd4402b94c7957555ff3135.png?raw=true =" width="700px" />

<img src="https://github.com/Chilldd/CLR_via_C_Sharp_Note/blob/main/IMG/4.4/ffef8e8100d74b74a6b8da7eac50770e.png?raw=true =" width="700px" />

> 注意，如果Employee的Lookup方法发现Joe是Employee而不是Manager, Lookup 会在内部构造一个 Employee对象，它的类型对象指针将引用Employee类型对象。这样最终执行的就是Employee的GetProgressReport实现，而不是Manager的。

> 至此，我们已经讨论了源代码、IL 和JIT编译的代码之间的关系。还讨论了线程栈、实参、局部变量以及这些实参和变量如何引用托管堆上的对象。还知道对象含有一个指针指向对象的类型对象(类型对象中包含静态字段和方法表)。还讨论了JIT编译器如何决定静态方法、非虚实例方法以及虚实例方法的调用方式。理解这一-切之后，可以深刻地认识CLR的工作方式。以后在建构、设计和实现类型、组件以及应用程序时，这些知识会带来很大帮
> 助。
> 
> 结束本章之前，让我们深入探讨一下CLR内部发生的事情。
> 注意Employee和Manager类型对象都包含“类型对象指针”成员。这是由于**类型对象本质上也是对象**。CLR创建类型对象时，必须初始化这些成员。初始化成什么呢? 你肯定会这样问。**CLR开始在一个进程中运行时，会立即为MSCorLib.dll中定义的System.Type类型创建一个特殊的类型对象**。Employee 和Manager 类型对象都是**该类型的“实例”**。因此，它们的**类型对象指针**成员会初始化成**对System.Type类型对象的引用**，如图4-13所示。
> 当然，**System.Type 类型对象本身也是对象，内部也有“类型对象指针”成员**。这个指针指向什么? **它指向它本身**，因为System.Type类型对象本身是一个类型对象的“实例”。
> 现在，我们总算理解了CLR的整个类型系统及其工作方式。顺便说一句，**System.Object的GetType方法返回存储在指定对象的“类型对象指针”成员中的地址**。也就是说, GetType方法返回指向对象的类型对象的指针。这样就可判断系统中任何对象(包括类型对象本身)的真实类型。

<img src="https://github.com/Chilldd/CLR_via_C_Sharp_Note/blob/main/IMG/4.4/67eb99bda5d04b729e75f794fc65e54d.png?raw=true =" width="700px" />

> gy_note:
>
> 类型对象：
>
> 每个对象都是一个类型的实例，而每个类型本身都有一个Type类型的实例来表示，对象的类型指针就是指向该类型的Type实例的指针。举个例子就清楚多了，我们知道，typeof（String）的值是一个Type类型的实例，这个Type类型的实例也就是所有的String对象的类型指针所指向的东西。
>
> 摘自《https://www.cnblogs.com/instance/archive/2012/09/10/2679297.html》
>
> 
>
> 任何时候在堆上创建对象，CLR都自动初始化内部的“类型对象指针”成员来引用与对象对应的类型对象。在JIT编译器将IL代码转换成本机CPU指令的时候，利用程序集的元数据，CLR提取与代码中类型有关的信息，创建一些数据结构来表示类型本身。
>
> CLR开始在一个进程中运行时，利用MSCorLib.dll中定义的System.Type类型创建一个特殊的类型对象，代码中的类型对象都是该类型的“**实例**”，因此，它们的**类型对象指针指向的就是System.Type类型对象**。
>
> System.Object的GetType方法返回存储在指定对象的“类型对象指针”成员中的地址。也就是说，GetType方法返回指向对象的类型对象的指针。这样就可以判断系统中任何对象（包括类型对象本身）的真实类型。
>
> 
>
> 静态字段，方法表都存在类型对象中。
