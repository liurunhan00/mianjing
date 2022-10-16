## **trait**

### always_false

- **[c++ 17 inline constexpr](https://stackoverflow.com/questions/49913011/are-all-constexpr-variable-implicitly-inline)**
- **[static_assert(false)](https://stackoverflow.com/questions/14637356/static-assert-fails-compilation-even-though-template-function-is-called-nowhere)**

```c++
template <typename...>
inline constexpr bool always_false = false;

// use-case1
template <typename T>
void foo(T value) {
  if constexpr (std::is_integral_v<T>) foo_integral(value);
  else if constexpr (std::is_same_v<T, std::string>) foo_string(value);
  else static_assert(always_false<T>, "Unsupported type");
}

// use-case2
template <typename T>
struct Foo {
  static_assert(always_false<T>, "Unsupported type");
};
template <>
struct Foo<int> {};

Foo<int> a;         // fine
Foo<std::string> b; // fails! And you get a nice (custom) error message

```
---
### **is_constexpr_default_constructible_**
- [constexprt constructor](https://stackoverflow.com/questions/31375381/does-specifying-constexpr-on-constructor-automatically-makes-all-objects-created)
- [void-decltye-mean](https://stackoverflow.com/questions/39279074/what-does-the-void-in-decltypevoid-mean-exactly)

```c++
struct is_constexpr_default_constructible_ {
  template <typename T>
  static constexpr auto make(tag_t<T>) -> decltype(void(T()), 0) {
    return (void(T()), 0);
  }
  // second param should just be: int = (void(T()), 0)
  // but under clang 10, crash: https://bugs.llvm.org/show_bug.cgi?id=47620
  // and, with assertions disabled, expectation failures showing compiler
  // deviation from the language spec
  // xcode renumbers clang versions so detection is tricky, but, if detection
  // were desired, a combination of __apple_build_version__ and __clang_major__
  // may be used to reduce frontend overhead under correct compilers: clang 12
  // under xcode and clang 10 otherwise
  template <typename T, int = make(tag<T>)>
  static std::true_type sfinae(T*);
  static std::false_type sfinae(void*);
  template <typename T>
  static constexpr bool apply =
      decltype(sfinae(static_cast<T*>(nullptr)))::value;
};

```
---
---
## **Unit**
- **read notes**
```c++
/// In functional programming, the degenerate case is often called "unit". In
/// C++, "void" is often the best analogue. However, because of the syntactic
/// special-casing required for void, it is frequently a liability for template
/// metaprogramming. So, instead of writing specializations to handle cases like
/// SomeContainer<void>, a library author may instead rule that out and simply
/// have library users use SomeContainer<Unit>. Contained values may be ignored.
/// Much easier.
///
/// "void" is the type that admits of no values at all. It is not possible to
/// construct a value of this type.
/// "unit" is the type that admits of precisely one unique value. It is
/// possible to construct a value of this type, but it is always the same value
/// every time, so it is uninteresting.
```
---
---
## **try**
  - [__attribute__((__visibility__("default")](https://blog.csdn.net/fengbingchun/article/details/78898623)
  - [inplace_t](https://juejin.cn/post/6917786024835825671)