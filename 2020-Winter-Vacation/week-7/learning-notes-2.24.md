# Learning notes of Week 7

## 2.24 Mon.

+ new 后面是 `()` 还是 `[]` 是有区别的，`()` 意思是以圆括号内的实参调用构造函数，而 `[]` 意思是申请一个该类型的数组，长度为方括号内的值：

  ```c++
  char *a = new char(55);
  char *b = new char[55];
  cout << strlen(a) << endl;  // 输出 1，因为初始化了一个 char
  cout << strlen(b) << endl;  // 输出 0，申请了数组，但是没初始化
  ```

+ sizeof 无法返回动态数组的大小，因为 sizeof 的值是编译时常量，而 new 的大小需要到运行时刻才能计算出来：

  ```c++
  int *a = new int(5);
  int *b = new int[5];
  int c[5];
  cout << sizeof a << endl;  // 8，指针
  cout << sizeof b << endl;  // 8，指针
  cout << sizeof c << endl;  // 20，5 个 int
  ```

+ sizeof 后面到底需不需要加括号？

  sizeof 本质是一个运算符，如果 sizeof 作用于类型名，需要加括号，否则编译器会将类型名看成变量声明；如果 sizeof 作用于表达式，由于 sizeof 优先级比大部分表达式运算符高，所以最好加上括号，但是如果表达式仅仅是一个变量名，则不需要加括号。

+ 悬挂指针（野指针）：指向用户不该使用的内存的指针。可能的原因主要有两个：

  + 指针变量没有初始化，如果没有初始化为 null 的话，指针变量的默认值是随机的。
  + 指向的内存被释放了，而这个指针仍然指在那，比如 free 一个指针释放了它指向的内存但并未将该指针置为 null，再比如指针指向的对象是 local 的，超出作用域后被释放了。

+ 拷贝构造函数和赋值函数：

  C++ 的空类默认会有默认构造函数、析构函数、拷贝构造函数以及赋值函数（赋值运算符重载）

  + 拷贝构造函数和赋值函数接受一个该类对象的**引用**作为入参。拷贝构造函数入参必须是引用，不能是值传递，如果拷贝构造函数入参是值传递的话，传值的过程本身要调用一次拷贝构造函数，就无限循环了。
  + 拷贝构造函数用一个对象初始化另一个对象，赋值函数将一个对象赋值给另一个对象，区别在于另一个对象此前是否存在
  + 拷贝构造函数的主要应用场景：
    + 对象作为函数的**参数**，以值传递的方式传给函数
    + 对象作为函数的**返回值**，以值的方式从函数返回
    + 使用一个对象给另一个对象**初始化**

  考虑以下情形：

  ```c++
  class Person {
  public:
      Person() {}
      Person(const Person& p) { cout << "Copy Constructor" << endl; }
      Person& operator=(const Person& p) { cout << "Assign" << endl; return *this; }
  };
  
  void f(Person p) { return; }
  
  Person f1() { Person p; return p; }
  
  int main() 
  {
      Person p;
      Person p1 = p;    // 1, Copy Constructor
      
      Person p2;
      p2 = p;           // 2, Assign
      
      f(p2);            // 3, Copy Constructor
  
      p2 = f1();        // 4, (Copy Constructor) + Assign
  
      Person p3 = f1(); // 5, (Copy Constructor) + (Copy Constructor)
  
      return 0;
  }
  ```

  + 前三个例子很好理解
  + 第四个例子理论上应该是先拷贝构造再赋值，因为有一次返回值的值传递和一次赋值操作。但是我实际运行的时候发现只有一次赋值，猜想应该是编译器做了优化，把拷贝构造函数的第二个应用场景（返回值值传递）给优化掉了，比如返回的时候就不再产生一个中间的临时对象了。
  + 第五个例子理论上应该是会调用两次拷贝构造函数，一次返回值值传递，一次用对象初始化，但是由刚才的例子可以知道编译器会将返回值值传递那一次调用优化掉。但是实际上，我运行的时候一次拷贝构造函数都没有调用，应该也是编译器做了更进一步的优化，比如将 p3 直接作为 f1() 中的 p 来初始化，这样整个过程只用调一次默认构造函数。之所以有这个猜测是因为我发现 p3 和 f1() 中的 p 地址相同，更进一步地，在 f1() 中定义很多 Person 对象，只有 p 对象的地址没有按序排列在其中，而是和 main 中的 p3 相同。

+ 内联函数：

  + 优点：减少函数调用带来的开销
  + 缺点：每一处内联函数的调用都将在编译时展开，需要更多的内存空间

  所以以下两种情况**不适合**用内联函数：

  + 相比函数调用带来的开销，函数执行的时间要长得多，没有必要声明为内联函数
  + 函数体代码很长，导致内存消耗代价过大

  内联函数和宏定义的对比：

  + 内联函数在编译时展开，嵌入代码；宏定义在预编译时展开，文本替换。
  + 内联函数是函数，会做一系列的类型检查操作

  需要注意的是：

  + 内联函数无法获取其地址

  + 内联函数不能声明为虚函数，很好理解，内联函数在编译时刻展开，而虚函数需要运行时类型确定具体调用哪一个函数，况且虚函数在编译时刻需要被重写成查虚表的形式。

    （构造函数和静态函数也不能声明为虚函数）

  另外，构造函数和析构函数可以被声明为内联函数，但是不建议这样，因为编译器在编译时往往会向构造函数和析构函数中加入很多代码（augment），所以构造/析构函数实际上要比看上去长得多。

+ 有哪几种情况只能用**构造函数初始化列表**而不能用赋值初始化？

  + const 成员
  + 引用成员

  可以理解为需要在第一时间（声明时刻）就初始化的变量要用初始化列表。

##### Last-modified date: 2020.2.24, 8 p.m.