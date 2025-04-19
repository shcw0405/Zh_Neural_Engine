# 我的模型在使用 ANE 吗？

尽管你可以使用`MLModelConfiguration`告诉 Core ML 你对使用 CPU/GPU/ANE 的偏好，但没有 API 可以在运行时查询它当前在哪种硬件上运行模型。

Core ML 也可能将你的模型分成多个部分，并使用不同的处理器运行每个部分。所以它可能在同一推理过程中同时使用 ANE*和*CPU 或 GPU。

## H11ANEServicesThread

当你的应用在设备上运行时，在调试器中按下暂停按钮。如果有一个名为**H11ANEServicesThread**的线程，那么 Core ML 至少在你的模型的某些部分使用了神经引擎。

## 尝试`computeUnits`

将`MLModelConfiguration`的`computeUnits`选项从`.all`更改为`.cpuAndGPU`和`.cpuOnly`。

如果你的模型使用`.all`的性能与使用其他选项相比并没有快很多，那么 Core ML 可能只在模型的一部分上使用 ANE——或者根本没有使用。

## 断点

确定 Core ML 正在哪个处理器上运行模型的一种方法是设置**符号断点**并进入调试器。

想知道你的模型是否在 ANE 上运行？设置一个断点在`-[_ANEModel program]`（重要：准确地拼写它，前面带有破折号。）如果这个断点被触发，Core ML 正在使用 ANE。

然而...Core ML 可能决定在 ANE 上运行你的模型的一部分，其余部分在 GPU 或 CPU 上运行。仅仅因为你的应用触发了`-[_ANEModel program]`并不意味着*整个*模型都在 ANE 上运行。为了确保这一点，你需要设置几个其他断点。

Core ML 使用三种不同的"引擎"，它们都在私有的 Espresso 框架中实现：

- ANE — `Espresso::ANERuntimeEngine`
- GPU — `Espresso::MPSEngine`和`Espresso::MetalLowmemEngine`
- CPU — `Espresso::BNNSEngine`

在运行你的模型时，它可以在这些引擎之间切换。通过查看调试器的堆栈跟踪中出现的引擎名称，你可以看到当前正在使用哪个引擎/处理器。

要查找模型是否（也）在 GPU 或 CPU 上运行，你可以使用以下符号断点：

- `Espresso::MPSEngine::context::__launch_kernel`
- `Espresso::BNNSEngine::convolution_kernel::__launch`
- `Espresso::elementwise_kernel_cpu::__launch`

### 要寻找哪些符号？

上面提到的符号可能会在 iOS 版本之间改变。要找到可以用于断点的其他可能的符号，在设备或模拟器上运行你的应用并进入调试器。在调试器提示符下输入以下内容：

```nohighlight
(lldb) image list Espresso
```

这会打印出私有 Espresso 框架的路径，类似这样（在你的机器上会不同）：

```nohighlight
/Users/matthijs/Library/Developer/Xcode/iOS DeviceSupport/13.3 (17C54) arm64e/Symbols/System/Library/PrivateFrameworks/Espresso.framework/Espresso
```

接下来，执行以下操作：

```nohighlight
(lldb) image dump symtab 'put-the-path-here'
```

这会从 Espresso 框架中转储整个符号表。你应该能在其中找到一些有趣的符号。:-)

如果所有其他方法都失败了，使用以下断点之一，并用调试器手动逐步执行代码：

- `Espresso::layer::__launch`
- `Espresso::net::__forward`

如果你最终进入了一个类似`Espresso::elementwise_kernel_cpu::__launch`的函数，这是一个很好的提示，表明你现在正在 CPU 上运行。

## Instruments 时间分析器

如果你不确定要设置哪些断点，或者你想了解模型不同部分的运行速度，请使用**时间分析器**（Time Profiler）工具运行你的应用。

我建议创建一个新的空应用。在循环中调用你的 Core ML 模型大约 100 次左右。

使用时间分析器模板在 Instruments 中运行应用，并打开调用树。

> **提示：** 按住 Option 键单击一个项目可以展开其下的所有内容。这将节省你大量点击操作。

找到符号`-[MLNeuralNetworkEngine predictionFromFeatures:...]`。右键单击它，选择**聚焦于子树**（Focus on subtree）。

现在你将看到在模型执行过程中被调用的所有函数，以及它们相对占用的时间。这些函数的名称将提供关于模型在哪里执行的良好线索，你也可以将它们用作断点。

当使用 ANE 时，还会有一个对`-[_ANEClient evaluateWithModel...]`的调用。这似乎是测量在 ANE 上运行模型所需的时间。

## 使用`powermetrics`

当你的应用在 MacOS 机器上运行时，在终端中输入`sudo powermetrics`，它会告诉你每种处理单元的功耗等信息，例如：

```
CPU Power: 335 mW
GPU Power: 37 mW
ANE Power: 0 mW
Combined Power (CPU + GPU + ANE): 372 mW
```
