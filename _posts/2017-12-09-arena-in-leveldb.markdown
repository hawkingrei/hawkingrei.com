---
layout:     post
title:      "LevelDB代码阅读：Arena"
date:       2017-12-09 14:00:00
author:     "Hawkingrei"
header-img: "img/post-bg-2015.jpg"
catalog:    true
tags:
    - LevelDB
---

首先我们来看定义

```c++
class Arena {
 public:
  Arena();
  ~Arena();  

  // Return a pointer to a newly allocated memory block of "bytes" bytes.
  char* Allocate(size_t bytes);

  // Allocate memory with the normal alignment guarantees provided by malloc
  char* AllocateAligned(size_t bytes);

  // Returns an estimate of the total memory usage of data allocated
  // by the arena.
  size_t MemoryUsage() const {
    return reinterpret_cast<uintptr_t>(memory_usage_.NoBarrier_Load());
  }

 private:
  char* AllocateFallback(size_t bytes);
  char* AllocateNewBlock(size_t block_bytes);

  // Allocation state
  char* alloc_ptr_;
  size_t alloc_bytes_remaining_;

  // Array of new[] allocated memory blocks
  std::vector<char*> blocks_;

  // Total memory usage of the arena.
  port::AtomicPointer memory_usage_;

  // No copying allowed
  Arena(const Arena&);
  void operator=(const Arena&);
};
```
从定义来看，除了构造函数和析构函数，用户只需要调用```Allocate```、```AllocateAligned```和```MemoryUsage```就可以完成内存的调用。

其中MemoryUsage的定义如下，

```c++
  size_t MemoryUsage() const {
    return reinterpret_cast<uintptr_t>(memory_usage_.NoBarrier_Load());
  }
```

MemoryUsage直接读取私有的memory_usage，转换成uintptr_t类型，

```NoBarrier_Load```方法是Leveldb的AtomicPointer实现，其位置在Leveldb的port目录下，是专门处理Leveldb跨平台特性的。之后我会来介绍。




```c++
inline char* Arena::Allocate(size_t bytes) {
  // The semantics of what to return are a bit messy if we allow
  // 0-byte allocations, so we disallow them here (we don't need
  // them for our internal use).
  assert(bytes > 0);
  if (bytes <= alloc_bytes_remaining_) {
    char* result = alloc_ptr_;
    alloc_ptr_ += bytes;
    alloc_bytes_remaining_ -= bytes;
    return result;
  }
  return AllocateFallback(bytes);
}
```

首先检查输入的```bytes```是否大于0，然后申请内存大小```bytes```是否小于剩余预先分配的内存，如果小于，则移动```alloc_ptr_```指针，修改剩余内存大小，否则向系统申请内存。
