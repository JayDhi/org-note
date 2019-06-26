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