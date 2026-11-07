# Day 7: Set Up Password-less SSH from the Jump Host to All App Servers

## Task
Scripts on the jump host run at regular intervals and perform operations on all app servers in the Stratos Datacenter.

The `thor` user on the jump host needs password-less SSH access to all app servers through their respective sudo users:

| Server | Hostname | User |
|--------|----------|------|
| App Server 1 | `stapp01` | `tony` |
| App Server 2 | `stapp02` | `steve` |
| App Server 3 | `stapp03` | `banner` |

## Solution

### 1. Generate an SSH key pair on the jump host (as `thor`)
```bash
ssh-keygen
```

### 2. Copy the public key to each app server
```bash
ssh-copy-id tony@stapp01
ssh-copy-id steve@stapp02
ssh-copy-id banner@stapp03
```
Each command asks for that user's password once.

### 3. Verify
```bash
ssh tony@stapp01
ssh steve@stapp02
ssh banner@stapp03
```

## Key Takeaway
- `ssh-copy-id user@host` is the quickest way to install a public key. It also sets the correct permissions on `~/.ssh` and `authorized_keys`.
