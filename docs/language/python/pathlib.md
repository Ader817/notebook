---
counter: True
---

# pathlib

> https://docs.python.org/3/library/pathlib.html

pathlib 是 Python 标准库中用于处理文件路径的模块，可以理解为一种更现代、更优雅的路径处理方式（替代 os.path）

基本使用方式：

```python
from pathlib import Path
p = Path("data/file.txt")
# 等价于 p = Path("data") / "file.txt"
p.name        # file.txt
p.stem        # file
p.suffix      # .txt
p.parent      # data/
p.exists()    # 是否存在
p.is_file()   # 是否文件
p.is_dir()    # 是否目录
```
