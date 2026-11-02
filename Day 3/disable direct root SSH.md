# Day 3: Disable Direct Root SSH Login

## Task
Following a security audit, xFusionCorp Industries requires direct root SSH login to be disabled on **all app servers** in the Stratos Datacenter (`stapp01`, `stapp02`, `stapp03`).

## Solution

### 1. Connect to each app server
| Server | Command |
|--------|---------|
| App Server 1 | `ssh tony@stapp01` |
| App Server 2 | `ssh steve@stapp02` |
| App Server 3 | `ssh banner@stapp03` |

### 2. Edit the SSH config
```bash
sudo vim /etc/ssh/sshd_config
```
Find the `PermitRootLogin` line (it may be commented out with `#`) and set it to:
```
PermitRootLogin no
```

### 3. Restart the SSH service
```bash
sudo systemctl restart sshd
```

## What went wrong at first
My first attempt was adding `DenyUsers root` to `sshd_config`. The task was marked as failed.

`DenyUsers root` blocks root from logging in, but it is a different directive from what the task asked for. The checker looks for `PermitRootLogin no`, and that is also the standard, widely recognised way to disable root SSH login. Switching to `PermitRootLogin no` fixed it.

## Key Takeaway
- Use `PermitRootLogin no` in `/etc/ssh/sshd_config` to disable direct root SSH login, then restart `sshd` for it to take effect.
