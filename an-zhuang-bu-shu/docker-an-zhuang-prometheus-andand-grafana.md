1、安装docker（指定版本号）

[https://yum.dockerproject.org/repo/main/centos/7/Packages/](https://yum.dockerproject.org/repo/main/centos/7/Packages/) 选择版本号

```
wget https://yum.dockerproject.org/repo/main/centos/7/Packages/docker-engine-1.12.6-1.el7.centos.x86_64.rpm
yum localinstall -y docker-engine*
systemctl enable docker.service
systemctl start docker.service
```

2、下载镜像

```
docker pull docker.io/prom/prometheus
docker pull quay.io/prometheus/node-exporter  
docker pull grafana/grafana
docker pull google/cadvisor
```

3、配置prometheus

    vim /data/docker/prometheus/config/prometheus.yml
    # my global config
    global:
      scrape_interval:     15s # Set the scrape interval to every 15 seconds. Default is every 1 minute.
      evaluation_interval: 15s # Evaluate rules every 15 seconds. The default is every 1 minute.
      # scrape_timeout is set to the global default (10s).

    # Alertmanager configuration
    alerting:
      alertmanagers:
      - static_configs:
        - targets:
          # - alertmanager:9093

    # Load rules once and periodically evaluate them according to the global 'evaluation_interval'.
    rule_files:
      # - "first_rules.yml"
      # - "second_rules.yml"

    # A scrape configuration containing exactly one endpoint to scrape:
    # Here it's Prometheus itself.
    scrape_configs:
      # The job name is added as a label `job=<job_name>` to any timeseries scraped from this config.
      - job_name: 'prometheus'

        # metrics_path defaults to '/metrics'
        # scheme defaults to 'http'.

        static_configs:
        - targets: ['localhost:9090','localhost:9100','localhost:8080']


    生成容器

    docker run -d -p 9090:9090 --net=host -v /data/docker/prometheus/config/prometheus.yml:/etc/prometheus/prometheus.yml /data/docker/prometheus/data:/prometheus  --name prometheus prom/prometheus

4、配置export

```
docker run -d -p 9100:9100   --net="host"   --name=promethus-node-export   quay.io/prometheus/node-exporter 
docker run -d -i -p 8080:8080 --detach=true --name=cadvisor --net=host google/cadvisor:latest
```

5、配置grafana

```
docker run -d -i -p 3000:3000 --net=host -e "GF_SECURITY_ADMIN_PASSWORD=admin"  grafana/grafana

```

后期注意将 /var/lib/grafana 和  /etc/grafana/grafana.ini 挂载出来。

