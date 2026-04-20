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
