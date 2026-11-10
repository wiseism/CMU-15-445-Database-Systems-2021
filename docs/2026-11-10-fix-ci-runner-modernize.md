# 2026-11-10 CMake Build Matrix 全链路修复

## 现象

`CMake Build Matrix` workflow Ubuntu GCC/Clang job 失败（checkout 步骤附近 `Process completed with exit code 1`），`macOS Latest Clang` job 被 fail-fast 取消。

## 根因

workflow 是 2021 年 BusTub 课程模板，早已不适配当前 runner：

- `actions/checkout@v2`（node12 runtime 已废弃）；
- `macos-10.15` 不是可用 runner，依赖 `llvm@8` 的 brew 路径也早已失效；
- `ubuntu-latest` 于 2026-10 迁移至 Ubuntu 26.04，apt 源里没有 `clang-8/clang-format-8/clang-tidy-8`；
- `build_support/packages.sh` 仅支持 Ubuntu 18.04/20.04。

## 修复（.github/workflows/cmake.yml 重写）

- `checkout@v4`；`fail-fast: false`；
- matrix 只保留 ubuntu-24.04（GCC + Clang）双工具链；删除失效的 macOS 10.15 配置；
- 依赖安装不再走 18/20.04-only 的 packages.sh，改用 apt 直接安装 `clang/clang-format/clang-tidy/cmake/zlib1g-dev`；
- Configure 时 `CC/CXX` 指向 runner 自带编译器，`CLANG_FORMAT_BIN/CLANG_TIDY_BIN` 指向系统 clang-format/clang-tidy（CMakeLists 找不到时仅告警，不影响构建）；
- 保留完整 Configure + Build（含 gtest 测试目标构建）。
