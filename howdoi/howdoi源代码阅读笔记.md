# howdoi



## 文件结构

- requirements文件夹 ： 分别对应不同场景下的依赖关系，这里我们用 `dev.txt`
- howdoi/howdoi.py： 代码主体部分
- test_howdoi.py: 测试模块

此外，在贡献开源项目时，需要注意”pass all the tests and should not have any flake8 or pylint errors.“

## 粗略阅读howdoi.py

一些学到的意识流/开源项目经验Tips：

- 代码习惯：写一些成熟项目，养成写 `__init__.py` 的习惯，例如可以把version放到 `__init__.py` 里（ `__version__` ) 然后调用。
- 代码可读性： 写命令行的parser的时候，不妨试一下单独将 `parser` 和 `parser.add_argument` 放到一个函数里，例如 `get_parser`  然后返回该parser

一些模块的Tips：

- textwrap: 将string固定分割特定长度且转为list，可以用textwrap.wrap(text,12)
- textwrap:使用 `textwrap.dedent(text)` 在三引号下，想要将字符串与显示的左边缘对齐，同时仍然以缩进的形式在源代码中显示它们。
    
    ```json
    textwrap.dedent('''\
                                         environment variable examples:
                                           HOWDOI_COLORIZE=1
                                           HOWDOI_DISABLE_CACHE=1
                                           HOWDOI_DISABLE_SSL=1
                                           HOWDOI_SEARCH_ENGINE=google
                                           HOWDOI_URL=serverfault.com
                                         '''
    ```
    
- os:  `os.environ` 读取和设置统环境变量的思维方式来开发一款python项目
- inspect: 可以用来返回调用函数得名称。这里howdoi用到了 `inspect.currentframe` 和 `inspect.getouterframes`
    
    源代码：
    
    ```python
    def _get_cache_key(args):
        frame = inspect.currentframe()
        calling_func = inspect.getouterframes(frame)[1].function
        return calling_func + str(args) + __version__
    ```
    
    代码解释：
    
    在给定上下文中，`inspect` 模块允许您检查和获取有关 Python 代码的详细信息，包括函数调用堆栈、源文件和类结构等。
    
    在这个函数中，`inspect.currentframe()` 方法返回当前调用帧的参考，也就是该函数的帧。 `inspect.getouterframes(frame)` 方法返回包含所有外部帧的堆栈帧对象列表，并且 `getouterframes` 中的第一个帧是调用者。因此，调用 `inspect.getouterframes(frame)[1].function` 返回调用此函数的函数的名称。
    
    更多底层知识点探索：
    
    知识点1：先了解**[栈帧是什么，以及程序调用与栈帧](https://www.cnblogs.com/samo/articles/3092895.html)** 
    
- 使用[cachelib.FileSystemCache](https://cachelib.readthedocs.io/en/stable/file/)对于一些重复输出进行缓存。
    
    对应的源代码：
    
    ```python
    def _get_from_cache(cache_key):
        # As of cachelib 0.3.0, it internally logging a warning on cache miss
        current_log_level = logging.getLogger().getEffectiveLevel()
        # Reduce the log level so the warning is not printed
        logging.getLogger().setLevel(logging.ERROR)
        page = cache.get(cache_key)  # pylint: disable=assignment-from-none
        # Restore the log level
        logging.getLogger().setLevel(current_log_level)
        return page
    ```
    
- any： 善用 `any` 判断整个iterable内得元素是否全为False 或 有一个为True

## 阅读测试模块

这里采用的是 `test case` 的独立单元模块的测试，关于测试框架方面的系统知识参考python官方[文档](https://docs.python.org/zh-tw/3/library/unittest.html) 。这里为了学习相关的知识/开源贡献，在运行测试时，可以加 `-v` 以确认每个测试流程 `python -m test_howdoi -v` 

这里的测试环境被称为 test fixture：在没个TestCase下调用了  `setUp()` , `tearDown`  ， 

## 编程思想/设计理念学习

### 缓存机制

关键得函数有 `_get_cache_key`  和 `_get_from_cache` ，以及 `cache.set` , `_get_cache_key` 负责生成缓存关键字，传入参数为args，并返回当前调用该函数的函数名+args+`__version__` 返回得字符串，从而保证缓存机制不会混淆。而`_get_from_cache` 则负责根据传入的缓存关键字进行检索（调用[cachelib.FileSystemCache](https://cachelib.readthedocs.io/en/stable/file/)）

其中调用该函数得有：

- `_get_links_with_cache`
- `_get_answer`
- `howdoi`

结果有无缓存：首先 `howdoi` 检查当前 `args` 有无对应的缓存.

Link有无缓存：在没有获取到结果缓存的前提下，获取问题结果，使用函数 `_get_answer` 。在该函数内调用函数 `_get_links_with_cache` 里查看 `args['query']` 有无缓存（有则会用cached link）否则会调用 `_get_links`  .最后会调用一次 `cache.set` 将相关的cache_key进行保存。

## vscode debug

launch.json

```json
{
    // Use IntelliSense to learn about possible attributes.
    // Hover to view descriptions of existing attributes.
    // For more information, visit: https://go.microsoft.com/fwlink/?linkid=830387
    "version": "0.2.0",
    "configurations": [
        {
            "name": "Python: Module",
            "type": "python",
            "request": "launch",
            "module": "howdoi",
            "justMyCode": true,
            // "env": {
            //     "PYTHONPATH": "${workspaceFolder}"
            // },
            "args": ["hello world"],
        }
    ]
}
```
