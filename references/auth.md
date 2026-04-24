# Auth Command Reference

## Login

```bash
tronclass auth login [--fju] [--password <p>] [--base-url <u>] <username>
tronclass auth login --fju --non-interactive <username>
```

Login has three modes. The SDK handles CAS / Keycloak automatically.

### 1. Interactive (human users)

```bash
# Generic: prompts for base URL, password, and CAPTCHA (if required)
tronclass auth login <username>

# FJU shortcut: base URL preset to https://elearn2.fju.edu.tw,
# only prompts for password + CAPTCHA
tronclass auth login --fju <student_id>
```

When a CAPTCHA is required, the CLI downloads the image, opens it with the system image viewer, and prompts for input.

### 2. Non-interactive (no CAPTCHA required)

For scripts and deployments that don't serve a CAPTCHA:

```bash
tronclass auth login --password 'my_password' --base-url https://elearn.example.edu.tw <username>
```

### 3. Deferred-CAPTCHA (FJU, for agents)

Two-step flow for agents that cannot solve the CAPTCHA inline. The password is **never persisted** — it is supplied only at resume time.

```bash
# Step 1: parse the form, download the CAPTCHA image, save pending state.
# Exits 0 and prints a captcha ID. No password requested.
tronclass auth login --fju --non-interactive <student_id>

# Step 2: supply the password and the solved CAPTCHA code.
tronclass auth captcha --password '<password>' <captcha_id> <code>
```

Pending state is stored at `~/.tronclass-cli/pending-captcha/<id>.json` (mode `0600`) and expires after **10 minutes**.

### Flags

| Flag | Description |
|---|---|
| `--fju` | Preset base URL to `https://elearn2.fju.edu.tw`. Mutually exclusive with `--base-url`. |
| `--non-interactive` | Defer the CAPTCHA step so the password can be supplied at resume time. **Requires `--fju`**; cannot be combined with `--password` on `auth login`. |
| `--password <p>` | Supply the password non-interactively. For the deferred-CAPTCHA flow, pass it to `auth captcha`, not to `auth login --non-interactive`. |
| `--base-url <u>` | Supply the base URL non-interactively. Mutually exclusive with `--fju`. |

## Captcha

```bash
tronclass auth captcha --password <p> <id> <code>
```

Completes a pending FJU login that was paused on a CAPTCHA challenge.

- `<id>` — the captcha ID printed by `auth login --fju --non-interactive`.
- `<code>` — the characters read from the saved CAPTCHA image.
- `--password <p>` — the user's password (not stored from step 1).

On success, the pending-state file and CAPTCHA image are removed and the session is saved as usual. On failure (wrong code, expired state, network error), the pending state is also cleared (the CAS `execution` token is single-use) — the caller must re-run `auth login --fju --non-interactive` to start over.

```bash
tronclass auth captcha --password 'my_password' abc123def456 3xyz
```

## Check

```bash
tronclass auth check
```

Prints a formatted table with authenticated username, student ID, base URL, school config, login time, expiry, and remaining time. The **Remaining** field is color-coded: green (>24 h), yellow (<24 h), red (<1 h or expired). Sessions last **24 hours** from login.

## Logout

```bash
tronclass auth logout
```

Clears cookies and config from `~/.tronclass-cli/`.

## Session Storage

All session data lives in `~/.tronclass-cli/`:
- `config.json` — username, studentId, baseUrl, school
- `cookies.json` — session cookies (CookieJar format)
- `pending-captcha/<id>.json` — deferred-login state, mode `0600`, TTL 10 minutes
