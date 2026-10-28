---
name: google-coding-style
description: Apply Google's published coding style guides when writing, reviewing, or refactoring supported languages and Google style is requested or selected for the project. Load only the relevant language's Markdown reference; long guides are editorially condensed.
---

# Google 编码风格

先遵从用户指示与目标项目已有规范，再用对应 Google 指南补充未约定的风格。围绕当前任务应用规则，不因加载本 skill 扩大修改范围。

识别本次涉及的语言，直接读取下表对应文件；每项只有一份 Markdown。长指南为编辑浓缩版，保留可执行规则、约束程度与关键例外；短指南保留原文整理。只加载相关语言，无需执行脚本或默认访问原文链接。读取时确认正文没有被工具输出上限截断。

| 语言或格式 | 参考文件 | 范围说明 |
| --- | --- | --- |
| C++ | [cpp.md](references/cpp.md) | C++ 指南 |
| C# | [csharp.md](references/csharp.md) | C# 指南 |
| Common Lisp | [common-lisp.md](references/common-lisp.md) | 风格指南与 Effective Programming 补充 |
| Go | [go.md](references/go.md) | Overview、Guide、Decisions、Best Practices 合并 |
| Haskell | [haskell.md](references/haskell.md) | Haskell 指南 |
| HTML / CSS | [html-css.md](references/html-css.md) | 同一指南，亦涉及 Sass / GSS |
| Java | [java.md](references/java.md) | Java 指南 |
| JavaScript | [javascript.md](references/javascript.md) | 上游已停止更新的历史指南 |
| JSON | [json.md](references/json.md) | Google JSON API 风格；不是 JSONC 指南 |
| Markdown | [markdown.md](references/markdown.md) | 风格、文档实践与 README 指导合并 |
| Objective-C | [objective-c.md](references/objective-c.md) | Objective-C 指南 |
| Python | [python.md](references/python.md) | Python 指南 |
| R | [r.md](references/r.md) | Google 相对 Tidyverse 的差异；外部基线未内置 |
| Rust | [rust.md](references/rust.md) | 保留适用代码库与外部基线说明 |
| Shell / Bash | [shell.md](references/shell.md) | 主要针对 Bash |
| TypeScript | [typescript.md](references/typescript.md) | TypeScript 指南 |
| Verilog / SystemVerilog | [verilog.md](references/verilog.md) | SystemVerilog compliant 的设计和 testbench |
| Vimscript | [vimscript.md](references/vimscript.md) | 简版与更严格的深入指南合并 |
| XML | [xml.md](references/xml.md) | XML 文档格式设计指南 |

按原文区分必须、建议、可选项及例外；代码示例中标为错误或反例的内容只用于解释。Go 的 Guide 是 canonical 且 normative，Decisions 是 normative 但非 canonical，Best Practices 是非规范建议；Vimscript 深入指南的额外严格要求按其原定受众理解。合并资料中的补充内容不自动提升为主指南的硬性规则。

Swift、Angular、Dart、Kotlin 仅有上游外链，尚未内置。其他语言也应明确说明缺少参考；只有确需额外资料时，才按当前任务另行查证。各文件末尾保留来源、固定版本和许可证。
