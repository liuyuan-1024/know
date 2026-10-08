+ `rustup`是一个管理 Rust 版本和相关工具的命令行工具。
+ `Cargo`是内置的依赖管理器和构建工具，它能轻松增加、编译和管理依赖，并使依赖在 Rust 生态系统中保持一致。
+ `Rustfmt`格式化工具确保开发者遵循一致的代码风格。
+ `rust-analyzer`为集成开发环境（IDE）提供了强大的代码补全和内联错误信息功能。

# Cargo
> Cargo 是 Rust 的构建系统和包管理器。大多数 Rustacean 们使用 Cargo 来管理他们的 Rust 项目，因为它可以为你处理很多任务，比如构建代码、下载依赖库并编译这些库。
>

```toml
[package]
name = "hello_cargo"
version = "0.1.0"
edition = "2024"

[dependencies]
```

## Cargo 基本命令
Cargo 期望源文件存放在 src 目录中。项目根目录只存放 README、license 信息、配置文件和其他跟代码无关的文件。

```bash
# 新建一个名为“hello_cargo”的rust项目
# cargo new 会初始化一个git仓库，并生成一个.gitignore文件；
# 如果在一个已经存在的git仓库中运行 cargo new 则这些git相关文件则不会生成；
cargo new hello_cargo

# --lib：创建一个hell_world的库项目
cargo new hello_world --lib

# 使用cargo管理一个已存在的Rust项目
cargo init # 创建 Cargo.toml 文件
```

```bash
# cargo build默认为调试（debug）构建，在 target/debug 目录下生成可执行文件，构建速度较快。为了开发，需要快速且频繁地重新构建。
cargo build
# cargo build --release 来优化编译项目。这会在 target/release 目录下生成可执行文件。这些优化可以让 Rust 代码运行的更快，不过启用这些优化也需要消耗更长的编译时间。
cargo build --release

# cargo run 执行前会增量编译项目（cargo build）
cargo run
# > 表明我们希望将标准输出流重定向到 output.txt 文件而非终端
cargo run > output.txt
# -p 运行指定的二进制包
cargo run -p crate名称

# 快速检查代码确保其可以编译，但并不产生可执行文件
cargo check

# 运行 test mod
cargo test
# 运行工作空间中所有的 test mod
cargo test --workspace
# 运行值的嗯 crate 的test mod
cargo test -p crate名称
```

`cargo install` 命令用于在本地安装和使用二进制 crate。它并不打算替换系统中的包；它意在作为一个方便 Rust 开发者们安装其他人已经在 [crates.io](https://crates.io/) 上共享的工具的手段。只有拥有二进制目标文件的包能够被安装。**二进制目标** 文件是在 crate 有 _src/main.rs_ 或者其他指定为二进制文件时所创建的可执行程序，这不同于自身不能执行但适合包含在其他程序中的库目标文件。通常 crate 的 _README_ 文件中有该 crate 是库、二进制目标还是两者兼有的信息。

所有来自 `cargo install` 的二进制文件都安装到 Rust 安装根目录的 _bin_ 文件夹中。如果你是使用 _rustup.rs_ 来安装 Rust 且没有自定义任何配置，这个目录将是 _$HOME/.cargo/bin_。确保将这个目录添加到 `$PATH` 环境变量中就能够运行通过 `cargo install` 安装的程序了。

```bash
cargo install 二进制crate名称
```

## Cargo 进阶
### Cargo 自定义扩展命令
Cargo 的设计使得开发者可以通过新的子命令来对 Cargo 进行扩展，而无需修改其本身。如果 `$PATH` 中有类似 `cargo-something` 的二进制文件，就可以通过 `cargo something` 来像 Cargo 子命令一样运行它。像这样的自定义命令也可以运行 `cargo --list` 来展示出来。能够通过 `cargo install` 向 Cargo 安装扩展并可以如内建 Cargo 工具那样运行它们是 Cargo 设计上的一个非常方便的优点！

### 自定义配置
Cargo 有两个主要的配置：运行 `cargo build` 时采用的 `dev` 配置和运行 `cargo build --release` 的 `release` 配置。`dev` 配置为开发定义了良好的默认配置，`release` 配置则为发布构建定义了良好的默认配置。

当项目的 _Cargo.toml_ 文件中没有显式增加任何 `[profile.*]` 部分的时候，Cargo 会对每一个配置都采用默认设置。通过增加任何希望定制的配置对应的 `[profile.*]` 部分，我们可以选择覆盖任意默认设置的子集。

更多配置项请参考：[**https://doc.rust-lang.org/cargo/reference/profiles.html**](https://doc.rust-lang.org/cargo/reference/profiles.html)

```toml
# 开发环境配置
[profile.dev]
opt-level = 0 # 控制 Rust 会对代码进行何种程度的优化。配置值：0～3。值越大编译时间越长。

# 发布环境配置
[profile.release]
opt-level = 3
```

### 发布 crate 到 Crates.io
> 我们曾经在项目中使用 [crates.io](https://crates.io) 上的包作为依赖，不过你也可以通过发布自己的包来向他人分享代码。[crates.io](https://crates.io) 上的 crate 注册表会分发你包的源代码，因此它主要托管开源代码。
>

> 发布 crate 时请多加小心，因为发布是**永久性的**（_permanent_）。对应版本不可能被覆盖，其代码也不可能被删除。[crates.io](https://crates.io) 的一个主要目标是作为一个存储代码的永久文档服务器，这样所有依赖 [crates.io](https://crates.io) 中的 crate 的项目都能一直正常工作。而允许删除版本没办法达成这个目标。然而，可以被发布的版本号却没有限制。
>

为了方便别人使用我们发布的 crate，编写有用的文档注释是相当有必要的。当然了，文档注释可以使用 markdown 语法。

```rust
/// Adds one to the number given.
///
/// # Examples
///
/// ```
/// let arg = 5;
/// let answer = my_crate::add_one(arg);
///
/// assert_eq!(6, answer);
/// ```
pub fn add_one(x: i32) -> i32 {
    x + 1
}
```

运行 `cargo doc` 来生成这个文档注释的 HTML 文档。这个命令运行由 Rust 分发的工具 `rustdoc` 并将生成的 HTML 文档放入 _target/doc_ 目录。为了方便起见，运行 `cargo doc --open` 会构建当前 crate 文档（同时还有所有 crate 依赖的文档）的 HTML 并在浏览器中打开。

在发布之前，你需要在 crate 的 _Cargo.toml_ 文件的 `[package]` 部分增加一些本 crate 的元数据（metadata）。

```toml
[package]
name = "guessing_game" # 在 crates.io 仓库中保持唯一
version = "0.1.0"
edition = "2024"
description = "A fun game where you guess what number the computer has chosen."
license = "MIT OR Apache-2.0" # 开源协议
```

访问位于 [crates.io](https://crates.io) 的首页并使用 GitHub 账号登录。登录之后，查看位于 [https://crates.io/me/](https://crates.io/me/) 的账户设置页面并获取 API token。

```bash
# 运行该命令，然后粘贴 API token
cargo login
abcdefghijklmnopqrstuvwxyz012345
```

```bash
# 发布
cargo publish
# 发布工作空间的指定 crate
catgo publish -p crate名称
```

虽然你不能删除 crate 的历史版本，但是可以阻止任何将来的项目将它们加入到依赖中。这在某个版本因为这样或那样的原因被破坏的情况很有用。

**撤回**某个版本会阻止新项目依赖此版本，不过所有现存此依赖的项目仍然能够下载和依赖这个版本。从本质上说，撤回意味着所有带有 _Cargo.lock_ 的项目的依赖不会被破坏，同时任何新生成的 _Cargo.lock_ 将不能使用被撤回的版本。

撤回功能并不能删除不小心上传的秘密信息。如果出现了这种情况，请立即重新设置这些秘密信息。

```bash
# 撤回 crate
cargo yank --vers 0.1.2

# 取消撤回 crate
cargo yank --vers 0.1.2 --undo
```

```bash
# clippy 做更全面的检查。Clippy 有各种设置或标志，而对于我们这些初学者来说，最合适的是 petantic，允许 clippy 提出所有可能的成语改进建议。
cargo clippy -- -W clippy::all  -W clippy::pedantic
```

## Cargo 工作空间
> 我们构建一个包含二进制 crate 和库 crate 的包。你可能会发现，随着项目开发的深入，库 crate 持续增大，而你希望将其进一步拆分成多个库 crate。Cargo 提供了一个叫**工作空间**（_workspaces_）的功能，它可以帮助我们管理多个相关的协同开发的包。
>

在工作空间目录下新建一个 Cargo.toml 文件

```toml
[workspace]
resolver = "3" # 在工作区中使用 Cargo 最新且最强大的解析算法
members = ["crate名称1", "crate名称2"] # 子包，在工作空间中运行 cargo run 会将新建包自动加入到工作空间中。
```

工作空间在顶级目录只有一个 _target_ 目录，用于存放编译生成的产物；子包并没有自己的 _target_ 目录。即使进入子包目录运行 `cargo build`，构建结果也位于 _**工作空间/target**_ 而非 **工作空间/子包/target**。工作空间中的 crate 之间相互依赖。如果每个 crate 有其自己的 _target_ 目录，为了在自己的 _target_ 目录中生成构建结果，工作空间中的每一个 crate 都不得不相互重新编译其他 crate。通过共享一个 _target_ 目录，工作空间可以避免其他 crate 重复构建。

# Rust 基础语法
## 预导入与导入
> 默认情况下，Rust 设定了若干个会自动导入到每个程序作用域中的标准库内容，这组内容被称为 _预导入（prelude）_ 内容。你可以在[标准库文档](https://doc.rust-lang.org/std/prelude/index.html)中查看预导入的所有内容。
>
> 如果你需要的类型不在预导入内容中，就必须使用`use`语句显式地将其引入作用域。
>

## 变量、常量和遮蔽
> 变量默认是不可改变的（immutable）。Rust 编译器保证，如果声明一个值不会变，它就真的不会变，所以你不必自己跟踪它。
>
> 尽管变量默认是不可变的，你仍然可以在变量名前添加 `mut` 来使其可变。
>

> _常量 (constants)_ 是绑定到一个名称的不允许改变的值.
>
> 不允许对常量使用 `mut`。常量不光默认不可变，它总是不可变。
>
> 声明常量使用 `const` 关键字而不是 `let`，并且 _必须_ 注明值的类型。
>
> 常量只能被设置为常量表达式，而不可以是其他任何只能在运行时计算出的值。
>

```rust
let x = 5;
let mut x = 5;
const x = 5 + 5;
```

> 定义一个与之前变量同名的新变量。Rustacean 们称之为第一个变量被第二个 **遮蔽（Shadowing）** 了，这意味着当您使用变量的名称时，编译器将看到第二个变量。实际上，第二个变量遮蔽了第一个变量，此时任何使用该变量名的行为中都会视为是在使用第二个变量，直到第二个变量自己也被遮蔽或第二个变量的作用域结束。可以用相同变量名称来遮蔽一个变量，以及重复使用 `let` 关键字来多次遮蔽。
>

> 遮蔽与将变量标记为 `mut` 是有区别的。当不小心尝试对变量重新赋值时，如果没有使用 `let` 关键字，就会导致编译时错误。通过使用 `let`，我们可以用这个值进行一些计算，不过计算完之后变量仍然是不可变的。
>
> `mut` 与遮蔽的另一个区别是，当再次使用 `let` 时，实际上创建了一个新变量，我们可以改变值的类型，并且复用这个名字。
>

```rust
fn main(){
    let x = 5;
    let x = x + 1;
    {
        let x = x * 2;
        println!("代码块中的 x = {x}");
    }
    println!("main函数体中的 x = {x}");
}
```

## 注释
```rust
// 单行注释，解释代码，增加代码易读性

/// 文档注释，它们会生成 HTML 文档。
/// 这些 HTML 展示公有 API 文档注释的内容，
/// 它们意在让对库感兴趣的程序员理解如何使用这个 crate，
/// 而不是它是如何被实现的。

//! 为“包含注释的项” 而不是 “位于注释之后的项” 来增加文档。这通常用于 crate 根文件（通常是 src/lib.rs）或模块的根文件为 crate 或模块整体提供文档。使用它们描述其容器整体的目的来帮助 crate 用户理解你的代码组织。
```

## 数据类型
> 在 Rust 中，每一个值都有一个特定 **数据类型**，这告诉 Rust 它被指定为何种数据，以便明确数据处理方式。我们将看到两类数据类型子集：标量（scalar）和复合（compound）。
>
> 记住，Rust 是 **静态类型**（_statically typed_）语言，也就是说在编译时就必须知道所有变量的类型。根据值及其使用方式，编译器通常可以推断出我们想要用的类型。当多种类型均有可能时，必须增加类型注解。
>

### 标量类型
| 整型 | 长度 | 有符号 | 无符号 |
| :---: | --- | --- | --- |
| | 8-bits | i8 | u8 |
| | 16-bits | i16 | u16 |
| | 32-bits | i32（默认） | u32 |
| | 64-bits | i64 | u64 |
| | 128-bits | i128 | u128 |
| | 架构相关 | isize | usize |


| 浮点型 | 长度 | 有符号 |
| :---: | --- | --- |
| | 32-bits | f32 |
| | 64-bits | f64（默认） |


| 布尔型 bool | 长度 | 值 |
| :---: | --- | --- |
| | 1-bits | true/false |


| 字符型 char | 长度 | 值 |
| :---: | --- | --- |
| | 32-bits（4-bytes） | 'Unicode 标量值' |


### 复合类型
| 元组 | 结构 | 解构 |
| :---: | --- | --- |
| | let tup: (i32, f64, char) = (1, 1.2, 'a'); | let (x, y, z) = tup; |


| 数组 | 结构 | 解构 |
| :---: | --- | --- |
| | let arr: [i32, 5] = [1, 2, 3, 4, 5]; | let a = arr[ 索引 ]; |


### 常用标准库类型
| 数据类型声明 | 结构 | 说明 |
| :---: | --- | --- |
| String | let s: String = String;;from("hello") | <!-- 这是一张图片，ocr 内容为： -->
![](https://cdn.nlark.com/yuque/0/2025/svg/43050341/1763521754120-ac52ca12-5f51-48ec-889e-1b572bdc4f18.svg) |
| &str | let s: &str = "hello"; | 字符串字面量 |


String 与 &str 的区别：String 类型的值存放在 **堆** 上；字符串字面量由于是不可变的固定大小，因此是存放在 **栈** 上的。

## 函数
+ **语句**（_Statements_）是执行一些操作但不返回值的指令。结尾有分号。
+ **表达式**（_Expressions_）计算并产生一个值。结尾没有分号。

> 函数调用是一个表达式。宏调用是一个表达式。用大括号创建的一个新的块作用域也是一个表达式。
>

```rust
# 
fn func(x: i32, y: f64) -> char{
    // 函数体

    // 显示返回
    return 'a';
    // 隐示返回，没有分号且为函数体最后一个语句
    'a' // 大部分函数隐式返回最后的表达式
}
```

## 控制流
```rust
// 条件语句
if(bool) {
    // bool = true 执行
} else if(bool){
    // bool = true 执行
} else {
    // 其余所有情况，均执行此代码快
}

// loop循环语句
let mut count = 0;
let result loop{
    count += 1;
    // 跳过循环迭代中的任何剩余代码，并转到下一个迭代。
    continue;
    // 停止循环，并返回表达式的值
    break count * 2;
}
// 循环标签，一般用于嵌套循环
'loop_label_1: loop{
    'loop_label_2: loop{
        if(count % 2 == 0) {
            break 'loop_label_1;
        } else {
            break 'loop_label_2;
        }
    }
}
// while条件循环
let mut number = 3;
while number != 0 {
    println!("{number}!");
    number -= 1;
}
// for循环
let arr = [10, 20, 30, 40, 50];
for element in arr {
    println!("the value is: {element}");
}

```

## 所有权
> **所有权**（_ownership_）是 Rust 用于如何管理内存的一组规则。所有程序都必须管理其运行时使用计算机内存的方式。一些语言中具有垃圾回收机制，在程序运行时有规律地寻找不再使用的内存；在另一些语言中，程序员必须亲自分配和释放内存。Rust 则选择了第三种方式：通过所有权系统管理内存，编译器在编译时会根据一系列的规则进行检查。如果违反了任何这些规则，程序都不能编译。在运行时，所有权系统的任何功能都不会减慢程序的运行。
>

> 所有权的主要目的就是管理堆数据。
>

### 栈（Stack）与堆（Heap）
> 在 Rust 这样的系统编程语言中，值是位于栈上还是堆上在更大程度上影响了语言的行为以及为何必须做出这样的抉择。
>

栈和堆都是代码在运行时可供使用的内存，但是它们的结构不同，因此存储目标也不同，栈存储固定大小的数据，堆存储不固定大小的数据。

栈：增加数据叫做 **入栈**（_pushing onto the stack_），而移出数据叫做 **出栈**（_popping off the stack_）。栈中的所有数据都必须占用已知且固定的大小。在编译时大小未知或大小可能变化的数据，要改为存储在堆上。

堆是缺乏组织的：当向堆放入数据时，你要请求一定大小的空间。内存分配器（memory allocator）在堆的某处找到一块足够大的空位，把它标记为已使用，并返回一个表示该位置地址的 **指针**（_pointer_）。这个过程称作 **在堆上分配内存**（_allocating on the heap_），有时简称为 “分配”（allocating）。（将数据推入栈中并不被认为是分配）。

因为指向放入堆中数据的指针是已知的并且大小是固定的，你可以将该指针存储在栈上，不过当需要实际数据时，必须访问指针。

入栈比在堆上分配内存要快，因为（入栈时）分配器无需为存储新数据去搜索内存空间；其位置总是在栈顶。相比之下，在堆上分配内存则需要更多的工作，这是因为分配器必须首先找到一块足够存放数据的内存空间，并接着做一些记录为下一次分配做准备。

访问堆上的数据比访问栈上的数据慢，因为必须通过指针来访问。现代处理器在内存中跳转越少就越快。

当你的代码调用一个函数时，传递给函数的值（包括可能指向堆上数据的指针）和函数的局部变量被压入栈中。当函数结束时，这些值被移出栈。

### 所有权规则
> 所有权是针对 **值** 的，而非其他的什么东西。值的所有权在变量和函数之间流转。
>

1. Rust 中的每一个值都有一个 **所有者**（_owner_）。
2. 值在任一时刻有且只有一个所有者。
3. 当所有者离开作用域，这个值将被丢弃。

> 作用域：就是 **{ }** 包裹的区域
>
> 当变量离开作用域后，Rust 自动调用 `drop` 函数并清理变量的堆内存
>

当两个指针指向同一块堆内存，Rust 认为前一个指针不再有效，当前一个指针离开作用域时， Rust 将不再进行任何清理操作。

```rust
let s1 = String::from("hello");
let s2 = s1;
println!("{s1}, world!");

# 你将得到类似如下的错误
$ cargo run
Compiling ownership v0.1.0 (file:///projects/ownership)
                            error[E0382]: borrow of moved value: `s1`
                            --> src/main.rs:5:15
                            |
                            2 |     let s1 = String::from("hello");
                            |         -- move occurs because `s1` has type `String`, which does not implement the `Copy` trait
                            3 |     let s2 = s1;
                            |              -- value moved here
                            4 |
                            5 |     println!("{s1}, world!");
                            |               ^^^^ value borrowed here after move
                            |
                            = note: this error originates in the macro `$crate::format_args_nl` which comes from the expansion of the macro `println` (in Nightly builds, run with -Z macro-backtrace for more info)
                            help: consider cloning the value if the performance cost is acceptable
                            |
                            3 |     let s2 = s1.clone();
                            |                ++++++++

                            For more information about this error, try `rustc --explain E0382`.
                            error: could not compile `ownership` (bin "ownership") due to 1 previous error
                            # 这是Rust在禁止你使用无效的引用
```

其他语言中有 **浅拷贝**（_shallow copy_）和 **深拷贝**（_deep copy_），但是 Rust 在“浅拷贝”过程中，会使第一个变量无效，因此这个操作被称为 **移动（move）** ，而不是浅拷贝。也就是在移动之后，前一个变量就自动失效了。对应的，深拷贝在 Rust 中称为 **复制** 。

| Rust | 其他语言 |
| --- | --- |
| 移动（会使前一个变量无效） | 浅拷贝 |
| 复制（无任何影响） | 深拷贝 |


所有的标量类型数据由于是固定大小的，会存放在栈上，也就是说不会分配内存（分配内存特指堆），因此，标量类型数据都是“深拷贝”，会复制其值到栈上，而不是使前一个变量无效。

这里还隐含了一个设计选择：Rust 永远也不会自动创建数据的 “深拷贝”。因此，任何**自动**的复制都可以被认为是对运行时性能影响较小的。

> 变量的所有权总是遵循相同的模式：将值赋给另一个变量时，所有权会移动；将值传入函数时，所有权会移动。当持有堆中数据值的变量离开作用域时，其值将通过 `drop` 被清理掉，除非数据被移动为另一个变量所有。
>

## 引用与借用
> **引用**（_reference_）像一个指针，因为它是一个地址，我们可以由此访问储存于该地址的属于其他变量的数据。与指针不同，引用在其生命周期内保证指向某个特定类型的有效值。引用的值，在引用结束后，也不会丢弃，因为值的所有权并没有转移。
>

> 引用的作用域从声明的地方开始一直持续到最后一次使用为止。
>

```rust
fn main() {
    let s1 = String::from("hello");

    let len = calculate_length(&s1);

    println!("The length of '{s1}' is {len}.");
}

# &数据类型，表示引用该数据类型变量的值，而非获取该值的所有权。
fn calculate_length(s: &String) -> usize {
    s.len()
}
```

> 我们将创建一个引用的行为称为 **借用**（_borrowing_）。这将意味着，我们无法修改借用的值。
>
> 如果我们需要修改借用的值呢？
>
> **可变引用** 
>
> 1. 定义参数时，必须标明是可变引用 &mut
> 2. 值的所有者必须是 mut 的
> 3. 传入该值时，必须标明是可变引用 &mut
>
> **注意：**如果你有一个对该变量的可变引用，你就不能再创建对该变量的引用。这样，Rust 保证了在编译时就避免数据竞争。同样的，不可以同时存在对同一个变量的不可变引用和可变引用，不可变引用的借用者可不希望在借用时值会突然发生改变！但多个不可变引用是可以的，因为没有哪个只能读取数据的引用者能够影响其他引用者读取到的数据。
>
> **数据竞争**（_data race_）
>
> + 两个或更多指针同时访问同一数据。
> + 至少有一个指针被用来写入数据。
> + 没有同步数据访问的机制。
>
> 当然，我们可以使用大括号来创建一个新的作用域，以拥有多个不可变引用和可变引用，只是不能同时拥有而以。
>

```rust
fn main() {
    let mut s = String::from("hello");

    change(&mut s);
}

fn change(some_string: &mut String) {
    some_string.push_str(", world");
}
```

### 悬垂引用
在具有指针的语言中，很容易通过释放内存时保留指向它的指针而错误地生成一个**悬垂指针**（_dangling pointer_）—— 指向可能已被分配给其他用途的内存位置的指针。相比之下，在 Rust 中编译器确保引用永远也不会变成悬垂引用：当你拥有一些数据的引用，编译器确保数据不会在其引用之前离开作用域。

```rust
fn main() {
    let reference_to_nothing = dangle();
}

fn dangle() -> &String { // dangle 返回一个字符串的引用

    let s = String::from("hello"); // s 是一个新字符串

    &s // 返回字符串 s 的引用
} // 这里 s 离开作用域并被丢弃。其内存被释放。危险！
```

### 引用的规则
+ 在任意给定时间，**要么**只能有一个可变引用，**要么**只能有多个不可变引用。
+ 引用必须总是有效的。

## Slice 类型（另一种引用）
**切片**（_slice_）允许你引用集合中一段连续的元素序列，而不用引用整个集合。slice 是一种引用，所以它不拥有所有权。

### 字符串 slice（string slice）
> **字符串 slice **是 `String` 中一部分值的引用，类型声明写作：&str
>

```rust
let s = String::from("hello world");
// [starting_index..ending_index]
// starting_index 是 slice 的第一个位置，
// ending_index 则是 slice 最后一个位置的后一个值。
// 两个索引都可以缺省，意味着从0到字符串长度。
let hello = &s[0..5];
let world = &s[6..11];
```

<!-- 这是一张图片，ocr 内容为： -->
![](https://cdn.nlark.com/yuque/0/2025/svg/43050341/1763525755856-a051b18e-a659-4eb8-b7d7-a709ce61cb71.svg)

### 其他类型的 slice
```rust
let a = [1, 2, 3, 4, 5];

let slice: &[i32] = &a[1..3];

assert_eq!(slice, &[2, 3]);
```

## 结构体
> **结构体**（_struct_），或者 _structure_，是一个自定义数据类型，允许你包装和命名多个相关的值，从而形成一个有意义的组合。
>

定义结构体，需要使用 `struct` 关键字并为整个结构体提供一个名字。结构体的名字需要描述它所组合的数据的意义。

```rust
struct User {
    active: bool,
    username: String,
    email: String,
    sign_in_count: u64,
}
```

一旦定义了结构体后，为了使用它，通过为每个字段指定具体值来创建这个结构体的**实例**。创建一个实例需要以结构体的名字开头，接着在大括号中使用 `key: value` 键 - 值对的形式提供字段，其中 key 是字段的名字，value 是需要存储在字段中的数据值。实例中字段的顺序不需要和它们在结构体中声明的顺序一致。

```rust
let mut user1 = User {
    active: true,
    username: String::from("someusername123"),
    email: String::from("someone@example.com"),
    sign_in_count: 1,
};
// 为了从结构体中获取某个特定的值，可以使用点号。
user1.email = String::from("anotheremail@example.com");
// 注意，我将整个实例都标注为可变的，Rust 并不允许只将某个字段标记为可变。
```

使用 **字段初始化简写语法**（_field init shorthand_）来重写 `build_user`

```rust
// 重写前
fn build_user(email: String, username: String) -> User {
    User {
        active: true,
        username: username,
        email: email,
        sign_in_count: 1,
    }
}
// 重写后
fn build_user(email: String, username: String) -> User {
    User {
        active: true,
        username,
        email,
        sign_in_count: 1,
    }
}
```

**结构体更新语法**（_struct update syntax_）实现使用旧实例的大部分值但改变其部分值来创建一个新的结构体实例。

```rust
// 未使用结构体更新语法
let user2 = User {
    active: user1.active,
    username: user1.username,
    email: String::from("another@example.com"),
    sign_in_count: user1.sign_in_count
};
// 使用结构体更新语法
// ..user1 必须放在最后，以指定其余的字段应从 user1 的相应字段中获取其值，但我们可以选择以任何顺序为任意字段指定值，而不用考虑结构体定义中字段的顺序。
let user2 = User {
    email: String::from("another@example.com"),
    ..user1
};
// 注意：结构体更新语法移动了实例的属性的值，因此会导致非标量数据类型的属性无效。
```

定义与元组类似的结构体，称为 **元组结构体**（_tuple structs_）。元组结构体有着结构体名称提供的含义，但没有具体的字段名，只有字段的类型。

```rust
struct Color(i32, i32, i32);
struct Point(i32, i32, i32);

fn main() {
    let black = Color(0, 0, 0);
    let origin = Point(0, 0, 0);
}
```

定义一个没有任何字段的结构体！它们被称为 **类单元结构体**（_unit-like structs_）因为它们类似于 `()`，即“元组类型”一节中提到的 unit 类型。类单元结构体常常在你想要在某个类型上实现 trait 但不需要在类型中存储数据的时候发挥作用。

```rust
struct AlwaysEqual;

fn main() {
    let subject = AlwaysEqual;
}
```

### 方法
**方法**（method）与函数类似：它们使用 `fn` 关键字和名称声明，可以拥有参数和返回值，同时包含在某处调用该方法时会执行的代码。不过方法与函数是不同的，因为它们在结构体的上下文中被定义（或者是枚举或 trait 对象的上下文），并且它们第一个参数总是 `self`，它代表调用该方法的结构体实例。

```rust
#[derive(Debug)]
struct Rectangle {
    width: u32,
    height: u32,
}

// impl 块中的所有内容都将与 Rectangle 类型相关联。
impl Rectangle {
    fn area(&self) -> u32 {
        self.width * self.height
    }

    fn can_hold(&self, other: &Rectangle) -> bool {
        self.width > other.width && self.height > other.height
    }
}

fn main() {
    let rect1 = Rectangle {
        width: 30,
        height: 50,
    };

    println!(
        "The area of the rectangle is {} square pixels.",
        rect1.area()
    );
}
```

每个结构体都允许拥有多个 `impl` 块。第十章讨论泛型和 trait 时会看到实用的多 `impl` 块的用例。

### 关联函数
所有在 `impl` 块中定义的函数被称为 **关联函数**（_associated functions_），因为它们与 `impl` 后面命名的类型相关。我们可以定义不以 `self` 为第一参数的关联函数（因此不是方法），因为它们并不作用于一个结构体的实例。比如，在 `String` 类型上定义的 `String::from` 函数。

```rust
impl Rectangle {
    fn square(size: u32) -> Self {
        Self {
            width: size,
            height: size,
        }
    }
}
// 调用关联函数，::语法用于关联函数和模块创建的命名空间。
let r: Rectangle = Rectangle::square(3);
```

#### 方法和关联函数的联系与区别
+ 联系：方法是关联函数的子集，即方法是特殊的关联函数。
+ 区别：关联函数与类型是强绑定的，通过 `类型::关联函数()` 来调用关联函数；  
	   方法与类型实例是强绑定的，通过 `实例::方法()` 来调用方法。

## 枚举和模式匹配
> **枚举**（_enumerations_），也被称作 _enums_。枚举允许你通过列举可能的 **变体**（_variants_）来定义一个类型。一个特别有用的枚举，叫做 `Option`，它代表一个值要么是某个值要么什么都不是。`match` 表达式中用模式匹配。
>

> 结构体给予你将字段和数据聚合在一起的方法，像 `Rectangle` 结构体有 `width` 和 `height` 两个字段。而枚举给予你一个途径去声明某个值是一个集合中的一员。
>

```rust
enum IpAddrKind {
    V4,
    V6,
}
// 注意：枚举的变体位于其标识符的命名空间中，并使用两个冒号分开。这么设计的益处是：IpAddrKind::V4 和 IpAddrKind::V6 都是 IpAddrKind 类型的。
let four = IpAddrKid::V4;
let five = IpAddrKid::V5;

// 枚举的变体可以关联值
// 每一个我们定义的枚举变体的名字也变成了一个构建枚举的实例的函数。也就是说，IpAddr::V4() 是一个获取 String 参数并返回 IpAddr 类型实例的函数调用。作为定义枚举的结果，这些构造函数会自动被定义。
// 用枚举替代结构体还有另一个优势：每个变体可以处理不同类型和数量的数据。
enum IpAddr {
    V4(String),
    V6(String),
}
let home = IpAddr::V4(String::from("127.0.0.1"));
let loopback = IpAddr::V6(String::from("::1"));

//
enum Message {
    Quit,
    Move { x: i32, y: i32 },
    Write(String),
    ChangeColor(i32, i32, i32),
}
```

可以像使用 `impl` 来为结构体定义方法那样，在枚举上定义方法。

```rust
impl Message {
    fn call(&self) {
        // 在这里定义方法体
    }
}

let m = Message::Write(String::from("hello"));
m.call();
```

### Option 枚举
> `Option` 类型应用广泛因为它编码了一个非常普遍的场景，即一个值要么有值要么没值。
>
> Rust 中没有 Null 的概念，而是采用 Option 枚举来表达空与非空。
>

```rust
enum Option<T> {
    None,
    Some(T),
}
```

### match 表达式
> `match` 表达式就是这么一个处理枚举的控制流结构：它会根据枚举的变体运行不同的代码，这些代码可以使用匹配到的值中的数据。
>

```rust
enum Coin {
    Penny,
    Nickel,
    Dime,
    Quarter,
}

fn value_in_cents(coin: Coin) -> u8 {
    match coin {
        Coin::Penny => 1,
        Coin::Nickel => 5,
        Coin::Dime => 10,
        Coin::Quarter => 25,
    }
}
```

当 `match` 表达式执行时，它将结果值按顺序与每一个分支的模式相比较。如果模式匹配了这个值，这个模式相关联的代码将被执行。如果模式并不匹配这个值，将继续执行下一个分支，可以拥有任意多的分支。

每个分支相关联的代码是一个表达式，而表达式的结果值将作为整个 `match` 表达式的返回值。

如果分支代码较短的话通常不使用大括号。如果想要在分支中运行多行代码，可以使用大括号，而分支后的逗号是可选的。

> 匹配分支的另一个有用的功能是可以绑定匹配的模式的部分值。这也就是如何从枚举变体中提取值的。
>

```rust
#[derive(Debug)] // 这样可以立刻看到州的名称
enum UsState {
    Alabama,
    Alaska,
}

enum Coin {
    Penny,
    Nickel,
    Dime,
    Quarter(UsState),
}
// 如果调用 value_in_cents(Coin::Quarter(UsState::Alaska))，coin 将是 Coin::Quarter(UsState::Alaska)。当将值与每个分支相比较时，没有分支会匹配，直到遇到 Coin::Quarter(state)。这时，state 绑定的将会是值 UsState::Alaska。接着就可以在 println! 表达式中使用这个绑定了，像这样就可以获取 Coin 枚举的 Quarter 变体中内部的州的值。
fn value_in_cents(coin: Coin) -> u8 {
    match coin {
        Coin::Penny => 1,
        Coin::Nickel => 5,
        Coin::Dime => 10,
        Coin::Quarter(state) => {
            println!("State quarter from {state:?}!");
            25
        }
    }
}

```

### 匹配 Option<T>
```rust
fn plus_one(x: Option<i32>) -> Option<i32> {
    match x {
        None => None,
        Some(i) => Some(i + 1),
    }
}

let five = Some(5);
let six = plus_one(five);
let none = plus_one(None);
```

### 匹配是有穷尽的
Rust 知道我们没有覆盖所有可能的情况甚至知道哪些模式被忘记了！Rust 中的匹配是 **穷尽的**（_exhaustive_）：必须穷举到最后的可能性来使代码有效。

### 通配模式和 _ 占位符
> 使用枚举，我们也可以针对少数几个特定值执行特殊操作，而对其他所有值采取默认操作。
>

```rust
let dice_roll = 9;
match dice_roll {
    3 => add_fancy_hat(),
    7 => remove_fancy_hat(),
    // 必须将通配分支放在最后，因为模式是按顺序匹配的。
    other => move_player(other),
    // 匹配任意值但不绑定到该值，这告诉 Rust 我们不会使用这个值，所以 Rust 也不会警告我们存在未使用的变量。
    _ => reroll();
}

fn add_fancy_hat() {}
fn remove_fancy_hat() {}
fn move_player(num_spaces: u8) {}
```

### if let 和 let else 简洁控制流
> `if let` 语法让我们以一种不那么冗长的方式结合 `if` 和 `let`，来处理只匹配一个模式的值而忽略其他模式的情况。可以认为 `if let` 是 `match` 的一个语法糖，它当值匹配某一模式时执行代码而忽略所有其他值。
>

```rust
let mut count = 0;
match coin {
    Coin::Quarter(state) => println!("State quarter from {state:?}!"),
    _ => count += 1,
}

let mut count = 0;
if let Coin::Quarter(state) = coin {
    println!("State quarter from {state:?}!");
} else {
    count += 1;
}
```

```rust
fn describe_state_quarter(coin: Coin) -> Option<String> {
    let state = if let Coin::Quarter(state) = coin {
        state
    } else {
        return None;
    };

    if state.existed_in(1900) {
        Some(format!("{state:?} is pretty old, for America!"))
    } else {
        Some(format!("{state:?} is relatively new."))
    }
}

fn describe_state_quarter(coin: Coin) -> Option<String> {
    let Coin::Quarter(state) = coin else {
        return None;
};

if state.existed_in(1900) {
    Some(format!("{state:?} is pretty old, for America!"))
} else {
    Some(format!("{state:?} is relatively new."))
}
}
```

## 包、Crate 和模块
Rust 有许多功能可以让你管理代码的组织，包括哪些细节可以被公开，哪些细节作为私有部分，以及程序中各个作用域中有哪些名称。这些特性，有时被统称为 “模块系统（the module system）”，包括：

+ **包**（_Packages_）：Cargo 的一个功能，它允许你构建、测试和分享 crate。
+ **Crates** ：一个模块的树形结构，它形成了库或可执行文件项目。
+ **模块**（_Modules_）和 **use**：允许你控制作用域和路径的私有性。
+ **路径**（_path_）：一个为例如结构体、函数或模块等项命名的方式。

### 包和 Crate
crate 是 Rust 在编译时最小的代码单位。即使你用 `rustc` 而不是 `cargo` 来编译一个单独的源代码文件，编译器还是会将那个文件视为一个 crate。crate 可以包含模块，模块可以定义在其他文件，然后和 crate 一起编译。

crate 有两种形式：二进制 crate 和库 crate。**二进制 crate**（_Binary crates_）可以被编译为可执行程序，比如命令行程序或者服务端。它们必须有一个名为 `main` 函数来定义当程序被执行的时候所需要做的事情。

**库 crate**（_Library crates_）并没有 `main` 函数，它们也不会编译为可执行程序。相反它们定义了可供多个项目复用的功能模块。这与其他编程语言中 “library” 概念一致。

**crate root** 是一个源文件，Rust 编译器以它为起始点，并构成你的 crate 的根模块。



_包_（_package_）是提供一系列功能的一个或者多个 crate 的捆绑。一个包会包含一个 _Cargo.toml_ 文件，阐述如何去构建这些 crate。Cargo 实际上就是一个包，它包含了用于构建你代码的命令行工具的二进制 crate。其他项目也依赖 Cargo 库来实现与 Cargo 命令行程序一样的逻辑。

包中可以包含至多一个库 crate(library crate)。包中可以包含任意多个二进制 crate(binary crate)，但是必须至少包含一个 crate（无论是库的还是二进制的）。

### 模块
**模块**让我们可以将一个 crate 中的代码进行分组，以提高可读性与重用性。因为一个模块中的代码默认是私有的，所以还可以利用模块控制项的**私有性**（_privacy_）。私有项是不可为外部使用的内在详细实现。我们也可以将模块和它其中的项标记为公开的，这样，外部代码就可以使用并依赖于它们。

```rust
mod front_of_house {
    mod hosting {
        fn add_to_waitlist() {}

        fn seat_at_table() {}
    }

    mod serving {
        fn take_order() {}

        fn serve_order() {}

        fn take_payment() {}
    }
}
```

为了向 Rust 指示在模块树中从何处查找某个项，我们使用路径，就像在文件系统中使用路径一样。为了调用一个函数，我们需要知道它的路径。

+ **绝对路径**（_absolute path_）是以 crate 根（root）开头的完整路径；对于外部 crate 的代码，是以 crate 名开头的绝对路径，对于当前 crate 的代码，则以字面值 `crate` 开头。
+ **相对路径**（_relative path_）从当前模块开始，以 `self`、`super` 或当前模块中的某个标识符开头。

```rust
mod father {
    mod child {
        fn child_fn() {
            // 绝对路径
            crate::father::father_fn();

            // 相对路径，super相当于进入父模块
            super::father_fn();
        }
    }

    fn father_fn(){}
}
```

```rust
mod front_of_house {
    // 模块上的 pub 只允许其父模块引用它，而不允许访问内部代码。
    pub mod hosting {
        // 函数前的 pub 允许外部访问
        pub fn add_to_waitlist() {}
    }
}
```

如果我们在一个结构体定义的前面使用了 `pub`，这个结构体会变成公有的，但是这个结构体的字段仍然是私有的。我们可以根据情况决定每个字段是否公有。

```rust
mod back_of_house {
    pub struct Breakfast {
        pub toast: String,
        seasonal_fruit: String,
    }

    impl Breakfast {
        pub fn summer(toast: &str) -> Breakfast {
            Breakfast {
                toast: String::from(toast),
                seasonal_fruit: String::from("peaches"),
            }
        }
    }
}
```

### use
> 无论我们选择函数的绝对路径还是相对路径，每次我们想要调用该函数时，都必须指定其父模块。我们可以使用 `use` 关键字创建一个捷径，然后就可以在作用域中的任何地方使用这个更短的名字。如果希望将一个路径下**所有**公有项引入作用域，可以指定路径后跟 `*` glob 运算符。
>

> use 类似文件系统的软连接（符号连接，symbolic link）。
>

```rust
mod front_of_house {
    pub mod hosting {
        pub fn add_to_waitlist() {}
    }
}

use crate::front_of_house::hosting;
// 一般只在测试模块 tests 中使用
use std::collections::*;

// 正确捷径
pub fn eat_at_restaurant() {
    hosting::add_to_waitlist();
}

// 错误捷径，use 只能创建 use 所在的特定作用域内的捷径。
mod customer {
    pub fn eat_at_restaurant() {
        hosting::add_to_waitlist();
    }
}
```

使用 `use` 将两个同名类型引入同一作用域的解决办法

```rust
// 第一种：引入其父模块（父模块的名称不同）
use std::fmt;
use std::io;

fn function1() -> fmt::Result {}

fn function2() -> io::Result<()> {}

// 第二种：使用 as 关键字设置别名
use std::fmt::Result;
use std::io::Result as IoResult;

fn function1() -> Result {}

fn function2() -> IoResult<()> {}
```

使用 `use` 关键字，将某个名称导入当前作用域后，该名称对此作用域之外还是私有的。若要让作用域之外的代码能够像在当前作用域中一样使用该名称，可以将 `pub` 与 `use` 组合使用。这种技术被称为**重导出**（_re-exporting_），因为在把某个项目导入当前作用域的同时，也将其暴露给其他作用域。

```rust
mod front_of_house {
    pub mod hosting {
        pub fn add_to_waitlist() {}
    }
}

pub use crate::front_of_house::hosting;

pub fn eat_at_restaurant() {
    hosting::add_to_waitlist();
}
```

多数时候，我们都需要使用外部依赖

```rust
# Cargo.toml
[dependencies] # 外部依赖，Cargo会从crates.io上下载对应依赖及其所需依赖
rand = "0.8.5"
```

```rust
use rand::Rng;

fn main() {
    let secret_number = rand::thread_rng().gen_range(1..=100);
}
```

## 常见集合
> Rust 标准库中包含一系列被称为 **集合**（_collections_）的非常有用的数据结构。大部分其他数据类型都代表一个特定的值，不过集合可以包含多个值。不同于内建的数组和元组类型，这些集合指向的数据是储存在堆上的，这意味着数据的数量不必在编译时就已知，并且还可以随着程序的运行增长或缩小。
>

### 向量（vector）
> 类型声明：`Vec<T>`，也被称为 _vector_。
>
> vector 允许我们在一个单独的数据结构中储存多于一个的值，它在内存中彼此相邻地排列所有的值。vector 只能储存相同类型的值。
>

```rust
// 定义vector变量
let v: Vec<i32> = Vec::new();
let v = vec![1, 2, 3];
```

```rust
// 更新vector
let mut v: Vec<i32> = Vec::new();
v.push(4);
```

```rust
// 读取vector
let v = vec![1, 2, 3, 4, 5];
let third: &i32 = &v[2];
println!("The third element is {third}");
let third: Option<&i32> = v.get(2);
match third {
    Some(third) => println!("The third element is {third}"),
    None => println!("There is no third element."),
}
```

```rust
let mut v = vec![100, 32, 57];
for i in &v {
    println!("{i}");
}
for i in &mut v {
    // 为了修改可变引用所指向的值，在使用 += 运算符之前必须使用解引用运算符（*）获取 i 中的值。
    *i += 50;
}
```

当 vector 被丢弃时，所有其内容也会被丢弃，这意味着这里它包含的整数将被清理。借用检查器确保了任何 vector 中内容的引用仅在 vector 本身有效时才可用。

现在我们了解了一些使用 vector 的最常见的方式，请一定去看看标准库中 `Vec` 定义的很多其他实用方法的 [API 文档](https://doc.rust-lang.org/std/vec/struct.Vec.html)。

### 字符串（String）
> Rust 的核心语言中只有一种字符串类型，字符串 slice `&str` ，它通常以被借用的形式出现。
>
> 字符串（`String`）类型由 Rust 标准库提供，而不是编入核心语言，它是一种可增长、可变、可拥有、**UTF-8 编码** 的字符串类型。
>
> 当提及 Rust 中的 "字符串 "时，指的可能是 `String` 或 `&str` 类型，而不仅仅是其中一种类型。
>

很多 `Vec<T>` 上可用的操作在 `String` 中同样可用，事实上 `String` 被实现为一个带有一些额外保证、限制和功能的字节 vector 的封装。

```rust
let s: &str = "hellp";
let s: String = String::from("hell0, world");
let s: String = "hell0, world".to_string();
```

```rust
let s1 = String::from("Hello, ");
let s2 = String::from("world!");
let s3 = s1 + &s2; // 注意 s1 被移动了，不能继续使用

// + 运算符使用了add函数，第一个参数是自身，会获取其所有权，第二个参数是引用，不会获取其所有权
fn add(self, s: &str) -> String {}
```

```rust
let s1 = String::from("Hello, ");
let s2 = String::from("world!");
// format宏函数不会获取任何参数的所有权
let s = format!("{}_{}", s1, s2);
```

Rust 的字符串不支持索引。

原因在于 Rust 的 `String` 是一个 `Vec<u8>` 的封装。UTF-8 编码占用一个字节，Unicode 标量值需要两个字节存储。因此一个字符串字节值的索引并不总是对应一个有效的 Unicode 标量值。当我们使用索引字符串时，希望返回该索引位置的字符，但 Rust 只能提供在该索引位置的字节值，这并不是开发者想看到的，因此 Rust 不会编译这些代码。

相比使用 `[]` 和单个值的索引，可以使用 `[]` 和一个 range 来创建含特定字节的字符串 slice：

```rust
let hello = "Здравствуйте";

// s = Зд，四个字节
let s = &hello[0..4];
```

操作字符串每一部分的最好的方法是明确表示需要字符还是字节。对于单独的 Unicode 标量值使用 `chars` 方法。`bytes` 方法返回字符串的每一个原始字节。

```rust
let hello = "Здравствуйте";
for c in hello.chars() {
    println!("{}", c);
}
for c in hello.bytes() {
    println!("{}", c);
}
```

### HashMap
> 类型声明：HashMap<K, V>
>
> 储存了一个键类型 `K` 对应一个值类型 `V` 的映射。它通过一个**哈希函数**（_hashing function_）来实现映射，决定如何将键和值放入内存中。
>

```rust
use std::collections::HashMap;

// 新建哈希表
let mut map = HashMap::<String, i32>::new();
// 插入键值对，当键已存在时更新值
map.insert("hello".to_string(), 10);
// 仅当键不存在时插入值，否则返回值
map.entry(String::from("world")).or_insert(20);
// 获取引用值：Option<&i32>
let v_opt_quote = map.get("hello");
// 复制值：Option<i32>
let v_opt = v_opt_quote.copied();
// 获取具体值：i32，当值不存在时返回默认值0
let v = v_opt.unwrap_or(0);
```

对于像 `i32` 这样的实现了 `Copy` trait 的类型，其值可以拷贝进哈希 map。对于像 `String` 这样拥有所有权的值，其值将被移动而哈希 map 会成为这些值的所有者。

```rust
use std::collections::HashMap;

let field_name = String::from("Favorite color");
let field_value = String::from("Blue");

let mut map = HashMap::new();
map.insert(field_name, field_value);
// 这里 field_name 和 field_value 不再有效，
// 尝试使用它们看看会出现什么编译错误！
```

## 错误处理
> 错误是软件开发中不可避免的事实，所以 Rust 有一些处理出错情况的特性。 
>

> Rust 将错误分为两大类：**可恢复的**（_recoverable_）和 **不可恢复的**（_unrecoverable_）错误。对于一个可恢复的错误，比如文件未找到的错误，我们很可能只想向用户报告问题并重试操作。不可恢复的错误总是 bug 出现的征兆，比如试图访问一个超过数组末端的位置，因此我们要立即停止程序。
>

> 大多数语言并不区分这两种错误，并采用类似异常（exception）这样方式统一处理它们。Rust 没有异常。相反，它有 `Result<T, E>` 类型，用于处理可恢复的错误，还有 `panic!` 宏，在程序遇到不可恢复的错误时停止执行。
>

### panic!处理不可恢复错误
> 在实践中有两种方法造成 panic：执行会造成代码 panic 的操作（比如访问超过数组结尾的内容）或者显式调用 `panic!` 宏。这两种情况都会使程序 panic。通常情况下这些 panic 会打印出一个错误信息，展开并清理栈数据，然后退出程序。
>

当出现 panic 时，程序默认会开始 **展开**（_unwinding_），这意味着 Rust 会回溯栈并清理它遇到的每一个函数的数据，不过这个回溯并清理的过程有很多工作。另一种选择是直接 **终止**（_abort_），这会不清理数据就退出程序。

那么程序所使用的内存需要由操作系统来清理。如果你需要项目的最终二进制文件越小越好，panic 时通过在 _Cargo.toml_ 的 `[profile]` 部分增加 `panic = 'abort'`，可以由展开切换为终止。例如，如果你想要在 release 模式中 panic 时直接终止，可添加：

```rust
[profile.release]
panic = 'abort'
```

```rust
fn main() {
    panic!("crash and burn");
}
```

```rust
$ cargo run
   Compiling panic v0.1.0 (file:///projects/panic)
    Finished `dev` profile [unoptimized + debuginfo] target(s) in 0.25s
     Running `target/debug/panic`
# panic!函数输出的错误信息
thread 'main' panicked at src/main.rs:2:5:
crash and burn
note: run with `RUST_BACKTRACE=1` environment variable to display a backtrace
```

#### panic!的 backtrace
> 可以设置 `RUST_BACKTRACE` 环境变量来得到一个 backtrace。_backtrace_ 是一个执行到目前位置所有被调用的函数的列表。Rust 的 backtrace 跟其他语言中的一样：阅读 backtrace 的关键是从头开始读直到发现你编写的文件。这就是问题的发源地。这一行往上是你的代码所调用的代码；往下则是调用你的代码的代码。这些行可能包含核心 Rust 代码，标准库代码或用到的 crate 代码。让我们将 `RUST_BACKTRACE` 环境变量设置为任何不是 `0` 的值来获取 backtrace 看看。
>

```rust
$ RUST_BACKTRACE=1 cargo run
thread 'main' panicked at src/main.rs:4:6:
index out of bounds: the len is 3 but the index is 99
stack backtrace:
   0: rust_begin_unwind
             at /rustc/4d91de4e48198da2e33413efdcd9cd2cc0c46688/library/std/src/panicking.rs:692:5
   1: core::panicking::panic_fmt
             at /rustc/4d91de4e48198da2e33413efdcd9cd2cc0c46688/library/core/src/panicking.rs:75:14
   2: core::panicking::panic_bounds_check
             at /rustc/4d91de4e48198da2e33413efdcd9cd2cc0c46688/library/core/src/panicking.rs:273:5
   3: <usize as core::slice::index::SliceIndex<[T]>>::index
             at file:///home/.rustup/toolchains/1.85/lib/rustlib/src/rust/library/core/src/slice/index.rs:274:10
   4: core::slice::index::<impl core::ops::index::Index<I> for [T]>::index
             at file:///home/.rustup/toolchains/1.85/lib/rustlib/src/rust/library/core/src/slice/index.rs:16:9
   5: <alloc::vec::Vec<T,A> as core::ops::index::Index<I>>::index
             at file:///home/.rustup/toolchains/1.85/lib/rustlib/src/rust/library/alloc/src/vec/mod.rs:3361:9
   6: panic::main
             at ./src/main.rs:4:6
   7: core::ops::function::FnOnce::call_once
             at file:///home/.rustup/toolchains/1.85/lib/rustlib/src/rust/library/core/src/ops/function.rs:250:5
note: Some details are omitted, run with `RUST_BACKTRACE=full` for a verbose backtrace.
```

### Result 处理可恢复错误
```rust
enum Result<T, E> {
    Ok(T),
    Err(E),
}
```

```rust
use std::fs::File;
use std::io::ErrorKind;

fn main() {
    let greeting_file_result = File::open("hello.txt");

    let greeting_file = match greeting_file_result {
        Ok(file) => file,
        Err(error) => match error.kind() {
            ErrorKind::NotFound => match File::create("hello.txt") {
                Ok(fc) => fc,
                Err(e) => panic!("Problem creating the file: {e:?}"),
            },
            _ => {
                panic!("Problem opening the file: {error:?}");
            }
        },
    };
}
```

```rust
use std::fs::File;
use std::io::ErrorKind;

fn main() {
    let greeting_file = File::open("hello.txt").unwrap_or_else(|error| {
        if error.kind() == ErrorKind::NotFound {
            File::create("hello.txt").unwrap_or_else(|error| {
                panic!("Problem creating the file: {error:?}");
            })
        } else {
            panic!("Problem opening the file: {error:?}");
        }
    });
}
```

#### unwrap 和 expect
> `Result<T, E>` 类型定义了很多辅助方法来处理各种更为特定的任务。
>

> `unwrap` 方法是一个快捷方式，其内部实现与我们在 Listing 9-4 中编写的 `match` 表达式相同。如果 `Result` 值是变体 `Ok`，`unwrap` 会返回 `Ok` 中的值。如果 `Result` 是变体 `Err`，`unwrap` 会为我们调用 `panic!`。
>

```rust
use std::fs::File;

fn main() {
    let greeting_file = File::open("hello.txt").unwrap();
}
```

> `expect` 方法允许我们自定义 `panic!` 的错误信息。使用 `expect` 并提供一个好的错误信息可以表明你的意图并更易于追踪 panic 的根源。
>

```rust
use std::fs::File;

fn main() {
    let greeting_file = File::open("hello.txt")
        .expect("hello.txt should be included in this project");
}
```

#### 传播错误
> 当函数的实现中调用了可能会失败的操作时，除了在这个函数中处理错误外，还可以选择让调用者知道这个错误并决定该如何处理。这被称为**传播**（_propagating_）错误，这样能更好的控制代码调用，因为比起你代码所拥有的上下文，调用者可能拥有更多信息或逻辑来决定应该如何处理错误。
>

```rust
use std::fs::File;
use std::io::{self, Read};

// 一般写法
fn read_username_from_file() -> Result<String, io::Error> {
    let username_file_result = File::open("hello.txt");

    let mut username_file = match username_file_result {
        Ok(file) => file,
        // 出现错误时，将错误返回给调用者
        Err(e) => return Err(e),
    };

    let mut username = String::new();

    match username_file.read_to_string(&mut username) {
        Ok(_) => Ok(username),
        Err(e) => Err(e),
    }
}

// ?语法糖
fn read_username_from_file() -> Result<String, io::Error> {
    // Result 值之后的 ? ：Ok返回Ok的值，Err则将Err作为整个函数的返回值并立即返回。
    let mut username_file = File::open("hello.txt")?;
    
    let mut username = String::new();
    username_file.read_to_string(&mut username)?;
    Ok(username)
}

// 链式调用
fn read_username_from_file() -> Result<String, io::Error> {
    let mut username = String::new();
    
    File::open("hello.txt")?.read_to_string(&mut username)?;
    
    Ok(username)
}
```

什么时候能使用 `?` 运算符呢？

只能在返回 `Result`、`Option` 或者其它实现了 `FromResidual` 的类型的函数中使用 `?` 运算符。

### panic!与 Result 的抉择
> 那么，该如何决定何时应该 `panic!` 以及何时应该返回 `Result` 呢？
>
> 如果代码 panic，就没有恢复的可能。你可以选择对任何错误场景都调用 `panic!`，不管是否有可能恢复，不过这样就是你代替调用者决定了这是不可恢复的。
>
> 选择返回 `Result` 值的话，就将选择权交给了调用者，而不是代替他们做出决定。调用者可能会选择以符合他们场景的方式尝试恢复，或者也可能干脆就认为 `Err` 是不可恢复的，所以他们也可能会调用 `panic!` 并将可恢复的错误变成了不可恢复的错误。因此返回 `Result` 是定义可能会失败的函数的一个好的默认选择。
>

在当有可能会导致有害状态（bad state）的情况下建议使用 `panic!` —— 在这里，**有害状态**（_bad state_）是指当一些假设、保证、协议或不可变性被打破的状态，例如无效的值、自相矛盾的值或者被传递了不存在的值 —— 外加如下几种情况：

+ 有害状态是非预期的行为，与偶尔会发生的行为相对，比如用户输入了错误格式的数据。
+ 在此之后代码的运行依赖于不处于这种有害状态，而不是在每一步都检查是否有问题。
+ 没有可行的手段来将有害状态信息编码进所使用的类型中的情况。我们会在第十八章[“将状态和行为编码为类型”](https://kaisery.github.io/trpl-zh-cn/ch18-03-oo-design-patterns.html#%E5%B0%86%E7%8A%B6%E6%80%81%E5%92%8C%E8%A1%8C%E4%B8%BA%E7%BC%96%E7%A0%81%E4%B8%BA%E7%B1%BB%E5%9E%8B)部分通过一个例子来说明我们的意思。

如果别人调用你的代码并传递了一个没有意义的值，尽最大可能返回一个错误，如此库的用户就可以决定在这种情况下该如何处理。然而在继续执行代码是不安全或有害的情况下，最好的选择可能是调用 `panic!` 并警告库的用户他们的代码中有 bug，这样他们就会在开发时进行修复。类似的，如果你正在调用不受你控制的外部代码，并且它返回了一个你无法修复的无效状态，那么 `panic!` 往往是合适的。

然而当错误预期会出现时，返回 `Result` 仍要比调用 `panic!` 更为合适。这样的例子包括解析器接收到格式错误的数据，或者 HTTP 请求返回了一个表明触发了限流的状态。在这些例子中，应该通过返回 `Result` 来表明失败预期是可能的，而调用者就必须决定该如何处理这个问题。

## **泛型**（_generics_）
> 具体类型或其他属性的抽象替代。
>

```rust
// 限制T仅对实现了 std::cmp::{artialOrd 的类型有效
fn largest<T: std::cmp::PartialOrd>(list: &[T]) -> &T {}
```

```rust
struct Point<T> {
    x: T,
    y: T,
}

impl<T> Point<T> {
    fn x(&self) -> &T {
        &self.x
    }
}

// 仅为 T = f32 的类型定义方法
impl Point<f32> {
    fn distance_from_origin(&self) -> f32 {
        (self.x.powi(2) + self.y.powi(2)).sqrt()
    }
}
```

```rust
struct Point<T, U> {
    x: T,
    y: U,
}
```

```rust
// Result就是一个经典的例子
enum Result<T, E> {
    Ok(T),
    Err(E),
}
```

### 泛型的性能
> Rust 通过在编译时进行泛型代码的**单态化**（_monomorphization_）来保证效率。单态化是一个通过填充编译时使用的具体类型，将通用代码转换为特定代码的过程。这使得泛型并不会使程序比具体类型运行得慢。
>
> 因为 Rust 会将每种情况下的泛型代码编译为具体类型，使用泛型没有运行时开销。当代码运行时，它的执行效率就跟好像手写每个具体定义的重复代码一样。这个单态化过程正是 Rust 泛型在运行时极其高效的原因。
>

## Trait：定义共同行为
> _trait_ 定义了某个特定类型拥有可能与其他类型共享的功能。可以通过 trait 以一种抽象的方式定义共同行为，但没有具体实现。可以使用 _trait bounds_ 指定泛型是任何拥有特定行为的类型。
>
> 注意：_trait_ 类似于其他语言中的常被称为 **接口**（_interfaces_）的功能，虽然两者还是有一些不同。
>

```rust
pub trait Summary {
    // 没有默认实现
    fn summarize(&self) -> String;
    // 默认实现
    fn summarize(&self) -> String {
        String::from("默认实现")
    }
}
```

```rust
pub struct NewsArticle {
    pub headline: String,
    pub location: String,
    pub author: String,
    pub content: String,
}

// 自己实现具体逻辑
impl Summary for NewsArticle {
    fn summarize(&self) -> String {
        format!("{}, by {} ({})", self.headline, self.author, self.location)
    }
}

// 使用默认实现
impl Summary for NewArticle {}

// 有条件地实现方法，Pair是另一个结构体，只有当Pair实例的泛型是实现了Summary Trait类型时，才有下面的方法。
impl<T: Summary> Pair<T> {
    fn summary_pair(&self) {
        println!("x={}, y={}", self.x.summarize(), self.y.summarize());
    }
}

// 为所有实现了特定trait的类型实现一个trait，为所有实现了Summary的类型实现一个ToSummary Trait。
impl<T: Summary> ToSummary for T {}
```

```rust
// trait bound
pub fn notify<T: Summary>(item: &T) {
    println!("Breaking news! {}", item.summarize());
}

// 语法糖
pub fn notify(item: &impl Summary) {
    println!("Breaking news! {}", item.summarize());
}

// 多个 trait bound
pub fn notify<T: Summary + Display>(item: &T) {}
pub fn notify(item: &(impl Summary + Display)) {}

// 当函数有过多的trait时，可以使用 where 来简化
fn notify<T, U>(t: &T, u: &U) -> i32
where
    T: Display + Clone,
    U: Clone + Debug,
{}
```

```rust
fn returns_summarizable() -> impl Summary {
    SocialPost {
        username: String::from("horse_ebooks"),
        content: String::from(
            "of course, as you probably already know, people",
        ),
        reply: false,
        repost: false,
    }
}
```

## 生命周期
> 生命周期是另一类我们已经使用过的泛型。不同于确保类型有期望的行为，生命周期用于保证引用在我们需要的整个期间内都是有效的。
>

> Rust 中的每一个引用都有其**生命周期**（_lifetime_），也就是引用保持有效的作用域。大部分时候生命周期是隐含并可以推断的，正如大部分时候类型也是可以推断的一样。类似于当因为有多种可能类型的时候必须注明类型，也会出现引用的生命周期以一些不同方式相关联的情况，所以 Rust 需要我们使用泛型生命周期参数来注明它们的关系，这样就能确保运行时实际使用的引用绝对是有效的。
>

> 生命周期语法：`'a`
>

```rust
&i32	// 引用
&'a i32 // 带有显式生命周期的引用
&'a mut i32 // 带有显式生命周期的可变引用
```

生命周期语法是用于将函数的多个参数与其返回值的生命周期进行关联的。一旦它们形成了某种关联，Rust 就有了足够的信息来允许内存安全的操作并阻止会产生悬垂指针亦或是违反内存安全的行为。

```rust
// 函数签名中所有引用必须有相同的生命周期 'a，泛型生命周期 'a 的具体生命周期等同于 x 和 y 的生命周期中较小的那一个。因为我们用相同的生命周期参数 'a 标注了返回的引用值，所以返回的引用值就能保证在 x 和 y 中较短的那个生命周期结束之前保持有效。
fn longest<'a>(x: &'a str, y: &'a str) -> &'a str {
    if x.len() > y.len() { x } else { y }
}
```

我们定义的结构体全都包含拥有所有权的类型。也可以定义包含引用的结构体，不过这需要为结构体定义中的每一个引用添加生命周期注解。

```rust
// 这个注解意味着 ImportantExcerpt 的实例不能比其 part 字段中的引用存在的更久。
struct ImportantExcerpt<'a> {
    part: &'a str,
}

fn main() {
    let novel = String::from("Call me Ishmael. Some years ago...");
    let first_sentence = novel.split('.').next().unwrap();
    let i = ImportantExcerpt {
        part: first_sentence,
    };
}
```

### **生命周期省略规则**
1. 编译器为每一个引用参数都分配一个生命周期参数。换句话说就是，函数有一个引用参数的就有一个生命周期参数：`fn foo<'a>(x: &'a i32)`，有两个引用参数的函数就有两个不同的生命周期参数，`fn foo<'a, 'b>(x: &'a i32, y: &'b i32)`，依此类推。
2. 如果只有一个输入生命周期参数，那么将它赋予给所有输出生命周期参数：`fn foo<'a>(x: &'a i32) -> &'a i32`。
3. 如果方法有多个输入生命周期参数并且其中一个参数是 `&self` 或 `&mut self`，说明这是个方法，那么所有输出生命周期参数被赋予 `self` 的生命周期。第三条规则使得方法更容易读写，因为只需更少的符号。

```rust
// 开始时签名中的引用并没有关联任何生命周期
fn first_word(s: &str) -> &str {}
// 第一条规则：每个引用参数都有其自己的生命周期。
fn first_word<'a>(s: &'a str) -> &str {}
// 第二条规则：只有一个输入参数，那么输出也将具有相同的生命周期
fn longest(x: &str, y: &str) -> &str {}
// 
```

### 静态生命周期
> `'static`，其生命周期**能够**存活于整个程序期间。所有的字符串字面值都拥有 `'static` 生命周期。一般不指定静态生命周期，思考一下这个引用是否真的在整个程序的生命周期里都有效，以及你是否希望它存在得这么久。
>

```rust
// 结合泛型类型参数、trait bounds 和生命周期
use std::fmt::Display;

fn longest_with_an_announcement<'a, T>(
    x: &'a str,
    y: &'a str,
    ann: T,
) -> &'a str
where
    T: Display,
{
    println!("Announcement! {ann}");
    if x.len() > y.len() { x } else { y }
}
```

### 借用检查器
> Rust 编译器有一个**借用检查器**（_borrow checker_），它比较作用域来确保所有的借用都是有效的。
>

## 闭包
> **闭包**（_closures_）是可以保存在变量中或作为参数传递给其他函数的匿名函数。你可以在一个地方创建闭包，然后在不同的上下文中执行闭包运算。不同于函数，闭包允许捕获其被定义时所在作用域中的值。
>

> 闭包语法：| 参数列表 | 闭包体
>

### 使用闭包捕获环境中的值
> 环境指的是：定义闭包的环境
>
> **闭包体 **中使用的 ”环境中的变量“ 就是闭包所捕获的环境中的变量。
>

> 闭包可以通过三种方式捕获其环境中的值，它们直接对应到函数获取参数的三种方式：不可变借用、可变借用和获取所有权。闭包将根据函数体中对捕获值的操作来决定使用哪种方式。
>

```rust
// 定义了一个捕获名为 list 的 vector 的不可变引用的闭包，因为只需不可变引用就能打印其值
let list = vec![1, 2, 3];
println!("Before defining closure: {list:?}");

let only_borrows = || println!("From closure: {list:?}");

// 变量可以绑定一个闭包定义，并且可以像使用函数名一样，使用变量名和括号来调用该闭包。
println!("Before calling closure: {list:?}");
only_borrows();
println!("After calling closure: {list:?}");
```

```rust
let mut list = vec![1, 2, 3];
println!("Before defining closure: {:?}", list);

let mut borrows_mutably = || list.push(7);
// 注意在 borrows_mutably 闭包的定义和调用之间不再有 println!，这是因为当 borrows_mutably 被定义时，它捕获了对 list 的可变引用。根据借用规则，最多可存在一个可变引用，且在可变引用使用期间不可存在不可变引用。因此，在闭包定义和调用之间不能有不可变引用来进行打印。
borrows_mutably();
println!("After calling closure: {:?}", list);
```

即使闭包体不严格需要所有权，如果希望强制闭包获取它在环境中所使用的值的所有权，可以在参数列表前使用 move 关键字。当将闭包传递到一个新的线程时，这个技巧特别有用，因为它将数据的所有权移动到新线程中。

```rust
use std::thread;

fn main() {
    let list = vec![1, 2, 3];
    println!("Before defining closure: {list:?}");

    thread::spawn(move || println!("From thread: {list:?}"))
        .join()
        .unwrap();
}
```

将捕获的值移出闭包和 Fn trait

> 一旦闭包捕获了定义它的环境中的某个值的引用或所有权（也就影响了什么会被移**进**闭包，如有），闭包体中的代码则决定了在稍后执行闭包时，这些引用或值将如何处理（也就影响了什么会被移**出**闭包，如有）。闭包体可以执行以下任一操作：将一个捕获的值移出闭包，修改捕获的值，既不移动也不修改值，或者一开始就不从环境中捕获任何值。
>

闭包捕获和处理环境中的值的方式会影响闭包实现哪些 trait，而 trait 是函数和结构体指定它们可以使用哪些类型闭包的方式。根据闭包体如何处理这些值，闭包会自动、渐进地实现一个、两个或全部三个 `Fn` trait。

1. `FnOnce` 适用于只能被调用一次的闭包。所有闭包至少都实现了这个 trait，因为所有闭包都能被调用。一个会将捕获的值从闭包体中移出的闭包只会实现 `FnOnce` trait，而不会实现其他 `Fn` 相关的 trait，因为它只能被调用一次。
2. `FnMut` 适用于不会将捕获的值移出闭包体，但可能会修改捕获值的闭包。这类闭包可以被调用多次。
3. `Fn` 适用于既不将捕获的值移出闭包体，也不修改捕获值的闭包，同时也包括不从环境中捕获任何值的闭包。这类闭包可以被多次调用而不会改变其环境，这在会多次并发调用闭包的场景中十分重要。

```rust
impl<T> Option<T> {
    pub fn unwrap_or_else<F>(self, f: F) -> T
    where
        F: FnOnce() -> T
    {
        match self {
            Some(x) => x,
            None => f(),
        }
    }
}
```

```rust
#[derive(Debug)]
struct Rectangle {
    width: u32,
    height: u32,
}

fn main() {
    let mut list = [
        Rectangle { width: 10, height: 1 },
        Rectangle { width: 3, height: 5 },
        Rectangle { width: 7, height: 12 },
    ];

    let mut sort_operations = vec![];
    let value = String::from("closure called");

    list.sort_by_key(|r| {
        // push 使 vector 获取了 value 的所有权，因此这个闭包应该是实现了 FnOnce 的，只能使用一次。由于此闭包会多次使用，所以会在第二次执行push时报错。
        sort_operations.push(value);
        r.width
    });
    println!("{list:#?}");
}
```

## 迭代器
> 迭代器模式允许你依次对一个序列中的项执行某些操作。**迭代器**（_iterator_）负责遍历序列中的每一项并确定序列何时结束的逻辑。使用迭代器时，你无需自己重新实现这些逻辑。迭代器不会影响程序性能，因为编译器会在编译时进行优化。
>

> 在 Rust 中，迭代器是**惰性的**（_lazy_），这意味着在调用消费迭代器的方法之前不会执行任何操作。
>

> 迭代器的类型声明：Iter<'_, i32>
>

迭代器都实现了名为 `Iterator` 的定义于标准库的 trait。这个 trait 的定义看起来像这样：

```rust
pub trait Iterator {
    type Item;

    // 每次返回迭代器中的一个项，封装在 Some 中，并且当迭代完成时，返回 None。
    fn next(&mut self) -> Option<Self::Item>;

    // 此处省略了方法的默认实现
}
```

注意我们需要将 `v1_iter` 声明为可变的：在迭代器上调用 `next` 方法会改变迭代器内部的状态，该状态用于跟踪迭代器在序列中的位置。换句话说，代码**消费**（consume）了，或者说用尽了迭代器。每一次 `next` 调用都会从迭代器中消费一个项。使用 `for` 循环时无需使 `v1_iter` 可变因为 `for` 循环会获取 `v1_iter` 的所有权并在后台使 `v1_iter` 可变。

从 `next` 调用中获取的值是对 vector 中值的不可变引用。`iter` 方法生成一个不可变引用的迭代器。如果我们需要一个获取 `v1` 所有权并返回拥有所有权的迭代器，则可以调用 `into_iter` 而不是 `iter`。类似地，如果我们希望迭代可变引用，可以调用 `iter_mut` 而不是 `iter`。



+ **消费适配器**（_consuming adaptors_）：调用 `next` 方法的方法被称为，因为调用它们会消耗迭代器。
+ **迭代器适配器**（_iterator adaptors_）：`Iterator` trait 中定义了另一类方法，它们不会消耗当前的迭代器，而是通过改变原始迭代器的某些方面来生成不同的迭代器。

```rust
let v1: Vec<i32> = vec![1, 2, 3];

v1.iter().map(|x| x + 1);
```

可以链式调用多个迭代器适配器来以一种易读的方式进行复杂的操作。不过因为所有的迭代器都是惰性的，你必须调用一个消费适配器方法，才能从这些迭代器适配器的调用中获取结果。

## 智能指针
> **指针**（_pointer_）是一个包含内存地址的变量的通用概念。这个地址会引用，或者说 “指向”（points at）一些其它数据。Rust 中最常见的指针是第四章介绍的**引用**（_reference_）。引用使用 `&` 符号来表示，而且会借用它们指向的值。它们除了引用数据之外没有任何其他特殊功能，也没有额外开销。
>

> **智能指针**（_smart pointers_）是一类数据结构，它们的表现类似指针，但是也拥有额外的元数据和功能。
>
> 在 Rust 中因为引用和借用的概念，引用和智能指针有一个额外的区别：引用只会借用数据，而智能指针在很多时候**拥有**它们指向的数据。
>

> 智能指针通常使用结构体实现。智能指针不同于普通结构体的地方在于它实现了 `Deref` 和 `Drop` trait。`Deref` trait 允许智能指针结构体实例表现得像引用一样，这样就可以编写既用于引用、又用于智能指针的代码。`Drop` trait 允许我们自定义当智能指针离开作用域时运行的代码。
>

接下来，我们来看几种常用的智能指针：

### Box<T>指向堆上的数据
> 最简单直接的智能指针是 _box_，其类型是 `Box<T>`。box 允许你将一个值放在堆上而不是栈上。留在栈上的则是指向堆数据的指针。
>

除了数据被储存在堆上而不是栈上之外，box 没有性能损失。不过也没有很多额外的功能。它们多用于如下场景：

+ 当有一个在编译时未知大小的类型，而又想要在需要确切大小的上下文中使用这个类型值的时候
+ 当有大量数据并希望在确保数据不被拷贝的情况下转移所有权的时候
+ 当希望拥有一个值并只关心它的类型是否实现了特定 trait 而不是其具体类型的时候

```rust
fn main() {
    // 使用 Box<T>在堆上存储数据
    let b = Box::new(5);
    println!("b = {b}");
    // 我们可以像数据是储存在栈上的那样访问 box 中的数据。正如任何拥有数据所有权的值那样，当像 b 这样的 box 在 main 的末尾离开作用域时，它将被释放。这个释放过程作用于 box 本身（位于栈上）和它所指向的数据（位于堆上）。
}
```

```rust
enum List {
    Cons(i32, List),
    Nil,
}

use crate::List::{Cons, Nil};

fn main() {
    // 尝试定义一个递归枚举时产生错误：这个类型 “有无限的大小”（“has infinite size”）。Rust 无法计算为了存放 List 值到底需要多少空间。
    let list = Cons(1, Cons(2, Cons(3, Nil)));

    // 使用 Box<T> 代替递归
    let list = Cons(1, Box::new(Cons(2, Box::new(Cons(3, Box::new(Nil))))));
}
```

### Rc<T>引用计数智能指针
> 有些情况单个值可能会有多个所有者。例如，在图数据结构中，多个边可能指向相同的节点，而这个节点从概念上讲为所有指向它的边所拥有。节点在没有任何边指向它从而没有任何所有者之前，都不应该被清理掉。
>

> 为了启用多所有权需要显式地使用 Rust 类型 `Rc<T>`，其为**引用计数**（_reference counting_）的缩写。引用计数意味着记录一个值的引用数量来知晓这个值是否仍在被使用。如果某个值有零个引用，就代表没有任何有效引用并可以被清理。
>

> `Rc<T>` 用于当我们希望在堆上分配一些内存供程序的多个部分读取，而且无法在编译时确定程序的哪一部分会最后结束使用它的时候。如果确实知道哪部分是最后一个结束使用的话，就可以令其成为数据的所有者，正常的所有权规则就可以在编译时生效。
>

> 通过不可变引用， `Rc<T>` 允许在程序的多个部分之间只读地共享数据。
>

> 注意 `Rc<T>` 只能用于单线程场景；
>

```rust
enum List {
    Cons(i32, Rc<List>),
    Nil,
}

use crate::List::{Cons, Nil};
use std::rc::Rc;

fn main() {
    let a = Rc::new(Cons(5, Rc::new(Cons(10, Rc::new(Nil)))));
    // 使用 Rc::clone 增加引用计数
    let b = Cons(3, Rc::clone(&a));
    let c = Cons(4, Rc::clone(&a));
}
// Drop trait 的实现当 Rc<T> 值离开作用域时自动减少引用计数。
```

### RefCell<T>和内部可变性模式
> 内部可变性：不可变值的可变借用。  
`RefCell<T>` 是一个获得内部可变性的方法。
>

> **在不可变值内部改变值就是内部可变性（**_interior mutability_**）模式。**
>
> **内部可变性**（_Interior mutability_）是 Rust 中的一个设计模式，它允许你即使在有不可变引用时也可以改变数据，这通常是借用规则所不允许的。为了改变数据，该模式在数据结构中使用 `unsafe` 代码来模糊 Rust 通常的可变性和借用规则。不安全代码表明我们手动检查这些规则而不是让编译器替我们检查。
>

> 当可以确保代码在运行时会遵守借用规则，即使编译器不能保证的情况，可以选择使用那些运用内部可变性模式的类型。所涉及的 `unsafe` 代码将被封装进安全的 API 中，而外部类型仍然是不可变的。
>

对于引用和 `Box<T>`，借用规则的不可变性（invariants）在编译时就会被强制执行。对于 `RefCell<T>`，这些不可变性作用于**运行时**。对于引用，如果违反这些规则，会得到一个编译错误。而对于 `RefCell<T>`，如果违反这些规则程序会 panic 并退出。

> 类似于 `Rc<T>`，`RefCell<T>` 只能用于单线程场景。
>

如下为选择 `Box<T>`，`Rc<T>` 或 `RefCell<T>` 的理由：

+ `Rc<T>` 允许相同数据有多个所有者；`Box<T>` 和 `RefCell<T>` 则只有单一所有者。
+ `Box<T>` 允许在编译时执行不可变或可变借用检查；`Rc<T>` 仅允许在编译时执行不可变借用检查；`RefCell<T>` 允许在运行时执行不可变或可变借用检查。
+ 因为 `RefCell<T>` 允许在运行时执行可变借用检查，所以我们可以在即便 `RefCell<T>` 自身是不可变的情况下修改其内部的值。

RefCell<T>在运行时记录借用，同样的，RefCell<T>在任何时候只允许有多个不可变借用或一个可变借用。

#### 结合 Rc<T> 和 RefCell<T> 来拥有多个可变数据所有者
> `RefCell<T>` 的一个常见用法是与 `Rc<T>` 结合。回忆一下 `Rc<T>` 允许对相同数据有多个所有者，不过只能提供数据的不可变访问。如果有一个储存了 `RefCell<T>` 的 `Rc<T>` 的话，就可以得到有多个所有者**并且**可以修改的值了！
>

```rust
#[derive(Debug)]
enum List {
    Cons(Rc<RefCell<i32>>, Rc<List>),
    Nil,
}

use crate::List::{Cons, Nil};
use std::cell::RefCell;
use std::rc::Rc;

fn main() {
    let value = Rc::new(RefCell::new(5));

    let a = Rc::new(Cons(Rc::clone(&value), Rc::new(Nil)));

    let b = Cons(Rc::new(RefCell::new(3)), Rc::clone(&a));
    let c = Cons(Rc::new(RefCell::new(4)), Rc::clone(&a));

    *value.borrow_mut() += 10;

    println!("a after = {a:?}");
    println!("b after = {b:?}");
    println!("c after = {c:?}");
}
```

### 使用 Deref Trait 将智能指针当作常规引用处理
> 实现 `Deref` trait 允许我们定制**解引用运算符**（_dereference operator_）`*`（不要与乘法运算符或通配符相混淆）。通过这种方式实现 `Deref` trait 的智能指针可以被当作常规引用来对待，可以编写操作引用的代码并同样适用于智能指针。
>

常规引用是一个指针类型，一种理解指针的方式是将其看成指向储存在其他某处值的箭头。

```rust
fn main() {
    let x = 5;
    let y = &x;
    let z = Box::new(x);

    assert_eq!(5, x);
    assert_eq!(5, *y);
    assert_eq!(5, *z);
}
```

```rust
// 自定义了一个MyBox来替代Box
struct MyBox<T>(T);

impl<T> MyBox<T> {
    fn new(x: T) -> MyBox<T> {
        MyBox(x)
    }
}

fn main() {
    let x = 5;
    let y = MyBox::new(x);

    assert_eq!(5, x);
    // 错误：rust不知道如何解引用
    assert_eq!(5, *y);
}
```

```rust
use std::ops::Deref;

// 为 MyBox 实现 Deref Trait，这样rust就知道如何解引用了
impl<T> Deref for MyBox<T> {
    type Target = T;

    fn deref(&self) -> &Self::Target {
        &self.0
    }
}

```

没有 `Deref` trait 的话，编译器只会解引用 `&` 引用类型。`deref` 方法向编译器提供了获取任何实现了 `Deref` trait 的类型的值，并且调用这个类型的 `deref` 方法来获取一个它知道如何解引用的 `&` 引用的能力。

当我们 运行 `*y`，在底层上运行的是：`*(y.deref())`。

`deref` 方法返回 `值的引用`，以及 `*(y.deref())` 的普通解引用符仍然必须存在的原因在于所有权。如果 `deref` 方法直接返回值而不是值的引用，其值将被移出 `self`。在这里以及大部分使用解引用运算符的情况下我们并不希望获取 `MyBox<T>` 内部值的所有权。

注意，每次当我们在代码中使用 `*` 时， `*` 运算符都被替换成了先调用 `deref` 方法再接着使用 `*` 解引用的操作，且只会发生一次，不会对 `*` 操作符无限递归替换，解引用出值就停止了。

### Deref 强制类型转换
**Deref 强制转换**（_deref coercions_）将实现了 `Deref` trait 的类型的引用转换为另一种类型的引用。例如，Deref 强制转换可以将 `&String` 转换为 `&str`，因为 `String` 实现了 `Deref` trait 因此可以返回 `&str`。Deref 强制转换是 Rust 在函数或方法传参上的一种便利操作，并且只能作用于实现了 `Deref` trait 的类型。当这种特定类型的引用作为实参传递给和形参类型不同的函数或方法时将自动进行。这时会有一系列的 `deref` 方法被调用，把我们提供的类型转换成了参数所需的类型。

Deref 强制转换的加入使得 Rust 程序员编写函数和方法调用时无需增加过多显式使用 `&` 和 `*` 的引用和解引用。这个功能也使得我们可以编写更多同时作用于引用或智能指针的代码。

```rust
fn main() {
    let m = MyBox::new(String::from("Rust"));
    // 强制转换
    hello(&m);
    // 非强制转换
    hello(&(*m)[..]);
}
```

当所涉及到的类型定义了 `Deref` trait，Rust 会分析这些类型并使用任意多次 `Deref::deref` 调用以获得匹配参数的类型。这些解析都发生在编译时，所以利用 Deref 强制转换并没有运行时开销！



类似于如何使用 `Deref` trait 重载不可变引用的 `*` 运算符，Rust 提供了 `DerefMut` trait 用于重载可变引用的 `*` 运算符。

Rust 在发现类型和 trait 实现满足三种情况时会进行 Deref 强制转换：

1. 当 `T: Deref<Target=U>` 时从 `&T` 到 `&U`。
2. 当 `T: DerefMut<Target=U>` 时从 `&mut T` 到 `&mut U`。
3. 当 `T: Deref<Target=U>` 时从 `&mut T` 到 `&U`。
+ 第一种情况表明：如果有一个 `&T`，而 `T` 实现了返回 `U` 类型的 `Deref`，则可以透明地得到 `&U`。
+ 第二种情况表明：对于可变引用也有着相同的行为。
+ 第三种情况表明：Rust 也会将可变引用强转为不可变引用。但反之是**不可能** 的：不可变引用永远也不能强转为可变引用。因为根据借用规则，如果有一个可变引用，其必须是这些数据的唯一引用。将一个可变引用转换为不可变引用并不会打破借用规则。然而，将不可变引用转换为可变引用则需要初始的不可变引用是数据唯一的不可变引用，而借用规则无法保证这一点。



### 使用 Drop Trait 运行清理代码
> 对于智能指针模式来说第二个重要的 trait 是 `Drop`，其允许我们在值要离开作用域时自定义要执行的操作。你可以为任何类型提供 `Drop` trait 的实现，同时所指定的代码被用于释放类似于文件或网络连接的资源。
>

```rust
struct CustomSmartPointer {
    data: String,
}

impl Drop for CustomSmartPointer {
    fn drop(&mut self) {
        println!("Dropping CustomSmartPointer with data `{}`!", self.data);
    }
}

fn main() {
    let c = CustomSmartPointer {
        data: String::from("my stuff"),
    };
    let d = CustomSmartPointer {
        data: String::from("other stuff"),
    };
    println!("CustomSmartPointers created.");
}
```

当实例离开作用域 Rust 会自动调用 `drop`，并调用我们指定的代码。变量以被创建时相反的顺序被丢弃，所以 `d` 在 `c` 之前被丢弃。

```bash
CustomSmartPointers created.
Dropping CustomSmartPointer with data `other stuff`!
Dropping CustomSmartPointer with data `my stuff`!
```

Rust 并不允许我们主动调用 `Drop` trait 的 `drop` 方法；当我们希望在作用域结束之前就强制释放变量的话，我们应该使用的是由标准库提供的 `std::mem::drop` 函数。

```rust
fn main() {
    let c = CustomSmartPointer {
        data: String::from("some data"),
    };
    println!("CustomSmartPointer created.");
    drop(c);
    println!("CustomSmartPointer dropped before the end of main.");
}
```

```bash
CustomSmartPointer created.
Dropping CustomSmartPointer with data `some data`!
CustomSmartPointer dropped before the end of main.
```

### Weak<T> 防止引用循环
> Rc<T>类型：多个变量拥有值的所有权，被称为 **强引用** 。
>
> Weak<T>类型：多个变量只拥有值的引用，但不拥有值的所有权，被称为 **弱引用** 。
>

```rust
let weak: Weak<T> = Rc::downgrade(Rc::new(T类型实例));
```

> 调用 `Rc::clone` 会增加 `Rc<T>` 实例的 `strong_count`，和只在其 `strong_count` 为 0 时 `Rc<T>` 实例才会被清理。弱引用不表达所有权关系，当 `Rc<T>` 实例被清理时其计数没有影响。它们不会造成引用循环，因为任何涉及弱引用的循环会在其相关的值的强引用计数为 0 时被打断。
>

> 调用 `Rc::downgrade` 时会得到 `Weak<T>` 类型的智能指针。不同于将 `Rc<T>` 实例的 `strong_count` 加 1，调用 `Rc::downgrade` 会将 `weak_count` 加 1。`Rc<T>` 类型使用 `weak_count` 来记录其存在多少个 `Weak<T>` 引用，类似于 `strong_count`。其区别在于 `weak_count` 无需计数为 0 就能使 `Rc<T>` 实例被清理。
>

> 因为 `Weak<T>` 引用的值可能已经被丢弃了，为了使用 `Weak<T>` 所指向的值，我们必须确保其值仍然有效。为此可以调用 `Weak<T>` 实例的 `upgrade` 方法，这会返回 `Option<Rc<T>>`。如果 `Rc<T>` 值还未被丢弃，则结果是 `Some`；如果 `Rc<T>` 已被丢弃，则结果是 `None`。因为 `upgrade` 返回一个 `Option<Rc<T>>`，Rust 会确保处理 `Some` 和 `None` 的情况，所以它不会返回无效指针。
>

## 无畏并发
> **并发编程**（_Concurrent programming_），代表程序的不同部分相互独立地执行，而**并行编程**（_parallel programming_）代表程序不同部分同时执行。
>

> 注意：出于简洁的考虑，我们将很多问题归类为**并发**，而不是更准确的区分**并发和/或并行**。当区分二者更为重要时，我们会使用更准确的表述。
>

在大部分现代操作系统中，已执行程序的代码在一个**进程**（_process_）中运行，操作系统则会负责管理多个进程。在程序内部，也可以拥有多个同时运行的独立部分。这些运行这些独立部分的功能被称为**线程**（_threads_）。

将程序中的计算拆分进多个线程可以改善性能，因为程序可以同时进行多个任务，不过这也会增加复杂性。因为线程是同时运行的，所以无法预先保证不同线程中的代码的执行顺序。这会导致诸如此类的问题：

+ 竞态条件（Race conditions），多个线程以不一致的顺序访问数据或资源
+ 死锁（Deadlocks），两个线程相互等待对方，这会阻止两者继续运行
+ 只会发生在特定情况且难以稳定重现和修复的 bug

### spawn 创建新线程
为了创建一个新线程，需要调用 `thread::spawn` 函数并传递一个闭包，并在其中包含希望在新线程运行的代码。

`thread::sleep` 调用强制线程停止执行一小段时间，这会允许其他不同的线程运行。

```rust
use std::thread;
use std::time::Duration;

fn main() {
    thread::spawn(|| {
        for i in 1..10 {
            println!("hi number {i} from the spawned thread!");
            thread::sleep(Duration::from_millis(1));
        }
    });

    for i in 1..5 {
        println!("hi number {i} from the main thread!");
        thread::sleep(Duration::from_millis(1));
    }
}
```

注意当 Rust 程序的主线程结束时，所有新线程也会结束，而不管其是否执行完毕。

### join 等待所有线程结束
因为无法保证线程运行的顺序，我们甚至不能实际保证新建线程会被执行！

可以通过将 `thread::spawn` 的返回值储存在变量中来修复新建线程部分没有执行或者完全没有执行的问题。`thread::spawn` 的返回值类型是 `JoinHandle<T>`。`JoinHandle<T>` 是一个拥有所有权的值，当对其调用 `join` 方法时，它会等待其线程结束。

```rust
use std::thread;
use std::time::Duration;

fn main() {
    let handle = thread::spawn(|| {
        for i in 1..10 {
            println!("hi number {i} from the spawned thread!");
            thread::sleep(Duration::from_millis(1));
        }
    });

    for i in 1..5 {
        println!("hi number {i} from the main thread!");
        thread::sleep(Duration::from_millis(1));
    }

    handle.join().unwrap();
}
```

### move 闭包与线程一同使用
`move` 关键字经常用于传递给 `thread::spawn` 的闭包，因为闭包会获取从环境中取得的值的所有权，因此会将这些值的所有权从一个线程传送到另一个线程。

```rust
use std::thread;

fn main() {
    let v = vec![1, 2, 3];

    // Rust 会推断闭包如何捕获 v，因为 println! 只需要 v 的引用，闭包尝试借用 v。然而这有一个问题：Rust 不知道这个新建线程会执行多久，所以无法知晓对 v 的引用是否一直有效。
    let handle = thread::spawn(|| {
        println!("Here's a vector: {v:?}");
    });

    handle.join().unwrap();
}

fn main() {
    let v = vec![1, 2, 3];

    // 使用 move 关键字强制获取它使用的值的所有权
    let handle = thread::spawn(move || {
        println!("Here's a vector: {v:?}");
    });

    handle.join().unwrap();
}
```

### 消息传递与信道
> 一个日益流行的确保安全并发的方式是**消息传递**（_message passing_），这里线程或 actor 通过发送包含数据的消息来相互沟通。这个思想来源于 [Go 编程语言文档](https://golang.org/doc/effective_go.html#concurrency) 中的口号：“不要通过共享内存来通讯；而要通过通讯来共享内存。”（“Do not communicate by sharing memory; instead, share memory by communicating.”）
>

> 为了实现消息传递并发，Rust 标准库提供了一个**信道**（_channel_）实现。信道是一个通用编程概念，表示数据从一个线程发送到另一个线程。
>

> 信道有两个组成部分：一个发送端（transmitter）和一个接收端（receiver）。代码中的一部分调用发送端的方法以及希望发送的数据，另一部分则检查接收端收到的消息。当发送端或接收端任一被丢弃时可以认为信道被**关闭**（_closed_）了。
>

使用 `mpsc::channel` 函数创建一个新的信道；`mpsc` 是 **多生产者，单消费者**（_multiple producer, single consumer_）的缩写。简而言之，Rust 标准库实现信道的方式意味着一个信道可以有多个产生值的 **发送端**（_sending_），但只能有一个消费这些值的**接收端**（_receiving_）。

`mpsc::channel` 函数返回一个元组：第一个元素是发送侧 -- 发送端，而第二个元素是接收侧 -- 接收端。由于历史原因，`tx` 和 `rx` 通常作为**发送端**（_transmitter_）和 **接收端**（_receiver_）的传统缩写。

信道的接收端有两个有用的方法：`recv` 和 `try_recv`。

`recv`会阻塞主线程执行直到从信道中接收一个值。一旦发送了一个值，`recv` 会在一个 `Result<T, E>` 中返回它。当信道发送端关闭，`recv` 会返回一个错误表明不会再有新的值到来了。

`try_recv`不会阻塞，相反它立刻返回一个 `Result<T, E>`：`Ok` 值包含可用的信息，而 `Err` 值代表此时没有任何消息。

```rust
use std::sync::mpsc;
use std::thread;

fn main() {
    let (tx, rx) = mpsc::channel();

    thread::spawn(move || {
        let val = String::from("hi");
        tx.send(val).unwrap();
    });

    let received = rx.recv().unwrap();
    println!("Got: {received}");
}
```

#### 信道与所有权转移
```rust
use std::sync::mpsc;
use std::thread;

fn main() {
    let (tx, rx) = mpsc::channel();

    thread::spawn(move || {
        let val = String::from("hi");
        // end 函数获取其参数的所有权并移动这个值归接收端所有。
        tx.send(val).unwrap();
        // 这可以防止在发送后意外地再次使用这个值；所有权系统检查一切是否合乎规则。
        println!("val is {val}");
    });

    let received = rx.recv().unwrap();
    println!("Got: {received}");
}
```

#### 克隆发送端创建多个生产者
发送端调用了 `clone` 方法，这会给我们一个可以传递给第一个新建线程的发送端句柄。我们会将原始的信道发送端传递给第二个新建线程。这样就会有两个线程，每个线程将向信道的接收端发送不同的消息。

```rust
use std::sync::mpsc;
use std::thread;

fn main() {
    let (tx, rx) = mpsc::channel();

    let tx1 = tx.clone();
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
        println!("Got: {received}");
    }
}
```

### 共享状态的并发
消息传递是一个很好的处理并发的方式，但并不是唯一一个。另一种方式是让多个线程访问相同的共享数据。

在某种程度上，任何编程语言中的信道都类似于单所有权，因为一旦将一个值传送到信道中，将无法再使用这个值。共享内存类似于多所有权：多个线程可以同时访问相同的内存位置。

#### 互斥器 Mutex<T>
> **互斥器**（_mutex_）是互相排斥（_mutual exclusion_）的缩写，因为在同一时刻，它只允许一个线程访问数据。为了访问互斥器中的数据，线程首先需要通过获取互斥器的**锁**（_lock_）来表明其希望访问数据。锁是一个数据结构，作为互斥器的一部分，它记录谁有数据的专属访问权。因此我们讲，互斥器通过锁系统**保护**（_guarding_）其数据。
>

```rust
use std::sync::Mutex;

fn main() {
    let m = Mutex::new(5);

    {
        let mut num = m.lock().unwrap();
        *num = 6;
    }

    println!("m = {m:?}");
}
```

我们使用关联函数 `new` 来创建一个 `Mutex<T>`。使用 `lock` 方法来获取锁，从而可以访问互斥器中的数据。这个调用会阻塞当前线程，直到我们拥有锁为止。

如果另一个线程拥有锁，并且那个线程 panic 了，则 `lock` 调用会失败。在这种情况下，没人能够再获取锁，所以我们调用 `unwrap`，使当前线程 panic。

一旦获取了锁，就可以将返回值（命名为 `num`）视为一个其内部数据的可变引用了。类型系统确保了我们在使用 `m` 中的值之前获取锁。`m` 的类型是 `Mutex<i32>` 而不是 `i32`，所以**必须**调用 `lock` 才能使用这个 `i32` 值。我们不能忘记这么做，因为如果没有获取锁，类型系统就不允许访问内部的 `i32` 值。

`Mutex<T>` 是一个智能指针。更准确的说，`lock` 调用会**返回**一个叫做 `MutexGuard` 的智能指针。`MutexGuard` 智能指针实现了 `Deref` 来指向其内部数据；它也实现了 `Drop`，当 `MutexGuard` 离开作用域时，自动释放锁。这样一来，就不会有忘记释放锁从而导致互斥器阻塞无法被其他线程使用的潜在风险，因为锁的释放是自动发生的。

#### 多个线程间共享 Mutex<T>
```rust
use std::rc::Rc;
use std::sync::Mutex;
use std::thread;

fn main() {
    let counter = Rc::new(Mutex::new(0));
    let mut handles = vec![];

    for _ in 0..10 {
        let counter = Rc::clone(&counter);
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

我们用智能指针 `Rc<T>` 来创建引用计数，使得一个值有了多个所有者。会产生编译错误。原因在于，`Rc<T>` 并不能安全的在线程间共享。当 `Rc<T>` 管理引用计数时，它必须在每一个 `clone` 调用时增加计数，并在每一个克隆体被丢弃时减少计数。`Rc<T>` 并没有使用任何并发原语，无法确保改变计数的操作不会被其他线程打断。这可能使计数出错，并导致诡异的 bug，比如可能会造成内存泄漏，或在使用结束之前就丢弃一个值。

我们所需要的是一个与 `Rc<T>` 完全一致，又以线程安全的方式改变引用计数的类型：原子引用计数 Arc<T>。原子类型就像基本类型一样，可以安全地在线程间共享。

```rust
use std::sync::{Arc, Mutex};
use std::thread;

fn main() {
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



## Send 和 Sync 的可扩展并发
#### Send 允许在线程间转移所有权
> `Send` trait 表明实现了 `Send` 的类型值的所有权可以在线程间传送。几乎所有的 Rust 类型都是`Send` 的。
>

> 不过有一些例外，包括 `Rc<T>`：这是不能实现 `Send` 的，因为如果克隆了 `Rc<T>` 的值并尝试将克隆的所有权转移到另一个线程，这两个线程都可能同时更新引用计数。为此，`Rc<T>` 被实现为用于单线程场景，这时不需要为拥有线程安全的引用计数而付出性能代价。
>

> 任何完全由 `Send` 的类型组成的类型也会自动被标记为 `Send`。几乎所有基本类型都是 `Send` 的，除了裸指针（raw pointer）。
>

#### Sync 允许多线程访问
> `Sync` trait 表明一个实现了 `Sync` 的类型可以安全的在多个线程中拥有其值的引用。换一种方式来说，对于任意类型 `T`，如果 `&T`实现了 `Send` 的话 `T` 就实现了 `Sync`，这意味着其引用就可以安全的发送到另一个线程。
>

> `Rc<T>` 也没有实现 `Sync`，出于其没有实现 `Send` 相同的原因。`RefCell<T>` 和 `Cell<T>` 系列类型没有实现 `Sync`。`RefCell<T>` 在运行时所进行的借用检查也不是线程安全的。`Mutex<T>` 实现了 `Sync`，正如 [“在多个线程间共享 Mutex<T>”](https://kaisery.github.io/trpl-zh-cn/ch16-03-shared-state.html#%E5%9C%A8%E5%A4%9A%E4%B8%AA%E7%BA%BF%E7%A8%8B%E9%97%B4%E5%85%B1%E4%BA%AB-mutext) 部分所讲的它可以被用来在多线程中共享访问。
>

> 类似于 `Send` 的情况，基本类型都实现了 `Sync`，完全由实现了 `Sync` 的类型组成的类型也实现了 `Sync`。
>

#### 手动实现 Send 和 Sync 是不安全的
通常并不需要手动实现 `Send` 和 `Sync` trait，因为完全由实现了 `Send` 和 `Sync` 的类型组成的类型，自动实现了 `Send` 和 `Sync`。因为它们是标记 trait，甚至都不需要实现任何方法。它们只是用来加强并发相关的不可变性的。

手动实现这些标记 trait 涉及到编写不安全的 Rust 代码，第二十章将会讲述具体的方法；当前重要的是，在创建新的由不是 `Send` 和 `Sync` 的部分构成的并发类型时需要多加小心，以确保维持其安全保证。

Rust 本身很少有处理并发的部分内容，有很多的并发方案都由 crate 实现。它们比标准库要发展的更快；请在网上搜索当前最新的用于多线程场景的 crate。

## Async 和 await
### 并行与并发
+ 并行：多个人干多个任务

<!-- 这是一张图片，ocr 内容为： -->
![](https://cdn.nlark.com/yuque/0/2025/svg/43050341/1764145424238-09c664b7-7f68-426f-89cc-2cb663d5fff7.svg)

+ 并发：一个人干多个任务

<!-- 这是一张图片，ocr 内容为： -->
![](https://cdn.nlark.com/yuque/0/2025/svg/43050341/1764145398438-e99d7ad4-6e81-4e69-8624-92b3ba2db9f2.svg)

### Futures 和 async 语法
> _future_ 是一个现在可能还没有准备好但将在未来某个时刻准备好的值。
>

> 在 Rust 中，我们称实现了 `Future` trait 的类型为 future。每个 future 会维护自身的进度状态信息以及对 “ready” 的定义。
>

> `async` 关键字可以用于代码块和函数，表明它们可以被中断并恢复。在一个 async 块或 async 函数中，可以使用 `await` 关键字来 _await 一个 future_（即等待其就绪）。async 块或 async 函数中每一个等待 future 的地方都可能是一个 async 块或 async 函数中断并随后恢复的点。检查一个 future 并查看其值是否已经准备就绪的过程被称为 _轮询_（polling）。
>

#### Future trait
```rust
use std::pin::Pin;
use std::task::{Context, Poll};

pub trait Future {
    type Output;

    fn poll(self: Pin<&mut Self>, cx: &mut Context<'_>) -> Poll<Self::Output>;
}
```

当你见到使用 `await` 的代码时，Rust 会在底层将其编译为调用 `poll` 的代码。

```rust
enum Poll<T> {
    Ready(T), // 表明 future 已经完成了其工作并且 T 的值是可用的。
    Pending, // 表明 future 仍然还有工作要进行，所有调用者稍后需要再次检查。
}
```

#### Pin 和 Unpin traits
> `Pin` 是一个类指针类型的封装，比如 `&`，`&mut`，`Box` 和 `Rc`。（从技术上来说，`Pin` 适用于实现了 `Deref` 或 `DerefMut` trait 的类型，不过这实际上等同于只能适用于指针。）`Pin` 本身并不是一个指针并且也不具备类似 `Rc` 和 `Arc` 那样引用计数的功能；它单纯地是一个编译器可以用来约束指针使用的工具。
>

通过 `await` 直接 await 一个 future 会隐式地 pin 住这个函数。这也就是为什么我们不需要在任何想要 await future 的地方使用 `pin!`。

这正是 Rust 的借用检查器所要求的：在安全代码中，禁止移动任何有自身活动引用的项。而 `Pin` 正是在此基础上，为我们提供了所需的保证。当我们通过 `Pin` 封装一个值的引用来 **pin** 住它时，它就无法再移动了。也就是说，如果有 `Pin<Box<SomeType>>`，你实际上 pin 住了 `SomeType` 的值，而**不是**`Box` 指针。

<!-- 这是一张图片，ocr 内容为：PINNED PIN FUT B1 -->
![](https://cdn.nlark.com/yuque/0/2025/png/43050341/1764154681665-5cd135da-663d-4c8d-946e-7832bb4eb280.png)

事实上，`Box` 指针仍然可以随意移动。请记住：我们关心确保最终被引用的数据保持不动。如果指针移动了，**但是它指向的数据还在相同的位置**。

然而，大多数类型即使被封装在 `Pin` 后面，也完全可以安全地移动。只有当项中含有内部引用的时候才需要考虑 pin。

`Unpin` 是一个标记 trait（marker trait），类似于 `Send` 和 `Sync` trait，因此它们自身没有功能。标记 trait 的存在只是为了告诉编译器在给定上下文中可以安全地使用实现了给定 trait 的类型。`Unpin` 告知编译器这个给定类型**无需**对所涉及的值是否可以安全地移动做出任何保证。

正如 `Send` 和 `Sync` 一样，编译器自动为所有被证明为安全的类型实现 `Unpin`。同样类似于 `Send` 和 `Sync`，有一个特殊的例子**不会**为类型实现 `Unpin`。这个例子的符号是 `impl !Unpin for _SomeType_`，这里 `_SomeType_` 指的是这样的一种类型：为了确保内存安全，当一个指向它的指针被用于 `Pin` 时，无论何时它都**必须**维护其不被移动的安全保证。

关于 `Pin` 与 `Unpin` 的关系有两点需要牢记。首先，`Unpin` 用于 “正常” 情况，而 `!Unpin` 用于特殊情况。其次，不管一个类型是实现了 `Unpin` 或者实现了 `!Unpin`，它**只在**你使用了一个被 pin 住的指向类似 `Pin<&mut _SomeType_>` 类型的指针时才会产生影响。

#### Stream trait
```rust
use std::pin::Pin;
use std::task::{Context, Poll};

trait Stream {
    type Item;

    fn poll_next(
        self: Pin<&mut Self>,
        cx: &mut Context<'_>
    ) -> Poll<Option<Self::Item>>;
}
```

`Stream` trait 定义了一个名为 `Item` 的关联类型来作为流所产生项的类型。

`Stream` 也定义了一个获取这些项的方法。名为 `poll_next`，来明确它以 `Future::poll` 同样的方式轮询并以 `Iterator::next` 同样的方式产生一系列的项。其返回类型用 `Option` 组合了 `Poll`。外部类型是 `Poll`，因为它必须检查可用性，就像 future 一样。内部类型是 `Option`，因为它需要表明是否有更多消息，就像迭代器一样。

### 消息传递
> 在 future 之间共享数据也与线程类似：我们会再次使用消息传递，不过这次使用的是异步版本的类型和函数。
>

```rust
let (tx, mut rx) = trpl::channel();

let vals = vec![
    String::from("hi"),
    String::from("from"),
    String::from("the"),
    String::from("future"),
];

for val in vals {
    tx.send(val).unwrap();
    trpl::sleep(Duration::from_millis(500)).await;
}

while let Some(value) = rx.recv().await {
    println!("received '{value}'");
}
```

`trpl::Receiver::recv` 则不会阻塞，因为它是异步的。不同于阻塞，它将控制权交还给运行时，直到接收到一个消息或者信道的发送端关闭。

### 流（Streams）：顺序的 Futures
> 流类似于一种异步形式的迭代器。它以与 `Iterator` 相同的方式提供下一个元素，但采用异步的方式实现。
>

Rust 中迭代器和流的相似性意味着我们实际上可以从任何迭代器上创建流。

```rust
trpl::run(async {
    let values = 1..101;
    let iter = values.map(|n| n * 2);
    let stream = trpl::stream_from_iter(iter);

    let mut filtered =
        stream.filter(|value| value % 3 == 0 || value % 5 == 0);

    while let Some(value) = filtered.next().await {
        println!("The value was: {value}");
    }
});
```

### future、任务和线程
线程作为同步操作集的边界；线程**之间**的并发是可能的。任务作为**异步**操作集的边界，任务**之间**和**之内**的并发是可能的，因为任务可以在其内部切换 future。最后，future 是 Rust 中最细粒度的并发单位，同时每一个 future 可能代表一棵由其它 future 组成的树。

当思考何时采用哪种方法时，考虑这些经验法则：

+ 如果工作是**非常可并行的**，例如处理大量数据其中每一部分数据都可以单独处理时，线程是更佳的选择。
+ 如果工作是**非常并发的**，例如处理大量不同来源的消息，它们可能有着不同的间隔或者速率，异步是更佳的选择。

## 模式与模式匹配
**模式**（_Patterns_）是 Rust 中一种特殊的语法，它用来匹配类型的结构，无论类型是简单还是复杂。结合使用模式和 `match` 表达式以及其他结构可以提供更多对程序控制流的支配权。模式由如下一些内容组合而成：

+ 字面值
+ 已解构的数组、枚举、结构体或者元组
+ 变量
+ 通配符
+ 占位符

所有可能会用到模式的位置：

+ match 分支
+ if let 条件表达式
+ for 循环
+ let 语句
+ 函数参数

匹配守卫：一个指定于 `match` 分支模式之后的额外 `if` 条件，它也必须被满足才能选择此分支。（仅 match 表达式中可用）。

```rust
let num = Some(4);

match num {
    Some(x) if x % 2 == 0 => println!("The number {x} is even"),
    Some(x) => println!("The number {x} is odd"),
    None => (),
}
```

@运算符：在创建一个存放值的变量的同时测试其值是否匹配模式。与 **匹配守卫** 相似。使用 `@` 可以在一个模式中同时测试和保存变量值。

```rust
enum Message {
    Hello { id: i32 },
}

let msg = Message::Hello { id: 10 };

match msg {
    Message::Hello { id: id @ 3..=7 } => println!("Found an id in range: {id}"),
    Message::Hello { id: 10..=12 } => {
        println!("Found an id in another range")
    }
    Message::Hello { id } => println!("Found some other id: {id}"),
}
```

