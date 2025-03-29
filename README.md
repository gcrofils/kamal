# Kamal: Deploy web apps anywhere

From bare metal to cloud VMs, deploy web apps anywhere with zero downtime. Kamal uses [kamal-proxy](https://github.com/basecamp/kamal-proxy) to seamlessly switch requests between containers. Works seamlessly across multiple servers, using SSHKit to execute commands. Originally built for Rails apps, Kamal will work with any type of web app that can be containerized with Docker.

➡️ See [kamal-deploy.org](https://kamal-deploy.org) for documentation on [installation](https://kamal-deploy.org/docs/installation), [configuration](https://kamal-deploy.org/docs/configuration), and [commands](https://kamal-deploy.org/docs/commands).

## About this fork

As a follow-up to [#1377](https://github.com/basecamp/kamal/issues/1377), after a few hours of trial and error, here’s a KISS workaround to use a wildcard Cloudflare certificate with `kamal`.

**Disclaimer:**
- This is a temporary, hardcoded setup
- Manual cert installation on a single target server
- Goal: migrate a low-traffic multitenant service from Heroku

---

### 1. Forked Kamal and modified `Kamal::Configuration::Proxy#deploy_options`:

```ruby
def deploy_options
  {
    ...
    "log-response-header": proxy_config.dig("logging", "response_headers"),
    "tls-certificate-path": "/home/kamal-proxy/.config/certs/cert.pem",
    "tls-private-key-path": "/home/kamal-proxy/.config/certs/key.pem",
  }.compact
end
```

---

### 2. In `Gemfile`:

```ruby
gem 'kamal', require: false, git: "https://github.com/USERNAME/kamal"
```

---

### 3. Manually copy the certs to the server:

```bash
/etc/kamal/certs/
├── cert.pem  (644)
└── key.pem   (644)
```

---

### 4. Add `.kamal/proxy/options` on the server:

```bash
--publish 80:80 --publish 443:443 --log-opt max-size=10m --volume /etc/kamal/certs:/home/kamal-proxy/.config/certs:ro --env
```

This works because `kamal` reads this file when launching the proxy container:

https://github.com/basecamp/kamal/blob/8fe2f921645ad8416fabd2aae64a0ca2000487ad/test/commands/proxy_test.rb#L18


## Contributing to the documentation

Please help us improve Kamal's documentation on the [the basecamp/kamal-site repository](https://github.com/basecamp/kamal-site).

## License

Kamal is released under the [MIT License](https://opensource.org/licenses/MIT).
