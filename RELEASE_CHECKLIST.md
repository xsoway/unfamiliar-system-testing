# Release Checklist — unfamiliar-system-testing

> 逐项核对，每项附证据（命令输出、文件存在、git 状态）。缺证据视为未通过。生成时间：2026-10-01。运行工具：oss-release-prep。

## D. 敏感信息扫描（发布红线，最高优先级）

- [x] 无真实 API key / token / cookie / 私钥 / 密码   **证据**：`grep -rInE 'sk-|ghp_|AKIA|xox|BEGIN PRIVATE KEY|authorization:|api key|bearer' unfamiliar-system-testing/` → 零命中。
- [x] 无绝对本机路径（如 HOME 或用户名宿主目录）        **证据**：绝对路径正则 `/[Uu]sers|/[Hh]ome` 扫描 → 零命中。
- [x] `.env` / 凭据 / 构建产物 / 本地评测目录不被跟踪   **证据**：`.gitignore` 已排除 `.skill-up-workspaces/`、`skill-up-workspaces/`、`__pycache__/`、`.venv/`、`dist/` 等。
- [x] 示例 / 测试数据 / README / 主页无真实用户数据或密钥 **证据**：README 与主页无密钥与绝对路径（验证阶段重扫零命中）。
- [x] 所有 `.md` / `.yaml` / 主页跑密钥正则扫描零命中     **证据**：验证阶段 `validate_skill_package.py` PASS + 全量重扫。

> ⚠️ 注意：本仓库根之上（`test-qa-tool/` 层）存在 `.skill-up-workspaces/` 本地评测产物，含绝对本机宿主路径（HOME / 用户名目录）。仓库根为 `unfamiliar-system-testing/` 时它本就在仓库外；`.gitignore` 防御性保留对应条目。若用户后续把仓库根上移，须先确认该目录排除后再提交。

## A. 版本库状态

- [ ] 项目已 `git init` 且已提交   **原因**：未执行 git（用户选择"只产出文件，不执行 git/发布"）。待用户自行运行：
  ```bash
  cd unfamiliar-system-testing
  git init -b main
  git add .
  git commit -m "Initial release of unfamiliar-system-testing skill"
  ```
- [ ] 默认分支 `main`，跟踪关系正确   **原因**：同上，未 init。
- [ ] 无未提交的敏感文件被跟踪   **原因**：未 init；`.gitignore` 已就位，`git add .` 时会自动排除。

## B. 必备文件

- [x] `README.md`（英文版）    **证据**：文件存在，位于仓库根。
- [x] `README.zh-CN.md`（中文版）切换链接互指   **证据**：两文件顶部互指 `./README.md` / `./README.zh-CN.md`。
- [x] `LICENSE`（MIT，版权 2026 xulanzhong）   **证据**：文件存在，README License 节链接有效。
- [x] `.gitignore`    **证据**：文件存在，覆盖运行产物 / IDE / OS / 密钥 / 本地评测目录。
- [x] `.gitattributes`（`* text=auto eol=lf`）   **证据**：文件存在。

## C. README 质量

- [x] 结构：徽章 + 定位 + 目录 + 快速开始 + 结构/用法 + License + 维护者   **证据**：两版均含完整结构（详尽版）。
- [x] 快速开始命令有依据且可运行   **证据**：`python3 scripts/validate_skill_package.py unfamiliar-system-testing` 已实际运行 → `PASS: unfamiliar-system-testing package contract`（exit 0）。`skill-up` 命令依据 `eval.yaml`（schema v1alpha1）。
- [x] 目录树 / 配置 / 功能与代码一致   **证据**：目录树照 `unfamiliar-system-testing/` 真实结构生成；功能点逐一核对 SKILL.md、prompts、references。
- [x] 双语两版信息一致、覆盖同一组主题   **证据**：两版结构逐节镜像（What/Why/Concepts/Structure/Quick Start/Features/Safety/Validating/FAQ/Roadmap/Contributing/License/Maintainer）。

## E. 可验证性

- [x] 构建 / 测试命令已运行且通过   **证据**：`python3 scripts/validate_skill_package.py unfamiliar-system-testing` → PASS，exit 0。
- [x] 依赖、运行环境、安装步骤有据可查   **证据**：README 声明 Python 3 + 标准库、无第三方依赖。
- [x] 提供校验脚本的运行方法与输出解读   **证据**：README"评测与自检"节给出命令与预期输出 `PASS`。

## F. 发布动作授权（绝不默认执行）

- [x] `git push` / `gh repo create` / 覆盖线上内容未执行   **原因**：未授权。用户选择"只产出文件"。暂不执行，如下命令由用户自行运行（先编辑 About/topics）：
  ```bash
  # 创建远程仓库并推送
  gh repo create <owner>/unfamiliar-system-testing --public --source . --push
  ```

## G. 发布可发现性（About / Topics / Discussions）

- [ ] `About` 一行定位描述   **原因**：需 `gh repo edit --description`，发布动作未授权。建议值：`Codex skill that guides a tester to start testing an unfamiliar system — minimal model, guided test points, unknown management.`
- [ ] `Topics` 配置   **原因**：需 `gh repo edit --add-topic`，未授权。建议：`ai`、`llm`、`skill`、`agent`、`codex`、`testing`、`qa`。
- [ ] 验证 `gh repo view`   **原因**：仓库尚未创建。
- [ ] About Website 字段指向主页   **原因**：仓库未创建；主页 `index.html` 已生成，指向 `https://<owner>.github.io/unfamiliar-system-testing/`（或经 GitHub Pages 部署后填实际地址）。

## H. 项目主页

- [x] `index.html` 存在，gruvbox-material 黑金风格（深底 `#1d2021`/`#282828` + 金色 `#d79921`/`#d8a657`）   **证据**：文件生成，验证阶段无头浏览器自测通过。
- [x] 顶部中英切换（`English ⇄ 简体中文`），单页即时切换   **证据**：无头浏览器自测验证切换无残留/无失效。
- [x] 页面信息与 README 一致   **证据**：定位、结构、命令、License、维护者取自同一份事实。
- [ ] About Website 字段指向主页   **原因**：仓库未创建（见 G 节）。

## I. 社区运营（GitHub Discussions）

- [ ] 已启用 GitHub Discussions   **原因**：仓库未创建（需在仓库 Settings → Features → Discussions 或创建后开启）。
- [ ] 配置至少一个类别（Q&A / General / Ideas）   **原因**：同上。
- [ ] README 提问/贡献入口指向 Discussions   **原因**：仓库未创建时无法填写真实链接；当前 README 用 GitHub Issues 约定 URL。

## J. 发布产物与 Release Assets

- [ ] 已用构建工具产出可分发包   **原因**：本仓库为 skill 包（markdown/yaml/python 源码），非可安装 Python 包，`uv build` 不适用。等价"分发物"即仓库源码本身 + release tag。若需以 pip 分发包形式发布，需另行评估是否包成 `pyproject.toml` 的 Python 包。
- [ ] `gh release create` 附 assets   **原因**：未授权且无 tag。用户自行运行：
  ```bash
  git tag v1.0
  gh release create v1.0 --title "v1.0" --notes "Initial release"
  ```

## K. 发布质量门禁（监管工具 + 外部审查）

- [x] 监管工具检查：`validate_skill_package.py` PASS   **证据**：契约校验通过（exit 0）。
- [ ] 外部 reviewer 代码审查   **原因**：未安排独立 reviewer。发布前建议一次独立于本次改动的人审查，重点核对 skill 方法学正确性、智能体中英文引用完整性（`SKILL.md` 提到的英文版路径当前不存在）。

## 输出模板

```text
- [x] LICENSE 存在（MIT，2026 xulanzhong）      证据：文件存在，README License 链接有效
- [x] D 敏感信息扫描通过                         证据：密钥正则 + 绝对路径正则零命中（validate no fail）
- [x] README 双语两版存在并互指                  证据：README.md / README.zh-CN.md 各含切换链接
- [ ] 未 git init / 未 commit / 未 push           原因：未获授权；命令已给出
- [ ] About/topics/Discussions 未配置             原因：仓库未创建
- [ ] 外部 reviewer 审查未执行                    原因：未安排；发布前建议补
```