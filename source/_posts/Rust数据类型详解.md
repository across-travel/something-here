---
title: Rust数据类型详解
date: 2015-08-15 18:40
tags:
 - Rust
---

# 一：原生类型
- 布尔类型（bool）：有两个值true和false。
- 字符类型 (char)：表示单个Unicode值，存储为4个字节。Rust支持(u8)单字节字符b'H',仅限制于ASCII字符。
- 数值类型 (int/float)：有符号整数 (i8,i16,i32,i64,isize)、无符号整数 (u8,u16,u32,u64,usize) 浮点数(f32,f64).整形默认为i32，浮点型默认为f64.
- 字符串类型(str)：Unicode string slices.分为字符串切片&str和堆分配字符串String，字符串切片是静态分配，有固定大小，且不可变，堆分配字符串是可变的。Rust还支持单字节字符串b"Hello"和原始字节字符串使用br#"hello"#，仅限于ASCII字符,不需要对特殊字符进行转义。
- 指针 ( pointer )：最底层的是裸指针\*const T和\*mut T，但解引用它们是不安全的，必须放到unsafe块里。
- 函数 ( fn )：具有函数类型的变量实质上是一个函数指针。
- 元类型：即()，其唯一的值也是()。

# 二：复合类型
- 数组  ( array )：具有固定大小，并且元素都是同种类型，表示[T; N]，标准库动态的数组即向量 Vec。不多于32个元素的数组在值传递时是自动复制的.数组切片(&[T])：一个数组引用部分数据并且不需要拷贝，可表示为&[T]。
- 元组  ( tuple )：固定大小的有序列表，每个元素都有自己的类型，通过解构或者索引来获得元素的值。可使用==和!=运算符来判断是否相同，不多于12个元素的元组在值传递时是自动复制的。

# 三：自定义类型
- 结构体(struct) : 
- 枚举(enum) : 

# 四：标准库数据类型
- String : A UTF-8 encoded, growable string.
- Box : 
- Sequences: Vec, VecDeque, LinkedList
- Maps: HashMap, BTreeMap
- Sets: HashSet, BTreeSet
- Misc: BinaryHeap

## 字符串

一个*字符串*是一串UTF-8字节编码的Unicode量级值的序列。所有的字符串都确保是有效编码的UTF-8序列。另外，字符串并不以null结尾并且可以包含null字节。Rust有两种主要的字符串类型：`&str`和`String`。让我们先看看`&str`。这叫做*字符串片段*（*string slices*）。字符串常量是`&'static str`类型的：

```rust
let greeting = "Hello there."; // greeting: &'static str
```

`"Hello there."`是一个字符串常量而它的类型是`&'static str`。字符串常量是静态分配的字符串切片，也就是说它储存在我们编译好的程序中，并且整个程序的运行过程中一直存在。这个`greeting`绑定了一个静态分配的字符串的引用。任何接受一个字符串切片的函数也接受一个字符串常量。

字符串常量可以跨多行。有两种形式。第一种会包含新行符和之前的空格：

```rust
let s = "foo
    bar";

assert_eq!("foo\n        bar", s);
```

第二种，带有`\`，会去掉空格和新行符：

```rust
let s = "foo\
    bar";

assert_eq!("foobar", s);
```

Rust 当然不仅仅只有`&str`。一个`String`，是一个在堆上分配的字符串。这个字符串可以增长，并且也保证是UTF-8编码的。`String`通常通过一个字符串片段调用`to_string`方法转换而来。

```rust
let mut s = "Hello".to_string(); // mut s: String
println!("{}", s);

s.push_str(", world.");
println!("{}", s);
```

`String`可以通过一个`&`强制转换为`&str`：

```rust
fn takes_slice(slice: &str) {
    println!("Got: {}", slice);
}

fn main() {
    let s = "Hello".to_string();
    takes_slice(&s);
}
```

这种强制转换并不发生在接受`&str`的trait而不是`&str`本身作为参数的函数上。例如，[TcpStream::connect](http://doc.rust-lang.org/stable/std/net/struct.TcpStream.html#method.connect)，有一个`ToSocketAddrs`类型的参数。`&str`可以不用转换不过`String`必须使用`&*`显式转换。

```rust
use std::net::TcpStream;

TcpStream::connect("192.168.0.1:3000"); // &str parameter

let addr_string = "192.168.0.1:3000".to_string();
TcpStream::connect(&*addr_string); // convert addr_string to &str
```

把`String`转换为`&str`的代价很小，不过从`&str`转换到`String`涉及到分配内存。除非必要，没有理由这样做！

## 索引（Indexing）

因为字符串是有效UTF-8编码的，它不支持索引：

```rust
let s = "hello";

println!("The first letter of s is {}", s[0]); // ERROR!!!
```

通常，用`[]`访问一个数组是非常快的。不过，字符串中每个UTF-8编码的字符可以是多个字节，你必须遍历字符串来找到字符串的第N个字符。这个操作的代价相当高，而且我们不想误导读者。更进一步来讲，Unicode实际上并没有定义什么“字符”。我们可以选择把字符串看作一个串独立的字节，或者代码点（codepoints）：

```rust
let hachiko = "忠犬ハチ公";

for b in hachiko.as_bytes() {
    print!("{}, ", b);
}

println!("");

for c in hachiko.chars() {
    print!("{}, ", c);
}

println!("");
```

这会打印出：

```text
229, 191, 160, 231, 138, 172, 227, 131, 143, 227, 131, 129, 229, 133, 172,
忠, 犬, ハ, チ, 公,
```

如你所见，这有比`char`更多的字节。

你可以这样来获取跟索引相似的东西：

```rust
# let hachiko = "忠犬ハチ公";
let dog = hachiko.chars().nth(1); // kinda like hachiko[1]
```

这强调了我们不得不遍历整个`char`的列表。

你可以使用切片语法来获取一个字符串的切片：

```rust
let dog = "hachiko";
let hachi = &dog[0..5];
```

注意这里是*字节*偏移，而不是*字符*偏移。所以如下代码在运行时会失败：

```rust
let dog = "忠犬ハチ公";
let hachi = &dog[0..2];
```

给出如下错误：

```text
thread '<main>' panicked at 'index 0 and/or 2 in `忠犬ハチ公` do not lie on
character boundary'
```

### 拼接字符串：(结果是String)
1. &str不能直接拼接，
2. 拼接的首个字符串必须是String，之后的String需要一个 & 转换成 &str，这个功能叫做 Deref  转换。
3. 满足1，2条件的String与&str可以多个交叉拼接。

这是因为`&String`可以自动转换为一个`&str`。这个功能叫做** `Deref`转换**。

```rust
let    hello      =  "Hello".to_string();        // String
let    hello_str  =  "Hello!";                 // &'static str
let    world      =  "World!".to_string();      
let    world_str  =  "World!";             
let    tom        =  "tom!".to_string();  
let    tom_str    =  "tom!";
let    lucy       =  "lucy!".to_string();  
let    lucy_str   =  "lucy!";
let    lilei      =  "lilei!".to_string();  
let    lilei_str  =  "lilei!";

let    hello_world  =  hello + world_str;
let    hello_world  =  hello + &world;
let    hello_world  =  hello + world_str + tom_str + lucy_str;
let    hello_world  =  hello + &world + &tom + &lucy;
let    hello_world  =  hello + hello_str + &world + world_str + tom_str + &tom + &lucy + &lilei + lucy_str + lilei_str;
```


## 元组（Tuples）
元组（tuples）是固定大小的有序列表。如下：

```rust
let x = (1, "hello");
let x: (i32, &str) = (1, "hello");

let mut x = (1, 2); // x: (i32, i32)
let y = (2, 3); // y: (i32, i32)
let	e	=	x.1;
x = y;
```

你可以通过一个*解构let*（*destructuring let*）访问元组中的字段。下面是一个例子：

```rust
let (x, y, z) = (1, 2, 3);

println!("x is {}", x);
```

可以在`let`左侧写一个模式，如果它能匹配右侧的话，我们可以一次写多个绑定。这种情况下，`let`“解构”或“拆开”了元组，并分成了三个绑定。

你可以一个逗号来消除一个单元素元组和一个括号中的值的歧义：

```rust
(0,); // single-element tuple
(0); // zero 
```

### 元组索引（Tuple Indexing）
你也可以用索引语法访问一个元组的字段,它使用`.`

```rust
let tuple = (1, 2, 3);

let x = tuple.0;
println!("x is {}", x);
```

### 数组(array)与Vec

**定义数组**：
```rust
let mut array: [i32; 3] = [0; 3];   // 方式一
let arr = [8,	9,	10];            // 方式二
array[1] = 1;
array[2] = 2;

assert_eq!([1, 2], &array[1..]);

// This loop prints: 0 1 2
for x in &array {
    print!("{} ", x);
}
```

 Vec : pronounced 'Vector'是一个动态或“可增长”的数组，被实现为标准库类型[`Vec<T>`](http://doc.rust-lang.org/std/Vec/)（其中`<T>`是一个[泛型](Generics 泛型.md)语句）。Vec总是在堆上分配数据。Vec与数组切片就像`String`与`&str`一样。你可以使用`vec!`宏来创建它：

```rust
let v1: Vec<i32> = Vec::new();    // 方式一
let v = vec![1, 2, 3, 4, 5];    // v: Vec<i32>   方式二
let	v2 = vec![0;10];	        //	声明一个初始长度为10的值全为0的动态数组
```

（与我们之前使用`println!`宏时不一样，我们在`vec!`中使用中括号`[]`。为了方便，Rust 允许你使用上述各种情况。）

对于重复初始值有另一种形式的`vec!`：

```rust
let v = vec![0; 10]; // ten zeroes
```

#### 访问元素
为了Vec特定索引的值，我们使用`[]`,索引从`0`开始，所以第3个元素是`v[2]`.

```rust
let v = vec![1, 2, 3, 4, 5];
let r0 = v[1];
let r1 = &v[2];             //当引用一个不存在的元素时，会造成panic!.
let r2 = v.get(3);           //当引用一个不存在的元素时，返回None.
println!("The third element of v is{}, {},{}", r0, r1,r2);
```

另外值得注意的是你必须用`usize`类型的值来索引：

```rust
let v = vec![1, 2, 3, 4, 5];

let i: usize = 0;
let j: i32 = 0;

// works
v[i];

// doesn’t
v[j];
```

用非`usize`类型索引的话会给出类似如下的错误：

```text
error: the trait `core::ops::Index<i32>` is not implemented for the type
`collections::Vec::Vec<_>` [E0277]
v[j];
^~~~
note: the type `collections::Vec::Vec<_>` cannot be indexed by `i32`
error: aborting due to previous error
```

信息中有很多标点符号，不过核心意思是：你不能用`i32`来索引。

#### 越界访问（Out-of-bounds Access）

如果你尝试访问并不存在的索引：

```rust
let v = vec![1, 2, 3];
println!("Item 7 is {}", v[7]);
```

那么当前的线程会**panic**并输出如下信息：

```text
thread '<main>' panicked at 'index out of bounds: the len is 3 but the index is 7'
```

如果你想处理越界错误而不是 panic，你可以使用像[`get`](http://doc.rust-lang.org/std/Vec/struct.Vec.html#method.get)或[`get_mut`](http://doc.rust-lang.org/std/Vec/struct.Vec.html#method.get)这样的方法，他们当给出一个无效的索引时返回`None`：

```rust
let v = vec![1, 2, 3];
match v.get(7) {
    Some(x) => println!("Item 7 is {}", x),
    None => println!("Sorry, this Vector is too short.")
}
```

使用枚举来储存多种类型
```rust

enum SpreadsheetCell {
    Int(i32),
    Float(f64),
    Text(String),
}

let row = vec![
    SpreadsheetCell::Int(3),
    SpreadsheetCell::Text(String::from("blue")),
    SpreadsheetCell::Float(10.12),
];
```

#### 迭代
当你有了一个Vector，我可以用`for`来迭代它的元素。有3个版本：

```rust
let mut v = vec![1, 2, 3, 4, 5];

for i in &v {
    println!("A reference to {}", i);
}

for i in &mut v {
    println!("A mutable reference to {}", i);
}

for i in v {
    println!("Take ownership of the Vector and its element {}", i);
}
```

Vector还有很多有用的方法，你可以看看[Vector的API文档](http://doc.rust-lang.org/nightly/std/Vec/)了解它们。

## 数组切片（Slices）
一个*切片*（*slice*）是一个数组的引用（或者“视图”）。它有利于安全，有效的访问数组的一部分而不用进行拷贝。比如，你可能只想要引用读入到内存的文件中的一行。原理上，片段并不是直接创建的，而是引用一个已经存在的变量。片段有预定义的长度，可以是可变也可以是不可变的。

### 切片语法

你可以用一个`&`和`[]`的组合从多种数据类型创建一个切片。`&`表明切片类似于**引用**，带有一个范围的`[]`，允许你定义切片的长度：

```rust
let a = [0, 1, 2, 3, 4];
let complete = &a[..]; // A slice containing all of the elements in a
let middle = &a[1..4]; // A slice of a: just the elements 1, 2, and 3
```

片段拥有`&[T]`类型。当我们涉及到**泛型**时会讨论这个`T`。

你可以在[标准库文档](http://doc.rust-lang.org/stable/std/primitive.slice.html)中找到更多关于`slices`的文档。


## 函数
函数也有一个类型！它们看起来像这样：

```rust
fn foo(x: i32) -> i32 { x }

let x: fn(i32) -> i32 = foo;
```

在这个例子中，`x`是一个“函数指针”，指向一个获取一个`i32`参数并返回一个`i32`值的函数。

## 表达式 VS 语句
Rust 主要是一个基于表达式的语言。只有两种语句，其它的一切都是表达式。

表达式返回一个值，而语句不是。这就是为什么这里我们以“不是所有控制路径都返回一个值”结束：`x + 1;`语句不返回一个值。Rust 中有两种类型的语句：“声明语句”和“表达式语句”。其余的一切是表达式。让我们先讨论下声明语句。

在 Rust 中，使用`let`引入一个绑定并*不是*一个表达式。下面的代码会产生一个编译时错误：

```rust
let x = (let y = 5); // expected identifier, found keyword `let`
```

编译器告诉我们这里它期望看到表达式的开头，而`let`只能开始一个语句，不是一个表达式。

注意赋值一个已经绑定过的变量（例如，`y = 5`）仍是一个表达式，即使它的（返回）值并不是特别有用。不像其它语言中赋值语句返回它赋的值（例如，前面例子中的`5`），在 Rust 中赋值的值是一个空的元组`()`：

```rust
let mut y = 5;

let x = (y = 6);  // x has the value `()`, not `6`
```

Rust中第二种语句是*表达式语句*。它的目的是把任何表达式变为语句。在实践环境中，Rust 语法期望语句后跟其它语句。这意味着你用分号来分隔各个表达式。这意味着Rust看起来很像大部分其它使用分号做为语句结尾的语言，并且你会看到分号出现在几乎每一行你看到的 Rust 代码。

那么我们说“几乎”的例外是神马呢？你已经见过它了，在这些代码中：

```rust
fn add_one(x: i32) -> i32 {
    x + 1
}
```

我们的函数声称它返回一个`i32`，不过带有一个分号，它会返回一个`()`。Rust意识到这可能不是我们想要的，并在我们之前看到的错误中建议我们去掉分号。

## 提早返回（Early returns）
不过提早返回,Rust确实有这么一个关键字，`return`,使用`return`作为函数的最后一行是可行的，不过被认为是一个糟糕的风格：

```rust
fn foo(x: i32) -> i32 {
    return x;

    // we never run this code!
    x + 1
}
```

## 发散函数（Diverging functions）
Rust有些特殊的语法叫“发散函数”，这些函数并不返回：

```rust
fn diverges() -> ! {
    panic!("This function never returns!");
}
```

`panic!`是一个宏，类似我们已经见过的`println!()`。与`println!()`不同的是，`panic!()`导致当前的执行线程崩溃并返回指定的信息。因为这个函数会崩溃，所以它不会返回，所以它拥有一个类型`!`，它代表“发散”。

如果你添加一个叫做`diverges()`的函数并运行，你将会得到一些像这样的输出：

```text
thread ‘<main>’ panicked at ‘This function never returns!’, hello.rs:2
```

如果你想要更多信息，你可以设定`RUST_BACKTRACE`环境变量来获取 backtrace ：

```bash
$ RUST_BACKTRACE=1 ./diverges
thread '<main>' panicked at 'This function never returns!', hello.rs:2
stack backtrace:
   1:     0x7f402773a829 - sys::backtrace::write::h0942de78b6c02817K8r
   2:     0x7f402773d7fc - panicking::on_panic::h3f23f9d0b5f4c91bu9w
   3:     0x7f402773960e - rt::unwind::begin_unwind_inner::h2844b8c5e81e79558Bw
   4:     0x7f4027738893 - rt::unwind::begin_unwind::h4375279447423903650
   5:     0x7f4027738809 - diverges::h2266b4c4b850236beaa
   6:     0x7f40277389e5 - main::h19bb1149c2f00ecfBaa
   7:     0x7f402773f514 - rt::unwind::try::try_fn::h13186883479104382231
   8:     0x7f402773d1d8 - __rust_try
   9:     0x7f402773f201 - rt::lang_start::ha172a3ce74bb453aK5w
  10:     0x7f4027738a19 - main
  11:     0x7f402694ab44 - __libc_start_main
  12:     0x7f40277386c8 - <unknown>
  13:                0x0 - <unknown>
```

`RUST_BACKTRACE`也可以用于 Cargo 的`run`命令：

```bash
$ RUST_BACKTRACE=1 cargo run
     Running `target/debug/diverges`
thread '<main>' panicked at 'This function never returns!', hello.rs:2
stack backtrace:
   1:     0x7f402773a829 - sys::backtrace::write::h0942de78b6c02817K8r
   2:     0x7f402773d7fc - panicking::on_panic::h3f23f9d0b5f4c91bu9w
   3:     0x7f402773960e - rt::unwind::begin_unwind_inner::h2844b8c5e81e79558Bw
   4:     0x7f4027738893 - rt::unwind::begin_unwind::h4375279447423903650
   5:     0x7f4027738809 - diverges::h2266b4c4b850236beaa
   6:     0x7f40277389e5 - main::h19bb1149c2f00ecfBaa
   7:     0x7f402773f514 - rt::unwind::try::try_fn::h13186883479104382231
   8:     0x7f402773d1d8 - __rust_try
   9:     0x7f402773f201 - rt::lang_start::ha172a3ce74bb453aK5w
  10:     0x7f4027738a19 - main
  11:     0x7f402694ab44 - __libc_start_main
  12:     0x7f40277386c8 - <unknown>
  13:                0x0 - <unknown>
```

发散函数可以被用作任何类型：

```rust
fn diverges() -> ! {
   panic!("This function never returns!");
}
let x: i32 = diverges();
let x: String = diverges();
```

## 函数指针

我们也可以创建指向函数的变量绑定：

```rust
let f: fn(i32) -> i32;
```

`f`是一个指向一个获取`i32`作为参数并返回`i32`的函数的变量绑定。例如：

```rust
fn plus_one(i: i32) -> i32 {
    i + 1
}

// without type inference
let f: fn(i32) -> i32 = plus_one;

// with type inference
let f = plus_one;
```

你可以用`f`来调用这个函数：

```rust
# fn plus_one(i: i32) -> i32 { i + 1 }
# let f = plus_one;
let six = f(5);
```
## 高阶函数
  高阶函数与普通函数的不同在于，它可以使用一个或多个函数作为参数，可以将函数作为返回值。rust的函数是first class type，所以支持高阶函数。而，由于rust是一个强类型的语言，如果要将函数作为参数或返回值，首先需要搞明白函数的类型。下面先说函数的类型，再说函数作为参数和返回值。

#### 函数类型
  前面说过，关键字`fn`可以用来定义函数。除此以外，它还用来构造函数类型。与函数定义主要的不同是，构造函数类型不需要函数名、参数名和函数体。在Rust Reference中的描述如下：
  > The function type constructor fn forms new function types. A function type consists of a possibly-empty set of function-type modifiers (such as unsafe or extern), a sequence of input types and an output type.

  来看一个简单例子：
  ```rust
fn inc(n: i32) -> i32 {//函数定义
  n + 1
}

type IncType = fn(i32) -> i32;//函数类型

fn main() {
  let func: IncType = inc;
  println!("3 + 1 = {}", func(3));
}
  ```
  上例首先使用`fn`定义了`inc`函数，它有一个`i32`类型参数，返回`i32`类型的值。然后再用`fn`定义了一个函数类型，这个函数类型有i32类型的参数和i32类型的返回值，并用`type`关键字定义了它的别名`IncType`。在`main`函数中定义了一个变量`func`，其类型就为`IncType`，并赋值为`inc`，然后在`pirntln`宏中调用：`func(3)`。可以看到，`inc`函数的类型其实就是`IncType`。  
  这里有一个问题，我们将`inc`赋值给了`func`，而不是`&inc`，这样是将`inc`函数的拥有权转给了`func`吗，赋值后还可以以`inc()`形式调用`inc`函数吗？先来看一个例子：
  ```rust
fn main() {
  let func: IncType = inc;
  println!("3 + 1 = {}", func(3));
  println!("3 + 1 = {}", inc(3));
}

type IncType = fn(i32) -> i32;

fn inc(n: i32) -> i32 {
  n + 1
}
  ```
  我们将上例保存在rs源文件中，再用rustc编译，发现并没有报错，并且运行也得到我们想要的结果：
  ```
3 + 1 = 4
3 + 1 = 4
  ```
  这说明，赋值时，`inc`函数的所有权并没有被转移到`func`变量上，而是更像不可变引用。在rust中，函数的所有权是不能转移的，我们给函数类型的变量赋值时，赋给的一般是函数的指针，所以rust中的函数类型，就像是C/C++中的函数指针，当然，rust的函数类型更安全。可见，rust的函数类型，其实应该是属于指针类型（Pointer Type）。rust的Pointer Type有两种，一种为引用（Reference`&`），另一种为原始指针（Raw pointer `*`），详细内容请看[Rust Reference 8.18 Pointer Types](http://doc.rust-lang.org/reference.html#pointer-types)。而rust的函数类型应是引用类型，因为它是安全的，而原始指针则是不安全的，要使用原始指针，必须使用`unsafe`关键字声明。

#### 函数作为参数
  函数作为参数，其声明与普通参数一样。看下例：
  ```rust
fn main() {
  println!("3 + 1 = {}", process(3, inc));
  println!("3 - 1 = {}", process(3, dec));
}

fn inc(n: i32) -> i32 {
  n + 1
}

fn dec(n: i32) -> i32 {
  n - 1
}

fn process(n: i32, func: fn(i32) -> i32) -> i32 {
  func(n)
}
  ```
  例子中，`process`就是一个高阶函数，它有两个参数，一个类型为`i32`的`n`，另一个类型为`fn(i32)->i32`的函数`func`，返回一个`i32`类型的参数；它在函数体内以`n`作为参数调用`func`函数，返回`func`函数的返回值。运行可以得到以下结果：
  ```
3 + 1 = 4
3 - 1 = 2
  ```
  不过，这不是函数作为参数的唯一声明方法，使用泛型函数配合特质（`trait`）也是可以的，因为rust的函数都会实现一个`trait`:`FnOnce`、`Fn`或`FnMut`。将上例中的`process`函数定义换成以下形式是等价的：
  ```rust
fn process<F>(n: i32, func: F) -> i32
    where F: Fn(i32) -> i32 {
    func(n)
}
  ```

#### 函数作为返回值
  函数作为返回值，其生命与普通函数的返回值类型声明一样。看例子：
  ```rust
fn main() {
   let a = [1,2,3,4,5,6,7];
   let mut b = Vec::<i32>::new();
   for i in &a {
       b.push(get_func(*i)(*i));
   }
   println!("{:?}", b);
}

fn get_func(n: i32) -> fn(i32) -> i32 {
    fn inc(n: i32) -> i32 {
        n + 1
    }
    fn dec(n: i32) -> i32 {
        n - 1
    }
    if n % 2 == 0 {
        inc
    } else {
        dec
    }
}
  ```
  例子中的高阶函数为`get_func`，它接收一个i32类型的函数，返回一个类型为`fn(i32) -> i32`的函数，若传入的参数为偶数，返回`inc`，否则返回`dec`。这里需要注意的是，`inc`函数和`dec`函数都定义在`get_func`内。在函数内定义函数在很多其他语言中是不支持的，不过rust支持，这也是rust灵活和强大的一个体现。不过，在函数中定义的函数，不能包含函数中（环境中）的变量，若要包含，应该闭包。

#### 返回多个值

rust的函数不支持多返回值,但是我们可以利用元组来返回多个值,配合rust的模式匹配,使用起来十分灵活。先看例子:
```rust
fn main() {
    let (p2,p3) = pow_2_3(789);
    println!("pow 2 of 789 is {}.", p2);
	println!("pow 3 of 789 is {}.", p3);
}
fn pow_2_3(n: i32) -> (i32, i32) {
	(n*n, n*n*n)
}
```
可以看到,上例中, pow_2_3函数接收一个i32类型的值,返回其二次方和三次方的值,这两个值包装在一个元组中返回。在 main函数中, let语句就可以使用模式匹配将函数返回的元组进行解构,将这两个返回值分别赋给 p2和p3,从而可以得到 789二次方的值和三次方的值。

# 二：复合类型

## 1. 结构体

结构体是一个创建更复杂数据类型的方法,大写字母开头并且驼峰命名法.结构体中的值默认是不可变的.

```rust
struct Point {
    x: i32,
    y: i32,
}

fn main() {
    let origin = Point { x: 0, y: 0 }; // origin: Point
    let mut do = Point { x: 1, y: 2 };
    do.x = 5;

    println!("The origin is at ({}, {})", origin.x, origin.y);
    println!("The origin is at ({}, {})", do.x, do.y);
}
```

Rust 在语言级别不支持字段可变性，可变性是绑定的一个属性，不是结构体自身的。如果你习惯于字段级别的可变性，这开始可能看起来有点奇怪，不过这样明显地简化了问题。它甚至可以让你使变量只可变一段临时时间：

```rust
struct Point {
    x: i32,
    y: i32,
}

fn main() {
    let mut point = Point { x: 0, y: 0 };

    point.x = 5;

    let point = point; // now immutable

    point.y = 6; // this causes an error
}
```

你的结构体仍然可以包含`&mut`指针，它会给你一些类型的可变性：

```rust
struct Point {
    x: i32,
    y: i32,
}

struct PointRef<'a> {
    x: &'a mut i32,
    y: &'a mut i32,
}

fn main() {
    let mut point = Point { x: 0, y: 0 };

    {
        let r = PointRef { x: &mut point.x, y: &mut point.y };

        *r.x = 5;
        *r.y = 6;
    }

    assert_eq!(5, point.x);
    assert_eq!(6, point.y);
}
```

## 更新语法（Update syntax）
一个包含`..`的`struct`表明你想要使用一些其它结构体的拷贝的一些值。例如：

```rust
struct Point3d {
    x: i32,
    y: i32,
    z: i32,
}

let mut point = Point3d { x: 0, y: 0, z: 0 };
point = Point3d { y: 1, .. point };
```

这给了`point`一个新的`y`，不过保留了`x`和`z`的值。这也并不必要是同样的`struct`，你可以在创建新结构体时使用这个语法，并会拷贝你未指定的值：

```rust
# struct Point3d {
#     x: i32,
#     y: i32,
#     z: i32,
# }
let origin = Point3d { x: 0, y: 0, z: 0 };
let point = Point3d { z: 1, x: 2, .. origin };
```

## 元组结构体
Rust有像另一个[元组](Primitive Types 原生类型.md#tuples)和结构体的混合体的数据类型。元组结构体有一个名字，不过它的字段没有。他们用`struct`关键字声明，并元组前面带有一个名字：

```rust
struct Color(i32, i32, i32);
struct Point(i32, i32, i32);

//这里`black`和`origin`并不相等，即使它们有一模一样的值：
let black = Color(0, 0, 0);
let origin = Point(0, 0, 0);
```

使用结构体几乎总是好于使用元组结构体。不过有种情况元组结构体非常有用，就是当元组结构体只有一个元素时。我们管它叫*新类型*（*newtype*），因为你创建了一个与元素相似的类型：

```rust
struct Inches(i32);

let length = Inches(10);

let Inches(integer_length) = length;
println!("length is {} inches", integer_length);
```

如你所见，你可以通过一个解构`let`来提取内部的整型，就像我们在讲元组时说的那样，`let Inches(integer_length)`给`integer_length`赋值为`10`。

## 类单元结构体（Unit-like structs）
你可以定义一个没有任何成员的结构体：

```rust
struct Electron {} // Use empty braces...
struct Proton;     // ...or just a semicolon.

// Use the same notation when creating an instance.
let x = Electron {};
let y = Proton;
```

这样的结构体叫做“类单元”因为它与一个空元组类似，`()`，这有时叫做“单元”。就像一个元组结构体，它定义了一个新类型。

就它本身来看没什么用（虽然有时它可以作为一个标记类型），不过在与其它功能的结合中，它可以变得有用。例如，一个库可能请求你创建一个实现了一个特定特性的结构来处理事件。如果你并不需要在结构中存储任何数据，你可以仅仅创建一个类单元结构体。

## 2. 枚举

Rust 中的一个`enum`是一个代表数个可能变量的数据的类型。每个变量都可选是否关联数据：

```rust
enum Message {
    Quit,
    ChangeColor(i32, i32, i32),
    Move { x: i32, y: i32 },
    Write(String),
}
```

定义变量的语法与用来定义结构体的语法类似：你可以有不带数据的变量（像类单元结构体），带有命名数据的变量，和带有未命名数据的变量（像元组结构体）。然而，不像单独的结构体定义，一个`enum`是一个单独的类型。一个枚举的值可以匹配任何一个变量。因为这个原因，枚举有时被叫做“集合类型”：枚举可能值的集合是每一个变量可能值的集合的总和。

我们使用`::`语法来使用每个变量的名字：它们包含在`enum`名字自身中。这样的话，以下的情况都是可行的：

```rust
enum Message {
     Move { x: i32, y: i32 },
}
let x: Message = Message::Move { x: 3, y: 4 };

enum BoardGameTurn {
    Move { squares: i32 },
    Pass,
}

let y: BoardGameTurn = BoardGameTurn::Move { squares: 1 };
```

这两个变量都叫做`Move`，不过他们包含在枚举名字中，他们可以无冲突的使用。

枚举类型的一个值包含它是哪个变量的信息，以及任何与变量相关的数据。这有时被作为一个“标记的联合”被提及。因为数据包括一个“标签”表明它的类型是什么。编译器使用这个信息来确保安全的访问枚举中的数据。例如，我们不能简单的尝试解构一个枚举值，就像它是其中一个可能的变体那样：

```rust
fn process_color_change(msg: Message) {
    let Message::ChangeColor(r, g, b) = msg; // compile-time error
}
```

不支持这些操作（比较操作）可能看起来更像限制。不过这是一个我们可以克服的限制。有两种方法：我们自己实现相等（比较），或通过[`match` ](Match 匹配.md)表达式模式匹配变量，你会在下一部分学到它。我们还不够了解Rust如何实现相等，不过我们会在[特性](Traits.md)找到它们。

## 构造器作为函数（Constructors as functions）
一个枚举的构造器总是可以像函数一样使用。例如：

```rust
enum Message {
Write(String),
}
let m = Message::Write("Hello, world".to_string());
```

与下面是一样的：

```rust
enum Message {
Write(String),
}
fn foo(x: String) -> Message {
    Message::Write(x)
}

let x = foo("Hello, world".to_string());
```

这对我们没有什么直接的帮助，直到我们要用到[闭包](Closures 闭包.md)时，这时我们要考虑将函数作为参数传递给其他函数。例如，使用[迭代器](Iterators 迭代器.md)，我们可以这样把一个`String`的Vector转换为一个`Message::Write`的Vector：

```rust
enum Message {
Write(String),
}

let v = vec!["Hello".to_string(), "World".to_string()];

let v1: Vec<Message> = v.into_iter().map(Message::Write).collect();
```

# 三：不定长类型

大部分类型有一个特定的大小，以字节为单位，它们在编译时是已知的。例如，一个`i32`是32位大，或者4个字节。然而，有些类型有益于表达，却没有一个定义的大小。它们叫做“不定长”或者“动态大小”类型。一个例子是`[T]`。这个类型代表一个特定数量`t`的序列。不过我们并不知道有多少，所以大小是未知的。

Rust知道几个这样的类型，不过它们有一些限制。这有三个：

1. 我们只能通过指针操作一个不定长类型的实例。`&[T]`刚好能正常工作，不过`[T]`不行。一个` &[T]`能正常工作，不过一个`[T]`不行。
2. 变量和参数不能拥有动态大小类型。
3. 只有一个`struct`的最后一个字段可能拥有一个动态大小类型；其它字段则不可以拥有动态大小类型。枚举变量不可以用动态大小类型作为数据。

所以为什么这很重要？好吧，因为`[T]`只能用在一个指针之后，如果我们没有对不定长类型的语言支持，它将不可能这么写：

```rust
impl Foo for str {
```

或者

```rust
impl<T> Foo for [T] {
```

相反，你将不得不这么写：

```rust
impl Foo for &str {
```

意味深长的是，这个实现将只能用于[引用](References and Borrowing 引用和借用.md)，并且不能用于其它类型的指针。通过`impl for str`，所有指针，包括（在一些地方，这里会有bug需要修复）用户自定义的智能指针，可以使用这个`impl`。

## `?Sized`
如果你想要写一个接受动态大小类型的函数，你可以使用这个特殊的限制，`?Sized`：

```rust
struct Foo<T: ?Sized> {
    f: T,
}
```
这个`?`，读作“`T`可能是`Sized`的”，意味着这个限制是特殊的：它让我们的匹配更宽松，而不是相反。这几乎像每个`T`都隐式拥有` T: Sized`一样，`?`放松了这个默认（限制）。

# 四：关联类型

关联类型是Rust类型系统中非常强大的一部分。它涉及到‘类型族’的概念，换句话说，就是把多种类型归于一类。这个描述可能比较抽象，所以让我们深入研究一个例子。如果你想编写一个`Graph`trait，你需要泛型化两个类型：点类型和边类型。所以你可能会像这样写一个trait，`Graph<N, E>`：

```rust
trait Graph<N, E> {
    fn has_edge(&self, &N, &N) -> bool;
    fn edges(&self, &N) -> Vec<E>;
    // etc
}
```

虽然这可以工作，不过显得很尴尬，例如，任何需要一个`Graph`作为参数的函数都需要泛型化的`N`ode和`E`dge类型：

```rust
fn distance<N, E, G: Graph<N, E>>(graph: &G, start: &N, end: &N) -> u32 { ... }
```

我们的距离计算并不需要`Edge`类型，所以函数签名中`E`只是写着玩的。

我们需要的是对于每一种`Graph`类型，都使用一个特定的的`N`ode和`E`dge类型。我们可以用关联类型来做到这一点：

```rust
trait Graph {
    type N;
    type E;

    fn has_edge(&self, &Self::N, &Self::N) -> bool;
    fn edges(&self, &Self::N) -> Vec<Self::E>;
    // etc
}
```

现在，我们使用一个抽象的`Graph`了：

```rust
fn distance<G: Graph>(graph: &G, start: &G::N, end: &G::N) -> uint { ... }
```

这里不再需要处理`E`dge类型了。

让我们更详细的回顾一下。

## 定义关联类型
让我们构建一个`Graph`trait。这里是定义：

```rust
trait Graph {
    type N;
    type E;

    fn has_edge(&self, &Self::N, &Self::N) -> bool;
    fn edges(&self, &Self::N) -> Vec<Self::E>;
}
```

十分简单。关联类型使用`type`关键字，并出现在trait体和函数中。

这些`type`声明跟函数定义一样。例如，如果我们想`N`类型实现`Display`，这样我们就可以打印出点类型，我们可以这样写：

```rust
use std::fmt;

trait Graph {
    type N: fmt::Display;
    type E;

    fn has_edge(&self, &Self::N, &Self::N) -> bool;
    fn edges(&self, &Self::N) -> Vec<Self::E>;
}
```

## 实现关联类型

就像任何 trait，使用关联类型的 trait 用`impl`关键字来提供实现。下面是一个`Graph`的简单实现：

```rust
# trait Graph {
#     type N;
#     type E;
#     fn has_edge(&self, &Self::N, &Self::N) -> bool;
#     fn edges(&self, &Self::N) -> Vec<Self::E>;
# }
struct Node;

struct Edge;

struct MyGraph;

impl Graph for MyGraph {
    type N = Node;
    type E = Edge;

    fn has_edge(&self, n1: &Node, n2: &Node) -> bool {
        true
    }

    fn edges(&self, n: &Node) -> Vec<Edge> {
        Vec::new()
    }
}
```

这个可笑的实现总是返回`true`和一个空的`Vec<Edge>`，不过它提供了如何实现这类 trait 的思路。首先我们需要3个`struct`，一个代表图，一个代表点，还有一个代表边。如果使用别的类型更合理，也可以那样做，我们只是准备使用`struct`来代表这 3 个类型。

接下来是`impl`行，它就像其它任何 trait 的实现。

在这里，我们使用`=`来定义我们的关联类型。trait 使用的名字出现在`=`的左边，而我们`impl`的具体类型出现在右边。最后，我们在函数声明中使用具体类型。

## trait 对象和关联类型

这里还有另外一个我们需要讨论的语法：trait对象。如果你创建一个关联类型的trait对象，像这样：

```rust
# trait Graph {
#     type N;
#     type E;
#     fn has_edge(&self, &Self::N, &Self::N) -> bool;
#     fn edges(&self, &Self::N) -> Vec<Self::E>;
# }
# struct Node;
# struct Edge;
# struct MyGraph;
# impl Graph for MyGraph {
#     type N = Node;
#     type E = Edge;
#     fn has_edge(&self, n1: &Node, n2: &Node) -> bool {
#         true
#     }
#     fn edges(&self, n: &Node) -> Vec<Edge> {
#         Vec::new()
#     }
# }
let graph = MyGraph;
let obj = Box::new(graph) as Box<Graph>;
```

你会得到两个错误：

```text
error: the value of the associated type `E` (from the trait `main::Graph`) must
be specified [E0191]
let obj = Box::new(graph) as Box<Graph>;
          ^~~~~~~~~~~~~~~~~~~~~~~~~~~~~
24:44 error: the value of the associated type `N` (from the trait
`main::Graph`) must be specified [E0191]
let obj = Box::new(graph) as Box<Graph>;
          ^~~~~~~~~~~~~~~~~~~~~~~~~~~~~
```

我们不能这样创建一个trait对象，因为我们并不知道关联的类型。相反，我们可以这样写：

```rust
# trait Graph {
#     type N;
#     type E;
#     fn has_edge(&self, &Self::N, &Self::N) -> bool;
#     fn edges(&self, &Self::N) -> Vec<Self::E>;
# }
# struct Node;
# struct Edge;
# struct MyGraph;
# impl Graph for MyGraph {
#     type N = Node;
#     type E = Edge;
#     fn has_edge(&self, n1: &Node, n2: &Node) -> bool {
#         true
#     }
#     fn edges(&self, n: &Node) -> Vec<Edge> {
#         Vec::new()
#     }
# }
let graph = MyGraph;
let obj = Box::new(graph) as Box<Graph<N=Node, E=Edge>>;
```

`N=Node`语法允许我们提供一个具体类型，`Node`，作为`N`类型参数。`E=Edge`也是一样。如果我们不提供这个限制，我们不能确定应该`impl`那个来匹配trait对象。



关联类型是Rust类型系统中非常强大的一部分。它涉及到‘类型族’的概念，换句话说，就是把多种类型归于一类。这个描述可能比较抽象，所以让我们深入研究一个例子。如果你想编写一个`Graph`trait，你需要泛型化两个类型：点类型和边类型。所以你可能会像这样写一个trait，`Graph<N, E>`：

```rust
trait Graph<N, E> {
    fn has_edge(&self, &N, &N) -> bool;
    fn edges(&self, &N) -> Vec<E>;
    // etc
}
```

虽然这可以工作，不过显得很尴尬，例如，任何需要一个`Graph`作为参数的函数都需要泛型化的`N`ode和`E`dge类型：

```rust
fn distance<N, E, G: Graph<N, E>>(graph: &G, start: &N, end: &N) -> u32 { ... }
```

我们的距离计算并不需要`Edge`类型，所以函数签名中`E`只是写着玩的。

我们需要的是对于每一种`Graph`类型，都使用一个特定的的`N`ode和`E`dge类型。我们可以用关联类型来做到这一点：

```rust
trait Graph {
    type N;
    type E;

    fn has_edge(&self, &Self::N, &Self::N) -> bool;
    fn edges(&self, &Self::N) -> Vec<Self::E>;
    // etc
}
```

现在，我们使用一个抽象的`Graph`了：

```rust
fn distance<G: Graph>(graph: &G, start: &G::N, end: &G::N) -> uint { ... }
```

这里不再需要处理`E`dge类型了。

让我们更详细的回顾一下。

## 定义关联类型
让我们构建一个`Graph`trait。这里是定义：

```rust
trait Graph {
    type N;
    type E;

    fn has_edge(&self, &Self::N, &Self::N) -> bool;
    fn edges(&self, &Self::N) -> Vec<Self::E>;
}
```

十分简单。关联类型使用`type`关键字，并出现在trait体和函数中。

这些`type`声明跟函数定义一样。例如，如果我们想`N`类型实现`Display`，这样我们就可以打印出点类型，我们可以这样写：

```rust
use std::fmt;

trait Graph {
    type N: fmt::Display;
    type E;

    fn has_edge(&self, &Self::N, &Self::N) -> bool;
    fn edges(&self, &Self::N) -> Vec<Self::E>;
}
```

## 实现关联类型

就像任何 trait，使用关联类型的 trait 用`impl`关键字来提供实现。下面是一个`Graph`的简单实现：

```rust
# trait Graph {
#     type N;
#     type E;
#     fn has_edge(&self, &Self::N, &Self::N) -> bool;
#     fn edges(&self, &Self::N) -> Vec<Self::E>;
# }
struct Node;

struct Edge;

struct MyGraph;

impl Graph for MyGraph {
    type N = Node;
    type E = Edge;

    fn has_edge(&self, n1: &Node, n2: &Node) -> bool {
        true
    }

    fn edges(&self, n: &Node) -> Vec<Edge> {
        Vec::new()
    }
}
```

这个可笑的实现总是返回`true`和一个空的`Vec<Edge>`，不过它提供了如何实现这类 trait 的思路。首先我们需要3个`struct`，一个代表图，一个代表点，还有一个代表边。如果使用别的类型更合理，也可以那样做，我们只是准备使用`struct`来代表这 3 个类型。

接下来是`impl`行，它就像其它任何 trait 的实现。

在这里，我们使用`=`来定义我们的关联类型。trait 使用的名字出现在`=`的左边，而我们`impl`的具体类型出现在右边。最后，我们在函数声明中使用具体类型。

## trait 对象和关联类型

这里还有另外一个我们需要讨论的语法：trait对象。如果你创建一个关联类型的trait对象，像这样：

```rust
# trait Graph {
#     type N;
#     type E;
#     fn has_edge(&self, &Self::N, &Self::N) -> bool;
#     fn edges(&self, &Self::N) -> Vec<Self::E>;
# }
# struct Node;
# struct Edge;
# struct MyGraph;
# impl Graph for MyGraph {
#     type N = Node;
#     type E = Edge;
#     fn has_edge(&self, n1: &Node, n2: &Node) -> bool {
#         true
#     }
#     fn edges(&self, n: &Node) -> Vec<Edge> {
#         Vec::new()
#     }
# }
let graph = MyGraph;
let obj = Box::new(graph) as Box<Graph>;
```

你会得到两个错误：

```text
error: the value of the associated type `E` (from the trait `main::Graph`) must
be specified [E0191]
let obj = Box::new(graph) as Box<Graph>;
          ^~~~~~~~~~~~~~~~~~~~~~~~~~~~~
24:44 error: the value of the associated type `N` (from the trait
`main::Graph`) must be specified [E0191]
let obj = Box::new(graph) as Box<Graph>;
          ^~~~~~~~~~~~~~~~~~~~~~~~~~~~~
```

我们不能这样创建一个trait对象，因为我们并不知道关联的类型。相反，我们可以这样写：

```rust
# trait Graph {
#     type N;
#     type E;
#     fn has_edge(&self, &Self::N, &Self::N) -> bool;
#     fn edges(&self, &Self::N) -> Vec<Self::E>;
# }
# struct Node;
# struct Edge;
# struct MyGraph;
# impl Graph for MyGraph {
#     type N = Node;
#     type E = Edge;
#     fn has_edge(&self, n1: &Node, n2: &Node) -> bool {
#         true
#     }
#     fn edges(&self, n: &Node) -> Vec<Edge> {
#         Vec::new()
#     }
# }
let graph = MyGraph;
let obj = Box::new(graph) as Box<Graph<N=Node, E=Edge>>;
```

`N=Node`语法允许我们提供一个具体类型，`Node`，作为`N`类型参数。`E=Edge`也是一样。如果我们不提供这个限制，我们不能确定应该`impl`那个来匹配trait对象。

# 类型别名

## \`type\`别名


`type`关键字让你定义另一个类型的别名：

```rust
type Name = String;
```

你可以像一个真正类型那样使用这个类型：

```rust
type Name = String;

let x: Name = "Hello".to_string();
```

然而要注意的是，这一个*别名*，完全不是一个新的类型。换句话说，因为Rust是强类型的，你可以预期两个不同类型的比较会失败：

```rust
let x: i32 = 5;
let y: i64 = 5;

if x == y {
   // ...
}
```

这给出

```text
error: mismatched types:
 expected `i32`,
    found `i64`
(expected i32,
    found i64) [E0308]
     if x == y {
             ^
```

不过，如果我们有一个别名：

```rust
type Num = i32;

let x: i32 = 5;
let y: Num = 5;

if x == y {
   // ...
}
```

这会无错误的编译。从任何角度来说，`Num`类型的值与`i32`类型的值都是一样的。

你也可以在泛型中使用类型别名：

```rust
use std::result;

enum ConcreteError {
    Foo,
    Bar,
}

type Result<T> = result::Result<T, ConcreteError>;
```

这创建了一个特定版本的`Result`类型，它总是有一个`ConcreteError`作为`Result<T, E>`的`E`那部分。这通常用于标准库中创建每个子部分的自定义错误。例如，[`io::Result`](http://doc.rust-lang.org/nightly/std/io/type.Result.html)。
