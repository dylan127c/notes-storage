# notes-storage

This repository is only be used to save my personal markdown notes. 

<p align="center">
<img src="images/README.images/image-20230822052311538.png" alt="image-20230822052311538"  />
</p>

Typora 是一款轻量级 Markdown 编辑器，适用于 OS X、Windows 和 Linux 三种操作系统。于个人而言，Typora 在 PC 端的表现十分优秀，但遗憾的是 Typora 并不支持移动端。

出于个人习惯，目前采用类似于“中转”的方案，即借助 GitHub 来完成 Markdown 文件在 PC 端和移动端之间同步，以实现在移动端阅读或编辑 Markdown 文件。移动端选择的 Markdown 编辑器为 Mweb，因为 Mweb 支持添加外部文件夹，不过其实只要是支持添加外服文件夹的编辑器就都能够使用，一般问题不大。

需要注意，不同的 Markdown 编辑器会采用不同的解析引擎，因此在不同的操作系统上使用不同的 Markdown 编辑器时，建议使用更为精确的语法，精确的语法可以保持笔记在不同 Markdown 编辑器中的阅读表现。

例如，在 Markdown 的表格结构中如若需要使用竖线，那么一般需要对它进行转义。

但如果使用的是 Typora 编辑器，情况则有所不同，因为 Typora 允许用户在表格结构内**直接**使用竖线，即不需要进行转义。显然，对于 Typora 来说这是一种优化，它能够显著减少用户在编写文件时所使用的代码量。

遗憾的是，这样**富有个性的语法**在交由 Mweb 编辑器解析时会出现错误，Mweb 的解析引擎会将未转义的竖线视为表格结构之一，从而导致某些表格内容的显示缺失。

解决这种解析异常最直接的方式，就是尽可能少用或不用编辑器提供的特殊语法，并尽量使用更为标准的 Markdown 语法。
