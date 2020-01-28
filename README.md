# swarmstack/teampass

Docker compose file for [TeamPass](https://teampass.net). Requires Docker swarm.

## USAGE

Edit docker-compose.yaml and replace fqdn.example.com with your swarm http address. TeamPass database credentials for later configuration are also found within this same file.

MariaDB will initially deploy a container with --skip-networking enabled, which prevents TeamPass from connecting. You should bounce the stack after MariaDB first initializes before attempting to configure TeamPass. The script below will wait for the MariaDB container to come online and then wait 2 minutes for it to initialize, but you can also just watch the container logs.

```
docker stack deploy -c docker-compose.yml teampass
while true; do docker service ls | grep mariadb | grep 1/1; if [ $? -eq 0 ]; then break; fi; done; sleep 120
docker stack rm teampass
docker stack deploy -c docker-compose.yml teampass
```

[swarmstack](https://github.com/swarmstack/swarmstack) users should use docker-compose-swarmstack.yml above instead.

---

You can then configure TeamPass at http://fqdn.example.com:6443/index.php

For Teampass configuration, the database host is 'db', and database credentials are found within the Docker compose file.

Per [https://github.com/nilsteampassnet/TeamPass](https://github.com/nilsteampassnet/TeamPass): Use /var/www/html/sk as your "Absolute path to saltkey" during installation.

---

It is HIGHLY recommended to remove the exposed "ports:" section within the compose file, and instead secure the traffic behind HTTPS/TLS via a proxy.

swarmstack users can add the following stanza to their existing Caddy proxy configuration, and remove the "ports:" section from the compose file:

```
{$CADDY_URL}:6443 {
  errors stderr
  prometheus 0.0.0.0:9180 {
    hostname 6443
  }
  tls {$CADDY_CERT} {$CADDY_KEY}

  proxy / teampass:80 {
    transparent
  }
}
```
