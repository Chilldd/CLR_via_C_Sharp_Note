<h2>5.2/5.3 引用/值类型</h2>

<h3>引用类型值类型</h3>

> CLR支持两种类型: 引用类型和值类型。
>
> 虽然FCL的大多数类型都是引用类型，但程序员用得最多的还是值类型。引用类型总是从**托管堆**分配，C#的new操作符返回对象内存地址一即**指向对象数据的内存地址**。使用引用类型必须留意性能问题。首先要认清楚以下四个事实：
>
> 1. 内存必须从托管堆分配。
> 2. 堆上分配的每个对象都有一些额外成员，这些成员必须初始化。
> 3. 对象中的其他字节(为字段而设)总是设为零。
> 4. 从托管堆分配对象时，可能强制执行一次垃圾回收。
>
> 
>
> 如果所有类型都是引用类型，应用程序的性能将显著下降。设想每次使用Int32 值时都进行一次内存分配，性能会受到多么大的影响! 为了提升简单和常用的类型的性能，CLR提供了名为“值类型”的轻量级类型。值类型的实例一般在**线程栈**上分配(虽然也可作为字段嵌入引用类型的对象中)。在**代表值类型实例的变量中不包含指向实例的指针**。相反，**变量中包含了实例本身的字段**。由于变量已包含了实例的字段，所以操作实例中的字段不需要提领指针。**值类型的实例不受垃圾回收器的控制**。因此，**值类型的使用缓解了托管堆的压力，并减少了应用程序生存期内的垃圾回收次数**。
>
> > 文档清楚指出哪些类型是引用类型，哪些是值类型。在文档中查看类型时，任何称为“类”的类型都是引用类型。例如，System.Exception 类、System.IO.FileStream 类以及System.Random类都是引用类型。相反，所有值类型都称为结构或枚举。例如，System.Int32
> > 结构、System.Boolean结构、System.Decimal结构、System.TimeSpan结构、System.DayOfWeek枚举、System.I0.filAtributes 枚举以及System.Drawing.FontStyle 枚举都是值类型。
>
> > 进一步研究文档，会发现所有**结构都是抽象类型System.ValueType的直接派生类**。**System.ValueType本身又直接从System.Object 派生**。根据定义，所有值类型都必须从System.ValueType派生。所有枚举都从System.Enum 抽象类型派生，后者又从System.ValueType派生。CLR和所有编程语言都给予枚举特殊待遇”。欲知枚举类型的详情，请参见第15章“枚举类型和位标志”
>
> 虽然不能在定义值类型时为它选择基类型，但如果愿意，**值类型可实现一个或多个接口**。除此之外，所有值类型都**隐式密封**，目的是**防止将值类型用作其他引用类型或值类型的基类型**。例如，无法将Boolean, Char，Int32， Ulnt64， Single， Double, Decimal 等作为基类型来定义任何新类型
>
> <img src="https://github.com/Chilldd/CLR_via_C_Sharp_Note/blob/main/IMG/5.2/642a8f3dd1904d3985eff66f930486a1.png?raw=true =" width="700px" />
>
> <img src="https://github.com/Chilldd/CLR_via_C_Sharp_Note/blob/main/IMG/5.2/0a07100986f4477191dad4e14d417fe8.png?raw=true =" width="700px" />
>
> 设计自己的类型时，要仔细考虑类型是否应该定义成值类型而不是引用类型。值类型有时能提供更好的性能。具体地说，除非满足以下全部条件，否则不应将类型声明为值类型: 
>
> 1. 类型具有基元类型的行为。也就是说，是十分简单的类型，没有成员会修改类型的何实例字段。如果类型没有提供会更改其字段的成员,就说该类型是不可变(immutable)类型。事实上，对于许多值类型，我们都建议将全部字段标记为readonly(详情参见第7章“常量和字段”)。
> 2. 类型不需要从其他任何类型继承。
> 3. 类型也不派生出其他任何类型。
>
> **类型实例大小**也应在考虑之列，因为**实参默认以传值方式传递**，**造成对值类型实例中的字段进行复制**，对性能造成损害。同样地，被定义为返回一个值类型的方法在返回时，实例中的字段会复制到调用者分配的内存中，对性能造成损害。所以，要将类型声明为值类型，除了要满足以上全部条件，还必须满足以下任意条件。
>
> 1. 类型的实例较小(16字节或更小)。
> 2. 类型的实例较大(大于16字节)，但不作为方法实参传递，也不从方法返回。
>
> 值类型的**主要优势是不作为对象在托管堆上分配**。当然，与引用类型相比，值类型也存在自身的一些局限。下面列出了值类型和引用类型的一些区别: 
>
> 1. 值类型对象有两种表示形式: **未装箱**和**已装箱**，详情参见下一节。相反，引用类型总是处于**已装箱**形式。
> 2. 值类型从 System.ValueType 派生。该类型提供了与System.Object 相同的方法。但System.ValueType重写了Equals 方法，能在两个对象的字段值完全匹配的前提下返回true。此外，System.ValueType 重写了GetHashCode 方法。生成哈希码时，这个重写方法所用的算法会将对象的实例字段中的值考虑在内。由于这个默认实现存在性能问题，所以定义自己的值类型时应写Equals和GetHashCode方法，并提供它们的显式实现。本章末尾会讨论Equals和GetHashCode方法。
> 3. 由于不能将值类型作为基类型来定义新的值类型或者新的引用类型，所以不应在值类型中引入任何新的虚方法。所有方法都不能是抽象的，所有方法都隐式密封(不可重写)。
> 4. 引用类型的变量包含堆中对象的地址。**引用类型的变量创建时默认初始化为null，表明当前不指向有效对象**。试图使用null引用类型变量会抛出NullReferenceException异常。相反，值类型的变量总是包含其基础类型的一个值，而且值类型的所有成员都初始化为0。值类型变量不是指针，访问值类型不可能抛出NullReferenceException异常。CLR确实允许为值类型添加“可空”(nullability)标识。可空类型将在第19章“可空值类型”详细讨论。
> 5. 将值类型变量赋给另一个值类型变量，会执行**逐字段的复制**。将引用类型的变量赋给另一个引用类型的变量**只复制内存地址**。
> 6. 基于上一条，两个或多个引用类型变量能引用堆中同一个对象，所以对一个变量执行的操作可能影响到另一个变量引用的对象。相反，值类型变量自成一体，对值类型变量执行的操作不可能影响另一个值类型变量。
> 7. 由于未装箱的值类型不在堆上分配，一旦定义了该类型的一个实例的方法不再活动，为它们分配的存储就会被释放，而不是等着进行垃圾回收。
>
> <img src="https://github.com/Chilldd/CLR_via_C_Sharp_Note/blob/main/IMG/5.2/dd0faac110c04c5b99d60c1fc0bd11e6.png?raw=true =" width="700px" />
>
> 摘自《CLR via C# 》第一百零六页至一百一十页

<h3>装箱拆箱</h3>

> 将值类型转换成引用类型要使用装箱机制。下面总结了对值类型的实例进行装箱时所发生的事情。
> 1. 在托管堆中分配内存。分配的内存量是值类型各字段所需的内存量，还要加上托管堆所有对象都有的两个额外成员(类型对象指针和同步块索引)所需的内存量。
> 2. 值类型的字段复制到新分配的堆内存。
> 3. 返回对象地址。现在该地址是对象引用，值类型成了引用类型。
>
> C#编译器自动生成对值类型实例进行装箱所需的IL代码，但仍需理解内部发生的事情，对代码长度和性能心中有数。
> C#编译器自动生成对值类型实例进行装箱所需的IL代码,但你仍然需要理解内部的工作机制才能体会到代码的大小和性能问题。
>
> 知道装箱如何进行后，接着谈谈拆箱。假定要用以下代码获取ArrayList的第一个元素:
> Point p = (Point) a[0] ;
> 它获取ArrayList的元素0包含的引用(或指针)，试图将其放到Point值类型的实例p中。为此，已装箱Point对象中的所有字段都必须复制到值类型变量p中，后者在线程栈上。CLR分两步完成复制: 
>
> 1. 获取已装箱Point对象中的各个Point字段的地址。这个过程称为拆箱(unboxing)。
> 2. 将字段包含的值从堆复制到基于栈的值类型实例中。
>
> 拆箱不是直接将装箱过程倒过来。拆箱的代价比装箱低得多。拆箱其实就是获取指针的过程，该指针指向包含在一个对象中的原始值类型(数据字段)。其实，指针指向的是已装箱实例中的未装箱部分。所以和装箱不同，拆箱不要求在内存中复制任何字节。知道这个重要区别之后，还应知道的一个重点是，往往紧接着土拆箱发生一次字段复制。 
>
> 已装箱值类型实例在拆箱时，内部发生下面这些事情。
> 1. 如果包含“对已装箱值类型实例的引用”的变量为null, 抛出NullReferenceException异常。
> 2. 如 果引用的对象不是所需值类型的已装箱实例，抛出InvalidCastException异常。
>
> 
>
> <img src="https://github.com/Chilldd/CLR_via_C_Sharp_Note/blob/main/IMG/5.2/8f74e74fe2c64e219d4621b4b20505aa.png?raw=true =" width="700px" />
>
> 前面说过，未装箱值类型比引用类型更“轻”。这要归结于以下两个原因。
>
> 1. 不在托管堆上分配。
> 2. 堆上的每个对象都有的额外成员: “ 类型对象指针”和“同步块索引”。
>
> **由于未装箱值类型没有同步块索引，所以不能使用System.Threading.Monitor类型的方法(或者C#lock语句)让多个线程同步对实例的访问。**
>
> 虽然未装箱值类型没有类型对象指针，但仍可调用由类型继承或重写的虚方法(比如Equals，GetHashCode 或者ToString)。如果值类型重写了其中任何虚方法，那么CLR可以**非虚**的调用该方法，因为值类型隐式密封，不可能有类型从它们派生，而且调用虚方法的值类型实例没有装箱。然而，如果重写的虚方法要**调用在基类中的实现**，那么在调用基类的实现时，值类型实例会装箱，以便能够通过**this指针将对一个堆对象的引用传给基方法**。但在调用非虚的、继承的方法时(比如GetType或MemberwiseClone)，无论如何都要对值类型进行装箱。因为这些方法由System.Object定义，**要求this实参是指向堆对象的指针**。
>
> 此外，将值类型的未装箱实例转型为类型的某个接口时要对实例进行装箱。这是因为接口变量必须包含对堆对象的引用(接口主题将在第13章“接口”中讨论)。以下代码对此进行了演示: 
>
> <img src="https://github.com/Chilldd/CLR_via_C_Sharp_Note/blob/main/IMG/5.2/99006589e30e464a9685faed86b30f64.png?raw=true =" width="700px" />

