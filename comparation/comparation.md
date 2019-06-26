##  动态类型
  
###  一、`python`中的动态类型
  
####  1. 获取类型信息
  
```python
class A(object):
    pass

a = A()
type(A)
# <class 'type'>
# 每个class是一个type实例
type(a)
# <class '__main__.A>
```
####  2. 动态地生成类

```python
class Animal(object):
    def run(self):
        print("run")

# 使用与类无关的属性的类方法
@staticmethod
def eat_one_apple(apples):
    apples -= 1

# 需要使用类属性的类方法
@classmethod
def introduce(cls):
    print(cls.__name__)

# 只能通过实例调用
def self_introduce(self):
    if hasattr(self, 'name'):
        print(self.name)
    else:
        print("Anonymous")
def set_name(self, name)
    self.name = name

Male = type("Male", 
              (Animal,),
              {"eat_one_apple": eat_one_apple,
               "introduce": introduce,
               "set_name": set_name,
               "self_introduce": self_introduce})
Female = type("Female", 
              (Animal,),
              {"eat_one_apple": eat_one_apple,
               "introduce": introduce,
               "set_name": set_name,
               "self_introduce": self_introduce})
apples = 5

Female.eat_one_apple(apples)
Female.introduce()

Male.eat_one_apple(apples)
Male.introduce()

marry = Female()
marry.set_name("marry")
marry.self_introduce()
marry.eat_one_apple(apples)

anonymous = Male()
anonymous.self_introduce()
anonymous.eat_one_apple(apples)

john = Male()
john.set_name("john")
john.self_introduce()
john.eat_one_apple(apples)
```
####  3. 为类添加属性

```python
class Person(obeject):
    pass

name_someone(person, name):
    if not hasattr(person, 'name'):
        setattr(person, 'name', name)
        print(person.name)
    else:
        print("Already named: "+person.name)
person = Person()
name_someone(person, 'james')
```  
###  二、`C++`中的~~动态~~组合类型
  
####  1. 获取类型信息
  
---
##  模块
  
###  一、局部引用与`using`、`import`、`#include`

#### 1. 在函数内使用`using`以局部引用

在函数内使用`using`能够起到局部引用的作用
```cpp
#include <iostream>
  
void say_hello(){
    using std::cout;
    using std::endl;
    // 或者直接using namespace std;
    cout << "Hello fellas" << endl;
}
  
int main(){
    say_hello();
    std::cout << "In main()" << std::endl;
    return 0;
}
```
#### 2. `import`

在函数内使用`import`能够起到局部引用的作用
```python
# Hello.py
def hello():
    print("Hello there")
```
```python
# A.py
def say_hello():
    from Hello import hello
    hello()

if __name__ == '__main__':
    say_hello()
    hello()
```
#### 3. `#include`

在函数内使用`#include`不能起到局部引用的作用，反而会造成各种各样的错误。例如
```cpp
// A.h
struct Node{
    // elements;
};
void helloWorld(){
    // do something;
}
```
```cpp
// B.cpp
void test(){
    #include "A.h"
    Node n; // errorp
    helloWorld(); // correct
}
```
因为`#include`会将包含的文件的文本在此处展开, 因此`B.cpp`在编译时等效于
```cpp
void test(){
    struct Node{
        // elements;
    };
    void helloWorld(){
        // do something;
    }
    Node n; // error
    helloWorld(); // correct
}
```
出现了函数嵌套定义, 必然会报错
  