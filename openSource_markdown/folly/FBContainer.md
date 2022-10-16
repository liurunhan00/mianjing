## **FBVector**

### **ScopeGuard**
- if exception happened, cathch and execute function_
```c++
void execute() noexcept(InvokeNoexcept) {
  if (InvokeNoexcept) {
    using R = decltype(function_());
    auto catcher_word = reinterpret_cast<uintptr_t>(&terminate);
    auto catcher = reinterpret_cast<R (*)()>(catcher_word);
    catch_exception(function_, catcher);
  } else {
    function_();
  }
}
```