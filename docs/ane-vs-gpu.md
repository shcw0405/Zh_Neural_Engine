# ANE 和 GPU 不是一样的吗？

ANE 和 GPU 是两个独立的处理器。它们与 CPU 核心一起位于同一芯片上，但它们是不同的部件。

以下是 A12 Bionic 的样子（图片来自[anandtech.com 和 TechInsights](https://www.anandtech.com/show/13393/techinsights-publishes-apple-a12-die-shot-our-take)）。如你所见，ANE（这里标记为 NPU）和 GPU 核心是芯片上的不同区域。

![A12芯片](https://images.anandtech.com/doci/13393/A12.jpg)

要编程 GPU，你需要使用**Metal**，苹果的 GPU 编程语言。除了图形着色器外，Metal 还允许你编写计算着色器。

iOS 和 macOS 都提供了**Metal Performance Shaders**（MPS），一个拥有许多图像处理任务预构建着色器的框架。MPS 还有用于神经网络操作（如卷积）的计算着色器。实际上，当 Core ML 在 GPU 上运行你的模型时，它在底层使用的是 MPS。

Metal 不能用于编程 ANE，它专门用于 GPU。目前没有直接编程 ANE 的公共框架。

Core ML 对不同处理器使用以下框架：

- **CPU：** BNNS，或 Basic Neural Network Subroutines，Accelerate.framework 的一部分
- **GPU：** Metal Performance Shaders（MPS）
- **ANE：** 私有框架

Core ML 可以拆分模型，使一部分在 ANE 上运行，另一部分在 GPU 或 CPU 上运行。当它这样做时，会在这些不同的框架之间切换。

iPhone、iPad 和搭载 Apple Silicon 的 Mac 拥有**共享内存**，这意味着 CPU、GPU 和 ANE 都使用相同的 RAM。共享内存的优势是你不需要将数据从一个处理器上传到另一个。但这并不意味着在处理器之间切换没有成本：数据仍需要转换为适合的格式。例如，在 GPU 上，数据需要先放入纹理对象中。

在基于 Intel 的 Mac 上，GPU 可能有自己的 RAM（通常称为 VRAM）。
