# Dockerpug

## A small system for docker tests

#### projects

- nginx, ssl and letsencrypt


### nginx, ssl and letsencrypt

#### setup nginx

Start a project folder then in the root create the `compose.yaml` file then try `docker compose up`. Once the images are pulled the server should start, `curl http://localhost` should output the nginx default page

While here create the folders for later
 - `nginx/www`
 - `nginx/conf`
 - `certbot/www`
 - `certbot/conf`

Next step is to only have the first part of the `nginx/conf/nginx.conf` file, before the certs are generated only the port 80 server will work. Wait until later to apply the port 443 section

#### get certs

Restart `docker compose restart` and in another terminal we can start the certbot dance while keeping an eye on the logs to see traffic from letsencrypt

First check that all seems good
`docker compose run --rm  certbot certonly --webroot --webroot-path /var/www/certbot/ --dry-run -d example.com` 
and then remove the `--dry-run` flag and get certs for real

Now certbot should have left the .pem files in our certbot/conf folder we can update the nginx.conf file to have the port 443 section

Restart docker again, make sure to leave a cool website in the nginx/www folder and everything should be working

#### autorenew certs

Not really a docker thing but would be added automatically on a non-docker certbot install, we'll use systemctl timers on the docker host to recheck the certs twice a day (seems to be standard). This will give a bit more visibility of when the command last ran than cron

First test certbot `docker compose run --rm certbot renew --dry-run` If thats all good then we can write the unit files but note carefully that the paths need to be fully declared including the /path/to/the/compose.yaml !!!

Included is a post-hook which will restart nginx so it reads the updated certs to memory when done, otherwise its possible if the server never goes down to hold onto stale certs past their expiry

- `/etc/systemd/system/renew-certbot.service`
- `/etc/systemd/system/renew-certbot.timer`

Then run
```
systemctl enable renew-certbot.timer
systemctl start renew-certbot.timer
```
and it should show up in the output of `systemctl list-timers`

#### Todos

- fix restarting of nginx if certs are refreshed
