### 自定义 Side Channel

你可以在C#和Python中创建自己的side channel，并使用它在两者之间传递自定义数据结构。这对于发送数据过于复杂或结构化而无法使用内置的`EnvironmentParameters`，或者与任何特定Agent（智能体）无关的数据（因此不适合作为Agent（智能体）观察）非常有用。

## 概述

为了使用side channel，必须在Unity和Python中实现它。

### Unity端

在Unity中，side channel需要实现`SideChannel`抽象类，并实现以下方法：

- `OnMessageReceived(IncomingMessage msg)`：你必须实现此方法并从IncomingMessage中读取数据。数据必须按照写入的顺序读取。

side channel还必须在构造函数中分配一个`ChannelId`属性。`ChannelId`是一个Guid（在Python中称为UUID），用于唯一标识一个side channel。这个Guid在C#和Python中必须相同。在通信期间只能有一个具有特定id的side channel。

要从C#向Python发送数据，创建一个`OutgoingMessage`实例，添加数据到其中，在side channel内部调用`base.QueueMessageToSend(msg)`方法，并调用`OutgoingMessage.Dispose()`方法。

要在Unity端注册一个side channel，调用`SideChannelManager.RegisterSideChannel`，并将side channel作为唯一参数。

### Python端

在Python中，side channel需要实现`SideChannel`抽象类。你必须实现：

- `on_message_received(self, msg: "IncomingMessage") -> None`：你必须实现此方法并从IncomingMessage中读取数据。数据必须按照写入的顺序读取。

side channel还必须在构造函数中分配一个`channel_id`属性。`channel_id`是一个UUID（在C#中称为Guid），用于唯一标识一个side channel。这个编号在C#和Python中必须相同。在通信期间只能有一个具有特定id的side channel。

要分配`channel_id`，请按如下方式调用抽象类构造函数，并传入适当的`channel_id`：

```python
super().__init__(my_channel_id)
```

要从Python向C#发送字节数组，创建一个`OutgoingMessage`实例，添加数据到其中，并在side channel内部调用`super().queue_message_to_send(msg)`方法。

要在Python端注册一个side channel，在创建`UnityEnvironment`对象时，将side channel作为参数传递。构造函数的一个参数（`side_channels`）是一个side channel列表。

## 示例实现

下面是一个简单的side channel实现示例，它将在Unity环境和Python之间交换ASCII编码的字符串。

### Unity C#代码示例

第一步是在Unity项目中创建`StringLogSideChannel`类。这里是一个`StringLogSideChannel`的实现，它将监听来自Python的消息并将其打印到Unity调试日志中，同时将错误消息从Unity发送到Python。

```csharp
using UnityEngine;
using Unity.MLAgents;
using Unity.MLAgents.SideChannels;
using System.Text;
using System;

public class StringLogSideChannel : SideChannel
{
    public StringLogSideChannel()
    {
        ChannelId = new Guid("621f0a70-4f87-11ea-a6bf-784f4387d1f7");
    }

    protected override void OnMessageReceived(IncomingMessage msg)
    {
        var receivedString = msg.ReadString();
        Debug.Log("From Python : " + receivedString);
    }

    public void SendDebugStatementToPython(string logString, string stackTrace, LogType type)
    {
        if (type == LogType.Error)
        {
            var stringToSend = type.ToString() + ": " + logString + "\n" + stackTrace;
            using (var msgOut = new OutgoingMessage())
            {
                msgOut.WriteString(stringToSend);
                QueueMessageToSend(msgOut);
            }
        }
    }
}
```

定义自定义side channel类后，我们需要确保实例化并注册它。通常可以在与side channel逻辑相关的地方进行此操作，例如在可能需要访问side channel数据的MonoBehaviour对象上。这里展示了一个简单的MonoBehaviour对象，它实例化并注册新的side channel。如果还没有这样做，请确保注册side channel的MonoBehaviour附加到将在Unity场景中存在的GameObject上。

```csharp
using UnityEngine;
using Unity.MLAgents;

public class RegisterStringLogSideChannel : MonoBehaviour
{
    StringLogSideChannel stringChannel;
    public void Awake()
    {
        // 创建Side Channel
        stringChannel = new StringLogSideChannel();

        // 当创建Debug.Log消息时，将其发送到stringChannel
        Application.logMessageReceived += stringChannel.SendDebugStatementToPython;

        // 必须使用SideChannelManager类注册该channel
        SideChannelManager.RegisterSideChannel(stringChannel);
    }

    public void OnDestroy()
    {
        // 注销Debug.Log回调
        Application.logMessageReceived -= stringChannel.SendDebugStatementToPython;
        if (Academy.IsInitialized)
        {
            SideChannelManager.UnregisterSideChannel(stringChannel);
        }
    }

    public void Update()
    {
        // 可选：如果按下空格键，抛出一个错误！
        if (Input.GetKeyDown(KeyCode.Space))
        {
            Debug.LogError("这是一个假的错误。空格键在Unity中被按下。");
        }
    }
}
```

### Python代码示例

现在我们已经创建了必要的Unity C#类，可以创建它们的Python对应类。

```python
from mlagents_envs.environment import UnityEnvironment
from mlagents_envs.side_channel.side_channel import (
    SideChannel,
    IncomingMessage,
    OutgoingMessage,
)
import numpy as np
import uuid

# 创建StringLogChannel类
class StringLogChannel(SideChannel):

    def __init__(self) -> None:
        super().__init__(uuid.UUID("621f0a70-4f87-11ea-a6bf-784f4387d1f7"))

    def on_message_received(self, msg: IncomingMessage) -> None:
        """
        注意：我们必须实现此方法以接收来自Unity的消息
        """
        # 我们简单地从消息中读取一个字符串并打印出来。
        print(msg.read_string())

    def send_string(self, data: str) -> None:
        # 将字符串添加到OutgoingMessage
        msg = OutgoingMessage()
        msg.write_string(data)
        # 我们调用此方法以排队要发送的数据
        super().queue_message_to_send(msg)
```

然后我们可以实例化新的side channel，启动一个带有该side channel活动的`UnityEnvironment`，并使用它从Python向Unity环境发送一系列消息。

```python
# 创建channel
string_log = StringLogChannel()

# 我们启动与Unity Editor的通信，并传入string_log side channel作为输入
env = UnityEnvironment(side_channels=[string_log])
env.reset()
string_log.send_string("环境已重置")

group_name = list(env.behavior_specs.keys())[0]  # 获取第一个group_name
group_spec = env.behavior_specs[group_name]
for i in range1000):
    decision_steps, terminal_steps = env.get_steps(group_name)
    # 我们发送数据到Unity：包含每一步Agent（智能体）数量的字符串
    string_log.send_string(
        f"第 {i} 步发生，有 {len(decision_steps)} 个决定中的Agent（智能体）和 "
        f"{len(terminal_steps)} 个终端Agent（智能体）"
    )
    env.step()  # 将模拟向前推进

env.close()
```

现在，如果运行此脚本并在提示时按下Unity Editor中的`Play`按钮，Unity Editor中的控制台将每一步显示一条消息。此外，如果在Unity引擎中按下空格键，终端中会显示一条消息。