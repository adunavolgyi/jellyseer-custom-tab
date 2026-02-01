# Accessing Jellyseer as a Custom Tab in Jellyfin

This repository is a guide on how to achieve a Jellyseer and Jellyfin connection, revealing Jellyseer's interface as a custom tab in Jellyfin by managing authentication cookies. The method described here is my setup, so different solutions might exist for other specific cases. Regardless, you can either copy it or suggest improvements.

## Requirements

 - Jellyfin: v10.11.5
 - Jellyseer: v2.7.3
 - IAmParadox's [Custom Tabs](https://github.com/IAmParadox27/jellyfin-plugin-custom-tabs) plugin: v0.2.5.0
 - IAmParadox's [File Transformation](https://github.com/IAmParadox27/jellyfin-plugin-file-transformation) plugin: v2.5.2.0
 - Nginx Proxy Manager: v2.13.5 (or [NPMPlus](https://github.com/ZoeyVid/NPMplus) v2.12.3 fork)

In my instance, I used NPMPlus, since I required GeoIP blocking for my services. I have checked and it works with regular Nginx Proxy Manager too, but I am detailing the NPMPlus settings here. Most of these can also be found in Nginx.

## Jellyseer Settings

### General

- Application URL: `https://jellyseer.domain.com`

### Users

- Disable Local Logins  
- Enable Jellyfin Logins  
 
### Jellyfin

- External URL: `https://jellyfin.domain.com`

## NPM Plus (or NPM) Settings

For this to work, the same domain should be used for both the Jellyfin and Jellyseer instances, with different subdomains. In this example, I am going to set:

- Jellyfin to `https://jellyfin.domain.com/`
- Jellyseer to `https://jellyseer.domain.com/`

The subdomain can be changed to unique ones (stream, requests, etc.).

### Common Settings for the Proxies

- General: WebSockets **ON** ✅  
- SSL/TLS: Enable HSTS and security headers **OFF** ❌  

### Jellyseer Proxy Settings

**Advanced → Custom settings:**

```nginx
# --- Required proxy headers ---
proxy_set_header Host $host;
proxy_set_header X-Forwarded-Host $host;
proxy_set_header X-Forwarded-Proto $scheme;
proxy_set_header X-Forwarded-Port 443;
proxy_set_header X-Forwarded-For $remote_addr;
proxy_set_header X-Forwarded-Ssl on;

# --- WebSocket support (important for Jellyseerr) ---
proxy_http_version 1.1;
proxy_set_header Upgrade $http_upgrade;
proxy_set_header Connection "upgrade";

# --- Allow Jellyfin to embed Jellyseerr ---
proxy_hide_header X-Frame-Options;
add_header X-Frame-Options "ALLOW-FROM https://stream.felhohub.xyz" always;

add_header Content-Security-Policy "frame-ancestors 'self' https://stream.felhohub.xyz" always;

```

### Jellyfin Proxy Settings

**Advanced → Custom settings:**
```nginx
# --- Required proxy headers ---
proxy_set_header Host $host;
proxy_set_header X-Forwarded-Host $host;
proxy_set_header X-Forwarded-Proto $scheme;
proxy_set_header X-Forwarded-Port 443;
proxy_set_header X-Forwarded-For $remote_addr;
proxy_set_header X-Forwarded-Ssl on;

# --- Jellyfin absolutely needs WebSockets ---
proxy_http_version 1.1;
proxy_set_header Upgrade $http_upgrade;
proxy_set_header Connection "upgrade";
```

### Jellyfin Custom Tab HTML
Add a new tab of the Custom Tabs plugin and name it (e.g. Discover New).
Paste this HTML to the **Html Content** field:
```html
    <style>
        :root {
            --save-gut: max(env(safe-area-inset-left), 3.3%);
        }
       @media only screen and (max-width: 990px) {
           .requestIframe{
               height: calc(100vh - 105.15px)!important;
               margin-top: -6.15px!important;
           }
       } 
        .requestIframe {
            width: 100%;
            height: calc(100vh - 50.716px);
            position: absolute;
            border: 0;
            margin-top: -20.716px;
        }
    </style>
    <div class="sections">
      <iframe class="requestIframe" src="https://jellyseer.domain.com"></iframe>
    </div> 
```
Change the `https://jellyseer.domain.com` domain accordingly. After that, delete the cache and force refresh the client that you are using (Web Client, mobile apps, JMP). The new tab should appear in the client, with Jellyseer popping up. With this method, authentication cookies should also work, and you should be able to log in to your account.

## Issues and Pull Requests
Please send an issue if you encounter any problems with this method. Also, if you have managed to make this happen using another method, please send a pull request to this repository updating this README, and I will be happy to include it in this guide to spread this knowledge.
