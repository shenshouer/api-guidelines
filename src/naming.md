# 命名

<a id="c-case"></a>

## 大小写遵循 RFC 430 (C-CASE)

Rust 的基本命名约定在 [RFC 430]中有描述。

一般来说, Rust 倾向于使用 `UpperCamelCase`(大写驼峰形式) 来构建 "类型级别(type-level)" (类型与 traits) 和 `snake_case`(蛇形) 来构建 "值级别(value-level)". 更准确的如下描述:

| 项                                    | 约定                                                                               |
| ------------------------------------- | ---------------------------------------------------------------------------------- |
| Crates(库)                            | [unclear](https://github.com/rust-lang/api-guidelines/issues/29)                   |
| Modules(模块)                         | `snake_case`                                                                       |
| Types(类型)                           | `UpperCamelCase`                                                                   |
| Traits(特征)                          | `UpperCamelCase`                                                                   |
| Enum variants(枚举变体)               | `UpperCamelCase`                                                                   |
| Functions(函数)                       | `snake_case`                                                                       |
| Methods(方法)                         | `snake_case`                                                                       |
| General constructors(通用构造函数)    | `new` 或 `with_more_details`                                                       |
| Conversion constructors(转换构造函数) | `from_some_other_type`                                                             |
| Macros(宏指令)                        | `snake_case!`                                                                      |
| Local variables(本地变量)             | `snake_case`                                                                       |
| Statics(静态类型)                     | `SCREAMING_SNAKE_CASE`                                                             |
| Constants(常量类型)                   | `SCREAMING_SNAKE_CASE`                                                             |
| Type parameters(类型参数)             | 简洁的 `UpperCamelCase`, 通常单个大写字母: `T`                                     |
| Lifetimes(生命周期)                   | short `lowercase`, 通常单个字母: `'a`, `'de`, `'src`                               |
| Features(特征)                        | [unclear](https://github.com/rust-lang/api-guidelines/issues/101) 参见 [C-FEATURE] |

在 `UpperCamelCase`(大写驼峰形式)下, 复合词的缩略语和缩略语算作一个词: 使用 `Uuid` 不用 `UUID`, 使用`Usize` 不用 `USize` 或 使用`Stdin` 不用 `StdIn`. 在 `snake_case`(蛇形)下, 首字母缩略词和缩写词为小写: `is_xid_start`.

在 `snake_case`(蛇形) 或 `SCREAMING_SNAKE_CASE`(大写蛇形), 一个单词("word")绝不可能由一个单独的字母(letter)组成除非是最后一个单词("word")。所以, 我们使用 `btree_map` 而不用
`b_tree_map`, 但是使用 `PI_2` 不用 `PI2`。

Crate 命名不使用`-rs` 或 `-rust`作为前缀或后缀。Rust 中每一个 crate 都是这样! 经常提醒用户这一点没有任何意义.

[RFC 430]: https://github.com/rust-lang/rfcs/blob/master/text/0430-finalizing-naming-conventions.md
[C-FEATURE]: #c-feature

### 标准库的示例

整个标准库。该指南应该很容易！

<a id="c-conv"></a>

## 特定目的约定 `as_`, `to_`, `into_` 约定 (C-CONV)

转换应该以方法的形式提供，并带有如下前缀的名称:

| 前缀    | 开销(Cost)        | 所有权(Ownership)                                                                                                                  |
| ------- | ----------------- | ---------------------------------------------------------------------------------------------------------------------------------- |
| `as_`   | 无开销(Free)      | 借用(borrowed) -\> 借用(borrowed)                                                                                                  |
| `to_`   | 大开销(Expensive) | 借用(borrowed) -\> 借用(borrowed)<br>借用(borrowed) -\> 拥有(owned) (不可 Copy 类型)<br>拥有(owned) -\> 拥有(owned) (可 Copy 类型) |
| `into_` | 多变(Variable)    | 拥有(owned) -\> 拥有(owned) (不可 Copy 类型)                                                                                       |

例如:

- [`str::as_bytes()`] 给出一个`str`的视图作为一个 UTF-8 字节 slice, 这是没有开销的. 输入是一个借用 `&str` 并且输出也是一个借用 `&[u8]`。
- [`Path::to_str`] 在一个操作系统路径的字节上执行一个大开销的 UTF-8 检查。 输入与输出都是借用。调用`as_str`是不合适的，因为这个方法在运行时有很大的开销。
- [`str::to_lowercase()`] 产生与 Unicode 小写等效的`str`，涉及迭代字符串的字符，并且可能需要内存分配。输入是借用`＆str`，输出是拥有所有权的`String`。
- [`f64::to_radians()`] 将浮点数从角度转换成弧度。 输入是 `f64`. 传递一个引用`&f64`是不被允许的，因为`f64`复制(Copy)开销小。调用函数 into_radians 将引起误导，因为没有消费输入。
- [`String::into_bytes()`] 提取一个`String`潜在的的`Vec<u8>`是没有开销的。这将获取一个`String`的所有权并且返回一个拥有所有权的`Vec<u8>`。
- [`BufReader::into_inner()`] 获取一个缓存读的所有权并提取潜在的读取者,这是无开销的。在缓存区的数据将被丢弃。
- [`BufWriter::into_inner()`] 获取一个缓存写的所有权并提取潜在的写，这需要一个潜在的 flush 开销用于缓存区的任何数据。

[`str::as_bytes()`]: https://doc.rust-lang.org/std/primitive.str.html#method.as_bytes
[`path::to_str`]: https://doc.rust-lang.org/std/path/struct.Path.html#method.to_str
[`str::to_lowercase()`]: https://doc.rust-lang.org/std/primitive.str.html#method.to_lowercase
[`f64::to_radians()`]: https://doc.rust-lang.org/std/primitive.f64.html#method.to_radians
[`String::into_bytes()`]: https://doc.rust-lang.org/std/string/struct.String.html#method.into_bytes
[`BufReader::into_inner()`]: https://doc.rust-lang.org/std/io/struct.BufReader.html#method.into_inner
[`BufWriter::into_inner()`]: https://doc.rust-lang.org/std/io/struct.BufWriter.html#method.into_inner

带有前缀`as_`和`into_`的转换通常会 _降低抽象度_，要么在底层表现形式（`as_`）暴露一个视图，要么将数据解构为底层表现形式（`into_`）。另一方面，以`to_`为前缀的转换通常停留在相同的抽象级别，但需要做一些工作才能从一种表示转换为另一种表示。

当类型包裹一个值(wraps a single value)以将其与更高级别的语义相关联时，访问包值(wrapped value)应该由`into_inner()`方法提供。这适用于提供缓冲的包装器，如[`BufReader`]，编码或解码如[`GzDecoder`]，原子访问如[`AtomicBool`]，或任何类似的语义。

[`BufReader`]: https://doc.rust-lang.org/std/io/struct.BufReader.html#method.into_inner
[`GzDecoder`]: https://docs.rs/flate2/0.2.19/flate2/read/struct.GzDecoder.html#method.into_inner
[`AtomicBool`]: https://doc.rust-lang.org/std/sync/atomic/struct.AtomicBool.html#method.into_inner

如果转换方法名称中的`mut`限定词构成了返回类型，它应该像在类型中一样出现。例如[`Vec::as_mut_slice`]返回一个可变的切片；它就像方法名称一样去处理。这个名字是优先于`as_slice_mut`。

[`Vec::as_mut_slice`]: https://doc.rust-lang.org/std/vec/struct.Vec.html#method.as_mut_slice

```rust
// 返回类型是一个可变切片。
fn as_mut_slice(&mut self) -> &mut [T];
```

##### 标准库中的更多示例

- [`Result::as_ref`](https://doc.rust-lang.org/std/result/enum.Result.html#method.as_ref)
- [`RefCell::as_ptr`](https://doc.rust-lang.org/std/cell/struct.RefCell.html#method.as_ptr)
- [`slice::to_vec`](https://doc.rust-lang.org/std/primitive.slice.html#method.to_vec)
- [`Option::into_iter`](https://doc.rust-lang.org/std/option/enum.Option.html#method.into_iter)

<a id="c-getter"></a>

## Getter 命名遵循 Rust 约定 (C-GETTER)

除少数例外，Rust 代码中的 getter 不使用`get_`前缀。

```rust
pub struct S {
    first: First,
    second: Second,
}

impl S {
    // 不使用 get_first.
    pub fn first(&self) -> &First {
        &self.first
    }

    // 不使用 get_first_mut, get_mut_first, 或 mut_first.
    pub fn first_mut(&mut self) -> &mut First {
        &mut self.first
    }
}
```

`get`命名仅在存在单个并且明显的通过 getter 获取的事物时。例如[`Cell::get`]访问`Cell`的内容。

[`Cell::get`]: https://doc.rust-lang.org/std/cell/struct.Cell.html#method.get

对于执行运行时验证（如边界检查）的 Getters，请考虑添加不安全的`_unchecked`变体。通常，这些将具有以下内容签名。

```rust
fn get(&self, index: K) -> Option<&V>;
fn get_mut(&mut self, index: K) -> Option<&mut V>;
unsafe fn get_unchecked(&self, index: K) -> &V;
unsafe fn get_unchecked_mut(&mut self, index: K) -> &mut V;
```

获取(getter)和转换(conversions)（[C-CONV](＃c-conv)）之间的区别可能很小而且并不总是很明确。例如[`TempDir::path`]可以理解为临时目录文件系统路径的获取器，而[`TempDir::into_path`]是一种转换，可将删除临时目录给调用者。由于`path`是 getter，因此将其称为`get_path`或`as_path`是不正确的。

[`TempDir::path`]: https://docs.rs/tempdir/0.3.5/tempdir/struct.TempDir.html#method.path
[`TempDir::into_path`]: https://docs.rs/tempdir/0.3.5/tempdir/struct.TempDir.html#method.into_path

### 标准库的示例

- [`std::io::Cursor::get_mut`](https://doc.rust-lang.org/std/io/struct.Cursor.html#method.get_mut)
- [`std::ptr::Unique::get_mut`](https://doc.rust-lang.org/std/ptr/struct.Unique.html#method.get_mut)
- [`std::sync::PoisonError::get_mut`](https://doc.rust-lang.org/std/sync/struct.PoisonError.html#method.get_mut)
- [`std::sync::atomic::AtomicBool::get_mut`](https://doc.rust-lang.org/std/sync/atomic/struct.AtomicBool.html#method.get_mut)
- [`std::collections::hash_map::OccupiedEntry::get_mut`](https://doc.rust-lang.org/std/collections/hash_map/struct.OccupiedEntry.html#method.get_mut)
- [`<[T]>::get_unchecked`](https://doc.rust-lang.org/std/primitive.slice.html#method.get_unchecked)

<a id="c-iter"></a>

## 集合上生成迭代器的方法遵循 `iter`, `iter_mut`, `into_iter` (C-ITER)

Per [RFC 199].

对于元素类型为`U`的容器，迭代器方法应命名为:

```rust
fn iter(&self) -> Iter             // Iter 实现了 Iterator<Item = &U>
fn iter_mut(&mut self) -> IterMut  // IterMut 实现了 Iterator<Item = &mut U>
fn into_iter(self) -> IntoIter     // IntoIter 实现了 Iterator<Item = U>
```

该指南适用于概念上是同质集合的数据结构。作为反例，`str`类型是保证为有效 UTF-8 的字节片。从概念上讲，它比同类集合更细微，因此与其提供 `iter` / `iter_mut` / `into_iter` 组迭代器方法，不如提供[`str::bytes`]作为字节进行迭代，并提供[`str::chars`]迭代为 chars。

[`str::bytes`]: https://doc.rust-lang.org/std/primitive.str.html#method.bytes
[`str::chars`]: https://doc.rust-lang.org/std/primitive.str.html#method.chars

本指南仅适用于方法，不适用于函数。例如来自 `url` crate 的[percent_encode`]返回一个经过百分比编码的迭代器字符串片段。通过使用`iter` /`iter_mut` /`into_iter`约定来获取就显得不够清晰了。

[`percent_encode`]: https://docs.rs/url/1.4.0/url/percent_encoding/fn.percent_encode.html
[rfc 199]: https://github.com/rust-lang/rfcs/blob/master/text/0199-ownership-variants.md

### 标准库中的示例

- [`Vec::iter`](https://doc.rust-lang.org/std/vec/struct.Vec.html#method.iter)
- [`Vec::iter_mut`](https://doc.rust-lang.org/std/vec/struct.Vec.html#method.iter_mut)
- [`Vec::into_iter`](https://doc.rust-lang.org/std/vec/struct.Vec.html#method.into_iter)
- [`BTreeMap::iter`](https://doc.rust-lang.org/std/collections/struct.BTreeMap.html#method.iter)
- [`BTreeMap::iter_mut`](https://doc.rust-lang.org/std/collections/struct.BTreeMap.html#method.iter_mut)

<a id="c-iter-ty"></a>

## 迭代器类型命名与产生它们的方法相对应 (C-ITER-TY)

一个名为`into_iter()`的方法应该返回一个名为`IntoIter`的类型,对于返回迭代器的所有其他方法亦是类似。

本指南主要适用于方法(method)，但通常对函数(function)有意义也是如此。例如，来自`URL`Create 的[`percent_encode`]方法返回一个名为 [`PercentEncode`][percentencode-type] 的迭代器类型。

[percentencode-type]: https://docs.rs/url/1.4.0/url/percent_encoding/struct.PercentEncode.html

这些类型命名在带有其所属模块的前缀时最有意义， 例如 [`vec::IntoIter`].

[`vec::IntoIter`]: https://doc.rust-lang.org/std/vec/struct.IntoIter.html

### 标准库中的示例

- [`Vec::iter`] returns [`Iter`][slice::iter]
- [`Vec::iter_mut`] returns [`IterMut`][slice::itermut]
- [`Vec::into_iter`] returns [`IntoIter`][vec::intoiter]
- [`BTreeMap::keys`] returns [`Keys`][btree_map::keys]
- [`BTreeMap::values`] returns [`Values`][btree_map::values]

[`Vec::iter`]: https://doc.rust-lang.org/std/vec/struct.Vec.html#method.iter
[`slice::iter`]: https://doc.rust-lang.org/std/slice/struct.Iter.html
[`Vec::iter_mut`]: https://doc.rust-lang.org/std/vec/struct.Vec.html#method.iter_mut
[`slice::itermut`]: https://doc.rust-lang.org/std/slice/struct.IterMut.html
[`Vec::into_iter`]: https://doc.rust-lang.org/std/vec/struct.Vec.html#method.into_iter
[vec::intoiter]: https://doc.rust-lang.org/std/vec/struct.IntoIter.html
[`BTreeMap::keys`]: https://doc.rust-lang.org/std/collections/struct.BTreeMap.html#method.keys
[btree_map::keys]: https://doc.rust-lang.org/std/collections/btree_map/struct.Keys.html
[`BTreeMap::values`]: https://doc.rust-lang.org/std/collections/struct.BTreeMap.html#method.values
[btree_map::values]: https://doc.rust-lang.org/std/collections/btree_map/struct.Values.html

<a id="c-feature"></a>

## 特征(Feature)命名不包含占位符单词 (C-FEATURE)

不要用名称中包含的单词来命名[Cargo feature]，这没有任何意义，如使用`use-abc` 或 `with-abc`。直接使用`abc`来命名 feature

[cargo feature]: http://doc.crates.io/manifest.html#the-features-section

这些规范最常见于通常依赖于 Rust 标准库的 crate 上。正确执行此操作的规范方法如下：

```toml
# 在 Cargo.toml 内

[features]
default = ["std"]
std = []
```

```rust
// 在 lib.rs 内

#![cfg_attr(not(feature = "std"), no_std)]
```

请勿使用`use-std`或`with-std`或任何非`std`的名称来命名 feature。此命名约定与隐式功能的命名保持一致由 Cargo 推断出可选依赖项。考虑 crate `x` 对 Serde 和 Rust 标准库的可选依赖：

```toml
[package]
name = "x"
version = "0.1.0"

[features]
std = ["serde/std"]

[dependencies]
serde = { version = "1.0", optional = true }
```

当我们依赖于`x`时，我们可以启用可选的 Serde 依赖关系`features = ["serde"]`。同样，我们可以启用可选的标准库依赖于`features = ["std"]`。crate 隐式推断
被称为`Serde`的可选的依赖性，而不用`use-serde` 或 `with-serde`，所以我们喜欢使用类似的行为来明确 features。

作为相关说明，crate 要求 features 是附加的，所以一个呈否定名称`no-abc`命名的 features 是不正确的。

<a id="c-word-order"></a>

## 命名使用一致的单词顺序 (C-WORD-ORDER)

这里有一些标准库中定义的错误类型:

- [`JoinPathsError`](https://doc.rust-lang.org/std/env/struct.JoinPathsError.html)
- [`ParseBoolError`](https://doc.rust-lang.org/std/str/struct.ParseBoolError.html)
- [`ParseCharError`](https://doc.rust-lang.org/std/char/struct.ParseCharError.html)
- [`ParseFloatError`](https://doc.rust-lang.org/std/num/struct.ParseFloatError.html)
- [`ParseIntError`](https://doc.rust-lang.org/std/num/struct.ParseIntError.html)
- [`RecvTimeoutError`](https://doc.rust-lang.org/std/sync/mpsc/enum.RecvTimeoutError.html)
- [`StripPrefixError`](https://doc.rust-lang.org/std/path/struct.StripPrefixError.html)

所有这些都使用动词-对象-错误(verb-object-error)词序。如果我们添加错误来表示解析地址错误，因为我们想要使用动词-对象-错误(verb-object-error)词序顺序来命名它，使用如`Parseaddror`而不是`addrparseerror`。

单词顺序的具体选择并不重要，但要注意 crate 内以及与标准库的一致性。
