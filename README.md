# Accessing Jellyseer as a Custom Tab in Jellyfin
This repository is a guide on how to achieve Jellyseer and Jellyfin connection, revealing Jellyseer's interface as a custom tab in Jellyfin by managing authentication cookies.
The method described here is my setup, so different solutions might exist for other specific cases. Regardless, you can either copy it or suggests.
### Requirements
 - Jellyfin: v10.11.5
 - Jellyseer: v2.7.3
 - IAmParadox's [Custom Tabs](https://github.com/IAmParadox27/jellyfin-plugin-custom-tabs) plugin: v0.2.5.0
 - IAmParadox's [File Transformation](https://github.com/IAmParadox27/jellyfin-plugin-file-transformation) plugin: v2.5.2.0
 - Nginx Proxy Manager: v2.13.5 (or [NPMPlus](https://github.com/ZoeyVid/NPMplus) v2.12.3 fork)

In my instance I used NPMPlus, since I required GeoIP blocking for my services. I have checked and it works with regular Nginxy Proxy Manager too, but I am detailing the NPMPlus settings here. Most of these can also be found in Nginx.

### Jellyseer Settings:
General:
 - Application URL: https://jellyseer.domain.com
 
Users:
 - Disable Local Logins   
 - Enable Jellyfin Logins
 
Jellyfin:
 - Extrernal URL: https://jellyfin.domain.com

### NPM Plus (or NPM) settings:
For this to work. the same domain should be used for both the Jellyfin and Jellyseer instances, with different subdomains. In this example, I am going to set:
 - Jellyfin to https://jellyfin.domain.com/
 - Jellyseer to https://jellyseer.domain.com/

The subdomain can be changed to unique (stream and requests, etc) ones.

Commong settings for the proxies should be:
General: WebSockets ON ✅
SLS/TLS: Enable HSTS and security headers: OFF ❌

Jellyseer proxy settings: 
Advanced, Custom settings:

    proxy_set_header X-Forwarded-Proto $forward_scheme;
    proxy_set_header X-Forwarded-Ssl on;
    proxy_set_header Host $server;
    add_header Content-Security-Policy "upgrade-insecure-requests" always;
    add_header Content-Security-Policy "block-all-mixed-content" always;

Jellyfin proxy settings:
Advanced, Custom settings:

    proxy_set_header Host $host;
    proxy_set_header X-Forwarded-Host $host;
    proxy_set_header X-Forwarded-Proto $forward_scheme;
    proxy_set_header X-Forwarded-Port 443;
    proxy_set_header X-Forwarded-Ssl on;

### Jellyfin Custom Tab HTML
Add a new tab of the Custom Tabs plugin and name it (e.g. Discover New).
Paste this HTML to the Html Content field:

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

Change the domain accordingly. After that, delete the cache and force refresh the client that you are using (WebClient, Mobile apps, JMP). 
The new tab should appear in the client, with Jellyseer popping up. With this method, authentication cookies should also work and you should be able to log in to your account.

## Issues and Pull Requests
Please send an issue if you encounter any problems with this method. Also, if you have managed to make this happen with another method, please send a pull requests to this repository updating this README, and I will be happy to include it in this guide to spread this knowledge.
