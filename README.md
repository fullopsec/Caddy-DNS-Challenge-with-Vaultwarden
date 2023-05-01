# Caddy-DNS-Challenge-with-Vaultwarden

This is a guide to configure Caddy with Vaultwarden using Cloudflare DNS challenges to obtain SSL certificates.

## Video tutorial

https://www.youtube.com/watch?v=0Ri3GVDc4pM

## Requirements
* A domain name linked to a Cloudflare account.
* Docker and Docker Compose installed on your server.

## Setup
1. Create a custom token for the Cloudflare API by following these steps:
    * Click the person icon on the upper right and go to My Profile, then select the API Tokens tab. URL: https://dash.cloudflare.com/profile/api-tokens
    * Click the Create Token button, then go "Create Custom Token".
    * Edit the Token name field if you prefer a more descriptive name.
    * Under Permissions must be, choose Zone / DNS / Edit.
    * Add another permission: Zone / Zone / Read.
    * Under Zone Resources, set Include / Specific zone / example.com (replace example.com with your domain).
    * Under TTL, set an End Date for when your token will become inactive. You might want to choose one far in the future.
    * Create the token and copy the token value.
2. Test your token with the provided command.
3. Download the Caddy DNS wrapper for Cloudflare from https://caddyserver.com/download?package=github.com%2Fcaddy-dns%2Fcloudflare.
4. Put the downloaded file in a directory on your server, then rename it as "caddy".
5. Copy caddy to /usr/local/bin/ by running `sudo cp caddy /usr/local/bin/caddy`.
6. Make it executable by running `sudo chmod +x /usr/local/bin/caddy` and `sudo chmod +x caddy`.
7. Give the caddy binary the ability to bind to privileged ports by running `sudo setcap cap_net_bind_service=+ep /usr/local/bin/caddy`.
8. Create a file named "Caddyfile" with the following content:
```
{$DOMAIN}:443 {
  log {
    level INFO
    output file {$LOG_FILE} {
      roll_size 10MB
      roll_keep 10
    }
  }

  # Use the ACME DNS-01 challenge to get a cert for the configured domain.
  tls {
    dns cloudflare {$CLOUDFLARE_TOKEN} 
  }

  # Disable this setting if you have compatibilities issues in browsers
  encode gzip

  # Notifications redirected to the WebSocket server
  reverse_proxy /notifications/hub vaultwarden:3012

  # Proxy everything else to Rocket
  reverse_proxy vaultwarden:80
}
```
9. Create temporary variables to test the Caddyfile by running:
```
export CLOUDFLARE_TOKEN=<YOUR_TOKEN>
export LOG_FILE=/data/access.log
```
10. Start Caddy by running `caddy run`. Stop it after a few seconds when everything seems loaded.
11. In your DNS (Cloudflare for this guide), add the desired subdomain for the service you are going to install (Vaultwarden in this case). Use the **LOCAL** IP of your server as content and do not proxy the requests (DNS only).
12. Create a docker-compose.yml file with the following content:
```
version: '3'

services:
  vaultwarden:
    image: vaultwarden/server:latest
    container_name: vaultwarden
    restart: always
    environment:
      WEBSOCKET_ENABLED: "true"  # Enable WebSocket notifications.
    volumes:
      - ./vaultwarden:/data

  caddy:
    image: caddy:2
    container_name: caddy
    restart: always
    ports:
      - 80:80
      - 443:443
    volumes:
      - ./caddy:/usr/bin/caddy  # Your custom build of Caddy.
      - ./Caddyfile:/etc/caddy/Caddyfile:ro
      - ./config:/config
      - ./data:/data
    environment:
      DOMAIN: "https://<YOUR_SUBDOMAIN>.<YOUR_DOMAIN>"  # Your domain. Example: https://vaultwarden.fullopsec.com
      EMAIL: "<YOUR_USERNAME>@<YOUR_DOMAIN>"                 # The email address to use for ACME registration. Example fullopsec@fullopsec.com
      CLOUDFLARE_TOKEN: "<YOUR_TOKEN>"                   # Your DNS token.
      LOG_FILE: "/data/access.log"
```

You can change any path as you wish. This is an example of a simple setup with everything in the same folder.

Don't forget to put your CLOUDFLARE_TOKEN and the correct DOMAIN.

13. Launching:

`sudo docker compose up -d`

Then browse to the subdomain (https://vaultwarden.fullopsec.com in my case)
