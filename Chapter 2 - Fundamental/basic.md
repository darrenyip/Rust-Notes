### cargo

Rust 使用名为 cargo 的工具来管理项目，它类似 Node.js 的 npm、Golang 的 go，用来做依赖管理以及开发过程中的任务管理，比如编译、运行、测试、代码格式化等等。

- Rust 的变量默认是不可变的，如果要修改变量的值，需要显式地使用 mut 关键字。

- 除了 let / static / const / fn 等少数语句外，Rust 绝大多数代码都是表达式（expression）。所以 if / while / for / loop 都会返回一个值，函数最后一个表达式就是函数的返回值，这和函数式编程语言一致。

- Rust 支持面向接口编程和泛型编程。

- Rust 有非常丰富的数据类型和强大的标准库。Rust 有非常丰富的控制流程，包括模式匹配（pattern match）。

### 变量和函数

Rust 支持类型推导，在编译器能够推导类型的情况下，变量类型一般可以省略，但常量（const）和静态变量（static）必须声明类型。

定义变量的时候，根据需要，你可以添加 mut 关键字让变量具备可变性。**默认变量不可变**是一个很重要的特性，它符合最小权限原则（Principle of Least Privilege），有助于我们写出健壮且正确的代码。当你使用 mut 却没有修改变量，Rust 编译期会友好地报警，提示你移除不必要的 mut。



```rust
fn apply(value: i32, f: fn(i32) -> i32) -> i32 {
    f(value)
}
fn square(value: i32) -> i32 {
    value * value
}
fn cube(value: i32) -> i32 {
    value * value * value
}

fn main() {
    println!("apple square: {}", apply(2, square));
    println!("apple cube: {}", apply(2, cube))
}
```



这里 fn(i32) -> i32 是 apply 函数第二个参数的类型，它表明接受一个函数作为参数，这个传入的函数必须是：参数只有一个，且类型为 i32，返回值类型也是 i32。

Rust 函数参数的类型和返回值的类型**都必须显式定义**，**如果没有返回值可以省略**，返回 unit。**函数内部如果提前返回，需要用 return 关键字**，否则最后一个表达式就是其返回值。**如果最后一个表达式后添加了; 分号，隐含其返回值为 unit。**

### 数据结构

数据结构是程序的核心组成部分，在对复杂的问题进行建模时，我们就要自定义数据结构。Rust 非常强大，可以用 **struct** 定义结构体，用 **enum** 定义标签联合体（tagged union），**还可以像 Python 一样随手定义元组（tuple）类型。**

比如我们可以这样定义一个聊天服务的数据结构

```rust
#[derive(Debug)]
enum Gender {
    Unspecified = 0,
    Female = 1,
    Male = 2,
}

#[derive(Debug, Copy, Clone)]
struct UserId(u64);

#[derive(Debug, Copy, Clone)]
struct TopicId(u64);

#[derive(Debug)]
struct User {
    id: UserId,
    name: String,
    gender: Gender,
}

#[derive(Debug)]
struct Topic {
    id: TopicId,
    name: String,
    owner: UserId,
}

// 定义聊天室中可能发生的事件
#[derive(Debug)]
enum Event {
    Join((UserId, TopicId)),
    Leave((UserId, TopicId)),
    Message((UserId, TopicId, String)),
}

fn main() {
    let alice = User {
        id: UserId(1),
        name: "Alice".into(),
        gender: Gender::Female,
    };
    let bob = User {
        id: UserId(2),
        name: "Bob".into(),
        gender: Gender::Male,
    };

    let topic = Topic {
        id: TopicId(1),
        name: "rust".into(),
        owner: UserId(1),
    };
    let event1 = Event::Join((alice.id, topic.id));
    let event2 = Event::Join((bob.id, topic.id));
    let event3 = Event::Message((alice.id, topic.id, "Hello world!".into()));

    println!(
        "event1: {:?}, event2: {:?}, event3: {:?}",
        event1, event2, event3
    );
}

```

1. Gender：一个枚举类型，在 Rust 下，使用 enum 可以定义类似 C 的枚举类型
2. UserId/TopicId ：struct 的特殊形式，称为元组结构体。它的域都是匿名的，可以用索引访问，适用于简单的结构体。
3. User/Topic：标准的结构体，可以把任何类型组合在结构体里使用。
4. Event：标准的标签联合体，它定义了三种事件：Join、Leave、Message。每种事件都有自己的数据结构。



### 控制流程

顺序执行就是一行行代码往下执行。在执行的过程中，遇到函数，会发生函数调用。函数调用是代码在执行过程中，调用另一个函数，跳入其上下文执行，直到返回。Rust 的循环和大部分语言都一致，支持死循环 loop、条件循环 while，以及对迭代器的循环 for。循环可以通过 break 提前终止，或者 continue 来跳到下一轮循环。

```rust
fn fib_loop(n: u8) {
    let mut a = 1;
    let mut b = 2;
    let mut i = 2u8;

    loop {
        let c = a + b;
        a = b;
        b = c;
        i += 1;
        println!("loop next value: {:?}", b);
        if i >= n {
            break;
        }
    }
}
fn fib_while(n: u8) {
    let (mut a, mut b, mut i) = (1, 1, 2);
    while i < n {
        let c = a + b;
        a = b;
        b = c;
        i += 1;
        println!("next value: {:?}", b);
    }
}
fn fib_for(n: u8) {
    let (mut a, mut b) = (1, 1);
    for _i in 2..n {
        let c = a + b;
        a = b;
        b = c;
        println!("next value: {:?}", b);
    }
}
fn main() {
    let n = 10;
    fib_loop(n);
    fib_while(n);
    fib_for(n);
}

```

这里需要指出的是，**Rust 的 for 循环可以用于任何实现了 IntoIterator trait 的数据结构。**

在执行过程中，IntoIterator 会生成一个迭代器，for 循环不断从迭代器中取值，直到迭代器返回 None 为止。因而，**for 循环实际上只是一个语法糖**，编译器会将其展开使用 loop 循环对迭代器进行循环访问，直至返回 None。

### 模式匹配

Rust 的模式匹配吸取了函数式编程语言的优点，强大优雅且效率很高。它可以用于 struct / enum 中匹配部分或者全部内容，比如上文中我们设计的数据结构 Event，可以这样匹配

```rust

fn process_event(event: &Event) {
    match event {
        Event::Join((uid, _tid)) => println!("user {:?} joined", uid),
        Event::Leave((uid, tid)) => println!("user {:?} left {:?}", uid, tid),
        Event::Message((_, _, msg)) => println!("broadcast: {}", msg),
    }
}
```

从代码中我们可以看到，可以直接对 enum 内层的数据进行匹配并赋值，这比很多只支持简单模式匹配的语言，例如 JavaScript 、Python ，可以省出好几行代码。



除了使用 match 关键字做模式匹配外，我们还可以用 if let / while let 做简单的匹配，如果上面的代码我们只关心 Event::Message，可以这么写（代码）：

```rust

fn process_message(event: &Event) {
    if let Event::Message((_, _, msg)) = event {
        println!("broadcast: {}", msg);   
    }
}
```

Rust 的模式匹配是一个很重要的语言特性，被广泛应用在状态机处理、消息处理和错误处理中，如果你之前使用的语言是 C / Java / Python / JavaScript ，没有强大的模式匹配支持，要好好练习这一块。

### 错误处理

Rust 没有沿用 C++/Java 等诸多前辈使用的异常处理方式，而是借鉴 Haskell，把错误封装在 Result 类型中，同时提供了 ? 操作符来传播错误，方便开发。Result 类型是一个泛型数据结构，T 代表成功执行返回的结果类型，E 代表错误类型。