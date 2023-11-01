# The Cherno C++ Series

[*Markdown语法*](https://markdown.com.cn/basic-syntax/headings.html "Clik it")  
[*Markdown扩展语法*](https://markdown.com.cn/extended-syntax/fenced-code-blocks.html "bruh-")

---

![rock](markdown_pics/shiprock.c3b9a023.jpg "Let's Go!")

## reference "&"

**实际上是一种语法糖，为变量创建一个别名**

* > "语法糖"（Syntactic Sugar）是一种编程语言特性或语法结构，它并不提供新的功能，但可以使代码更容易阅读、编写和理解。

---

## difference bettween struct & class

  **唯一**区别就在于，class中声明的对象默认**private**

* > 所以可以在struct中创建函数, 甚至一个class可以继承至一个struct

  而struct中默认**public**,它在C++中的存在是为了向下与C兼容

---

## Static

1. ### [**class和struct外部的static**](global_is_terrible "可以类比class中的private global is terrible")

        **static**修饰的变量只会在这个翻译单元内部链接
        * > 1.**extern**关键字修饰的变量表示此变量在翻译单元外部，但不可链接到外部的static变量<br>2.[如果在头文件中创建了一个static变量，并且在两个不同的源文件中include了这个头文件，实际只是在两个源文件中分别创建了一个静态变量](useless "#include的本质就是复制粘贴")

2. ### 内部的static

     * **如何访问静态**
         创建了一个此类的静态变量/方法：

         ```C++
         # 创建一个类
         class Entity()
         {
             static int x, y;
             static void print()
             {
                 std::cout << x << ',' << y << std::endl;
             }
         }
         # 实例化一个对象
         Entity e1;
         👇实际上是不规范的访问静态成员方法
         e1.x = 1
         应改为👇:
         Entity::x = 1;
         同理访问静态类方法：
         Entity::print();
         实际上表示这个静态对象是属于这个类的，并不从属于当前实例。
         （当前方法并不将当前实例作为 参数 输入。🤔见下方）
         ```

     * **静态方法**⚠  
       实际上类的**非静态**成员方法都会将类的实例作为一个**隐藏参数**传入，而类的**静态**方法不会得到这个隐藏参数  
       *以下陈述一个非静态方法的事实*

       ```C++
       /* 
       本质上非静态类方法的编译就像是
       在类外创建了一个类的方法并将当前实例作为参数传入此方法👇
       */
       class Entity()
       {
           static int x, y;
           void print()
           {
               std::cout << x << ',' << y << std::endl;
           }
       }
       /*
       将上述类中的print方法提出,并将Entity实例作为参数传入
       （注意此时的print为非静态方法）
       这就是非静态方法在编译时的真实样子👇
       */
       void print(Entity e)
       {
           std::cout << e.x << ',' << e.y << std::endl;
       }
       ```

       ☠静态方法不能访问非静态成员的原因就是☠：
       😂👉**没有将当前类实例作为参数传入，不知道类中的非静态成员是什么**

3. ### 单例类

      *只能具有一个实例的类*
       > **Q**：如何才能只具有一个实例?🤔*
       **A**：将构造函数**私有化**：不能通过一般方法初始化类的实例，而通过**提供公有的静态方法**： 在类中提供一个公有的静态方法，该方法负责在类的内部创建并返回类的实例,（**为何能在内部创建**?因为类内部可访问private成员函数）这种静态方法通常被称为工厂方法。类的外部代码可以调用这个静态方法来获取对象实例。

      eg:

      ```C++
      class Singleton
      {
       public:
            // 创建公有静态方法,实际上是返回一个静态的reference
           static Singleton& GetInstance()
           {
               static Singleton instance;
               return instance;
           }
           
           // 私有构造函数
       private:
           Singleton()
           {
               // 构造函数的实现
           }
      };

      实例化👇(使用调用类静态方法的形式)
      Singleton Singleton::GetInstance();

      ------------------------------------还有一种运用局部静态变量实现的方法
      ---------------------------------👇这种实现方式称为：Meyers’ Singleton
       class Singleton 
       {
       public:
           static Singleton& getInstance() 
           { /*这里因为返回的是引用，所以
           必须添加static来保证其不会被创建在栈上*/
               static Singleton instance;
               return instance;
           }
           
           // 其他成员函数和数据成员
       private:
           Singleton() {
               // 私有构造函数，防止外部实例化
           }
           
           Singleton(const Singleton&) = delete;
           Singleton& operator=(const Singleton&) = delete;
       };
       int main() {
           Singleton& s1 = Singleton::getInstance();
           Singleton& s2 = Singleton::getInstance();
           
           // s1和s2都是同一个实例
           assert(&s1 == &s2);
           
           return 0;
       }
       /*在第一次调用时会创建一个static instance
       后面再调用GetInstance()方法时只会return这个static instance*/

      ```

---

## enum

* 他只是一种命名方法
* 枚举数就是一个整数,只能是整数🤯
* 不同于枚举类

```C++
#include <iostream>

calss eg()
{
public:
    enum Example : char  /* 
    -Example并未创建一个命名空间，enum内部的元素可根据类空间进行访问：见下方main函数
    -这里如不进行显示控制内部元素类型，默认
        是int，绝不能声明为float，因为enum只能是整数
    -这里的“:”用于声明
        元素类型的方式也是enum独有的 
    */
    {
        A = 0, B, C  // 默认从0开始，但最好显示的声明
    };
}

int main()
{
    char a = eg::A;  // 类空间访问

    eg eg1;  // 实例化
    char b = eg1.A;  // 实例对象的成员变量
}
```

---

## 构造函数

还是拿例子来说  
以下是一个**不含构造函数的类**：

```C++
#include <iostream>

class Example
{
public:
    float X, Y;  // 两个成员变量(⚠这里并未赋初值)

    void Print()
    {
        std::cout << X << ',' << Y << std::endl;  // 一个成员函数
    }
};

int main()
{
    Example e1;
    e1.Print();
    return 0;
}
//执行结果👇
>> 2.24208e-044,0
```

在未设定构造函数的情况下，成员变量的值未经过初始化，因此打印出了未经初始化的内存之前存在的值。  
**🤡这就是打印未初始化内存值的方法！🤡**  
再看👇

```C++
// 如果使用这种方式进行调用
int mian()
{
    Example e1;
    std::cout << e1.X << std::endl;
    // 某些情况下这样编译会出错(报错使用未初始化内存?)，但我没有🤔，可能创建了默认构造函数
}

by the way构造函数的赋值方法之一：
class Example
{
public:
    int A, B;
    Example(int a, int b) : A(a), B(b) {}
};
```

**🤡1如果你不想让编译器给你创建一个默认的构造函数 & 2不允许你的类实例化🤡**：

```C++
// 1
class Example
{
public:
    Example() = delete; // 这样你就永远无法实例化这个类
    void func()
    {
        // 公有成员函数
    }
}
但你可以调用类内的public成员函数(但private函数是无法访问的)：
int main()
{
    Example::func();
}

// 2
class Example
{
private:
    Example()
    {
        // 私有构造函数
    } 
    /* 这样也无法创建实例,无法访问到构造函数,但可通过实现一个公有
    函数访问此private构造函数并在类内部创建实例。
    ?如果共有函数返回静态实例的引用，那这就是一种控制此类可用实例数量的方法?
    */
}
```

## 析构函数

析构函数在作用域结束时会将**栈对象**删除 **`or`** 在调用`delete`的时候将分配在**堆内存**中的对象删除

```C++
#include <iostream>
如果尝试手动调用析构函数，但是此对象分配在栈空间时，当作用域结束，会再次调用析构函数，如下代码👇
此时如果析构函数当中进行了某个内存的释放，可能会报错。
(总之慎用显式调用析构函数，最好别用)
class Example
{
public:
    Example()
    {
        // initializing...
    }
    ~Example()
    {
        // destroying...
    }
};

void func()
{
    Example e1;
}

int main()
{
    func();
    // func()函数作用域结束，再次调用析构函数
    std::cin.get();
    return 0;
}
```

## 类的继承

* 为了代码复用，完成**父类👉派生类**这样一种高效的复用形式
* 为了给基类扩展功

```C++
#include <iostream>

class Entity
{
public:
    int X, Y;
private:
    int Z;
};  
    // ⚠：类内的静态成员只能由静态函数访问，在main函数中无法赋值操作(main中是动态的？static无法修饰main)
    // 但是可以在创建类后立即在类外对静态成员赋值：假设成员Z为static int Z
    // 赋值操作：int Entity::Z = 1;

class Example : public Entity
{
public:
    int x, y;
    void Print()
    {
        std::cout << x << ' ' << y;
    }
}

int main()
{
    Entity E;
    Example e;
    std::cout << sizeof(E) << ' ' << sizeof(e) << std::endl;
}
>>  12 20
```

## 虚函数

既然类继承了，那么在子类中对父类的方法进行 👉**重写**吧😵  
直接上例子：

* 子类中实现一个和基类名字相同的函数

  ```C++
  #include <iostream>
  #include <string>

  class Entity
  {
  public:
      void Print_name()
      {
          std::cout << name << std::endl;
      }
  private:
      std::string name = "Entity";
  };

  class Example : public Entity
  {
  public:
      void Print_name()
      {
          std::cout << name << std::endl;
      }
  private:
      const char* name = "example";
  };
  
  // 此处由于传入的参数类型是Entity， 所以默认会在Entity中去找Print_name函数
  // 而在基类中未声明virtual的情况下，不会为这个函数生成 👉虚表👈(一种实现多态的机制,见下文)
  // 那么在最终打印时，实际上是直接调用了Entity类内部的函数
  void Func(Entity* e)
  {
      e->Print_name();
  }

  int main()
  {
      Entity* E = new Entity();
      Example* e = new Example();
      Func(E);
      Func(e);  // e是一个Example类对象也是一个Entity类对象，所以这样传入没问题😀
      delete E, e;
  }
  >>  Entity
      Entity
  ```

  >**虚表(Vtable)**<sup>👆👇</sup>  
  >用处💡  
  1.当一个类包含虚拟函数时，编译器会在该类的对象中插入一个指向虚表的指针。这个指针指向了包含虚拟函数地址的虚表。当你调用一个虚拟函数时，实际上是通过虚表来查找正确的函数并执行它，而不是直接调用类的函数  
  2.虚表会存**储子类重载（覆盖）的函数指针**。在继承层次结构中，如果子类重载了基类的虚拟函数，那么子类的虚拟函数实现将被放入虚表中，取代基类的实现。这使得在运行时，可以根据对象的实际类型来调用正确的函数，实现多态性。<br>
  >
  >缺点(空间&时间)⏰💥实际上这些影响并不大  
  1.消耗额外内存存储Vtable。  
  2.调用子类重载函数时会遍历虚表找到正确的重载函数，而**不是直接调用**。  

  **开始进行重载：**  
  注：当一个类包含虚拟函数时，编译器会在该类的对象中插入一个指向虚表的指针。这个指针指向了包含虚拟函数地址的虚表。

  ```C++
  #include <iostream>
  #include <string>

  class Entity
  {
  public:
      virtual void Print_name()  // 插入virtual关键字表明可重载函数并创建虚表(Vtable)
      {
          std::cout << name << std::endl;
      }
  private:
      std::string name = "Entity";
  };

  class Example : public Entity
  {
  public:
      void Print_name() override  // 插入override关键字(C++11特性)表明子类的重载函数，并且这个关键字可以帮助提示错误
      {
          std::cout << name << std::endl;
      }
  private:
      const char* name = "example";
  };

  void Func(Entity* e)
  {
      e->Print_name();
  }

  int main()
  {
      Entity* E = new Entity();
      Example* e = new Example();
      Func(E);
      Func(e);
      delete E, e;
  }
  >> Entity
     example
  ```

## 纯 虚函数

1.定义：在基类**定义**一个没有实现的函数<br>🤐🔫：强迫子类实该函数  
2.有什么用？🤔：这种实现被称为**接口(interface)**,类中只包含未实现的方法作为你继承的模板(所以这个类实际上是无法实例化的，但如果你实现了这些函数，或者继承自一个实现了基类所有函数的子类，这个类就可以被实例化)  
你需要这个类的实例都具有这些方法(接口)，有点类似Dataset类中需要实现的len, getitem等方法。

```C++
#include <iostream>
#include <string>

class Pure_virtual
{
public:
    virtual void print_name() = 0;  // 💡纯 虚函数的定义：virtual + ‘= 0’
};

class child_virtual : public Pure_virtual  // 若此类中没有实现继承的基类的纯虚函数，便无法实例化
{
public:
    void print_name() override  // 实现纯虚函数
    {
        print();
    }
    void print()
    {
        std::cout << name << std::endl;
    }
private:
    const char* name = "name";
};

int main()
{
    child_virtual cv = new child_virtual();  // 若未实现纯虚函数，这里会报错
    cv->print();
    return 0;
}
>> name
```

## Cpp中的可见性

> in brief: **private protected public**  
> 如`class` `struct`中默认的成员可见性(private, public) <sup>*（**友元类(friend)**可以访问到此类内部的private成员）*</sup>

1. 💡private修饰的成员: **类内部**以及**友元**可访问
2. 💡protected修饰的成员：**类内部**以及继承至此类的**子类**
3. public：就是public
