# Linode - Mastodon - Kickstart

This repo containss work instructions for setting up Mastodon via Docker with Caddy for 
Letsencrypt and http proxying.

It's intended as a quick start guide, and not a thorough guide. It assumes some 
comfortability with the Linux commandline, Linode and Docker.

No support intended, use at your own risk etc.

Full credit  to Robert Riemann for their post here, which is almost the same thing give or take some of the commands.

https://blog.riemann.cc/digitalisation/2022/02/09/mastodon-setup-with-docker-and-caddy/

## Start

* Setting up Mastodon on a Linode instance:

Brand new Linode VM, follow these instructions to harden Ubuntu and set up Fail2Ban etc: [How to Set Up and Secure a Linode Compute Instance 
|Linode](https://www.linode.com/docs/guides/set-up-and-secure/#connect-to-the-instance)

Make sure the DNS of your domain points to your new Linode VM.

For a single user instance, the 2GB shared Linode VM seems to hold up well, with Elastic-search turned off. This is how this docker file is 
configured. If you need this turned on, you'll need to edit the docker-compose.yml file to enable it, and to follow the steps in setting up 
Mastodon in it's script to enable (or editing your `mastodon.env.production` file if you're switching it on after setting up.

* From root:

Install docker and docker compose: `apt-get install docker-compose`

```
sudo adduser --disabled-login mastodon
sudo adduser mastodon docker
sudo adduser mastodon sudo # optional, remove later
sudo su mastodon # switch to that user
```

## In the mastodon user's home directory

* Create env file
```
LETS_ENCRYPT_EMAIL=you@youremail.com
MASTODON_DOMAIN=social.example.com # Change to your domain
FRONTEND_SUBNET="172.22.0.0/16"
# check the latest version here: https://hub.docker.com/r/tootsuite/mastodon/ta>
MASTODON_VERSION=v4.0.2
COMPOSE_PROJECT_NAME=mastodon
```

* Add docker file from this repo
* Create caddy directory `mkdir -p ./caddy/etc-caddy`
* Place Caddyfile in `./caddy/etc-caddy`

## Run commands

```
# mastodon
touch mastodon.env.production
sudo chown 991:991 mastodon.env.production
mkdir -p mastodon/public
sudo chown -R 991:991 mastodon/public
mkdir -p mastodon/elasticsearch
sudo chmod g+rwx mastodon/elasticsearch
sudo chgrp 0 mastodon/elasticsearch

# first time: setup mastodon
# ⚠ ️ make sure to use the container name for the DB and redis, mastodon-db, and mastodon-redis when following through Mastodon's setup script.
sudo docker-compose run --rm -v $(pwd)/mastodon.env.production:/opt/mastodon/.env.production -e RUBYOPT=-W0 mastodon-web bundle exec rake 
mastodon:setup

# subsequent times: skip generation of config and only setup database
docker-compose run --rm -v $(pwd)/mastodon.env.production:/opt/mastodon/.env.production mastodon-web bundle exec rake db:setup
```

Remove mastodon user from sudo:
```
sudo deluser mastodon sudo
```	

## You should now be ready to rock, and or roll...

from your mastodon user:

`docker-compose up -d` 

will start the service

If you need to debug anything, then `docker compose logs -f` in your mastodon user's home directory should show the activity from your mastodon 
instance.

## Have fun!

I'm not intending on supporting this (lack of time!), but you can find me over at @davidgarywood@social.davidgarywood.com on the Fediverse!

Dave ✨
