---
id: konfigurisanje
title: "Fajl za konfigurisanje"
---
Ovaj fajl je osnova verdaccio-a. U okviru njega, možete vršiti izmene zadatih podešavanja, možete aktivirati plugin-e i spoljašnje resurse (features).

Fajl "default configuration file" se kreira prilikom prvog pokretanja `verdaccio-a`.

## Podrazumevane postavke (Default Configuration)

Podrazumevane postavke podržavaju **scoped** pakete za sve korisnike, ali samo **autorizovanim korisnicima omogućavaju da publikuju**.

```yaml
storage: ./storage
auth:
  htpasswd:
    file: ./htpasswd
uplinks:
  npmjs:
    url: https://registry.npmjs.org/
packages:
  '@*/*':
    access: $all
    publish: $authenticated
    proxy: npmjs
  '**':
    proxy: npmjs
logs:
  - {type: stdout, format: pretty, level: http}
```

## Sekcije

Sekcija u nastavku daje objašnjenja za svako svojstvo i opciju.

### Memorija za skladištenje

Je lokacija na kojoj se vrši skladištenje podataka. **Verdaccio je inicijalno podešen kao local file system**.

```yaml
storage: ./storage
```

### Plugins

Je lokacija plugin directorijuma. Ovo je korisno za deployment baziran na Docker/Kubernetes.

```yaml
plugins: ./plugins
```

### Autentifikacija

Ovde se vrši podešavanje (set up). Podrazumevana auth je bazirana na `htpasswd` i već je ugrađena. Možete izvršiti modifikacije načina rada (behaviour) putem [plugin-a](plugins.md). Za više informacija o ovoj sekciji pročitajte [auth stranu](auth.md).

```yaml
auth:
  htpasswd:
    file: ./htpasswd
    max_users: 1000
```

### Security

<small>Since: <code>verdaccio@4.0.0</code> due <a href="https://github.com/verdaccio/verdaccio/pull/168">#168</a></small>

The security block allows you to customise the token signature. To enable [JWT (json web token)](https://jwt.io/) new signture you need to add the block `jwt` to `api` section, `web` uses by default `jwt`.

The configuration is separated in two sections, `api` and `web`. To use JWT on `api`, it has to be defined, otherwise will use the legacy token signature (`aes192`). For JWT you might customize the [signature](https://github.com/auth0/node-jsonwebtoken#jwtsignpayload-secretorprivatekey-options-callback) and the token [verification](https://github.com/auth0/node-jsonwebtoken#jwtverifytoken-secretorpublickey-options-callback) with your own properties.

    security:
      api:
        legacy: true
        jwt:
          sign:
            expiresIn: 29d
          verify:
            someProp: [value]
       web:
         sign:
           expiresIn: 7d # 7 days by default
         verify:
            someProp: [value]
    

> We highly recommend move to JWT since legacy signature (`aes192`) is deprecated and will disappear in future versions.

### Web UI

This property allow you to modify the look and feel of the web UI. For more information about this section read the [web ui page](web.md).

```yaml
web:
  enable: true
  title: Verdaccio
  logo: logo.png
  scope:
```

### Uplinks

Uplinks is the ability of the system to fetch packages from remote registries when those packages are not available locally. For more information about this section read the [uplinks page](uplinks.md).

```yaml
uplinks:
  npmjs:
    url: https://registry.npmjs.org/
```

### Packages

Packages allow the user to control how the packages are gonna be accessed. For more information about this section read the [packages page](packages.md).

```yaml
packages:
  '@*/*':
    access: $all
    publish: $authenticated
    proxy: npmjs
```

## Napredna podešavanja

### Offline Publish

By default `verdaccio` does not allow to publish when the client is offline, that behavior can be overridden by setting this to *true*.

```yaml
publish:
  allow_offline: false
```

<small>Since: <code>verdaccio@2.3.6</code> due <a href="https://github.com/verdaccio/verdaccio/pull/223">#223</a></small>

### URL Prefix

```yaml
url_prefix: https://dev.company.local/verdaccio/
```

Since: `verdaccio@2.3.6` due [#197](https://github.com/verdaccio/verdaccio/pull/197)

### Max Body Size

By default the maximum body size for a JSON document is `10mb`, if you run in errors as `"request entity too large"` you may increase this value.

```yaml
max_body_size: 10mb
```

### Listen Port

`verdaccio` runs by default in the port `4873`. Changing the port can be done via [cli](cli.md) or in the configuration file, the following options are valid.

```yaml
listen:
# - localhost:4873            # default value
# - http://localhost:4873     # same thing
# - 0.0.0.0:4873              # listen on all addresses (INADDR_ANY)
# - https://example.org:4873  # if you want to use https
# - "[::1]:4873"                # ipv6
# - unix:/tmp/verdaccio.sock    # unix socket
```

### HTTPS

To enable `https` in `verdaccio` it's enough to set the `listen` flag with the protocol *https://*. For more information about this section read the [ssl page](ssl.md).

```yaml
https:
    key: ./path/verdaccio-key.pem
    cert: ./path/verdaccio-cert.pem
    ca: ./path/verdaccio-csr.pem
```

### Proxy

Proxies are special-purpose HTTP servers designed to transfer data from remote servers to local clients.

#### http_proxy i https_proxy

If you have a proxy in your network you can set a `X-Forwarded-For` header using the following properties.

```yaml
http_proxy: http://something.local/
https_proxy: https://something.local/
```

#### no_proxy

This variable should contain a comma-separated list of domain extensions proxy should not be used for.

```yaml
no_proxy: localhost,127.0.0.1
```

### Obaveštenja

Enabling notifications to third-party tools is fairly easy via web hooks. For more information about this section read the [notifications page](notifications.md).

```yaml
notify:
  method: POST
  headers: [{'Content-Type': 'application/json'}]
  endpoint: https://usagge.hipchat.com/v2/room/3729485/notification?auth_token=mySecretToken
  content: '{"color":"green","message":"New package published: * {{ name }}*","notify":true,"message_format":"text"}'
```

> For more detailed configuration settings, please [check the source code](https://github.com/verdaccio/verdaccio/tree/master/conf).

### Audit

<small>Since: <code>verdaccio@3.0.0</code></small>

`npm audit` is a new command released with [npm 6.x](https://github.com/npm/npm/releases/tag/v6.1.0). Verdaccio includes a built-in middleware plugin to handle this command.

> If you have a new installation it comes by default, otherwise you need to add the following props to your config file

```yaml
middlewares:
  audit:
    enabled: true
```