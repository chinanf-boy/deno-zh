# denoland/deno [![explain]][source] [![translate-svg]][translate-list]

<!-- [![size-img]][size] -->

[explain]: http://llever.com/explain.svg
[source]: https://github.com/chinanf-boy/Source-Explain
[translate-svg]: http://llever.com/translate.svg
[translate-list]: https://github.com/chinanf-boy/chinese-translate-list
[size-img]: https://packagephobia.now.sh/badge?p=Name
[size]: https://packagephobia.now.sh/result?p=Name

「 desc 」

[中文](./readme.md) | [english](https://github.com/denoland/deno)

---

## 校对 🀄️

<!-- doc-templite START generated -->
<!-- repo = 'denoland/deno' -->
<!-- commit = 'c2663e1d82521e9b68a7e2e96197030a4ee00c30' -->
<!-- time = '2018-09-10' -->

| 翻译的原文 | 与日期        | 最新更新 | 更多                       |
| ---------- | ------------- | -------- | -------------------------- |
| [commit]   | ⏰ 2018-09-10 | ![last]  | [中文翻译][translate-list] |

[last]: https://img.shields.io/github/last-commit/denoland/deno.svg
[commit]: https://github.com/denoland/deno/tree/c2663e1d82521e9b68a7e2e96197030a4ee00c30

<!-- doc-templite END generated -->

- [ ] [README.md](README.md)
- [ ] [Docs.md](Docs.md)
- [ ] [Roadmap.md](Roadmap.md)
- [ ] [tests/README.md](tests/README.md)
- [ ] [tools/ts_library_builder/README.md](tools/ts_library_builder/README.md)
- [ ] [website/README.md](website/README.md)


### 贡献

欢迎 👏 勘误/校对/更新贡献 😊 [具体贡献请看](https://github.com/chinanf-boy/chinese-translate-list#贡献)

## 生活

[help me live , live need money 💰](https://github.com/chinanf-boy/live-need-money)

---

# deno

|                                              **Linux&Mac**                                              |                                                                   **窗户**                                                                    |
| :-----------------------------------------------------------------------------------------------------: | :-------------------------------------------------------------------------------------------------------------------------------------------: |
| [![Travis](https://travis-ci.com/denoland/deno.svg?branch=master)](https://travis-ci.com/denoland/deno) | [![Appveyor](https://ci.appveyor.com/api/projects/status/yel7wtcqwoy0to8x?branch=master&svg=true)](https://ci.appveyor.com/project/deno/deno) |


### 目录

<!-- START doctoc -->
<!-- END doctoc -->


## 构建在 V8 上的安全 TypeScript 运行时

- 支持 TypeScript 3.0.1.使用 V8 6.9.297.也就是说,它是非常现代的 JavaScript.

- 不`package.json`. 没有 NPM.与 Node 不显式兼容.

- 只导入参考源代码 URL.

  ````
  	```
  ````

  从"导入{测试}"<https://unpkg.com/deno_testing@0.0.5/testing.ts>"import{log}from./util.ts"

  ````
  	```
  ````

  在第一次执行时,获取并缓存远程代码,并且在使用`--reload`旗帜.(所以,这在飞机上仍然有效.)见`~/.deno/src`有关缓存的详细信息.)

- 可以控制文件系统和网络访问,以便运行沙箱代码.默认为只读文件系统访问且没有网络访问.V8(非特权)和 Rust(特权)之间的访问仅通过以下定义的序列化消息完成[flatbuffer](https://github.com/denoland/deno/blob/master/src/msg.fbs). 这使得审计变得容易.要显式启用写访问,请使用`--allow-write`和`--allow-net`用于网络访问.

- 单个可执行文件:

  ````
  	```
  ````

  > ls-lh out/./deno-rwxr-xr-x 1 rld 职员 48M Aug 2 13:24 out/./deno otool-L out/./deno out/./deno out/deno:/usr/lib/libSystem.B.dylilib(兼容性版本 1.0.0,当前版本 1252.50.4)/usr/lib/libresolv.9.dylib(兼容性版本 1.0.0,当前版本 1.0.0)/System/Library/Frameworks/Security.framework/Ver./A/Security(兼容性版本 1.0.0,当前版本 58286.51.6)/usr/lib/libc++.1.dylib(兼容性版本 1.0.0,当前版本 400.9.0)
  >
  > ````
  > 	```
  > ````

- 总是死于未发现的错误.

- 支持顶级`await`.

- 目标是与浏览器兼容.

## 安装

```
curl -sSf https://raw.githubusercontent.com/denoland/deno_install/master/install.py | python
```

## 状态

正在开发中.

我们发布二进制版本[here](https://github.com/denoland/deno/releases).

对未来版本的进展进行跟踪[here](https://github.com/denoland/deno/milestones).

路线图是[here](https://github.com/denoland/deno/blob/master/Roadmap.md). 也看到[this presentation](http://tinyclouds.org/jsconf2018.pdf).

[Chat room](https://gitter.im/denolife/Lobby).

## 构建指令

为了确保可再现的构建,Deno 的大部分依赖项都在 git 子模块中.但是,您需要单独安装:

1.  [Rust](https://www.rust-lang.org/en-US/install.html)
2.  [Node](http://nodejs.org/)
3.  Python 2.[Not 3](https://github.com/denoland/deno/issues/464#issuecomment-411795578).
4.  [ccache](https://developer.mozilla.org/en-US/docs/Mozilla/Developer_guide/Build_Instructions/ccache)(可选的,但对加速 V8 的重建很有帮助.)

建造:

```
# Fetch deps.
git clone --recurse-submodules https://github.com/denoland/deno.git
cd deno
./tools/setup.py

# Build.
./tools/build.py

# Run.
./out/debug/deno tests/002_hello.ts

# Test.
./tools/test.py

# Format code.
./tools/format.py
```

其他有用的命令:

```
# Call ninja manually.
./third_party/depot_tools/ninja -C out/debug
# Build a release binary.
DENO_BUILD_MODE=release ./tools/build.py :deno
# List executable targets.
./third_party/depot_tools/gn ls out/debug //:* --as=output --type=executable
# List build configuation.
./third_party/depot_tools/gn args out/debug/ --list
# Edit build configuration.
./third_party/depot_tools/gn args out/debug/
# Describe a target.
./third_party/depot_tools/gn desc out/debug/ :deno
./third_party/depot_tools/gn help
```

EVV VARS:`DENO_BUILD_MODE`,`DENO_BUILD_PATH`,`DENO_BUILD_ARGS`.

## 贡献

1.  叉子[this repository](https://github.com/denoland/deno)从`master`.
2.  找你的零钱.
3.  确保`./tools/test.py`传球.
4.  使用`./tools/format.py`.
5.  确保`./tools/lint.py`传球.
6.  发送拉动请求.
7.  签署[CLA](https://cla-assistant.io/denoland/deno)如果你还没有.
