### 背景：PyTorch

正如我们在机器学习背景页面中讨论的那样，我们在ML-Agents工具包中提供的许多算法都利用了某种形式的深度学习。更具体地说，我们的实现是基于开源库PyTorch的。在本页中，我们提供了PyTorch和TensorBoard的简要概述，这些工具在ML-Agents工具包中被使用。

#### PyTorch

PyTorch是一个用于使用数据流图进行计算的开源库，这是深度学习模型的基础表示。它在桌面、服务器或移动设备上的CPU和GPU上都能促进训练和推理。在ML-Agents工具包中，当您训练一个代理的行为时，输出的是一个模型（.onnx）文件，您可以将其与代理关联起来。除非您实现一个新的算法，否则PyTorch的使用大多是抽象的，隐藏在幕后。

#### TensorBoard

使用PyTorch训练模型的一个组成部分是设置某些模型属性的值（称为超参数）。找到这些超参数的正确值可能需要几次迭代。因此，我们利用了一个名为TensorBoard的可视化工具。它允许在训练过程中可视化某些代理属性（例如奖励），这有助于建立对不同超参数的直觉，并为您的Unity环境设置最佳值。我们在“训练ML-Agents”页面中提供了更多关于设置超参数的详细信息。如果您不熟悉TensorBoard，我们推荐我们的指南“使用TensorBoard与ML-Agents”或本教程。

参考链接：
- [PyTorch](https://pytorch.org/)
- [TensorFlow Summaries and TensorBoard](https://www.tensorflow.org/programmers_guide/summaries_and_tensorboard)
- [TensorBoard教程](https://github.com/dandelionmane/tf-dev-summit-tensorboard-tutorial)
