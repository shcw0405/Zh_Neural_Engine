# 如何让我的模型在 ANE 上运行？

Core ML 会尽可能地尝试在 ANE 上运行你的模型，但你不能*强制*Core ML 使用 ANE。

在创建 Core ML 模型实例时，你可以传入一个`MLModelConfiguration`对象：

```swift
let config = MLModelConfiguration()
config.computeUnits = .all

let model = try MyModel(configuration: config)
```

要允许模型在 ANE 上运行，使用`computeUnits = .all`。

请注意，这并不*保证*模型将在 ANE 上运行，它只是告诉 Core ML 你更倾向于在可用时使用 ANE。如果设备没有 ANE，Core ML 将自动回退到 GPU 或 CPU。

如果可能，Core ML 将在 ANE 上运行整个模型。然而，当遇到不支持的层时，它会切换到另一个处理器。即使 Core ML 理论上可以在 GPU 上运行模型的第二部分，它也可能决定使用 CPU。在这里你不能假设任何事情。（请注意，在许多神经网络操作上，CPU 实际上可能比 GPU 更快，所以这不一定是坏事。）
