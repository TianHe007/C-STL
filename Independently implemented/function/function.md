- C++中function可以接受函数，接收可调用对象，我们的function也应该可以这样做，而且也应该支持赋值操作，那么我们的fuction就应该可以满足下面的测试代码

```cpp
class ADD_MULT {
public :
    ADD_MULT(int x) : x(x) {}
    int operator()(int a, int b) {
        cout << "functor : ";
        return (a + b) * x;
    }
private :
    int x;
};

int main() {
    ADD_MULT add_mult(2);
    my::function<int(int, int)> t1 = func;
    my::function<int(int, int)> t2 = add_mult;
    cout << t1(3, 4) << endl;
    cout << t2(3, 4) << endl;
    t1 = t2;
    cout << t1(3, 4) << endl;
    my::function<int(int)> dfs = [&](int a) -> int {
        if (a == 1) return 1;
        return a + dfs(a - 1);
    };
    cout << dfs(100) << endl;
    return 0;   
}
```

- 知道了怎么使用，那么就可以着手开始设计了
- 首先从接口来看，我们这个工具类一定是一个变参模板类，而且使用的是偏特化版本

```cpp
template<typename T, typename ...ARGS> class function;
template<typename T, typename ...ARGS> 
class function<T(ARGS...)> {
public :
    
};
```

- 由于可以接受函数指针，可以接受可调用对象，那么此类就需要支持两种完全不同的构造方法
    - 第一个方法：接受普通的函数指针
    - 第二个方法：接受可调用对象
- 首先第一个方法就是普通的函数指针，很容易想到是下面的类型

```cpp
function(T (*func)(ARGS...));
```

- 但是第二个方法，我们传入的是一个不一定是哪个类的可调用对象，因此可以传入任意类型！
- 那么这个怎么办，是不是就需要使用模板，因此第二个构造函数是带有模板的构造函数

```cpp
template<typename CLASS_T>
function(CLASS_T obj);
```

- 此后，显然我们function类一定是重载了()运算符的，因此重载()类型

```cpp
T operator()(ARGS... args) {}
```

- 那有了这个东西，我们总的去调用一些东西吧
- 所以将目光集中到构造函数中的func普通函数指针和obj可调用对象中，也就是说不管用什么样的形式，都需要将他两存储到某个对象中去，某个成员属性中去，然后通过调用成员属性完成函数过程的调用
- **所以需要一个成员属性ptr，使得它既能存储普通函数，又能存储可调用对象**
- 那么这个ptr是什么类型呢，你有意识到什么东西吗？？？？
- 这个ptr自己肯定由一种类型，然后呢，这种类型他其实由两个具体的应用场景，而这两种具体的应用场景呢一种是应用在普通函数上，一种是应用在可调用对象身上，然后我ptr如果是一种更加抽象的类型的话，那么这个时候就能抽象化出来这么一个概念：
    - 将普通函数封装成一种类型，将可调用对象封装成为另外一种类型
    - 这两种类型同时派生自ptr的类型，那么这就形成了我们所谓的**继承**关系，也就是概念上的递进
    - **所以实现继承的时候并不是说我们实现完了父类再去实现子类，通常情况下是我们从一个大的层面先思考这个继承关系，然后构建更加抽象的父类**
- 创造出同时存储可调用对象和函数指针的基础的概念类，以及派生出来的普通函数指针类和可调用对象类

```cpp
template<typename T, typename ...ARGS>
class Base {
public :

};

template<typename T, typename ...ARGS>
class normal_function : public Base<T, ARGS...> {
public :

};

template<typename T, typename ...ARGS>
class functor : public Base<T, ARGS...> {
public :

};

```

- 此时构建出`Base`类，派生出`normal_function`类和`functor`类，因此`Base`类型的指针，要么指向`normal_function`，要么指向`functor`
- 因此在`function`中定义一个成员属性`ptr`，指向`Base`类型的指针

```cpp
Base<T, ARGS...> *ptr;
```

- 那么在()重载中，ptr要么指向函数要么指向可调用对象， 我们设计run方法，那么run方法一定就是Base类的纯虚函数

```cpp
T operator()(ARGS... args) {
        return ptr->run(args);
 }
```

- 然后完善function函数的构造函数，此时就可以给ptr赋值了
- 使得ptr指向对应类型的类的指针，要new出来

```cpp
template<typename T, typename ...ARGS> class function;
template<typename T, typename ...ARGS> 
class function<T(ARGS...)> {
public :
    function(T (*func)(ARGS...)) 
    : ptr(new normal_function<T, ARGS...>(func)) {}
    template<typename CLASS_T>
    function(CLASS_T obj)
    : ptr(new functor<T, ARGS...>(obj) {}
    T operator()(ARGS... args) {
        return ptr->run(args);
    }
private :
    Base<T, ARGS...> *ptr;
};

```

- 随后设计`normal_function`类的构造函数，使得正确传递参数

```cpp
template<typename T, typename ...ARGS>
class normal_function : public Base<T, ARGS...> {
public :
    normal_function(T (*ptr)(ARGS...)) : func(ptr) {}
    T run(ARGS... args) override {
        return func(args...);
    }
private :
    T (*func)(ARGS...);
};
```

- 上述代码中，接受函数指针类型，因此使用函数指针类型func进行参数接受，然后就可以直接调用`func(args…)`，但是这样的参数传递有问题吗？？
- 还记得当时一个代码传递右值然后右值变成了左值的那个场景吗？所以此时这样传递就会使得所有的右值直接变成左值
- 因此需要使用forward方法，<ARGS>表示将args中的参数按照原始类型继续向下传递，后面的三个点表示我当前要解析的是变参列表：正确传递代码如下：

```cpp
template<typename T, typename ...ARGS>
class normal_function : public Base<T, ARGS...> {
public :
    normal_function(T (*ptr)(ARGS...)) : func(ptr) {}
    T run(ARGS... args) override {
        return func(forward<ARGS>(args)...);
    }
private :
    T (*func)(ARGS...);
};
```

- 因此`function`中`run`方法的参数传递也要修改为类似的方法

```cpp
 T operator()(ARGS... args) {
        return ptr->run(forward<ARGS>(args)...);
    }
```

- 到这里是不是可以自主实现传入的是可调用对象的场景？
- 在实现的时候，是否会遇到一个问题，传入的CLASS_T类型我怎么存呢？
- 正解是此类模板需要多添加一个参数，然后将CLASS_T传进来，就可以使用CLASS_T进行成员属性的定义：

```cpp
template<typename CLASS_T, typename T, typename ...ARGS>
class functor : public Base<T, ARGS...> {
public :
    functor(CLASS_T obj) : obj(obj) {}
    T run(ARGS... args) override {
        return obj(forward<ARGS>(args)...);
    }
private :
    CLASS_T obj;
};

class function::
		function(CLASS_T obj)
    : ptr(new functor<CLASS_T, T, ARGS...>(obj)) {}
```

- 此时已经实现了大半，还剩下一个赋值运算符，由于是赋值的是指针类型，因此要进行的是深拷贝

```cpp
 function &operator=(const function<T(ARGS...)> &func) {}
```

- 首先需要删除当前类中的`ptr`，由于是`new`出来的，因此需要`delete this→ptr`
- 然后我们应该获取`func.ptr`的一份拷贝赋值给`this→ptr` 而不是把他的地址赋值给他，因为使用浅拷贝就会出现问题，此时需要深拷贝
- 那么我应该如何拷贝一份它原本的对象呢：
    - `ptr`是`Base`类型的，但是我`Base`类型是一个虚拟类，实际对象的类型要不然就是`normal_function`，要么就是`functor`类型
    - 所以应该如何获取一份实际对象的拷贝呢？
    - 我们可以通过虚函数来实现，虚函数跟着对象走！

```cpp
function &operator=(const function<T(ARGS...)> &func) {
        delete this->ptr;
        this->ptr = func.ptr->getCopy();
        return *this;
    }
```

- 在Base抽象类中设计纯虚函数getCopy方法，然后让他所有的子类都实现这个方法

```cpp
class Base {
public :
    virtual T run(ARGS...) = 0;
    virtual Base<T, ARGS...> *getCopy() = 0;
};

class normal_function : 
Base<T, ARGS...> *getCopy() override {
        return new normal_function(*this);
}
    
    
class functor :
Base<T, ARGS...> *getCopy() override {
        return new functor(*this);
}
```

- 至此就已经实现完成了拷贝构造
- 最后我们还需要设计析构函数，注意将Base类的析构函数设计为虚函数

```cpp
class Base {
public :
    virtual T run(ARGS...) = 0;
    virtual Base<T, ARGS...> *getCopy() = 0;
    virtual ~Base() {}
};

class function<T(ARGS...)> {
public :
    ~function() {
        delete this->ptr;
    }
};
```

- **至此我们已经实现完成了function类模板的设计，总代码：**

```cpp
#include <iostream>
#include <functional>
using namespace std;

namespace my {
template<typename T, typename ...ARGS> 
class Base {
public :
    virtual T run(ARGS...) = 0;
    virtual Base<T, ARGS...> *getCopy() = 0;
    virtual ~Base() {}
};

template <typename T, typename ...ARGS>
class normal_function : public Base<T, ARGS...> {
public :
    normal_function(T (*ptr)(ARGS...)) : func(ptr) {}
    T run(ARGS ...args) override {
        return func(forward<ARGS>(args)...);
    }
    Base<T, ARGS...> *getCopy() override {
        return new normal_function(*this);
    }
private :
    T (*func)(ARGS...);
};

template <typename CLASS_T, typename T, typename ...ARGS>
class functor : public Base<T, ARGS...> {
public :
    functor(CLASS_T obj) : obj(obj) {}
    T run(ARGS ...args) override {
        return obj(forward<ARGS>(args)...);
    }
    Base<T, ARGS...> *getCopy() override {
        return new functor(*this);
    }
private :
    CLASS_T obj;
};

template <typename T, typename ...ARGS> class function;
template <typename T, typename ...ARGS>
class function<T(ARGS...)> {
public :
    function(T(*ptr)(ARGS...)) : ptr(new normal_function<T, ARGS...>(ptr)) {}
    template<typename CLASS_T>
    function(CLASS_T obj) : ptr(new functor<CLASS_T, T, ARGS...>(obj)) {}
    T operator()(ARGS... args) { 
        return ptr->run(forward<ARGS>(args)...);
    }
    function &operator=(const function<T(ARGS...)> &func) {
        delete this->ptr;
        this->ptr = func.ptr->getCopy();
        return *this;
    }
    ~function() {
        delete this->ptr;
    }
private :
    Base<T, ARGS...> *ptr;
};

}

int func(int a, int b) {
    cout << "normal function : ";
    return a + b;
}

class ADD_MULT {
public :
    ADD_MULT(int x) : x(x) {}
    int operator()(int a, int b) {
        cout << "functor : ";
        return (a + b) * x;
    }
private :
    int x;
};

int main() {
    ADD_MULT add_mult(2);
    my::function<int(int, int)> t1 = func;
    my::function<int(int, int)> t2 = add_mult;
    cout << t1(3, 4) << endl;
    cout << t2(3, 4) << endl;
    t1 = t2;
    cout << t1(3, 4) << endl;
    my::function<int(int, int)> t3 = [&](int a, int b) {
        return a + b;
    };
    my::function<int(int)> dfs = [&](int a) -> int {
        if (a == 1) return 1;
        return a + dfs(a - 1);
    };
    cout << dfs(100) << endl;
    return 0;   
}
```
