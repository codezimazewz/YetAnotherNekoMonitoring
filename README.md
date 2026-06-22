<div align="center">
  <img src="https://media1.tenor.com/m/m65_yUALT48AAAAC/anime-nekopara.gif" width="480" height="350" alt="Cute Cat">
</div>

<h1 align="center">YetAnotherNekoMonitoring ( ^..^)ﾉ</h1>

<div align="center">
  <img src="https://img.shields.io/badge/Docker-Compose-blue?style=for-the-badge&logo=docker" alt="Docker">
  <img src="https://img.shields.io/badge/Prometheus-2.45-orange?style=for-the-badge&logo=prometheus" alt="Prometheus">
  <img src="https://img.shields.io/badge/Grafana-10.0-orange?style=for-the-badge&logo=grafana" alt="Grafana">
  <img src="https://img.shields.io/badge/Graylog-5.0-blue?style=for-the-badge&logo=graylog" alt="Graylog">
  <img src="https://img.shields.io/badge/Neko-Approved-ff69b4?style=for-the-badge" alt="neko">
</div>

<p align="center">
  A complete monitoring and logging stack based on Docker Compose for infrastructure. Includes metrics collection, visualization, alerting, and centralized logging. ( ˶^▾^˶ )
</p>

⚠️ **Important**: Adapt the configuration to your infrastructure. Change passwords, configure TLS and backups before production.

## Who will benefit from this?
- *For developers and sysadmins who want a full monitoring stack on a home server or small production environment that works out of the box with little to no setup.*

## 🚀 Features ( ˶^▾^˶ )

- **Metrics**: Prometheus + Node Exporter + cAdvisor
- **Visualization**: Grafana with pre-built dashboards
- **Alerting**: Alertmanager with configurable rules
- **Logging**: Graylog + Filebeat for container logs
- **Auto-configuration**: All services start with minimal configuration
- **Scalability**: Easily extensible for new services

### Components

- **Prometheus**: Metrics collection and storage
- **Grafana**: Metrics and logs visualization
- **Graylog**: Centralized logging
- **OpenSearch**: Search engine for logs
- **MongoDB**: Graylog configuration storage
- **Alertmanager**: Alert management
- **Node Exporter**: Host metrics (CPU, memory, disk)
- **cAdvisor**: Docker container metrics
- **Filebeat**: Log collector from containers

## 📋 Quick Start

### Prerequisites

- Docker >= 20.10 (check w/ *docker -v*)
- Docker Compose >= 2.0  (check /w *docker compose version*)
- Minimum 6GB RAM, 2 CPU cores
- Minimum 20GB free disk space (for metrics and logs)

### Installation and Launch

1. **Clone the repository:**
   ```bash
   git clone <repository-url>
   cd YetAnotherNekoMonitoring
   ```

2. **Copy the example env file and set secrets:**
   ```bash
   cp .env.example .env
   # Edit .env to set passwords and secrets
   # Generate SHA-256 for Graylog root password:
   # echo -n "your-strong-password" | sha256sum
   ```

3. **Start the stack:**
   ```bash
   docker compose up -d
   ```

4. **Wait 1-2 minutes** for all services to fully start (especially Graylog, MongoDB, and OpenSearch).

5. **Check status:**
   ```bash
   docker compose ps
   ```

All services will start automatically with ready configuration.

## 🐾 Post-Installation
The stack starts in "Vanilla Mode". To get the graphs purring:

1. **Grafana**: Login with `GRAFANA_ADMIN_USER` / `GRAFANA_ADMIN_PASSWORD` from `.env`.
2. **Data source**: Prometheus datasource is provisioned automatically (`http://prometheus:9090`).
3. **Dashboards**: Node Exporter dashboard is provisioned from `grafana/dashboards/`.
4. **Graylog**: GELF UDP input on `12201/udp` is created automatically by `graylog-init`.
5. **Verify**: Visit Prometheus and make sure all targets are UP.

## 🌐 Access to Services
After launch, the stack is available at the following addresses:

| Service       | URL                          | Login/Password     | Description |
|---------------|------------------------------|--------------------|-------------|
| **Grafana**   | http://localhost:3000        | from `.env` (`GRAFANA_ADMIN_USER` / `GRAFANA_ADMIN_PASSWORD`) | Dashboards and visualization |
| **Prometheus**| http://localhost:9090        | -                 | Metrics and alerts |
| **Graylog**   | http://localhost:9000        | from `.env` (`GRAYLOG_ADMIN_USER` / `GRAYLOG_ROOT_PASSWORD_PLAIN`) | Logs and search |
| **Alertmanager**| http://localhost:9093      | -                 | Alert management |
| **cAdvisor**  | http://localhost:8080        | -                 | Container metrics |
| **Node Exporter**| http://localhost:9100     | -                 | System metrics |

## 📊 Monitoring

### Metrics

- **System**: CPU, memory, disk, network (Node Exporter)
- **Containers**: Resource usage, statistics (cAdvisor)
- **Services**: Uptime, errors, performance (Prometheus)

## 📝 Logging

- **Collection**: Filebeat automatically collects logs from all Docker containers
- **Aggregation**: Graylog receives logs via GELF UDP (port 12201)
- **Storage**: OpenSearch indexes logs for fast search
- **Search**: Use Graylog UI for filtering and log analysis

## 🚨 Alerting

- **Rules**: Configured in `prometheus/alert-rules.yml`
- **Example rule**: Alerts for service unavailability (`InstanceDown`)

### Example notification flow

After configuring a webhook in `alertmanager/config.yml`, you can send alerts to Telegram (or any other service).

> Example: When Node Exporter goes down, Alertmanager can send a Telegram message via a webhook.

- **Delivery**: Webhook (change in `alertmanager/config.yml` to Slack/Email/Telegram/etc.)

## ⚙️ Configuration

### Main Files

- `docker-compose.yml`: Service definitions
- `.env` (from `.env.example`): Credentials and secrets
- `prometheus/prometheus.yml`: Metrics collection config
- `prometheus/alert-rules.yml`: Alert rules
- `alertmanager/config.yml`: Notification settings
- `grafana/provisioning/`: Data sources and dashboards
- `filebeat/filebeat.yml`: Log collection config

### Customization

1. **Add a new service for monitoring:**
   - Add a job in `prometheus/prometheus.yml`
   - Restart: `docker compose restart prometheus`

2. **New dashboard in Grafana:**
   - Import JSON in Grafana UI
   - Or add to `grafana/dashboards/`

3. **Additional alerts:**
   - Edit `prometheus/alert-rules.yml`
   - Reload rules: `curl -X POST http://localhost:9090/-/reload`

4. **Passwords/secrets rotation:**
   - Update values in `.env`
   - Ensure `GRAYLOG_ROOT_PASSWORD_SHA2` matches `GRAYLOG_ROOT_PASSWORD_PLAIN`
   - Restart affected services: `docker compose up -d`

## 🔧 Extending

### Adding New Exporters

```yaml
# In docker-compose.yml
new_exporter:
  image: prom/new-exporter:latest
  ports:
    - "9999:9999"
  networks:
    - monitoring
```

Then add to `prometheus/prometheus.yml`:

```yaml
- job_name: "new_exporter"
  static_configs:
    - targets: ["new_exporter:9999"]
```

### Integration with External Services

- **Nginx reverse proxy**: Proxy ports through the internal `monitoring` network
- **TLS**: Add certificates and configure HTTPS
- **Authentication**: Enable in Grafana/Graylog
- **Backup**: Configure volumes for regular backups

## 🐛 Troubleshooting

### OpenSearch system requirement

OpenSearch may require a higher `vm.max_map_count` value on Linux hosts.

```bash
sudo sysctl -w vm.max_map_count=262144
```
To make this change permanent across reboots:
```bash
echo "vm.max_map_count=262144" | sudo tee -a /etc/sysctl.conf
```

### Services Not Starting

```bash
# Check logs
docker compose logs <service-name>

# Restart specific service
docker compose restart <service-name>
```

### Graylog Not Receiving Logs

- Ensure GELF UDP input is created (automatically via `graylog-init`)
- Check port 12201/UDP is open
- Filebeat logs: `docker compose logs filebeat`

### Metrics Not Collected

- Check targets in Prometheus UI
- Ensure exporters are accessible in the `monitoring` network

### Memory/Disk

- **OpenSearch**: May require more RAM — increase `OPENSEARCH_JAVA_OPTS`
- **Graylog**: Configure retention in Graylog UI
- **Volumes**: Clean unused volumes: `docker volume prune`

## 📚 Additional Resources

- [Prometheus Docs](https://prometheus.io/docs/)
- [Grafana Docs](https://grafana.com/docs/)
- [Graylog Docs](https://docs.graylog.org/)
- [Docker Compose](https://docs.docker.com/compose/)

## 🤝 Contributing

PRs are welcome! ( ^..^)ﾉ For major changes, create an issue first.

## 📄 License

MIT License - see [LICENSE](LICENSE) file (if exists).

---
```
 /\_/\  
( o.o ) 
 > ^ <
```

