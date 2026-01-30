

# Generated instructions for cooperate.py

## Original prompt
make wheel-scan.sh which can scan directories for functions, methods, definitions, etc by using various tools, executed using the following sytntax './wheel-scan.sh [--path /some/path] [--filter *.cpp] [--filter *.h] [--flter *.qml] [--filter ...] [--max-depth 4]'  which accepts arbitrary number of --filters that wildcard match against the files found inside of --path and any subdirectories contained within up to --max-depth levels deep and then uses filetype specific rules to try and extract functions, definitions, and references from the files found in the directories and adding them to a json array with the potentially matching text, its type, signatures, file where it was found, and line number in each object in the json array, one object per function or definition

## GPT-5.1 plan
# AGENTS.md – wheel-scan.sh Implementation Guide

This document instructs an AI coding agent to implement `wheel-scan.sh` as a production-ready shell utility.

The document:
- Embeds the original user prompt verbatim
- Specifies an end-to-end implementation plan
- Includes edge cases, quality checks, instrumentation, testing hooks, configuration defaults, and explicit deliverables
- Avoids code fences and special characters that commonly break shell piping

============================================================
1. Original User Prompt (Verbatim)
============================================================

make wheel-scan.sh which can scan directories for functions, methods, definitions, etc by using various tools, executed using the following sytntax './wheel-scan.sh [--path /some/path] [--filter *.cpp] [--filter *.h] [--flter *.qml] [--filter ...] [--max-depth 4]'  which accepts arbitrary number of --filters that wildcard match against the files found inside of --path and any subdirectories contained within up to --max-depth levels deep and then uses filetype specific rules to try and extract functions, definitions, and references from the files found in the directories and adding them to a json array with the potentially matching text, its type, signatures, file where it was found, and line number in each object in the json array, one object per function or definition

============================================================
2. Overall Goals and Deliverables
============================================================

Goal:
Implement a portable Bash script wheel-scan.sh that:
- Scans a directory tree for files by extension, with filters and depth control
- For each supported file type, extracts potential functions, methods, class definitions, and similar constructs
- Outputs a single well-formed JSON array to stdout, one object per detected item, containing:
  - file path
  - line number
  - raw matched text
  - type (e.g., function, method, class, signal, property)
  - language or file_type
  - a best-effort signature string

Primary deliverable:
- wheel-scan.sh: POSIX-ish Bash script
  - With inline usage/help
  - With robust argument parsing and validation
  - With defensive handling of edge cases
  - With minimal but useful logging to stderr
  - With deterministic, stable JSON output

Secondary deliverables:
- Optional: wheel-scan-test.sh with basic smoke tests (not required, but design script to be testable)
- Example command snippets included in script comments

Constraints:
- Implement in Bash (assume /usr/bin/env bash is available).
- Use only common Unix tools: find, grep, sed, awk, xargs, printf, etc.
- JSON must be simple and valid UTF-8, no trailing commas.
- No external JSON libraries.

============================================================
3. Command-line Interface Specification
============================================================

Executable:
- ./wheel-scan.sh

Syntax:
- ./wheel-scan.sh [--path PATH] [--max-depth N] [--filter PATTERN] [--filter PATTERN2] ...

Arguments:
1) --path PATH
   - Optional; default: current directory "."
   - Must be an existing directory.
   - May be relative or absolute.
   - Single value; later occurrences override earlier ones.

2) --filter GLOB_PATTERN
   - Optional; can be provided multiple times.
   - Each filter is a shell-style pattern that is passed to find -name "PATTERN".
   - Examples: *.cpp, *.h, *.qml, *.sh, *.py, etc.
   - If no filters are provided, default behavior:
     - Either: scan a default set of extensions (recommended: *.cpp, *.hpp, *.h, *.c, *.cc, *.cxx, *.qml, *.py, *.sh), OR
     - Scan all files (less recommended).
   - Implement a default list: *.c *.cc *.cpp *.cxx *.h *.hpp *.hh *.hxx *.qml *.py *.sh

3) --max-depth N
   - Optional; default: 0 means unlimited depth.
     - Implementation detail: 0 means do not add -maxdepth to find.
   - N must be a positive integer (1, 2, 3, ...).
   - If provided as 1, scan only the specified directory (no subdirs).
   - If invalid value is given (non-integer or <= 0), fail with usage error.

4) --help or -h
   - Print usage help text to stdout and exit 0.

Other considerations:
- Any unknown flag should cause a usage error and exit 1.
- Arguments can be in any order.
- All output JSON must go to stdout.
- All logs, warnings, and errors must go to stderr.

============================================================
4. Behavior and Processing Flow
============================================================

High-level pipeline:
1) Parse arguments and validate.
2) Discover candidate files using find and filters.
3) For each file:
   - Determine file type by extension.
   - Dispatch to appropriate scanner function.
4) Collect all scan results.
5) Emit a single JSON array to stdout:
   - [ { ... }, { ... }, ... ]

Error behavior:
- If invalid arguments: print usage to stderr, exit 1.
- If path exists but no matching files: output an empty JSON array: []
- If some files fail to read (permissions, etc.):
  - Log warning(s) to stderr.
  - Continue processing other files.
- Exit code:
  - 0 on success (even if 0 results).
  - Non-zero only on argument/usage errors or fundamental runtime issues.

============================================================
5. File Discovery Module
============================================================

Requirements:
- Implement a function to:
  - Accept: path, max_depth, filters[]
  - Return
