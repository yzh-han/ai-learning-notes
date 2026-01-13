
## 快速开始（uv / 可长期维护项目）

> 目标：从 0 开始创建一个可维护、可复现（可在 CI 上稳定安装）的 Python 项目。
> 约定：依赖声明在 `pyproject.toml`，精确版本锁在 `uv.lock`，环境隔离在 `./.venv`，运行统一用 `uv run ...`。

---

### 0) 前置：安装 uv（macOS）
```bash
brew install uv
uv --version
````

---

### 1) 初始化（项目骨架 + Python 版本 + venv）

```bash
uv init myproj && cd myproj --python 3.12
# 创建初始目录结构（包含 pyproject.toml、README、示例 main.py 等）
# 或者在项目文件中 uv init
# --package：创建可安装的包（有包目录、版本、构建后端配置等）默认。
# --no-package：偏脚本/应用（可能不创建包目录，或不配置 build 相关字段）。

uv python install 3.12
# 可选：让 uv 自己下载并管理 Python 3.12（managed Python）
# 如果你已经 `brew install python@3.12`，通常也可以跳过这步：
# uv 能发现系统 Python（system Python），只要它满足项目要求就能用。

uv python pin 3.12
# 固定项目默认 Python 版本：写入/更新 .python-version
# 作用：团队协作一致；uv venv / uv run 会参考它选择解释器

uv venv
# 创建项目虚拟环境 ./ .venv（默认按 .python-version 选择 Python）
# 说明：即使不手动执行，首次运行 uv run / uv sync 时也会自动创建 .venv
```

> ⚠️ Python 版本升级提示（例如 3.12 -> 3.14）
>
> ```bash
> uv python pin 3.14 # 会修改 .python-version 文件中的python 版本到 3.14, 这是默认工作环境python版本 
> ```
>
> 同时请检查 `pyproject.toml` 里的 `requires-python`：
>
> * 如果写死了 `==3.12`，需要改成包含 3.14，例如 `>=3.12` 或 `>=3.14`, `~=3.14` 在3.14版本
> * `.python-version` 是“默认用哪个 Python”，`requires-python` 是“项目声明支持/要求的范围”

---

### 2) 依赖（开发依赖 / 锁大版本或小版本 / lock & sync）

```bash
uv add pytorch # 运行时
uv add --dev ruff pytest
# 添加开发依赖：写入 pyproject.toml（dev 组）并通常会更新 uv.lock + 同步环境

uv sync --no-dev # 部署环境（只装运行时）
uv sync # 运行时 + 默认 groups（通常包含 dev，取决于你的配置）
# 同步环境：把 .venv 安装成 uv.lock 指定的精确版本集合
# 默认行为：如果发现 uv.lock 与 pyproject.toml 不一致，会先 re-lock（等价需要时执行 uv lock）
```

#### 分组

`uv add --dev` 和 `uv add --group dev` 将添加到依赖分组

```bash
[dependency-groups]
dev = ["pytest", "ruff"]
docs = ["mkdocs", "mkdocs-material"]
lint = ["ruff"]
```

控制安装哪些组:

```
uv sync                  # 默认装（通常包含 dev 组）
uv sync --no-dev         # 不装 dev 组（只装运行时依赖）
uv sync --only-group docs  # 只装 docs 组
uv sync --all-groups     # 装所有组
uv sync --no-default-groups  # 不装默认组（只装 project.dependencies 等）
```


#### 2.1（可选但推荐）锁定依赖范围：锁大版本/小版本

> 默认 `uv add pkg` 常见会写成 `pkg>=x.y.z`（只有下界）。
> 如果你想在依赖声明里就限制升级范围（避免未来大版本 breaking changes），用 `--bounds`：

```bash
uv add --bounds major fastapi
# 写成类似：fastapi>=x.y.z,<下一主版本（锁大版本）

uv add --bounds minor pydantic
# 写成类似：pydantic>=x.y.z,<下一小版本（锁小版本）

uv add --bounds exact numpy
# 写成：numpy==x.y.z（锁死精确版本，不建议普遍使用）
```

#### 2.2（推荐用于 CI）严格按锁文件安装

```bash
uv sync --frozen
# CI/Docker 推荐：完全按 uv.lock 安装，不允许更新 lock
# 目的：可复现构建（防止 CI “顺手改了 lock”）
```

#### 2.3（可选）只检查 lock 是否与项目一致（不改动）

```bash
uv sync --locked
# 如果发现需要更新 uv.lock：直接失败（要求你先显式 uv lock）
```

#### 2.4（可选）显式生成/更新 lock

```bash
uv lock
# 只做“锁定解析”：更新 uv.lock（不直接管安装）
```

---

### 3) 运行 / 测试 / 代码质量（统一用 uv run）

```bash
uv run python main.py
# 在项目 .venv 中运行命令（不必手动 activate）
# 如果 .venv 不存在，uv 会自动创建并按 lock 同步后再运行

uv run pytest -q
# 跑测试

uv run ruff format .
# 格式化

uv run ruff check .
# 静态检查（可加 --fix 自动修一部分）
```

`uv run` 可能会修改lock文件(锁文件与项目配置不匹配时).

```bash
uv run --locked ... # 禁止自动 lock；如果 lock 需要更新就直接报错，让你手动 uv lock。

uv run --frozen ... # 更强硬，按现有 lock 原样使用（常用于 CI/构建），避免任何写入。
```

---

### 4) 最低版本解析（库项目/兼容性验证常用，可选）

> 用途：验证你写的“最低版本下界”是否真的能跑（常见于库 / SDK）。
> 注意：这通常是 CI 里跑的 job，不一定要把这个解析结果提交为主分支的 uv.lock。

```bash
uv lock --resolution lowest-direct
uv sync --frozen
uv run pytest
# lowest-direct：只把“直接依赖”尽量选到下界，间接依赖仍尽量新
# lowest：直接+间接都尽量最低（更严格）
```

---

### 5) Git 提交建议

* ✅ 提交：`pyproject.toml`、`uv.lock`、源码、测试、README
* ❌ 不提交：`.venv/`

新人拉仓库后：

```bash
uv sync --frozen
uv run pytest
```

```

如果你愿意，我还能给你加两个常用段落（也都是“注释说明版”）：
1) **推荐目录结构**（`src/` + `tests/`）以及为什么要用 `src/`  
2) 一份更完整的 `pyproject.toml`（ruff/pytest 配置 + 常用脚本命令），让团队命令更统一（例如 `uv run lint` 之类）

你只要说你项目类型是：**应用/服务**、**CLI**、还是**库**。
::contentReference[oaicite:0]{index=0}
```
