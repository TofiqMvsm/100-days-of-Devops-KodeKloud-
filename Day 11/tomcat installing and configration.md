# Day 11: Deploy a Java Application on Tomcat (App Server 3)

## Task
Deploy a Java application using Apache Tomcat on **App Server 3** (`stapp03`) in the Stratos Datacenter:

1. Install Tomcat on App Server 3.
2. Configure it to run on port `8082` instead of the default `8080`.
3. Deploy the provided `ROOT.war` so the application is available at the base URL.

## Solution

### 1. Connect to App Server 3
```bash
ssh banner@stapp03
```

### 2. Install Tomcat from the package manager
```bash
sudo yum install tomcat -y
```

### 3. Change the port to 8082
```bash
sudo vi /etc/tomcat/server.xml
```
Find the HTTP connector and change `8080` to `8082`:
```xml
<Connector port="8082" protocol="HTTP/1.1" ...
```

### 4. Deploy ROOT.war
`ROOT.war` is the packaged Java application. It lives on the jump host, so copy it to the app server first:
```bash
scp /path/to/ROOT.war banner@stapp03:/tmp/
```
Then on App Server 3, move it into Tomcat's `webapps` directory:
```bash
sudo mv /tmp/ROOT.war /var/lib/tomcat/webapps/
```
A WAR named `ROOT.war` is served at the base URL (`/`) instead of `/ROOT`.

### 5. Enable and start the service
```bash
sudo systemctl enable tomcat
sudo systemctl start tomcat
sudo systemctl status tomcat
```

### 6. Verify
```bash
curl http://stapp03:8082
```
The application's page should be returned.

## What went wrong at first
I spent about 2 hours installing Tomcat manually: downloading the tarball with `wget`, extracting it, and writing my own `tomcat.service` file. The service file had mismatched Tomcat versions and paths, and I kept chasing firewall rules and directories that weren't the problem.

I also didn't realise that `ROOT.war` is the application itself and has to be copied to the server and placed in `webapps`.

The simple solution was `yum install tomcat`. The package supplies the service file, the `tomcat` user and the directory layout.

## Key Takeaway
- Check whether the distro's package manager has what you need before building from source. It is faster and less error-prone.
- Tomcat's port is set in `server.xml` (`/etc/tomcat/server.xml` for the package install).
- A `.war` is a packaged Java web application. Put it in `webapps` and Tomcat deploys it automatically.
- `ROOT.war` is served at `/`, and any other name is served at `/<name>`.
- Restart Tomcat after changing `server.xml`.