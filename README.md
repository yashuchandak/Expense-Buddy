# ExpenseBuddy

ExpenseBuddy is a small Model Context Protocol (MCP) server for recording and reviewing expenses. It exposes three tools to an MCP-compatible client:

- `record_expense(amount, category, comment)` records an expense with the current local date and time.
- `fetch_expenses(days)` lists expenses from the last requested number of calendar days and reports their total.
- `delete_expense(expense_id)` removes an expense by the ID returned from `fetch_expenses`.

The server stores data in CSV files beside `expense_buddy.py`. Each user code gets its own file, named using a SHA-256 hash of the code. The code itself is not used as a filename.

## Requirements

- Python 3.11 or later
- pip

## Install

From the project directory:

```bash
python3 -m venv .venv
source .venv/bin/activate
python -m pip install --upgrade pip
pip install -r requirements.txt
```

## Start the server

```bash
source .venv/bin/activate
python expense_buddy.py
```

The server listens on port `8000` and serves MCP over Streamable HTTP at:

```text
http://<server-host>:8000/mcp?user=<8-character-random-code>
```

Use a different random 8-character code for each user. The server rejects requests to `/mcp` when the `user` query parameter is missing, repeated, or not exactly eight characters from `A-Z`, `a-z`, `0-9`, `_`, and `-`. Keep each code private; anyone with a user's connection URL can access that user's expense tools and data.

For a local connection, use `127.0.0.1` or `localhost` as the host. If exposing the server beyond localhost, put it behind an appropriately configured HTTPS reverse proxy and firewall; do not expose the unauthenticated port directly to the public internet. The user code is a simple access gate, not a replacement for robust authentication.

## Expense categories

The allowed categories are:

`Food`, `Petrol`, `Recharge`, `Entertainment`, `Sports`, `Travel`, `Health`, `Personal Care`, `Shopping`, `Household`, and `Other`.

Amounts must be greater than zero and may have at most two decimal places. Comments cannot be empty. `fetch_expenses` accepts 1 through 3650 calendar days, including today.

## Data files

On the first tool call for a user, the server creates a CSV with the required headers. Files are stored in the project directory as `expenses_<user-hash>.csv`. Back up these CSV files to preserve expense records. Do not commit private expense data to source control.

## Stop the server

Press `Ctrl+C` in the terminal where the server is running.
