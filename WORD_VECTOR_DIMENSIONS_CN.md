# 词向量维度说明 (Word Vector Dimensions)

## 概述 (Overview)

该脑机接口文本生成系统使用了多个不同维度的向量表示来处理从大脑信号到文本的转换过程。

## 系统中使用的词向量维度 (Word Vector Dimensions Used in the System)

### 1. 神经信号特征维度 (Neural Signal Features)
- **维度**: 512维
- **描述**: 来自大脑运动皮层的神经信号特征
- **组成**: 
  - 256个电极，每个电极提取2个特征（阈值交叉和尖峰带功率）
  - 512个特征 = 256电极 × 2特征/电极
- **配置位置**: `model_training/rnn_args.yaml` 中的 `n_input_features: 512`

### 2. RNN隐藏状态维度 (RNN Hidden State Dimensions)
- **维度**: 768维
- **描述**: GRU神经网络解码器的隐藏状态维度
- **用途**: 将神经信号特征转换为音素预测
- **配置位置**: `model_training/rnn_args.yaml` 中的 `n_units: 768`

### 3. 语言模型词向量维度 (Language Model Word Embedding Dimensions)
- **维度**: 4096维
- **模型**: Facebook OPT-6.7B
- **描述**: 用于语言建模和文本重排序的预训练Transformer模型
- **词汇表大小**: 50,272个词汇
- **用途**: 对音素序列生成的候选句子进行重排序和优化

## 系统架构流程 (System Architecture Flow)

```
大脑信号 (512维) → RNN解码器 (768维隐藏状态) → 音素序列 → N-gram语言模型 → OPT-6.7B重排序 (4096维词向量) → 最终文本
```

## 配置文件位置 (Configuration File Locations)

1. **神经网络模型配置**: `model_training/rnn_args.yaml`
2. **语言模型配置**: `language_model/language-model-standalone.py`

## 技术细节 (Technical Details)

### RNN模型参数
- 输入维度: 512 (神经特征)
- 隐藏层维度: 768
- 网络层数: 5层GRU
- 输出类别数: 41 (音素类别)

### OPT-6.7B语言模型参数
- 词嵌入维度: 4096
- 词汇表大小: 50,272
- 模型参数量: 67亿
- 注意力头数: 32
- 隐藏层数: 32

## 参考文献 (References)

Card, N.S., Wairagkar, M., Iacobacci, C., et al. (2024). An Accurate and Rapidly Calibrating Speech Neuroprosthesis. *The New England Journal of Medicine*.