[![CI](https://github.com/DominiqueFuchs/ansible-loki-compose/actions/workflows/ci.yaml/badge.svg?branch=main&event=push)](https://github.com/DominiqueFuchs/ansible-loki-compose/actions/workflows/ci.yaml)

Role Name
=========

Ansible role to deploy production-ready (**as in** fully configured and safely architected/deployed ready-to-use unit - **not as in** scalable, elastic and highly-available architecture. There are [other setup options](https://grafana.com/docs/loki/latest/installation/) and [deployment modes](https://grafana.com/docs/loki/latest/fundamentals/architecture/deployment-modes/) that are more sophisticated and should be considered) Loki environments with Docker Compose.

The deployed environment includes Loki, Grafana and Alertmanager services as well as (optional, enabled by default) nginx-proxy and ACME services to provide a proxied, SSL-enabled and Let's Encrypt validated web interface for Grafana. In every case, there exists only two individually configurable network bindings to the host-side (one for the Loki service and one for either for the proxy-service or for the Grafana service if the proxy service is disabled), all other communication is handled solely within the docker network that is set up by the compose file.

While the the services itself are stateless in a sense that configuration is completely handled by the provided configuration options (and the generated configurations are bound read-only within the container), you can individually configure bind-mounts for Grafana (settings, dashboards, users, ...) and Loki (mostly index and chunk data if you're using a filesystem backend) in case you want to persist or back up the corresponding data locally.

**Note:**
* The username of the built-in Grafana admin user is set to 'grafana_admin' (which differs from the default 'admin')
* Anonymous access is disabled for Grafana

Requirements
------------

This role does not take care of docker installation or configuration but relies on a working docker setup on the host. You may want to have a look at the [ansible docker role from geerlingguy](https://github.com/geerlingguy/ansible-role-docker) for an easy way to ensure such an installation on the targets as well as the configuration variables regarding te docker command for this role below.

Role Variables
--------------

A description of the settable variables for this role should go here, including any variables that are in defaults/main.yml, vars/main.yml, and any variables that can/should be set via parameters to the role. Any variables that are read from other roles and/or the global scope (ie. hostvars, group vars, etc.) should be mentioned here as well.

Dependencies
------------

None.

Example Playbook
----------------

With a working docker compose (v2) setup on the target, the shortest (but by relying solely on default values neither highly useful nor secure) example to test this role out would simply be:

    - hosts: all
      roles:
        - role: dfuchs.loki_compose
          vars:
            lc_enable_acme_proxy: false

---

For a more customized and useful environment, one may want to

- specify a service user to use (must exist and be a member of the docker group)
- set the bind address of Loki to an internal interfaces IP (in the example below, 'internal_interface' is a variable definition that may be set on the host within the inventory, e.g. 'ens10')
- set the vhost / target domain for the Grafana server (with a corresponding DNS record pointing to your targets address)
- provide an own password for the grafana_admin
- disable test (staging) mode for Let's Encrypt and provide a valid contact email
- install the systemd service and ensure it is enabled and immediately started
 
Which gives:

    - hosts: all
      roles:
        - role: dfuchs.loki_compose
          vars:
            lc_service_user: loki
            lc_grafana_vhost: your.domain.tld
            lc_loki_bind_address: lc_loki_bind_address: '{{ hostvars[inventory_hostname]["ansible_"~internal_interface].ipv4.address }}'
            lc_grafana_password: 'l3A6MN94$BGq'
            lc_letsencrypt_test: false
            lc_acme_mail: 'yourname@example.tld'
            lc_service_manage: true

License
-------

MIT

