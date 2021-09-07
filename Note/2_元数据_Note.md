<h2>2.3元数据概述</h2>



> <img src="https://github.com/Chilldd/CLR_via_C_Sharp_Note/blob/main/IMG/2.3/873d7eb28f09467da95843b8665a6b8d.png?raw=true =" width="700px" />
>
> 编译器编译源代码时，代码定义的任何东西都导致在表2-1列出的某个表中创建一个记录项。此外，编译器还会检测源代码引用的类型、字段、方法、属性和事件，并创建相应的元数据表记录项。在创建的元数据中包含一组引用表，它们记录了所引用的内容。表2-2总结了常用的引用元数据表。
>
> <img src="https://github.com/Chilldd/CLR_via_C_Sharp_Note/blob/main/IMG/2.3/6f8b473e267441f9b233306f9844e12d.png?raw=true =" width="700px" />
>
> <img src="https://github.com/Chilldd/CLR_via_C_Sharp_Note/blob/main/IMG/2.3/83ea863780334061a846b7c2354055ee.png?raw=true" width="700px" />
>
> <a href="https://docs.microsoft.com/zh-cn/dotnet/framework/tools/ildasm-exe-il-disassembler">ILDASM工具(IL反汇编工具)</a>
>
> <img src="https://github.com/Chilldd/CLR_via_C_Sharp_Note/blob/main/IMG/2.3/8106dd9362274e4e9d90a8a2a991e69a.png?raw=true" width="700px" />
>
> <img src="https://github.com/Chilldd/CLR_via_C_Sharp_Note/blob/main/IMG/2.3/08c606662e3b403dbbb03f148f218888.png?raw=true" width="700px" />
>
> <img src="https://github.com/Chilldd/CLR_via_C_Sharp_Note/blob/main/IMG/2.3/f256570e98254524b68822b86abf44f4.png?raw=true" width="700px" />
>
> <img src="https://github.com/Chilldd/CLR_via_C_Sharp_Note/blob/main/IMG/2.3/26c4a137c3b14a5ab54de392742faa70.png?raw=true" width="700px" />
>
> 从中可以看出文件大小(字节数)以及文件各部分大小(字节数和百分比)。对于这个Program.exe应用程序，PE头和元数据占了相当大的比重。事实上，IL 代码只有很少的字节。当然，随着应用程序规模的增大，它会重用大多数类型以及对其他类型和程序集的引用，元数据和头信息在整个文件中的比重越来越小。



