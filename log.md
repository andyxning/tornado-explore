# log

## Desc
处理tronado框架中的日志记录。

 
## Notices
1. `tornado.log`模块配置的是Python的`logging`模块中的`root logger`。`tornado.log`模块的log相关参数设置都是针对`root logger`。Tornado框架中的各个自定义的`logger`最后的日志记录请求都会被委托给`root logger`，包括日志记录级别(`log level`)和相应的日志处理器（`handler`)

2. Python中`logging`[模块的原理](https://docs.python.org/2/library/logging.html#logging.Logger.setLevel)。
  * Python中的`logging`模块中的`logger`是树形结构，根节点是`root logger`，子节点是自定义的`logger`。每个`logger`的`parent`属性指示父`logger`。`root logger`的`parent`属性是`None`。
  * 默认，`root logger`在导入`logging`模块的时候就会被创建，`root logger`的日志级别是`WARNING`。
  * 当一个`logger`被创建的时候，日志级别默认是`NOTSET`，这会导致：如果接收请求的`logger`是`root logger`则所有级别的日志信息都被记录；否则会把相应的日志记录请求委托给父`logger`。
    * 将日志记录请求委托给父`logger`的意思：
      * 如果接收请求的`logger`日志级别是`NOTSET`,接收请求的`logger`的所有祖先`logger`会被逆序遍历直到一个祖先`logger`的日志级别不是`NOTSET`或者遍历到`root logger`。
      * 如果一个日志级别不是`NOTSET`的祖先`logger`被遍历到，则这个祖先`logger`的日志级别会被作为当前`logger`的有效日志级别，并根据这个有效日志级别来确定是否能处理日志记录请求。
      * 如果最后`root logger`被遍历到，并且`root logger`的日志级别是`NOTSET`，则所有的日志请求都会被满足；否则，将`root logger`的日志级别作为接收请求的`logger`的有效日志级别。
      * `logging.Logger.getEffectiveLevel`
      ```
      def getEffectiveLevel(self):
        """
        Get the effective level for this logger.

        Loop through this logger and its parents in the logger hierarchy,
        looking for a non-zero logging level. Return the first one found.
        """
        logger = self
        while logger:
            if logger.level:
                return logger.level
            logger = logger.parent
        return NOTSET
      ```

  * 处理器(`handler`)
    * 在处理日志记录请求的时候，会调用`logging.Logger.Callhandlers`。该函数的主要功能是在接收请求的`logger`和祖先`logger`中进行遍历，直到一个祖先`logger`的`propagate`属性变成`False`或者到达`root logger`（`root logger`的`parent`属性为`None`)。
    ```
    def callHandlers(self, record):
        """
        Pass a record to all relevant handlers.

        Loop through all handlers for this logger and its parents in the
        logger hierarchy. If no handler was found, output a one-off error
        message to sys.stderr. Stop searching up the hierarchy whenever a
        logger with the "propagate" attribute set to zero is found - that
        will be the last logger whose handlers are called.
        """
        c = self
        found = 0
        while c:
            for hdlr in c.handlers:
                found = found + 1
                if record.levelno >= hdlr.level:
                    hdlr.handle(record)
            if not c.propagate:
                c = None    #break out
            else:
                c = c.parent
        if (found == 0) and raiseExceptions and not self.manager.emittedNoHandlerWarning:
            sys.stderr.write("No handlers could be found for logger"
                             " \"%s\"\n" % self.name)
            self.manager.emittedNoHandlerWarning = 1
    ```

3. `root logger`
  * 通过`tornado.log.enable_pretty_logging`进行配置，包括日志级别(`log level`)和处理(`handler`)。
  ```
  def enable_pretty_logging(options=None, logger=None):
    """Turns on formatted logging output as configured.

    This is called automatically by `tornado.options.parse_command_line`
    and `tornado.options.parse_config_file`.
    """
    if options is None:
        from tornado.options import options
    if options.logging is None or options.logging.lower() == 'none':
        return
    if logger is None:
        logger = logging.getLogger()
    logger.setLevel(getattr(logging, options.logging.upper()))
    if options.log_file_prefix:
        rotate_mode = options.log_rotate_mode
        if rotate_mode == 'size':
            channel = logging.handlers.RotatingFileHandler(
                filename=options.log_file_prefix,
                maxBytes=options.log_file_max_size,
                backupCount=options.log_file_num_backups)
        elif rotate_mode == 'time':
            channel = logging.handlers.TimedRotatingFileHandler(
                filename=options.log_file_prefix,
                when=options.log_rotate_when,
                interval=options.log_rotate_interval,
                backupCount=options.log_file_num_backups)
        else:
            error_message = 'The value of log_rotate_mode option should be ' +\
                            '"size" or "time", not "%s".' % rotate_mode
            raise ValueError(error_message)
        channel.setFormatter(LogFormatter(color=False))
        logger.addHandler(channel)

    if (options.log_to_stderr or
            (options.log_to_stderr is None and not logger.handlers)):
        # Set up color if we are in a tty and curses is installed
        channel = logging.StreamHandler()
        channel.setFormatter(LogFormatter())
        logger.addHandler(channel)
  ```

4. `access_log`/`gen_log`/`app_log`
  * 在`tornado.log`模块级别进行定义
    ```
    # Logger objects for internal tornado use
    access_log = logging.getLogger("tornado.access")
    app_log = logging.getLogger("tornado.application")
    gen_log = logging.getLogger("tornado.general")

    ```
  * 根据1和2的解析，不难理解`access_log`/`gen_log`/`app_log`都没有定义日志级别和注册处理器，所以它们默认会根据2中的解释使用`root logger`的日志级别和处理器