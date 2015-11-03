# options

```tornado.options```模块负责Tornado参数的解析，包括命令行参数和配置文件参数解析。

## 命令行
根据```tornado.options.parse_command_line```函数的实现，命令参数的解析具有以下规范：
1. 命令行参数只支持长选项，即`--long-option value`或`--long-option=vlaue`([long options](http://www.gnu.org/software/libc/manual/html_node/Argument-Syntax.html))，并且长选项只支持`--long-option=value`的方式，不支持`--long-option value`的方式。  

    如果你在程序中使用的是`--long-option value`的方式，则会报错说找不到"long-option"参数对应的值:
``tornado.options.Error: Option 'long-option' requires a value``
2. 命令行参数支持GNU `--`([参考资料]())。`--`会终止命令行参数的解析，并且在`--`之后的所有参数都不会被当作参数被解析，即使这些参数是合法的格式。
3. 

## 配置文件
