# Deploying PHPAdmin with DNS

## 1. First of all you must install all the dependencies needed:

```
sudo apt update
sudo apt install nginx
sudo apt install certbot python3-certbot-nginx
```

Also we must also install Docker, we separate another article to explain it, but here are the commands you must follow:

```
sudo apt-get remove docker docker-engine docker.io containerd runc
sudo apt-get update
sudo apt-get install ca-certificates curl gnupg
sudo install -m 0755 -d /etc/apt/keyrings
curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo gpg --dearmor -o /etc/apt/keyrings/docker.gpg
sudo chmod a+r /etc/apt/keyrings/docker.gpg
echo "deb [arch=$(dpkg --print-architecture) signed-by=/etc/apt/keyrings/docker.gpg] https://download.docker.com/linux/ubuntu $(. /etc/os-release && echo "$VERSION_CODENAME") stable" | sudo tee /etc/apt/sources.list.d/docker.list > /dev/null
sudo apt-get update
sudo apt-get install docker-ce docker-ce-cli containerd.io docker-buildx-plugin docker-compose-plugin
```

If you want to check if everything you need is installed, please follow this commands:

```
nginx -version
docker --version
cerbot --version
```

## 2. DNS configuration

You must go to your DNS provider, in this case we use DonDomino as our DNS provider, add your DNS with thsi information, in this case our domain would be cosmos.szapatar.dev:

```
Type: A (For IPv4 or AAA for IPv6)
Host: cosmos.szapatar.dev
IP Address: Your IP address
TTL: Default
```

After doing this process we must confirm if the domain is already linked with our server, please run this command:

```
nslookup cosmos.szapatar.dev //Your domain
```

The answer must be something like this:

```
Server:		127.0.0.53
Address:	127.0.0.53#53

Non-authoritative answer:
Name:	cosmos.szapatar.dev //Your domain
Address: 157.180.40.190 //Your IP address
```

## 3. Create docker container

In this step you must first create the MySQL database, wit this command:

```
docker run --name some-mysql  -p 3306:3306 -e MYSQL_ROOT_PASSWORD=password -d mysql
```

After we create this we should initialize the PHPAdmin container, but there is a step before that, please follow this command:

```
docker ps
```

The answer must be something like this:

```
CONTAINER ID   IMAGE        COMMAND                  CREATED        STATUS        PORTS
17df41c5ee5a   mysql        "docker-entrypoint.s…"   11 hours ago   Up 11 hours   0.0.0.0:3306->3306/tcp, [::]:3306->3306/tcp, 33060/tcp   some-mysql
```

Please copy the docker CONTAINER ID, it's extremely important for the next step, please follow this command:

```
docker run --name phpmyadmin -d --link thePreviousStepContainerID:db -p 8080:80 phpmyadmin
```

## 4. Install the SSL Certificate

In this case we are using a .dev domain, it means to make everything work we must install a SSL certificate for our website, we can do this following this command:

```
sudo certbot --nginx -d yourDomain
```

The SSL certificate expires everything 90 days, so you must renew it, you can do this following this command:

```
sudo certbot renew --dry-run
```

## 5. NGINX configuration

We use NGINX to manage the requests from the browser, if you have install the VIM editor, please follow this command:

```
vim /etc/nginx/sites-avaliable/cosmos
                              /could we any name
```

And in the VIM editor please paste this command:

```nginx
server {
    listen 80;                          # Listen for incoming HTTP traffic on port 80
    server_name cosmos.szapatar.dev;    # Domain name this server block responds to

    return 301 https://$host$request_uri; # Redirect all HTTP requests to HTTPS (permanent redirect)
}

server {
    listen 443 ssl;                     # Listen for HTTPS traffic on port 443 and enable SSL
    server_name cosmos.szapatar.dev;   # Domain name for HTTPS requests

    # Your SSL certificates (issued by Let's Encrypt in this case)
    ssl_certificate     /etc/letsencrypt/live/cosmos.szapatar.dev/fullchain.pem; # Public certificate chain
    ssl_certificate_key /etc/letsencrypt/live/cosmos.szapatar.dev/privkey.pem;   # Private key for SSL

    ssl_protocols TLSv1.2 TLSv1.3;     # Allow only secure TLS versions (disable older insecure ones)
    ssl_prefer_server_ciphers on;      # Prefer server-defined secure ciphers over client ones

    access_log /var/log/nginx/cosmos.access.log; # File where all successful requests are logged
    error_log /var/log/nginx/cosmos.error.log;   # File where errors and issues are logged

    location / {                       # Match all incoming request paths
        proxy_pass http://127.0.0.1:8080; # Forward requests to backend server running locally on port 8080

        proxy_http_version 1.1;        # Use HTTP/1.1 for backend communication (needed for keep-alive/WebSockets)
        proxy_set_header Host $host;  # Pass the original host (domain) to the backend
        proxy_set_header X-Real-IP $remote_addr; # Send the real client IP address to the backend
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for; # Append client IP to forwarded chain
    }
}

```

- The first `server` block redirects all HTTP traffic to HTTPS.
- The second `server` block handles secure HTTPS connections.
- SSL certificates enable encryption.
- Logs help monitor traffic and debug issues.
- The `location /` block forwards requests to your backend application running on `localhost:8080`.

In NGINX the sites-avalible are the sites you can deploy, but actual deployed sites are in the sites-anable directory, so we should make a copy of out sites-avalible in our sites-enable with this command:

```
sudo ln -s /etc/nginx/sites-available/cosmos /etc/nginx/sites-enabled/
```

Also NGINX by default creates a default enable site, for this exercise you must remove it with this command:

```
rm /etc/nginx/sites-enabled/default2
```

## 6. Reload the NGINX server

Check that everything is working fine:

```
sudo nginx -t
```

If so, restart the NGINX:

```
systemctl reload nginx
```

Then you can check your domain and it will be the phpAdmin login page.
