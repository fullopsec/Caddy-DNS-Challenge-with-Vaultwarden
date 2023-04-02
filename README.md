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

Give the caddy binary the ability to bind to privileged ports :

`sudo setcap cap\_net\_bind_service=+ep /usr/local/bin/caddy`

Create a file named Caddyfile with the following content:
