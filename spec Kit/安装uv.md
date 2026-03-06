# uv 是什么
uv 是由 Astral 公司开发的一款 Rust 编写的 Python 包管理器和环境管理器，它的主要目标是提供比现有工具快 10-100 倍的性能，同时保持简单直观的用户体验。

uv 可以替代 pip、virtualenv、pip-tools 等工具，提供依赖管理、虚拟环境创建、Python 版本管理等一站式服务。

# 安装 uv
在 macOS 上安装
推荐使用 Homebrew 安装：

brew install uv
或者使用官方安装脚本：

curl -LsSf https://astral.sh/uv/install.sh | sh
在 Linux 上安装
curl -LsSf https://astral.sh/uv/install.sh | sh
在 Windows 上安装
使用 Winget：

winget install uv
或者使用官方安装脚本：

irm https://astral.sh/uv/install.ps1 | iex
安装完成后，验证安装是否成功：

uv --version
输出内容类似如下，表明安装成功：

uv 0.8.14 (Homebrew 2025-08-28)

