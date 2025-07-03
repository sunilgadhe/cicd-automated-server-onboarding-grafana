Jenkins Build Step Script
#!/bin/bash
set -e

echo "SSH Key file path: $SSH_KEY"
ls -l "$SSH_KEY"

IFS=',' read -ra IP_ARRAY <<< "$TARGET_IPS"

for ip in "${IP_ARRAY[@]}"; do
  echo "==> Copying script to $ip:/opt"

  scp -o StrictHostKeyChecking=no -i "$SSH_KEY" install_node_exporter.sh ubuntu@$ip:/tmp/

  ssh -o StrictHostKeyChecking=no -i "$SSH_KEY" ubuntu@$ip <<EOF
    sudo mv /tmp/install_node_exporter.sh /opt/install_node_exporter.sh
    sudo chmod +x /opt/install_node_exporter.sh
    sudo bash /opt/install_node_exporter.sh
EOF

  echo "==> Node Exporter installed on $ip"

  echo "==> Adding $ip to Prometheus targets on <Prometheus_Server_IP>"

  ssh -o StrictHostKeyChecking=no -i "$SSH_KEY" ubuntu@<Prometheus_Server_IP> bash -c "'
    sudo cp /home/ubuntu/prometheus/prometheus.yml /home/ubuntu/prometheus/prometheus.yml.bak.\$(date +%F-%T)

    # Append new target IP to node job targets
    if ! sudo yq e \".scrape_configs[] | select(.job_name == \\\"node\\\") | .static_configs[0].targets[]\" /home/ubuntu/prometheus/prometheus.yml | grep -q \"$ip:9100\"; then
      sudo yq e -i \".scrape_configs[] |=
        (select(.job_name == \\\"node\\\") | .static_configs[0].targets += [\\\"$ip:9100\\\"])\" /home/ubuntu/prometheus/prometheus.yml
    else
      echo \"$ip:9100 already exists in Prometheus config, skipping\"
    fi

    # Reload Prometheus to apply config changes
    sudo systemctl reload prometheus || sudo kill -HUP \$(pidof prometheus) #Works if prom & grafana deployed without containers
  '"

done
