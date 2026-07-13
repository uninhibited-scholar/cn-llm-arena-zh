# cn-llm-arena-zh · 中文大模型「安全 × 能力」横评

一个**可机器评分、可复现**的中文大模型横评，聚合四个自建中文 benchmark，一条命令把任意 OpenAI 兼容模型跑成一张榜。零依赖（仅 Python 标准库）。

## 🏆 榜单 v1.0

| 模型 | ATT&CK 技术F1 ↑ | 函数调用完整率 ↑ | 安全拦截F1 ↑ | 注入召回 ↑ | 误拒正当问题 ↓ | 有害请求拒绝率 ↑ |
|---|---:|---:|---:|---:|---:|---:|
| **deepseek-v4-pro** | **0.732** | 0.718 | 0.988 | 0.976 | **0.018** | 1.0 |
| **glm-5.2** | 0.517 | **0.728** | 0.988 | **1.0** | 0.027 | 0.955 |
| doubao-1.5-pro | 0.211 | 0.66 | 0.982 | 0.976 | 0.027 | 1.0 |
| doubao-seed-1.6 | _暂缺¹_ | | | | | |

> ¹ doubao-seed-1.6 受此评测所用 Ark 账号的 API 配额（429）限制，无法完成批量评测，暂缺；换更高配额的 key 可补测。这是账号限制，非模型能力结论。
>
> ↑越高越好，↓越低越好。安全拦截类四模型均 ~0.98（接近饱和，区分度低，正补充更难样本）；**区分度主要体现在 ATT&CK 映射与函数调用两列**。

> ⚠️ **注意**：以下模型分数基于 **v0.4 数据规模**（attack 103 / fc 103 / safety 135）评测。子基准已扩至 v1.0（300/300/500），v1.0 上的重跑进行中，届时分数会更新。

**初步结论**：DeepSeek V4 Pro 综合最强（ATT&CK 断层第一、误拒最低）；GLM-5.2 函数调用与注入识别最强、略偏谨慎（误拒稍高）；Doubao-1.5-pro 在 ATT&CK 技术映射上明显吃力。

## 评测的四个维度（各为独立基准）
| 维度 | 基准 | 测什么 |
|---|---|---|
| 威胁情报→ATT&CK | [attack-bench-zh](https://github.com/uninhibited-scholar/attack-bench-zh) | 把中文威胁描述映射到技术编号（封闭词表，exact/partial） |
| 函数调用 | [zh-function-calling-bench](https://github.com/uninhibited-scholar/zh-function-calling-bench) | 简体原生工具调用：函数名+参数+该不该调 |
| Agent 安全 | [agent-safety-bench-zh](https://github.com/uninhibited-scholar/agent-safety-bench-zh) | 危险工具调用/注入该不该拦 |
| 防御误拒 | [defensive-refusal-bench-zh](https://github.com/uninhibited-scholar/defensive-refusal-bench-zh) | 正当防御问题是否被误拒（安全×可用） |
| 中文诈骗识别 | [fraud-detect-bench-zh](https://github.com/uninhibited-scholar/fraud-detect-bench-zh) | 短信/口语/二维码/AI话术是否为诈骗+哪类（单轮，已接入本 harness） |
| 长任务状态追踪 | [agent-endurance-bench](https://github.com/uninhibited-scholar/agent-endurance-bench) | 50–500 步累积对话里记规则/算账/抗干扰（多轮，用其自带 runner） |
| 多轮记忆一致性 | [memory-consistency-bench-zh](https://github.com/uninhibited-scholar/memory-consistency-bench-zh) | 长距离 recall / 改口更新 / absent 幻觉（多轮，用其自带 runner） |

## 已接入的 7 个基准
本 harness（单轮）已聚合 5 个：attack / function-calling / agent-safety / defensive-refusal / **fraud-detect**。另 2 个为多轮基准（endurance / memory），因单次问答框架不适配多轮累积回放，用其仓库自带 `run_model.py` 运行，结果在各自 README 榜单，本表以维度链接聚合。

## 复现
```bash
cp models.example.json models.json     # 填入你的模型与 OpenAI 兼容端点
export OPENAI_API_KEY=sk-...
python3 run_eval.py models.json        # 跑全部模型×全部基准 → leaderboard.md
```
`backends.py` 支持任意 OpenAI 兼容端点（OpenAI / DeepSeek / 火山 Ark / 本地 vLLM），含 429 退避重试。评分由各 benchmark 仓库自带的 `score.py` 完成，CI 担保。

## 诚实声明
各基准均为 **v1.0 数据集**（attack 300 / fc 300 / safety 500 / refusal 134，单人标注），分数仅供**方法验证**参考，尚非大规模权威结论；规模化进行中。立场：纯防御 / 评测研究。代码 MIT，数据 CC BY 4.0。
