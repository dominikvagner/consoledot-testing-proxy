# Consoledot Testing Proxy
[![Static Badge](https://img.shields.io/badge/quay.io-container-badge?labelColor=red&color=black)](https://quay.io/repository/dvagner/consoledot-testing-proxy)

Configurable container proxy for UI/E2E testing, implemented using the
[caddy](https://caddyserver.com/) proxy extended with a custom header transforming
extension.

> REMINDER: This is very WIP. :)

## Usage

This container proxy will expose a [env | stage].foo.redhat.com:[port | 1337]
endpoint that will proxy request to routes defined by you and the rest to the
chosen consoledot environment. \
The container needs to bind directly to the Podman/Docker host's network,
with no network isolation, which is why you might need to run it as
root/superuser.

Usage (testing against stage):

```sh
sudo podman run -d
  -e HTTPS_PROXY=$RH_STAGE_PROXY_URL
  -v "$(pwd)/config:/config:ro,Z"
  --network=host
  consoledot-testing-proxy quay.io/dvagner/consoledot-testing-proxy
```

## Setup

All you really need is Podman or Docker and the app you want to test :)

### Hosts setup

In order to access the https://[env].foo.redhat.com in your browser, you have
to add entries to your /etc/hosts file. This is a one-time setup that has to
be done only once (unless you modify hosts) on each machine.

For each env you will need to add this to your `/etc/hosts` file, default env is
'stage':

```sh
127.0.0.1 [env].foo.redhat.com
::1 [env].foo.redhat.com
```

## Configuration

This proxy has configurable routes and the consoledot (HCC) environment

### Routes

The proxy can be configured for your apps/needs by providing a JSON configuration
file that defines the to be proxied routes and a flag for endpoints that require
the RH identity header (APIs for example might, but not the static files).

This proxy is meant to be used along a locally running console app in the static
mode, i.e.: in your app run `npm|yarn fec static`. And if you want also your
backend.

By default the container will expect the routes JSON config in `/config/routes.json`,
but if needed this can be changed by setting the `ROUTES_JSON_PATH` environment
variable.

Example:

```json
{
  // STATIC FILES FE
  "/apps/NAME-OF-YOUR-APP*": { "url": "http://127.0.0.1:8003" },
  // YOUR BACKEND API
  "/api/NAME-OF-YOUR-APP/*": {
    "url": "http://127.0.0.1:8000",
    "rh-identity-headers": true
  }
}
```

### Environment

The environment can be configured by setting the `HCC_ENV_URL` to something
different than the default of `console.stage.redhat.com` and all the uncatched
requests by your routes/matchers will be directed there.
Other than that you can also set `HCC_ENV` and `HCC_PORT` variables that just
change the exposed URL you are gonna be using.

For testing against stage you will also need to set the `HTTPS_PROXY` environment
variable to the RH stage proxy URL.
