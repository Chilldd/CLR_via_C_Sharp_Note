<h2>1.4 执行程序集的代码</h2>

<h3>IL</h3>

> 托管程序集同时包含元数据和IL。IL是与CPU无关的机器语言，是Microsoft在请教了外面的几个商业及学术性语言/编译器的作者之后，费尽心思开发出来的。IL比大多数CPU机器语言都**高级**。IL能访问和操作对象类型，并提供了指令来创建和初始化对象、调用对象上的虚方法以及直接操作数组元素。甚至提供了抛出和捕捉异常的指令来实现错误处理。可将IL视为一种**面向对象的机器语言**。
>
> 摘自《CLR via C# 》第十一页



<h3>IL如何转换为CPU指令/CLR如何进行方法调用</h3>

> 为了执行方法，首先必须把方法的IL转换成本机(native)CPU指令。这是 CLR 的 JIT (just-in-time或者“即时”)编译器的职责。
>
> <img src="https://github.com/Chilldd/CLR_via_C_Sharp_Note/blob/main/IMG/1.4/be43483a314945f2bdde6c2d48594c02.png?raw=true =" width="700px" />
>
> 就在Main方法执行之前，CLR会检测出Main的代码引用的所有类型。这导致CLR分配一个**内部数据结构来管理对引用类型的访问**。图1-4的Main方法引用了一个Console类型，导致CLR分配一个内部结构。在这个内部数据结构中，**Console 类型定义的每个方法都有一个对应的记录项**。每个记录项都含有一个地址，根据此**地址即可找到方法的实现**。对这个结构初始化时，CLR将每个记录项都设置成(指向)包含在CLR内部的一个**未编档函数**。我将该函数称为**JITCompiler**。
> Main方法**首次调用WriteLine时，JITCompiler 函数会被调用**。**JITCompiler函数负责将方法的IL代码编译成本机CPU指令**。由于IL是“**即时**”(just in time)编译的，所以通常将CLR的这个组件称为**JITter或者JIT编译器**。
>
> JITCompiler函数被调用时，它知道要调用的是哪个方法，以及具体是什么类型定义了该方法。然后，JITCompiler会在定义(该类型的)程序集的**元数据中查找被调用方法的IL**。接着，JITCompiler**验证IL代码**，并将IL代码**编译成本机CPU指令**。**本机CPU指令保存到动态分配的内存块中**。然后，JITCompiler 回到CLR为类型创建的内部数据结构，找到与被调用方法对应的那条记录，修改最初对JITCompiler的引用，使其指向内存块(其中包含了刚才编译好的本机CPU指令)的地址。最后，JITCompiler函数跳转到内存块中的代码。这些代码正是WriteLine方法(获取单个String参数的那个版本)的具体实现。代码执行完毕并返回时，会回到Main中的代码，并像往常一样继续执行。
>
> <img src="https://github.com/Chilldd/CLR_via_C_Sharp_Note/blob/main/IMG/1.4/07ce396b5a314c64afb948075a5a6370.png?raw=true =" width="700px" />
>
> 摘自《CLR via C# 》第十一页至十三页
>
> > gy_note：
> >
> > 简单来说，CLR内部有一个数据结构管理对所有引用类型的访问，每个引用类型都有自己的方法表，方法表中每个方法默认对应一个未编档函数(JITCompiler)。当方法首次调用时，会触发JITCompiler函数，JIT函数会找到方法的IL代码，并将IL代码转换为本机CPU指令存储在内存中，然后修改方法表中方法的指针，指向刚才编译好的CPU指令，这个操作是实时的。方法第一次执行完后，由于IL已经被编译成CPU指令保存在内存中了，所以后续执行就不会在触发JIT函数。
> >
> > 另：由于编译的CPU指令是保存在内存中，所以项目停止运行后，编译好的CPU指令会被丢弃。
> >
> > JIT函数执行非常快，并且每个方法只会执行一次，所以不必担心性能问题。



> 我个人认为，IL最大的优势不是它对底层CPU的抽象，而是应用程序的**健壮性和安全性**。将IL编译成本机CPU指令时，CLR执行一个名为**验证(verification)**的过程。这个过程会检查高级IL代码，确定代码所做的一切都是**安全**的。例如，会核实调用的每个方法都有正确数量的参数，传给每个方法的每个参数都有正确的类型，每个方法的返回值都得到了正确的使用，每个方法都有一个返回语句，等等。托管模块的元数据包含验证过程要用到的所有方法及类型信息。
>
> Windows的每个进程都有自己的虚拟地址空间。独立地址空间之所以必要，是因为不能简单地信任一个应用程序的代码。应用程序完全可能读写无效的内存地址(令人遗憾的是，这种情况时有发生)。将每个Windows进程都放到独立的地址空间，将获得健壮性与稳定性: 一个进程干扰不到另一个进程。
> 然而，通过验证托管代码，可确保代码**不会不正确地访问内存，不会干扰到另一个应用程序的代码**。这样就可以放心地将多个托管应用程序放到同一个Windows虚拟地址空间运行。
> 由于Windows进程需要大量操作系统资源，所以进程数量太多，会损害性能并制约可用的资源。用一个进程运行多个应用程序，可减少进程数，从而增强性能，减少所需的资源，健壮性也没有丝毫下降。这是托管代码相较于非托管代码的另一个优势。
>
> 事实上，CLR确实提供了在一个操作系统进程中执行多个托管应用程序的能力。每个托管应用程序都在一个AppDomain中执行。每个托管EXE文件默认都在它自己的独立地址空间中运行，这个地址空间只有一个AppDomain。然而，CLR的宿主进程(比如IIS 或者Microsoft SQL Server)可决定在一个进程中运行多个AppDomain。第22章“CLR寄宿和AppDomain”会详细讨论AppDomain.
>
> 摘自《CLR via C# 》第十五页至十六页

