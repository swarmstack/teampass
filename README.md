# swarmstack/teampass

Docker compose file for [TeamPass](https://teampass.net). Requires Docker swarm.

## USAGE

Edit docker-compose.yaml and replace fqdn.example.com with your swarm http address. TeamPass database credentials for later configuration are also found within this same file.

```
docker stack deploy -c docker-compose.yml teampass
```

[swarmstack](https://github.com/swarmstack/swarmstack) users should use docker-compose-swarmstack.yml above instead.

---

You can then configure TeamPass at http://fqdn.example.com:6443/index.php

It is HIGHLY recommended to remove the exposed "ports:" section within the compose file, and instead secure the traffic behind HTTPS/TLS via a proxy. swarmstack users can add the following stanza to their existing Caddy proxy configuration:

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
