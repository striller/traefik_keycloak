# Traefik + Docker Compose + Keycloak example

![Traefik Docker Keycloak Logo](/images/logo.jpg)

Example for usage [Traefik](https://traefik.io/) + [Keycloak](https://www.keycloak.org) with postgres as a database.

Tested on Ubuntu 20.04 and Docker 19.03.13, build 4484c46d9d and Docker Compose 1.25.0

## Setup Traefik

### Step 1
We’ll use the htpasswd utility to create this encrypted password. First, install the utility, which is included in the apache2-utils package:

```bash
sudo apt-get install apache2-utils
```

Then generate the password with htpasswd

```bash
htpasswd -nb admin secure_password
```

Please replace admin and secure_password.

```bash
htpasswd -nb secureuser securepassword
secureuser:$apr1$wEvgCB5t$E/TLN7IIBA1dv0dQeffyv.
```

Copy the entire output line so we need it in Step 2.

### Step 2
Open traefik.toml and replace $YOUR_EMAIL your emailadress.
Edit traefik_dynamic.toml and replace $htpasswdoutput.

```toml
[http.middlewares.simpleAuth.basicAuth]
  users = [
    "secureuser:$apr1$wEvgCB5t$E/TLN7IIBA1dv0dQeffyv."
  ]
```
and replace $DOMAIN_FOR_TRAEFIK with your subdomain like monitor.example.com.

Please change the [ownership](https://chmodcommand.com/chmod-600/) from the acme.json. 

```bash
chmod 600 acme.json
```

### Running the Traefik Container
First we create a Docker network for the proxy to share with containers. 

```bash
docker network create web
```
Finally, create the Traefik container with this command:

```bash
docker run -d   -v /var/run/docker.sock:/var/run/docker.sock   -v $PWD/traefik.toml:/traefik.toml   -v $PWD/traefik_dynamic.toml:/traefik_dynamic.toml   -v $PWD/acme.json:/acme.json   -p 80:80   -p 443:443   --network web   --name traefik   traefik:v2.2
```

Access the monitoring dashboard by pointing your browser to https://monitor.example.com You will be prompted for your username and password, which are admin and the password you configured in Step 1.


## Setup Keycloak
Inside the keycloak folder. Please copy env.dist to .env inside the keycloak folder and set your enviroments.

Edit the Traefik section and replace $domain_for_keycloak with your domain in the docker-compose file.

```yaml
- traefik.http.routers.keycloak.rule=Host(`keycloak.example.com`)
```


## Usage

```bash
docker-compose up
```

Open your Website keycloak.example.com and login with your creditails from the .env file.

## Backup Postgres
Backup PostgresSQL to the local filesystem with periodic rotating backups, based on [postgres-backup-local](https://github.com/prodrigestivill/docker-postgres-backup-local)
