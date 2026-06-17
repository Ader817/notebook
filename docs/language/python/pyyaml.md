# PyYAML

> [Official Documentation](https://wiki.python.org/python/YAML.html)
>
> [Python YAML tutorial](https://python.land/data-processing/python-yaml)

## Install and importing PyYAML

PyYAML 不属于 Python 标准库的一部分，需要用 pip 来安装它: `pip install pyyaml`

要在 python scripts 中使用 PyYAML，需要通过 `import yaml` 来导入模块

## Reading and parsing a YAML file with python

一旦导入了 YAML 解析器，我们就可以加载并解析 YAML 文件。YAML 文件通常带有扩展名 `.yaml` 或 `.yml` 。让我们使用以下名为 `config.yaml` 的示例 YAML 文件：

```yaml
rest:
  url: "https://example.org/primenumbers/v1"
  port: 8443

prime_numbers: [2, 3, 5, 7, 11, 13, 17, 19]
```

加载、解析和使用此配置文件与使用 Python JSON 库加载 JSON 类似。首先，我们打开文件。接着，我们使用 `yaml.safe_load()` 函数进行解析。请注意，我稍微调整了输出格式，以便您更清晰地阅读：

```python
>>> import yaml
>>> with open('config.yml', 'r') as file
...    prime_service = yaml.safe_load(file)

>>> prime_service
{'rest':
  { 'url': 'https://example.org/primenumbers/v1',
    'port': 8443
  },
  'prime_numbers': [2, 3, 5, 7, 11, 13, 17, 19]}

>>> prime_service['rest']['url']
https://example.org/primenumbers/v1
```

YAML 解析器返回一个最匹配数据的常规 Python 对象。在本例中，它是一个 Python 字典。这意味着可以使用所有常规字典功能，例如使用带默认值的 `get()`

!!! important

    字典的 get() 方法：`dict.get(key, default=None)`

## Parsing YAML strings with python

你可以使用 `yaml.safe_load()` 来解析各种有效的 YAML 字符串。以下是一个将简单项目列表解析为 Python 列表的示例：

```python
>>> import yaml
>>>
>>> names_yaml = """
... - 'eric'
... - 'justin'
... - 'mary-kate'
... """
>>>
>>> names = yaml.safe_load(names_yaml)
>>> names
['eric', 'justin', 'mary-kate']
```

## Writing (or dumping) YAML to a file

```python
>>> import yaml
>>>
>>> data = {
...    "name": "Alice",
...    "age": 25
...}
>>>
>>> yaml_str = yaml.dump(data)
>>> print(yaml_str)
age: 25
name: Alice
>>>
>>> with open("config.yaml", "w") as f:
... 	yaml.dump(data, f)
```
