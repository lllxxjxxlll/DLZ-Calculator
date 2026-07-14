# DLZ Calculator

<p align="center">
  <b>一款高性能科学计算器，支持自定义函数、符号求导、数值积分与方程求解</b>
</p>

DLZ Calculator 是基于 [Kalker](https://github.com/PaddiM8/kalker) 二次开发的科学计算器。它使用 Rust 编写，通过 WebAssembly 技术支持在浏览器中运行，具备完整的数学表达式解析与数值计算能力。

---

## 目录

- [核心架构](#核心架构)
- [功能特性](#功能特性)
- [快速开始](#快速开始)
- [使用示例](#使用示例)
- [项目结构](#项目结构)
- [相关库](#相关库)
- [贡献指南](#贡献指南)

---

## 核心架构

DLZ Calculator 采用经典的编译前端管道架构处理数学表达式：

```
输入字符串
    │
    ▼
  Lexer（词法分析） → Token 流
    │
    ▼
  Parser（语法分析） → 抽象语法树（AST）
    │
    ▼
  Analysis（语义分析） → 标注后的 AST
    │
    ▼
  Interpreter（解释执行） → 计算结果
```

- **Lexer**：手写状态机，支持 Unicode 数学符号识别、科学记数法、多进制前缀（`0b`/`0o`/`0x`）及下标进制标记
- **Parser**：递归下降解析器，按运算符优先级逐层构建 AST，支持隐式乘法、分段函数和向量推导式
- **Interpreter**：树遍历求值，使用函数式风格管理复数/向量/矩阵运算，调用前暂存参数/调用后恢复以支持递归
- **数值方法**：tanh-sinh 双指数求积法（积分）、Newton-Raphson 迭代（单变量/方程组求根）、二次方程求根公式（精确解析解）

---

## 功能特性

| 类别 | 说明 | 示例 |
|------|------|------|
| 基本运算 | `+` `-` `*` `/` `^` `!` `%` `>>` `<<` | `2^10 = 1024` |
| 分组符号 | `()` 分组, `[]` 向量/矩阵, `⌈⌉` ceil, `⌊⌋` floor, `\|` abs | `| -5 | = 5` |
| 向量 | `(a, b, c)` 形式，支持逐元素运算与点积 | `(1,2,3) * 2` |
| 矩阵 | `[a, b; c, d]` 形式，支持加减法和矩阵乘法 | `[1,2;3,4] * [5;6]` |
| 预定义函数 | 三角函数、反三角、双曲、指数对数等 | `sin(pi/2)`、`log(e)` |
| 自定义函数 | 单行定义，支持递归调用 | `f(x) = x^2 + 2x + 1` |
| 方程求根 | Newton-Raphson 迭代 + 二次方程解析求解 | `x^2 - 4x + 3 = 0` 得到两个根 |
| 方程组 | 多维 Newton-Raphson 数值求解 | `{x + y = 5; x - y = 1}` |
| 数值微分 | 中心差分法，支持高阶导数 | `f'(2)`、`sin''(pi)` |
| 数值积分 | tanh-sinh 双指数求积法 | `∫(0, pi, sin(x) dx)` |
| 求和/求积 | 循环迭代计算 | `sum(1, 100, n^2)` |
| 多进制 | 前缀或下标标记 | `0b1101`、`11₂` |
| 分段函数 | `{ expr if cond; otherwise }` 语法 | `f(x) = { 0 if x<0; x otherwise }` |
| ans 变量 | 引用上一次计算结果 | `ans * 2` |
| 模糊语法 | 支持隐式乘法、省略括号 | `2xy` → `2*x*y` |

> **对数函数说明**：`log(x)` 和 `ln(x)` 均为自然对数；`log10(x)` 为以 10 为底的常用对数；二元形式 `log(x, b)` 表示以 b 为底的对数。

---

## 快速开始

### 前置要求

- Rust 工具链（最低版本 1.70.0）
- 如需高精度模式：`diffutils`、`gcc`、`make`、`m4`（Windows 用户需在 MSYS2 中安装 `mingw-w64-x86_64-rust`）

### 从源码编译

```bash
git clone https://github.com/lllxxjxxlll/DLZ-Calculator.git
cd DLZ-calculator
cargo build --release
```

编译完成后，可执行文件位于 `target/release/dlz-calculator.exe`（Windows）或 `target/release/dlz-calculator`（macOS/Linux）。

### 使用 Cargo 运行

```bash
cargo run --release
```

### Web 版

Web 版通过 WebAssembly 在浏览器中运行：

```bash
cd kalk
wasm-pack build --target web -- --no-default-features
# 将 pkg/ 目录中的文件复制到 web 服务器的静态目录
```

---

## 使用示例

```
> 1 + 2 * 3
= 7

> sin(pi/2)
= 1

> f(x) = x^2 + 2x + 1
> f(3)
= 16

> x^2 - 4x + 3 = 0
x = 3
x = 1

> x^2 - 6x + 7 = 0
x = 4.4142135624
x = 1.5857864376

> x^2 + 2x + 5 = 0
x = -1 + 2i
x = -1 - 2i

> {x + y = 5; x - y = 1}
(x, y) = 3
(x, y) = 2

> integral(0, pi, sin(x), dx)
= 2

> log(e)
= 1

> log10(100)
= 2

> 0b1101
= 13

> [1, 2; 3, 4] * [5, 6; 7, 8]
[19, 22
 43, 50]
```

---

## 项目结构

```
kalker/
├── cli/                 # 命令行界面（基于 rustyline 的交互式 REPL）
├── kalk/                # 核心计算库
│   └── src/
│       ├── ast.rs            # 抽象语法树定义
│       ├── lexer.rs          # 词法分析器
│       ├── parser.rs         # 递归下降语法分析器
│       ├── interpreter.rs    # 树遍历解释器
│       ├── numerical.rs      # 数值方法（求导/积分/求根/方程组）
│       ├── kalk_value/       # 核心数值类型（实数/虚数/单位）
│       ├── prelude/          # 内置函数库与数学常量
│       ├── symbol_table.rs   # 符号表（变量/函数/单位管理）
│       ├── analysis.rs       # 语义分析（隐式乘法拆分等）
│       └── errors.rs         # 错误类型定义
├── web/                 # Web 前端
└── tests/               # 集成测试
```

---

## 相关库

| 名称 | 说明 |
|------|------|
| [kalk](https://crates.io/crates/kalk) | Rust crate，核心计算引擎 |
| [@paddim8/kalk](https://www.npmjs.com/package/@paddim8/kalk) | JavaScript 绑定（WebAssembly） |
| [@paddim8/kalk-component](https://www.npmjs.com/package/@paddim8/kalk-component) | 命令行风格的 Web 组件 |

---

## 贡献指南

1. 修改核心计算库（`kalk/`）后，在项目根目录运行 `cargo run` 即可使用 CLI 测试改动
2. 所有 Rust 代码请使用 `rustfmt` 格式化
3. 提交前请确保 `cargo test` 通过

### Web 版开发

```bash
cd kalk
wasm-pack build --target web -- --no-default-features
# 产物位于 pkg/ 目录，将其复制到 web 项目中即可
```

---

## 致谢

本项目在原 [Kalker](https://github.com/PaddiM8/kalker) 的基础上进行了以下增强：

- 方程求根支持多个初始猜测，可收敛到不同根
- 新增二次方程解析求解（含复数根输出）
- 修复 `log`/`log10`/`ln` 的语义一致性
- 优化词法分析器以支持 `log10` 等复合标识符
- 改进数值结果的等于号/约等于号显示逻辑

---

## 许可证

本项目继承原 Kalker 项目的 [MIT 许可证](LICENSE)。