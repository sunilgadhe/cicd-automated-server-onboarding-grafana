# Jenkins Pipeline: Grafana Server Onboarding via Node Exporter

This repository contains a Jenkins pipeline that automates the onboarding of remote servers to Prometheus/Grafana by installing Node Exporter, labeling them, and updating Prometheus configuration dynamically.

---

## 🚀 High-Level Workflow

1. Jenkins prompts the user for input (IPs, labels like `TEAM`, `ENV`, `PROJECT`)
2. Copies a Bash installer script (`install_node_exporter.sh`) to `/opt` on remote servers
3. SSH into remote servers using a private key stored securely in Jenkins credentials
4. Executes the script to install Node Exporter
5. Updates Prometheus target configuration with fixed labels
6. Reloads Prometheus to reflect new targets

---

## ✅ Prerequisites

Ensure the following are set up before using this pipeline:

- Jenkins is installed and running
- Admin access to Jenkins
- A Bash script: `install_node_exporter.sh`
- Build step script (included in Jenkins job)
- EC2 private key (PEM file)
- IP addresses of remote target servers
- `yq` installed on the Prometheus server:

```bash
# For Linux AMD64
sudo wget https://github.com/mikefarah/yq/releases/latest/download/yq_linux_amd64 -O /usr/local/bin/yq
sudo chmod +x /usr/local/bin/yq
