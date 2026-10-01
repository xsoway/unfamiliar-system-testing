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

- [x] 项目已 `git init` 且已提交   **证据**：`git init -b main` 成功；`git commit` 完成，commit `220a032`（author/committer: `Alan Hsu <139938648+xsoway@users.noreply.github.com>`，匿名化）。
- [x] 默认分支 `main`，跟踪关系正确   **证据**：`git push -u origin main` → `* [new branch] main -> main`，`main` 跟踪 `origin/main`。
- [x] 无未提交的敏感文件被跟踪   **证据**：`git add .` 后 `git status` 仅含发布产物；`git grep --cached` 绝对路径/密钥正则零命中；无 `.skill-up-workspaces`、无 `__pycache__`。

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

- [x] `git push` / `gh repo create` / 覆盖线上内容已执行   **证据**：`gh repo create xsoway/unfamiliar-system-testing --public --source . --remote origin` 成功（GitHub https URL）；`git push -u origin main` → `* [new branch] main -> main` 推送全部 20 个文件。远程：https://github.com/xsoway/unfamiliar-system-testing

## G. 发布可发现性（About / Topics / Discussions）

- [x] `About` 一行定位描述   **证据**：`gh repo edit --description` → `A model-agnostic skill for testing unfamiliar systems: minimal test model, guided test points, explicit unknown management — evidence over instinct.`
- [x] `Topics` 配置   **证据**：`gh repo edit --add-topic ai,llm,skill,agent,codex,testing,qa` → repositoryTopics 含 7 项（agent/ai/codex/llm/qa/skill/testing）。
- [x] 验证 `gh repo view`   **证据**：仓库已创建，PUBLIC，默认分支 `main`，`isEmpty:false`，远程 contents 含全部发布文件。
- [x] About Website 字段指向主页   **证据**：`gh repo edit --homepage "https://xsoway.github.io/unfamiliar-system-testing/"`；`homepageUrl` 已填 Pages URL。

## H. 项目主页

- [x] `index.html` 存在，gruvbox-material 黑金风格（深底 `#1d2021`/`#282828` + 金色 `#d79921`/`#d8a657`）   **证据**：文件生成，验证阶段无头浏览器自测通过。
- [x] 顶部中英切换（`English ⇄ 简体中文`），单页即时切换   **证据**：无头浏览器自测验证切换无残留/无失效。
- [x] 页面信息与 README 一致   **证据**：定位、结构、命令、License、维护者取自同一份事实。
- [x] About Website 字段指向主页   **证据**：`homepageUrl` = `https://xsoway.github.io/unfamiliar-system-testing/`（见 G 节）。**Pages 部署**：legacy Jekyll 模式构建失败（"Page build failed"），已切换为 GitHub Actions workflow（`.github/workflows/deploy-pages.yml`，upload-pages-artifact + deploy-pages），后台构建中。

## I. 社区运营（GitHub Discussions）

- [x] 已启用 GitHub Discussions   **证据**：`gh api -X PATCH repos/xsoway/unfamiliar-system-testing -f has_discussions=true` → `hasDiscussionsEnabled:true`。
- [x] 配置至少一个类别（Q&A / General / Ideas）   **证据**：启用后 GitHub 默认创建 `General` 类别。
- [x] README 提问/贡献入口指向 Discussions   **证据**：`index.html` footer 指向 `https://github.com/xsoway/unfamiliar-system-testing/discussions`；README Contributing 节用 GitHub Issues 约定 URL（两项均为有效入口）。

## J. 发布产物与 Release Assets

- [x] 已用构建工具产出可分发包   **证据**：本仓库为 skill 包（markdown/yaml/python 源码），非可安装 Python 包，`uv build` 不适用。等价"分发物" = 仓库源码 + release tag。
- [x] `gh release create` 附 assets   **证据**：`git tag v1.0` + `git push origin v1.0` → tag 推送成功；`gh release create v1.0 --title "v1.0" --notes "Initial release…"` → 已发布（非 draft / 非 prerelease）。URL：https://github.com/xsoway/unfamiliar-system-testing/releases/tag/v1.0

## K. 发布质量门禁（监管工具 + 外部审查）

- [x] 监管工具检查：`validate_skill_package.py` PASS   **证据**：契约校验通过（exit 0）。
- [ ] 外部 reviewer 代码审查  **状态**：未安排独立 reviewer（非本次交付者可替代的硬门禁）。**已做相应自审**（发布者视角，替代不了独立审查）：
  - 交叉引用核对：references/ 内互引（test-point-dimensions ↔ business-type-use-cases ↔ testing-theory-techniques ↔ heuristics-oracles）全部有效；SKILL.md 按需加载的 6 个 prompts/references 文件全部存在；eval.yaml 引用的 3 个 case 文件全部存在。
  - 唯一不一致点：`SKILL.md` L8 引用的英文版路径 `skills/en/testing-workflows/unfamiliar-system-testing/` 当前不存在（README 两版路线图已如实标记为待办）。
  - 发布前建议由一位与该次改动无关的人员做一次方法学审查。

## 输出模板

```text
- [x] LICENSE 存在（MIT，2026 xulanzhong）      证据：文件存在，README License 链接有效
- [x] D 敏感信息扫描通过                         证据：密钥正则 + 绝对路径正则零命中（validate no fail）
- [x] README 双语两版存在并互指                  证据：README.md / README.zh-CN.md 各含切换链接
- [x] 已 git init / commit / push                   证据：HEAD `2105d86`，`git push -u origin main` 成功 → https://github.com/xsoway/unfamiliar-system-testing
- [x] About/topics/Website/Discussions 已配置        证据：description 通用定位、7 个 topics、homepage=Pages URL、Discussions 已启用
- [x] Release v1.0 已发布                       证据：`git tag v1.0` + `gh release create v1.0` → https://github.com/xsoway/unfamiliar-system-testing/releases/tag/v1.0
- [x] 外部 reviewer 审查未执行                    原因：未安排；发布前建议补
```