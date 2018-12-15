# Deno Roadmap

> Deno 路线图

API 和特性请求应该作为 PR 提交到本文档。

## Security Model (partially implemented)

> 安全模式(部分已实现)

- 我们希望在默认情况下是安全的;用户应该能够运行不受信任的代码，但不会遭受损失,比如网络。
- 威胁模型:
  - 修改/删除本地文件
  - 泄露私人信息
- 禁止默认:
  - 网络接入
  - 本地写访问
  - 非 JS 扩展
  - 子过程
  - 环境访问
- 允许默认:
  - 本地读访问.
  - argv、stdout、stderr、stdin 访问总是允许的.
  - 暂定:临时目录的写访问。(但是如果他们在那里，创建符号链接呢?)
- 当软件试图做它没有权限做的事情时,用户会收到提示。
- 当请求访问时,可以选择获取堆栈跟踪。
- 担心由于猴子补丁技术,授予每个文件的访问权限会带来错误的安全感。所以应该为每个程序(js 上下文)授予访问权限.

示例的安全提示。选项是:YES, NO, PRINT STACK

```
Program requests write access to "~/.ssh/id_rsa". Grant? [yNs]
http://gist.github.com/asdfasd.js requests network access to "www.facebook.com". Grant? [yNs]
Program requests access to environment variables. Grant? [yNs]
Program requests to spawn `rm -rf /`. Grant? [yNs]
```

- cli 标志用于提前授予访问，如
  `--allow-all --allow-write --allow-net --allow-env --allow-exec`
- 在版本 2 中,我们将增加更细粒度访问的能力
  `--allow-net=facebook.com`

## Top-level Await (Not Implemented)

> 顶层 Await（还没实现)

[#471](https://github.com/denoland/deno/issues/471)

这将被推迟到，至少 deno2 里程碑 1 才能完成。主要问题之一是顶级 await 调用在语法上，不是有效的 TypeScript。

### [Broken] List dependencies of a program.

> [中断] 列出一个项目的依赖项

当前中断在:<https://github.com/denoland/deno/issues/1011>

```
% deno --deps http://gist.com/blah.js
http://gist.com/blah.js
http://gist.com/dep.js
https://github.com/denoland/deno/master/testing.js
%
```
