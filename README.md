# Linux Production Server

A shared file server for a fictional podcast production team, built on Ubuntu 24.04 using the commands from The Linux Command Line (3rd edition), chapters 1-9.

I spent years running production teams. This project is the IT side of that job: setting up the server a team works from, deciding who can touch what, and proving the rules hold.

## What it does

Two shows, three episodes each, with raw, edit, and final folders for every episode. Three users with different access:

| User  | Role     | Groups             | Access                                    |
|-------|----------|--------------------|-------------------------------------------|
| janet | producer | producers, editors | everything, including the archive         |
| tony  | editor   | editors            | shows and assets, no archive              |
| guest | outsider | none               | top-level folder and templates, read only |

## Skills used

- Navigating and exploring the filesystem (`ls`, `file`, `less`, `/etc/passwd`)
- Building folder trees with brace expansion and wildcards
- Copying, moving, and linking files, including a symlink to the current episode
- Identifying command types and writing persistent aliases in `~/.bashrc`
- Redirection and pipes: `>`, `>>`, `2>&1`, `tee`, `grep`, `sort`, `wc`
- Users and groups: `useradd`, `groupadd`, `usermod -a -G`
- Ownership and permissions: `chown`, `chmod`, setgid on shared folders, `umask`
- Searching command history and turning it into a runbook

## Proof

- [`reports/access-tests.txt`](reports/access-tests.txt): six access tests run as each user, all matching expectations
- [`reports/permissions.txt`](reports/permissions.txt): full permission listing of the server
- [`reports/users.txt`](reports/users.txt): users and group memberships
- [`reports/inventory.txt`](reports/inventory.txt): structure snapshot built with redirection

## What broke

- A typo gave a user the shell `/bing/bash`, so she couldn't log in. Found it in `/etc/passwd`, fixed it with `usermod -s`.
- `usermod -G` without `-a` wiped a user's existing groups. Demonstrated it, then restored them.
- `sudo -u` asked for a password that didn't exist. `sudo -l` showed my account could only run commands as root, so I switched to `runuser`.
- Wildcards stopped working once folders were locked, because they expand as the current user. Setting file permissions before folder permissions solved it.

Full notes for each phase are in [`docs/`](docs/).

## Rebuild it

Every command needed to rebuild the server is in [`docs/runbook.md`](docs/runbook.md).

## Environment

Built in GitHub Codespaces (Ubuntu 24.04.5 LTS) as a personal learning project for my Linux course at Santa Monica College.
