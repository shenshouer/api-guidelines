# Rust API 准则

这是关于如何设计和展示 Rust 编程语言的 API 的一组建议。它们主要由 Rust 库团队基于在 Rust 中生态系统构建标准库和其 crate 的经验所撰写。

这些只是准则，有些准则比其他准则更为严格。在某些情况下，它们仍然有些模糊，当前仍在开发中完善中。
Rust crate 作者应在 Rust 库开发中将它们视为惯用与相互协作的一组重要因素考虑，并在他们认为合适的情况下使用。
这些准则不应以任何方式被视为 crate 创建者必须遵循的任务，尽管他们可能会发现，符合这些指导原则的 crate 可以比那些没有遵循的 crate 更好地与现有的 crate 生态系统集成。

这本书由两部分组成：所有单独准则的简明清单，适合在回顾 crate 时快速浏览；和局部
包含详细解释指南的章节。

如果您有兴趣对 API 准则做出贡献，请查看[contributing.md]并加入我们的[Gitter channel]。

[checklist]: checklist.html
[contributing.md]: https://github.com/rust-lang/api-guidelines/blob/master/CONTRIBUTING.md
[gitter channel]: https://gitter.im/rust-impl-period/WG-libs-guidelines
