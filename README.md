# Claude Chunked Write Fix

[简体中文](./README.zh-CN.md) | English

A skill for Claude Code that solves large file write errors through automatic chunking.

## Problem

When using Claude Code for development, you often encounter these errors when trying to write large amounts of content to files:

- `Error writing file` - File write error
- `Invalid tool parameters` - Invalid tool parameters

These errors typically occur when:
- Writing files with more than 300 lines
- Generating large code files
- Batch creating configuration files

## Solution

This skill provides an automated recovery strategy that solves large file write problems through **chunked writing**:

### Core Strategy

1. **Smart Chunking**
   - Split content into chunks of maximum 200 lines
   - For very large files (>1000 lines), use 150-line chunks
   - Count actual lines, not estimated size

2. **Step-by-Step Execution**
   - First chunk: Use `Write` tool to create the file
   - Subsequent chunks: Use `Edit` tool to append content
   - Add unique placeholders at the end of each chunk for appending

3. **Automatic Verification**
   - After completing all chunks, read the file to verify completeness
   - Ensure no content is lost between chunks

### Example

```
# For a 500-line file:

# Chunk 1 (lines 1-200): Write tool
Write(file_path, content_lines_1_to_200)

# Chunk 2 (lines 201-400): Edit tool (append)
Edit(file_path, old_string="[end of chunk 1]", new_string="[end of chunk 1]\n" + content_lines_201_to_400)

# Chunk 3 (lines 401-500): Edit tool (append)
Edit(file_path, old_string="[end of chunk 2]", new_string="[end of chunk 2]\n" + content_lines_401_to_500)
```

## Installation

### Method 1: Manual Installation

1. Locate Claude Code's skills directory:
   ```bash
   ~/.claude/skills/
   ```

2. Create the skill folder:
   ```bash
   mkdir -p ~/.claude/skills/chunked-write
   ```

3. Copy the skill file:
   ```bash
   cp SKILL.md ~/.claude/skills/chunked-write/
   ```

### Method 2: Clone with Git

```bash
cd ~/.claude/skills/
git clone https://github.com/XixiangWu/claude-chunked-write-fix.git chunked-write
```

### Verify Installation

After restarting Claude Code, the skill will be automatically loaded. You can verify by:

1. Type `/help` in Claude Code to see available skills
2. Look for the `chunked-write` skill

## Usage Scenarios

This skill automatically triggers in the following situations:

- ✅ After encountering "Error writing file" error
- ✅ After encountering "Invalid tool parameters" error
- ✅ When writing files with more than 300 lines
- ✅ Proactively handling files with more than 500 lines

## Critical Rules

- **Never retry** a write operation that just failed - it will fail again
- **Always use smaller chunks** than you think necessary
- **Use Edit for appending**, not multiple Write calls
- **Don't explain** the chunking process to users unless they ask - just do it
- **If Edit fails**, use an even smaller chunk size (100 lines)

## Technical Details

- Maximum chunk size: 200 lines (regular files)
- Large file chunk size: 150 lines (>1000 line files)
- Retry chunk size on failure: 100 lines
- Uses `Write` tool to create initial file
- Uses `Edit` tool to append subsequent content

## Contributing

Issues and Pull Requests are welcome!

## License

MIT License

## Author

XixiangWu
