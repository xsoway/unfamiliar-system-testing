---
name: unfamiliar-system-testing
description: Use this skill when a tester must start testing a system they do not yet understand; guides onboarding via a minimal test model, prompts what test points and directions to consider, and provides cross-business-type checklists and testing-theory techniques; triggers include 陌生系统测试、不懂的系统怎么测、测试方向、测试点探索、onboarding testing and unfamiliar system testing.
---

# 陌生系统测试引导（中文版）

**英文版：** 见 `skills/en/testing-workflows/unfamiliar-system-testing/`。

## 何时使用

- 拿到一个还没理解的新系统 / 新模块 / 紧急工单，需要马上开始测试。
- 需求简短、懂业务的人没空、代码/接口/历史缺陷还没看，但需要给出测试方向。
- 想让测试人员被引导着逐项考虑「该测哪些测试点、往哪些方向测」。
- 需要一份能给不同业务类型（电商/支付/CRM/搜索/权限/Agent 等）兜底的测试点参考和测试理论方法。

## 本 skill 的定位（重要）

本 skill 是「**上手引导 + 测试点启发**」的陪伴工具，不是执行型技能，也**不是技能路由器**：

- 它帮助你在**还不懂系统**时，建立最小测试模型、把未知拆成可验证的问题、生成可考虑的测试点与方向。
- 如果你已经明确了要做哪一类测试（如 API 契约、性能、安全、UI 自动化），`discover-testing` 会把你路由到更专的 `testing-types/*` 技能；本 skill 提供的是进入那些技能之前的「理解 + 方向生成」层。
- 最终是否采纳某个测试点、方向或业务规则，由测试人员判断；本 skill 禁止把未经验证的规则写死成结论。

## 核心约束

- 先建立最小测试模型，再谈测试点；不理解系统时不虚构业务规则。
- 区分「已确认事实」与「假设」：凡是口头解释或旧文档，必须能转成可观察、可复现的证据。
- 始终检查 happy path（正常路径）之外的 failure path（输入/权限/依赖/网络/状态）。
- 信息缺口是「还没懂」，不是「这不重要」：Unknown 要显式写出，而不是用未经验证的答案填补。
- 测试点清单只是启发，必须结合当前系统裁剪；不同业务类型先查对应兜底层，再在此基础上扩展。

## 执行流程

1. 读入用户提供的需求 / 变更 / 系统背景，识别阶段（新系统上手 / 紧急工单 / 变更回归）。
2. 阅读并遵循 `prompts/unfamiliar-system-testing.md`：按 8 问建立最小测试模型，列出未知与信息负责人。
3. 按测试点维度清单（`references/test-point-dimensions.md`）逐项生成当前可考虑的测试点与方向。
4. 命中业务类型时，先读对应兜底层（`references/business-type-use-cases.md`）；懒加载，不全部输出。
5. 涉及测试理论方法选择时，读 `references/testing-theory-techniques.md` 选用合适技术（边界值/等价类/决策表/状态迁移/探索式/基于风险等）。
6. 需要用 AI 复核或挑战测试计划时，遵循 `references/ai-usage.md`：AI 当复核者/挑战者，不当结论来源。
7. 用 round 输出：最小模型 → 测试点/方向 → 未知清单与下一步。

## 按需加载

- 产出前必须阅读并遵循 `prompts/unfamiliar-system-testing.md`（最低覆盖、输出结构、质量要求）。
- 需要测试点维度清单时：读 `references/test-point-dimensions.md`。
- 需要分业务类型用例参考时：读 `references/business-type-use-cases.md`。
- 需要测试理论与技术方法时：读 `references/testing-theory-techniques.md`。
- 需要 AI 辅助用法时：读 `references/ai-usage.md`。
- 需要判断「怎么知道结果是错的」、启发式模型或风险矩阵时：读 `references/heuristics-oracles.md`。
- 需要评测/回归本 skill 时：使用 `evals/`，并用 skill-up 校验与运行。
- 需要结构/安全自检时：运行 `python3 scripts/validate_skill_package.py <skill-dir>`。

## 交付前自检

- [ ] 已遵循主提示词的输出结构
- [ ] 已建立最小测试模型，且区分事实与假设
- [ ] 已覆盖 happy path 与 failure path（输入/权限/依赖/网络/状态），或标明为何省略
- [ ] 测试点来自当前系统而非照搬清单；未编造业务规则
- [ ] 所有 Unknown / 信息缺口已显式列出，并给出可询问的信息负责人
- [ ] 高风险项有明确优先级
- [ ] AI 建议经过人工判断，未把生成结果直接当结论

## 常见误区

- 一上来就想把系统架构、数据模型、全部历史都学完再测（本 skill 主张先建最小模型、边测边补）。
- 把测试点清单整段照搬，不做系统裁剪。
- 用「我认为应该如此」代替「系统已证明如此」。
- 信息不足时假装已经能确定且可落地。
- 用 AI 直接生成大量用例而不检查前提（等于把「你不知道自己不知道什么」的风险交给 AI）。