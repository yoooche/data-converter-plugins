---
name: data-converter
description: Convert data from input .xlsx/.csv files into target template.csv file by column mapping. Supports one-to-one (single source) and many-to-one (multiple sources merged by stacking rows) conversion modes.
disable-model-invocation: true
allowed-tools: Read, Grep, Write, Glob
---

# Data Converter Guide

This skill provides guidance for generating JavaScript code to convert data from input .xlsx/.csv files into a target template.csv file by column mapping. It supports two modes:

- **One-to-one**: a single source file mapped to one output file.
- **Many-to-one (merge)**: multiple source files with the same structure, whose rows are stacked (union) into one output file.

## Step 0: Detect Mode and Resolve Source Files

Ask user type the path of template and source files one by one, let user type the path and filename then ask next question util all of questions are answered, then determine the conversion mode from the user's request:

- **One-to-one**: user specifies a single source file (e.g. "convert source-a.xlsx").
  - Script name: `convert-single.js`

- **Many-to-one**: user specifies multiple files (e.g. "convert source-a.xlsx + source-b.xlsx") or a folder/glob pattern (e.g. "convert all .xlsx files in ./source/").
  - Resolve source files: if a glob/folder pattern is given, use Glob to list all matching files; if an explicit list is given, use that list directly.
  - Confirm the resolved file list with the user before proceeding.
  - Script name: `convert-merged.js`

## Step 1: Mapping Columns

1. Parse all columns in template file.
2. Use the letters in the first row as indexes.
3. For **one-to-one**: use the indexes to find the matching columns in source file.
   For **many-to-one**: use the indexes to find the matching columns in the first resolved source file. Verify that all other source files share the same column structure before proceeding.
4. Complete the mapping table listing both files' column names. And ask user what particular mapping rules I want to apply or just analyze the mapping-rules.md file in `./reference/`. Example:

|target_column|source_column|
|---|---|
|用戶名|客戶名|
|余额|用戶餘額|
|上级代理|代理|

## Step 2: Generate JavaScript Data Converting Script

1. Analyze the type of source and target files (.csv or .xlsx). Use appropriate libs:

|.xlsx|.csv|
|---|---|
|`node-xlsx`|`papaparse`|

Import both libs if source files are mixed types.

2. Use Node.js Core Modules (fs, stream, path) to process files with massive rows and columns streamly.
3. Use source filename + `_converted.csv` to name the output file.
4. Some columns have special mapping rules — read from the path provided by user.
5. For **many-to-one**: the script must iterate over all resolved source files in order, apply the same column mapping to each, and concatenate all data rows into a single output. The header row must appear only once in the output (from the template).
6. Ask me where I want to save the script after generation.

## Step 3: Execute JavaScript Data Converting Script

1. For **one-to-one**: run with 1/10 of the source file's rows to pre-validate.
   For **many-to-one**: run with 1/10 of the first source file's rows to pre-validate.
2. If the script is correct, run it to convert the full dataset. And output the result to <root>/output/` directory which would be created if not exists.
3. If the script is incorrect, go back to Step 2 and fix it.

## Step 4: Validate Target CSV File

1. **Row count**:
   - One-to-one: output rows must equal source file rows.
   - Many-to-one: output rows must equal the sum of all source files' data rows.
2. Check the columns number of target CSV file equals the template CSV file, even if some columns are blank.
3. Check column value formats:
    - Birthday column must be in format `YYYY-MM-DD`.
    - Columns related to operation dates must be in format `YYYY-MM-DD HH:mm:ss`, UTC+8.
    - Balance column must be in float format.