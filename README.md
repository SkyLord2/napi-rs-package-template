# 项目说明：napi-rs Node 原生扩展包模板

> 本项目用于快速创建“Rust 实现核心逻辑 + Node.js/TypeScript 暴露 API”的 N-API 原生扩展包。模板基于 napi-rs，默认集成构建、测试、CI 与多平台预编译发布流程，适合发布到 npm 并在各平台免编译安装。

![CI](https://github.com/napi-rs/package-template/workflows/CI/badge.svg)

## 目标与适用场景

- **性能敏感逻辑 Rust 化**：CPU 密集型计算、解析、压缩/加密等放在 Rust 层，JS 层只做编排。
- **稳定 ABI**：使用 Node-API（N-API）确保跨 Node 版本 ABI 兼容，减少“换 Node 版本就坏”的风险。
- **多平台分发**：通过为不同平台发布独立 npm 包，并在主包中使用 optionalDependencies 自动选择对应二进制，避免 postinstall 运行时下载带来的网络与依赖问题。

## 项目结构（关键文件）

- [src/lib.rs](./src/lib.rs)：Rust 原生扩展入口，使用 `#[napi]` 宏导出给 JS/TS。
- `npm/`：多平台发布相关的子包配置（不同平台/架构的包）。
- `index.js` / `index.d.ts`（或生成产物）：JS 入口与 TS 类型定义（以项目实际文件为准）。

## 快速开始（使用模板）

1. 在 GitHub 点击 **Use this template** 创建新仓库并克隆到本地
2. 安装依赖

```bash
pnpm install
```

3. 重命名包名与二进制名（建议立即执行）

```bash
pnpm napi rename -n @your-scope/your-package -b your_binary_name
```

说明：

- `-n` 指定 npm 包名（支持 scope）
- `-b` 指定生成的原生二进制基名（影响 `.node` 文件名与各平台子包命名）

## 构建产物说明

执行构建后：

```bash
pnpm build
```

你会在项目根目录看到类似以下的原生二进制文件（名称随 `-b` 变化）：

- `your_binary_name.darwin.node`
- `your_binary_name.win32.node`
- `your_binary_name.linux.node`

这些文件由 Rust 源码 [src/lib.rs](./src/lib.rs) 编译生成，供 Node 在运行时 `require()` 加载。

## 本地开发与调试

### 环境要求

- Rust：最新稳定版（建议 `rustup update`）
- Node.js：建议 LTS（模板 CI 默认覆盖 node@20 / node@22）
- pnpm：用于依赖管理（版本以你当前项目锁定为准）

### 常用命令

```bash
pnpm build
pnpm test
```

模板默认使用 [ava](https://github.com/avajs/ava) 进行测试。测试成功时会看到类似输出：

```bash
$ ava --verbose

  ✔ sync function from native code
  ✔ sleep function from native code (201ms)
  ─

  2 tests passed
```

## API 设计与实现建议（Rust / TS 边界）

### Rust 侧（导出给 Node）

- 使用 `#[napi]` 标记导出函数/结构体；对外错误建议返回 `napi::Result<T>` 并用 `?` 传递错误。
- CPU 密集型任务不要阻塞事件循环：优先使用 napi-rs 的异步任务机制（如 AsyncTask）将计算卸载到线程池。
- 传递大数据时优先使用 Buffer/切片，减少不必要的拷贝。

### TypeScript 侧（对外暴露体验）

- 尽量提供类型清晰、参数扁平化的 API；复杂配置使用对象参数并给出合理默认值。
- 将“平台差异”隐藏在内部：使用者只关心主包，实际加载由 optionalDependencies 选择正确的子包二进制。

## CI 与发布（多平台预编译）

### CI

模板默认在 GitHub Actions 中对以下矩阵进行构建与测试：

- Node：`20` / `22`
- OS：macOS / Linux / Windows

### 发布流程

1. 在 GitHub 仓库配置 `NPM_TOKEN`：
   - `Settings -> Secrets and variables -> Actions -> New repository secret`
   - Name：`NPM_TOKEN`
2. 发版本并推送 tag/提交（以项目的 release 工作流为准）：

```bash
npm version [<newversion> | major | minor | patch | premajor | preminor | prepatch | prerelease [--preid=<id>] | from-git]
git push
```

GitHub Actions 会自动构建各平台产物并发布到 npm。

> 注意：不要手动 `npm publish`，以免破坏多平台包发布的一致性与依赖关系。

## 试用安装（模板包本身）

如果你只是想验证模板机制，也可以安装示例包：

```bash
pnpm add @napi-rs/package-template
```
