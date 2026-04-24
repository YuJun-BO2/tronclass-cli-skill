---
name: tronclass-cli
description: Interact with the TronClass learning management system (currently optimized for FJU) via the `tronclass` CLI — check courses, assignments, deadlines, grades, announcements, and course materials; download lecture slides or PDFs; submit homework. Use whenever the user asks anything about TronClass, even if they don't say "TronClass" explicitly. Triggers include checking what homework is due, downloading lecture slides or PDFs, submitting assignments, listing enrolled courses, reading school or course announcements, or viewing activity details. If the user mentions school, assignments, due dates, course files, or anything that sounds like an LMS task, lean toward using this skill.
license: MIT
metadata:
  authors: YuJun-BO2
  cli-version: 0.3.1
  repository: https://github.com/YuJun-BO2/tronclass-cli-skill
  homepage: https://github.com/YuJun-BO2/tronclass-cli-ts
  openclaw:
    emoji: "🎓"
    homepage: https://github.com/YuJun-BO2/tronclass-cli-skill
    requires:
      bins:
        - tronclass
    install:
      - id: npm
        kind: node
        package: tronclass-cli
        bins:
          - tronclass
        label: Install TronClass CLI (npm)
---

# TronClass CLI Skill

You have access to `tronclass-cli`, a command-line tool for managing the user's TronClass account.

## Setup

Install globally (once):
```bash
npm install -g tronclass-cli
```

## General Guidelines

- **Authentication**: Most commands require a saved session. If a command fails, tell the user to run `tronclass auth login <username>`. Sessions last 24 hours and are reused automatically. `tronclass auth check` shows remaining time.
- **FJU users**: Use `tronclass auth login --fju <student_id>` for the CAS flow with CAPTCHA support.
- **Non-interactive login (agents/scripts)**: Pass `--password <p>` (and `--base-url <u>` for generic deployments) to skip prompts. For FJU, where a human must solve the CAPTCHA, use the two-step deferred flow: `tronclass auth login --fju --non-interactive <id>` prints a captcha ID and opens the captcha image; then `tronclass auth captcha --password <p> <id> <code>` completes the login. The password is never written to disk — it is supplied only at resume time.
- **Finding IDs**: The typical lookup chain is `courses list` → `activities list <course_id>` → `activities view <activity_id>`. The `todo` command shows activity IDs directly in the first column.
- **Aliases**: `activities` → `a`, `courses` → `c`, `homework` → `hw`/`h`, `todo` → `td`/`t`, `announcements` → `ann`. Subcommands also have short aliases (`list`→`l`/`ls`, `view`→`v`, `download`→`d`/`dl`, `submit`→`s`).

## Command Quick Reference

| Command | What it does |
|---|---|
| `tronclass auth login [--fju] [--password <p>] [--base-url <u>] <user>` | Log in (interactive by default; `--password` / `--base-url` make it non-interactive) |
| `tronclass auth login --fju --non-interactive <user>` | Start a deferred-CAPTCHA FJU login; prints a captcha ID, no password on disk |
| `tronclass auth captcha --password <p> <id> <code>` | Complete a deferred FJU login with the solved CAPTCHA |
| `tronclass auth check` | Show current session info, expiry, and remaining time |
| `tronclass auth logout` | Clear saved session |
| `tronclass todo` | Pending tasks — first column is the activity ID |
| `tronclass courses list [--all] [--raw]` | List courses |
| `tronclass activities list <course_id>` | List activities in a course |
| `tronclass activities view <activity_id>` | Rich view: metadata + description + attachments |
| `tronclass activities download <ref_id> [output]` | Download file (defaults to ~/Downloads/) |
| `tronclass homework list <course_id>` | List homework for a course |
| `tronclass homework submit <activity_id> <files...>` | Submit homework files |
| `tronclass ann list [course_id]` | List announcements (school-wide if no course_id) |
| `tronclass ann view <ann_id> [course_id]` | Rich view: metadata + HTML-rendered body + attachments |
| `tronclass ann download <ref_id> [output]` | Download an announcement attachment |

Most list/view commands support `--fields f1,f2,...` to customize displayed columns.

## Key Behaviors

**`tronclass todo`** — shows `id, course_name, title, end_time` by default. The `id` column is the activity ID you can pass directly to `activities view`.

**`tronclass activities view <id>`** — displays a formatted table (not raw JSON). The Attachments section lists each file with its `ref_id` and a ready-to-run download command. No need to manually dig through JSON.

**`tronclass activities download <ref_id>`** — `output_file` is optional. If omitted, the file is saved to `~/Downloads/<filename>` using the server-provided filename.

**`tronclass ann list`** — with no argument, lists the school-wide announcement feed (first page, ~30 items). Pass a `course_id` to list announcements for a specific course.

**`tronclass ann view <ann_id>`** — renders the announcement's HTML body in the terminal (bold, links, lists, images). If you know which course the announcement belongs to, pass `course_id` as the second argument to skip a multi-page search of the global feed.

## Common Workflows

**Check what's due:**
```bash
tronclass todo
```

**Download a course file:**
```bash
tronclass courses list                        # get course_id
tronclass activities list <course_id>         # get activity_id
tronclass activities view <activity_id>       # see Attachments table → note ref_id
tronclass activities download <ref_id>        # saves to ~/Downloads/ automatically
```

**Submit homework:**
```bash
tronclass homework list <course_id>           # find activity_id
tronclass homework submit <activity_id> ./my_essay.pdf
```

**Read announcements:**
```bash
tronclass ann list                            # school-wide feed
tronclass ann list <course_id>                # course-specific feed
tronclass ann view <ann_id> [course_id]       # full content, HTML rendered
```

## Reference Files

For detailed option lists, field names, and edge cases, load the relevant reference:

- `references/auth.md` — login flows, session management
- `references/todo.md` — todo fields, filtering
- `references/courses.md` — course fields, `--raw` flag
- `references/activities.md` — activities list/view/download in depth
- `references/homework.md` — homework list, submit options, draft mode
- `references/announcements.md` — announcements list, view (HTML rendering), download
