## 腾讯云 Cloud Studio
链接： https://ide.cloud.tencent.com/dashboard/workspace
微信登陆

## 创建工作空间
###  创建Jupyter Notebook环境
![alt text](image.png)
空间名称：Agent101
代码来源：空
开发环境：Jupyter Notebook
规格配置：旗舰版 8核16G内存/64G存储
![alt text](image-1.png)

创建完成之后，自动进入工作空间，页面会显示一个VS Code编辑器。
打开终端，后续命令都在终端里执行：
![alt text](image-2.png)

### 创建项目Conda环境
腾讯云已经默认安装了 Miniconda，并且默认使用腾讯云镜像，所以我们只需要创建项目专用环境即可：
```
# 创建名为 agent101 的环境，指定 Python 3.10.18
conda create -n agent101 python=3.10.18 -y

# 激活环境
conda activate agent101

# 验证 Python 版本
python --version
# 输出：Python 3.10.18

# 验证 pip 版本
pip --version
```

### JupyterLab 安装与配置
安装 JupyterLab

确保已激活 agent101 环境：
```
# 激活环境
conda activate agent101

# 使用 conda 安装 JupyterLab
conda install -c conda-forge jupyterlab -y

# 验证安装
jupyter lab --version
# 输出示例：4.0.9
```
### 生成配置文件
```
# 生成 JupyterLab 配置文件
jupyter lab --generate-config

# 输出示例：
# Writing default config to: /root/.jupyter/jupyter_lab_config.py
```
配置文件位置：~/.jupyter/jupyter_lab_config.py

### 配置 JupyterLab
编辑配置文件：
vim ~/.jupyter/jupyter_lab_config.py
不保存退出：ESC → :q → 回车
保存退出：ESC → :wq → 回车
强制不保存退出：ESC → :q! → 回车

添加或修改以下配置：
```
# 允许 root 用户启动（如果使用 root 用户）
c.ServerApp.allow_root = True

# 设置工作目录
c.ServerApp.root_dir = '/workspace/Agent101/code'

# 修改默认端口
c.ServerApp.port = 8000

# 设置可访问的 IP（0.0.0.0 表示允许所有 IP 访问）
c.ServerApp.ip = '0.0.0.0'

# 设置访问令牌（token）作为密码
# 方式1：自定义 token
c.ServerApp.token = 'fly123'

# 方式2：禁用 token（不推荐，仅本地使用）
# c.ServerApp.token = ''
# c.ServerApp.password = ''

# 禁用浏览器自动打开
c.ServerApp.open_browser = False

# 启用扩展
c.ServerApp.jpserver_extensions = {}
```
**重要配置说明**：

| 配置项            | 说明              | 推荐值                        |
| -------------- | --------------- | -------------------------- |
| `allow_root`   | 是否允许 root 用户运行  | True（如使用 root）             |
| `root_dir`     | JupyterLab 工作目录 | `/workspace/Agent101/code` |
| `port`         | 访问端口            | `8000`                     |
| `ip`           | 监听 IP           | `0.0.0.0`（允许远程访问）          |
| `token`        | 访问令牌            | 自定义强密码                     |
| `open_browser` | 自动打开浏览器         | False（服务器模式）               |

**说明：**

`root_dir` 指定的是 `/workspace/Agent101/code`，因为腾讯云默认的工作空间是 `/workspace`，所以项目代码也放在这里，可以在VS Code文件列表直接看到，方便操作。

## 安装 ipykernel（重要）
为了在 JupyterLab 中使用 conda 环境，必须安装 ipykernel：
```
# 确保在 agent101 环境中
conda activate agent101

# 安装 ipykernel
pip install ipykernel

# 将环境注册为 Jupyter 内核
python -m ipykernel install --user --name=agent101 --display-name="Python (agent101)"

# 验证内核安装
jupyter kernelspec list
```
**输出示例**：

```
Available kernels:
  agent101    /root/.local/share/jupyter/kernels/agent101
  python3     /root/miniconda3/envs/agent101/share/jupyter/kernels/python3
```
**管理内核**：

```
# 查看已安装的内核
jupyter kernelspec list

# 删除内核
jupyter kernelspec remove agent101

# 重新安装内核
python -m ipykernel install --user --name=agent101 --display-name="Python (agent101)"
```
## 测试 JupyterLab
```
# 启动 JupyterLab（前台运行）
jupyter lab --config=/root/.jupyter/jupyter_lab_config.py

# 输出示例：
# [I 2025-01-01 10:00:00.000 ServerApp] Jupyter Server 2.x.x is running at:
# [I 2025-01-01 10:00:00.000 ServerApp] http://0.0.0.0:8000/lab?token=fly123
```

点击`打开浏览器`，会在新页面访问 JupyterLab：

输入 token：`fly123`

**停止 JupyterLab**：按 `Ctrl+C` 两次

## 代码下载
创建代码目录
```bash
# 创建代码根目录
mkdir -p Agent101/code

# 进入代码目录
cd Agent101/code
```
## 克隆项目代码

```bash
# 克隆 Agent_In_Action 项目
git clone https://github.com/FlyAIBox/Agent_In_Action.git

# 进入项目目录
cd Agent_In_Action

# 查看项目结构
tree -L 2

# 或使用 ls
ls -lah
```
**项目结构**：

```
Agent_In_Action/
├── 01-agent-tool-mcp/          # 项目1-2: 智能体基础与 MCP 集成
├── 02-agent-multi-role/        # 项目3: 深度研究助手
├── 03-agent-build-docker-deploy/  # 项目4: 旅行规划系统
├── 04-agent-evaluation/        # 项目5: 监控与评估
├── 05-agent-model-finetuning/  # 项目6: 模型微调
├── docs/                       # 文档目录
└── README.md                   # 项目说明
```

## 安装项目依赖
```bash
# 项目1-2：智能体基础与 MCP
cd 01-agent-tool-mcp/mcp-demo
pip install -r requirements.txt

# 项目3：深度研究助手
cd ../../02-agent-multi-role/deepresearch/deployment
pip install -r requirements.txt

# 项目4：旅行规划系统
cd ../../../03-agent-build-docker-deploy
pip install -r backend/requirements.txt
pip install -r frontend/requirements.txt

# 项目5：评估监控
pip install langfuse langchain langgraph

# 返回项目根目录
cd /workspace/Agent101/code/Agent_In_Action
```
## 常用依赖包

项目所需的主要依赖包：

```bash
# 核心依赖
pip install langchain langgraph langchain-openai langchain-community
pip install openai anthropic
pip install fastapi uvicorn streamlit
pip install python-dotenv

# 工具包
pip install tavily-python serpapi
pip install requests httpx

# 监控评估
pip install langfuse langsmith

# 数据处理
pip install pandas numpy
```

```bash
# 验证 Python 包
python -c "import langchain; print(f'LangChain: {langchain.__version__}')"
python -c "import langgraph; print('LangGraph installed')"
python -c "import openai; print(f'OpenAI: {openai.__version__}')"

# 查看已安装的包
pip list | grep -E "langchain|openai|fastapi"
```

结果如下：
![alt text](image-3.png)

至此，基本的配置已经完成，注意代码根目录是`/workspace/Agent101/code/Agent_In_Action`。

## 环境变量配置

有两种配置方式：

1. **全局配置**：添加到 `~/.bashrc`（推荐，所有终端会话生效）
2. **项目配置**：在项目目录创建 `.env` 文件（项目独立配置）

### 全局环境变量配置

编辑 `~/.bashrc` 文件：

```bash
vim ~/.bashrc
```

在文件末尾添加以下内容：

```bash
# ==========================================================
# AI Agent 101 环境变量配置
# ==========================================================
# --- OpenAI 配置 ---
export OPENAI_BASE_URL="https://api.openai.com/v1"
export OPENAI_API_KEY="sk-your_openai_api_key_here"
export MODEL_NAME="gpt-4o"

# 或使用 DeepSeek 作为 OpenAI 兼容接口
# export OPENAI_BASE_URL="https://api.deepseek.com"
# export OPENAI_API_KEY="sk-your_deepseek_api_key_here"
# export MODEL_NAME="deepseek-chat"

# --- DeepSeek 配置 ---
export DEEPSEEK_BASE_URL="https://api.deepseek.com"
export DEEPSEEK_API_KEY="sk-your_deepseek_api_key_here"

# --- Langfuse 配置（监控评估）---
export LANGFUSE_HOST="https://cloud.langfuse.com"
export LANGFUSE_PUBLIC_KEY="pk-lf-your_public_key_here"
export LANGFUSE_SECRET_KEY="sk-lf-your_secret_key_here"

# --- 搜索工具配置 ---
export SERPAPI_API_KEY="your_serpapi_key_here"
export TAVILY_API_KEY="tvly-your_tavily_key_here"

# --- LangSmith 配置（可选）---
export LANGSMITH_API_KEY="lsv2_your_langsmith_key_here"
export LANGSMITH_PROJECT="aiagent101"
export LANGSMITH_TRACING_V2="true"

# --- 和风天气 API 配置 ---
# 参考：https://dev.qweather.com/
export QWEATHER_API_BASE="your_qweather_base_url"
export QWEATHER_API_KEY="your_qweather_key_here"

# ==========================================================
# 代理配置（如果使用 v2raya）
# ==========================================================
# export http_proxy="http://127.0.0.1:20171"
# export https_proxy="http://127.0.0.1:20171"
# export no_proxy="localhost,127.0.0.1,192.168.0.0/16,10.0.0.0/8"
应用配置：

```bash
source ~/.bashrc
```

### 7.3 API 密钥获取指南

| API 服务 | 获取地址 | 用途 | 费用 | 备注 |
|---------|---------|------|------|------|
| **OpenAI** | [platform.openai.com](https://platform.openai.com) | GPT-4/GPT-4o 模型 | 按量付费 | 需要信用卡 |
| **OpenAI等国外大模型国内代理** | https://api.apiyi.com/register/?aff_code=we80 | GPT/Claude等 模型 | 按量付费 | 新用户注册送0.1美金，注册成功后在以下表格中填写你的账号，平台会再赠送2美金（5个工作日到账）<br/><br/>---- 【腾讯文档】API易账号收集 https://docs.qq.com/form/page/DQm1qb1VBQU9wR2xq<br/> |
| **DeepSeek** | [platform.deepseek.com](https://platform.deepseek.com) | DeepSeek-Chat/V3 | 价格实惠 | 支持国内支付 |
| **和风天气** | [dev.qweather.com](https://dev.qweather.com) | 天气查询工具 | 免费额度 1000次/天 | 需要实名认证 |
| **Tavily** | [tavily.com](https://tavily.com) | AI 搜索 API | 免费 1000次/月 | 邮箱注册即可 |
| **SerpAPI** | [serpapi.com](https://serpapi.com) | Google 搜索 API | 免费 100次/月 | 需要信用卡验证 |
| **Langfuse** | [cloud.langfuse.com](https://cloud.langfuse.com) | LLM 监控评估 | 免费版 5万 events | GitHub 登录 |
| **LangSmith** | [smith.langchain.com](https://smith.langchain.com) | 调试追踪 | 免费 5千 traces | GitHub 登录 |

### 7.4 项目级 .env 配置（可选）

如果需要项目独立配置，可以在项目目录创建 `.env` 文件：

```bash
# 进入项目目录
cd /Agent101/code/Agent_In_Action/03-agent-build-docker-deploy/backend

# 创建 .env 文件
vim .env
```

`.env` 文件内容：

```bash
# OpenAI 配置
OPENAI_API_KEY=sk-your_key_here
OPENAI_BASE_URL=https://api.openai.com/v1

# 其他配置...
```

在 Python 代码中加载：

```python
from dotenv import load_dotenv
import os

# 加载 .env 文件
load_dotenv()

# 读取环境变量
api_key = os.getenv("OPENAI_API_KEY")
```

### 7.5 验证环境变量

```bash
# 查看环境变量
echo $OPENAI_API_KEY
echo $DEEPSEEK_API_KEY
echo $LANGFUSE_PUBLIC_KEY

# 在 Python 中验证
python << EOF
import os
print("OpenAI API Key:", os.getenv("OPENAI_API_KEY")[:20] + "...")
print("DeepSeek API Key:", os.getenv("DEEPSEEK_API_KEY")[:20] + "...")
print("Langfuse Public Key:", os.getenv("LANGFUSE_PUBLIC_KEY")[:20] + "...")
EOF
```

---

## 步骤8：JupyterLab 服务管理

### 8.1 创建 JupyterLab 启动脚本

创建专用启动脚本，解决 Conda 环境问题：

```bash
# 创建脚本目录
sudo mkdir -p /Agent101/app/jupyter

# 创建启动脚本
sudo vim /Agent101/app/jupyter/start_jupyter.sh
```

脚本内容：

```bash
#!/bin/bash

# ==========================================================
# JupyterLab 启动脚本
# ==========================================================

# --- API 环境变量配置 ---
# 注意：将下面的占位符替换为实际的 API Key

export OPENAI_BASE_URL="https://api.openai.com/v1"
export OPENAI_API_KEY="sk-your_openai_api_key_here"
export MODEL_NAME="gpt-4o"

export DEEPSEEK_BASE_URL="https://api.deepseek.com"
export DEEPSEEK_API_KEY="sk-your_deepseek_api_key_here"

export LANGFUSE_HOST="https://cloud.langfuse.com"
export LANGFUSE_PUBLIC_KEY="pk-lf-your_public_key_here"
export LANGFUSE_SECRET_KEY="sk-lf-your_secret_key_here"

export SERPAPI_API_KEY="your_serpapi_key_here"
export TAVILY_API_KEY="tvly-your_tavily_key_here"

export LANGSMITH_API_KEY="lsv2_your_langsmith_key_here"
export LANGSMITH_PROJECT="aiagent101"

export QWEATHER_API_BASE="your_qweather_base_url"
export QWEATHER_API_KEY="your_qweather_key_here"

# ==========================================================
# Conda 初始化
# ==========================================================
__conda_setup="$('/root/miniconda3/bin/conda' 'shell.bash' 'hook' 2> /dev/null)"
if [ $? -eq 0 ]; then
    eval "$__conda_setup"
else
    if [ -f "/root/miniconda3/etc/profile.d/conda.sh" ]; then
        . "/root/miniconda3/etc/profile.d/conda.sh"
    else
        export PATH="/root/miniconda3/bin:$PATH"
    fi
fi
unset __conda_setup

# 激活 conda 环境
conda activate agent101

# 运行 JupyterLab
# 使用 exec 确保 systemd 能正确跟踪主进程
exec /usr/local/bin/jupyter-lab \
    --config=/root/.jupyter/jupyter_lab_config.py \
    --no-browser
```

**重要说明**：
- 将所有 `your_*_key_here` 替换为实际的 API Key
- 或者通过 `source ~/.bashrc` 从全局配置加载

设置脚本权限：

```bash
sudo chmod +x /Agent101/app/jupyter/start_jupyter.sh
```

测试脚本：

```bash
# 手动运行测试
/Agent101/app/jupyter/start_jupyter.sh
# 按 Ctrl+C 停止
```

### 8.2 创建 systemd 服务

创建 systemd 服务文件：

```bash
sudo vim /etc/systemd/system/jupyter.service
```

服务配置内容：

```ini
[Unit]
Description=Jupyterlab Service
Documentation=https://jupyter.org/
After=network.target

[Service]
Type=simple
User=root
Group=root
WorkingDirectory=/Agent101/code
ExecStart=/Agent101/app/jupyter/start_jupyter.sh
Restart=always
RestartSec=10
StandardOutput=journal
StandardError=journal

[Install]
WantedBy=multi-user.target
```

**配置说明**：

| 配置项 | 说明 | 值 |
|--------|------|-----|
| `Description` | 服务描述 | JupyterLab Service |
| `After` | 启动顺序 | network.target（网络服务之后） |
| `Type` | 服务类型 | simple |
| `User` | 运行用户 | root（或你的用户名） |
| `WorkingDirectory` | 工作目录 | /Agent101/code |
| `ExecStart` | 启动命令 | 启动脚本路径 |
| `Restart` | 重启策略 | always（失败自动重启） |
| `RestartSec` | 重启等待时间 | 10秒 |

### 8.3 启动和管理服务

```bash
# 1. 重载 systemd 配置
sudo systemctl daemon-reload

# 2. 启用开机自启
sudo systemctl enable jupyter.service

# 3. 启动服务
sudo systemctl start jupyter.service

# 4. 查看服务状态
sudo systemctl status jupyter.service
```

**服务状态输出示例**：
```
● jupyter.service - Jupyterlab Service
     Loaded: loaded (/etc/systemd/system/jupyter.service; enabled; vendor preset: enabled)
     Active: active (running) since Mon 2025-01-01 10:00:00 CST; 5min ago
   Main PID: 12345 (jupyter-lab)
      Tasks: 12
     Memory: 150.0M
        CPU: 2.5s
     CGroup: /system.slice/jupyter.service
             └─12345 /root/miniconda3/envs/agent101/bin/python /usr/local/bin/jupyter-lab...
```

### 8.4 服务管理命令

```bash
# 启动服务
sudo systemctl start jupyter.service

# 停止服务
sudo systemctl stop jupyter.service

# 重启服务
sudo systemctl restart jupyter.service

# 重新加载配置（不中断服务）
sudo systemctl reload jupyter.service

# 查看服务状态
sudo systemctl status jupyter.service

# 查看服务日志
sudo journalctl -u jupyter.service

# 实时查看日志
sudo journalctl -u jupyter.service -f

# 查看最近 50 行日志
sudo journalctl -u jupyter.service -n 50

# 禁用开机自启
sudo systemctl disable jupyter.service
```

### 8.5 访问 JupyterLab

服务启动后，可以通过以下方式访问：

```
# 本地访问
http://localhost:8000

# 远程访问（使用虚拟机 IP）
http://192.168.x.x:8000

# 使用 token 登录
Token: fly123
```

**首次登录后设置密码（可选）**：
- 登录后 → 右上角 → Settings → Set Password
- 设置永久密码，无需每次输入 token

---

## 步骤9：验证环境

### 9.1 验证系统环境

```bash
# 检查操作系统版本
lsb_release -a
# 输出应包含：Description: Ubuntu 22.04.4 LTS

# 检查系统资源
free -h    # 内存
df -h      # 磁盘
nproc      # CPU 核心数
```

### 9.2 验证 Python 和 Conda

```bash
# 激活环境
conda activate agent101

# 检查 Python 版本
python --version
# 输出：Python 3.10.18

# 检查 conda 版本
conda --version

# 检查 pip 版本
pip --version

# 查看已安装的环境
conda env list

# 查看当前环境的包
pip list | head -20
```

### 9.3 验证 JupyterLab

```bash
# 检查 JupyterLab 版本
jupyter lab --version

# 检查已注册的内核
jupyter kernelspec list
# 应该看到 agent101 内核

# 检查 JupyterLab 服务状态
sudo systemctl status jupyter.service
# 应该显示 active (running)
```

### 9.4 验证环境变量

```bash
# 检查环境变量
echo "OpenAI Base URL: $OPENAI_BASE_URL"
echo "OpenAI API Key: ${OPENAI_API_KEY:0:20}..."
echo "DeepSeek API Key: ${DEEPSEEK_API_KEY:0:20}..."
echo "Langfuse Host: $LANGFUSE_HOST"

# Python 中验证
python << 'EOF'
import os
import sys

print("="*60)
print("环境验证报告")
print("="*60)

# Python 版本
print(f"\n✓ Python 版本: {sys.version}")

# 环境变量
env_vars = [
    "OPENAI_API_KEY",
    "DEEPSEEK_API_KEY",
    "LANGFUSE_PUBLIC_KEY",
    "TAVILY_API_KEY"
]

print("\n环境变量配置:")
for var in env_vars:
    value = os.getenv(var)
    if value:
        print(f"  ✓ {var}: {value[:20]}...")
    else:
        print(f"  ✗ {var}: 未配置")

# 导入测试
print("\n核心包导入测试:")
packages = {
    "langchain": "langchain",
    "langgraph": "langgraph",
    "openai": "openai",
    "fastapi": "fastapi",
    "langfuse": "langfuse"
}

for name, module in packages.items():
    try:
        pkg = __import__(module)
        version = getattr(pkg, "__version__", "unknown")
        print(f"  ✓ {name}: {version}")
    except ImportError:
        print(f"  ✗ {name}: 未安装")

print("\n="*60)
EOF
```

### 9.5 在 JupyterLab 中测试

访问 JupyterLab：`http://localhost:8000`

创建新的 Notebook，选择 `Python (agent101)` 内核，运行以下代码：

```python
# Cell 1: 系统信息
import sys
import platform

print(f"Python 版本: {sys.version}")
print(f"平台: {platform.platform()}")
print(f"处理器: {platform.processor()}")

# Cell 2: 环境变量
import os

print("环境变量检查:")
print(f"OPENAI_API_KEY: {os.getenv('OPENAI_API_KEY', 'Not Set')[:20]}...")
print(f"DEEPSEEK_API_KEY: {os.getenv('DEEPSEEK_API_KEY', 'Not Set')[:20]}...")
print(f"LANGFUSE_PUBLIC_KEY: {os.getenv('LANGFUSE_PUBLIC_KEY', 'Not Set')[:20]}...")

# Cell 3: 核心包测试
import langchain
import langgraph
import openai
import fastapi
import streamlit

print("核心包版本:")
print(f"LangChain: {langchain.__version__}")
print(f"OpenAI: {openai.__version__}")
print(f"FastAPI: {fastapi.__version__}")
print(f"Streamlit: {streamlit.__version__}")

# Cell 4: 简单 API 测试（需要有效的 API Key）
from openai import OpenAI

try:
    client = OpenAI(
        api_key=os.getenv("OPENAI_API_KEY"),
        base_url=os.getenv("OPENAI_BASE_URL")
    )
    
    response = client.chat.completions.create(
        model="gpt-3.5-turbo",
        messages=[{"role": "user", "content": "Hello!"}],
        max_tokens=10
    )
    
    print("✓ OpenAI API 连接成功!")
    print(f"响应: {response.choices[0].message.content}")
except Exception as e:
    print(f"✗ API 测试失败: {str(e)}")
```

### 9.6 验证项目代码

```bash
# 进入项目目录
cd /Agent101/code/Agent_In_Action

# 查看项目结构
tree -L 2 -d

# 检查各项目的依赖文件
find . -name "requirements.txt" -type f

# 验证项目1的代码
cd 01-agent-tool-mcp/mcp-demo
ls -la

# 返回根目录
cd /Agent101/code/Agent_In_Action
```

### 9.7 完整验证脚本

创建一个验证脚本：

```bash
vim /tmp/verify_env.sh
```

脚本内容：

```bash
#!/bin/bash

echo "=================================="
echo "  AI Agent 101 环境验证脚本"
echo "=================================="

# 颜色定义
RED='\033[0;31m'
GREEN='\033[0;32m'
NC='\033[0m' # No Color

# 检查函数
check_command() {
    if command -v $1 &> /dev/null; then
        echo -e "${GREEN}✓${NC} $1: $(command -v $1)"
        return 0
    else
        echo -e "${RED}✗${NC} $1: 未安装"
        return 1
    fi
}

# 系统信息
echo -e "\n[1] 系统信息"
echo "操作系统: $(lsb_release -d | cut -f2)"
echo "内核版本: $(uname -r)"
echo "CPU 核心: $(nproc)"
echo "内存: $(free -h | awk '/^Mem:/ {print $2}')"

# 基础工具
echo -e "\n[2] 基础工具"
check_command git
check_command wget
check_command curl
check_command vim

# Python 和 Conda
echo -e "\n[3] Python 和 Conda"
check_command python
check_command conda
check_command pip
echo "Python 版本: $(python --version 2>&1)"
echo "Conda 版本: $(conda --version 2>&1)"

# Jupyter
echo -e "\n[4] JupyterLab"
check_command jupyter
if [ $? -eq 0 ]; then
    echo "JupyterLab 版本: $(jupyter lab --version 2>&1)"
    echo "已注册的内核:"
    jupyter kernelspec list
fi

# JupyterLab 服务
echo -e "\n[5] JupyterLab 服务"
if systemctl is-active --quiet jupyter.service; then
    echo -e "${GREEN}✓${NC} JupyterLab 服务: 运行中"
else
    echo -e "${RED}✗${NC} JupyterLab 服务: 未运行"
fi

# 环境变量
echo -e "\n[6] 环境变量"
vars=("OPENAI_API_KEY" "DEEPSEEK_API_KEY" "LANGFUSE_PUBLIC_KEY")
for var in "${vars[@]}"; do
    if [ -n "${!var}" ]; then
        echo -e "${GREEN}✓${NC} $var: 已配置"
    else
        echo -e "${RED}✗${NC} $var: 未配置"
    fi
done

# 项目代码
echo -e "\n[7] 项目代码"
if [ -d "/Agent101/code/Agent_In_Action" ]; then
    echo -e "${GREEN}✓${NC} 项目目录: /Agent101/code/Agent_In_Action"
    echo "项目结构:"
    tree -L 1 -d /Agent101/code/Agent_In_Action 2>/dev/null || ls -d /Agent101/code/Agent_In_Action/*/
else
    echo -e "${RED}✗${NC} 项目目录不存在"
fi

# Docker (可选)
echo -e "\n[8] Docker (可选)"
if check_command docker &>/dev/null; then
    echo "Docker 版本: $(docker --version 2>&1)"
else
    echo "Docker 未安装（某些项目需要）"
fi

echo -e "\n=================================="
echo "  验证完成"
echo "=================================="
```

运行验证：

```bash
chmod +x /tmp/verify_env.sh
/tmp/verify_env.sh
```

---