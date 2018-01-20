#C++语法细节

####变量类型
######理解变量类型

* 类型修饰符(*或&)仅仅修饰单个变量
* 从右向左阅读r的定义。离变量名最近的符号对变量的类型有最直接的影响。
```cpp
int *p， q;   //p是int型指针，q是int
int *&r = p;  //r是对指针p的引用
int (*a)[n];  //数组指针：指向int数组的指针
int   *a[n]； //指针数组：[ ]的优先级高，a是一个数组，存放int*类型元素。
```

######声明、定义、初始化
* 变量能且只能被定义一次，但是可以被声明多次。
* 值初始化是值使用了初始化器（即使用了圆括号或花括号）但却没有提供初始值的情况。
	* 注意，当不采用动态分配内存的方式（即不采用new运算符）时，写成int a();是错误的值初始化方式，因为这种方式声明了一个函数而不是进行值初始化。如果一定要进行值初始化，必须结合拷贝初始化使用，即写成int a=int();值初始化和默认初始化一样，对于内置类型初始化为0，对于类类型则调用其默认构造函数。
```cpp
extern int i;			//声明i
int j;       			//声明并定义j
extern double pi=3.1416;//定义并初始化,此时虽然有extern语句，但是因为加了等号,所以成为定义
int *p=new int();       //值初始化
vector< string> vec(10);//值初始化
```

######数组的特殊性
* 数组不允许拷贝和赋值
* 在很多用到数组名字的地方，编译器会自动把其替换成一个指向数组第一个元素的指针。使用auto时是这样，**decltype时这种转变不会发生**。
```cpp
int a[] = {0,1,2};
int a2[] = a;    //错误：不允许使用一个数组初始化另一个数组
a2 = a;          //错误：不允许把一个数组直接赋值给另一个数组
auto ia(a);      //ia是一个整型指针，指向a的第一个元素
decltype(a) ia3={1,2.3}； //decltype(a)返回的类型是由3个元素构成的数组
```

######类型转换
* 无符号类型注意不能赋值成负数。否则值实际为对这个负数取模。
* 赋值给带符号类型一个超出它表示范围的值时，结果是未定义的。
* 显式类型转换（cast）
	* static_cast:处理具有明确定义的类型转换，不包含底层const。，这种强制转换只会在编译时检查。 如果编译器检测到您尝试强制转换完全不兼容的类型，则static_cast会返回错误。
	* reinterpret:处理非关联的类型转换，通过改变对象的位模式。例如 pointer 和 int的无关类型的转换。
	* const_cast:只能改变运算对象的底层const,
	* dynamic_cast：在运行时检查基类指针和派生类指针之间的强制转换。


___
####作用域
######作用域嵌套
* 内层作用域可以重定义外层作用域已有的名字。
* 但可以通过作用域操作符来访问外层变量。::是**作用域操作符**。全局作用域没有名字，所以::name是特指全局作用域里的name变量。


___
####const限定符

######const基本对象
* const对象一旦创建后其值就不能改变，所以const对象必须初始化
* 默认情况下const对象仅在文件内有效，不同文件需要定义同名的const变量，它们是独立的。
* 如果想只在一个文件内定义，可以使用extern关键字。**定义语句前加extern关键字**。
```cpp
//文件1中定义，该常量能被其他文件访问
extern cnost int bufSize = fcn();
//文件2中的声明，与文件1中是同一个
extern const int bufSize;
```

######const与引用
* 对const的引用，简称常量引用，即把引用绑定到const对象上。**底层const**。
* 想要引用常量必须使用常量引用，但常量引用可以引用非常量对象。
* 允许为一个常量引用绑定非常量的对象、字面值，甚至一般表达式。而非常量引用是不可以的。

######const与指针
* 指向常量的指针：想要存放常量对象的地址，只能使用指向常量的指针。**底层const**。
* const指针：指针本身是const，即指针初始化后的值（那个地址）不能改变。**顶层const**。
```cpp
const int a;
const int &b =a;     //常量引用，底层const
const int *p1 = &a;  //指向常量的指针，底层const
int *const p2 = &a;  //常量指针，p2永远指向a,顶层const
```
**在拷贝操作时，拷入和拷出的变量必须具有相同的底层const资格，或者两个对象的数据类型必须能够转换。
一般来说，非常量可以转换成常量，反之则不行**


___
####异常处理

######try语句块

* throw表达式，可以用来引发异常。
* try语句块处理异常，以try关键字开始，并以多个catch子句结束。
* 异常类，用于在throw表达式和相关的catch子句之间传递异常的具体信息。


___
####函数

######尾置返回类型
* 以auto开头，以->返回类型结尾。
```cpp
int (*func(int i))[10];         //直接写法
auto func(int i) -> int(*)[10]; //尾置返回类型。返回一个指针，该指针指向含有10个整数的数组
typedef int arrT[10];			//arrT是一个类型别名
using arrT= int[10];			//arrT的等价声明
arrT* func(int i);              //使用类型别名。返回类型同上
```
######局部静态对象
* 局部静态对象在程序执行路径第一次经过对象定义语句时初始化，直到程序终止才被销毁。在此期间即使对象所在的函数结束执行，也不会对它造成影响。**它的值可以改变。**

######含有可变形参的函数
* initializer_list形参：这是在标准库中定义的一个模板类，其中的对象永远是const值无法改变。而且拷贝或赋值一个initializer_list对象使用的是**传引用**赋值。
* 省略符形参：为了便于C++程序访问某些特殊的C代码而设置的，通常不用于其他目的。
```cpp
    void error_msg(initializer_list<string> il);
    error_msg({"a","b"});
    error_msg({"a","b","c"});
    void foo(parm_list, ...); //逗号是可选的
```

####类
######数据成员和成员函数
* 在类体内定义的成员函数默认为inline函数，在类体外定义的成员函数默认情况下不是内联的。
* 类内初始值有限制：可以放在花括号里，或者等于号右边，**但不能使用圆括号。**

######构造函数
* 默认构造函数：如果存在类内初始值用它们初始化，否则默认初始化。
	
	* **很多时候需要自己定义默认构造函数：**
		1. 当类没有声明任何构造函数时，编译器才会自动生成默认构造函数。
		2. 如果定义在块中的内置类型或符复合类型（比如数组和指针）默认初始化，它们的值将是不确定的。
		3. 如果类内有其他类类型的成员，且这个成员的类型没有默认构造函数，那编译器将无法初始化该成员。
	* = default：在C++11里，如果需要默认行为，可以通过在参数列表后面加上 = default来要求编译器生成构造函数。```Data() = default;  ```
	* 当某个数据成员被构造函数初始值列表忽略时，它将以与合成默认构造函数相同的方式隐式初始化。
* 能通过一个实参调用的构造函数定义了一条从构造函数的参数类型向类类型隐式转换的规则。
* 要意志隐式转换，可以使用explicit关键字。explicit关键字定义的构造函数可以用于显式地强制类型转换。


######友元
* 可以把非成员函数定义为友元，也可以把其他的类定义成友元，还可以把其他类（之前已定义）的成员函数定义为友元。
* 友元不具有传递性，每个类负责控制自己的友元类或友元函数。
* 把一组重载函数作为友元，需要对这组函数中的每一个进行声明。

######静态成员
* 通常类的静态成员不能在类的内部初始化。然而如果静态成员是constexper的就可以。
* 静态成员能用于某些场景，而普通成员不能
	* 静态数据成员可以是不完全类型。
	* 可以使用静态数据成员作为成员函数的默认实参。


####STL
######泛型算法
* 大多定义在头文件algorithm中，还有一部分在numeric中。
* 因为泛型算法，只运行在迭代器之上，而不执行容器操作。**所以泛型算法永远不会改变容器的大小**。即只可能改变元素的值，或者移动元素，淡不会直接添加或者删除元素。使用插入迭代器可以改变容器大小，但这个和算法本身无关。
* 只读算法
	* 通常最好使用cbegin()和cend()。但如果想用算法返回的迭代器来改变元素，就需要使用begin()和end()。
	* 那些只接受一个单一迭代器来表示第二个序列的算法，都假定第二个序列至少与第一个序列一样长。
	* 从两个序列读元素的算法，构成这两个序列的元素可以来自于不同类型的容器。如一个vector，一个list。
* 写容器元素的算法
	* 向目的位置迭代器写如数据的算法假定目的位置足够大，能容纳要写入的元素。
* lambda表达式：[捕获列表] (参数列表) -> 返回类型 { 函数体 }
	* 可以忽略参数列表和返回类型。但必须永远包括捕获列表和函数体。
	* 如果忽略返回类型，lambda根据函数体中的代码推断出返回类型。如果函数体只是一个return语句，则返回类型从返回表达式推断。否则返回类型为void。
	* 捕获值是在创建时拷贝，而不是调用时。
* 参数绑定，bind函数：auto newCallable = bind(callable , arglist);
	* bind函数可以看成一个函数适配器，它接受一个可调用对象，生成一个新的可调用对象来“适应”原对象的参数列表。当我们调用newCallable时，newCallable会调用callable，并传递给它arg_list参数。callable对象原有多少参数，arg_list就应该有多少个。

####内存管理
1.静态内存：保存局部static对象、类static数据成员以及定义在任何函数之外的变量。static对象在使用前分配，在程序结束时销毁。
2.栈内存：保存定义在函数内的非static对象。栈对象仅在其定义的程序块运行时才存在。
3.动态内存:程序用堆来储存动态分配的对象，即那些在程序运行时分配的对象。动态对象的生存期由程序来控制，也就是说必须显式销毁。一般程序使用动态内存出于以下三种原因之一：
* 程序不知道自己需要使用多少对象
* 程序不知道所需对象的准确类型
* 程序需要在多个对象间共享数据

######智能指针
* shared_ptr：我们可以认为每个shared_ptr都有一个关联的计数器，通常称为引用计数。
	* 计数器递增：当用一个shared_ptr初始化另一个shared_ptr，或将它作为参数传递给函数，以及作为函数的返回值。
	* 计数器递减：当给shared_ptr赋一个新值，或者一个局部的shared_ptr离开其作用域。
	* shared_ptr在无用后任然保留的一种情况是，你将shared_ptr存放在一个容器中，随后重排了容器，从而不再需要某些元素。这种情况应该用erase删除。
	* get使用：(1)确定代码不会delete指针的情况下，才使用get。(2)永远不要用get初始化另一个智能指针或者为另一个智能指针赋值。
* unique_ptr：一个unique_ptr“拥有”它所指向的对象，所以不支持普通的拷贝和赋值操作。但可以拷贝或赋值一个将要被销毁的unique_ptr。最常见的例子是从函数返回一个unique_ptr。

######动态数组
```cpp
int n=5;
int *pia = new int[n];//pia指向第一个int
delete [] pia;//pia必须指向一个动态数组或为空
```

######allocator类
* 标准库allocator类定义在memory中，它帮助我们将内存分配和对象构造分离开。











<br>
<br>
<br>
<br>
<br>
<br>
<br>