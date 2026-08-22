# 从零开始构建你自己的 LLM：GPT 与 Claude 背后的五阶段流水线（中文）

**原文来源：** [X 帖子](https://x.com/sairahul1/status/2066076937806856661)　**作者：** [@sairahul1](https://x.com/sairahul1)　**发布日期：** 2026-06-14

本文为上述原文的中文译文。文中的数据、引语与判断均归属于原作者；核对原意时请参阅本页下方的“英文原文”。

![图像](https://pbs.twimg.com/media/HKwoQdtacAAecbc?format=jpg\&name=large)

![图像](https://pbs.twimg.com/media/HKweGDsa0AAk3KQ?format=jpg\&name=large)

人人都在谈论 LLM。

却没人说清它们底层到底是怎么工作的。

GPT。Claude。Gemini。Llama。

它们都来自同一个五阶段流水线。

而一旦你理解了它，你也能亲手构建一个。

不是 GPT-4 的复制品。

而是一个真正能够工作的语言模型。

一个能从文本中学习、生成新文本、并且输出真正有意义内容的模型。

我亲手做了一个。下面就是它的完整工作原理。

无需博士学位。代码已附。

## 关于 LLM 的普遍误解

大多数人认为，构建 LLM 的关键在于架构。

Transformer。注意力头。层数。

事实并非如此。

Transformer 架构早已公开发表。每个主流实验室使用的底层模块都大同小异。

如果架构是秘诀，那人人都会拥有 GPT-4。

真正的秘诀在于：**数据、训练与对齐。**

架构只占一段话的篇幅。其余一切，才是决定模型成败的战场。

以下是五个阶段。

***

## 阶段一：数据（模型成败的真正战场）

![图像](https://pbs.twimg.com/media/HKweaA6bwAEf_kK?format=jpg\&name=large)

从互联网上抓来的原始文本，简直是一团糟。

你不能直接用它训练。

Common Crawl——这个被大多数 LLM 用于训练的公开网络爬虫数据集——包含 2500 亿个页面，超过 100 万 GB。

但其中大部分是垃圾。

重复页眉。垃圾邮件。有害内容。个人数据。只有三个字的低质量页面。

在训练开始之前，你必须执行一套严苛的多步过滤：

→ 从原始 HTML 中提取干净文本

→ 过滤有害、NSFW（Not Safe For Work，不宜在工作场所浏览的内容）和个人数据

→ 按 URL、文档和行级别去重

→ 根据字数和 token 密度剔除低质量文档

→ 运行基于模型的质量评分——维基百科编辑会引用这个页面吗？

→ 在代码、书籍、科学文献和网页之间平衡数据配比

结果：一个干净的数据集，体量只有原始数据的很小一部分，但质量却有了质的飞跃。

**请记住这条铁律：**

数据质量胜过数据数量。无一例外。

这个领域最严守的秘密不是架构。

而是数据是如何被清洗的。

***

## 阶段二：Tokenization（文本分词）

![图像](https://pbs.twimg.com/media/HKwfo7aacAAyFSd?format=jpg\&name=large)

模型从不阅读原始文本。

它阅读的是 token。

一个 token 并不总是一个完整的词。它可能是词的一部分——一个模型学会将其视为最小单位的片段。

"playing" → \["play", "ing"] "unbelievable" → \["un", "believ", "able"] "dog" → \["dog"]

标准方法叫做**字节对编码（Byte-Pair Encoding, BPE）**。

它从单个字符开始，反复合并出现频率最高的字符对，直到形成一个固定大小的词表——通常在 32,000 到 100,000 个 token 之间。

下面是一个最简 tokenizer 的 Python 实现：

```python
from tokenizers import Tokenizer, models, trainers, pre_tokenizers

# 初始化 BPE tokenizer
# Initialize BPE tokenizer
tokenizer = Tokenizer(models.BPE())
tokenizer.pre_tokenizer = pre_tokenizers.Whitespace()

# 在语料上训练
# Train on your corpus
trainer = trainers.BpeTrainer(
    vocab_size=32000,
    special_tokens=["<PAD>", "<BOS>", "<EOS>", "<UNK>"]
)
tokenizer.train(files=["your_data.txt"], trainer=trainer)
tokenizer.save("tokenizer.json")

# 测试
# Test it
output = tokenizer.encode("Building an LLM from scratch is powerful")
print(output.tokens)
# ['Building', 'an', 'LL', 'M', 'from', 'scratch', 'is', 'powerful']

print(output.ids)
# [4821, 271, 3728, 44, 505, 8905, 318, 6787]
```

经验法则：1 个 token ≈ 0.75 个词。

1,000 个 token ≈ 750 个词。

10 万 token 的上下文窗口，大致相当于一本完整的小说。

***

## 阶段三：训练（一个简单到近乎荒谬的目标）

![图像](https://pbs.twimg.com/media/HKwgBktbkAAmzoE?format=jpg\&name=large)

整个训练目标简单到近乎荒谬：

**预测下一个 token。**

仅此而已。

给定 "The cat sat on the"，预测 "mat"。

在数万亿个样本上反复训练，奇迹就会出现。

模型学会了语法。然后是事实。然后是推理。然后是写代码、翻译语言、解数学题。

这些能力，没有一项是有人专门教给它的。

它们都是从海量数据上的下一 token 预测中涌现出来的。

下面是一个最简的仅解码器 Transformer（decoder-only transformer）的 PyTorch 实现——这也是所有主流 LLM 采用的架构：

```python
import torch
import torch.nn as nn
import math

class CausalSelfAttention(nn.Module):
    def __init__(self, embed_dim, num_heads):
        super().__init__()
        self.num_heads = num_heads
        self.head_dim = embed_dim // num_heads
        self.qkv = nn.Linear(embed_dim, 3 * embed_dim, bias=False)
        self.proj = nn.Linear(embed_dim, embed_dim, bias=False)

    def forward(self, x):
        B, T, C = x.shape
        q, k, v = self.qkv(x).chunk(3, dim=-1)
        
        # 拆分为多个注意力头
        # Split into heads
        q = q.view(B, T, self.num_heads, self.head_dim).transpose(1, 2)
        k = k.view(B, T, self.num_heads, self.head_dim).transpose(1, 2)
        v = v.view(B, T, self.num_heads, self.head_dim).transpose(1, 2)

        # 缩放点积注意力 + 因果掩码
        # Scaled dot-product attention with causal mask
        scale = math.sqrt(self.head_dim)
        attn = (q @ k.transpose(-2, -1)) / scale
        
        # 因果掩码：只能关注当前位置及之前的 token
        # Causal mask: can only attend to past tokens
        mask = torch.tril(torch.ones(T, T, device=x.device))
        attn = attn.masked_fill(mask == 0, float('-inf'))
        attn = torch.softmax(attn, dim=-1)
        
        out = (attn @ v).transpose(1, 2).contiguous().view(B, T, C)
        return self.proj(out)

class TransformerBlock(nn.Module):
    def __init__(self, embed_dim, num_heads, ff_dim, dropout=0.1):
        super().__init__()
        self.attn = CausalSelfAttention(embed_dim, num_heads)
        self.ff = nn.Sequential(
            nn.Linear(embed_dim, ff_dim),
            nn.GELU(),
            nn.Linear(ff_dim, embed_dim),
            nn.Dropout(dropout)
        )
        self.ln1 = nn.LayerNorm(embed_dim)
        self.ln2 = nn.LayerNorm(embed_dim)

    def forward(self, x):
        x = x + self.attn(self.ln1(x))   # 注意力 + 残差连接
        x = x + self.ff(self.ln2(x))     # 前馈网络 + 残差连接
        return x

class MiniLLM(nn.Module):
    def __init__(self, vocab_size, embed_dim, num_heads,
                 ff_dim, num_layers, max_seq_len, dropout=0.1):
        super().__init__()
        self.token_emb = nn.Embedding(vocab_size, embed_dim)
        self.pos_emb = nn.Embedding(max_seq_len, embed_dim)
        self.blocks = nn.ModuleList([
            TransformerBlock(embed_dim, num_heads, ff_dim, dropout)
            for _ in range(num_layers)
        ])
        self.ln_final = nn.LayerNorm(embed_dim)
        self.output = nn.Linear(embed_dim, vocab_size, bias=False)
        self.dropout = nn.Dropout(dropout)

    def forward(self, token_ids):
        B, T = token_ids.shape
        positions = torch.arange(T, device=token_ids.device).unsqueeze(0)
        
        x = self.dropout(
            self.token_emb(token_ids) + self.pos_emb(positions)
        )
        for block in self.blocks:
            x = block(x)
        
        x = self.ln_final(x)
        return self.output(x)  # 输出词表上的 logits

# 初始化一个小型但真实的模型
model = MiniLLM(
    vocab_size=32000,
    embed_dim=512,
    num_heads=8,
    ff_dim=2048,
    num_layers=6,
    max_seq_len=1024
)

total_params = sum(p.numel() for p in model.parameters())
print(f"Parameters: {total_params:,}")
# Parameters: 44,082,176 — 一个 4400 万参数的模型
```

接下来是训练循环：

```python
import torch.optim as optim
from torch.nn.utils import clip_grad_norm_

device = torch.device("cuda" if torch.cuda.is_available() else "cpu")
model = model.to(device)

optimizer = optim.AdamW(model.parameters(), lr=3e-4, weight_decay=0.01)
criterion = nn.CrossEntropyLoss()

def train_epoch(model, dataloader):
    model.train()
    total_loss = 0
    
    for input_ids, target_ids in dataloader:
        input_ids = input_ids.to(device)
        target_ids = target_ids.to(device)
        
        # 前向传播
        logits = model(input_ids)
        
        # 展平以计算损失
        loss = criterion(
            logits.view(-1, logits.size(-1)),   # (batch * seq_len, vocab)
            target_ids.view(-1)                  # (batch * seq_len)
        )
        
        # 反向传播
        optimizer.zero_grad()
        loss.backward()
        clip_grad_norm_(model.parameters(), max_norm=1.0)  # 防止梯度爆炸
        optimizer.step()
        
        total_loss += loss.item()
    
    return total_loss / len(dataloader)
```

模型真正学到的是：

→ 每个输入 token 都会关注它之前的所有 token

→ 因果掩码（causal mask）阻止它偷看未来

→ 损失 = 模型对真实下一个 token 的"惊讶程度"

→ 损失越低 = 预测越准 = 模型正在学会语言

***

## 阶段四：对齐（将文本预测器转变为助手）

![图像](https://pbs.twimg.com/media/HKwgz62bEAA7RsK?format=jpg\&name=large)

预训练结束后，你会得到一个看似很厉害的东西——但用来聊天，完全派不上用场。

问它一个问题，它可能回你三个问题。

因为预测下一个 token，不等于理解你想要什么。

两步就能解决这个问题。

**第一步：监督微调（Supervised Fine-Tuning, SFT）**

向模型展示数千个示例：

提示词 → 理想回复

模型学会模仿优质答案的格式。

令人惊讶的部分是：你其实只需要很少的数据。

几千个示例就足够了，因为知识已经蕴含在预训练模型中。

SFT 所做的，只是教会模型用正确的格式表达这些知识。

```python
# SFT 训练示例结构
sft_examples = [
    {
        "prompt": "Explain what an API is in simple terms.",
        "response": "An API is like a waiter in a restaurant. You (the app) tell the waiter (API) what you want. The waiter goes to the kitchen (server), gets it, and brings it back. You never go to the kitchen directly."
    },
    {
        "prompt": "What is the capital of France?",
        "response": "The capital of France is Paris."
    }
    # 几千个这样的示例就足够了
]

# 格式化为：<prompt> [SEP] <response> <EOS>
# 在预训练模型上微调这些配对
# 训练循环与预训练相同——只是数据不同
```

**第二步：RLHF（基于人类反馈的强化学习）**

SFT 教会格式。RLHF 教会偏好。

模型生成两个答案。人类选择更好的那个。这些偏好被用来训练一个奖励模型。LLM 被优化以最大化该奖励。

这就是为什么 ChatGPT 和 Claude 给人的感觉是"助手"——而不是随机文本生成器。

没有 RLHF：

→ 流畅。有能力。但不可靠。

→ 自信地犯错。

→ 不知道何时该说"我不知道"。

有了 RLHF：

→ 有帮助。清晰。安全。

→ 学会"一个好答案"究竟意味着什么。

***

## 阶段五：评估（证明它真的有效）

![图像](https://pbs.twimg.com/media/HKwhflrbwAAPa80?format=jpg\&name=large)

只训练不评估，无异于蒙眼射箭。

**预训练期间——衡量困惑度（perplexity）。**

困惑度衡量模型对真实文本的"惊讶程度"。

困惑度越低 = 模型预测文本越准 = 它在学习。

从 2017 年到 2023 年，最好的模型将困惑度从约 70（相当于面对 70 个等概率选择）降到了不到 10。

```python
import torch
import math

def calculate_perplexity(model, dataloader, device):
    model.eval()
    total_loss = 0
    total_tokens = 0
    criterion = nn.CrossEntropyLoss(reduction='sum')
    
    with torch.no_grad():
        for input_ids, target_ids in dataloader:
            input_ids = input_ids.to(device)
            target_ids = target_ids.to(device)
            
            logits = model(input_ids)
            
            loss = criterion(
                logits.view(-1, logits.size(-1)),
                target_ids.view(-1)
            )
            total_loss += loss.item()
            total_tokens += target_ids.numel()
    
    avg_loss = total_loss / total_tokens
    perplexity = math.exp(avg_loss)
    return perplexity

# 示例困惑度变化：
# Epoch 1: Perplexity = 847.3  (模型几乎一无所知)
# Epoch 5: Perplexity = 124.6  (正在进步)
# Epoch 20: Perplexity = 23.4  (真正学会了语言)
```

**对齐之后——困惑度不再有效。**

微调后的模型困惑度分数反而更差，但实用性却大幅提升。

你需要人工基准测试：

→ **MMLU（Massive Multitask Language Understanding，大规模多任务语言理解）**：涵盖 57 个学科的选择题——衡量知识广度

→ **Chatbot Arena**：人类对两个模型进行盲测投票——衡量人类偏好

→ **AlpacaEval**：LLM 评判 LLM——与人工评判相关性高达 98%，成本仅 10 美元

**说实话：没有任何单一分数能够全面衡量一个模型的好坏。**

同一个模型在同一个基准上的得分可能是 0.637 或 0.488——仅仅取决于提示词的格式。

评估之难，货真价实。

至今无人彻底解决。

***

## 如何让模型生成文本

![图像](https://pbs.twimg.com/media/HKwiLJEbAAAqsLO?format=jpg\&name=large)

模型已经训练好了。

现在让它生成点什么。

```python
def generate(model, tokenizer, prompt, max_new_tokens=100,
             temperature=0.8, device='cuda'):
    model.eval()
    
    # 将提示词编码为 token ID
    token_ids = tokenizer.encode(prompt).ids
    input_tensor = torch.tensor(token_ids, dtype=torch.long,
                                device=device).unsqueeze(0)
    
    with torch.no_grad():
        for _ in range(max_new_tokens):
            
            # 只保留最后 max_seq_len 个 token（上下文窗口）
            context = input_tensor[:, -1024:]
            
            # 前向传播
            logits = model(context)
            
            # 只取最后一个 token 的 logits
            next_token_logits = logits[:, -1, :]
            
            # 应用温度（越高 = 越有创意）
            next_token_logits = next_token_logits / temperature
            
            # 从概率分布中采样
            probs = torch.softmax(next_token_logits, dim=-1)
            next_token = torch.multinomial(probs, num_samples=1)
            
            # 追加并继续
            input_tensor = torch.cat([input_tensor, next_token], dim=1)
            
            # 遇到序列结束 token 则停止
            if next_token.item() == tokenizer.token_to_id("<EOS>"):
                break
    
    # 解码回文本
    generated_ids = input_tensor[0].tolist()
    return tokenizer.decode(generated_ids)

# 测试
output = generate(
    model, tokenizer,
    prompt="The most important thing about machine learning is",
    max_new_tokens=100,
    temperature=0.8
)
print(output)
```

温度（temperature）控制创造力：

→ temperature = 0.1 → 安全、可预测、重复

→ temperature = 0.8 → 自然、多变，推荐默认值

→ temperature = 1.5 → 有创意、出人意料，有时不连贯

***

## 完整流水线长什么样

![图像](https://pbs.twimg.com/media/HKwi9ABagAAQANk?format=jpg\&name=large)

之前：原始互联网文本，100 万 GB 之巨，完全不可用。

阶段一之后：干净、经过过滤的数据集，可以开始训练。

之前：原始文本，对模型毫无意义。

阶段二之后：带有 ID 的 token，模型的"母语"。

之前：随机权重，输出垃圾。

阶段三之后：一个理解语言规律的模型。

之前：一个文本预测器，用更多问题来回答你的问题。

阶段四之后：一个遵循指令、安全可靠的助手。

之前：不知道模型到底好不好。

阶段五之后：基准测试、困惑度分数、人工评估。

每个阶段都建立在前一个阶段之上。

跳过任何一个，整个系统就会崩溃。

***

## 五个会毁掉 LLM 项目的错误

**1. 过度迷恋架构。**

Transformer 已经标准化。已发表。被复制。

架构是最不重要的部分。

**2. 把数据当作大宗商品。**

无论算力多强，脏数据都会限制模型的上限。

顶级实验室在数据清洗上的投入，比模型设计还多。

**3. 忽略缩放数学。**

模型相对于数据量过大，会导致训练不足并浪费算力。

最优比例：每个参数约对应 20 个训练 token。

**4. 停在 SFT 就结束。**

微调后的模型只会模仿。没有 RLHF，它永远学不会人类真正偏好什么。

**5. 对齐后仍然迷信困惑度。**

训练后阶段改变了模型的输出分布。

一旦运行 SFT，困惑度就不再有意义。

立即切换到人工基准测试。

***

## 一个令人不安的真相

一个伟大的 LLM 不是"训练"出来的。

它是"工程化"出来的。

五个阶段。不是一个。

架构只是阶段三里的一段话。

真正重要的东西，全在另外四个阶段里。

数据质量。缩放数学。对齐。诚实的评估。

这些才是区分 GPT-4 和业余模型的关键。

两个使用相同架构的实验室，产出的模型可能天差地别。

架构是公开的。

真正重要的东西，不是。

***

## 动手实践

想自己动手跑一遍？这里是最简配置：

```plaintext
# 最简 LLM 训练完整配置
# 依赖：pip install torch tokenizers datasets

# 1. 获取数据
from datasets import load_dataset
dataset = load_dataset("wikitext", "wikitext-103-v1", split="train")
text = "\n\n".join([t.strip() for t in dataset['text'] if t.strip()])

# 2. 训练 tokenizer
from tokenizers import Tokenizer, models, trainers, pre_tokenizers
tokenizer = Tokenizer(models.BPE())
tokenizer.pre_tokenizer = pre_tokenizers.Whitespace()
trainer = trainers.BpeTrainer(vocab_size=8000,
                               special_tokens=["<PAD>","<BOS>","<EOS>"])
with open("corpus.txt", "w") as f:
    f.write(text[:5_000_000])  # 先用 5MB
    # use 5MB to start
tokenizer.train(["corpus.txt"], trainer)

# 3. 构建模型（使用上方阶段三的 MiniLLM 类）
model = MiniLLM(
    vocab_size=8000,
    embed_dim=256,       # 小但真实
    num_heads=8,
    ff_dim=1024,
    num_layers=4,
    max_seq_len=256
)
# ~1500 万参数——在笔记本 GPU 上几小时即可训练完成

# 4. 训练（使用上方阶段三的 train_epoch）
# 5. 生成（使用上方阶段五的 generate()）

print("Your LLM is running.")
```

从小做起。1500 万参数。WikiText 数据集。Google Colab 的免费 GPU。

看着困惑度在几小时内从 800 降到 50。

那个下降曲线，就是模型正在学习语言的信号。

实时发生。

那一刻，一切豁然开朗。

***

## 现在让它变得有用：构建一个真正的垂直领域 LLM

![图像](https://pbs.twimg.com/media/HKwlBG7bUAAILY-?format=jpg\&name=large)

WikiText 是一个学习用的数据集。

真正的价值——也是真正的乐趣——在于在特定领域上训练，看着你的模型变成某个领域的专家。

以下是五个你可以立即着手构建的垂直领域。同样的流水线。不同的数据。

***

## 垂直领域一：编程助手 LLM（主要示例：影响力最大，效果最显著）

这是你应该首先构建的那个。

痛点是普适的。每个开发者都经历过：

→ 你盯着一个不工作的函数。

→ Stack Overflow 上有 12 个答案，全都来自 2014 年。

→ 你粘贴到 ChatGPT 里，得到的结果差不多对，但又不完全对。

一个在正确数据上训练的编程 LLM，可以离线、原生地做到这一点，并且针对你的技术栈量身定制。

**数据来源：**

```python
from datasets import load_dataset

# 代码数据集——来自 GitHub 的 640 万个 Python 文件
# The Code dataset — 6.4M Python files from GitHub
code_dataset = load_dataset("codeparrot/github-code",
                             languages=["Python"],
                             streaming=True,
                             split="train")

# Stack Overflow 问答对——真实的开发者问题 + 被采纳的答案
# Stack Overflow Q&A pairs — real developer problems + accepted answers
so_dataset = load_dataset("koutch/stackoverflow_python",
                           split="train")

# The Stack——30 种编程语言，已清洗并去重
# The Stack — 30 programming languages, cleaned and deduplicated
the_stack = load_dataset("bigcode/the-stack",
                          data_dir="data/python",
                          split="train",
                          streaming=True)
```

**训练对长什么样：**

```python
# 格式 1：代码补全
# 输入：函数签名 + 文档字符串
# 目标：完整实现

input_example = """
def calculate_compound_interest(principal, rate, time, n=12):
    \"\"\"
    Calculate compound interest.
    Args:
        principal: Initial investment amount
        rate: Annual interest rate (as decimal, e.g. 0.05 for 5%)
        time: Time period in years
        n: Compounding frequency per year (default: monthly)
    Returns:
        Final amount after compound interest
    \"\"\"
"""

target_example = """
    amount = principal * (1 + rate/n) ** (n * time)
    return round(amount, 2)
"""

# 格式 2：错误 → 修复对（来自 Stack Overflow）
# Format 2: Error → Fix pairs (from Stack Overflow)
input_example_2 = """
# ERROR: TypeError: unsupported operand type(s) for +: 'int' and 'str'
user_age = input("Enter your age: ")
next_year_age = user_age + 1
print(f"Next year you will be {next_year_age}")

# Fix this code:
"""

target_example_2 = """
user_age = int(input("Enter your age: "))  # convert string to int
next_year_age = user_age + 1
print(f"Next year you will be {next_year_age}")
"""

# 格式 3：自然语言 → 代码（指令遵循）
# Format 3: Natural language → Code (instruction following)
input_example_3 = """
# Write a Python function that:
# - Takes a list of dictionaries
# - Filters by a given key-value pair
# - Returns sorted results by a specified field
"""

target_example_3 = """
def filter_and_sort(data, filter_key, filter_value, sort_field):
    filtered = [item for item in data
                if item.get(filter_key) == filter_value]
    return sorted(filtered, key=lambda x: x.get(sort_field, ''))
"""
```

**训练前后对比，最能说明价值：**

训练前：

提示词："Write a Python decorator that retries a function on failure" 输出："A decorator is a design pattern in Python that allows..." **说明：** 泛泛而谈。毫无用处。教科书式回答。

在 GitHub + Stack Overflow 上训练 10 小时后：

提示词："Write a Python decorator that retries a function on failure" 输出：

```python
import time
from functools import wraps

def retry(max_attempts=3, delay=1.0, exceptions=(Exception,)):
    def decorator(func):
        @wraps(func)
        def wrapper(*args, **kwargs):
            for attempt in range(max_attempts):
                try:
                    return func(*args, **kwargs)
                except exceptions as e:
                    if attempt == max_attempts - 1:
                        raise
                    time.sleep(delay * (attempt + 1))
        return wrapper
    return decorator

# Usage:
@retry(max_attempts=3, delay=0.5, exceptions=(ConnectionError,))
def fetch_data(url):
    ...
# 正确。生产级代码。包含指数退避。
```

模型学会了资深开发者会怎么写。

不是被教会的。而是通过阅读 600 万个 Python 文件和每一个被采纳的 Stack Overflow 答案学会的。

![图像](https://pbs.twimg.com/media/HKwjhhBbkAAsjhh?format=jpg\&name=large)

***

## 垂直领域二：SQL 查询生成器

痛点：每个不懂技术的创始人都有数据，却拿不到。

数据就在那里。在他们的数据库里。他们只是不会写查询。

他们用自然语言描述想要什么。你的模型写出 SQL。

**数据来源：**

```python
# Spider 数据集——10,000+ 条带自然语言描述的 SQL 查询
# Spider dataset — 10,000+ SQL queries with natural language descriptions
# 覆盖 200 个数据库、138 个领域
spider = load_dataset("spider", split="train")

# WikiSQL——80,654 条自然语言 + SQL 配对
# WikiSQL — 80,654 natural language + SQL pairs
wikisql = load_dataset("wikisql", split="train")

# 一个训练对示例：
# What a training pair looks like:
example = {
    "question": "Show me all users who signed up in the last 30 days and made at least one purchase",
    "sql": """
        SELECT u.id, u.email, u.created_at, COUNT(p.id) as purchase_count
        FROM users u
        JOIN purchases p ON u.id = p.user_id
        WHERE u.created_at >= NOW() - INTERVAL '30 days'
        GROUP BY u.id, u.email, u.created_at
        HAVING COUNT(p.id) >= 1
        ORDER BY u.created_at DESC;
    """
}
```

**训练前后对比：**

```python
# 不懂技术的创始人输入：
"Which of my customers spent the most money last quarter
 but haven't bought anything in the last 60 days?"

# 模型输出：
SELECT
    c.customer_id,
    c.email,
    SUM(o.total_amount) as q_spend,
    MAX(o.created_at) as last_order_date
FROM customers c
JOIN orders o ON c.customer_id = o.customer_id
WHERE o.created_at BETWEEN DATE_TRUNC('quarter', NOW() - INTERVAL '3 months')
                        AND DATE_TRUNC('quarter', NOW())
GROUP BY c.customer_id, c.email
HAVING MAX(o.created_at) < NOW() - INTERVAL '60 days'
ORDER BY q_spend DESC
LIMIT 20;
```

谁会为此付费：每个 SaaS 创始人、每个电商运营者、每个坐在企业主和数据库之间的分析师。

他们会毫不犹豫地每月付 20 美元。

***

## 垂直领域三：美国法律文件摘要生成器

痛点：一份 40 页的合同。一份租赁协议。一份保密协议（NDA）。

大多数人签字时根本看不懂。

律师收费 300 美元/小时来读它。

你的模型 3 秒就读完。

**数据来源：**

```python
# Free Law Project——数百万条美国法院意见，公共领域
# Free Law Project — millions of US court opinions, public domain
free_law = load_dataset("free-law/courtlistener-opinion-clustering",
                         split="train")

# MultiLegalPile——美国各司法管辖区的法律文本
# MultiLegalPile — legal text across US jurisdictions
multi_legal = load_dataset("joelniklaus/multi_legal_pile",
                            languages=["en"],
                            split="train")

# 训练对示例：
# What training pairs look like:
example = {
    "input": """
    SECTION 12.4 INDEMNIFICATION. Licensee shall defend, indemnify,
    and hold harmless Licensor and its officers, directors, employees,
    agents, and successors from and against any and all losses, damages,
    liabilities, deficiencies, claims, actions, judgments, settlements,
    interest, awards, penalties, fines, costs, or expenses of whatever
    kind, including reasonable attorneys' fees, that are incurred by
    Licensor arising out of or relating to Licensee's breach of any
    representation, warranty, covenant, or obligation under this Agreement...
    [continues for 3 more paragraphs of dense legalese]
    """,

    "summary": """
    PLAIN ENGLISH: If you break any part of this contract and it causes
    the other party legal trouble or financial loss, YOU pay for everything —
    their lawyers, court costs, damages, all of it. This is a broad
    indemnification clause that heavily favors the licensor.

    RED FLAGS:
    • "any and all losses" — extremely broad, no cap on liability
    • "reasonable attorneys' fees" — you pay their legal bills too
    • No carve-out for licensor's own negligence

    NEGOTIATE: Ask to add "except to the extent caused by Licensor's
    gross negligence or willful misconduct" before signing.
    """
}
```

**让它值得付费的输出格式：**

```python
输入：[粘贴任何美国合同条款]

输出：
━━━━━━━━━━━━━━━━━━
简明英语摘要
━━━━━━━━━━━━━━━━━━
[2-3 句任何人都能理解的话]

风险警示 🚩
• [该条款中的具体风险]
• [它允许对方做什么]

缺失的保护 ⚠️
• [平衡条款应包含什么]
• [律师会要求添加什么]

风险等级：高 / 中 / 低
━━━━━━━━━━━━━━━━━━
```

谁会为此付费：审阅客户合同的自由职业者、小企业主、审阅租赁协议的租客、签署首个 SaaS 协议的初创公司创始人。

市场是每一个请不起律师但还得签字的人。

***

## 垂直领域四：医疗症状解释器

痛点：你在 Google 上搜索一个症状，WebMD 告诉你只剩 3 天可活。

一个在临床笔记和患者教育材料上训练的医疗 LLM 会做出不同的回应。

它用平实的英语解释症状实际意味着什么。不会危言耸听。

```python
# PubMed 摘要——3500 万篇医学研究论文
# PubMed abstracts — 35 million medical research papers
pubmed = load_dataset("pubmed", split="train")

# MedQA——美国医学执照考试题目 + 答案
# MedQA — US Medical Licensing Exam questions + answers
medqa = load_dataset("bigbio/med_qa", split="train")

# 训练对格式：
# Training pair format:
example = {
    "patient_question": "I've had a dull ache behind my left eye for 3 days. It gets worse when I lean forward. No fever. What could this be?",

    "clinical_explanation": """
    Based on your description — dull ache behind one eye, worsening when leaning forward,
    no fever, lasting 3 days — the most likely explanation is:

    MOST LIKELY: Sinusitis (sinus pressure)
    The maxillary or frontal sinuses sit directly behind and above your eyes.
    Inflammation or congestion increases pressure when you change head position.
    The fact that it worsens leaning forward is a classic sinusitis pattern.

    ALSO POSSIBLE: Tension headache, eye strain, or mild migraine variant

    LESS LIKELY but worth noting: If you also have vision changes, nausea,
    or the pain is sudden and severe — see a doctor same day.

    WHAT TO TRY: Saline nasal rinse, steam inhalation, ibuprofen for pressure relief.
    If no improvement in 7-10 days or fever develops → see a doctor.

    NOTE: This is general health information, not a diagnosis.
    Always consult a licensed physician for medical advice.
    """
}
```

关键：每条回复都以明确的免责声明和升级信号结尾。

这就是区分有用健康信息和危险建议的地方。

***

## 垂直领域五：电商产品描述撰写器

痛点：一个 Shopify 店铺有 500 个产品。每个描述都是一堵没人会读的规格表文字墙。

好的产品描述只做一件事：让人产生某种感觉。

一个在高质量转化文案上训练的 LLM，能精准学会哪些词能驱动点击。

```python
# 训练数据结构：
# Training data structure:
example = {
    "product_specs": """
    Product: Ceramic Coffee Mug
    Material: Stoneware ceramic
    Capacity: 14oz
    Dimensions: 3.5" diameter x 4.2" height
    Colors: Matte black, cream white, sage green
    Dishwasher safe: Yes
    Microwave safe: Yes
    Weight: 0.8 lbs
    """,

    "high_converting_description": """
    Some mornings deserve better than a paper cup.

    This is the mug that stays on your desk. The one your coworkers
    ask about. Heavy enough to feel intentional. Smooth enough to
    actually enjoy holding at 7am.

    14oz — the right amount. Not the novelty bucket. Not the tiny
    espresso thing. The one you actually finish.

    Matte stoneware that doesn't show fingerprints. Dishwasher safe
    because life is short. Three colors that work with any kitchen
    that isn't trying too hard.

    You already have mugs.
    You don't have this one.
    """,

    "meta_description": "Stoneware ceramic mug, 14oz, matte finish. Dishwasher and microwave safe. The coffee mug that stays.",

    "keywords": ["ceramic coffee mug", "stoneware mug", "matte black mug",
                 "14oz mug", "minimalist coffee mug", "handmade style mug"]
}
```

**数据来源：** 爬取流量排名前 1000 的 Shopify 店铺。提取产品标题、规格和描述。筛选评价数高的产品——这些描述已被证明能转化。用这些来训练。

你的模型学会了规格表和能卖货的文案之间的区别。

***

## 五个垂直领域的共同模式

看看它们共享什么：

→ 一个清晰、具体、用户每天都在感受的痛点

→ 一个已经存在且公开可获取的数据源

→ 一个对该行业的任何人来说都一目了然的"前后对比"

→ 付费意愿，因为替代方案花费更多时间或金钱

五阶段流水线对每一个都完全相同。

你只改变一样东西：训练数据。

同样的 tokenizer 配置。同样的 Transformer 架构。同样的训练循环。同样的评估方法。

不同的数据 → 不同的专家 → 不同的产品。

这就是杠杆所在。

一个流水线。五个产品。五条收入流。

***

如果这篇文章对你有用：

→ 转发分享，让更多学习 AI 的开发者看到

→ 关注 [@sairahul1](https://x.com/@sairahul1) 获取更多类似的深度解析

→ 收藏本文——代码可用，今晚就跑起来

我写 AI、写产品、写那些你睡着时也在替你运转的系统。
