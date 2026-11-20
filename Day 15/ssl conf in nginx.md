# Day 15: Install Nginx and Deploy a Self-Signed SSL Certificate

## Task
Prepare `App Server 1` (`stapp01`) in the Stratos Datacenter for an application deployment:

1. Install and configure `nginx` on App Server 1.
2. A self-signed SSL certificate and key are at `/tmp/nautilus.crt` and `/tmp/nautilus.key`. Move them to a suitable location and deploy them in Nginx.
3. Create an `index.html` with the content `Welcome!` under the Nginx document root.
4. Test from the jump host with `curl -Ik https://<app-server-name>/`.



## Solution

### 1. Install nginx on App Server 1
```bash
ssh tony@stapp01
sudo yum install nginx -y
```

### 2. Move the certificate and key to the location Nginx's config expects
```bash
ls -l /tmp/nautilus.crt /tmp/nautilus.key
sudo mv /tmp/nautilus.crt /etc/pki/nginx/
sudo mv /tmp/nautilus.key /etc/pki/nginx/private/
```
I used the paths shown in the default `nginx.conf` TLS block.

### 3. Enable the SSL block in the config
```bash
sudo vi /etc/nginx/nginx.conf
```
Remove the `#` from the commented-out HTTPS `server` block, and point it at the files:
```
ssl_certificate "/etc/pki/nginx/nautilus.crt";
ssl_certificate_key "/etc/pki/nginx/private/nautilus.key";
```

### 4. Create the index page
```bash
echo 'Welcome!' | sudo tee /usr/share/nginx/html/index.html
```

### 5. Open the HTTPS port in the firewall
```bash
sudo firewall-cmd --permanent --add-service=https
sudo firewall-cmd --reload
```

### 6. Test the config and start nginx
```bash
sudo nginx -t
sudo systemctl enable nginx
sudo systemctl restart nginx
```

### 7. Verify from the jump host
```bash
curl -Ik https://stapp01/
```
Expected: `HTTP/1.1 200 OK`

## How it works
- A TLS certificate (`.crt`) is the public part that Nginx sends to clients. The private key (`.key`) proves Nginx owns it and must stay secret.
- **Self-signed** means no trusted authority vouches for it, so browsers and curl warn about it. That's why the test uses `curl -k`, which skips certificate verification.
- `curl -I` fetches only the headers, which is enough to confirm the server answers.
- `nginx -t` checks the config for syntax errors before restarting, so a typo doesn't take the server down.

## Key Takeaway
- Always run `nginx -t` before restarting.
- Keep private keys in a protected directory and never commit them to Git.
- Opening the port in the firewall is a separate step from configuring the web server. A working config still won't be reachable if the port is blocked.
- The document root for the package install is `/usr/share/nginx/html`.
