# 使用 os_log 查看警告/错误消息

Core ML 可能会打印类似"[coreml] Error computing NN outputs -1"或"Error plan build: -1"的错误消息到控制台。不幸的是，这些消息对诊断错误原因并不是很有帮助。

要获取更有用的消息，请在**Instruments 中使用 os_log 工具**运行你的应用。Core ML 将在此处打印更多信息。

这对于确定你的模型中的所有层是否在 ANE 上受支持也很有用。如果不支持，Core ML 可能会打印相关消息。

你也可以从设备下载日志并检查它们。例如，要下载最近 1 天（`1d`）的日志：

```nohighlight
$ sudo log collect --device --last 1d
```

这会创建一个`system_logs.logarchive`文件夹。你可以双击它在 Console 应用中打开，然后搜索`coreml`或`espresso`。

使用命令行：

```nohighlight
$ log show --archive system_logs.logarchive --predicate '(subsystem IN {"com.apple.espresso","com.apple.coreml"}) && (category IN {"espresso","coreml"})' --info --debug --last 1d
```

日志会为模型及其层的名称显示`<private>`，但你可以通过[在 Mac 上安装设备配置文件](https://superuser.com/questions/1532031/how-to-show-private-data-in-macos-unified-log)来解决这个问题。

如果你在 Mac 上运行 Core ML，你可以使用`log stream`实时查看与 Core ML 相关的日志消息：

```nohighlight
$ log stream --predicate '(subsystem IN {"com.apple.espresso","com.apple.coreml"}) && (category IN {"espresso","coreml"})' --info --debug
```
