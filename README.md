# Google Coding Style

一个非官方的 Google 编码风格 skill：一个 `SKILL.md` 入口，每种语言直接读取一份 Markdown。长指南采用英文编辑浓缩版，短指南保留原文整理；入口与说明使用中文。无需运行脚本、安装依赖或联网即可使用内置规则。

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

表中为 `o200k_base` 全文实测，含来源及许可尾注；修订前以 [a06f1ece](https://github.com/Zander-1024/google-coding-style/commit/a06f1eceafdc0e65c385d30334b60d4cda36edc7) 的完整整理版本为基线。

| 语言或格式 | 参考文件 | 适用范围或限制 | 修订前 tokens | 当前 tokens |
| --- | --- | --- | ---: | ---: |
| C++ | [cpp.md](references/cpp.md) | 编辑浓缩版 | 46,970 | 11,770 |
| C# | [csharp.md](references/csharp.md) | 完整上游指南 | 5,303 | 5,303 |
| Common Lisp | [common-lisp.md](references/common-lisp.md) | 风格指南与 Effective Programming 补充合并浓缩 | 24,979 | 11,512 |
| Go | [go.md](references/go.md) | 4 份资料合并浓缩，保留各自权威级别 | 70,330 | 11,736 |
| Haskell | [haskell.md](references/haskell.md) | 完整上游指南 | 6,297 | 6,297 |
| HTML / CSS | [html-css.md](references/html-css.md) | 亦涉及 Sass / GSS，没有额外独立指南 | 6,304 | 6,304 |
| Java | [java.md](references/java.md) | 完整上游指南 | 10,726 | 10,726 |
| JavaScript | [javascript.md](references/javascript.md) | 历史指南的编辑浓缩版；上游已停止更新，推荐迁移 TypeScript | 30,629 | 17,267 |
| JSON | [json.md](references/json.md) | JSON API 风格，不是 JSONC 规范 | 9,532 | 9,532 |
| Markdown | [markdown.md](references/markdown.md) | 5 份文档写作资料合并 | 8,380 | 8,380 |
| Objective-C | [objective-c.md](references/objective-c.md) | 完整上游指南 | 16,616 | 16,616 |
| Python | [python.md](references/python.md) | 编辑浓缩版 | 24,540 | 17,528 |
| R | [r.md](references/r.md) | 仅 Google 相对 Tidyverse 的差异；未包含完整 Tidyverse 基线 | 1,100 | 1,100 |
| Rust | [rust.md](references/rust.md) | 保留 Google 代码库范围及公共 Rust 指南依赖说明 | 13,139 | 13,139 |
| Shell / Bash | [shell.md](references/shell.md) | 主要针对 Bash，不代表所有 shell | 9,841 | 9,841 |
| TypeScript | [typescript.md](references/typescript.md) | 编辑浓缩版，保留其适用环境说明 | 26,294 | 12,415 |
| Verilog / SystemVerilog | [verilog.md](references/verilog.md) | SystemVerilog compliant 的可综合代码与 testbench；编辑浓缩版 | 39,637 | 16,173 |
| Vimscript | [vimscript.md](references/vimscript.md) | 简版及更严格的深入指南合并 | 9,124 | 9,124 |
| XML | [xml.md](references/xml.md) | XML 文档格式设计，不是通用 XML 自动格式化器 | 5,519 | 5,519 |

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

HTML 转为纯 Markdown，Markdown 清理站点模板与展示标签；同语言资料合并到同一文件。C++、Common Lisp、Go、JavaScript、Python、TypeScript、Verilog 采用编辑浓缩版，优先保留可执行规范、must / should / may 的约束程度、关键例外及必要正反例，删减重复解释、冗长背景和重复示例。其他较短资料保留原文整理，不为凑预算扩写。

浓缩文档不是完整原文或逐字副本。每份开头和末尾标明整理方式，保留署名、固定版本链接和许可证。原文链接用于可选核实详细理由与上下文，日常使用不默认跳转或加载额外资料。格式整理与编辑浓缩不代表 Google 修改了原规则，也不将建议提升为强制规范。图片示例保留原图链接及文字说明。

## 上下文预算

每份参考文档（含来源及许可尾注）以 `tiktoken 0.12.0 / o200k_base` 实测不超过 20,000 tokens；同时校验 `SKILL.md + 任一参考文档 <= 20,000 tokens`。本次将超限文档以 18,000 tokens 为目标，为入口留出空间。仍为单入口、19 份平级语言文件，不通过拆文件或增加阅读依赖规避预算。

本次测量：入口 **795** tokens；最大参考文档为 `python.md`，**17,528** tokens，入口加该文档为 **18,323**。19 份参考资料总计由 **365,260** 降至 **200,282** tokens（-45.17%）。额外以现有 `cl100k_base` 复测，入口加最大文档为 **18,436** tokens，也低于 20,000。

计数针对文件全文文本，不含消息和工具输出包装。同次多语言任务应按实际加载文件累加；其他模型或分词器的数量可能不同，这不是通用模型的绝对上下文上限。

后续更新直接比对新旧上游版本，重新整理受影响的语言文件并核对规则覆盖与 token 预算，同时更新本节及文件末尾的版本。仓库没有运行时更新机制、manifest 或脚本依赖。

本项目由独立维护者整理，与 Google 无隶属关系，也不代表 Google 对本 skill 的认可或背书。

## 许可证

- 新写的 `SKILL.md`、本 README 与项目包装采用 [MIT](LICENSE)。
- 参考文档及其格式转换、编辑浓缩保留上游的 [CC BY 3.0](LICENSES/CC-BY-3.0.txt) 许可，不由 MIT 重新许可。
- 上游代码示例保留 [Apache 2.0](LICENSES/Apache-2.0.txt) 许可。
- R 原文另外署名 Hadley Wickham、引用 Tidyverse，并链接 [CC BY-SA 2.0](https://creativecommons.org/licenses/by-sa/2.0/)。这些特定声明保留在 R 参考文件中，不能用仓库总体声明或 MIT 覆盖。

原文中已有的作者、版权及其他特定声明继续有效。各参考文件末尾明确标记格式转换、合并及编辑浓缩等改动。
