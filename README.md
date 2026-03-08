# Claude Chunked Write Fix

一个用于 Claude Code 的技能(Skill),解决大文件写入时的错误问题。

## 问题背景

在使用 Claude Code 进行开发时,当尝试一次性写入大量内容到文件时,经常会遇到以下错误:

- `Error writing file` - 文件写入错误
- `Invalid tool parameters` - 无效的工具参数

这些错误通常发生在:
- 写入超过 300 行的文件时
- 生成大型代码文件时
- 批量创建配置文件时

## 解决方案

本技能提供了一个自动化的恢复策略,通过**分块写入**的方式解决大文件写入问题:

### 核心策略

1. **智能分块**
   - 将内容分割成最多 200 行的块
   - 对于超大文件(>1000行),使用 150 行的块
   - 精确计算行数,而非估算大小

2. **分步执行**
   - 第一块:使用 `Write` 工具创建文件
   - 后续块:使用 `Edit` 工具追加内容
   - 在每块末尾添加唯一占位符,便于追加操作

3. **自动验证**
   - 完成所有块的写入后,读取文件验证完整性
   - 确保块之间没有内容丢失

### 示例

```
# 对于一个 500 行的文件:

# 块 1 (第 1-200 行): Write 工具
Write(file_path, content_lines_1_to_200)

# 块 2 (第 201-400 行): Edit 工具(追加)
Edit(file_path, old_string="[end of chunk 1]", new_string="[end of chunk 1]\n" + content_lines_201_to_400)

# 块 3 (第 401-500 行): Edit 工具(追加)
Edit(file_path, old_string="[end of chunk 2]", new_string="[end of chunk 2]\n" + content_lines_401_to_500)
```

## 安装方法

### 方法 1: 手动安装

1. 找到 Claude Code 的技能目录:
   ```bash
   ~/.claude/skills/
   ```

2. 创建技能文件夹:
   ```bash
   mkdir -p ~/.claude/skills/chunked-write
   ```

3. 复制技能文件:
   ```bash
   cp SKILL.md ~/.claude/skills/chunked-write/
   ```

### 方法 2: 使用 Git 克隆

```bash
cd ~/.claude/skills/
git clone https://github.com/XixiangWu/claude-chunked-write-fix.git chunked-write
```

### 验证安装

重启 Claude Code 后,技能会自动加载。你可以通过以下方式验证:

1. 在 Claude Code 中输入 `/help` 查看可用技能
2. 查找 `chunked-write` 技能

## 使用场景

该技能会在以下情况下自动触发:

- ✅ 遇到 "Error writing file" 错误后
- ✅ 遇到 "Invalid tool parameters" 错误后
- ✅ 写入超过 300 行的文件时
- ✅ 主动处理超过 500 行的文件时

## 关键规则

- **永远不要重试**刚刚失败的写入操作 - 它会再次失败
- **总是使用更小的块**,比你认为必要的还要小
- **使用 Edit 追加**,而不是多次 Write 调用
- **不要向用户解释**分块过程,除非他们询问 - 直接执行即可
- **如果 Edit 失败**,使用更小的块大小(100 行)

## 技术细节

- 最大块大小:200 行(常规文件)
- 超大文件块大小:150 行(>1000 行的文件)
- 失败重试块大小:100 行
- 使用 `Write` 工具创建初始文件
- 使用 `Edit` 工具追加后续内容

## 贡献

欢迎提交 Issue 和 Pull Request!

## 许可证

MIT License

## 作者

XixiangWu
