## TEST

### mock-allocator
- [gmock](https://blog.csdn.net/weixin_34174322/article/details/86010459?spm=1001.2101.3001.6661.1&utm_medium=distribute.pc_relevant_t0.none-task-blog-2%7Edefault%7ECTRLIST%7ERate-1-86010459-blog-51438033.pc_relevant_aa&depth_1-utm_source=distribute.pc_relevant_t0.none-task-blog-2%7Edefault%7ECTRLIST%7ERate-1-86010459-blog-51438033.pc_relevant_aa&utm_relevant_index=1)
- [UNIT TEST](https://zhangyuyu.github.io/cpp-unit-test/)

- nest template
```c++
// Allocator = mock_allocator<T>
template <typename Allocator> class allocator_ref { 
  ...
  public:
   Allocator* get() const { return alloc_; }
 
   value_type* allocate(size_t n) {
     return std::allocator_traits<Allocator>::allocate(*alloc_, n);
   }
   void deallocate(value_type* p, size_t n) { alloc_->deallocate(p, n); }

template <typename T> class mock_allocator {
 public:
  mock_allocator() {}
  mock_allocator(const mock_allocator&) {}
  using value_type = T;
  MOCK_METHOD1_T(allocate, T*(size_t n));
  MOCK_METHOD2_T(deallocate, void(T* p, size_t n));
};
```