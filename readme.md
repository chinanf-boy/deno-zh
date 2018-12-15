# denoland/deno [![explain]][source] [![translate-svg]][translate-list]

<!-- [![size-img]][size] -->

[explain]: http://llever.com/explain.svg
[source]: https://github.com/chinanf-boy/Source-Explain
[translate-svg]: http://llever.com/translate.svg
[translate-list]: https://github.com/chinanf-boy/chinese-translate-list
[size-img]: https://packagephobia.now.sh/badge?p=Name
[size]: https://packagephobia.now.sh/result?p=Name

「 在 V8 上 构建一个安全 TypeScript 运行引擎 」

[中文](./readme.md) | [english](https://github.com/denoland/deno)

---

## 校对 ✅

<!-- doc-templite START generated -->
<!-- repo = 'denoland/deno' -->
<!-- commit = '0bb43ebbfcbc378810f75c43a2be3369729921f7' -->
<!-- time = '2018-12-14' -->
翻译的原文 | 与日期 | 最新更新 | 更多
---|---|---|---
[commit] | ⏰ 2018-12-14 | ![last] | [中文翻译][translate-list]

[last]: https://img.shields.io/github/last-commit/denoland/deno.svg
[commit]: https://github.com/denoland/deno/tree/0bb43ebbfcbc378810f75c43a2be3369729921f7

<!-- doc-templite END generated -->

- [x] [README.md](README.md)
- [x] [Docs.md](Docs.zh.md)
- [x] [Roadmap.md](Roadmap.zh.md)

### 贡献

欢迎 👏 勘误/校对/更新贡献 😊 [具体贡献请看](https://github.com/chinanf-boy/chinese-translate-list#贡献)

## 生活

[help me live , live need money 💰](https://github.com/chinanf-boy/live-need-money)

---

# deno

|       **Linux&Mac**        |        **Windows**         |
| :------------------------: | :------------------------: |
| [![][tci badge]][tci link] | [![][avy badge]][avy link] |

## A secure TypeScript runtime built on V8

> 在 V8 上 构建一个安全 TypeScript 运行引擎。

- 支持开箱即用的 TypeScript。使用最新版本的 V8。也就是说,它是非常现代的 JavaScript.

- 不用`package.json`。 没有 NPM。与 Node 不显式兼容.

- 只导入参考源代码 URL.

  ```typescript
  import {test} from 'https://unpkg.com/deno_testing@0.0.5/testing.ts';
  import {log} from './util.ts';
  ```

  在第一次执行时,获取并缓存远程代码，除非使用`--reload`参数。(所以,这在飞机上仍然有效。)见`~/.deno/src`中有关缓存的详细信息。)

- 可以控制文件系统和网络访问,以便运行沙箱代码。默认为只读文件系统访问，且没有网络访问。V8(非特权)和 Rust(特权)之间的访问，仅通过以下定义的[flatbuffer](https://github.com/denoland/deno/blob/master/src/msg.fbs)序列化消息完成。 这使得审计变得容易。要显式启用写访问,请使用`--allow-write`和，`--allow-net`用于网络访问.

- 单个可执行文件:

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

- 总是死于，未发现的错误。

- [目的之一是支持 顶层 `await`.](https://github.com/denoland/deno/issues/471)

- 目标是与浏览器兼容。

## Install

使用 Python:

```
curl -L https://deno.land/x/install/install.py | python
```

用 PowerShell:

```powershell
iex (iwr https://deno.land/x/install/install.ps1)
```

_注意:根据您的安全设置,您可能必须先运行`Set-ExecutionPolicy RemoteSigned -Scope CurrentUser`，才能允许下载的脚本执行._

试试看:

```
> deno https://deno.land/thumb.ts
```

也可看看[deno_install](https://github.com/denoland/deno_install).

## Status

正在开发中.

我们发布二进制版本，[here](https://github.com/denoland/deno/releases).

文档是[这里](./Docs.zh.md).

<!-- prettier-ignore -->
[avy badge]: https://ci.appveyor.com/api/projects/status/yel7wtcqwoy0to8x?branch=master&svg=true
[avy link]: https://ci.appveyor.com/project/deno/deno
[tci badge]: https://travis-ci.com/denoland/deno.svg?branch=master
[tci link]: https://travis-ci.com/denoland/deno
