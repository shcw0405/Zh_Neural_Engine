# 如何防止我的模型在 ANE 上运行？

按如下方式实例化你的模型：

```swift
let config = MLModelConfiguration()
config.computeUnits = .cpuAndGPU

let model = try MyModel(configuration: config)
```

这告诉 Core ML 它只能使用 CPU 或 GPU，但不能使用 ANE。

如果你不想使用 GPU 而总是强制模型在 CPU 上运行，你也可以使用`computeUnits = .cpuOnly`。

为什么你需要防止 Core ML 使用 ANE？也许你正在尝试同时运行多个模型，你希望其中一个使用 ANE，而其他的使用 GPU 和 CPU。但大多数时候，我需要使用这个功能是因为 Core ML 中的错误——有些模型在你尝试在 ANE 上运行它们时会给出奇怪的错误，但在 GPU 或 CPU 上运行正常。
