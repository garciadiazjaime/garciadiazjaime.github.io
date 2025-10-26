---
layout: post
title: "Setting a Reverse Proxy with Nginx"
date: 2025-10-27 10:00:00 -0500
categories: nginx reverse-proxy software-architecture
---

![Setting a Reverse Proxy with Nginx](/assets/nginx-reverse-proxy/banner.png)

Last week, I needed to run a POC and wanted to use a reverse proxy so that the URLs looked clean during the demo. The demo was running locally, and I didn’t really need the reverse proxy — but since it was simple to set up, why not? So I’m leaving here the steps I followed.

### 1. Install `nginx`

I already had `nginx` installed, and I’m on macOS. If you don’t have it installed, you can use brew.

```sh
brew install nginx
```

### 2. `demo.conf`

I created a `demo.conf` file in this directory:

```sh
/opt/homebrew/etc/nginx/servers/demo.conf
```

That’s usually where Homebrew installs packages, and any file named `*.conf` will be read automatically by Nginx.

Nginx allows you to define a reverse proxy, which is a way to configure a server — in my case, `demo.local` — and then, based on the path, redirect the request to one location or another. For example, when I open `http://demo.local`, the request is served from `http://127.0.0.1:3000/`, which is a simple Next.js application. And whenever I open `http://demo.local/coupons`, the request is served from `https://coupons.garitacenter.com/`.

Notice the transparency: without knowing that there’s a reverse proxy, it would be hard to guess that we’re interacting with two different applications.

Here’s the Nginx configuration to route the requests:

```sh
# /opt/homebrew/etc/nginx/servers/demo.conf
server {
    listen 80;
    server_name demo.local;

    location /coupons {
        proxy_pass https://coupons.garitacenter.com/;
        proxy_set_header Host coupons.garitacenter.com;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto $scheme;
        proxy_ssl_server_name on;
    }

    location / {
        proxy_pass http://127.0.0.1:3000/;
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto $scheme;
    }
}
```

As you can see, it listens on port `80` and assigns the `server_name` to `demo.local`. Then, based on the path (`/coupons` vs `/`), it redirects the request to one place or another.

Any change made to this file requires Nginx to be restarted.

```sh
brew services restart nginx
```

### 3. Update `hosts`

Additionally, I needed to update my `hosts` configuration file so that `demo.local` points to my `localhost`.

```sh
# /etc/hosts
127.0.0.1       demo.local
```

Changes here are picked up automatically by the machine, so as soon as the file is modified, you can start using `demo.local`.

In the following image, notice how `http://demo.local/nginx-reverse-proxy` and `http://localhost:3000/nginx-reverse-proxy` render the same content.

![Nginx reverse proxy](/assets/nginx-reverse-proxy/localhost_and_custom_url.png)

Now let’s take a look at the other location:

![Other location](/assets/nginx-reverse-proxy/other_location.png)

---

Browsers nowadays, by default, try to use `https`, which means the above instructions might work in incognito mode, but in a regular session you might see a **“Your connection is not private”** message.

![Your connection is not private Browser Error](/assets/nginx-reverse-proxy/browser-error.png)

## Certificates

### 4. Install `mkcert`

```sh
brew install mkcert nss
```

### 5. Create certificates

```sh
mkcert -install

mkcert -cert-file /opt/homebrew/etc/nginx/certs/demo.local.pem -key-file /opt/homebrew/etc/nginx/certs/demo.local-key.pem demo.local localhost 127.0.0.1
```

Note: If you need to use `sudo` to create the certificates, make sure to change the owner of the files to your regular user afterwards.

```sh
ls -la /opt/homebrew/etc/nginx/certs/

-rw-------@  1 myuser  admin  1708 Oct 19 14:26 demo.local-key.pem
-rw-r--r--@  1 myuser  admin  1562 Oct 19 14:26 demo.local.pem
```

### 6. Update `demo.conf`

```sh
server {
    listen 80;
    server_name local.media.patientpoint.io;

    # Redirect HTTP to HTTPS
    return 301 https://$server_name$request_uri;
}

server {
    listen 443 ssl;
    server_name demo.local;

    ssl_certificate /opt/homebrew/etc/nginx/certs/demo.local.pem;
    ssl_certificate_key /opt/homebrew/etc/nginx/certs/demo.local-key.pem;

    location /coupons {
        proxy_pass https://coupons.garitacenter.com/;
        proxy_set_header Host coupons.garitacenter.com;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto $scheme;
        proxy_ssl_server_name on;
    }

    location / {
        proxy_pass http://127.0.0.1:3000/;
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto $scheme;
    }
}
```

### 7. Restart Nginx

```sh
brew services restart nginx
```

You should see output like this:

```sh
$ brew services restart nginx

Stopping `nginx`... (might take a while)
==> Successfully stopped `nginx` (label: homebrew.mxcl.nginx)
==> Successfully started `nginx` (label: homebrew.mxcl.nginx)
```

If everything went well, you can now try the URL again, and the browser shouldn’t complain about security:

![HTTPS Enable For Localhost](/assets/nginx-reverse-proxy/nginx-https.png)

### How to debug

If you face any issues, you can check the Nginx logs:

- Error logs:

```sh
tail -f /opt/homebrew/var/log/nginx/error.log
```

- Access logs:

```sh
tail -f /opt/homebrew/var/log/nginx/access.log
```

These two files should give a good indication of what’s going on. Sometimes it’s a permission issue (e.g., the certificates are owned by `admin`), or they might not be found because of a wrong path, or the request might be missing some headers. In any case, when facing an error, these two logs are where you’ll find the root cause.

## Summary

This is a simple example of how to use `nginx` as a `reverse proxy` on a `local machine`. Now, two different web applications can be accessed through a custom URL, and based on the path — either root `/` or `/coupons` — the responses come from different servers.

Although this setup is running locally, the concept applies in production. In many cases, a proxy will sit in front of our services. Of course, in production you would put more effort into security, caching, and possibly work with a vendor like AWS, but the concept remains the same.
