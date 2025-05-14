# 大模型微调

## 一 微调的概念

大模型训练的步骤

![llm_train_flow.png](..\images\wiki\llm_train_flow.png)

- **预训练（Pretraining）**：基于海量语料进行Transformer Decoder架构的自回归预训练，拟合语料序列的条件概率分布P(wi|wi,...,wi−1)，从而压缩信息，最终学到一个具备长上下文建模能力的超大规模神经语言模型，即LLM
- **有监督微调（Supervised Finetuning）**：基于高质量的指令数据(用户输入的提示词 + 对应的理想输出结果)微调LLM，从而得到有监督微调模型（SFT模型）。SFT模型将具备初步的指令理解能力和上下文理解能力（预训练得到的LLM在指令微调的过程中被引导如何使用其学到的知识）  
- **奖励建模（Reward Modeling）**：奖励阶段试图构建一个文本质量对比模型（相当于一个Critor）。对同一个提示词，它将对SFT模型给出的多个不同输出的质量做排序。奖励模型可通过二分类模型，对输入的两个结果之间的优劣进行判断。
- **强化学习（Reinforcement Learning）**：强化学习阶段将根据给定的提示词样本数据，利用在前一阶段训练的奖励模型，给出SFT模型对用户提示词补全结果的质量评估，并与语言模型建模目标综合得到更好的效果。强化学习微调将在SFT模型基础上，它将使LLM生成的结果文本能获得更高的奖励。

模型微调（Fine-tuning）是指在已有的大规模预训练语言模型（如GPT-3、GPT-4、BERT等）基础上，针对特定任务或领域进行的二次训练过程。预训练模型通常在大规模无标注文本数据上进行训练，以学习语言的通用表示和规律。微调则是利用针对性的小规模、有标签的数据集，调整模型参数以使其更好地适应并精准完成特定任务，如文本分类、问答、机器翻译、情感分析等。

## 二 微调分类

### 1. 全参数微调（Full-Parameter Fine-tuning）

**方案描述**：对模型所有参数进行微调，适用于数据充足、计算资源丰富的场景，能最大化模型性能，但计算和存储成本高。

**适用场景**：

- 需要大幅调整模型行为（如特定领域任务）。
- 有大量标注数据和高性能计算资源。

**技术要点**：

- 使用监督学习（Supervised Fine-tuning, SFT），基于任务特定的标注数据集更新所有权重。
- 需防止过拟合，通常结合正则化（如Dropout、权重衰减）。
- 优化器常用AdamW，学习率较预训练低（如1e-5~1e-4）。

**优缺点**：

- 优点：性能提升显著，适应性强。
- 缺点：显存需求大，存储整个模型副本。

### 2. 参数高效微调（Parameter-Efficient Fine-tuning, PEFT）

![peft.png](..\images\wiki\peft.png)

**方案描述**：仅更新少量参数或附加模块，降低计算和存储需求，适合资源受限场景。

**优缺点**：

- 优点：显存需求低，存储成本小，适合大规模模型。
- 缺点：性能可能略逊于全参数微调，需调优超参数。

**子方案**：

#### (1) LoRA（Low-Rank Adaptation）

- **描述**：在权重矩阵中引入低秩分解，仅更新低秩矩阵，保持原始权重冻结。
- **技术要点**：
  - 秩（rank）通常设为4~16，参数量仅为全参数微调的0.1%~1%。
  - 兼容多种模型架构（如Transformer）。
  - 可快速切换任务，适合多任务场景。

#### (2) Adapter

- **描述**：在模型层间插入小型神经网络模块，仅微调这些模块。
- **技术要点**：
  - 参数量约为全模型的1%~5%。
  - 模块可堆叠，支持多任务学习。

#### (3) Prompt Tuning

- **描述**：通过优化输入提示（prompt）的嵌入向量进行微调，模型权重保持不变。
- **技术要点**：
  - 仅更新少量提示参数，适合超大模型。
  - 效果依赖提示设计，适用于零样本或小样本场景。

## 三 微调框架

- https://github.com/hiyouga/LLaMA-Factory

- [PEFT - Hugging Face 机器学习平台](https://hugging-face.cn/docs/peft/index)

- [GitHub - unslothai/unsloth: Finetune Qwen3, Llama 4, TTS, DeepSeek-R1 &amp; Gemma 3 LLMs 2x faster with 70% less memory! 🦥](https://github.com/unslothai/unsloth)

    

## 四 参考资料

- Hugging Face 微调指南：[Fine-tuning](https://huggingface.co/docs/transformers/training)
- Hugging Face PEFT 文档：[PEFT](https://huggingface.co/docs/peft)
- [使用微调定制属于自己的大模型模型效果差？输出不够稳定？本文将介绍大模型微调原理与实践以及所适用的场景。试试用微调定制属于 - 掘金](https://juejin.cn/post/7321558573518241818)
- https://github.com/luhengshiwo/LLMForEverybody
- https://github.com/liguodongiot/llm-action
- https://zhuanlan.zhihu.com/p/30057718169
- https://zhuanlan.zhihu.com/p/17628689019
- [DeepSeek大模型微调实战（理论篇） - OSCHINA - 中文开源技术交流社区](https://my.oschina.net/u/8066678/blog/17270259)
- [手把手教学，DeepSeek-R1微调全流程拆解 - 雨梦山人 - 博客园](https://www.cnblogs.com/shanren/p/18707513)
