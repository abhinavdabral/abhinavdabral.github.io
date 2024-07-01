---
layout: post
title: Traefik (v3) with wild-card certificates and without docker labels
---

As of today, most of the traefik configuration that I came across are using docker labels. And while that's perfectly fine, I find it hard to read and manage. Which is why, when I decided to move from [nginx-proxy-manager](https://github.com/NginxProxyManager/nginx-proxy-manager) over to [traefik](https://github.com/traefik/traefik), I wanted to keep to keep everything clean, readable and most-importantly, understandable.

_Last updated: 2024-07-01 - Updated Traefik v2 to v3 after verifying that it works with v3_

# Use case

My current self-hosted setup is fairly average. There's barely any traffic. There are maybe about 5 active users on an average and those are just family members and close friends. That being said, my use case is as follows:

- Multiple virtual machines running multiple services.
- Most applications are containerized using docker.
- Everything is hosted locally on physical machines, no cloud servers involved.
- Exposed port `:443` (HTTPS) for all external traffic.
- Exposed port `:80` (HTTP) only for ACME HTTP challenge for certificate renewal.
- No high-availability setup. No K3S or anything.
- No SLA violations to worry about.
  - Still, I do my maintenance during off-hours when there are no active users.

# Configuring Traefik

So here's a list of things we'll be doing here

- Setting up the folder structure
- Creating docker-compose configuration
- Creating traefik configuration
  - Static configuration
    - Providers (file, in our case)
    - Entry points
    - Certificate resolvers
  - Dynamic configuration
    - Routes
    - Services
    - Middlewares
- Creating blank `acme.json` files for certificates
- Starting the proxy

## 1. Folder structure

Since we're not going to be using any docker labels to do any of the configuration, let's start by creating a folder-structure where Traefik will get the configuration from.

This is not a "mandatory" folder structure. It's just what I like to use. You can modify things as you see fit.

```
 - traefik/
 |-- docker-compose.yaml
 |-- config/
 | |-- traefik.yaml
 | |-- acme/
 | | |-- acme-<your-domain>.json (Perm: 600)
 | | |-- acme-<other-domain>.json (Perm: 600)
 | |-- dynamic/
 | | |-- _middlewares.yaml
 | | |-- _services.yaml
 | | |-- <your-domain>.yaml
 | | |-- <other-domain>.yaml
```

## 2. Docker compose

#### `docker-compose.yaml`

```yaml
version: "3"

services:
  traefik:
    container_name: traefik
    image: traefik:v3.0

    # Enables dashboard access through HTTP
    # which is fine since you don't necessarily need to expose
    # this anyway.
    command: --api.insecure=true

    # Overriding DNS is not necessary but isn't harmful either.
    # My reason is because I solve my domain name to my
    # local IP instead of the actual external IP. This saves
    # me unnecessary round-trips.

    dns:
      - 1.1.1.1
      - 1.0.0.1

    # Required if you have something like Cloudflare
    # Access keys for DNS challenge. Although I'd encourage to
    # keep them in .env file instead.
    environment:
      - "CLOUDFLARE_DNS_API_TOKEN=your-dns-api-token"
      - "CLOUDFLARE_ZONE_API_TOKEN=your-zone-api-token"

    ports:
      # Needed for HTTP-01 challenge only
      - "80:80"

      # Needed for HTTPS
      - "443:443"

      # The Web UI (enabled by --api.insecure=true)
      - "8080:8080"

    volumes:
      - ./traefik/config:/etc/traefik

    network_mode: bridge
```

## 3. Traefik configuration

Before we proceed ahead, I'd like to give a very short TL;DR of certain things in how traefik handles configuration.

- `abc@file` basically means that you intend to load something called `abc` which exist is present from the `file` provider.
- Any configurations that's loaded from files is all merged, so you don't need to worry about "which" file something is loaded from, as long as the file is being loaded and watched from the main traefik configuration.
- Mixing and matching is allowed but be aware of what's happening at any point of time.
  - Example, you can specify middlewares at `routes`, OR even at the main `entryPoints` configuration. Both works, but both have different use cases.
- It's generally good idea to keep `static` configuraiton separate from `dynamic` one. Which is what I'm doing here as well. All static configuration lives in `traefik.yaml`, whereas `dynamic` configuration is all contained within separate files that are in the `dynamic` directory.

### 3.1 Static configuration

#### `config/trarfik.yaml`

Make sure to replace your own parameters within `certificateResolvers` section. And in-general I'd encourage you to read the comments as well to have general idea of what's happening in the configuration.

> I have multiple domains and some use DNS challenge and some HTTP challenge, so I've included both configurations here. You can chose to keep whichever ones are suitable for your use-case and remove the other one.

Also, for details regarding any of these options, feel free to check [Traefik documentation](https://doc.traefik.io/traefik/).

```yaml
global:
  checkNewVersion: true
  sendAnonymousUsage: false

providers:
  file:
    # This allows you to import all *.yaml files from this
    # directory and merge their configurations in real-time.
    # This is same where we are keeping `_services.yaml`
    # `_middlewares.yaml` and others.
    directory: "/etc/traefik/dynamic"

serversTransport:
  insecureSkipVerify: true

api:
  dashboard: true
  insecure: true
#  debug: true

# log:
#   level: debug

entryPoints:
  http:
    address: ":80"
    http:
      redirections:
        entryPoint:
          to: "https"
          scheme: "https"
          permanent: true

  https:
    address: ":443"
    http:
      # tls: ....
      # You can also specify the TLS section here instead
      # of the routers, if you only have a single domain name.
      # Then, within routers you can just put `tls: {}` everywhere
      # and it will work
      middlewares:
        # IMPORTANT:
        # This tells traefik to apply this set of middleware
        # to `ALL` the traffic going through this entry point.

        - "secured@file"

certificatesResolvers:
  cloudflare:
    acme:
      email: "email-registered-with-cloudflare@email.com"
      storage: /etc/traefik/acme/acme-<your-domain>.json
      dnsChallenge:
        provider: cloudflare
        resolvers:
          - "1.1.1.1:53"
          - "1.0.0.1:53"

  myOtherDomainResolver:
    acme:
      email: "email-for-certificate-alerts@email.com"
      storage: /etc/traefik/acme/acme-<other-domain>.json
      httpChallenge:
        entryPoint: "http"
```

### 3.2 Dynamic configurations

#### 3.2.1 `config/dynamic/_middlewares.yaml`

There's a pretty generic set of configuration that should suffice the most common setups out there.

```yaml
http:
  middlewares:
    secured:
      headers:
        # For enabling HSTS
        forceSTSHeader: true
        stsIncludeSubdomains: true
        stsPreload: true
        stsSeconds: 31536000

        customRequestHeaders:
          # enables upgrade to websocket
          X-Forwarded-Proto: https

    nextcloud:
      headers:
        browserXssFilter: true
        contentTypeNosniff: true
        frameDeny: true
        sslRedirect: true
        #HSTS Configuration
        customFrameOptionsValue: "SAMEORIGIN"

    dav-redirect:
      replacePathRegex:
        regex: ^/.well-known/ca(l|rd)dav
        replacement: /remote.php/dav/

    onlyoffice:
      headers:
        accessControlAllowOriginList:
          - https://<nextcloud-domain-name>.com

    # This example is included for services that are hosted in
    # sub-directories and we need to strip the sub-directories
    # at the proxy level, to make them work as expected
    gotify:
      stripPrefix:
        prefixes:
          - "/gotify"
    auth1:
      basicAuth:
        users:
          # You can use any tool that can generate HTPASSWD
          # like https://wtools.io/generate-htpasswd-online
          # The below one is just admin:admin
          - "admin:$apr1$ptzkgbw2$mkohRpevjIzSTMBuVbt3I."
```

#### 3.2.2 `config/dynamic/_services.yaml`

You need to replace these with whatever services you have running on your server.

```yaml
http:
  services:
    uptime-kuma:
      loadBalancer:
        servers:
          - url: "http://10.0.0.10:8082"
    nextcloud: # can be anything
      loadBalancer:
        servers:
          # replace with wherever your actual
          # service is running
          - url: "http://10.0.0.10:8080"

    gotify: # can be anything
      loadBalancer:
        servers:
          # replace with wherever your actual
          # service is running
          - url: "http://10.0.0.10:8081"
```

#### 3.2.3 `config/dynamic/your-domain.yaml`

These are some different configuration examples of services which make use of hosting at:

- Domain level (`your-domain.com`)
- Sub-domain level (`dashboard.your-domain.com`)
- Sub-sub-domain level (`your-domain.com`)
- Sub-directory level

> **NOTE:** TO expose the dashboard to a route, we can simply use the service `api@internal`.

```yaml
http:
  routers:
    status:
      rule: "Host(`your-domain.com`)"
      entryPoints:
        - https

      # service from _services.yaml
      service: "uptime-kuma@file"
      tls:
        # Specify whichever certificate resolver is
        # applicable for this host. This should match
        # one of the cert resolvers mentioned in the
        # traefik.yaml static configuration
        certResolver: cloudflare
        domains:
          - main: "your-domain.com"
            # sans block is optional but can be used
            # to define subdomains or wild-card subdomains
            # where the certificate should also be considered
            # valid.
            sans:
              - "*.your-domain.com"
              - "*.local.your-domain.com"

    nextcloud:
      rule: "Host(`nextcloud.your-domain.com`)"
      entryPoints:
        - https

        # service from _services.yaml
      service: "nextcloud@file"
      middlewares:
        # middleware from _middlewares.yaml
        - "nextcloud@file"
        - "dav-redirect@file"

      # Notice that we don't need to re-specify the
      # certResolver again. As long as at least one
      # router for that host defines those settings,
      # other don't need to define it again.
      # But we put this block on place to infer that
      # TLS needs to be used for this route
      tls: {}

    # doesn't need to match the service's name but generally
    # it's a good idea to keep them same/similar to reduce
    # confusion
    gotify:
      rule: "Host(`your-domain.com`) && PathPrefix(`/gotify/`)"
      entryPoints:
        - https
      service: "gotify@file" # service from _services.yaml
      middlewares:
        - "gotify@file" # middleware from _middlewares.yaml
      tls: {}

    dashboard:
      rule: "Host(`dashboard.local.your-domain.com`)"
      entryPoints:
        - https
      service: "api@internal"
      middlewares:
        - auth1 # basic auth middleware we created above
      tls: {}
```

### 3.3 Creating ACME files

Assuming you're on \*nix system. Just go within the `config/acme` folder and create acme files for your domains. You can chose to keep 1 file per-domain or keep 1 file for everything. It's a personal choice but I like to keep things separate.

The most important thing is to make sure that the files have the permission of `600` (owner: read+write, group: none, others: none)

```sh
$ touch acme-your-domain.json
$ chmod 600 acme-your-domain.json
```

We don't need to add any contents in those files as that will be done by LetsEncrypt when we initialize the acme configuration and load the certificates in it.

### 3.4 Go live!

Start the service by running the docker-compose configuration.

```sh
$ docker-compose up
```

Keep and eye on the logs and see if anything is wrong. But if everything works as expected, you can try to visit your domain and see if it works. (Assuming your ports are already forwarded and everything else is working as intended, outside traefik)

# Conclusion

The setup that I presented above, works great for me. It helps me clearly visualize the different parts of the configuration and manage them separately.

I hope that someone finds this post to be helpful in setting up their own traefik reverse proxy. If there are any questions or suggestions to improve this post, please feel free to open an issue on [GitHub repository](https://github.com/abhinavdabral/abhinavdabral.github.io).
