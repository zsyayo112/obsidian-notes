---
tags:
  - 工具知识
  - repopilot
created: 2026-07-13
---

# 🛠️ Makefile 是命令别名本

> [!note] 一句话
> **Makefile 是一个项目的"命令别名本"——纯文本，里面写好一堆快捷方式。敲 `make test` 它就去执行 `test:` 目标下面那行命令。它不是框架、不属于任何语言。**

## 长什么样
```makefile
test:
	pytest -q --cov
build:
	python -m build
```
敲 `make test` → 执行 `pytest -q --cov`。

## 类比你可能见过的
Node 的 `package.json` 里的 `"scripts": {"test": ...}`——**Makefile 的 `test:` 就是同一个东西**，只是更老、语言中立。Python/Go/Rust/C 项目都能用。

## 在 RepoPilot 里的意义（为什么 detector 排最后）
一个 Python 项目常常**又有 pyproject.toml、又有 Makefile**，而 Makefile 的 `test:` 往往只是**包了一层** `pytest`。所以：
- 认出 "make" 比认出 "python" **信息量更少**（丢了精确的语言身份）。
- **特殊优先于通用**：语言专属标志（Cargo.toml/go.mod/pyproject.toml）必须比 Makefile 兜底更优先。所以 `_detect_make` 在 [[框架无关|detector 注册表]] 里排**最后**。

关联：[[框架无关]] · [[RepoPilot MOC]]
