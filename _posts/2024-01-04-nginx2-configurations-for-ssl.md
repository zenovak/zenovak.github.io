---
title: NGINX - 2 Configurations for SSL

date: YYYY-MM-DD HH:MM:SS +/-TTTT #TTTT Means timezone
categories: [Web, Nginx]
tags: [web, nginx, http, https, ssl]     # TAG names should always be lowercase
---


# About
Previously we have seen a rough predefined field of the default configuration file, and managed to launch our website. 

This time we will go over setting up some  of the other commonly required services like HTTPS.

We can begin by completely deleting the entire contents of the default config site at:
```
sudo nano /etc/nginx/sites-available/default
```
Dont worry, we will learn how to write the configurations ourselfs.

<br>

## The minimally required block
You may copy the following code into the config file. Change the IP_ADDRESS to the server's external IP address. Do note its just the ip address. not `https://34.150.150.150/` or anything fancy. Once done, save, exit, and restart nginx.

```
server {
    listen 80;  
    server_name IP_ADDRESS;
    
    location / {
        include proxy_params;  
        proxy_pass http://localhost:5000/;  
    }
}
```

- `proxy_pass` here is where your application is running on localhost. Besure to change the port number!
- `listen`  defines the port where users from the internet accesses your website. Nginx will forward requests from this port to the `proxy_pass` url
- `server_name` is where NGIX receives requests from. This should be the external ip address of your VPS or your domain name

Once you have restarted nginx you'll notice your site is still running without any issue. This is the minimally required configuration to run your site.

### About server_name
In the default configurations we saw they had used  `server_name _;` instead. This is a wildcard declaration of saying "for every server_name I received..."

This will be useful later on when we filter for requests that used a DNS wildcard (`www.site.com` instead of `site.com`) or IP address. But for now, lets keep it using our IP address.

<br><br>

---
# Advanced connections


### Websockets
If your application uses websockets to push realtime changes on the webpage, using some of the following libraries:
- socketIO (python)

You will need to extend your configuration to include the following new parameters:
```
...
location / {
    include proxy_params;
    proxy_pass http://localhost:5000/;
    proxy_http_version 1.1;
    proxy_set_header Upgrade $http_upgrade;
    proxy_set_header Connection "upgrade";
    proxy_set_header Host $host;
}
```

<br>

## SSL (Https)
Our localhost applications will never use a HTTPS unless specified or created otherwise. Enabling HTTPS is often done within nginx configurations.

SSL requires the following components:
- domain name (not IP addresses).
- SSL Certificate 

Luckily we do not need to get the certificate and configure nginx ourself, and we can use certbot for this. [Reference](https://medium.com/@bjnandi/how-to-install-free-ssl-tls-certificate-on-nginx-web-server-in-ubuntu-22-04-lts-ef3d751de166), [2](https://certbot.eff.org/instructions?ws=nginx&os=debianbuster)


> [!INFO] Dependency
> Certbot uses the snap command to install. Besure snap is installed. Get it from here:
> https://snapcraft.io/docs/installing-snap-on-debian


```
sudo apt-get remove certbot
sudo snap install --classic certbot
sudo ln -s /snap/bin/certbot /usr/bin/certbot
```


Then run the certbot to inject nginx configrations and get our ssl cert automatically:
```
sudo certbot --nginx -d yourdomain.com
```

To confirm that it is working, visit the `https` variant of your website.

<br>

## SSL(HTTPS) Manual setup
> [*Reference*](https://www.digicert.com/kb/csr-ssl-installation/nginx-openssl.htm)

An option to use SSL without domain name is to do a self signed certificate. Which will show a warning on chrome when a user visits your website.

You will need to configure Nignx SSL and generate your own SSL certificate using openSSL

### Step 1: Generate your 2 SSL files
SSL configuration comes in 2 files, note the file extensions
- `domain.name.pem` or `bundle.crt`
- `domain.name.key`

Here's how each file looks like:
```
-----BEGIN CERTIFICATE-----
MIIEnDCCA4SgAwIBAgIURFoO1AM2RclKdKmWouIlOYD5OfEwDQYJKoZIhvcNAQEL
BQAwgYsxCzAJBgNVBAYTAlVTMRkwFwYDVQQKExBDbG91ZEZsYXJlLCBJbmMuMTQw
MgYDVQQLEytDbG91ZEZsYXJlIE9yaWdpbiBTU0wgQ2VydGlmaWNhdGUgQXV0aG9y
aXR5MRYwFAYDVQQHEw1TYW4gRnJhbmNpc2NvMRMwEQYDVQQIEwpDYWxpZm9ybmlh
MB4XDTI0MDUyOTA4MTcwMFoXDTM5MDUyNjA4MTcwMFowYjEZMBcGA1UEChMQQ2xv
dWRGbGFyZSwgSW5jLjEdMBsGA1UECxMUQ2xvdWRGbGFyZSBPcmlnaW4gQ0ExJjAk
BgNVBAMTHUNsb3VkRmxhcmUgT3JpZ2luIENlcnRpZmljYXRlMIIBIjANBgkqhkiG
9w0BAQEFAAOCAQ8AMIIBCgKCAQEA1F3Y/Ha5ilCwy0h5O4kjS4qYH7yrvMJ89TaQ
7WBXjdaTwf9g7kF3PCka/f/lA/LH4PldgCaTW992ww0suXd11xaC4UOuE1LyCX7W
pm2WdIgwy744cFjKYvq20HOQPfWd3imWn9k6Oxmkh4hkvT+QNJ4fOCbPN7hA+LMR
KU19n3dYsJ2JZN/z9FzgH4uejIufPBkqRccZQaTylCSI+p6pcGpa6JSc6CL5zyv8
vsJkXakvOGAqWDOgLY4UTlZBlP2b+v9j/DoMU6uXJp/ZG2fagQIDAQABo4IBHjCC
ARowDgYDVR0PAQH/BAQDAgWgMB0GA1UdJQQWMBQGCCsGAQUFBwMCBggrBgEFBQcD
ATAMBgNVHRMBAf8EAjAAMB0GA1UdDgQWBBSXxENWD+kbBP7S/+EQDvjOZWliEDAf
BgNVHSMEGDAWgBQk6FNXXXw0QIep65TbuuEWePwppDBABggrBgEFBQcBAQQ0MDIw
MAYIKwYBBQUHMAGGJGh0dHA6Ly9vY3NwLmNsb3VkZmxhcmUuY29tL29yaWdpbl9j
K6AphidodHRwOi8vY3JsLmNsb3VkZmxhcmUuY29tL29yaWdpbl9jYS5jcmwwDQYJ
KoZIhvcNAQELBQADggEBACwVfMjiIcrBpvDrCd9F25xoQh4C86Hb7bq4tZM5Pfx9
QNNyXVfYBt8rK5hb0gK1sbXuRWwLNqk41+CWLyvVEFNmeNirlVsOaWcRkWnWco0n
8lcIecjdWr2KYkP0LlmPFkAZ0AIpPMoKXrblk0lLGLr1D4nVCTs4x9IWgX91Jl1X
pRemeOuc56jjRVvd/B7Pvrrcl1tZuzd2OZuU4W8UgNjTKYdit+NC1y/fppjnAq3q
7DKBnUVcR0v+yySH/b0h2fFrFha1WC4TPRAyZlo4P61PuhnOApBaVXqipJ+RnKWe
Esg6kMEtJ9SkGzz5YuCK7I/290jKMSGxJSlyEOMBtmw=
-----END CERTIFICATE-----
```

```
-----BEGIN PRIVATE KEY-----
MIIEvQIBADANBgkqhkiG9w0BAQEFAASCBKcwggSjAgEAAoIBAQDUXdj8drmKULDL
SHk7iSNLipgfvKu8wnz1NpDtYFeN1pPB/2DuQXc8KRr9/+UD8sfg+V2AJpNb33bD
DSy5d3XXFoLhQ64TUvIJftambZZ0iDDLvjhwWMpi+rbQc5A99Z3eKZaf2To7GaSH
iGS9P5A0nh84Js83uED4sxHfT1lcaHejf1m+VD0HWhMBvPUfMH6mrCyik0jWI6VZ
JIj6nqlwalrolJzoIvnPK/y+wmRdqS84YCpYM6AtjhROVkGU/Zv6/2P8OgxTq5cm
n9kbZ9qBAgMBAAECggEAKjU61ra8Grb95HFTkxcnGjECGjC0C3C2SEBfnqZK0IiI
R9NXQVEMbI66/gQx83aZOVoXs06H6c4nar6bkxeKkEKrHsxr2/21gBiLIVCSiLBn
wwjcUlipKsmrKStMRUTNcpVKJ1Lr0+/JtSBLjsL7hUxWKEQz/WnA14XMTz5ZsVGk
zYHF/KDd0G/i7NT7CYTsyqRMDE84dFXUteT+Rb9cqrIvF4JgbHpkw2qlTEXyG4Ne
UKZgCHoVK9zWpop5Z+bMOEu3aVyNW8pBx0axP0JjBwKBgQD7BSlY8xvyvIGA8DRE
7tEvSonY1TWFVvk1mTOeLxE63J7hdTSY5zIdv6+EMlH3DizxGfEEf7Uapp/Rq9xE
7G21NUHZMiQgXpm2QL/iT16OZNVzom9LfV6/7Xkkwn83d+cDOzBPaM+y209rbrt4
xLByJhF2bN0Jg6t8iqaijhGOvwKBgQDYlGEAkFFjRfcajCADdepqP5ygxh9UnPBd
3g1TrziBTYnZx6LR7mjJXmm/1lglLgraEZjJcIj4yHPeWtgu8AAS2yircU0XBlso
IIj6TP+I8CXfqglfKGtEuXMSp0jbj6zt85ocEhEuixfS1isaXo+mURpj+Oyi5Tvo
sqoRJGImvwKBgD9JVtpZOKOjSRdD+Dmk6FJ+/XAQmRTMD7qmrG/mN/baJqh7D065
g1YivNKciTO7fDMxMiXLONLGTabkKH2sCiDUk4x56sfKcgCUJIyfLBzEaVhlDKBA
tIG5EoDlFIPck/6pjo2GxE64ojZYzaUuGbo9xMtRuQysCLE2l7qGDQErAoGBAITC
Z7e9v3YYFEHctV8Jr/kTJ0LST7BBR4JytD6hAQUZ769klaUT/H27dx1WGdAoqhRE
hyCr7/p4fbZGf5A+I/1rBEIbgMLlbYlqcCzmeBmMA9tX0sjW8PI+r5A2pQ2Zw8pU
8hnU5V1fe+oMyH0wi+PKgV/Y3c14sUGSC3fYkqXnAoGASjvjGodFUG5pexdt7ktw
WKfkb+YF8bdzeeOnjvwJDKrEvnWLnWSAE8t0UiN4n0d+BYb/BCKAK896Z5xQoDdR
Upw57vTbId5QGh1Kobv/v7P0iGm03zX65K9mrgS+VU+/07ZIQcBQlVd+fRf5/Qhx
y+hTSi8wniL5ukxu8up60o4=
-----END PRIVATE KEY-----

```


> [!INFO] I did not get a `.pem`
> Some SSL generators will create bundle files instead of `.pem` this is normal as SSL certificates are consists of a primary and intermediate files (crt) you will have to concatonate them. Read more [here](https://www.digicert.com/kb/csr-ssl-installation/nginx-openssl.htm)

<br>

### Step 2: Add them to server
Next you will need to write these contents to your nginx server respectively:
```
sudo nano /etc/ssl/stone.com.pem
sudo nano /etc/ssl/stone.com.key
```

Make sure to use your own domain names!

<br>

### Step 3: Configure Nginx
Then within the configuration file:
```
sudo nano /etc/nginx/sites-available/default
```

we add the following directive to indicate SSL usage
```
server {
    listen 443 ssl;
    server_name stone.com;
    
    ssl_certificate     "/etc/ssl/stone.com.pem";
    ssl_certificate_key "/etc/ssl/stone.com.key";
    
    location / {
        include proxy_params;
        proxy_pass http://localhost:3000/;
        proxy_http_version 1.1;
        proxy_set_header Upgrade $http_upgrade;
        proxy_set_header Connection "upgrade";
        proxy_set_header Host $host;
    }
}
```

That's it. Save, exit, test, and restart nginx. Your site can now be accessed via HTTPS
```
sudo nginx -t
sudo systemctl restart nginx
```


<br><br>

---
# Hiding IP Addresses and enforce domain name


### [SSL Security concerns](https://askubuntu.com/questions/1151042/only-allow-access-to-nginx-server-when-using-the-full-domain-name-and-not-the-ip)
When you have registered a domain name, you'll notice how you can still access your website directly via IP, circumventing the domain name. This is not good as it will allow attackers to bypass DNS security infrastructures like Cloudflares'

To address this gap, we must configure our nginx to only allow connections via domain name:
```
sudo nano /etc/nginx/sites-available/default
```

We need to define a default server block in which all connection attempts made via IP will be immediately rejected:
```
# The default always sits on top. 

# Normal http block
server {
    listen 80;
    server_name stone.com;
    location / {
        include proxy_params;
        proxy_pass http://localhost:3000/;
        proxy_http_version 1.1;
        proxy_set_header Upgrade $http_upgrade;
        proxy_set_header Connection "upgrade";
        proxy_set_header Host $host;
    }
}
# The normal SSL block
server {
    listen 443 ssl;
    server_name lwp.asia;
    
    ssl_certificate     "/etc/ssl/stone.com.pem";
    ssl_certificate_key "/etc/ssl/stone.com.key";
    
    location / {
        include proxy_params;
        proxy_pass http://localhost:3000/;
        proxy_http_version 1.1;
        proxy_set_header Upgrade $http_upgrade;
        proxy_set_header Connection "upgrade";
        proxy_set_header Host $host;
    }
}

# Our catch all. This will reject all http connection via IP
server {
    listen      80 default_server;
    listen      [::]:80 default_server;
    server_name "";
    return      444;
    # Or alternatively redirect to domain
    # return 301 http://stone.com$request_uri;
}
```

But wait, we can also access the website via direct IP using HTTPS! We need extend to our default block for direct HTTPS as well:
```
server {
  listen 80 default_server;
  listen [::]:80 default_server;
  listen 443 default_server;
  listen [::]:443 default_server;

  ssl_reject_handshake on;

  server_name "";
  return 444;
}
```
[*stackoverflow*](https://stackoverflow.com/questions/29104943/how-to-disable-direct-access-to-a-web-site-by-ip-address)

<br>

### [With Cloudflare, too many redirects](https://stackoverflow.com/questions/41583088/http-to-https-nginx-too-many-redirects)
Sometimes with cloudflare's DNS redirect requests we may encounter too many redirects error when we configure nginx to redirect http requests to https

```
# Normal http block
server {
    listen 80;
    server_name lwp.asia;
    return 301 https://$host$request_uri; 
}
```

The problem is caused by cloudflare always using http to connect to your origin server. To solve this you must configure on cloudflare side to use `strict SSL`, thus rendering your http nginx config not used at all.

<br>


