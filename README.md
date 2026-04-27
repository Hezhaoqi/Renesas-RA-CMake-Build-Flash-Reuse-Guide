# Renesas RA + CMake Build + Flash Reuse Guide
# (Renesas RA + CMake 可复用编译 + 下载模块

这份模板面向 **Renesas RA + CMake** 工程，目标是让你在任意同类工程里快速复用「一键编译 + 一键烧录」流程。

核心思路：

- 用 CMake 做唯一构建链路
- 用 J-Link 做下载
- 在任务里自动处理 RA 工程常见的两个坑（`bsp_linker_info.h` 与 `fsp.ld` include）

## 1) 你需要复制什么

复制以下文件到你的工程：

- `.vscode/tasks.json`
- `README_VSCODE_BUILD_FLASH.md`（可选，用作团队说明文档）

不需要复制 `jlink_flash.jlink`。模板会在运行时自动生成 `.vscode/jlink_flash.auto.jlink`。

## 2) 新工程需要满足什么

工程至少包含：
(创建项目时"IDE Project Type"要选择"e2 Studio CMake")
("Toolchains" 要选择 "GNU ARM Embedded")

- `cmake/gcc.cmake`
- 顶层 `CMakeLists.txt`
- `script/fsp.ld`
- RA 标准目录（`ra/`、`ra_gen/`、`ra_cfg/`、`src/`）

默认 ELF 路径按 `${workspaceFolder}/build/Debug/${workspaceFolderBasename}.elf` 计算。  
如果你的 ELF 名称不同，只改 `tasks.json` 里 `JLink: Flash ELF` 任务中的 `$elf=...` 一行。

## 3) 两个一键任务怎么用

- `First Time: Full Setup + Rebuild + Flash`
  - 用于：首次接入、换电脑、换工具链、改 `configuration.xml`、大版本切换后
  - 内部步骤：`Configure -> 准备头文件 -> 修正链接脚本 -> Rebuild -> Flash`

- `Daily: Build + Flash (Fast)`
  - 用于：日常改代码
  - 内部步骤：`Build -> Flash`（更快）

## 4) 迁移到另一台电脑要改哪些

通常只要改 `tasks.json` 里 `JLink: Flash ELF` 任务的这几项：

- `JLink.exe` 路径
- `-device`（芯片型号）
- `-if`（`SWD`/`JTAG`）

如果该电脑的 `make.exe` 路径不同，再改 `CMake: Configure (Debug)` 里的：

- `-DCMAKE_MAKE_PROGRAM=...`

## 5) 迁移到另一个芯片/项目要改哪些

最常见只改一处：

- `JLink: Flash ELF` 里的 `-device`

可能还要改：

- `-if`（接口类型）
- `$elf` 路径（当 ELF 文件名不等于工程目录名时）

其余构建任务通常可直接复用。

## 6) 常见问题速查

- `JLink.exe ... CommandNotFoundException`
  - 检查 `JLink: Flash ELF` 里的可执行文件路径。

- `cannot open linker script file memory_regions.ld`
  - 先跑 `First Time`（会自动修正 `script/fsp.ld` 的 `INCLUDE` 路径）。

- `undefined reference to g_init_info`
  - 先跑 `First Time`（会自动生成 `ra_gen/bsp_linker_info.h` 转发头）。

- `_read/_write/_close is not implemented`
  - `nosys` 常见 warning，一般不影响生成和下载。

## 7) 推荐团队使用方式

- 首次编译：用 `First Time`
- 日常开发：用 `Daily`
- 遇到奇怪构建问题：用 `First Time`

这样可以最大限度减少环境差异导致的问题。

