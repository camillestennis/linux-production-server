# Runbook: Linux Production Server

Steps to rebuild the server from scratch on Ubuntu 24.04. Run from any directory unless noted.

## 1. Structure

```
sudo mkdir /srv/production
sudo chown codespace:codespace /srv/production
cd /srv/production
mkdir -p shows/{race-sleeper,very-alpha}/ep{01..03}/{raw,edit,final} assets/{music,sfx,artwork} archive templates
touch shows/very-alpha/ep01/raw/{host,guest}-track.wav
touch templates/show-notes.txt
cp templates/show-notes.txt shows/very-alpha/ep01/edit/ep01-notes.txt
ln -s shows/very-alpha/ep03 current
```

## 2. Users

```
sudo useradd -m -s /bin/bash janet
sudo useradd -m -s /bin/bash tony
sudo useradd -m -s /bin/bash guest
```

## 3. Groups

```
sudo groupadd producers
sudo groupadd editors
sudo usermod -a -G producers janet
sudo usermod -a -G editors janet
sudo usermod -a -G editors tony
```

Always use `-a` with `-G`. Without it, usermod replaces the user's groups.

## 4. Ownership

```
sudo chown root:root /srv/production
sudo chown -R root:editors /srv/production/shows /srv/production/assets
sudo chown -R root:producers /srv/production/archive /srv/production/templates
```

## 5. Permissions

Set files before folders. Wildcards expand as the current user, who can't see inside folders once they're locked.

```
sudo chmod 660 /srv/production/shows/*/ep*/*/*
sudo chmod 2770 /srv/production/shows /srv/production/shows/* /srv/production/shows/*/ep* /srv/production/shows/*/ep*/*
sudo chmod 2775 /srv/production/assets /srv/production/assets/*
sudo chmod 2770 /srv/production/archive
sudo chmod 2775 /srv/production/templates
sudo chmod 664 /srv/production/templates/show-notes.txt
```

Users who share files in these folders should run with `umask 002` so their teammates can edit what they create.

## 6. Verify

In Codespaces, `sudo -u` asks for a password, so use `runuser`.

```
id janet tony guest
sudo runuser -u guest -- ls /srv/production/shows      # expect Permission denied
sudo runuser -u tony -- ls /srv/production/shows       # expect race-sleeper very-alpha
sudo runuser -u tony -- ls /srv/production/archive     # expect Permission denied
sudo runuser -u janet -- ls /srv/production/archive    # expect OK
```
