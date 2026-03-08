# Chunked Write Recovery

**Trigger**: Use when encountering "Error writing file" or "Invalid tool parameters" errors during file write operations.

**Purpose**: Automatically recover from write failures by chunking content into smaller pieces.

---

## Instructions

You have encountered a file write error. This typically happens when trying to write too much content at once.

**Recovery Strategy**:

1. **Identify the failed write operation**
   - What file were you trying to write?
   - What was the approximate size of the content?

2. **Chunk the content aggressively**
   - Split content into chunks of **maximum 200 lines each**
   - For very large files (>1000 lines), use **150-line chunks**
   - Count actual lines, not estimated size

3. **Execute chunked write**
   - **First chunk**: Use `Write` tool to create the file
   - **Remaining chunks**: Use `Edit` tool with append operations
   - Add a unique placeholder comment at the end of each chunk (except the last) to help with appending

4. **Verification**
   - After completing all chunks, read the file to verify completeness
   - Check that no content was lost between chunks

## Example Pattern

```
# For a 500-line file:

# Chunk 1 (lines 1-200): Write tool
Write(file_path, content_lines_1_to_200)

# Chunk 2 (lines 201-400): Edit tool (append)
Edit(file_path, old_string="[end of chunk 1]", new_string="[end of chunk 1]\n" + content_lines_201_to_400)

# Chunk 3 (lines 401-500): Edit tool (append)
Edit(file_path, old_string="[end of chunk 2]", new_string="[end of chunk 2]\n" + content_lines_401_to_500)
```

## Critical Rules

- **Never retry the same write** that just failed - it will fail again
- **Always chunk smaller** than you think necessary
- **Use Edit for appends**, not multiple Write calls
- **Don't explain the chunking process** to the user unless they ask - just do it
- **If Edit fails**, use an even smaller chunk size (100 lines)

## When to Use This Skill

- After seeing "Error writing file"
- After seeing "Invalid tool parameters"
- When writing files >300 lines
- Proactively for any file >500 lines

---

**Remember**: The goal is to complete the write successfully, not to write it all at once. Chunk aggressively and succeed.
