# 互通性

<a id="c-common-traits"></a>

## 类型最好实现通用特征(traits) (C-COMMON-TRAITS)

Rust 的 trait(特征)系统不允许*孤儿(orphans)*：大体上，每一个`impl`必须放置在 trait 定义的 crate 中或实现的类型中
因此，定义了新类型的 crate 应该立即实现所有适用、常见的特征。

要了解为什么，考虑以下情况：

- Crate `std` 定义 `Display` trait.
- Crate `url` 定义了类型 `Url`, 没有实现 `Display`.
- Crate `webapp` 引用 `std` 与 `url`,

这里没有途径为`webapp`中`Url` 添加 `Display`实现, 因为`webapp`中没有定义这些。(注意：newtype 模式可以提供有效的方案，但这是不方便的替代方法)

`std`中用以实现的很重要的公共 trait:

- [`Copy`](https://doc.rust-lang.org/std/marker/trait.Copy.html)
- [`Clone`](https://doc.rust-lang.org/std/clone/trait.Clone.html)
- [`Eq`](https://doc.rust-lang.org/std/cmp/trait.Eq.html)
- [`PartialEq`](https://doc.rust-lang.org/std/cmp/trait.PartialEq.html)
- [`Ord`](https://doc.rust-lang.org/std/cmp/trait.Ord.html)
- [`PartialOrd`](https://doc.rust-lang.org/std/cmp/trait.PartialOrd.html)
- [`Hash`](https://doc.rust-lang.org/std/hash/trait.Hash.html)
- [`Debug`](https://doc.rust-lang.org/std/fmt/trait.Debug.html)
- [`Display`](https://doc.rust-lang.org/std/fmt/trait.Display.html)
- [`Default`](https://doc.rust-lang.org/std/default/trait.Default.html)

注意对于类型实现`Default`与一个空的`new`构造方法是很常见的。`new`在 Rust 中是构造方法的惯例，并且用户希望存在，如果是这样的话，基本构造函数不接受参数是合理的，即使它在功能上与`Default`相同。

<a id="c-conv-traits"></a>

## 转换使用标准库特征(traits) `From`, `AsRef`, `AsMut` (C-CONV-TRAITS)

以下转换 trait 在合理的地方需要实现是有意义的:

- [`From`](https://doc.rust-lang.org/std/convert/trait.From.html)
- [`TryFrom`](https://doc.rust-lang.org/std/convert/trait.TryFrom.html)
- [`AsRef`](https://doc.rust-lang.org/std/convert/trait.AsRef.html)
- [`AsMut`](https://doc.rust-lang.org/std/convert/trait.AsMut.html)

绝不应该实现以下转换 trait：

- [`Into`](https://doc.rust-lang.org/std/convert/trait.Into.html)
- [`TryInto`](https://doc.rust-lang.org/std/convert/trait.TryInto.html)

这些 trait 有一个基于`From`和`TryFrom`有隐藏实现。实现将替代这些。

### 标准库中的示例

- `From<u16>` 被`u32`实现了，因为小整数总是可以转换成大整数。
- `From<u32>` 没有针对`u16`实现，因为如果整数过大转换不成功。
- `TryFrom<u32>` 有针对`u16`的实现，如果整数超过`u16`将返回一个错误。
- [`From<Ipv6Addr>`] 有针对可以代表 v4 与 v6 IP 地址的类型[`IpAddr`]的实现。

[`from<ipv6addr>`]: https://doc.rust-lang.org/std/net/struct.Ipv6Addr.html
[`IpAddr`]: https://doc.rust-lang.org/std/net/enum.IpAddr.html

<a id="c-collect"></a>

## 集合实现 `FromIterator` 与 `Extend` (C-COLLECT)

[`FromIterator`] 与 [`Extend`] 使集合能够方便地使用以下迭代器方法：

[`FromIterator`]: https://doc.rust-lang.org/std/iter/trait.FromIterator.html
[`extend`]: https://doc.rust-lang.org/std/iter/trait.Extend.html

- [`Iterator::collect`](https://doc.rust-lang.org/std/iter/trait.Iterator.html#method.collect)
- [`Iterator::partition`](https://doc.rust-lang.org/std/iter/trait.Iterator.html#method.partition)
- [`Iterator::unzip`](https://doc.rust-lang.org/std/iter/trait.Iterator.html#method.unzip)

`FromIterator` 用于从一个迭代器中创建一个包含项的心集合，`Extend` 用于从一个迭代器中的项来添加到一个存在的集合中

### 标准库中的示例

- [`Vec<T>`] 实现了 `FromIterator<T>` 与 `Extend<T>`。

[`Vec<T>`]: https://doc.rust-lang.org/std/vec/struct.Vec.html

<a id="c-serde"></a>

## 数据结构(Data structures)实现 Serde 的 `Serialize`, `Deserialize` (C-SERDE)

作为数据结构的类型应该实现 [`Serialize`] 与 [`Deserialize`]。

[`Serialize`]: https://docs.serde.rs/serde/trait.Serialize.html
[`Deserialize`]: https://docs.serde.rs/serde/trait.Deserialize.html

在明了与不明了的数据结构的类型之间的灰色区域存在连续类型。[`LinkedHashMap`]与[`IpAddr`]是数据结构。
从一个 JSON 文件中读取[`LinkedHashMap`]与[`IpAddr`]，或通过 IPC 发送给另外一个进程是完全合理的。
[`LittleEndian`] 不是一个数据结构。它是`byteorder` crate 使用的标记，以在编译时优化特定顺序的字节，
实际上`LittleEndian`的实例在运行时永远都不存在。这些都是明确的例子；
如果需要，#rust 或#serde IRC 频道可以帮助了解到更多的不明朗的案例。

[`LinkedHashMap`]: https://docs.rs/linked-hash-map/0.4.2/linked_hash_map/struct.LinkedHashMap.html
[`IpAddr`]: https://doc.rust-lang.org/std/net/enum.IpAddr.html
[`LittleEndian`]: https://docs.rs/byteorder/1.0.0/byteorder/enum.LittleEndian.html

如果一个 crate 因为某些原因没有依赖 Serde,可能希望通过一个 crago cfg gate 来实现 Serde 。
下游库如果需要这些实现存在，只需要关注实现 Serde 编译。

为了与其他基于 Serde 的库保持一致，请使用 Cargo cfg 的名称应该是简单的`serde`。请勿为 cfg 使用其他名称，例如`serde_impls`或`serde_serialization`。

不使用 derive 时的规范实现如下：

```toml
[dependencies]
serde = { version = "1.0", optional = true }
```

```rust
#[cfg(feature = "serde")]
extern crate serde;

struct T { /* ... */ }

#[cfg(feature = "serde")]
impl Serialize for T { /* ... */ }

#[cfg(feature = "serde")]
impl<'de> Deserialize<'de> for T { /* ... */ }
```

使用 derive 时:

```toml
[dependencies]
serde = { version = "1.0", optional = true, features = ["derive"] }
```

```rust
#[cfg(feature = "serde")]
#[macro_use]
extern crate serde;

#[cfg_attr(feature = "serde", derive(Serialize, Deserialize))]
struct T { /* ... */ }
```

<a id="c-send-sync"></a>

## 类型尽可能是 `Send` 与 `Sync` (C-SEND-SYNC)

编译器在合适的时机自动实现 [`Send`] 与 [`Sync`]。

[`Send`]: https://doc.rust-lang.org/std/marker/trait.Send.html
[`Sync`]: https://doc.rust-lang.org/std/marker/trait.Sync.html

在操作原始指针的类型中，应警惕你的数据类型的`Send`与`Sync`状态能够反应是线程安全的。
如下测试可以帮助捕获类型是否需要实现`Send`与`Sync`。

```rust
#[test]
fn test_send() {
    fn assert_send<T: Send>() {}
    assert_send::<MyStrangeType>();
}

#[test]
fn test_sync() {
    fn assert_sync<T: Sync>() {}
    assert_sync::<MyStrangeType>();
}
```

<a id="c-good-err"></a>

## 错误类型是有意义且行为良好的 (C-GOOD-ERR)

一个错误类型是任何在你的 crate 中任何公共方法返回的`Result<T, E>`中的`E`类型。
错误类型应该实现[`std::error::Error`] trait，这是错误处理库如果[`error-chain`]通过不同错误类型抽象机制，
可以允许错误作为另外一个错误的[`source()`]。

[`std::error::error`]: https://doc.rust-lang.org/std/error/trait.Error.html
[`error-chain`]: https://docs.rs/error-chain
[`source()`]: https://doc.rust-lang.org/std/error/trait.Error.html#method.source

此外，错误类型应该实现[`Send`]与[`Sync`]trait。一个没有实现`Send`的错误类型不能在通过[`thread::spawn`]允许的线程中返回。
一个没有实现`Sync`的错误类型不能在线程间使用[`Arc`]传递。这在一个多线程应用中是通用  基础的错误处理。

[`Send`]: https://doc.rust-lang.org/std/marker/trait.Send.html
[`Sync`]: https://doc.rust-lang.org/std/marker/trait.Sync.html
[`Thread::spawn`]: https://doc.rust-lang.org/std/thread/fn.spawn.html
[`Arc`]: https://doc.rust-lang.org/std/sync/struct.Arc.html

`Send` 与 `Sync`对使用[`std::io::Error::new`]将自定义错误包装成 IO 错误也很重要，这需要一个有`Error + Send + Sync`约束的 trait。

[`std::io::error::new`]: https://doc.rust-lang.org/std/io/struct.Error.html#method.new

关于此指南的一个要警惕的地方是返回错误 trait 对象的方法，例如[`reqwest::Error::get_ref`]。
典型的对于调用者`Error + Send + Sync + 'static`将非常有用。附加`'static`允许 trait 对象可以与[`Error::downcast_ref`]配合使用。

[`reqwest::Error::get_ref`]: https://docs.rs/reqwest/0.7.2/reqwest/struct.Error.html#method.get_ref
[`error::downcast_ref`]: https://doc.rust-lang.org/std/error/trait.Error.html#method.downcast_ref-2

不要使用`()`作为一个错误类型，即使在没有任何有用附加信息用于错误上时。

- `()` 没有实现 `Error` 因此不能配合错误处理库如`error-chain`一起使用。
- `()` 没有实现 `Display` 因此如果因为一个错误需要失败时用户需要自定义错误消息。
- `()` 对于决定使用`unwrap()`错误的用户 `Debug`没有帮助信息显示。
- 对下游库为他们的错误类型实现`From<()>`不会有语义意义，所以`()`作为错误类型不能使用`?`操作。

相反，定义特定于箱子或箱子的有意义的错误类型
个别功能。提供适当的“错误”和“显示”“魔”。如果有
对携带错误没有有用的信息，它可以作为一个单位实现
结构。

相反，为 crate 或一个独立的功能定义一个有意义的详细错误类型。提供`Error` 与 `Display`合适的实现。
如果没有有用信息提供给错误，可以基于一个元 struct(unit struct)实现。

```rust
use std::error::Error;
use std::fmt::Display;

// Instead of this...
fn do_the_thing() -> Result<Wow, ()>

// Prefer this...
fn do_the_thing() -> Result<Wow, DoError>

#[derive(Debug)]
struct DoError;

impl Display for DoError { /* ... */ }
impl Error for DoError { /* ... */ }
```

错误类型的“ Display”表示形式给出的错误消息应小写而不会在后面加上标点符号，并且通常比较简洁。

通过一个错误类型的`Display`展示的错误消息内容通常都是小写并且不要在后面加上标点符号，这样比较简洁。

[`Error::description()`] 不应该被实现。已经被废弃了，用户应该使用`Display`替代`description()`来打印错误。

[`Error::description()`]: https://doc.rust-lang.org/std/error/trait.Error.html#tymethod.description

### 标准库中的示例

- [`ParseBoolError`] 在无法从字符串中解析 bool 时会返回。

[`ParseBoolError`]: https://doc.rust-lang.org/std/str/struct.ParseBoolError.html

### 错误消息示例

- "unexpected end of file"
- "provided string was not \`true\` or \`false\`"
- "invalid IP address syntax"
- "second time provided was later than self"
- "invalid UTF-8 sequence of {} bytes from index {}"
- "environment variable was not valid unicode: {:?}"

<a id="c-num-fmt"></a>

## 二进制数类型提供了`Hex`, `Octal`, `Binary` 格式 (C-NUM-FMT)

- [`std::fmt::UpperHex`](https://doc.rust-lang.org/std/fmt/trait.UpperHex.html)
- [`std::fmt::LowerHex`](https://doc.rust-lang.org/std/fmt/trait.LowerHex.html)
- [`std::fmt::Octal`](https://doc.rust-lang.org/std/fmt/trait.Octal.html)
- [`std::fmt::Binary`](https://doc.rust-lang.org/std/fmt/trait.Binary.html)

这些 trait 通过`{:X}`、`{:x}`、`{:o}`与`{:b}`格式化类型标识符控制类型表示形式。尤其适合 bitflag 类型。

在任何 number 类型上实现这些 trait 应该考虑如`|`或`&`按位操作。数值数量类型如`struct Nanoseconds(u64)`可能不需要这么做。

<a id="c-rw-value"></a>

## 泛型读/写方法从值中获取`R: Read` 与`W: Write` (C-RW-VALUE)

标准库中包含了这两个实现：

```rust
impl<'a, R: Read + ?Sized> Read for &'a mut R { /* ... */ }

impl<'a, W: Write + ?Sized> Write for &'a mut W { /* ... */ }
```

这意味着任何接受`R: Read`或 `W: Write`泛型参数的函数可以在需要的时候使用 mut 引用调用值。

在此方法的文档中，简要提醒用户注意一个 mut 引用可以传递。Rust 新手经常为此感到困扰。他们可能有打开了一个文件，想从中读取多个片段的数据，但是方法从 reader 中消耗读取一个片段值，就卡住了。解决办法是利用上述实现传递`&mut f`替代`f`作为 reader 参数。

### 例子

- [`flate2::read::GzDecoder::new`]
- [`flate2::write::GzEncoder::new`]
- [`serde_json::from_reader`]
- [`serde_json::to_writer`]

[`flate2::read::GzDecoder::new`]: https://docs.rs/flate2/0.2/flate2/read/struct.GzDecoder.html#method.new
[`flate2::write::GzEncoder::new`]: https://docs.rs/flate2/0.2/flate2/write/struct.GzEncoder.html#method.new
[`serde_json::from_reader`]: https://docs.serde.rs/serde_json/fn.from_reader.html
[`serde_json::to_writer`]: https://docs.serde.rs/serde_json/fn.to_writer.html
