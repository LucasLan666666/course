---
title: Hypothesis
description: 可用来生成激励
categories: [示例项目, 教程]
tags: [examples, docs]
weight: 52
---


#  Hypothesis
在上一节中，我们通过手动编写测试用例，并为每个用例指定输入和预期输出。这种方式存在一些问题，例如测试用例覆盖不全面、**边界条件** 容易被忽略等。它是一个用于属性基于断言的软件测试的 Python 库。Hypothesis 的主要目标是使测试**更简单、更快速、更可靠**。它使用了一种称为**属性基于断言**的测试方法，即你可以为你的代码编写一些假（hypotheses），然后 Hypothesis 将会自动生成测试用例并验证这些假设。这使得编写全面且高效的测试变得更加容易。Hypothesis 可以自动生成各种类型的输入数据，包括基本类型（例如整数、浮点数、字符串等）、容器类型（例如列表、集合、字典等）、自定义类型等。然后，它会根据你提供的属性（即断言）进行测试，如果发现测试失败，它将尝试缩小输入数据的范围以找出最小的失败案例。通过 Hypothesis，你可以更好地覆盖代码的边界条件，并发现那些你可能没有考虑到的错误情况。这有助于提高代码的质量和可靠性。
## 基本概念
- 测试函数：即待测试的函数或方法，我们需要对其进行测试。
- 属性：定义了测试函数应该满足的条件。属性是以装饰器的形式应用于**测试函数**上的。
- 策略：用于生成测试数据的生成器。Hypothesis 提供了一系列内置的策略，如整数、字符串、列表等。我们也可以**自定义策略**。
- 测试生成器：基于策略生成测试数据的函数。Hypothesis 会自动为我们生成测试数据，并将其作为参数传递给测试函数。

本文将基于测试需求简单介绍Hypothesis的用法，其[完整手册](https://hypothesis.readthedocs.io/en/latest/)在这里，供同学们进行深入学习。
## 安装

使用pip安装，在python中导入即可使用
```shell
pip install hypothesis

import hypothesis
```


## 基本用法

### 属性和策略   
Hypothesis 使用属性装饰器来定义测试函数的属性。最常用的装饰器是 @given，它指定了测试函数应该满足的属性。    
我们可以通过@given 装饰器定义了一个测试函数 test_addition。并给x 添加对应的属性，测试生成器会自动为测试函数生成测试数据，并将其作为参数传递给函数，例如
```python
def addition(number: int) -> int:
    return number + 1

@given(x=integers(), y=integers())　　
def test_addition(x, y):　　   
	assert x + 1 == addition（1）
```

其中integers () 是一个内置的策略，用于生成整数类型的测试数据。Hypothesis 提供了丰富的内置策略，用于生成各种类型的测试数据。除了integers ()之外，还有字符串、布尔值、列表、字典等策略。例如使用 text () 策略生成字符串类型的测试数据，使用 lists (text ()) 策略生成字符串列表类型的测试数据
```python
@given(s=text(), l=lists(text()))
def test_string_concatenation(s, l):　　   
	result = s + "".join(l)　　   
	assert len(result) == len(s) + sum(len(x) for x in l)
```

除了可以使用内置的策略以外，还可以使用自定义策略来生成特定类型的测试数据，例如我们可以生产一个非负整形的策略
```python
def non_negative_integers():
　　return integers(min_value=0)
@given(x=non_negative_integers())
　　def test_positive_addition(x):
　　assert x + 1 > x
```

### 期望
我们可以通过expect 来指明需要的函数期待得到的结果
```python
@given(x=integers())
def test_addition(x):
    expected = x + 1
    actual = addition(x)
```
### 假设和断言
在使用 Hypothesis 进行测试时，我们可以使用标准的 Python 断言来验证测试函数的属性。Hypothesis 会自动为我们生成测试数据，并根据属性装饰器中定义的属性来运行测试函数。如果断言失败，Hypothesis 会尝试缩小测试数据的范围，以找出导致失败的最小样例。

假如我们有一个字符串反转函数，我们可以通过assert 来判断翻转两次后他是不是等于自身
```python
def test_reverse_string(s):
    expected = x + 1
    actual = addition(x)
	assert actual == expected
```

## 编写测试

- Hypothesis 中的测试由两部分组成：一个看起来像您选择的测试框架中的常规测试但带有一些附加参数的函数，以及一个@given指定如何提供这些参数的装饰器。以下是如何使用它来验证我们之前验证过的全加器的示例：

- 在上一节的代码基础上，我们进行一些修改，将生成测试用例的方法从随机数修改为integers ()方法，修改后的代码如下：

```python
from Adder import *
import pytest
from hypothesis import given, strategies as st

# 使用 pytest fixture 来初始化和清理资源
@pytest.fixture(scope="class")
def adder():
    # 创建 DUTAdder 实例，加载动态链接库
    dut = DUTAdder()
    # yield 语句之后的代码会在测试结束后执行，用于清理资源
    yield dut
    # 清理DUT资源，并生成测试覆盖率报告和波形
    dut.Finish()

class TestFullAdder:
    # 将 full_adder 定义为静态方法，因为它不依赖于类实例
    @staticmethod
    def full_adder(a, b, cin):
        cin = cin & 0b1
        Sum = a
        Sum += b + cin
        Cout = (Sum >> 128) & 0b1
        Sum &= 0xffffffffffffffffffffffffffffffff
        return Sum, Cout
    # 使用 hypothesis 自动生成测试用例
    @given(
        a=st.integers(min_value=0, max_value=0xffffffffffffffff),
        b=st.integers(min_value=0, max_value=0xffffffffffffffff),
        cin=st.integers(min_value=0, max_value=1)
    )
    # 定义测试方法，adder 参数由 pytest 通过 fixture 注入
    def test_full_adder_with_hypothesis(self, adder, a, b, cin):
        # 计算预期的和与进位
        sum_expected, cout_expected = self.full_adder(a, b, cin)
        # 设置 DUT 的输入
        adder.a.value = a
        adder.b.value = b
        adder.cin.value = cin
        # 执行一次时钟步进
        adder.Step(1)
        # 断言 DUT 的输出与预期结果相同
        assert sum_expected == adder.sum.value
        assert cout_expected == adder.cout.value

if __name__ == "__main__":
    # 以详细模式运行指定的测试
    pytest.main(['-v', 'test_adder.py::TestFullAdder'])

```
>这个例子中，@given 装饰器和 strategies 用于生成符合条件的随机数据。st.integers() 是生成指定范围整数的策略，用于为 a 和 b 生成 0 到 0xffffffffffffffff 之间的数，以及为 cin 生成 0 或 1。Hypothesis会自动重复运行这个测试，每次都使用不同的随机输入，这有助于揭示潜在的边界条件或异常情况。
- 运行测试，输出结果如下：
```shell
collected 1 item                                                               

 test_adder.py ✓                                                 100% ██████████

Results (0.42s):
       1 passed
```
> 可以看到在很短的时间里我们已经完成了测试