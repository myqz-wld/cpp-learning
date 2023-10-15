## 右值引用和move函数
C++11增加了一个新的非常数引用类别——右值引用（right value reference），代码中用 $ T\&\& $ 表示。  
右值引用的引入是为了减少不必要的数据拷贝开销。考虑以下场景：
```Cpp
T a;    // 调用构造方法初始化变量a
T b(a); // 调用拷贝构造方法初始化变量b
```

在C++11以前，对于变量b的初始化只能通过拷贝构造函数，不可避免地要经历一次内存分配。假设我们在变量b的初始化之后就不再需要变量a，能否让变量b直接使用变量a所分配到的内存呢？  
在C++11的新标准下，答案是可以，我们可以通过move函数将变量a转换为右值，然后通过移动构造函数初始化变量b。  
```Cpp
class T {
public:
    // 移动构造方法
    T(T&& t) {
        // ...
    }
};
int main() {
    T a;
    // move将变量a转换为右值，然后调用移动构造方法
    T b(std::move(a));
}
```

move函数源码（涉及到类型推导和引用折叠）：
```Cpp
template <class _Tp>
_LIBCPP_NODISCARD_EXT inline _LIBCPP_INLINE_VISIBILITY _LIBCPP_CONSTEXPR typename remove_reference<_Tp>::type&&
move(_Tp&& __t) _NOEXCEPT {
  // 解除引用
  typedef _LIBCPP_NODEBUG typename remove_reference<_Tp>::type _Up;
  // 实质是通过static_cast<T&&>来完成类型转换
  return static_cast<_Up&&>(__t);
}
```

remove_reference源码（涉及到模版特化）：
```Cpp
template <class _Tp> 
struct _LIBCPP_TEMPLATE_VIS remove_reference {
    typedef _LIBCPP_NODEBUG _Tp type;
};
template <class _Tp> 
struct _LIBCPP_TEMPLATE_VIS remove_reference<_Tp&> {
    typedef _LIBCPP_NODEBUG _Tp type;
};
template <class _Tp> 
struct _LIBCPP_TEMPLATE_VIS remove_reference<_Tp&&> {
    typedef _LIBCPP_NODEBUG _Tp type;
};
```

## constexpr关键词
const关键词严格来说代表的是只读，其修饰的变量的内容还是在运行期才能够确定的，甚至能够过某些tricky的手段在运行期进行修改。而constexpr关键词才代表真正的常量，在编译期就能够确定。  
constexpr除了用来修饰常数变量，也能够用来修饰常数函数，比如：
```Cpp
constexpr int get_one() {
    return 1;
}

constexpr int add(int a, int b) {
    return a * b;
}

constexpr int val0 = get_one();
constexpr int val1 = add(2, 5);
```