# CITS4407 Assignment 2 — YouTube Trending Videos Bash Scripts

## Project Goal

Complete **CITS4407 Assignment 2 2026** by implementing and testing two top-level Bash scripts:

1. `clean` — performs error checking and data cleaning on `trending_videos_unclean.csv`
2. `analyse` — performs analysis on `trending_videos_clean.csv`

The final submission must be a zip file named using the student number, for example:

```bash
24957079.zip
```

The zip file must contain exactly the required assignment files:

```text
clean
analyse
README.md
prompts.pdf
git_backup/
```

The `.git` folder must be copied into `git_backup` before packaging:

```bash
cp -r .git git_backup
```

---

## Important Academic Integrity Constraint

AI may be used only to assist with:

- syntax corrections in Bash code,
- grammar and spelling improvements in comments,
- debugging based on the logic already written by the student,
- testing and identifying inconsistencies.

Do **not** replace the assignment with unexplained generated logic. The final scripts must be simple, readable, and fully understandable by the student.

When editing, keep the code maintainable and comment the reasoning clearly.

---

## Files Expected in the Project Directory

The following input CSV files should be present while developing/testing:

```text
trending_videos_unclean.csv
trending_videos_clean.csv
```

The following executable scripts must be created:

```text
clean
analyse
```

Both scripts must have **no file extension**.

Make them executable:

```bash
chmod +x clean analyse
```

---

# Part 1 — `clean`

## Usage

The script must be run as:

```bash
./clean trending_videos_unclean.csv
```

It should read the input CSV from the current directory and write the cleaned output to:

```text
trending_videos_clean.csv
```

---

## Required Error Checks

The script must perform these checks before cleaning.

### 1. No input file specified

If no command-line argument is provided, print exactly:

```text
ERROR: No input CSV file provided
```

### 2. Input file not found in current directory

If the specified file does not exist in the current directory, print exactly:

```text
ERROR: Input file not found in the current directory
```

### 3. Input file is not CSV

If the input file does not have a `.csv` extension, print exactly:

```text
ERROR: Input file expected in a CSV format
```

### 4. Input file is empty

If the input file is empty, print exactly:

```text
ERROR: Empty file provided
```

### 5. Header does not contain exactly 7 fields

The clarification says the error check applies to the **number of fields in the header**, not every row.

If the header does not contain exactly 7 fields, print exactly:

```text
ERROR: Expected 7 columns in the header
```

---

## Required Cleaning Behaviour

The original input has 7 columns:

```text
video_id,publish_date,views,likes,dislikes,comments_disabled,ratings_disabled
```

The cleaned output should remove the `ratings_disabled` column and therefore contain 6 columns:

```text
video_id,publish_date,views,likes,dislikes,comments_disabled
```

The `clean` script must:

1. Delete the column named `ratings_disabled`.
2. Delete rows that contain **any empty fields**.
   - This is from the assignment clarification.
   - Even if a row has the correct number of comma-separated fields, remove it if any field is empty.
3. Delete duplicate rows.
4. Delete rows that do not contain a `video_id`.
5. Delete rows where `likes` is zero.
6. Delete rows where `dislikes` is zero.
7. Remove the time component from `publish_date`.
   - Example:
     ```text
     2009-09-18T15:36:33.000Z
     ```
     should become:
     ```text
     2009-09-18
     ```

---

## Suggested Implementation Notes for `clean`

Use standard Unix tools only, such as:

- `awk`
- `sed`
- `sort`
- `uniq`
- `mktemp`
- `test`
- shell parameter checks

Important implementation details:

- Do not leave temporary files behind.
- Prefer `mktemp` if temporary files are needed.
- Use `trap` to remove temporary files on exit.
- Preserve the header.
- Ensure duplicate handling does not accidentally remove the header.
- Do not assume the input is already clean.
- The order of columns in the input will not change.

---

## Manual Tests for `clean`

Run these checks before final submission.

### Missing argument

```bash
./clean
```

Expected output:

```text
ERROR: No input CSV file provided
```

### File does not exist

```bash
./clean missing.csv
```

Expected output:

```text
ERROR: Input file not found in the current directory
```

### Non-CSV file

```bash
touch input.txt
./clean input.txt
```

Expected output:

```text
ERROR: Input file expected in a CSV format
```

### Empty CSV file

```bash
touch empty.csv
./clean empty.csv
```

Expected output:

```text
ERROR: Empty file provided
```

### Incorrect header field count

Create a bad file:

```bash
printf "a,b,c\n1,2,3\n" > bad_header.csv
./clean bad_header.csv
```

Expected output:

```text
ERROR: Expected 7 columns in the header
```

### Valid file

```bash
./clean trending_videos_unclean.csv
```

Expected result:

```text
trending_videos_clean.csv
```

The output should contain 6 columns and no invalid rows.

---

# Part 2 — `analyse`

## Usage

The script must be run as:

```bash
./analyse trending_videos_clean.csv
```

It should print results to standard output.

---

## Required Analysis Tasks

The cleaned CSV has 6 columns:

```text
video_id,publish_date,views,likes,dislikes,comments_disabled
```

The script must calculate and print:

1. The `video_id` of the video with the most occurrences in the data.
2. The mean number of views, printed to 2 decimal places.
3. The `video_id` of the video with the maximum number of dislikes.
4. The `video_id` and `publish_date` of the video with the highest engagement rate.
5. The `video_id` and `publish_date` of the video with the least net sentiment rate.

---

## Formulas

### Engagement rate

```text
(likes + dislikes) / views
```

### Net sentiment rate

```text
(likes - dislikes) / views
```

---

## Required Output Format

The assignment sample output is:

```text
Most frequent video, ID: id4667
Mean number of views: 2355595.97
Max dislikes video, ID: id2798
Highest engagement rate video, ID: id2282, dated: 2018-01-04
Least sentiment rate video, ID: id2219 , dated: 2017-12-13
```

Use this format as closely as possible.

If there is a tie, print all matching rows in a clear and sensible format.

---

## Tie Handling Requirements

The assignment explicitly says:

> If there is a tie between two or more videos, print the required data for all of them in a clear and sensible format.

So the script should not arbitrarily keep only one result when multiple rows/videos share the same best value.

Handle ties for:

- most frequent video ID,
- max dislikes,
- highest engagement rate,
- least sentiment rate.

For the mean views, there is no tie issue.

---

## Suggested Implementation Notes for `analyse`

Use `awk` as the main tool.

Recommended approach:

- Skip the header row using `NR > 1`.
- Track occurrence counts by `video_id`.
- Accumulate total views and row count for mean views.
- Track maximum dislikes and store all tied video IDs.
- Calculate engagement rate using numeric arithmetic.
- Calculate sentiment rate using numeric arithmetic.
- Store all tied video/date pairs for highest engagement and least sentiment.
- Use `printf "%.2f"` for the mean number of views.
- Be careful with numeric comparisons and floating-point precision.

---

## Manual Tests for `analyse`

Run:

```bash
./analyse trending_videos_clean.csv
```

Check that:

- The script prints all five required result groups.
- Mean views has exactly two decimal places.
- IDs and dates are taken from the correct columns.
- Engagement rate uses `(likes + dislikes) / views`.
- Net sentiment rate uses `(likes - dislikes) / views`.
- Ties are handled clearly.
- There are no temporary files left behind.

---

# Git Requirements

The assignment gives 2 marks for Git usage.

Make several meaningful commits over time. Avoid only one initial and one final commit.

Example commit sequence:

```bash
git init
git add README.md
git commit -m "Add assignment requirements summary"

git add clean
git commit -m "Implement initial clean script error checks"

git add clean
git commit -m "Add data cleaning logic for CSV rows"

git add analyse
git commit -m "Implement analysis calculations"

git add clean analyse README.md
git commit -m "Test scripts and refine comments"
```

Before packaging:

```bash
cp -r .git git_backup
```

---

# Final Packaging Checklist

Before creating the zip, confirm the project directory contains:

```text
clean
analyse
README.md
prompts.pdf
git_backup/
```

Confirm scripts are executable:

```bash
ls -l clean analyse
```

Expected to show executable permissions, such as:

```text
-rwxr-xr-x
```

Create the final zip:

```bash
zip -r 24957079.zip clean analyse README.md prompts.pdf git_backup
```

Check the zip contents:

```bash
unzip -l 24957079.zip
```

The zip should contain only the required files/folders.

---

# Docker / Final Environment Testing

The assignment says the final version should be tested using the class Docker image before submission.

Inside the Docker/class Linux environment:

```bash
chmod +x clean analyse
./clean trending_videos_unclean.csv
./analyse trending_videos_clean.csv
```

Also test error cases manually.

---

# Style and Maintainability Expectations

The scripts should be:

- simple,
- readable,
- commented,
- easy to follow,
- free of unused temporary files,
- robust to invalid input,
- aligned with the exact required output messages.

Avoid over-engineering.

Prefer clear `awk`/shell logic over unnecessarily complex pipelines.

---

# Codex Task Instructions

Please help finish this assignment by:

1. Reviewing the existing `clean` and `analyse` scripts if present.
2. Checking them against every requirement in this README.
3. Fixing syntax issues, edge cases, and output formatting.
4. Keeping the logic simple and explainable.
5. Adding or improving comments where helpful.
6. Running all manual tests listed above.
7. Confirming final output and packaging readiness.
8. Ensuring no temporary files are left behind.
9. Ensuring `README.md` remains accurate.
10. Ensuring the final zip includes:
   - `clean`
   - `analyse`
   - `README.md`
   - `prompts.pdf`
   - `git_backup/`

Do not add unnecessary files to the final zip.
