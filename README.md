[![CI](https://github.com/DominiqueFuchs/ansible-grafana-compose/actions/workflows/ci.yaml/badge.svg?branch=main&event=push)](https://github.com/DominiqueFuchs/ansible-grafana-compose/actions/workflows/ci.yaml)
[![Ansible Role](https://img.shields.io/ansible/role/62334?label=galaxy&logo=ansible)](https://galaxy.ansible.com/dominiquefuchs/grafana_compose)

     ansible-galaxy install dominiquefuchs.grafana_compose

Ansible Role grafana_compose
=========

Ansible role to deploy production-ready (**as in** fully configured and safely architected/deployed ready-to-use unit - **not as in** scalable, elastic and highly-available architecture. There are other setup options and deployment modes that are more sophisticated and should be considered) Grafana environments with Docker Compose.

The deployed environment can include Grafana, Prometheus, Loki and Alertmanager services as well as nginx-proxy and ACME services to provide a proxied, SSL-enabled web interface for Grafana.

While the the services itself are stateless in a sense that configuration is completely handled by the provided configuration options (and the generated configurations are bound read-only within the container), you can individually configure bind-mounts for Grafana, Prometheus and Loki in case you want to persist or back up the corresponding data locally.

**Note:**
* The username of the built-in Grafana admin user is set to 'grafana_admin' (which differs from the default 'admin')
* Anonymous access is disabled for Grafana

Requirements
------------

This role does not take care of docker installation or configuration but relies on a working docker setup on the host. You may want to have a look at the [ansible docker role from geerlingguy](https://github.com/geerlingguy/ansible-role-docker) for an easy way to ensure such an installation on the targets as well as the configuration variables regarding te docker command for this role below.

Role Variables
--------------

All variables used within this role (internal ones as well as defaults meant to be overriden) start with an *gc_* prefix. Below is a list of all default variables that can be used to configure the resulting deployment.

| Variable                      | Default value and explanation |
|---                            |---                                  |
| gc_base_directory             | */srv* — Base directory for the deployment of the compose fileset (which will be placed in *{{gc_base_directory}}/GrafanaCompose/*) |
| gc_service_user               | *root* — The user that will be owner of the deployment files and run the containers (will also be set in systemd service, if enabled) |
| gc_compose_major_version      | *2* — Major version of the compose installation. This setting also determines the binary command (*docker-compose* vs. *docker compose*) embedded in the systemd service, if enabled |
| gc_docker_bin_dir             | *None* – If left empty, docker binary path will be automatically determined at runtime of the play |
| gc_loki_bind_addresses        | *['127.0.0.1']* — Bind address(es) for the port definition of the Loki container. Port is fixed on 3100 |
| gc_prometheus_bind_addresses  | *['127.0.0.1']* — Bind address(es) for the port definition of the Prometheus container. Port is fixed on 3100 |
| gc_grafana_bind_addresses     | *['127.0.0.1']* — Bind address(es) for the port definition of the Grafana container. Port is fixed on 3000. Will only take effect if *gc_enable_acme_proxy* is disabled, otherwise Grafana container is **not** bound to the host side and only exposed within the compose network |
| gc_grafana_vhost              | *grafana.example.tld* — domain record to be used for the virtual host definition of the proxy and ACME services, if enabled. Unused otherwise |
| gc_grafana_password            *p@ssword_to_be_replaced* — self-explanatory |
| gc_enable_acme_proxy          | *true* — Enable or disable generation of proxy and ACME container services. If disabled, the Grafana container will be bound to the host interface directly (see also *gc_grafana_bind_address*) |
| gc_nginx_no_robots            | *true* — Enable or disable X-Robots-Tag header containing 'noindex, nofollow, nosnippet, noarchive' |
| gc_nginx_bind_address         | *0.0.0.0* — Bind address for the port defiition of the nginx proxy container. Ports are fixed to 80 and 443 |
| gc_letsencrypt_test_mode      | *true* — Enable or disable the staging mode of the ACME service. When enabled, the Let's Encrypt staging URL is used and resulting certificates will generate a browser warning |
| gc_acme_mail                  | *admin@example.tld* — The contact email that is used during the registration process of new certificates from Let's Encrypt |
| gc_alertmanager_default_receiver  | *black-hole* — Default receiver to use for unmatched alerts. *black-hole* is a dummy receiver without any action and is only generated when no other receivers are defined (see *gc_alertmanager_receivers*) |
| gc_alertmanager_receivers     | *[]* — Receiver configuration(s) for Alertmanager. See the [Alertmanager documentation on receiver configurations](https://prometheus.io/docs/alerting/latest/configuration/#receiver) as well as the [Alertmanager example configuration](https://github.com/prometheus/alertmanager/blob/main/doc/examples/simple.yml). Each receiver needs a unique name (which can in turn be set as *gc_alertmanager_default_receiver* and a corresponding config element) |
| gc_service_manage             | *false* — Enable or disable service creation and state control for systemd. The service name is grafana-compose |
| gc_service_state              | *started* — Set the desired service state to ensure. Possible values are 'started', 'stopped', 'restarted' and 'reloaded' |
| gc_service_enabled            | *true* — Enable or disable the systemd service to run at startup |
| gc_loki_bind_mount_dir        | *None* — Host directory to use for bind mount volume of the Loki containers data directory. If unset or empty, no named or bind mount volume will be created |
| gc_prometheus_bind_mount_dir  | *None* — Host directory to use for bind mount volume of the Prometheus containers data directory. If unset or empty, a named volume will be created instead |
| gc_grafana_bind_mount_dir     | *None* — Host directory to use for bind mount volume of the Grafana containers data directory. If unset or empty, a named volume will be created instead |
| gc_nginx_log_bind_mount_dir   | *None* — Host directory to use for bind mount volume of the nginx containers log directory. If unset or empty, no named or bind mount volume will be created |
| gc_loki_retention_period      | *31d* — Retention period to apply to stored data. Note that automatic deletion of chunk data only happens when filesystem storage is used, Loki will not delete data on remote/cloud backends |
| gc_node_exporters             | *[]* — List of IPs/FQDNs including ports with node exporter instances that should be scraped |

Dependencies
------------

None.

Example Playbook
----------------

With a working docker compose (v2) setup on the target, the shortest (but by relying solely on default values neither highly useful nor secure) example to test this role out would simply be:

    - hosts: all
      roles:
        - role: dominiquefuchs.grafana_compose
          vars:
            gc_enable_acme_proxy: false

---

For a more customized and useful environment, one may want to

- specify a service user to use (must exist and be a member of the docker group)
- set the vhost / target domain for the Grafana server (with a corresponding DNS record pointing to your targets address)
- set the bind address of Loki and Prometheus to an internal interfaces IP (in the example below, 'internal_interface' is a variable definition that may be set on the host within the inventory, e.g. 'ens10')
- provide an own password for the grafana_admin
- disable test (staging) mode for Let's Encrypt and provide a valid contact email
- install the systemd service and ensure it is enabled and immediately started
- define a receiver for alertmanager and set it as default receiver - the example below configures a Telegram bot/channel (the correct URL would be *https://api.telegram.org* - just put a fake tld there to prevent spamming from copy/paste startups)
- define a list of node exporter instances for Prometheus to scrape
 
Which gives:

    - hosts: all
      roles:
        - role: dominiquefuchs.grafana_compose
          vars:
            gc_service_user: grafana
            gc_grafana_vhost: your.domain.tld
            gc_loki_bind_addresses: ['127.0.0.1', '{{ hostvars[inventory_hostname]["ansible_"~internal_interface].ipv4.address }}']
            gc_prometheus_bind_addresses: ['127.0.0.1', '{{ hostvars[inventory_hostname]["ansible_"~internal_interface].ipv4.address }}']
            gc_grafana_password: 'l3A6MN94$BGq'
            gc_letsencrypt_test_mode: false
            gc_acme_mail: 'yourname@example.tld'
            gc_service_manage: true
            gc_alertmanager_default_receiver: test-receiver
            gc_alertmanager_receivers:
              - name: test-receiver
                telegram_configs:
                  - bot_token: '123456'
                    send_resolved: true
                    api_url: 'https://api.telegram.example'
                    chat_id: 123456
                    parse_mode: ''
            gc_node_exporters: ['1.2.3.4:9100']

License
-------

MIT

Credits
-------

 🦸‍♂️ - [geerlingguy](https://github.com/geerlingguy) for his great collection of ansible roles and helpful content

 ⚗️ - [Grafana Labs](https://github.com/grafana) for providing a really neat, clean and beautiful stack of log, monitoring and visualization components
