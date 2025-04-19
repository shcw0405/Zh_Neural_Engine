# 神经引擎（Neural Engine）— 我们对它了解多少？

大多数新型 iPhone 和 iPad 都配备了**神经引擎**，这是一种可以让机器学习模型运行得非常快的特殊处理器，但关于这种处理器实际如何工作，公开的信息并不多。

苹果神经引擎（Apple Neural Engine，简称 ANE）是一种**NPU**，即神经网络处理单元（Neural Processing Unit）。它类似于 GPU，但不同的是，NPU 不是用来加速图形处理，而是加速神经网络操作，如卷积和矩阵乘法。

ANE 并不是唯一的 NPU——除了苹果之外，许多公司也在开发自己的 AI 加速芯片。除了神经引擎，最著名的 NPU 是[谷歌的 TPU](https://en.wikipedia.org/wiki/Tensor_processing_unit)（即张量处理单元，Tensor Processing Unit）。

## 为什么要写这份文档？

当我还在为 iOS 提供机器学习咨询服务时，经常收到人们的邮件，他们感到困惑为什么他们的模型似乎没有在神经引擎上运行，或者**为什么它运行得如此缓慢**，而 ANE 应该比 GPU 快得多...

事实证明，**并非每个 Core ML 模型都能充分利用 ANE**。原因可能很复杂，因此本文档试图回答最常见的问题。

ANE 非常适合让机器学习模型在 iPhone 和 iPad 上快速运行。针对 ANE 优化的模型性能将明显优于 CPU 和 GPU。但 ANE 也有局限性。不幸的是，**苹果没有为第三方开发者提供任何关于如何优化模型以利用 ANE 的指导**。要弄清楚什么有效，什么无效，主要是一个试错的过程。

> **注意：** 这里的所有信息都是通过实验获得的。我不在苹果工作，也从未在那里工作过，所以我不了解这个芯片的任何实现细节。这些信息中有些可能是错误的。它肯定是不完整的。如果你知道这里没有解释的内容，或者你发现有错误或遗漏的信息，请[提交问题](https://github.com/hollance/neural-engine/issues)或[提出拉取请求](https://github.com/hollance/neural-engine/pulls)。谢谢！

我原本计划写一篇[博客文章](http://machinethink.net/blog)，但决定把它放在 GitHub 上，使其成为一个社区资源，这样其他人也可以为其贡献。请踊跃参与！

## 目录

- [哪些设备有 ANE？](docs/supported-devices.md)
- [为什么我应该关心 ANE？](docs/why-care.md)
- [如何让我的模型在 ANE 上运行？](docs/running-on-ane.md)
- [如何防止我的模型在 ANE 上运行？](docs/prevent-running-on-ane.md)
- [我的模型在使用 ANE 吗？](docs/is-model-using-ane.md)
- [我可以直接编程 ANE 吗？](docs/programming-ane.md)
- [ANE 和 GPU 不是一样的吗？](docs/ane-vs-gpu.md)
- [ANE 是 16 位的吗？](docs/16-bit.md)
- [哪些 Core ML 层不被 ANE 支持？](docs/unsupported-layers.md)
- [使用 os_log 查看警告/错误消息](docs/os-log.md)
- [如何替换不支持的层](docs/model-surgery.md)
- [ANE 内部如何工作？](docs/internals.md)
- [其他奇怪的问题](docs/other.md)
- [逆向工程 ANE](docs/reverse-engineering.md)

## 致谢

本仓库是[hollance/neural-engine](https://github.com/hollance/neural-engine)的中文翻译版本。感谢原作者 Matthijs Hollemans 的精彩工作和对社区的贡献。
