# Integration Tests

此路径包含集成测试.当运行集成测试时,测试工具将执行在`.test`文件并位于此路径的基础中.

一`.test`文件是一种简单的配置格式,其中每个选项都在一行中指定.键是`:`分隔符和值是右边的字符串.

| 钥匙 | 要求的 | 描述 |
| --- | --- | --- |
| `args` | 是的 | 指定测试的命令行参数.这通常是测试的输入脚本,以及`--reload`帮助确保Deno没有利用缓存. |
| `output` | 是的 | 这是一个文本文件,表示命令的输出.弦`[WILDCARD]`可以在输出中使用来指定接受任何输出的文本范围. |
| `exit_code` | 不 | 如果不存在,则假定脚本将正常退出(`0`)如果指定,安全带将确保接收到正确的代码. |
