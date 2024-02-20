---
title: 那些年我在 python 中扑过的街
date: 2016-01-10 23:03:01
category: 作为工程师
---
> 前事不忘，后事之师。 ——《战国策·赵策一》

## python 中的模块导入
### 包导入
这个比较简单，一个例子带过吧，但刚开始确实扑街了…

```python
#文件结构：
packages
├── __init__.py
└── my_module.py
# 这种是包导入
import packages
# 后续是从__init__.py 中定义的属性中获取的
packages.some_module.some_method
# 这种是把包当做搜索路径上的一环
from packages import some_module
some_module.some_method
```

### 模块导入搜索路径
例如下面的使用场景：

```python
# django 中有一个 common app，并且已经添加到已安装 app 列表中
# common app 文件结构
common
├── __init__.py
├── common.py
├── others.py
└── somethings.py
# somethings.py 中存在导入语句
from common.others import some_func   # 会报错
```

在上面的代码示例中，由于 common 的名称相同，以及搜索路径默认是先从当前执行代码所在目录开始搜索，所以 common 文件 会在 common 包之前被搜索到，造成搜索路径上的覆盖。而一旦搜索到 common 时，就会去 common 文件中查找 some_func，但是无法找到，这样就会报错。

## try-except-finally 中语句执行顺序
在 python 的异常处理中，无论 try 语句中是否有异常抛出，finally 语句总会被执行。由于这个特性，finally 语句经常被用来做一些清理工作，例如关闭文件，数据库等等。

如果当 try-except-finally 中出现异常，并且未被妥善处理时，python 会先把发生的异常暂存，当 finally 中的动作执行完毕之后，把保存的异常返回给上级。try-except-finally 中的语句执行顺序，决定了它会有一些潜在的陷阱。

看以下例子：

```python
def finallyTest():
    print 'start------'
    while  True:
        try:
            print "running------"
            raise IndexError("r")   #抛出异 IndexError 异常
        except NameError as e:
            print 'NameError happended {0}'.format(str(e))
            break
        finally:
            print 'finally executed'
            break #finally 语句中有 break 语句
            
# 执行测试函数
FinallyTest()
#输出结果为：
start------
running------
finally executed
```

在上面的例子中，在 try 语句中，raise 出了一个 IndexError 异常，而且 except 语句也没有捕获这个错误。按照平常的理解，这个错误会向上级（也就是调用这个函数的程序）传递，在本例中会传递到解释器，并引发一个 IndexError 错误。但是整个函数执行完之后并没有异常出现。

原因就在于在当 try 块中发生异常的时候，如果在 except 语句中找不到对应的异常处理，异常将会被「临时保存起来」。当 finally 执行完毕的时候，临时保存的异常将会「再次被抛出」，但如果 finally 语句中产生了新的异常或者执行了 return 或者 break 语句，那么临时保存的异常将会被丢失，从而导致「异常屏蔽」。

另外一个稍微复杂一些例子：

```py
import logging
logger = logging.Logger(__name__)
def returnTest(a):
    try:
        if a == False:
            raise ValueError("data can not be False")
        else:
            return a  
    except ValueError as e:
        logger.exception("exception!, {detail}").fomat(detail)
        return "return exception in except statement" 
    finally:
        return "return in finally statement"
print returnTest(False)
print returnTest(True)
```

按照平常理解，第一个 print 语句会触发错误，然后进入 except 语句打印异常，而同时 finally 部分中的语句必然会执行，所以这里会返回一个’return in finally statement’值。第二个则会返回 True。实际上则不然，返回值竟然都是’return in finally statement’。

原因是和上面的例子中是一样的，无论是 try 复合语句中，还是 except 复合语句中的 return/break 语句，这种会产生程序跳转的语句会先被保存起来。然后再去执行 finally 中的语句，而一旦 finally 中出现了跳转语句，就会直接跳转了，这样早成的结果就是上面保存状态的消失。

## 字典推导的限制
在做一个项目的时候，遇到一个需要多次请求数据库的操作，详细情况就是，有一些公司，这些公司本身有名字，公司代号这样的组织，如果每次都从数据库中取公司信息，会有性能问题。于是就采用用字典缓存下来，于是想使用字典推导。发现推导时只能生成单元素，之前我没有留意过这一点，在此记录一下。

```py
# 这样是可以的
>>>{model.key: model.value for Modle in models }
# 结果就是个普通的字典
>>>{key1: value1, key2: value2, ...}
# 但是想要下面这样使用是不支持的
>>>{model.key1: model.value1, moedel.key2: model.value2 for model in models}
# 折中的办法是将上面的 value 值换成一个字典，或者是使用列表推导再封装一层列表
>>>{model.key1: {models.key1: models.value1, model.key2: model.value2, ...} for model in models}
```
补充一下实际的例子，其中 branch 相关的是分公司信息，subbranch 相关的是中支公司信息，目的是构造一个，分公司名称为 key 值，分公司的主键，编号，以及该分公司辖下的中支公司为 value 值的字典，其中的中支公司也是一个字典：以中支公司名称为 key 值，中支公司的其他信息为 value 值。这样就构成了一个公司数据的二阶字典缓存列表。

```py
companys_by_name = {
        branch.name: {
            "pk": branch.pk,
            "number": branch.number,
            "subbranch": {
                subbranch.name: {"pk":subbranch.pk, "number":subbranch.number} for subbranch in SubbranchCompany.objects.filter(branch_company=branch)
            } for branch in BranchCompany.objects.all()
}
# 详细含义如下，其中中支公司是分工司辖下的公司
公司信息字典 = {
     分公司名字:{
        "主键": 当前迭代分公司主键，
        "公司号": 当前迭代公司公司号，
        "中支公司":{
            中支公司名字:{
                "主键": 当前迭代中支公司主键,
                "中支公司号": 当前迭代中支公司号,
            } for 中支公司 in 当期迭代分公司下中支公司集合
        }
     } for 分公司 in 分公司集合
}
```