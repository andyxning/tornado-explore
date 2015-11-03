# options

```tornado.options```模块负责tornado的参数的解析，包括命令行参数和配置文件解析。

### 命令行
根据```tornado.options.parse_command_line```函数的实现，命令参数的解析具有以下规范：
1. 命令行参数只支持长选项，即"--long-name-option value"或"--long-name-option=vlaue"([long-names options](https://www.gnu.org/prep/standards/html_node/Command_002dLine-Interfaces.html))，并且长选项只支持"--long-name-option=value"的方式。

