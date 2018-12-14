# Deno Roadmap

API和特性请求应该作为PR提交到本文档.

## Security Model (partially implemented)

-   我们希望在默认情况下是安全的;用户应该能够运行不受信任的代码,比如网络.
-   威胁模型:
    -   修改/删除本地文件
    -   泄露私人信息
-   禁止默认:
    -   网络接入
    -   本地写访问
    -   非JS扩展
    -   子过程
    -   接入网
-   允许默认:
    -   本地读访问.
    -   argv、stdout、stderr、stdin访问总是允许的.
    -   也许:临时写访问.(但是如果他们在那里创建符号链接呢?)
-   当软件试图做它没有权限做的事情时,用户会收到提示.
-   当请求访问时,可以选择获取堆栈跟踪.
-   担心由于猴子修补技术,授予每个文件的访问权限会带来错误的安全感.应该为每个程序(js上下文)授予访问权限.

示例安全提示.选项是:是,不,打印堆栈

```
Program requests write access to "~/.ssh/id_rsa". Grant? [yNs]
http://gist.github.com/asdfasd.js requests network access to "www.facebook.com". Grant? [yNs]
Program requests access to environment variables. Grant? [yNs]
Program requests to spawn `rm -rf /`. Grant? [yNs]
```

-   cli标志用于提前授予访问--.-all--.-write--.-net--.-env--.-exec
-   在版本2中,我们将增加更细粒度访问的能力——.-net=facebook.com

## Top-level Await (Not Implemented)

[#471](https://github.com/denoland/deno/issues/471)

这将被推迟到至少deno2里程碑1完成.主要问题之一是顶级等待调用在语法上不是有效的TypeScript.

### [Broken] List dependencies of a program.

当前中断:<https://github.com/denoland/deno/issues/1011>

```
% deno --deps http://gist.com/blah.js
http://gist.com/blah.js
http://gist.com/dep.js
https://github.com/denoland/deno/master/testing.js
%
```
