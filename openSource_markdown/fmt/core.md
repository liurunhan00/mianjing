## **core**
---
### Traits
 - [std::conditional](https://blog.csdn.net/photon222/article/details/99327989)
 - [std::declval](https://segmentfault.com/a/1190000040841943)
 - [std::is_constructible](https://blog.csdn.net/hyl999/article/details/120647175?spm=1001.2101.3001.6650.2&utm_medium=distribute.pc_relevant.none-task-blog-2%7Edefault%7EBlogCommendFromBaidu%7ERate-2-120647175-blog-101074915.t0_edu_mix&depth_1-utm_source=distribute.pc_relevant.none-task-blog-2%7Edefault%7EBlogCommendFromBaidu%7ERate-2-120647175-blog-101074915.t0_edu_mix&utm_relevant_index=3)
 - [std::allocator_traits](https://learn.microsoft.com/zh-cn/cpp/standard-library/allocator-traits-class?view=msvc-170)
---
### **buffer**
#### **buffer**
- `store on stack(SSO: but longer length)`
- some basic member and functions
  ```
  member: T* ptr, size, capacity
  grow(): Increases the buffer capacity to hold at least *capacity* elements.  
  clear() -> {size = 0} // not clear ptr
  ~buffer() is not virtual because nerver use buffer directly
  ```
#### **buffer_traits**
- pure offer two traits func
  ```c++
  auto count() const -> size_t { return 0; }
  auto limit(size_t size) -> size_t { return size; }
  ```
#### **fixed_buffer_traits**
- offer fixed traits func
  ```c++
  auto count() const -> size_t { return count_; }
  auto limit(size_t size) -> size_t {
    size_t n = limit_ > count_ ? limit_ - count_ : 0;
    count_ += size;
    return size < n ? size : n;
  }
  ```
#### **iterator_buffer**
- buffer size = 256
- flush
  ```c++
  void flush() {
    auto size = this->size();
    this->clear();
    out_ = copy_str<T>(data_, data_ + this->limit(size), out_);
  }
  ```

---
### **string**
#### **basic_string_view**
- **`An implementation of ``std::basic_string_view`` for pre-C++17`**

- **member:**
  ```c++
  const Char* data_;
  size_t size_;
  ```
- **why use:** 
  - [avoid copy string](https://zhuanlan.zhihu.com/p/529073150)
  
#### **compile_string**
- **` A base class for compile-time strings.`**
  ```c++
  struct compile_string {};
  ```
  - [usage-of-empty-struct](https://stackoverflow.com/questions/60685261/usage-of-empty-structs-in-c)

#### **copy_str**
  ```c++
  template <typename Char, typename T, typename U,
          FMT_ENABLE_IF(
              std::is_same<remove_const_t<T>, U>::value&& is_char<U>::value)>
  FMT_CONSTEXPR auto copy_str(T* begin, T* end, U* out) -> U* { 
    if (is_constant_evaluated())  // constexpr optimization
      return copy_str<Char, T*, U*>(begin, end, out);
    auto size = to_unsigned(end - begin);
    memcpy(out, begin, size * sizeof(U)); // POD is really fast
    return out + size;
  }
  ```
  - [is_constant_evaluated()](https://qingcms.gitee.io/cppreference/20210212/zh/cpp/types/is_constant_evaluated.html)

---
### **context**

#### **basic_format_parse_context**
#### **compile_parse_context**
`not use atomic ref i guess because it's compile_parse or there is another lock outside the context`

---
### **value**
`A formatting argument value`
- **types:** 
  ```c++
    union {
    monostate no_value;
    int int_value;
    unsigned uint_value;
    long long long_long_value;
    unsigned long long ulong_long_value;
    int128_opt int128_value;
    uint128_opt uint128_value;
    bool bool_value;
    char_type char_value;
    float float_value;
    double double_value;
    long double long_double_value;
    const void* pointer;
    string_value<char_type> string;
    custom_value<Context> custom;
    named_arg_value<char_type> named_args;
  };
  ```
- **constructors**
  - `custom constructors and error handle constructors`
    ```c++
    struct unformattable {};
    struct unformattable_char : unformattable {};
    struct unformattable_const : unformattable {};
    struct unformattable_pointer : unformattable {};
    template <typename T> FMT_CONSTEXPR FMT_INLINE value(T& val) {
    using value_type = remove_cvref_t<T>;
    custom.value = const_cast<value_type*>(&val);
    // Get the formatter type through the context to allow different contexts
    // have different extension points, e.g. `formatter<T>` for `format` and
    // `printf_formatter<T>` for `printf`.
    custom.format = format_custom_arg<
        value_type,
        conditional_t<has_formatter<value_type, Context>::value,
                      typename Context::template formatter_type<value_type>,
                      fallback_formatter<value_type, char_type>>>;
    }
    value(unformattable);
    value(unformattable_char);
    value(unformattable_const);
    value(unformattable_pointer);
    ```
  - `look how to handle this error`
    ```c++
    template <typename Context, typename T>
    FMT_CONSTEXPR FMT_INLINE auto make_value(T&& val) -> value<Context> {
      const auto& arg = arg_mapper<Context>().map(FMT_FORWARD(val));

      constexpr bool formattable_char =
          !std::is_same<decltype(arg), const unformattable_char&>::value;
      static_assert(formattable_char, "Mixing character types is disallowed.");

      constexpr bool formattable_const =
          !std::is_same<decltype(arg), const unformattable_const&>::value;
      static_assert(formattable_const, "Cannot format a const argument.");

      // Formatting of arbitrary pointers is disallowed. If you want to output
      // a pointer cast it to "void *" or "const void *". In particular, this
      // forbids formatting of "[const] volatile char *" which is printed as bool
      // by iostreams.
      constexpr bool formattable_pointer =
          !std::is_same<decltype(arg), const unformattable_pointer&>::value;
      static_assert(formattable_pointer,
                    "Formatting of non-void pointers is disallowed.");

      constexpr bool formattable =
          !std::is_same<decltype(arg), const unformattable&>::value;
      static_assert(
          formattable,
          "Cannot format an argument. To make type T formattable provide a "
          "formatter<T> specialization: https://fmt.dev/latest/api.html#udt");
      return {arg};
    }
    ```
#### **custom_value**
`which mean you can define your own value type`
```c++
template <typename Context> struct custom_value { // 自定义值
  using parse_context = typename Context::parse_context_type;
  void* value;
  void (*format)(void* arg, parse_context& parse_ctx, Context& ctx);
};
```
`see core-test.cc`
```c++
struct custom_context {
  using char_type = char;
  using parse_context_type = fmt::format_parse_context;

  bool called = false;

  template <typename T> struct formatter_type {
    auto parse(fmt::format_parse_context& ctx) -> decltype(ctx.begin()) {
      return ctx.begin();
    }

    const char* format(const T&, custom_context& ctx) {
      ctx.called = true;
      return nullptr;
    }
  };

  void advance_to(const char*) {}
};

struct test_struct {};

TEST(arg_test, make_value_with_custom_context) {
  auto t = test_struct();
  fmt::detail::value<custom_context> arg(
      fmt::detail::arg_mapper<custom_context>().map(t));
  auto ctx = custom_context();
  auto parse_ctx = fmt::format_parse_context("");
  arg.custom.format(&t, parse_ctx, ctx);
  EXPECT_TRUE(ctx.called);
}
```

#### **named_arg_value**

- `which is a pair of {const Char* , int}`





### **`print`**
- `after read fmt these days, i realize read code should from top then go to bottom...`

```c++
// e.g. fmt::print("Elapsed time: {0:.2f} seconds", 1.23);
template <typename... T>
FMT_INLINE void print(format_string<T...> fmt, T&&... args) {
  const auto& vargs = fmt::make_format_args(args...);
  return detail::is_utf8() ? vprint(fmt, vargs)
                           : detail::vprint_mojibake(stdout, fmt, vargs);
}
```
- easy to understand the structure
  - **args parse**
