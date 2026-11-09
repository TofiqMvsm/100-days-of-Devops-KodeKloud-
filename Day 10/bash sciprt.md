# Day 10: Bash Script to Archive Website Files and Copy Them to the Storage Server

## Task
Create a bash script `news_archive.sh` on **App Server 1** (`stapp01`), placed in `/scripts`, that:

1. Creates a zip archive `xfusioncorp_news.zip` of `/var/www/html/news`.
2. Saves it in `/archives/` on App Server 1 (temporary storage, cleaned weekly).
3. Copies it to `/archives/` on the Nautilus Storage Server (`ststor01`).
4. Runs without asking for a password, and can be run by the `tony` user.
5. Does not use `sudo` inside the script.

The `zip` package must be installed manually beforehand, outside the script.

## Solution

### 1. Connect to App Server 1
```bash
ssh tony@stapp01
```

### 2. Set up password-less SSH to the storage server
```bash
ssh-keygen
ssh-copy-id natasha@ststor01
```
I entered the password once for `ssh-copy-id`, so the script does not need one later. This must be done as the user that will run the script (`tony`).

### 3. Create the script
```bash
sudo vi /scripts/news_archive.sh
```
```bash
#!/bin/bash

zip -r /archives/xfusioncorp_news.zip /var/www/html/news
scp /archives/xfusioncorp_news.zip natasha@ststor01:/archives/
```

### 4. Make it executable
```bash
sudo chmod +x /scripts/news_archive.sh
```

### 5. Run and verify
```bash
/scripts/news_archive.sh
ls -l /archives/xfusioncorp_news.zip
ssh natasha@ststor01 ls -l /archives/
```
The archive should exist on both servers, with no password prompt.


## Key Takeaway
- Password-less `scp` needs key-based SSH (`ssh-keygen` + `ssh-copy-id`) set up for the user who runs the script.
- Cron jobs and scripts can't type passwords, so automation depends on SSH keys.
- Avoid `sudo` inside scripts, because it can prompt for a password. Set up permissions beforehand.