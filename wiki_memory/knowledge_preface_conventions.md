---
name: knowledge_preface_conventions
 description: AXI/ACE5 规范前言中的排版、时序图、信号和数字表示约定
---

# 知识点摘要

- 规范用斜体强调重要注释、特殊术语、内部交叉引用和引用；粗体表示信号名；等宽字体用于汇编语法、伪代码和源代码示例；小型大写用于具有特定技术含义的术语。
- 时序图中阴影总线/信号区域表示未定义，阴影期间可以取任意值且不影响正常操作；图中未明确说明的时序信息不能自行推断。
- 信号名首尾的小写 `n` 表示 active-Low；信号名第二个字母为小写 `x` 时，表示读写共用的集合术语，例如 `AXCACHE` 同时指 `ARCACHE` 和 `AWCACHE`。
- 数字默认使用十进制；二进制使用 `0b` 前缀，十六进制使用 `0x` 前缀，均以等宽字体表示。

# 关键细节

- 全局时钟/信号的具体定义应以协议章节为准；本部分只规定阅读规范时的符号和图示解释。
- 对单比特信号使用 HIGH/LOW 的时序图，不能把与描述无关的伴随信号值当成额外协议约束。

# 原文引用

- 文档：`/home/yuanxi/yuanxi_cc/wiki_files/amba_prot/AXIACE5.pdf`
- 版本/日期：文档页脚显示 ARM IHI 0022H.c，Copyright 2003–2021。
- 位置：PDF 物理页 19–21 附近；印刷页 Preface 的 `Conventions`、`Signals`、`Numbers`。
- 依据：前言明确列出排版约定、时序图约定、active-Low 标记、读写共用信号命名和二进制/十六进制前缀。

# 适用条件与例外

- 这些规则是阅读本规范的表示约定，不是 AXI 事务本身的握手或时序要求。
- 本批次读取范围为 PDF 物理页 19–38；页码同时记录了 PDF 物理页和文档印刷页，二者不可混用。

# 关联章节

- Chapter A1 Introduction
- Chapter A2 Signal Descriptions
- A3 Basic transactions

# 待核验问题

- 无
