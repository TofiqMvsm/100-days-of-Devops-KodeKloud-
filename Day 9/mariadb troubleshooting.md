# Day 9: Fix MariaDB Service Not Starting

## Task
The Nautilus application in Stratos DC was down. The production support team found that the MariaDB service on the database server (`stdb01`) wasn't running. Find the cause and fix it.

## Solution

### 1. Connect to the database server
```bash
ssh peter@stdb01
```

### 2. Try to start the service
```bash
sudo systemctl start mariadb
```
It failed with: `Job for mariadb.service failed because the control process exited with error code`.

### 3. Check the service's own log
`systemctl status` and `journalctl` did not show the real reason, so I looked at MariaDB's log:
```bash
sudo tail -30 /var/log/mariadb/mariadb.log
```
It showed:
```
[ERROR] mariadbd: Can't create/write to file '/run/mariadb/mariadb.pid' (Errcode: 13 "Permission denied")
[ERROR] Can't start server: can't create PID file: Permission denied
```

### 4. Fix the ownership and permissions of the directory
```bash
ls -ld /run/mariadb
sudo chown -R mysql:mysql /run/mariadb
sudo chmod 755 /run/mariadb
```

### 5. Start the service and verify
```bash
sudo systemctl start mariadb
sudo systemctl status mariadb
```

## What went wrong at first
`systemctl status` and `journalctl` only said the service failed, with no reason. I lost time looking in the wrong places. The actual cause was in MariaDB's own log file, `/var/log/mariadb/mariadb.log`.

## Why it failed
MariaDB runs as the `mysql` user, not root. On startup it creates a PID file in `/run/mariadb/`. That directory was not writable by `mysql`, so the server could not create the file and refused to start (Errcode 13 means permission denied).

## Key Takeaway
- When a service fails to start, `systemctl status` often isn't enough. Check the application's own logs (for MariaDB: `/var/log/mariadb/mariadb.log`).
- Services usually run as their own user (`mysql`, `postgres`, `apache`), and the files and directories they write to must be owned by or writable for that user.

