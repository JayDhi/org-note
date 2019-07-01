# `C++`模板编程
## 函数模板
### 模板参数
* 不能指定 __缺省的模板实参__
* 例如以下`实参类型不同的max()模板`:
  ```cpp
  template <typename T1, typename T2>
  inline T1 max(T1 const& a, T2 const& b){
      return a < b ? b : a;
  }
  ```
  * 该模板的主要问题就是：必须声明返回类型，如果使用的是其中的一个参数类型，那么另一个参数的实参就可能要转型为返回类型，而不会在意调用者的意图
  * 比如`max(int, double)`和`max(double, int)`的返回类型就完全不一样
  * 解决方法：
    * 定义新的模板参数作为返回类型参数
        ```cpp
        template <typename T1, typename T2, typename RT>
        inline RT max(T1 const& a, T2 const& b){
            return a < b ? b : a;
        }
        ```
        调用方法
        ```cpp
        max<int, double, double>(4, 4.2)
        ```
        但因为通常情况下只需指定 __最后一个不能被隐式推演的模板实参前__ 的 __所有实参类型__，在本例中`T1`，`T2`是调用参数，在模板被调用时能被自动推演得出，而表示返回类型的参数`RT`则需要显示指定，所以`RT`及其前面的参数必须显式指出，而`T1`，`T2`这两个参数可以不必显式指出，所以调换`RT`与`T1`，`T2`的位置可以减少调用时所需指定的参数
        ```cpp
        template <typename RT, typename T1, typename T2>
        inline RT max(T1 const& a, T2 const& b){
            return a < b ? b : a;
        }
        ```
        调用方法
        ```cpp
        max<double>(4, 4.2)
        ```

### 特化、偏特化、全特化
* 全特化
    * 指定全部模板参数类型
        ```cpp
        template <> Class A<double, int>{}
        ```
    * 特点: 
      * 模板参数列表: 置空
      * 调用参数列表: 显式指定类型

* 偏特化
  * 指定部分模板参数类型
    ```cpp
    template <typename T> Class A<T, int>{}
    ```
  * 进一步限制模板参数(引用、指针)
    ```cpp
    template <typename T1, typename T2> Class A<T1&, T2&>{}
    template <typename T1, typename T2> Class A<T1*, T2*>{}
    ```
  * 特点: 
    * 模板参数列表: 保留范型参数
    * 调用参数列表: (限制型)范型参数+特化类型
```cpp
template <typename T> Class A{}
template <> Class A<int>{}
template <typename T> Class A<const T *>{}
template <typename T> Class A<T *>{}
template <typename T> Class A<T, int>{}
```