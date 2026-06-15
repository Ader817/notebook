# dotenv

Python-dotenv 是 python 用于处理和加载环境变量的第三方库

安装方式：

```bash
pip install python-dotenv
```

常见用法：

在项目根目录下放一个目录 `.env` 文件

```plaintext
MODEL_NAME=your_model_name_here
API_KEY=your_api_key_here
```

```python
from dotenv import load_dotenv
import os

load_dotenv()

model_name = os.getenv("MODEL_NAME")
api_key = os.getenv("OPENAI_API_KEY")

...
```

`load_dotenv()` 会尝试读取 `.env` 文件，并把这些值放进当前进程的环境变量里，所以你后面就能用 `os.getenv()` 来取。官方文档也提到，它会查找脚本所在目录或更高层目录中的 `.env` 文件
