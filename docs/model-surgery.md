# 如何替换不支持的层

这里有一个简单的脚本，可以让你将`addBroadcastable`层替换为普通的旧式`add`层。

可广播版本是在 Core ML 3 中引入的，功能更强大，但在 ANE 上[并不总是有效](unsupported-layers.md)。在大多数情况下，旧的`add`层执行等效操作（它在某种程度上也支持广播）并且与 ANE 兼容。

```python
import coremltools

model = coremltools.models.MLModel("YourModel.mlmodel")
spec = model._spec
nn = spec.neuralNetwork

# 注意：如果你的模型是分类器，请使用以下内容：
# nn = spec.neuralNetworkClassifier

for layer in nn.layers:
    if layer.WhichOneof("layer") == "addBroadcastable":
        layer.add.MergeFromString(b"")

new_model = coremltools.models.MLModel(spec)
new_model.save("YourNewModel.mlmodel")
```

你可能还需要对模型进行其他更改。要了解如何编辑 mlmodel 文件，请查看我的电子书[Core ML 生存指南](https://leanpub.com/coreml-survival-guide)。
