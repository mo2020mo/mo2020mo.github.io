[目录](README.md)   [下一章](python-1007.md)    [上一章](python-1005.md)

# 程序的基本结构（五）：异常处理
> Python 提供的异常处理机制可以用下面的模板来说明

```python
    try:
    # 把有可能出现异常的代码放在 try 后面
    # 当出现异常时解释器会捕获异常
    # 并根据异常的类型执行后面的对应代码块
    do_something_nasty()
    except ValueError:
        # 如果发生 ValueError 类型的异常则执行这个代码块
        pass
    except (TypeError, ZeroDivisionError):
        # 可以一次指定几个不同类型的异常在一起处理exceptions
        # 如果出现 TypeError 或者 ZeroDivisionError 则执行这个代码块
        pass
    except:
        # 所有上面没有专门处理的类型的异常会在这里处理
        pass
    else:
        # 当且仅当 try 代码块里无异常发生时这个代码块会被执行
        pass
    finally:
        # 无论发生了什么这个代码块都会被执行
        # 通常这里是清理性的代码，比如我们在 try 里面打开一个文件进行处理
        # 无论过程中有没有异常出现最后都应该关闭文件释放资源
        # 这样的操作就适合在这里执行
```
# 小结
- 程序处理用户或其他系统提供的输入时可能出现预期之外的异常状况，可以使用异常处   理来捕获异常并进行应急处置
- 理解 Python 异常处理的模板含义
- 通过例子初步了解异常处理可能的应用场景