# Deno Documentation

## Disclaimer

警告:Deno正在开发中.我们鼓励勇敢的早期采用者,但期望bug的大小不同.该API可能会更改,恕不另行通知.

[Bug reports](https://github.com/denoland/deno/issues)帮帮忙!

## Install

Deno在OSX、Linux和Windows上工作.Deno是单个二进制可执行文件.它没有外部依赖性.

[deno_install](https://github.com/denoland/deno_install)提供方便的脚本来下载和安装二进制文件.

使用Python:

```
curl -L https://deno.land/x/install/install.py | python
```

或者使用PowerShell:

```powershell
iex (iwr https://deno.land/x/install/install.ps1)
```

*注意:根据您的安全设置,您可能必须运行`Set-ExecutionPolicy RemoteSigned -Scope CurrentUser`首先允许执行下载的脚本.*

通过下载tarball或zip文件,也可以手动安装Deno[github.com/denoland/deno/releases](https://github.com/denoland/deno/releases). 这些包只包含一个可执行文件.您必须在Mac和Linux上设置可执行位.

试试看:

```
> deno https://deno.land/thumb.ts
```

## API Reference

要获得deno的运行时API的确切引用,请在命令行中运行以下命令:

```
> deno --types
```

或者看到[doc website](https://deno.land/typedoc/index.html).

如果将deno嵌入Rust程序,请参见[the rust docs](https://deno.land/rustdoc/deno/index.html).

## Tutorial

### An implementation of the unix "cat" program

在这个程序中,每个命令行参数都被假定为文件名,文件被打开,并打印到stdout.

```ts
import * as deno from "deno";

(async () => {
  for (let i = 1; i < deno.args.length; i++) {
    let filename = deno.args[i];
    let file = await deno.open(filename);
    await deno.copy(deno.stdout, file);
    file.close();
  }
})();
```

这个`copy()`这里的函数实际上只复制必要的内核->用户空间->内核副本.也就是说,从文件中读取数据的同一内存被写入stdout.这说明了Deno中I/O流的总体设计目标.

试试这个程序:

```
> deno https://deno.land/x/examples/cat.ts /etc/passwd
```

### TCP echo server

这是一个简单的服务器的示例,它接受端口8080上的连接,并向客户机返回它所发送的任何内容.

```ts
import { listen, copy } from "deno";

(async () => {
  const addr = "0.0.0.0:8080";
  const listener = listen("tcp", addr);
  console.log("listening on", addr);
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

出于安全原因,拒绝不允许程序在没有明确许可的情况下访问网络.为了避免控制台提示,使用命令行标志:

```
> deno https://deno.land/x/examples/echo_server.ts --allow-net
```

要测试它,请尝试使用curl向它发送HTTP请求.请求被直接写回客户端.

```
> curl http://localhost:8080/
GET / HTTP/1.1
Host: localhost:8080
User-Agent: curl/7.54.0
Accept: */*
```

值得注意的是`cat.ts`例如,`copy()`这里的函数也不进行不必要的内存复制.它从内核接收一个包并返回,没有进一步的复杂性.

### Linking to third party code

在上面的示例中,我们看到Deno可以从URL执行脚本.与浏览器JavaScript类似,Deno可以直接从URL导入库.此示例使用URL导入测试运行程序库:

```ts
import { test, assertEqual } from "https://deno.land/x/testing/testing.ts";

test(function t1() {
  assertEqual("hello", "hello");
});

test(function t2() {
  assertEqual("world", "world");
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

注意,我们不必提供`--allow-net`这个程序的标志,但是它访问了网络.运行时对下载导入并将其缓存到磁盘具有特殊的访问权限.

Deno将远程导入缓存在`$DENO_DIR`环境变量.默认为`$HOME/.deno`如果`$DENO_DIR`未指定.下次运行该程序时,将不会进行下载.如果程序没有改变,它也不会重新编译.

**但是如果`https://deno.land/`下去?**依靠外部服务器进行开发是方便的,但是在生产中是脆弱的.生产软件应该总是捆绑其依赖项.在Deno中,这是通过检查`$DENO_DIR`进入源代码控制系统,并将该路径指定为`$DENO_DIR`运行时的环境变量.

**如何导入到特定版本?**只需在URL中指定版本.例如,此URL完全指定正在运行的代码:`https://unpkg.com/liltest@0.0.5/dist/liltest.js`. 结合上述设置技术`$DENO_DIR`在生产到存储的代码中,可以完全指定正在运行的确切代码,并且在没有网络访问的情况下执行代码.

**似乎到处导入URL都很笨拙.如果其中一个URL链接到一个稍微不同的库版本呢?在大型项目中到处维护URL不是很容易出错吗?**解决方案是导入并重新导出位于中央的外部库`package.ts`文件(与Node的用途相同`package.json`文件).例如,假设您在一个大型项目中使用了上面的测试库.而不是进口`"https://deno.land/x/testing/testing.ts"`在任何地方,都可以创建`package.ts`将导出的第三方代码归档:

```ts
export { test, assertEqual } from "https://deno.land/x/testing/testing.ts";
```

在整个项目中,可以从`package.ts`并且避免对同一URL进行许多引用:

```ts
import { test, assertEqual } from "./package.ts";
```

这种设计避免了由包管理软件、集中式代码存储库和多余的文件格式产生的过多的复杂性.

## Useful command line flags

V8有许多命令行标志,您可以通过`--v8-options`. 下面是一些特别有用的:

```
--async-stack-traces
```

## How to Profile deno

为了开始分析,

```sh
# Make sure we're only building release.
export DENO_BUILD_MODE=release
# Build deno and V8's d8.
./tools/build.py d8 deno
# Start the program we want to benchmark with --prof
./target/release/deno tests/http_bench.ts --allow-net --prof &
# Exercise it.
third_party/wrk/linux/wrk http://localhost:4500/
kill `pgrep deno`
```

V8将在当前目录中写入如下文件:`isolate-0x7fad98242400-v8.log`. 检查此文件:

```sh
D8_PATH=target/release/ ./third_party/v8/tools/linux-tick-processor
isolate-0x7fad98242400-v8.log > prof.log
# on macOS, use ./third_party/v8/tools/mac-tick-processor instead
```

`prof.log`将包含关于不同呼叫的蜱虫分布的信息.

要用Web UI查看日志,请生成日志的JSON文件:

```sh
D8_PATH=target/release/ ./third_party/v8/tools/linux-tick-processor
isolate-0x7fad98242400-v8.log --preprocess > prof.json
```

正常开放`third_party/v8/tools/profview/index.html`在浏览器中,选择`prof.json`以图形方式查看分布.

了解更多`d8`以及配置文件,请查看以下链接:

-   <https://v8.dev/docs/d8>
-   <https://v8.dev/docs/profile>

## How to Debug deno

我们可以使用LLDB来调试deno.

```sh
lldb -- target/debug/deno tests/worker.js
> run
> bt
> up
> up
> l
```

要调试Rust代码,可以使用`rust-lldb`. 应该会来的`rustc`并且是LLDB的包装器.

```sh
rust-lldb -- ./target/debug/deno tests/http_bench.ts --allow-net
# On macOS, you might get warnings like
# `ImportError: cannot import name _remove_dead_weakref`
# In that case, use system python by setting PATH, e.g.
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

## Build Instructions *(for advanced users only)*

### Prerequisites:

为了确保可再现的构建,deno的大部分依赖项都在git子模块中.但是,您需要单独安装:

1.  [Rust](https://www.rust-lang.org/en-US/install.html)> 1.30.0
2.  [Node](https://nodejs.org/)
3.  Python 2.[Not 3](https://github.com/denoland/deno/issues/464#issuecomment-411795578).
4.  [ccache](https://developer.mozilla.org/en-US/docs/Mozilla/Developer_guide/Build_Instructions/ccache)(可选的,但对加速V8的重建很有帮助.)
5.  Windows用户的额外步骤:
    1.  添加`python.exe`到`PATH`(例如)`set PATH=%PATH%;C:\Python27\python.exe`)
    2.  得到[VS Community 2017](https://www.visualstudio.com/downloads/). 请确保选择安装C++工具和Windows SDK的选项.
    3.  使能`Debugging Tools for Windows`. 去`Control Panel`>`Windows 10 SDK`->右击->`Change`>`Change`>`Check Debugging Tools for Windows`>`Change`>`Finish`.

### Build:

```
# Fetch deps.
git clone --recurse-submodules https://github.com/denoland/deno.git
cd deno
./tools/setup.py

# Build.
./tools/build.py

# Run.
./target/debug/deno tests/002_hello.ts

# Test.
./tools/test.py

# Format code.
./tools/format.py
```

其他有用的命令:

```
# Call ninja manually.
./third_party/depot_tools/ninja -C target/debug

# Build a release binary.
DENO_BUILD_MODE=release ./tools/build.py :deno

# List executable targets.
./third_party/depot_tools/gn ls target/debug //:* --as=output --type=executable

# List build configuration.
./third_party/depot_tools/gn args target/debug/ --list

# Edit build configuration.
./third_party/depot_tools/gn args target/debug/

# Describe a target.
./third_party/depot_tools/gn desc target/debug/ :deno
./third_party/depot_tools/gn help

# Update third_party modules
git submodule update
```

环境变量:`DENO_BUILD_MODE`,`DENO_BUILD_PATH`,`DENO_BUILD_ARGS`,`DENO_DIR`.

## Internals

### Internal: libdeno API.

丹诺的特权方面将主要在Rust中编程.然而,将会有一个封装V8到1的小的C API)定义低级消息传递语义,2)提供低级测试目标,3)提供用于Rust的ANSI C API绑定接口.V8加上这个C API被称为"libdeno",API的重要位在这里指定:<https://github.com/denoland/deno/blob/master/libdeno/deno.h>
<https://github.com/denoland/deno/blob/master/js/libdeno.ts>

### Internal: Flatbuffers provide shared data between Rust and V8

我们使用Flatbuffers来定义TypeScript和Rust之间的公共结构和枚举.这些公共数据结构定义为<https://github.com/denoland/deno/blob/master/src/msg.fbs>

### Internal: Updating prebuilt binaries

V8需要很长时间来构建——大约一个小时.我们使用存储在Google Storage桶中的预构建的V8库,而不是每次都从头重新构建.然而,我们的构建系统是这样设置的,以便我们能够在必要时将V8构建为Deno构建的一部分(对于调试或更改V8中的各种配置或构建预构建的二进制文件本身很有用).若要控制是否使用预构建的V8,请使用`use_v8_prebuilt`GN论点.

使用`tools/gcloud_upload.py`上传新的预构建文件.

## Contributing

见[CONTRIBUTING.md](https://github.com/denoland/deno/blob/master/.github/CONTRIBUTING.md).

## Changelog

### 2018.11.27 / v0.2.0 / Mildly usable

[An intro talk was recorded.](https://www.youtube.com/watch?v=FlTG0UXRAkE)

稳定性和可用性的改进.`fetch()`现在有90%的功能了.增加了基本的REPL支持.增加了对Shebang的支持.命令行参数解析得到了改进.转运服务`https://deno.land/x`为Deno代码设置.示例代码已发布[deno.land/x/examples](https://github.com/denoland/deno_examples)和[deno.land/x/net](https://github.com/denoland/net).

添加资源表以抽象各种类型的I/O流和其他分配状态.资源是映射到某个Rust对象的整数标识符.它可以与各种操作一起使用,尤其是读和写.

### 2018.10.18 / v0.1.8 / Connecting to Tokio / Fleshing out APIs

大多数文件系统操作都实现了.实现了基本的TCP组网.基本stdio流暴露.并且暴露了许多随机OS设施(例如,环境变量)

Tokio被选择为支持事件循环库.将JS承诺仔细映射到Rust Futures上,保留了错误处理和在主线程中同步执行的能力.

增加了连续的基准:<https://denoland.github.io/deno/>性能问题开始得到解决.

"deno--."被添加到引用运行时API.

朝着<https://github.com/denoland/deno/milestone/2>我们预计v0.2将在去年10月或11月初发布.

### 2018.09.09 / v0.1.3 / Scale binding infrastructure

ETA诉0.2 2018年10月<https://github.com/denoland/deno/milestone/2>

我们决定使用东京.<https://tokio.rs/>提供异步I/O、线程池执行以及作为各种互联网协议(如HTTP)的高级支持的基础.Tokio是围绕Future这个概念而设计的,它很好地映射到了JavaScript的承诺上.我们希望尽可能容易地从JavaScript开始Tokio的未来,并获得处理它的承诺.我们希望这将导致初步的文件系统操作,即http.此外,我们正在致力于CI、发布和基准测试基础架构以扩展开发.

### 2018.08.23 / v0.1.0 / Rust rewrite / V8 snapshot

<https://github.com/denoland/deno/commit/68d388229ea6ada339d68eb3d67feaff7a31ca97>

完成!<https://github.com/denoland/deno/milestone/1>

Go是一种垃圾收集语言,我们担心将它与V8的GC结合将导致以后的困难争用问题.

V8Works2绑定/概念正在移植到一个新的C++库中,称为LIbDNO.libdeno将包括整个JS运行时作为V8快照.它仍然遵循消息传递范例.Rust将被绑定到这个库以实现deno的特权部分.有关详细信息,请参阅deno2/README.md.

V8快照允许deno避免在启动时重新编译TypeScript编译器.这已经起作用了.

当重写与Go原型的特性相当时,我们将发布二进制文件供人们尝试.

### 2018.09.32 / v0.0.0 / Golang Prototype / JSConf talk

<https://github.com/denoland/deno/tree/golang>

<https://www.youtube.com/watch?v=M3BM9TB-8yA>

<https://tinyclouds.org/jsconf2018.pdf>

### 2007-2017 / Prehistory

<https://github.com/ry/v8worker>

<https://libuv.org/>

<https://tinyclouds.org/iocp-links.html>

<https://nodejs.org/>

<https://github.com/nodejs/http-parser>

<https://tinyclouds.org/libebb/>
