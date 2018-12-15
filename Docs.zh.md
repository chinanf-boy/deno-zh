# Deno Documentation

> Deno 文档

## Disclaimer

> 免责声明

警告:Deno 正在开发中.我们鼓励勇敢的早期勇敢的开采者，但不管 bug 的大小。该 API 可能会更改，恕不另行通知。

[Bug reports](https://github.com/denoland/deno/issues)帮帮忙!

## Install

> 安装

Deno 在 OSX、Linux 和 Windows 上工作。Deno 是单个二进制可执行文件。它没有外部依赖性。

[deno_install](https://github.com/denoland/deno_install)提供方便的脚本来下载和安装二进制文件.

使用 Python:

```
curl -L https://deno.land/x/install/install.py | python
```

或者使用 PowerShell:

```powershell
iex (iwr https://deno.land/x/install/install.ps1)
```

_注意:根据您的安全设置,您可能先必须运行`Set-ExecutionPolicy RemoteSigned -Scope CurrentUser`，才允许执行下载的脚本._

通过下载 tarball 或 zip 文件,也可以手动安装 Deno[github.com/denoland/deno/releases](https://github.com/denoland/deno/releases)。 这些包只包含一个可执行文件。您必须在 Mac 和 Linux 上设置可执行位置（`$PATH`).

试试看:

```
> deno https://deno.land/thumb.ts
```

## API Reference

> API 参考

要获得 deno 的运行时 API 的确切参考,请在命令行中运行以下命令:

```
> deno --types
```

或者看到[doc 网站](https://deno.land/typedoc/index.html).

如果将 deno 嵌入 Rust 程序,请参见[the rust 文档](https://deno.land/rustdoc/deno/index.html).

## Tutorial

> 教程

### An implementation of the unix "cat" program

> 一个 unix "cat"(终端打印命令) 项目的实现

在这个程序中,每个命令行参数都被假定为文件名，而后文件会被打开,并打印到 stdout。

```ts
import * as deno from 'deno';

(async () => {
  for (let i = 1; i < deno.args.length; i++) {
    let filename = deno.args[i];
    let file = await deno.open(filename);
    await deno.copy(deno.stdout, file);
    file.close();
  }
})();
```

这个`copy()`函数，在这里实际上只复制 必要的内核->用户空间->内核副本。也就是说，是从文件中读取数据的同一内存，被写入到 stdout。这也说明了 Deno 中， I/O 流的总体设计目标。

试试这个程序:

```
> deno https://deno.land/x/examples/cat.ts /etc/passwd
```

### TCP echo server

> TCP echo 服务器

这是一个简单的服务器的示例,它接受端口 8080 上的连接，并向客户机返回它所收到的任何内容。

```ts
import {listen, copy} from 'deno';

(async () => {
  const addr = '0.0.0.0:8080';
  const listener = listen('tcp', addr);
  console.log('listening on', addr);
  while (true) {
    const conn = await listener.accept();
    copy(conn, conn);
  }
})();
```

当启动该程序时,提示用户允许在网络上监听:

```
> deno https://deno.land/x/examples/echo_server.ts
deno requests network access to "listen". Grant? [yN] y
listening on 0.0.0.0:8080
```

出于安全原因,拒绝不允许程序，在没有明确许可的情况下，访问网络。为了避免控制台提示,使用命令行标志:

```
> deno https://deno.land/x/examples/echo_server.ts --allow-net
```

要测试它，请尝试使用 curl 向它发送 HTTP 请求。请求被直接写回客户端.

```
> curl http://localhost:8080/
GET / HTTP/1.1
Host: localhost:8080
User-Agent: curl/7.54.0
Accept: */*
```

值得注意的是，如同上面的`cat.ts`示例，这里的`copy()`函数也省略了不必要的内存复制。它从内核接收一个包并返回,没有进一步的复杂性。

### Linking to third party code

> 链接到第三方代码

在上面的示例中,我们看到 Deno 可以从 URL 执行脚本。与浏览器 JavaScript 类似，Deno 同样可以直接从 URL 导入库。此示例使用 URL 导入测试运行程序库:

```ts
import {test, assertEqual} from 'https://deno.land/x/testing/testing.ts';

test(function t1() {
  assertEqual('hello', 'hello');
});

test(function t2() {
  assertEqual('world', 'world');
});
```

试着运行这个:

```
> deno https://deno.land/x/examples/example_test.ts
Compiling /Users/rld/src/deno_examples/example_test.ts
Downloading https://deno.land/x/testing/testing.ts
Downloading https://deno.land/x/testing/util.ts
Compiling https://deno.land/x/testing/testing.ts
Compiling https://deno.land/x/testing/util.ts
running 2 tests
test t1
... ok
test t2
... ok

test result: ok. 2 passed; 0 failed; 0 ignored; 0 measured; 0 filtered out
```

注意,我们并没有提供`--allow-net`参数给这个程序，但是它确实访问了网络。运行引擎会下载导入的，并将其缓存到磁盘，同时具有特殊的访问权限。

Deno 将远程导入，缓存在`$DENO_DIR`环境变量。默认为`$HOME/.deno`(如果`$DENO_DIR`未指定)。下次运行该程序时，将不会进行下载。如果程序没有改变,它也不会重新编译。

**但是如果`https://deno.land/`失联了?** 依靠外部服务器进行开发是方便的,但是在生产中是脆弱的。生产的软件应该总是将其依赖项捆绑在一起。在 Deno 中,这是通过检查`$DENO_DIR`进入源代码控制系统,并将该路径指定为`$DENO_DIR`运行时的环境变量.

**如何导入到特定版本?**只需在 URL 中指定版本。例如,此 URL 完全表示正在运行的代码为:`https://unpkg.com/liltest@0.0.5/dist/liltest.js`。 结合上述在生产模式中，设置`$DENO_DIR`存储代码的技术，可以完全指定正在运行的确切代码,并且在没有网络访问的情况下执行代码。

**似乎到处导入 URL 都很笨拙。如果其中一个 URL 链接到一个稍微不同的库版本呢?在大型项目中到处维护 URL 不是很容易出错吗?**解决方案是用中心的`package.ts`文件，导入并重新导出你外部库，(与 Node 的`package.json`文件用途相同)。例如,假设您在一个大型项目中，使用了上面的测试库。而不是在某处导入`"https://deno.land/x/testing/testing.ts"`，都可以创建`package.ts`来将导出的第三方代码归档:

```ts
export {test, assertEqual} from 'https://deno.land/x/testing/testing.ts';
```

在整个项目中,可以从`package.ts`导入，并且避免对同一 URL 进行许多引用:

```ts
import {test, assertEqual} from './package.ts';
```

这种设计避免了由包管理软件、集中式代码存储库和多余的文件格式产生的过多的复杂性。

## Useful command line flags

> 有用的命令行标志(参数)

V8 有许多命令行标志,您可传递`--v8-options`试试。 下面是一些特别有用的:

```
--async-stack-traces
```

## How to Profile deno

> 如何分析 deno

为了开始分析,

```sh
# 确保我们只构建 release.
export DENO_BUILD_MODE=release
# 构建 deno 和 V8's d8.
./tools/build.py d8 deno
# 用 --prof 启动我们的 基准程序
./target/release/deno tests/http_bench.ts --allow-net --prof &
# Exercise it.
third_party/wrk/linux/wrk http://localhost:4500/
kill `pgrep deno`
```

V8 将在当前目录中,写入如下文件:`isolate-0x7fad98242400-v8.log`。 检查此文件:

```sh
D8_PATH=target/release/ ./third_party/v8/tools/linux-tick-processor
isolate-0x7fad98242400-v8.log > prof.log
# 在 macOS, 要使用 ./third_party/v8/tools/mac-tick-processor
```

`prof.log`将包含关于不同呼叫(调用函数)的时间片分布的信息.

要用 Web UI 查看日志,请生成日志的 JSON 文件:

```sh
D8_PATH=target/release/ ./third_party/v8/tools/linux-tick-processor
isolate-0x7fad98242400-v8.log --preprocess > prof.json
```

在浏览器中链接`third_party/v8/tools/profview/index.html`,选择`prof.json`会以图形方式查看分布。

了解更多`d8`以及配置文件,请查看以下链接:

- <https://v8.dev/docs/d8>
- <https://v8.dev/docs/profile>

## How to Debug deno

> 如何调试 deno

我们可以使用 LLDB 来调试 deno.

```sh
lldb -- target/debug/deno tests/worker.js
> run
> bt
> up
> up
> l
```

要调试 Rust 代码，可以使用`rust-lldb`。 应该会跟着`rustc`一起，且是 LLDB 的包装器.

```sh
rust-lldb -- ./target/debug/deno tests/http_bench.ts --allow-net
# 在 macOS, 你可能得到 警告，如
# `ImportError: cannot import name _remove_dead_weakref`
# 这时, 通过设置 PATH 使用 系统 python, e.g.
# PATH=/System/Library/Frameworks/Python.framework/Versions/2.7/bin:$PATH
(lldb) command script import "/Users/kevinqian/.rustup/toolchains/1.30.0-x86_64-apple-darwin/lib/rustlib/etc/lldb_rust_formatters.py"
(lldb) type summary add --no-value --python-function lldb_rust_formatters.print_val -x ".*" --category Rust
(lldb) type category enable Rust
(lldb) target create "../deno/target/debug/deno"
Current executable set to '../deno/target/debug/deno' (x86_64).
(lldb) settings set -- target.run-args  "tests/http_bench.ts" "--allow-net"
(lldb) b op_start
(lldb) r
```

## Build Instructions _(for advanced users only)_

> 构建指令*(仅高级用户)*

### Prerequisites:

> 先决条件

为了确保可持续发展的构建,deno 的大部分依赖项都在 git 子模块中。但是,您需要单独安装:

1.  [Rust](https://www.rust-lang.org/en-US/install.html)> 1.30.0
2.  [Node](https://nodejs.org/)
3.  Python 2.[不是 3](https://github.com/denoland/deno/issues/464#issuecomment-411795578).
4.  [ccache](https://developer.mozilla.org/en-US/docs/Mozilla/Developer_guide/Build_Instructions/ccache)(可选的,但对加速 V8 的重建很有帮助.)
5.  Windows 用户的额外步骤:
    1.  添加`python.exe`到`PATH`(e.g. `set PATH=%PATH%;C:\Python27\python.exe`)
    2.  得到[VS Community 2017](https://www.visualstudio.com/downloads/)。 请确保选择安装 C++工具和 Windows SDK 的选项.
    3.  启用`Debugging Tools for Windows`。 位于`Control Panel`>`Windows 10 SDK`->右击->`Change`>`Change`>`Check Debugging Tools for Windows`>`Change`>`Finish`.

### Build:

> 构建

```
# 获取 deps.
git clone --recurse-submodules https://github.com/denoland/deno.git
cd deno
./tools/setup.py

# 构建.
./tools/build.py

# 运行.
./target/debug/deno tests/002_hello.ts

# 测试.
./tools/test.py

# 格式代码.
./tools/format.py
```

其他有用的命令:

```
# 手动呼叫忍者。
./third_party/depot_tools/ninja -C target/debug

# 构建发布二进制文件。
DENO_BUILD_MODE=release ./tools/build.py :deno

# 列出可执行目标。
./third_party/depot_tools/gn ls target/debug //:* --as=output --type=executable

# 列表构建配置。
./third_party/depot_tools/gn args target/debug/ --list

# 编辑构建配置。
./third_party/depot_tools/gn args target/debug/

# 描述一个目标。
./third_party/depot_tools/gn desc target/debug/ :deno
./third_party/depot_tools/gn help

# 更新third_party模块
git submodule update
```

环境变量:`DENO_BUILD_MODE`,`DENO_BUILD_PATH`,`DENO_BUILD_ARGS`,`DENO_DIR`.

## Internals

> 内部

### Internal: libdeno API.

> 内部: libdeno API

Deno 的特权(权限)方面主要在 Rust 中编写。然而,将会有一个封装 V8 的小的 C API:

- 1)定义低层消息传递语义,
- 2)提供低层测试目标,
- 3)提供用于 Rust 的 ANSI C API 绑定接口.

V8 加上这个 C API 被称为"libdeno", 而 API 的重要部分位在:<https://github.com/denoland/deno/blob/master/libdeno/deno.h>
<https://github.com/denoland/deno/blob/master/js/libdeno.ts>

### Internal: Flatbuffers provide shared data between Rust and V8

> 内部: Flatbuffers 提供 Rust 与 V8 数据分享

我们使用 Flatbuffers 来定义 TypeScript 和 Rust 之间的公共结构和枚举。这些公共数据结构定义为<https://github.com/denoland/deno/blob/master/src/msg.fbs>

### Internal: Updating prebuilt binaries

> 更新 预构建的二进制文件

V8 需要很长时间来构建——大约一个小时。我们使用存储在 Google Storage 桶中的预构建的 V8 库，而不是每次都从头重新构建。然而,我们的构建系统是这样设置的, 为了必要时刻(对于调试或更改 V8 中的各种配置或构建预构建的二进制文件本身很有用)，将 V8 构建为 Deno 构建的一部分。若要控制是否使用预构建的 V8，请使用`use_v8_prebuilt`的 GN 参数。

使用`tools/gcloud_upload.py`上传新的预构建文件。

## Contributing

> 贡献

见[CONTRIBUTING.md](https://github.com/denoland/deno/blob/master/.github/CONTRIBUTING.md).

## Changelog

> 变更日志

### 2018.11.27 / v0.2.0 / Mildly usable

> 2018.11.27 / v0.2.0 / 可堪用

[一场 介绍交流会 已记录.](https://www.youtube.com/watch?v=FlTG0UXRAkE)

稳定性和可用性的改进。`fetch()`现在有 90%的功能了。增加了基本的 REPL 支持。增加了对 Shebang 的支持。命令行参数解析得到了改进。为 Deno 代码设置了`https://deno.land/x`(转载/运输)服务。示例代码已发布到[deno.land/x/examples](https://github.com/denoland/deno_examples)和[deno.land/x/net](https://github.com/denoland/net).

添加资源表，以抽象各种类型的 I/O 流和其他分配状态。资源是映射到某个 Rust 对象的整数标识符。它可以与各种操作一起使用,尤其是读和写。

### 2018.10.18 / v0.1.8 / Connecting to Tokio / Fleshing out APIs

> 2018.10.18 / v0.1.8 / 连接到 Tokio / 充实 APIs

大多数文件系统操作都实现了。实现了基本的 TCP 组网。公开基本 stdio 流。并且公开了许多随机 OS 设施(例如,环境变量)

Tokio 被选为事件循环库的后端。将 JS Promises 仔细映射到 Rust Futures 上，保留了错误处理和在主线程中同步执行的能力。

增加了连续的基准:<https://denoland.github.io/deno/>，性能问题开始得到解决。

"deno --types"，被添加到运行时参考 API。

向<https://github.com/denoland/deno/milestone/2>进发，我们预计 v0.2 将在下个 10 月或 11 月初发布。

### 2018.09.09 / v0.1.3 / Scale binding infrastructure

> 2018.09.09 / v0.1.3 / 扩展绑定基础架构

ETA v.0.2 2018 年 10 月<https://github.com/denoland/deno/milestone/2>

我们决定使用 Tokio<https://tokio.rs/>提供异步 I/O、线程池执行以及作为各种互联网协议(如 HTTP)的高级支持的基础。Tokio 是围绕 Future 这个概念而设计的，它很好地映射到了 JavaScript 的 Promises 上。我们希望尽可能容易地从 JavaScript 使用到 Tokio 的 future，而后获得 Promise，再处理它。我们希望这将导致初步的文件系统操作,还有 http 的 fetch()。此外,我们正在致力于 CI、发布和基准测试基础架构，以扩展开发。

### 2018.08.23 / v0.1.0 / Rust rewrite / V8 snapshot

> 2018.08.23 / v0.1.0 / Rust 重写 / V8 快照

<https://github.com/denoland/deno/commit/68d388229ea6ada339d68eb3d67feaff7a31ca97>

<https://github.com/denoland/deno/milestone/1>完成!

Go 是一种垃圾收集语言,我们担心将它与 V8 的 GC(垃圾收集) 结合，将导致以后困难的竞争问题。

V8Works2 绑定/概念库正在移植到一个新的 C++库中，称为 libdeno。libdeno 将包括整个 JS 运行时作为 V8 快照。它仍然遵循消息传递范例。Rust 将被绑定到这个库以实现 deno 的特权(权限)部分。有关详细信息,请参阅 deno2/README.md.

V8 快照可让 deno 避免在启动时，重新编译 TypeScript 编译器。这已经在使用了。

当重写好与 Go 原型的相当特性时，我们将发布二进制文件供人们尝试。

### 2018.09.32 / v0.0.0 / Golang Prototype / JSConf talk

> 2018.09.32 / v0.0.0 / Golang 原型 / JSConf 交流会

<https://github.com/denoland/deno/tree/golang>

<https://www.youtube.com/watch?v=M3BM9TB-8yA>

<https://tinyclouds.org/jsconf2018.pdf>

### 2007-2017 / Prehistory

> 2007-2017 / 前史

<https://github.com/ry/v8worker>

<https://libuv.org/>

<https://tinyclouds.org/iocp-links.html>

<https://nodejs.org/>

<https://github.com/nodejs/http-parser>

<https://tinyclouds.org/libebb/>
