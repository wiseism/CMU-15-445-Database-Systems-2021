# 2026-11-10 timestamp_type.cpp format-overflow 编译错误修复

## 现象

 workflow 现代化后（ubuntu-24.04 + GCC/Clang 13+/18），`BusTub` 构建在两个工具链同时失败：

```
src/type/timestamp_type.cpp:129:18: error: '%02d' directive writing between 2 and 10 bytes
   into a region of size 5 [-Werror=format-overflow=]
```

## 根因

`TimestampType::ToString` 中 `char zone[5]` 用于 `sprintf(zone, "%02d", tz)`；新版 GCC 的
`-Wformat-overflow` 判定区域过小（未来 tz 取值范围扩大时可能越界），课程代码在 `-Werror` 下直接失败。

## 修复

`char zone[5]` → `char zone[8]`（同时清理旁边的注释掉的 zone[7] 行）。
