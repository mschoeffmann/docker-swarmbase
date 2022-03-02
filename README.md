# Docker Swarmbase

A minimalistic Docker Swarm base setup featuring ready-to-use [Portainer](https://www.portainer.io) container management, [Prometheus](https://prometheus.io)/[Grafana](https://grafana.com) monitoring and [Traefik](https://traefik.io) with [Let's Encrypt](https://letsencrypt.org) as ingress proxy on [Ubuntu Linux](https://ubuntu.com) systems.

[[_TOC_]]

## Features
- Automated installation and initialization of Docker Swarm
- Basic firewall rules to allow only ports 22, 80, 443 and 8443
- Portainer instance for stack- & container management
- Grafana and Prometheus monitoring inkl. ready-to-use dashboards
- Traefik ingress proxy with included SSL/TLS support and Let's Encrypt certificate management

## Requirements
- Ubuntu Linux 20.04 LTS host system
- Public IPv4 address

## Installation

Replace `swarmbase.example.com` with your own hostname for the whole installation process.

### Preparing the installation
- Install your Ubuntu base system
- Create DNS entries `swarmbase.example.com` and `*.swarmbase.example.com` (wildcard)  
  pointing to your server's IP address
- Start up the server and log in as root

### Updating the server
```bash
apt update
apt upgrade
reboot
```

### Installing Swarmbase
This automatically installs all required packages and configures the docker swarm stack.
```
git clone https://gitlab.com/mschoeffmann/docker-swarmbase.git
cd docker swarmbase
./swarmbase install 
```

## Configuration
The local configuration file `.config` is only used for the first installation.
- `SWARM_MANAGER`: The internal IP address in case you have an internal network between swarm hosts. Can be empty.
- `LETSENCRYPT_SERVER`: The Let's Encrypt server. Use `acme-v02` for production or `acme-staging-v02` for development.
- `LETSENCRYPT_MAIL`: Your e-mail address for Let's Encrypt.
- `ADMIN_HOSTNAME`: The hostname of your Swarmbase server. Make sure you created the DNS records (including wildcard).
- `ADMIN_PASSWORD`: The admin password. This is set on initial installation.

## Administration

### Web administration
After a successful installation, you have the following web management interfaces available:
- Portainer (management): `https://portainer.swarmbase.example.com:8443`
- Grafana (monitoring): `https://grafana.swarmbase.example.com:8443`

*Replace `swarmbase.example.com` with your server's hostname.*  
Username: `admin`  
Password: *as specified on installation*

### Swarmbase CLI
The `swarmbase` script offers a few cli commands.

```bash
# update the admin stack
./swarmbase update_admin

# update the proxy stack
./swarmbase update_proxy
```

```bash
# (re-)deploy the admin stack
./swarmbase deploy_admin

# (re-)deploy the proxy stack
./swarmbase deploy_proxy
```

```bash
# install docker packages
./swarmbase install_docker

# init the docker swarm
./swarmbase init_swarm

# (re-)create required networks
./swarmbase create_network

# configure (and install) iptables firewall
./swarmbase configure_firewall
```

## Example Stack
You can use add a test service to your Docker Swarmbase server:
1. Open Portainer and log in
2. Go to: Stacks > Add stack
3. Choose a name for the stack: `test-service`
4. Use the following example as content for the web editor, but make sure you change `swarmbase.example.com` to your server's hostname.
```yaml
version: "3.8"

services:
  test-service:
    image: containous/whoami:latest
    volumes:
      - /etc/localtime:/etc/localtime:ro
    networks:
      - proxy
    deploy:
      mode: replicated
      placement:
        constraints: 
          - node.platform.os == linux
      replicas: 1
      labels:
        - "traefik.enable=true"
        - "traefik.http.routers.test-service.rule=Host(`test-service.swarmbase.example.com`)"
        - "traefik.http.routers.test-service.entrypoints=https"
        - "traefik.http.routers.test-service.tls=true"
        - "traefik.http.routers.test-service.tls.certresolver=leresolver"
        - "traefik.http.services.test-service.loadbalancer.server.port=80"
        - "traefik.docker.network=proxy"
          
networks:
  proxy:
    external: true
```

For production deployment, every service has to have a unique *router* and *services* id, so make sure you change `test-service` to something unique for each service on your swarm.

More information can be found at [Compose file version 3 reference](https://docs.docker.com/compose/compose-file/compose-file-v3/) and [Traefik & Docker](https://doc.traefik.io/traefik/providers/docker/).