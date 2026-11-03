# Day 4: Grant Executable Permissions to a Script

## Task
The script `/tmp/xfusioncorp.sh` on **App Server 1** (`stapp01`) has no executable permissions.

Grant it executable permissions so that all users can execute it.

## Solution

### 1. Connect to App Server 1
```bash
ssh tony@stapp01
```

### 2. Grant read and execute permissions to all users
```bash
sudo chmod a+rx /tmp/xfusioncorp.sh
```

## What went wrong at first
I first ran `chmod +x` only, and it did not work for all users. The script also needed **read** permission, so I added it with `+xr`.

A script is not run directly by the kernel. The interpreter (`bash`) has to open and read the file, so a user with only the execute bit gets "Permission denied". Both `r` and `x` are needed.

## Key Takeaway
- Scripts need read (`r`) and execute (`x`) permission to be run.
- Always verify with `ls -l`.