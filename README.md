# Ansible Docker Compose

Yet another role to deploy and manage docker compose services with ansible

## Example Configuration

```yaml
# Networks:
docker_networks:
  - web
  - grafana

# Volumes:
docker_volumes:
  - name: prometheus_data
    state: present
  - name: grafana_data
    state: present

# Services:
docker_compose_services:
  - name: prometheus
    state: present
    pull: true
    restarted: true

    environment:
      PROMETHEUS_VERSION: v2.30.3
      PROMETHEUS_HOST: "prometheus.{{ fqdn }}"

    # Additional host specific files to rsync to the host
    synchronize: 
      - dest: /var/lib/prometheus
        src: "{{ inventory_hostname }}/prometheus/"
        delete: true

  - name: grafana
    state: present
    pull: true
    restarted: true
    
    environment:
      GRAFANA_VERSION: 8.2.1
      GRAFANA_HOST: "grafana.{{ fqdn }}"
      GRAFANA_ADMIN_USER: "{{ secret_grafana_admin_user }}"
      GRAFANA_ADMIN_PASSWORD: "{{ secret_grafana_admin_password }}"

    synchronize:
      - dest: /var/lib/grafana
        src: "{{ inventory_hostname }}/grafana/"
        delete: false

  # Git & custom compose file support 
  - name: my-app
    repo: git@github.com:A/my-app.git
    files:
      - docker-compose.prod.yml
    version: master
    state: present
    pull: true
    build: true
    restarted: true
    environment:
      MY_APP_HOST: my-app.{{ fqdn }}

  ## Pull image from a private registry
  - name: my-app
    state: present
    pull: true

    docker_login:
      registry_url: https://ghcr.io/
      username: "{{ secret_github_username }}"
      password: "{{ secret_github_pat }}"

    environment:
      CPUS: 28
```
