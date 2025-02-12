# 无谓并发
Here are the topics we’ll cover in this chapter:

* How to create threads to run multiple pieces of code at the same time
* Message-passing concurrency, where channels send messages between threads
* Shared-state concurrency, where multiple threads have access to some piece of data
* The Sync and Send traits, which extend Rust’s concurrency guarantees to user-defined types as well as types provided by the standard library

## thread::spawn
> 通过thread::spawn来创建线程

```rust
use std::thread;
use std::time::Duration;

fn main() {
    // spawn之后线程就开始跑了。
    thread::spawn(|| {
        for i in 1..10 {
            println!("hi number {} from the spawned thread!", i);
            thread::sleep(Duration::from_millis(1));
        }
    });

    for i in 1..5 {
        println!("hi number {} from the main thread!", i);
        thread::sleep(Duration::from_millis(1));
    }
}
```

> 使用 join 等待线程执行完毕

```rust
use std::thread;
use std::time::Duration;

fn main() {
    let handle = thread::spawn(|| {
        for i in 1..10 {
            println!("hi number {} from the spawned thread!", i);
            thread::sleep(Duration::from_millis(1));
        }
    });

    for i in 1..5 {
        println!("hi number {} from the main thread!", i);
        thread::sleep(Duration::from_millis(1));
    }

    handle.join().unwrap();
}
```

> 线程中使用 move closures
```rust
// 错误代码
use std::thread;

fn main() {
    let v = vec![1, 2, 3];
    // 根据 闭包的捕获规则（是啥？）v会被 引用捕获。因为会新开线程，并不知道在当前线程 v后序会不会被搞，所以这里是会报错的
    let handle = thread::spawn(|| {
        println!("Here's a vector: {:?}", v);
    });

    handle.join().unwrap();
}
```
```rust
use std::thread;

fn main() {
    let v = vec![1, 2, 3];
    // 使用 move closure。直接发生所有权的移交。
    let handle = thread::spawn(move || {
        println!("Here's a vector: {:?}", v);
    });

    handle.join().unwrap();
}
```

## 线程之间的数据传输
这部分和golang的channel相似。`Do not communicate by sharing memory; instead, share memory by communicating.”`
* rust实现的是 `multiple producer, single consumer` 的 channel
* rx.send 的时候，会将 所有权也 send出去！
* 
```rust
use std::sync::mpsc;
use std::thread;
use std::time::Duration;

fn main() {
    let (tx, rx) = mpsc::channel();
    println!("spawning thread");
    thread::spawn(move || {
        let val = String::from("hi");
        thread::sleep(Duration::from_secs(3));
        tx.send(val).unwrap();
    });
    println!("spawning thread done");

    let received = rx.recv().unwrap(); //阻塞接收。也可以使用 try_recv(非阻塞！)
    println!("Got: {}", received);
}
```

* 构建 multiple producer
```rust
let (tx, rx) = mpsc::channel();

let tx1 = mpsc::Sender::clone(&tx); //构建 multiple producer 的核心
thread::spawn(move || {
    let vals = vec![
        String::from("hi"),
        String::from("from"),
        String::from("the"),
        String::from("thread"),
    ];

    for val in vals {
        tx1.send(val).unwrap();
        thread::sleep(Duration::from_secs(1));
    }
});

thread::spawn(move || {
    let vals = vec![
        String::from("more"),
        String::from("messages"),
        String::from("for"),
        String::from("you"),
    ];

    for val in vals {
        tx.send(val).unwrap();
        thread::sleep(Duration::from_secs(1));
    }
});

for received in rx {
    println!("Got: {}", received);
}
```

## 数据共享
> 上述的 channel 是数据共享的一种方式。现在介绍另一种方式。

* mutex：和 c++, go的mutex 不同。rust的mutex是个模板，还能存值？？？
```rust
use std::sync::Mutex;

fn main() {
    let m = Mutex::new(5);

    {
        let mut num = m.lock().unwrap(); // unwrap 得到的实际上是一个 MutexGuard. 可以通过该值操作 mutex里面的值。该 对象出了 scope 还会自动 unlock
        *num = 6;
    }

    println!("m = {:?}", m);
}
```

* 多线程如何共享 `Mutex` ?
通过智能指针`Rc<T>`，可以使得一个值拥有多个owner。但是`Rc<T>` 无法用在多线程场景下。在多线程场景下，使用`Arc<T>` 来代替 `Rc<T>`
```rust
use std::sync::{Arc, Mutex};
use std::thread;

fn main() {
    let counter = Arc::new(Mutex::new(0)); // Mutex<T> provides interior mutability
    let mut handles = vec![];

    for _ in 0..10 {
        let counter = Arc::clone(&counter);
        let handle = thread::spawn(move || { //这里依旧是 move closure。
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

## `Sync & Send` marker trait

> `send` & `sync` 是 marker trait，什么是 `marker trait` 呢？就是编译器负责做标记，rust用户无法干预的 trait。



`Send`: 如果一个类被标记了 `Send`，那么可以在**线程间传递其所有权**。

* 只有被标记了 `Send` 的类的对象在 线程间传递所有权时，编译器才不会报错。
* `Send` 标记保证了对象线程间传递所有权的正确性
* 所以在判断一个 类是不是 `Send` 的时候，就需要考虑其 在线程间传递所有权会不会有什么问题
  * 比如：`Rc<T>`. 线程间传递所有权就会导致 `reference count` 计算出现问题，所以就不是 `Send`
  * 但是像 `i32, char ...` 这些的，线程间移动也没得什么问题。因为不涉及什么需要同步的数据的修改。

一个常见的线程间传递所有权的场景为(和多线程共享 `Mutex` 使用同一份代码解释)：

```rust
use std::sync::{Arc, Mutex};
use std::thread;

fn main() {
    let counter = Arc::new(Mutex::new(0)); // Mutex<T> provides interior mutability
    let mut handles = vec![];

    for _ in 0..10 {
        let counter = Arc::clone(&counter);
      	// main线程 counter 的所有权 移交给了 spawn 的线程
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

> rust中，几乎所有类型都是 `Send` 。 但是有一些特殊情况，比如 `Rc<T>` 就不是 `Send`，这是因为当我们 `clone` 一个 `Rc<T>` 将其移动到另一个线程的时候，两个线程有可能同时修改其 `reference count`，这会导致 `data race`。 
>
> 所以，目前来看，包含 引用计数的类，如果想 `Send` 的话，那么其 引用计数的修改必须要同步的，比如 `Arc<T>`.
>
> `Send` 的标记规则：如果一个类的所有字段是 `Send`, 那么该类就是 `Send`. Almost all primitive types are `Send`, aside from raw pointers,



`Sync`: 如果一个类被标记了 `Sync` , 那么该类的对象可以被多个线程同时 引用( `Reference`). 比如：`Mutex<T>` 就可以被多个线程同时引用(`Arc<Mutex<T>>`) 。

* `Sync` 表示的是一个对象是否可以被多个线程同时访问。`Mutex<T>` 就可以，`Cell<T>, RefCell<T>` 就不行，他俩多线程访问，可能会出现 `data race` 问题。

* 如果 `Arc<T>` 是 `Send` ，那么 `T` 是 `Sync`。

* 


所以多线程共享变量的基本流程为：

1. 创建一个 `Sync` 对象，（想修改的话用 `Mutex`（不要用 `Cell, RefCell`）, 如果不想修改用原始值就OK了）
2. 然后将其封装到 `Arc` 中构建出一个 `Send`. 
3. 然后将其移动到不同的线程中。
4. 多线程就可以共享 `Sync` 对象了。



编码时常见问题解决方式：

* 如果编译器报错某 对象 不是 `Sync` ，可以套一个 `Mutex` 解决。
* 如果编译器报错 某对象不是 `Send`, 那就换一个 `Send` 的对象吧。
