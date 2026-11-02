# Day 2: Create a User with an Expiry Date

## Task
A developer named `john` needs temporary access to the Nautilus project.

Create a user named `john` on **App Server 1** (`stapp01`) with an account expiry date of `2027-04-15`. The username must be lowercase

## Solution

### 1. Connect to App Server 1
```bash
ssh tony@stapp01
```

### 2. Create the user with an expiry date
```bash
sudo useradd -e 2027-04-15 john
```

### 3. Verify the expiry date
```bash
sudo chage -l john
```

## Key Takeaway
The `-e` flag in `useradd` sets an account expiration date (format `YYYY-MM-DD`), so temporary accounts get disabled automatically. This is better than remembering to delete them manually. You can check or change the date later with `chage`.