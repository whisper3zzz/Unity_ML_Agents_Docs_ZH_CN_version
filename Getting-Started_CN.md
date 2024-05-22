# 入门指南

本指南将逐步介绍如何在Unity中打开我们的[示例环境](Learning-Environment-Examples.md)，训练其中的Agent，并将训练好的模型嵌入Unity环境中。阅读本教程后，你应该能够训练任意一个示例环境。如果你不熟悉[Unity引擎](https://unity3d.com/unity)，请查看我们的[背景：Unity](Background-Unity.md)页面获取有用的提示。此外，如果你不熟悉机器学习，请查看我们的[背景：机器学习](Background-Machine-Learning.md)页面以获取简要概述和有用的提示。

![3D Balance Ball](images/balance.png)

在本指南中，我们将使用**3D Balance Ball**环境，其中包含多个Agent（智能体）立方体和球（都是彼此的副本）。每个Agent（智能体）立方体通过水平或垂直旋转来防止其球掉落。在此环境中，Agent（智能体）立方体是一个**Agent（智能体）（Agent）**，每平衡球一步就会获得奖励。Agent（智能体）掉落球时也会受到负奖励。训练过程的目标是让Agent（智能体）学习在其头上平衡球。

让我们开始吧！

## 安装

如果尚未安装，请按照[安装说明](Installation.md)进行操作。然后，打开包含所有示例环境的Unity项目：

1. 在菜单中导航到 `Window -> Package Manager` 打开包管理器窗口。
2. 导航到ML-Agents包并点击它。
3. 找到 `3D Ball` 示例并点击 `Import`。
4. 在 **Project** 窗口中，转到 `Assets/ML-Agents/Examples/3DBall/Scenes` 文件夹并打开 `3DBall` 场景文件。

## 了解Unity环境

Agent（智能体）是观察和与环境交互的自主行为体。在Unity的上下文中，环境是一个包含一个或多个Agent对象的场景，以及Agent（智能体）交互的其他实体。

![Unity Editor](images/mlagents-3DBallHierarchy.png)

**注意**：在Unity中，场景中所有内容的基础对象是_GameObject_。GameObject本质上是一个包含其他内容的容器，包括行为、图形、物理等。要查看构成GameObject的组件，选择场景窗口中的GameObject，然后打开Inspector窗口。Inspector会显示GameObject上的每个组件。

打开3D平衡球场景后，你可能会注意到其中包含多个Agent（智能体）立方体。场景中的每个Agent（智能体）立方体都是一个独立的Agent（智能体），但它们都共享相同的行为。3D平衡球通过这种方式加速训练，因为所有十二个Agent（智能体）并行贡献于训练。

### Agent（智能体）

Agent（智能体）是观察环境并采取行动的行为体。在3D平衡球环境中，Agent（智能体）组件放置在十二个“Agent”GameObjects上。基础Agent（智能体）对象具有一些影响其行为的属性：

- **行为参数（Behavior Parameters）** — 每个Agent（智能体）必须有一个行为。行为决定了Agent（智能体）如何做出决策。
- **最大步数（Max Step）** — 定义在Agent（智能体）的剧集结束之前可以发生的模拟步数。在3D平衡球中，Agent（智能体）在5000步后重新开始。

#### Behavior Parameters : Vector Observation Space

在做出决策之前，Agent（智能体）会收集关于其在世界中的状态的观察。向量观察是一个包含相关信息的浮点数向量，供Agent（智能体）做出决策使用。

3D平衡球示例的行为参数使用了`空间大小`为8。这意味着包含Agent（智能体）观察的特征向量包含八个元素：Agent（智能体）立方体旋转的`x`和`z`分量以及球的相对位置和速度的`x`、`y`和`z`分量。

#### Behavior Parameters : Actions

Agent（智能体）以Action的形式接受指令。ML-Agents工具包将动作分为两类：连续动作和离散动作。3D平衡球示例被编程为使用连续动作，即一个浮点数向量，可以连续变化。更具体地说，它使用`空间大小`为2来控制自身施加的`x`和`z`旋转量，以保持球在其头上的平衡。

## 运行预训练模型

我们为Agent（智能体）提供了预训练模型（`.onnx`文件），并使用[Unity推理引擎](Unity-Inference-Engine.md)在Unity中运行这些模型。在本节中，我们将使用3D Ball示例的预训练模型。

1. 在 **Project** 窗口中，转到 `Assets/ML-Agents/Examples/3DBall/Prefabs` 文件夹。展开`3DBall`并点击`Agent`预制件。在 **Inspector** 窗口中你应该能看到`Agent`预制件。

   **注意**：`3DBall`场景中的平台是使用`3DBall`预制件创建的。与其单独更新所有12个平台，不如更新`3DBall`预制件。

   ![平台预制件](images/platform_prefab.png)

2. 在 **Project** 窗口中，将位于 `Assets/ML-Agents/Examples/3DBall/TFModels` 中的 **3DBall** 模型拖到 **Inspector** 窗口中 `Behavior Parameters (Script)` 组件下的`Model`属性中。

   ![3dball学习大脑](images/3dball_learning_brain.png)

3. 你应该注意到 **Hierarchy** 窗口中每个`3DBall`下的每个`Agent`现在都包含`Behavior Parameters`中的`Model`为 **3DBall**。**注意**：你可以使用场景层次结构中的搜索栏一次选择多个场景中的游戏对象进行修改。
4. 将模型的 **推理设备（Inference Device）** 设置为 `CPU`。
5. 点击Unity编辑器中的 **播放（Play）** 按钮，你会看到平台使用预训练模型来平衡球。

## 使用强化学习训练新模型

虽然我们为此环境中的Agent（智能体）提供了预训练模型，但你自己制作的任何环境都需要从头开始训练Agent（智能体），以生成新的模型文件。在本节中，我们将演示如何使用ML-Agents Python包中的强化学习算法来实现这一目标。我们提供了一个便捷的命令`mlagents-learn`，它接受用于配置训练和推理阶段的参数。

### 训练环境

1. 打开命令或终端窗口。
2. 导航到你克隆`ml-agents`库的文件夹。**注意**：如果你按照默认[安装](Installation.md)进行操作，则应该能够从任何目录运行`mlagents-learn`。
3. 运行`mlagents-learn config/ppo/3DBall.yaml --run-id=first3DBallRun`。
   - `config/ppo/3DBall.yaml`是我们提供的默认训练配置文件的路径。`config/ppo`文件夹包含所有示例环境的训练配置文件，包括3DBall。
   - `run-id`是本次训练会话的唯一名称。
4. 当屏幕上显示“_按下Unity编辑器中的播放按钮开始训练_”消息时，你可以按Unity中的 **播放（Play）** 按钮开始在编辑器中进行训练。

如果`mlagents-learn`运行正确并开始训练，你应该会看到如下内容：

```console
INFO:mlagents_envs:
'Ball3DAcademy' started successfully!
Unity Academy name: Ball3DAcademy

INFO:mlagents_envs:Connected new brain:
Unity brain name: 3DBallLearning
        Number of Visual Observations (per agent): 0
        Vector Observation space size (per agent): 8
        Number of stacked Vector Observation: 1
INFO:mlagents_envs:Hyperparameters for the PPO Trainer of brain 3DBallLearning:
        batch_size:          64
        beta:                0.001
        buffer_size:         12000
        epsilon:             0.2
        gamma:               0.995
        hidden_units:        128
        lambd:               0.99
        learning_rate:       0.0003
        max_steps:           5.0e4
        normalize:           True
        num_epoch:           3
        num_layers:          2
        time_horizon:        1000
        sequence_length:     64
        summary_freq:        1000
        use_recurrent:       False
        memory_size:         256
        use_curiosity:       False
        curiosity_strength:  0.01
        curiosity_enc_size:  128
        output_path: ./results/first3DBallRun/3DBallLearning
INFO:mlagents.trainers: first3DBallRun: 3DBallLearning: Step: 1000. Mean Reward: 1.242. Std of Reward: 0.746. Training.
INFO:mlagents.trainers: first3DBallRun: 3DBallLearning: Step: 2000. Mean Reward: 1.319. Std of Reward: 0.693. Training.
INFO:mlagents.trainers: first3DBallRun: 3DBallLearning: Step