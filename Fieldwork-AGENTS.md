# AGENTS.md — 给智能体（AI agent）的协作与推送说明

本仓库是 **Fieldwork（星夜版做题工具）** 的源码，`survey-agent` 包，当前版本见 `pyproject.toml`。
仓库地址（私有，需被邀请才能访问）：
`https://github.com/jonsnow-1/fieldwork-source.git`，默认分支 `main`。

## 0. 硬性规则（先读）

1. **改动前必须先同步远程**（第 2 节），有别人的新提交先合并，**禁止覆盖**。
2. **禁止 `git push --force` / `--force-with-lease` 到 `main`**，禁止改写已推送的历史。
3. **禁止提交任何密钥或个人配置**：API Key、代理账号密码、星夜账密、私钥、`.env`、
   `config/settings.json5`、`certs/*.pfx`、`certs/.pfx-password`、构建产物 `dist/` `build/`、日志、截图。
   这些已在 `.gitignore` 里，**不要用 `git add -f` 强行添加**。提交前用 `git diff --cached --stat` 自查。
4. **不要修改星夜/ancodeinsight 服务端**。本仓库只作为客户端与它对接（读取/调用接口），服务端代码不在此仓库。
5. **不要登录、创建账号、读取或粘贴任何凭据**。推送用的是使用者本机已保存的 GitHub 登录；
   若推送要求登录/返回 403，见第 5 节，**停下来把原因告诉使用者**，不要自己想办法绕过。
6. 不要把与本项目无关的文件（VPS 部署脚本、服务器 IP、构建日志）加入仓库。

## 1. 项目结构

| 路径 | 内容 |
|---|---|
| `apps/` | 桌面界面 `desktop_app.py`、监控台 `monitor_window.py`、主题 `ui_theme.py`、命令行 `cli.py` |
| `src/survey_agent/` | 核心逻辑：`browser` `harness` `llm` `persona` `traps` `intervene` `monitor` `xingye` `geo`、`proxy_build.py`、`updater.py` 等 |
| `prompts/` `config/` `injected_js/` | 提示词、默认配置、注入网页的 JS |
| `tests/` | pytest 测试 |
| `packaging/` `scripts/` | PyInstaller `.spec`、Inno Setup `.iss`、打包/签名脚本（PowerShell） |
| `docs/` `README.md` | 使用文档 |

运行环境：Windows、Python 3.12。安装：`pip install -e ".[dev]"`。测试：`python -m pytest tests -q`。

## 2. 改动前：先读取远程是否有更新

```bash
git fetch origin
git status -sb
git log --oneline HEAD..origin/main
```

- `HEAD..origin/main` 有输出 = 远程有新提交，先 `git pull --rebase origin main`（有冲突就解决冲突，
  解决不了就停下来告诉使用者），然后再开始改。
- 没有输出 = 已是最新，可以开始改。

## 3. 推荐的改动流程（多个智能体/多人同时改时用分支）

```bash
git switch main
git pull --rebase origin main
git switch -c agent/<简短主题>        # 例如 agent/fix-proxy-timeout
# ……修改代码……
python -m pytest tests -q            # 有相关测试就先跑
git add <具体改动的文件>              # 不要无脑 git add -A，先看 git status
git diff --cached --stat             # 自查：没有密钥/日志/构建产物
git commit -m "简短说明改了什么和为什么"
```

## 4. 推送

```bash
git fetch origin
git rebase origin/main               # 推送前再同步一次，避免被拒
git push -u origin HEAD              # 推自己的分支（默认做法）
```

- 默认**推分支，不直接推 `main`**；由使用者或维护者合并到 `main`。
- 只有使用者明确说“直接推 main”时，才 `git push origin main`。
- 推送被拒（`non-fast-forward`）= 远程有新提交：重新 `fetch` + `rebase`，**不要强推**。
- 提交信息写清楚“改了什么、为什么”，一次提交只做一件事。

## 5. 推送失败时怎么办（不要绕过）

| 现象 | 含义 | 该做什么 |
|---|---|---|
| `403` / `Permission denied to <账号>` | 当前保存的 GitHub 账号没有本仓库的写权限 | 告诉使用者：需要在仓库 Settings → Collaborators 给该账号 **Write** 权限并接受邀请，或换用有权限的账号登录 |
| `404` / `Repository not found` | 私有仓库，当前账号未被邀请 | 同上，让使用者邀请该账号 |
| 弹出/等待登录 | 本机没有保存登录 | 让使用者在**自己的终端**执行一次 `git push` 完成浏览器登录；智能体不要输入账号密码或令牌 |
| `non-fast-forward` | 远程有新提交 | `git fetch` → `git rebase origin/main` → 再推，不要 `--force` |

## 6. 一段可以直接发给智能体的指令

> 请先阅读仓库根目录的 `AGENTS.md` 并严格遵守。仓库：`https://github.com/jonsnow-1/fieldwork-source.git`（私有）。
> 开始改动前先 `git fetch origin`，如果 `origin/main` 有新提交先 `git pull --rebase origin main`。
> 在新分支 `agent/<主题>` 上修改，提交前确认没有密钥、日志、构建产物；`git push -u origin HEAD` 推送分支，
> 不要强推、不要直接推 `main`。若推送报 403/404 或要求登录，停下来告诉我原因，不要自己登录或绕过。
