# Rust API 准则清单

<!-- Read CONTRIBUTING.md before writing new guidelines -->

- **命名** _(crate 与 Rust 命名约定一致)_
  - [ ] 大小写遵循 RFC 430 ([C-CASE])
  - [ ] 特定目的约定 `as_`, `to_`, `into_` 约定 ([C-CONV])
  - [ ] Getter 命名遵循 Rust 约定 ([C-GETTER])
  - [ ] 集合上生成迭代器的方法遵循 `iter`, `iter_mut`, `into_iter` ([C-ITER])
  - [ ] 迭代器类型命名与产生它们的方法相对应 ([C-ITER-TY])
  - [ ] 特征(Feature)命名不包含占位符单词 ([C-FEATURE])
  - [ ] 命名使用一致的单词顺序 ([C-WORD-ORDER])
- **互通性** _(crate 与其他库功能很好地交互)_
  - [ ] 类型最好实现通用特征(traits) ([C-COMMON-TRAITS])
    - `Copy`, `Clone`, `Eq`, `PartialEq`, `Ord`, `PartialOrd`, `Hash`, `Debug`,
      `Display`, `Default`
  - [ ] 转换使用标准库特征(traits) `From`, `AsRef`, `AsMut` ([C-CONV-TRAITS])
  - [ ] 集合实现 `FromIterator` 与 `Extend` ([C-COLLECT])
  - [ ] 数据结构(Data structures)实现 Serde 的 `Serialize`, `Deserialize` ([C-SERDE])
  - [ ] 类型尽可能是 `Send` 与 `Sync` ([C-SEND-SYNC])
  - [ ] 错误类型是有意义且行为良好的 ([C-GOOD-ERR])
  - [ ] 二进制数类型提供了`Hex`, `Octal`, `Binary` 格式 ([C-NUM-FMT])
  - [ ] 泛型读/写方法从值中获取`R: Read` 与`W: Write` ([C-RW-VALUE])
- **宏指令** _( crate 提供行为良好的宏)_
  - [ ] 输入语法唤起输出 ([C-EVOCATIVE])
  - [ ] 宏可以很好地与属性组合在一起 ([C-MACRO-ATTR])
  - [ ] 项宏(Item macros)可以在任何允许项的地方工作 ([C-ANYWHERE])
  - [ ] 项目宏支持可见性说明符 ([C-MACRO-VIS])
  - [ ] 类型片段很灵活 ([C-MACRO-TY])
- **文档** _(crate 有大量的文档)_
  - [ ] Crate 级别文档是必要的，并且包含例子 ([C-CRATE-DOC])
  - [ ] 所有项目都有一个 rustdoc 示例 ([C-EXAMPLE])
  - [ ] 使用 `?`, 而不是 `try!`, 与 `unwrap`的例子 ([C-QUESTION-MARK])
  - [ ] 函数文档包括 error, panic 和考虑安全事项 ([C-FAILURE])
  - [ ] 文档中包含到相关事物的超链接 ([C-LINK])
  - [ ] Cargo.toml 包含所有公共的原数据 ([C-METADATA])
    - authors, description, license, homepage, documentation, repository,
      readme, keywords, categories
  - [ ] Crate 设置 html_root_url 属性 "https://docs.rs/CRATE/X.Y.Z" ([C-HTML-ROOT])
  - [ ] 发布说明记录重大有意义的变更 ([C-RELNOTES])
  - [ ] Rustdoc 不要展示无用的实现细节 ([C-HIDDEN])
- **可预测性** _(crate 启用行为清晰的代码)_
  - [ ] 智能指针不添加固有方法 ([C-SMART-PTR])
  - [ ] 转换依赖于所涉及的最确切的类型 ([C-CONV-SPECIFIC])
  - [ ] 带有明确的接收者的函数就是方法 ([C-METHOD])
  - [ ] 函数以输入参数作为输出参数 ([C-NO-OUT])
  - [ ] 操作符重载并不奇怪 ([C-OVERLOAD])
  - [ ] 仅有智能指针实现 `Deref` 与 `DerefMut` ([C-DEREF])
  - [ ] 构造函数是静态的固有的方法 ([C-CTOR])
- **灵活性** _(Crate 支持各种实际使用案例)_
  - [ ] 函数公开中间结果以避免重复工作 ([C-INTERMEDIATE])
  - [ ] 调用者决定哪儿复制与哪里放数据 ([C-CALLER-CONTROL])
  - [ ] 函数通过使用泛型来最小化对参数的假设 ([C-GENERIC])
  - [ ] 如果 trait 对象是有用的，那么它们就是对象安全的 ([C-OBJECT])
- **类型安全** _(crate 有效地利用了类型系统)_
  - [ ] Newtypes 提供静态区别 ([C-NEWTYPE])
  - [ ] 参数通过类型表达意义, 而非 `bool` 或 `Option` ([C-CUSTOM-TYPE])
  - [ ] 一组标记的类型是 `bitflags`, 而不是枚举 ([C-BITFLAG])
  - [ ] 构建器可以构建复杂的值 ([C-BUILDER])
- **可靠性** _(crate 不太可能做错误的事情)_
  - [ ] 函数验证它们的参数 ([C-VALIDATE])
  - [ ] 析构函数从不失败 ([C-DTOR-FAIL])
  - [ ] 可能阻塞的析构函数有其他替代方案 ([C-DTOR-BLOCK])
- **可调试性** _(crate 对简单调试有帮助)_
  - [ ] 所有公共类型都继承 `Debug` ([C-DEBUG])
  - [ ] `Debug` 表示永远不会是空的([C-DEBUG-NONEMPTY])
- **前瞻检验** _(Crate 可以自由改进而不会破坏用户的代码)_
  - [ ] 未知特性(Sealed traits)要防止下游实现 ([C-SEALED])
  - [ ] 结构有私有字段 ([C-STRUCT-PRIVATE])
  - [ ] Newtypes 封装实现细节 ([C-NEWTYPE-HIDE])
  - [ ] 数据结构(Data structures)不要重复衍生 trait 界定 ([C-STRUCT-BOUNDS])
- **须知** _(对于关注的人，这些真的很重要)_
  - [ ] 稳定的 Crate 的公共依赖性稳定 ([C-STABLE])
  - [ ] Crate 及其依赖有许可证 ([C-PERMISSIVE])

[C-CASE]: naming.html#c-case
[C-CONV]: naming.html#c-conv
[C-GETTER]: naming.html#c-getter
[C-ITER]: naming.html#c-iter
[C-ITER-TY]: naming.html#c-iter-ty
[C-FEATURE]: naming.html#c-feature
[C-WORD-ORDER]: naming.html#c-word-order
[C-COMMON-TRAITS]: interoperability.html#c-common-traits
[C-CONV-TRAITS]: interoperability.html#c-conv-traits
[C-COLLECT]: interoperability.html#c-collect
[C-SERDE]: interoperability.html#c-serde
[C-SEND-SYNC]: interoperability.html#c-send-sync
[C-GOOD-ERR]: interoperability.html#c-good-err
[C-NUM-FMT]: interoperability.html#c-num-fmt
[C-RW-VALUE]: interoperability.html#c-rw-value
[C-EVOCATIVE]: macros.html#c-evocative
[C-MACRO-ATTR]: macros.html#c-macro-attr
[C-ANYWHERE]: macros.html#c-anywhere
[C-MACRO-VIS]: macros.html#c-macro-vis
[C-MACRO-TY]: macros.html#c-macro-ty
[C-CRATE-DOC]: documentation.html#c-crate-doc
[C-EXAMPLE]: documentation.html#c-example
[C-QUESTION-MARK]: documentation.html#c-question-mark
[C-FAILURE]: documentation.html#c-failure
[C-LINK]: documentation.html#c-link
[C-METADATA]: documentation.html#c-metadata
[C-HTML-ROOT]: documentation.html#c-html-root
[C-RELNOTES]: documentation.html#c-relnotes
[C-HIDDEN]: documentation.html#c-hidden
[C-SMART-PTR]: predictability.html#c-smart-ptr
[C-CONV-SPECIFIC]: predictability.html#c-conv-specific
[C-METHOD]: predictability.html#c-method
[C-NO-OUT]: predictability.html#c-no-out
[C-OVERLOAD]: predictability.html#c-overload
[C-DEREF]: predictability.html#c-deref
[C-CTOR]: predictability.html#c-ctor
[C-INTERMEDIATE]: flexibility.html#c-intermediate
[C-CALLER-CONTROL]: flexibility.html#c-caller-control
[C-GENERIC]: flexibility.html#c-generic
[C-OBJECT]: flexibility.html#c-object
[C-NEWTYPE]: type-safety.html#c-newtype
[C-CUSTOM-TYPE]: type-safety.html#c-custom-type
[C-BITFLAG]: type-safety.html#c-bitflag
[C-BUILDER]: type-safety.html#c-builder
[C-VALIDATE]: dependability.html#c-validate
[C-DTOR-FAIL]: dependability.html#c-dtor-fail
[C-DTOR-BLOCK]: dependability.html#c-dtor-block
[C-DEBUG]: debuggability.html#c-debug
[C-DEBUG-NONEMPTY]: debuggability.html#c-debug-nonempty
[C-SEALED]: future-proofing.html#c-sealed
[C-STRUCT-PRIVATE]: future-proofing.html#c-struct-private
[C-NEWTYPE-HIDE]: future-proofing.html#c-newtype-hide
[C-STRUCT-BOUNDS]: future-proofing.html#c-struct-bounds
[C-STABLE]: necessities.html#c-stable
[C-PERMISSIVE]: necessities.html#c-permissive
