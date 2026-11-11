# Day 12: Troubleshoot Apache Not Reachable on Port 6100

## Task
The monitoring tool reported that Apache is not reachable on port `6100` on one of the app servers in the Stratos Datacenter.

Find the faulty server, fix the issue without disabling the firewall or flushing its rules, and make sure Apache answers on port `6100` on every app server. Do not change the website content in `/var/www/html`.

## Solution

### 1. Find the faulty server
From the jump host, test every app server:
```bash
curl http://stapp01:6100
curl http://stapp02:6100
curl http://stapp03:6100
```
Only `stapp01` failed, so I connected to it:
```bash
ssh tony@stapp01
```

### 2. Check what is using the port
First attempt:
```bash
ss sport = :6100
```
It showed nothing, which was misleading. Without `-l`, `ss` only shows connected sockets and hides listening ones.

Correct command:
```bash
sudo ss -4 -tlnp | grep 6100
```
This showed that `sendmail` was listening on port 6100 and gave its PID.

### 3. Check the Apache config
On CentOS the Apache service is `httpd`, not `apache2`.
```bash
grep -i '^Listen' /etc/httpd/conf/httpd.conf
```
The port was already `Listen 6100`, so the config was fine and the problem was elsewhere.

### 4. Check the service status and logs
```bash
sudo systemctl status httpd
sudo journalctl -xeu httpd
```
The error was `Address already in use`: another process was holding port 6100.

### 5. Find the process using the port
```bash
sudo ss -4 -tlnp | grep 6100
```
This showed `sendmail` listening on port 6100.

### 6. Stop the conflicting process and start Apache
```bash
sudo kill <PID>
sudo systemctl start httpd
```

### 7. Verify
```bash
sudo ss -tlnp | grep 6100
curl http://stapp01:6100
```


## What went wrong at first
- I assumed the Apache service was called `apache2`. On CentOS and RHEL it is `httpd`.
- I ran `ss sport = :6100` and got no output, so I thought nothing was using the port. But `ss` hides listening sockets unless `-l` is given. If Apache had been running correctly I would have seen `httpd` there, so empty output was a clue that something was off.
- Running `ss -4 -tlnp | grep 6100` showed the real culprit: `sendmail` was listening on 6100 and blocking Apache.


## Key Takeaway
- Troubleshooting order: test from outside (`curl`), check if something listens (`ss -tlnp`), check the config, then check service status and logs.
- `kill <PID>` stops the process now, but a systemd service may restart it. Stop and disable that service to fix it permanently.
- Never turn off or flush the firewall to fix a connectivity problem. Open only the specific port that is needed.
- Service names differ by distro: `httpd` on CentOS/RHEL, `apache2` on Debian/Ubuntu.
- `ss` hides listening sockets by default. Use `-tlnp` (TCP, listening, numeric, process) and `sudo` to see which process owns a port.
- Empty output from a diagnostic command is not proof that something is absent. Check that the command actually covers what you're looking for.