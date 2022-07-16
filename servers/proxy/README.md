# Server setups

## proxy server

Debain 11 running an nginx reverse proxy with TLS termination
`apt update && apt install nginx`

### nginx

[https://www.hostwinds.com/tutorials/nginx-reverse-proxy-with-ssl](https://www.hostwinds.com/tutorials/nginx-reverse-proxy-with-ssl)

edit the configs to point to internal ip/port

delete link to default in `/etc/nginx/sites-enabled`

### acme.sh

Originally planned on using certbot but acme.sh doesn't need snap

```
wget -O -  https://get.acme.sh | sh -s email=my@example.com
acme.sh --issue -d <DOMAIN NAME HERE> --standalone
```
Make sure to create the folders used below first
```
acme.sh --install-cert -d <DOMAIN NAME HERE> \
--key-file       /etc/letsencrypt/live/<DOMAIN NAME HERE>/privkey.pem  \
--fullchain-file /etc/letsencrypt/live/<DOMAIN NAME HERE>/fullchain.pem \
--reloadcmd     "service nginx force-reload"
```