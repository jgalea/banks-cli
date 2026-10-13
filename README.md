<div align="center">

# banks

[![License](https://img.shields.io/badge/LICENSE-MIT-5C9E31?style=for-the-badge)](LICENSE)
[![Python](https://img.shields.io/badge/PYTHON-3.9%2B-3776AB?style=for-the-badge&logo=python&logoColor=white)](https://www.python.org)
[![Built by](https://img.shields.io/badge/BUILT%20BY-JGALEA-8A2BE2?style=for-the-badge&logo=github&logoColor=white)](https://github.com/jgalea)

**Balances and transactions from all your European bank accounts in one terminal command, through open banking. Read-only.**

</div>

It uses [Enable Banking](https://enablebanking.com), a licensed account information provider that connects to banks and e-money providers across Europe, including N26, Revolut, Wise and most national banks. Their restricted mode is free for one person reading their own accounts, which is what this tool is built for.

It can't move money. The API it talks to is account information only, and the application you register has no payment access.

Needs Python 3.9+ with `pyjwt` and `cryptography`.

## Setup

Install:

```
pip install pyjwt cryptography
mkdir -p ~/.local/bin
curl -fsSL https://raw.githubusercontent.com/jgalea/banks-cli/main/banks -o ~/.local/bin/banks
chmod +x ~/.local/bin/banks
```

Make a key pair and certificate. Only the certificate goes to Enable Banking; the private key stays on your machine:

```
mkdir -p ~/.config/banks && cd ~/.config/banks
openssl req -new -x509 -days 3650 -nodes -newkey rsa:4096 -keyout private.pem -out public.crt -subj "/CN=banks"
chmod 600 private.pem
```

Register the application:

1. Sign in at [enablebanking.com/cp](https://enablebanking.com/cp/applications). It's email only; you get a one-time sign-in link.
2. Under "Add a new application", pick "Production", then "Generate outside the browser and import public certificate".
3. Give it a name, set the redirect URL to `https://localhost/`, fill in the description, a data protection email and privacy and terms URLs, and paste in `public.crt`.
4. Register, then click "Activate by linking accounts" and link one of your banks. The application stays inactive until at least one account is linked this way.

Point `banks` at it, using the application ID shown in the control panel:

```
banks setup <application-id> ~/.config/banks/private.pem --redirect https://localhost/
```

## Linking a bank

```
banks banks PT wise          is it available, and for how long can consent last
banks link Wise PT           approve access in the browser
```

`link` opens the bank's approval page. When the bank sends you back to `https://localhost/...` the page won't load. That's expected: copy the address from the address bar and `banks` picks it up from the clipboard on macOS, or asks you to paste it elsewhere. Use `--no-browser` to print the approval URL instead of opening it.

In restricted mode Enable Banking only returns accounts that are also linked in the control panel. Before running `banks link` for a bank, link it once under "Link accounts" on your application. If you skip this, `link` succeeds but reports 0 accounts.

Look banks up under the country you're a customer in. Neobanks like Revolut, N26 and Wise are listed under each country they serve.

Consent lasts up to 180 days for most banks. `banks status` shows how many days each one has left; link the bank again when it runs out.

## Commands

```
banks balances [filter]            current balance per account, totals per currency
banks tx [filter] [--days 30]      transactions, newest first
banks status                       linked accounts and days until consent expires
banks banks <country> [name]       banks available in a country
banks link <bank> <country>        approve access to a bank
banks finish <redirect-url>        complete a link from the redirect URL if link timed out
banks unlink <bank>                revoke a bank's consent
```

Every command takes `--json`.

Banks limit how often an app can read your data without you present, usually about four times a day per account under PSD2. If a bank hits its limit, `balances` shows it as unavailable and carries on with the rest.

## Where things are stored

Everything lives in `~/.config/banks/`, readable only by you: `config.json` (application ID and redirect URL), `private.pem`, and `sessions.json` (one entry per linked bank, with account IDs and consent expiry). No balances or transactions are written to disk.

## Not affiliated

Unofficial and not made by or connected to Enable Banking or any bank. Read Enable Banking's terms before use: restricted mode is for your own accounts, not for running a service for others.
