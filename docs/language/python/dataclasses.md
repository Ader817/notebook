# dataclasses

> https://docs.python.org/3/library/dataclasses.html

`dataclasses` 是 Python 在 3.7+ 引入的一个标准库，用来快速定义数据类（Data Class），自动生成常见的样板代码，例如

- `__init__`
- `__repr__`
- `__eq__`

举个例子：

```python
from dataclasses import dataclass

@dataclass
class InventoryItem:
    """Class for keeping track of an item in inventory."""
    name: str
    unit_price: float
    quantity_on_hand: int = 0

    def total_cost(self) -> float:
        return self.unit_price * self.quantity_on_hand
```

会自动添加一个类似这样的  `__init__` 函数：
```python
def __init__(self, name: str, unit_price: float, quantity_on_hand: int = 0):
    self.name = name
    self.unit_price = unit_price
    self.quantity_on_hand = quantity_on_hand
```

## Module contents

### dataclass

@dataclasses.**dataclass**(*, *init=True*, *repr=True*, *eq=True*, *order=False*, *unsafe_hash=False*, *frozen=False*, *match_args=True*, *kw_only=False*, *slots=False*, *weakref_slot=False*)[¶](https://docs.python.org/3/library/dataclasses.html#dataclasses.dataclass)

这个函数是一个 decorator，以下几种使用方式是等价的：

```python
@dataclass
class C:
    ...

@dataclass()
class C:
    ...

@dataclass(init=True, repr=True, eq=True, order=False, unsafe_hash=False, frozen=False,
           match_args=True, kw_only=False, slots=False, weakref_slot=False)
class C:
    ...
```

!!! important

    `@property` 是 python 自带的一个 decorater，它的作用是把一个方法变成一个能够像属性一样访问的东西，如下所示：

    ```python
    class User:
        @property
        def name(self):
            return "Alice"

    u = User()
    u.name   # 不用加 ()
    ```

    在 dataclass decorater 中可以这么使用：

    ```python
    from dataclasses import dataclass

    @dataclass
    class Rectangle:
        width: float
        height: float

        @property
        def area(self):
            return self.width * self.height

    r = Rectangle(3, 4)
    print(r.area)  # 12
    ```

### field

dataclasses.**field**(***, *default=MISSING*, *default_factory=MISSING*, *init=True*, *repr=True*, *hash=None*, *compare=True*, *metadata=None*, *kw_only=MISSING*, *doc=None*)[¶](https://docs.python.org/3/library/dataclasses.html#dataclasses.field)

用于特化 dataclass 中的字段（不一定所有字段都要遵从 dataclass 的参数行为）

- `default`: 字段的默认值
  - 类定义时创建一次默认值
  - 示例创建时直接复用这个值
- `default_factory`: 一个无参函数，用于生成默认值
  - 类定义时保存一个函数
  - 实例创建时调用函数生成新值
