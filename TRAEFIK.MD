# Setting up traefik with local domain for development

## Setup dnsmasq

* Install dnsmasq
* Add next row int /etc/dnsmasq.conf

```
address=/<domain-name>/127.0.0.1/
```

* Setting up dhcpclient for auto resolve host
* Add to `/etc/dhcp/dhclient.conf` next row:

```
prepend domain-name-servers 127.0.0.1;
```
* Add to `/etc/dhcpcd.cong` next row:
```
option domain_name_servers 127.0.0.1, 8.8.8.8, 8.8.4.4
```

* Setting up dnsmasq with Network Manager

Add to section `[main]`:

```
dns=dnsmasq
```

* Restart network manager

## Setting up traefik

Example traefik.toml config:

```
debug = true
logLevel = "DEBUG"
defaultEntryPoints = ["http", "https"]
[entryPoints]
    [entryPoints.http]
    address = ":80"
    [entryPoints.https]
    address=":443"
#    [entryPoints.http.redirect]
#      entryPoint = "https"
    [entryPoints.https.tls]
#        [entryPoint.https.tls.certificates]
#            certFile="/etc/traefik/keys/sonrac.info/cert1.pem"
#            keyFile="/etc/traefik/keys/sonrac.info/cert1.pem"

    [entryPoints.traefik]
        address=":8082"
        [entryPoints.traefik.auth]
            [entryPoints.traefik.auth.basic]
                users = [
                    "traefik:$2b$10$P2plq9gAWGK1i0B/YMREzuGzs62oyXcUCTAnqvfxWDRTFXnd8/42K"
                ]
    [entryPoints.ping]
        address=":8084"
        [entryPoints.ping.auth]
            [entryPoints.ping.auth.basic]
                users = [
                    "traefik:$2b$10$P2plq9gAWGK1i0B/YMREzuGzs62oyXcUCTAnqvfxWDRTFXnd8/42K"
                ]
filePath = "log/traefik.txt"

[api]
entrypoint="traefik"
dashboard = true
debug=true
# Enable more detailed statistics.
[api.statistics]
recentErrors = 100

[ping]
entrypoint="traefik"

[docker]
domain = "stage.ministra"
exposedbydefault = false
watch=true
swarmmode = true

#[acme]
#email = "doniysa@gmail.com"
#onHostRule = true
#caServer = "https://acme-staging-v02.api.letsencrypt.org/directory"
#storage = "/etc/traefik/acme/account" # the key where to store your certificates in the KV store
#storageKey = "/etc/traefik/acme/acme.json"
#entryPoint = "https"
#acmeLogging=true
#    [acme.httpChallenge]
#    entryPoint = "http"
#    [acme.dnsChallenge]
#        provider="godaddy"
#        delayBeforeCheck=0
```

## Setting up container deploy to swarm with traefik proxy

Example docker-compose.yml for portainer

```
version: '3.6'

services:
  agent:
    image: docker.gitlab.infomir.com.ua:5000/backend/infrastructure/agent
    build:
      context: ./portainer/
      dockerfile: Dockerfile.agent
    environment:
      # REQUIRED: Should be equal to the service name prefixed by "tasks." when
      # deployed inside an overlay network
      AGENT_CLUSTER_ADDR: tasks.agent
      # AGENT_PORT: 9001
      # LOG_LEVEL: debug
    volumes:
      - /var/run/docker.sock:/var/run/docker.sock
    networks:
      - traefik-net
      - agent_network
      - postgresql
      - mariadb
      - mysql
      - ministra
      - smac
      - coupons
      - notifications
    deploy:
      mode: global

  portainer:
    image: portainer/portainer
    command: -H tcp://tasks.agent:9001 --tlsskipverify
    volumes:
      - portainer_data:/data
    networks:
      - traefik-net
      - agent_network
    deploy:
      mode: replicated
      replicas: 1
      placement:
        constraints: [node.role == manager]
      labels:
        - "traefik.enable=true" # Enable traefik for this container
        - "traefik.portainer.enable=true" # Enable backend in traefik (alias)
        - "traefik.backend=traefik" # Backend name
        - "traefik.portainer.frontend.rule=Host:docker-admin.stage.ministra" # Front end rule host
        - "traefik.docker.network=traefik-net" # Traefik network. Must be attachable to traefik container network
        - "traefik.frontend.entrypoints=https,http" # Entrypoints fot container
        - "traefik.http.redirect=https" # Force redirect option
        - "traefik.port=9000" # Listen port in container
        - "traefik.portainer.port=9000" # Listen port in container (alias for backend)

```

## Init swarm, start traefik and deploy container

```
# Init swarm mode
docker swarm init
# Create traefik overlay network
docker network create -d overlay --attachable traefik-net
# Deploy traefik
docker stack deploy -c traefik-stack-local.yml traefik
# Deploy portainer container
docker stack deploy -c portainer-local-stack.yml portainer
```

After it you can open (traefik)[http://traefik.devinf] and (portainer)[http://docker-admin.devinf]

## See [example configs](./files)
