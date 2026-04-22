# TronClass CLI Skill

An Agent Skill that lets an AI agent operate the [TronClass](https://tronclass.com/) learning management system on your behalf, currently optimized for Fu Jen Catholic University (FJU).

Once installed, you can ask your agent in plain language to check assignments, download course files, or submit homework — no more manually clicking through the TronClass web UI.

## Install

Install from [ClawHub](https://clawhub.ai/yujun-bo2/tronclass-cli).

Then install the underlying CLI tool:

```bash
npm install -g tronclass-cli
```

## First-time login

FJU students (CAS flow with CAPTCHA):

```bash
tronclass auth login --fju <student_id>
```

Other TronClass users:

```bash
tronclass auth login <username>
```

The session is saved locally and reused automatically whenever the agent invokes a command.

## Example prompts

After logging in, just talk to your agent:

- "What assignments do I still need to submit?"
- "Download all the lecture slides from my Data Structures course this semester."
- "Submit ~/Documents/hw3.pdf to homework 3 of Algorithms."
- "List the courses I'm enrolled in this semester."
- "What's due before next Friday?"

The agent will pick the right `tronclass` commands, chain the lookups (courses → activities → download), and summarize the result.

## What this skill provides

This skill gives the agent:

- Purpose, aliases, and fields for every `tronclass-cli` command
- The lookup chain for common workflows (e.g. listing courses before listing activities before downloading a file)
- Detailed reference docs for each subcommand under `references/`

So when you vaguely say "check what's due this week," the agent knows to run `tronclass todo`, knows the first column is the activity ID, and can pipe it straight into `activities view` for details.

## Links

- Skill page: <https://clawhub.ai/yujun-bo2/tronclass-cli>
- Skill repo: <https://github.com/YuJun-BO2/tronclass-cli-skill>
- CLI tool repo: <https://github.com/YuJun-BO2/tronclass-cli-ts>

## License

MIT
