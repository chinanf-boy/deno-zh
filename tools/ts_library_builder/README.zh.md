# ts_library_builder

该工具允许我们生成单个 TypeScript 声明文件,该文件描述完整的 Deno 运行时,包括全局变量和内置变量`deno`模块.这个工具的输出,`lib.deno_runtime.d.ts`用于几个目的:

1.  它被传递给 TypeScript 编译器`js/compiler.ts`因此,TypeScript 知道需要什么类型,并且可以根据运行时环境验证代码.
2.  它通过以下方式输出到标准输出`deno --types`因此,用户可以很容易地访问完整的声明文件.编辑器将来可以使用它来执行类型检查.
3.  因为 JSDocs 是维护的,所以它充当 Deno 的简单文档页面.我们将来使用这个文件来生成 HTML 文档.

该工具依赖于两个库:

- [`ts-node`](https://www.npmjs.com/package/ts-node)为工具本身提供及时的类型脚本转换.
- [`ts-simple-ast`](https://www.npmjs.com/package/ts-simple-ast)它为 TypeScript AST 提供了更加合理和功能性的接口,使操作更加容易.
- [`prettier`](https://www.npmjs.com/package/prettier)和[`@types/prettier`](https://www.npmjs.com/package/@types/prettier)对输出进行格式化.

## Design

理想情况下,我们根本不需要构建这个工具,只需要使用`tsc`输出这个声明文件.虽然,`--emitDeclarationsOnly`,`--outFile`和`--module AMD`生成一个声明文件,它不干净.它从来不是为库生成而设计的,在运行时环境中可用的内容与创建该环境结构的代码显著不同.

因此,该工具注入了一些关于 Deno 运行时环境中发生的情况的知识,并确保输出文件对于最终用户来说更加干净和合理.在 deno 运行时中,代码在定义为`js/global.ts`. 这包含 JavaScript 运行时合理预期的全局范围项,如`console`. 它还定义了自省的全局范围.`window`变量.目前用户只能使用 Deno 特定 API 的一个模块.这是在`js/deno.ts`.

这个工具利用了 TypeScript 的一个实验特性,即并非真正打算成为公共 API 一部分的项用注释`@internal`然后在生成类型定义时不发出.此外,TypeScript 将*树摇*任何与"隐藏"API 相关的依赖项,并删除它们.这确实有助于保持公共 API 的整洁,并根据需要尽可能地最小化.

为了创建默认类型库,高层次的过程如下所示:

- 我们将所有运行时环境定义代码读入 TypeScript AST 解析器"project".
- 我们只将 TypeScript 类型定义发送到另一个 AST 解析器"project".
- 我们处理`deno`名称空间/模块,通过"扁平化"类型定义文件.
  - 我们为`js/deno.ts`.
  - 我们创建`gen/msg_generated.ts`它在构建过程中生成,并且包含与 deno 的特权部分和用户土地之间通信的平面缓冲区结构相关的类型信息.目前,该工具不能进行完全复杂的依赖关系分析以确定需要从该文件中提取什么,因此我们显式地提取我们需要的类型信息.
  - 我们遍历模块的所有导入/导出,只导出那些最终由`js/deno.ts`.
  - 我们将导入/导出替换为源文件的类型信息.
  - 此过程假设所有提供内容的模块`js/deno.ts`将具有没有名称冲突的公共类型 API.
- 我们处理`js/globals.ts`用于生成全局名称空间的文件.
  - 我们创建`Window`接口和`global`范围扩展命名空间.
  - 我们反复进行扩充,以`window`在文件中声明的变量,提取类型信息,并将其应用于全局变量声明和`Window`接口.
  - 我们标识模块中的任何类型别名并在全局声明它们.
- 我们将每个名称空间导入`js/globals.ts`我们解析发出的声明`.d.ts`文件并在全局范围内创建它作为自己的命名空间.仅仅平铺这些是不安全的,因为存在冲突的高风险,而且它还使得在生成的接口和变量声明中编写类型更加容易.
- 然后,我们验证所得到的定义文件,并将其写入适当的构建路径.

## TODO

- 工具没有*树摇*当进口趋于平缓时.这意味着,包含一些实际上并不需要的外部类型,这意味着`gen/msg_generated.ts`必须明确划分.
- 完成测试……我们有一些保险,但不是很多`ast_util_test`这正在被隐式地测试.
