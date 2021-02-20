# 互通性

<a id="c-common-traits"></a>

## 类型最好实现通用特征(traits) (C-COMMON-TRAITS)

Rust's trait system does not allow _orphans_: roughly, every `impl` must live
either in the crate that defines the trait or the implementing type.
Consequently, crates that define new types should eagerly implement all
applicable, common traits.

To see why, consider the following situation:

- Crate `std` defines trait `Display`.
- Crate `url` defines type `Url`, without implementing `Display`.
- Crate `webapp` imports from both `std` and `url`,

There is no way for `webapp` to add `Display` to `Url`, since it defines
neither. (Note: the newtype pattern can provide an efficient, but inconvenient
workaround.)

The most important common traits to implement from `std` are:

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

Note that it is common and expected for types to implement both
`Default` and an empty `new` constructor. `new` is the constructor
convention in Rust, and users expect it to exist, so if it is
reasonable for the basic constructor to take no arguments, then it
should, even if it is functionally identical to `default`.

<a id="c-conv-traits"></a>

## 转换使用标准库特征(traits) `From`, `AsRef`, `AsMut` (C-CONV-TRAITS)

The following conversion traits should be implemented where it makes sense:

- [`From`](https://doc.rust-lang.org/std/convert/trait.From.html)
- [`TryFrom`](https://doc.rust-lang.org/std/convert/trait.TryFrom.html)
- [`AsRef`](https://doc.rust-lang.org/std/convert/trait.AsRef.html)
- [`AsMut`](https://doc.rust-lang.org/std/convert/trait.AsMut.html)

The following conversion traits should never be implemented:

- [`Into`](https://doc.rust-lang.org/std/convert/trait.Into.html)
- [`TryInto`](https://doc.rust-lang.org/std/convert/trait.TryInto.html)

These traits have a blanket impl based on `From` and `TryFrom`. Implement those
instead.

### 标准库中的示例

- `From<u16>` is implemented for `u32` because a smaller integer can always be
  converted to a bigger integer.
- `From<u32>` is _not_ implemented for `u16` because the conversion may not be
  possible if the integer is too big.
- `TryFrom<u32>` is implemented for `u16` and returns an error if the integer is
  too big to fit in `u16`.
- [`From<Ipv6Addr>`] is implemented for [`IpAddr`], which is a type that can
  represent both v4 and v6 IP addresses.

[`from<ipv6addr>`]: https://doc.rust-lang.org/std/net/struct.Ipv6Addr.html
[`ipaddr`]: https://doc.rust-lang.org/std/net/enum.IpAddr.html

<a id="c-collect"></a>

## 集合实现 `FromIterator` 与 `Extend` (C-COLLECT)

[`FromIterator`] and [`Extend`] enable collections to be used conveniently with
the following iterator methods:

[`fromiterator`]: https://doc.rust-lang.org/std/iter/trait.FromIterator.html
[`extend`]: https://doc.rust-lang.org/std/iter/trait.Extend.html

- [`Iterator::collect`](https://doc.rust-lang.org/std/iter/trait.Iterator.html#method.collect)
- [`Iterator::partition`](https://doc.rust-lang.org/std/iter/trait.Iterator.html#method.partition)
- [`Iterator::unzip`](https://doc.rust-lang.org/std/iter/trait.Iterator.html#method.unzip)

`FromIterator` is for creating a new collection containing items from an
iterator, and `Extend` is for adding items from an iterator onto an existing
collection.

### 标准库中的示例

- [`Vec<T>`] implements both `FromIterator<T>` and `Extend<T>`.

[`vec<t>`]: https://doc.rust-lang.org/std/vec/struct.Vec.html

<a id="c-serde"></a>

## 数据结构(Data structures)实现 Serde 的 `Serialize`, `Deserialize` (C-SERDE)

Types that play the role of a data structure should implement [`Serialize`] and
[`Deserialize`].

[`serialize`]: https://docs.serde.rs/serde/trait.Serialize.html
[`deserialize`]: https://docs.serde.rs/serde/trait.Deserialize.html

There is a continuum of types between things that are clearly a data structure
and things that are clearly not, with gray area in between. [`LinkedHashMap`]
and [`IpAddr`] are data structures. It would be completely reasonable for
somebody to want to read in a `LinkedHashMap` or `IpAddr` from a JSON file, or
send one over IPC to another process. [`LittleEndian`] is not a data structure.
It is a marker used by the `byteorder` crate to optimize at compile time for
bytes in a particular order, and in fact an instance of `LittleEndian` can never
exist at runtime. So these are clear-cut examples; the #rust or #serde IRC
channels can help assess more ambiguous cases if necessary.

[`linkedhashmap`]: https://docs.rs/linked-hash-map/0.4.2/linked_hash_map/struct.LinkedHashMap.html
[`ipaddr`]: https://doc.rust-lang.org/std/net/enum.IpAddr.html
[`littleendian`]: https://docs.rs/byteorder/1.0.0/byteorder/enum.LittleEndian.html

If a crate does not already depend on Serde for other reasons, it may wish to
gate Serde impls behind a Cargo cfg. This way downstream libraries only need to
pay the cost of compiling Serde if they need those impls to exist.

For consistency with other Serde-based libraries, the name of the Cargo cfg
should be simply `"serde"`. Do not use a different name for the cfg like
`"serde_impls"` or `"serde_serialization"`.

The canonical implementation looks like this when not using derive:

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

And when using derive:

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

[`Send`] and [`Sync`] are automatically implemented when the compiler determines
it is appropriate.

[`send`]: https://doc.rust-lang.org/std/marker/trait.Send.html
[`sync`]: https://doc.rust-lang.org/std/marker/trait.Sync.html

In types that manipulate raw pointers, be vigilant that the `Send` and `Sync`
status of your type accurately reflects its thread safety characteristics. Tests
like the following can help catch unintentional regressions in whether the type
implements `Send` or `Sync`.

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

An error type is any type `E` used in a `Result<T, E>` returned by any public
function of your crate. Error types should always implement the
[`std::error::Error`] trait which is the mechanism by which error handling
libraries like [`error-chain`] abstract over different types of errors, and
which allows the error to be used as the [`source()`] of another error.

[`std::error::error`]: https://doc.rust-lang.org/std/error/trait.Error.html
[`error-chain`]: https://docs.rs/error-chain
[`source()`]: https://doc.rust-lang.org/std/error/trait.Error.html#method.source

Additionally, error types should implement the [`Send`] and [`Sync`] traits. An
error that is not `Send` cannot be returned by a thread run with
[`thread::spawn`]. An error that is not `Sync` cannot be passed across threads
using an [`Arc`]. These are common requirements for basic error handling in a
multithreaded application.

[`send`]: https://doc.rust-lang.org/std/marker/trait.Send.html
[`sync`]: https://doc.rust-lang.org/std/marker/trait.Sync.html
[`thread::spawn`]: https://doc.rust-lang.org/std/thread/fn.spawn.html
[`arc`]: https://doc.rust-lang.org/std/sync/struct.Arc.html

`Send` and `Sync` are also important for being able to package a custom error
into an IO error using [`std::io::Error::new`], which requires a trait bound of
`Error + Send + Sync`.

[`std::io::error::new`]: https://doc.rust-lang.org/std/io/struct.Error.html#method.new

One place to be vigilant about this guideline is in functions that return Error
trait objects, for example [`reqwest::Error::get_ref`]. Typically `Error + Send + Sync + 'static` will be the most useful for callers. The addition of
`'static` allows the trait object to be used with [`Error::downcast_ref`].

[`reqwest::error::get_ref`]: https://docs.rs/reqwest/0.7.2/reqwest/struct.Error.html#method.get_ref
[`error::downcast_ref`]: https://doc.rust-lang.org/std/error/trait.Error.html#method.downcast_ref-2

Never use `()` as an error type, even where there is no useful additional
information for the error to carry.

- `()` does not implement `Error` so it cannot be used with error handling
  libraries like `error-chain`.
- `()` does not implement `Display` so a user would need to write an error
  message of their own if they want to fail because of the error.
- `()` has an unhelpful `Debug` representation for users that decide to
  `unwrap()` the error.
- It would not be semantically meaningful for a downstream library to implement
  `From<()>` for their error type, so `()` as an error type cannot be used with
  the `?` operator.

Instead, define a meaningful error type specific to your crate or to the
individual function. Provide appropriate `Error` and `Display` impls. If there
is no useful information for the error to carry, it can be implemented as a unit
struct.

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

The error message given by the `Display` representation of an error type should
be lowercase without trailing punctuation, and typically concise.

[`Error::description()`] should not be implemented. It has been deprecated and users should
always use `Display` instead of `description()` to print the error.

[`error::description()`]: https://doc.rust-lang.org/std/error/trait.Error.html#tymethod.description

### 标准库中的示例

- [`ParseBoolError`] is returned when failing to parse a bool from a string.

[`parseboolerror`]: https://doc.rust-lang.org/std/str/struct.ParseBoolError.html

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

These traits control the representation of a type under the `{:X}`, `{:x}`,
`{:o}`, and `{:b}` format specifiers.

Implement these traits for any number type on which you would consider doing
bitwise manipulations like `|` or `&`. This is especially appropriate for
bitflag types. Numeric quantity types like `struct Nanoseconds(u64)` probably do
not need these.

<a id="c-rw-value"></a>

## 泛型读/写方法从值中获取`R: Read` 与`W: Write` (C-RW-VALUE)

The standard library contains these two impls:

```rust
impl<'a, R: Read + ?Sized> Read for &'a mut R { /* ... */ }

impl<'a, W: Write + ?Sized> Write for &'a mut W { /* ... */ }
```

That means any function that accepts `R: Read` or `W: Write` generic parameters
by value can be called with a mut reference if necessary.

In the documentation of such functions, briefly remind users that a mut
reference can be passed. New Rust users often struggle with this. They may have
opened a file and want to read multiple pieces of data out of it, but the
function to read one piece consumes the reader by value, so they are stuck. The
solution would be to leverage one of the above impls and pass `&mut f` instead
of `f` as the reader parameter.

### 例子

- [`flate2::read::GzDecoder::new`]
- [`flate2::write::GzEncoder::new`]
- [`serde_json::from_reader`]
- [`serde_json::to_writer`]

[`flate2::read::gzdecoder::new`]: https://docs.rs/flate2/0.2/flate2/read/struct.GzDecoder.html#method.new
[`flate2::write::gzencoder::new`]: https://docs.rs/flate2/0.2/flate2/write/struct.GzEncoder.html#method.new
[`serde_json::from_reader`]: https://docs.serde.rs/serde_json/fn.from_reader.html
[`serde_json::to_writer`]: https://docs.serde.rs/serde_json/fn.to_writer.html
