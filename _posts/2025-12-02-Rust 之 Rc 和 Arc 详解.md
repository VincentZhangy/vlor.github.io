---
layout: post
title: Rust 之 Rc 和 Arc 详解
subtitle: "\"深入理解Rust引用计数机制：从单线程到多线程的共享方案\""
date: 2025-12-02
author: Vlor
header-img: img/bg_151.png
catalog: true
tags:
    - Rust
    - 智能指针
    - 引用计数
    - 并发编程
---

# 概述

在 Rust 的内存安全模型中， **所有权系统** 确保了每个值只有一个所有者，当所有者离开作用域时，值会被自动释放。但在实际开发中，我们经常需要**多个所有者共享同一数据**的场景——比如在图形界面中多个组件可能引用同一个数据源，或者在多线程环境中多个线程需要访问同一份数据。

`Rc<T>`（Reference Counted）和 `Arc<T>`（Atomic Reference Counted）正是为解决此类问题而生的智能指针。它们通过**引用计数**机制实现了多所有权模型：当最后一个所有者离开作用域时，数据才会被释放。两者的核心区别在于：`Rc<T>` 适用于单线程环境，而 `Arc<T>` 则通过原子操作保证了线程安全，可用于多线程环境。

## 核心概念

### 引用计数原理

引用计数的工作原理非常直观：

- 当创建一个 `Rc<T>` 或 `Arc<T>` 实例时，引用计数初始化为 **1**
- 调用 `clone()` 方法会创建一个新的所有者，同时将引用计数 **加 1**
- 当任何一个所有者离开作用域时，引用计数 **减 1**
- 当引用计数降至 **0** 时，底层数据会被自动释放

### 与所有权模型的关系

`Rc<T>` 和 `Arc<T>` 并没有打破 Rust 的所有权规则，而是通过巧妙的设计实现了**共享所有权**：

- 每个 `Rc<T>`/`Arc<T>` 实例都是数据的所有者之一
- 所有所有者共享对同一数据的访问权
- 只有当所有所有者都放弃所有权（引用计数为 0）时，数据才会被释放

这种模型特别适合以下场景：

- 数据需要在程序的多个部分共享，但无法确定哪个部分最后使用它
- 数据没有明确的单一所有者
- 需要在不复制数据的情况下共享大型数据结构

## Rc<T>详解

### 定义与基本使用

`Rc<T>` 位于 `std::rc` 模块，其基本定义如下：

```rust
pub struct Rc<T: ?Sized> {
    ptr: NonNull<RcBox<T>>,
    phantom: PhantomData<RcBox<T>>,
}

// RcBox 是实际存储数据和引用计数的结构
struct RcBox<T: ?Sized> {
    strong: Cell<usize>, // 强引用计数
    weak: Cell<usize>,   // 弱引用计数（用于解决循环引用）
    value: T,            // 实际存储的数据
}
```

**基本使用示例**：

```rust
use std::rc::Rc;

fn main() {
    // 创建 Rc 实例，引用计数 = 1
    let a = Rc::new(10);
    println!("引用计数: {}", Rc::strong_count(&a)); // 输出: 1

    // 克隆 Rc，引用计数 = 2
    let b = Rc::clone(&a);
    println!("引用计数: {}", Rc::strong_count(&a)); // 输出: 2

    // 再次克隆，引用计数 = 3
    let c = a.clone(); // 等价于 Rc::clone(&a)
    println!("引用计数: {}", Rc::strong_count(&a)); // 输出: 3

    // 离开作用域时，b 和 c 先被释放，引用计数减为 1
    // 最后 a 被释放，引用计数减为 0，数据被释放
}
```

### 常用方法解析

| 方法 | 作用 | 示例 |
|------|------|------|
| `Rc::new(value)` | 创建新的 Rc 实例 | `let rc = Rc::new(5);` |
| `Rc::clone(&rc)` | 克隆 Rc，增加引用计数 | `let rc2 = Rc::clone(&rc);` |
| `Rc::strong_count(&rc)` | 获取强引用计数 | `println!("{}", Rc::strong_count(&rc));` |
| `Rc::downgrade(&rc)` | 创建弱引用（Weak） | `let weak = Rc::downgrade(&rc);` |
| `rc.as_ref()` | 获取内部数据的不可变引用 | `let val = rc.as_ref();` |

### 不可变性限制

`Rc<T>` 只能提供对内部数据的**不可变访问**。这是因为 Rust 要确保在任何时候，要么有一个可变引用，要么有多个不可变引用，以避免数据竞争。

```rust
use std::rc::Rc;

fn main() {
    let rc = Rc::new(10);

    // 错误示例：无法获取可变引用
    // let mut val = rc.as_mut(); // 编译错误
}
```

如果需要在使用 `Rc<T>` 时修改数据，必须配合**内部可变性**机制，如 `RefCell<T>`：

```rust
use std::rc::Rc;
use std::cell::RefCell;

fn main() {
    let rc = Rc::new(RefCell::new(10));

    // 通过 RefCell 实现内部可变性
    *rc.borrow_mut() = 20;
    println!("修改后的值: {}", rc.borrow()); // 输出: 20
}
```

### 单线程限制

`Rc<T>` 不是线程安全的，因为它的引用计数操作**没有使用同步机制**。尝试在多线程中使用 `Rc<T>` 会导致编译错误：

```rust
use std::rc::Rc;
use std::thread;

fn main() {
    let rc = Rc::new(10);

    // 错误示例：Rc 不能跨线程传递
    // thread::spawn(move || {
    //     println!("{}", rc); // 编译错误：Rc<{integer}> cannot be sent between threads safely
    // }).join().unwrap();
}
```

编译器会提示 `Rc<T>` 没有实现 `Send` trait，因此不能安全地在线程间传递。这正是 `Arc<T>` 要解决的问题。

## Arc<T>详解

### 定义与线程安全

`Arc<T>`（Atomic Reference Counting）位于 `std::sync` 模块，其设计与 `Rc<T>` 非常相似，但使用**原子操作**来更新引用计数，从而保证了线程安全。

```rust
pub struct Arc<T: ?Sized> {
    ptr: NonNull<ArcInner<T>>,
    phantom: PhantomData<ArcInner<T>>,
}

// ArcInner 是实际存储数据和原子引用计数的结构
struct ArcInner<T: ?Sized> {
    strong: AtomicUsize, // 原子强引用计数
    weak: AtomicUsize,   // 原子弱引用计数
    data: T,             // 实际存储的数据
}
```

**原子操作**确保了即使在多线程环境中，引用计数的增减也是线程安全的，不会出现竞态条件。这使得 `Arc<T>` 实现了 `Send` 和 `Sync` trait（当 `T` 也实现这些 trait 时）。

### 基本使用示例

`Arc<T>` 的 API 与 `Rc<T>` 几乎完全相同，这使得它们可以在大多数情况下相互替换：

```rust
use std::sync::Arc;
use std::thread;

fn main() {
    // 创建 Arc 实例，引用计数 = 1
    let arc = Arc::new(10);
    println!("初始引用计数: {}", Arc::strong_count(&arc)); // 输出: 1

    // 创建 5 个线程共享这个 Arc
    let mut handles = Vec::new();
    for _ in 0..5 {
        // 克隆 Arc，引用计数增加
        let arc_clone = Arc::clone(&arc);

        // 将克隆的 Arc 移动到新线程
        let handle = thread::spawn(move || {
            println!("线程中的值: {}, 引用计数: {}", arc_clone, Arc::strong_count(&arc_clone));
        });

        handles.push(handle);
    }

    // 等待所有线程完成
    for handle in handles {
        handle.join().unwrap();
    }

    // 所有线程结束后，引用计数回到 1
    println!("最终引用计数: {}", Arc::strong_count(&arc)); // 输出: 1
}
```

### 与 Mutex 和 RwLock 配合使用

和 `Rc<T>` 一样，`Arc<T>` 只提供对数据的不可变访问。要在多线程中修改共享数据，需要配合**线程安全的内部可变性**机制，如 `Mutex<T>` 或 `RwLock<T>`：

```rust
use std::sync::{Arc, Mutex};
use std::thread;

fn main() {
    // Arc 包裹 Mutex，提供线程安全的共享可变性
    let counter = Arc::new(Mutex::new(0));
    let mut handles = Vec::new();

    for _ in 0..10 {
        let counter_clone = Arc::clone(&counter);

        let handle = thread::spawn(move || {
            // 锁定 Mutex 以获取可变访问权
            let mut num = counter_clone.lock().unwrap();
            *num += 1;
        });

        handles.push(handle);
    }

    for handle in handles {
        handle.join().unwrap();
    }

    println!("最终计数: {}", *counter.lock().unwrap()); // 输出: 10
}
```

`Mutex<T>` 提供了互斥访问——任何时候只有一个线程能获取锁并修改数据。如果需要更细粒度的控制（如允许多个读者同时访问），可以使用 `RwLock<T>`。

### 性能考量

`Arc<T>` 的线程安全是有代价的——原子操作比普通的整数操作**慢得多**。在单线程环境中，`Rc<T>` 的性能明显优于 `Arc<T>`：

| 操作 | Rc<T> (ns) | Arc<T> (ns) | 性能差异 |
|------|-----------|------------|----------|
| 创建新实例 | ~1 | ~1 | 基本相同 |
| clone() | ~1 | ~10-20 | Arc 慢 10-20 倍 |
| 释放（drop） | ~1 | ~10-20 | Arc 慢 10-20 倍 |

因此，**在单线程环境中应优先使用 `Rc<T>`，只在多线程环境中才使用 `Arc<T>` **。

## Rc 与 Arc 的对比

| 特性 | Rc<T> | Arc<T> |
|------|-------|--------|
| 线程安全 | ❌ 非线程安全 | ✅ 线程安全（原子操作） |
| 性能 | 更快（普通整数操作） | 较慢（原子操作） |
| 适用场景 | 单线程共享数据 | 多线程共享数据 |
| 实现 trait | 未实现 Send 和 Sync | 实现 Send 和 Sync（当 T 实现时） |
| API | `std::rc::Rc` | `std::sync::Arc` |
| 引用计数 | Cell<usize> | AtomicUsize |
| 典型配合类型 | RefCell<T>（单线程内部可变性） | Mutex<T>/RwLock<T>（线程安全内部可变性） |

### 选择指南

在实际开发中，如何选择 `Rc<T>` 和 `Arc<T>`？可以遵循以下原则：

1. **默认使用 `Rc<T>`**：如果确定代码运行在单线程环境
2. **需要多线程共享时使用 `Arc<T>`**：当数据需要跨线程传递或访问
3. **需要修改数据时**：
   - 单线程：`Rc<RefCell<T>>`
   - 多线程：`Arc<Mutex<T>>` 或 `Arc<RwLock<T>>`
4. **性能敏感场景**：优先考虑 `Rc<T>`，避免不必要的原子操作

## 弱引用与循环引用

### 循环引用问题

`Rc<T>` 和 `Arc<T>` 都存在一个潜在问题：**循环引用**。当两个或多个 `Rc<T>` 实例相互引用时，它们的引用计数永远不会降为 0，导致**内存泄漏**。

```rust
use std::rc::Rc;
use std::cell::RefCell;

struct Node {
    value: i32,
    next: Option<Rc<RefCell<Node>>>, // 指向 next 节点
}

impl Drop for Node {
    fn drop(&mut self) {
        println!("Node {} 被释放", self.value);
    }
}

fn main() {
    // 创建循环引用：a -> b -> a
    let a = Rc::new(RefCell::new(Node { value: 1, next: None }));
    let b = Rc::new(RefCell::new(Node { value: 2, next: None }));

    // 构建循环
    a.borrow_mut().next = Some(Rc::clone(&b));
    b.borrow_mut().next = Some(Rc::clone(&a));

    // 离开作用域时，a 和 b 的引用计数都是 2，不会被释放！
    // 因此 Drop 不会被调用，没有输出
}
```

在这个例子中，`a` 和 `b` 形成了循环引用，导致它们的引用计数永远不会降为 0，从而造成内存泄漏。

### Weak<T>解决方案

为了解决循环引用问题，`Rc<T>` 和 `Arc<T>` 提供了**弱引用**机制（`Weak<T>`）。弱引用不会增加强引用计数，因此不会阻止数据的释放。

```rust
use std::rc::{Rc, Weak};
use std::cell::RefCell;

struct Node {
    value: i32,
    next: Option<Weak<RefCell<Node>>>, // 使用 Weak 而非 Rc
}

impl Drop for Node {
    fn drop(&mut self) {
        println!("Node {} 被释放", self.value);
    }
}

fn main() {
    let a = Rc::new(RefCell::new(Node { value: 1, next: None }));
    let b = Rc::new(RefCell::new(Node { value: 2, next: None }));

    // a 强引用 b，b 弱引用 a
    a.borrow_mut().next = Some(Rc::downgrade(&b)); //  downgrade 创建弱引用
    b.borrow_mut().next = Some(Rc::downgrade(&a));

    // 现在 a 的引用计数是 1，b 的引用计数是 1
    // 离开作用域时，两者都会被释放
}
```

弱引用的主要特点：

- 通过 `Rc::downgrade(&rc)` 或 `Arc::downgrade(&arc)` 创建
- 弱引用计数不影响数据的生命周期
- 使用 `upgrade()` 方法可以将弱引用转换为强引用（返回 `Option<Rc<T>>` 或 `Option<Arc<T>>`）
- 如果底层数据已被释放，`upgrade()` 会返回 `None`

**弱引用使用示例**：

```rust
use std::rc::{Rc, Weak};
use std::cell::RefCell;

fn main() {
    let strong = Rc::new(RefCell::new(42));
    println!("强引用计数: {}", Rc::strong_count(&strong)); // 输出: 1

    // 创建弱引用
    let weak = Rc::downgrade(&strong);
    println!("弱引用计数: {}", Rc::weak_count(&strong));   // 输出: 1

    // 将弱引用升级为强引用
    if let Some(strong2) = weak.upgrade() {
        println!("升级成功，值: {}", strong2.borrow()); // 输出: 42
        println!("强引用计数: {}", Rc::strong_count(&strong)); // 输出: 2
    }

    // 释放 strong2 后，强引用计数回到 1
    drop(strong);

    // 此时底层数据已被释放，升级会失败
    if let Some(_) = weak.upgrade() {
        println!("升级成功");
    } else {
        println!("升级失败，数据已被释放"); // 输出: 升级失败，数据已被释放
    }
}
```

弱引用非常适合表示**非拥有关系**，如树结构中的父节点引用子节点，或图结构中的非强连接关系

## 实践应用

### 单线程应用：DOM 树结构

在前端框架或 GUI 库中，DOM 元素通常需要被多个父元素引用，`Rc<RefCell<Node>>` 是理想的选择：

```rust
use std::rc::Rc;
use std::cell::RefCell;
use std::collections::VecDeque;

struct DomNode {
    tag: String,
    children: VecDeque<Rc<RefCell<DomNode>>>,
    parent: Option<Weak<RefCell<DomNode>>>, // 父节点使用弱引用避免循环
}

impl DomNode {
    fn new(tag: &str) -> Rc<RefCell<Self>> {
        Rc::new(RefCell::new(DomNode {
            tag: tag.to_string(),
            children: VecDeque::new(),
            parent: None,
        }))
    }

    fn append_child(&mut self, child: Rc<RefCell<DomNode>>) {
        child.borrow_mut().parent = Some(Rc::downgrade(&Rc::new(RefCell::new(self.clone()))));
        self.children.push_back(child);
    }

    fn get_parent(&self) -> Option<Rc<RefCell<DomNode>>> {
        self.parent.as_ref().and_then(|weak| weak.upgrade())
    }
}

fn main() {
    let body = DomNode::new("body");
    let div = DomNode::new("div");
    let p = DomNode::new("p");

    // 构建 DOM 树: body -> div -> p
    div.borrow_mut().append_child(p);
    body.borrow_mut().append_child(div);

    // 遍历 DOM 树
    println!("根节点: {}", body.borrow().tag);
    if let Some(first_child) = body.borrow().children.front() {
        println!("子节点: {}", first_child.borrow().tag);
    }
}
```

### 多线程应用：线程池

`Arc<T>` 非常适合实现线程池，其中多个工作线程需要共享对任务队列的访问：

```rust
use std::sync::{Arc, Mutex, mpsc};
use std::thread;

// 工作线程
struct Worker {
    id: usize,
    thread: Option<thread::JoinHandle<()>>,
}

impl Worker {
    fn new(id: usize, receiver: Arc<Mutex<mpsc::Receiver<Job>>>) -> Self {
        let thread = thread::spawn(move || loop {
            // 从通道接收任务
            let job = receiver.lock().unwrap().recv();

            match job {
                Ok(job) => {
                    println!("工作线程 {} 执行任务", id);
                    job();
                },
                Err(_) => {
                    println!("工作线程 {} 终止", id);
                    break;
                }
            }
        });

        Worker {
            id,
            thread: Some(thread),
        }
    }
}

// 任务类型（函数指针）
type Job = Box<dyn FnOnce() + Send + 'static>;

// 线程池
struct ThreadPool {
    workers: Vec<Worker>,
    sender: mpsc::Sender<Job>,
}

impl ThreadPool {
    // 创建线程池
    fn new(size: usize) -> Self {
        assert!(size > 0);

        let (sender, receiver) = mpsc::channel();
        let receiver = Arc::new(Mutex::new(receiver));

        let mut workers = Vec::with_capacity(size);
        for id in 0..size {
            workers.push(Worker::new(id, Arc::clone(&receiver)));
        }

        ThreadPool { workers, sender }
    }

    // 提交任务
    fn execute<F>(&self, f: F) where F: FnOnce() + Send + 'static {
        let job = Box::new(f);
        self.sender.send(job).unwrap();
    }
}

fn main() {
    // 创建包含 4 个工作线程的线程池
    let pool = ThreadPool::new(4);

    // 提交 8 个任务
    for i in 0..8 {
        pool.execute(move || {
            println!("任务 {} 正在执行", i);
        });
    }
}
```

在这个例子中，`Arc<Mutex<Receiver<Job>>>` 允许所有工作线程安全地共享同一个任务接收器，而不需要复制数据。

### 性能优化技巧

1. **最小化 Arc/Mutex 粒度**：只对需要共享的部分使用 `Arc<T>`，避免将整个大型结构体放入 `Arc<T>`

```rust
// 不推荐：整个结构体都被 Arc 包裹
struct Data {
    config: Config,       // 只读配置
    counter: Mutex<u32>,  // 需要共享修改的计数器
}
let data = Arc::new(Data { ... });

// 推荐：只对需要共享的部分使用 Arc
struct Data {
    config: Config,       // 直接存储，不需要共享
}
let data = Data { ... };
let counter = Arc::new(Mutex::new(0)); // 只共享计数器
```

2. **减少克隆操作**：`Arc::clone()` 虽然比复制数据便宜，但仍有开销，应尽量减少
3. **使用弱引用缓存**：对于频繁访问但不需要长期拥有的数据，使用 `Weak<T>` 作为缓存
4. **单线程优先**：在确定单线程的场景中，始终优先使用 `Rc<T>` 而非 `Arc<T>`

## 常见错误与解决方案

### 错误 1：尝试修改 Rc<T> 中的数据

**问题**：直接尝试修改 `Rc<T>` 中的数据会导致编译错误。

```rust
use std::rc::Rc;

fn main() {
    let rc = Rc::new(10);
    *rc = 20; // 编译错误：cannot assign to data in an `Rc`
}
```

**解决方案**：使用 `Rc<RefCell<T>>` 组合提供内部可变性：

```rust
use std::rc::Rc;
use std::cell::RefCell;

fn main() {
    let rc = Rc::new(RefCell::new(10));
    *rc.borrow_mut() = 20; // 正确
    println!("{}", rc.borrow()); // 输出: 20
}
```

### 错误 2：在多线程中使用 Rc<T>

**问题**：尝试在多线程中传递 `Rc<T>` 会导致编译错误。

```rust
use std::rc::Rc;
use std::thread;

fn main() {
    let rc = Rc::new(10);
    thread::spawn(move || {
        println!("{}", rc); // 编译错误：Rc<{integer}> cannot be sent between threads safely
    }).join().unwrap();
}
```

**解决方案**：使用 `Arc<T>` 代替 `Rc<T>`：

```rust
use std::sync::Arc;
use std::thread;

fn main() {
    let arc = Arc::new(10);
    thread::spawn(move || {
        println!("{}", arc); // 正确
    }).join().unwrap();
}
```

### 错误 3：循环引用导致内存泄漏

**问题**：`Rc<T>` 或 `Arc<T>` 的循环引用会导致内存泄漏。

```rust
use std::rc::{Rc, Weak};
use std::cell::RefCell;

struct A {
    b: Option<Rc<RefCell<B>>>,
}

struct B {
    a: Option<Rc<RefCell<A>>>,
}

fn main() {
    let a = Rc::new(RefCell::new(A { b: None }));
    let b = Rc::new(RefCell::new(B { a: None }));

    a.borrow_mut().b = Some(Rc::clone(&b));
    b.borrow_mut().a = Some(Rc::clone(&a));
    // 循环引用导致 a 和 b 的引用计数永远不为 0
}
```

**解决方案**：使用弱引用打破循环：

```rust
use std::rc::{Rc, Weak};
use std::cell::RefCell;

struct A {
    b: Option<Rc<RefCell<B>>>,
}

struct B {
    a: Option<Weak<RefCell<A>>>, // 使用弱引用
}

fn main() {
    let a = Rc::new(RefCell::new(A { b: None }));
    let b = Rc::new(RefCell::new(B { a: None }));

    a.borrow_mut().b = Some(Rc::clone(&b));
    b.borrow_mut().a = Some(Rc::downgrade(&a));
    // 现在 a 的引用计数是 1，b 的引用计数是 1
    // 离开作用域时，两者都会被正确释放
}
```

### 错误 4：过度使用 Arc<Mutex<T>>

**问题**：盲目使用 `Arc<Mutex<T>>` 会导致性能问题和不必要的复杂性

```rust
// 不推荐：过度使用 Arc<Mutex>
struct User {
    id: Arc<Mutex<u64>>,
    name: Arc<Mutex<String>>,
    email: Arc<Mutex<String>>,
}
```

**解决方案**：只对需要共享修改的部分使用 `Arc<Mutex<T>>`，其他部分使用普通值：

```rust
// 推荐：最小化 Arc<Mutex> 的使用范围
struct UserData {
    id: u64,
    name: String,
    email: String,
}

struct User {
    data: Arc<Mutex<UserData>>, // 整个 UserData 共享修改
}
```

或者，如果只有部分字段需要修改，可以将这些字段单独包装：

```rust
struct User {
    id: u64, // 不可变，不需要 Mutex
    name: Arc<Mutex<String>>, // 只有 name 需要修改
    email: String, // 不可变，不需要 Mutex
}
```

## 总结要点

1. **Rc<T> 和 Arc<T> 提供共享所有权**：允许多个所有者共享同一数据，通过引用计数管理生命周期

2. **单线程用 Rc<T>，多线程用 Arc<T>**：
   - `Rc<T>` 性能更好，但不线程安全
   - `Arc<T>` 线程安全，但通过原子操作实现，有性能开销

3. **不可变性默认**：两者都只提供对数据的不可变访问，需要配合内部可变性机制进行修改：
   - 单线程：`Rc<RefCell<T>>`
   - 多线程：`Arc<Mutex<T>>` 或 `Arc<RwLock<T>>`

4. **循环引用问题**：使用弱引用 `Weak<T>` 可以打破循环引用，避免内存泄漏

5. **性能考量**：
   - 避免不必要的 `clone()` 操作
   - 最小化共享状态的范围
   - 在不需要线程安全时选择 `Rc<T>`

6. **适用场景**：
   - 数据需要在多个部分共享且无法确定所有者
   - 大型数据结构需要高效共享（避免复制）
   - 多线程环境中共享不可变数据或通过同步机制共享可变数据

`Rc<T>` 和 `Arc<T>` 是 Rust 所有权系统的重要补充，它们使得在保持内存安全的同时实现共享所有权成为可能。理解并正确使用这些智能指针，对于编写高效、安全的 Rust 代码至关重要。

## 进阶资源

- [Rust 官方文档 - Rc<T>](https://doc.rust-lang.org/std/rc/struct.Rc.html)
- [Rust 官方文档 - Arc<T>](https://doc.rust-lang.org/std/sync/struct.Arc.html)
- [Rustonomicon - 引用计数](https://doc.rust-lang.org/nomicon/rc.html)
- [Rust By Example - Rc](https://doc.rust-lang.org/rust-by-example/std/rc.html)
- [Rust 标准库源码 - rc.rs](https://github.com/rust-lang/rust/blob/master/library/alloc/src/rc.rs)
- [Rust 标准库源码 - arc.rs](https://github.com/rust-lang/rust/blob/master/library/alloc/src/sync/arc.rs)
