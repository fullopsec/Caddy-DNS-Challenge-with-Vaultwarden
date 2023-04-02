# Caddy-DNS-Challenge-with-Vaultwarden

Vaultwarden forces us to use SSL.

I don't want to open any ports on my internet Box.

That why we are using Caddy DNS challenges to get our certificates.

__________________________



Go create a custom token for the API of cloudflare(I use cloudflare)

```
In the upper right, click the person icon and navigate to My Profile, and then select the API Tokens tab.
Click the Create Token button, and then Use template on Edit zone DNS.
Edit the Token name field if you prefer a more descriptive name.
Under Permissions must be,  Zone / DNS / Edit 
Add another permission: Zone / Zone / Read.
Under Zone Resources, set Include / Specific zone / example.com 
Under TTL, set an End Date for when your token will become inactive. You might want to choose one far in the future.
Create the token and copy the token value.
```

Test your token with the provided command.

Copy your API token!

Go download the caddy dns wrapper with cloudflare
>https://caddyserver.com/download?package=github.com%2Fcaddy-dns%2Fcloudflare

Make a directory on your server and put the downloaded file into it.

Rename the file as "caddy"

Copy caddy in /usr/local/bin/:

`sudo cp caddy /usr/local/bin/caddy`

Make it executable:

`sudo chmod +x /usr/local/bin/caddy`

`sudo chmod +x caddy`


Give the caddy binary the ability to bind to privileged ports :

`sudo setcap cap\_net\_bind_service=+ep /usr/local/bin/caddy`

Create a file named "Caddyfile" with the following content:

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

Create temp variables to test the Caddyfile:

```
export CLOUDFLARE_TOKEN=<YOUR_TOKEN>
export LOG_FILE=/data/access.log
```

Start caddy by running

`caddy run`

In your DNS(cloudflare for me) add the desired subdomain for the service you are going to install(Vaultwarden here)

You must use the **LOCAL** IP of your server as content.

You must not proxy the requests(DNS only)

![image](https://user-images.githubusercontent.com/114147068/229375994-2687dc99-bd8b-4de7-9f2b-d149a22a782a.png)

Now we need to make the new vaultwarden docker-compose file:

Put the compiled caddy binary in the same folder as the compose

docker-compose.yml:

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
      DOMAIN: "https://vaultwarden.fullopsec.com"  # Your domain.
      EMAIL: "fullopsec@fullopsec.com"                 # The email address to use for ACME registration.
      CLOUDFLARE_TOKEN: "<YOUR_TOKEN>"                   # Your DNS token.
      LOG_FILE: "/data/access.log"
```

You can change any path as ou wish, this is an exemple of a simple setup with everything in the same folder.

Dont Forget to put your CLOUDFLARE_TOKEN and the correct DOMAIN (which include the newly created subdomain).

Try it:

`sudo docker compose up -d`

Then browse to the DOMAIN (https://vaultwarden.fullopsec.com here)
