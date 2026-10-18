# Phase 5: Users, Groups, and Permissions

The Linux Command Line, Chapter 9

## The plan

| User  | Groups             | Access                                   |
|-------|--------------------|------------------------------------------|
| janet | producers, editors | everything, including the archive        |
| tony  | editors            | shows and assets, no archive             |
| guest | none               | top-level folder and templates, read only |

## What I built

- Created three users with `useradd -m -s /bin/bash`
- Created two groups, `producers` and `editors`, and added members with `usermod -a -G`
- Handed ownership of `/srv/production` to root and the right groups with `chown -R`
- Locked down folders with `chmod 2770` (shows, archive) and `2775` (assets, templates)
- The leading `2` is setgid. New files inside a folder inherit the folder's group, not the creator's personal group.

## What broke

**A typo in janet's shell.** `/etc/passwd` showed `/bing/bash` instead of `/bin/bash`. She could not have logged in. Fixed with `sudo usermod -s /bin/bash janet`. A bad shell path is a real "user can't log in" ticket.

**The -a flag.** I ran `usermod -G editors janet` without `-a` and watched `producers` disappear from her groups. Without `-a`, usermod replaces groups instead of adding to them. Do that to an admin account and you lose sudo.

**sudo -u asked for a password I never set.** `sudo -l` showed `(root) NOPASSWD: ALL`. My account can run anything as root, but only as root. Workaround: `sudo runuser -u tony -- <command>`. Become root first, then drop to the user.

**Order mattered with wildcards.** I set file permissions before folder permissions. Wildcards expand as me, and once the folders were `2770`, my account could not see inside them to expand the pattern.

## What I learned

**Setgid gets the group right. Umask gets the write access right.** Tony created two files in a shared folder. Both landed in group `editors` because of setgid. The one made with the default umask (`022`) came out `-rw-r--r--`, and janet got Permission denied trying to edit it. The one made with umask `002` came out `-rw-rw-r--`, and janet could edit it. A shared folder needs both.

**Aliases belong to the user.** Logged in as janet, `prodls` returned "command not found." My aliases live in my `~/.bashrc`, not hers.

**System startup scripts run for everyone.** Logging in as janet triggered a codespace startup script that failed with Permission denied. My permissions blocked a script I didn't even write.

**sudo does not cover the redirect.** In `sudo ls -lR /srv/production > reports/permissions.txt`, sudo runs `ls` as root, but my own shell writes the file.

## Proof

- `reports/access-tests.txt`: six access tests, all results matched expectations
- `reports/users.txt`: group memberships for janet, tony, and guest
- `reports/permissions.txt`: full permission listing of `/srv/production`

## Next

Phase 6: turn my command history into a clean runbook.
