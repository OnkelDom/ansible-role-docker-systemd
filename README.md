# Ansible Role: docker-systemd

# !!! IN ALPHA !!!!

[![License](https://img.shields.io/badge/license-MIT%20License-brightgreen.svg)](https://opensource.org/licenses/MIT)
[![GitHub tag](https://img.shields.io/github/tag/OnkelDom/ansible-role-docker-systemd.svg)](https://github.com/OnkelDom/ansible-role-docker-systemd/tags)
[![GitHub issues](https://img.shields.io/github/issues/OnkelDom/ansible-role-docker-systemd)](https://github.com/OnkelDom/ansible-role-docker-systemd/issues)
[![GitHub license](https://img.shields.io/github/license/OnkelDom/ansible-role-docker-systemd)](https://github.com/OnkelDom/ansible-role-docker-systemd/blob/master/LICENSE)

## Description

Role to manage docker containers with systemd

All files stored in `docker_config_folder_local` creates autoamtic folders and files on remote.

## Requirements

- Ansible >= 3 (It might work on previous versions, but we cannot guarantee it)

## Role Variables

All variables which can be overridden are stored in [defaults/main.yml](defaults/main.yml) file as well as in table below.

| Name           | Default Value | Description                        |
| -------------- | ------------- | -----------------------------------|
| `proxy_env` | `{}` | Set proxy vars |
| `docker_container_default_timezone` | Europe/Berlin | Default timezone |
| `docker_owner` | nobody | Default doker user |
| `docker_volumes_basepath` | /opt/docker | Base path to store docker volumes |
| `docker_config_folder_remote` | "./opt/docker" | Path to Store configs on remote |
| `docker_config_folder_local` | "../files/configs" | Automatic managed config path |
| `docker_config` | {} | See [defaults/main.yml](defaults/main.yml) |
| `docker_volumes` | {} | See [defaults/main.yml](defaults/main.yml) |
| `docker_networks` | {} | See [defaults/main.yml](defaults/main.yml) |
| `docker_containers` | {} | See [defaults/main.yml](defaults/main.yml) |

## Example

```yaml
 - hosts: all
   roles:
     - onkeldom.ansible-role-docker-systemd
  vars:
    # Need by Traefik
    DOMAINNAME: local.lan
    CLOUDFLARE_EMAIL: 
    CLOUDFLARE_API_KEY: 
    # Path to config files
    docker_config_folder: "../files/configs"
    # Define Docker Volumes
    docker_volumes_basepath: /opt/docker
    docker_volumes:
      - traefik
    # Define Docker Networks
    docker_networks:
      - name: proxy
        subnet: 192.168.2.0/26
    # Define Docker Containers
    docker_containers:
      - name: traefik
        image: traefik:latest
        network: proxy
        user: nobody
        ports:
          - "80:80/tcp"
          - "443:443/tcp"
          - "8080:8082/tcp"
        volumes:
          - "traefik:/data"
          - "/var/run/docker.sock:/var/run/docker.sock:ro"
        labels:
          traefik.enable: "true"
          traefik.http.routers.http-catchall.entrypoints: "http"
          traefik.http.routers.http-catchall.rule: "HostRegexp(`{host:.+}`)"
          traefik.http.routers.http-catchall.middlewares: "redirect-to-https"
          traefik.http.middlewares.redirect-to-https.redirectscheme.scheme: "https"
          traefik.http.routers.traefik-rtr.entrypoints: "https"
          traefik.http.routers.traefik-rtr.rule: "HostHeader(`traefik.{{ DOMAINNAME }}`)"
          traefik.http.routers.traefik-rtr.service: "api@internal"
        envs:
          TZ: Europe/Berlin
          CF_API_EMAIL: "{{ CLOUDFLARE_EMAIL }}"
          CF_API_KEY: "{{ CLOUDFLARE_API_KEY }}"
        commands:
          - "--global.checkNewVersion=true"
          - "--global.sendAnonymousUsage=true"
          - "--entryPoints.http.address=:80"
          - "--entryPoints.https.address=:443"
          - "--entryPoints.traefik.address=:8080"
          - "--entryPoints.metrics.address=:8082"
          - "--serversTransport.insecureSkipVerify=true"
          - "--entrypoints.https.forwardedHeaders.trustedIPs=173.245.48.0/20,103.21.244.0/22,103.22.200.0/22,103.31.4.0/22,141.101.64.0/18,108.162.192.0/18,190.93.240.0/20,188.114.96.0/20,197.234.240.0/22,198.41.128.0/17,162.158.0.0/15,104.16.0.0/12,172.64.0.0/13,131.0.72.0/22"
          - "--api=true"
          - "--log=true"
          - "--log.level=WARN"
          - "--log.format=json"
          - "--accessLog=true"
          - "--accessLog.filePath=/data/access.log"
          - "--accessLog.bufferingSize=100"
          - "--accessLog.filters.statusCodes=200,300-302,400-410,500-502"
          - "--accesslog.fields.names.StartUTC=drop"
          - "--accesslog.format=json"
          - "--metrics.prometheus=true"
          - "--metrics.prometheus.addEntryPointsLabels=true"
          - "--metrics.prometheus.addServicesLabels=true"
          - "--metrics.prometheus.entryPoint=metrics"
          - "--providers.docker=true"
          - "--providers.docker.exposedByDefault=false"
          - "--entrypoints.https.http.tls.certresolver=dns-cloudflare"
          - "--entrypoints.https.http.tls.domains[0].main={{ DOMAINNAME }}"
          - "--entrypoints.https.http.tls.domains[0].sans=*.{{ DOMAINNAME }}"
          - "--providers.docker.network=proxy"
          - "--providers.docker.swarmMode=false"
          - "--providers.file.directory=/data/rules"
          - "--providers.file.watch=true"
          # TEST ACME - comment out when all running
          - "--certificatesResolvers.dns-cloudflare.acme.caServer=https://acme-staging-v02.api.letsencrypt.org/directory"
          - "--certificatesResolvers.dns-cloudflare.acme.email={{ CLOUDFLARE_EMAIL }}"
          - "--certificatesResolvers.dns-cloudflare.acme.storage=/data/acme.json"
          - "--certificatesResolvers.dns-cloudflare.acme.dnsChallenge.provider=cloudflare"
          - "--certificatesResolvers.dns-cloudflare.acme.dnsChallenge.resolvers=1.1.1.1:53,1.0.0.1:53"
          - "--certificatesResolvers.dns-cloudflare.acme.dnsChallenge.delayBeforeCheck=90"
```

## Contributing

See [contributor guideline](CONTRIBUTING.md).

## License

This project is licensed under MIT License. See [LICENSE](/LICENSE) for more details.
