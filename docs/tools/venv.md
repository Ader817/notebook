---
counter: true
---

# venv
## 虚拟环境创建与激活
### 创建虚拟环境
``` bash
python3 -m venv myenv # 创建名为myenv的虚拟环境
```

你的项目会形成类似这样的文件结构

```plaintext
my_project/                  # 项目根目录
├── venv/                    # 虚拟环境目录（通过 venv 创建）
│   ├── bin/                 # 可执行文件（激活脚本、Python 解释器等）
│   ├── lib/                 # 虚拟环境专用库（site-packages）
│   └── pyvenv.cfg           # 虚拟环境配置文件（记录 Python 路径等）
├── src/                     # 源代码目录
│   ├── __init__.py          # 声明 Python 包
│   ├── main.py              # 主程序入口
│   └── utils/               # 工具模块
│       ├── __init__.py
│       └── helper.py
├── tests/                   # 测试目录
│   ├── __init__.py
│   ├── test_main.py         # 主程序测试
│   └── test_utils.py        # 工具模块测试
```
### 激活虚拟环境
``` bash
source myenv/bin/activate  # 激活虚拟环境
```
激活后终端提示符会显示(myenv)，表示已进入虚拟环境

!!! note "alias"
    可以在配置文件（如~/.bashrc）中加入以下语句

    ```bash
    # alias for activating virtual environment
    # when in the same directory as venv
    alias activate='source venv/bin/activate'
    ```

    方便打字激活虚拟环境

### 验证激活状态
``` bash
pip list # 查看当前环境安装的包（若仅显示基础包，说明激活成功）
```

### 退出虚拟环境
```Bash
deactivate  # 退出当前虚拟环境 
```

## 依赖管理和包操作
### 安装包
```bash
pip install package_name # 安装单个包
pip install -r requirements.txt # 从项目给出的依赖文件批量安装依赖
```

### 生成依赖列表
```bash
pip freeze > requirements.txt
```

### 卸载包
``` bash
pip uninstall package_name # 移除指定包
```

### 升级包
``` bash
pip list --outdated # 查看环境中所有可以升级的包
pip install --upgrade package_name # 升级单个包
```

## 虚拟环境管理
### 删除虚拟环境
```bash
rm -rf myenv #直接删除虚拟环境目录
```

### 复制环境到其他主机
1. 导出依赖：
```bash
pip freeze > requirements.txt
```
2. 在新主机创建虚拟环境并安装依赖
```bash
python3 -m venv newenv && source newenv/bin/activate
pip install -r requirements.txt
```
3. 查看python解释器路径
```bash
which python
```

