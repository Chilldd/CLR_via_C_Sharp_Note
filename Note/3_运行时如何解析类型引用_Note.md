<h2>3.8 运行时如何解析类型引用</h2>

>  示例代码

<img src="https://github.com/Chilldd/CLR_via_C_Sharp_Note/blob/main/IMG/3.8/e59780de77724821b0d25eb66a97daa6.png?raw=true =" width="700px" />

> 编译上图代码并生成程序集(名为MicroService.SystemManage.dll)。运行应用程序，CLR会加载并初始化自身，读取程序集的CLR头，查找标识了应用程序入口方法(Main)的MethodDefToken, 检索MethodDef元数据表找到方法的IL代码在文件中的偏移量，将IL代码JIT编译成本机代码(编译时会对代码进行验证以确保类型安全),最后执行本机代码。下面就是Main方法的IL代码。要查看代码，请对程序集运行ILDasm.exe并选择“视图”|“显示字节”，双击树形视图中的Main方法。



> 示例代码的IL和元数据

<img src="https://github.com/Chilldd/CLR_via_C_Sharp_Note/blob/main/IMG/3.8/c4a7270238d0433f82c49e912952126f.png?raw=true =" width="700px" />

<img src="https://github.com/Chilldd/CLR_via_C_Sharp_Note/blob/main/IMG/3.8/5aa03a2927754a25979854da649b096b.png?raw=true =" width="700px" />

<img src="https://github.com/Chilldd/CLR_via_C_Sharp_Note/blob/main/IMG/3.8/10fb6a38b237492183bbe9e127a7b74f.png?raw=true =" width="700px" />

> 对这些代码进行JIT编译，CLR会检测所有类型和成员引用，加载它们的定义程序集(如果尚未加载)。上述IL代码包含对System.Console.WriteLine的引用。具体地说，IL call指令引用了元数据token **0A000003**。该token对应MemberRef元数据表中的记录项。CLR检查该MemberRef记录项，发现它的字段引用了TypeRef表中的记录项(System.Console类型)。按照TypeRef记录项，CLR被引导至一个AssemblyRef记录项。这时CLR就知道了它需要的是哪个程序集。接着，CLR必须定位并加载该程序集。
>
> 解析引用的类型时，CLR可能在以下三个地方找到类型：
>
> 1. 相同文件
>    编译时便能发现对相同文件中的类型的访问，这称为早期绑定(early binding)”。类型直接从文件中加载，执行继续。
>
> 2. 不同文件，相同程序集
>    “运行时”确保被引用的文件在当前程序集元数据的FileDef表中，检查加载程序集清单文件的目录，加载被引用的文件，检查哈希值以确保文件完整性。发现类型的成员，执行继续。
> 3. 不同文件，不同程序集
>    如果引用的类型在其他程序集的文件中，“运行时”会加载被引用程序集的清单文件。如果需要的类型不在该文件中，就继续加载包含了类型的文件。发现类型的成员，执行继续。
>
> 解析类型引用时有任何错误(找不到文件、文件无法加载、哈希值不匹配等)都会抛出相应异常。
>
> 在上例中, CLR发现System.Console在和调用者不同的程序集中实现。所以，CLR必须查找那个程序集，加载包含程序集清单的PE文件。然后扫描清单，判断是哪个PE文件实现了类型。如果被引用的类型就在清单文件中，一切都很简单。如果类型在程序集的另一个文件中，CLR必须加载那个文件，并扫描其元数据来定位类型。然后，CLR创建它的内部数据结构来表示类型，JIT 编译器完成Main方法的编译。最后，Main方法开始执行。图3-2演示了类型绑定过程。

<img src="https://github.com/Chilldd/CLR_via_C_Sharp_Note/blob/main/IMG/3.8/5a1fb74540704e81b8e41fe71fa894ad.png?raw=true =" width="700px" />

> 部分摘自《CLR via C# 》第七十页至七十二页，示例代码、反编译代码，元数据等是自己测试截图

