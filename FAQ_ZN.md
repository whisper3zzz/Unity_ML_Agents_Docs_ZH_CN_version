# 常见问题解答

## 安装问题

### 环境权限错误

如果你在不通过编辑器构建的情况下直接导入你的Unity环境，可能需要为其授予额外的执行权限。

如果在macOS上收到这样的权限错误，请运行以下命令：

```sh
chmod -R 755 *.app
```

如果在Linux上收到这样的权限错误，请运行以下命令：

```sh
chmod -R 755 *.x86_64
```

在Windows上，可以参考[此处的说明](https://docs.microsoft.com/en-us/previous-versions/windows/it-pro/windows-server-2008-R2-and-2008/cc754344(v=ws.11))。

### 环境连接超时

如果你能够从`UnityEnvironment`启动环境，但收到如下的超时错误：

```
UnityAgentsException: The Communicator was unable to connect. Please make sure the External process is ready to accept communication with Unity.
```

可能有多种原因：

- **原因**：场景中可能没有代理。
- **原因**：在OSX上，防火墙可能阻止了与环境的通信。**解决方案**：按照[此处的说明](https://support.apple.com/en-us/HT201642)将已构建的环境二进制文件添加到防火墙的例外列表中。
- **原因**：Unity环境中发生了错误，阻止了通信。**解决方案**：查看Unity环境生成的[日志文件](https://docs.unity3d.com/Manual/LogFiles.html)以了解发生了什么错误。
- **原因**：你在环境变量中设置了`HTTP_PROXY`和`HTTPS_PROXY`值。**解决方案**：删除这些值然后重试。
- **原因**：你在无图形界面的环境中运行（例如，远程连接到服务器）。**解决方案**：在`mlagents-learn`中传递`--no-graphics`，或者在`RemoteRegistryEntry.make()`或`UnityEnvironment`初始化器中传递`no_graphics=True`。如果需要图形用于视觉观察，则需要设置`xvfb`（或等效工具）。

### 通信端口{}仍在使用

如果收到异常 `"Couldn't launch new environment because communication port {} is still in use."`，可以在调用

```python
UnityEnvironment(file_name=filename, worker_id=X)
```

时更改Python脚本中的worker编号。

### 平均奖励：nan

如果在尝试使用PPO训练模型时收到`Mean reward : nan`的消息，这是由于学习环境的剧集未终止。为了解决这个问题，请在场景检查器中为代理设置`Max Steps`为大于0的值。或者，可以在脚本中手动设置自定义剧集终止事件的`done`条件。

### "文件名"无法打开，因为无法验证开发者。

如果你在macOS 10.15 (Catalina)或更高版本上通过github网站下载了代码库，尝试在Unity项目中播放场景时可能会看到此错误。解决方法包括使用Unity包管理器安装包（这是官方支持的方法 - 参见[这里](Installation.md)），或按照[这里](https://support.apple.com/en-us/HT202491)的说明逐个文件验证你机器上的相关文件。