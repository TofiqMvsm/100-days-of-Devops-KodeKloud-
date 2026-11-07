# Day 7: Install Ansible 4.7.0 on the Jump Host with pip3

## Task
The Nautilus DevOps team will use the jump host as the Ansible controller.

Install Ansible version `4.7.0` on the jump host using `pip3` only, and make sure the `ansible` binary is available globally for all users.

## Solution

### 1. Install Ansible with pip3
```bash
sudo pip3 install ansible==4.7.0 --prefix=/usr/local
```

### 2. Verify
```bash
which ansible
ansible --version
```

## What went wrong at first
My first attempt was without `sudo`. A normal user can't write to system directories, so the install failed or went into `~/.local`, where only my user would find it.

I also didn't know about `--prefix`. It sets where pip installs packages and executables. With `--prefix=/usr/local`, the `ansible` binary goes into `/usr/local/bin`, which is in every user's `PATH`.

## Key Takeaway
- Use `sudo` for a system-wide install, otherwise pip installs only for the current user.
- `--prefix=/usr/local` puts executables in `/usr/local/bin` so all users can run them.