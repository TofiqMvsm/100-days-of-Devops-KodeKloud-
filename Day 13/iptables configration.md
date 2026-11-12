# Day 13: Restrict Apache Port 8085 with iptables

## Task
Apache's port `8085` is open to everyone on the app servers because there is no firewall. Add a security layer on `stapp01`, `stapp02` and `stapp03`:

1. Install `iptables` and `iptables-services` on each app server.
2. Block incoming port `8085` for everyone except the load balancer host `stlb01`.
3. Make the rules persistent: save them to `/etc/sysconfig/iptables` and enable the `iptables` service. No reboot.

## Solution

### 1. Find the load balancer's IP
```bash
ssh loki@stlb01
ip addr
```
The IP was `10.244.164.53`.

### 2. Connect to each app server
| Server | Command |
|--------|---------|
| App Server 1 | `ssh tony@stapp01` |
| App Server 2 | `ssh steve@stapp02` |
| App Server 3 | `ssh banner@stapp03` |

### 3. Install iptables and iptables-services
```bash
sudo yum install iptables iptables-services -y
```
`iptables-services` provides the service and the saved-rules file, but not the `iptables` command, so both packages are needed.

### 4. Add the rules
```bash
sudo iptables -I INPUT -p tcp --dport 8085 -j DROP
sudo iptables -I INPUT -s 10.244.164.53/32 -p tcp --dport 8085 -j ACCEPT
```
`-I` inserts at the top of the chain, so the second command places the `ACCEPT` for the load balancer **above** the `DROP`.

### 5. Save the rules and enable the service
```bash
sudo iptables-save > /etc/sysconfig/iptables
sudo systemctl enable iptables
```

### 6. Verify
From the jump host (should hang or time out):
```bash
curl http://stapp01:8085
```
From the load balancer (should return the page):
```bash
curl http://stapp01:8085
```

I repeat the same steps 2 more times on other 2 app servers.

## How the rules work
- `-I INPUT`: put the rule in the chain for **incoming** traffic.
- `-p tcp --dport 8085`: match TCP traffic going to port 8085.
- `-s 10.244.164.53/32`: match only that one source IP (`/32` means a single address).
- `-j DROP` silently discards the packet, so the client waits and times out. `-j REJECT` refuses it immediately.
- Rules are checked from top to bottom, and the first match wins.

## Key Takeaway
- Rule **order matters**. If the `DROP` is above the `ACCEPT`, the load balancer gets blocked too.
- `-I` puts a rule at the top, while `-A` appends it to the bottom.
- iptables rules live in memory and are lost on reboot. They are persistent only after being saved to `/etc/sysconfig/iptables` and with the `iptables` service enabled.