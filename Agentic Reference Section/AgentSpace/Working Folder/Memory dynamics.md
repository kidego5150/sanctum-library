
● Bash(grep -r "^@" /work/claude/*/CLAUDE.md /home/claude/.claude/CLAUDE.md 2>/dev/null | head -20)
  ⎿  (No output)

                                                                                                                              03:37 PM claude-sonnet-4-6
● No existing @ import usage anywhere. Let me verify the exact syntax before writing anything — I know Claude Code supports file imports in CLAUDE.md but I want
   to confirm before touching the global file.

● claude-code-guide(Verify CLAUDE.md import syntax)
  ⎿  Prompt:
       I need to verify: does Claude Code support importing another file's contents inside a CLAUDE.md file using @/absolute/path syntax? What is the exact
        syntax, and does it work in the global ~/.claude/CLAUDE.md file? I want to import /work/claude/sanctum-memory/CLAUDE.md from the global CLAUDE.md
       as a fallback. Please confirm the exact syntax and any gotchas. Report in under 150 words.
  ⎿  Fetch(url: "https://code.claude.com/docs/en/claude_code_docs_map.md", prompt: "Does Claude Code support importing file contents in CLAUDE.md using
  ⎿  Fetch(url: "https://code.claude.com/docs/en/memory.md", prompt: "What is the exact syntax for importing another file's contents in CLAUDE.md using
  ⎿  Response:
       Perfect. I found the answer in the documentation.

       Yes, Claude Code supports file imports in CLAUDE.md using @path/to/file syntax. The syntax works with both relative and absolute paths. Relative
       paths resolve relative to the file containing the import, not the working directory.

       For your specific use case: You can add this to ~/.claude/CLAUDE.md to import your project CLAUDE.md:

       @/work/claude/CLAUDE.md

       Key points:
       - Imports work in the global ~/.claude/CLAUDE.md (yes, confirmed)
       - Both absolute (@/absolute/path) and relative (@./relative/path) paths are allowed
       - Imported files can recursively import other files up to 5 levels deep
       - On first import, Claude shows an approval dialog listing the files
       - Imported content is expanded and loaded at startup alongside the importing CLAUDE.md

       Gotcha: Relative paths resolve relative to the file containing the import—if you import from ~/.claude/CLAUDE.md, @./sanctum-memory/CLAUDE.md would
       resolve relative to ~/.claude/, not your working directory.
  ⎿  Done (2 tool uses · 44.4k tokens · 11s)