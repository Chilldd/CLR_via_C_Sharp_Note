<h2>3.8 运行时如何解析类型引用</h2>

<img src="https://github.com/Chilldd/CLR_via_C_Sharp_Note/blob/main/IMG/3.8/e59780de77724821b0d25eb66a97daa6.png?raw=true =" width="700px" />

> 编译上图代码并生成程序集(名为MicroService.SystemManage.dll)。运行应用程序，CLR会加载并初始化自身，读取程序集的CLR头，查找标识了应用程序入口方法(Main)的MethodDefToken, 检索MethodDef元数据表找到方法的IL代码在文件中的偏移量，将IL代码JIT编译成本机代码(编译时会对代码进行验证以确保类型安全),最后执行本机代码。下面就是Main方法的IL代码。要查看代码，请对程序集运行ILDasm.exe并选择“视图”|“显示字节”，双击树形视图中的Main方法。

<img src="https://github.com/Chilldd/CLR_via_C_Sharp_Note/blob/main/IMG/3.8/c4a7270238d0433f82c49e912952126f.png?raw=true =" width="700px" />

<img src="https://github.com/Chilldd/CLR_via_C_Sharp_Note/blob/main/IMG/3.8/5aa03a2927754a25979854da649b096b.png?raw=true =" width="700px" />

> 对这些代码进行JIT编译，CLR会检测所有类型和成员引用，加载它们的定义程序集(如果尚未加载)。上述IL代码包含对System.Console.WriteLine的引用。具体地说，IL call指令引用了元数据token **0A000003**。该token标识MemberRef元数据表(表0A)中的记录项3。
> CLR检查该MemberRef记录项，发现它的字段引用了TypeRef表中的记录项
> (System.Console类型)。按照TypeRef记录项，CLR被引导至一个AssemblyRef记录项:
> "mscorlib, Version= =4.0.0.0, Culture= =neutral, PublicKeyToken=b77a5c561934e089"。这时CLR
> 就知道了它需要的是哪个程序集。接着，CLR必须定位并加载该程序集。

