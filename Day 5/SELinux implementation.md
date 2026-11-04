# Day 5: Install SELinux Packages and Disable It Permanently

## Task
The security team wants to test SELinux on **App Server 1** (`stapp01`) in the Stratos Datacenter.

1. Install the required SELinux packages.
2. Permanently disable SELinux for now (it will be re-enabled later).
3. Do not reboot, since a maintenance reboot is already scheduled for tonight.
4. Ignore the live status from the command line. The final status after the reboot should be `disabled`.

## Solution

### 1. Connect to App Server 1
```bash
ssh tony@stapp01
```

### 2. Install the SELinux packages
```bash
sudo yum install selinux-policy selinux-policy-targeted policycoreutils
```

### 3. Check what is installed
```bash
rpm -aq | grep selinux
```

### 4. Disable SELinux permanently
```bash
sudo vim /etc/selinux/config
```
Change the line to:
```
SELINUX=disabled
```
No reboot is needed. The change takes effect on the next boot.

## What went wrong at first
I started with a guide written for CentOS 7 that suggested `policycoreutils-python`, `setools` and `setroubleshoot`. My server runs CentOS 9, where some package names changed, so that command did not work. I found the right package names (`selinux-policy`, `selinux-policy-targeted`, `policycoreutils`) from another source.

## Key Takeaway
- SELinux has three modes: `enforcing`, `permissive` and `disabled`.
- `/etc/selinux/config` controls the mode **across reboots**. `setenforce` only changes it until the next boot, and it cannot set `disabled`.
- SELinux is often disabled temporarily because enabling it before configuration would block applications. Admins usually configure and test first (often in permissive mode), then switch to enforcing. Changing to `disabled` needs a reboot to take effect.
- Package names change between distro versions, so check which OS version a guide was written for. `dnf search` helps find the current names.