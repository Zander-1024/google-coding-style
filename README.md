# Google Coding Style

一个非官方的 Google 编码风格 skill：一个 `SKILL.md` 入口，按语言直接读取一份完整 Markdown。规范正文保留英文原文，入口与说明使用中文。资料没有经过模型摘要压缩，不需要运行脚本、安装依赖或联网才能阅读已内置规则。

## 使用

在支持 `SKILL.md` 的 agent 中导入本仓库根目录。以 Codex 常用个人目录为例，可自行执行以下命令（目标目录应尚不存在）：

```sh
git clone https://github.com/Zander-1024/google-coding-style.git "${CODEX_HOME:-$HOME/.codex}/skills/google-coding-style"
```

然后在任务中使用，例如：

```text
使用 $google-coding-style 检查这次 Python 修改；优先遵从项目已有约定。
```

入口直接列出语言与文件映射。只读取当前任务所需语言；来源链接是核实资料的依据，不是使用时必经的跳转。使用本项目时，用户指示及目标项目已有规范优先。

## 内置范围

共 19 份平级参考文件，包含编程语言、标记格式与文档指南；这不是“19 种编程语言”的声明。

| 语言或格式 | 参考文件 | 适用范围或限制 |
| --- | --- | --- |
| C++ | [cpp.md](references/cpp.md) | 完整上游指南 |
| C# | [csharp.md](references/csharp.md) | 完整上游指南 |
| Common Lisp | [common-lisp.md](references/common-lisp.md) | 风格指南与 Effective Programming 补充合并 |
| Go | [go.md](references/go.md) | 4 份资料合并，保留各自权威级别 |
| Haskell | [haskell.md](references/haskell.md) | 完整上游指南 |
| HTML / CSS | [html-css.md](references/html-css.md) | 亦涉及 Sass / GSS，没有额外独立指南 |
| Java | [java.md](references/java.md) | 完整上游指南 |
| JavaScript | [javascript.md](references/javascript.md) | 上游已停止更新，原文推荐迁移 TypeScript |
| JSON | [json.md](references/json.md) | JSON API 风格，不是 JSONC 规范 |
| Markdown | [markdown.md](references/markdown.md) | 5 份文档写作资料合并 |
| Objective-C | [objective-c.md](references/objective-c.md) | 完整上游指南 |
| Python | [python.md](references/python.md) | 完整上游指南 |
| R | [r.md](references/r.md) | 仅 Google 相对 Tidyverse 的差异；未包含完整 Tidyverse 基线 |
| Rust | [rust.md](references/rust.md) | 保留 Google 代码库范围及公共 Rust 指南依赖说明 |
| Shell / Bash | [shell.md](references/shell.md) | 主要针对 Bash，不代表所有 shell |
| TypeScript | [typescript.md](references/typescript.md) | 完整上游指南及其适用环境说明 |
| Verilog / SystemVerilog | [verilog.md](references/verilog.md) | SystemVerilog compliant 的可综合代码与 testbench |
| Vimscript | [vimscript.md](references/vimscript.md) | 简版及更严格的深入指南合并 |
| XML | [xml.md](references/xml.md) | XML 文档格式设计，不是通用 XML 自动格式化器 |

Swift、Angular、Effective Dart、Kotlin 由上游列出外部链接，未包含在本 skill 中：

- [Swift](https://google.github.io/swift/)
- [Angular](https://angular.dev/style-guide)
- [Effective Dart](https://dart.dev/effective-dart)
- [Kotlin](https://developer.android.com/kotlin/style-guide)

没有从编辑器配置推断独立的 C 语言支持，也没有打包上游编辑器工具、格式化器或配置文件。指南内引用的外部标准、库文档及 Google 内部页面也不代表这些内容已被内置。

## 来源与整理方式

来源是 [google/styleguide](https://github.com/google/styleguide) 及其[官方网站](https://google.github.io/styleguide/)。当前固定上游版本：

```text
572c673d9058242c049fbc8248b17029726c7afa
```

对应 `gh-pages` 分支，提交时间为 2026-10-28 UTC。每份参考文件末尾列出原始文件、固定 commit 链接、官网地址、转换说明与许可证。

HTML 转为 Markdown；Markdown 清理站点模板与展示标签。规则、例外、理由、代码和正反例含义保留。同语言多份资料合为一份，保留原文层级与适用范围。格式整理不代表 Google 更新了原规则，也不将建议提升为强制规范。原文外链保留用于核实来源；内部交叉引用尽量指向本地同一文件或对应语言文件。图片示例保留原图链接及文字说明。

后续更新直接比对新旧上游版本，重新整理受影响的语言文件并检查完整性，同时更新本节及文件末尾的版本。仓库没有运行时更新机制、manifest 或脚本依赖。

本项目由独立维护者整理，与 Google 无隶属关系，也不代表 Google 对本 skill 的认可或背书。

## 许可证

- 新写的 `SKILL.md`、本 README 与项目包装采用 [MIT](LICENSE)。
- 参考文档及其格式转换保留上游的 [CC BY 3.0](LICENSES/CC-BY-3.0.txt) 许可，不由 MIT 重新许可。
- 上游代码示例保留 [Apache 2.0](LICENSES/Apache-2.0.txt) 许可。
- R 原文另外署名 Hadley Wickham、引用 Tidyverse，并链接 [CC BY-SA 2.0](https://creativecommons.org/licenses/by-sa/2.0/)。这些特定声明保留在 R 参考文件中，不能用仓库总体声明或 MIT 覆盖。

原文中已有的作者、版权及其他特定声明继续有效。各参考文件末尾明确标记格式转换与合并等改动。
