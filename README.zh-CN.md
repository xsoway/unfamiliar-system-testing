<p align="center">
  <img src="https://img.shields.io/badge/version-1.0-blue" alt="版本 1.0">
  <img src="https://img.shields.io/badge/license-MIT-green" alt="License MIT">
  <img src="https://img.shields.io/badge/status-stable-success" alt="状态 稳定">
  <img src="https://img.shields.io/badge/language-EN%20%7C%20%E4%B8%AD-orange" alt="双语文档 EN + 中文">
  <img src="https://img.shields.io/badge/runtime-Python%203%2C%20no%20deps-lightgrey" alt="运行环境 Python 3，无第三方依赖">
</p>

<h1 align="center">unfamiliar-system-testing</h1>

<p align="center">
  <strong>还不懂系统？先建最小测试模型、生成测试方向、显式管理未知——再开始测</strong>
</p>

<p align="center">
  <a href="./README.md">English</a>
</p>

<p align="center">
  <b>上手引导 · 暴露未知 · 生成测试点 · 估算风险</b>
</p>

---

## 目录

- [是什么](#是什么)
- [为什么需要](#为什么需要)
- [核心概念](#核心概念)
- [目录结构](#目录结构)
- [快速开始](#快速开始)
- [功能与用法](#功能与用法)
- [安全边界与设计原则](#安全边界与设计原则)
- [评测与自检](#评测与自检)
- [FAQ（常见问题）](#faq常见问题)
- [路线图](#路线图)
- [贡献指南](#贡献指南)
- [License](#license)
- [维护者](#维护者)

## 是什么

`unfamiliar-system-testing` 是一个**模型无关的通用 skill**——自包含的技能包（`SKILL.md` + prompts + references + evals + 校验脚本），Codex、Claude 及其他任意模型宿主均可加载使用。它引导测试人员在**还不理解系统**的情况下开始测试。它不抛出通用清单，而是先建立**最小测试模型**（8 问），把信息分成**事实 / 假设 / 未知**，按维度生成该考虑的测试点与方向，并提供跨业务类型兜底层和测试理论方法——让测试人员知道**该考虑什么、往哪测、下一步找谁确认**。

它是「**上手引导 + 测试点启发**」的陪伴工具，不是执行引擎，也不是技能路由器。当测试人员明确了要做哪类测试（API 契约、性能、安全、UI 自动化）时，`discover-testing` 会路由到更专的 `testing-types/*` 技能；本 skill 提供的是进入那些技能之前的「理解 + 方向生成」层。

## 为什么需要

在陌生系统上开始测试，通常要么对着空白发懵，要么贴来一大堆无关紧要的通用用例。两者源于同一个根因——**对一个还没验证的系统做推理**：

| 直觉（反模式） | 本 skill 的做法 |
|---|---|
| "先把整个架构学完再测" | 先建*最小*模型，边测边学 |
| 照搬通用测试点清单 | 结合当前系统裁剪，不适用就标 `不适用` |
| 把口头解释 / 旧文档当事实 | 标为"假设"，设计证据去验证 |
| 信息不足仍假装确定 | 显式列出未知，写明找谁确认 |
| "我觉得错了"却没有依据 | 先定义预言机（凭什么判断对错） |
| 把判断权外包给 AI | AI 只当挑战者/复核者，不当结论来源 |

## 核心概念

- **最小测试模型**：只回答当前任务需要的 8 问（目的、用户、输入输出、业务规则、状态、依赖、失败行为、最大损失点）。不必第一天就懂所有模块。
- **未知管理**："我还没懂"是工作状态，不是结论。每个缺口记录三件事：缺哪块信息、谁最可能知道、可用什么观察/实验验证。**显式写出未知，比用一个未经验证的答案填上去更安全**。
- **事实 vs 假设 vs 未知**：口头说明和旧文档在转成可观察、可复现的证据之前都是假设。
- **预言机优先**：每个测试方向都要有判断结果对错的标准。没有预言机的断言就是"我觉得它错了"，不可信。
- **happy path + failure path**：不仅测正确操作路径，还要覆盖输入 / 权限 / 依赖 / 网络 / 状态这些失败路径。
- **风险优先级**：用影响 × 可能性的风险矩阵量化"最大损失处"（第 8 问），决定先测谁。

## 目录结构

```text
unfamiliar-system-testing/
├── SKILL.md                        # 激活入口（frontmatter + 规则）
├── prompts/unfamiliar-system-testing.md   # 完整执行规范（8 问模型、输出结构）
├── agents/openai.yaml              # 发现元数据 + 隐式调用策略
├── references/
│   ├── test-point-dimensions.md    # 10 类测试点维度，结合当前系统裁剪
│   ├── business-type-use-cases.md  # 跨业务类型兜底（电商/支付/CRM/搜索/IAM/Agent/...）
│   ├── testing-theory-techniques.md# 等价类/边界值/决策表/状态迁移/探索式/基于风险/...
│   ├── ai-usage.md                 # AI 当复核者/挑战者，不当结论来源
│   └── heuristics-oracles.md       # 测试预言机、SFDPOT、FEW HICCUPPS、风险矩阵、charter 模板
├── evals/
│   ├── eval.yaml                   # 评测 harness（schema v1alpha1，claude_code 引擎）
│   └── cases/                      # basic-success / edge-incomplete-input / edge-narrow-unknown
└── scripts/validate_skill_package.py  # 契约校验 + 密钥/绝对路径扫描
```

| 路径 | 职责 |
|---|---|
| `SKILL.md` | 激活入口：何时使用、定位、核心约束、执行流程、按需加载、交付前自检、常见误区。 |
| `prompts/unfamiliar-system-testing.md` | 完整执行规范：8 问最小模型、未知管理、方向生成 Step 1–6、预言机优先、输出结构、最低覆盖清单。 |
| `references/test-point-dimensions.md` | 10 个维度分类（功能、输入校验、状态、权限、依赖、网络、性能、兼容性、安全、发布），含"裁剪或标注不适用"指引。 |
| `references/business-type-use-cases.md` | 分业务类型兜底测试点（A–H），⭐ = 最容易翻车的高风险点，命中才懒加载。 |
| `references/testing-theory-techniques.md` | 12 种经典技术，附"陌生系统首轮先选谁"的对照表。 |
| `references/ai-usage.md` | 如何在有基线的前提下把 AI 当挑战者/复核者；AI 不知道什么。 |
| `references/heuristics-oracles.md` | 预测言机问题、SFDPOT / FEW HICCUPPS 启发式、2×2 风险矩阵、探索式 charter 模板、API 黑盒契约检查。 |
| `agents/openai.yaml` | 发现元数据（display name、描述、隐式调用策略）。 |
| `evals/` | 规则化评测：一个成功用例 + 两个边界用例（输入不完整、窄目标未知）。 |
| `evals/cases/*.yaml` | 每个用例的 prompt、`must_not_contain`、规则化 judge。 |
| `scripts/validate_skill_package.py` | 校验包契约（必备文件、标题、懒加载引用、用例接线），并扫描凭据特征与绝对本机路径。 |

## 快速开始

**环境要求：** Python 3（仅标准库，无第三方依赖）。

**校验包契约**

```bash
python3 scripts/validate_skill_package.py unfamiliar-system-testing
# → PASS: unfamiliar-system-testing package contract
```

**运行评测用例**（用 skill-up 针对你的引擎跑）

```bash
skill-up run unfamiliar-system-testing   # 或：skill-up run evals/eval.yaml
```

评测 harness 含三个规则化用例——`basic-success`、`edge-incomplete-input`、`edge-narrow-unknown`——锁定"最小模型 + 未知管理"的预期行为。

## 功能与用法

- **8 问最小模型**：目的、用户、输入输出、规则、状态、依赖、失败行为、最大损失——只建当前任务所需；缺失项标 `Unknown`。
- **事实 / 假设 / 未知三分**：口头说明和旧文档变成带验证手段的假设；信息缺口显式标注，给出负责人与验证实验。
- **failure path 覆盖**：从一条 happy path 扩展出输入 / 权限 / 依赖 / 网络 / 状态各方向的测试点。
- **跨业务类型兜底**：电商、支付/账务、CRM、搜索、IAM、AI/LLM/Agent、数据集成、消息——命中才懒加载，⭐ 高风险点优先。
- **测试理论工具箱**：等价类、边界值、决策表、状态迁移、探索式（charter）、基于风险、成对、蜕变性、谓词/不变式。
- **预言机优先纪律**：每个方向先定义"用什么判断对错"；用影响 × 可能性矩阵给风险打分。
- **AI 当挑战者而非来源**：让 AI 挑战你的测试计划而不是替你产出结论。
- **可校验**：契约校验脚本 + 三个规则化评测用例。
- **零敏感信息**：校验脚本扫描凭据特征与绝对本机路径；示例从不携带真实密钥。

## 安全边界与设计原则

- **绝不虚构业务规则**。不理解系统时，禁止把未经验证的规则写成结论——未知要显式写出来，而不是掩盖。
- **先模型后点**。不给测试点时强行圆一个模型，或只有模型没有方向，都算不合格。
- **清单是围栏，不是剧本**。维度清单和业务类型兜底是"结合当前系统裁剪"的触发点，绝不整段照搬。
- **AI 是挑战者**。AI 生成的内容未经人工判断，一律不当业务规则或结论使用。
- **不虚构内容**。输出结构、优先级、预言机、下一步都来自当前系统和测试人员的证据，不沿用外部约定。

## 评测与自检

- 包契约 + 密钥/绝对路径扫描：

  ```bash
  python3 scripts/validate_skill_package.py unfamiliar-system-testing
  # 期望输出：PASS
  ```

- 评测 harness（规则化，三个用例）：

  ```bash
  skill-up run unfamiliar-system-testing
  ```

## FAQ（常见问题）

**这是测试执行框架吗？**
不是。它产出最小模型、测试方向、未知清单和下一步。当需要特定测试类型时，`discover-testing` 路由到专门的 `testing-types/*` 技能。

**用户几乎不给信息怎么办？**
"未知管理"纪律接管：输出带缺口的最小模型，写明每个缺口找谁确认、用什么实验验证——绝不为显得自信而编造业务规则。

**覆盖所有业务类型吗？**
兜底层覆盖 A–H（电商、支付、CRM、搜索、IAM、AI/Agent、数据集成、消息）。类型命中才懒加载；⭐ 高风险点是入手的最安全位置。

**为什么每个方向都要预言机？**
在陌生系统上，"看起来不对"而没有标准是不可靠的。先定义预言机（一致性、文档、领域模型、历史等），测试方向才有意义。

**能让 AI 直接生成测试计划吗？**
在你还没有基线时，本 skill 明确反对。它把 AI 定位为挑战者/复核者：你先写第一版计划，AI 挑战，你逐条判断。

## 路线图

- [x] 8 问最小测试模型 + 事实/假设/未知三分
- [x] 10 维测试点清单，含"裁剪或标不适用"指引
- [x] A–H 跨业务类型兜底层，⭐ 高风险点标记
- [x] 测试理论工具箱与选择对照表
- [x] 预言机优先纪律 + 影响×可能性风险矩阵
- [x] AI 当挑战者的用法与提示词模板
- [x] 包契约校验脚本 + 三个规则化评测用例
- [x] 双语 README（中英文）+ gruvbox-material 主页
- [ ] skill 正文英文版（当前在 `skills/en/testing-workflows/unfamiliar-system-testing/`）
- [ ] 更多业务类型兜底（如金融科技、DevOps）与逐类型评测用例

## 贡献指南

欢迎提交 Pull Request。提交前请让包契约保持绿色：

```bash
python3 scripts/validate_skill_package.py unfamiliar-system-testing
```

遵循本 skill 自己的证据规则：说明改了什么、如何验证、有何风险。任何文件不得包含真实密钥、个人数据或绝对本机路径。

## License

[MIT](./LICENSE)

## 维护者

[xulanzhong](https://github.com/xulanzhong) · <xulanzhong521@gmail.com>