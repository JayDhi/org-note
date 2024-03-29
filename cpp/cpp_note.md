#  `C++`语法

##  一、`using`  

###  1. 使用命名空间  

###  2. 使用基类成员
  
```cpp
class Base{
public:
    Base(): value(55){} //构造函数&初始化列表
    virtual ~Base(){}
    void test(){
        using std::cout;
        using std::endl;
        cout << "Base::test()" << endl;
    }
//proctected:
private:
    int value;
}

class Derived: 
    private Base{
using Base::value
public:
    void test_(){
        using std::cout;
        using std::endl;
        cout << "value is " << value << end;
    }
}
```
###  3. 别名
  
* 例如`using value_type = _Ty`, 就可以用`_Ty value`代替`value_type value`
* 跟`typedef`相比:
  * `typedef`:
    ```cpp
    typedef std::unique_ptr<std::unordered_map<std::string, std::string>> UPtrMapSS;
    ```
  * `using`:
    ```cpp
    using UPtrMapSS = std::unique_ptr<std::unordered_map<std::string, std::string>>;
    ```
  * 更多例子:
  
    *  
      ```cpp
      typedef void (*FP) (int, const std::string&);
  
      using FP = void (*) (int, const std::string&);
      ```
    * 
      ```cpp
      typedef std::string (Foo::* fooMemFnPtr) (const std::string&);
  
      using fooMemFnPtr = std::string (Foo::*) (const std::string&);
      ```
    * 
      ```cpp
      template <typename T>
      using Vec = MyVector<T, MyAlloc<T>>;
      // usage
      Vec<int> vec;
  
      // typedef与template的设计思路不相容, 因此应当使用using
      template <typename T>
      typedef MyVector<T, MyAlloc<T>> Vec;
      // usage
      Vec<int> vec;
      // error: a typedef cannot be a template
      ```


## 二、`explicit`
### 避免单参构造函数导致的隐式转换
  * 单参构造函数导致的隐式转换
    * 如果有未被标注`explicit`的单参构造函数，那么在编译时，编译器会根据单参构造函数，会对赋给该类类型变量、符合单参构造函数参数类型的数据进行隐式转换，将该数据转换为一个类实例
    * 这种特性主要是在未对`=`重载的情况下利用赋值语句对对象进行简单的赋值
    * 例如:
        ```cpp
        class A{
        public:
            A(int i = 5){
                _i = i;
            }
        private:
            int _i;
        }
        ```
        <!--不要让一个对象使用任何类型都能构造，而是应该严格限制，例如接受到某种类型的参数的时候抛出异常，以使上游代码更加规范-->
        如果要对一个`A`对象进行赋值，需要重载`=`运算符以实现其他类型到`A`类型的转换<!--举例说明其他类型到A类型的转换，如char->A, *p->A-->但由于上述的隐式转换存在，使得以下赋值成为合法语句
        ```cpp
        A a = 10;
        ```
        因为`A`的构造函数以`int`类型的数据为变量，所以编译器进行了以下隐式转换
        ```cpp
        A temp = A(10);
        A a =temp;
        ```
        将`int`换成其他类型<!--float?是否会隐式转换，如A a = 1.11-->就会出错，因为`A`的构造函数不接受其他类型的参数
        * 基本数据类型如`int`，`double`，`char`存在隐式转换链，对此，使用`delete`可避免这种隐式的转换，例如
            ```cpp
            class A{
            public:
                A(int i = 5){
                    _i = i;
                }
                A(char c) = delete;
                /*
                相当于在试图调用以char类型变量为参数的构造函数时调用析构函数，以表明这种类型的构造是无效的
                */
            private:
                int _i;
            }
            ```

这种转换在赋值的时候虽然会在未实现`=`重载的情况下提供赋值的便利，但是有可能导致潜在的问题
  * 例如
    ```cpp
    class CppString{
    public:
        char * _pstr;
        int _size;
        CppString(int size){
            _size = size;
            _pstr = malloc(size+1);
            memset(_pstr, 0, size+1);
        }
        CppString(const char * p){
            int size = strlen(p);
            _pstr = malloc(size+1);
            strcpy(_pstr, p);
            _size = strlen(_pstr);
        }
        ~CppString(){
            delete[] _pstr;
            //释放指针指向的内存
        }
    }
    ```
    如果有
    ```cpp
    CppString s1 = 10;
    ```
    则相当于
    ```cpp
    CppString tmp = CppString(10);
    CppString s1 = tmp;
    ```
    上述转换是可以接受的，但是假如在试图以字符`a`构造`CppString`对象的时候，因为输入错误，将
    ```cpp
    CppString s2 = "a";
    ```
    输入成
    ```cpp
    CppString s2 = 'a';
    ```
    则相当于
    ```cpp
    CppString tmp = CppString(97);
    CppString s2 = tmp;
    ```
    本来`CppString`是没有接受字符类型的构造函数的，但是因为`a`可以被转换成`int`，并且存在`int -> CppString`的隐式转换，本应出错的语句被错误地执行，字符类型错误地被接受，在编译时期应该暴露的错误被隐藏，这会产生潜在的问题

## 三、函数模板
### 1. `std::transform`
有数组`A`，现需要将`A`内的每个元素`+5`，将结果放到数组`B`中
* 写法一
  * 假设`A`，`B`的类型是`int`，那么
    ```cpp
    // 伪码
    int * add_int_5(int * A, int * B){
        for (i : len(A)){
            B[i] = A[i]+5;
        }
    }
    ```
  * 假设`A`，`B`的类型是`float`，那么
    ```cpp
    // 伪码
    float * add_float_5(float * A, float * B){
        for (i : len(A)){
            B[i] = A[i]+5;
        }
    }
    ```
* 写法二
  * 假设`A`，`B`的类型是`int`，那么
    ```cpp
    // 伪码
    int * add_int_value(int * A, int * B, int value){
        for (i : len(A)){
            B[i] = A[i]+value;
        }
    }
    ```
  * 假设`A`，`B`的类型是`float`，那么
    ```cpp
    // 伪码
    float * add_float_value(float * A, float * B, float value){
        for (i : len(A)){
            B[i] = A[i]+value;
        }
    }
    ```
> 写法一、二是弹性最差的两种写法：
>  * 写法一将类型、改变量绑定函数上：针对不同类型的`A`，`B`以及不同的改变量都要实现相应的函数，如`add_int_1`，`add_int_2`，`add_float_1`，`add_float_2`
>  * 写法二将类型绑定在函数上，虽然与写法一相比，解耦了改变量，改变量可以以参数的形式传入函数，但是对于不同的参数类型，仍然需要实现对应的函数，如`add_int_value`，`add_float_value`，`add_double_value`
* 写法三
  * 假设`A`，`B`的类型是`int`，那么 
    ```cpp
    vector<int> A, B;
    std::transform(A.begin(),
                   A.end(),
                   B.begin(),
                   [](int a){ a *= 5; });
    ```
  * 假设`A`，`B`的类型是`float`，那么 
    ```cpp
    vector<float> A, B;
    std::transform(A.begin(),
                   A.end(),
                   B.begin(),
                   [](float a){ a *= 5; });
    ```
* 写法四
  * 先实现一个函数模板
    ```cpp
    template <typename T, T value>
    T addValue(const T& x){
        return x + value;
    }
    ```
  * 假设`A`，`B`的类型是`int`，那么
    ```cpp
    std::transform(A.begin(),
                   A.end(),
                   B.begin(),
                   addValue(std::vector<int>::value_type, 10));
    ```
  * 假设`A`，`B`的类型是`float`，那么
    ```cpp
    std::transform(A.begin(),
                   A.end(),
                   B.begin(),
                   addValue(std::vector<float>::value_type, 10));
    ```
    * 关于`std::vector<int>::value_type`
      * 在`std::vector`中的定义：
        ```cpp
        typedef T value_type;
        ```
      * 相当于一个新的类型声明[[`typedef`关键字]]()
        ```cpp
        template <typename T>
        class A{
        public:
            typedef T type_1;
            typedef struct{
                int a;
                int b;
            } type_2;
            typedef double type_3;
        }
        typedef A class_a;
        ```
        其中`type_1`，`type_2`，`type_3`是`class A`中的类型，相当于嵌套类
    * 关于`addValue<int, 5>`的强制类型转换
      * [参考](https://blog.csdn.net/lanchunhui/article/details/49634077)
      * [cpp_core_issue 115](http://www.open-std.org/jtc1/sc22/wg21/docs/cwg_active.html)
      * `addValue<int, 5>`是函数模板的一个实例化版本，而 __函数模板的实例通常被视为命名一组重载函数的集合(即便集合中只有一个元素)__ 。在一些较老的`C++`版本中，重函数的集合不能用于模板参数(即`addValue<int, 5`>不能直接传入`std::transform()`)，需要进行强制转换
        ```cpp
        std::transform(ivec.begin(), ivec.end(), dst.begin(),
                       (int(*)(const int&))addValue<int, 5>);
        ```
      * 但是在如下编译环境下，不进行上述强制类型转换也能成功编译
        ```bash
        Configured with: --prefix=/Applications/Xcode.app/Contents/Developer/usr --with-gxx-include-dir=/Applications/Xcode.app/Contents/Developer/Platforms/MacOSX.platform/Developer/SDKs/MacOSX10.14.sdk/usr/include/c++/4.2.1
        Apple LLVM version 10.0.1 (clang-1001.0.46.4)
        Target: x86_64-apple-darwin18.6.0
        Thread model: posix
        InstalledDir: /Applications/Xcode.app/Contents/Developer/Toolchains/XcodeDefault.xctoolchain/usr/bin
        ```
      * 可能与下述内容相关
        - [ ] 模板的类型推断
        - [ ] 指向模板函数的指针
        - [ ] 函数重载与函数指针

## 四、常量指针与指针常量
![const&ptr](https://jaydhipic.oss-cn-beijing.aliyuncs.com/2019-06/30/const%26ptr.jpg)