# Day 6: Install Cronie and Add a Cron Job

## Task
The Nautilus team wants scheduled scripts on all app servers in Stratos DC. Before that, test the setup with a sample cron job:

1. Install the `cronie` package on all app servers and start the `crond` service.
2. Add the cron job `*/5 * * * * echo hello > /tmp/cron_text` for the root user.

## Solution

### 1. Connect to each app server
| Server | Command |
|--------|---------|
| App Server 1 | `ssh tony@stapp01` |
| App Server 2 | `ssh steve@stapp02` |
| App Server 3 | `ssh banner@stapp03` |

### 2. Install cronie and start crond
```bash
sudo yum install cronie
sudo systemctl start crond
```

### 3. Add the cron job for root
```bash
sudo crontab -e
```
Add this line, then save and exit:
```
*/5 * * * * echo hello > /tmp/cron_text
```

## Understanding the schedule
```
*/5  *    *    *    *
min  hour dom  mon  dow
```
- `*` means every value.
- `*/5` in the minute field means every 5 minutes.
- Cron's smallest unit is one minute, so it cannot run jobs every few seconds.

## Key Takeaway
- `cronie` provides the `crond` service, which runs scheduled jobs.
- `crontab -e` edits the cron table of the current user. With `sudo`, it edits root's.
- The five fields are minute, hour, day of month, month and day of week.