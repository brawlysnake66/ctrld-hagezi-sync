# ctrld-hagezi-sync

Keeps your Control D profile(s) in sync with [Hagezi DNS blocklists](https://github.com/hagezi/dns-blocklists/tree/main/controld), automatically triggered whenever Hagezi releases an update.

Everything runs on GitHub Actions — no local setup, no `.env` file.

## Setup

### 1. Fork this repository

Click *Fork* on the top right of this page.

After forking, go to the *Actions* tab in your fork and enable workflows if prompted.

### 2. Get your Control D credentials

*API token*
1. Log in to your Control D account.
2. Go to *Preferences → API*.
3. Click *+* to create a new token and copy it.

*Profile ID*
1. Open the profile you want to sync.
2. Copy the ID from the URL:

```
https://controld.com/dashboard/profiles/abc123xyz/filters
                                        ^^^^^^^^^^^
```

### 3. Add repository secrets

Go to *Settings → Secrets and variables → Actions → New repository secret* and add:

| Secret    | Value                                                        |
|-----------|--------------------------------------------------------------|
| `TOKEN`   | Your Control D API token                                     |
| `PROFILE` | One or more profile IDs, comma-separated (e.g. `id1,id2`)   |

That's it. The workflows will run automatically from now on.

## How it works

| Workflow              | Trigger                         | What it does                                                  |
|-----------------------|---------------------------------|---------------------------------------------------------------|
| `check-release.yml`   | Every 2 hours                   | Checks for a new Hagezi release and triggers sync only if the lists in `lists.txt` actually changed |
| `sync.yml`            | Triggered by check, or manually | Builds the binary and runs the sync against your profile(s)   |
| `remove.yml`          | Manual only                     | Removes all synced folders from your profile(s)               |
| `clear-cache.yml`     | Manual only                     | Clears all workflow caches, forcing the next check to run as if from scratch |

You can also trigger a manual sync anytime via *Actions → Sync → Run workflow*.

After each run, a summary with the number of folders and rules synced per profile is available under the *Summary* tab of the workflow run.

## Synced lists

Control D already includes Hagezi's main blocklists (Light, Normal, Pro, Pro Plus, TIF, Ultimate) under *3rd Party Filters*. However, Hagezi also publishes additional folders — such as Spam TLDs, Spam IDNs, Badware Hoster, and Referral Allow — that are **not available there**. The only way to use them is via the API, and they need to be refreshed manually every time Hagezi updates them.

This project automates both: it imports the folders via the API and keeps them up to date automatically whenever Hagezi publishes a new release.

Lists are configured in `lists.txt` — one URL per line. Lines starting with `#` are ignored. The repository comes pre-configured with:

- **Badware Hoster** — domains known to host malware, ransomware, and phishing pages
- **Referral Allow** — allowlist for legitimate referral and affiliate tracking domains, preventing breakage on e-commerce sites
- **Spam IDNs** — internationalized domain names (IDNs) that use lookalike Unicode characters to impersonate real domains
- **Spam TLDs** — entire top-level domains with extremely high abuse rates (e.g. `.li`, `.es`, `.sbs`)
- **Spam TLDs Allow** — exceptions for legitimate sites on the blocked TLDs, so nothing real gets broken

Although pre-configured for Hagezi, the tool supports any list in Control D's JSON folder format. To add or remove lists, edit `lists.txt`. Run `make list` to see all available Hagezi lists with their raw URLs ready to paste.

## License

MIT

---

If this project is useful to you, consider leaving a ⭐ on GitHub — it helps others find it.
