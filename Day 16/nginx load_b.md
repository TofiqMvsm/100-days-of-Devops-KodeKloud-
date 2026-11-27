# Day 16: Configure Nginx as a Load Balancer for the App Servers

## Task
Traffic to the Nautilus website is growing, so the app is being moved to a high availability stack. Only the load balancer (LBR) server is left to configure:

1. Install `nginx` on the LBR server if it is not installed.
2. Configure load balancing in the `http` context using all app servers. Update only `/etc/nginx/nginx.conf`.
3. Do not change the Apache port on the app servers, and make sure Apache is running on all of them.
4. Test with `curl http://stlb01:80`.

## Solution

### 1. Make sure Apache is running on every app server
```bash
ssh tony@stapp01     # then steve@stapp02, banner@stapp03
sudo systemctl enable --now httpd
sudo systemctl status httpd
```

### 2. Find the port Apache listens on
```bash
sudo ss -tlnp | grep httpd
grep -i '^Listen' /etc/httpd/conf/httpd.conf
```
Apache was on port `8089` on all three servers. I left it unchanged.

### 3. Install nginx on the LBR server
```bash
ssh loki@stlb01
sudo dnf install nginx -y
```

### 4. Configure the load balancer
```bash
sudo vi /etc/nginx/nginx.conf
```
Inside the `http { }` block, add an `upstream` group and make the `server` block use it:
```nginx
http {
    upstream backend {
        server stapp01:8089;
        server stapp02:8089;
        server stapp03:8089;
    }

    server {
        listen 80;

        location / {
            proxy_pass http://backend;
        }
    }
}
```

### 5. Test the config, then start nginx
```bash
sudo nginx -t
sudo systemctl enable --now nginx
```

### 6. Verify
```bash
curl http://stlb01:80
```
It should return the website's page.

## What went wrong at first
I added only the `upstream` block and forgot the `location / { proxy_pass http://backend; }` part, so nginx returned an error. An `upstream` block only defines a group of servers. Nothing sends traffic to the group until a `proxy_pass` refers to it.

## How it works
- `upstream backend { ... }` defines a named pool of servers. It goes in the `http` context, outside any `server` block.
- `proxy_pass http://backend;` forwards each request to a server in that pool. The name must match the upstream name.
- By default nginx uses **round robin**, so requests go to each server in turn.
- If a server goes down, nginx stops sending it traffic, so the site stays up on the others.

## Key Takeaway
- A load balancer spreads traffic over several servers, which gives better performance and keeps the site up if one server fails.
- The upstream needs the app servers' real port (8089 here), not the load balancer's port 80.
- `server` lines in an `upstream` block take host and port only, with no `http://`.
- Always run `nginx -t` before starting or reloading.
- Use `ss -tlnp | grep <service>` to find which port a service listens on.