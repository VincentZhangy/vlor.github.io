---
layout:     post
title:      Rust 之 Send 和 Sync 详解
subtitle:   "\"深入理解Rust并发安全的基石：Send和Sync trait\""
date:       2025-11-24
author:     Vlor
header-img: img/post-bg-rust.jpg
catalog: true
tags:
    - Rust
    - 并发编程
    - 多线程
    - 内存安全
---

# 概述
Send和Sync是Rust语言中保证并发安全的两个核心标记trait，它们在编译期就能防止数据竞争，是Rust无数据竞争并发编程的基石。

# 核心概念

### 什么是Send和Sync？
- **Send**: 表示类型的值可以安全地跨线程传递所有权
- **Sync**: 表示类型的引用可以安全地在多个线程间共享
- **标记trait**: 不包含任何方法，只用于编译期安全检查

### 数据竞争防护
Rust通过所有权系统结合Send/Sync，在编译期就杜绝了以下情况：
- 两个线程同时写入同一数据
- 一个线程写入数据时另一个线程读取
- 缺乏同步机制的并发访问

# Send Trait详解

### 定义与作用
`Send` trait表明类型的实例所有权可以安全地转移到另一个线程。

```rust
pub unsafe auto trait Send {}
```

### 实现Send的类型
```rust
// 基本类型都是Send
let x: i32 = 42;        // Send ✓
let s: String = "hello".to_string();  // Send ✓
let v: Vec<i32> = vec![1, 2, 3];      // Send ✓

// 包含Send类型的结构体自动成为Send
struct MyStruct {
    data: i32,
    name: String,
} // 自动实现Send ✓
```

### 非Send类型示例
```rust
use std::rc::Rc;

let rc = Rc::new(42);
// Rc<i32>不是Send，以下代码无法编译：
// std::thread::spawn(move || {
//     println!("{}", rc);
// });
```
#### 常见非Send类型：

* Rc<T> - 非原子引用计数

* *const T, *mut T - 裸指针

* Cell<T> - 非线程安全的内部可变性

# Sync Trait详解
### 定义与作用
Sync trait表明类型的不可变引用(&T)可以安全地在多个线程间共享。

```rust
pub unsafe auto trait Sync {}
```
### 实现Sync的类型
```rust
// 基本类型的不可变引用都是Sync
let x: &i32 = &42;           // Sync ✓
let s: &String = &"hello".to_string(); // Sync ✓

// 同步原语
use std::sync::Mutex;
let lock: Mutex<i32> = Mutex::new(0); // Sync ✓
```

### 非Sync类型示例
```rust
use std::cell::RefCell;

let cell = RefCell::new(42);
// &RefCell<i32>不是Sync，以下代码无法编译：
// std::thread::spawn(move || {
//     *cell.borrow_mut() = 100;
// });
```

### 常见非Sync类型：

* Rc<T> - 非原子引用计数

* Cell<T>, RefCell<T> - 非线程安全的内部可变性

# 自动推导机制
### 编译器自动实现
Rust编译器会自动为复合类型实现Send和Sync：

```rust
// 自动实现Send的例子
struct Point {
    x: i32,
    y: i32,
} // 自动实现Send和Sync ✓

// 包含非Send类型导致整体非Send
struct NotSend {
    data: Rc<i32>,
} // 自动实现!Send和!Sync ✗
```
### 手动实现（不安全）
在极少数情况下需要手动实现：

```rust
use std::marker::PhantomData;

struct MyUnsafeType<T> {
    data: *mut T,
    _marker: PhantomData<T>,
}

// 不安全的手动实现
unsafe impl<T> Send for MyUnsafeType<T> {}
unsafe impl<T> Sync for MyUnsafeType<T> {}
```

# 实践应用
### 多线程编程
```rust
use std::thread;
use std::sync::{Arc, Mutex};

fn main() {
    // Arc是Send + Sync，可以安全跨线程共享
    let counter = Arc::new(Mutex::new(0));
    let mut handles = vec![];

    for _ in 0..10 {
        let counter = Arc::clone(&counter);
        let handle = thread::spawn(move || {
            let mut num = counter.lock().unwrap();
            *num += 1;
        });
        handles.push(handle);
    }

    for handle in handles {
        handle.join().unwrap();
    }

    println!("Result: {}", *counter.lock().unwrap());
}
```

|类型	Send	Sync	说明
|---|---|---|---
|i32, f64, bool	✓	✓	基本类型
|String, Vec<T>	✓	✓	集合类型
|Rc<T>	✗	✗	非原子引用计数
|Arc<T>	✓	✓	原子引用计数
|Mutex<T>	✓	✓	互斥锁
|Cell<T>	✓	✗	线程内部可变性
|RefCell<T>	✓	✗	线程内部可变性

## 常见错误解决
```rust
// 错误：Rc不能跨线程
// let rc = Rc::new(5);
// thread::spawn(move || { println!("{}", rc); });

// 解决方案：使用Arc代替Rc
use std::sync::Arc;
let arc = Arc::new(5);
thread::spawn(move || { println!("{}", arc); });
```

# 总结要点
1、Send关注所有权转移：能否安全地将值移动到另一个线程

2、Sync关注引用共享：能否安全地在多个线程间共享不可变引用

3、自动推导：编译器自动为复合类型实现，基于字段类型

4、编译期检查：所有并发安全问题在编译时发现

5、安全抽象：通过标准库类型（Arc、Mutex等）构建安全并发程序

# 进阶资源
* [The Rust Programming Language - 并发章节](https://doc.rust-lang.org/book/ch16-00-concurrency.html)
* [Rustonomicon - 并发安全](https://doc.rust-lang.org/nomicon/send-and-sync.html)
* [标准库文档 - Send和Sync](https://doc.rust-lang.org/std/marker/trait.Send.html)

# Rust社区风采
[https://example.com/rust-community.jpg](https://example.com/rust-community.jpg)
Rust社区致力于构建安全、并发的系统编程未来
