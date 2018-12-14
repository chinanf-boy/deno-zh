# deno

| **Linux&Mac** | **窗户** |
| :-----------: | :----: |
| [![][tci badge]][tci link] | [![][avy badge]][avy link] |

## A secure TypeScript runtime built on V8

-   支持开箱即用的TypeScript.使用最新版本的V8.也就是说,它是非常现代的JavaScript.

-   不`package.json`. 没有NPM.与Node不显式兼容.

-   只导入参考源代码URL.

    ```typescript
    import { test } from "https://unpkg.com/deno_testing@0.0.5/testing.ts";
    import { log } from "./util.ts";
    ```

    在第一次执行时,获取并缓存远程代码,并且在使用`--reload`旗帜.(所以,这在飞机上仍然有效.)见`~/.deno/src`有关缓存的详细信息.)

-   可以控制文件系统和网络访问,以便运行沙箱代码.默认为只读文件系统访问且没有网络访问.V8(非特权)和Rust(特权)之间的访问仅通过以下定义的序列化消息完成[flatbuffer](https://github.com/denoland/deno/blob/master/src/msg.fbs). 这使得审计变得容易.要显式启用写访问,请使用`--allow-write`和`--allow-net`用于网络访问.

-   单个可执行文件:

    ```
    > ls -lh target/release/deno
    -rwxr-xr-x  1 rld  staff    48M Aug  2 13:24 target/release/deno
    > otool -L target/release/deno
    target/release/deno:
      /usr/lib/libSystem.B.dylib (compatibility version 1.0.0, current version 1252.50.4)
      /usr/lib/libresolv.9.dylib (compatibility version 1.0.0, current version 1.0.0)
      /System/Library/Frameworks/Security.framework/Versions/A/Security (compatibility version 1.0.0, current version 58286.51.6)
      /usr/lib/libc++.1.dylib (compatibility version 1.0.0, current version 400.9.0)
    >
    ```

-   总是死于未发现的错误.

-   [Aims to support top-level `await`.](https://github.com/denoland/deno/issues/471)

-   目标是与浏览器兼容.

## Install

使用Python:

```
curl -L https://deno.land/x/install/install.py | python
```

与PowerShell:

```powershell
iex (iwr https://deno.land/x/install/install.ps1)
```

*注意:根据您的安全设置,您可能必须运行`Set-ExecutionPolicy RemoteSigned -Scope CurrentUser`首先允许执行下载的脚本.*

试试看:

```
> deno https://deno.land/thumb.ts
```

也见[deno_install](https://github.com/denoland/deno_install).

## Status

正在开发中.

我们发布二进制版本[here](https://github.com/denoland/deno/releases).

文档是[here](https://github.com/denoland/deno/blob/master/Docs.md).

<!-- prettier-ignore -->

[avy badge]: https://ci.appveyor.com/api/projects/status/yel7wtcqwoy0to8x?branch=master&svg=true

[avy link]: https://ci.appveyor.com/project/deno/deno

[tci badge]: https://travis-ci.com/denoland/deno.svg?branch=master

[tci link]: https://travis-ci.com/denoland/deno
