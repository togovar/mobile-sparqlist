# mobile-sparqlist

SPARQList for TogoVar mobile.
See descriptions in the SPARQLet markdown files for details.

### Start and stop

```
$ docker-compose up -d
$ docker-compose ps
NAME                           IMAGE                                      COMMAND                                          SERVICE     CREATED         STATUS         PORTS
mobile-sparqlist-nginx-1       nginx:1.21                                 "/docker-entrypoint.sh nginx -g 'daemon off;'"   nginx       5 seconds ago   Up 4 seconds   0.0.0.0:12082->80/tcp
mobile-sparqlist-sparqlist-1   ghcr.io/dbcls/sparqlist:snapshot-fffa8c9   "docker-entrypoint.sh /bin/sh -c 'npm start'"    sparqlist   5 seconds ago   Up 4 seconds
dbcls3284:mobile-sparqlist mitsuhashi$
$ docker-compose down
```

### .env file
An .env file is required to be located at the same directory as docker-compose.yml.
```
# Project name
#COMPOSE_PROJECT_NAME=

# Nginx
NGINX_PORT=<<port number to be filled in>>

# SPARQList
SPARQLIST_ADMIN_PASSWORD=<<to be filled in>>

## Default SPARQL endpoint for development
SPARQLIST_TOGOVAR_SPARQL=https://grch38.togovar.org/sparql

TOGOVAR_MOBILE_SPARQLET=./sparqlist
```