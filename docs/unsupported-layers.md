# 哪些 Core ML 层不被 ANE 支持？

> :warning: **警告！** 此处提供的信息不完整且**可能有误**。它甚至可能在 ANE 的不同版本之间有所不同。在调试机器学习模型的性能问题时，可将此作为起点，但不要将其视为绝对真理。如果你知道这里没有解释的内容，或者你发现有错误或遗漏的信息，请[提交问题](https://github.com/hollance/neural-engine/issues)或[提出拉取请求](https://github.com/hollance/neural-engine/pulls)。谢谢！

Core ML 不在 ANE 上运行模型的主要原因是该模型包含某些层类型。

Core ML 可能会在 ANE 上运行模型的第一部分，然后切换到 GPU（或 CPU）运行模型的其余部分。理论上，它可以进行多次这样的切换，但设备之间的每次切换都会带来开销。

如果你的模型是`S → U → S → U → S → U`，其中 S 是支持的层，U 是不支持的层，Core ML 可能只会执行一次`ANE → GPU`，而不是`ANE → GPU → ANE → GPU → 等`。然而，`ANE → CPU → ANE → CPU → 等`似乎确实会发生，因为在 ANE 和 CPU 之间切换比在 ANE 和 GPU 之间切换成本更低。

话虽如此，Core ML 主要是一个**黑盒**。我们无法了解 Core ML 如何决定在哪个处理器上运行模型的哪个部分。唯一的方法是[设置一些断点](is-model-using-ane.md)并进行尝试。

**提示：** 如果你的模型只有一两层不被 ANE 支持，明智的做法是[编辑 mlmodel 文件](https://leanpub.com/coreml-survival-guide)，用一个 ANE*能够*支持的替代方案替换这些层。在 ANE 上运行整个模型将使你的模型更快速和更节能。

## 有问题的层

这个列表绝不是详尽的，但以下层类型已知在 ANE 上不工作：

- 自定义层
- RNN 层，如 LSTM 或 GRU
- gather（收集）
- 扩张卷积
- 可广播和"ND"层
- 某些广播操作（例如，`CxHxW`和`Cx1x1`张量的乘法）
- 内核大小大于 13 或步长大于 2 的池化层
- 缩放因子大于 2 的上采样

> **注意：** 其中一些是猜测。很难确定 Core ML 一半时间*实际上*在做什么。

## 可广播层

Core ML 3[添加了许多新层](https://machinethink.net/blog/new-in-coreml3/)，支持广播，例如：

- `AddBroadcastableLayer`
- `DivideBroadcastableLayer`
- `MultiplyBroadcastableLayer`
- `SubtractBroadcastableLayer`
- 以及其他一些...

还有一些新层的名称中带有"ND"，表示它们可以在任何秩的张量上运行：

- `ConcatNDLayer`
- `SplitNDLayer`
- `LoadConstantNDLayer`
- 等等...

这些层不会在 ANE 上运行。当遇到这种"可广播"或"ND"层类型时，Core ML 将回退到 CPU 或 GPU。

好消息是：大多数时候你实际上不需要这些新的层类型！它们主要在张量没有通常的`(batch, channels, height, width)`形状时有用。

你通常可以用 Core ML 2 中较旧的层类型替换这些可广播层。这些旧层类型仍然允许有限的广播。而且它们在 ANE 上大多能正常工作。

问题是 coremltools 4 中的新转换器倾向于使用这些新的可广播层。同样，当你使用参数`minimum_ios_deployment_target='13'`时，旧转换器也是如此。

发生这种情况时，如果你的模型没有[在 ANE 上运行](is-model-using-ane.md)，你可以进行一些[模型手术](model-surgery.md)，用旧版本替换这些层：

- `AddBroadcastableLayer` → `AddLayer`
- `MultiplyBroadcastableLayer` → `MultiplyLayer`或`ScaleLayer`或线性`Activation`层
- `ConcatND` → `Concat`
- 等等...

例如，你可以用较旧的`LoadConstant`层替换`LoadConstantND`。旧版本只支持 3 级张量，所以如果你的张量使用更多维度，你可能需要将其中一些维度扁平化。这是否可能取决于你的模型架构。

可能并不总是有兼容的旧式层可用——例如，没有可以替换`SubtractBroadcastableLayer`的`SubtractLayer`——因此你可能需要聪明地引入额外的层作为解决方法（例如，使用 alpha = -1 的线性激活层对张量内容进行取反，然后使用`AddLayer`）。

## 自定义层

[自定义层](https://machinethink.net/blog/coreml-custom-layers/)允许你向 Core ML 添加新功能。但有一个缺点：你只能提供 CPU 和 GPU 实现。因为没有[编程 ANE](programming-ane.md)的公共 API，自定义层无法在 ANE 上运行。

如果自定义层出现在模型的末尾附近，将模型分成两部分并一个接一个地运行它们可能是有意义的。第一部分可以使用 ANE，而包含自定义层的第二部分将使用 CPU 或 GPU。这可能比在 CPU 或 GPU 上运行整个模型要快。

## 能正常工作的层

- 上采样：最近我使用的一个模型中，上采样层似乎在 ANE 上正常运行（Core ML 还有一个`Espresso::ANERuntimeEngine::upsample_kernel`），所以我猜这些是有效的。

- 反卷积

## 更多？

我还没有在 ANE 上尝试所有可能的 Core ML 层类型（这需要很多工作！）。如果你知道任何在 ANE 上有问题的层，请[发送反馈](https://github.com/hollance/neural-engine/issues)。谢谢！
