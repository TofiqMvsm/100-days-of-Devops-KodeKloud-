# Day 1: Create a User with a Non-Interactive Shell

## Task
The backup agent tool at xFusionCorp Industries requires a user with a non-interactive shell.

Create a user named `james` with a non-interactive shell on **App Server 1** (`stapp01`).


## Solution

### 1. Connect to App Server 1
```bash
ssh tony@stapp01
```

### 2. Create the user with a non-interactive shell
```bash
sudo useradd -s /sbin/nologin james
```
(`sudo adduser -s /sbin/nologin james` also works on many systems.)


## Key Takeaway
Service accounts should use a non-interactive shell (`/sbin/nologin`) to reduce the attack surface, since nobody can open a shell session as that user.