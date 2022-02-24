# 1. Prometheus

## 1.1 安装和启动

```shell
# 创建文件夹
mkdir -p /data/app/prometheus && cd /data/app/prometheus
# 解压安装包
tar xzvf prometheus-2.28.1.linux-amd64.tar.gz
# 进入解压后的目录
cd prometheus-2.28.1.linux-amd64/

# 备份并修改配置
cp prometheus.yml prometheus.yml.bak

# 启动服务
nohup ./prometheus --config.file=./prometheus.yml --web.listen-address=:9090 --web.enable-admin-api --web.enable-lifecycle > /dev/null 2>&1 &

############## 或者 - start ##############
# 添加到系统服务，方便管理
vim /etc/systemd/system/prometheus.service
# 文件内容如下：
[Unit]
Description=Prometheus Monitoring System
Documentation=Prometheus Monitoring System
 
[Service]
ExecStart=/data/app/prometheus/prometheus-2.28.1.linux-amd64/prometheus \
  --config.file=/data/app/prometheus/prometheus-2.28.1.linux-amd64/prometheus.yml \
  --web.listen-address=:9090 \
  --web.enable-admin-api \
  --web.enable-lifecycle
 
[Install]
WantedBy=multi-user.target

# 设置开机自动启动
sudo systemctl enable prometheus

# 启动
sudo systemctl daemon-reload
sudo systemctl start prometheus
sudo systemctl status prometheus
############## 或者 - end ##############

# 访问地址：http://10.16.226.51:9090/targets
```

## 1.2 配置监控

### 1.2.1 Linux 主机

```shell
# 创建文件夹
mkdir -p /data/app/prometheus/node_exporter && cd /data/app/prometheus/node_exporter
# 解压安装包
tar xzvf node_exporter-1.1.2.linux-amd64.tar.gz
# 进入解压后的目录
cd node_exporter-1.1.2.linux-amd64/
# 启动
nohup ./node_exporter > /dev/null 2>&1 &
# 验证
curl http://localhost:9100/metrics | grep "node_"

# 修改 Prometheus 配置文件
vim /data/app/prometheus/prometheus-2.28.1.linux-amd64/prometheus.yml
# 在配置文件末尾添加如下内容（注意缩进）：
  - job_name: 'nodes'
    static_configs:
    - targets:
      - "10.16.226.51:9100"
      - "10.16.2.52:9100"
      # 可以继续添加多个主机 ……

# 重新加载 Prometheus 服务器配置
curl -X POST http://10.16.226.51:9090/-/reload
```

### 1.2.2 Spring Boot 微服务

```shell
# 修改 Prometheus 配置文件
vim /data/app/prometheus/prometheus-2.28.1.linux-amd64/prometheus.yml
# 在配置文件末尾添加如下内容（注意缩进）：
  - job_name: 'spring-boot'
    metrics_path: '/actuator/prometheus'
    consul_sd_configs:
      - server: 'http://10.16.2.52:7088'
        services: []
        username: "occ"
        password: "occ2018"
```

### 1.2.3 Redis

```shell
# 创建文件夹
mkdir -p /data/app/prometheus/redis_exporter && cd /data/app/prometheus/redis_exporter
# 解压安装包
tar xzvf redis_exporter-v1.24.0.linux-amd64.tar.gz
# 进入解压后的目录
cd redis_exporter-v1.24.0.linux-amd64/
# 启动
nohup ./redis_exporter -redis.addr 10.16.2.52:6379 -redis.password Occ2018 > /dev/null 2>&1 &

# 修改 Prometheus 配置文件
vim /data/app/prometheus/prometheus-2.28.1.linux-amd64/prometheus.yml
# 在配置文件末尾添加如下内容（注意缩进）：
  - job_name: 'redis'
    static_configs:
    - targets:
      - "10.16.226.51:9121"

# 重新加载 Prometheus 服务器配置
curl -X POST http://10.16.226.51:9090/-/reload
```

### 1.2.4 Zookeeper

```shell
# 创建文件夹
mkdir -p /data/app/zookeeper/prometheus && cd /data/app/zookeeper/prometheus
# 创建配置文件
vi zookeeper.yaml
rules:
  # replicated Zookeeper
  - pattern: "org.apache.ZooKeeperService<name0=ReplicatedServer_id(\\d+)><>(\\w+)"
    name: "zookeeper_$2"
    type: GAUGE
  - pattern: "org.apache.ZooKeeperService<name0=ReplicatedServer_id(\\d+), name1=replica.(\\d+)><>(\\w+)"
    name: "zookeeper_$3"
    type: GAUGE
    labels:
      replicaId: "$2"
  - pattern: "org.apache.ZooKeeperService<name0=ReplicatedServer_id(\\d+), name1=replica.(\\d+), name2=(\\w+)><>(Packets\\w+)"
    name: "zookeeper_$4"
    type: COUNTER
    labels:
      replicaId: "$2"
      memberType: "$3"
  - pattern: "org.apache.ZooKeeperService<name0=ReplicatedServer_id(\\d+), name1=replica.(\\d+), name2=(\\w+)><>(\\w+)"
    name: "zookeeper_$4"
    type: GAUGE
    labels:
      replicaId: "$2"
      memberType: "$3"
  - pattern: "org.apache.ZooKeeperService<name0=ReplicatedServer_id(\\d+), name1=replica.(\\d+), name2=(\\w+), name3=(\\w+)><>(\\w+)"
    name: "zookeeper_$4_$5"
    type: GAUGE
    labels:
      replicaId: "$2"
      memberType: "$3"
  # standalone Zookeeper
  - pattern: "org.apache.ZooKeeperService<name0=StandaloneServer_port(\\d+)><>(\\w+)"
    type: GAUGE
    name: "zookeeper_$2"
  - pattern: "org.apache.ZooKeeperService<name0=StandaloneServer_port(\\d+), name1=InMemoryDataTree><>(\\w+)"
    type: GAUGE
    name: "zookeeper_$2"

# 创建 java.env 文件
cd /data/app/zookeeper/apache-zookeeper-3.5.9-bin
vim conf/java.env
JMX_DIR="/data/app/zookeeper/prometheus"
export SERVER_JVMFLAGS="-javaagent:$JMX_DIR/jmx_prometheus_javaagent-0.16.0.jar=7070:$JMX_DIR/zookeeper.yaml $SERVER_JVMFLAGS"
# 重启 Zookeeper
bin/zkServer.sh restart

# 修改 Prometheus 配置文件
vim /data/app/prometheus/prometheus-2.28.1.linux-amd64/prometheus.yml
# 在配置文件末尾添加如下内容（注意缩进）：
  - job_name: 'zookeeper'
    static_configs:
    - targets: ['10.16.2.52:7070']

# 重新加载 Prometheus 服务器配置
curl -X POST http://10.16.226.51:9090/-/reload
```

### 1.2.5 RabbitMQ

```shell
### Docker 启动的 RabbitMQ 3.7.x ###
# 在 RabbitMQ 所在的宿主机上：
# 创建文件夹
mkdir -p /data/app/rabbitmq/plugins && cd /data/app/rabbitmq/plugins

# 拷贝宿主机上的插件文件到容器内部
docker cp /data/rabbitmq/plugins/accept-0.3.3.ez rabbitmq-occ:/plugins
docker cp /data/rabbitmq/plugins/prometheus-3.5.1.ez rabbitmq-occ:/plugins
docker cp /data/rabbitmq/plugins/prometheus_cowboy-0.1.4.ez rabbitmq-occ:/plugins
docker cp /data/rabbitmq/plugins/prometheus_httpd-2.1.8.ez rabbitmq-occ:/plugins
docker cp /data/rabbitmq/plugins/prometheus_process_collector-1.3.1.ez rabbitmq-occ:/plugins
docker cp /data/rabbitmq/plugins/prometheus_rabbitmq_exporter-3.7.2.3.ez rabbitmq-occ:/plugins

# 进入容器内部
docker exec -it rabbitmq-occ /bin/bash
# 启用插件
rabbitmq-plugins enable accept
rabbitmq-plugins enable prometheus
rabbitmq-plugins enable prometheus_cowboy
rabbitmq-plugins enable prometheus_httpd
rabbitmq-plugins enable prometheus_process_collector
rabbitmq-plugins enable prometheus_rabbitmq_exporter
# 退出并重启容器
exit
docker restart rabbit-occ

# 修改 Prometheus 配置文件
vim /data/app/prometheus/prometheus-2.28.1.linux-amd64/prometheus.yml
# 在配置文件末尾添加如下内容（注意缩进）：
  - job_name: 'rabbitmq'
    metrics_path: '/api/metrics'
    static_configs:
    - targets:
      - "10.16.2.57:15672"
### Docker 启动的 RabbitMQ 3.7.x ###

### Docker 启动的 RabbitMQ 3.8.x ###
docker run -d --name rabbitmq-occ -p 5672:5672 -p 15672:15672 -p 15692:15692 rabbitmq:3-management
# 在 RabbitMQ 所在的宿主机上：
# 进入容器内部
docker exec -it rabbitmq-occ /bin/bash
# 启用插件
rabbitmq-plugins enable rabbitmq_prometheus
# 退出并重启容器
exit
docker restart rabbit-occ
# 初步验证
curl -v -H "Accept:text/plain" "http://10.16.2.57:15692/metrics"

# 修改 Prometheus 配置文件
vim /data/app/prometheus/prometheus-2.28.1.linux-amd64/prometheus.yml
# 在配置文件末尾添加如下内容（注意缩进）：
  - job_name: 'rabbitmq'
    metrics_path: '/metrics'
    static_configs:
    - targets:
      - "10.16.2.57:15692"
### Docker 启动的 RabbitMQ 3.8.x ###

# 重新加载 Prometheus 服务器配置
curl -X POST http://10.16.226.51:9090/-/reload
```

### 1.2.6 Elasticsearch

```shell
# 创建文件夹
mkdir -p /data/app/prometheus/elasticsearch_exporter && cd /data/app/prometheus/elasticsearch_exporter
# 解压安装包
tar xzvf elasticsearch_exporter-1.2.1.linux-amd64.tar.gz
# 进入解压后的目录
cd elasticsearch_exporter-1.2.1.linux-amd64/
# 启动
nohup ./elasticsearch_exporter --es.uri http://10.16.226.51:9200 > /dev/null 2>&1 &

# 修改 Prometheus 配置文件
vim /data/app/prometheus/prometheus-2.28.1.linux-amd64/prometheus.yml
# 在配置文件末尾添加如下内容（注意缩进）：
  - job_name: 'elasticsearch'
    static_configs:
      - targets: ['10.16.226.51:9114']

# 重新加载 Prometheus 服务器配置
curl -X POST http://10.16.226.51:9090/-/reload
```

## 1.3 配置告警

```shell
# 创建文件夹
mkdir -p /data/app/prometheus/alertmanager && cd /data/app/prometheus/alertmanager
# 解压安装包
tar xzvf alertmanager-0.22.2.linux-amd64.tar.gz
# 进入解压后的目录
cd alertmanager-0.22.2.linux-amd64/
# 配置告警通知
vim alertmanager.yml
global:
  smtp_from: 'wangruiv@yonyou.com'
  smtp_smarthost: 'mail.yonyou.com:465'
  smtp_auth_username: 'wangruiv@yonyou.com'
  smtp_auth_password: '******'
  smtp_require_tls: false
route:
  group_by: ['alertname']
  group_wait: 30s
  group_interval: 5m
  repeat_interval: 1h
  receiver: 'email'
receivers:
- name: 'email'
  email_configs:
  - to: '1871363@qq.com'
    send_resolved: true
inhibit_rules:
  - source_match:
      severity: 'critical'
    target_match:
      severity: 'warning'
    equal: ['alertname', 'dev', 'instance']

# 启动
nohup ./alertmanager > alertmanager.log 2>&1 &
# 验证
curl http://10.16.226.51:9093

# 创建告警规则配置文件
cd /data/app/prometheus/prometheus-2.28.1.linux-amd64
vi alert_rules.yml
# 复制以下内容到文件中：
groups:
- name: example
  rules:
  # Alert for any instance that is unreachable for more than 2 minutes
  - alert: InstanceDown
    expr: up == 0
    for: 2m
    labels:
      severity: page
    annotations:
      summary: "Instance {{ $labels.instance }} down"
      description: "{{ $labels.instance }} of job {{ $labels.job }} has been down for more than 2 minutes."

  # Alert for any instance which has low free memory(<5%) for more than 2 minutes
  - alert: LowFreeMemory
    expr: (node_memory_MemFree / node_memory_MemTotal) * 100 < 5
    for: 2m
    annotations:
      summary: "Low free memory on {{ $labels.instance }}"
      description: "{{ $labels.instance }} has low free memory for more than 2 minutes (current value: {{ $value }}%)"

# 修改 Prometheus 配置文件
vim prometheus.yml
# Alertmanager configuration
alerting:
  alertmanagers:
  - static_configs:
    - targets:
      - "10.16.226.51:9093"

# Load rules once and periodically evaluate them according to the global 'evaluation_interval'.
rule_files:
  - "alert_rules.yml"

# 重新加载 Prometheus 服务器配置
curl -X POST http://10.16.226.51:9090/-/reload
```



# 2. Grafana

## 2.1 安装和启动

```shell
# 创建文件夹
mkdir -p /data/app/grafana && cd /data/app/grafana
# 使用 RPM 文件安装
wget https://dl.grafana.com/oss/release/grafana-8.0.4-1.x86_64.rpm
yum install -y grafana-8.0.4-1.x86_64.rpm

# 设置开机自动启动
systemctl enable grafana-server

# 启动
systemctl daemon-reload
systemctl start grafana-server
systemctl status grafana-server

# 访问地址：http://10.16.226.51:3000
```

## 2.2 配置

### 2.2.1 数据源

- 访问首页 http://10.16.226.51:3000，点击新增 Data Source；
- 选择 Prometheus，在 URL 字段中输入 http://10.16.226.51:9090，点击”Save & test“，显示绿色则成功。

### 2.2.2 Linux 主机

- 点击左侧的加号，点击 Import 菜单，在 Import via grafana.com 下面的文本框中输入 **22**，点击 Load，保存。

### 2.2.3 Spring Boot 微服务

- 点击左侧的加号，点击 Import 菜单，在 Import via grafana.com 下面的文本框中输入 **6756**，点击 Load，保存。
- Dashboard Setting -> Variables，选择相应的变量进行修改：
  - application：label_values(application)
  - instance：label_values(jvm_memory_used_bytes{application="$application"},instance)

### 2.2.4 Redis

- 点击左侧的加号，点击 Import 菜单，在 Import via grafana.com 下面的文本框中输入 **763**，点击 Load，保存。

### 2.2.5 Zookeeper

- 点击左侧的加号，点击 Import 菜单，在 Import via grafana.com 下面的文本框中输入 **10465**，点击 Load，保存。

### 2.2.6 RabbitMQ

- 点击左侧的加号，点击 Import 菜单，在 Import via grafana.com 下面的文本框中输入 **10991**，点击 Load，保存。

### 2.2.7 Elasticsearch

- 点击左侧的加号，点击 Import 菜单，在 Import via grafana.com 下面的文本框中输入 **2322**，点击 Load，保存。

```sh
 --prefix=/data/nginx --sbin-path=/data/nginx/sbin/nginx --conf-path=/etc/nginx/nginx.conf --error-log-path=/data/nginx/logs/error.log --pid-path=/var/run/nginx/nginx.pid --lock-path=/var/lock/nginx.lock --user=nginx --group=nginx --with-http_ssl_module --with-http_flv_module --with-http_gzip_static_module --http-log-path=/data/nginx/logs/access.log --http-client-body-temp-path=/media/disk1/nginx/client --http-proxy-temp-path=/media/disk1/nginx/proxy --http-fastcgi-temp-path=/media/disk1/nginx/fcgi --with-http_stub_status_module --with-poll_module --with-http_realip_module --with-http_image_filter_module --add-module=/data/ngx_cache_purge-2.3 --add-module=/data/fastdfs-nginx-module/src --with-cc-opt=-Wno-error

```

