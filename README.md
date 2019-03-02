# caddy-etcd

This is a clustering plugin for Caddy that will store Caddy-managed certificates and any other assets in etcd rather than the filesystem.

By using this plugin, you can run multiple instances of caddy and have them share configuration information and TLS certificates.  You must have
already set up your own etcd cluster for this plugin to work.  

## Configuration

Caddy clustering plugins are enabled and configured through environment variables.  The table below lists the available options, but **to enable this plugin
you must first set `CADDY_CLUSTERING="etcd"`.**


| Environment | Function | Default |
| --- | --- | ---|
| CADDY_CLUSTERING_ETCD_SERVERS | A comma or semicolon separated list of etcd servers for caddy to connect to. The servers must be specified as a full URL including scheme, ex: https://127.0.0.1:2379. | http://127.0.0.1:2379 |
| CADDY_CLUSTERING_ETCD_PREFIX | A prefix that will be added to each Caddy-managed file to separate it from other keys you have in your etcd cluster | /caddy |
| CADDY_CLUSTERING_ETCD_TIMEOUT | The timeout for locks on Caddy resources.  In the event of a failure or network issue, the lock on a particular resource will timeout after this value, allowing another operation to try to write that value.  Must be expressed as a Go-style duration, like 5m, 30s. | 5m |
| CADDY_CLUSTERING_ETCD_CADDYFILE | The plugin includes a Caddyfile loader that will read Caddyfile configuration from <Key Prefix>/caddyfile.  If this file exists in etcd, it will be used as the Caddyfile configuration.  This environment variable allows you to bootstrap a clustered configuration from an existing Caddyfile on disk.  When set, it will load this file and store it in etcd for other cluster members to use.  If both etcd and a Caddyfile on disk exist, the configuration in etcd will be used. | |
| CADDY_CLUSTERING_ETCD_CADDYFILE_LOADER | To disable loading/storing Caddyfile configuration from etcd, set this to "disable" | enable |

## Building Caddy with this Plugin

This plugin requires caddy to be built with go modules.  It cannot be built by the current buildserver on caddyserver.com because it does not support modules.  This project uses mage to build the caddy binary.  To run it, first download mage:

```
go get -u github.com/magefile/mage
```

You must have the following binaries available on your system to run the build:

```
go >= 1.11
sed
```

Then build by running

```
mage build
```

See the `magefile.go` for other customizations, such as including other plugins in your custom build.

## Testing

This project uses go modules and must be tested with the `-mod vendor` flag in order to use vendored dependencies that have been modified to work with caddy.

```
go test -v -cover -race -mod vendor
```

## Roadmap

[ ] etcd mutual TLS support

[ ] incremental Caddyfile configuration - allow new keys inserted under <KeyPrefix>/caddyfile to modify the running Caddy configuration (e.g., add a new site by writing to etcd under /caddy/caddyfile/mysite with just the site's configuration)

[ ] make plugin buildable on caddyserver.com