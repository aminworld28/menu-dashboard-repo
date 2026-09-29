# Menu Dashboard

A small internal web dashboard that lets non-technical staff (e.g. the marketing team) update menu **prices, items and deals** and push them to a third-party delivery/ordering platform's menu API, store by store, without touching JSON or Postman.

- Runs on any always-on office computer. No cloud needed.
- **One file (`app.py`), Python standard library only.** No `pip install` needed to run it.
- Data lives in a local SQLite file (`menu.db`).

## Features

- Edit prices (enter the price **incl. VAT**; net price is calculated automatically) and switch items/deals on or off
- Add, edit and delete items and deals; deal prices and inner item prices are calculated automatically
- Publish to many stores in one click, with an automatic **rate limiter** (default 5 stores/minute), retries and live progress
- **Test mode** (saves JSON files, sends nothing) and **Live mode**
- Login page with sessions, logout and brute-force lockout; publish history (who, when, result)
- Automatic token handling: logs in to the API itself and renews expired tokens

## Quick start

Requires Python 3.8+.

```bash
python app.py import sample_menu.xlsx   # one-time: load a menu (only needs openpyxl: pip install openpyxl)
python app.py                           # first run creates config.json
```

1. Open `config.json` and set the API details and your own dashboard passwords (see below).
2. Restart, then open `http://SERVER-IP:8000` and sign in.
3. Use **Test mode** first. Go Live only after checking the JSON output.

Windows: double-click `start.bat`.

> Re-running `import` **replaces** the whole menu with the spreadsheet contents. After the first import, the dashboard is the source of truth.

## Connecting to your third-party API

Everything about the API lives in `config.json` (see `config.example.json`). It is never committed to git.

| Setting | What it is |
|---|---|
| `endpoint_url` / `method` | The menu upload URL and HTTP method |
| `rate_per_min` | The API's rate limit. The publish queue paces itself to this |
| `auth_header` / `token_prefix` | How the token is sent, e.g. `Authorization` + `Bearer ` |
| `token` | Optional ready-made token (works until it expires) |
| `login` | Token URL and credentials, so the app fetches and renews tokens itself |
| `extra_headers` | Any other headers the API requires |

To adapt this to a different provider:

1. **Copy the working request from Postman / the API docs** (URL, method, headers, body) into the settings above.
2. **Login:** set `login.form` to `true` for form-encoded bodies or `false` for JSON, and set `login.token_path` to where the token appears in the response (e.g. `access_token`, or `data.token` for nested responses).
3. **Payload shape:** the JSON sent to the API is built by the `build()` function in `app.py`. If a provider expects different field names, that is the only place to change.
4. Run a **Test** publish, compare `output/<store>.json` with what the API expects, then do one **Live** publish to a single store.

## Spreadsheet format (for the one-time import)

`sample_menu.xlsx` shows the expected layout: three sheets named `Items`, `Deals` and `Deal_Items`, each with a `StoreCode` column. The menu is assumed to be the same for all stores; the first store found is used as the master menu, and other stores are added to the store list.

## Security notes

- **Never commit** `config.json`, `menu.db`, `output/` or real spreadsheets. `.gitignore` already excludes them.
- Change the default dashboard passwords in `config.json` before first use.
- Designed for a **private office network**. There is no HTTPS; do not expose the port to the internet.
- Restrict who can read the server folder. `config.json` contains API credentials.
- Back up `menu.db` regularly.

## Roadmap ideas

Edit item names/categories, roles (edit vs publish), "unpublished changes" warning, password hashing, per-store overrides.
"# menu-dashboard-repo" 
