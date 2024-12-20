# Hashicorp Consul Agent Input Plugin

This plugin collects metrics from a [Consul agent][agent]. Telegraf may be
present in every node and connect to the agent locally. Tested on Consul v1.10.

⭐ Telegraf v1.22.0
🏷️ server
💻 all

[agent]: https://developer.hashicorp.com/consul/commands/agent

## Global configuration options <!-- @/docs/includes/plugin_config.md -->

In addition to the plugin-specific configuration settings, plugins support
additional global and plugin configuration settings. These settings are used to
modify metrics, tags, and field or create aliases and configure ordering, etc.
See the [CONFIGURATION.md][CONFIGURATION.md] for more details.

[CONFIGURATION.md]: ../../../docs/CONFIGURATION.md#plugins

## Configuration

```toml @sample.conf
# Read metrics from the Consul Agent API
[[inputs.consul_agent]]
  ## URL for the Consul agent
  # url = "http://127.0.0.1:8500"

  ## Use auth token for authorization.
  ## If both are set, an error is thrown.
  ## If both are empty, no token will be used.
  # token_file = "/path/to/auth/token"
  ## OR
  # token = "a1234567-40c7-9048-7bae-378687048181"

  ## Set timeout (default 5 seconds)
  # timeout = "5s"

  ## Optional TLS Config
  # tls_ca = /path/to/cafile
  # tls_cert = /path/to/certfile
  # tls_key = /path/to/keyfile
```

## Metrics

Consul collects various metrics. For every details, please have a look at
[Consul's documentation](https://www.consul.io/api/agent#view-metrics).

## Example Output
