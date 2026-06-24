# 中文模型横评榜 v0.4（4 任务 · 机器评分）

> 数据：attack 103 / fc 103 / safety 135 / refusal 134（v0.4 种子集，单人标注+CI 担保）。↑越高越好，↓越低越好。

| 模型 | ATT&CK 技术F1 ↑ | 函数调用完整率 ↑ | 安全拦截F1 ↑ | 注入召回 ↑ | 过度拒绝率 ↓ | 误拒正当问题 ↓ | 有害请求拒绝率 ↑ |
|---|---:|---:|---:|---:|---:|---:|---:|
| **deepseek-v4-pro** | **0.732** | 0.718 | 0.988 | 0.976 | 0.0 | **0.018** | **1.0** |
| **glm-5.2** | 0.517 | **0.728** | 0.988 | **1.0** | 0.0 | 0.027 | 0.955 |
| doubao-1.5-pro | 0.211 | 0.66 | 0.982 | 0.976 | 0.0 | 0.027 | 1.0 |
| doubao-seed-1.6 | _暂缺¹_ | | | | | | |

---

## 初步结论

**DeepSeek V4 Pro 综合最强**：ATT&CK 技术映射断层第一（0.732），误拒率最低（0.018），有害请求全部拦截（1.0）；实际用途中最可信。

**GLM-5.2 紧随**：函数调用和注入识别最强（0.728 / 1.0），略偏谨慎（误拒率 0.027），适合对安全有极高要求的场景。

**Doubao-1.5-pro 在 ATT&CK 上明显吃力**：0.211 F1 说明对"中文威胁描述→技术编号"这类知识型任务吃力，可能需要微调或 RAG 补强。

---

## 看点与局限

- **ATT&CK / 函数调用区分度好**：两列能清晰看出模型实力差异
- **安全拦截类四家都 ~0.98（饱和）**：现有样本难度不够，正补充更刁钻样本（benign-but-scary, 伪装型破坏等）以拉开区分
- **所有基准都是诚实标注 + CI 可复现**：分数仅供方法验证参考，尚非大规模权威结论；规模化进行中

¹ **doubao-seed-1.6 暂缺原因**：思考型模型在此测试账号受 API 配额限制（429），无法完成批量评测。这是账号限制，非模型能力结论；换配额更高的 key 可补测。

---

## 使用这个榜单

```bash
git clone https://github.com/uninhibited-scholar/cn-llm-arena-zh
cd cn-llm-arena-zh

cp models.example.json models.json
# 填入你的模型与 OpenAI 兼容端点
export OPENAI_API_KEY=sk-...
python3 run_eval.py models.json        # 跑全部模型×基准 → 自动生成新榜单
```

零依赖，支持任意 OpenAI 兼容端点（OpenAI / DeepSeek / Ark / 本地 vLLM）。评分由各基准的 `score.py` 完成，CI 担保。

---

## 相关基准

| 基准 | GitHub | 什么问题 | v0.4 规模 |
|---|---|---|---|
| [attack-bench-zh](https://github.com/uninhibited-Scholar/attack-bench-zh) | 威胁→ATT&CK | 把中文威胁描述映射到技术编号（知识型） | 103 条 |
| [zh-function-calling-bench](https://github.com/uninhibited-scholar/zh-function-calling-bench) | 函数调用 | 简体原生工具调用：函数名+参数+该不该调 | 103 条 |
| [agent-safety-bench-zh](https://github.com/uninhibited-scholar/agent-safety-bench-zh) | Agent 安全 | 危险工具/注入该不该拦 | 135 条（105 原+30 硬样本） |
| [defensive-refusal-bench-zh](https://github.com/uninhibited-scholar/defensive-refusal-bench-zh) | 防御误拒 | 正当防御问题是否被误拒（安全×可用） | 134 条 |

都是 CC BY 4.0，单人标注，机器评分，CI 保证。

**立场**：纯防御 / 评测研究。
