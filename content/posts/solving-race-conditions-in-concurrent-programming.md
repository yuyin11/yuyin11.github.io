+++
date = '2025-12-05T10:29:29+08:00'
draft = false
title = '解决条件竞争的三种方法'
tags = ['C++并发编程']
+++

# 前言

> 最近在学习C++并发编程  
> 为了保护共享数据，我们需要加锁；但为了保持高性能和避免死锁，我们又需要尽量减少持锁的时间  
> 下面将介绍三种方法，如何解决条件竞争

# 问题
我们以一个经典的线程安全栈的pop操作为例。
```cpp
template<typename T>
class naive_stack {
  std::stack<T> s;
  mutable std::mutex m;
public:
  bool empty() const {
    std::lock_guard<std::mutex> lock(m);
    return s.empty();
  }
  void push(T value) {
    std::lock_guard<std::mutex> lock(m);
    s.push(std::move(value));
  }
  //有问题的pop操作
  T pop() {
    std::lock_guard<std::mutex> lock(m);
    if (s.empty()) {
      throw empty_stack();
    }
    T value = s.top();
    s.pop();
    return value;
  }
}
```
异常安全问题：返回元素时的复制构造可能抛出异常(return value)，导致“元素已弹出但未被成功传递”的矛盾状态。
- 锁已释放，元素丢失。

# 解决
## 传入引用
> 调用者预先分配对象，函数内部通过移动/复制将栈顶元素直接写入调用者的对象（value = std::move(data_stack.top()))  
> 即使移动/复制抛出异常，栈未被修改，无数据丢失风险
```cpp
void pop(T& value) {
  std::lock_guard<std::mutex> lock(m);
  if (s.empty()) throw empty_stack();
  value = s.top();
  s.pop();
}
```
缺点：
- 构造开销，预先构造一个T类型对象，如果构造代价很高，这将是一种浪费。
- 可读性较差：需要先创建一个对象再传入
- 无法用于某些情况：如果类型T没有默认构造函数，将会很麻烦。

## noexcept构造函数
> 要求T的移动/复制构造函数标记为noexcept，确保返回时不会抛出异常  
> 因为std::move和返回value操作不会抛出异常，所以整个pop操作是安全连续的，没有中间状态会让数据暴露在竞争风险中

优点：
- 接口干净直观。

缺点：
- 对类型T提出了硬性要求，对于某些类型可能不现实
- 如果T的移动构造函数实际上抛出了异常，虽然数据不会丢失，程序会崩溃

## 返回指针，指向弹出的元素
> 存储std::shared_ptr<T>,返回指针  
> 即使返回时复制指针，也不会抛出异常，且所有权转移通过指针完成，无数据丢失风险

```cpp
std::shared_ptr<T> pop() {
  std::lock_guard<std::mutex> lock(m);
  if (s.empty()) throw empty_stack();
  std::shared_ptr<T> const res(std::move(s.top()));
  s.pop();
  return res;
}
void push(T value) {
  std::lock_guard<std::mutex> lock(m);
  s.push(std::make_shared<T>(std::move(value)));
}
```
优点：
- 避免了返回对象时可能发生的昂贵拷贝操作
- 接口清晰

缺点：
- 动态内存分配开销
- 需管理指针生命周期
